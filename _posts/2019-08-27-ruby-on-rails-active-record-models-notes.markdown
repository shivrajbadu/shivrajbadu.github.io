---
layout: post
title:  "Features and functions of Active Record Model"
date:   2019-08-27 07:12:58 +0545
categories: [Ruby on Rails, Active Record cheatsheet]
tags: [ruby on rails, active_record]
---

## Notes on various features and functions of Active Record Model

#### Query Methods

```Ruby
obj = User
  .where(email: 'info@mydomain.np')
  .where('id = 2')
  .where('id = ?', 2)
  .order(:tag_line)
  .order(tag_line: :desc)
  .order("tag_line DESC")
  .reorder(:tag_line) # Replaces any existing order defined on the relation with the specified order.
  .where(active: true)
  .rewhere(active: false) # Allows to change a previously set where condition for a given attribute, instead of appending to that condition.
  .offset(1)
  .limit(2)
  .uniq
```

#### Some other query methods

```Ruby
items = Employer
  .select(:id)
  .select([:id, :name])
  .group(:title)   # GROUP BY name
  .group('title AS grouped_title, age')
  .having('SUM(salary) > 25000')  # needs to be chained with .group
  .includes(:user)
  .includes(user: [:articles])
  .references(:comments)
```

#### Finder methods

```Ruby
item = ModelName.find(id)
item = ModelName.find_by_email(email)
item = ModelName.where(email: email).first
Model
  .first
  .last
  .exists?(5)
  .exists?(name: "ShivRaj")
  .find_nth(4, [offset])
```

#### Persistence

```Ruby
item.new_record?
item.persisted?
item.destroyed?

item.serialize_hash # Returns a serialized hash of your object
item.save
item.save!      # It does same as save, but raises an Exception
item.update  name: 'ShivRaj'  # Save the record immediately
item.update! name: 'ShivRaj'
item.update_column  :name, 'ShivRaj'  # It skips validations and callbacks
item.update_columns  name: 'ShivRaj'
item.update_columns! name: 'ShivRaj'
item.touch                 # It updates :updated_at
item.touch :published_at
item.destroy
item.delete  # It skips callbacks
Model.create     # It does same task which is done by new and save
Model.create!    # It does same as create but raises an Exception
```

#### Attribute Assignment

```Ruby
item.attributes                         # <Hash>
item.attributes = { name: 'ShivRaj' }   # Merges attributes in but it Doesn't save.
item.assign_attributes name: 'ShivRaj'  # Merges attributes in but it Doesn't save.
```

#### Validations

```Ruby
item.valid?
item.invalid?
```

#### Dirty

```Ruby
item.changed?
item.changed             # ['name']
item.changed_attributes  # { 'name' => 'ShivRaj' } - original values
item.changes             # { 'name' => ['ShivRaj', 'PushpaRaj'] }
item.previous_changes    # available after #save
item.restore_attributes
item.name = 'ShivRaj'
item.name_was         # 'ShivRaj'
item.name_change      # [ 'ShivRaj', 'PushpaRaj' ]
item.name_changed?    # true
item.name_changed?(from: 'ShivRaj', to: 'PushpaRaj')
```

#### Calculations

```Ruby
Person.count
Person.count(:age)    # counts non-nil's
Person.average(:age)
Person.maximum(:age)
Person.minimum(:age)
Person.sum('2 * age')
Person.calculate(:count, :all)
Person.distinct.count
Person.group(:city).count
```

#### Dynamic attribute-based finders

```Ruby
# Returns one record
Person.find_by_name(name)
Person.find_last_by_name(name)
Person.find_or_create_by_name(name)
Person.find_or_initialize_by_name(name)
# Returns a list of records
Person.find_all_by_name(name)
# Add a bang to make it raise an exception
Person.find_by_name!(name)
# You may use `scoped` instead of `find`
Person.scoped_by_user_name
```

#### belongs to Association

```Ruby
  belongs_to :author,
  :dependent      => :destroy    # or :delete
  :class_name     => "Seller"
  :select         => "*"
  :counter_cache  => true
  :counter_cache  => :custom_counter
  :include        => "Product"
  :readonly       => true

  :conditions     => 'published = true'

  :touch          => true
  :touch          => :sellers_last_updated_at

  :primary_key    => "name"
  :foreign_key    => "author_name"
```

#### Has many Association

```Ruby
belongs_to :parent, :foreign_key => 'parent_id' class_name: 'Folder'
has_many :folders, :foreign_key => 'parent_id', class_name: 'Folder'

has_many :comments,    :order      => "posted_on"
has_many :comments,    :include    => :author
has_many :people,      :class_name => "Person"
has_many :people,      :conditions => "deleted = 0"
has_many :tracks,      :order      => "position"
has_many :comments,    :dependent  => :nullify
has_many :comments,    :dependent  => :destroy
has_many :tags,        :as         => :taggable
has_many :reports,     :readonly   => true
has_many :subscribers, :through    => :subscriptions, class_name: "User", :source => :user
has_many :subscribers, :finder_sql =>
    'SELECT DISTINCT people.* ' +
    'FROM people p, post_subscriptions ps ' +
    'WHERE ps.post_id = #{id} AND ps.person_id = p.id ' +
    'ORDER BY p.first_name'
```

