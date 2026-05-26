---
link: https://cylab.be/blog/366/a-brief-overview-of-passkey
excerpt: You may have come across terms like “passkeys” or the intriguing idea
  of going “passwordless.” These concepts might sound confusing, especially
  since we’re all so used to securing everything with passwords—and constantly
  reminded of the importance of having strong ones. So, how could a world
  without passwords possibly be secure? In this blog post, we’ll explore this
  new method of authentication and break down how it works in a simple,
  easy-to-understand way. We won’t dive into the technical details, but you’ll
  get a clear overview of what passkeys are and how they can change the way we
  stay secure online.
slurped: 2026-05-19T08:43
title: A brief overview of passkey
---

You may have come across terms like “passkeys” or the intriguing idea of going “passwordless.” These concepts might sound confusing, especially since we’re all so used to securing everything with passwords—and constantly reminded of the importance of having strong ones. So, how could a world without passwords possibly be secure? In this blog post, we’ll explore this new method of authentication and break down how it works in a simple, easy-to-understand way. We won’t dive into the technical details, but you’ll get a clear overview of what passkeys are and how they can change the way we stay secure online.

## What are passkeys[](https://cylab.be/blog/366/a-brief-overview-of-passkey#what-are-passkeys)

Passkeys utilize standard public key cryptography techniques to authenticate users with service providers, such as Google accounts, banking platforms, and more.

The concept behind passkeys is relatively straightforward:

- The user generates a pair of keys: a public key (PK) and a private key (SK).
- The public key is then sent to the online service for future authentication.
- The private key remains securely stored on the user’s device and never leaves it.

When the user needs to log in to the service, they simply prove they have the private key associated with the public key that the service has on record. To do this, the server sends a unique challenge to the user, who then signs it using their private key. If the signature matches, the user is authenticated and logged in.

Here’s a visual representation of the process:

[![FIDO_signed.drawio.png](https://cylab.be/storage/blog/366/files/4ouTmPKLmTpM7B9L/FIDO_signed.drawio.png)](https://cylab.be/storage/blog/366/files/4ouTmPKLmTpM7B9L/FIDO_signed.drawio.png)

As shown, the security weak point shifts from relying on easily guessed passwords, like your pet’s name, to the proper management and protection of your private key. This makes it crucial to ensure that the private key is securely stored on your device, as it becomes the key to accessing your accounts.

## Storing private keys[](https://cylab.be/blog/366/a-brief-overview-of-passkey#storing-private-keys)

There are two primary methods for storing these private keys:

- **Embedded (or bound) authenticator** : This keeps the private key securely on the user’s device, managed by a password manager that can handle multiple keys and streamline the login process.
- **Roaming authenticator**: In this method, the private key is stored on a separate, portable device, such as a hardware token or a smartphone, which can authenticate across multiple devices.

### Embedded authenticator

The key to this new authentication technique lies in securely managing the private key. The simplest method is to use a password manager, which stores the private key in an encrypted format on your device. This encrypted key can be unlocked with a password or PIN. Although this approach is more secure than traditional passwords—since an attacker would need physical access to your device and knowledge of your password or PIN—it still carries some risk. If someone gains access to your device and knows your PIN or password, they could potentially log in as you.

The second option for securing the private key is to use biometric authentication, such as a fingerprint or facial recognition. With biometrics, only you can authenticate, as it’s directly tied to something unique about you. This method enhances security by ensuring that your private key is accessible only with your unique biological features. Importantly, your biometric data is **never** transmitted to the online service, so your privacy remains protected.

### Roaming authenticator

A roaming authenticator is simply an external device used to securely store your private key. One of the most well-known examples of this is a FIDO key, such as a YubiKey. In this setup, the roaming authenticator acts like a wallet for your passkeys, meaning you’ll need to take extra care in managing it.

With a roaming authenticator, you authenticate by physically possessing the key and entering a PIN to unlock the private key. This adds a layer of security by combining something you have (the physical key) with something you know (the PIN). This method ensures that even if someone gains access to your PIN, they’d still need the physical key to log in as you.

## Passkey Advantages[](https://cylab.be/blog/366/a-brief-overview-of-passkey#passkey-advantages)

You might wonder what advantages passkeys offer if you still need to remember a PIN or password to unlock your private key.

The real benefit of passkeys lies in the underlying protocol we discussed earlier. Unlike traditional passwords, there is no exchange of secrets between the user and the web service. Instead, only a proof of ownership (the signature) is sent to the online service. This design significantly reduces the risk of phishing attacks, as there’s no sensitive information that can be intercepted.

However, there are some drawbacks. If you don’t have access to your private key wallet—whether it’s a roaming authenticator or an on-device manager—you won’t be able to log in. On the flip side, this means that no one else can log in without your passkey wallet.

Lastly, because no secret information is stored on the online service, a data breach on their side doesn’t compromise your security. You no longer need to worry about checking sites like [Have I Been Pwned](https://haveibeenpwned.com/) to see if your credentials have been leaked.

## Conclusion[](https://cylab.be/blog/366/a-brief-overview-of-passkey#conclusion)

To summarize the advantages of passkeys:

- **Phishing Resistant**: Passkeys eliminate the risk of phishing attacks, as there are no secrets exchanged between the user and the service.
- **Brute Force/Password Spraying Resistant**: Without a password to guess, attackers have no opportunity to brute force or spray passwords to gain access.
- **Data Breach Protection**: They do not expose any sensitive information in the event of a data breach, keeping your accounts safe.
- **Tied to Unique Attributes**: Passkeys are linked to something you possess (like a device) or are (such as your biometrics), adding an extra layer of security.

However, it’s important to note that this is still a relatively new technology, so it may not yet be available on all the services you use.