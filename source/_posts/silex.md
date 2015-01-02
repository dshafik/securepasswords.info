---
title: "Silex"
layout: "post"
color: "FFF"
background: "4353CD"
author: "Javier Eguiluz"
author_twitter: "javiereguiluz"
gravatar: "9dc22dfc54bd9c666c72ebcd73d2fdd7"
sponsor: "SensioLabs"
sponsor_url: "https://sensiolabs.com"
sponsor_twitter: sensiolabs
sponsor_image: "/themes/dshafik/securepasswords.info/assets/images/sensiolabs.png"
---

## Installation

Silex relies on PHP hashing algorithms to encode passwords. The recommended
algorithm is called `bcrypt` and it's available in PHP 5.5+ via the
[ext/password extension] [1]. If you are using an earlier PHP version, install
the [password_compat] [2] library in your projects to enable `bcrypt` in PHP
5.3.17+ versions:

```sh
$ composer require ircmaxell/password_compat
```

## Usage

By default, Silex hashes passwords using the `sha512` algorithm. In order to 
use `bcrypt`, override the default settings of the `security.encoder.digest`
service:

```php
use Symfony\Component\Security\Core\Encoder\BCryptPasswordEncoder;

$app['security.encoder.digest'] = $app->share(function ($app) {
    // '10' is the cost used to hash passwords. Its value must be between 4 and 31
    return new BCryptPasswordEncoder(10);
});
```

### Hashing a Password

Use the `encodePassword()` method of the encoder related to the user:

```php
$encoder = $app['security.encoder_factory']->getEncoder($user);
$hashedPassword = $encoder->encodePassword($plainPassword, $user->getSalt());
```

`$user` is the object that represents the user and it must implement the
`Symfony\Component\Security\Core\User\UserInterface` class.

### Verifying a Password

Use the `isPasswordValid()` method of the encoder related to the user:

```php
$encoder = $app['security.encoder_factory']->getEncoder($user);
$isValid = $encoder->isPasswordValid($user->getPassword(), $plainPassword, $user->getSalt());
```

`$user` is the object that represents the user and it must implement the
`Symfony\Component\Security\Core\User\UserInterface` class.

## Resources

* [Silex SecurityServiceProvider](http://silex.sensiolabs.org/doc/providers/security.html)

[1]: http://php.net/password
[2]: https://github.com/ircmaxell/password_compat
