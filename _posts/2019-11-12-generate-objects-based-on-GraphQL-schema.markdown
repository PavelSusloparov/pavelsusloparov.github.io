---
layout: post
title:  "Generate objects based on GraphQL schema"
date:   2019-11-12 15:50:36 -0500
categories: graphQL automation
---

This the post for you if you use microservices architecture and use GraphQL for your API.
I'm sharing two tips on how to keep GraphQL contracts up-to-date between server and client.

### Tip #1 is 'Generate your server side model'

Save time with POJO object generation with
* [https://github.com/kobylynskyi/graphql-java-codegen](https://github.com/kobylynskyi/graphql-java-codegen)
* [https://github.com/kobylynskyi/graphql-java-codegen-gradle-plugin](https://github.com/kobylynskyi/graphql-java-codegen-gradle-plugin)

The plugin allows generating Java model classes based on the GraphQL schema.
In my example for the server-side, I use Kotlin as a programming language and Spring Boot as a framework with GraphQl
HTTP API

In your build.gradle.kts
* register the task with parameters
* include generated files to the 'main' source set
* run the job as part of the build

```kotlin
tasks.register<GraphqlCodegenGradleTask>("graphqlCodegenService") {
    graphqlSchemaPaths = listOf("$projectDir/src/main/resources/graphql/schema.graphqls")
    outputDir = File("$buildDir/generated/outputDir")
    packageName = "com.example.service"
    customTypesMapping = mutableMapOf(Pair("EpochMillis", "java.time.LocalDateTime"))
    customAnnotationsMapping = mutableMapOf(Pair("EpochMillis", "com.fasterxml.jackson.databind.annotation" +
        ".JsonDeserialize(using = com.example.service.EpochMillisScalarDeserializer.class)"))
    modelValidationAnnotation = "javax.validation.constraints.NotNull"
}

sourceSets {
    getByName("main").java.srcDirs("$buildDir/generated/outputDir")
}

val graphqlCodegenService: DefaultTask by tasks
tasks.withType<KotlinCompile<KotlinJvmOptions>> {
    dependsOn(graphqlCodegenService)
    kotlinOptions {
        freeCompilerArgs = listOf("-Xjsr305=strict")
        jvmTarget = "1.8"
    }
}
```

The build command generates classes in your build directory, and they are available for import

### Tip #2 is 'Generate your client side model'

For the client-side, we want to generate classes based on the server-side schema and client-side GraphQL queries
For that, I use [apollo codegen](https://github.com/apollographql/apollo-tooling#apollo-clientcodegen-output)

```bash
npx apollo client:download-schema \
    --endpoint=http://localhost:8104/graphql \
    src/client/generated/graphql/generatedSchema.json
npx apollo client:codegen \
    --target=typescript \
    --localSchemaFile=src/client/generated/graphql/generatedSchema.json \
    --includes='./src/**/*.{ts,tsx}' \
    --addTypename \
    --tagName=gql \
    --globalTypesFile=src/client/generated/graphql/graphqlGlobalTypes.ts \
    --outputFlat \
    src/client/generated/graphql/generatedTypes.ts
```

The first command fetches the schema and outputs it to generatedSchema.json
The second command generates typescript types and interfaces based on generatedSchema.json and outputs them to two files:
* Types go to graphqlGlobalTypes.ts
* Interfaces go to generatedTypes.ts

The server needs objects for whole GraphQL schema
The client needs objects based on server GraphQL schema and client GraphQL queries.
