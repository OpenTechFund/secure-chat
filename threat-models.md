# Threat models

Thread models for end-to-end encrypted chat.

Adapted from: https://gist.github.com/Geal/8228049

## Threat model goals

In the short term, the goal with Newchat is to be resistant to the following attacks:

* attacker reads messages on the wire
* attacker modifies messages on the wire
* attacker drops messages going to other participants
* attacker drops messages coming from other participants
* attacker sends messages to a conversation (not a participant)
* attacker obtains the encryption key for one message
* attacker obtains the encryption key(s) for multiple messages
* attacker obtains a shared session key of two participants
* attacker obtains all the previous session keys of two participants at a certain point
* attacker obtains all the previous session keys of all participants at a certain point
* attacker sends a message on behalf of a user
* attacker replays a message
* attacker sends different messages to different participants
* attacker asks for a rekeying
* attacker adds multiple users
* attacker adds a lot of bogus users
* attacker connects to the conversation, and tries to impersonate a previous participant

In the long term:

* attacker discovers one or more participants in a conversation
* attacker splits the conversation in two or more sets of participants

## Threats that are out of scope

These attacks might be too difficult to address, except in the long term:

* attacker observes message size on the wire
* attacker delays messages to other participants
* attacker delays messages from other participants
* attacker prevents a participant from joining a conversation
* attacker prevents a participant from following the conversation
* attacker prevents multiple participants from following the conversation
* attacker abruptly disconnects participants
* attacker sends a lot of bogus messages
* attacker sends a lot of valid messages (ex: replays? attacker is a participant?)
* participant attacker asks for a rekeying
* participant attacker floods the channel
