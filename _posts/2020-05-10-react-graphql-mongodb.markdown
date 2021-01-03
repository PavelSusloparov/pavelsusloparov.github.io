---
layout: post
title: "JavaScript - React - GraphQL - MongoDb"
date:   2020-05-10 20:25:36 -0500
categories: javaScript graphQL mongoDb
postId: 10
---

# React app

I wrote a small full-stack sample on JavaScript, which includes
* Frontend React based component
* Backend GraphQL based component
* MongoDB for storing the state

Source code is here - [https://github.com/PavelSusloparov/friends-registry](https://github.com/PavelSusloparov/friends-registry)

The app allows you to add a friend by providing a first and last name and shows the list of current friends.

## In depths

### MongoDB

MongoDB is a non-relational database, which has a fast bootstrap as a docker container and a free client.
Perfect for sample projects, when a document contains all required information.
I like to run sample databases in a docker container by creating a docker-compose configuration,
so it is easy to wipe the data and start from scratch.

Find more information about bootstrap and connection to the database in the repo link above.

### Server

GraphQL server is Apollo server - [https://www.apollographql.com/docs/apollo-server/](https://www.apollographql.com/docs/apollo-server/)
It positions itself as a gateway to combine multiple APIs and provide a clean interface to the frontend component.
I combined the ORM layer with the Apollo server for the sake and simplicity of the demo project.
In the production project, I recommend creating a separate backend service with the ORM layer and calling the backend service from the gateway component through an interface.

Apollo gateway provides a neat browser console with capabilities to execute GraphQL queries. See the project's documentation for more details.
The gateway provides three types of interfaces:
* Query
* Mutation
* Subscription

Queries fetch data; mutations modify data; subscription emits events on data modification with capabilities to react on client-side.

### Client

The client is a React-based application, created using create-react-app bootstrap repo and showcase client connection between React and Apollo GraphQL.
When a user adds a new friend, the client makes a GraphQL mutation call for the creation of a new friend.
Since I know the new friend at the moment of the update, I can populate the list of friends with a fake id and refresh the id created by the database on the page refresh.
Also, the client subscribes to new friends creation events during the application bootstrap, which allows updating the friend list modified by another client without refreshing the web page.
Only one way to showcase the subscription functionality is to call GraphQL to create friend mutation through the GraphQL console or insert a new friend into the database with MongoDB client.
In this case, the new friend will appear on the friend's list without the web page refresh.
I found the Apollo GraphQL client is much easier for understanding and use for simple use cases comparing with the Redux library.

Thank you for reading!


 
 






