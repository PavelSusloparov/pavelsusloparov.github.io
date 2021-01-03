---
layout: post
title:  "Custom source set for Gradle project"
date:   2019-11-17 18:32:36 -0500
categories: kotlin test source-sets
postId: 5
---

`Source set` is a package, which has it is own compilation and runtime configuration as well as dependencies.
IntelijIdea project builder creates `main` and `test` source sets for a Gradle project module by default.

My two reasons why you want to create a custom source set:

* You want to have separate your integration and e2e tests out of `test` source set.
* You want to separate dependencies between packages within the same module.

## Preparation

To see the example in code, check [sudoku-workshop](https://github.com/PavelSusloparov/sudoku-workshop)

### File structure

* Create a new folder under /src. An example case is sudoku-book/testUtil
* Keep a similar structure as your primary source set.

For kotlin:

```bash
src/kotlin
src/resources
```

Namespace stays the same as in `main`
```bash
src/kotlin/com/workshop/sudokubook
```

### Gradle configuration

* Update build.gradle.kts configuration with `contractTest` source set configuration

```kotlin
sourceSets.create("contractTest") {
	withConvention(KotlinSourceSet::class) {
		kotlin.srcDirs("src/contractTest/kotlin")
		resources.srcDirs("src/contractTest/resources")
		compileClasspath += sourceSets["main"].output + sourceSets["testUtil"].output
		runtimeClasspath += sourceSets["main"].output + sourceSets["testUtil"].output
	}
}

configurations.getByName("contractTestCompile").extendsFrom(configurations["compile"], configurations["testUtilCompile"])
configurations.getByName("contractTestRuntime").extendsFrom(configurations["runtime"], configurations["testUtilRuntime"])

val contractTestCompile = configurations.getByName("contractTestCompile")
val contractTestRuntime = configurations.getByName("contractTestRuntime")

tasks.register<Test>("contractTest") {
	description = "Runs Cucumber contract tests."
	group = "verification"
	testClassesDirs = sourceSets["contractTest"].output.classesDirs
	classpath = sourceSets["contractTest"].runtimeClasspath
	outputs.upToDateWhen { false }

	testLogging {
		events = setOf(TestLogEvent.SKIPPED, TestLogEvent.FAILED)
		showStandardStreams = true
	}
	systemProperty("cucumber.options", System.getProperty("scenario"))
}
```

compileClasspath and runtimeClasspath have multiple dependencies on `main` and `testUtil` source sets. It means that
classes defined in these spaces available for import in `contractTest`

New registered test task `contractTest` has testClassesDirs and classpath properties, which reference on contractTest
source set.

### Dependencies

```kotlin
dependencies {
    implementation(kotlin("stdlib-jdk8"))
    ...
    compile(kotlin("script-runtime"))
    ...
    testImplementation("org.junit.platform:junit-platform-console:1.4.0")
    ...
    componentTestCompile(platform(kotlin("bom", version = "1.3.21")))
    ...
    testUtilCompile(platform(kotlin("bom", version = "1.3.21")))
    ...
}
```

* `implementation` keyword stands for `main` source set compilation and runtime dependencies.

* `compile` keyword stands for `main` source set compilation dependencies only.

* `runtime` keyword stands for `main` source set runtime dependencies only.

Custom source set always extends `main` configuration,
so it has a similar three types of dependencies with your registered keywords.

Try to use source sets on your project, and I personally found them useful.
