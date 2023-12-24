---
layout: post
title:  "Association in Ruby on Rails!"
date:   2018-04-21 09:23:58 +0545
categories: [Ruby on Rails, Association]
tags: [ruby on rails, association]
---

# Ruby on Rails Association

When column on database table grows, the column needs to be put into new table if the records emphasizes on data redundancy and data dependency and data normalization takes place. So, various tables are created which are linked or connected with one another via foreign keys. When connection takes place between two associated Active Record Models it is called as an Association.

##### Various Types of Association

1. One-to-One
2. One-to-Many
3. Many-to-Many
4. Polymorphic One-to-Many

#### One-to-One

In this type of association, the data records contains one instance of another model.

Model look like this:

```
class User < ApplicationRecord           class CitizenshipNumber < ApplicationRecord
    has_one :citizenship_number             belongs_to :user                        
end                                      end
```

Table look like this:

```
  users                      profiles
  -----                      ---------
  id  (primary key)           id
  username                    name
  password                    user_id (foreign key)
```

#### One-to-Many

In this type of association, instance of first model can have zero or more than one instances of second model and second model belongs to only first model.

Model look like this:

```
class User < ApplicationRecord         class Post < ApplicationRecord
    has_many :posts                       belongs_to :user
end                                    end
```

Table look like this:

```
  users                      posts
  -----                      ---------
  id  (primary key)           id
  username                    title
  password                    user_id (foreign key)
```

#### Many to Many

It can be handled in two ways: has and belongs to many and has many through relationships.

##### Has and belongs to many

In this type of association, has_and_belongs_to_many methods is called from both the models in order to create many to many connection with another model. Rails migration need to be created in the following format in order to create join table.

```
rails g migration CreateJoinTableUserPost user post
```

which generates migration file like this:

```
class CreateJoinTableUserPost < ActiveRecord::Migration[5.0]
  def change
    create_join_table :users, :posts do |t|
      t.index [:user_id, :post_id]
    end
  end
end
```

Model look like this:

```
class User < ApplicationRecord         class Post < ApplicationRecord
  has_and_belongs_to_many :posts         has_and_belongs_to_many :users
end                                    end
```

Table look like this:

```
  users                users_posts                posts
  -----                -----------                ---------
  id  (primary key)    id                         id
  username             user_id (foreign key)      name
  password             post_id (foreign key)      
```

##### Has many through

In this type of many-to-many association, unlike join intermediate table was created in has_and_belongs_to_many, but join intermediate model is created which points both the associated parent model.

Model look like this:

```
class User < ApplicationRecord                class Post < ApplicationRecord
  has_many :posts, through: user_posts           has_many :users, through: user_posts
  has_many :user_posts                           has_many :user_posts
end                                           end

class UserPost < ApplicationRecord
    belongs_to :users
    belongs_to :posts
end
```

Table look like this:

```
  users                user_posts                 posts
  -----                -----------                ---------
  id  (primary key)    id                         id
  username             user_id (foreign key)      name
  password             post_id (foreign key)      
```

#### Polymorphic One-to-Many

In this type of association, one model is belongs to many different models on a single association. Let's say Post has video, Article has Video, UserProfile has video, Blog has video. So all these models Post, Article, UserProfile, Blog needs to be handled by polymorphic interface called as Videoable.

Model look like this:

```
class Post < ApplicationRecord         class Article < ApplicationRecord
  has_many :video, as: :videoable         has_many :video, as: :videoable
end                                    end

class UserProfile < ApplicationRecord  class Blog < ApplicationRecord
  has_many :video, as: :videoable         has_many :video, as: :videoable
end                                    end

class Videoable < ApplicationRecord
    belongs_to :videoable, polymorphic: true
end
```

Generally polymorphic table needs type column (videoable_type: string) and foreign_key column (videoable_id: integer)

Table look like this:

```
  videos               Post      Article   Blog    UserProfile    
  -----                -----     -------   -----   -----------
  id  (primary key)     id        id        id      id
  videoable_id          title     title     title   full_name
  videoable_type 
```

Migration:

```
class CreateVideos < ActiveRecord::Migration
  def change
    create_table :videos do |t|
      t.integer :videoable_id
      t.string :videoable_type
      t.timestamps
    end
    add_index :videos, :videoable_id
  end
end

# or

class CreateVideos < ActiveRecord::Migration
  def change
    create_table :videos do |t|
      t.references :videoable, polymorphic: true, index: true
      t.timestamps
    end
  end
end
```