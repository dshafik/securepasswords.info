---
title: "Golang"
layout: "post"
color: "fff"
background: "375eab"
author: "Lorenzo Fontana"
author_url: "http://github.com/fntlnz"
author_twitter: "fntlnz"
gravatar: "7b10b1839f3fded11e74a72dfb382b4c"
---

Golang doesn't have a built-in bcrypt implementation but has an official implementation wrote by the core team
which is available [here](https://github.com/golang/crypto/tree/master/bcrypt)

## Installation

```bash
$ go get golang.org/x/crypto/bcrypt
```

## Usage

The golang bcrypt implementation is very easy to use, you only need to import the bcrypt package and call
the `GenerateFromPassword` function to obtain a secure password hash that can be checked by the `CompareHashAndPassword`
function or by any other implementation of the bcrypt hashing algorithm.

### Hashing a Password

```go
import (
    "golang.org/x/crypto/bcrypt"
)
```

To hash a password, do the following:

```go
originalPassword := []byte("mysupersecretpassword")

cost := 16

hash, err := bcrypt.GenerateFromPassword(originalPassword, cost)

if err != nil {
   // Do something to handle the error
}
```

The `originalPassword` has been hashed with a `cost` of 16, even if default value is 10
nowadays computers are getting faster so it makes sense to increase this.

### Verifying a Password

To verify a password we call the `CompareHashAndPassword` function with hash and originalPassword
as arguments. If nil is returned the password is valid.

```go
// Compare the original password with the bcrypt hash

// Compare the original password with the bcrypt hash
if bcrypt.CompareHashAndPassword(hash, originalPassword) {
   // password matches
} else {
   // password does not match
}
```

## Links

- [package bcrypt Documentation](https://godoc.org/golang.org/x/crypto/bcrypt)