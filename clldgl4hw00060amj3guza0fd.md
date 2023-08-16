---
title: "Go GraphQL Go!!!"
seoTitle: "A beginner's guide to GraphQL in Go using Ent."
datePublished: Wed Aug 16 2023 08:16:34 GMT+0000 (Coordinated Universal Time)
cuid: clldgl4hw00060amj3guza0fd
slug: go-graphql-go
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1692173308546/1e5cbd65-ed7e-4e02-9cea-d876d18dc442.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1692173737769/18dc0264-7fef-4847-9a73-ed7b0b0b3f83.png
tags: entity-framework, go, graphql, beginners, ent

---

Have you ever wondered how computers and applications communicate with each other to fetch information? Well, they use something called an API, which stands for Application Programming Interface. APIs act as a bridge that allows different software systems to talk to each other and exchange data.

In the early days of the Internet, building APIs was challenging. Developers had to design them in a way that made sense for everyone using them. It was like trying to order a drink from a bartender who had a complex menu with too many options. You would often end up getting more information than you needed or not enough, like ordering a simple orange juice and receiving the entire fruit basket! This caused frustration and wasted time for developers who had to sift through unnecessary data or make multiple requests to get what they wanted. Imagine having to ask the bartender for a drink, but instead of a simple order, you received an entire catalogue of beverages! To make matters worse, traditional APIs, known as REST APIs, relied on a large number of endpoints. These endpoints acted as specific paths to access different parts of the data. It was like having a maze with countless doors to navigate through. But then, something game-changing happened. Facebook introduced GraphQL, a revolutionary approach to building APIs in 2012 that turned the tables completely. With GraphQL, developers finally said goodbye to the headaches of over-fetching and under-fetching data. Fast forward to 2015, Facebook open-sourced GraphQL and in 2018 it donated GraphQL to the Linux Foundation.

GraphQL is a query language for API or some may say it is a new standard for developing APIs.

In this blog, we're going to explore how GraphQL tackles the challenges faced by traditional REST APIs. We'll also embark on a hands-on journey of building a GraphQL server in Go and to make our development process even more exciting and efficient, we'll leverage the power of Ent, an amazing entity library designed specifically for Go.

### GraphQL the saviour

In the introduction, we mentioned issues of RESTful APIs. Let’s try to understand them and look into how GraphQL solves them.

Imagine you're at a library, and you want to gather information about different books. You go to the librarian and request details about a book's title, author, and publication date. The librarian gives you the title, but when you ask for the author's email or address or other books published by the same author, they tell you to go to a different librarian. To get all the information you need, you have to keep bouncing between different librarians. This is called ***under-fetching*** of data.

In the world of software and applications, a similar situation occurs when fetching data from servers. 

Let's consider a scenario where you're using a book catalogue API to fetch information about different books. If you want to retrieve the name of the author for a specific book, you would typically need to make multiple API calls to different endpoints.

For instance, the initial endpoint might be ***/books/:$ID/author***, where ***$ID*** represents the unique identifier of the book. Underneath the surface, the API would first fetch all the book profiles from the Books entity. From that dataset, it would then extract the specific book using the provided ID and from that dataset, it’ll fetch the author ID. And finally using this ID server will make calls to ***/authors/:$ID*** endpoint which will make fetch all the details of that author from the Author entity.

```bash
Request 1: GET /books
Response 1:
{
  "books": [
    {
      "id": 1,
      "title": "The Ink Black Heart",
      "genre": "Mystery",
      "publicationDate": "30 August 2022",
      "isbn": "9780316413138"
    },
    ...
  ]
}

Request 2: GET /authors
Response 2:
{
  "authors": [
    {
      "id": 1,
      "name": "J. K. Rowling"
    },
    ...
  ]
}
```

As you can see, the server has to make multiple calls to different endpoints to fulfil this request, resulting in what we call ***under-fetching*** of data. It means that the API fails to retrieve all the required data in a single call, leading to additional requests and unnecessary processing.

On the other hand, there's the issue of ***over-fetching*** data. 

Imagine you're at a magical restaurant where you can order any food or drink you desire. You walk up to the bartender and say, "I'd like a drink, please." The bartender nods, disappears for a moment, and returns with a tray filled with every drink imaginable: water, soda, juice, cocktails, and even a bowl of soup! You only wanted a simple glass of lemonade, but now you're overwhelmed with choices. This is called ***over-fetching*** of data.

