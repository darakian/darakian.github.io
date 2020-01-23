---
layout: post
title:  "Exploring x509 by creating a toy PKI"
date:   2020-01-23 18:00:00 -0700
---
# PKI and x509
I've had a good opportunity this week to play with PKIs at work and after reading a number of poorly sourced and downright incorrect posts I felt that I should make a post to summarize my experiences. First off what is a Public Key Infrastructure (PKI)? Broadly speaking a PKI is system that allows for the creation and distribution of public keys. In a PKI you tend to have certificates (certs) and certificate authorities (CAs) to manage them. A CA will issue signed certs which contain public keys, some information about the certificate holder and some lifetime for the certificate. These certs can be validated by checking the signature on them against public key of the CA. There are key distribution problems and trust issues here, but if you can deal with those you have a method to distribute trust. The term PKI can mean imply a number of specifics depending on the context, but for this post PKI refer to the x509 system defined by [RFC 5280](https://tools.ietf.org/html/rfc5280). For most web systems this will be the only relevant PKI.

# So what are we doing here
We've struggled with a number of disparate issues at work which all have the same basic trust pattern to them; we have some code out in the wild which we would like to talk to and we would like to know that we're talking to our own code. Traditionally we've had solutions built around peered networks or fragile firewall rules, but a new situation arose for which those solutions would not work. We now need a method to determine the authenticity of some remote code running in an arbitrary network. Thankfully this is a problem that we can apply asymmetric cryptography to.

# Certs in the wild
Lets consider the general web PKI for a moment. Websites offer up certs to their users which are signed by some trusted CA which may in turn be signed by some CA which they trust and so on up to what are called the Root CAs. If we inspect all the certs along this chain we see something like
```
someWebsite <- someCA <- ... <- Root
```
Where each `<-` represents a signing. When presented with the website cert we can verify these signatures and then use the associated public key to initiate a TLS connection to said website. It's important to note that just the signature chain itself doesn't imply any sort of trust, but if we trust the root then we can infer trust from this chain.

# A toy example
Lets construct a simple three step PKI to see how this works in practice. Something that looks like
```
leaf <- intermediateCA <- Root
```
This gives us a CA with which we can create certs for whatever service we have but also gives us a layer of indirection so that we can keep the root private key offline and rotate the intermediateCA if need be. We will use openssl to accomplish this and we'll be using elliptic rather than RSA keys.  
First we need to generate our root keys which we can do with

## Building the root
```
❯❯❯ openssl ecparam -out root.pem -name secp384r1 -genkey
```
This will unsurprisingly create a file called `root.pem` which will contain a public-private key pair. In this example we use the `secp384r1` curve as it has a high security margin (in early 2020) and is widely compatible. If you want to explore other options give `openssl ecparam -list_curves` a look for supported curves. Anyway, reading this file you'll see something like
```
❯❯❯ openssl ecparam -in root.pem -text
ASN1 OID: secp384r1
NIST CURVE: P-384
-----BEGIN EC PARAMETERS-----
BgUrgQQAIg==
-----END EC PARAMETERS-----
```
If we dig further we can read the private key information out as well, but I'll leave that to the reader. Next we want to create a certificate which embeds the root private key and which is signed by the root private key. This is called a self signed cert and can be created with
```
❯❯❯ openssl req -new -key root.pem -out root.csr -subj "/C=US/ST=YourState/L=YourCity/O=YourOrganization/OU=YourUnit/CN=ThisIsMyRoot"
❯❯❯ openssl x509 -sha256 -in root.csr -out root.crt -req -signkey root.pem -days 30
```
You should see two new files in your directory; `root.csr` and `root.crt`. You can read your signing request with `openssl req -in root.csr -noout -text`
 which will dump out something like
