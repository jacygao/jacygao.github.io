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

## Encryption Keys

There are two kinds of encryption keys:

- Symetric Keys

- Asymetric key

With Symetric key, both encryption and decryption processes using the same key. Symetric keys are commonly used for encryption at rest.

With Asymetric key, two separate keys are used for encryption and decryption. One common example is the private and public keys used for TLS. The two keys have to be used in pairs. Both keys can be used as the encryption key or decryption key. One can decrypt messages encrypted by the other key.

## Encryption Algorithms

Popular encryption algorithms include RSA

# Digital Certificate

A Digital Certificate is a file that proves the autheticity of a server, device or user. A certificate file contains the following main components:

- Digital Siginiture of the Certificate Authority(CA)
- Certificate owner's Public Key 
- Expiry Dates
- Name of certificate holder
- Serial number

## Certificate Authority(CA)

Certficate Authoritis are entities that issue certificates. CA acts as the trusted third party between the owner of the certificate and the party relying upon on the certificate. 

A Certificate Authority can be public or private. Public CAs such as [IdenTrust](https://en.wikipedia.org/wiki/IdenTrust) and [DigiCert](https://en.wikipedia.org/wiki/DigiCert) can issue certificate trusted by most browsers and operating systems used in public channels such as a web server. An organisaction can also creates its own private CA to issue digital certificates used within the organisation's IT environment.

## Server Certificate vs Client Certificate

There are two kinds of digital certificates, Server Certificate and Client Certificate. Both certificates are issues by a trusted CA.

A server certificate is used to validate the identity of a server. For example, a valid server certificate can prove that you are a legit owner of the webserver and ensure data sent to and received from your web service is encrypted. This encryption process is called TLS Handshake. It is also known as Data encryption in transit.

A client certificate is used to validate the identity of a client. Differently from Server Certificates, Client Certificates do not encrypt any data. It is usually used as a replacement of a simple auth (username and password). When a client tries to connect to a server with a client certificate

## Self Signed Certificate

## Certificate Storage

## Signing

## Chain of Trust

# TLS/SSL

## TLS Handshake (server and client)