Let's say you only need the name of the author for a particular book. However, the server, following a traditional RESTful approach, fetches and sends all the available information about the author, such as their ***ID, phone number, email, and address***. This extra data retrieval and processing are considered over-fetching, as it includes more information than necessary.

These types of data requests can strain system resources, resulting in high traffic and reduced performance. The continuous accumulation of such inefficient requests can degrade the system's overall performance and scalability over time.

The GraphQL solves above issues with only one API call.

```graphql
Request: Get /graphql
Body:
{
  "query": "
    query {
      books {
        title
        genre
        author {
          name
        }
      }
    }
  "
}

Response:
{
  "data": {
    "books": [
      {
        "title": "The Ink Black Heart",
        "genre": "Mystery",
        "author": {
          "name": "J. K. Rowling"
        }
      },
      ...
    ]
  }
}
```

As you can see, the GraphQL query precisely defines the fields needed: **book title, genre, and the nested author's name**. The response contains only the requested data, eliminating under-fetching and over-fetching issues. This approach reduces network overhead, improves performance, and enhances the overall developer and user experience.

In the case of smaller companies with a limited user base, the challenges of over-fetching and under-fetching data may not pose significant problems. However, for behemoths like Facebook, which handles an enormous amount of data and millions of requests per second, these issues become critical. When serving user requests, Facebook often needs to make multiple REST calls to fetch the precise data required. The multiplication effect of even a few million extra requests can overload the servers with processing overhead, leading to high traffic and reduced performance.

### Query and Mutation

To fully understand GraphQL we need to understand the Query and Mutation first.

**Queries:**

In GraphQL, queries are used to retrieve data from the server. They allow you to specify the exact data you need and the shape of the response. You define a query by specifying the fields you want to fetch and their relationships. The server processes the query and returns the requested data in a structured format, typically JSON.

For example, a query in GraphQL for retrieving book information may look like this:

```graphql
query {
  book(id: 123) {
    title
    genre
    author {
      name
    }
  }
}
```

This query asks the server to fetch the title and genre of a book with the ID 123, along with the name of its author. The response will only contain the requested fields.

**Mutations:**

Mutations in GraphQL are used to modify or create data on the server. They allow you to perform operations such as creating new records, updating existing ones, or deleting data. Mutations are analogous to the POST, PUT, or DELETE methods in RESTful APIs.

For example, a mutation in GraphQL for creating a book and its author may look like this:

```graphql
mutation {
  createBook(title: "The Ink Black Heart", genre: "Mystery", author: "J. K. Rowling") {
    id
    title
    genre
    author {
      id
      name
    }
  }
}
```

This mutation creates a new book with the title "The Ink Black Heart", genre "Mystery", and assigns it to the author "J. K. Rowling". The response will contain the ID, title, genre, and the ID and name of the author of the created book.

By using queries and mutations, GraphQL provides a flexible and efficient way to retrieve and modify data from a server. Clients can request precisely what they need, and mutations enable them to modify the data while maintaining a clear and consistent API contract.

