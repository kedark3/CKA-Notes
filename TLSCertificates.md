# SSL vs TLS

##  SSL 
- Secure Sockets Layer
- Developed by NetScape in early 1990s
- V1-3 are available, v3 released in 1996
- Superseded by TLS, hence should no longer be used
- Prone to POODLE Attack- Malicious JS can compromise secure HTTP session cookies, HeartBleed bug- which did not check for length of the message and respond with requested length message, so attacker can send "Dog, 60" and response would be "Dog" followed by 57 characters that were in memory and that can cause unwanted data leaks

## TLS

- Better and more secure than SSL
- Man-in-the-middle(MITM) attack tring to downgrade the security protocol used can be mitigated with TLS_FALLBACK_SCSV
- Disable SSL, and enable TLS 1.1 or better
- Introduced in 1999
- TLS v1.0 to 1.3, v1.0 is old and we should disable it. Such as the Beast Attack.


# TLS Certificates

In this document, TLS Certificates and how they are used in K8s.


## How are TLS Certificates used and why?

Let's assume that you are trying to connect to a server over SSH. You will 1st exchange your public key with the server. Once you pass public key onto server, then you will be able to use your Private key to authenticate with the server.

What if you are trying to do same thing, have secure communication with a WebServer. In the case of Web communication we need to have secure key exchange without using any passwords.
- In this case you will initiate with the WebServer.
- WebServer sends you a Certificate that is trusted by a CA that you and WebServer both trust(and it contains server's own public key).
- You use that Certificate to verify the identity and public key from the message to encrypt your symmetric key and send it to the Server. Before you do that though, your browser tries to use CA's pub key to validate the cert is indeed signed by a CA. If not, you may see warning in your browser.
- Server then decrypts the symmetric key you sent and then you can start sending requests to the server that are encrypted with your symmetric key and server can decrypt it using symmetric key it has from you.
- When servers sends back the data, you can use the certificate that server sent you earlier to validate the contents of the messages you are receiving are indeed from the server.

See Certificate as a public key and key as private key in this scenario, analogous to SSH.

An important thing to note here is that Server uses Asymmetric Key while client is using Symmetric key. What that means is, when Client sends the key to the server, he is sending the 1 key he has, that can encrypt and decrypt the data using only that one key. But when initial handshake and key transfer happens, Client uses server's Public key(Cert) to encrypt his own symmetric key and server uses his own Private key to decrypt the message from client and extract his symmetric key.