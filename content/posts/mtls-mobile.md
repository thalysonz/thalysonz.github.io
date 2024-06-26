---
title: "Exploring mTLS in Mobile Applications"
date: 2024-06-27
tags: ["web", "mobile", "bug-bounty", "en"]
author: "Me"
# author: ["Me", "You"] # multiple authors
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "mtls in mobile real scenario"
disableHLJS: true # to disable highlightjs
disableShare: false
disableHLJS: false
hideSummary: false
searchHidden: true
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
ShowRssButtonInSectionTermList: true
UseHugoToc: true
---

## Exploring mTLS in Mobile Applications

Today, I will present a case study on mTLS that I conducted in a private bug bounty program.

**mTLS** (Mutual TLS) is a method of mutual authentication that ensures both parties in a network connection are who they claim to be. In an mTLS connection, the server verifies the client's certificate, and the client verifies the server's certificate. If a proxy like Burp Suite were used in this connection, the server would reject the connection.

### What is mTLS?

Mutual TLS (mTLS) is a security protocol used in network communications to provide an enhanced level of security. In a standard TLS setup, only the server is authenticated, while the client remains unauthenticated. This means that while the client can trust the server, the reverse isn't necessarily true.

In contrast, mTLS adds an extra layer of security by requiring both the client and the server to authenticate themselves to each other before establishing a secure connection. This is achieved through the use of digital certificates on both ends. When a client attempts to connect to a server using mTLS, the following steps typically occur:

1. The client presents its certificate to the server.
2. The server verifies the client's certificate.
3. The server presents its certificate to the client.
4. The client verifies the server's certificate.

### How Does the Client Certificate Verification Flow Work?

In the case of this application, a request containing a Certificate Signing Request (CSR) certificate was sent to the server. The server's response was a CA (Certificate Authority) certificate signed by the company.

### Certificate Signing Request (CSR) in mTLS

A crucial component in the mTLS process is the Certificate Signing Request (CSR). Let's break down what a CSR is and its role in mTLS:

- A CSR is one of the initial steps in obtaining an SSL/TLS certificate.
- It's generated on the server where you plan to install the certificate.
- The CSR contains essential information such as the common name, organization, and country.
- It also includes the public key that will be part of your certificate.
- The CSR is signed with the corresponding private key.


### PKCS #12: Bundling Cryptographic Objects

PKCS #12 is an archive file format used for storing multiple cryptography objects in a single file. It's commonly used to:

- Bundle a private key with its X.509 certificate
- Bundle all members of a chain of trust

This format plays a crucial role in managing the various certificates and keys used in mTLS.


### APP

#### CSR Request and CA Response
![CSR Request and CA Response](https://cdn-images-1.medium.com/max/2400/1*PHXBzazlYOZrqlqAGM6SmQ.png)

In the image above, we have a section of the application code responsible for creating the CSR certificate that will be sent to the server.

#### Hooking Process

Using Frida, I converted my keys (public and private) to the format expected by the `generateCertificateSigningRequest(args)`.

![Hooking Process](https://cdn-images-1.medium.com/max/2400/1*yTJH8I0LLP1kXtQOF-Jakg.png)


We can then hook the generateCertificateSigningRequest(args) function and alter the public and private keys used in the CSR creation process. Now the application will send the CSR certificate containing the new keys and generate a CA that we can use to create a pkcs12 file and import it into Burp Suite.

![Hooking Process](https://cdn-images-1.medium.com/max/2400/1*ME0HQovHdDGk2zLLUCOqlQ.png)
![Hooking Process](https://cdn-images-1.medium.com/max/2400/1*lkQdRVt6AjmeRlk4-UEoHQ.png)

![CAcreated](https://cdn-images-1.medium.com/max/1600/1*LHu7Psz_2UZLdFC8BgO3Zw.png)

#### PKCS#12 Import in Burp Suite
![PKCS#12 Import in Burp Suite](https://cdn-images-1.medium.com/max/2400/1*J6azIAinTn-sqqTSvI40JQ.png)

![](https://cdn-images-1.medium.com/max/2400/1*5aQ2OTRY5QNC_nLKody3ow.png)


#### Conclusion

Mutual TLS provides a robust method of ensuring secure and authenticated communication between clients and servers. In the context of a bug bounty program, understanding and exploiting mTLS can be crucial for identifying potential security flaws and vulnerabilities.

For more detailed information on mTLS and related topics, you can refer to the following resources:

- [Cloudflare: What is Mutual TLS?](https://www.cloudflare.com/pt-br/learning/access-management/what-is-mutual-tls/)
- [GlobalSign: What is a Certificate Signing Request (CSR)?](https://www.globalsign.com/pt-br/blog/what-is-a-certificate-signing-request-csr)
- [Hostinger: What is SSL/TLS?](https://www.hostinger.com.br/tutoriais/o-que-e-ssl-tls-https)


Happy securing!