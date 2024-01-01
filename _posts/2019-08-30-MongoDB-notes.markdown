---
layout: post
title:  "Notes on MongoDB"
date:   2019-08-30 09:23:58 +0545
categories: [MongoDB, Data Operation]
tags: [mongodb, data_operation]
---

#### Introduction to MongoDB

MongoDB is a open source document-oriented NoSQL database used for high volume data storage. 
If database is not already created switch to the database and insert data into it, this way database is created.
Each record in a MongoDB collection is a document. MongoDB collections are like table and documents are like rows of the relational databases.

## Create Database

```
use NewDatabase # switched to db NewDatabase
db.products.insert({name: 'product', price: 20}) # Create a collection name as products with new document as a record
```

## View database and collections

```
> show dbs;

admin
config
local
NewDatabase

> use NewDatabase

switched to db NewDatabase

> show collections

products
```

## Delete Database

```
> db.dropDatabase()

{ "dropped" : "NewDatabase", "ok" : 1 }
```

## Crud operations

##### Insert a Single Document

db.collection.insertOne() inserts a single document into a collection.

MongoDB adds the _id field with an ObjectId value to the new document.

```
db.customers.insertOne(
    {
        profile_name: 'customer name',
        email: 'email@example.com',
        age: 32,
        tags: ["regular"],
        full_name: { first_name: 'firstname', mid_name: 'midname', last_name: 'last_name' }
    }
)
```
#### when you run the above command, you will get following output
```
{
	"acknowledged" : true,
	"insertedId" : ObjectId("5d6ccbfda82b6d69714cebeb")
}
```

##### Insert multiple documents

```
db.collection.insertMany()
```

```
db.customers.insertMany(
    [
        {
          profile_name: 'customer name 2',
          email: 'email10@example.com',
          age: 22,
          tags: ["regular"],
          full_name: { first_name: 'firstname2', mid_name: 'midname2', last_name: 'last_name2' }
        },
        {
          profile_name: 'customer name 3',
          email: 'email11@example.com',
          age: 22,
          tags: ["regular"],
          full_name: { first_name: 'firstname3', mid_name: 'midname3', last_name: 'last_name3' }
        },
        {
          profile_name: 'customer name 4',
          email: 'email12@example.com',
          age: 22,
          tags: ["regular"],
          full_name: { first_name: 'firstname4', mid_name: 'midname4', last_name: 'last_name4' }
        }
    ]
)
```

=> output when executing above query
```
{
	"acknowledged" : true,
	"insertedIds" : [
		ObjectId("5d6ccdf9a82b6d69714cebec"),
		ObjectId("5d6ccdf9a82b6d69714cebed"),
		ObjectId("5d6ccdf9a82b6d69714cebee")
	]
}

```

##### View Record

```
> db.customers.find({profile_name: 'customer name'})
{ "_id" : ObjectId("5d6ccbfda82b6d69714cebeb"), "profile_name" : "customer name", "email" : "email@example.com", "age" : 32, "tags" : [ "regular" ], "full_name" : { "first_name" : "firstname", "mid_name" : "midname", "last_name" : "last_name" } }
```

# multiple matched records
```
> db.customers.find({age: 22})
{ "_id" : ObjectId("5d6ccdf9a82b6d69714cebee"), "profile_name" : "customer name 4", "email" : "email4@example.com", "age" : 22, "tags" : [ "regular" ], "full_name" : { "first_name" : "firstname4", "mid_name" : "midname4", "last_name" : "last_name4" } }
{ "_id" : ObjectId("5d6cd0eb68f1285dc9559364"), "profile_name" : "customer name 2", "email" : "email10@example.com", "age" : 22, "tags" : [ "regular" ], "full_name" : { "first_name" : "firstname2", "mid_name" : "midname2", "last_name" : "last_name2" } }
{ "_id" : ObjectId("5d6cd0eb68f1285dc9559365"), "profile_name" : "customer name 3", "email" : "email11@example.com", "age" : 22, "tags" : [ "regular" ], "full_name" : { "first_name" : "firstname3", "mid_name" : "midname3", "last_name" : "last_name3" } }
{ "_id" : ObjectId("5d6cd0eb68f1285dc9559366"), "profile_name" : "customer name 4", "email" : "email12@example.com", "age" : 22, "tags" : [ "regular" ], "full_name" : { "first_name" : "firstname4", "mid_name" : "midname4", "last_name" : "last_name4" } }
```

