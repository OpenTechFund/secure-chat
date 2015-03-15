# Introduction

The way people use chat and instance message has evolved considerable since the days of simple SMS or ICQ. Since then, a new type of chat has started to rapidly overtake old chat. The user experience of "new chat" or "chat 2.0" is marked by support for asynchronous messages (aka offline messages), private groups, rich inline media, no status updates, and the ability to smoothly switch from device to device without losing the conversation. New chat promises that your data will always be available, your message will always get delivered, you can read your chat history, and you can use whatever device you want.

New chat is not simply old chat with better features. It is a distinct user experience that has replaced old methods of instant messaging. Many users are starting to use new chat apps as a replacement for social networking: they create private chat groups where they share updates, pictures, and links. Unfortunately, the tools we have for secure chat have not kept pace with this development.

This document is a draft roadmap for creating the next generation of secure chat that is easy, secure, with the user experience that people have come to expect. As a placeholder, this document will refer to this new secure chat as "Newchat".

# Required features

These are the required and highly desirable features of a new chat system in the long run. The essential question this document attempts to ask is, "what is the best way to incrementally get there?"

User experience features:

* Asynchronous: The ability send messages to people when they are offline and ensure the recipient will receive them.
* Multi-devices: The user must be able to seamlessly switch devices without losing any messages. If stored, user messages must be encrypted locally when saved to local disk or backed up to cloud storage.
* Friendly identity: Users should be identified by an easy to remember identifier that is stable over time. The assumed format is username@domain for unique identifiers (like email, SIP, and XMPP chat).
* Groups: Users today demand the ability to form private group chats. The primary need here is not for ad-hoc group chats, but private chats where the list of allowed users is determined in advance.
* Media-rich: Users must be able to securely attach links, images, media streams, and other resources to their chat messages.
* Robust mobile support: Good support for mobile requires more than just a nice mobile app, it also requires alternatives to always-open TCP connections (aka push notifications) and support for "Stream Management" (so the server and client can gracefully restore a dropped connection).
* Guaranteed delivery: If a message does not reach the recipient, the sender should be alerted to the error.
* Offline support: Despite the proliferation of network connectivity, network quality in practice can be very spotty. Ideally, client applications should allow the user to interact with the chat system while offline, and automatically re-synchronize with other users when network is restored.

Security features:

* Forward secret message encryption: Forward secrecy ensures that an attacker cannot later decrypt messages once they have discovered the private key of any of the participants or once they have cracked the encryption by other means. This is typically accomplished by using "ephemeral" message encryption keys that are frequently discarded, making decryption of past messages very difficult.
* Authenticity: The public keys of other chat participants must be strongly authenticated.
* Invisible keys: The confidentiality of messaging encryption relies on the proper use of public key cryptography, but in practice this is both conceptually and practically too difficult for the common user to manage correctly. Instead, the goal here is "invisible keys" where all the key management happens automatically and without the user's knowledge, but with the highest level of security. This includes functions such as key generation, key discovery, key validation, key expiration, and key synchronization.
* Meta-data protection: The routing and message encryption protocols must not leak metadata which could disclose the parties involved in a given communication.

Infrastructure features:

* Federated: The protocol must support open federation that allows participants from different service providers to be able to easily communicate.
* Provider best practices: The long term goal of this roadmap is to entirely remove liability from the service provider and allow the user to use the provider without trusting them. However, in the medium term, there are likely to be many areas where a service provider can still enhance the security of the user, such as in ensuring authenticated, encrypted transport when relaying messages among servers, preventing timing attacks, and aiding in key validation by endorsing user keys.

# Chat Protocols

In order for two or more Newchat compatible clients to communicate with one another, they must each support the following protocols:

* Message Transport and Routing
* Message Encryption
* Key Discovery and Validation
* Resource Linking

## Message Transport and Routing

A baseline assumption of this roadmap is that Newchat will use a model of open federation, possibly with some peer-to-peer elements for securing routing metadata.

In the traditional model of federated XMPP chat, messages are passed from the user's client to their service provider, relayed to the recipient's provider, and then finally delivered to the recipient's client. A federated model ensures that the user is not locked into a single provider, they are able to communicate with users from other providers, and that multiple organizations all have a stake in seeing the system work well and securely. This model also provides users with a high degree of data availability and responsiveness (messages are available when they want them and they are delivered quickly).

