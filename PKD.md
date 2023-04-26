DRAFT
## PROPOSED STANDARD
### A Standard to provide public keys for secure communications.

## Abstract

## Status of this Memo
This is a draft that is open for comment and feedback by the public.

It is intended to be worked into a functioning standard to be submitted to [the Internet Engineering Task Force (IETF)](https://www.ietf.org/) for adoption by the public.

## Copyright Notice
This draft and subsequent contributions are licensed under [Creative Commons CC-BY-SA 4.0 International](https://creativecommons.org/licenses/by-sa/4.0/).
#### [You can find the License here.](./LICENSE.md)


### 1. Introduction
#### 1.1. Motivation, Prior Work, and Scope
Existing end-to-end - encrypted communication, espechally those that ain't single-vendor and/or single-provider solutions, will usually utilize Public - Key Cryptography, necessitating some sort of public key exchange beforehand.

Existing mechanisms don't lend themselves to good automation and are ripe for abuse since they don't check the source and authenticity of submitted public keys, allowing anyone to impersonate anyone with the only safety margin being fingerprints and hashsums that may be easily overlooked by most users.

This standard aims to provide a simple yet effective & domain-authoritative solution to provide public keys for said communications.

#### 1.2. Terminology
The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in BCP 14 [[RFC2119](https://www.rfc-editor.org/rfc/rfc2119)] [[RFC8174](https://www.rfc-editor.org/rfc/rfc8174)] when, and only when, they appear in all capitals, as shown here.

### 2. The Specification
The 

### 3. Location
To be agnostic to the application that uses the public keys for communication, it must utilize a two-step approach.
#### 3.1. DNS entry
Similar to Sender Policy Framework [SPF] as per [[RFC7208](https://www.rfc-editor.org/rfc/rfc7208)], a DNS entry type "PKD" - Short for: "Pubkey Directory" - is intended to be used.

As transitional Mechanism, the use TXT fields is.

### 4. Public Key File Format Description
The Public Keys MUST be plain text (MIME type "text/plain") as defined in Section 4.1.3 of [[RFC2046](https://www.rfc-editor.org/rfc/rfc2046)] and MUST be encoded using UTF-8 [[RFC3629](https://www.rfc-editor.org/rfc/rfc3629)] in Net-Unicode form [[RFC5198](https://www.rfc-editor.org/rfc/rfc5198)].

### 5. Security Considerations
Because of the use of URIs and well-known resources, security considerations of [[RFC3986](https://www.rfc-editor.org/rfc/rfc3986)] and [[RFC8615](https://www.rfc-editor.org/rfc/rfc8615)] apply here, in addition to the considerations outlined below.
#### 5.1. Compromised Files and Incident Response
#### 5.2. Redirects
#### 5.3. Incorrect or Stale Information
#### 5.4. Intentionally Malformed Files, Resources, and Reports
#### 5.5. Abusive Traffic
#### 5.6. Multi-User Environments
#### 5.7. Protecting Data in Transit 
#### 5.8. Spam
