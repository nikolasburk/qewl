# Qewl

**Qewl** (pronounced: /kuːl/) is a **GraphQL Application Framework**.
It is inspired by Koa, built on Apollo Server, and turns your GraphQL endpoint into a Koa-like application, with support for context, middleware, and many other features.

It is great for working with **remote schemas** and **schema stitching** and things like **authentication**. But it also makes it very easy to set up a GraphQL Server **from scratch**.

## Concepts

Qewl crosses the bridge between commonly used patterns for HTTP middleware frameworks like Koa and Express, and GraphQL endpoints.
A GraphQL server is composed of three components:
- GraphQL schemas
- Resolvers
- Context

Qewl uses these same three components:
- GraphQL schemas define the **routes** of our server
- Resolvers define the **implementation** of these routes
- Middleware is used to construct the **context** passed to every resolver, and to add **common functionality** to the resolvers

## Installation
Qewl requires **node v7.6.0** or higher for ES2015 and async function support.
```bash
$ yarn add qewl
```

## Getting started

This example demonstrates the basics of setting up your own GraphQL server. If you are already familiar with the concepts involved, you can skip to the **Deep Dive example** [here](#deep-dive).

Let's start with a basic Express server:
```ts
import * as express from 'express'

async function run() {

  const app = express()

  app.listen(3000)
}

run()
```
Now, let's create a route for our GraphQL server:
```diff
import * as express from 'express'
+import * as cors from 'cors'
+import * as bodyParser from 'body-parser'
+import { expressPlayground } from 'graphql-playground-middleware'

async function run() {

  const app = express()

+ app.use('/graphql', cors(), bodyParser.json())

+ app.use('/playground', expressPlayground({ endpoint: '/graphql' }))

- app.listen(3000)
+ app.listen(3000, () => console.log('Server running. Open http://localhost:3000/playground to run queries.'))
}

run()
```
We have added an endpoint `/graphql` where our GraphQL server is going to live. We also created an endpoint `/playground` using the excellent [`graphql-playground-middleware`](https://github.com/graphcool/graphql-playground) package from [Graphcool](https://graph.cool), a GraphQL playground on steroids, so we can easily test our queries later on.

Now it's time to set up our Qewl middleware! We start by initializing our Qewl instance, and defining the schema for our GraphQL server. Qewl exposes an Express middleware that we're adding to our `/graphql` endpoint.
```diff
import * as express from 'express'
import * as cors from 'cors'
import * as bodyParser from 'body-parser'
import { expressPlayground } from 'graphql-playground-middleware'

async function run() {

  const app = express()

+ const qewl = new Qewl()
+   .schema(`
+     type HelloPayload {
+       message: String
+     }
+
+     type Query {
+       hello: HelloPayload
+     }
+   `)

- app.use('/graphql', cors(), bodyParser.json())
+ app.use('/graphql', cors(), bodyParser.json(), await qewl.middleware())

  app.use('/playground', expressPlayground({ endpoint: '/graphql' }))

  app.listen(3000, () => console.log('Server running. Open http://localhost:3000/playground to run queries.'))
}

run()
```
Congratulations, you have created your GraphQL server! You can now visit [http://localhost:3000/playground](http://localhost:3000/playground) and run your first query.

![first-attempt](https://user-images.githubusercontent.com/852069/32813123-dc222878-c9a8-11e7-9e70-dd078c64d5e9.png)

As you see, the query doesn't return any data yet. We have defined our GraphQL schema, but we haven't defined any **implementation** for the routes defined in our schema.
So let's get back to the code, and add our **resolver**. Our resolvers live in `qewl.router`.
```diff
import * as express from 'express'
import * as cors from 'cors'
import * as bodyParser from 'body-parser'
import { expressPlayground } from 'graphql-playground-middleware'

async function run() {

  const app = express()

  const qewl = new Qewl()
    .schema(`
      type HelloPayload {
        message: String
      }

      type Query {
        hello: HelloPayload
      }
    `)
+   .router
+     .resolve('Query.hello', async (event) => {
+       return { message: `Hello ${event.args.name}!` }
+     })

  app.use('/graphql', cors(), bodyParser.json(), await qewl.middleware())

  app.use('/playground', expressPlayground({ endpoint: '/graphql' }))

  app.listen(3000, () => console.log('Server running. Open http://localhost:3000/playground to run queries.'))
}

run()
```
We have defined a resolver for our GraphQL route `Query.hello`. Let's run that query again...

![second-attempt](https://user-images.githubusercontent.com/852069/32813445-816489ba-c9aa-11e7-8474-d3ab42b2fe14.png)

Now, that looks a lot better. With just a few lines of codes, you have created a GraphQL Server from scratch, defining the schema and resolver and running the server as Express middleware.

## Deep Dive
d
