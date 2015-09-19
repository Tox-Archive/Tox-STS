Single Tox Standard Draft v.0.1.1
===
As Tox grows and more clients are created, we feel it is time to  create a Tox standard. Doing so will enable us to promote a single, cohesive brand despite the numerous clients. STS aims to offer a handbook for existing and potential client developers to follow so they can all remain consistent, yet unique in their own way. This will prevent confusion for users who wish to switch clients, and allow Tox to focus on pushing a single brand. At the moment, we are strongly recommending adherence to the STS, as it will foster a more productive environment, but we recognize that it's the right of the developer to choose.

##Table of Contents

1. User Identification & Interaction
  * [User Identification](#user-identification--interaction)
  * [Friend Requests](#friend-requests)
  * [Friends List](#friends-list)
  * [Group Messaging](#group-messaging)
  * [Multimedia Messaging](#multimedia-messaging)
  * [File Transfers](#file-transfers)
  * [File Resuming](#file-resuming)
  * [Emoticons](#emoticons)
  * [Message splitting](#message-splitting)
  * [Encodings](#encodings)
2. Client paths
  * [Tox data directory](#tox-data-directory)
  * [Logging](#logging)
3. [User profiles](#user-profiles)
  * [Profile management](#profile-management)
  * [Recommendations](#recommendations-regarding-profile-management)
4. User Safety
  * [User Profile Encryption](#user-profile-encryption)
  * [NoSpam Changing](#nospam-changing)
5. [Avatars](#avatars)
  * [No support for avatars](#no-avatars)
6. User Discovery
  * [Tox URI Scheme](#tox-uri-scheme)
  * [DNS Discovery](#dns-discovery)
7. Customization
  * [Themes Support](#themes-support)


##User Identification & Interaction
###The Tox ID
- Users exchange a 76-character string containing hexadecimal representations of a public key, a nospam value, and a checksum to be used as an identifier. This is to be called the **Tox ID**.
- Users may set a **Name** which offers a simpler means of identification once the friend request has been accepted. These names may be changed on-the-fly and do not alter the Tox ID in any way.
- Users may set a **Status**. The three statuses are **"Online, Away, and Busy"**.
- Users may set a **Status message** , which is an arbitrary string of text that allows for updates about life activities, current willingness to converse, etc.

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
A/V support is mandatory in all clients unless the platform does not support it. (EX: video support isn't necessary for CLI clients.) Developers should strive to reduce delay as much as possible. Clients that have trouble keeping audio and video in perfect sync with each other or that add unacceptable latency to audio or video are not acceptable.

###File Transfers
The ability to send and receive files is _a required feature for any useful instant messaging client_. Toxcore will handle all of your data transfer needs. All you have to do is provide the data, and any helpful information to pass along to your friend. At the very least, you'll need to give Toxcore:
 * The `friend_id` you want to send the file to.
 * The `kind` of data you want to send. 
 * The `size` of data you plan to send. 
 * A `name` for the file you're sending, along with the `length` of the name string
(don't include the NULL term in the length, but do include one!).
 
Optionally, you can also provide a `file_id`, and an int pointer if you'd like an error code if something goes wrong.  
The API function looks like this:
```c
tox_file_send(tox, friend_number, TOX_FILE_KIND_DATA, file_size, file_id, filename, filename_length, error);
```

If the file transfer completes, the client **may** delete all stored info about the file. A file completes when the receiver has received it completely as such there is no need to try resuming completed files so any other info can be deleted, at the clients discretion. Toxcore will make one final callback to the chuck request function with a length of 0. At that point Toxcore considers the file completed, and purges any information about the file transfer and will reuse the file transfer number.

###File Resuming
Good clients support file resuming. Great clients do so even across client/core restarts!  

Toxcore does **not** keep data that might have expired. So file transfers for a specific friend are purged from core _whenever that friend goes offline/disconnects_. Toxcore also makes **no** attempt to save file transfer info in the core data file. This means that it's the sole responsibility of **the client** to index, save, and restore file transfer information. Clients **must** take care of resuming file transfers using the same `file_id` parameter and make use of the seek api to avoid duplicating transfered data. If you don't want to create your own `file_id`, Toxcore will generate one for you that you can retrieve with:  
```c
tox_file_get_file_id(tox, friend_number, file_number, new_id, error);
```

You're always welcome to supply a `file_id` yourself, but the _general recommendation_ is to let Toxcore generate one for you. (You can do this by setting file_id to NULL).  

When you're ready to resume a broken/interrupted transfer, you'll need to start the transfer **from** the original sending client. The sending client will save the `file_id` (`tox_file_get_file_id()`) along with file data/location, and any other useful information that your client may need to restart the file. Again, this information **must** be saved in long term by the client, because Toxcore will **not** save anything for you.  

If the client is closed and reopened it will reload all the saved file info from disk.  
The expectation of clients is that every time a friend comes online, the client will restart any pending transfers. Restarting a transfer is the same as starting a new transfer with the exception of the `file_id` parameter. For resumed transfers it **MUST NOT BE NULL**! Again Toxcore has **no** information about this file and will generate a **NEW** and **UNIQUE** `file_id` one that will **not** match the original. If the `file_id` changes the receiving client will have **no** means of differentiating it from a new transfer.  

Complient clients will first check if it has any incomplete files with the same `file_id`. If it does, it will open the incomplete file and look at how much data was received already, and use the following code with the 4th parameter equal to the number of bytes **already** received:  
```c
tox_file_seek(tox, friend_number, file_number, size_transferred, error)
```

It will then accept the transfert by using:
```c
tox_file_control(tox, friend_number, file_number, TOX_FILE_CONTROL_RESUME, error)
```

_It's up to your client_ if you want to prompt the user to **RE**-accept the transfer, or just resume the transfer automatically for the user. See also Resuming Suggestions.  

When a receiving file completes, all stored info related to the file transfer _can_, and **should** be deleted, (**not** the file transfer data it self, only the data associated with transferring that data). **Doing so will prevent future conflicts new/different transfers.**

###Emoticons
While toxcore supports Unicode, user support varies.
Sending clients, and only the sending clients, are responsible for transforming recognized text sequences into Unicode emoji characters.  

Receiving clients shall display the received text without applying further transformations to received text. If Unicode emoji characters are not supported on the receiving side, it's time to invest in a better text rendering system.

###Message splitting
Clients should split messages at the last whitespace character closest to ``TOX_MAX_MESSAGE_LENGTH`` (Currently 1368) characters. Whitespace character(s) are(is) `` `` (most symbols end with a space). An example of this follows:
- user enters over ``TOX_MAX_MESSAGE_LENGTH`` characters in a text box.
- Text box follows with this unless it has some restrictions or breaks the UI scaling.
- Client splits message at ``TOX_MAX_MESSAGE_LENGTH`` and searches backwards for the next whitespace character.
- The data between the start of the set of bytes and the whitespace character are sent
- The data after the whitespace character is appended to the rest of the message
- the client repeats step 3 until the bytes left is under ``TOX_MAX_MESSAGE_LENGTH``
- The UI on the sender would show a new sent message for each broken section so they are aware.
- The receiving client gets multiple separate messages like normal and displays them normally.

###Encodings
All text sent over Tox should be encoded in UTF-8. While clients can not expect that all data they receive is encoded in UTF-8, they should do one of the following when receiving non-UTF-8 text:
- Discard the received text completely
- Attempt to guess the encoding and re-encode as UTF-8 before showing the text to the user.
If a user enters non-UTF-8 text, client should re-encode them as far as possible, using the Unicode replacement character (U+FFFD) for characters they cannot encode.

##Client paths/formats
###Tox data directory

The path for Tox data on Windows is ``%APPDATA%/tox/`` (should this be roaming app data?)

The path for Tox data on Linux is ``~/.config/tox/`` 

This was chosen to work with as many existing clients as possible while allowing users to switch clients easily without loosing friends and IDs.


###Logging

***Discussion in progress, in case where you want to contribute to it, please comment in relevant [issue #56](https://github.com/Tox/Tox-STS/issues/56).***

##User Safety

###User Profile Encryption
In order to prevent the threat of local data theft, all Tox clients should, however the method or implementation (including choice of crypto), provide a method to encrypt all local data. This is not STS-required, but heavily suggested in order to keep the users of Tox safe. (under discussion)

###NoSpam Changing
All Tox IDs have attached a small NoSpam Key to prevent friend request spam. All clients should include a quick method of changing the NoSpam Key in the event of spam. This should never be done automatically, and should require an explicit action from the user. For more information on what the NoSpam Key is, and what it does, visit our [API Docs](https://libtoxcore.so)


##User profiles

The Tox ID and its associated friends list and save file is called a profile. Save files should use the ".tox" extension to denote that they are Tox save files; clients should recognize .tox save files, preferably via integration with the operating system. That way, merely by double clicking a .tox file, the client will import the file for use. Here, import means copy the file into the data directory (see "Tox data directory" below). When talking about a profile's name, we mean the base name of the file.

###Profile management

Clients should be able to track more than one Tox ID (i.e., its save file) in the same data directory. Clients should thus offer the ability to switch profiles (i.e., disconnect from the network, load a different profile, and reconnect), as well as import, rename, export, and delete profiles. These operations are relative to the data directory: importing means copying a file to the data directory, exporting means allowing the user to save a profile in a location of their choice, and deleting means removing the profile from the data directory. Together, these operations make it easy for a user to share a Tox ID and friend's list among whatever devices he or she uses.

###Recommendations regarding profile management

Clients should encourage good user habits:

1. Warn the user when they are about to export an unencrypted profile
2. Warn the user when they are about to delete a profile currently in use

Clients on operating systems where accessing the file system directly is difficult should integrate with that OS's means of sharing files, by e.g. offering to attach a profile to an email (where warning 1 applies).

## Avatars

__Clients should offer support for avatars, but it is not mandatory.__

If your client does not have support for avatars, look [here](#no-avatars).


Clients should allow user to have own avatar and see friend's avatars.

Avatars are PNG images with a maximum size of (2^16) bytes, enforced by client.

Avatars are sent as a file transfer with ``TOX_FILE_KIND_AVATAR``. Every time a friend goes offline you must send them a new avatar transfer. The ``file_id`` will be set to the hash of the avatar. If no avatar is set, the size of the avatar will be set to zero and the ``file_id`` will be set to ``NULL``. If the avatar transfer is cancelled by the friend, the other client must assume the friend already has the avatar and will not try to send it again, as long as friend will stay online. Upon friend going offline and back online, client must try to send avatar again.

If the avatar is changed, a new avatar transfer must be started to all online friends.

When receiving a new file transfer with type ``TOX_FILE_KIND_AVATAR``:
* check whether size is 0
* if it is, cancel the file transfer and remove avatar of the friend if it was set
* if the size is non-zero:
  - check whether hash of the current avatar of a friend matches the ``file_id`` for the incoming avatar transfer
  - if matches, the client will cancel the transfer
  - if it doesn't, the client will accept the file transfer


It is desirable that the user avatar and the cached friends avatars could be shared among different Tox clients in the same system. This not only makes switching from one client to another easier, but also minimizes the need of data transfers, as avatars already downloaded by other clients can be reused.

* Avatars are stored in a directory called ``avatars`` and named as ``xxxxx.png``, where ``xxxxx`` is the complete public key (but **not** friend address!), encoded as an uppercase hexadecimal string and ``png`` is the extension for the PNG avatar. As new image formats may be used in the future, clients should ensure that no other file ``xxxxx.*`` exists. No file should be kept for a user that has no avatar.
* The client's own avatar is not special and must be stored like any other. This is partially for simplicity, and partially in anticipation of profiles.
* The avatar should be stored as it is received, without any modifications done by client for display purposes.

Example for Linux and other Unix systems, with an user called ``gildor``:
```
Tox data directory: /home/gildor/.config/tox/
Tox data file:      /home/gildor/.config/tox/tox_save
Avatar data dir:    /home/gildor/.config/tox/avatars/
Gildor's avatar:    /home/gildor/.config/tox/avatars/446F4E6F744D6564646C65496E546865416666616972734F6657697A61726473.png
Elrond's avatar:    /home/gildor/.config/tox/avatars/43656C65627269616E20646F6E277420546F782E426164206D656D6F72696573.png
Elladan's avatar:   /home/gildor/.config/tox/avatars/49486174655768656E48756D616E735468696E6B49416D4D7942726F74686572.png
Elrohir's avatar    /home/gildor/.config/tox/avatars/726568746F7242794D6D41496B6E696854736E616D75486E6568576574614849.png
```

<a name="no-avatars" />
### No support for avatars

In case where client doesn't support avatars, all file transfers ``TOX_FILE_KIND_AVATAR`` **must** be canceled.



##User Discovery
###Tox URI scheme
The Tox URI scheme is as follows: `tox:`. A client must accept `{CLIENT_NAME} tox:{PASSED}`. A client must then check to see if this is a standard ID or DNS discovery ID. If this is a standard ID, a client should show the user the ID, asking if they want to add said ID. A negative response should close the client (unless the client was already open prior to the URI event), while an affirmative response should add the ID. If a DNS discovery ID is detected, a client should resolve this. A client then follows DNS Discovery procedure to verify this, notifying the user if it is wrong. Having resolved this, the user should be asked whether he wishes to add the ID. As before, a negative response should close the client while a positive should add the ID. If the ID is malformed, the client should alert the user, closing the client after acknowledgement.

#####BNF of Tox URI Scheme
```
<uri-scheme> ::= "tox:" <tox-id>
<uri-scheme> ::= "tox:" <tox user>@<domain>
```

###DNS Discovery
A DNS discovery ID goes in the following format: `user@domain`. Users should not enter a DNS discovery ID in any way differently to adding a Tox ID - clients should be capable of identifying whether the input is a DNS discovery ID or a Tox ID. On adding a DNS discovery ID, a client must resolve a DNS TXT record for the value `user._tox.domain.`. Note the end dot. In this case the `@` is replaced with `_tox`, allowing the use of subdomains while ensuring a record is a Tox record. Typical records lack spaces, though clients should be able to deal with oddly formatted cases. Clients are also encouraged to check DNSSEC, though this is not a requirement.

The `tox:` URI has 3 versions, tox1, tox2, and tox3. Tox3 is encrypted, though doesn't work with standard DNS servers and is hard to setup. Tox2 attempts to stop DNS request spamming and DNS poisoning attacks by using the a form of the nospam as a unique pin. Tox1 ignores these issues by listing just the Tox ID.

In the case of multiple records of different versions, clients should prioritize the highest version record. If there are more than one record of the same version, a client should use the first one in the order reported by the DNS lookup.

| Tox1        | Tox2          |
| ------------- |:-------------:|
| The value of a Tox DNS record goes `v=version;id=Tox ID`, where the version is tox1, and id is the users Tox ID. A client then follows the standard `tox:` scheme. Typical records lack spaces, though clients should be able to deal with oddly formatted cases.      | The value of a Tox DNS record goes `v=version;pub=public key;check=checksum`, where the version is tox2, pub is the public key, and the checksum is the XOR'd value of the nospam and public key. A client then asks the user for a pin, then it appends `==` to it, converts it from base64 to hexadecimal, and XOR's this against the key, checking it against the checksum to verify. |


#####BNF of Tox DNS URI Scheme v1 and v2

```
<dns-uri-scheme> ::= "tox:" <user> "@" <domain>
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

#####Tox3 details
Tox3 uses toxdns in clients to do encryption/decryption. Clients get the server public key used in tox3 from ``_tox.<domain>``. This is in the following format: ```{public key}```

######raw details:
request:
[4 byte nonce][temporary client public key][encrypted with crypto_box(temporary client public key, server long term public key, 4 byte nonce + 20 byte zeros)[queried username]]
 
-> base32 (a to z, 0 to 5) -> separate with . as needed.
 
Notes:
- the temporary public key is a public key temporarily generated that is discarded right after the session is.
- 4 byte nonce must be increased by 1 on every request (it must not be generated randomly because birthday problem)
- every 2^32 requests (if that ever happens) the temporary client public key must be changed 
(two requests with the same nonce must never ever happen.)
 
response:
DNS TXT record:
[encrypted with crypto_box(temporary client public key, server long term public key, 4 byte nonce(sent in request) + 1 byte equal to 1 and 19 bytes of zeros (this is the same 4 byte nonce that was sent in the request))[Tox id (binary format)]]
 
-> base32 (same as above)
 
Possible issues with this:
- no PFS (find some way of making the server change its key every couple of days maybe?)
- clients need a local list of tox dns server long term public keys.

####Domain signing
Domain signing is an extension of DNS Discovery designed to further ensure DNS Discovery records have not been modified in transit or via existing DNS attacks. This becomes important with tox1 records where things like poisoning have not been mitigated. Domain signing works by appending an optional sign= to existing tox1 and tox2 records with a signature of the data of the record where this is compared to the known signing key for a domain. Domain signing uses crypto_sign_ed25519 from NaCl to sign and verify records.

- The signed data for Tox1 is the name of the record before the _tox + the Tox ID

- The signed data for Tox2 is the name of the record before the _tox + the public key + the check.

All data is signed in bytes to preserve space.

To validate this data a client would take the looked up user before the _tox and append it to the bytes version, comparing this to the output of the verifying function in NaCl. If this data is the same, the record is valid.

With Domain signing, the public key is also stored in a TXT record, using the format ```v=tox;pub={public key}```. It is important to note that due to potential issues wherein the public key may be the result of a poisoning attack, clients are encouraged to maintain a list of popular domains and keys. One such list is [here](http://wiki.tox.im/Domain_keys).

## Customization
####Themes support
Clients **should** have a way to change their appearance, some suggested themes are:
 * a light theme.
 * a dark theme (or a night theme).
 * a high contrast theme.

Of course the more there are, the better.

## Translation of STS Terminology
Client developers must choose translations that resemble the English variations as closely as possible, except in the case where the Tox trademark is being used. For example, "Tox ID" is to remain, untouched, in English. In the future, translations will be provided for non-English Tox clients.
