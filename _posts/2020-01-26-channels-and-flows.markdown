---
layout: post
title: "Kotlin coroutines. Channels and Flows"
date:   2019-01-26 13:35:36 -0500
categories: Kotlin, Coroutines, Channels, Flows
---

# Channels and Flows

The structure of your application is defined by how data flows through it.
Do various components communicate directly?
Do sources of data expose event listeners so that interested components may subscribe to changes?
Does your data consistently flow in one direction?

Whatever your strategy may be, intentionality is critical.
Channels and flows represent streams of data that you can subscribe to, and they achieve this in different ways.
Let's look at Kotlin's built-in stream support and show how to send data throughout your application.

## Channels

### Channels Basic

Channels are a mechanism for communicating between coroutines. This Channel does nothing on its own - you need to send some data to it.
This quality of channels is called `Hot.` Channels are `Hot` because a coroutine publishes to the Channel without the receiver needed.
 
In the [NumberPrinter.kt](https://github.com/PavelSusloparov/kotlin-channel-flows/blob/master/channels-and-flows/src/main/kotlin/NumberPrinter.kt)
I launch coroutines to send numbers from 0 to 9 to a channel. Meanwhile, another coroutine listens to the Channel and print numbers to the standard output.
`runBlocking` operator helps synchronize coroutines.

Let's transform the example and separate each of the `launch` body to be a separate `suspend` function.
This type of function allows us to use other `suspend` functions inside and invoke function within the `launch` block.
The updated version [NumberPrinterV2.kt](https://github.com/PavelSusloparov/kotlin-channel-flows/blob/master/channels-and-flows/src/main/kotlin/NumberPrinterV2.kt)

### Channels application

Let's create a use case for the channels with the publisher/subscriber model.
I use an application based on [Ktor](https://ktor.io/) server, which provides an interface for fetching imaginary character information.
By calling [http://127.0.0.1:8080/](http://127.0.0.1:8080/), I can fetch one character at a time.
Implementation of the server routes in [Application.kt](https://github.com/PavelSusloparov/kotlin-channel-flows/blob/master/character-data-api/src/Application.kt).
Characters parameters in [Character.kt](https://github.com/PavelSusloparov/kotlin-channel-flows/blob/master/character-data-api/src/Character.kt).

**Note:**: Run the server by applying `./gradlew run`

Let's write a process, which broadcast characters and prints characters multiple times using non-blocking operations.
Working example is [CharacterFetcher.kt](https://github.com/PavelSusloparov/kotlin-channel-flows/blob/master/channels-and-flows/src/main/kotlin/CharacterFetcher.kt)
I use the `publish` method to broadcast characters in an infinitive while loop. In the main method, we receive ten characters and close the Channel.
Other subscribers won't be able to receive characters from the Channel after this point.

## Flows

### Flows Basics

Flows are Kotlin's way to represent asynchronous, cold streams of data. They are similar to sequences, but they can run asynchronously with a coroutine.

The flow example in [Flows.kt](https://github.com/PavelSusloparov/kotlin-channel-flows/blob/master/channels-and-flows/src/main/kotlin/Flows.kt)
This sequence takes the first one thousand prime numbers and emits them in a synchronous stream.
The advantage of using a sequence instead of a more traditional list for this prime number example is that sequences are unbounded.
You do not have to guess how many numbers it will take to find the first one thousand primes because sequences can emit a potentially infinite number of values.
The downside of sequences is that they operate synchronously.
If you had a more complex mathematical operation or were running in a resource-constrained environment, then the need for asynchronous execution becomes more pressing.
Flows provide a non-blocking solution to data streaming that may feel familiar to you if you have used a reactive streams protocol like RxJava.

In the example, I transform a sequence to flow and collect and print Fibonacci filtered numbers.
In contrary to Channels, Flows are `Cold.` It means that the sender starts to send to emit messages when the sender starts to collect them.

In the example, I generate the Flow with a flow builder.
Another option to generate a flow is the `flow` operator. In the example [FlowsV2.kt](https://github.com/PavelSusloparov/kotlin-channel-flows/blob/master/channels-and-flows/src/main/kotlin/FlowsV2.kt),
I emit messages in a while loop as a flow.

### Flows operators

There are multiple flow operators:
* filter - filter values by given parameter
* take - take several elements from the Flow
* collect - invoke the Flow and receive items from the Flow. It is a terminal operator.

Because flows are coroutine-based, any time that you bridge the gap between the synchronous and asynchronous world, you will use a terminal operator.

There is an option to create your operators by creating an extension function for a flow as, [FlowsV3.kt](https://github.com/PavelSusloparov/kotlin-channel-flows/blob/master/channels-and-flows/src/main/kotlin/FlowsV3.kt).
`filterIfBelowFreezing` is an extension function to `Flow<Int>.`

As a next step, I replace `filter` with `transform` and `emit`, which becomes a flow transformation step [FlowsV4.kt](https://github.com/PavelSusloparov/kotlin-channel-flows/blob/master/channels-and-flows/src/main/kotlin/FlowsV4.kt).
`filterIfBelowFreezing` is an example of an operator that is simple enough that basing its definition off of the existing filter operator is sensible.
For a more complex or custom operator, starting from scratch with the transform operator can be the right approach.
Note that as the take operator, the transform operator is also an experimental part of the coroutines API.

In the [FlowsV5.kt](https://github.com/PavelSusloparov/kotlin-channel-flows/blob/master/channels-and-flows/src/main/kotlin/FlowsV5.kt),
`filterIfBelowFreezing` can work with multiple temperature unit as an argument.
The flow builder approach is hugely flexible for building your custom pipeline of events.

## Learn more

Learn more about [coroutines](https://kotlinlang.org/docs/reference/coroutines/basics.html),
[channels](https://kotlinlang.org/docs/reference/coroutines/channels.html),
[flows](https://kotlinlang.org/docs/reference/coroutines/flow.html). 

