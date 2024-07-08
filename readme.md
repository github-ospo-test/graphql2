---
title: GraphQL in 1 Hour
separator: <!--s-->
verticalSeparator: <!--v-->
# theme: moon
revealOptions:
  transition: "slide"
  center: true
  maxScale: 1.0
---

# GraphQL Crash Course test

## Ready in 1 hour or it's free

<sub>Made with ❤️ by @ajhenry</sub>

<!--s-->

## What is GraphQL

**Formal Definition**: GraphQL is [a standard](https://github.com/graphql/graphql-spec) for defining and querying data between a server and client(s).

This means a server that implements GraphQL can be queried by any client that implements GraphQL.

<!--v-->

**My Definition**: GraphQL is a query language that is built on top of different runtimes that implement a unified execution engine (this is where the standard is important!).

And why we have a Ruby GraphQL server that can be queried by a JavaScript client or Swift or Ruby or Go or Java or Python or ...

<!--v-->

## Why GraphQL

GraphQL has taken the microservice world by storm. It's a great way to define a contract between a client and a server. It's also a great way to define a contract _between services_.

<!--v-->

A few reasons:

- GraphQL is a query language, not a transfer protocol. This means it can be used with any transfer protocol (HTTP, gRPC, etc.)
- Establishes a **contract** between client and server
- Strongly typed (in the languages that allow that sort of thing)
- There is a single endpoint for all your data
- No data over-fetching (or under-fetching)
- Error handling is neat, there is no status code other than 200
- Self documenting

<!--v-->

## Why Not GraphQL

It's great and all but it's not a silver bullet. There are some downsides to GraphQL:

- Tedious if you have unshaped data (or maybe even impossible)
- Hard(er) to implement in some languages
- Might not need it, REST works just fine and is often easier to implement
- No N+1 problems 😨

<!--s-->

## GraphQL vs Other API Architectures

Why even bother with GraphQL? What's wrong with REST? Or gRPC? Or SOAP? Or CORBA? Or XML-RPC? Or...

<!--v-->

### GraphQL vs REST

REST is a great way to build APIs. It's simple, easy to understand, and easy to implement. It's also _very_ easy to get wrong.

Take this example:

> We own a bookshop, where users can checkout books and return them.

Here's our REST API

```http
GET /books
GET /books/:id
POST /books
POST /books/checkout/:id
```

<!--v-->

Now let's say we want to add audiobooks and movies to our shop.

We can't use the same API routes because we need to add a new endpoint for each new resource.

So we add

```http
GET /movies
GET /movies/:id
POST /movies
POST /movies/checkout/:id
```

<!--v-->

Now what if we wanted a unified top level `checkout` endpoint. Well now we have to add a new endpoint for that.

```http
POST /checkout/:id
```

Now what if we wanted to add a new field to our books? Or change the shape of the data? Or add a new field to our checkout? How will our clients know?

Now we have to implement versioning

```http
GET /v1/books
GET /v1/books/:id
POST /v1/books
POST /v1/books/checkout/:id
```

<!--v-->

So we can see that REST quickly spirals into a large collection of routes. It's not a bad architecture, but it's not a great one for APIs that might change often. It is exceptionally good at sending a lot of data over the wire.

GraphQL can DRY up some of these pitfalls

<!--v-->

A single endpoint

```http
POST /graphql
```

```graphql
type Book {
  id: ID!
  title: String!
}

type AudioBook {
  id: ID!
  title: String!
}

type Movie {
  id: ID!
  title: String!
}

type Resource = Book | AudioBook | Movie

type Query {
  books: [Book!]!
  book(id: ID!): Book!
  audioBooks: [AudioBook!]!
  audioBook(id: ID!): AudioBook!
  movies: [Movie!]!
  movie(id: ID!): Movie!
}

type Mutation {
  checkout(id: ID!): Resource!
}
```

<!--v-->

With this we can add new resources without adding new endpoints. We can also add new fields to our resources without breaking the contract.

```graphql
type Book {
  id: ID!
  title: String! @deprecated(reason: "Use `bookTitle` instead")
  bookTitle: String!
}

# Or even make a new book query altogether
type BookV2 {
  id: ID!
  bookTitle: String!
}

type ReturnBook = Book | BookV2

type Query {
  books: [Book!]!
  book(id: ID!): ReturnBook!
}
```

(Changing fields is a bit harder but still very manageable with the `@deprecated` directive)

<!--s-->

## Learning GraphQL

Welcome to the basics of GraphQL. This is not meant for you to become an expert, but to get a basic understanding of everything GraphQL has to offer.

<!--v-->

### Basics

There is a only a single endpoint for all your data. This is called the **entry point**. This is where all queries, mutations, and subscriptions are sent. It only accepts a `POST` request.

```http
curl -X POST \
-H "Content-Type: application/json" \
-d '{"query": "{ books { title } }"}' \
http://localhost:8080/graphql
```

<!--v-->

### Schema

The contract between client and server. This is where you define your types, queries, mutations, and subscriptions. These are the only actions that your client can perform.

```graphql
type Book {
  title: String!
}

type Query {
  books: [Book!]!
}

type Mutation {
  checkoutBook(id: ID!): Book!
}
```

<!--v-->

Clients **must** specify exactly what fields it needs from the server. This is called the **selection set**. The server will only respond with what the client asks for (more accurately, the server just throws away any data the client doesn't ask for).

From there, the server responds with a JSON response that matches the data shape of the selection set.

```graphql
query {
  books {
    title
  }
}
```

<!--v-->

```json
{
  "data": {
    "books": [
      {
        "title": "The Hobbit"
      },
      {
        "title": "The Lord of the Rings"
      }
    ]
  }
}
```

<!--v-->

There are no ways to select all the fields of a type. This is to prevent over-fetching of data.

```graphql
# Does NOT work
query {
  books {
    *
  }
}
```

There have been [many proposals](https://github.com/graphql/graphql-spec/issues/127) to add this functionality but then this would break the explicit contract between client and server.

And then add the fact that types can be self referential and you have a recipe for disaster.

<!--v-->

On another note, the GitHub GraphQL schema is over 60k lines long!

A funny excerpt from the schema file

```graphql
"""
The possible durations that a user can be blocked for.
"""
enum UserBlockDuration {
  """
  The user was blocked for 1 day
  """
  ONE_DAY

  """
  The user was blocked for 30 days
  """
  ONE_MONTH

  """
  The user was blocked for 7 days
  """
  ONE_WEEK

  """
  The user was blocked permanently
  """
  PERMANENT

  """
  The user was blocked for 3 days
  """
  THREE_DAYS
}
```

<!--v-->

#### Introspection

A note mentioned earlier was that GraphQL is self-documenting. This is because documentation is built into the schema. This is called `introspection`. A client can view the schema a server offers at any time.

All docstrings are available to the client. This is why you can use tools like GraphiQL and GraphQL Playground to explore a GraphQL API.

```graphql
"""
Book is a standard book that you'd see on a self
"""
type Book {
  """
  The title of the book
  """
  title: String!
}
```

<!--v-->

There are some special types associated with introspection.

`__schema` and `__type`

These allow you to query the schema itself and figure out what the type of an object is.

```graphql
query {
  __schema {
    types {
      name
    }
  }
}
```

<!--v-->

### GraphQL Types

In addition to the base types that GraphQL provides, you can define your own types. These types can be objects, enums, unions, interfaces, and scalars.

With these constructs, you can build out your schema to be as complex as you need it to be.

<!--v-->

#### Objects

These are the foundation of all of the types in a schema. They are equivalent to an object in JSON.

```graphql
type Book {
  title: String! # ! denotes non-nullable
  topics: [String] # [] denotes an array
  authors: [Author!]! # denotes a non-nullable array of non-nullable objects
}
```

<!--v-->

#### Enums

These are a special type of object that can only have a finite number of values. They are equivalent to an enum in TypeScript. They are returned as strings in the response.

```graphql
enum BookStatus {
  AVAILABLE
  CHECKED_OUT
}
```

<!--v-->

#### Unions

These are a special type of object that can be **one** of many types. They are equivalent to a union in TypeScript.

```graphql
type Book {
  title: String!
  pages: Int!
}

type AudioBook {
  title: String!
  length: Int!
}

union Resource = Book | AudioBook
```

<!--v-->

Unions are a funky type in GraphQL. They are useful for when you have a field that can be one of many types, like when performing a Global Search. For example, if you have a `Resource` type that can be a `Book`, `AudioBook`, or `Movie`. You can use a union to represent this. However, this represents a few problems.

```graphql
union Resource = Book | AudioBook | Movie
```

How do we know what type of Object we're getting back and how do we query the fields on that object?

<!--v-->

This is where the `__typename` field comes in. This is a special field that is available on all types. It is a string that represents the type of the object.

<!--v-->

Let's say we have a `Search` query that returns a list of `Resource`s. We can use the `__typename` field to figure out what type of resource we are getting back.

```graphql
query {
  search(query: "The Hobbit") {
    __typename
  }
}
```

<!--v-->

This is the data we'd get back

```json
{
  "data": {
    "search": [
      {
        "__typename": "Book"
      },
      {
        "__typename": "AudioBook"
      }
    ]
  }
}
```

<!--v-->

But then how do we query the fields on the `Book` or `AudioBook`? This is where `Fragments` come into play

<!--v-->

#### Fragments

Fragments are a way to define a set of fields that you want to query on a type. This is useful for when you have a union or interface type and you want to query the fields on the underlying type.

```graphql
query {
  search(query: "The Hobbit") {
    __typename
    ... on Book {
      title
    }
    ... on AudioBook {
      title
    }
  }
}
```

<!--v-->

Fragments can be inline like the one above or they can be defined separately.

```graphql
fragment BookFields on Book {
  title
  pages
}

fragment AudioBookFields on AudioBook {
  title
  length
}
```

```graphql
query {
  search(query: "The Hobbit") {
    __typename
    ...BookFields
    ...AudioBookFields
  }
}
```

<!--v-->

That gives us this response now

```json
{
  "data": {
    "search": [
      {
        "__typename": "Book",
        "title": "The Hobbit",
        "pages": 300
      },
      {
        "__typename": "AudioBook",
        "title": "The Hobbit",
        "length": 300
      }
    ]
  }
}
```

<!--v-->

#### Interfaces

Interface are a special type that allows you to define a set of fields that other types can implement. They are equivalent to an interface in TypeScript or go.

<!--v-->

Notice that both `Book` and `AudioBook` implement the `LibraryItem` interface. This means that they must implement all of the fields defined in the interface. This creates a unified set of fields that can be queried on any type that implements the interface.

```graphql
interface LibraryItem {
  id: ID!
  title: String!
}

type Book implements LibraryItem {
  id: ID!
  title: String!
  pages: Int!
}

type AudioBook implements LibraryItem {
  id: ID!
  title: String!
  length: Int!
}
```

<!--v-->

<sub>GitHub actually does this with the `Node` interface that nearly all of their types implement.</sub>

```graphql
query UserSearch {
  node(id: "MDQ6VXNlcjI0OTIzNDA2") {
    ... on User {
      id
      login
    }
  }
}
```

This means you can search for anything globally

<!--v-->

#### Directives

Directives are a powerful feature to modify the behavior of a Schema. They are equivalent to decorators in TypeScript/Python.

Schema

```graphql
directive @deprecated(reason: String!) on FIELD_DEFINITION

type Query {
  books: [Book!]! @deprecated(reason: "Use `searchBooks` instead")
  searchBooks(keyword: String!): [Book!]!
}
```

<!--v-->

You can completely change the behavior of a resolver with a directive. For example, you can add caching to a field with a directive.

```graphql
directive @cache(ttl: Int!) on FIELD_DEFINITION

type Query {
  books: [Book!]! @cache(ttl: 60)
}
```

Which then the resolver\* can use to cache the result of the query.

<!--v-->

Or even add ownership to a field — this is especially useful for a company with a lot of distributed teams 😉

```graphql
directive @ownedBy(team: String!) on FIELD_DEFINITION

type Query {
  books: [Book!]! @ownedBy(team: "@library/backend")
}
```

<!--v-->

#### All Types

Full list of all types:

```graphql
# defines an object
type Book {
  title: String!
}

# defines a non-nullable type
type Book {
  title: String!
}

# defines an array
type Authors = [String!]!

# defines a query
type Query {
  books: [Book!]!
}

# defines a mutation
type Mutation {
  checkoutBook(id: ID!): Book!
}

# defines a subscription
type Subscription {
  bookCheckedOut: Book!
}

# defines a scalar type
scalar Date

# defines an enum type
enum BookType {
  HARDCOVER
  PAPERBACK
}

# defines an interface type
interface Book {
  title: String!
}

# defines a union type
union Book = HardcoverBook | PaperbackBook

# defines an input type
input BookInput {
  title: String!
}

# defines a directive
directive @deprecated(reason: String!) on FIELD_DEFINITION
```

<!--s-->

### Queries

A query is a request for data. It is the only way to get data from a GraphQL server. Queries ~~are~~ _should be_ read-only.

```graphql
type Query {
  books: [Book!]!
}
```

Notice that we use the `Query` type in the schema definition to put our queries. This is a special type in the schema however, this is implemented the same way the rest of the types are.

<!--v-->

Fields can take arguments and return values. Arguments are defined in the schema and are available to the client to use.

```graphql
type Query {
  books(keyword: String, limit: Int): [Book!]!
}
```

<!--v-->

#### Aliases

You can alias any field in a query. This is useful for when you want to query the same field multiple times with different arguments.

schema

```graphql
type Query {
  books(keyword: String): [Book!]!
}
```

Query

```graphql
query {
  LoTR: books(keyword: "The Hobbit") {
    title
  }
  HP: books(keyword: "Harry Potter") {
    title
  }
}
```

<!--v-->

Response

```json
{
  "LoTR": [
    {
      "title": "The Hobbit"
    }
  ],
  "HP": [
    {
      "title": "Harry Potter and the Sorcerer's Stone"
    }
  ]
}
```

<!--v-->

You can also alias queries and mutations

```graphql
query LoTR {
  books(keyword: "The Hobbit") {
    title
  }
}

query HP {
  books(keyword: "Harry Potter") {
    title
  }
}
```

<!--v-->

#### Variables

We can also add variables to our queries. This is useful for when we want to pass in dynamic values to our queries.

```graphql
query Books($keyword: String, $limit: Int) {
  books(keyword: $keyword, limit: $limit) {
    title
  }
}
```

Notice that we have to type our variables. All variables must start with a `$`.

<!--v-->

Variables are defined in the query and then passed in as an object to the `execute` function.

```js
const query = `
  query Books($keyword: String, $limit: Int) {
    books(keyword: $keyword, limit: $limit) {
      title
    }
  }
`;

const variables = {
  keyword: "The Hobbit",
  limit: 10,
};

const result = await execute({ query, variables });
```

<!--s-->

### Mutations

A mutation is a request to change data. It is the only way to change data on a GraphQL server. Mutations ~~are~~ _should be_ write-only. They are defined the same way as queries and require a selection set to execute.

<!--v-->

Mutation

```graphql
mutation {
  checkoutBook(id: "123") {
    title
  }
}
```

Response

```json
{
  "data": {
    "checkoutBook": {
      "title": "The Hobbit"
    }
  }
}
```

<!--v-->

Arguments are crucial for updating data and use the `input` keyword to define an input type. This is handy if you want to have a grouping of keyword arguments instead of positional arguments — otherwise you can just use a scalar type.

```graphql
input CheckoutBookInput {
  id: ID!
  date: Date
}

type Mutation {
  checkoutBook(input: CheckoutBookInput!): [Book!]!
  deleteBook(id: ID!): Book!
}
```

<!--v-->

You **cannot** use a regular type as an input type. And vice versa.

```graphql
type CheckoutBookInput {
  id: ID!
  date: Date
}

type Mutation {
  # Invalid!
  checkoutBook(input: CheckoutBookInput!): [Book!]!
  deleteBook(id: ID!): Book!
}
```

This is an intentional limitation of the GraphQL schema system. This is to prevent the overlap of Unions and Interfaces within Input Types.

<!--v-->

A fun thought question, what would happen if we did this?

```graphql
input CheckoutBookInput {
  id: ID!
  date: Date
  additionalInfo: CheckoutBookInput!
}
```

<!--v-->

Circular input references are actually allowed, albeit they have to be null.

```graphql
input CheckoutBookInput {
  id: ID!
  date: Date
  additionalInfo: CheckoutBookInput # Goodbye !
}
```

<!--s-->

### Subscriptions

A subscription is a request to get data that changes over time. Subscriptions are Queries under the hood that utilize WebSockets to send data to the client (or polling when WebSockets aren't available).

<!--v-->

Query

```graphql
type Subscription {
  bookCheckedOut: Book!
}
```

Response

```jsonc
// Initially
{
  "data": {
    "bookCheckedOut": null
  }
}

// Eventually
{
  "data": {
    "bookCheckedOut": {
      "title": "The Hobbit"
    }
  }
}
```

<!--s-->

### Resolvers

A resolver (or **Field Resolver**) is a function that returns data for a field. Resolvers are functions that are called by the GraphQL execution engine. They are responsible for fetching data from a database, cache, or other source and _fulfilling the contract between client and server_.

```typescript
const resolvers = {
  Query: {
    books: () => {
      return [
        {
          title: "The Hobbit",
        },
        {
          title: "The Lord of the Rings",
        },
      ];
    },
  },
};
```

<!--v-->

Resolvers take arguments. These arguments are passed in as the second argument to the resolver function.

```typescript
const resolvers = {
  Query: {
    books: (_, { keyword, limit }) => {
      // Do stuff with those arguments
    },
  },
};
```

<!--v-->

Resolvers can be asynchronous. The execution engine is smart enough to wait for them to finish before sending the result. This is useful for fetching data from a database or other sources.

```typescript
const resolvers = {
  Query: {
    books: async (_, { keyword, limit }) => {
      const books = await db.getBooks(keyword, limit);
      return books;
    },
  },
};
```

<!--v-->

Resolvers (Field Resolvers) can also fulfill singular fields. This is useful for transforming data or fetching data from a different source.

```typescript
const resolvers = {
  Query: {
    books: async (_, { keyword, limit }) => {
      const books = await db.getBooks(keyword, limit);
      return books;
    },
  },
  // Executes for every Book type in the schema
  Book: {
    title: (book) => {
      // MAKE BOOKS LOUDER
      return book.title.toUpperCase();
    },
  },
};
```

<!--s-->

## Execution Engine

The execution engine is responsible for taking a query and turning it into a response. It does this by traversing the query and calling the resolvers for each field. It also handles things like authorization and error handling.

<!--v-->

### Resolving Data

Each field in a schema is backed by a function (Resolver) that is provided by the GraphQL server. This function is responsible for returning the data for that field. The execution engine calls these functions in a specific order to resolve the data for a query.

<!--v-->

### Root Resolvers

Root resolvers are the entry point for a query. They are the first resolvers that are called by the execution engine. Typically, there are three root resolvers: `Query`, `Mutation`, and `Subscription`.

```typescript
Query: {
  books(obj, args, context, info) {
    return context.db.getAllBooks(args.limit);
  }
}
```

<!--v-->

A resolver takes 4 arguments:

**obj** — The previous object, useful for nested resolvers.

**args** — The arguments provided to the field in the GraphQL query.

**context** — A value which is provided to every resolver and holds important contextual information like the currently logged in user, or access to a database.

**info** — A value which holds field-specific information relevant to the current query as well as the schema details.

<!--v-->

### Field Resolvers

Root resolvers are the entry point for a query, so what about the rest of the nested data we requested?

This is where field level resolvers come into play.

<!--v-->

Let's go back to our example of books and add some complexity.

```graphql
type Book {
  title: String!
  pages: Int!
  authors: [Author!]!
}

type Author {
  name: String!
  books: [Book!]!
}

type Query {
  books: [Book!]!
}
```

Now we have a nested field on the `Book` type called `authors`. This field is backed by a resolver that returns an array of `Author` types.

<!--v-->

Here's what the resolvers would look like for this schema.

```typescript
const resolvers = {
  Query: {
    books(obj, args, context, info) {
      return context.db.getAllBooks(args.limit);
    },
  },
  Book: {
    authors(book, args, context, info) {
      return context.db.getAuthorsForBook(book.id);
    },
  },
  Author: {
    books(author, args, context, info) {
      return context.db.getBooksForAuthor(author.id);
    },
  },
};
```

<!--v-->

How this data gets filled in order.

1. The `books` query is called.

   1. This makes a database call to get all the books: `select * from books limit ${limit}`

      ```json
      {
        "books": [
          {
            "book_id": 1,
            "title": "The Hobbit"
          },
          {
            "book_id": 2,
            "title": "The Lord of the Rings"
          }
        ]
      }
      ```

<!--v-->

2. Each book is passed to the `Book` resolver.

   1. This makes a database call to get all the authors for a book: `select * from authors where book_id = ${book.id}`

      ```json
      {
        "books": [
          {
            "book_id": 1,
            "title": "The Hobbit",
            "authors": [
              {
                "author_id": 1,
                "name": "J.R.R. Tolkien"
              }
            ]
          },
          {
            "book_id": 2,
            "title": "The Lord of the Rings",
            "authors": [
              {
                "author_id": 1,
                "name": "J.R.R. Tolkien"
              }
            ]
          }
        ]
      }
      ```

<!--v-->

3. Each author is passed to the `Author` resolver.

   1. This makes a database call to get all the books for an author:
   2. `select * from books where author_id = ${author.id}`

      ```json
      {
        "books": [
          {
            "book_id": 1,
            "title": "The Hobbit",
            "authors": [
              {
                "author_id": 1,
                "name": "J.R.R. Tolkien",
                "books": [
                  {
                    "book_id": 1,
                    "title": "The Hobbit"
                  },
                  {
                    "book_id": 2,
                    "title": "The Lord of the Rings"
                  }
                ]
              }
            ]
          },
          {
            "book_id": 2,
            "title": "The Lord of the Rings",
            "authors": [
              {
                "author_id": 1,
                "name": "J.R.R. Tolkien",
                "books": [
                  {
                    "book_id": 1,
                    "title": "The Hobbit"
                  },
                  {
                    "book_id": 2,
                    "title": "The Lord of the Rings"
                  }
                ]
              }
            ]
          }
        ]
      }
      ```

<!--v-->

Since we have no restrictions on query depth (only execution time limit), a query like this is perfectly valid

```graphql
query {
  books {
    title
    authors {
      name
      books {
        title
        authors {
          name
          books {
            title
            authors {
              name
            }
          }
        }
      }
    }
  }
}
```

<!--v-->

### N+1 Problem

The N+1 problem is a common problem in GraphQL. It occurs when a query is made that requires multiple requests to the database. If a field is backed by a resolver that makes a database request, then _each field_ will make a database request.

<!--v-->

Take the following Schema

```graphql
type Book {
  title: String!
  author: Author!
}

type Author {
  name: String!
}

type Query {
  books: [Book!]!
}
```

<!--v-->

Now take the following GraphQL query

```graphql
query {
  books {
    title
    author {
      name
    }
  }
}
```

How many database calls do we expect for this query with 2 books?

<!--v-->

This would execute the following SQL statement(s)

```sql
-- Top level resolver (books)
select * from books;
-- First auth resolver (author)
select * from authors where book_id = 1;
-- Second auth resolver (author)
select * from authors where book_id = 2;
```

This is a total of 3 database calls for 2 books. This is a problem because it can cause performance issues.

<!--v-->

But where does the N+1 meaning come from?

```sql
-- First call (1)
select * from books;

-- Total of books (N)
select * from authors where book_id = 1;
select * from authors where book_id = 2;

-- Total of 1+N or better known as N+1
```

<!--v-->

However this This is where the `DataLoader` comes in (but won't be covered here since it's implementation specific).

But essentially we frontload the database calls and then batch them together to form something like this.

```sql
select * from books where id in (1, 2)
select * from authors where book_id in (1, 2)
```

<!--v-->

### Errors

GraphQL has a very specific way of handling errors. It's not like a REST API where you can return a 500 error and call it a day. GraphQL has a quite clever approach.

```graphql
query {
  books {
    title
    author {
      name
    }
  }
}
```

Oh no! Our Author cluster is down and can't access any authors. What should we do?

<!--v-->

Since author is **non-nullable** we can't just return `null`. We need to return an error. But how do we do that?

<!--v-->

GraphQL will try to fill as much of the request it can, and then `null` out the fields that can't be filled due to an error. From there, a new field appears on our response objects: `errors`.

```json
{
  "data": {
    "books": [
      {
        "title": "The Hobbit",
        "author": null
      }
    ]
  },
  "errors": [
    {
      "message": "Author cluster is down",
      "locations": [
        {
          "line": 4,
          "column": 5
        }
      ],
      "path": ["books", 0, "author"]
    }
  ]
}
```

This allows the UI to still show parts it can but also show the errors that occurred.

<!--s-->

## Fun Exercises

Let's do GraphQL exercises!

_TOGETHER_ 😈

<!--v-->

Let's find out all the types we can query from the schema (think back to the introspection query)

<!--v-->

```graphql
query Schema {
  __schema {
    types {
      name
      kind
    }
  }
}
```

<!--v-->

Let's find out if we are actually staff members

<!--v-->

```graphql
query isStaff {
  viewer {
    login
    name
    company
    isSiteAdmin
  }
}
```

<!--v-->

```json
{
  "data": {
    "viewer": {
      "login": "ajhenry",
      "name": "Andrew Henry",
      "company": "@github",
      "isSiteAdmin": false
    }
  }
}
```

Fun fact, this appears to be a bug since the same query on the REST API returns `true` 😉

```jsonc
// https://api.github.com/users/ajhenry

{
  "login": "ajhenry",
  "name": "Andrew Henry",
  "company": "@github",
  "site_admin": true
}
```

<!--v-->

Let's find out what the total count of people we can sponsor is

🤑

<!--v-->

```graphql
query Money {
  sponsorables {
    totalCount
  }
}
```

<!--v-->

Create a new repo

<!--v-->

```graphql
mutation CreateRepo {
  createRepository(
    input: {
      visibility: PRIVATE
      name: "new-repo"
      ownerId: "MDQ6VXNlcjI0OTIzNDA2"
    }
  ) {
    repository {
      nameWithOwner
    }
  }
}
```

<!--v-->

Let's delete the repository we created

<!--v-->

Too bad, this isn't available in the GraphQL API 😢

Have to use REST

```bash
curl -X DELETE \
  -H "Accept: application/vnd.github.v3+json" \
  -H "Authorization: token ${token}" \
   https://api.github.com/repos/${username}/${reponame}
```

<!--v-->

Let's find some repository metrics

<!--v-->

```graphql
query Metrics {
  repository(owner: "ajhenry", name: "react-google-slides") {
    metrics {
      openIssueCount
      openPullRequestCount
      closedIssueCount
      closedPullRequestCount
      mergedPullRequestCount
      # lastContributionDate
    }
  }
}
```

This one is interesting since it uses a feature flag to hide the scheme.

We have to add the following header

```json
{
  "GraphQL-Features": "ospo_metrics_api"
}
```

<!--s-->

# Thank you!

<sub>Sorry no refunds 🙂</sub>

<!--s-->

## Internal Workings

What you probably didn't need (or want) to know about how GraphQL works under the hood

Fair warning, this goes super in depth and has a heavy emphasis on compilers 🤓

<!--v-->

### Part 1: Schema

How does a schema get created and how does it work?

How does GraphQL know what types are available from this?

```graphql
type Book {
  title: String!
}

type Query {
  books: [Book!]!
}
```

<!--v-->

#### Lexer

Since GraphQL is a typed language, it needs to know what types are available. This is where the **lexer** comes in. The lexer is responsible for taking the schema and turning it into a list of tokens. These tokens are then passed to the parser.

GraphQL is also a _context-free grammar_ language. This means that the order of the tokens doesn't matter. This is why you can define types in any order you want. It will produce the same result after being parsed.

<!--v-->

The GraphQL spec explicitly lists out all the grammar for the language and how to interpret it. (This is why GraphQL is so easy to implement in other languages).

Our _very simple_ example

```graphql
type Book {
  title: String!
}
```

<!--v-->

Lexer output (in JSON)

```json
[
  {
    "kind": "NAME",
    "value": "type",
    "line": 1,
    "column": 1
  },
  {
    "kind": "NAME",
    "value": "Book",
    "line": 1,
    "column": 6
  },
  {
    "kind": "BRACE_L",
    "value": null,
    "line": 1,
    "column": 11
  },
  {
    "kind": "NAME",
    "value": "title",
    "line": 2,
    "column": 3
  },
  {
    "kind": "COLON",
    "value": null,
    "line": 2,
    "column": 8
  },
  {
    "kind": "NAME",
    "value": "String",
    "line": 2,
    "column": 10
  },
  {
    "kind": "BANG",
    "value": null,
    "line": 2,
    "column": 16
  },
  {
    "kind": "BRACE_R",
    "value": null,
    "line": 3,
    "column": 1
  }
]
```

This is what is passed to the parser to produce an AST.

<!--v-->

#### Parser and ASTs

There is where ASTs are used in GraphQL

Stephen Snider once said: `[AST is a] fancy way of saying heavily nested objects`. And that statement holds very true here.

Here's what the schema gets compiled into once run through the AST parser

There's actually nothing stopping you from writing a GraphQL schema like this.

### Part 2: Validation

How does GraphQL validate queries?

### Part 3: Execution

How does GraphQL execute queries?

### Part 4: Response

How does GraphQL respond to queries?
