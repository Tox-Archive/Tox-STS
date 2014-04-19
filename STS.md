Single Tox Standard Draft v.0.1.0
===
As Tox grows and more clients are created, we feel it is time to  create a Tox standard. Doing so will enable us to promote a single, cohesive brand despite the numerous clients. STS aims to offer a handbook for existing and potential client developers to follow so they can all remain consistent, yet unique in their own way. This will prevent confusion for users who wish to switch clients, and allow Tox to focus on pushing a single brand. At the moment, we are strongly recommending adherence to the STS, as it will foster a more productive environment, but we recognize that it’s the right of the developer to choose.

##Table of Contents

1. User Identification & Interaction
  * [User Profiles](#user-identification--interaction)
  * [Friend Requests](#user-profiles)
  * [Friends List](#friends-list)
  * [Group Messaging](#group-messaging)
  * [Multimedia Messaging](#multimedia-messaging)
  * [Emotes](#emotes)

2. User Safety
  * [User Profile Encryption](#user-profile-encryption)
  * [NoSpam Changing](#nospam-changing)
3. User Discovery
  * [Tox URI Scheme](#tox-uri-scheme)
  * [DNS Discovery](#dns-discovery)


##User Identification & Interaction
###User Profiles
- Users exchange a 76-character string that contains a public key, a nospam value, and a checksum to be used as an identifier. This is to be called the **Tox ID**.
- Users can set a **Name** which offers a simpler means of identification once the friend request has been accepted. These names may be changed on-the-fly and do not alter the Tox ID in any way.
- Users may set a status, along with an optional message that allows for updates about life activities, current willingness to conversate, etc. This unit will be called the **Status**, and the three quick-statuses, usually selected via a drop-down menu, are to be  **"Online, Away, and Busy"** (Optional support for invisible). A field for a custom status message must also be offered.

###Friend Requests
There are three fields a Tox client must offer when a user goes to add another. While two of these fields are optional for the end-user to fill out, it is required that Tox clients provide all three fields in the UI. These are, and should be labeled as, as follows:

| Type | Requirement | Description
| --- | --- | ---
| **Tox ID** | Required | The user can paste a Tox ID into the field, or, alternatively, if the user is on a mobile device, the option to use scan a QR code should also be offered
| **Nickname** | Optional | The user can set a custom nickname for the friend they're about to add  
| **Message** | Optional | The user can send a custom message to be displayed to the friend they're adding, either for identification purposes, or anything else.

###Friends List
Each client should include a way to manage friend lists, including the ability to edit Nicknames, view last time a user has logged on, and perform various actions, such as friend deletion, etc.

###Group Messaging
Tox allows for group messaging, where users may join a specified Tox ID and will be able to communicate in one "room". These are to be referred to as **Groupchats**. Clients will have a lot of leeway when it comes to the implementation of Groupchats, as each will offer a unique approach to it. However, they are a basic necessity and are required to be implemented, whatever the approach may be.

###Multimedia messaging
The Tox Core allows for encrypted video and audio calling, as a well file sharing. All three services must be implemented to be considered STS-compliant, but as of writing, no client has all three. Implementing video is a difficult task, and in acknowledgement of this, the Single Tox Standard suggests a 'best-effort' approach to compliance with this section.

###Emotes
Emotes pack used in clients should be at most a 1:1 map of Unicode ones: http://pastebin.com/raw.php?i=n8Vgy352
There cannot be an emote implemented that doesn't have an unicode correspondent.
Emotes should have identical commands for when you change clients. :-) for a "happy face", for example, should be standardized (full list of this to be created)


##User Safety

###User Profile Encryption
In order to prevent the threat of local data theft, all Tox clients should, however the method or implementation (including choice of crypto), provide a method to encrypt all local data. This is not STS-required, but heavily suggested in order to keep the users of Tox safe. (under discussion)

###NoSpam Changing
All Tox IDs have attached a small NoSpam Key to prevent friend request spam. All clients should include a quick method of changing the NoSpam Key in the event of spam. This should never be done automatically, and should required explicit action from the user. For more information on what the NoSpam Key is, and what it does, visit our [API Docs](http://api.libtoxcore.so)


##User Discovery
###Tox URI scheme
The Tox URI scheme is as follows: `tox://`. A client must accept `{CLIENT_NAME} tox://{PASSED}`. A client must then check to see if this is a standard ID or DNS discovery ID. If this is a standard ID, a client should show the user the ID, asking if they want to add said ID. A negative response should close the client (unless the client was already open prior to the URI event), while a positive response should add the ID. If a DNS discovery ID is detected, a client should ask the user for the IDs pin if not provided. A client then follows DNS Discovery procedure to verify this, notifying the user if it is wrong. Having resolved this, the user should be asked whether he wishes to add the ID, showing the ID and email. As before, a negative response should close the client while a positive should add the ID. If the ID is malformed, the client should alert the user, closing the client after acknowledgement.

#####BNF of Tox URI Scheme
```
<uri-scheme> ::= "tox://" <tox-id> "?" <key> = <value> "&" <key> = <value>
```
Note that information after the <tox-id> is optional.

Additionally, a query string may be attached to the URI, containing auxiliary information such as a pre-defined message, or a PIN (see DNS Discovery below). Clients **must** ignore any unknown key/value pairs.

| Key         | Support required? | Notes                                                              |
| ----------- | ----------------- | ------------------------------------------------------------------ |
| `message`   | Recommended       | Use this value to fill in the Message field of the friend request. |
| `pin    `   | Recommended       | Only in DNS Discovery format. If it is not a valid PIN for the version of the record, ignore it. |
| `x-name `   | No                | Use this value to pre-set a custom name for the added user.        |

It is recommended that client-specific query parameters prefix the key name with an "x" (e.g. `tox://james@nsa.gov?x-gov-employee-id=11925334`).

A set of URIs to test your implementation against can be found [here](https://kirara.ca/toxurl_testcases.html).

###DNS Discovery
A DNS discovery ID goes in the following format: `user@domain`. Users should not enter a DNS discovery ID in any way differently to adding a Tox ID - clients should be capable of identifying whether the input is a DNS discover ID or a Tox ID. On adding a DNS discovery ID, a client must resolve a DNS TXT record for the value `user._tox.domain`. In this case the `@` is replaced with `_tox`, allowing the use of subdomains while ensuring a record is a Tox record. Typical records lack spaces, though clients should be able to deal with oddly formatted cases. Clients are also encouraged to check DNSSEC, though this is not a requirement.

The `tox://` URI has 2 versions, tox1 and tox2. Tox2 attempts to stop dns request spamming and dns poisoning attacks by using the a form of the nospam as a unique pin. Tox1 ignores these issues by listing just the ID. Use of Tox1 for records of users is **strongly advised against**, as this version is reserved **solely for clients that do not interface with a human**, who need a well known and easy to add ID. With this in mind, tox1 still serves the niche case of services like GroupBot where a pin isn't needed.

In the case of multiple records, clients should first look for the highest version record and default to that, with this becoming the highest priority. In the case of multiple records of the same priority, a client is free to choose the first one reported by the DNS lookup.

| Tox1        | Tox2          |
| ------------- |:-------------:|
| The value of a Tox DNS record goes `v=version;id=Tox ID`, where the version is tox1, and id is the users Tox ID. A client then follows the standard `tox://` scheme. Typical records lack spaces, though clients should be able to deal with oddly formatted cases.      | The value of a Tox DNS record goes `v=version;pub=public key;check=checksum`, where the version is tox2, pub is the public key, and the checksum is the XOR'd value of the nospam and public key. A client then asks the user for a pin, then it appends `==` to it, converts it from base64 to hexadecimal, and XOR's this against the key, checking it against the checksum to verify. |


#####BNF of Tox DNS URI Scheme v1 and v2

```
<dns-uri-scheme> ::= "tox://" <user> "@" <domain>
```

#####BNF of DNS Record v1 and v2

```
<dns-record-format> ::= <dns-record-type> " " <dns-record-name> "=" <dns-record-value>
<dns-record-type> ::= "TXT"
<dns-record-name> ::= <user> "._tox." <domain>
<dns-record-value> ::= "v=" <dns-record-version-1> ";" "id=" <tox-id> | "v=" <dns-record-version-2> ";" "pub=" <public-key> ";" "check=" <checksum>
<dns-record-version-1> ::= "tox1"
<dns-record-version-2> ::= "tox2"
```

`<user>` and `<domain>` you get from the URI,

`<tox-id>` is obtained from toxcore, `<public-key>` and `<checksum>` can be taken from <tox-id> (more info [here](http://api.libtoxcore.so/core_concepts.html#the-tox-id)).

####Domain signing
Domain signing is an extension of DNS Discovery designed to further ensure DNS Discovery records have not been modified in transit or via existing DNS attacks. This becomes important with tox1 records where things like poisoning have not been mitigated. Domain signing works by appending an optional sign= to existing tox1 and tox2 records with a signature of the data of the record where this is compared to the known signing key for ao domain. Domain signing uses crypto_sign_ed25519 from NaCL to sign and verify records.

- The signed data for Tox1 is the name of the record before the _tox + the id

- The signed data for Tox2 is the name of the record before the _tox + the public key + the ID.

All data is signed in bytes, to preserve space.

To validate this data a client would take the looked up user before the _tox and append it to the bytes version, comparing this to the output of the verifying function in NaCl. If this data is the same, the record is valid.

With Domain signing, the public key is also stored in a txt record, using the format ```v=tox;pub={public key}```. It is important to note that due to potential issues wherein the public key may be the result of a poisoning attack, clients are encouraged to maintain a list of popular domains and keys. One such list is [here](http://wiki.tox.im/Domain_keys).

##


## Translation of STS Terminology
Client developers must choose translations that resemble the English variations as closely as possible, except in the case where the Tox trademark is being used. For example, Tox ID is to remain, untouched, in English. In the future, translations will be provided for used i non-English Tox clients.
