Single Tox Standard Draft v.0.0.2
===

###User Identification
- Users exchange a 76-character string that contains a public key, a nospam value, and a checksum to be used as an identifier. This is to be called the **Tox ID**. "User ID", "ID", and other derivatives are not valid and fail to follow the STS. 
- Users can set a **Nickname** or **Name** to present a quick way of identification once the friend request has been accepted. These names may be changed on-the-fly and do not alter the Tox ID in anyway. "Username", "Nick", and other derivatives are not valid and fail to follow the STS.
- Users may set a status, along with an optional message that allows for updates about life activities, current willingness to conversate, etc. This unit will be called the **Status**, and the three quick-statuses, usually selected via a drop-down menu, are to be  **"Online, Away, and Busy"** (Optional support for invisible). A field for a custom message must also be offered. *(UNDER HEAVY REVISION)*

###Friend Requests
There are three fields a Tox client must offer when a user goes to add another. While two of these fields are optional for the end-user to fill out, it is required that Tox clients provide all three fields in the UI. These are, and should be labeled as, as follows: <stuff here>


###Tox URI scheme (should be reworded)
The Tox URI scheme is as follows: tox://. A client must accept {CLIENT_NAME} tox://{PASSED}. A client must then check to see if this is a standard ID or DNS discovery ID. If this is a standard ID, a client should show the user the ID, asking if they want to add said ID, a negative response should close the client while a positive should add the ID. If a DNS discovery ID is detected, a client should resolve this and ask the user if he wants to add said ID, showing the ID and email. As before, a negative response should close the client while a positive should add the ID. On a malformed ID the client should alert the user, closing the client after acknowledgement.

###DNS Discovery (should be reworded)
A DNS discovery ID goes in the following format: user@domain. Users should not enter a DNS discovery ID in any way they don't normally add a Tox ID, clients should be able to figure out what is what. On adding a DNS discovery ID, a client must resolve a DNS TXT record for the value user._tox.domain. In this case the @ is replaced with _tox, allowing the use of subdomains while ensuring a record is a Tox record. The value of a Tox DNS record goes v=version;id=Tox ID, where the version is always tox1, and the ID corresponds to the user. Typical records lack spaces, though clients should be able to deal with oddly formatted cases. Clients are also encouraged to check DNSSEC, though this is not a requirement.

## Translation of STS Terminology
Client developers must choose translations that resemble the English variations as closely as possible, except in the case where the Tox trademark is being used. For example, Tox ID is to remain, untouched, in English. In the future, we will provide translations to be used in non-English Tox clients.