However, even when all messages are encrypted, a strict federation of this type has one fatal flaw: the user is forced to trust their service provider with all their metadata and routing information. In one variation of the federated model, this problem is solved by making the message sender's client application deliver a message directly to the recipient's provider, via some anonymization system.

Other possible models include:

* Central Authority: In this model, a single organization is responsible for routing all messages. A purely centralized model is easier to implement, at the cost of making it very difficult to externally verify the security properties of the system. Centralized systems locks the user into a particular provider and limits their control over their own data. Unless the software is carefully distributed independently of the central authority, the user must always trust the central authority no matter what additional cryptography is added to the system.
* Peer-to-Peer: In this model, message routing and transport is handled by a network of dynamic and ad-hoc nodes, typically comprised of software running on the end users' devices. A purely decentralized peer-to-peer model takes the provider out the picture, but typically requires identifiers that are not human friendly, is difficult to make resistant to traffic analysis, is typically too resource intensive for mobile devices, is slower, and sacrifices data availability.

It is likely that Newchat will eventually need to combine elements of all three models (centralized, federated, and peer-to-peer) in order to take advantage of the particular solutions that each approach solves better than the others. For example, imagine a system with the following elements:

1. An optional centralized registry that maps phone numbers to user identifiers, in order to allow automatic upgrades to a secure channel when communicating via traditional mobile SMS or voice. This registry could be a "closed federation", with a fixed number of authorized registrars.
1. An open federation of chat providers that give each user a stable unique identifier and the ability to receive messages.
1. An optional peer-to-peer mechanism for delivering outgoing chat messages to the recipient server without the server being able to record the message metadata.

### Possible candidates

**XMPP**

 XMPP provides a solid foundation for federated chat. After a decade, the libraries for XMPP are starting to actually mature. The fatal flaw of XMPP is the huge number of possible extensions to the protocol, which makes it difficult to ever achieve smooth compatibility between any two clients or servers. If Newchat were to use XMPP, it should be modified in the following manner:

* A Newchat compatible client and server should only support a very specific subset of the available XMPP extensions.
* Most XMPP extensions that are allowed should also be required, such as offline message support.
* All transport connections should be required to be encrypted using TLS with PFS ciphers.
* The content of every message should be encrypted using Newchat Message Encryption.
* For mobile devices, the client should not keep an open TCP connection at all times. Instead, the device should establish a new connection after it has received a push notification that new content is available.
* Newchat should not support continual presence notification. A traditional XMPP system is designed so that every user receives regular updates regarding the online status of their contacts. For security and performance reasons, Newchat should allow presence information only as a response to a specific query for it.

[dgoulet] That is kind of a problem for online/offline state of clients which is quite important for someone to know relatively fast if a buddy is online or not. Maybe that does not fit in the "continual presence notification" so if not maybe be more specific?

* Newchat should include additional extensions not part of existing XMPP protocol, such as the ability for a sender to request that the recipient client disable message logging.

Although XMPP does not support secure routing that protects metadata, an XMPP based protocol could include extensions for it.

**TextSecure**

Whispersystems TextSecure mobile application uses a client-server architecture that currently operates as a "closed federation". There is nothing that prevents this protocol and software from being modified to support an open federation with username@domain identifiers.

Attractive properties of the TextSecure transport protocol:

* It is modern protocol designed for mobile devices, using a REST API and push notifications, rather than the ancient always-open TCP connection using XML found in XMPP.
* The reference server implementation is well written, fast and stable.

## Message Encryption

The Message Encryption protocol specifies how each message is encrypted, and how a conversation is validated. It is orthogonal to whatever routing mechanism is used. For Newchat, the message encryption should include the following properties:

* confidentiality: attacker should not be able to intercept a message and read it.
* authenticity: the receiver should be assured the message has not been modified after being sent.
* optional non-repudiation: a message sender should have the option of providing proof that the message came from them or not.
* perfect forward secrecy: attacker who obtains a shared ephemeral key should not be able to read prior messages.
* unlinkable: the message encryption should into leak information that can be used to correlate traffic (mostly, this is the responsibility of the routing protocol, but the message encryption also needs to make it hard to map who is communicating with whom).

[dgoulet] "should into leak" --> "should NOT leak" ? 

