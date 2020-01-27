---
layout: post
title:  "Approach for unit testing"
date:   2019-11-08 10:22:36 -0500
categories: kotlin testing junit5 springboot
---

The quality code must be unit tested. It gives code supportability, regression verification, and live documentation.
Confidence in your code is crucial.

I share two tips to make you better in the art of unit testing.

### Tip #1 is 'Use defined structure.'

I fun of Kotlin and Spring Boot with dependency injection.
It gives simple ability to initialize the class under the test and mock out the dependencies.
Given a class with three dependency injection classes

```kotlin
@Component
class ExampleService(
      val exampleDao: ExampleDao,
      val exampleClient: ExampleClient,
      val exampleUtil: ExampleUtil
) {

    fun saveExampleDetails(id: String): String {

        val exampleDetails = exampleClient.fetchExampleDetails(id)
        val exampleEntity = exampleDao.save(exampleUtil.map(exampleDetails))
        return exampleEntity.id
  }
}
```

The structure of the test is

```kotlin
@ExtendWith(SpringExtension::class)
class ExampleServiceTest {

    private lateinit var exampleClient: ExampleClient
    private lateinit var exampleDao: ExampleDao
    private lateinit var exampleUtil: ExampleUtil

    private lateinit var exampleService: ExampleService

    @BeforeEach
    fun setup() {
        exampleClient = mock()
        exampleDao = mock()
        exampleUtil = mock()

        exampleService = ExampleService(exampleClient, exampleDao, exampleUtil)
    }

    @Nested
    @DisplayName("saveExampleDetails")
    internal inner class SaveExampleDetails {

        private lateinit var exampleDetails: ExampleDetails
        private lateinit var exampleEntity: ExampleEntity
        private lateinit var id: String

        @BeforeEach
        fun setup() {
            id = "id"

            exampleDetails = Fixtures.External.exampleDetails().copy(
                    attribute = "value"
            )
            exampleEntity = Fixtures.Entity.exampleEntity()

            doReturn(exampleDetails).whenever(exampleClient).fetchExampleDetails(id)
            doReturn(exampleEntity).whenever(exampleUtil).map(exampleEntity)
            doReturn(exampleEntity).whenever(exampleDao).save(exampleEntity)
        }

        @Test
        fun `happy path`() {
            exampleService.saveExampleDetails(id).run {
                assertEquals("ids match", id, this)
            }
        }

        @Test
        fun `throws an exception when client throws an exception`() {
            doThrow(RuntimeException()).whenever(exampleClient).fetchExampleDetails(id)

            assertThrows<RuntimeException> {  exampleService.saveExampleDetails(id) }
        }
    }
}
```

Use this template for any class and method under test.
I recommend to save it under the shortcut in your text editor for reference.
I love to use IntelijIdea and it has code snapshot shortcut saving functionality.

The breakdown of the structure is the following:

* Define dependencies above the class under the test
* Initialization happens before each test in the global setup
* Each method under test has it's own nested class
* The nested class defines objects required for the method under the test
* Setup of the nested class contains initialization for happy path test, which includes objects initialization and mock behavior 
* The happy path contains only action and verification
* The negative scenario contains modification for the happy path to support the behavior change, action and verification

### The tip #2 is 'Use Fixtures'.

Fixtures are the library class, which initializes objects for testing.
It gives the ability to keep all test objects in one place and avoid creating bulky helper classes per object.

```kotlin
class Fixtures {

    class Entity {

        companion object {

            fun exampleEntity() = ExampleEntity(
                    id = "id",
                    attribute = "attribute"
            )
        }
    }

    class External {

        companion object {

            fun exampleDetails() = ExampleDetails(
                    id = "id",
                    attribute = "attribute",
                    date = LocalDateTime.now()
            )
        }
    }
}
```

The trick with Fixture methods is that they don't have input variables.
If the object needs to be modified, you can edit it in the place where it is needed - in the test.

```kotlin
exampleDetails = Fixtures.External.exampleDetails().copy(
        attribute = "value")
```

### The tip #3 is 'Test your Dao with in-memory database'.

Have different type testing configuration for Controller, Service and Dao classes
Controller and Service should use mocks and isolate method under the test.
Dao should have access to the in memory instance of database, insert data in setup and clean the data in tear down type of methods.

```kotlin
@DataMongoTest
@Import(MongoConfig::class, RetryConfig::class, ExampleDao::class)
@Nested
@DisplayName("saveExampleDetails")
internal inner class SaveExampleDetails
```

Craft your unit tests, style comes with repetition.