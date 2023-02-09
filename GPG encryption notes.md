Using GPG
=========

GPG is a free implementation of the PGP encryption standard, which is a public key encryption system that can be used to do things like encrypt email, or files that are going to be sent to others, etc.

Basics of public key encryption
-------------------------------

Public key encryption works by having a pair of keys: a secret or private key, and a public key.

Information is encrypted with the public key, and decrypted with the private key. The public key is not able to decrypt, and so it is safe to share it publicly in order to let others send you encrypted data.

For example, you've likely seen one of the following:

```


-----BEGIN PGP PUBLIC KEY BLOCK-----

mI0EYvlVowEEANIgYtWtMWMd5VVHVq8efME9bP5xo8nLL0Qrt4kp8ICg7DrqNAp9
DL0MQAF7O5xJtvb/wJ4OpZDavyVhCIN/4+mvMEvdlbicejxuMKx087A+u5dQEo/8
JNIE7Kk646gqCfZXfoN1wk+QUf+q0WW4RTd0B8vNSwSHvFOpDqg0NIFvABEBAAG0
DGV4YW1wbGVfbmFtZYjOBBMBCgA4FiEETjTfCJ1Yeu4po2+hGcAS+bVZggYFAmL5
VaMCGwMFCwkIBwIGFQoJCAsCBBYCAwECHgECF4AACgkQGcAS+bVZggaqsQP/YaJy
pztBsIKYp5VJqPJSa4JWBElHAM4dbLfXzCealLSUDKKz+zi5kvzY2SGeZODdGeNa
YcSO8vR0spZoJNDvV0/vy9bjhHK46WjGeATyxQrG7gcdnPsEV75nOeoqsqwfBDfe
5MVlkkFB7hVbLQYz55wTu+VqHK6RwJzP1CVN/MO4jQRi+VWjAQQA1cvnTIJno1lg
NQjWAei6a+bMd9QY09SHmGeLztKNBCYWtv3I9iKgiFyOYce3/2W5yLRt0Aa1zO+k
E3cufASvmgV8kfQaN9rlSPt/sd2aRFZWBAHRzCKXVzIdKs9Y1GAVgO+HsoUghqkY
FcGjKJxZJlMFB7eZrWf49m3L4lihDW8AEQEAAYi2BBgBCgAgFiEETjTfCJ1Yeu4p
o2+hGcAS+bVZggYFAmL5VaMCGwwACgkQGcAS+bVZggZarAP+KoHtJJpcGuzoVDl9
CjRs+TzdNSvE+WM7eAp5BYV8vjbN1DD+88mhxI+OWKraVBMxrKOKFcenf90ubu9+
s7roSoVA8g9KIbVI6pTUZ5kTNONePjyQ19GcJQN24bejhpKwzq0V8WeqLAMxYlnl
8fKoYWnB8pgNsgpc8+iFFlvwc7A=
=d/SA
-----END PGP PUBLIC KEY BLOCK-----

```

This is a public key, exported as ASCII so as to be more human readable. If you were to import that key into GPG, you could then use it to encrypt data and send it to that user. Once you've encrypted it, if you were to send the encrypted data to that user, he could use his private key to decrypt it and read the data!

By posting your public key like this, you can enable others to send you encrypted data!

Using GPG
---------

#### Generating and importing/exporting Keys

To start off, you'll want to generate a key-pair for yourself. This can be done like so:

```
gpg --full-gen-key
```

This will lead you through a brief wizard that generates a key-pair. Bear in mind that this wizard will prompt you to generate a pass*phrase*, and not a simple password. This is significantly more secure!

Once we've generated our keys, we can export it to share with others with the following:

```
gpg --armor --export [key_username]
```

This will create one of those big PGP key blocks like the one above that can be readily shared with other humans, e.g. on websites, etc. Alternatively, we can output a binary key that's more useful for machines like so:

```
gpg --output [output filename] --export [key_username]
```

This will output a binary file containing the key, which is *not* particularly human-readable.

In order to import another person's public key, we'd do a few things. Firstly, if we're importing an ASCII key, we would copy the *entire* key block -- including the "-----BEGIN PGP PUBLIC KEY BLOCK-----" and "-----END PGP PUBLIC KEY BLOCK-----" lines -- and save these to a file. We could then type:

```
gpg --import [path to public key]
```

#### Encrypting and Decrypting

At this point, we have everything we need to send encrypted data to someone. We would do that like so:

```
gpg --output [output file] --encrypt [plaintext file] --recipient [keyname]
```

This will output an encrypted file that only our intended recipient will be able to decrypt.

Decrypting is similar:

```
gpg --output [filename] --decrypt [encrypted file]
```

You would then be prompted for your passphrase, and it will output the plaintext to whatever filename you passed to the output option.

#### Listing and deleting keys

To see what keys you have:

```

gpg --list-keys
gpg --list-secret-keys
gpg --list-public-keys
```

These three commands will show you what's on the tin: all keys, just secret keys, and just public keys, respectively.

And finally, to delete keys, e.g. for users who no longer exist, etc:

```

gpg --delete-key [key name]
gpg --delete-secret-key [key name]
```

The first will remove a public key, while the second removes private keys. Note that for users that you have both keys for, you must delete the secret key first!

And that's about it for GPG. Go encrypt things now!
