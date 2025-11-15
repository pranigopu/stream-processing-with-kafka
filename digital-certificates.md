<h1>DIGITAL CERTIFICATES</h1>

---

**Contents**:

- [Definition](#definition)
- [Digital signature](#digital-signature)

---

# Definition
A public key certificate (or digital certificate) is a piece of digital information that consists of the public portion of a private/public key pair as well as the metadata values (name, company name, certificate expiry date, etc.); these identify the holder of the certificate. A certificate is said to be "signed" when a certificate authority (CA) or individual uses a private key to encrypt a hash of a message.

Terminology:

- Issuer: The entity that issues the certificate (possibly also signing it)
- Subject: The entity that must be authenticated (verified for its identity) via the certificate

If issuer = subject, the certificate is said to be self-signed.

# Digital signature
A cryptographic mechanism used to verify the authenticity and integrity of digital messages or documents. They work by creating a unique code (signature) based on the content of the message and the sender's private key. The recipient can then use the sender's public key to verify the signature, confirming that the message has not been altered and is indeed from the claimed sender. In the case of an issuer's digital signature for public key certificates