---
layout: post
title:  "How to create Rails application with MongoDB?"
date:   2019-08-30 09:23:58 +0545
categories: [MongoDB, Ruby on Rails]
tags: [mongodb, rails_setup]
---

# Setup Rails 5 with mongoid gem

At first we need to install MongoDB in our system, the steps to install MongoDB is descripted in my previous blog. Confirm mongoDB is installed by browsing http://localhost:27017/ and you will get following message:
```
It looks like you are trying to access MongoDB over HTTP on the native driver port.
```

Create a Rails application with the keyword "--skip-active-record" so that ActiveRecord is not included in the generated app.

```
rails new myapp --skip-active-record
```

Edit your Gemfile

```
Remove this Gem if exists
gem 'sqlite3'


And add following Gems:

gem 'mongoid', '~> 6.2.0'
gem 'bson_ext'
```

Generate configuration file to support MongoDB which generates config/mongoid.yml
```
rails g mongoid:config
```

There is a file called /config/mongoid.yml' which contains database configuration and it is required.

The Rails generators for 'model', 'scaffold' etc have been overridden by Mongoid. Any models, scaffolds etc that you create will create classes that include the Mongoid::Document module instead of inheriting from ApplicationRecord in the models folder.

#### Association

```
rails generate scaffold article title:string
rails generate scaffold comment body:string article_id:string # Here article_id required when implementing has_many association but not required in case of embedds many and even records are not saved inside Comment document which is included inside Article document.
```

Association embeds_many with embedded_in

```
class Article
  include Mongoid::Document
  field :title, type: String

  embeds_many :comments
end

class Comment
  include Mongoid::Document
  field :title, type: String
  field :article_id, type: String

  embedded_in :article
end

article = Article.new(title: "Embeds Many association on MongoDB")
article.comments.build(title: 'Embeds Many association will connect child records inside parent record')


# if you check the mongo console `db.articles.find()` you will notice Comments records are also included inside the Article record and no seperate comment document is created inside comment collection.
{ "_id" : ObjectId("5d6f75b567ef9b0d9e8373b4"), "title" : "Embeds Many association on MongoDB", "comments" : [ { "_id" : ObjectId("5d6f75bd67ef9b0d9e8373b5") }, { "_id" : ObjectId("5d6f75ef67ef9b0d9e8373b6"), "title" : "Embeds Many association will connect child records inside parent record" } ] }
```

Association has_many with belongs_to

```
class Article
  include Mongoid::Document
  field :title, type: String

  has_many :comments
end

class Comment
  include Mongoid::Document
  field :title, type: String
  field :article_id, type: String

  belongs_to :article
end

`db.articles.find()`
{ "_id" : ObjectId("5d6f6a8067ef9b06fed2c32e"), "title" : "first article" }

`db.comments.find()`
{ "_id" : ObjectId("5d6f6e2567ef9b0c888373b1"), "title" : "comment title", "article_id" : ObjectId("5d6f6a8067ef9b06fed2c32e") }

In this senario, the records are saved inside two independed mongoDB collection Articles and Comments. And inside Comment Document we will have article_id whose value is the corressponding Article Id. But in embeds_many technique the child records do not save inside the Comment collection but inside Article Collection included inside Article document.
```