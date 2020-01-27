---
layout: post
title:  "Dockerize SpringBoot application with JIB"
date:   2019-11-19 9:49:36 -0500
categories: Kotlin, Gradle, Docker, Jib
---

JIB is the plugin which create a docker container from you SpringBoot application.
It publishes the image to container registry of your choice.
Pligin abstracts creation of a docker file and publishing logic.

Find more about JIB in [https://github.com/GoogleContainerTools/jib](https://github.com/GoogleContainerTools/jib)

## Configuration

To see the example in code, check [sudoku-workshop](https://github.com/PavelSusloparov/sudoku-workshop)

```kotlin
plugins {
    id("com.google.cloud.tools.jib") version "1.4.0"
}

jib {
	container {
		ports = listOf("8103")
		mainClass = "com.workshop.sudokubook.WorkshopApplicationKt"

		// good defauls intended for Java 8 (>= 8u191) containers
		jvmFlags = listOf(
			"-server",
			"-Djava.awt.headless=true",
			"-XX:InitialRAMFraction=2",
			"-XX:MinRAMFraction=2",
			"-XX:MaxRAMFraction=2",
			"-XX:+UseG1GC",
			"-XX:MaxGCPauseMillis=100",
			"-XX:+UseStringDeduplication"
		)
	}
}
```

Simple configuration gives you three additional gradle tasks.
```bash
./gradlew tasks
```

```bash
...
Jib tasks
---------
jib - Builds a container image to a registry.
jibBuildTar - Builds a container image to a tarball.
jibDockerBuild - Builds a container image to a Docker daemon.
```

Run `jibDockerBuild` to get the image ready.
I prefer using `docker-compose` to run multiple images together.

Example of the configuration is

```yaml
version: '2'
services:
  db:
    container_name: mysql01
    image: mysql:latest
    ports:
      - "3308:3306"
    env_file:
      - sec.env
    volumes:
      - "./conf.d:/etc/mysql/conf.d:ro"
      - "./docker-entrypoint-initdb.d:/docker-entrypoint-initdb.d"
  sudoku:
    container_name: sudoku
    restart: always
    image: sudoku:0.0.1-SNAPSHOT
    env_file:
      - sec.env
    ports:
      - "8102:8102"
  sudoku-book:
    container_name: sudoku-book
    restart: always
    image: sudoku-book:0.0.1-SNAPSHOT
    env_file:
      - sec.env
    ports:
      - "8103:8103"
    depends_on:
      - db
  start_dependencies:
    image: dadarek/wait-for-dependencies
    depends_on:
      - db
    command: db:3306
```

Two exciting things in the configuration above:

* `dadarek/wait-for-dependencies` image, which gives the ability for `sudoku-book` wait until
MySQL will be up and running.
* Volume configuration for MySQL container contains `docker-entrypoint-initdb.d`.
This is the place to define bootstrap SQL script to seed your database instance with initial data such as custom users, permissions and database schema.

I prefer to wire docker-compose and Gradle in a bash script, which gives a user one button run.
The advantage is abstract application bootstrap and makes it simple.

```bash
#!/usr/bin/env bash

ROOT_DIR=`dirname $0`
cd ${ROOT_DIR}
ROOT_DIR=`pwd`

# Generate image for Sudoku application
cd ${ROOT_DIR}/sudoku
./gradlew jibDockerBuild

cd ${ROOT_DIR}/sudoku-book
# Generate image for Sudoku Book application
./gradlew jibDockerBuild

cd ${ROOT_DIR}
# Start both applications
docker-compose up
```

I always strive to simplify developer life for onboarding new people to the application and encourage you to invest time in proper setup.
