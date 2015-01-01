---
title: "Authentication library Aura.Auth"
layout: "post"
color: "fff"
background: "59a93a"
author: "Hari KT"
author_url: "http://harikt.com"
author_twitter: harikt
gravatar: "895c943fbd5beb697f6c2d7bf0c3b279"
sponsor: "Dflydev"
sponsor_url: "http://dflydev.com"
sponsor_twitter: dflydev
sponsor_image: "https://avatars0.githubusercontent.com/u/199259?v=3&s=200"
---

Aura.Auth provides authentication functionality and session tracking using various adapters. Currently supported are:

* Apache htpasswd files
* SQL tables via the PDO extension
* IMAP/POP/NNTP via the imap extension
* LDAP and Active Directory via the ldap extension
* OAuth via customized adapters

It makes use of built in PHP functions in PHP 5.5 or with the help of  [ircmaxell/password-compat](https://packagist.org/packages/ircmaxell/password-compat) as mentioned in [article](http://securepasswords.info/php/).

## Installation

You can either clone the repo `https://github.com/auraphp/Aura.Auth` and include the `autoload.php` file or install via [composer](https://getcomposer.org/) as below.

```sh
composer require "aura/auth:2.0.0-beta2"
```

## Usage

In this example we are looking into authentication via database using pdo. The `Aura\Auth\Verifier\PasswordVerifier` class help you to make use of different type of hashing algorithms in PHP. You can pass `PASSWORD_DEFAULT` to make use of  [password_](http://php.net/password) functions or `md5`, `sha256` etc. It is recommended to make use of `PASSWORD_DEFAULT`.

```php
<?php
require_once __DIR__ . '/vendor/autoload.php';

$auth_factory = new \Aura\Auth\AuthFactory($_COOKIE);
$auth = $auth_factory->newInstance();

$pdo = new \PDO(...);
$cols = array(
    'username', // "AS username" is added by the adapter
    'password', // "AS password" is added by the adapter
    'email',
    'fullname',
    'website'
);
$from = 'users';
$where = 'active = 1';

$hash = new \Aura\Auth\Verifier\PasswordVerifier(PASSWORD_DEFAULT);

$pdo_adapter = $auth_factory->newPdoAdapter($pdo, $hash, $cols, $from, $where);
```

> Assuming you have a database table as below

```sql
CREATE TABLE `users` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `username` varchar(255) NOT NULL COMMENT 'Username',
  `email` varchar(255) NOT NULL COMMENT 'Email',
  `password` varchar(255) NOT NULL COMMENT 'Password',
  `fullname` varchar(255) NOT NULL COMMENT 'Full name',
  `website` varchar(255) DEFAULT NULL COMMENT 'Website',
  `active` int(11) NOT NULL COMMENT '0',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

See more complex example using joins in [readme](https://github.com/auraphp/Aura.Auth)

## Login service

Assuming your database `password` field being hashed with the corresponding algorithm passed to `PasswordVerifier`, the login service will verify and throw exceptions according to the error happened.

```php
$login_service = $auth_factory->newLoginService($pdo_adapter);
try {
    $login_service->login($auth, array(
            'username' => $_POST['username'],
            'password' => $_POST['password'],
        )
    );
    echo "You are now logged into a new session.";
} catch (\Aura\Auth\Exception\UsernameMissing $e) {
    echo "The 'username' field is missing or empty.";
} catch (\Aura\Auth\Exception\PasswordMissing $e) {
    echo "The 'password' field is missing or empty.";
} catch (\Aura\Auth\Exception\UsernameNotFound $e) {
    echo "The username you entered was not found.";
} catch (\Aura\Auth\Exception\MultipleMatches $e) {
    echo "There is more than one account with that username.";
} catch (\Aura\Auth\Exception\PasswordIncorrect $e) {
    echo "The password you entered was incorrect.";
} catch (\Aura\Auth\Exception\ConnectionFailed $e) {
    echo "Cound not connect to IMAP or LDAP server.";
    echo "This could be because the username or password was wrong,";
    echo "or because the the connect operation itself failed in some way. ";
    echo $e->getMessage();
} catch (\Aura\Auth\Exception\BindFailed $e) {
    echo "Cound not bind to LDAP server.";
    echo "This could be because the username or password was wrong,";
    echo "or because the the bind operations itself failed in some way. ";
    echo $e->getMessage();
}
```

## Resume service

Like PHP, Aura.Auth doesnot start the session automatically, [read why here](https://github.com/auraphp/Aura.Auth#resuming-a-session). So `$_SESSION` will not be populated. If you need to check whether the user is logged in on the next request, you should either start the session via `session_start` or call the resume service first and then only check the `Auth` methods.

```php
// start session
session_start();

// or use the service to resume any previously-existing session

// $resume_service = $auth_factory->newResumeService($pdo_adapter);
// $resume_service->resume($auth);

echo $auth->getStatus();
```

## Logout service

Same applies to logout, for session is not started you should either call `session_start` or resume service before you try logout. Else session data will not be removed.

```php
session_start();
$logout_service = $auth_factory->newLogoutService($pdo_adapter);
$logout_service->logout($auth);

if ($auth->isAnon()) {
    echo "You are now logged out.";
} else {
    echo "Something went wrong; you are still logged in.";
}
```

Depending upon the adapter methods, you can swap the adapters for convenience. Eg : `Aura\Auth\Adapter\PdoAdapter::logout` method does nothing, so you can pass a `Aura\Auth\Adapter\NullAdapter`. But it is not recommended.

Checkout the full example code of the tutorial over  [https://github.com/harikt/authentication-pdo-example](https://github.com/harikt/authentication-pdo-example)
