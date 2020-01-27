---
layout: post
title:  "Evolvability of architectures styles"
date:   2019-11-22 17:35:36 -0500
categories: architecture
---

The post describes conventional software architecture approaches and compares them based on the given architecture problem.

## Definitions

* Incremental change for the application means that a developer makes a new feature or fix a bug in the application code.
* Deployment means bundling application code into an image or binary and deploying it to the production server.
* Appropriate coupling implies the level of interaction between components within the application.
* Fitness function represents each requirement for architecture.

### Examples of Fitness Functions

* Application metrics
* Architecture metrics
* Business logic metrics
* Unit tests
* Contract tests
* Integration tests
* Certification for changes
* Compliance certification

### Fitness function (FF) classification

* Atomic *VS* Holistic
    - Atomic FF run against a singular context
    - Holistic FF run against a shared context
* Triggered *VS* Continual
    - Triggered FF run based on a particular event
    - Continual FF execute constant verification of architectural aspect
* Static *VS* Dynamic
    - Static FF have a fixed result, such as binary Pass/Fail. Those are usually metrics or tests.
    - Dynamic FF rely on shifting definition based on extra context.
* Automated *VS* Manual
* Temporal
    - It is usually a time component, which will be removed in future.
* Intentional Over Emergent
    - Intentional FF defines at the beginning of the project
    - Emergent FF defines as project evolves and developers find *Unknown Unknowns*
* Domain-Specific
    - Additional requirements such as stress test or security governance

## Architecture classifications

Since we defined basic definitions, let's look at the known architectures.
The table below designed to help consider architecture for a new project or evolve existing project to desired level based on functional and non functional requirements.

| Architecture Classification | How hard is to make an incremental change? | How hard is to make a deployment? | How hard is to make a guided change with a fitness function? | What is the level of appropriate coupling? |
| :------------: | :----------------------------------------: | :-------------------------------: | :----------------------------------------------------------: | :----------------------------------------: |
| **Chaotic structure** |
| [Big Ball of Mud](https://en.wikipedia.org/wiki/Big_ball_of_mud) | Difficult | Difficult | Impossible | Impossible |
| [Unstructured Monoliths](https://en.wikipedia.org/wiki/Monolithic_application) | Difficult | Difficult | Difficult | Tight |
| **Structured** |
| [Multitier](https://en.wikipedia.org/wiki/Multitier_architecture) | Easy | Difficult | Normal | Normal |
| Modular monolith | Easy | Difficult | Easy | Normal |
| [Micro kernel](https://en.wikipedia.org/wiki/Monolithic_kernel) | Easy | Easy for plugins. Difficult for core | Easy | Normal |
| **Event Driven** |				
| [Brokers](https://en.wikipedia.org/wiki/Broker_pattern) | Easy for component. Medium for application | Easy with proper Continuous Integration | Easy. Difficult to test e2e scenarios | Loose |
| [Mediators](https://en.wikipedia.org/wiki/Mediator_pattern)	| Easy for component. Medium for application | Easy. Normal for testing e2e scenarios | Normal |
| **Service Oriented** | 					
| [Enterprise Service Bus driven](https://en.wikipedia.org/wiki/Enterprise_service_bus) | Difficult | Difficult | Difficult| Normal |
| [Service-oriented](https://en.wikipedia.org/wiki/Service-oriented_architecture)	| Normal | Normal | Normal | Normal |
| [Micro Services](https://en.wikipedia.org/wiki/Microservices) | Easy | Easy with proper Continuous Integration | Easy | Loose |
| **Server-less**					
| [Lambda functions](https://en.wikipedia.org/wiki/Lambda_architecture) | Easy | Easy | Easy | Loose |

### Coupling classification

* Tight coupling gives guaranteed side effects of changes and no breaking interface contracts.
* Normal level of coupling gives possible side effects of changes and potential breaking interface contracts.
* Loose coupling provides low side effects of changes breaking interface contracts.

### Deployment

* With a proper CI, every architecture is easy to deploy.
* The time to make a CI and total deployment time per build differ proportionally to coupling.
* With tight coupling, it takes more time to deploy the application to production.

### Testing

* Unit testing is possible in any architecture approach.
* Integration testing is easier in Structured architecture.
* Contract testing required for Loosely coupled architecture.
* E2E testing is easier for Tight coupled architecture. Also, it is easier for architecture with synchronous components interaction.

