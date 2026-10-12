---
link: https://nonfunctional.substack.com/p/passkeys-are-a-trap-do-not-use-them
byline: The Hack
site: Non-Functional
date: 2026-10-10T21:33
excerpt: "Intended audience: non-technical citizens of the web"
slurped: 2026-10-12T08:09
title: Passkeys are a trap. Do not use them.
---

Seems like every website wants us to stop using passwords. There is a replacement we are being asked to use instead called _passkeys_. This should be a tad concerning if passwords are familiar to you and you don’t fully understand these nebulous passkeys.

Websites say that passkeys are _more secure_ or _more convenient_ or something similarly beneficial. While it is true that passkeys can be far more secure than passwords, and may indeed be more convenient, that’s not the whole story. Passkeys are not designed to prioritize your interests. Like the Trojan Horse, passkeys are legitimately elegant on the surface, but there is something troubling inside.

What’s important to know is that unlike with any other kind of login credential, **you do not have ultimate control over your passkeys**. With passwords, you can choose where to write them down; with authenticator apps, you can decide for yourself which ones to use (they’re pretty interchangeable); and with physical security keys you can hand them to your colleague if you really trust them. But with passkeys come a hidden dial that lets the websites you use them on gradually remove your ability to do as you please with them.

I will explain how that is possible further down, but let’s first take a look at who wants you to be using passkeys.

