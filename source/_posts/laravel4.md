---
title: "Laravel 4"
tile: "Laravel 4"
layout: "post"
color: "fff"
background: "f4645f"
author: "nickurt"
author_url: "http://github.com/nickurt"
---

Laravel 4 is coming out of the box with a very simple authentication. The authentication config file is located at `app/config/auth.php` and by default it ships with a `User` model (`app/models`) which can be used with the Eloquent authentication driver.

## Usage

When you want to generate a new password, you can use the Laravel `Hash` class.

```php
$password = Hash::make('secret');
```

### Password verifying

```php
if (Hash::check('secret', $hashedPassword))
{
	// Matched!
}
```
### Password rehashing
```php
if (Hash::needsRehash($hashed))
{
	$hashed = Hash::make('secret');
}
```
### Authenticating Users

You can log a user into your application using the `Auth::attempt` method.

```php
if(Auth::attempt(['username' => $username, 'password' => $password]))
{
	// Works
}
else
{
	// Don't work
}
```

### Hash strength increasing
To increase the hash strength of the Bcrypt hash, you can change the `rounds` value
```php
$password = Hash::make('secret', array('rounds' => 12));
```