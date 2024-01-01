---
layout: post
title:  "Elastic Search with Chewy"
date:   2019-10-25 07:01:18 +0545
categories: [Ruby on Rails, Elastic search Chewy]
tags: [elastic_search, chewy]
---

Chewy is one of the elastic search Ruby client.

Chewy usages:

* Multi-model indices
You can define several types for index one per indexed model.

* Every index is observable by all the related models.
Most of the indexed models are related to other and it is necessary to denormalize this related data and put at the same object. Chewy is useful for example when we need index for an array of tags together with an article since it specify updated index for every model seperately so corressponding articles will be reindexed on any tag update.

* Bulk import everywhere
It supports bulk elastic search api for full reindex and index updates.

* Powerful querying DSL
Chewy has an ActiveRecord style query DSL.

* Support for ActiveRecord, Mongoid and Sequel.

#### Installation Steps:

```Ruby
gem 'chewy'

bundle install
```

or gem install chewy


#### Client settings:

`Chewy.settings` hash and `chewy.yml` are two ways in which Chewy client can be configured.

Run the command `rails g chewy:install` to generate the file or create one manually.

```Ruby
# config/chewy.yml
# separate environment configs
test:
  host: 'localhost:9250'
  prefix: 'test'
development:
  host: 'localhost:9200'
```

#### config/initializers/chewy.rb

```Ruby
Chewy.settings = {host: 'localhost:9250'} # do not use environments
```

#### Aws Elastic Search

Configuration for using AWS's elastic search using an IAM user policy, sign your requests for the es:* action by injecting the headers passing a proc to transport_options.

```Ruby
Chewy.settings = {
    host: 'http://my-es-instance-on-aws.us-east-1.es.amazonaws.com:80',
    transport_options: {
      headers: { content_type: 'application/json' },
      proc: -> (f) do
          f.request :aws_signers_v4,
                    service_name: 'es',
                    region: 'us-east-1',
                    credentials: Aws::Credentials.new(
                      ENV['AWS_ACCESS_KEY'],
                      ENV['AWS_SECRET_ACCESS_KEY'])
      end
    }
  }
```

#### Type access

Following API is used to access index-defined types

```Ruby
UsersIndex::User
UsersIndex.type_hash['user']
UsersIndex.type('user')
UsersIndex.type('foo')
UsersIndex.types # [UserIndex::User]
UsersIndex.type_names # ["user"] 
```

#### Index Manipulation

```Ruby
UsersIndex.delete # destroy existed index
UsersIndex.delete!

UsersIndex.create # create index
UsersIndex.create!

UsersIndex.purge
UsersIndex.purge! # deletes then creates index

UsersIndex::User.import # import with 0 arguments process all the data specified in type definition
UsersIndex::User.import User.where('rating > 100') # or import specified users scope
UsersIndex::User.import User.where('rating > 100').to_a # or import specified users array
UsersIndex::User.import [1, 2, 42] # pass even ids for import, it will be handled in the most effective way
UsersIndex::User.import user: User.where('rating > 100')  # if update fields are specified - it will update their values only with the `update` bulk action.
UsersIndex.reset! # purges index and imports default data for all types
```

#### Practical on Ruby on Rails application

app/chewy/user_index.rb


```Ruby
class UserIndex < Chewy::Index
    settings analysis: {
      analyzer: {
        email: {
          tokenizer: 'keyword',
          filter: ['lowercase']
        }
      }
    }
  
    define_type User do
      field :name, {type: 'text'}
      field :email, analyzer: 'email'
      field :phone, {type: 'text'}
    end
  end
```

app/controllers/users_controller.rb

```Ruby
class UsersController < ApplicationController
    def search
      @users = UsersIndex.query(query_string: { fields: [:name, :email, :phone], query: search_params[:query], default_operator: 'and' })
  
      render json: @users.to_json, status: :ok
    end
  
    private
  
    def search_params
      params.permit(:query, :page, :per)
    end
  end
```

app/models/user.rb

```Ruby
class User < ApplicationRecord
    update_index('user') { self }
    enum status: { unconfirmed: 0, confirmed: 1 }
end
```

routes.rb

```Ruby
resources :users do
    get :search, on: :collection
end
```

If you access the url `http://localhost:3000/users/search?query=test1`

Following results are seen on the browser

```Ruby
0	
id	"18"
name	"test1"
status	"unconfirmed"
email	"test1@example.com"
phone	"090111111"
_score	0.5389965
_explanation	null
1	
id	"3"
name	"test1"
status	"unconfirmed"
email	"test1@example.com"
phone	"090111112"
_score	0.5389965
_explanation	null
2	
id	"45"
name	"test1"
email	"test1@example.com"
phone	"090111111"
_score	0.5389965
_explanation	null
```

and if we inspect the result of @users object on controller.first on console, we will see

```Ruby
@_data=
  {"_index"=>"user",
   "_type"=>"user",
   "_id"=>"18",
   "_score"=>0.5389965,
   "_source"=>{"name"=>"test1", "status"=>"unconfirmed", "email"=>"test1@example.com", "phone"=>"090111111"}},
 @attributes=
  {"id"=>"18",
   "name"=>"test1",
   "status"=>"unconfirmed",
   "email"=>"test1@example.com",
   "phone"=>"090111111",
   "_score"=>0.5389965,
   "_explanation"=>nil}
```

We can refactor the searching as:

Create a dir called as app/searches/user_search.rb

```Ruby
# user_search.rb
# frozen_string_literal: true

class UserSearch
  include ActiveModel::Model

  DEFAULT_PER_PAGE = 10
  DEFAULT_PAGE = 0

  attr_accessor :query, :page, :per

  def search
    [query_string].compact.reduce(&:merge).page(page_num).per(per_page)
  end

  def query_string
    index.query(query_string: { fields: [:name, :email, :phone], query: query, default_operator: 'and' }) if query.present?
  end

  private

  def index
    UsersIndex
  end

  def page_num
    page || DEFAULT_PAGE
  end

  def per_page
    per || DEFAULT_PER_PAGE
  end
end
```

Now call the UserSearch class and implement it inside the UsersController

```Ruby
class UsersController < ApplicationController
  def search
    user_search = UserSearch.new(search_params)
    @users = user_search.search

    render json: @users, status: :ok
  end

  private

  def search_params
    params.permit(:query, :page, :per)
  end
end
```

Now modify the search action as:

```Ruby
class UsersController < ApplicationController
  def search
    user_search = UserSearch.new(search_params)
    @users = user_search.search
  end

  private

  def search_params
    params.permit(:query, :page, :per)
  end
end
```

search.html.erb

```Ruby
<% if @users.any? %>
    <table border="1">
        <tr>
            <th>Id</th>
            <th>Name</th>
            <th>Phone</th>
            <th>Email</th>
            <th>Status</th>
        </tr>
        <% @users.each do |user| %>
        <% res = user.attributes %>
        <tr>
            <td><%= res["id"] %></td>
            <td><%= res["name"] %></td>
            <td><%= res["phone"] %></td>
            <td><%= res["email"] %></td>
            <td><%= res["status"] %></td>
        </tr>
        <% end %>
    </table>
<% else %>
    <p>No users found.</p>
<% end %>
```

{% include inarticle-adsense.html %}