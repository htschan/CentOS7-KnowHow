# Linux


# OpenSSL

## Encrypt / Decrypt Files

Encrypt:

`tar -czf - * | openssl enc -e -aes256 -out secured.tar.gz`

You will beasked to enter the encryption password.

Decrypt:

`openssl enc -d -aes256 -in secured.tar.gz | tar xz -C test`
