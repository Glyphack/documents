---
title: ‌GraphQL server with Go introduction
published: false
description: Introduction to Building GraphQL apis with golang and gqlgen.
tags: graphql, go, api, gqlgen
---
## Table Of Contents
- [Table Of Contents](#table-of-contents)
  - [Motivation ](#motivation)
    - [What is a GraphQL server?](#what-is-a-graphql-server)
    - [Schema-Driven Development](#schema-driven-development)
  - [Getting started ](#getting-started)
    - [Project Setup](#project-setup)
  - [Queries](#queries)
    - [What Is A Query](#what-is-a-query)
    - [Simple Query](#simple-query)
  - [Mutations](#mutations)
    - [What Is A Mutation](#what-is-a-mutation)
    - [A Simple Mutation](#a-simple-mutation)
  - [Database](#database)
      - [Setup MySQL](#setup-mysql)
      - [Models and migrations](#models-and-migrations)
  - [Create and Retrieve Links](#create-and-retrieve-links)
    - [CreateLinks](#createlinks)
    - [links Query](#links-query)
  - [Authentication](#authentication)
    - [Setup JWT](#setup-jwt)
      - [Database Table](#database-table)
      - [Authentication Middleware](#authentication-middleware)
    - [CreateUser](#createuser)
    - [Login](#login)
    - [Enhance our](#enhance-our)

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
}

input NewUser {
  username: String!
  password: String!
}

input Login {
  username: String!
  password: String!
}

type Mutation {
  createLink(input: NewLink!): Link!
  createUser(input: NewUser!): User!
  login(input: Login!): String!
}
```
Now run the command to regenerate files;
```go
go run github.com/99designs/gqlgen
```
After gqlgen generated code for us with have to implement our schema, we do that in ‍‍‍‍`resolver.go`, as you see there is functions for Queries and Mutations we defined in our schema.
Now let's see what we got, run app with `go run server/server.go` and go to localhost you should see graphiql page.


### Queries
In the previous chapter we setup up a server that runs graphql, Now we try to implement a Query that we defined in `schema.grpahql`.

#### What Is A Query
a query in graphql is asking for data, you ask for the data type you want and specify what you want from that type and graphql will return it back to you

#### Simple Query
 open `resolver.go` file and take a look at Links function,
```go
func (r *queryResolver) Links(ctx context.Context) ([]*Link, error) {
```
this function returns slice of Links, Let's make a dummy response for this function for now
```go
func (r *queryResolver) Links(ctx context.Context) ([]*Link, error) {
	var links []*Link
	links = append(links, &Link{Title: "our dummy link", Address: "https://address.org", User: &User{Name: "admin"}})
	return links, nil
}
```
now run graphql server and send this query:
```
query {
	links{
    title
    address,
    user{
      name
    }
  }
}
```
And you will get:
```
{
  "data": {
    "links": [
      {
        "title": "our dummy link",
        "address": "https://address.org",
        "user": {
          "name": "admin"
        }
      }
    ]
  }
}
```
Nice! now you know how we generate response for our graphql server. But this response is just a dummy response we want be able to query all other users links, In the next chapter we setup database for our app to be able to save links and retrieve them from database.


### Mutations
#### What Is A Mutation
Simply mutations are just like queries but they can cause a data write, Technically Queries can be used to write data too however it's not suggested to use it.
So mutations are like queries, they have names, parameters and they can return data.
#### A Simple Mutation
Let's try to implement the createLink mutation, since we do not have a database set up yet(we'll get it done in the next chapter) we just receive the link data and construct a link object and send it back for response!
Open `resolver.go` and Look at `CreateLink` function:
```go
func (r *mutationResolver) CreateLink(ctx context.Context, input NewLink) (*Link, error) {
```
This function receives a `NewLink` with type of `input` we defined NewLink structure in our `schema.graphql` try to look at the structure and try Construct a `Link` object that be defined in our `schema.ghraphql`:
```go
func (r *mutationResolver) CreateLink(ctx context.Context, input NewLink) (*Link, error) {
	var link Link
	var user User
	link.Address = input.Address
	link.Title = input.Title
	user.ID = input.UserID
	user.Name = "test"
	link.User = &user
	return &link, nil
}
```
now run server and use the mutation to create a new link:
```
mutation {
  createLink(input: {title: "new link", userId: "1", address:"http://address.org"}){
    title,
    user{
      name
    }
    address
  }
}
```
and you will get:
```
{
  "data": {
    "createLink": {
      "title": "new link",
      "user": {
        "name": "test"
      },
      "address": "http://address.org"
    }
  }
}
```
Nice now we know what are mutations and queries we can setup our database and make these implementations more functional.


### Database
Before we jump into implementing GraphQL schema we need to setup database to save users and links, This is not supposed to be tutorial about databases in go but here is what we are going to do:
* setup MySQL and create table
* define our models and create migrations

##### Setup MySQL
If you have docker you can run [Mysql image]((https://hub.docker.com/_/mysql)) from docker and use it.
`docker run --name mysql -e MYSQL_ROOT_PASSWORD=dbpass -d mysql:latest`
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

##### Models and migrations
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
  migrate -database mysql://root:dbpass@(172.17.0.2:3306)/hackernews -path internal/db/migrations up
```

Last thing is that we need a connection to our database, for this we create a mysql.go under mysql folder(We name this file after mysql since we are now using mysql and if we want to have multiple databases we can add other folders) with a function to initialize connection to database for later use.
internals/db/mysql/mysql.go:
```go
package database

import 	(
	"database/sql"
	_ "github.com/go-sql-driver/mysql"
	"github.com/golang-migrate/migrate"
	"github.com/golang-migrate/migrate/database/mysql"
	_ "github.com/golang-migrate/migrate/source/file"
	"log"
)

var db *sql.DB

func InitDB() {
	db, err := sql.Open("mysql", "root:bozbozack@(172.17.0.2:3306)/hackernews")
	if err != nil {
		log.Panic(err)
	}

	if err = db.Ping(); err != nil {
		log.Panic(err)
	}
}
```

Then call `InitDB` In main func to create database connection at the start of the app:
```go
func main() {
	port := os.Getenv("PORT")
	if port == "" {
		port = defaultPort
	}

	database.Migrate()
	database.InitDB()

	http.Handle("/", handler.Playground("GraphQL playground", "/query"))
	http.Handle("/query", handler.GraphQL(hackernews.NewExecutableSchema(hackernews.Config{Resolvers: &hackernews.Resolver{}})))

	log.Printf("connect to http://localhost:%s/ for GraphQL playground", port)
	log.Fatal(http.ListenAndServe(":"+port, nil))
}

```


### Create and Retrieve Links
Now we have our database ready we can start implementing our schema!

#### CreateLinks
Lets implement CreateLink mutation; first we need a function to let us write a link to database.
Create a folders links and users inside internal folder, these packages are layers between database and our app.
users/users.go:
```go
package users

type User struct {
	ID       string `json:"id"`
	Name     string `json:"name"`
	Password string `json:"password"`
}
```
links/links.go:
```go
package links

import (
	database "github.com/glyphack/hackernews-graphql-go/internal/pkg/db/mysql"
	"github.com/glyphack/hackernews-graphql-go/internal/users"
	"log"
)
// #1
type Link struct {
	ID      string
	title   string
	address string
	user    *users.User
}

//#2
func (link Link) Save() int64{
  //#3
	statement, err := database.Db.Prepare("INSERT INTO Links(Title,Address) VALUES(?,?)")
	if err != nil {
		log.Fatal(err)
  }
  //#4
	res, err := statement.Exec(link.Title, link.Address)
	if err != nil {
		log.Fatal(err)
  }
  //#5
	id, err := res.LastInsertId()
	if err != nil {
		log.Fatal("Error:", err.Error())
	}
	log.Print("Row inserted!")
	return id
}
```
In users.go we just defined a `struct` that represent users we get from database, But let me explain links.go part by part:
* #1: definition of struct that represent a link.
* #2: function that insert a Link object into database and returns it's ID.
* #3: our sql query to insert link into Links table. you see we used prepare here instead of db.Exec at once, the prepared statements helps you with security and also performance improvement in some cases. you can read more about it [here](https://www.postgresql.org/docs/9.3/sql-prepare.html).
* #4: execution of our sql statement.
* #5: retrieving Id of inserted Link.

Now we use this function in our CreateLink resolver:
resolver.go:
```go
func (r *mutationResolver) CreateLink(ctx context.Context, input NewLink) (*Link, error) {
	var link links.Link
	link.Title = input.Title
	link.Address = input.Address
	linkId := link.Save()
	return &Link{ID: strconv.FormatInt(linkId, 10), Title:link.Title, Address:link.Address}, nil
}
```
Hopefully you Understand this piece of code, we create a link object from input and save it to database then return newly created link.
note that here we have 2 structs for Link here, one is use for our graphql server and one is for our database.
open graphiql to test what we just wrote:
```
mutation create{
  createLink(input: {title: "something", address: "somewhere"}){
    title,
    address,
    id,
  }
}
```
```
{
  "data": {
    "createLink": {
      "title": "something",
      "address": "somewhere",
      "id": "10"
    }
  }
}
```
Grate job!

#### links Query
Just like how we implemented CreateLink mutation we implement links query, we need a function to retrieve links from database and pass it to graphql server in our resolver.
Create a function named GetAll
`internal/links/links.go`:
```go
func GetAll() []Link {
	stmt, err := database.Db.Prepare("select id, title, address from Links")
	if err != nil {
		log.Fatal(err)
	}
	defer stmt.Close()
	rows, err := stmt.Query()
	if err != nil {
		log.Fatal(err)
	}
	defer rows.Close()
	var links []Link
	for rows.Next() {
		var link Link
		err := rows.Scan(&link.ID, &link.Title, &link.Address)
		if err != nil{
			log.Fatal(err)
		}
		links = append(links, link)
	}
	if err = rows.Err(); err != nil {
		log.Fatal(err)
	}
	return links
}
```

Return links from GetAll in Links query.
`resolver.go`:
```go
func (r *queryResolver) Links(ctx context.Context) ([]*Link, error) {
	var resultLinks []*Link
	var dbLinks []links.Link
	dbLinks = links.GetAll()
	for _, link := range dbLinks{
		resultLinks = append(resultLinks, &Link{ID:link.ID, Title:link.Title, Address:link.Address})
	}
	return resultLinks, nil
}
```
Now query Links at graphiql:
```
{
  "data": {
    "createLink": {
      "title": "something",
      "address": "somewhere",
      "id": "10"
    }
  }
}
```
result:
```
{
  "data": {
    "links": [
      {
        "id": "1",
        "title": "something",
        "address": "somewhere"
      },
    ]
  }
}
```

### Authentication
#### Setup JWT
##### Database Table
##### Authentication Middleware

#### CreateUser
#### Login
#### Enhance our 