* sequence integrity: an attacker should not be able to drop messages or replay messages without participants noticing.

Additionally, the protocol should have these features:

* Is able to communicate securely in multi-user chats with a predetermined access list (e.g. groups). This might be simulated groups, with some method of transcript consistency.
* Is able to send and receive secure messages asynchronously (when the recipient is not online).

### Possible candidates

**OTR**

OTR is the current gold standard for chats with PFS, but it does not support asynchronous messages. Group support (mpOTR) is highly experimental and necessarily slow.

**OpenPGP**

OpenPGP is not often used for chat and it does not support PFS. However, if used with a long lived signing key and very short lived encryption keys that are disposed, OpenPGP can offer some degree of forward secrecy (albeit with a much larger session window than other schemes). OpenPGP has the advantage of supporting groups and offline messages.

**SCIMP**

SCIMP is the protocol used by SilentCircle chat. SCIMP has some improvements over OTR, but is more brittle than Axolotl. Current SCIMP specification leaks information that can be used to correlate traffic. There are no free or open source implementations of SCIMP. It does not support groups.

**Axolotl**

Axolotl is a new protocol from Trevor Perrin and Moxie Marlinspike. It has a key ratchet system that offers resilient PFS and can be made to work with asynchronous messages (fully PFS on the first message if the client publishes in advance a pool of ephemeral ECC public keys).

The current TextSecure implementation of Axolotl simulates groups by maintaining many 1:1 chat sessions. The chat transcripts are not yet validated to confirm that every user saw the same chat conversation, nor is there a way to manage the membership in a group.

**Other**

There are several other schemes for end-to-end XMPP message encryption, including Encrypted Session Negotiation (ESession) and XMPP Transport Layer Security (XTLS). In addition to not being used by anyone, they don't support both asynchronous and forward secret messages at the same time.

[dgoulet] So the mpOTR project (actually will be renamed but...) is ongoing and actually did some good progress recently after their summit in April 2014. This could be a good solution for group chat but might be problematic for instance in a high latency environment. Still, worth following their progress. One thing to consider here is that OTR has been around for 10 years and got way more scrutiny then Axolotl which addresses some missing use cases. In a perfect world, I would see this Newchat having all three methods of message encryption (OTR, Axolotl and mpOTR) and use the right one depending on the situation (low bandwidth, high latency, power limitation, group chat, etc...). This is easily doable nowadays by simply regurlarly detecting the ressourses. Example:

## Key Discovery and Validation

In many ways, proper validation of public keys is bedrock precondition for any message security: you cannot ensure confidentiality or integrity without confirming the authenticity of the other party. Unfortunately, current schemes for validating the authenticity of a user's public key have fallen out of favor as insecure, too difficult, or for leaking metadata (e.g. Certificate Authorities, the Web of Trust). Many so called "secure" chat systems, like Telegram or Skype, don't have any key validation at all, resulting in a system with no actual confidentiality (Telegram recently added manual fingerprint validation for some chats, but it is clumsy and optional).

Almost all the new "easy" security tools approach key validation by using Trust on First Use (TOFU), where keys are assumed to be valid when they are first seen. TOFU is much better than no validation, but has many limitations (although schemes that use TOFU can be considerably strengthened by layering additional checks, such as network perspective or endorsement).

Although there are many methods of solving the problem of binding a user identifier to a cryptographic key, none have emerged as a widely adopted standard.

Requirements:

* Auto discovery: The client application should be able to automatically discover the public keys associated with a particular identifier without user intervention.
* Auto validation: The client application should be able to automatically validate the authenticity of public keys, securely binding an identifier to the public key, without user intervention.
* Meta-data protection: The user should not need to leak to a third party what keys they are interested in.
* No spam, please: The user identifiers should not be globally enumerable.
* Global view: In contrast to the user identifiers, it is desirable to have a global view into all public user keys to ensure that any bogus authentications are made public.
* Auditability: Authentications by authenticating parties should be externally auditable by anyone (in other words, it should not be up to the user to be the sole auditor of the authentications made for their keys).

### Possible candidates

**DNSSEC with DANE**

DANE is a key discovery protocol over DNS that uses DNSSEC to allow the provider to securely endorse the results. DANE is an open standard, but it is unused, is troublesome to implement, and leaks metadata to a network observer.

**Nickym**

