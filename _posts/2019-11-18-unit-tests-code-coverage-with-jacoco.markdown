---
layout: post
title:  "Unit tests code coverage with Jacoco"
date:   2019-11-18 9:49:36 -0500
categories: programming, gradle, jacoco, code coverage
---

Code coverage is a code quality metric.
It helps to monitor the application quality trend by storing the metric value over time.

I feel skeptical about having 100% coverage:
* Instead of writing useful unit tests, developers chasing reaching the threshold coverage number
* it brings unnecessary coverage for setters and getters

My team found a compromise for 75% code coverage as a quality gate.
It gives us the power to move forward with features delivery in a reasonable time.

Jacoco plugin gives ability to produce a metric based on JUnit test run.

More about plugin in [https://www.jacoco.org/jacoco/trunk/index.html](https://www.jacoco.org/jacoco/trunk/index.html)

Make the task run as part of the application test nad you can track the coverage locally.
The next step is to monitor the metric over time. For that, you can use CI pipeline to publish reports in SonarQube,
so you can see how product and engineering decisions affect the quality of your applications.

## Configuration setup

To see the example in code, check [sudoku-workshop](https://github.com/PavelSusloparov/sudoku-workshop)

build.gradle.kts configuration

```kotlin
plugins {
	// Code Coverage plugin
	jacoco
}

// Jacoco Plugin
jacoco {
	toolVersion = "0.8.4"
	reportsDir = file("$buildDir/reports/jacoco")
}

// Jacoco Coverage Report
val jacocoTestReport by tasks.getting(JacocoReport::class) {
	reports {
		html.isEnabled = true // human readable
		xml.isEnabled = true  // required by coveralls
	}
	doLast {
		println("Jacoco tests coverage report: " + project.projectDir.toString() + "/build/reports/jacoco/test/html/index.html")
	}
}

// Jacoco Enforce Code Coverage
val jacocoTestCoverageVerification by tasks.getting(JacocoCoverageVerification::class) {
	violationRules {
		// Every class should be tested
		rule {
			enabled = true
			element = "CLASS"
			includes = listOf("com.example.workshop.*")

			// Coverage for classes. Strive for 75%.
			limit {
				counter = "CLASS"
				value = "COVEREDRATIO"
				minimum = BigDecimal.valueOf(0.75)

				excludes = listOf(
					"com.example.workshop.WorkshopApplication",
					"com.example.workshop.WorkshopApplicationKt",
					//Application
					"com.example.workshop.collections.*",
					"com.example.workshop.configuration.*",
					"com.example.workshop.controllers.*"
				)
			}
		}

		// Coverage on lines of code. Strive for 75%.
		rule {
			enabled = true
			element = "CLASS"
			includes = listOf("com.example.workshop.*")

			limit {
				counter = "LINE"
				value = "COVEREDRATIO"
				minimum = BigDecimal.valueOf(0.75)

				excludes = listOf(
					"com.example.workshop.WorkshopApplication",
					"com.example.workshop.WorkshopApplicationKt",
					//Application
					"com.example.workshop.collections.*",
					"com.example.workshop.configuration.*",
					"com.example.workshop.controllers.*"
				)
			}
		}
	}
}

test.finalizedBy(jacocoTestCoverageVerification, jacocoTestReport)

// always run tests before code coverage is collected
jacocoTestReport.dependsOn(test)
jacocoTestCoverageVerification.dependsOn(test)
```

In `jacocoTestCoverageVerification,` there are two rule sections, which allow you to exclude classes from the code coverage verification.

My advice is to be careful with updating lists, and they might grow if you tend to compromise the quality over the speed of delivery.
Use code review process and code owners to prevent the list of growing.
