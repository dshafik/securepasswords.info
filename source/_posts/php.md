---
title: "PHP"
layout: "post"
color: "fff"
background: "757eb1"
author: "Davey Shafik"
author_url: "http://daveyshafik.com"
author_twitter: "dshafik"
gravatar: "fee39f0c0ffb29d9ac21607ed188be6b"
sponsor: "Engine Yard"
sponsor_url: "https://www.engineyard.com"
sponsor_twitter: EngineYard
---

With PHP 5.5, a new [password extension](http://php.net/password) was added to that makes password security easy by default.

## Installation

The password extension is enabled by default in PHP 5.5+. If you are not yet using PHP 5.5, you can use the [password_compat](https://github.com/ircmaxell/password_compat) library, as far back as PHP 5.3.17.

To install password_compat, you can use composer:

```sh
$ composer require ircmaxell/password_compat
```

## Usage

The implementation consists of four functions:

- [`password_hash()`](http://php.net/password_hash)
	Create a new password hash
- [`password_verify()`](http://php.net/password_verify)
	Confirm a given password matches the hash
- [`password_needs_rehash()`](http://php.net/password_needs_rehash)
	Check if a password meets the desired hash settings (algorithm, cost)
- [`password_get_info()`](http://php.net/password_get_info)
	Returns information about the hash such as algorithm and cost
	
### Hashing a Password
	
To hash a password, do the following:

```php
<?php
$password = password_hash($_POST['password'], PASSWORD_DEFAULT, ['cost' => 11]);
?>
```

You may notice the second argument of `PASSWORD_DEFAULT`. This will automatically select the best algorithm at the time, as decided by PHP. Currently it is bcrypt, but this option gives you automatic forward compatible security, should bcrypt be compromised or become ineffectual in some way.

Additionally, we supply a third argument with the `cost`. The cost is what determines[a] how long the password takes to generate. It defaults to 10, however this is just a general guideline and your hardware will determine what makes more sense. The difference between an Amazon EC2 m3.medium and a c3.8xlarge is obviously going to have a significant impact on how long it takes to generate a hash.

### Verifying a Password

To verify a password, we do:

```php
<?php
$user = User::findByUsername($username);
if (password_verify($_POST['password'], $user->password)) {
     // Password matches
} else {
     // Password does not match
}
?>
```

### Keeping Your Users Secure

The last part of the puzzle is ensuring that passwords are kept up to date. We can do this in two ways. One by using `password_get_info()` to check the algorithm and cost against our current settings. Or two, by using the built-in function `password_needs_rehash()` to check the hash against the settings automatically. By checking this on login, we can automatically rehash and store their password using the latest settings as supplied by `PASSWORD_DEFAULT`, or a change in the cost option:

```php
<?php
$user = User::findByUsername($username);
if (password_verify($_POST['password'], $user->password)) {
        if (password_needs_rehash($user->password, PASSWORD_DEFAULT, ['cost' => 12])) {
        	$user->updatePassword($_POST['password']);
    	}
} else {
     // Password does not match
}
?>
```

The `password_needs_rehash()` function will return `true` for MD5 and SHA-1 hashes also. This means that you can check prior to `password_verify()` and use your legacy mechanism for verification instead. Once verified, you can then update the password using `password_hash()`. In this way you can roll out security updates for your users whilst avoiding mass password resets that would inconvenience your users. Though, if you are using (unsalted, especially) MD5 or SHA-1, it’s probably a good idea to do a mass reset anyway.