For more information about GraphQL and what it can do or maybe on writing complex queries or mutations, you can visit the [official documentation](https://graphql.org/code/).

## Let's Code

In this blog, we’ll develop a very minimal graphql server for the book catalogue application where we’ll be able to create book and author entities in the database and fetch them. The source code for this project is available at [pratikjagrut/book-catalog](https://github.com/pratikjagrut/book-catalog.git).

### **Prerequisite**

[Go](https://go.dev/doc/install)

### **Setting up Ent**

Ent is an open-source entity framework designed specifically for Go. Think of it as a tool that helps us work with databases in an easier and more organized way. Ent has gained popularity in the Go community for its unique features and benefits.

Originally developed at Facebook, Ent was created to address the challenges of managing applications with large and complex data models. It was successfully used within Facebook for a year before being open-sourced in 2019. Since then, Ent has grown and even joined the Linux Foundation in 2021.

For detailed information on Ent see the [docs](https://entgo.io/).

After installing prerequisite dependencies, create a directory for the project and initialize a Go module:

```bash
mkdir book-catalog
cd book-catalog
go mod init github.com/pratikjagrut/book-catalog
```

**Installation**

Run the following Go commands to install Ent, and tell it to initialize the project structure along with a **Book** schema.

```bash
go get -d entgo.io/ent/cmd/ent
go run -mod=mod entgo.io/ent/cmd/ent new Book
go run -mod=mod entgo.io/ent/cmd/ent new Author
```

After installing Ent and running **ent new**, your project directory should look like this:

```bash
➜  book-catalog git:(main) ✗ tree .
.
├── ent
│   ├── generate.go
│   └── schema
│       ├── author.go
│       └── book.go
├── go.mod
└── go.sum

2 directories, 5 files
```

**Code Generation**

When you run the ent new command it generates a schema file for you at ***ent/schema/book.go***

```go
package schema

import "entgo.io/ent"

// Book holds the schema definition for the Book entity.
type Book struct {
   ent.Schema
}

// Fields of the Book.
func (Book) Fields() []ent.Field {
   return nil
}

// Edges of the Book.
func (Book) Edges() []ent.Edge {
   return nil
}
```

As you can see, initially, the schema has no fields or edges defined. Let's run the command for generating assets to interact with the Book and Author entity:

```bash
go generate ./ent
```

When we run the command ***go generate ./ent***, it triggers Ent's automatic code generation tool. This tool takes the schemas we define in the schema package and generates the corresponding Go code. This generated code will enable us to interact with the database.

After running the code generation, you'll find a file named ***client.go*** under the ***./ent*** directory. This file contains client code that allows us to perform queries and mutations on the entities. It serves as our gateway to interact with the database.

Let's create a test to use this. We'll use [SQLite](https://github.com/mattn/go-sqlite3) in this test case for testing Ent.

```bash
go get github.com/mattn/go-sqlite3
go get github.com/stretchr/testify/assert
touch ent/book-catalog_test.go
```

You can add the following code to your ***book-catalog\_test.go*** file. This code creates an instance of ***ent.Client*** automatically generates all the necessary schema resources in the database, including tables and columns.

At this stage, the test only establishes a connection and creates a schema without any fields or edges. However, we will update and expand this test as we progress through the blog.

```go
package ent

import (
   "context"
   "testing"

   "github.com/stretchr/testify/assert"

   _ "github.com/mattn/go-sqlite3"
)

func TestBookCatalog(t *testing.T) {
   client, err := Open("sqlite3", "file:book-catalog.db?cache=shared&_fk=1")

   assert.NoErrorf(t, err, "failed opening connection to sqlite")
   defer client.Close()

   ctx := context.Background()

   // Run the automatic migration tool to create all schema resources.
   err = client.Schema.Create(ctx)
   assert.NoErrorf(t, err, "failed creating schema resources")
}
```

Then, run the ***go test -v ./ent***, it will create a schema with empty books table.

```bash
➜  book-catalog git:(main) ✗ go test -v ./ent
=== RUN   TestBookCatalog
--- PASS: TestBookCatalog (0.00s)
PASS
ok  	github.com/pratikjagrut/book-catalog/ent	0.660s
```

**Create database schema:**

With the basic setup in place, we are now ready to expand our Book entity by adding fields and proceeding with building queries and mutations.

Let’s define a schema for our database:

The author has the following properties:

* **ID**: A unique identifier for the book. Auto-generated.
    
* **Name: Name of the author**
    
* **Email: Email address of the author**
    

In addition, we need to establish a relationship, or edge, between the Author and Book entities. In this scenario, an author can write multiple books, creating a One-to-Many (1-&gt;M) relationship.

Add the following fields and edges to the **ent/schema/author.go file**.

```go
package schema

import (
   "entgo.io/ent"
   "entgo.io/ent/schema/edge"
   "entgo.io/ent/schema/field"
)

// Author holds the schema definition for the Author entity.
type Author struct {
   ent.Schema
}

// Fields of the Author.
func (Author) Fields() []ent.Field {
   return []ent.Field{
       field.String("name"),
       field.String("email"),
   }
}

// Edges of the Author.
func (Author) Edges() []ent.Edge {
   return []ent.Edge{
       edge.To("books", Book.Type),
   }
}
```

The book has the following properties:

* **ID**: A unique identifier for the book. Auto-generated.
    
* **Title**: The title or name of the book.
    
* **Genre**: The genre or category to which the book belongs.
    
* **PublicationDate**: The date when the book was published.
    
* **ISBN**: The International Standard Book Number assigned to the book.
    
* **CreatedAt**: The date and time of record creation in the database.
    

Here, the relation between the book and the author will be ***many-to-one.*** So we’ll create an inverse edge.

Now, add these fields to the **ent/schema/book.go file**.

```go
package schema

import (
   "time"

   "entgo.io/ent"
   "entgo.io/ent/schema/edge"
   "entgo.io/ent/schema/field"
)

// Book holds the schema definition for the Book entity.
type Book struct {
   ent.Schema
}

// Fields of the Book.
func (Book) Fields() []ent.Field {
   return []ent.Field{
       field.String("title").NotEmpty(),
       field.String("genre").NotEmpty(),
       field.String("publication_date").NotEmpty(),
       field.String("isbn").NotEmpty(),
       field.Time("created_at").Default(time.Now()),
   }
}

// Edges of the Book.
func (Book) Edges() []ent.Edge {
   return []ent.Edge{
       edge.From("author", Author.Type).
           Ref("books").
           Unique(),
   }
}
```

**Create mutations and queries**

Once again, let's run ***go generate ./ent*** to generate the necessary mutations for the fields we defined in our Author and Book entity and check schema with ***go run -mod=mod*** [***entgo.io/ent/cmd/ent***](http://entgo.io/ent/cmd/ent) ***describe ./ent/schema*** command.

```bash
➜ go run -mod=mod entgo.io/ent/cmd/ent describe ./ent/schema
Author:
	+-------+--------+--------+----------+----------+---------+---------------+-----------+------------------------+------------+---------+
	| Field |  Type  | Unique | Optional | Nillable | Default | UpdateDefault | Immutable |       StructTag        | Validators | Comment |
	+-------+--------+--------+----------+----------+---------+---------------+-----------+------------------------+------------+---------+
	| id    | int    | false  | false    | false    | false   | false         | false     | json:"id,omitempty"    |          0 |         |
	| name  | string | false  | false    | false    | false   | false         | false     | json:"name,omitempty"  |          0 |         |
	| email | string | false  | false    | false    | false   | false         | false     | json:"email,omitempty" |          0 |         |
	+-------+--------+--------+----------+----------+---------+---------------+-----------+------------------------+------------+---------+
	+-------+------+---------+---------+----------+--------+----------+---------+
	| Edge  | Type | Inverse | BackRef | Relation | Unique | Optional | Comment |
	+-------+------+---------+---------+----------+--------+----------+---------+
	| books | Book | false   |         | O2M      | false  | true     |         |
	+-------+------+---------+---------+----------+--------+----------+---------+

Book:
	+------------------+-----------+--------+----------+----------+---------+---------------+-----------+-----------------------------------+------------+---------+
	|      Field       |   Type    | Unique | Optional | Nillable | Default | UpdateDefault | Immutable |             StructTag             | Validators | Comment |
	+------------------+-----------+--------+----------+----------+---------+---------------+-----------+-----------------------------------+------------+---------+
	| id               | int       | false  | false    | false    | false   | false         | false     | json:"id,omitempty"               |          0 |         |
	| title            | string    | false  | false    | false    | false   | false         | false     | json:"title,omitempty"            |          1 |         |
	| genre            | string    | false  | false    | false    | false   | false         | false     | json:"genre,omitempty"            |          1 |         |
	| publication_date | string    | false  | false    | false    | false   | false         | false     | json:"publication_date,omitempty" |          1 |         |
	| isbn             | string    | false  | false    | false    | false   | false         | false     | json:"isbn,omitempty"             |          1 |         |
	| created_at       | time.Time | false  | false    | false    | true    | false         | false     | json:"created_at,omitempty"       |          0 |         |
	+------------------+-----------+--------+----------+----------+---------+---------------+-----------+-----------------------------------+------------+---------+
	+--------+--------+---------+---------+----------+--------+----------+---------+
	|  Edge  |  Type  | Inverse | BackRef | Relation | Unique | Optional | Comment |
	+--------+--------+---------+---------+----------+--------+----------+---------+
	| author | Author | true    | books   | M2O      | true   | true     |         |
	+--------+--------+---------+---------+----------+--------+----------+---------+
```

With Ent, creating migrations and performing common operations such as creating a record or fetching records is straightforward and practical.

To create a new record in the database, we can simply call the following code:

```go
author, _ := client.Author.Create().
       SetName("J. K. Rowling").
       SetEmail("jk@gmail.com").
       Save(context.Background())

book, _ := client.Book.Create().
       SetTitle("The Ink Black Heart").
       SetGenre("Mystery").
       SetIsbn("9780316413138").
       SetPublicationDate("30 August 2022").
       SetAuthor(author).
       Save(context.Background())
```

This code snippet creates a new author and book record and adds the edge using the ***SetAuthor*** method.

To fetch all the books from the database, we can use the following code:

```go
books, err := client.Book.Query().All(ctx)
```

This code retrieves all the Book records stored in the database.

Ent provides many more useful functions and options for creating, fetching, and manipulating data in the database. Such features make database operations more manageable and efficient. I encourage you to explore the [Ent documentation](https://entgo.io/docs/getting-started) for a deeper understanding of Ent's capabilities and how to make the most of it in your projects. 

**Test ent setup**

Let’s update our **ent/book-catalog\_test.go** with migrations.

```go
package ent

import (
   "context"
   "testing"

   "github.com/stretchr/testify/assert"

   _ "github.com/mattn/go-sqlite3"
)

func TestBookCatalog(t *testing.T) {
   // client, err := Open("sqlite3", "file:book-catalog.db?cache=shared&_fk=1")
   client, err := Open("sqlite3", "file:ent?mode=memory&cache=shared&_fk=1")
   assert.NoErrorf(t, err, "failed opening connection to sqlite")
   defer client.Close()

   ctx := context.Background()

   // Run the automatic migration tool to create all schema resources.
   err = client.Schema.Create(ctx)
   assert.NoErrorf(t, err, "failed creating schema resources")

   author, err := client.Author.Create().
       SetName("J. K. Rowling").
       SetEmail("jk@gmail.com").
       Save(ctx)
   assert.NoError(t, err)

   _, err = client.Book.Create().
       SetTitle("The Ink Black Heart").
       SetGenre("Mystery").
       SetIsbn("9780316413138").
       SetPublicationDate("30 August 2022").
       SetAuthor(author).
       Save(ctx)
   assert.NoError(t, err)

   author, err = client.Author.Create().
       SetName("George R. R. Martin").
       SetEmail("grrm@gmail.com").
       Save(ctx)
   assert.NoError(t, err)

   _, err = client.Book.Create().
       SetTitle("A Game of Thrones").
       SetGenre("Fantasy Fiction").
       SetIsbn("9780553593716").
       SetPublicationDate("1 August 1996").
       SetAuthor(author).
       Save(ctx)
   assert.NoError(t, err)

   books, err := client.Book.Query().All(ctx)
   assert.NoError(t, err)
   assert.Equal(t, len(books), 2)
}
```

Now run the go test and it should pass.

```bash
➜  book-catalog git:(main) ✗ go test -v ./ent
=== RUN   TestBookCatalog
--- PASS: TestBookCatalog (0.00s)
PASS
ok  	github.com/pratikjagrut/book-catalog/ent
```

## Setup GraphQL with Ent

Now, let's take our Ent setup with the SQLite database a step further by integrating GraphQL. This integration will provide a more advanced and practical way to handle database queries.

By integrating GraphQL with Ent, we can leverage GraphQL's flexible querying capabilities to efficiently retrieve data. With the help of [99designs /gqlgen](https://github.com/99designs/gqlgen), a powerful Go library, we can automatically generate GraphQL schemas and resolvers based on our Ent data models. This simplifies the process of building GraphQL APIs, allowing us to focus on defining schemas and writing resolver functions.

#### **Install and configure entgql**

Ent offers a convenient extension called [contrib/entgql](https://pkg.go.dev/entgo.io/contrib/entgql) that allows it to generate GraphQL schemas seamlessly. By installing and utilizing this extension, we can effortlessly generate GraphQL schemas based on our Ent data models.

To get started with contrib/entgql, you can install it by running the following command:

```bash
go get entgo.io/contrib/entgql
```

To enable query and mutation capabilities for the Autor and Book schema using Ent and contrib/entgql, we need to add the following annotations to the schema of both entities.

Add the following code to **ent/schema/author.go:**

```go
func (Author) Annotations() []schema.Annotation {
   return []schema.Annotation{
       entgql.QueryField(),
       entgql.Mutations(entgql.MutationCreate()),
   }
}
```

Add the following code to ***ent/schema/book.go***:

```go
func (Book) Annotations() []schema.Annotation {
   return []schema.Annotation{
       entgql.QueryField(),
       entgql.Mutations(entgql.MutationCreate()),
   }
}
```

With the annotations we added, we're telling contrib/entgql to generate essential GraphQL query and mutation fields for our entities to fetch and create data.

Let's create a new file named **ent/entc.go** and add the following content:

```go
//go:build ignore

package main

import (
    "log"

    "entgo.io/ent/entc"
    "entgo.io/ent/entc/gen"
    "entgo.io/contrib/entgql"
)

func main() {
    ex, err := entgql.NewExtension(
        // Generate a GraphQL schema for the Ent schema
        // and save it as "ent.graphql".
        entgql.WithSchemaGenerator(),
        entgql.WithSchemaPath("ent.graphql"),
    )
    if err != nil {
        log.Fatalf("failed to create entgql extension: %v", err)
    }
    opts := []entc.Option{
        entc.Extensions(ex),
    }
    if err := entc.Generate("./ent/schema", &gen.Config{}, opts...); err != nil {
        log.Fatalf("failed to run ent codegen: %v", err)
    }
}
```

In this code, we have a **main** function that performs the Ent code generation. We use the **entgql** extension to generate a GraphQL schema based on our Ent schema. The generated GraphQL schema will be saved as **ent.graphql**.

It's worth noting that the **ent/entc.go** file is ignored during the build process using a **//go:build ignore** tag. To execute this file, we will utilize the **go generate** command, which is triggered by the **generate.go** file in your project.

Remove the **ent/generate.go** file and create a new one in the **root of the project** with the following contents. In the next steps, gqlgen commands will be added to this file as well.

```go
package bookcatalog

//go:generate go run -mod=mod ./ent/entc.go
```

#### **Running schema generation**

Once you have installed and configured **entgql**, you can execute the code generation process by running the following command:\\

```bash
go generate .
```

***Note***: If you encounter package inconsistencies or missing package errors in your IDE after running ***go generate***, you can resolve them by running ***go mod tidy***. 

You'll notice a new file was created named **ent.graphql**:

```graphql
directive @goField(forceResolver: Boolean, name: String) on FIELD_DEFINITION | INPUT_FIELD_DEFINITION
directive @goModel(model: String, models: [String!]) on OBJECT | INPUT_OBJECT | SCALAR | ENUM | INTERFACE | UNION
type Author implements Node {
 id: ID!
 name: String!
 email: String!
 books: [Book!]
}
type Book implements Node {
 id: ID!
 title: String!
 genre: String!
 publicationDate: String!
 isbn: String!
 createdAt: Time!
 author: Author
}
"""
CreateAuthorInput is used for create Author object.
Input was generated by ent.
"""
input CreateAuthorInput {
 name: String!
 email: String!
 bookIDs: [ID!]
}
"""
CreateBookInput is used for create Book object.
Input was generated by ent.
"""
input CreateBookInput {
 title: String!
 genre: String!
 publicationDate: String!
 isbn: String!
 createdAt: Time
 authorID: ID
}
"""
...
```

#### **Install and configure gqlgen**

Install **99designs/gqlgen**:

```bash
go get github.com/99designs/gqlgen
```

To configure the **gqlgen** package, we need to create a **gqlgen.yml** file in the root directory of our project. This file is automatically loaded by **gqlgen** and provides the necessary configuration for generating the GraphQL server code.

Let's add the **gqlgen.yml** file to the root of our project and follow the comments within the file to understand the meaning of each configuration directive.

```yaml
# schema tells gqlgen where the GraphQL schema is located.
schema:
- ent.graphql

# resolver reports where the resolver implementations go.
resolver:
layout: follow-schema
dir: .

# gqlgen will search for any type names in the schema in these go packages
# if they match it will use them, otherwise it will generate them.

# autobind tells gqngen to search for any type names in the GraphQL schema in the
# provided package. If they match it will use them, otherwise it will generate new.
autobind:
- github.com/pratikjagrut/book-catalog/ent
- github.com/pratikjagrut/book-catalog/ent/author
- github.com/pratikjagrut/book-catalog/ent/book

# This section declares type mapping between the GraphQL and Go type systems.
models:
# Defines the ID field as Go 'int'.
ID:
  model:
    - github.com/99designs/gqlgen/graphql.IntID
Node:
  model:
    - github.com/pratikjagrut/book-catalog/ent.Noder
```

To inform Ent about the **gqlgen** configuration, we need to make a modification to the **ent/entc.go** file. We will pass the **entgql.WithConfigPath("gqlgen.yml")** argument to the **entgql.NewExtension()** function.

Open the **ent/entc.go** file and copy and paste the following code:

```go
//go:build ignore

package main

import (
   "log"

   "entgo.io/contrib/entgql"
   "entgo.io/ent/entc"
   "entgo.io/ent/entc/gen"
)

func main() {
   ex, err := entgql.NewExtension(
       // Tell Ent to generate a GraphQL schema for
       // the Ent schema in a file named ent.graphql.
       entgql.WithSchemaGenerator(),
       entgql.WithSchemaPath("ent.graphql"),
       entgql.WithConfigPath("gqlgen.yml"),
   )
   if err != nil {
       log.Fatalf("creating entgql extension: %v", err)
   }
   opts := []entc.Option{
       entc.Extensions(ex),
   }
   if err := entc.Generate("./ent/schema", &gen.Config{}, opts...); err != nil {
       log.Fatalf("running ent codegen: %v", err)
   }
}
```

Add the gqlgen generate command ***//go:generate go run -mod=mod*** [***github.com/99designs/gqlgen***](http://github.com/99designs/gqlgen) to the **generate.go** file:

```go
package bookcatalog

//go:generate go run -mod=mod ./ent/entc.go
//go:generate go run -mod=mod github.com/99designs/gqlgen
```

Now, we're ready to run go generate to trigger ent and gqlgen code generation. Execute the **go generate** command from the root of the project and you may notice new files created.

```bash
➜  book-catalog git:(main) ✗ tree -L 1
.
├── ent
├── ent.graphql
├── ent.resolvers.go
├── generate.go
├── generated.go
├── go.mod
├── go.sum
├── gqlgen.yml
└── resolver.go

1 directory, 8 files
```

## The Server

In order to build the GraphQL server, we need to set up the main schema Resolver as defined in **resolver.go**. The gqlgen library provides the flexibility to modify the generated Resolver and add dependencies to it.

To include the **ent.Client** as a dependency, you can add the following code snippet to **resolver.go**:

```go
package bookcatalog

import (
   "github.com/pratikjagrut/book-catalog/ent"

   "github.com/99designs/gqlgen/graphql"
)

// Resolver is the resolver root.
type Resolver struct{ client *ent.Client }

// NewSchema creates a graphql executable schema.
func NewSchema(client *ent.Client) graphql.ExecutableSchema {
   return NewExecutableSchema(Config{
       Resolvers: &Resolver{client},
   })
}
```

In the above code, we define a Resolver struct with a field named client of type **\*ent.Client**. This allows us to use the ent.Client as a dependency within our Resolver.

The NewSchema function is responsible for creating the GraphQL executable schema. It takes the ent.Client as a parameter and initializes the Resolver with it.

By adding this code to resolver.go, we ensure that our GraphQL server has access to the ent.Client and can utilize it for database operations.

To create the entry point for our GraphQL server, we'll start by creating a new directory called **server**. Inside this directory, we'll create a **main.go** file, which will serve as the entry point for our GraphQL server. 

```bash
mkdir server
touch server/main.go
```

Run the server using the command below, and open [localhost:8081](http://localhost:8081) to access the Graph*i*QL IDE.

```bash
go run server/main.go
```

![](https://lh4.googleusercontent.com/nkoFjRL5sD_hQ7yqOOqxHFqBxnhI3CaHb5Qk9IsY48yVS-pYVs3Fw_zcYpE3FftExo6Qjbb4f7CMnfi_hJxkT0NzC01zQYf4yibRb-sLClAvy11QBsP2tWaSymaVUWDPx_sA4gMGSmPFMatyiDUQYiY align="left")

### Query Data

When you run the GraphQL server using UI on [localhost:8081](http://localhost:8081) and send a query or mutation request, you will notice a "not implemented" message with a panic error displayed in the terminal. This occurs because we have not yet implemented the corresponding query and mutation functions in the GraphQL resolver.

```graphql
{
  books {
    title
    author {
      name
    }
    genre
    publicationDate
    isbn
  }
}
```

Output:

```graphql
{
  "errors": [
    {
      "message": "internal system error",
      "path": [
        "books"
      ]
    }
  ],
  "data": null
}
```

Just replace the following implementation in ***ent.resolver.go*** with the following code.

```go
// Authors is the resolver for the authors field.
func (r *queryResolver) Authors(ctx context.Context) ([]*ent.Author, error) {
   return r.client.Author.Query().All(ctx)
}

// Books is the resolver for the books field.
func (r *queryResolver) Books(ctx context.Context) ([]*ent.Book, error) {
   return r.client.Book.Query().All(ctx)
}
```

Now, restart the server and run the above query again in Graph*i*QL IDE. This time you should see the output below.

```bash
{
  books {
    title
    author {
      name
    }
    genre
    publicationDate
    isbn
  },
  
  authors{
    name
  }
}
```

Output:

```bash
{
  "data": {
    "books": [],
    "authors": []
  }
}
```

### Mutating Data

As observed in the previous example, our GraphQL schema currently returns an empty list of books and authors. To populate the list with data, we can leverage GraphQL mutations to create new book entries. Fortunately, Ent automatically generates mutations for creating and updating nodes and edges, making this process seamless.

We start by extending our GraphQL schema with custom mutations. Let's create a new file named **book.graphql** and add our Mutation type:

```graphql
type Mutation {
 # The input and the output are types generated by Ent.
 createAuthor(input: CreateAuthorInput!): Author
 createBook(input: CreateBookInput!): Book
}
```

Add the custom GraphQL schema to **gqlgen.yml** configuration:

```yaml
# schema tells gqlgen where the GraphQL schema is located.
schema:
 - ent.graphql
 - book.graphql
```

As you can see, gqlgen generated for us a new file named **book.resolvers.go** Copy and paste the following code snippet:

```go
// CreateAuthor is the resolver for the createAuthor field.
func (r *mutationResolver) CreateAuthor(ctx context.Context, input ent.CreateAuthorInput) (*ent.Author, error) {
   return r.client.Author.Create().SetInput(input).Save(ctx)
}

// CreateBook is the resolver for the createBook field.
func (r *mutationResolver) CreateBook(ctx context.Context, input ent.CreateBookInput) (*ent.Book, error) {
   return r.client.Book.Create().SetInput(input).Save(ctx)
}
```

Now, restart the server and run the following mutation query in Graph*i*QL IDE.

```graphql
mutation CreateBook {
  createAuthor(input: {name: "J. K. Rowling", email: "jk@gmail.com"}){id},
  createBook(input: {title: "The Ink Black Heart", 
    genre: "Mystery", 
    publicationDate:"30 August 2022", 
    isbn: "9780316413138",
    authorID: "1",
  }){id, title, author{name} }
}
```

Output:

```graphql
{
  "data": {
    "createAuthor": {
      "id": "1"
    },
    "createBook": {
      "id": "4294967297",
      "title": "The Ink Black Heart",
      "author": {
        "name": "J. K. Rowling"
      }
    }
  }
}
```

Now let’s query the data:

```graphql
{
  books {
    title
    author {
      name
    }
    genre
    publicationDate
    isbn
  },
  
  authors{
    name
    email
  }
}
```

Output:

```graphql
{
  "data": {
    "books": [
      {
        "title": "The Ink Black Heart",
        "author": {
          "name": "J. K. Rowling"
        },
        "genre": "Mystery",
        "publicationDate": "30 August 2022",
        "isbn": "9780316413138"
      }
    ],
    "authors": [
      {
        "name": "J. K. Rowling",
        "email": "jk@gmail.com"
      }
    ]
  }
}
```

## Conclusion

In this blog post, we discovered the power of combining GraphQL and Ent for creating efficient APIs in Go. GraphQL offers flexibility in working with data, while Ent simplifies database management. Integrating these tools enables seamless API development in Go.

We learned how to build a GraphQL server using Ent and explored essential features. Additionally, we leveraged GraphQL for querying and modifying data in the database.

I hope this blog post served as a valuable introduction to GraphQL and Ent, inspiring you to consider them for your future API development projects.

You can find the source code for the project on my [GitHub](https://github.com/pratikjagrut/book-catalog.git).

Thank you for reading, and happy coding!