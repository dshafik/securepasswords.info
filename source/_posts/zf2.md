---
title: ZF 2
layout: post
color: fff
background: 59a93a 
---

Zend Framework 2 ships with the `Zend\Crypt` component, which can be used to generate secure password hashes.

## Installation

To use it, first install `Zend\Crypt` via [composer](https://getcomposer.org/):

```sh
composer require "zendframework/zend-crypt:2.*"
```

## Usage

Whenever you want to generate a new password, just create a new `Bcrypt` instance as following:

```php
use Zend\Crypt\Password\Bcrypt;

require_once __DIR__ . '/vendor/autoload.php';

$bcrypt = new Bcrypt();

$secureHash = $bcrypt->create($clearTextPassword);
```

### Verifying a password

Verifying a password is also done via the `Bcrypt` instance:

```php
if ($bcrypt->verify($clearTextPassword, $secureHash)) {
    echo "Password matches!";
} else {
    echo "Password does NOT match!";
}
```

### Authenticating a user against a database

Please note that you should always authenticate your users against a database by first fetching the identity by
identifier or username, and **only then** verifying the user against the given password:

```php
$user = $usersGateway->select(['username' => $inputUsername]);

if ($user && $bcrypt->verify($inputPassword, $user->getPasswordHash())) {
    // valid login, may store user identity into session here
}
```

### Integration with `Zend\Mvc`

Authentication and security are hard: if you use the full-stack ZF2 framework, then you can just plug
the [`ZfcUser`](https://github.com/ZF-Commons/ZfcUser) module into your applications.
Using ZfcUser instead of your own homebrew authentication system will give you the advantage of always
having an the latest known security vulnerabilities fixed for you by experienced community members.
