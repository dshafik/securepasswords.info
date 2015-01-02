---
title: "Symfony"
layout: "post"
color: "FFF"
background: "222"
author: "Javier Eguiluz"
author_twitter: "javiereguiluz"
gravatar: "9dc22dfc54bd9c666c72ebcd73d2fdd7"
sponsor: "SensioLabs"
sponsor_url: "https://sensiolabs.com"
sponsor_twitter: sensiolabs
sponsor_image: "/themes/dshafik/securepasswords.info/assets/images/sensiolabs.png"
---

## Installation

Symfony relies on PHP hashing algorithms to encode passwords. The recommended
algorithm is called `bcrypt` and it's available in PHP 5.5+ via the
[ext/password extension] [1]. If you are using an earlier PHP version, install
the [password_compat] [2] library in your  projects to enable `bcrypt` in PHP
5.3.17+ versions:

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

In **Symfony 2.6 or newer** versions, use the `security.password_encoder`
service:

```php
$hashedPassword = $this->container->get('security.password_encoder')
    ->encodePassword($user, $plainPassword);
```

`$user` is the object that represents the user and it must implement the
`Symfony\Component\Security\Core\User\UserInterface` interface.

In **previous Symfony versions**, use the `security.encoder_factory` service:

```php
$encoder = $this->container->get('security.encoder_factory')->getEncoder($user);
$hashedPassword = $encoder->encodePassword($plainPassword, $user->getSalt());
```

### Verifying a Password

In **Symfony 2.6 or newer** versions, use the `security.password_encoder` service:

```php
$isValid = $this->container->get('security.password_encoder')
    ->isPasswordValid($user, $plainPassword);
```

`$user` is the object that represents the user and it must implement the
`Symfony\Component\Security\Core\User\UserInterface` interface.

In **previous Symfony versions**, use the `security.encoder_factory` service:

```php
$encoder = $this->container->get('security.encoder_factory')->getEncoder($user);
$isValid = $encoder->isPasswordValid($user->getPassword(), $plainPassword, $user->getSalt());
```

## Resources

* [Security Chapter](http://symfony.com/doc/current/book/security.html) of the
  official Symfony Book.
* [Symfony Security tutorials](http://symfony.com/doc/current/cookbook/security/index.html)

[1]: http://php.net/password
[2]: https://github.com/ircmaxell/password_compat
