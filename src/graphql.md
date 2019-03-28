---
title: GraphQL
revealOptions:
  transition: "slide"
---

<style>
  .cols {
    display: flex;
  }
  .col {
    flex: 1;
  }

  .rows {
    display: flex;
    flex-direction: column;
  }

  .arrow.arrow {
    white-space: none;
    margin: 0 16px;
    align-self: center;
    justify-self: center;
  }

  .small.small {
    font-size: 24px;
  }

  .tall.tall * {
    max-height: 600px;
  }

  .logo.logo {
    border: 0;
    width: 100px;
    background: none;
    box-shadow: none;
  }
</style>

<!-- .slide: class="main-slide" -->

# GraphQL
<img class="logo" src="./assets/graphql-logo.svg" />
<!-- <div class="name">
  Andi | Developer
</div> -->

---

### GraphQL is
A query language for APIs and runtime for fulfilling those queries with your existing data.  <!-- .element: class="quote" -->

- Gives clients the power to ask for what they need <!-- .element: class="small" -->
- Makes it easier to evolve APIs over time <!-- .element: class="small" -->
- Enables powerful developer tools. <!-- .element: class="small" -->

---

### HTTP Rest example <!-- .element: class="heading" -->

`GET /migration-plans/<id>`

```json
[{ "id": 1, "name": "plan1" }, { "id": 1, "name": "plan2" }]
```

`GET /progress/<id>`

```json
{
  "progress": 50,
  "status": "IN_PROGRESS"
}
```

Note: To get plan and migration for specific id,
2 round trips are needed for each endpoint

---

### GraphQL example <!-- .element: class="heading" -->

`(POST|GET) /graphql` <!-- .element: class="fragment" -->

```graphql
query {
  migrationPlans(id: '5') { # equal to GET /migration-plans/<id>
    id
    name
  }
  progress(id: '5') { # equal to GET /progress/<id>
    progress
    status
  }
}
```

<!-- .element: class="fragment" -->

<div class="fragment">
  `POST → { "query": "..." }`
</div>
<div class="fragment">
  `GET → /graphql?query=...`
</div>

----

<!-- .slide: data-transition="zoom concave convex" -->

Response:

```json
{
  "data": {
    "migrationPlans": [
      { "id": 1, "name": "plan1" },
      { "id": 1, "name": "plan2" }
    ],
    "progress": {
      "progress": 50,
      "status": "IN_PROGRESS"
    }
  }
}
```

Contains both plan data and progress data

Note: For GraphQL, first define the query and the fields that's needed
(include both plan query and progress query),
then send the request to the graphql server endpoint, either by using POST or GET,
then the GraphQL server will return the data.
Only 1 request needed to get both data

---

### Client declare shape <!-- .element: class="heading" -->

<div class="rows">
  <div class="cols">
    <pre class="fragment"><code data-trim>
      query {
        progress(id: '5') {
          progress
          status
        }
      }
    </code></pre>
    <div class="fragment arrow">→</div>
    <pre class="fragment"><code data-trim>
      {
        "data": {
          "progress": {
            "progress": 50,
            "status": "IN_PROGRESS"
          }
        }
      }
    </code></pre>
  </div>
  <hr />
  <div class="cols">
    <pre class="fragment"><code data-trim>
      query {
        progress(id: '5') {
          progress
        }
      }
    </code></pre>
    <div class="fragment arrow">→</div>
    <pre class="fragment"><code data-trim>
      {
        "data": {
          "progress": {
            "progress": 50,
          }
        }
      }
    </code></pre>
  </div>
</div>

---

### How does it work <!-- .element: class="heading" -->

GraphQL is a spec

NodeJS implementation:

