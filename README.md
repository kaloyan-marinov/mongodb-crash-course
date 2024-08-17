# Introduction

The contents of this repository is based on
the following YouTube video tutorial
(Traversy Media >> MongoDB Crash Course 2022).

First of all, it is important to note that
there are some adjectives,
which are commonly used to describe MongoDB
but which are, strictly speaking, inaccurate. The following table sets the record straight:

| Commonly-used adjective | More accurate adjective | What it means |
|-------------------------|-------------------------|---------------|
| NoSQL | NotOnlySQL | MongoDB _can_ use SQL |
| Non-Relation | NotOnlySQL | you _can_ store relational data in MongoDB (just stored differently than in a relational database) |

MongoDB is actually a <u>document database</u>.

- data is stored as BSON under the hood,
  but it's represented as JSON

- BSON is an extension of JSON
  and adds support for "extended data types"

A schema in a:

| relational database | document database |
|---------------------|-------------------|
| [required to be] pre-defined | optional |
| strict | flexible |

In a document database:

- a single document can store all of the data for each record
  (instead of splitting the data like a relational database does)

- generally, data that gets accessed together gets stored together

- querying data, in most operations, is faster than relational databases

- the schema is flexible,
  i.e. you can store any type of data in any document



# Concrete steps for trying out MongoDB on your machine

In order to be able to follow the rest of this section,
you need to have Docker
(or another Open-Container-Initiative-compliant tool
for managing containers and images such as Podman)
installed on your machine.

[step 1]

create `.env` file within your local repository by taking the following steps:

   ```bash
   $ cp \
      .env.template \
      .env

   # Edit the content of the created file as per the comments/instructions therein.
   ```

[step 2]

