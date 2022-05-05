# TLS

## Authentication

Verifies the identity of the communicating parties with assymetric cryptography.

## Confidentiality
  
Protect the exchanged data from unauthorized access with symmetric cryptography.

## Integrity

Prevent alteration of data during transmission with message authentication code.

## How TLS works

There are two phases that happen when establishing client and server communication:

### 1. Handshake protocol

This covers the **Authentication** part.

- Negotiate TLS version
- Select cryptographic algorithms: cipher suite
- Authenticate by assymetric cryptography
- Establish a secret key for symmetric encryption

The main purpose of the handshake is for authentication and key exchange.

### 2. Record protocol

This covers the **Confidentiality** and **Integrity** parts.

- Encrypt outgoing messages with the secret key
- Transmit the encrypted messages
- Decrypt incoming messages with the secret key
- Verify that the messages are not modified

Because the data in phase can be large, it is often called Symmetric Bulk Encryption.

## The Shared Secret Key

### Encryption

Data between parties are encrypted and decrypted with a shared key.
This is symmetric cryptography. The encryption algorithm is typically:

- AES-256-GCM
or
- CHACHA20

The encryption algorithm also takes as inputs the shared secret key,
and a random Nonce (an initialization vector)
This outputs the encrypted message.

The second step is to authenticate the encryption.
The shared secret key, the Nonce and the encrypted message
becomes inputs to a MAC algorithm. The algorithm is

- GMAC for AES-256-GCM
or
- POLY1305 for CHACHA20

This MAC algorithm acts like a cryptographic hash function,
and its output is a MAC (message authentication code). This MAC
will be tagged onto the encrypted message and this final result
will be sent. Because of this the MAC is sometimes called an
authentication tag.

Sometimes some associated data is put into the MAC algorithm like

- Addresses
- Ports
- Protocol version
- Sequence number

This data is un-encrypted by both parties.

This whole process is called **AEAD** -- Authenticated Encryption with Associated Data.

### Decryption

The decryption is simply a reverse of the process. The encrypted message is separated
from the MAC tag. The encrypted message along with the shared secret key and the same
Nonce and associated data is put into the MAC algorithm. The output tag is then compared
to the received tag and verified that they are same. If they are the message is decrypted
with the same encryption algorithm, shared secret key and Nonce.

### How is the key shared

The question is how do they share the key with each other?
The answer is with **Assymetric (Public-Key) Cryptography**. Specifically:

- Diffie-Hellman Ephemeral (DHE)
or
- Elliptic Curve Diffie-Hellman Ephemeral (ECDHE)

Without getting into the details, assymetric encryption works as follows.
Suppose you have parties A and B. They need to come up with a shared secret
key between them for symmetric encryption.

So in the handshake process A sends a public key to B and B sends a public key
to A. These public keys become the encryption keys for assymetric messages. However,
through using one of the assymetric algorithms A and B creates a private key for the
public key. So when A sends to B, A encrypts the message with B's public key. But B will
decrypt the message using its private key. This is secure against hacking because
a hacker can intercept the encrypted message and even though key has the
public keys, he can't decrypt it without the private keys.

What a hacker can do however, is intercept the message that contains the public key sharing
and replace it with its own public key. This means if A received a false public key it will
send and encrypted message to B that B cannot decrypt.

So we need a way to verify public keys -- **Digital Certificates**.

### Certificates

In a client and server situation, usually only the server will need a certificate.
The certificate will contain information about the server along with its public key.
It acts like a passport.

But what stops the hacker from creating a fake certificate in the Server's name.

Well, just like a passport is signed and verified by a passport authority (the government),
a certificate is signed and verified by a certificate authority (CA).

#### Certificate Signing Process

We'll call the server B.
B has a pair of public and private keys.
B creates a **Certificate Signing Request (CSR)**. This CSR contains his public key,
and some identity information like

- Name (which acts as the domain)
- Organization
- Email
- etc.

B then signs the CSR with its private key and sends it to the CA.
The CA verifies B via official communication and so forth.

If everything is valid the CA will also sign the certificate with its own private key,
and send the signed certificate to B.

The server will then send its certificate, which contains its public key to the client to
verify itself.

The client receives this certificate and verifies the certificate (and thus indirectly
the server's public key) by using the CA's public key.

The hacker cannot create a false certificate because he doesn't have the CA's private key
to sign the certificate. This all works because we agree to trust the CA.

There is a chain of trust.
You get the **Root Certificate Authority** who signs their own certificate.
- They also sign the certificates of subordinates or **Intermediate Certificate Authorities**
  - The intermediate authority can sign the certificates of other intermediates, or they can
  sign the certificate of the end-entity

Each leaf certificate will have a reference to their higher level certificate, all the way to
the root.

Operating systems and browsers store a list of all Root CAs. That way they can verify all certificates.

## The TLS 1.3 Full Handshake

The client sends a hello message, which contains

- The supported TLS versions
- The supported AEAD symmetric cipher suites
- The supported assymetric key exchange groups
- The client's public key
- Supported signature algorithms. The is the algorithm that the server will use
  to sign the whole handshake.

After receiving the client's hello message, the server sends its hello message, which contains

- The selected TLS version
- The selected cipher suite
- The selected key exchange group
- The server's public key
- A request for the client's certificate (this is optional)
- The server's certificate
- Certificate verify. A signature of the entire handshake so far.
  The entire handshake so far is called the handshake context. The handshake
  context combined with the server certificate is hashed to create a signature.
- Server finish. The MAC of the entire handshake. This is the handshake context, the server
  certificate, and the certificate verify signature all combined and hashed.

The server certificate, certificate verify and MAC are called the authentication messages,
because they are used to verify the server.

After the client receives the server's hello message, it will validate the server's
certificate against a root CA.

After the server is verified the client sends a final finish message that contains

- The client's certificate (if requested)
- Certificate verify. A signature of the entire handshake
- Client finish. A MAC of the entire handshake

# Create and Sign TLS Certificate

In this demo we will play the part of the CA also that needs to sign the CSR.
All the following steps requires OpenSSL.
The steps are

## Generate CA's private key and self-signed certificate

```bash
  openssl req -x509 -newkey rsa:4096 -days 365 -keyout ca-key.pem -out ca-cert.pem
```

The `-x509` flag tells openssl that it is creating a self-signed certificate
instead of processing a CSR. X509 is standard for defining the format
of public key certificates.

The `rsa:4096` flag tells openssl to create both a new private key, and a
certificate request at the same time. Because of the **X509** option it
will output the certificate instead of the certificate request.

The `-keyout` flag specifies an output file, which contains the key.

The `-out` flag specified an out file, which contains the certificate.

When we execute the command it will create the `ca-key.pem` file and then ask
us for a pass phrase. The pass phrase is used to encrypt the private key to protect
it incase it gets leaked.

Next openssl will ask for some verification info.

To view the certificate in readable form execute

```bash
  openssl x509 -in ca-cert.pem -noout -text
```

## Generate web server's private key and its certificate signing request (CSR)

The command to this is very similar to the first command.

```bash
  openssl req -newkey rsa:4096 -keyout server-key.pem -out server-req.pem
```

## Use CA's private key to sign web server's CSR and get back the signed certificate

```bash
  openssl x509 -req -in server-req.pem -CA ca-cert.pem -CAkey ca-key.pem -CAcreateserial -out server-cert.pem
```
# Additional Reading

- https://devblogs.microsoft.com/dotnet/configuring-https-in-asp-net-core-across-different-platforms/
