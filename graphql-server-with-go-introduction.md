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
```bash
go mod init github.com/[username]/hackernews
````

after that use gqlgen init command to setup a gqlgen project.
```go
go run github.com/99designs/gqlgen init
```
Here is a description from gqlgen about the generated files:
* gqlgen.yml — The gqlgen config file, knobs for controlling the generated code.
* generated.go — The GraphQL execution runtime, the bulk of the generated code.
* models_gen.go — Generated models required to build the graph. Often you will override these with your own models. Still very useful for input types.
* resolver.go — This is where your application code lives. generated.go will call into this to get the data the user has requested.
* server/server.go — This is a minimal entry point that sets up an http.Handler to the generated GraphQL server.
run the code with `go run server.go` and open your browser and you should see the graphql playground, So setup is right!

Now let's start with defining schema file with features we need for our API. We have two types Link and User each of them for representing Link and User to client, a `links` Query to return list of Links. a input for creating NewLink and mutation for creating link and returns created link. then run the command below to regenerate models.
```
type Link {
  id: ID!
  title: String!
  address: String!
  user: User!
}

type User {
  id: ID!
  name: String!
}

type Query {
  links: [Link!]!
}

input NewLink {
  title: String!
  address: String!
  userId: ID!
}

type Mutation {
  createLink(input: NewLink!): Link!
}
```
Now run the command to regenerate files;
```go
go run github.com/99designs/gqlgen
```
After gqlgen generated code for us with have to implement our schema, we do that in ‍‍‍‍`resolver.go`, as you see there is functions for Queries and Mutations we defined in our schema.

##### Setup database
Before we jump into implementing GraphQL schema we need to setup database to save users and links, This is not supposed to be tutorial about databases in go but here is what we are going to do:
* setup MySQL and create table
* define our models and create migrations

###### Setup MySQL
If you have docker you can run [Mysql image]((https://hub.docker.com/_/mysql)) from docker and use it.
`docker run --name mysql -e MYSQL_ROOT_PASSWORD=secret -d mysql:latest`
now run `docker ps` and you should see our mysql image is running:
```
CONTAINER ID        IMAGE                                                               COMMAND                  CREATED             STATUS              PORTS                  NAMES
8fea71529bb2        mysql:latest                                                        "docker-entrypoint.s…"   2 hours ago         Up 2 hours          3306/tcp, 33060/tcp    mysql

```

Now create a database for our application:
```bash
docker exec -it mysql bash
mysql -u root -p
CREATE DATABASE hackernews;
```

###### Models and migrations
We need to create migrations for our app so every time our app runs it creates tables it needs to work properly, we are going to use [golang-migrate](https://github.com/golang-migrate/migrate) package.
create a folder structure for our database files:
```
go-graphql-hackernews
--internals
----db
------migrations
------mysql
```
Install go mysql driver and golang-migrate packages then create migrations:
```
go get -u github.com/go-sql-driver/mysql
go build -tags 'mysql' -ldflags="-X main.Version=$(git describe --tags)" -o $GOPATH/bin/migrate github.com/golang-migrate/migrate/cmd/migrate
migrate create -ext sql -dir migrations -seq create_users_table
migrate create -ext sql -dir migrations -seq create_links_table
```
migrate command will create two files for each migration ending with .up and .down; up is responsible for applying migration and down is responsible for reversing it.
open create_users_table.up.sql and add table for our users:
```sql
CREATE TABLE IF NOT EXISTS Users(
    ID INT NOT NULL UNIQUE,
    Username VARCHAR (127) NOT NULL UNIQUE,
    Password VARCHAR (127) NOT NULL,
    PRIMARY KEY (ID)
)
```
in create_links_table.up.sql:
```sql
CREATE TABLE IF NOT EXISTS Links(
    ID INT NOT NULL UNIQUE,
    Title VARCHAR (255) ,
    Address VARCHAR (255) ,
    UserID INT ,
    FOREIGN KEY (UserID) REFERENCES Users(ID) ,
    PRIMARY KEY (ID)
)
```

We need one table for saving links and one table for saving users, Then we apply these to our database using migrate command.

```bash
migrate -database mysql://root:secret@(172.17.0.2:3306)/hackernews -path internal/db/migrations up
```