```
Certificate Request:
    Data:
        Version: 0 (0x0)
        Subject: C=US, ST=YourState, L=YourCity, O=YourOrganization, OU=YourUnit, CN=ThisIsMyRoot
        Subject Public Key Info:
            Public Key Algorithm: id-ecPublicKey
                Public-Key: (384 bit)
                pub:
                    04:5b:...
                ASN1 OID: secp384r1
                NIST CURVE: P-384
        Attributes:
            a0:00
    Signature Algorithm: ecdsa-with-SHA256
         30:64:...
```
Similarly you can read your root cert with `openssl x509 -in root.crt -text` and see something like
```
Certificate:
    Data:
        Version: 1 (0x0)
        Serial Number: 10667317781089311673 (0x9409ed0108095bb9)
    Signature Algorithm: ecdsa-with-SHA256
        Issuer: C=US, ST=YourState, L=YourCity, O=YourOrganization, OU=YourUnit, CN=ThisIsMyRoot
        Validity
            Not Before: Jan 23 21:39:19 2020 GMT
            Not After : Feb 22 21:39:19 2020 GMT
        Subject: C=US, ST=YourState, L=YourCity, O=YourOrganization, OU=YourUnit, CN=ThisIsMyRoot
        Subject Public Key Info:
            Public Key Algorithm: id-ecPublicKey
                Public-Key: (384 bit)
                pub:
                    04:5b:...
                ASN1 OID: secp384r1
                NIST CURVE: P-384
    Signature Algorithm: ecdsa-with-SHA256
         30:66:...
-----BEGIN CERTIFICATE-----
MII...
-----END CERTIFICATE-----
```
Great. We've got our first certificate. Lets go ahead and verify the certificate
```
❯❯❯ openssl verify root.crt
root.crt: C = US, ST = YourState, L = YourCity, O = YourOrganization, OU = YourUnit, CN = ThisIsMyRoot
error 18 at 0 depth lookup:self signed certificate
OK
```
openssl sees that it's a self signed cert, but errors out because it's checking against the certificates preloaded into my operating system. We can overcome this by telling openssl to explicitly trust our new root cert which looks like
```
❯❯❯ openssl verify -CAfile root.crt root.crt
root.crt: OK
```
This may look a bit funny, but it checks out.

