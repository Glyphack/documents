---
title: ‌GraphQL server with Go introduction
published: false
description: Introduction to GraphQL apis with golang.
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
      - [JWT](#jwt)
      - [Setup](#setup)
        - [Generating and Parsing JWT Tokens](#generating-and-parsing-jwt-tokens)
        - [User Signup and Login Functionality](#user-signup-and-login-functionality)
        - [Authentication Middleware](#authentication-middleware)
  - [Continue Implementing schema](#continue-implementing-schema)
    - [CreateUser](#createuser)
    - [Login](#login)
	- [RefreshToken](#refresh-token)
    - [Completing Our App](#completing-our-app)

### Motivation <a name="motivation"></a>
[**Go**](https://golang.org/) is a modern general purpose programming language designed by google; best known for it's simplicity, concurrency and fast performance. It's being used by big players in the industry like Google, Docker, Lyft and Uber. If you are new to golang you can start from [golang tour](https://tour.golang.org/) to learn fundamentals.

[**gqlgen**](https://gqlgen.com/) is a library for creating GraphQL applications in Go.


In this tutorial we create a Hackernews clone GraphQL API with `golang` and `gqlgen` and learn about GraphQL fundamentals along the way.

#### What is a GraphQL server? <a name="#what-is-a-graphql-server"></a>
GraphQL is a query language for API so you can send queries and ask what you need and exactly get that piece of data.
sample query:
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
sample response:
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

#### Schema-Driven Development <a name="#schema-driven-development"></a>
In GraphQL your API starts with a schema that defines all your types, queries and mutations, It helps others to understand your API so all of your team can understand how to work with your API, a contract between server and the client.
Whenever you need to add a new capability to a GraphQL API you must redefine schema file and then implement that part in your code. GraphQL has it's [Schema Definition Language](http://graphql.org/learn/schema/) for this.
gqlgen library has a nice feature and generate code based on your schema definition.

### Getting started <a name="getting-started"></a>
In this tutorial we are going to create a Hackernews clone with Go and gqlgen, So our API will be able to handle registration, authentication, submitting links and returning list of links.

#### Project Setup a<name="project-setup"></a>
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
start the server with `go run server.go` and open your browser and you should see the graphql playground, So setup is right!

Now let's start with defining schema file with features we need for our API. We have two types Link and User each of them for representing Link and User to client, a `links` Query to return list of Links. a input for creating NewLink and mutation for creating link and returns created link. then run the command below to regenerate models.
```js
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

input RefreshTokenInput{
  token: String!
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
  createUser(input: NewUser!): String!
  login(input: Login!): String!
  # we'll talk about this in authentication section
  refreshToken(input: RefreshTokenInput!): String!
}
```
Now run the command to regenerate files;
```go
go run github.com/99designs/gqlgen
```
After gqlgen generated code for us with have to implement our schema, we do that in ‍‍‍‍`resolver.go`, as you see there is functions for Queries and Mutations we defined in our schema.
Now let's see what we got, run app with `go run server/server.go` and go to localhost you should see graphiql page.


### Queries a<name="queries"></a>
In the previous chapter we setup up a server that runs graphql, Now we try to implement a Query that we defined in `schema.grpahql`.

#### What Is A Query a<name="what-is-a-query"></a>
a query in graphql is asking for data, you ask for the data type you want and specify what you want from that type and graphql will return it back to you

#### Simple Query a<name="simple-query"></a>
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


### Mutations a<name="mutations"></a>
#### What Is A Mutation a<name="what-is-a-mutation"></a>
Simply mutations are just like queries but they can cause a data write, Technically Queries can be used to write data too however it's not suggested to use it.
So mutations are like queries, they have names, parameters and they can return data.
#### A Simple Mutation a<name="a-simple-mutation"></a>
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


### Database a<name="database"></a>
Before we jump into implementing GraphQL schema we need to setup database to save users and links, This is not supposed to be tutorial about databases in go but here is what we are going to do:
* setup MySQL and create table
* define our models and create migrations

##### Setup MySQL a<name="setup-mysql"></a>
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

##### Models and migrations a<name="models-and-migrations"></a>
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


### Create and Retrieve Links a<name="create-and-retrieve-links"></a>
Now we have our database ready we can start implementing our schema!

#### CreateLinks a<name="createlinks"></a>
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

#### links Query a<name="links-query"></a>
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

### Authentication a<name="authentication"></a>
One of most common layers in web applications is authentication system, our app is no exception. For authentication we are going to use jwt tokens as our way to authentication users, lets see how it works.

##### JWT a<name="jwt"></a>
[JWT](https://jwt.io/) or Json Web Token is a string containing a hash that helps us verify who is using application. Every token is constructed of 3 parts like 'xxxxx.yyyyy.zzzzz' and name of these parts are: Header, Payload and Signature. Explanation about these parts are more about JWT than our application you can read more about them [here](https://jwt.io/introduction/).
whenever a user login to an app server generates a token for user, Usually server saves some information like username about the user in token to be able to recognize the user later using that token.This tokens get signed by a key so only the issuer app can reopen the token.
We are going to implement this behavior in our app. 

##### Setup a<name="setup"></a>
In our app we need to be able to generate a token for users when they sign up or login and a middleware to authenticate users by the given token, then in our views we can know the user interacting with app. We will be using `github.com/dgrijalva/jwt-go` library to generate and prase JWT tokens.
###### Generating and Parsing JWT Tokens a<name="generating-and-parsing-jwt-tokens"></a>
We create a new directory pkg in the root of our application, you have seen that we used internal for what we want to only be internally used withing our app, pkg directory is for files that we don't mind if some outer code imports it into itself and generation and validation jwt tokens are this kind of codes.
There is a concept named claims it's not only limited to JWT
`pkg/jwt/token.go`:
```go
package jwt

import (
	"github.com/dgrijalva/jwt-go"
	"log"
	"time"
)

// secret key being used to sign tokens
const (
	SecretKey = "secret"
)

//data we save in each token
type Claims struct {
	username string
	jwt.StandardClaims
}

//GenerateToken generates a jwt token and assign a username to it's claims and return it
func GenerateToken(username string) (string, error) {
	claims := &Claims{
		username: username,
		StandardClaims: jwt.StandardClaims{
			ExpiresAt: time.Now().Add(time.Minute * time.Duration(5)).Unix(),
			IssuedAt:  time.Now().Unix(),
		},
	}
	token := jwt.NewWithClaims(jwt.SigningMethodHS256, claims)
	tokenStr, err := token.SignedString([]byte(SecretKey))
	if err != nil {
		log.Fatal("Error in Generating key")
		return "", err
	}
	return tokenStr, nil

}

//ParseToken parses a jwt token and returns the username it it's claims
func ParseToken(tokenStr string) (string, error) {
	claims := &Claims{}

	tkn, err := jwt.ParseWithClaims(tokenStr, claims, func(token *jwt.Token) (interface{}, error) {
		return SecretKey, nil
	})

	if err != nil || !tkn.Valid {
		return "", err
	}

	return claims.username, nil
}

```
Let's talk about what above code does:
* GenerateToken function is going to be used whenever we want to generate a token for user, we save username in token claims and set token expire time to 5 minutes later also in claims.
* ParseToken function is going to be used whenever we receive a token and want to know who sent this token.

###### User SignUp and Login Functionality a<name="user-signup-and-login-functionality"></a>
Til now we can generate a token for each user but before generating token for every user, we need to assure user exists in our database. Simply we need to query database to match the user with given username and password.
Another thing is when a user tries to register we insert username and password in our database.
`internal/users/users.go`:
```go
package users

import (
	"database/sql"
	"github.com/glyphack/hackernews-graphql-go/internal/pkg/db/mysql"
	"golang.org/x/crypto/bcrypt"

	"log"
)

type User struct {
	ID       string `json:"id"`
	Username     string `json:"name"`
	Password string `json:"password"`
}

func (user *User) Create() {
	statement, err := database.Db.Prepare("INSERT INTO Users(Name,Password) VALUES(?,?)")
	print(statement)
	if err != nil {
		log.Fatal(err)
	}
	hashedPassword, err := HashPassword(user.Password)
	_, err = statement.Exec(user.Username, hashedPassword)
	if err != nil {
		log.Fatal(err)
	}
}

func (user *User) Authenticate() bool{
	statement, err := database.Db.Prepare("select Password from Users WHERE Username = ?")
	if err != nil {
		log.Fatal(err)
	}
	row := statement.QueryRow(user.Username)

	var hashedPassword string
	err = row.Scan(&hashedPassword)
	if err != nil {
		if err == sql.ErrNoRows {
			return false
		}else{
			log.Fatal(err)
		}
	}

	return CheckPasswordHash(user.Password, hashedPassword)
}

//HashPassword hashes given password
func HashPassword(password string) (string, error) {
	bytes, err := bcrypt.GenerateFromPassword([]byte(password), 14)
	return string(bytes), err
}

//CheckPassword hash compares raw password with it's hashed values
func CheckPasswordHash(password, hash string) bool {
	err := bcrypt.CompareHashAndPassword([]byte(hash), []byte(password))
	return err == nil
}
```
The Create function is much like the [CreateLink](#createlinks) function we saw earlier but let's break down the Authenticate code:
* first we have a query to select password from users table where username is equal to the username we got from resolver.
* We use QueryRow instead of Exec we used earlier; the difference is `QueryRow()` will return a pointer to a `sql.Row`.
* Using `.Scan` method we tell the program to fill the hashedPassword variable with the password from database.
* then we check if any user with given username exists or not, if there is not any we return `false`, and if we found any we check the user hashedPassword with the raw password given.(Notice that we save hashed passwords not raw passwords in database in line 23)

In the next part we set the tools we have together to detect the user that is using the app.
###### Authentication Middleware a<name="authentication-middleware"></a>
Every time a request comes to our resolver before sending it to resolver we want to recognize the user sending request, for this purpose we have to write a code before every resolver, but using middleware we can have a auth middleware that executes before request send to resolver and does the authentication process. to read more about middlewares visit.


`internal/users/users.go`:
```go
//GetUserByUsername check if a user exists in database by given username
func (user *User) GetUserByUsername() (int, error) {
	statement, err := database.Db.Prepare("select ID from Users WHERE Username = ?")
	if err != nil {
		log.Fatal(err)
	}
	row := statement.QueryRow(user.Username)

	var Id int
	err = row.Scan(&Id)
	if err != nil {
		if err != sql.ErrNoRows {
			log.Print(err)
		}
		return 0, err
	}

	return Id, nil
}
```

`internal/auth/middleware.go`:
```go
package auth

import (
	"context"
	"net/http"
	"strconv"

	"github.com/glyphack/hackernews-graphql-go/internal/users"
	"github.com/glyphack/hackernews-graphql-go/pkg/jwt"
)

var userCtxKey = &contextKey{"user"}

type contextKey struct {
	name string
}

func Middleware() func(http.Handler) http.Handler {
	return func(next http.Handler) http.Handler {
		return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
			c, err := r.Cookie("token")

			// Allow unauthenticated users in
			if err != nil || c == nil {
				next.ServeHTTP(w, r)
				return
			}

			//validate jwt token
			tokenStr := c.Value
			username, err := jwt.ParseToken(tokenStr)
			if err != nil {
				http.Error(w, "Invalid token", http.StatusForbidden)
				return
			}

			// create user and check if user exists in db
			user := users.User{Username: username}
			id, err := user.GetUserByUsername()
			if err != nil {
				next.ServeHTTP(w, r)
				return
			}
			user.ID = strconv.Itoa(id)

			// put it in context
			ctx := context.WithValue(r.Context(), userCtxKey, user)

			// and call the next with our new context
			r = r.WithContext(ctx)
			next.ServeHTTP(w, r)
		})
	}
}

// ForContext finds the user from the context. REQUIRES Middleware to have run.
func ForContext(ctx context.Context) *users.User {
	raw, _ := ctx.Value(userCtxKey).(*users.User)
	return raw
}
```

Now we use the middleware we declared in our server:
`server/server.go`:
```go
package main

import (
	"github.com/glyphack/hackernews-graphql-go/internal/auth"
	"log"
	"net/http"
	"os"

	"github.com/99designs/gqlgen/handler"
	hackernews "github.com/glyphack/hackernews-graphql-go"
	"github.com/glyphack/hackernews-graphql-go/internal/pkg/db/mysql"
	"github.com/go-chi/chi"
)

const defaultPort = "8080"

func main() {
	port := os.Getenv("PORT")
	if port == "" {
		port = defaultPort
	}

	router := chi.NewRouter()

	router.Use(auth.Middleware())

	database.InitDB()
	//database.Migrate()
	server := handler.GraphQL(hackernews.NewExecutableSchema(hackernews.Config{Resolvers: &hackernews.Resolver{}}))
	router.Handle("/", handler.Playground("GraphQL playground", "/query"))
	router.Handle("/query", server)

	log.Printf("connect to http://localhost:%s/ for GraphQL playground", port)
	log.Fatal(http.ListenAndServe(":"+port, router))
}
```

### Continue Implementing schema a<name="continue-implementation"></a>
Now that we have working authentication system we can get back to implementing our schema.
#### CreateUser a<name="createuser"></a>
As you may guess, we need a function to interact with our database:
`internal/users.go`:
```go
func (user *User) Create() error {
	statement, err := database.Db.Prepare("INSERT INTO Users(Name,Password) VALUES(?,?)")
	if err != nil {
		return err
	}
	// 1
	hashedPassword, err := HashPassword(user.Password)
	_, err = statement.Exec(user.Username, hashedPassword)
	if err != nil {
		return err
	}
	return nil
}

//HashPassword hashes given password
func HashPassword(password string) (string, error) {
	bytes, err := bcrypt.GenerateFromPassword([]byte(password), 14)
	return string(bytes), err
}
Explanation:
* Obviously you don't want to [save raw passwords](https://security.blogoverflow.com/2011/11/why-passwords-should-be-hashed/) in your database.
```

We continue our implementation by using this function in our mutation to create users.
`resolver.go`:
```go
func (r *mutationResolver) CreateUser(ctx context.Context, input NewUser) (string, error) {
	var user users.User
	user.Username = input.Username
	user.Password = input.Password
	err := user.Create()
	token, err := jwt.GenerateToken(user.Username)
	if err != nil{
		return "", err
	}
	return token, nil
}
```
In our mutation first we create a user using given username and password and then generate a token for the user so we can recognize the user in requests.
#### Login a<name="login"></a>
For this mutation, first we have to check if user exists in database and given password is correct, then we generate a token for user and give it bach to user.
`internal/users.go`:
```go
func (user *User) Authenticate() bool {
	statement, err := database.Db.Prepare("select Password from Users WHERE Username = ?")
	if err != nil {
		log.Fatal(err)
	}
	row := statement.QueryRow(user.Username)

	var hashedPassword string
	err = row.Scan(&hashedPassword)
	if err != nil {
		if err == sql.ErrNoRows {
			return false
		} else {
			log.Fatal(err)
		}
	}

	return CheckPasswordHash(user.Password, hashedPassword)
}

//CheckPassword hash compares raw password with it's hashed values
func CheckPasswordHash(password, hash string) bool {
	err := bcrypt.CompareHashAndPassword([]byte(hash), []byte(password))
	return err == nil
}
```
Explanation:
* we select the user with the given username and then check if hash of the given password is equal to hashed password that we saved in database.

`resolver.go`
```go
func (r *mutationResolver) Login(ctx context.Context, input Login) (string, error) {
	var user users.User
	user.Username = input.Username
	user.Password = input.Password
	correct := user.Authenticate()
	if !correct {
		// 1
		return "", &users.WrongUsernameOrPasswordError{}
	}
	token, err := jwt.GenerateToken(user.Username)
	if err != nil{
		return "", err
	}
	return token, nil
}
```
We used the Authenticate function declared above and after that if the username and password are correct we return a new token for user and if not we return error, `&users.WrongUsernameOrPasswordError`, here is implementation for this error:
`internal/users/errors.go`:
```go
package users

type WrongUsernameOrPasswordError struct{}

func (m *WrongUsernameOrPasswordError) Error() string {
	return "wrong username or password"
}
```
To define a custom error in go you need a struct with Error method implemented, here is our error for wrong username or password with it's Error() method.

#### Refresh Token <a name="refresh-token"></a>
This is the last endpoint we need to complete our authentication system, imagine a user has loggedIn in our app and it's token is going to get expired after minutes we set(when generated the token), now we need a solution to keep our user loggedIn. One solution is to have a endpoint to get tokens that are going to expire and regenerate a new token for that user so that app uses new token.
So our endpoint should take a token, Parse the username and generate a token for that username.
`resolver.go`:
```go
func (r *mutationResolver) RefreshToken(ctx context.Context, input RefreshTokenInput) (string, error) {
	username, err := jwt.ParseToken(input.Token)
	if err != nil {
		return "", fmt.Errorf("access denied")
	}
	token, err := jwt.GenerateToken(username)
	if err != nil {
		return "", err
	}
	return token, nil
}
```
Implementation is pretty straightforward so we skip the explanation for this.


#### Completing Our app a<name="completing-our-app"></a>
Our CreateLink mutation left incomplete because we could not authorize users back then, so let's get back to it and complete the implementation.
With what we did in [authentication middleware](#authentication-middleware) we can retrieve user in resolvers using ctx argument. so in CreateLink function add these lines:
`resolver.go`:
```go
func (r *mutationResolver) CreateLink(ctx context.Context, input NewLink) (*Link, error) {
	// 1
	user := auth.ForContext(ctx)
	if user == nil {
		return &Link{}, fmt.Errorf("access denied")
	}
	.
	.
	.
	// 2
	link.User = user
	linkId := link.Save()
	return &Link{ID: strconv.Itoa(linkId), Title: link.Title, Address: link.Address}, nil
}
```
Explanation:
* 1: we get user object from ctx and if user is not set we return error with message access denied.
* 2: then we set user of that link equal to the user is requesting to create the link.

The part that is left here is our database operation for creating link, We need to create foreign key from the link we inserting to that user.
`internal/links/links.go`:
In our Save method from links changed the query statement to:
```go
statement, err := database.Db.Prepare("INSERT INTO Links(Title,Address, UserID) VALUES(?,?, ?)")
```
and the line that we execute query to:
```go
res, err := statement.Exec(link.Title, link.Address, link.User.ID)
```

and Our app is finally complete.