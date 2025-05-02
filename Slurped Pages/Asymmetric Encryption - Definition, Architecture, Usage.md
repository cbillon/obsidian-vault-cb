---
link: https://www.okta.com/identity-101/asymmetric-encryption/
excerpt: Asymmetric encryption allows users to keep their communication secure. Learn more about asymmetric key encryption with Okta.
slurped: 2025-02-20T16:11
title: "Asymmetric Encryption: Definition, Architecture, Usage"
tags:
  - pki
  - ssh
  - asymetric
  - ssl
---

## How Does Asymmetric Cryptography Work?

Sensitive messages move through a process of encryption and decryption with public and private keys. 

An algorithm starts the process. A mathematical function generates a key pair. Each key is different, but they are related to one another mathematically. 

Key generation protocols differ, and the keys they create are different too. In the Microsoft environment, for example, you need [about four lines of code](https://docs.microsoft.com/en-us/dotnet/standard/security/generating-keys-for-encryption-and-decryption) to start the development of a pair of asymmetric keys. Other programs work in a similar manner. 

Imagine that someone wants to send an encrypted message to another person. The process looks like this:

- **Registration:** The user and the sender have connected with an official entity that generated both public and private keys. 
- **Lookup:** The sender scours a [public-key directory](https://www.thekumachan.com/public-key-directory/) for the recipient's public key information. 
- **Encrypt:** The sender creates a message, encrypts it with the recipient's public key, and sends it. 
- **Decode:** The recipient uses the private key to unscramble the message. 
- **Reply:** If the recipient wants to respond, the process moves in reverse. 

Now, imagine that someone wants to communicate with an entity, not an individual. Certificates become important. 

SSL certificates, [commonly used by websites](https://neilpatel.com/blog/ssl-certificate-guide/), work a bit like handshakes. An organization jumps through a hoop, such as registering with an entity and proving ownership, and a certificate is created. When a user accesses a site like this, the user's computer and the website verify private and public keys before information is passed. But once verification happens, the data passes through symmetric encryption, allowing for speed. 

[Web authentication](https://www.okta.com/video/oktane20-how-web-authentication-works/) is relatively easy to understand. We've all used it. But plenty of other entities use the technique to keep their users safe. Bitcoin, for example, leans heavily on asymmetric encryption. A transaction is associated with a public key, but a private key is required for a person to move that transaction from one account to another.

## Pros & Cons of Asymmetric Encryption 

All site administrators and security-minded individuals require some kind of encryption tool. But asymmetric solutions aren't right for everyone. 

### Pros

- **Security:** Without both keys, a hacker can only access useless data. 
- **Transparency:** Public keys can be openly distributed, as losing them will not devolve into a security risk. 
- **Appearance:** Weary consumers demand robust security. [Zoom discovered this](https://www.wired.com/story/zoom-security-encryption/) in 2020, for example, when reporters uncovered lapses in code that meant Zoom calls weren't truly encrypted end to end, as mentioned in their marketing materials. Delivering a truly secure environment is critical to many people. 

### Cons

- **Speed:** Systems need time for decryption. Users sending plenty of bulk files will have a long wait. 
- **Vulnerabilities:** Lose a private key, and anyone who finds it can read all messages, even if they're private. A lost key can result in a man-in-the-middle attack too. 
- **Loss:** If you lose your private key, you won't be able to decrypt messages sent to you.
- **Long-term sustainability**: In the future, [quantum computing](https://www.ibm.com/quantum-computing/learn/what-is-quantum-computing/) will break most asymmetric and symmetric approaches.

You may read through this list and decide that the benefits far outweigh the risks. But if you decide that the risks are too great, you could use symmetric encryption instead. You'll have lower risks that stem from loss and speed, but your data could be slightly less secure.

## History of Asymmetric Encryption 

The idea of protecting data isn't new. In fact, the concepts that lie beneath asymmetric cryptography were defined decades ago.

In 1977, two researchers at Stanford University [published a paper](https://ee.stanford.edu/~hellman/publications/24.pdf) with asymmetric encryption concepts clearly defined. They felt the new protocols were required, as consumers were moving from cash-only transactions to digital versions, and they needed ways to protect their finances.

In time, the ideas spread, and soon, individuals, public companies, and private endeavors all scrambled to implement this high level of security. In 1995, asymmetric encryption moved to the mainstream, as the [HTTPS protocol was released](https://www.wired.com/2017/01/half-web-now-encrypted-makes-everyone-safer/) for widespread use. Now, companies as large as Google use the technique to protect their communications. 

## We Can Help 

Learn more about how Okta [uses asymmetric encryption](https://www.okta.com/resources/whitepaper/okta-security-technical-white-paper/) to protect your organization. We can help you understand what solutions work best for your organization, and we can implement them for you. Contact us, and we'll help.

### References

[Why HTTPS for Everything?](https://https.cio.gov/everything/) CIO Council. 

[Generating Keys for Encryption and Decryption](https://docs.microsoft.com/en-us/dotnet/standard/security/generating-keys-for-encryption-and-decryption). (July 2020). Microsoft. 

[Public Key Directory](https://www.thekumachan.com/public-key-directory/). (March 2009). The Kumachan. 

[How to Choose Between These 5 SSL Certificates for Your Site](https://neilpatel.com/blog/ssl-certificate-guide/). Neil Patel. 

[So Wait, How Encrypted Are Zoom Meetings Really?](https://www.wired.com/story/zoom-security-encryption/) (April 2020). _Wired._ 

[New Directions in Cryptography](https://ee.stanford.edu/~hellman/publications/24.pdf). (November 1076). _IEEE Transactions on Information Theory._ 

[Half the Web is Now Encrypted. That Makes Everyone Safer](https://www.wired.com/2017/01/half-web-now-encrypted-makes-everyone-safer/). (January 2017). _Wired._ 

[Here's a Simple Introduction on How Browsers Encrypt Your Data](https://medium.com/@streetsofboston/heres-a-simple-introduction-on-how-browsers-encrypt-your-data-ab1c0ea2ee4). (January 2019). Anton Spaans.