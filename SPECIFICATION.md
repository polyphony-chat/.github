# Polyphony Specification

**v0.0.0** - Treat this as an unfinished draft.

- [Polyphony Specification](#polyphony-specification)
  - [1. Polyphony APIs](#1-polyphony-apis)
    - [1.1. Client-Server API](#11-client-server-api)
    - [1.2. Server-Server API](#12-server-server-api)
  - [2. Federated Identity](#2-federated-identity)
    - [2.1 Federation tokens](#21-federation-tokens)
    - [2.2 Signing keys and message signing](#22-signing-keys-and-message-signing)
    - [2.3 Reducing network strain when verifying signatures](#23-reducing-network-strain-when-verifying-signatures)
    - [2.4 Best practices](#24-best-practices)
      - [2.4.1 Signing keys](#241-signing-keys)
      - [2.4.2 Home server operation and design](#242-home-server-operation-and-design)
  - [3. Federating direct/group messages](#3-federating-directgroup-messages)
    - [3.1 Direct messages](#31-direct-messages)
    - [3.2 Group messages](#32-group-messages)
  - [4. Users](#4-users)
  - [5. Encryption](#5-encryption)
    - [5.1 Encrypted guild channels](#51-encrypted-guild-channels)
    - [5.2 Encrypted direct messages](#52-encrypted-direct-messages)
    - [5.3 Encrypted group messages](#53-encrypted-group-messages)


This document defines a set of protocols and APIs for a chat service primarily focused on communities. The document is intended to be used as a reference for developers who want to implement a client or server for the Polyphony chat service. Uses of this protocol, hereafter referred to as "the Polyphony protocol", include Instant Messaging, Voice over IP, and Video over IP, where your identity is federated between multiple servers.

It is imperative that implementations of this protocol respect all aspects of this specification.

The structure of this reference document is heavily inspired by the really well written [Matrix specification](https://spec.matrix.org/latest).

## 1. Polyphony APIs

The specification defines a set of APIs that are used to implement the Polyphony protocol. These APIs are:

- Client-Server API
- Server-Server API

### 1.1. Client-Server API

The Client-Server API is a RESTful API that is used by clients to communicate with the server. It is a modification of the Discord v9 API and is completely backwards compatible with it, even if not all endpoints are supported. An example of an unsupported endpoint would be the "Super-reactions" endpoint, which are treated as regular reactions by Polyphony.

### 1.2. Server-Server API

The Server-Server APIs are used to enable federation between multiple Polyphony servers (federated identity).
TODO

## 2. Federated Identity

Federating user identities means that users can fully participate on other instances. This means that users can, for example, DM users from another server or join external Guilds. Each Polyphony user/client must hold on to a private-key.

This key is given to the user by their home server, and is used to sign messages that the user sends to other servers.

**Example:**
Say that Alice is on server A, and Bob is on server B. Alice wants to send a message to Bob.

Alice's client will send a message to her home server (Server A), asking it to generate a federation token for registering on server B. Alice takes this token and sends it to server B. Server B will then ask server A if the token is valid. If all goes well, server B will send a newly generated session token back to Alice's client. Alice's client can then authenticate with server B using this token, and send the message to server B. Server B will then send the message to Bob's client.

```
Alice's Client              Server A            Server B            Bob's Client
|                           |                   |                   |
|-Federation token request->|                   |                   |
|                           |                   |                   |
|<-----Federation token-----|                   |                   |
|                    [Federation handshake start]                   |
|                           |                   |                   |
|---------Federation token+Profile------------->|                   |
|                           |                   |                   |
|                           |<--Verification?---|                   |
|                           |                   |                   |
|                           |-----Yes, valid--->|                   |
|                           |                   |                   |
|<----------------Session Token-----------------|                   |
|                           |                   |                   |
|                   [Federation handshake complete]                 |
|                           |                   |                   |
|---------Session Token+Signed message--------->|                   |
|                           |                   |                   |
|                           |                   |--Signed message-->|
|                           |                   |                   |
```

Fig. 1: Sequence diagram of a successful federation handshake.

If Alice's session token expires, or if Alice would like to sign in on another device, she can repeat this process of generating a federation token and exchanging it for a session token.

The usage of a federation token prevents a malicious user from generating an external session token on behalf of another user.

### 2.1 Federation tokens

Federation tokens are generated by the user's home server. The token is a JSON object that contains the following information:

- The time at which the token expires.
- The user's federation ID.
- The domain of the server that the token is intended for.
- The domain of the user's home server.
- A securely-randomly generated nonce

### 2.2 Signing keys and message signing

As mentioned in the previous section, users must hold on to a private key at all times. This key is used to sign all messages that the user sends to other instances. The key is generated by the user's home server, and is sent to the user's client when the user first registers on the server. The key is stored in the client's local storage. The signing key must not be used for encryption purposes.

A home server may choose to rotate a users signing key at any time. When this happens, the home server will send a new signing key to the user. The user's client will have to save this updated key and use it when communicating with other servers. The home server has to keep track of the old signing key, and use it to verify messages that were signed with the old key.

The signing key should be generated using the ed25519 algorithm. Signing keys should be signed using the home servers' private key, so that home servers act as a certificate authority for their users' keys.

Signing messages prevents a malicious server from impersonating a user.

TODO: Note about signing keys and how they are generated

### 2.3 Reducing network strain when verifying signatures

If Bob receives a message from Alice, he will ask Server B to provide the public key of Alice at the time the message was sent. Server B will then ask Server A for this key. Server A will then send the appropriate key to Server B. Server B will then store this key in its database and forward it to Bob. Bobs' client should then ask Server A for its signing key, cache this key and verify that Server B has stored/provided the correct public key for Alice at the time the message was sent. Should Bob want to re-verify the signature of Alice's message in the future, or should another User of Server B want to verify the signature of Alice's message, Server B will already have the public key cached.

Bob's client could always ask Server A for the public key of Alice, but this would put unnecessary strain on the network. This is why Server B should cache the public keys of users from other instances.

### 2.4 Best practices

#### 2.4.1 Signing keys

- Instance/user signing keys should be rotated at least every 30 days. This is to ensure that a compromised key can only be used for a limited amount of time.
- If Bobs client fails to verify the signature of Alice's message with the public key provided by Server B, it should ask Server A for the public key of Alice at the time the message was sent. If the verification fails again, the message should be treated with extreme caution.

#### 2.4.2 Home server operation and design

- Employ a caching layer to handle the potentially large amount of requests for public keys without putting unnecessary strain on the database.

## 3. Federating direct/group messages

### 3.1 Direct messages

Federating direct messages is relatively simple. When Alice sends a message to Bob, her client will send the message to her home server via an API request. Her home server will then send the message to Bob's client via an established WebSocket connection, and vice versa.

### 3.2 Group messages

Group messages work just like guilds. The data is stored by the home server of the group's creator, meaning that all group members will have to communicate with the group creator's home server. If the group creator leaves the group, the ownership of the group is transferred to another member. The group chat stays on the group creator's home server, unless a migration is initiated by the group owner.

## 4. Users

Each client must have a user associated with it. A user is identified by a unique federation ID (FID), which consists of the user's username (which must be unique on the instance) and the instance's root domain. A FID is formatted as follows: `user@domain.tld`, which makes for a globally unique user ID. Federation IDs are case-insensitive.

The following regex can be used to validate user IDs: `\b([A-Z0-9._%+-]+)@([A-Z0-9.-]+\.[A-Z]{2,})\b`.

## 5. Encryption

The Polyphony protocol offers end-to-end encryption for messages via Message Layer Security (MLS). Polyphony protocol compliant servers take on the role of both an Authentication Service and a Delivery Service in context of MLS.

Message Layer Security (MLS) is a cryptographic protocol that provides confidentiality, integrity, and authenticity guarantees for group messaging applications. MLS builds on top of the [Double Ratchet Algorithm](https://signal.org/docs/specifications/doubleratchet/) and [X3DH](https://signal.org/docs/specifications/x3dh/) to provide these security guarantees.

Clients and servers must support encryption, but whether to encrypt a message channel is up to the users.

Note, that in the below sequence diagrams, the MLS Welcome message and the MLS Group notify message are all encrypted using the public key of the recipient. The public key in this context is not to be confused with the public signing key.

TODO: Note about encryption keys and how they are generated

### 5.1 Encrypted guild channels

Encrypting a guild channel is done by a client with the `MANAGE_CHANNEL` permission. Upon successfully requesting enabling encryption of a channel, all future messages in it will be encrypted. Joining an encrypted channel is done by sending a join request to the server. The server will then notify the channels' members of the join request. The members will then decide whether to accept or reject the join request. If the join request is accepted by any member, that member will initiate the MLS welcoming process. If the member finds that the join request is invalid (perhaps due to an invalid public key), the join request must be denied.

```
     Charlie                                        Server                                            Alice                     Bob
     |                                              |                                                 |                         |
     | Channel join request + pubkey                |                                                 |                         |
     |--------------------------------------------->|                                                 |                         |
     |                                              |                                                 |                         |
     |                                              | Notify gatekeepers of join request              |                         |
     |                                              |-----------------------------------              |                         |
     |                                              |                                  |              |                         |
     |                                              |<----------------------------------              |                         |
     |                                              |                                                 |                         |
     |                                              | Channel join request + Charlie's pubkey         |                         |
     |                                              |------------------------------------------------>|                         |
     |                                              |                                                 |                         |
     |                                              |                                                 | Verify Charlie's pubkey |
     |                                              |                                                 |------------------------ |
     |                                              |                                                 |                       | |
     |                                              |                                                 |<----------------------- |
     |                                              |                                                 |                         |
     |                                              |             Notify group of new member: Charlie |                         |
     |                                              |<------------------------------------------------|                         |
     |                                              |                                                 |                         |
     |                                              |     MLS Welcome (encrypted w/ Charlie's pubkey) |                         |
     |                                              |<------------------------------------------------|                         |
     |                                              |                                                 |                         |
     |                                              | Forward: Notify group of new member: Charlie    |                         |
     |                                              |-------------------------------------------------------------------------->|
     |                                              |                                                 |                         |
     | Forward: Notify group of new member: Charlie |                                                 |                         |
     |<---------------------------------------------|                                                 |                         |
     |                                              |                                                 |                         |
     |               Forward: encrypted MLS Welcome |                                                 |                         |
     |<---------------------------------------------|                                                 |                         |
     |                                              |                                                 |                         |
```
Fig. 2: Sequence diagram of a successful encrypted channel join in which Alice acts as a gatekeeper. The sequence diagram assumes that Alice can verify Charlies' public key to indeed belong to Charlie, and that Alice accepts the join request.

### 5.2 Encrypted direct messages

Adding another person to a direct message is not possible, and would not make much sense, as the new person cannot see any messages that were sent before they joined the group. If Alice wants to add Charlie to a direct message with Bob, she will have to create a new direct message with Bob and Charlie.

```
Alice                                          Server                             Bob
|                                              |                                  |
| Request Bob's public key                     |                                  |
|--------------------------------------------->|                                  |
|                                              |                                  |
|                             Bob's public key |                                  |
|<---------------------------------------------|                                  |
|                                              |                                  |
| Verify Bob's public key                      |                                  |
| -----------------------                      |                                  |
|                       |                      |                                  |
|<-----------------------                      |                                  |
|                                              |                                  |
| Notify group of new member: Bob              |                                  |
|--------------------------------------------->|                                  |
|                                              |                                  |
| MLS Welcome (encrypted w/ Bob's pubkey)      |                                  |
|--------------------------------------------->|                                  |
|                                              |                                  |
|                                              | Forward: New group member: Bob   |
|                                              |--------------------------------->|
|                                              |                                  |
|                                              | Forward encrypted MLS Welcome    |
|                                              |--------------------------------->|
|                                              |                                  |
```
Fig. 3: Sequence diagram of a successful encrypted direct message creation. 


### 5.3 Encrypted group messages

Encrypted group messages work by using the traditional MLS protocol, with the additional concept of group owners. Only group owners can add new members to the group and forcibly remove others from the group. The Group owner is determined by the Client-Server API.

```
Alice (gatekeeper)                                 Server                                  Bob       Charlie
|                                                  |                                       |         |
| Request Bob's public key                         |                                       |         |
|------------------------------------------------->|                                       |         |
|                                                  |                                       |         |
|                                 Bob's public key |                                       |         |
|<-------------------------------------------------|                                       |         |
|                                                  |                                       |         |
| Verify Bob's public key                          |                                       |         |
|------------------------                          |                                       |         |
|                       |                          |                                       |         |
|<-----------------------                          |                                       |         |
|                                                  |                                       |         |
| Notify group of new member: Bob                  |                                       |         |
|------------------------------------------------->|                                       |         |
|                                                  |                                       |         |
| MLS Welcome (encrypted w/ Bob's pubkey)          |                                       |         |
|------------------------------------------------->|                                       |         |
|                                                  |                                       |         |
|                                                  | Forward: New group member: Bob        |         |
|                                                  |-------------------------------------->|         |
|                                                  |                                       |         |
|                                                  | Forward encrypted MLS Welcome         |         |
|                                                  |-------------------------------------->|         |
|                                                  |                                       |         |
| Request Charlie's public key                     |                                       |         |
|------------------------------------------------->|                                       |         |
|                                                  |                                       |         |
|                             Charlie's public key |                                       |         |
|<-------------------------------------------------|                                       |         |
|                                                  |                                       |         |
| Verify Charlie's public key                      |                                       |         |
|----------------------------                      |                                       |         |
|                           |                      |                                       |         |
|<---------------------------                      |                                       |         |
|                                                  |                                       |         |
| Notify group of new member: Charlie              |                                       |         |
|------------------------------------------------->|                                       |         |
|                                                  |                                       |         |
| MLS Welcome (encrypted w/ Charlie's pubkey)      |                                       |         |
|------------------------------------------------->|                                       |         |
|                                                  |                                       |         |
|                                                  | Forward: New group member: Charlie    |         |
|                                                  |-------------------------------------->|         |
|                                                  |                                       |         |
|                                                  | Forward: New group member: Charlie    |         |
|                                                  |------------------------------------------------>|
|                                                  |                                       |         |
|                                                  | Forward encrypted MLS Welcome         |         |
|                                                  |------------------------------------------------>|
|                                                  |                                       |         |
```
Fig. 4: Sequence diagram of a successful encrypted group creation with 3 members.
