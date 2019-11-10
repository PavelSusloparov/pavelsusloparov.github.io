---
layout: post
title:  "DataDog metrics with Spring Boot and Micrometer"
date:   2019-11-10 14:20:36 -0500
categories: programming
---

Monitoring is crucial when it comes to application support. In this post, I would show how to manage to send metrics to
DataDog and keep the way consistent between multiple applications within the organization.

I fun of the Spring Boot framework and use Kotlin as a programming language. The library which is responsible for sending
interface is `io.micrometer:micrometer-core:<version>`. In my case version is 1.1.7

The idea is to provide a unified approach so people easily can understand while switching between multiple applications.
The easiest way is to create a shared library and store the jar in a place where other application can get access to.

```kotlin
interface MetricSenderInterface {

    fun recordTimer(duration: Duration, vararg tags: String?)

    fun sendGaugeUpdate(updatedValue: Number, vararg tags: String?)

    fun incrementCounter(vararg tags: String?)

    fun recordDistribution(value: Double, vararg tags: String?)
}

/**
 * Section interface for Enum
 *
 * Usage:
 * enum class MetricSections(override val value: String):
    MetricSectionsInterface {
 *   COMPONENT_NAME_1("component_name_1"),
 *   COMPONENT_NAME_2("component_name_2");
 * }
 *
 */
interface MetricSectionsInterface { val value: String }

/**
 * Target interface for Enum
 *
 * enum class MetricTargets(override val value: String):
    MetricTargetsInterface {
 *   OBJECT_NAME_1("object_name_1"),
 *   OBJECT_NAME_2("object_name_2");
 * }
 *
 */
interface MetricTargetsInterface { val value: String }

/**
 * Actions interface for Enum
 *
 * enum class MetricActions(override val value: String):
    MetricActionsInterface {
 *   CREATED("created"),
 *   FETCHED("fetched"),
 *   UPDATED("updated");
 * }
 *
 */
interface MetricActionsInterface { val value: String }

/**
 * Actions interface for Enum
 *
 * enum class MetricResults(override val value: String):
    MetricResultsInterface {
 *   SUCCESS("success"),
 *   FAIL("fail");
 * }
 *
 */
interface MetricResultsInterface { val value: String }

fun String.normalize() = this.replace("-", "_").replace(" ", "_").toLowerCase()

/**
 * Micrometer has a bug in tags creation function
 * and does not handle nullable strings
 * or empty strings well for tags,
 * so we are adding that logic here
 */
fun tagValidator(vararg tags: String?) {
    if (tags.any { it.isNullOrEmpty() }) {
        throw Exception("Invalid tags: ${tags.contentToString()}")
    }
    require(tags.size % 2 != 1) { 
        "Tag size must be even, it is a set of key=value pairs: ${tags.contentToString()}"
    }
}

/**
 * Transform multiple key/values strings to a list of tags
 */
fun createTagsList(vararg tags: String?): MutableList<Tag> {
    val tagsList: MutableList<Tag> = mutableListOf()
    var i = 0
    while (i < tags.size) {
        val t = Tag.of(tags[i]!!, tags[i + 1]!!)
        i += 2
        tagsList.add(t)
    }
    return tagsList
}

/**
 * Usage:
 *
 * var metricSender = MetricSender(MetricSections.COMPONENT_NAME_1,
                                   MetricTargets.OBJECT_NAME_1,
                                   MetricActions.CREATED,
                                   MetricResults.SUCCESS)
 * metricSender.incrementCounter("status_code", HttpStatus.OK.toString(),
                                 "url", "/test", "method", "POST")
 *
 */
class MetricSender(
        section: MetricSectionsInterface,
        target: MetricTargetsInterface,
        action: MetricActionsInterface,
        result: MetricResultsInterface,
        applicationName: String = "application"
): MetricSenderInterface {

    val metricName = getMetricName(applicationName,
                                   section,
                                   target,
                                   action,
                                   result)

    private fun getMetricName(applicationName: String,
                              section: MetricSectionsInterface,
                              target: MetricTargetsInterface,
                              action: MetricActionsInterface,
                              result: MetricResultsInterface) = 
        "$applicationName.${section.value}.${target.value}.${action.value}.${result.value}".normalize()
    }

    ...

    /**
     * Transform static method to instance method for testing
     */
    fun metricsCounter(metricName: String, vararg tags: String?) {
        Metrics.counter(metricName, createTagsList(*tags)).increment()
    }

    /**
     * Tracks a monotonically increasing value.
     */
    override fun incrementCounter(vararg tags: String?) {
        try {
            tagValidator(*tags)
            logger.info("incrementCounter :: name: $metricName,
                        tags: ${tags.contentToString()}")
            metricsCounter("$metricName.counter", *tags)
        } catch (e: Exception) {
            logger.warn("incrementCounter :: name: $metricName, exception: $e")
        }
    }
    
    ...
}
```

* By using an enum as an input, you can restrict client applications of creating free form metrics.
* By providing clear documentation for the shared library, you can show an example for client applications the best way of
utilizing it.

**Note** that you can extend the interface of MetricSender with any other micrometer functionality,
since it is a simple wrapper on top of the library.

With the class above, we can start using it. However, there is a better way to provide the interface with Spring Boot.
The objective is to avoid specifying applicationName during every class initialization
and take advantage of dependency injection for testing the fact of sending the metric on the client-side.

For that, we create a MetricSender factory class.

```kotlin
/**
 * Usage:
 *
 * metricSender
 * .configure(applicationName,
              MetricSections.COMPONENT_NAME_1,
              MetricTargets.OBJECT_NAME_1,
              MetricActions.CREATED,
              MetricResults.SUCCESS)
 * .incrementCounter("status_code", HttpStatus.OK.toString(),
                     "url", "/test", "method", "POST")
 */
class Metrics(private val applicationName: String) {

    fun configure(section: MetricSectionsInterface,
                  target: MetricTargetsInterface,
                  action: MetricActionsInterface,
                  result: MetricResultsInterface)
            = MetricSender(section, target, action, result, this.applicationName)
}
```

And provide the bean for client to use

```kotlin
@Configuration
class MetricsConfig {

    /**
     * Usage:
     *
     * metrics
     * .configure(MetricSections.COMPONENT_NAME_1,
                  MetricTargets.OBJECT_NAME_1,
                  MetricActions.CREATED,
                  MetricResults.SUCCESS)
     * .incrementCounter("status_code", HttpStatus.OK.toString(),
                         "url", "/test", "method", "POST")
     */
    @Bean
    fun metrics(@Value("\${spring.application.name}")
                applicationName: String): Metrics {
        return Metrics(applicationName)
    }
}
```

The client applications should import the bean and define enums, which are specific for there.

```kotlin
@Import(MetricsConfig::class)
class ClientApplication
```

The each client application must have the log aggregator configuration in application.yml
```yaml
management:
  metrics:
    export:
      statsd:
        enabled: true
        flavor: datadog
        host: X.X.X.X
```

The recommended approach allows you to start DataDog dashboards and alerts **consistently**
with your metrics across all of Spring Boot applications in all environments.




