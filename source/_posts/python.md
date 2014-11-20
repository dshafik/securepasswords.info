---
title: Python
layout: post
color: fff
background: 1a3655
---

There are many Python libraries available that deal with hashing and security, but for managing passwords, one of the 
most popular is [passlib](https://pythonhosted.org/passlib/). Passlib provides a simple interface on top of a number of 
different hashing backends.

This guide will cover bcrypt which is _usually_ the best option available.


## Installation
Passlib supports 4 different bcrypt backends and provides its own pure-python fallback implementation (which should 
probably not be used in production). Passlib will use the first one of the following:


1. bcrypt
2. py-bcrypt
3. bcryptor
4. stdlib's crypt.crypt() on OSs that support bcrypt
5. passlib's fallback implementation (if enabled)


Install a backend and passlib using pip (or easy_install).

```sh
$ pip install bcrypt
$ pip install passlib
```

## Usage

Passlib's hashing interface exposes two main methods

- [`passlib.hash.bcrypt.encrypt(<string>[,salt=<salt>, rounds=12, ident='2a'])`](https://pythonhosted.org/passlib/lib/passlib.hash.bcrypt.html)
	Create a new password hash
- [`passlib.hash.bcrypt.verify(<string>, <hash>)`](https://pythonhosted.org/passlib/lib/passlib.hash.bcrypt.html)
	Confirm a given password matches the hash

### Hashing a Password
	
To hash a password, do the following:

```python
from passlib.hash import bcrypt

hash = bcrypt.encrypt("my_password")
```

The encrypt method takes a number of optional named params, most importantly rounds, which is an integer between 1 and 31 (inclusive) which
dictate how long it takes to hash a password. By default, the library will set this value to 12, but you can tune this up and down
based on the hardware your application is running on.

You also have the option of setting a salt. __NOT__ providing a salt will cause the library to generate a cryptographically secure random
salt for you, which is a very good thing.


### Verifying a Password

To verify a password, we do:

```python
if (bcrypt.verify("my_password", hash)):
    # Password matches
else:
    # Password doesn't match
```

