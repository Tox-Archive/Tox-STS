Single Tox Standard Draft v.0.0.3
===
As Tox grows and more clients are created, we feel it is time to  create a Tox standard. Doing so will enable us to promote a single, cohesive brand despite the numerous clients. STS aims to offer a handbook for existing and potential client developers to follow so they can all remain consistent, yet unique in their own way. This will prevent confusion for users who wish to switch clients, and allow Tox to focus on pushing a single brand. At the moment, we are strongly recommending adherence to the STS, as it will foster a more productive environment, but we recognize that it’s the right of the developer to choose.


##User Identification & Interaction
###User Profiles
- Users exchange a 76-character string that contains a public key, a nospam value, and a checksum to be used as an identifier. This is to be called the **Tox ID**. "User ID", "ID", and other derivatives are not valid and fail to follow the STS. 
- Users can set a **Nickname** or **Name** to present a quick way of identification once the friend request has been accepted. These names may be changed on-the-fly and do not alter the Tox ID in anyway. "Username", "Nick", and other derivatives are not valid and fail to follow the STS.
- Users may set a status, along with an optional message that allows for updates about life activities, current willingness to conversate, etc. This unit will be called the **Status**, and the three quick-statuses, usually selected via a drop-down menu, are to be  **"Online, Away, and Busy"** (Optional support for invisible). A field for a custom message must also be offered. *(UNDER HEAVY REVISION)*

###Friend Requests
There are three fields a Tox client must offer when a user goes to add another. While two of these fields are optional for the end-user to fill out, it is required that Tox clients provide all three fields in the UI. These are, and should be labeled as, as follows: <stuff here>


##Tox URI scheme (should be reworded)
The Tox URI scheme is as follows: `tox://`. A client must accept `{CLIENT_NAME} tox://{PASSED}`. A client must then check to see if this is a standard ID or DNS discovery ID. If this is a standard ID, a client should show the user the ID, asking if they want to add said ID, a negative response should close the client while a positive should add the ID. If a DNS discovery ID is detected, a client should ask the user for the IDs pin if not provided. A client then follows DNS Discovery procedure to verify this, notifying the user if it is wrong. Afterwards, resolve this and ask the user if he wants to add said ID, showing the ID and email. As before, a negative response should close the client while a positive should add the ID. On a malformed ID the client should alert the user, closing the client after acknowledgement.

#####BNF of Tox URI Scheme
```
<uri-scheme> ::= "tox://" <tox-id>
```

Additionally, a query string may be attached to the URI, containing auxiliary information such as a pre-defined message, or a PIN (see DNS Discovery below). Clients **must** ignore any unknown key/value pairs.

| Key         | Support required? | Notes                                                              |
| ----------- | ----------------- | ------------------------------------------------------------------ |
| `message`   | Recommended       | Use this value to fill in the Message field of the friend request. |
| `pin    `   | Recommended       | Only in DNS Discovery format. If it is not a valid PIN for the version of the record, ignore it. |
| `x-name `   | No                | Use this value to pre-set a custom name for the added user.        |

It is recommended that client-specific query parameters prefix the key name with an "x" (e.g. `tox://james@nsa.gov?x-gov-employee-id=11925334`).

##DNS Discovery (should be reworded)
A DNS discovery ID goes in the following format: `user@domain`. Users should not enter a DNS discovery ID in any way they don't normally add a Tox ID, clients should be able to figure out what is what. On adding a DNS discovery ID, a client must resolve a DNS TXT record for the value `user._tox.domain`. In this case the `@` is replaced with `_tox`, allowing the use of subdomains while ensuring a record is a Tox record. Typical records lack spaces, though clients should be able to deal with oddly formatted cases. Clients are also encouraged to check DNSSEC, though this is not a requirement.

The `tox://` URI has 2 versions, tox1 and tox2. Tox2 attempts to stop dns request spamming and dns poisoning attacks by using the a form of the nospam as a unique pin. Use of tox1 is **depreciated**, and use is **strongly discouraged**.

In the case of multiple records clients should first look for the highest version record and default to that, this becomes the highest priority. In the case of multiple records of the sam priority, a client is free to choose the first one the dns lookup reported.

| Tox1        | Tox2          |
| ------------- |:-------------:|
| The value of a Tox DNS record goes `v=version;id=Tox ID`, where the version is tox1, and id is the users Tox ID. A client then follows the standard `tox://` scheme. Typical records lack spaces, though clients should be able to deal with oddly formatted cases.      | The value of a Tox DNS record goes `v=version;pub=public key;check=checksum`, where the version is tox2, pub is the public key, and the checksum is the XOR'd value of the nospam and public key. A client then asks the user for a pin, then it appends `==` to it and turns it from base64 to hexidacimal and XOR's this against the key, checking it against the checksum to verify. |


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

`<tox-id>` you get from toxcore, `<public-key>` and `<checksum>` can be taken from <tox-id> (more info [here](http://api.libtoxcore.so/core_concepts.html#the-tox-id)).

###Domain signing
Domain signing is an extension of DNS Discovery designed to further ensure DNS Discovery records have not been modified in transit or via existing DNS attacks. This becomes important with tox1 records where things like poisoning have not been mitigated. Domain signing works by appending an optional sign= to existing tox1 and tox2 records and turned in to base64 without the ==, where this is compared to the known signing key for a domain. Domain signing uses crypto_sign_ed25519 from NaCL to sign and verify records, and needs to be added to toxcore as a toxsign function for verifying. In tox1 the signature is of the ID, while in tox2 the signature is of the public key + checksum. With Domain signing, the public key is also stored in a txt record, using the format ```v=tox;pub={public key}```. Keep in mind, due to potential issues where the public key is the result of a poisoning attack, clients are encouraged to maintain a list of popular domains and keys. One such list is [here](http://wiki.tox.im/Domain_keys).


  
  
## Translation of STS Terminology
Client developers must choose translations that resemble the English variations as closely as possible, except in the case where the Tox trademark is being used. For example, Tox ID is to remain, untouched, in English. In the future, we will provide translations to be used in non-English Tox clients.