> db.customers.find({profile_name: "customer name 4"})
```
{ "_id" : ObjectId("5d6ccdf9a82b6d69714cebee"), "profile_name" : "customer name 4", "email" : "email4@example.com", "age" : 22, "tags" : [ "regular" ], "full_name" : { "first_name" : "firstname4", "mid_name" : "midname4", "last_name" : "last_name4" } }
{ "_id" : ObjectId("5d6cd0eb68f1285dc9559366"), "profile_name" : "customer name 4", "email" : "email12@example.com", "age" : 22, "tags" : [ "regular" ], "full_name" : { "first_name" : "firstname4", "mid_name" : "midname4", "last_name" : "last_name4" } }
```

> db.customers.find({profile_name: "customer name 4", "email": "email4@example.com"})
```
{ "_id" : ObjectId("5d6ccdf9a82b6d69714cebee"), "profile_name" : "customer name 4", "email" : "email4@example.com", "age" : 22, "tags" : [ "regular" ], "full_name" : { "first_name" : "firstname4", "mid_name" : "midname4", "last_name" : "last_name4" } }
```

> db.customers.find({profile_name: "customer name 4"}).limit(1)
```
{ "_id" : ObjectId("5d6ccdf9a82b6d69714cebee"), "profile_name" : "customer name 4", "email" : "email4@example.com", "age" : 22, "tags" : [ "regular" ], "full_name" : { "first_name" : "firstname4", "mid_name" : "midname4", "last_name" : "last_name4" } }
```

#### Show all records
> db.customers.find()
```
{ "_id" : ObjectId("5d6ccbfda82b6d69714cebeb"), "profile_name" : "customer name", "email" : "email@example.com", "age" : 32, "tags" : [ "regular" ], "full_name" : { "first_name" : "firstname", "mid_name" : "midname", "last_name" : "last_name" } }
{ "_id" : ObjectId("5d6ccdf9a82b6d69714cebec"), "profile_name" : "customer name 2", "email" : "email2@example.com", "age" : 30, "tags" : [ "regular" ], "full_name" : { "first_name" : "firstname2", "mid_name" : "midname2", "last_name" : "last_name2" } }
{ "_id" : ObjectId("5d6ccdf9a82b6d69714cebed"), "profile_name" : "customer name 3", "email" : "email3@example.com", "age" : 36, "tags" : [ "regular" ], "full_name" : { "first_name" : "firstname3", "mid_name" : "midname3", "last_name" : "last_name3" } }
{ "_id" : ObjectId("5d6ccdf9a82b6d69714cebee"), "profile_name" : "customer name 4", "email" : "email4@example.com", "age" : 22, "tags" : [ "regular" ], "full_name" : { "first_name" : "firstname4", "mid_name" : "midname4", "last_name" : "last_name4" } }
{ "_id" : ObjectId("5d6cd0eb68f1285dc9559364"), "profile_name" : "customer name 2", "email" : "email10@example.com", "age" : 22, "tags" : [ "regular" ], "full_name" : { "first_name" : "firstname2", "mid_name" : "midname2", "last_name" : "last_name2" } }
{ "_id" : ObjectId("5d6cd0eb68f1285dc9559365"), "profile_name" : "customer name 3", "email" : "email11@example.com", "age" : 22, "tags" : [ "regular" ], "full_name" : { "first_name" : "firstname3", "mid_name" : "midname3", "last_name" : "last_name3" } }
{ "_id" : ObjectId("5d6cd0eb68f1285dc9559366"), "profile_name" : "customer name 4", "email" : "email12@example.com", "age" : 22, "tags" : [ "regular" ], "full_name" : { "first_name" : "firstname4", "mid_name" : "midname4", "last_name" : "last_name4" } }
```

## Upsert Option (updateOne(), updateMany(), replaceOne())

#### Replace a record except an ID field

```
db.customers.replaceOne({email: 'email@example.com'}, {"profile_name" : "customer name", "email" : "email@example.com", "age" : 15, "tags" : [ "nonregular" ], "full_name" : { "first_name" : "firstname", "mid_name" : "midname", "last_name" : "last_name" }})
```

`{ "acknowledged" : true, "matchedCount" : 1, "modifiedCount" : 1 }`

When you search the record which is just updated, you will notice update has been made:

