---
title: "Password Validator for PHP"
layout: "post"
color: "fff"
background: "F26522"
author: "Jeremy Kendall"
author_url: "http://about.me/jeremykendall"
author_twitter: JeremyKendall
gravatar: "f8ee60f3cb8ecc91f41ee093542e12f4"
sponsor: "Graph Story"
sponsor_url: "http://graphstory.com/"
sponsor_twitter: GraphStoryCo
sponsor_image: "http://graphstory.com/img/logo.png"
---

**Password Validator** *validates* [`password_hash`][2] generated passwords, *rehashes*
passwords as necessary, and will *upgrade* legacy passwords.

*Password Validator is available for all versions of PHP >= 5.3.7.*

## Installation

The only officially supported method of installation is via
[Composer](http://getcomposer.org).

Create a `composer.json` file in the root of your project ([always check
Packagist][1] for the most recent version):

``` json
{
    "require": {
        "jeremykendall/password-validator": "*"
    }
}
```

And then run: `composer install`

Add the autoloader to your project:

``` php
<?php

require_once '../vendor/autoload.php'
```

You're now ready to begin using the Password Validator.

## Usage

### Password Validation

If you're already using [`password_hash`][2] generated passwords in your
application, you need do nothing more than add the validator in your
authentication script. The validator uses [`password_verify`][3] to test
the validity of the provided password hash.

``` php
use JeremyKendall\Password\PasswordValidator;

$validator = new PasswordValidator();
$result = $validator->isValid($_POST['password'], $hashedPassword);

if ($result->isValid()) {
    // password is valid
}
```

If your application requires options other than the `password_hash` defaults,
you can set both the `salt` and `cost` options with `PasswordValidator::setOptions()`.

``` php
$options = array(
    'salt' => 'SettingYourOwnSaltIsNotTheBestIdea',
    'cost' => 11,
);
$validator->setOptions($options);
```

**IMPORTANT**: `PasswordValidator` uses a default cost of `10`. If your
existing hash implementation requires a different cost, make sure to specify it
using `PasswordValidator::setOptions()`. If you do not do so, all of your
passwords will be rehashed using a cost of `10`.

### Rehashing

Each valid password is tested using [`password_needs_rehash`][4]. If a rehash
is necessary, the valid password is hashed using `password_hash` with the
provided options. The result code `Result::SUCCESS_PASSWORD_REHASHED` will be
returned from `Result::getCode()` and the new password hash is available via
`Result::getPassword()`.

``` php
if ($result->getCode() === Result::SUCCESS_PASSWORD_REHASHED) {
    $rehashedPassword = $result->getPassword();
    // Persist rehashed password
}
```

**IMPORTANT**: If the password has been rehashed, it's critical that you
persist the updated password hash. Otherwise, what's the point, right?

### Upgrading Legacy Passwords

You can use the `PasswordValidator` whether or not you're currently using
`password_hash` generated passwords. The validator will transparently upgrade
your current legacy hashes to the new `password_hash` generated hashes as each
user logs in.  All you need to do is provide a validator callback for your
password hash and then [decorate][6] the validator with the `UpgradeDecorator`.

``` php
use JeremyKendall\Password\Decorator\UpgradeDecorator;

// Example callback to validate a sha512 hashed password
$callback = function ($password, $passwordHash, $salt) {
    if (hash('sha512', $password . $salt) === $passwordHash) {
        return true;
    }

    return false;
};

$validator = new UpgradeDecorator(new PasswordValidator(), $callback);
$result = $validator->isValid('password', 'password-hash', 'legacy-salt');
```

The `UpgradeDecorator` will validate a user's current password using the
provided callback.  If the user's password is valid, it will be hashed with
`password_hash` and returned in the `Result` object, as above.

All password validation attempts will eventually pass through the
`PasswordValidator`. This allows a password that has already been upgraded to
be properly validated, even when using the `UpgradeDecorator`.

### Persisting Rehashed Passwords

Whenever a validation attempt returns `Result::SUCCESS_PASSWORD_REHASHED`, it's
important to persist the updated password hash.

``` php
if ($result->getCode() === Result::SUCCESS_PASSWORD_REHASHED) {
    $rehashedPassword = $result->getPassword();
    // Persist rehashed password
}
```

While you can always perform the test and then update your user database
manually, if you choose to use the **Storage Decorator** all rehashed passwords
will be automatically persisted.

The Storage Decorator takes two constructor arguments: An instance of
`PasswordValidatorInterface` and an instance of the
`JeremyKendall\Password\Storage\StorageInterface`.

#### StorageInterface

The `StorageInterface` includes a single method, `updatePassword()`. A class
honoring the interface might look like this:

``` php
<?php

namespace Example;

use JeremyKendall\Password\Storage\StorageInterface;

class UserDao implements StorageInterface
{
    public function __construct(\PDO $db)
    {
        $this->db = $db;
    }

    public function updatePassword($identity, $password)
    {
        $sql = 'UPDATE users SET password = :password WHERE username = :identity';
        $stmt = $this->db->prepare($sql);
        $stmt->execute(array('password' => $password, 'username' => $identity));
    }
}
```

#### Storage Decorator

With your `UserDao` in hand, you're ready to decorate a
`PasswordValidatorInterface`.

``` php
use Example\UserDao;
use JeremyKendall\Password\Decorator\StorageDecorator;

$storage = new UserDao($db);
$validator = new StorageDecorator(new PasswordValidator(), $storage);

// If validation results in a rehash, the new password hash will be persisted
$result = $validator->isValid('password', 'passwordHash', null, 'username');
```

**IMPORTANT**: You must pass the optional fourth argument (`$identity`) to
`isValid()` when calling `StorageDecorator::isValid()`.  If you do not do so,
the `StorageDecorator` will throw an `IdentityMissingException`.

### Validation Results

Each validation attempt returns a `JeremyKendall\Password\Result` object. The
object provides some introspection into the status of the validation process.

* `Result::isValid()` will return `true` if the attempt was successful
* `Result::getCode()` will return one of three possible `int` codes:
    * `Result::SUCCESS` if the validation attempt was successful
    * `Result::SUCCESS_PASSWORD_REHASHED` if the attempt was successful and the password was rehashed
    * `Result::FAILURE_PASSWORD_INVALID` if the attempt was unsuccessful
* `Result::getPassword()` will return the rehashed password, but only if the password was rehashed

### Database Schema Changes

As mentioned above, because this library uses the `PASSWORD_DEFAULT` algorithm,
it's important your password field be `VARCHAR(255)` to account for future
updates to the default password hashing algorithm.

## cost-check Helper Script

The default `cost` used by `password_hash` is 10.  This may or may not be
appropriate for your production hardware, and it's entirely likely you can use
a higher cost than the default. `cost-check` is based on the [finding a good
cost][8] example in the PHP documentation. Simply run `./vendor/bin/cost-check` from the command line and an appropriate cost will be returned.

**NOTE**: The default time target is 0.2 seconds.  You may choose a higher or lower
target by passing a float argument to `cost-check`, like so:

``` bash
$ ./vendor/bin/cost-check 0.4
Appropriate 'PASSWORD_DEFAULT' Cost Found:  13
```

[1]: https://packagist.org/packages/jeremykendall/password-validator
[2]: http://www.php.net/manual/en/function.password-hash.php
[3]: http://www.php.net/manual/en/function.password-verify.php
[4]: http://www.php.net/manual/en/function.password-needs-rehash.php
[6]: http://en.wikipedia.org/wiki/Decorator_pattern
[8]: http://php.net/password_hash#example-875