#### Many-to-many Has many through Association

```Ruby
 If you have a join model:

class Programmer < ActiveRecord::Base
  has_many :assignments
  has_many :projects, :through => :assignments
end
 
 
class Project < ActiveRecord::Base
  has_many :assignments
  has_many :programmers, :through => :assignments
end
 
 
class Assignment
  belongs_to :project
  belongs_to :programmer
end
```

#### Many-to-many (HABTM) Association

```Ruby
has_and_belongs_to_many :projects
has_and_belongs_to_many :projects, :include => [ :milestones, :manager ]
has_and_belongs_to_many :nations, :class_name => "Country"
has_and_belongs_to_many :categories, :join_table => "prods_cats"
has_and_belongs_to_many :categories, :readonly => true
has_and_belongs_to_many :active_projects, :join_table => 'developers_projects', :delete_sql =>
"DELETE FROM developers_projects WHERE active=1 AND developer_id = #{id} AND project_id = #{record.id}"

```

#### Polymorphic associations

```Ruby
class Post
  has_many :attachments, as: :parent
end
 
class Image
  belongs_to :parent, polymorphic: true
end
 
And in migrations:

create_table :images do |t|
  t.references :post, polymorphic: true
end
```

#### Validation

```Ruby
class Person < ActiveRecord::Base
  # Presence
  validates :name,     presence: true
 
  # Acceptance
  validates :terms,    acceptance: true
  # Confirm
  validates :email,    confirmation: true
  # Unique
  validates :slug,     uniqueness: true
  validates :slug,     uniqueness: { case_sensitive: false }
  validates :holiday,  uniqueness: { scope: :year, message: 'yearly only' }
  # Format
  validates :code,     format: /regex/
  validates :code,     format: { with: /regex/ }
  # Length
  validates :name,     length: { minimum: 2 }
  validates :bio,      length: { maximum: 500 }
  validates :password, length: { in: => 6..20 }
  validates :number,   length: { is: => 6 }
  # Include/exclude
  validates :gender,   inclusion: %w(male female)
  validates :gender,   inclusion: { in: %w(male female) }
  validates :lol,      exclusion: %w(xyz)
  # Numeric
  validates :points,   numericality: true
  validates :played,   numericality: { only_integer: true }
  # ... greater_than, greater_than_or_equal_to,
  # ... less_than, less_than_or_equal_to
  # ... odd, even, equal_to
  # Validate the associated records to ensure they're valid as well
  has_many :books
  validates_associated :books
  # Length (full options)
  validates :content, length: {
    minimum:   300,
    maximum:   400,
    tokenizer: lambda { |str| str.scan(/\w+/) },
    too_short: "must have at least %{count} words",
    too_long:  "must have at most %{count} words" }
  # Multiple
  validates :login, :email, presence: true
  # Conditional
  validates :description, presence: true, if: :published?
  validates :description, presence: true, if: lambda { |obj| .. }
  validates :title, presence: true, on: :save   # :save | :create | :update
end
```

#### Custom validations

```Ruby
class Person < ActiveRecord::Base
  validate :foo_cannot_be_nil

  def foo_cannot_be_nil
    errors.add(:foo, 'cannot be nil')  if foo.nil?
  end
end
```

#### Errors

```Ruby
record.errors.valid?      # → false
record.errors             # → { :name => ["can't be blank"] }
record.errors.messages    # → { :name => ["can't be blank"] }
record.errors[:name].any?
```

#### Mass updates

```Ruby
# Updates article having id 8
Article.update 8, name: "", age: 34
Article.update [2,3], [{name: "Shiv"}, {name: "Raj"}]
```

#### Joining

```Ruby
# Basic joins
Employer.joins(:companies).where(companies: { type: 'private' })
Employer.joins(:companies).where('companies.type' => 'private' )
# Multiple associations
Blog.joins(:category, :comments)
# Nested associations
Blog.joins(comments: :guest)
# SQL
Author.joins(
  'INNER JOIN posts ' +
  'ON posts.author_id = authors.id ' +
  'AND posts.published = "t"'
)
```

#### Where interpolation

```Ruby
where('name = ?', 'Shiv')
where(['name = :name', { name: 'Shiv' }])
```

#### Serialize

````Ruby
class User < ActiveRecord::Base
  serialize :preferences
end
 
user = User.create(
  preferences: {
    'background' => 'black',
    'display' => 'large'
  }
)
````

#### You can also specify a class option as the second parameter that’ll raise an exception if a serialized object is retrieved as a descendant of a class not in the hierarchy.

```Ruby
# Only Hash allowed!
class User < ActiveRecord::Base
  serialize :preferences, Hash
end
 
# Reading it raises SerializationTypeMismatch
user = User.create(preferences: %w(one two three))
User.find(user.id).preferences
```

#### Overriding accessors

```Ruby
class Song < ActiveRecord::Base
  # Uses an integer of seconds to hold the length of the song

  def length=(minutes)
    write_attribute(:length, minutes.to_i * 60)
  end

  def length
    read_attribute(:length) / 60
  end
end
```
