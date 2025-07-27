---
layout: post
title: "Design Patterns in Rails: A Practical Guide"
date: 2025-05-28 18:38:00 +0545
categories: [Ruby on Rails, design-patterns, architecture]
tags: [ruby on rails, design-patterns, architecture]
---

Ruby on Rails (Rails) is known for its convention-over-configuration philosophy and rapid development capabilities. But as applications grow in complexity, simply following Rails conventions isn’t always enough. That’s where design patterns come into play.

In this post, we’ll explore how common design patterns are used in Rails applications to keep code maintainable, readable, and scalable.

---

## Why Design Patterns Matter in Rails

Design patterns are proven solutions to recurring software design problems. They provide a shared vocabulary for developers and help manage complexity.

While Rails promotes certain patterns out of the box (like MVC), seasoned developers often go further, adopting patterns like Service Objects, Decorators, and Form Objects to maintain clean architecture.

---

## 1. Model-View-Controller (MVC)

**Pattern Type**: Architectural  
**Purpose**: Separates data, user interface, and control logic.

Rails is built on MVC:

- **Model**: Handles business logic and database interactions.
- **View**: Renders the HTML (or other formats).
- **Controller**: Coordinates between model and view.

### Tip:

Keep your controllers skinny and models lean by pushing complex logic into service objects or concerns.

---

## 2. Service Objects

**Pattern Type**: Behavioral  
**Purpose**: Encapsulate business logic that doesn’t naturally fit into models or controllers.

```ruby
class ProcessPayment
  def initialize(order)
    @order = order
  end

  def call
    charge_customer
    send_receipt
  end

  private

  def charge_customer
    # Payment logic
  end

  def send_receipt
    # Email logic
  end
end
```

Use it in your controller:

```ruby
ProcessPayment.new(@order).call
```

---

## 3. Decorator Pattern

**Pattern Type**: Structural  
**Purpose**: Add responsibilities to objects without modifying their structure.

In Rails, you might use the [Draper](https://github.com/drapergem/draper) gem to create decorators for models.

```ruby
class OrderDecorator < Draper::Decorator
  def formatted_total
    h.number_to_currency(object.total)
  end
end
```

---

## 4. Presenter/ViewModel

**Pattern Type**: Structural  
**Purpose**: Encapsulate view-specific logic, often used in place of helpers or decorators.

```ruby
class DashboardPresenter
  def initialize(user)
    @user = user
  end

  def recent_orders
    @user.orders.recent.limit(5)
  end
end
```

---

## 5. Form Objects

**Pattern Type**: Structural  
**Purpose**: Manage complex forms that interact with multiple models or validations.

```ruby
class SignupForm
  include ActiveModel::Model

  attr_accessor :user, :account_name, :email, :password

  validates :email, :password, presence: true

  def save
    return false unless valid?
    create_user_and_account
  end

  private

  def create_user_and_account
    # handle multi-model logic
  end
end
```

---

## 6. Policy Objects (Pundit/Cancancan)

**Pattern Type**: Behavioral  
**Purpose**: Manage authorization logic.

```ruby
class PostPolicy < ApplicationPolicy
  def update?
    user.admin? || record.author == user
  end
end
```

---

## 7. Query Objects

**Pattern Type**: Behavioral  
**Purpose**: Encapsulate complex ActiveRecord queries.

```ruby
class RecentOrdersQuery
  def initialize(user)
    @user = user
  end

  def call
    @user.orders.where("created_at > ?", 1.week.ago)
  end
end
```

---

## When to Use Which Pattern?

| Pattern        | Use When                                                 |
| -------------- | -------------------------------------------------------- |
| Service Object | Logic doesn't belong in model or controller              |
| Decorator      | You need to format or enhance model output for the view  |
| Form Object    | Your form touches multiple models or complex validations |
| Query Object   | Queries get too long to be readable or reusable          |
| Policy Object  | Managing user permissions and access control             |

---

## Final Thoughts

Design patterns aren’t a silver bullet—but they are powerful tools. In Rails, the key is to use them _only when they provide clear benefits_. Start simple, and refactor when complexity grows.

{% include inarticle-adsense.html %}