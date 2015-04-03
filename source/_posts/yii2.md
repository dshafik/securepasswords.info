---
title: "Yii2 Framework"
tile: "Yii2"
layout: "post"
color: "fff"
background: "4e8acc"
author: "Jason Ragsdale"
author_twitter: Jasrags
gravatar: "c33a23d256e895d5cc6898ea3051528c"
sponsor: "Yiisoft"
sponsor_url: "http://www.yiiframework.com/"
sponsor_twitter: yiiframework
sponsor_image: "http://static.yiiframework.com/css/img/logo.png"
---

Yii2 Framework ships with support for [`crypt()`](http://php.net/crypt) and [`ext/password`](http://php.net/password) via it's security component.

## Installation
Yii2 security comes installed with a yii2 composer install, nothing special is required.

## Usage

By default Yii2 uses `crypt()` for hashing, but if you have PHP >= 5.5.0 we recommend you use `ext/password` by adding the following in your `config/web.php` file.

```php
<?php
return [
  ...
  'components' => [
    ...
    'security' => [
      'passwordHashStrategy' => 'password_hash'
    ...
    ]
  ]
];
```

For more security documentation please visit [Yii2 Security - Passwords](http://www.yiiframework.com/doc-2.0/guide-security-passwords.html)

## Hashing passwords

```php
$hash = Yii::$app->getSecurity()->generatePasswordHash($password);
```

## Verifying a password

```php
if (Yii::$app->getSecurity()->validatePassword($password, $hash)) {
  // all good, logging user in
} else {
  // wrong password
}
```