- [graphql-js](https://github.com/graphql/graphql-js)
- [apollo-server](https://github.com/apollographql/apollo-server)

Note: Depends on GraphQL implementation framework

---

### Demo

---

### Mutation <!-- .element: class="heading" -->

Technically any query could be implemented to cause a data write. Convention that any operations that cause writes named a mutation. <!-- .element: class="quote" -->

----

```graphql
type Mutation {
  createMessage(id: ID!, input: MessageInput): Message
}

input MessageInput { # can't have fields that are other objects
  content: String
  otherParam: String
}

type Message {
  id: ID!
  content: String
}
```

Note: https://graphql.org/graphql-js/mutations-and-input-types/

---

### Subscription <!-- .element: class="heading" -->

```graphql
type Subscription {
  postAdded: Post
}
```

Websocket

Pub sub -> Redis, Postgres, Kafka, etc

---

### Interface <!-- .element: class="heading" -->

```graphql
interface Progress {
  progress: Float!
}

type PlanProgress implements Progress {
  progress: Float!
  planId: ID!
}

type TaskProgress implements Progress {
  progress: Float!
  taskId: ID!
}

type Query {
  progress: Progress
}
```
<!-- .element: class="tall" -->

----

```graphql
progress {
  progress
  ... on PlanProgress {
    planId
  }

  ... on TaskProgress {
    taskId
  }
}
```

Inline fragment

---

### Union <!-- .element: class="heading" -->

```graphql
union CanBeQueried = Progress | Plan
```

---

### Nested query <!-- .element: class="heading" -->

```graphql
query {
  plan(id: "123") {
    name
    tasks(first: 10, offset: 2) {
      name
    }
  }
}
```

---

### Caching <!-- .element: class="heading" -->

Caching is not straightforward
- Default is POST
- There are too many query combinations

Note: https://medium.freecodecamp.org/five-common-problems-in-graphql-apps-and-how-to-fix-them-ac74d37a293c

----

### CDN <!-- .element: class="heading" -->

Akamai is hashing the query in some way to make sure order and format does not matter ([source](https://developer.akamai.com/blog/2018/10/29/overview-graphql-query-parsing-and-caching-edge))

[FastQL](https://fastql.io/) can also cache partial query

----

### Persisted query <!-- .element: class="heading" -->

Facebook is using it

[Details](https://blog.apollographql.com/persisted-graphql-queries-with-apollo-client-119fd7e6bba5)

----

### Persisted query <!-- .element: class="heading" -->

**Downside**

- Query must be resolved in build time in client
- For new query added, must be added to client before being used

[Automatic persisted query apollo](https://blog.apollographql.com/improve-graphql-performance-with-automatic-persisted-queries-c31d27b8e6ea)

Note: https://graphql.org/learn/caching/

----

### Data loader <!-- .element: class="heading" -->

https://github.com/graphql/dataloader

- Batching
  - 1 query hitting database 50 times
  - Batch 50 database query into 1
- Caching
  - API has been called before, return cache
- Ideally done per request, so no stale data
- Useful if query gets large

----

### Redis/Memcache <!-- .element: class="heading" -->

Cache response from source API instead

---

### Introspection <!-- .element: class="heading" -->

Query to retrieve available queries in the GraphQL server

```graphql
{
  __type(name: "Droid") {
    name
    fields {
      name
      type {
        name
        kind
        ofType {
          name
          kind
        }
      }
    }
  }
}
```
<!-- .element: class="fragment tall" -->

----

```json
{
  "data": {
    "__type": {
      "name": "Droid",
      "fields": [
        {
          "name": "id",
          "type": {
            "name": null,
            "kind": "NON_NULL",
            "ofType": {
              "name": "ID",
              "kind": "SCALAR"
            }
          }
        },
        ...
      ]
    }
  }
}
```
<!-- .element: class="tall" -->

----

Introspection is very useful

[apollo-tooling](https://github.com/apollographql/apollo-tooling)

```sh
apollo client:codegen [OUTPUT]
```

----

Conversion to Typescript

<div class="cols">
  <pre class="fragment"><code data-trim>
    type User {
      id: ID!
      username: String!
      email: String
    }
  </code></pre>
  <div class="fragment arrow">→</div>
  <pre class="fragment"><code data-trim>
    export interface User {
      id: string;
      username: string;
      email?: string | null;
    }
  </code></pre>
</div>

---

### Why <!-- .element: class="heading" -->

- Declarative data fetching <!-- .element: class="fragment" -->
- Documented by default <!-- .element: class="fragment" -->
- Less payload for client in multiple platform and versions (web, ios, android, v1, v2) <!-- .element: class="fragment" -->
- Move mapping to server, simpler client code 😈 <!-- .element: class="fragment" -->
- Schema stiching multiple GraphQL server to create gateway <!-- .element: class="fragment" -->
- Introspection -> generate docs, types etc <!-- .element: class="fragment" -->

---

### Note <!-- .element: class="heading" -->

- Apollo is a complete framework <!-- .element: class="fragment" -->
  - Apollo client (react, etc), no need for redux for data <!-- .element: class="fragment" -->
  - Caching, Batching, Refetching, Pagination, Optimistic update <!-- .element: class="fragment" -->
  - Monitoring, tracing, metrics per resolver <!-- .element: class="fragment" -->
- GraphQL can be used for server to server  <!-- .element: class="fragment" -->
- Maybe migration GraphQL server (MO, UMS, JMT all behind GraphQL) <!-- .element: class="fragment" -->

---

### Example in company <!-- .element: class="heading" -->

Note: show confluence/jira graphiql

---

## The end

- Code sandbox <!-- .element: class="small" -->
- reveal-md <!-- .element: class="small" -->

<!-- .slide: class="main-slide" -->

---

- Context/Header
  - Authentication/Authz -> specific to apollo, but everything is the same, it's just a network request
  - Throw authentication error
- Directives @
  - @deprecated -> not part of official spec
  - @skip
  - @include
- named query
- Mutation
- Fragment
- Subscription
  - Pub/sub
  - Redis/kafka/other pubsub engine
- Interface
- Union type
- Nested query
- Introspection
- Why
- Caching strategy
- Batching
  - Use DataLoader from facebook
    - e.g. getAuthor, load posts, and getPost load author (instead calling api twice, use data loader)
  - Memcached/redis for storing BE responses
- Schema stiching

- Error if no resolver from gql
- Error status
- Apollo client
  - cache by default with apollo store
  - batching query
- Duplicated data and resolver, but still fetching same data, e.g. for deprecation
