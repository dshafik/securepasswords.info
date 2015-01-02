---
title: "Password Validator for Symfony"
layout: "post"
color: "FFF"
background: "222"
author: "Javier Eguiluz"
author_twitter: "javiereguiluz"
gravatar: "9dc22dfc54bd9c666c72ebcd73d2fdd7"
sponsor: "SensioLabs"
sponsor_url: "https://sensiolabs.com"
sponsor_twitter: sensiolabs
sponsor_image: "http://securepasswords.info/themes/dshafik/securepasswords.info/assets/images/engineyard.png"
---

## Installation

Symfony relies on PHP hashing algorithms to encode passwords. The recommended
algorithm is called `bcrypt` and it's available in PHP 5.5+. If you are using
an earlier PHP version, install the `password_compat` library in your projects 
to enable `bcrypt` in PHP 5.3.17+ versions:

```sh
$ composer require ircmaxell/password_compat
```

## Usage

Before hashing and verifying passwords, define the algorithm to use in the
`app/config/security.yml` file:

```php
# app/config/security.yml
security:
    # ...

    encoders:
        Symfony\Component\Security\Core\User\User: bcrypt
```

In case you want to define the `cost` used to hash the password, define the
following configuration:

```php
# app/config/security.yml
security:
    # ...

    encoders:
        Symfony\Component\Security\Core\User\User:
            algorithm: bcrypt
            cost: 12
```

### Hashing a Password

Make use of the `security.password_encoder` service:

```php
$encoded = $this->container->get('security.password_encoder')
    ->encodePassword($user, $plainPassword);
```

`$user` is the object that represents the user and it must implement the
`Symfony\Component\Security\Core\User\UserInterface` interface.

### Verifying a Password

Make use of the `security.password_encoder` service:

```php
$isValid = $this->container->get('security.password_encoder')
    ->isPasswordValid($user, $plainPassword);
```

`$user` is the object that represents the user and it must implement the
`Symfony\Component\Security\Core\User\UserInterface` interface.

## Resources

* [Security Chapter](http://symfony.com/doc/current/book/security.html) of the
  official Symfony Book.
* [Recipes about Symfony Security](http://symfony.com/doc/current/cookbook/security/index.html)