create an empty database:

   ```bash
   docker network create \
      network-m-c-c

   docker run \
      --name container-m-c-c \
      --network network-m-c-c \
      --mount source=volume-m-c-c,destination=/data/db \
      --env MONGO_INITDB_ROOT_USERNAME=$(grep -oP '^MONGO_USERNAME=\K.*' .env) \
	   --env MONGO_INITDB_ROOT_PASSWORD=$(grep -oP '^MONGO_PASSWORD=\K.*' .env) \
      --env MONGO_INITDB_DATABASE=$(grep -oP '^MONGO_DATABASE=\K.*' .env) \
      --publish 27017:27017 \
      mongo:latest
   ```

   <u>TODO: (2024/08/17, 11:00)</u> as of the commit that adds this line, the preceding command creates "a simple user with the role `root⁠` in the `admin` authentication database⁠" (cf. https://hub.docker.com/_/mongo ); investigate (a) what "creation scripts in /docker-entrypoint-initdb.d/*.js" (cf. ) would need to be created and (b) how the preceding commands would need to be changed _in order for_ a non-`root` user to be created (cf. https://www.mongodb.com/docs/manual/core/security-users/#user-authentication-database )

[step 3]

connect to the containerized MongoDB, which was started in the preceding step,
via the `mongosh` command-line client/interface/utility

   ```bash
   # The following command:
   #  - starts a 2nd container (based on the same container-image),
   #    which - upon its exit - will be fully cleaned up by Docker
   #  - launches the command-line client
   #    from within the 2nd container
   #    in a way that establishes a connection to the MongoDB (server),
   #    which is running in the 1st container

   docker run \
      -it \
      --rm \
      --network network-m-c-c \
      mongo \
	      mongosh \
         --host container-m-c-c \
		   --username $(grep -oP '^MONGO_USERNAME=\K.*' .env) \
		   --password $(grep -oP '^MONGO_PASSWORD=\K.*' .env) \
		   --authenticationDatabase admin \
		   $(grep -oP '^MONGO_DATABASE=\K.*' .env)
   ```

[step 4]

if you performed the preceding step successfully,
you have a live instance of the MongoDB shell;
within that, you can perform Create/Read/Update/Delete (CRUD) operations;

- Commands for working with databases:

   ```
   db-4-m-c-c> // Show the current database.
   db-4-m-c-c> db
   db-4-m-c-c

   db-4-m-c-c> // Show all databases.
   db-4-m-c-c> show dbs
   admin   100.00 KiB
   config   12.00 KiB
   local    72.00 KiB
   db-4-m-c-c> // The preceding command does not list `db-4-m-c-c`, because there is nothing in it yet.

   db-4-m-c-c> use blog
   switched to db blog
   blog> show dbs
   admin   100.00 KiB
   config   12.00 KiB
   local    72.00 KiB

   blog> // Again: The preceding command does not list `blog`, because there is nothing in it yet.
   blog> use db-4-m-c-c
   switched to db db-4-m-c-c
   ```

- Commands for creating collections in a database
  and documents in a collection:

   ```
   db-4-m-c-c> // db.createCollection("posts")

   db-4-m-c-c> db.posts.insertOne({
   ...   title: 'Post 1',
   ...   body: 'Body of post.',
   ...   category: 'News',
   ...   likes: 1,
   ...   tags: ['news', 'events'],
   ...   date: Date()
   ... })
   {
   acknowledged: true,
   insertedId: ObjectId('66c0c2b547e4d45c14149f48')
   }

   db-4-m-c-c> db.posts.insertMany([
   {
      title: 'Post 2',
      body: 'Body of post.',
      category: 'Event',
      likes: 2,
      tags: ['news', 'events'],
      date: Date()
   },
   {
      title: 'Post 3',
      body: 'Body of post.',
      category: 'Tech',
      likes: 3,
      tags: ['news', 'events'],
      date: Date()
   },
   {
      title: 'Post 4',
      body: 'Body of post.',
      category: 'Event',
      likes: 4,
      tags: ['news', 'events'],
      date: Date()
   },
   {
      title: 'Post 5',
      body: 'Body of post.',
      category: 'News',
      likes: 5,
      tags: ['news', 'events'],
      date: Date()
   }
   ])
   {
   acknowledged: true,
   insertedIds: {
      '0': ObjectId('66c0c2cd47e4d45c14149f49'),
      '1': ObjectId('66c0c2cd47e4d45c14149f4a'),
      '2': ObjectId('66c0c2cd47e4d45c14149f4b'),
      '3': ObjectId('66c0c2cd47e4d45c14149f4c')
   }
   }
   ```

- Commands for reading documents from a collection:

   ```
   db-4-m-c-c> db.posts.find()
   [
   {
      _id: ObjectId('66c0c2b547e4d45c14149f48'),
      title: 'Post 1',
      body: 'Body of post.',
      category: 'News',
      likes: 1,
      tags: [ 'news', 'events' ],
      date: 'Sat Aug 17 2024 15:33:09 GMT+0000 (Coordinated Universal Time)'
   },
   {
      _id: ObjectId('66c0c2cd47e4d45c14149f49'),
      title: 'Post 2',
      body: 'Body of post.',
      category: 'Event',
      likes: 2,
      tags: [ 'news', 'events' ],
      date: 'Sat Aug 17 2024 15:33:33 GMT+0000 (Coordinated Universal Time)'
   },
   {
      _id: ObjectId('66c0c2cd47e4d45c14149f4a'),
      title: 'Post 3',
      body: 'Body of post.',
      category: 'Tech',
      likes: 3,
      tags: [ 'news', 'events' ],
      date: 'Sat Aug 17 2024 15:33:33 GMT+0000 (Coordinated Universal Time)'
   },
   {
      _id: ObjectId('66c0c2cd47e4d45c14149f4b'),
      title: 'Post 4',
      body: 'Body of post.',
      category: 'Event',
      likes: 4,
      tags: [ 'news', 'events' ],
      date: 'Sat Aug 17 2024 15:33:33 GMT+0000 (Coordinated Universal Time)'
   },
   {
      _id: ObjectId('66c0c2cd47e4d45c14149f4c'),
      title: 'Post 5',
      body: 'Body of post.',
      category: 'News',
      likes: 5,
      tags: [ 'news', 'events' ],
      date: 'Sat Aug 17 2024 15:33:33 GMT+0000 (Coordinated Universal Time)'
   }
   ]



   db-4-m-c-c> db.posts.find({ category: 'News' })
   [
   {
      _id: ObjectId('66c0c2b547e4d45c14149f48'),
      title: 'Post 1',
      body: 'Body of post.',
      category: 'News',
      likes: 1,
      tags: [ 'news', 'events' ],
      date: 'Sat Aug 17 2024 15:33:09 GMT+0000 (Coordinated Universal Time)'
   },
   {
      _id: ObjectId('66c0c2cd47e4d45c14149f4c'),
      title: 'Post 5',
      body: 'Body of post.',
      category: 'News',
      likes: 5,
      tags: [ 'news', 'events' ],
      date: 'Sat Aug 17 2024 15:33:33 GMT+0000 (Coordinated Universal Time)'
   }
   ]



   db-4-m-c-c> db.posts.find({ category: 'news' }).count()
   0
   db-4-m-c-c> db.posts.find({ category: 'News' }).count()
   2



   db-4-m-c-c> // db.posts.find().sort({ title: 1 })

   db-4-m-c-c> // db.posts.find().sort({ title: -1 })



   db-4-m-c-c> db.posts.find().limit(2)
   [
   {
      _id: ObjectId('66c0c2b547e4d45c14149f48'),
      title: 'Post 1',
      body: 'Body of post.',
      category: 'News',
      likes: 1,
      tags: [ 'news', 'events' ],
      date: 'Sat Aug 17 2024 15:33:09 GMT+0000 (Coordinated Universal Time)'
   },
   {
      _id: ObjectId('66c0c2cd47e4d45c14149f49'),
      title: 'Post 2',
      body: 'Body of post.',
      category: 'Event',
      likes: 2,
      tags: [ 'news', 'events' ],
      date: 'Sat Aug 17 2024 15:33:33 GMT+0000 (Coordinated Universal Time)'
   }
   ]



   db-4-m-c-c> db.posts.findOne()
   {
   _id: ObjectId('66c0c2b547e4d45c14149f48'),
   title: 'Post 1',
   body: 'Body of post.',
   category: 'News',
   likes: 1,
   tags: [ 'news', 'events' ],
   date: 'Sat Aug 17 2024 15:33:09 GMT+0000 (Coordinated Universal Time)'
   }

   db-4-m-c-c> db.posts.findOne({
   ...    likes: { $gt: 3 }
   ... })
   {
   _id: ObjectId('66c0c2cd47e4d45c14149f4b'),
   title: 'Post 4',
   body: 'Body of post.',
   category: 'Event',
   likes: 4,
   tags: [ 'news', 'events' ],
   date: 'Sat Aug 17 2024 15:33:33 GMT+0000 (Coordinated Universal Time)'
   }
   ```

- Commands for updating documents in a collection:

   ```
   db-4-m-c-c> db.posts.find({ title: "Post 1" })
   [
   {
      _id: ObjectId('66c0c2b547e4d45c14149f48'),
      title: 'Post 1',
      body: 'Body of post.',
      category: 'News',
      likes: 1,
      tags: [ 'news', 'events' ],
      date: 'Sat Aug 17 2024 15:33:09 GMT+0000 (Coordinated Universal Time)'
   }
   ]

   db-4-m-c-c> db.posts.updateOne({ title: 'Post 1' },
   ... {
   ...   $set: {
   ...     category: 'Tech'
   ...   }
   ... })
   {
   acknowledged: true,
   insertedId: null,
   matchedCount: 1,
   modifiedCount: 1,
   upsertedCount: 0
   }

   db-4-m-c-c> db.posts.find({ title: "Post 1" })
   [
   {
      _id: ObjectId('66c0c2b547e4d45c14149f48'),
      title: 'Post 1',
      body: 'Body of post.',
      category: 'Tech',
      likes: 1,
      tags: [ 'news', 'events' ],
      date: 'Sat Aug 17 2024 15:33:09 GMT+0000 (Coordinated Universal Time)'
   }
   ]



   db-4-m-c-c> db.posts.updateOne({ title: 'Post 6' },
   ... {
   ...   $set: {
   ...     title: 'Post 6',
   ...     body: 'Body of post.',
   ...     category: 'News'
   ...   }
   ... })
   {
   acknowledged: true,
   insertedId: null,
   matchedCount: 0,
   modifiedCount: 0,
   upsertedCount: 0
   }

   db-4-m-c-c> db.posts.find({ title: "Post 6" })

   db-4-m-c-c> db.posts.updateOne({ title: 'Post 6' },
   ... {
   ...   $set: {
   ...     title: 'Post 6',
   ...     body: 'Body of post.',
   ...     category: 'News'
   ...   }
   ... },
   ... {
   ...   upsert: true
   ... })
   {
   acknowledged: true,
   insertedId: ObjectId('66c0c7ff4d28bb07910962f6'),
   matchedCount: 0,
   modifiedCount: 0,
   upsertedCount: 1
   }

   db-4-m-c-c> db.posts.find({ title: "Post 6" })
   [
   {
      _id: ObjectId('66c0c7ff4d28bb07910962f6'),
      title: 'Post 6',
      body: 'Body of post.',
      category: 'News'
   }
   ]



   db-4-m-c-c> db.posts.find({ title: "Post 1" })
   [
   {
      _id: ObjectId('66c0c2b547e4d45c14149f48'),
      title: 'Post 1',
      body: 'Body of post.',
      category: 'Tech',
      likes: 1,
      tags: [ 'news', 'events' ],
      date: 'Sat Aug 17 2024 15:33:09 GMT+0000 (Coordinated Universal Time)'
   }
   ]
   db-4-m-c-c> db.posts.updateOne({ title: "Post 1" }, {
   ...   $inc: {
   ...     likes: 2
   ...   }
   ... })
   {
   acknowledged: true,
   insertedId: null,
   matchedCount: 1,
   modifiedCount: 1,
   upsertedCount: 0
   }
   db-4-m-c-c> db.posts.find({ title: "Post 1" })
   [
   {
      _id: ObjectId('66c0c2b547e4d45c14149f48'),
      title: 'Post 1',
      body: 'Body of post.',
      category: 'Tech',
      likes: 3,
      tags: [ 'news', 'events' ],
      date: 'Sat Aug 17 2024 15:33:09 GMT+0000 (Coordinated Universal Time)'
   }
   ]



   db-4-m-c-c> db.posts.updateMany({}, {
   ...   $inc: { likes: 1 }
   ... })
   {
   acknowledged: true,
   insertedId: null,
   matchedCount: 6,
   modifiedCount: 6,
   upsertedCount: 0
   }

   db-4-m-c-c> // For example, "Post 6" was created without any likes, but now:
   db-4-m-c-c> db.posts.findOne({ title: "Post 6" })
   {
   _id: ObjectId('66c0c7ff4d28bb07910962f6'),
   title: 'Post 6',
   body: 'Body of post.',
   category: 'News',
   likes: 1
   }
   ```

- Commands for deleting documents in a collection:

   ```
   db-4-m-c-c> db.posts.find().count()
   6

   db-4-m-c-c> db.posts.deleteOne({ title: 'Post 6' })
   { acknowledged: true, deletedCount: 1 }

   db-4-m-c-c> db.posts.find().count()
   5



   db-4-m-c-c> db.posts.find({ category: 'Tech' }).count()
   2

   db-4-m-c-c> db.posts.deleteMany({ category: 'Tech' })
   { acknowledged: true, deletedCount: 2 }

   db-4-m-c-c> db.posts.find().count()
   3

   db-4-m-c-c> db.posts.find({ category: 'Tech' }).count()
   0
   ```

[step 5]

to get Docker to clean up the created network, volume, and container,
you can issue

   ```bash
   ./clean-docker-artifacts.sh
   ```
