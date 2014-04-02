Single Tox Standard Draft v.0.0.3
===

###User Identification
- Users exchange a 76-character string that contains a public key, a nospam value, and a checksum to be used as an identifier. This is to be called the **Tox ID**. "User ID", "ID", and other derivatives are not valid and fail to follow the STS. 
- Users can set a **Nickname** or **Name** to present a quick way of identification once the friend request has been accepted. These names may be changed on-the-fly and do not alter the Tox ID in anyway. "Username", "Nick", and other derivatives are not valid and fail to follow the STS.
- Users may set a status, along with an optional message that allows for updates about life activities, current willingness to conversate, etc. This unit will be called the **Status**, and the three quick-statuses, usually selected via a drop-down menu, are to be  **"Online, Away, and Busy"** (Optional support for invisible). A field for a custom message must also be offered. *(UNDER HEAVY REVISION)*

###Friend Requests
There are three fields a Tox client must offer when a user goes to add another. While two of these fields are optional for the end-user to fill out, it is required that Tox clients provide all three fields in the UI. These are, and should be labeled as, as follows: <stuff here>


###Tox URI scheme (should be reworded)
The Tox URI scheme is as follows: `tox://`. A client must accept `{CLIENT_NAME} tox://{PASSED}`. A client must then check to see if this is a standard ID or DNS discovery ID. If this is a standard ID, a client should show the user the ID, asking if they want to add said ID, a negative response should close the client while a positive should add the ID. If a DNS discovery ID is detected, a client should ask the user for the IDs pin if not provided. A client then follows DNS Discovery procedure to verify this, notifying the user if it is wrong. Afterwards, resolve this and ask the user if he wants to add said ID, showing the ID and email. As before, a negative response should close the client while a positive should add the ID. On a malformed ID the client should alert the user, closing the client after acknowledgement.

###DNS Discovery (should be reworded)
A DNS discovery ID goes in the following format: `user@domain`. Users should not enter a DNS discovery ID in any way they don't normally add a Tox ID, clients should be able to figure out what is what. On adding a DNS discovery ID, a client must resolve a DNS TXT record for the value `user._tox.domain`. In this case the `@` is replaced with `_tox`, allowing the use of subdomains while ensuring a record is a Tox record. Typical records lack spaces, though clients should be able to deal with oddly formatted cases. Clients are also encouraged to check DNSSEC, though this is not a requirement.

The `tox://` URI has 2 versions, tox1 and tox2. Tox2 attempts to stop dns request spamming and dns poisoning attacks by using the a form of the nospam as a unique pin. Use of tox1 is **depreciated**, and use is **strongly discouraged**.

In the case of multiple records clients should first look for the highest version record and default to that, this becomes the highest priority. In the case of multiple records of the sam priority, a client is free to choose the first one the dns lookup reported.

| Tox1        | Tox2          |
| ------------- |:-------------:|
| The value of a Tox DNS record goes `v=version;id=Tox ID`, where the version is tox1, and id is the users Tox ID. A client then follows the standard `tox://` schema. Typical records lack spaces, though clients should be able to deal with oddly formatted cases.      | The value of a Tox DNS record goes `v=version;pub=public key;check=checksum`, where the version is tox2, pub is the public key, and the checksum is the XOR'd value of the nospam and public key. A client then asks the user for a pin, then it appends `==` to it and turns it from base64 to hexidacimal and XOR's this against the key, checking it against the checksum to verify. |


####tox1 BNF
    <uri> ::= "tox://" (<Tox id> | <user> "@" <domain>)
    <Tox id> ::= <public key><nospam><checksum>
    <checksum> ::= <public key> xor checksum <nospam>
    <DNS discovery ID> ::= <user> "@" <domain>
    <dns_record_text> ::= "v=tox1;ib="<Tox id>
    <dns_record_format> ::= "TXT" <user> "._tox." <domain> "=" <dns_record_check>

####tox2 BNF
    <uri> ::= "tox://" (<Tox id> | <user> "@" <domain>)
    <Tox id> ::= <public key><nospam><checksum>
    <checksum> ::= <public key> xor checksum <nospam>
    <DNS discovery ID> ::= <user> "@" <domain>
    <pin> ::= <nospam.base16> -> base64
    Checking a PIN ::= <pin.base64> -> <temp_checksum.base16>; (<public key> xor checksum <temp_checksum>) == <checksum>
    <dns_record_text> ::= "v=tox2;pub=" <public key> ";check=" <checksum>
    <dns_record_format> ::= "TXT" <user> "._tox." <domain> "=" <dns_record_check>
  
  
  
  
  
## Translation of STS Terminology
Client developers must choose translations that resemble the English variations as closely as possible, except in the case where the Tox trademark is being used. For example, Tox ID is to remain, untouched, in English. In the future, we will provide translations to be used in non-English Tox clients.

