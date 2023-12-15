# Polyphony Specification

This document defines a set of protocols and APIs for a chat service primarily focused on communities. The document is intended to be used as a reference for developers who want to implement a client or server for the Polyphony chat service. Uses of this protocol, hereafter referred to as "Polyphony" or "the Polyphony protocol", include Instant Messaging, Voice over IP, and Video over IP, where your identity is federated between multiple servers and clients.

This reference document is heavily inspired by the really well written [Matrix specification](https://spec.matrix.org/latest).

## 1. Polyphony APIs

The specification defines a set of APIs that are used to implement the Polyphony protocol. These APIs are:
- Client-Server API
- Server-Server API

### 1.1. Client-Server API

The Client-Server API is a RESTful API that is used by clients to communicate with the server. It is a modification of the Discord v9 API and is completely backwards compatible with it, even if not all endpoints are supported. An example of an unsupported endpoint would be the "Super-reactions" endpoint, which are treated as regular reactions by Polyphony.

### 1.2. Server-Server API

The Server-Server APIs are used to enable federation between multiple Polyphony servers (federated identity).

## 2. Federated Identity

Federating user identities means that users can fully participate with users/in guilds on other servers. This means that users can send messages to users on other servers, and join guilds on other servers. Each Polyphony user, regardless of their instance, holds on to two things:
1.  A federation-JWT which is signed using their home servers' private key. This token is used to authenticate with other servers, and is used to verify that the user is who they say they are.
2.  A private-key. This key is given to the user by their home server, and is used to sign messages that the user sends to other instances.

Say that Alice is on server A, and Bob is on server B. Alice wants to send a message to Bob. Alice's client will send a message to server B, checking if server B agrees to receive the message. This initial message contains Alice's federation-JWT, which was signed by Server A, as well as her public profile. Server B will now ask server A if Alice's federation-JWT is valid. If all goes well, server B will store Alice's public profile and send a newly generated user token back to Alice's client. Alice's client can then authenticate with server B using this token, and send the message to server B. Server B will then send the message to Bob's client.

## 2.1 Federation-JWT and Signing Key

A server may choose to rotate a users signing key/JWT token at any time. When this happens, the server will send a new JWT token and signing key to the user. The user's client will have to save these updated tokens, and use them when communicating with other servers. The server has to keep track of the old JWT token and signing key, and use them to verify messages that were signed with the old key.

## 2.2 Reducing network strain when verifying signatures

If Bob receives a message from Alice, he will ask Server B to provide the public key of Alice at the time the message was sent. Server B will then ask Server A for this key. Server A will then send the appropriate key to Server B. Server B will then store this key in its database and forward it to Bob. Bobs' client should then ask Server A for its signing key, cache this key and verify that Server B has stored/provided the correct public key for Alice at the time the message was sent. Should Bob want to re-verify the signature of Alice's message in the future, or should another User of Server B want to verify the signature of Alice's message, Server B will already have the public key cached.

Bob's client could always ask Server A for the public key of Alice, but this would put unnecessary strain on the network. This is why Server B should cache the public keys of users from other instances.

## 2.3 Best practices

### 2.3.1. Signing keys

- Signing keys should be rotated every 30 days. This is to ensure that a compromised key can only be used for a limited amount of time.
- If Bobs client fails to verify the signature of Alice's message with the public key provided by Server B, it should ask Server A for the public key of Alice at the time the message was sent. If the verification fails again, the message should be treated with extreme caution.

