# TLS Certificates

In this document, TLS Certificates and how they are used in K8s.


## How are TLS Certificates used and why?

Let's assume that you are trying to connect to a server over SSH. You will 1st exchange your public key with the server. Once you pass public key onto server, then you will be able to use your Private key to authenticate with the server.

What if you are trying to do same thing, have secure communication with a WebServer. In the case of Web communication we need to have secure key exchange without using any passwords.
- In this case you will initiate with the WebServer.
- WebServer sends you a Certificate that is trusted by a CA that you and WebServer both trust.
- You use that Certificate to encrypt your public key and send it to the Server.
- Server then decrypts the public key you sent and then you can start sending requests to the server that are encrypted with your private key and server can decrypt it using public key it has from you.
- When servers sends back the data, you can use the certificate that server sent you earlier to validate the contents of the messages you are receiving are indeed from the server.

See Certificate as a public key and key as private key in this scenario, analogous to SSH.