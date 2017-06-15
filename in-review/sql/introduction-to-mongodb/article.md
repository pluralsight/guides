Recently I had to start using [MongoDB](https://www.mongodb.com/) for an enterprise project and learned a lot on the way. After revving up to speed, I hope to share my experiences with you. In this article, I will introduce what MongoDB is and how to get started with using it. By the end of this tutorial, you will understand everything up to the basic CRUD (Create, Retrieve, Update, Delete) operations.  

## Introduction

MongoDB is a **NoSQL database framework.** NoSQL Databases are totally different from traditional Relational Databases (RDBs) like MySQL or postgreSQL. RDBs have specific pre-defined schema, fields, constraints, type of fields, triggers, and so on. 

In the case of a typical NoSQL Database, there's nothing like the above. There's no need to define a structure before building the database. This allows a MongoDB database to scale up or down depending on the application, whereas the traditional RDBs do not scale easily. NoSQL is generally much faster in most cases, so if you need to store or retrieve large quantities of data, NoSQL is *the* way to go.

There are [different types](https://www.mongodb.com/scale/types-of-nosql-databases) of NoSQL databases, such as key-value stores, document databases, wide-column stores, and graph databases. MongoDB, a Document Database, stores all schema and records in documents using a syntax like JSON (JavaScript Object Notation). If you are familiar with Web Development, MongoDB will seem comfortable.

## Installation

Please refer to the [official MongoDB guides](https://docs.mongodb.com/manual/installation/) to install the database essentials. 

Once you have installed MongoDB, add the `bin` directory to the path. You need to be aware of two binary executable files.

* `mongod` - This is the daemon (a program that always runs in the background as a service) for MongoDB Server.
* `mongo` - This is the command line client shell interface for MongoDB.

> **Note:** The MongoDB server usually runs in the port `27017`.

## Shell

The shell is started by executing the `mongo` command from any of your operating system command line terminal interfaces:

```language-bash
C:\Users\Praveen> mongo⏎                  #     Windows
Praveen-MBP:~ Praveen$ mongo⏎             #   Macintosh
praveen@ubuntu:~$ mongo⏎                  #      Ubuntu
```

Once you get into this part, you should see a black screen with the following:

```language-bash
praveen@ubuntu:~$ mongo
MongoDB shell version: 3.0.7
connecting to: test
Server has startup warnings:
[ some crazy error info messages ]
[ you don't need to worry about  ]
>
```

There will be a few crazy warnings, which you need not worry about. If we press <kbd>Ctrl</kbd> + <kbd>L</kbd> or type `cls` in the shell and hit <kbd>Enter</kbd>, all the messages will be cleared. And you will be left with the **MongoDB Shell**:

```language-bash
>
```

### Commands

#### Show all the Databases

To list all the databases available in the current server, we need to use the command `show dbs`. It shows the one default `local` database, and let's leave it aside without touching it.

```language-bash
> show dbs
local      0.000GB
>
```

#### Create a Database

To create and use a new database we need the `use` command. Let's create a new database `praveen`:

```language-bash
> use praveen
switched to db praveen
>
```

The good part is, when we use the `use` command, it creates a new database, if it doesn't exist and switches to the same too.

To check the current database we are in, we have a handy command called `db` to the rescue. It will give us the current database we are in.

```language-bash
> db
praveen
>
```

## Documents

The syntax of a document looks like JSON (JavaScript Object Notation). For example:

```language-javascript
{
  "field1": "value1",
  "field2": "value2",
  // --- and so on ---
  "fieldN": "valueN"
}
```

> **Note:** A valid JSON will not have a trailing comma. Look at the last value? `ValueN` doesn't end with a comma.

Let's consider a student record. A typical student record might contain basic details like name, email, and degree:

```language-javascript
{
  "name": "Praveen Kumar",
  "degree": "Cloud Computing",
  "email": "praveen@example.com"
}
```

The above set of data are just simple string values. Arrays and objects can also be values in our database. For example, our database might have a field for `subjects`, which keeps track of all classes in array format. In this example, each class or course would be an object that represents a subject's details. We could also keep students' phone numbers in array format as well. Each of these usages is shown below:

```language-javascript
{
  "name": "Praveen Kumar",
  "degree": "Cloud Computing",
  "email": "praveen@example.com",
  "subjects": [
    {
      "name": "Internet Networks",
      "prof": "Prof. Awesome Blossom"
    },
    {
      "name": "Cloud Computing",
      "prof": "Prof. Tech Ninja"
    },
    {
      "name": "Web Development",
      "prof": "Prof. Chunky Monkey"
    }
  ],
  "phone": ["9840035007", "9967728336", "7772844242"]
}
```

## Database Management

### User Management

To start using our freshly-brewed MongoDB Database, we need to create some users. The function for creating users is [`db.createUser()`](https://docs.mongodb.com/manual/reference/method/db.createUser/). There are different ways for this, but let's focus on the simple one:

```language-javascript
db.createUser(
  {
    user: "praveen",
    pwd: "praveen",
    roles: [ "readWrite", "dbAdmin" ]
  }
)
```

> **Note:** The `db` variable here denotes the current active database.

Doing this operation on the shell will give you a success output similar to the following:

```language-bash
> db.createUser(
...   {
...     user: "praveen",
...     pwd: "praveen",
...     roles: [ "readWrite", "dbAdmin" ]
...   }
... )
Successfully added user: { "user" : "praveen", "roles" : [ "readWrite", "dbAdmin" ] }
>
```

Now that we have a user now, let's proceed to add some data!

### Content Management

In traditional databases, we generally use schema (or tables), but there's no such hard and fast rule for NoSQL Databases. We have Collections instead of Tables here. Basically, they hold the documents or records.

#### Creating Collections

To create a collection, use the [`db.createCollection()`](https://docs.mongodb.com/manual/reference/method/db.createCollection/) method. It takes one parameter, the name of the collection. To create a collection for students, we give:

```language-bash
> db.createCollection("students");
{ "ok" : 1 }
>
```

The success message will be `ok` with the count of affected items (or created collections, in this case).

#### Listing Collections

To list out all the collections in this particular database, there's a command for it too. We can use `show collections`. The output will be similar to:

```language-bash
> show collections
students
>
```

#### Inserting into Collections

Inserting into collections is similar to an array's `push` function. We'll use the [`db.collection.insert()`](https://docs.mongodb.com/manual/reference/method/db.collection.insert/) function. In our case, the collection is `students`. So we use:

```language-javascript
db.students.insert({
  "name": "Praveen Kumar",
  "degree": "Cloud Computing",
  "email": "praveen@example.com",
  "subjects": [
    {
      "name": "Internet Networks",
      "prof": "Prof. Awesome Blossom"
    },
    {
      "name": "Cloud Computing",
      "prof": "Prof. Tech Ninja"
    },
    {
      "name": "Web Development",
      "prof": "Prof. Chunky Monkey"
    }
  ],
  "phone": ["9840035007", "9967728336", "7772844242"]
});
```

The success message will be similar to what you see here:

```language-bash
> db.students.insert({
...   "name": "Praveen Kumar",
...   "degree": "Cloud Computing",
...   "email": "praveen@example.com",
...   "subjects": [
...     {
...       "name": "Internet Networks",
...       "prof": "Prof. Awesome Blossom"
...     },
...     {
...       "name": "Cloud Computing",
...       "prof": "Prof. Tech Ninja"
...     },
...     {
...       "name": "Web Development",
...       "prof": "Prof. Chunky Monkey"
...     }
...   ],
...   "phone": ["9840035007", "9967728336", "7772844242"]
... });
WriteResult({ "nInserted" : 1 })
>
```

We can see in the result, it shows how many inserts have been done by showing `nInserted` value as `1`.

#### Listing Collection Items or Documents (Records)

To see if this has been really inserted, we can use the `find()` function to see if it has been inserted correctly. Use [`db.collection.find()`](https://docs.mongodb.com/manual/reference/method/db.collection.find/) as follows:

```language-bash
> db.students.find();
{ "_id" : ObjectId("592ebe7e8e61243307417cc4"), "name" : "Praveen Kumar", "degree" : "Cloud Computing", "email" : "praveen@example.com", "subjects" : [ { "name" : "Internet Networks", "prof" : "Prof. Awesome Blossom" }, { "name" : "Cloud Computing", "prof" : "Prof. Tech Ninja" }, { "name" : "Web Development", "prof" : "Prof. Chunky Monkey" } ], "phone" : [ "9840035007", "9967728336", "7772844242" ] }
>
```

Clearly our find operation has very complicated output. To the newbie reader, it makes no sense. 

Let's try to format it using the [`pretty()`](https://docs.mongodb.com/manual/reference/method/cursor.pretty/) function, which goes like:

```language-javascript
> db.students.find().pretty();
{
    "_id" : ObjectId("592ebe7e8e61243307417cc4"),
    "name" : "Praveen Kumar",
    "degree" : "Cloud Computing",
    "email" : "praveen@example.com",
    "subjects" : [
        {
            "name" : "Internet Networks",
            "prof" : "Prof. Awesome Blossom"
        },
        {
            "name" : "Cloud Computing",
            "prof" : "Prof. Tech Ninja"
        },
        {
            "name" : "Web Development",
            "prof" : "Prof. Chunky Monkey"
        }
    ],
    "phone" : [
        "9840035007",
        "9967728336",
        "7772844242"
    ]
}
>
```

Ah, now this is what we inserted. It looks better than previous machine output. All that we need to do is add `.pretty()` at the end of any result to make it look pretty.

> **Wait a minute!** Something's new!

We see that there's a new field called `_id` in the list of fields. We did not add it; MongoDB has a way of identifying records using `_id` and setting it to an Object with some crazily random value. This is used as a unique value to uniquely identify a single record. Please note that this field is automatically created by MongoDB, and we don't need to worry about creating one separately and making it auto increment and function as the primary key. These are all requirements in a RDB that are made obsolete in a NoSQL database.

#### Bulk Insertion

Yep, you heard it right. Like MySQL, you can also insert many records at a single point of time. The same `insert()` function takes not only objects, but also arrays as input. This time, let me be lazy and create something simple for the next two records.

```language-javascript
db.students.insert(
  [
    {
      "name": "Purushothaman",
      "degree": "Management",
      "email": "purush@example.com"
    },
    {
      "name": "Meaow Meaow",
      "degree": "Cat Study",
      "email": "meaow@example.com",
      "phone": ["9850420420"]
    },
  ]
);
```

You can find that the above records are both different. This leads us to the biggest advantage of Document Databases. You **don't** need a fixed schema for the records and each record can be any valid JavaScript expression.

> Changing schema "on the fly" is the biggest advantage of using NoSQL.

Now let's try executing the above insert query on our database.

```language-javascript
> db.students.insert(
...   [
...     {
...       "name": "Purushothaman",
...       "degree": "Management",
...       "email": "purush@example.com"
...     },
...     {
...       "name": "Meaow Meaow",
...       "degree": "Cat Study",
...       "email": "meaow@example.com",
...       "phone": ["9850420420"]
...     },
...   ]
... );
BulkWriteResult({
    "writeErrors" : [ ],
    "writeConcernErrors" : [ ],
    "nInserted" : 2,
    "nUpserted" : 0,
    "nMatched" : 0,
    "nModified" : 0,
    "nRemoved" : 0,
    "upserted" : [ ]
})
>
```

You get the output of the `BulkWriteResult` function, where it says nothing for errors and for those inserted records, we have got the count as `2`. Let's quickly check our table contents by using `find()` method.

```
> db.students.find();
{ "_id" : ObjectId("592ebe7e8e61243307417cc4"), "name" : "Praveen Kumar", "degree" : "Cloud Computing", "email" : "praveen@example.com", "subjects" : [ { "name" : "Internet Networks", "prof" : "Prof. Awesome Blossom" }, { "name" : "Cloud Computing", "prof" : "Prof. Tech Ninja" }, { "name" : "Web Development", "prof" : "Prof. Chunky Monkey" } ], "phone" : [ "9840035007", "9967728336", "7772844242" ] }
{ "_id" : ObjectId("592ed5818e61243307417cc5"), "name" : "Purushothaman", "degree" : "Management", "email" : "purush@example.com" }
{ "_id" : ObjectId("592ed5818e61243307417cc6"), "name" : "Meaow Meaow", "degree" : "Cat Study", "email" : "meaow@example.com", "phone" : [ "9850420420" ] }
>
```

Oh yes, we got the three records, but each has different schema, which is great! Since there are too many fields, let's make it `pretty()`:

```language-javascript
> db.students.find().pretty();
{
    "_id" : ObjectId("592ebe7e8e61243307417cc4"),
    "name" : "Praveen Kumar",
    "degree" : "Cloud Computing",
    "email" : "praveen@example.com",
    "subjects" : [
        {
            "name" : "Internet Networks",
            "prof" : "Prof. Awesome Blossom"
        },
        {
            "name" : "Cloud Computing",
            "prof" : "Prof. Tech Ninja"
        },
        {
            "name" : "Web Development",
            "prof" : "Prof. Chunky Monkey"
        }
    ],
    "phone" : [
        "9840035007",
        "9967728336",
        "7772844242"
    ]
}
{
    "_id" : ObjectId("592ed5818e61243307417cc5"),
    "name" : "Purushothaman",
    "degree" : "Management",
    "email" : "purush@example.com"
}
{
    "_id" : ObjectId("592ed5818e61243307417cc6"),
    "name" : "Meaow Meaow",
    "degree" : "Cat Study",
    "email" : "meaow@example.com",
    "phone" : [
        "9850420420"
    ]
}
>
```

Ah! There we go!

#### Querying Records

For querying or filtering the fields, we need to pass them as parameter objects in the `find()` function. One example will be, let's see if I can get the record with `Meaow Meaow`:

```language-javascript
> db.students.find({"name" : "Meaow Meaow"});
{ "_id" : ObjectId("592ed5818e61243307417cc6"), "name" : "Meaow Meaow", "degree" : "Cat Study", "email" : "meaow@example.com", "phone" : [ "9850420420" ] }
>
```

And as usual our `pretty()` would return:

```language-javascript
> db.students.find({"name" : "Meaow Meaow"}).pretty();
{
    "_id" : ObjectId("592ed5818e61243307417cc6"),
    "name" : "Meaow Meaow",
    "degree" : "Cat Study",
    "email" : "meaow@example.com",
    "phone" : [
        "9850420420"
    ]
}
>
```
This also works for items inside arrays. The `find()` method does exact matches of the values here. Now check one more example for matching a number within an array:

```language-javascript
> db.students.find({"phone": "9840035007"}).pretty();
{
    "_id" : ObjectId("592ebe7e8e61243307417cc4"),
    "name" : "Praveen Kumar",
    "degree" : "Cloud Computing",
    "email" : "praveen@example.com",
    "subjects" : [
        {
            "name" : "Internet Networks",
            "prof" : "Prof. Awesome Blossom"
        },
        {
            "name" : "Cloud Computing",
            "prof" : "Prof. Tech Ninja"
        },
        {
            "name" : "Web Development",
            "prof" : "Prof. Chunky Monkey"
        }
    ],
    "phone" : [
        "9840035007",
        "9967728336",
        "7772844242"
    ]
}
>
```

The above command correctly identifies my record and displays it, even though `9840035007` is only one of three numbers present.

#### Updating Records

The method we use here is [`db.collection.update()`](https://docs.mongodb.com/manual/reference/method/db.collection.update/). It takes up two parameters and the first parameter is an object of a key value pair for a match that is present in the records. The next parameter is the content that it's supposed to be replaced with.

So let's try something like this:

```language-javascript
db.students.update({"name": "Praveen Kumar"}, {
  "name": "Praveen Kumar",
  "degree": "Cloud Computing MSc",
  "email": "praveen@example.net",
  "subjects": [
    {
      "name": "Internet Networks",
      "prof": "Prof. Awesome Blossom"
    },
    {
      "name": "Cloud Computing",
      "prof": "Prof. Tech Ninja"
    },
    {
      "name": "Web Development",
      "prof": "Prof. Chunky Monkey"
    }
  ],
  "phone": ["9840035007", "9967728336", "7772844242"]
});
```

> **Note 1:** You need to give the full object in case of update function, because it makes a replacement of the whole record with the second parameter. If we didn't set the other values, it will just have one record with just two of the items: the email and the degree.

> **Note 2:** Do not find by name, AS I just did. There may be many records, matching the same parameter so use something unique, like `_id`.

Trying the above, we get:

```language-javascript
> db.students.update({"name": "Praveen Kumar"}, {
...   "name": "Praveen Kumar",
...   "degree": "Cloud Computing MSc",
...   "email": "praveen@example.net",
...   "subjects": [
...     {
...       "name": "Internet Networks",
...       "prof": "Prof. Awesome Blossom"
...     },
...     {
...       "name": "Cloud Computing",
...       "prof": "Prof. Tech Ninja"
...     },
...     {
...       "name": "Web Development",
...       "prof": "Prof. Chunky Monkey"
...     }
...   ],
...   "phone": ["9840035007", "9967728336", "7772844242"]
... });
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
>
```

Voila! Let's see the results:

```language-javascript
{
  "nMatched": 1,
  "nUpserted": 0,
  "nModified": 1
}
```

It matched one record and it modified it. There's none upserted (we'll cover this later). This means that, there's a possibility that it might match, but not update. Let's run the same command again and see what happens.

> Oh dear! It shows the same output. Maybe I was over-enthusiastic.

To see what has changed, we could try to run the `find()` function along with our pretty `pretty()` function.

```language-javascript
> db.students.find({"phone": "9840035007"}).pretty();
{
    "_id" : ObjectId("592ebe7e8e61243307417cc4"),
    "name" : "Praveen Kumar",
    "degree" : "Cloud Computing MSc",
    "email" : "praveen@example.net",
    "subjects" : [
        {
            "name" : "Internet Networks",
            "prof" : "Prof. Awesome Blossom"
        },
        {
            "name" : "Cloud Computing",
            "prof" : "Prof. Tech Ninja"
        },
        {
            "name" : "Web Development",
            "prof" : "Prof. Chunky Monkey"
        }
    ],
    "phone" : [
        "9840035007",
        "9967728336",
        "7772844242"
    ]
}
>
```
##### Using `$set`

However, it's a pain to add the whole record again, just to change one single value. The good news is, there's a way around that and that's using the [`$set`](https://docs.mongodb.com/manual/reference/operator/update/set/) operator. So let's take another example. Say we need to change my `email` to `praveen@example.net`. What we need to do is:

```language-javascript
db.students.update({
  "name": "Praveen Kumar"
}, {
  $set: {
    "email" : "praveen@example.net"
  }
});
```

Short and sweet! Now let's see the output of the above command.

```language-javascript
> db.students.update({
...   "name": "Praveen Kumar"
... }, {
...   $set: {
...     "email" : "praveen@example.net"
...   }
... });
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
>
```

Now, this `$set` is somewhat similar to our SQL's `UPDATE` query. When the query is fired, it keeps the already existing value intact and updates the respective fields. Similar to `UPDATE`, `$set` shows how many rows the query has affected.

If we see the above result, we can find that the command has matched one and modified one, but not upserted any. If we try to run the same command again, it will give the following result:

```language-javascript
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 0 })
```

Sweet! Now we know where this `matched` and `modified` have different counts. This means, even if you keep sending the same command for update without the `$set` operator, the modification keeps happening all the time, while the `$set` operator makes it happen only if there are different values.

> **Note:** `$set` is more performance efficient, if you are making a lot of updates.

#### Incrementing Numeric Values

There's another operator that helps us increment numeric values. Consider a record that has some numeric parameter, such as a field called `points`. `points` are great because they need to increment frequently. Being lazy, I am going to use the `$set` function:

```language-javascript
db.students.update({
  "name": "Praveen Kumar"
}, {
  $set: {
    "points" : 15
  }
});
```

Executing the above command, we get this:

```language-javascript
> db.students.update({
...   "name": "Praveen Kumar"
... }, {
...   $set: {
...     "points" : 15
...   }
... });
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
>
```

And checking if it has been updated, we fire out the `find()` command, we get:

```language-javascript
> db.students.find({"phone": "9840035007"}).pretty();
{
    "_id" : ObjectId("592ebe7e8e61243307417cc4"),
    "name" : "Praveen Kumar",
    "degree" : "Cloud Computing MSc",
    "email" : "praveen@example.com",
    "subjects" : [
        {
            "name" : "Internet Networks",
            "prof" : "Prof. Awesome Blossom"
        },
        {
            "name" : "Cloud Computing",
            "prof" : "Prof. Tech Ninja"
        },
        {
            "name" : "Web Development",
            "prof" : "Prof. Chunky Monkey"
        }
    ],
    "phone" : [
        "9840035007",
        "9967728336",
        "7772844242"
    ],
    "points" : 15
}
>
```

Excellent. We have a `points` field in my record with a value of `15`. 

That entire process should be clear by now. If something is unclear, look back at the previous section of the guide and/or fire off a question in the discussion section below.

Moving on, let's consider the case that I complete this article, thereby earning 5 more points. This condition can be produced with one small change:

```language-javascript
db.students.update({
  "name": "Praveen Kumar"
}, {
  $inc: {
    "points" : 5
  }
});
```

As you can see, changing `$set` to `$inc`, makes the difference. This is similar to using the augmented assignment operator `+=` in standard langauges rather than the assignment operator `=`.

```language-javascript
> db.students.update({
...   "name": "Praveen Kumar"
... }, {
...   $inc: {
...     "points" : 5
...   }
... });
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
> db.students.find({"phone": "9840035007"}).pretty();
{
    "_id" : ObjectId("592ebe7e8e61243307417cc4"),
    "name" : "Praveen Kumar",
    "degree" : "Cloud Computing MSc",
    "email" : "praveen@example.com",
    "subjects" : [
        {
            "name" : "Internet Networks",
            "prof" : "Prof. Awesome Blossom"
        },
        {
            "name" : "Cloud Computing",
            "prof" : "Prof. Tech Ninja"
        },
        {
            "name" : "Web Development",
            "prof" : "Prof. Chunky Monkey"
        }
    ],
    "phone" : [
        "9840035007",
        "9967728336",
        "7772844242"
    ],
    "points" : 20
}
>
```

Wow, that was an awesome increment for me from 15 to 20. Clearly, using `$inc` is a great way to increment numeric values.

#### Removing Fields

You can remove all or few fields from a single record. That's how NoSQL works. For this, we will be using a function called [`$unset`](https://docs.mongodb.com/manual/reference/operator/update/unset/), which is opposite to the `$set`, which we saw before.

The syntax for this is quite tricky. You need to pass the whole object as the second parameter. For example, let's say I don't want that `points` field now. Instead I want to remove the `points` category from my record. With NoSQL, I just need to execute the following command:

```language-javascript
db.students.update({
  "name": "Praveen Kumar"
}, {
  $unset: {
    "points" : ""
  }
});
```

Here, you can see that it follows the similar structure to `$inc` and `$set`. Only thing is that you can give any value for the field. Giving `""` or `75` works. Upon executing this, we have the following output:

```language-javascript
> db.students.update({
...   "name": "Praveen Kumar"
... }, {
...   $unset: {
...     "points" : ""
...   }
... });
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
> db.students.find({"phone": "9840035007"}).pretty();
{
    "_id" : ObjectId("592ebe7e8e61243307417cc4"),
    "name" : "Praveen Kumar",
    "degree" : "Cloud Computing MSc",
    "email" : "praveen@example.com",
    "subjects" : [
        {
            "name" : "Internet Networks",
            "prof" : "Prof. Awesome Blossom"
        },
        {
            "name" : "Cloud Computing",
            "prof" : "Prof. Tech Ninja"
        },
        {
            "name" : "Web Development",
            "prof" : "Prof. Chunky Monkey"
        }
    ],
    "phone" : [
        "9840035007",
        "9967728336",
        "7772844242"
    ]
}
>
```

Yippee! We have removed `points` from my record.

#### Upsertion

**Upsert** is a new term here and it's a combination of `update` and `insert`. Upserting is similar to saying, "if a record is present, update it, otherwise, insert it." An upsertion occurs when you try to find a match in the database, can't find it, and insert a vlaue instead of simply returning. Let's consider the same example, where we have three records and we'll try to update a non-existent record, say the student "Baahubali".

```language-javascript
db.students.update({
  "name": "Baahubali"
}, {
  "name": "Baahubali",
  "degree" : "Hill Climbing",
  "email" : "baahu@mahishmati.com"
});
```

If we go ahead and run the above command, we get:

```language-javascript
> db.students.update({
...   "name": "Baahubali"
... }, {
...   "name": "Baahubali",
...   "degree" : "Hill Climbing",
...   "email" : "baahu@mahishmati.com"
... });
WriteResult({ "nMatched" : 0, "nUpserted" : 0, "nModified" : 0 })
>
```

The result is 0 all across. Doing a simple `find()` will give the same results as they were before. If we want the database to check if the match is not found, then insert it, we need to use the option called **upsert**. Now let's use the same query and make a small change in it. Let's add a third parameter for the `update()` function, specifying an object with only `upsert` set to `true`.

```language-javascript
db.students.update({
  "name": "Baahubali"
}, {
  "name": "Baahubali",
  "degree" : "Hill Climbing",
  "email" : "baahu@mahishmati.com"
}, {
  "upsert": true
});
```

Executing the above command, we get the following:

```language-javascript
> db.students.update({
...   "name": "Baahubali"
... }, {
...   "name": "Baahubali",
...   "degree" : "Hill Climbing",
...   "email" : "baahu@mahishmati.com"
... }, {
...   "upsert": true
... });
WriteResult({
    "nMatched" : 0,
    "nUpserted" : 1,
    "nModified" : 0,
    "_id" : ObjectId("592ffa57962222f55cf0a823")
})
>
```

By setting the `upsert` parameter to `true`, we can see there's one record inserted (actually upserted) with the `_id` returned to us. This is similar to the MySQL Insert query on a table where there is `AUTO_INCREMENT` set. Now let's see what's there in the database by making a `find()`:

```language-javascript
> db.students.find().pretty();
{
    "_id" : ObjectId("592ebe7e8e61243307417cc4"),
    "name" : "Praveen Kumar",
    "degree" : "Cloud Computing MSc",
    "email" : "praveen@example.com",
    "subjects" : [
        {
            "name" : "Internet Networks",
            "prof" : "Prof. Awesome Blossom"
        },
        {
            "name" : "Cloud Computing",
            "prof" : "Prof. Tech Ninja"
        },
        {
            "name" : "Web Development",
            "prof" : "Prof. Chunky Monkey"
        }
    ],
    "phone" : [
        "9840035007",
        "9967728336",
        "7772844242"
    ]
}
{
    "_id" : ObjectId("592ed5818e61243307417cc5"),
    "name" : "Purushothaman",
    "degree" : "Management",
    "email" : "purush@example.com"
}
{
    "_id" : ObjectId("592ed5818e61243307417cc6"),
    "name" : "Meaow Meaow",
    "degree" : "Cat Study",
    "email" : "meaow@example.com",
    "phone" : [
        "9850420420"
    ]
}
{
    "_id" : ObjectId("592ffa57962222f55cf0a823"),
    "name" : "Baahubali",
    "degree" : "Hill Climbing",
    "email" : "baahu@mahishmati.com"
}
>
```

Yay! Now [Baahubali](https://en.wikipedia.org/wiki/Baahubali:_The_Beginning) is in our list of students. (Let's hunt for Kalakeya now! 😁) 

Clearly, upsert is a useful function that can save quite a bit of error handling and time down the line.

#### Renaming Fields

We have Baahubali as our student, but wait. In those Mahishmati times, they didn't have email, did they? Let's replace his `email` field with `pigeon`. We have a quick stuff for renaming things in MongoDB. As you guessed, the operator is [`$rename`](https://docs.mongodb.com/manual/reference/operator/update/rename/). This function is also similar to our `$set` function in its syntax, so to rename, it would be:

```language-javascript
db.students.update({
  "name": "Baahubali"
}, {
  $rename: {
    "email" : "pigeon"
  }
});
```

Now when we execute the above command and do a `find()` and see the results, we get:

```language-javascript
> db.students.update({
...   "name": "Baahubali"
... }, {
...   $rename: {
...     "email" : "pigeon"
...   }
... });
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
> db.students.find({"name": "Baahubali"}).pretty();
{
    "_id" : ObjectId("592ffa57962222f55cf0a823"),
    "name" : "Baahubali",
    "degree" : "Hill Climbing",
    "pigeon" : "baahu@mahishmati.com"
}
>
```

Ah! I love this pigeon stuff. We have got one record matched and modified. We have now successfully changed the `email` to `pigeon` for `Baahubali`.

> **Note to PETA:** No pigeons were hurt while writing this article.


#### Removing Records

We are in the final part of the CRUD (Create Retrieve Update Delete) operation. Make sure that you understand each of the other letters before moving onto deletion. 

Deleting records or removing documents is easier as well. We will be using the [`remove()`](https://docs.mongodb.com/manual/reference/method/db.collection.remove/) function for the removal operation. The function is similar to `update()`, but it doesn't take any other parameters except the `options` (optional) with `justOne` (we will talk about it sooner). Let's try adding a dumb record before trying to delete any of our students.

```language-javascript
> db.students.insert([{"name": "Bleh"}, {"name": "Bleh"}, {"name": "Blah"}]);
BulkWriteResult({
    "writeErrors" : [ ],
    "writeConcernErrors" : [ ],
    "nInserted" : 3,
    "nUpserted" : 0,
    "nMatched" : 0,
    "nModified" : 0,
    "nRemoved" : 0,
    "upserted" : [ ]
})
>
```

Yep, now we have these `Blah` and two `Bleh` students, who just have their names in them. Let's check the list of students quickly?

```language-javascript
> db.students.find();
{ "_id" : ObjectId("592ebe7e8e61243307417cc4"), "name" : "Praveen Kumar", "degree" : "Cloud Computing MSc", "..." ] }
{ "_id" : ObjectId("592ed5818e61243307417cc5"), "name" : "Purushothaman", "degree" : "Management", "email" : "purush@example.com" }
{ "_id" : ObjectId("592ed5818e61243307417cc6"), "name" : "Meaow Meaow", "degree" : "Cat Study", "email" : "meaow@example.com", "phone" : [ "9850420420" ] }
{ "_id" : ObjectId("592ffa57962222f55cf0a823"), "name" : "Baahubali", "degree" : "Hill Climbing", "pigeon" : "baahu@mahishmati.com" }
{ "_id" : ObjectId("59300e628e61243307417cc9"), "name" : "Bleh" }
{ "_id" : ObjectId("59300e628e61243307417cca"), "name" : "Bleh" }
{ "_id" : ObjectId("59300e628e61243307417ccb"), "name" : "Blah" }
>
```

Oh yes. We have three USOs (Unidentified Student Objects). Let's perform a little cleanup now. The syntax for calling the remove function would be something like this:

```language-javascript
db.students.remove({
  "name": "Blah"
});
```

Firing the above command, we are left with:

```language-javascript
> db.students.remove({
...   "name": "Blah"
... });
WriteResult({ "nRemoved" : 1 })
>
```

Nice, one record has been removed. Let's try the `Bleh` as well.

```language-javascript
db.students.remove({
  "name": "Bleh"
});
```

The output of the above command shows us:

```language-javascript
> db.students.remove({
...   "name": "Bleh"
... });
WriteResult({ "nRemoved" : 2 })
>
```

However, it removed all the matches. While right now removing all matches is completely safe, as we know that the `Bleh`s are not supposed to be in our list, removing everything can be a problem if we need to check some stuff and remove one entry.

Luckily MongoDB provides a way: the `justOne` option. `justOne` does the exact thing that we want it to do right now:

> `justOne` *`boolean` (optional)*

> To limit the deletion to just one document, set to `true`. Omit to use the default value of `false` and delete all documents matching the deletion criteria.

Let's add two more `Bleh`s as before and we'll try this `justOne`:

```language-javascript
> db.students.insert([{"name": "Bleh"}, {"name": "Bleh"}]);
BulkWriteResult({
    "writeErrors" : [ ],
    "writeConcernErrors" : [ ],
    "nInserted" : 2,
    "nUpserted" : 0,
    "nMatched" : 0,
    "nModified" : 0,
    "nRemoved" : 0,
    "upserted" : [ ]
})
> db.students.find();
{ "_id" : ObjectId("592ebe7e8e61243307417cc4"), "name" : "Praveen Kumar", "degree" : "Cloud Computing MSc", "email" : "praveen@example.com", "subjects" : [ { "name" : "Internet Networks", "prof" : "Prof. Awesome Blossom" }, { "name" : "Cloud Computing", "prof" : "Prof. Tech Ninja" }, { "name" : "Web Development", "prof" : "Prof. Chunky Monkey" } ], "phone" : [ "9840035007", "9967728336", "7772844242" ] }
{ "_id" : ObjectId("592ed5818e61243307417cc5"), "name" : "Purushothaman", "degree" : "Management", "email" : "purush@example.com" }
{ "_id" : ObjectId("592ed5818e61243307417cc6"), "name" : "Meaow Meaow", "degree" : "Cat Study", "email" : "meaow@example.com", "phone" : [ "9850420420" ] }
{ "_id" : ObjectId("592ffa57962222f55cf0a823"), "name" : "Baahubali", "degree" : "Hill Climbing", "pigeon" : "baahu@mahishmati.com" }
{ "_id" : ObjectId("5930132a8e61243307417ccc"), "name" : "Bleh" }
{ "_id" : ObjectId("5930132a8e61243307417ccd"), "name" : "Bleh" }
> db.students.remove({
...   "name": "Bleh"
... }, {
...   "justOne": true
... });
WriteResult({ "nRemoved" : 1 })
> db.students.find();
{ "_id" : ObjectId("592ebe7e8e61243307417cc4"), "name" : "Praveen Kumar", "degree" : "Cloud Computing MSc", "email" : "praveen@example.com", "subjects" : [ { "name" : "Internet Networks", "prof" : "Prof. Awesome Blossom" }, { "name" : "Cloud Computing", "prof" : "Prof. Tech Ninja" }, { "name" : "Web Development", "prof" : "Prof. Chunky Monkey" } ], "phone" : [ "9840035007", "9967728336", "7772844242" ] }
{ "_id" : ObjectId("592ed5818e61243307417cc5"), "name" : "Purushothaman", "degree" : "Management", "email" : "purush@example.com" }
{ "_id" : ObjectId("592ed5818e61243307417cc6"), "name" : "Meaow Meaow", "degree" : "Cat Study", "email" : "meaow@example.com", "phone" : [ "9850420420" ] }
{ "_id" : ObjectId("592ffa57962222f55cf0a823"), "name" : "Baahubali", "degree" : "Hill Climbing", "pigeon" : "baahu@mahishmati.com" }
{ "_id" : ObjectId("5930132a8e61243307417ccd"), "name" : "Bleh" }
>
```

What just happened? We inserted two students, both with the same name ("Bleh") to our database. We checked that insertion occurred using `find()`. Then we used `remove()` with the `justOne` attribute to remove a single element with the desired attribute (`"name": "Bleh"`).

Thus, the option `justOne` helps us to check if we are removing the correct way by limiting to just one of the matched documents.

#### Querying with Multiple Conditions

All this time we have been limited to single-condition querying. What if we need to use multiple conditions? For example, can `Bleh` and `Blah` be removed in a single command? Well, yes! You have the mighty [`$or`](https://docs.mongodb.com/manual/reference/operator/query/or/) operator. Let's see how it works by querying stuff.

```language-javascript
db.students.insert([
  {"name": "Bleh"},
  {"name": "Blah"}
]);
```

The above command has inserted two different documents, with different name. Let's try to get both of them by using `$or` operator. In our `find()` method, we need to pass the `$or` as an option and add the different objects to be matched as an array:

```language-javascript
db.students.find({
  $or: [
    {"name": "Blah"},
    {"name": "Bleh"}
  ]
});
```

Woohoo! Executing the above code, we get both the records now.

```language-javascript
> db.students.find({$or: [{"name": "Blah"}, {"name": "Bleh"}]});
{ "_id" : ObjectId("593028b68e61243307417cd0"), "name" : "Bleh" }
{ "_id" : ObjectId("593028b68e61243307417cd1"), "name" : "Blah" }
>
```

The next thing is that, we can send this to the `remove` function as well and get all our dirt cleared.

```language-javascript
> db.students.remove({$or: [{"name": "Blah"}, {"name": "Bleh"}]});
WriteResult({ "nRemoved" : 2 })
> db.students.find();
{ "_id" : ObjectId("592ebe7e8e61243307417cc4"), "name" : "Praveen Kumar", "degree" : "Cloud Computing MSc", "email" : "praveen@example.com", "subjects" : [ { "name" : "Internet Networks", "prof" : "Prof. Awesome Blossom" }, { "name" : "Cloud Computing", "prof" : "Prof. Tech Ninja" }, { "name" : "Web Development", "prof" : "Prof. Chunky Monkey" } ], "phone" : [ "9840035007", "9967728336", "7772844242" ] }
{ "_id" : ObjectId("592ed5818e61243307417cc5"), "name" : "Purushothaman", "degree" : "Management", "email" : "purush@example.com" }
{ "_id" : ObjectId("592ed5818e61243307417cc6"), "name" : "Meaow Meaow", "degree" : "Cat Study", "email" : "meaow@example.com", "phone" : [ "9850420420" ] }
{ "_id" : ObjectId("592ffa57962222f55cf0a823"), "name" : "Baahubali", "degree" : "Hill Climbing", "pigeon" : "baahu@mahishmati.com" }
>
```

Sweet! That works.

### Operators

There are four major types of Operators in MongoDB. They are:

* **Query and Projection Operators**  
  Query operators provide ways to locate data within the database and projection operators modify how data is presented.
* **Update Operators**  
  Update operators are operators that enable you to modify the data in your database or add additional data.
* **Aggregation Pipeline Operators**  
  Aggregation pipeline operations have a collection of operators available to define and manipulate documents in pipeline stages.
* **Query Modifiers**  
  Query modifiers determine the way that queries will be executed.

We'll be looking into just a few operators in this article. Starting with the comparison operators, we need more fields in our records, so adding a few more records and later on we'll delete them.

```language-javascript
db.students.insert([
  {"name": "Bebo", "age": 5},
  {"name": "Chinna", "age": 10},
  {"name": "Elukaludha", "age": 15},
  {"name": "Kaalejoo", "age": 20},
  {"name": "Vela Thedoo", "age": 25},
  {"name": "Kanna Lammoo", "age": 30},
  {"name": "Daydee", "age": 40},
  {"name": "Paati", "age": 55},
  {"name": "Thathaa", "age": 60}
]);
```

Executing the above, command we get the following success message:

```language-javascript
> db.students.insert([
...   {"name": "Bebo", "age": 5},
...   {"name": "Chinna", "age": 10},
...   {"name": "Elukaludha", "age": 15},
...   {"name": "Kaalejoo", "age": 20},
...   {"name": "Vela Thedoo", "age": 25},
...   {"name": "Kanna Lammoo", "age": 30},
...   {"name": "Daydee", "age": 40},
...   {"name": "Paati", "age": 55},
...   {"name": "Thathaa", "age": 60}
... ]);
BulkWriteResult({
    "writeErrors" : [ ],
    "writeConcernErrors" : [ ],
    "nInserted" : 9,
    "nUpserted" : 0,
    "nMatched" : 0,
    "nModified" : 0,
    "nRemoved" : 0,
    "upserted" : [ ]
})
> db.students.find();
{ "_id" : ObjectId("593142968e61243307417cd2"), "name" : "Bebo", "age" : 5 }
{ "_id" : ObjectId("593142968e61243307417cd3"), "name" : "Chinna", "age" : 10 }
{ "_id" : ObjectId("593142968e61243307417cd4"), "name" : "Elukaludha", "age" : 15 }
{ "_id" : ObjectId("593142968e61243307417cd5"), "name" : "Kaalejoo", "age" : 20 }
{ "_id" : ObjectId("593142968e61243307417cd6"), "name" : "Vela Thedoo", "age" : 25 }
{ "_id" : ObjectId("593142968e61243307417cd7"), "name" : "Kanna Lammoo", "age" : 30 }
{ "_id" : ObjectId("593142968e61243307417cd8"), "name" : "Daydee", "age" : 40 }
{ "_id" : ObjectId("593142968e61243307417cd9"), "name" : "Paati", "age" : 55 }
{ "_id" : ObjectId("593142968e61243307417cda"), "name" : "Thathaa", "age" : 60 }
>
```

> **Note:** Intentionally the other records have been hidden in the previous output.

Okay, now we have so many names with ages. Let's try to get those who are above 18. The command and query are as follows:

```language-javascript
db.students.find({
  "age": {
     $gt: 18
   }
});
```

With the above command, we have the list of students who are aged above 18.

```language-javascript
> db.students.find({"age": {$gt: 18}});
{ "_id" : ObjectId("593142968e61243307417cd5"), "name" : "Kaalejoo", "age" : 20 }
{ "_id" : ObjectId("593142968e61243307417cd6"), "name" : "Vela Thedoo", "age" : 25 }
{ "_id" : ObjectId("593142968e61243307417cd7"), "name" : "Kanna Lammoo", "age" : 30 }
{ "_id" : ObjectId("593142968e61243307417cd8"), "name" : "Daydee", "age" : 40 }
{ "_id" : ObjectId("593142968e61243307417cd9"), "name" : "Paati", "age" : 55 }
{ "_id" : ObjectId("593142968e61243307417cda"), "name" : "Thathaa", "age" : 60 }
>
```

The other similar ones are [`$gte`, `$lt`, `$lte`, `$eq`, `$ne`, `$in`, `$nin`](https://docs.mongodb.com/manual/reference/operator/query/).

### Sorting

What's a database management system without sorting. To sort, we will be using the [`sort()`](https://docs.mongodb.com/manual/reference/method/cursor.sort/) function on any cursor (result of `find()` operation and similar). The function takes one object parameter and it has the field names as keys and the values are either `1` or `-1`, where they are either ascending or descending. So, let's try sorting the students on their name. The command is as follows:

```language-javascript
db.students.find().sort({
  "name": 1
});
```

And executing the above, we get everything sorted.

```language-javascript
> db.students.find().sort({
...   "name": 1
... });
{ "_id" : ObjectId("593142968e61243307417cd2"), "name" : "Bebo", "age" : 5 }
{ "_id" : ObjectId("593142968e61243307417cd3"), "name" : "Chinna", "age" : 10 }
{ "_id" : ObjectId("593142968e61243307417cd8"), "name" : "Daydee", "age" : 40 }
{ "_id" : ObjectId("593142968e61243307417cd4"), "name" : "Elukaludha", "age" : 15 }
{ "_id" : ObjectId("593142968e61243307417cd5"), "name" : "Kaalejoo", "age" : 20 }
{ "_id" : ObjectId("593142968e61243307417cd7"), "name" : "Kanna Lammoo", "age" : 30 }
{ "_id" : ObjectId("593142968e61243307417cd9"), "name" : "Paati", "age" : 55 }
{ "_id" : ObjectId("593142968e61243307417cda"), "name" : "Thathaa", "age" : 60 }
{ "_id" : ObjectId("593142968e61243307417cd6"), "name" : "Vela Thedoo", "age" : 25 }
>
```

Let's try it the other way too:

```language-javascript
> db.students.find().sort({   "name": -1 });
{ "_id" : ObjectId("593142968e61243307417cd6"), "name" : "Vela Thedoo", "age" : 25 }
{ "_id" : ObjectId("593142968e61243307417cda"), "name" : "Thathaa", "age" : 60 }
{ "_id" : ObjectId("593142968e61243307417cd9"), "name" : "Paati", "age" : 55 }
{ "_id" : ObjectId("593142968e61243307417cd7"), "name" : "Kanna Lammoo", "age" : 30 }
{ "_id" : ObjectId("593142968e61243307417cd5"), "name" : "Kaalejoo", "age" : 20 }
{ "_id" : ObjectId("593142968e61243307417cd4"), "name" : "Elukaludha", "age" : 15 }
{ "_id" : ObjectId("593142968e61243307417cd8"), "name" : "Daydee", "age" : 40 }
{ "_id" : ObjectId("593142968e61243307417cd3"), "name" : "Chinna", "age" : 10 }
{ "_id" : ObjectId("593142968e61243307417cd2"), "name" : "Bebo", "age" : 5 }
>
```

Ah blimey! Both work flawlessly.

### Limiting Results

With the previous example, we were able to sort students based on age, in either descending or ascending order. Now, if we needed to find the oldest of the group, we'd simply sort them descending based on their age and then limit our result to one student. 

Similar to the aggregate functions in traditional RDBs, MongoDB has a function called [`limit()`](https://docs.mongodb.com/manual/reference/method/cursor.limit/) that can be used on the `find()` results AKA the cursor.

The `limit()` function takes one parameter, which is the integral value of limiting to the number of results given in it. In our case, it is going to be `1`.

```language-javascript
db.students.find().sort({
  "age": -1
}).limit(1);
```
Ah, here we go. We get only `Thatha` as our senior student.

```language-javascript
> db.students.find().sort({
...   "age": -1
... }).limit(1);
{ "_id" : ObjectId("593142968e61243307417cda"), "name" : "Thathaa", "age" : 60 }
>
```

### Counting Records

There should be a way to count the number of students we have, right? Yep, let's try the [`count()`](https://docs.mongodb.com/manual/reference/method/cursor.count/) on the cursor. To do that, we have the command:

```language-javascript
db.students.find().count();
```

And wow, we get the nine students:

```language-javascript
> db.students.find().count();
9
>
```

Both `limit()` and `count()` should feel very familiar if you've used MySQL (or any RDB)

### Iterating Results

We all know that MongoDB is nothing but JavaScript. So, why not use the Array and Object functions on the Database? Yep, it is possible using [`forEach()`](https://docs.mongodb.com/manual/reference/method/cursor.forEach/) method, which can be applied on `find()` results.

`forEach()` takes a function as its only argument. The current document or result or record will be passed to it. Basically, the function has the following syntax:

```language-javascript
db.students.find().forEach(function (stud) {
  print("Student Name: " + stud.name);
});
```

We can use the concatenation operator like our regular JavaScript and executing the above, we get:

```language-javascript
> db.students.find().forEach(function (stud) {
...   print("Student Name: " + stud.name);
... });
Student Name: Bebo
Student Name: Chinna
Student Name: Elukaludha
Student Name: Kaalejoo
Student Name: Vela Thedoo
Student Name: Kanna Lammoo
Student Name: Daydee
Student Name: Paati
Student Name: Thathaa
>
```
Clearly, having a JavaScript-like structure makes MongoDB more versatile than RDBs. Whereas MySQL would struggle to perform the task above, MongoDB crushes this problem.

## Conclusion

I hope this tutorial exposed the basics of MongoDB, an increasingly popular NoSQL database that ca enhance pretty much any application that relies on using lots of data. 

As usual, drop your suggestions and ideas in the comments section below. Thanks for reading!