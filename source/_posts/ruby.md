---
title: "Ruby"
layout: "post"
color: "fff"
background: "be2322"
author: "Davey Shafik"
author_url: "http://daveyshafik.com"
author_twitter: "dshafik"
gravatar: "fee39f0c0ffb29d9ac21607ed188be6b"
---
For Ruby, we use the [bcrypt gem](https://rubygems.org/gems/bcrypt) which allows us to hash and validate passwords.

## Installation

```sh
$ gem install bcrypt
```

Or, using bundler, add following to your `Gemfile`:

```ruby
gem 'bcrypt', '~> 3.1.7'
```

Then install:

```sh
$ bundle install
```

## Usage

The bcrypt gem is super simple, with a single method for generating hashes.

### Hashing a Password

```ruby
require ‘bcrypt’

hashed_password = BCrypt::Password.create(password, :cost => 11)
```

You can pass in an options hash including an option to set the `cost`.

### Verifying a Password

For validation, you can use the comparison operator:

```ruby
if hashed_password == password then
     # Password matches
else
     # Password does not match
end
```
