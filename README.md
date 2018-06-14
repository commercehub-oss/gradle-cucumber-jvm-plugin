# Gradle Cucumber-JVM Plugin

[![Build Status](https://travis-ci.org/commercehub-oss/gradle-cucumber-jvm-plugin.png?branch=master)](https://travis-ci.org/commercehub-oss/gradle-cucumber-jvm-plugin)

The gradle cucumber-jvm plugin provides the ability to run [cucumber](http://cukes.info) acceptance tests directly
from a gradle build.  The plugin utilizes the cucumber cli provided by the [cucumber-jvm](https://github.com/cucumber/cucumber-jvm)
project, while imposing a few constraints to encourage adopters to use cucumber in a gradle friendly manner. Some of
constraints include:

* A Cucumber test suite should be contained in a single source set.
* Glue code should be compiled by gradle and use annotations to glue to the features.
* Features should be in the resources folder of the source set representing the test suite.
* This plugin generates Cucumber reports based on [masterthought's Cucumber reporting project](https://github.com/masterthought/cucumber-reporting).

The inspiration for this plugin drew heavily from the work of
[Samuel Brown's Cucumber Plugin](https://github.com/samueltbrown/gradle-cucumber-plugin) and
[Camilo Ribeiro's Cucumber Gradle Parallel Example](https://github.com/camiloribeiro/cucumber-gradle-parallel).

> See [our security policy](SECURITY.md) for handling of security-related matters.

## Contributors

 * [Jay St.Gelais](http://github.com/JayStGelais)

## Using the plugin in your gradle build script

The following gradle configuration will create a new Cucumber based test suite named `cucumberTest` and configure it
to run up to 3 parallel forks. The `cucumberTest` source set will depend on the project's main source set.

```groovy
buildscript {
    repositories {
        maven {
            url "http://repo.bodar.com"
        }
        maven {
          url "https://plugins.gradle.org/m2/"
        }
    }
    dependencies {
        classpath "com.commercehub:gradle-cucumber-jvm-plugin:0.8"
    }
}
plugins {
    id 'java'
    id 'com.commercehub.cucumber-jvm' version '0.7'
}


addCucumberSuite 'cucumberTest'

cucumber {
    maxParallelForks = 3
}

cucumberTest {
    stepDefinitionRoots = ['cucumber.steps', 'cucumber.hooks'] // default
    systemProperties = [
        'myVar': 'myValue'
    ]
}

repositories {
    jcenter()
}

dependencies {
    cucumberTestCompile 'info.cukes:cucumber-java:1.2.2'
    // To use JUnit assertions in the step definitions:
    cucumberTestCompile 'junit:junit:4.12'
}
```

The corresponding directory layout will look like this:
```
src
├── cucumberTest
│   ├── java
│   │   └── cucumber
│   │       └── steps
│   │           └── DemoSteps.java
│   └── resources
│       └── features
│           └── demo.feature
└── main
    ├── java
    │   └── demopackage
    │       └── Demo.java
    └── resources
```

Running the following command will execute the test suite:

    gradle(w) cucumberTest

### Using the Kotlin DSL

Gradle also offers to use the new Kotlin DSL for build files.
An example `build.gradle.kts` file might look like this:

```kotlin
plugins {
    id("java")
    id("com.commercehub.cucumber-jvm").version("0.13")
}

cucumber {
    suite("cucumberTest")
    maxParallelForks = 3
}

repositories {
    mavenCentral()
}

dependencies {
    add("cucumberTestCompile", "info.cukes:cucumber-java:1.2.2")
    add("cucumberTestCompile", "junit:junit:4.12")
}
```

The main difference is that test suites need to be declared
inside the `cucumber` extension. Also, the dependency syntax
is different: dependencies of configurations need to be added
via the `add()` function.

## Cucumber Task Configuration

Cucumber tasks can be configured at two levels, globally for the project and individually for a test suite. This allows
for projects to contain multiple Cucumber test suites that can differ on some properties while inheriting other
property values form the project defaults. Both levels of configuration make the following settings available:

* `stepDefinitionRoots`: A list of root packages to scan the classpath for glue code. Defaults to `['cucumber.steps', 'cucumber.hooks']`.
* `featureRoots`: A list of root packages to scan the resources folders on the classpath for feature files. Defaults to `['features']`.
* `tags`: A list of tags to identify scenarios to run. Default to an empty list.
* `plugins`: A list of cucumber plugins passed for execution. Defaults to an empty list.
* `isStrict`: A boolean value indicating whether scenarios should be evaluated strictly. Defaults to `false`.
* `snippits`: Indicator to cucumber on what style to use for generated step examples. Valid values include `camelcase`, `underscore`. Defaults to `camelcase`.
* `maxParallelForks`: Maximum number of forked Java processes to start to run tests in parallel. Defaults to `1`.
* `jvmArgs`: List of custom jvm arguments to pass to test execution
* `systemProperties`: Map of properties to values (String → String) to pass to the forked test running JVMs as Java system properties.
* `suite("someName")`: Method to register a new suite. This can be used as an alternative to calling `addCucumberSuite()`. If you use the Kotlin DSL it is the only way to register a suite.

### Reporting

By default, this plugin will generate reports based on [masterthought's Cucumber reporting project](https://github.com/masterthought/cucumber-reporting).

JUnit reporting can be enabled by setting the `junitReport` property to `true`.
