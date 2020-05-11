---
layout: post
title: "React - GraphQL - MongoDb"
date:   2020-05-10 20:25:36 -0500
categories: javaScript graphQL mongoDb
---

# React app

I wrote a small full stack sample on JavaScript, which includes
* Frontend React based component
* Backend GraphQL based component
* MongoDb for storing the state

Source code is here - [https://github.com/PavelSusloparov/friends-registry](https://github.com/PavelSusloparov/friends-registry)

The app allows you to add a friend by providing a first and last name and shows the list of current friends.

## In depths

### MongoDb

Non-relational database, which has a fast bootstrap as a docker container and free client.
Perfect for sample projects, when a document contains all required information.
I like to run sample databases in a docker container by creating a docker-compose configuration,
so it is easy to wipe the data and start from scratch.

Find more information about bootstrap and connection to the database in the repo link above.

### Server

GraphQL server is Apollo server - [https://www.apollographql.com/docs/apollo-server/](https://www.apollographql.com/docs/apollo-server/)
It positions itself as a gateway to combine multiple APIs together and provide a clean interface to the frontend component.
I combined the ORM layer with Apollo server for the sake and simplicity of demo project.
In the production project, I recommend creating a separate backend service with ORM layer and calling the service from gateway component through an interface.

Apollo gateway provides a neat browser console with capabilities to execute GraphQL queries. See project's documentation for more details.
The gateway provides three types of interfaces:
* Query
* Mutation
* Subscription

Queries fetch data, mutations modify data, subscription emit events on data modification with capabilities to react on client side.

### Client

Client is a React based application, created using create-react-app bootstrap repo and showcase client connection between React and Apollo GraphQL.
When user adds a new friend the client makes a GraphQL mutation call for creation a new friend.
Since I know the new friend at the moment of the update, I can populate the list of friends with a fake id and refresh the id created by database on the page refresh.
Also, client subscribes on new friends creation events during the application bootstrap, which allows to update the friend list modified by another client without refreshing the web page.
Only one way to showcase the subscription functionality is to call GraphQL create friend mutation through the GraphQL console or insert a new friend into the database with MongoDb client.
In this case the new friend will appear in the friends list without the web page refresh.
I found the Apollo GraphQL client is much easier for understanding and use for a simple use cases comparing with the Redux library.

Thank you for reading!


 
 






