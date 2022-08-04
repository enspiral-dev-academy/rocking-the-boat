# Prism Break - A Brief look at [Prisma](https://www.prisma.io/docs/)

## What is this?

I really enjoyed Gerard's posts from [`burying-the-lede`](https://github.com/enspiral-dev-academy/burying-the-lede), so I'm writing a spin-off series called `rocking-the-boat`. So far, it's a series where I pick a popular technology and hypothetically hot-swap it with a piece of tech that we currently use, make a shallow comparison, write some thoughts, and share it with y'all. Enjoy, episode 1 is on Prisma and Knex. 🥳

## TypeScript and ESM in Knex-land

We're moving towards a future that is increasingly ESModules (ESM) friendly in preparation for TypeScript (TS). That means moving from:
```js
const knex = require('knex')
```
to:
```js
import knex from 'knex'
```

But, that doesn't come without its tradeoffs. Referencing [Gerard's post](https://github.com/enspiral-dev-academy/burying-the-lede/blob/main/posts/09-getting-into-modules.md#bloody-knex) on this, Knex doesn't play as nicely as we'd like with ESM. We can have Knex play a little nicer with ESM by explicitly stating that as we make migrations and seed files with `npx knex -x mjs migrate:make <name>`

And if we as staff are running into problems just getting things set up in a boilerplate, students are going to be in a constant brawl with knex in their complex friday projects 💥. We want students grappling with their business logic, not environment and config issues.

<img width="300px" height="auto" src="https://i.imgflip.com/6ouail.jpg" alt="student trying to write their app and being held back by knex and esm problems" />

Also, the TS support in knex is... well, as good as the person writing the TS.

To write a typed query in Knex, the student would need to write

### **a. The interface**
```ts
interface Fruit {
  id: number;
  name: string;
  color: string;
  createdAt: number;
  updatedAt: number;
}
```
### **b. The generics**
```ts
function getAllFruit(db = connection) {
  return db<Fruit>('fruit').select('*')
}
// return type is inferred to be Fruit[]
```

My main gripe with this is that the interface for a Fruit needs to be manually kept up to date with the *actual* schema of a Fruit defined in the migrations, which means it is another place for possibly stressful/overwhelming errors for students. 

But overall, the TS story for knex feels manual, and it feels laborious... My current mantra on teaching TypeScript is

> Writing TypeScript is hard, using TypeScript is easy, the more TypeScript students have to write, the harder it will be to teach.

That's where Prisma comes in...

## Prisma - what is it?

Prisma describes itself as...

> ...a next-generation ORM. It consists of the following parts
> - **Prisma Client:** Auto-generated and type-safe query builder for Node.js & TypeScript
> - **Prisma Migrate:** Migration system
> - **Prisma Studio:** GUI to view and edit data in your database

I think **Prisma Schema** is also a very important bit to talk about and I'll cover that first

---

### [**Prisma Schema**](https://www.prisma.io/docs/concepts/components/prisma-schema)

Prisma Schema is how we define our data. In Knex, our schema is defined by many migration files that show the change of the schema over time. In Prisma, our schema is defined in a single file that acts as a *source of truth*.

It's usually a `prisma.schema` file, and has its own syntax for defining schema. There is also a [VSCode extension](https://marketplace.visualstudio.com/items?itemName=Prisma.prisma) to format, lint, and add code completion for Prisma schemas.

For a table, we define a [model](https://www.prisma.io/docs/concepts/components/prisma-schema/data-model), joins are defined by using the other model as the type for the field (@relation() defines the relationship using primary/foreign key pairs, this can be code completed automagically 🪄).

<details>
<summary>An example schema based on Dreamfest</summary>

```prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "sqlite"
  url      = "file:dev.db"
}

model Location {
  // primary key
  id          Int     @id @default(autoincrement())
  name        String
  description String
  // a location can relate to many events
  events      Event[]
}

model Event {
  id          Int      @id @default(autoincrement())
  name        String
  day         String
  time        String
  description String
  // a location can relate to many events
  //                              // foreign key      // primary key
  location    Location @relation(fields: [locationId], references: [id])
  // foreign key
  locationId  Int
}
```
</details>

---

### [**Prisma Migrate**](https://www.prisma.io/docs/concepts/components/prisma-migrate)

In knex, to make a new migration, i.e., a change to our database schema, we

1. make a new migration file `npx knex migrate:make`
2. populate the exports.up and exports.down imperatively with the schema changes we want to make
3. run the migration with `npx knex migrate:latest`
4. change our seeds to match and run our seeds, `npx knex seed:run`

To do the same in Prisma, we

1. make a change to the prisma.schema file
2. (optional) push the change to our current development db with `npx prisma db push`
3. save and migrate the change to schema with `npx prisma migrate dev` (this makes a new migration file in `prisma/migrations`) (this also runs seeds if the seed script is set-up)

### [**Prisma Studio**](https://www.prisma.io/docs/concepts/components/prisma-studio)

Prisma Studio solves a small quality-of-life problem that students have the first time the interact with them... opening them.

With Prisma Studio, we can start a local server with `npx prisma studio` which opens a browser window where we can interact with our database (and perform CRUD). (This is great for students working over liveshare; this server can be shared and both students can view the contents of the database)

### [**Prisma Client**](https://www.prisma.io/docs/concepts/components/prisma-client)

As Prisma puts it,
>Prisma Client is an auto-generated and type-safe query builder that's *tailored* to your data.

In knex, query building is done by writing SQL-like commands as chained JS functions. In Prisma, it's done by writing nested objects with some SQL-like wording used (like `select` for column selection, `where` for filtering, etc. etc.)

<details>
<summary>Here are some example queries from Dreamfest</summary>

Notes:
- this is just one way of using prisma via dependency injection, I called it `ctx` (context) only because that's what they call it in the docs when talking about dependency injection
- these are the relevant db queries for events
- knex examples all come from dreamfest-solution in challenges
- assume input object has correct fields/types

```js
// ... init knex stuff

function getEventsByDay(day, db = connection) {
  return db('events')
    .join('locations', 'events.location_id', 'locations.id')
    .where('day', day)
    .select(
      'events.id',
      'events.day',
      'events.time',
      'events.name as eventName',
      'events.description',
      'locations.name as locationName'
    )
}

function addNewEvent(event, db = connection) {
  return db('events').insert(event)
}

function deleteEvent(id, db = connection) {
  return db('events').where('id', id).delete()
}


function getEventById(id, db = connection) {
  return db('events')
    .where('id', id)
    .first()
    .select(
      `id`,
      `location_id as locationId`,
      'day',
      'time',
      'name',
      'description'
    )
}

function updateEvent(updatedEvent, db = connection) {
  return db('events').where('id', updatedEvent.id).update(updatedEvent)
}


// ... init prisma stuff
const prismaCtx = prisma

// relations/joins
function getEventsByDay(day, ctx = prismaCtx) {
  return ctx.event.findMany({
    where: {
      day,
    },
    include: {
      location: true,
    },
  })
}

// create
function addNewEvent(
  event,
  ctx = prismaCtx
) {
  return ctx.event.create({
    data: {
      ...event
    },
  })
}

// read
function getEventById(id, ctx = prismaCtx) {
  return await ctx.event.findUnique({
    where: {
      id,
    },
    // add if you want location info
    // include: {
    //   location: true,
    // },
  })
}

// update
function updateEvent(id, event, ctx = prismaCtx) {
  return ctx.event.update({
    where: {
      id,
    },
    data: {
      ... event
    }
  })
}

// delete
function deleteEvent(id, ctx = prismaCtx) {
  return await ctx.event.delete({
    where: {
      id,
    },
  })
}

```
</details>

#### Small comparisons at a glance

| **_thing_**     | **knex**                                                               | **prisma**                                   |
|-----------------|------------------------------------------------------------------------|----------------------------------------------|
| one record      | `.first()`                                                             | `.findUnique()`                              |
| some columns    | `.select('column1', 'column2 as bananas')`                             | `select: {  column1: true,  column2: true }` |
| joins/relations | `db('table1').join('table2', 'table1.primaryKey', 'table2.foreignKey'` | `include: {   table2: true }`                |

### [**Joins and Relations**](https://www.prisma.io/docs/concepts/components/prisma-schema/relations)

This is probably big enough of a feature/difference to justify it's own h3 right...?

But here's a brief summary of a one-to-many relation in Prisma! They also accomodate one-to-one and many-to-many in a similar fashion, if you'd like to read about them, [follow me for relations hip advice](https://www.prisma.io/docs/concepts/components/prisma-schema/relations)

Table joins are really easy in prisma once your schema is set up to have them.

For example, if we use the Dreamfest schema

```prisma
// ... setup stuff

model Location {
  id          Int     @id @default(autoincrement())
  name        String
  description String
  events      Event[]
}

model Event {
  id          Int      @id @default(autoincrement())
  name        String
  day         String
  time        String
  description String
  location    Location @relation(fields: [locationId], references: [id])
  locationId  Int
}
```

It's really easy to write a db function to get an event and include it's location data too:

```js
function getEventById(id) {
  return prisma.event.findUnique({
    where: {
      id,
    },
    include: {
      location: true,
    },
  })
}
```

This function returns the following data structure:

```js
{
  id: 1,
  name: 'TangleEvent',
  day: 'Friday',
  time: '1pm - 2pm',
  description: 'A very nice event',
  locationId: 2,
  location: {
    id: 2,
    name: 'TangleStage',
    description: 'A very nice stage'
  }
}
```

This structure has no naming collisions because the joined fields are attached as a nested object.

### **Prisma's TypeScript story**

Prisma is built for and around TypeScript support. When working in a TS environment, writing and using database functions is rapid just because of the certainty it gives me that I got things right.

- When you write a model in the Prisma schema and run a migration, types are also generated in the Prisma Client.

  These can be imported anywhere in the codebase like:

  ```js
  import type { Event, Location } from '@prisma/client'
  ```

- When you write queries with Prisma, everything is type-safe and Prisma will yell at you if a field doesn't exist etc.

  ```js
  prisma.location.create({
    data: {
      name: 'TangleStage',
      speakers: 'Bose Tangle II X Pros',
    }
  })
  // type error because speakers doesn't exist on model Location
  // and type error because description is a required field on model Location
  ```

## Wrap it up

Welcome to the end of my first post on random tech, this time it's been on Prisma. Here's what I think:

- The Query Builder in Prisma feels much like Knex, with a few niceties (even more if you include TS)
- Prisma Studio is pretty cool, can be run alongside a normal server to add more data on the fly and test stuffs, cool for Liveshare situations too (not sure what workflow is like on-campus, I'm a online goblin 👺)
- Migrations have a different workflow to Knex, I don't have an opinion on better or worse, just different. It's nice to have the Prisma schema as a single source of truth which works nicely at the scale of applications students work with
- Re: TS, being able to define both schema and types in a single file and use those across the whole application is really useful. A student-friendly workflow I can see evolving is:
  - write the schema definition
  - write a db function (type-safe because of Prisma Client)
  - write a route (type-safe because of db function)
  - write an apiClient function (import and use type definition from @prisma/client to line up with route) <- only place a student has to actually write TypeScript
  - write a component/action (type-safe because of apiClient)
- Relations are easy and pain free, students can write more complex applications without getting bogged down by imperative SQL joins (although this _could_ be a poisoned chalice ☠️)
- Works in an ESM environment without hiccups

Thanks for reading episode 1 of roh's scribbly memoirs 🥳