## The intermediate step
Creating the intermediate CA begins the same way; generate some keys, a signing request and a cert. The different will come at the cert step where we will use our root to sign our intermediate.
```
❯❯❯ openssl ecparam -out inter.pem -name secp384r1 -genkey
❯❯❯ openssl req -new -key inter.pem -out inter.csr -subj "/C=US/ST=YourState/L=YourCity/O=YourOrganization/OU=YourUnit/CN=ThisIsMyIntermediate"
```
Before we create the intermediate CA cert we need to discuss x509 v3 extensions. Introduced as part of [RFC 2459 ](https://tools.ietf.org/html/rfc2459#section-3.1) the v3 extensions standardize what capabilities a given certificate is authorized for. Of immediate importance is the capability of being used as a CA. We need this for our application so we'll create a text file with one line
```
❯❯❯ echo basicConstraints=critical,CA:true,pathlen:0 > ext.txt
```
three things are happening here. First we're setting the basicConstraints extension to be critical so that it should not be ignored, second we're we define that we want a cert which is usage as a CA and can sign other certs and third we set the path length to zero which makes it so that no children of this certificate can be a CA themselves.
Now create our intermediate cert with
```
❯❯❯ openssl x509 -req -sha256 -days 30 -in inter.csr -CA root.crt -CAkey root.pem -CAcreateserial -extfile ext.txt -out inter.crt
```
Lets recall that we created our root cert with the command
```
openssl x509 -sha256 -in root.csr -out root.crt -req -signkey root.pem -days 30
```
and we see a few differences. First we're passing the flags `-CA root.crt` and `-CAkey root.pem` which define the issuing CA and the signing key respectively. This is straight forward and this format will be used for anything downstream of the root cert. Second we see the flag `-CAcreateserial` which will create the serial number file if one is not provided and finally we see `-extfile ext.txt` which passes our newly created extensions file in so that we can get a certificate for a CA.  

Reading the new intermediate cert we see something like
```
❯❯❯ openssl x509 -in inter.crt -text
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number: 18391173934830647663 (0xff3a9340cc00f96f)
    Signature Algorithm: ecdsa-with-SHA256
        Issuer: C=US, ST=YourState, L=YourCity, O=YourOrganization, OU=YourUnit, CN=ThisIsMyRoot
        Validity
            Not Before: Jan 23 22:50:32 2020 GMT
            Not After : Feb 22 22:50:32 2020 GMT
        Subject: C=US, ST=YourState, L=YourCity, O=YourOrganization, OU=YourUnit, CN=ThisIsMyIntermediate
        Subject Public Key Info:
            Public Key Algorithm: id-ecPublicKey
                Public-Key: (384 bit)
                pub:
                    04:b6:...
                ASN1 OID: secp384r1
                NIST CURVE: P-384
        X509v3 extensions:
            X509v3 Basic Constraints: critical
                CA:TRUE, pathlen:0
    Signature Algorithm: ecdsa-with-SHA256
         30:65:...
-----BEGIN CERTIFICATE-----
MII...
-----END CERTIFICATE-----
```
and we can verify it against the root with
```
❯❯❯ openssl verify -CAfile root.crt inter.crt
inter.crt: OK
```

## And finally the leafs
Now we can create the leaf certs in this setup. Begin as before with a key and signing request
```
openssl ecparam -out leaf.pem -name secp384r1 -genkey
openssl req -new -key leaf.pem -out leaf.csr -subj "/C=US/ST=YourState/L=YourCity/O=YourOrganization/OU=YourUnit/CN=ThisIsMyLeaf"
```
For the leafs we don't care about any extensions for the leaf and so we can create a simple cert with
```
openssl x509 -req -sha256 -days 30 -in leaf.csr -CA inter.crt -CAkey inter.pem -CAcreateserial -out leaf.crt
```
read it with
```
openssl x509 -in leaf.crt -text
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number: 15045666593868194343 (0xd0ccf20d4079a227)
    Signature Algorithm: ecdsa-with-SHA256
        Issuer: C=US, ST=YourState, L=YourCity, O=YourOrganization, OU=YourUnit, CN=ThisIsMyIntermediate
        Validity
            Not Before: Jan 23 22:59:46 2020 GMT
            Not After : Feb 22 22:59:46 2020 GMT
        Subject: C=US, ST=YourState, L=YourCity, O=YourOrganization, OU=YourUnit, CN=ThisIsMyLeaf
        Subject Public Key Info:
            Public Key Algorithm: id-ecPublicKey
                Public-Key: (384 bit)
                pub:
                    04:60:...
                ASN1 OID: secp384r1
                NIST CURVE: P-384
        X509v3 extensions:
            X509v3 Basic Constraints: critical
                CA:TRUE, pathlen:0
    Signature Algorithm: ecdsa-with-SHA256
         30:64:...
-----BEGIN CERTIFICATE-----
MII...
-----END CERTIFICATE-----
```
and verify it with
```
❯❯❯ openssl verify -CAfile root.crt -untrusted inter.crt leaf.crt
leaf.crt: OK
```

# Wrap up
In researching and writing this I came across a lot of conflicting information, outdated recommendations and some downright wrong information. In particular many posts mention the bundling of your root and intermediate certs into a single file and verifying the leaf as `openssl verify -CAfile bundle.crt leaf.crt`, however this will not not verify that that trust chain checks out and in fact if you omit the `CA:true` extension on the intermediate cert `openssl verify -CAfile bundle.crt leaf.crt` will pass while `openssl verify -CAfile root.crt -untrusted inter.crt leaf.crt` will fail. I've glossed over a lot here, but I hope you found this post useful.

```
Thanks for reading
```
