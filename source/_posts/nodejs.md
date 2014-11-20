---
title: "Node.js"
layout: "post"
color: "fff"
background: "6fb424"
author: "Davey Shafik"
author_url: "http://daveyshafik.com"
author_twitter: "dshafik"
gravatar: "fee39f0c0ffb29d9ac21607ed188be6b"
sponsor: "Engine Yard"
sponsor_url: "https://www.engineyard.com"
sponsor_twitter: EngineYard
---

Node.js has multiple packages available that provide bcrypt, the most popular is a wrapper around the OpenBSD C library.

## Installation

Installation is done with npm:

```sh
$ npm install bcrypt
```

## Usage

The package allows for both asynchronous (recommended) and synchronous usage.

### Asynchronous Usage
	
#### Hashing a Password
	
To hash a password asynchronously, we call `bcrypt.hash()`:

```js
var bcrypt = require('bcrypt');

bcrypt.hash('password', 10, function(err, hash) {
		// Store the hash
});
```

The first argument is the password, the second is the `cost`, and the last is a callback into which two arguments are passed:

- `err` — Details any errors
- `hash` — The resulting hash

This will automatically generate a salt, and use a cost of 10.

You can generate a salt manually using `bcrypt.genSalt()`:

```js
var bcrypt = require('bcrypt');

bcrypt.genSalt(10, function(err, salt) {
    bcrypt.hash("password", salt, function(err, hash) {
        // Store the hash
    });
});
```

Here we create a salt with a `cost` of `10`, and call the `bcrypt.hash()` function inside the `bcrypt.getSalt()` callback.

#### Verifying a Password

To verify a password asynchronously, we call `bcrypt.compare()`:

```js
bcrypt.compare("password", hash, function(err, valid) {
    if (valid == true) {
		// password matches
	} else if (valid == false) {
		// password does not match
	}
});
```

We pass in the user supplied password, and the stored hash, as well as a callback that will recieve two arguments:

- `err` — Any errors
- `valid` — Whether the password was valid or not

### Synchronous Usage

#### Hashing a Password

To hash a password synchronously, do the following:

```js
var bcrypt = require('bcrypt');

var hash = bcrypt.hashSync("password", 10);
```

Here we pass in the password, and a `cost` of `10`. The salt is generated automatically. The resulting hash is returned directly.

We can also generate the salt manually, supplying the `cost` to the `bcrypt.genSaltSync()` function, and then passing the resulting `salt` to
`bcrypt.hashSync()` in place of the `cost`.

```js
var bcrypt = require('bcrypt');

var salt = bcrypt.genSaltSync(10);
var hash = bcrypt.hashSync("password", salt);
```

#### Verifying a Password

To verify a password synchronously we simply pass the password, and stored hash to `bcrypt.compareSync()`. A boolean is returned:

```js
var valid = bcrypt.compareSync("password", hash);
if (valid == true) {
	// password matches
} else if (valid == false) {
	// password does not match
}
```