```
db.customers.find({email: 'email@example.com'})
{ "_id" : ObjectId("5d6ccbfda82b6d69714cebeb"), "profile_name" : "customer name", "email" : "email@example.com", "age" : 15, "tags" : [ "nonregular" ], "full_name" : { "first_name" : "firstname", "mid_name" : "midname", "last_name" : "last_name" } }
```

#### Update a record execpt an ID field

First track the record you want to update

```
> db.customers.find({email: 'email2@example.com'})

{ "_id" : ObjectId("5d6ccdf9a82b6d69714cebec"), "profile_name" : "customer name 2", "email" : "email2@example.com", "age" : 30, "tags" : [ "regular" ], "full_name" : { "first_name" : "firstname2", "mid_name" : "midname2", "last_name" : "last_name2" } }
>
```

Apply Update command

```
> db.customers.updateOne({email: 'email2@example.com'}, {$set: {"profile_name": 'customer name 2 updated', "full_name.last_name": "last_name2 updated", tags: ["regular updated"]}})

{ "acknowledged" : true, "matchedCount" : 1, "modifiedCount" : 1 }
```

Now check the record and you will notice the record is updated

```
> db.customers.find({email: 'email2@example.com'})

{ "_id" : ObjectId("5d6ccdf9a82b6d69714cebec"), "profile_name" : "customer name 2 updated", "email" : "email2@example.com", "age" : 30, "tags" : [ "regular updated" ], "full_name" : { "first_name" : "firstname2", "mid_name" : "midname2", "last_name" : "last_name2 updated" } }
```

Add lastModified field when you update the record which will add new column

```
> db.customers.updateOne({email: 'email2@example.com'}, {$set: {"profile_name": 'customer name 2 updated', "full_name.last_name": "last_name2 updated", tags: ["regular updated"]}, $currentDate: { lastModified: true }})

{ "_id" : ObjectId("5d6ccdf9a82b6d69714cebec"), "profile_name" : "customer name 2 updated", "email" : "email2@example.com", "age" : 30, "tags" : [ "regular updated" ], "full_name" : { "first_name" : "firstname2", "mid_name" : "midname2", "last_name" : "last_name2 updated" }, "lastModified" : ISODate("2019-09-02T08:55:35.114Z") }
``1

The update action involves following operations:

uses the $set operator to update the value of the size.uom field to "cm" and the value of the status field to "P",
uses the $currentDate operator to update the value of the lastModified field to the current date. If lastModified field does not exist, $currentDate will create the field. See $currentDate for details.


#### UpdateMany

Update all the documents where age is greater than 22

```
db.customers.updateMany(
    {
        "age": { $gt: 22 }
    },
    {
        $set: { "tags": "Multiple Update", active: "true" },
        $currentDate: { lastModified: true }
    }
)
```
>> { "acknowledged" : true, "matchedCount" : 2, "modifiedCount" : 2 }

Now it's time to check updated record:

```
> db.customers.find({"age": {$gt: 22}})
{ "_id" : ObjectId("5d6ccdf9a82b6d69714cebec"), "profile_name" : "customer name 2 updated", "email" : "email2@example.com", "age" : 30, "tags" : "Multiple Update", "full_name" : { "first_name" : "firstname2", "mid_name" : "midname2", "last_name" : "last_name2 updated" }, "lastModified" : ISODate("2019-09-02T12:56:37.700Z"), "active" : "true" }
{ "_id" : ObjectId("5d6ccdf9a82b6d69714cebed"), "profile_name" : "customer name 3", "email" : "email3@example.com", "age" : 36, "tags" : "Multiple Update", "full_name" : { "first_name" : "firstname3", "mid_name" : "midname3", "last_name" : "last_name3" }, "active" : "true", "lastModified" : ISODate("2019-09-02T12:56:37.700Z") }
```

#### Some other options:
```
db.collection.findOneAndReplace().
db.collection.findOneAndUpdate().
db.collection.findAndModify().
db.collection.save().
db.collection.bulkWrite()
```

#### Delete document (deleteOne(), deleteMany())

Delete only one record no matter multiple records for this profile name exists
```
> db.customers.deleteOne({profile_name: 'profile name'})

