DRAFT
## PROPOSED STANDARD
### A Standard to provide public keys for secure communications.

## Abstract

## Status of this Memo
#### This is a draft that is open for comment and feedback by the public.

###### It is intended to be worked into a functioning standard to be submitted to [the Internet Engineering Task Force (IETF)](https://www.ietf.org/) for adoption by the public.

## Copyright Notice
This draft and subsequent contributions are licensed under [Creative Commons CC-BY-SA 4.0 International](https://creativecommons.org/licenses/by-sa/4.0/).
#### [You can find the License here.](./LICENSE.md)


### 1. Introduction
#### 1.1. Motivation, Prior Work, and Scope
Existing end-to-end - encrypted communication, espechally those that ain't single-vendor and/or single-provider solutions, will usually utilize Public - Key Cryptography, necessitating some sort of public key exchange beforehand.

Existing mechanisms don't lend themselves to good automation and are ripe for abuse since they don't check the source and authenticity of submitted public keys, allowing anyone to impersonate anyone with the only safety margin being fingerprints and hashsums that may be easily overlooked by most users.

This standard aims to provide a simple yet effective & domain-authoritative solution to provide public keys for said communications using the domain name system and static paths that can be automatically queried for transparent public key exchange.

#### 1.2. Terminology
The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in BCP 14 [[RFC2119](https://www.rfc-editor.org/rfc/rfc2119)] [[RFC8174](https://www.rfc-editor.org/rfc/rfc8174)] when, and only when, they appear in all capitals, as shown here.

### 2. The Specification
The 

### 3. Location
To be agnostic to the application that uses the public keys for communication, it must utilize a two-step approach.
#### 3.1. DNS entry
Similar to Sender Policy Framework [SPF] as per [[RFC7208](https://www.rfc-editor.org/rfc/rfc7208)], a DNS entry type "PKD" - Short for: "Pubkey Directory" - is intended to be used.

As transitional Mechanism, the use TXT fields is recommended.

Furthermore, this Standard supports multiple domains, subdomains and services.
- eMail encryption [Public Keys PGP/MIME & inline-PGP]
- XMPP + GnuPG/OpenPGP
- SSH Public Keys
- Public Keys for File Encryption [using GnuPG or OpenPGP]
- Public Keys for Virtual Private Networks [VPNs]

##### 3.1.1. Example PKD Entries
For `domain.example` a global PKD entry would look like this:

`PKD    ="*.domain.example" pkd.domain.example/`
- As is customary for DNS, the PKD entries can also be set to apply for subdomains only and thus distinguish services.
  - There needs to be a seperate PKD specified for each seperate subdomain unless they are wildcard subdomains.
```
PKD    ="mx.domain.example"    pkd.domain.example/
PKD    ="*.mx.domain.example"  pkd.domain.example/
```

###### If a service has a dedicaded DNS record entry type - for example `MX` for eMail Servers - one or multiple domains can be specified to query these.
- Multiple Services will require multiple PKD entries regardless if they use the same keys or not. For Example:
```
PKD    ="chat.domain.example"  https://pkd.domain.example/xmpp/
PKD    ="MX"    http://pkd.domain.example/mail/
PKD    ="*.domain.example"     ftps://pkd.domain.example/
```


### 4. Public Key File Format Description
The Public Keys MUST be plain text (MIME type "text/plain") as defined in Section 4.1.3 of [[RFC2046](https://www.rfc-editor.org/rfc/rfc2046)] and MUST be encoded using UTF-8 [[RFC3629](https://www.rfc-editor.org/rfc/rfc3629)] in Net-Unicode form [[RFC5198](https://www.rfc-editor.org/rfc/rfc5198)].
- UNIX Line Endings ["LF"] MUST BE USED.
#### 4.1 Filenames and Location
The Public Key Files MUST be located at the directory specified in the PKD entry.
- To enshure the distinguishment of said files as pubkeys and enable automatic query of said keyfiles the extension `.pubkey` MUST be used.
  - A missing / nonexistant public key MUST be replied with an HTTP Status Code 404 "Not Found".

##### 4.1.1. Example Keyfile
For `example.com` with the `PKD record`
of `PKD    ="*.domain.example"     pkd.domain.example/`
the Public Key File to email `user@domain.example` MUST be located at `pkd.domain.example/user@domain.example.pubkey` in the descriped format.

```
version = 2
last_updated = 1672900000
expires = 1704400000
service = MX
user = user@domain.example
-----BEGIN PGP PUBLIC KEY BLOCK-----
[...]
-----END PGP PUBLIC KEY BLOCK-----

```


### 5. Security Considerations
Because of the use of URIs and well-known as well as easily guessabel resources, security considerations of [[RFC3986](https://www.rfc-editor.org/rfc/rfc3986)] and [[RFC8615](https://www.rfc-editor.org/rfc/rfc8615)] apply here, in addition to the considerations outlined below.
#### 5.1. Compromised Files and Incident Response
To enshure that only the correct files are submitted, administrators of said domain SHOULD allow users to automatically provide said public-keys "in-band" within the application or protocol being used.
- In the case of `eMail`, this could be done via submitting the Public Key in-client to a specified adress like  `pkd@domain.example` as an eMail attachment.
  - Organizations SHOULD thus exercise due diligence for said applications and be suspicious if public keys get changed very often - espechally way before their expiry date.
- In cases where this is not possible and/or not desireable, it is recommended to establish organization and/or application-specific management of said Public Keys.
  - In business applications, where confidentiality as well as auditability of electronic communications is necessary, public and private keys are already subject to archival requirements.
    - Said requirements and considerations vary widely as per juristiction and thus cannot be part of the scope of this standard.

#### 5.2. Redirects
While it is not forbidden to let the PKD entry point to a different domain - for example a keyserver operated and maintained by an eMail provider or server hoster - it is discouraged to do so.
- Mechanisms like DNSSEC on the side of the Requesting Party and automated, regular checks - may it be in-application or otherwise - by the Offering Party are the most effective workarounds.

Additionally, the use of external monitoring in the form of "probing" outside one's company network is RECOMMENDED, but not a hard requirement.
- We also recommend to digitally sign said files using either GnuPG/OpenPGP or X.509 Certificates.
  - In the case of X.509 Certificates, it is RECOMMENDED to use the same certificate that is being used for the domain - or at the very least from the same Certificate Authoritiy.

These risks and issues are not unique to this but mitigation is out of scope for this standard.

#### 5.3. Incorrect or Stale Information
Since said information may change, it is recommended to version the Public Keys.
- In order to make automatic detection of newer versions simpler, it is REQUIRED to add both a `version` counter a `last_updated` field before the actual key block.
  - A `expires` field is RECOMMENDED, but not required.
    - In no `expires` field is defined, applications MUST default to a maximum of 1 year from the moment it was queried.
    - It is recommended to set the `expires` field no later than 5 years from the `last_updated` date.
  - whilst `version` is to be an intreger counter stating with 0, the `last_updated` and `expires` fields must be UNIX Timestamps.

#### 5.4. Intentionally Malformed Files
Since the public keys are served from a publicly accessible server, applications should transparently detect absurdly large files.
-   It stands to reason that a file double the size of the expected key size - including the header of the `pubkey` file - is malicious, and applications should not only stop the query / download of said public key, but also discard the downloaded key.
-   Unlike regular Public Key Servers, this standard aims to deliver authoritate-by-hierachy data and thus should not allow random submissions of Public Keys, espechally external ones, to be accepted and hosted.

#### 5.5. Abusive Traffic
As the server will necessarily be publicly accessible, it is possible that that it may be subject to Distributed Denial of Service attacks (DDoS) or attempts onto these.
- Existing mechanisms such as rate-limiting IP-Adresses or blackholing entire Subnets where necessary should be sufficient to do so.
  - Mitigation of said attacks is out-of-scope for this standard, but considering the fact that these are in fact just plain text, DDoS attacks should not yield efficiency - espechally when a seperate [virtual] machine is being used to host said files.

#### 5.6. Multi-User & Domain Environments
As there may be scenarios where multiple domains use the same Public Key Directory, it's recommended to exercise caution and due diligence where applicable.
- It may necessitate specific rules based off the setup in question, from not allowing to submit critical pubkeys like `security@domain.example` that are specified in the [`security.txt`](https://securitytxt.org/) file as per [[RFC9116](https://www.rfc-editor.org/rfc/rfc9116)] 
  - Of course only administrators of the domains under their control should be allowed to submit private keys as to keep with the authoritative-by-hierachy nature in comparison to a keyserver.

#### 5.7. Protecting Data in Transit 
The use of SSL/TLS encryption is RECOMMENDED, but not a hard requirement.
- if multiple entries for a Public Key Directory exist, the ones using Transport Encryption should be prefered over those with no Transport Encryption.

#### 5.8. Spam
Since Public Key based Asymetric Cryptography is a computational intensive task, Spam with per-recipient encrypted eMails is extremely unlikely to happen as it's penalizingly hard and computationally expensive to facilitate.
- Thus it'll actually be a useful strategy to combat Spam eMails as it basically acts as a "proof of work" - alike penalty against Spammers.
  - In fact, with this proposed standard, it is hoped to make unencrypted messaging and communication a thing of the past and ease-in more users to effective end-to-end encryption.
    - Based off current hardware and developments, it's unlikely spammers will be able to keep sending unsolicited mass eMails if they are being forced to properly encrypt their eMails with individual, per-recipient keys.

#### 5.9. Malicious Providers & Hosters
Providers / Hosters may be coerced or forced into providing false public keys that could allow an attacker - regardless of legal status - to intercept and manipulate communications.
- As these attacks can be monitored for with an external probe, the risk is mitigateable to the point that any attempts to do so would be foolish due to their detectability.
  - So regardless of legality, said attacks would be detectable actively and passively.

### 6. Benefits
As with all proposed standards, the Public Key Directory aims to solve a Problem and provide a solution that also adds value and increases the user experience of encryption.

#### 6.1. Authoritiative Public Key Distribution
Current Systems - espechally public Keyservers - are essentially anarchism, as anyone can upload a public key pretending to be anyone with neither proof of authenticity nor deterrent to (attempted) impersonation.
- It is up to the end-user to decide which public key to trust and which to reject.
  - This may boil down to fingerprint comparisons which are time-consuming and hard-to-explain to users who are not familiar with the concepts of public-private cryptography in general.
- Lack of an easy distribution, update, verification and exchange mechanisms for Public Keys is one of the most cited reasons not to use GnuPG/OpenPGP, OTR, OMEMO or similar methods.

By Using DNS entries as authoritative data source for a domain, the Public Key Directory can be published in a way that it can be autoritatively queried for said Public Keys.

#### 6.2. Avoiding "Trust On First Use" situations.
Many Public Key Cryptography Systems and Applications do not employ good mechanisms to verify or authenticate the correctness of public keys prior to initial contact.
- This results in insecure "Trust On First Use" (TOFU) situations which are potentially ripe for abuse in the form of social engineering and impersonation-based attacks.
  - TOFU situations can only be clarified retroactively, meaning that they must be initiated by the users in question and usually necessitate different communication systems than the one to be secured.
    - This may cause a whole "chain of distrust" that may only be broken in-person, which may not be desireable or even possible in most cases.
- DNS records can be secured with existing techniques like DNSSEC against tampering and fraudulent data insertion in transit.
  - It also enshures that the distribution of keys and storage can be as flexible as eMail for said domain.

#### 6.3. Assisting in the adoption of end-to-end Encryption
The whole aspect of public key exchange is severely hampering the adoption of secure, end-to-end encryption by the masses.
- By making it easy for the users to provide and update their public keys, the barriers of entry and use are being significantly reduced.
  - Similar to how AAAA entries allow for seamless and fast IPv6 transition through allowing dual-stack connectivity.

#### 6.4. Enabling Compliance and Confidentiality
In most juristictions, tax-relevant communications - espechally those done within a corporation or other legal entitiy - are subject to auditability requirements, including up to:
- instant archival of all communications
- indexing of all communications including attached / transmitted files
  - full-text searchability of all emails and attachments.
- requirement of cryptograpic tamper-proof for said communications.

Normally, End-to-End - Encryption may be deemed incompatible with these requirements, but they ain't necessarily.

For example, private keys can be added to said archival systems in order to comply with said regulations.
- Furthermore, using said public - private cryptography can be used as an additional layer of authenticity to enshure the authenticity of the message.

In this context, automatic archival is as easy as if the communications were secured using X.509 certificates on S/MIME.
- Of course outgoing eMails may still require a copy to be BCC'd into an archival inbox that is encrypted with it's public key, but that can be automated in every modern eMail client.
  - Since all commonly existing eMail clients will also store sent messages and sync them to the server using IMAP, either an unencrypted or encrypted with the Sender's Public Key Version is usually being retained.

#### 6.5. Automatic Encryption of Messages
Currently, the complexity of distribution and updating public keys prevents the use in automated Messages - including critical and confidential system reports.
- With this standard it is possible to automatically pick and use the correct encryption key everytime without needing to know where it's being stored.

#### 6.6. Application-Agnostic & Simple Solution
The approach defined in this draft is not only agnostic of the application(s) being used but is also simple and scalable.
- Not only will it work with real End-to-End Encryption, but also with "Man-In-The-Middle" Appliances and Tools that transparently handle encryption and/or decryption.
  - Whilst the Authors do not endorse such means that break end-to-end principle, they do acknowledge that there may be the need and/or demand to use such methods.
- This approach works with all common methods that use URLs for users and/or systems.

This will allow basically any application to query for matching Public Keys and automatically import them.
- Similar to how SSL certificates are automatically being accepted due to the authoritative chain.

---
## Acknowledgements
The authors acknowledge the works of Edwin Foudil, Yakov Shafranovich et. al. that made the [`security.txt`](https://securitytxt.org/) Standard that was formalized into [[RFC9116](https://www.rfc-editor.org/rfc/rfc9116)].

---

## Authors' Adresses
- Kevin Karhan
  - [kevin@karhan.guru](mailto:kevin+pkd-draft@karhan.guru)

