---
title: ‌GraphQL server with Go introduction
published: false
description: Introduction to Building GraphQL apis with golang and gqlgen.
tags: graphql, go, api, gqlgen
---
## Table Of Contents
* [Motivation](#motivation)
* [Getting started](#getting-started)
* [Queries](#queries)
* [Mutations](#mutations)
* [Authentication](#authentication)
* [Error handling](#error-handling)
* [Filtering](#filtering)
* [Pagination](#pagination)
* [Summary](#summary)

### Motivation <a name="motivation"></a>
[**Go**](https://golang.org/) is a modern general purpose programming language designed by google; best known for it's simplicity, concurrency and fast performance. It's being used by big players in the industry like Google, Docker, Lyft and Uber. If you are new to golang you can start from [golang tour](https://tour.golang.org/) to learn fundamentals.

[**gqlgen**](https://gqlgen.com/) is a library for creating GraphQL applications in Go.


In this tutorial we create a Hackernews clone with GraphQL API with `golang` and `gqlgen` and learn GraphQL fundamentals along the way.

#### What is a GraphQL server?
GraphQL is a query language for API so you can send queries and ask what you need and get exactly that.
sample query:
```
```
sample response:
```
```

#### Schema-Driven Development
In GraphQL your API starts with a schema that defines all your types, queries and mutations, It helps others to understand your API so all of your team can understand how to work with your API, a contract between server and the client.
Whenever you need to add a new capability to a GraphQL API you must redefine schema file and then start implementation. GraphQL has it's [Schema Definition Language](http://graphql.org/learn/schema/) for this.
gqlgen library has a nice feature and generate code based on your schema definition.

### Getting started <a name="getting-started"></a>
In this tutorial we are going to create a Hackernews clone with Go and gqlgen, So our API will be able to handle registration, authentication, submitting links and returning list of links.

#### Project Setup
Create a directory for project and initialize go module:


after that use gqlgen init command to setup a gqlgen project.
go run github.com/99designs/gqlgen init
Take a look at structure of gqlgen project.
Project schema is inside schema.graphql file with some default value let's start with editing schema with features we need for our API.
```
schema
```
then run the command below to regenerate models.go with new schema.
go run github.com/99designs/gqlgen -v