o/p => { "acknowledged" : true, "deletedCount" : 1 }
```

Delete many records at once
```
> db.customers.deleteMany({profile_name: 'profile name 1'})
o/p => { "acknowledged" : true, "deletedCount" : 4 }
```

### Bulk Write with bulkWrite()

```
try {
    db.customers.bulkWrite([
        { insertOne: { "document": {
                        profile_name: 'foo bar',
                        email: 'foo@shivrajbadu.com.np',
                        age: 29,
                        tags: ["regular"],
                        full_name: { first_name: 'shiv', mid_name: 'raj', last_name: 'badu' }
                    }
                }
            },
            { insertOne: { "document": {
                    profile_name: 'foo baz',
                    email: 'baz@shivrajbadu.com.np',
                    age: 25,
                    tags: ["regular"],
                    full_name: { first_name: 'foo', mid_name: 'bar', last_name: 'baz' }
                    }
                }
            },
            { updateOne: {
                    "filter" : { email: 'email10@example.com' },
                    "update" : { $set: {"profile_name": 'new name', tags: ["irregular"], "full_name.first_name": "fn", "full_name.mid_name": "mn", "full_name.last_name": "ln" } }
                }
            },
            { deleteOne: {
                    "filter": { profile_name: 'customer name' }
                }
            },
            { replaceOne: { 
                "filter": { email: 'email@example.com' },
                "replacement": { "profile_name": 'profile name', "email": 'email@example.com', "age": 14, "tags": ["irregular"], "full_name": { "first_name": "fn", "last_name": "ln", "mid_name": "mn" }  }
             } }
        ]);
} catch (e) {
    print(e);
}
```

#### Text Search

To perform text search use `text index` and $text operator, `text` indexes can include any field whose value is a string or an array of string elements. To perform text search queries, you must have a `text` index on your collection. A collection can only have one text search index, but that index can conver multiple fields.

If index is not found you will get following error message:

```
Error: error: {
	"ok" : 0,
	"errmsg" : "text index required for $text query",
	"code" : 27,
	"codeName" : "IndexNotFound"
}
```
So, you need to create Index first

```
db.customers.createIndex({
    profile_name: "text",
    email: "text"
})
```
```
db.customers.find({
    $text: {
        $search: "myemail@shivrajbadu.com.np"
    }
})
```

```
db.customers.aggregate(
    [
        { $match: { $text: { $search: "first name" } } }
    ]
)
```

To get exact match result of searched text

```
db.customers.aggregate(
    [
        { $match: { $text: { $search: "\"customer name replaced\"" } } }
    ]
)
```

### References

```
user document             contact document                            articles document
-------------             -----------------                          -----------------
{                         {                                            {
    _id: <ObjectId1>,        _id: <ObjectId2>,                            _id: <ObjectId3>,
    username: 'xyz'           user_id: <ObjectId1>,                       user_id: <ObjectId1>,
}                             phone: '9852525252',                        title: 'first article',
                              email: 'contact@shivrajbadu.com.np'         body: 'article body'
                          }                                            }
```

### One-to-One Relationships with Embedded Documents
#### Contact document contains a reference to the User document. 

User Document
```
{
    _id: "unique_id",
    username: 'uniquename'
}

Contact Document
```
{
    _id: "ObjectId("5d6df862e1b6226e35c6c519")",
    _user_id: "unique_id",
    phone: "8585858585",
    email: "contact@shivrajbadu.com.np"
}
```

### One-to-Many Relationships with Embedded Documents
In the normalized data model, the articles documents contain a reference to the user document.

User Document
```
{
    _id: "unique_id",
    username: 'uniquename'
}

Article Document
```
{
    _id: "ObjectId("5d6df9b5e1b6226e35c6c522")",
    _user_id: "unique_id",
    title: "This is a title.",
    body: "This is a description."
}

{
    _id: "ObjectId("9e7df9b5e1b6226e35c6c435")",
    _user_id: "unique_id",
    title: "This is another title.",
    body: "This is description for another title."
}
```

When implement one to many relationships, many child records will have many child document records so multiple queries need to be issued to resolve the references, we can also use another solution to make single query as shown below:

```
{
    _id: "unique_id",
    username: 'uniquename',
    articles: [
        {
            _id: "ObjectId("5d6df9b5e1b6226e35c6c522")",
            _user_id: "unique_id",
            title: "This is a title.",
            body: "This is a description."
        },
        {
            _id: "ObjectId("9e7df9b5e1b6226e35c6c435")",
            _user_id: "unique_id",
            title: "This is another title.",
            body: "This is description for another title."
        }
    ]
}
```

{% include inarticle-adsense.html %}