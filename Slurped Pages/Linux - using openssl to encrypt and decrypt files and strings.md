---
link: https://fabianlee.org/2023/01/25/linux-using-openssl-to-encrypt-and-decrypt-files-and-strings/
site: "Fabian Lee : Software Engineer"
excerpt: "If you need a way to simply encrypt and decrypt files or strings with a symmetric key (same password for encryption and decryption), openssl provides this functionality.  Be sure to avoid weak ciphers such as 3DES, and employ salt as well as PBKDF2 iterations to reduce vulnerability to brute force attacks. Here is a simple ... Linux: using openssl to encrypt and decrypt files and strings"
twitter: "https://twitter.com/Fabian Lee : Software Engineer"
tags:
  - slurp/base64
  - slurp/cipher
  - slurp/decrypt
  - slurp/encrypt
  - slurp/file
  - slurp/openssl
  - slurp/string
  - slurp/symmetric
  - ssl
slurped: 2025-03-02T10:12
title: "Linux: using openssl to encrypt and decrypt files and strings"
---

![](https://fabianlee.org/wp-content/uploads/2018/10/gnu-logo.gif)If you need a way to simply encrypt and decrypt files or strings with a symmetric key (same password for encryption and decryption), openssl provides this functionality.  Be sure to avoid weak ciphers such as [3DES](https://www.techtarget.com/searchsecurity/tip/Expert-advice-Encryption-101-Triple-DES-explained), and employ [salt](https://asecuritysite.com/openssl/passwds) as well as [PBKDF2](https://support.1password.com/pbkdf2/) iterations to reduce vulnerability to brute force attacks.

Here is a simple example of encrypting a string, and then restoring it.

# write encrypted to file
echo "this is my secret" | openssl enc -aes-256-cbc -pbkdf2 -iter 1234567 -salt -pass pass:mypass > mysecret.enc

# read encrypted file and decrypt
cat mysecret.enc | openssl enc -d -aes-256-cbc -pbkdf2 -iter 1234567 -salt -pass pass:mypass

You should avoid the ‘[pass](https://www.openssl.org/docs/man3.1/man1/openssl-passphrase-options.html)‘ flag that exposes the password at the command line (I used it for simplicity of the example).  By default it prompts for user input but there are also options for specifying via environment variable, file descriptor, and file.

The same logic would apply to a binary file as shown below.

# write encrypted to file
cat mysecret.jpg | openssl enc -aes-256-cbc -pbkdf2 -iter 1234567 -salt -pass pass:mypass > mysecret.jpg.enc

# read encrypted file and decrypt
cat mysecret.jpg.enc | openssl enc -d -aes-256-cbc -pbkdf2 -iter 1234567 -salt -pass pass:mypass > myunencrypted.jpg

And if you need to work with ASCII text (friendly to copy-paste) instead of the encrypted binary, you can always wrap it with Base64 encoding.  Here is the first example again, but this time with a Base64 wrapper.

# write encrypted to file as base64
echo "this is my secret" | openssl enc -aes-256-cbc -pbkdf2 -iter 1234567 -salt -pass pass:mypass | base64 | tee mysecret.encb64

# decode base64 then decrypt
cat mysecret.encb64 | base64 --decode | openssl enc -d -aes-256-cbc -pbkdf2 -iter 1234567 -salt -pass pass:mypass

REFERENCES

[stackoverflow, use of openssl for encryption/decryption also shows gpg usage](https://stackoverflow.com/questions/16056135/how-to-use-openssl-to-encrypt-decrypt-files/31552829#31552829)

[wikipedia, PBKDF2 purpose](https://en.wikipedia.org/wiki/PBKDF2)

[1password.com, use of PBKDF2 and costs associated with higher values](https://support.1password.com/pbkdf2/)