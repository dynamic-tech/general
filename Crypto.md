# Overview

We consolidate all general-use crypto processes and know-how here.

# Table of Contents

- [OpenSSL Reference](#openssl-reference)
  - [Which OpenSSL Implementation?](#which-openssl-implementation)
- [Sending Encrypted Messages](#sending-encrypted-messages)
  - [PEM Key Format](#pem-key-format)
  - [Generate an Encryption Key](#generate-an-encryption-key)
  - [Encrypt the Encryption Key](#encrypt-the-encryption-key)
  - [Encrypt the Message](#encrypt-the-message)
- [Receiving Encrypted Messages](#receiving-encrypted-messages)
  - [Decrypt the Encryption Key](#decrypt-the-encryption-key)
  - [Decrypt the Message](#decrypt-the-message)

(TOC manually generated courtesy of [this generator](https://imthenachoman.github.io/nGitHubTOC/).)

# OpenSSL Reference

The man pages are consolidated [on OpenSSL's official site](https://www.openssl.org/docs/manmaster/man1/).

## Which OpenSSL Implementation?

[LibreSSL](https://www.libressl.org). Current version of this writing is `2.9.1`.

    openssl version

# Sending Encrypted Messages

Create a sample message:

    mkdir ~/temp
    cd ~/temp
    echo "Hello Yellow!" > data

## PEM Key Format

Use PEM key format. The format is Base64 encoded, for easy transport over text-based channels. In any case, you would eventually install your public-private key into your email system, which requires this key format.

**PEM** stands for **Privacy Enhanced Mail**.

Export your public and private key:

    openssl rsa -in id_rsa -outform pem > id_rsa.pem

    openssl rsa -in id_rsa -pubout -outform pem > id_rsa.pub.pem

## Generate an Encryption Key

Do not use your recipient's *public key* to directly encrypt the message you send.

**Reduce Key-Use Profile.** Encrypting a lot of data (say 1 year's worth of voluminous messages) with said *public key* will present a bigger profile for hackers to crack that *public key*. Granted, you will normally salt all encryptions, but it's still best to provide as little material for dictionary attacks (and similar) as possible.

**Generate a random encryption key** for each message you send. Generate one now.

    openssl rand -base64 32 > data.key

## Encrypt the Encryption Key

Encrypt the *encryption key* with your *public key*. This simulates you encrypting a secret (the *encryption key*, in this case) *with your recipient's public key* to send to your recipient. You would normally encrypt with your recipient's *public key*.

    openssl rsautl -encrypt \
      -inkey id_rsa.pub.pem -pubin \
      -in data.key -out data.key.enc

**Default Input Key Type.** Of special note is the `-inkey <public-key.pem> -pubin` phrase. It indicates that the input key file is a public key. By default, [the command](https://www.openssl.org/docs/manmaster/man1/rsautl.html) `openssl rsautl` expects the input key file to be a private key.

## Encrypt the Message

Now, we encrypt the secret message with our *encryption key*.

    openssl enc -aes-256-cbc -salt \
      -in data -out data.enc \
      -pass file:./data.key

We will now refer to [the man page](https://www.openssl.org/docs/manmaster/man1/enc.html) for `openssl enc`.

**Cipher to Use.** As per the man page, the easiest and safest cipher to use is the [AES block cipher](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard) in [CFB mode](https://en.wikipedia.org/wiki/Block_cipher_mode_of_operation#Cipher_Block_Chaining_(CBC)) using the longest key length of 256 bits.

**Salting** is required to prevent dictionary attacks.

**Password** is not needed; our argument to `-pass` is a file. We use the *encryption key* as "*password*", which is a lot more secure than a password that is humanly possible to type.

# Receiving Encrypted Messages

As per PKI, the recipient will use her private key for decryption.

## Decrypt the Encryption Key

Use the private key to decrypt the *encryption key*. Note that the *encryption key* is a *symmetric key*, so it acts as both an *encryption key* as well as a *decryption key*.

    openssl rsautl -decrypt \
      -inkey id_rsa.pem \
      -in data.key.enc -out received.key

## Decrypt the Message

Use the *encryption key* to decrypt the encrypted message.

    openssl enc -d -aes-256-cbc \
      -in data.enc -out received.data \
      -pass file:./received.key
