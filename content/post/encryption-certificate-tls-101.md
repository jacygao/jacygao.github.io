---
title: "Auth 101 - Key, Secret and Certificate"
date: 2022-03-13T22:17:27+08:00
draft: true
---

In a modern world of Cloud Computing and Distributed Systems, two key principles are Data Encryption in Transit and at Rest. This blog talks about the fundamentals involved in these two principles.  

# Encryption

## Encryption during World War II

During the world war II, a cipher device known as the Enigma Machine was employed extensively by the Nazi Germany military to keep communications secret. 

![Enigma Machine](https://jgao.io/auth101-enigma.jpg)

It works by manipulating paths of eletric signals through the plugboard on the front and the 3 rotors on the back to light up a letter bulb on the lampboard different from the key pressed on the Keyboard. Two Enigma Machines must be configured identically including the positions on the plugboard and the rotors to encryt and decrypt the messages. [If you are intereted in how the machine works, make sure to check out this video.](https://youtu.be/ybkkiGtJmkM)

The plugboard and the rotors are just like the key and algorithm of digital encryption in modern days. Simply put, if an Encryption (and Decription) process is defined using the following two methods, the implementation of these two method would be the algorithm.

```(go)
    Encrypt(string key, string text) string cipertext
    Decrypt(string key, string cipertext) string text
```

# Encryption Keys

There are two kinds of encryption keys:

- Symetric Keys

- Asymetric keys

With Symetric key, both encryption and decryption processes using the same key. This is sometimes defined as a secret or session key.

With Asymetric key, two separate keys are used for encryption and decryption. One common example is the private and public keys used for TLS. The two keys have to be used in pairs. Both keys can be used as the encryption key or decryption key. One can decrypt messages encrypted by the other key.

## Encryption Algorithms

- Examples of using encryption

# Certificate

- Server Certifcate

- Client Certificate

- Certificate Signiture

- Chain of Trust

- TLS Handshake (server and client)