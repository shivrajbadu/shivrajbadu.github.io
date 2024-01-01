---
layout: post
title:  "Elastic Search with Searchkick"
date:   2019-10-18 08:01:18 +0545
categories: [Ruby on Rails, Elastic search Searchkick]
tags: [elastic_search, searchkick, ruby on rails]
---

# What is Searchkick?

Searchkick is a smart and intillegent search engine Rubygems that creates quicker search results based on user search activity.

Before using Searchkick make sure Elasticsearch is installed on your system.

# Steps to use Searchkick

* Create a Rails application

```Ruby
rails new institutions -d postgres
```

* Generate the scaffold for Student

```Ruby
rails g scaffold Student name:string roll:integer grade:string fee:decimal
```

* Run rake db:create rake db:migrate

* Configure the routes

```Ruby
root "students#index"
resources :students
```

* Add following gem into Gemfile

```Ruby
gem 'searchkick'
```

Here is the Guide for Elasticsearch 6 or 7.

* In each models you need to add keyword `searchkick` to make searchkick work as shown below

```Ruby
class Student < ApplicationRecord
	searchkick
end
```

* Now add data to search index by using following code and you need to run this command everytime as model changes

```Ruby
Student.reindex
```

There are many ways search options based on necessity:

```Ruby
:word # default
:word_start
:word_middle
:word_end
:text_start
:text_middle
:text_end
```

Here is an example of using :word_start for partial match criteria

```Ruby
class Student < ApplicationRecord
  searchkick word_start: [:name, :role, :grade, :fee]
  def search_data
    {
      name: name,
      role: role,
      grade: grade,
      fee: fee
    }
  end
end
```

Search Everything

```Ruby
Student.search "*"
```

Partial Matches

```Ruby
Student.search "Shiv Raj Badu" # Shiv AND Raj AND Badu
Book.search "Shiv Raj Badu", operator: "or"
```

Exact Matches

```Ruby
Student.search params[:search], fields:[{fee: :exact}, :name]
```

Phrase Matches

```Ruby
Student.search "another name", match: :phrase
```

Model associations

```Ruby
Student.search "shiv raj", track: {user_id: current_user.id}
```

Autocomplete and Instant Search

```Ruby
class Student < ApplicationRecord
  searchkick match: :word_start, searchable: [:name, :roll]
end
```

Language supported based on [list](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-stemmer-tokenfilter.html#analysis-stemmer-tokenfilter)

```Ruby
searchkick word_start: [:title, :author, :genre], language: "turkish"
```

```Ruby
class StudentsController < ApplicationController
  before_action :set_student, only: [:show, :edit, :update]
  def searchcriteria
    render json: Student.search(params[:query], {
      fields: ["name", "roll", "grade", "fee"],
      limit: 10,
      load: false,
      misspellings: {below: 5}
    }).map(&:title)
  end
end
```

Implement JavaScript searchbox as below

```Ruby
<input type="text" id="query" name="query" />

  $("#query").typeahead({
    name: "student",
    remote: "/students/search_criteria?query=%QUERY"
  });
```

Suggestions generator

```Ruby
class Student < ApplicationRecord
  searchkick suggest: [:name, :roll, :fee, :grade]
end
```

Highlight search result fields like this:

```Ruby
class Student < ApplicationRecord
  searchkick highlight: [:name]
end
```

Create custom and advanced mapping like this:

```Ruby
class Student < ApplicationRecord
  searchkick mappings: {
    student: {
      properties: {
        name: {type: "string", analyzer: "keyword"},
        grade: {type: "string", analyzer: "keyword"}
      }
    }
  }
end
```
