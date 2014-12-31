---
title: "Yii2 Framework"
tile: "Yii2"
layout: "post"
color: "fff"
background: "59a93a"
author: "Jason Ragsdale"
author_url: "http://ocramius.github.io"
author_twitter: Jasrags
gravatar: "c33a23d256e895d5cc6898ea3051528c"
sponsor: "Yiisoft"
sponsor_url: "http://www.yiiframework.com/"
sponsor_twitter: yiiframework
sponsor_image: "http://static.yiiframework.com/css/img/logo.png"
---

Yii2 Framework ships with support for `bcrypt` and `password_hash` support via it's security component.

## Installation
Yii2 security comes installed with a yii2 composer install, nothing special is required.

## Usage
By default Yii2 uses bcrypt for hashing, but if you have php >= 5.5.0 you can use password_hash the following in your config file.

```php
$config = [
  'components' => [
    'security' => [
      'passwordHashStrategy' => 'password_hash'
    ]
  ]
];
```

For more security documentation please visit [Yii2 Security - Passwords](http://stuff.cebe.cc/yii2docs/guide-security-passwords.html)

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
