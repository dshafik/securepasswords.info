---
title: "Laravel"
tile: "Laravel"
layout: "post"
color: "fff"
background: "f4645f"
author: "nickurt"
author_url: "http://github.com/nickurt"
---

Laravel is coming out of the box with a very simple authentication. The authentication config file is located at `app/config/auth.php` (v4) or `config/auth.php` (v5) and by default it ships with a `User` model in `app/models` (v4) or `app` (v5) which can be used with the Eloquent authentication driver.

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
It is also possible to add extra conditions 
```php
if(Auth::attempt(['username' => $username, 'password' => $password, 'active' => 1]))
{
	// Works
}
```
### Hash strength increasing
To increase the hash strength of the Bcrypt hash, you can change the `rounds` value
```php
$password = Hash::make('secret', ['rounds' => 12]);
```
### Protecting Routes
```php
// Only authenticated users (v5 and v4)
Route::get('/profile', ['middleware' => 'auth', 'uses' => 'ProfileController@getIndex']);
Route::get('/profile', ['uses' => 'ProfileController@getIndex'])->before('auth');

// All users
Route::get('/profile', ['uses' => 'ProfileController@getIndex']);
```