Nicknym is an experimental, lightweight REST-based discovery and validation protocol designed for this very problem. It uses a combination of TOFU, provider endorsement, and network perspective. The primary limitation with Nicknym is that it does not provide a global view, so ultimately it relies on the user's nicknym agent to report any bogus keys signed for the user. See https://leap.se/nicknym for more information.

**Namecoin variants**

An experimental p2p message system called Twister has solved this binding problem in a totally decentralized way by using a global append only log (borrowed from Bitcoin and Namecoin). This approach is very promising, but perhaps too experimental, resource intensive, and untested to be considered for Newchat at this time.

**Modified Certificate Transparency**

Although CT is not designed for service providers to sign user keys, it could be modified to support this. For example, CT logs could consist of a hashed user identifier with the corresponding public key. There is a paper that proposes to extend CT to support user keys and add better support for revocation ([Enhanced Certificate Transparency and End-to-end Encrypted Email](http://www.internetsociety.org/doc/enhanced-certificate-transparency-and-end-end-encrypted-mail)).

## Media Linking

Chat today is media rich. Participants need to be able to attach links, images, audio, video, and other files inline with their messages.

Requirements:

* Securely embed rich media in chat messages via external resource URLs that point to encrypted content.
* Only participants of the chat session may decrypt the content.
* A third party observer should not be able to identify who is participating in the chat by how the participants access the media associated with the chat.

### Candidates

**Naive HTTP**

As a first approximation, Newchat clients could simply embed an HTTP URL in the message and encrypt the content retrievable at that URL using the current session key. So long as all currently participants download the URL immediately, this will work fine and is easy to implement.

There are several problems with this approach:

* This approach is not particularly efficient. In each participant wants to retain the ability to access the media resource, they need to download and save a copy for themselves.
* If every client fetches the media resource, this will leak to a network observer who is participating in the chat. This information is leaked both to the media server (by logging who requests a particular resource), and to a network observer (by tracking the size of HTTP response).

**Federated Storage**

A better long term approach would be to have a federated cloud storage system. In this model, when a user posts a media link, this link can be used to obtain read (or read/write) access rights to a resource. For example, if the system was based on Tahoe-LAFS, each participant would be given a resource identifier and a copy of the capabilities key.

# Application Protocols

In order to achieve the desired usability and security, all Newchat clients will need support in the following areas.

* Secure authentication and signup
* Key management
* Data synchronization
* Group management
* Push notification

These protocols, and the libraries for them, are not needed for client interoperability, but they are required in order for a client to provide the kind of user experience that Newchat promises. Each Newchat client could use different libraries and protocols for each of these features.

## Secure authentication and signup

Traditional authentication systems based on username and password typically rely on the server having access to a cleartext copy of the password at some moment in time. Although this creates all sorts of security problems, username-plus-password is still by far the most easy and approachable method of user authentication.

Newchat has two competing promises:

1. All user data is encrypted on the client device
2. Users can authenticate using the familiar method of username-plus-password

The primary problem is that if the server is able to see the password in cleartext at any time in history, and this password is also used to unlock the keys needed for client-side encryption, then the user is back to essentially placing blind trust in the provider.

Requirements:

* Password authentication that does not leak the password to the service provider.
* New user registration
* User destruction
* Session management
* Support for smart-cards and other non-password methods of authentication

### Candidates

**SRP-based systems**

There is a class of cryptographic protocols called "zero-knowledge proofs" that will nicely solve this problem. By using a zero-knowledge proof, a user can authenticate themselves with the server without ever handing over a copy of their password for the server to hash (as is required in traditional username-plus-password authentication schemes). In particular, Secure Remote Password (SRP) is zero-knowledge system designed specifically to solve the username-plus-password authentication problem.

LEAP's Bonafide is a simple REST api that allows for authentication via SRP, account registration and deletion. After authentication, the client is given a simple session token that can be used to for any number of different services. Bonafide includes a reference client implementation in python and javascript. Currently, Bonafide uses a couchdb backend, but could be written to work with any database. Bonafide does not yet support smart-cards.

**Signature-based systems**

An alternate approach to the same problem is for the client device to upload a public key at the moment of account creation. To authenticate, the user's device is presented with a challenge by the server composed of random characters. If the client device can send back this challenge signed with the private signing key, and the server verifies this signature using the initially uploaded public key, then the user is granted a session token.

Sureshot includes a signature based registration and authentication scheme. The advantage of a signature scheme is that it is more resistant to offline brute force attempts to guess the password. The disadvantage is that the user must have access to their private key in order to authenticate.

**KDF-based systems**

For a suitable Key Derivation Function `kdf`, the client can generate two hashes from the password using `kdf(password, 1)` and `kdf(password, 2)`. The result of one can be used to authenticate with the server, and the result of the other can be used to unlock encrypted storage.

This system is by far the most simple, but does not support some of the features that SRP has, such as the ability of the user to also authenticate the server and the derivation of an ephemeral shared secret. Also, if the record kept on the server of the user's hash is ever leaked, then an attacker can trivially authenticate as that user.

## Key management

Any attempt to make public-key cryptography easy on the user will necessitate more advanced software logic to automate proper maintenance of keys. Key management is fairly straightforward, until something goes wrong. The tricky part of the key manager is deciding how to handle key revocation, resolving key conflicts, and replacing a key with a new version.

Requirements:

* Generate public/private key pairs.
* Manage a database of discovered and validated public keys of other users.
* Make keys securely available on multiple devices.
* Implement client support for Key Discovery and Validation protocol.
* Securely handle key revocation, key replacement, and key conflicts.

### Candidates

**LEAP's Key Manager**

The key manager in the Bitmask application works as a "nickagent" and uses the Nicknym protocol and OpenPGP keyservers. It currently supports most of these features, but does not handle revocation, key replacement, key conflicts, or validation via network perspective (planned, but not implemented).

## Data synchronization

Users today demand "data availability," the ability to access their data on multiple devices and to have piece of mind that there data will not be lost forever if they lose a device. For chat applications, data synchronization is needed for ensuring the availability of keys, contact rosters, media files, and chat history.

There are several proprietary synchronization services offered by companies like Apple, Google, Dropbox, Microsoft, and Amazon. These services are insecure and designed to lock the user into continued use of a particular data provider. For Newchat, clients will need something much better.

Requirements:

* Data should be efficiently synchronized with all the user's devices
* Data should be backed up in cloud storage in case a user loses their devices.
* Data should be client-encrypted at all times (when stored locally, and when backed up to the cloud).
* Some data should be synchronized only when requested (such as large files).

### Candidates

**LEAP's Soledad**

Soledad is a client-encrypted synchronized document database, with the ability to index fields and query structured documents. The data is kept synchronized with a server, and among the user's devices. It currently only works in Python. It currently uses HTTP, but this has proved to be very limiting and future development will focus on a more efficient synchronization transport.

**Mozilla's Firefox Sync**

Firefox Sync supports client-encrypted synchronization, but does not include a facility for search or secure local storage. To date, I don't think anyone other software besides Firefox uses Firefox Sync.

**Spideroak's Crypton**

Crypton currently works only in the web browser. Crypton has been heavily audited by third parties, and could be adopted outside the browser.

## Group management

XMPP has a concept of a private group chat, but the user must entirely trust the provider to properly maintain the list of group members. For secure group chat, Newchat will require an API for managing groups securely.

In the short term, it is probably sufficient define groups by simple consensus: Once a group is created, if any member of this group adds someone, then all the other participants accept this change. The only way to remove someone is if they decide to leave.

In the long term, this simple model is not sufficient to achieve the full functionality of groups found in popular chat applications.

Requirements:

* Ability to create and destroy groups.
* Ability to add and remove users from a group.
* Ensure all group members get updates to the group membership list.
* Ensure that only authorized member may modify the group membership list.
* Ensure the server cannot replay an old membership list.

Candidates:

* None, new development needed.

## Efficient push notifications

Typically, XMPP works by keeping a TCP connection open at all times. This works for desktop clients, but does not perform well on mobile devices or where network service is spotty. We need a library that can create an abstraction of the different push notification systems, including the proprietary ones native to android and IOs, as well as the open source alternatives.

In the short term, a solution in this area may not be necessary. Many mobile chat applications get away with very inefficient polling without driving away their users. In the long term, this is an important problem that needs to be solved.

### Candidates

**TextSecure**

The TextSecure server has support for push notification via WebSockets, as well as using the Google Cloud Messaging (GCM) and Apple Push Notification Service (APN).

