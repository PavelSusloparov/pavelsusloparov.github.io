---
layout: post
title:  "Evolvability of architectures styles"
date:   2019-11-22 17:35:36 -0500
categories: architecture
---

## Architecture classification

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
| [Brokers](https://en.wikipedia.org/wiki/Broker_pattern) | Easy for component. Medium for application | Easy with proper Continuous Integration | Easy. Difficult to test e2e scenarios | Loosely |
| [Mediators](https://en.wikipedia.org/wiki/Mediator_pattern)	| Easy for component. Medium for application | Easy. Normal for testing e2e scenarios | Normal |
| **Service Oriented** | 					
| [Enterprise Service Bus driven](https://en.wikipedia.org/wiki/Enterprise_service_bus) | Difficult | Difficult | Difficult| Normal |
| [Service-oriented](https://en.wikipedia.org/wiki/Service-oriented_architecture)	| Normal | Normal | Normal | Normal |
| [Micro Services](https://en.wikipedia.org/wiki/Microservices) | Easy | Easy with proper Continuous Integration | Easy | Loosely |
| **Server-less**					
| [Lambda functions](https://en.wikipedia.org/wiki/Lambda_architecture) | Easy | Easy | Easy | Loosely |

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

