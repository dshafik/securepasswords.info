---
title: Go
layout: post
color: "000"
background: 85daf7 
---

Go&rsquo;s excellent [crypto libraries](https://golang.org/pkg/crypto) make it really easy to have secure, salted passwords.

## Installation

Beyond the crypto libraries in the standard library, we&rsquo;s also going to be making use of the non-standard library
[crypto package](https://godoc.org/golang.org/x/crypto) maintained by the Go team. You can install that just like any other
Go package:

```sh
$ go get golang.org/x/crypto
```

## Usage

To implement this, you&rsquo;ll need four functions:

- `Create(h func() hash.Hash, iters int, passphrase []byte) (result, salt []byte, err error)`
	Create a new password hash, generating a random salt as you do.
- `CalculateIterations(h func() hash.Hash, delay time.Duration) (int, error)`
	Calculate the number of iterations your hardware needs to reach the specified delay.
- `Hash(h func() hash.Hash, iters int, passphrase, salt []byte) []byte`
	Hash the input to return a slice of bytes to check against the stored hash.
- `Compare(candidate, expectation []byte) bool`
	Compare the hashed candidate against the stored expectation in constant time.

### Hashing a Password
	
To hash a password, you&rsquo;ll need the `Create` function defined. Here&rsquo;s what it looks like:

```go
import (
	"crypto/rand"
	"hash"
)

func Create(h func() hash.Hash, iters int, passphrase []byte) (result, salt []byte, err error) {
	salt = make([]byte, 32)
	_, err = rand.Read(salt)
	if err != nil {
		return "", "", err
	}
	result = Hash(h, iters, passphrase, salt)
	return result, salt, err
}
```

This works by passing in the hashing algorithm you want to use, the number of iterations of that algorithm
you want to use when generating your hashes, and the passphrase (as a slice of bytes) the user supplied.

We&rsquo;ll talk about getting the proper number of iterations and the right hash in a minute. But first,
let&rsquo;s look at what this function is doing with the information.

The first thing it does is to allocate 32 bytes of space and fill them with cryptographically-secure random
data. This is going to be your password-specific salt.

Then it uses your `Hash` function to generate the hash of your input, which it returns as the result.

The important thing here is that you need to store the following information, _per passphrase_, or you won&rsquo;t
be able to compare hashes again:

* the randomly generated salt
* the number of iterations of the hashing algorithm that were used
* the hashing algorithm (this can be hardcoded in your application and doesn&rsquo;t need to change per-user)
* the generated hash

Using the function looks something like this:

```go
import (
	"crypto/sha256"
)

func main() {
	h, salt, err := Create(sha256.New, 100000, "my super secure passphrase")
}
```

That would use the SHA-256 hashing algorithm, hash the passphrase 100,000 times, and use `"my super secure passphrase"`
as the passphrase.

### Verifying a Password

Verifying a password is a bit trickier. We need to:

- retrieve the user&rsquo;s authentication information (the salt, number of iterations, and generated hash)
- generate _another_ hash from their input
- compare the generated hash to the retrieved hash _in constant time_

Retrieving the authentication information from your datastore is outside the scope of this guide, but hashing the
input looks like this:

```go
import (
	"hash"

	"golang.org/x/crypto/pbkdf2"
)

func Hash(h func() hash.Hash, iters int, passphrase, salt []byte) []byte {
	hashInstance := h()
	return pbkdf2.Key(passphrase, salt, iters, hashInstance.Size(), h)
}
```

This uses the PBKDF2 algorithm to generate a cryptographic hash from your hash algorithm. And because the iterations
are variable, this algorithm can adapt to faster hardware over time, meaning you can protect users as time goes on.

We&rsquo;ve already seen this in action: we use this hashing function when creating your stored hash.

Once you have the correct hash retrieved and the candidate hash calculated, you can compare them to see if the
candidate is valid. But simple comparisons may be vulnerable to subtle timing attacks. To prevent this, use the
standard library&rsquo;s `crypto/subtle` library to do a constant-time comparison:

```go
import (
	"crypto/subtle"
)

func Compare(candidate, expectation []byte) bool {
	candidateConsistent := make([]byte, len(candidate))
	expectationConsistent := make([]byte, len(candidate))
	subtle.ConstantTimeCopy(1, candidateConsistent, candidate)
	subtle.ConstantTimeCopy(1, expectationConsistent, expectation)
	return subtle.ConstantTimeCompare(candidateConsistent, expectationConsistent) == 1
}
```

This function takes two hashes as slices of bytes. The first thing it does is create two new slices with enough
space to hold the candidate hash. It&rsquo;s important that the slices be the same length&mdash;the constant-time
comparison requires it&mdash;and we don&rsquo;t want to accidentally leak the length of the expectation, so we&rsquo;re
using the candidate&rsquo;s length for both. Next, we use a constant-time copy function to fill these slices with the
candidate and expectation. Finally, we use a constant-time comparison to return whether the slices are equal or not.

### Determining the Right Number of Iterations

The variable iterations are an important part of keeping your users secure, but what number should you use? There&rsquo;s a
fundamental tradeoff here: the more iterations you use, the longer it takes someone to break your encryption if the hashes
are compromised. On the other hand, however, the more iterations you use, the more computer resources you need to verify
your users&rsquo; passwords.

The rule of thumb I always use is that hashing the password should take a single second on your hardware. That&rsquo;s about
as much time as you can spend before users get noticeably frustrated. But how many iterations can your hardware do in a second?

Let&rsquo;s ask your hardware:

```go

import (
	"hash"
	"time"

	"golang.org/x/crypto/pbkdf2"
)

func CalculateIterations(h func() hash.Hash, duration time.Duration) (int, error) {
	hashInstance := h()
	salt := make([]byte, 32)
	_, err := rand.Read(salt)
	if err != nil {
		return 0, err
	}
	iter := 2048
	var iterDur time.Duration
	for iterDur < duration {
		iter = iter * 2
		timeStart := time.Now()
		pbkdf2.Key([]byte("password1"), salt, iter, hashInstance.Size(), h)
		iterDur = time.Since(time.Start)
	}
	return iter, nil
}
```

Now when starting your server up, you can call `CalculateIterations` before you start serving requests:

```go
import (
	"crypto/sha256"
	"time"
)

func main() {
	iterations, err := CalculateIterations(sha256.New, time.Second)
	if err != nil {
		panic(err)
	}
	// start your server, etc.
}
```

Now `iterations` holds the appropriate number of iterations for your specific hardware: the most users
will tolerate.

### Upgrade Your Encryptions

Hardware speeds up, so we need to reencrypt user passphrases with higher iterations as we go. But you
don&rsquo;t have access to the cleartext; you need the user to enter it again so you can reencrypt it
and store the hash. So here&rsquo;s a sample of how to do that:

```go

import (
	"crypto/sha256"
)

func handleLogin(user, pass string) bool {
	var iters int // this needs to be populated from your database; the number of iterations
	              // the passphrase was originally encrypted with
	var salt []byte // this needs to be populated from your database; the salt that generated
	                // for this passphrase
	var realPass []byte // this needs to be populated from your database; the hashed passphrase
	candidate := Hash(sha.256, iters, []byte(pass), salt)
	if !Compare(candidate, realPass) {
		// fail; this is an invalid combination
		return false
	}
	if CurrentIters <= iters { // if the server is configured with fewer iterations than the
		return true        // passphrase was originally encrypted with, don't reencrypt
	}
	// our new encryption will be stronger, so upgrade
	newPass, newSalt, err := Create(sha256.New, CurrentIters, []byte(pass))
	if err != nil {
		// log your error and bail out of the upgrade
		return true
	}
	// store newPass, newSalt, and CurrentIters for this user
	return true
}
```

Now when your users log in, if the number of iterations your server is configured to run on new
passwords is greater than the number of iterations the password was hashed with, it will be
rehashed with the stronger encryption, meaning as hardware advances, your password encryption will
keep pace.
