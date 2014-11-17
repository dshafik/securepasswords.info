---
title: Ruby on Rails
layout: post
color: fff
background: 9a1d26
---

Automatic support for bcrypt has been available in Rails since 3.0, thanks to `ActiveModel::SecurePassword`. 

## Installation

To use it, first add bcrypt to your `Gemfile`:

```ruby
gem 'bcrypt', '~> 3.1.7'
```

## Usage

Then add a `password_digest` attribute, and call `has_secure_password`:

```ruby
# Schema: User(name:string, password_digest:string)
class User < ActiveRecord::Base
  has_secure_password
end
```

### Hashing a Password

At this point, you can then pass in either a password and `password_confirmation` arguments, or just the `password` argument alongside setting the validations argument to `false`:

```ruby
user = User.new(name: 'Davey', password: 'password', password_confirmation: 'password')
user.save
```

Or without confirmation:

```ruby
user = User.new(name: 'Davey', password: 'password', validations: false)
user.save
```

### Verifying a Password

Then to authenticate, you can either call the authenticate method:

```ruby
user.authenticate('password')
```

Or you can do it when retrieving the user from the database:

```ruby
User.find_by(name: 'Davey').try(:authenticate, 'password')
```

This method will only return a valid user object if the password is correct, otherwise it will return `false`.

