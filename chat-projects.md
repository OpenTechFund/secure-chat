# Existing Chat Projects and Protocols

Does the world really need another secure chat project? Despite the large number of existing projects, none of these projects are positioned meet the "new chat" experience of popular applications such as WhatsApp, WeChat, Kik Messenger, etc.

## Open source projects

**TextSecure**

Advantages:

* Advanced Axolotl message encryption.
* Cross platform iOS and Android.
* Asynchronous messaging with PFS.
* Strong public key cryptography that is invisible to the user (it uses Trust on First Use to validate keys).
* Leverages your existing mobile contact list.

Disadvantages:

* TextSecure uses your phone number as a unique identifier. This prevents you from ever using aliases or pseudo-anonymous identifiers, and puts the user at the mercy of the telco.
* TextSecure does not support federation (outside a small set of pre-approved routing organizations).

Plans are underway to add the option of using an email address as an identifier and extending the protocol to support a more open federation.

**ChatSecure**

ChatSecure is a nice, general purpose XMPP client with OTR support for Android. It is a fine example of "old chat", and could be modified to provide a "new chat" experience.

**Telegram**

Despite Telegram's claim of using advanced super-secure encryption, their design essentially rests on trusting the server for everything. The user can have "secure" chats by manually verifying awkward keys, but it is unlikely that anyone does. Telegram should be considered snake-oil, albeit snake-oil with a lot of money behind it.

**P2pChat**

Peer-to-peer approaches like P2pChat (and the proprietary Bittorrent Chat) have the key advantage of not relying any third parties, but they suffer when it comes to the kind of user experience Newchat is trying to achieve:

Disadvantages:

* user identifiers are random and hard to remember
* there is no data availability, because nothing is backed up to the cloud
* there are problems with responsiveness and scalability, because of a reliance on spotty peer-to-peer networks instead of stable provider-to-provider networks.
* it is difficult to switch devices
* it can be resource intensive for a mobile device
* direct device to device message routing leaks association meta-data to a network observer

Advantages:

* There is no need to validate keys because user identifiers are the keys themselves.
* There is no need to trust any third party for anything.

**CryptoCat**

CryptoCat consists of two primary elements: one is the web-based chat application (typically loaded via a browser extension), and the other is the cryptocat protocol for encrypted group chat. The future plan for CryptoCat is to switch from a custom protocol to using mpOTR (multi-party Off-The-Record). Cryptocat currently uses OTR for two party chats.

Advantages:

* Very low barrier to entry, just open the web page or install the browser extension.

Disadvantages:

* No support for offline messages.
* The browser environment offers limited security.
* Public key authentication is awkward and identities are not stable.

**Surespot**

Surespot is an open source mobile messaging app designed to work with a single provider (it is not federated).

* Uses a custom chat encryption protocol. So far, no outside analysis.

Advantages:

* At first look, it seems to have reasonable security.

Disadvantages:

* Is not federated, and will only work with other surespot users. One could fork the code and host their own provider, but these users would not be able to talk with the surespot users.

## Proprietary projects

**Proprietary apps, proprietary protocols**

Recently, many proprietary secure chat applications have been released. These approaches all suffer from some common problems:

* The software and the service come from the same organization. Ultimately, despite the fancy crypto, it all boils down to trusting the organization.
* Key validation requires trusting the central organization.
* There is no interoperability with other providers.

These include:

* SilentCircle
* [Threema](https://threema.ch)
* Seecrypt
* Wickr
* Perzo
* Silent
* Wisper
* [Gliph](https://gli.ph)

**Proprietary apps, open protocols**

Heml.is is a proprietary chat application that uses open protocols (XMPP and OpenPGP). However, it does not support federation (single provider model). Ultimately, like other single-provider models, the user must place trust in the provider.

**Proprietary apps, p2p protocols**

Bittorrent Chat is a p2p-based chat application that uses bittorrents DHT to find other users. Messages are relayed direct from IP to IP. Users are identified by their public key. This has all the advantages and disadvantages of other p2p-based chat approaches (see above).