Passkeys are the invention of the [FIDO Alliance](https://en.wikipedia.org/wiki/FIDO_Alliance),[1](https://nonfunctional.substack.com/p/passkeys-are-a-trap-do-not-use-them#footnote-1) which is a consortium of a number of different competing interests – Big Tech, little tech, national governments, intelligence agencies, banks, various public institutions, and even a handful of nonprofits like Mozilla – who are all able to agree that passkeys are a good and righteous cause.

The members of the FIDO Alliance all have their own interests. Big Tech wants to bring little inconveniences into our life so they can sell us more ads. Intelligence agencies want to track everyone in the world. Banks want to be free of indemnity. Little tech wants cheap solutions to reduce liability. Mozilla wants to avoid having to die on any particular hill.[2](https://nonfunctional.substack.com/p/passkeys-are-a-trap-do-not-use-them#footnote-2) I make no particular claim that any of these are bad motivations. But it is notable that no consumer advocacy groups are members of the FIDO Alliance. (This stands in contrast to, for example, the [World Wide Web Consortium](https://en.wikipedia.org/wiki/World_Wide_Web_Consortium), the organization that decides the standards used on the web. Alongside the usual corporate members it has not only public interest groups but also universities and Tim Berners-Lee, the British computer scientist who invented the World Wide Web in 1989. Honestly, if Gandalf himself were to join the W3C he would not be out of place.) Regarding the FIDO Alliance, nobody is there to make a strong case for the user’s own interests.

The reason presented to the public for why everyone needs to use passkeys now is that passwords are insecure. On this point I agree. Consider that the servers behind the billion-dollar American IT company SolarWinds [were famously found](https://www.cnn.com/2021/02/26/politics/solarwinds123-password-intern/) to be using `solarwinds123`. Hawaii’s Emergency Management Agency (the same one that sent out that false missile alert) [accidentally revealed a password](https://www.businessinsider.com/hawaii-emergency-agency-password-discovered-in-photo-sparks-security-criticism-2018-1?op=1) due to a sticky note on someone’s desk in the background of a photograph. If your security relies on a password, you will always be anxious that somebody, somewhere, has somehow figured out what it is and already has access to your bank account.

On the other hand: what passwords have going for them is that they’re very intuitive, everyone knows how they work, people have been using them since time immemorial, and you can choose where you store them and who you share them with. Keep this in mind.

(Also keep this in mind: Netflix does not want you to share your passwords.)

But there already exist superior security mechanisms to passwords, to which the FIDO Alliance seems to be giving the cold shoulder: first there was _Two-Factor Authentication_ (in particular: receiving a text message or email with a one-time code), and later came the more robust system of _Time-Based One-Time Passwords_ (TOTP) – in common parlance, “authenticator apps._”_

What’s neat about TOTP is that it doesn’t rely on text messages or email or even the internet at all. Your authenticator software simply generates a random-looking number every 30 seconds based on the current time (and a random starting number called a “seed” or a “key” that is unique to you), and this is all that a website needs to verify you’re you. The term _authenticator app_ is misleading, not in the least because websites tend to heavily imply you need to download a _specific_ app, such as Google Authenticator. In reality, there is an open standard for TOTP (called [RFC 6238](https://en.wikipedia.org/wiki/Time-based_one-time_password)); that is to say, the formula used to generate these random-looking numbers is public knowledge, so you don’t need to use any one specific piece of software. In almost all cases you don’t need to use Google Authenticator just because the website suggests it. You don’t even need to use a smartphone; you can use your laptop or even a Game Boy[3](https://nonfunctional.substack.com/p/passkeys-are-a-trap-do-not-use-them#footnote-3) from the 90s for this purpose. You can use any software and hardware you like, so long as it can tell time decently and can handle a few lines of code. You can even write the software yourself in Python if you want, or if you feel like burning a moticum of jet fuel you can get an LLM to write it for you.

TOTP is a great solution. If you accidentally leak your TOTP in a photograph of your desk, you’re only at risk for a couple of minutes until the code expires. TOTP is certainly less intuitive than passwords, but at least like passwords you can still choose how you want to store them. You can choose where you want them to live and what software you want to use to generate the codes. And you can’t easily shoot yourself in the foot with an easy-to-guess TOTP code like you can with a `password123`; as they are nothing more than random-looking numbers, all TOTP codes are equally hard to guess. Oh, what’s more, you can send a code to your friend to let them log into your Netflix account temporarily while you’re away on business. You can even share your TOTP seed itself if you really want to so that your friends can generate your TOTP codes themselves and then they can let _their_ friends log into your Netflix account. You might get some strange looks if you write all your TOTP seeds down on a piece of paper like you do with your passwords, but at the end of the day this data is _yours_ to handle how you like. Aren’t open standards wonderful?

(Netflix does not want you to share your TOTP keys either.)

So what makes _passkeys_ so different? Well, they look very similar to TOTP on the surface. Both have an open standard (RFC 6238 for TOTP, WebAuthn for passkeys), so surely passkeys must have all these same nice properties as TOTP, right? At first glance it would seem so. Indeed, just now I generated a passkey using free and open source software, and then I emailed it to a friend, and now we can both access the same website using that passkey. At no point did I have to ask anyone for permission, not the website, not the FIDO Alliance; not even my friend, who kindly let me monopolize her time without objection. What’s more, I don’t have to type any numbers by hand to use a passkey; I don’t have to remember any magic words; passkeys just work. Judging from that alone, a passkey is indeed convenient. And it sure _seems_ like I can do whatever I want with my passkeys.

(But Netflix does not want you to share your passkeys.)

So where’s the catch?

The catch is that there is something else that is also called a “passkey” and it’s nothing like the nice kind that I just described.

In fact there are many different things called “passkeys” and the FIDO Alliance has intentionally named all of them the same thing. I believe this was done in order to deliberately confuse users and encourage them not to look too closely at the fine text. Indeed, in the technical documentation, the WebAuthn standard that defines passkeys barely uses the term _passkey_ at all. Instead there are _discoverable credentials_ and _undiscoverable credentials_; there are _hardware-backed credentials_, and _software-backed credentials_; there are _attested_ and _unattested_ credentials, and more and more.

There are so many optional components to what a passkey can be that the specs read less like a standard and more like a parts catalogue. So, let’s go over exactly what other things are in the catalogue alongside the nice and perhaps-almost-intuitive kind of passkey I described earlier. I will present them all in the best light I can manage so that you can understand why some people genuinely recommend passkeys as a good security solution, but by the end of the list I hope you will be able to see the problem I see.

Both kinds exist.

When you generate a passkey, it can be made to be _non-copyable_. The reason you might want this as a user is to be sure that so long as you have possession of your passkey – so long as your iPhone is in your pocket, say – nobody else can access your account. On the other hand, passkeys can be made to be _copyable_ instead, and you might want this ability in order to share one key between your devices.

If you make a passkey, make sure you know which of these two kinds you’re making! If you think your passkey is one of these and it’s actually the other, you will end up having a bad time someday – either when you realize you can’t rescue your only passkey off your dying laptop, or when somebody else turns out to have a copy of your passkey and you didn’t know that was possible.

Both kinds exist.

Surprisingly, even if a Passkey is _non-copyable_, it can still be _transferable_. This may violate your intuition about how data is supposed to work. Although it’s how physical objects have always worked, we don’t often encounter situations in the _digital_ world where sending something to your friend causes it to disappear from your own device.

You might want your passkey to be non-transferable in order to be completely sure that physical possession of one specific device is absolutely required to access your account. On the other hand, you might want your non-copyable passkey to be transferable in order for it to be able to move with you when you replace your smartphone for next year’s model. (Or maybe you just want to be able to play some kind of avant-garde digital relay race among your friends as an artistic statement.)

Both kinds exist.

Even if you’re not digitally savvy, you probably know what _files_ and _folders_ are, at least if you have a desktop computer. Admittedly, smartphones are designed to muddy the waters regarding where exactly that file you downloaded is and if _folders_ are even a copacetic concept. (Yes, smartphones _do_ use files and folders. They just don’t typically come with a file browser like your desktop does.)

So all these passkeys we’ve talked about so far are just files right? Well… not really. That is, they _can_ be. Or they might not be. As with all these other qualities a passkey can optionally have, passkeys can optionally not be files at all. Indeed, since files are copyable and transferable and all sorts of other intuitive things, if passkeys had to be files, it would not be clear how anyone could make a non-copyable passkey.

Enter _hardware-backed_ passkeys. These make use of a secure chip that comes with all modern devices which can store data that cannot be exfiltrated (meaning the data cannot leave the safety of the secure chip). There are different kinds of chips with different names: your laptop has a Trusted Platform Module and an iPhone has a “Secure Enclave” and so on. You will not find this area of your device in your file browser anywhere.[4](https://nonfunctional.substack.com/p/passkeys-are-a-trap-do-not-use-them#footnote-4) It is a separate component entirely. Because of their cryptographic use case (and perhaps their mystique), people have taken to calling these chips “cryptoprocessors.”

The consensus seems to be that cryptoprocessors are indeed very secure. You might wonder if someone with the know-how couldn’t just surgically open the chip with the right tools and extract the data off that way. That’s _theoretically_ possible; however, such chips employ all kinds of chemical and electronic tamper-detection measures and will self-destruct if you try to do this.[5](https://nonfunctional.substack.com/p/passkeys-are-a-trap-do-not-use-them#footnote-5) So unless you have the resources of a nation-state at your disposal, you can basically take as a given that (hypothetical manufacturer backdoors notwithstanding) nobody can remove the secret data from your cryptoprocessor; not Netflix, not your operating system, not even you.

You might prefer to use a software-backed key because you understand how files work and you already have a solution that works for you for storing your private data. You might want to use a hardware-backed key to get the extra security afforded by a nigh-impenetrable cryptoprocessor.

Both kinds exist.

Some passkeys require somebody to press a button or click a prompt to confirm authentication. Others can be used without anyone in the loop at all.

You might want the former kind because you yourself are a human. You might want the latter kind in case you want to authorize a piece of software to act on your behalf.

Both kinds exist.

Okay so maybe _a_ user was present, but how do we know _the right_ user was present? Passkeys can optionally require _biometrics_ (e.g. fingerprint sensors or retina scanners) or the use of PINs or passwords of their own in order to be extra sure that only one specific intended human being can use them.

You might want a user-verified passkey in order to be safe even in the situation a thief pilfers your device. You might want a non-user-verified passkey because you expect to lend your tablet to another family member so they can read your ebooks on Amazon and listen to music on your Spotify account and watch your movies on Disney+.

This is where things take a left turn.

I’ve presented all these properties so far as being _up to you to decide_. That may indeed be how it is on most websites – for now, at least. After all, what cause does Twitter have to tell me exactly which security measures out of the smorgasbord available I should opt into for my passkeys? But you should know that the FIDO Alliance has defined this standard such that your passkey software is required to tell the service you’re accessing _which_ of these features are enabled. They’re called _authenticator data flags_ in the WebAuthn standard.[6](https://nonfunctional.substack.com/p/passkeys-are-a-trap-do-not-use-them#footnote-6)

This means Twitter (formally X) can say to you, “sorry, we only accept the User-Present User-Verified Non-Copyable Hardware-Backed varieties of passkey here. But hey! You can still choose between Transferable and Non-Transferable passkeys!” Actually, I’m being optimistic – I suspect they are unlikely to phrase it in such clear terms. They will likely just say “unfortunately your passkey isn’t secure enough; download our app.”

But it’s an open standard, isn’t it? And maybe you have a friend experienced in writing Python code. Can’t you just make your own passkey software which just does the nice file-based kind of passkeys with none of those extra security features, and then have it tell the website anyway that it’s a User-Present User-Verified Non-Copyable Hardware-Backed passkey? Can’t you just _lie_?

Oh yes absolutely you can. Your passkey software can lie about whatever passkey features you like. Unless we’re talking about an _Attested_ User-Present User-Verified Non-Copyable Hardware-Backed passkey. In which case, no. That can’t be faked.

_Attested_ passkeys[7](https://nonfunctional.substack.com/p/passkeys-are-a-trap-do-not-use-them#footnote-7) are the last but most important type of passkeys you need to know about. These types of keys use what’s known as _remote attestation_ to cryptographically verify – using the aforementioned tamper-proof cryptoprocessor that lives in all your modern devices – that you are using one of the several _approved_ passkey managers, and _not_ using a custom one that you vibecoded yourself and which can lie.

This is the feature that means you no longer get to make the calls about your passkeys. Fortunately, attested passkeys are not yet required on most websites. But the FIDO Alliance standard makes attestation a part of the passkey ecosystem, whether you want it or not. And even if the website doesn’t require attestation now, simply by using passkeys you indicate to the website that your setup supports the FIDO Alliance’s passkey standard and that you as a user _are capable_ of using passkeys.[8](https://nonfunctional.substack.com/p/passkeys-are-a-trap-do-not-use-them#footnote-8) This means that once Twitter (formally X) is satisfied that the overwhelming majority of its users would be unaffected, they can flip the switch – perhaps not all at once, perhaps just for ten thousand users at a time so as not to cause a fuss – and start rejecting non-attested passkeys.

Critically, remote attestation requires the co-operation of your cryptoprocessor, your passkey manager software, your device’s operating system, and your device’s firmware to all pass the remote attestation check. After all, your phone’s operating system has total access to all the 0s and 1s in every app that’s ever run, so it’s not enough for Twitter (formally X) to trust your passkey manager if it can’t even trust your phone’s underlying operating system. If any single one of those things were to fail – say, because your phone stopped receiving security updates so you installed a custom operating system like LineageOS to keep yourself safe without having to buy a new device – then Twitter (formally X) will be able to notice and prevent you from logging in.

I’d like to be clear that there are legitimate uses of remote attestation. By using a website that requires attested passkeys you can be sure you stay safe from accidentally installing a scammer’s fake passkey application that steals your data – the website will reject such a passkey. You might want an attested Passkey even if the website doesn’t require it, since you can use attestation to verify that your software has not been tampered with by a hacker while you weren’t looking. (Of course, attestation may be overkill for this; even if your device _were_ hacked, the hardware-backing feature described above makes it impossible for the passkey to be exfiltrated from your device. Though admittedly that guarantee is less reassuring once you realize that even if the key stays firmly on your device a hacker could still use it if they have control of your phone otherwise. At the very least you can always power off the device to temporarily deny hackers access to your Non-Attested Hardware-Backed Non-Copyable Non-Transferable passkeys.)[9](https://nonfunctional.substack.com/p/passkeys-are-a-trap-do-not-use-them#footnote-9)

As for reasons to _not_ want attestation: you might want to be able to use software that Twitter (formally X) doesn’t specifically approve of, such as Linux. Or a phone with a refurbished operating system. (Or if you’re nefarious you might want to be able to lie to Netflix and tell them that _of course_ you and your three friends are absolutely all the same person using the same device.)

(But with attestation, Netflix can finally prevent you from sharing your passkeys.)

There is ample reason for you to be hesitant before you set up a passkey. If you’ve used only websites that require one certain kind of passkey then you might not notice when a website has you set up a different kind of passkey. Don’t forget that unlike passwords and SMS codes and TOTP, passkeys are designed to function in the background without you ever seeing the details. They’ll offer you at most a “confirm” prompt. (That’s the convenience aspect.) My expectation is that the industry will start off with very few constraints on your passkeys, and gradually add more and more, not all at the same time, but staggered on different sites, and then in a few years people will forget that they were ever offered a choice in which security measures they want to enable on their passkeys (if they even knew there were different kinds of passkeys at all).

So far, almost nobody has even heard of remote attestation. The vast majority of people use one of the four major operating systems (Windows, Android, macOS, and iOS) and not alternative operating systems like Linux, LineageOS, postmarketOS, or GrapheneOS. The overwhelming majority of people don’t replace the operating system on their phone when it stops receiving updates; instead they just buy a new phone. So almost nobody will ever encounter a situation where their device fails a remote attestation check.

Are attested passkeys so bad, though? After all, we really aren’t meant be sharing Netflix accounts to begin with.

Take another look at the list of members in the FIDO Alliance, and put on your tinfoil hat. Why do intelligence agencies care if I am sharing my Netflix account? Heck, why does Microsoft care if I am sharing my Netflix account?

For intelligence agencies, and for corporations like Microsoft, Google, Apple, Facebook, Amazon – giving you convenience and security is likely not their primary goal. I think they have an entirely ulterior motive. **They can use passkeys to prevent you from running unapproved software**.

Remote attestation can allow them to verify that your device is not capable of, say, sending and receiving encrypted messages, or running a competitor’s app, or using social media without verifying your ID. They don’t want you to be able to turn off the mandatory built-in AI image scanner on your phone that scans everything your phone sees for nudity. ([Yes, I’m serious.](https://www.gov.uk/government/news/new-plans-to-stop-children-taking-sharing-or-viewing-nude-images))

But I rather like deciding for myself what software I approve of and what software I don’t. Something I value about my web browser is that I can install extensions to modify aspects of websites that bother me. I can say no to auto-playing videos on news websites; I can decide which advertisements are too intrusive for me to let through; I can filter out irrelevant search results and disable AI summaries; I can block the “recommended video” feature on YouTube which always seems to promote outrage and division (or as Big Tech calls it, “engagement”). And if a website ends up monopolizing my time anyway, I have an extension that locks them down after a certain span of time. These extensions give me control over how the internet affects me, even if Big Tech would much rather make those decisions for me. But Google owns Google Chrome, which is the world’s most popular web browser, and they can remove support for these useful extensions ([as they did earlier this year](https://www.androidauthority.com/ublock-origin-final-chrome-update-3703164/) with uBlock Origin); and with remote attestation they can prevent me from using any other browser that doesn’t follow suit.

Remember those authenticator data flags I mentioned? There used to be only 4 of them. Earlier revisions of the FIDO Alliance’s security standard (called “WebAuthn Level 1” and “WebAuthn Level 2”) had fewer catalog options to choose from. I’m sweeping some details[10](https://nonfunctional.substack.com/p/passkeys-are-a-trap-do-not-use-them#footnote-10) under the rug here for brevity, but since you are given four distinct binary choices, you could call that 16 different kinds of passkey. In 2022 the FIDO Alliance added some more (“WebAuthn Level 3”), so that websites could know whether or not your credentials are copyable, or if backups of your credentials might exist somewhere. There are still two flags left “reserved for future use.” What might our passkeys require of us in the future?

I’m not one to argue about slippery slopes, so I won’t claim that the campaigns to protect children online today will tomorrow turn into laws designed to make it easy to identify political dissidents. But I will say that the widespread adoption of the passkey technology being advertised to you as convenient and secure is capable of permitting such laws to be easily enforced.

So if you care about your privacy and the privacy of others, if you believe people should be free to choose what software runs on their devices, that people should be allowed to learn how to write their own software and run it on their own devices, and that friends should be able to privately talk politics online without government interference, then I would ask you to reject these dozens of different security mechanisms which are all confusingly branded as the same “passkeys.” Wait for Big Tech to propose something just as convenient and secure but more clearly defined and with fewer strings attached. In the meantime if you want security, your Game Boy can provide you with TOTP.[11](https://nonfunctional.substack.com/p/passkeys-are-a-trap-do-not-use-them#footnote-11)

[1](https://nonfunctional.substack.com/p/passkeys-are-a-trap-do-not-use-them#footnote-anchor-1)

No relation to the Canadian mobile network of the same name.

[2](https://nonfunctional.substack.com/p/passkeys-are-a-trap-do-not-use-them#footnote-anchor-2)

I jest, but Mozilla has a reputation for putting up a fight and then eventually surrendering on issues like these. For instance, they eventually implemented Google’s proprietary _Widevine_ DRM system into the otherwise open-source Firefox browser, as rejecting it might have resulted in users being unable to watch Netflix.

[3](https://nonfunctional.substack.com/p/passkeys-are-a-trap-do-not-use-them#footnote-anchor-3)

Yes, someone [really made TOTP for Game Boy](https://github.com/dmang-dev/totp-gb). I cannot in good conscience actually recommend using this, but it will make you popular at parties. (At my parties in any case.)

[4](https://nonfunctional.substack.com/p/passkeys-are-a-trap-do-not-use-them#footnote-anchor-4)

In practice a hardware-backed key may have a file component too, but it will be encoded in such a way that it’s cryptographically useless without the cryptoprocessor it is made for.

[5](https://nonfunctional.substack.com/p/passkeys-are-a-trap-do-not-use-them#footnote-anchor-5)

Devices for consumers will generally just scramble themselves electronically as their anti-tamper mechanism, but more imaginative booby-traps are rumoured to exist. XEROX once [made a chip](https://techxplore.com/news/2015-09-xerox-parc-chip-shatters-shards.html) that could shatter into tiny shards as a defence measure.

[6](https://nonfunctional.substack.com/p/passkeys-are-a-trap-do-not-use-them#footnote-anchor-6)

The authenticator data flags don’t all line up _exactly_ with the features I presented above, but it’s close enough for the purposes of this blog post. For example, authenticator data flag `UP` is _User-Present_, authenticator data flag `UV` is _User-Verified_, and authenticator data flag `BE` is _Copyability_ (“Backup-Eligible”).

[7](https://nonfunctional.substack.com/p/passkeys-are-a-trap-do-not-use-them#footnote-anchor-7)

Authenticator data flag `AT`, if you were wondering.

[8](https://nonfunctional.substack.com/p/passkeys-are-a-trap-do-not-use-them#footnote-anchor-8)

In an early version of this article I incorrectly implied that there was a _direct_ way for platforms to check what users would be affected in advance of requiring remote attestation. To my knowledge, the WebAuthn standard does not provide any straightforward method allow websites to query whether a particular user’s passkey situation supports remote attestation, at least not without directly asking the user to create a remotely-attested passkey. But in practice, one’s operating system and browser are readily visible to websites, and because the vast majority of users will be on one of a few common setups, the site can make a very good guess about what portion of its users would be excluded by a remote attestation requirement.

[9](https://nonfunctional.substack.com/p/passkeys-are-a-trap-do-not-use-them#footnote-anchor-9)

One use case is particularly intriguing to me. In principle, remote attestation can also be run in reverse, to verify that the server you connect to is not doing anything _you_ don’t want _it_ to be doing (such as stealing your credit card number, or keeping backups of the posts you delete). Sadly, to my knowledge, no major platform has ever implemented “reverse” remote attestation this way.

[10](https://nonfunctional.substack.com/p/passkeys-are-a-trap-do-not-use-them#footnote-anchor-10)

As I understand it, “User-Verified” requires “User Present” to be set, so that would imply 12 distinct flavours. But in practice, security measures outside of the specific flags mentioned (such as requiring biometrics specifically, or non-transferability) are queryable and enforceable via remote attestation, so that actually means there are arbitrarily many passkey restrictions possible already with WebAuthn.

[11](https://nonfunctional.substack.com/p/passkeys-are-a-trap-do-not-use-them#footnote-anchor-11)

Addendum: I feel I have made a [compelling case](https://en.wiktionary.org/wiki/steelman) _for_ passkeys in this article. And I haven’t even pointed out one of their best security features: they are (in most use cases) _domain-bound_, meaning they are resistant to what are called _Man-In-The-Middle attacks_. Passkeys have so many incredible properties that I really _want_ to use and recommend them; it’s very unfortunate that the FIDO Alliance has ruined a good thing by slipping in remote attestation and by making no attempt to ensure the difference between these various kinds of passkeys remains evident to the user.

Fortunately, if you want the level of high security passkeys can give you without the strings attached, there is actually an even _more_ secure solution: security keys, such as a [YubiKey](https://en.wikipedia.org/wiki/YubiKey). These are basically just physical passkeys that you can carry around with you and use alongside your other devices. They are also part of the FIDO Alliance’s WebAuthn standard, but because they are entirely self-contained their remote attestation requirements don’t have to spill out and lock down your whole operating system.


Il semble que tous les sites web veuillent nous faire abandonner les mots de passe. On nous propose un système de remplacement appelé « clé d'accès ». Si vous êtes habitué aux mots de passe et que vous ne comprenez pas bien le fonctionnement de ces clés d'accès, cela peut être un peu inquiétant.

Les sites web affirment que les clés d'accès sont plus sûres, plus pratiques, ou présentent d'autres avantages similaires. S'il est vrai que les clés d'accès peuvent être bien plus sûres que les mots de passe, et effectivement plus pratiques, ce n'est pas tout. Les clés d'accès ne sont pas conçues pour privilégier vos intérêts. À l'instar du cheval de Troie, les clés d'accès sont séduisantes en apparence, mais cachent un problème.

Il est important de savoir que, contrairement à tout autre type d'identifiant de connexion, vous n'avez pas un contrôle total sur vos clés d'accès. Avec les mots de passe, vous pouvez choisir où les noter ; avec les applications d'authentification, vous pouvez choisir celles que vous souhaitez utiliser (elles sont généralement interchangeables) ; et avec les clés de sécurité physiques, vous pouvez les confier à un collègue en qui vous avez pleinement confiance. Mais les mots de passe cachent un mécanisme qui permet aux sites web sur lesquels vous les utilisez de vous en restreindre progressivement le contrôle.

J'expliquerai plus loin comment cela est possible, mais voyons d'abord qui souhaite que vous utilisiez des mots de passe.

Les mots de passe sont une invention de la FIDO Alliance¹, un consortium regroupant divers intérêts concurrents : les géants et les petites entreprises technologiques, les gouvernements, les services de renseignement, les banques, diverses institutions publiques et même quelques organisations à but non lucratif comme Mozilla. Tous s'accordent à dire que les mots de passe sont une bonne chose.

Chaque membre de la FIDO Alliance a ses propres intérêts. Les géants du numérique veulent nous compliquer la vie pour nous vendre plus de publicités. Les services de renseignement veulent surveiller chaque personne dans le monde. Les banques veulent s'exonérer de toute responsabilité. Les petites entreprises technologiques recherchent des solutions bon marché pour réduire leur responsabilité. Mozilla veut éviter de devoir prendre position sur des sujets précis². Je ne prétends pas que ces motivations soient mauvaises. Il est toutefois remarquable qu'aucune association de défense des consommateurs ne soit membre de la FIDO Alliance. (Ceci contraste avec, par exemple, le World Wide Web Consortium (W3C), l'organisation qui définit les normes utilisées sur le web. Outre les entreprises habituelles, le W3C compte parmi ses membres non seulement des associations de défense des consommateurs, mais aussi des universités et Tim Berners-Lee, l'informaticien britannique qui a inventé le World Wide Web en 1989. Franchement, si Gandalf lui-même rejoignait le W3C, il n'y serait pas déplacé.) Concernant la FIDO Alliance, personne ne défend avec vigueur les intérêts des utilisateurs.

La raison invoquée pour justifier l'utilisation généralisée de clés d'accès est l'insécurité des mots de passe. Sur ce point, je suis d'accord. On peut citer l'exemple des serveurs de SolarWinds, la multinationale américaine du secteur informatique, qui utilisaient le mot de passe « solarwinds123 ». L'Agence de gestion des urgences d'Hawaï (celle-là même qui a émis la fausse alerte aux missiles) a accidentellement divulgué un mot de passe à cause d'un post-it collé sur le bureau d'un employé, visible en arrière-plan d'une photo. Si votre sécurité repose sur un mot de passe, vous craindrez toujours que quelqu'un, quelque part, l'ait découvert et ait déjà accès à votre compte bancaire.

En revanche, les mots de passe présentent l'avantage d'être très intuitifs : tout le monde sait comment ils fonctionnent, ils sont utilisés depuis toujours, et vous pouvez choisir où les stocker et avec qui les partager. Gardez cela à l'esprit.

(N'oubliez pas non plus que Netflix ne souhaite pas que vous partagiez vos mots de passe.)

Cependant, il existe déjà des mécanismes de sécurité plus performants que les mots de passe, que l'Alliance FIDO semble ignorer : d'abord l'authentification à deux facteurs (notamment la réception d'un SMS ou d'un e-mail contenant un code à usage unique), puis le système plus robuste des mots de passe à usage unique basés sur le temps (TOTP), communément appelés « applications d'authentification ».
Ce qui est génial avec le TOTP, c'est qu'il ne dépend ni des SMS, ni des e-mails, ni même d'Internet. Votre application d'authentification génère simplement un nombre apparemment aléatoire toutes les 30 secondes, basé sur l'heure actuelle (et un nombre initial aléatoire appelé « graine » ou « clé », qui vous est propre). C'est tout ce dont un site web a besoin pour vérifier votre identité. L'expression « application d'authentification » est trompeuse, notamment parce que les sites web laissent souvent entendre qu'il faut télécharger une application spécifique, comme Google Authenticator. En réalité, le TOTP repose sur une norme ouverte (RFC 6238) ; autrement dit, la formule utilisée pour générer ces nombres aléatoires est publique, vous n'avez donc pas besoin d'utiliser un logiciel particulier. Dans la plupart des cas, il est inutile d'utiliser Google Authenticator simplement parce que le site web le suggère. Vous n'avez même pas besoin d'un smartphone ; un ordinateur portable ou même une Game Boy 3 des années 90 peuvent faire l'affaire. Vous pouvez utiliser le logiciel et le matériel de votre choix, pourvu qu'ils soient capables d'afficher l'heure correctement et de gérer quelques lignes de code. Vous pouvez même écrire le logiciel vous-même en Python si vous le souhaitez, ou, si vous avez envie de dépenser une fortune, vous pouvez faire appel à un développeur.

L'authentification TOTP est une excellente solution. Si vous divulguez accidentellement votre code TOTP sur une photo de votre bureau, vous n'êtes exposé au risque que pendant quelques minutes, jusqu'à son expiration. Certes, TOTP est moins intuitif que les mots de passe, mais au moins, comme pour ces derniers, vous pouvez choisir comment les stocker. Vous pouvez choisir leur emplacement et le logiciel utilisé pour les générer. De plus, contrairement à un mot de passe, vous ne risquez pas de vous faire piéger par un code TOTP facile à deviner ; comme il s'agit simplement de chiffres apparemment aléatoires, tous les codes TOTP sont tout aussi difficiles à deviner. Enfin, vous pouvez envoyer un code à un ami pour qu'il puisse se connecter temporairement à votre compte Netflix pendant votre déplacement professionnel. Vous pouvez même partager votre phrase de récupération TOTP si vous le souhaitez, afin que vos amis puissent générer eux-mêmes vos codes TOTP et permettre à leurs propres amis de se connecter à votre compte Netflix. Vous risquez d'attirer des regards étranges si vous notez toutes vos phrases de récupération TOTP sur un bout de papier, comme vous le faites pour vos mots de passe, mais après tout, ces données vous appartiennent et vous pouvez les gérer comme bon vous semble. Les standards ouverts, c'est formidable, non ?

(Netflix ne souhaite pas non plus que vous partagiez vos clés TOTP.)

Alors, qu'est-ce qui différencie les clés d'accès ? Eh bien, à première vue, elles ressemblent beaucoup aux TOTP. Les deux reposent sur un standard ouvert (RFC 6238 pour les TOTP, WebAuthn pour les clés d'accès), alors les clés d'accès doivent sûrement posséder les mêmes avantages que les TOTP, n'est-ce pas ? À première vue, on pourrait le croire. D'ailleurs, je viens de générer une clé d'accès à l'aide d'un logiciel libre et gratuit, puis je l'ai envoyée par e-mail à un ami, et maintenant nous pouvons tous les deux accéder au même site web grâce à cette clé. À aucun moment je n'ai eu à demander la permission à qui que ce soit, ni au site web, ni à la FIDO Alliance, ni même à mon amie, qui a pourtant gentiment accepté que j'utilise son temps sans rechigner. De plus, je n'ai pas besoin de saisir de chiffres manuellement pour utiliser un code d'accès ; je n'ai pas besoin de me souvenir de mots magiques ; les codes d'accès fonctionnent tout simplement. À ce seul titre, un code d'accès est effectivement pratique. Et il semble bien que je puisse faire ce que je veux avec mes codes d'accès.

(Mais Netflix ne souhaite pas que vous partagiez vos codes d'accès.)

Alors, où est le piège ?

Le piège, c'est qu'il existe autre chose appelée aussi « code d'accès », et ce n'est en rien comparable à celui que je viens de décrire.

En fait, il existe de nombreuses choses différentes appelées « codes d'accès », et la FIDO Alliance les a intentionnellement toutes nommées de la même façon. Je pense que cela a été fait délibérément pour embrouiller les utilisateurs et les inciter à ne pas trop regarder les petits caractères. En effet, dans la documentation technique, la norme WebAuthn qui définit les clés d'accès utilise à peine le terme « clé d'accès ». On y trouve plutôt des identifiants découvrables et des identifiants non découvrables ; des identifiants matériels et des identifiants logiciels ; des identifiants attestés et non attestés, et ainsi de suite.

Les composants optionnels d'une clé d'accès sont si nombreux que les spécifications ressemblent moins à une norme qu'à un catalogue de pièces détachées. Voyons donc précisément quels sont les autres éléments disponibles, en plus de la clé d'accès simple et intuitive que j'ai décrite précédemment. Je les présenterai tous sous leur meilleur jour afin que vous compreniez pourquoi certains recommandent sincèrement les clés d'accès comme une bonne solution de sécurité. J'espère qu'à la fin de cette liste, vous comprendrez le problème que je perçois.

Clés d'accès copiables et non copiables

Les deux types existent.

Lorsqu'une clé d'accès est générée, elle peut être conçue pour être non copiable. La raison pour laquelle vous pourriez souhaiter cela en tant qu'utilisateur est de vous assurer que tant que vous êtes en possession de votre clé d'accès (par exemple, tant que votre iPhone est dans votre poche), personne d'autre ne peut accéder à votre compte. En revanche, les clés d'accès peuvent être configurées pour être

### Clés d'accès copiables et non copiables

Les deux types existent.

Lorsque vous générez un mot de passe, il peut être rendu _non copiable_. La raison pour laquelle vous souhaiterez peut-être cela en tant qu'utilisateur est d'être sûr que tant que vous êtes en possession de votre mot de passe – tant que votre iPhone est dans votre poche, par exemple – personne d'autre ne peut accéder à votre compte. D'un autre côté, les clés d'accès peuvent être rendues _copiables_ à la place, et vous souhaiterez peut-être cette capacité afin de partager une clé entre vos appareils.

Si vous créez un mot de passe, assurez-vous de savoir lequel de ces deux types vous créez ! Si vous pensez que votre mot de passe est l’un de ces éléments et qu’il s’agit en réalité de l’autre, vous finirez par passer un mauvais moment – ​​soit lorsque vous réaliserez que vous ne pouvez pas récupérer votre seul mot de passe sur votre ordinateur portable mourant, soit lorsque quelqu’un d’autre s’avère avoir une copie de votre mot de passe et que vous ne saviez pas que c’était possible.

### Clés d'accès transférables et non transférables

Les deux types existent.

Étonnamment, même si un mot de passe est _non copiable_, il peut toujours être _transférable_. Cela peut violer votre intuition sur la façon dont les données sont censées fonctionner. Bien que les objets physiques aient toujours fonctionné ainsi, nous ne rencontrons pas souvent de situations dans le monde _numérique_ où l'envoi de quelque chose à votre ami le fait disparaître de votre propre appareil.

Vous souhaiterez peut-être que votre mot de passe ne soit pas transférable afin d'être complètement sûr que la possession physique d'un appareil spécifique est absolument requise pour accéder à votre compte. D’un autre côté, vous souhaiterez peut-être que votre mot de passe non copiable soit transférable afin qu’il puisse vous accompagner lorsque vous remplacerez votre smartphone par le modèle de l’année prochaine. (Ou peut-être voulez-vous simplement pouvoir jouer une sorte de course de relais numérique avant-gardiste entre vos amis en guise de déclaration artistique.)

### Clés d'accès matérielles et logicielles

Les deux types existent.

Même si vous n'êtes pas doué en numérique, vous savez probablement ce que sont les _fichiers_ et les _dossiers_, du moins si vous possédez un ordinateur de bureau. Certes, les smartphones sont conçus pour brouiller les cartes quant à l'endroit exact où se trouve le fichier que vous avez téléchargé et si les _dossiers_ sont même un concept copacétique. (Oui, les smartphones _utilisent_ des fichiers et des dossiers. Ils ne sont généralement pas livrés avec un navigateur de fichiers comme le fait votre bureau.)

Donc tous ces mots de passe dont nous avons parlé jusqu’à présent ne sont que des fichiers, n’est-ce pas ? Eh bien… pas vraiment. Autrement dit, ils _peuvent_ l'être. Ou peut-être qu’ils ne le sont pas. Comme pour toutes ces autres qualités qu'un mot de passe peut éventuellement avoir, les mots de passe peuvent éventuellement ne pas être du tout des fichiers. En effet, étant donné que les fichiers sont copiables et transférables et toutes sortes d'autres choses intuitives, si les clés d'accès devaient être des fichiers, il ne serait pas clair comment quiconque pourrait créer une clé d'accès non copiable.

Entrez les clés d'accès _sécurisées par le matériel_. Ceux-ci utilisent une puce sécurisée fournie avec tous les appareils modernes et capable de stocker des données qui ne peuvent pas être exfiltrées (ce qui signifie que les données ne peuvent pas quitter la sécurité de la puce sécurisée). Il existe différents types de puces portant des noms différents : votre ordinateur portable possède un module de plateforme sécurisée et un iPhone possède un « Secure Enclave », etc. Vous ne trouverez nulle part cette zone de votre appareil dans votre navigateur de fichiers.[4](https://nonfunction.substack.com/p/passkeys-are-a-trap-do-not-use-them#footnote-4) Il s'agit d'un composant entièrement distinct. En raison de leur cas d’utilisation cryptographique (et peut-être de leur mystique), les gens ont commencé à appeler ces puces « cryptoprocesseurs ».

Le consensus semble être que les cryptoprocesseurs sont effectivement très sécurisés. Vous vous demandez peut-être si une personne possédant le savoir-faire ne pourrait pas simplement ouvrir chirurgicalement la puce avec les bons outils et en extraire les données de cette façon. C'est _théoriquement_ possible ; cependant, ces puces utilisent toutes sortes de mesures de détection de falsification chimiques et électroniques et s'autodétruiront si vous essayez de le faire. cryptoprocesseur ; pas Netflix, pas votre système d'exploitation, pas même vous.

Vous préférerez peut-être utiliser une clé sauvegardée par logiciel, car vous comprenez comment fonctionnent les fichiers et vous disposez déjà d'une solution qui vous convient pour stocker vos données privées. Vous souhaiterez peut-être utiliser une clé matérielle pour bénéficier de la sécurité supplémentaire offerte par un cryptoprocesseur presque impénétrable.

### Clés d'accès présentes par l'utilisateur et non présentes par l'utilisateur

Les deux types existent.

Certains mots de passe nécessitent que quelqu'un appuie sur un bouton ou clique sur une invite pour confirmer l'authentification. D’autres peuvent être utilisés sans que personne ne soit impliqué dans la boucle.

Vous voudrez peut-être le premier type parce que vous êtes vous-même un humain. Vous souhaiterez peut-être ce dernier type au cas où vous souhaiteriez autoriser un logiciel à agir en votre nom.

### Clés d'accès vérifiées par l'utilisateur et non vérifiées par l'utilisateur

Les deux types existent.

D'accord, alors peut-être que _un_ utilisateur était présent, mais comment savoir si _le bon_ utilisateur était présent ? Les clés d'accès peuvent éventuellement nécessiter des _biométries_ (par exemple, des capteurs d'empreintes digitales ou des reti

Clés d'accès vérifiées par l'utilisateur vs. clés d'accès non vérifiées

Les deux types existent.

Imaginons qu'un utilisateur soit présent, mais comment s'assurer qu'il s'agit bien de la bonne personne ? Les clés d'accès peuvent exiger, en option, des données biométriques (par exemple, un capteur d'empreintes digitales ou un scanner rétinien) ou l'utilisation d'un code PIN ou d'un mot de passe spécifique afin de garantir qu'une seule personne autorisée puisse les utiliser.

Vous pourriez préférer une clé d'accès vérifiée par l'utilisateur pour être protégé même en cas de vol de votre appareil. Vous pourriez également opter pour une clé d'accès non vérifiée si vous prévoyez de prêter votre tablette à un membre de votre famille pour qu'il puisse lire vos livres numériques sur Amazon, écouter de la musique sur votre compte Spotify et regarder vos films sur Disney+.

Clés d'accès certifiées vs. clés d'accès non certifiées

C'est là que les choses se compliquent.

J'ai présenté ces différentes caractéristiques en vous laissant le choix. C'est probablement le cas sur la plupart des sites web, du moins pour l'instant. Après tout, de quel droit Twitter me dicte-t-il les mesures de sécurité à choisir parmi la multitude d'options disponibles pour mes clés d'accès ? Sachez toutefois que la FIDO Alliance a défini une norme exigeant que votre logiciel de gestion des clés d'accès indique au service auquel vous accédez quelles fonctionnalités sont activées. Dans la norme WebAuthn, on les appelle « indicateurs de données d'authentification ».⁶

Cela signifie que Twitter (anciennement X) peut vous dire : « Désolé, nous n'acceptons ici que les clés d'accès de type : présence de l'utilisateur, vérification par l'utilisateur, non copiable et reposant sur un matériel. Mais rassurez-vous ! Vous pouvez toujours choisir entre une clé d'accès transférable et une clé d'accès non transférable !» En réalité, je suis optimiste : je doute qu'ils le formulent aussi clairement. Ils diront probablement simplement : « Malheureusement, votre clé d'accès n'est pas suffisamment sécurisée ; téléchargez notre application.»

Mais c'est une norme ouverte, n'est-ce pas ? Et peut-être avez-vous un ami qui maîtrise la programmation Python. Ne pourriez-vous pas simplement créer votre propre logiciel de gestion de mots de passe, utilisant uniquement des fichiers sans aucune sécurité supplémentaire, et faire en sorte qu'il indique au site web qu'il s'agit d'un mot de passe « présent, vérifié, non copiable et protégé par matériel » ? Ne pourriez-vous pas mentir ?

Bien sûr que si. Votre logiciel peut falsifier les informations relatives aux mots de passe. Sauf s'il s'agit d'un mot de passe « attesté, présent, vérifié, non copiable et protégé par matériel ». Dans ce cas, non. Impossible de le falsifier.

Les mots de passe attestés sont le dernier type de mots de passe, mais non le plus important, à connaître. Ils utilisent l'attestation à distance pour vérifier cryptographiquement – ​​grâce au cryptoprocesseur inviolable présent dans tous vos appareils modernes – que vous utilisez bien un gestionnaire de mots de passe agréé, et non un gestionnaire personnalisé créé manuellement et susceptible de mentir.

C’est cette fonctionnalité qui signifie que vous ne contrôlez plus vos mots de passe. Heureusement, la plupart des sites web n’exigent pas encore de mots de passe certifiés. Cependant, la norme FIDO Alliance intègre la certification à l’écosystème des mots de passe, que vous le vouliez ou non. Même si un site web n’exige pas de certification actuellement, le simple fait d’utiliser des mots de passe indique qu’il est compatible avec la norme FIDO Alliance et que vous, en tant qu’utilisateur, êtes capable de les utiliser.⁸ Cela signifie que lorsque Twitter (anciennement X) sera convaincu que la grande majorité de ses utilisateurs ne sera pas affectée, il pourra activer cette mesure – progressivement, par exemple pour dix mille utilisateurs seulement afin d’éviter toute perturbation – et commencer à refuser les mots de passe non certifiés.

Point essentiel : la certification à distance nécessite la coopération de votre processeur de chiffrement, de votre logiciel de gestion des mots de passe, du système d’exploitation et du micrologiciel de votre appareil pour réussir le contrôle de certification à distance. Après tout, le système d'exploitation de votre téléphone a un accès complet à toutes les données (0 et 1) de chaque application exécutée. Il ne suffit donc pas à Twitter (anciennement X) de faire confiance à votre gestionnaire de mots de passe s'il ne peut même pas faire confiance au système d'exploitation sous-jacent de votre téléphone. Si l'un de ces éléments venait à faillir – par exemple, si votre téléphone ne recevait plus de mises à jour de sécurité et que vous installiez un système d'exploitation personnalisé comme LineageOS pour vous protéger sans avoir à acheter un nouvel appareil – Twitter (anciennement X) pourrait le détecter et vous empêcher de vous connecter.

Je tiens à préciser que l'attestation à distance a des usages légitimes. En utilisant un site web exigeant des mots de passe attestés, vous vous assurez de ne pas installer accidentellement une application frauduleuse utilisant un faux mot de passe et susceptible de voler vos données : le site web rejettera un tel mot de passe. Vous pourriez souhaiter un mot de passe attesté même si le site web ne l'exige pas, car l'attestation vous permet de vérifier que votre logiciel n'a pas été altéré par un pirate informatique à votre insu. (Bien sûr, l'attestation est peut-être excessive dans ce cas ; même si votre appareil était piraté, la fonctionnalité de protection matérielle décrite ci-dessus empêche la divulgation de la clé d'accès.)