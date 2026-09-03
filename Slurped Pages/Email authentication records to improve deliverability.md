---
link: https://www.mikestreety.co.uk/blog/email-authentication-records-to-improve-deliverability/
byline: Mike Street
site: Mike Street
date: 2025-10-13T02:00
excerpt: Ensuring email deliverability is a tricky business - this post walks
  through hints and tips for improving domain authority
slurped: 2026-08-22T22:10
title: Email authentication records to improve deliverability
---

2025-10-13T00:00:00.000Z Posted on **13th October 2025**. **3 mins** reading time

Sending emails in this mad world of spam is a tricky business. Spoofing and phishing are all too common, and email providers try to be smart to it, although sometimes at the detriment to honest and "real" emails.

If your website is sending emails at all (even to you for contact form responses), it is worth considering spending time to verify that you own the domain and you are allowed to send emails from it. Word of warning: this isn't going to be the most thrilling post, but it's one of those things that'll save you a proper headache down the line when your carefully crafted emails end up in spam folders.

**SPF**, **DKIM** and **DMARC** records all help with this and below each one is explained as to what it does and how you set it up. I'll be honest - I found these records a bit bewildering at first, but once you've set them up a few times they become second nature. Think of them as your email's passport - proving you are who you say you are.

Before we dive in, bookmark this:

- [mail-tester](https://www.mail-tester.com/) - Send an email to get real-world data (proper useful, this one - gives you a score out of 10 and tells you exactly what's wrong)

## SPF Record

- [SPF Records explained](https://mailtrap.io/blog/spf-records-explained/)
- [SPF Tester](https://mxtoolbox.com/spf.aspx)

SPF (Sender Policy Framework) is basically your domain saying "these are the mail servers allowed to send email on my behalf". Without it, anyone could pretend to send emails from your domain - which is as dodgy as it sounds.

This is how the most common SPF record looks:

```
v=spf1 a mx -all
```

Breaking this down: `v=spf1` is the version, `a` and `mx` mean your domain's A record and MX records are allowed to send mail, and `-all` means "reject anything else". That last bit is important - it's like saying "if it's not on the list, it's not coming in".

As an example, if your client uses **Google Workspace**, you'll need to add `include:_spf.google.com`. Same goes for services like Mailchimp:

```
v=spf1 a mx include:_spf.google.com include:mailchimpapp.net -all
```

Most services which send emails on your behalf will have some documentation detailing what SPF Record you need.

## DMARC Record

- [DMARC wizard](https://dmarcian.com/dmarc-record-wizard/) (weirdly satisfying to use, this one)

DMARC (Domain-based Message Authentication, Reporting and Conformance) tells receiving mail servers what to do if your SPF or DKIM checks fail. It also sends you reports so you can see if someone's trying to spoof your domain - certainly interesting to see what's being attempted in the wild.

A good standard is something like the following:

- Target: `_dmarc.@`
- Type: `TXT`
- Record:

```
v=DMARC1; p=quarantine; rua=mailto:[email protected]; aspf=r;
```

Or if you want to be a bit stricter:

```
v=DMARC1; p=quarantine; pct=100; aspf=s;
```

The `p=quarantine` means "if this looks dodgy, put it in spam rather than rejecting it outright". You can use `p=reject` if you're feeling confident, but I'd recommend starting with quarantine until you're sure everything's configured properly.

## DKIM Record

This can only be configured if the service you are using emits a DKIM signature or similar. CMS's, like TYPO3, does not include a DKIM header so make sure you know before you start.

DKIM (DomainKeys Identified Mail) adds a digital signature to your emails - like a wax seal on a letter proving it hasn't been tampered with. Your email service provider will generate the keys for you.

Check with the system for instructions - each provider does it slightly differently and they'll give you the specific DNS records to add.

## BIMI

Right, this one's a bit fancy and optional, but if you want your logo to appear next to your emails in supported clients (Gmail, Yahoo, etc.), BIMI is what you need.

- [BIMI Generator & Inspector](https://bimigroup.org/bimi-generator/)
- Use a 512px square SVG for the image (the Favicon SVG is perfect for this)
- We don't generally have a **VMC** (Verified Mark Certificate) available - these cost proper money and are only really worth it for big brands

Example BIMI:

- Target: `default._bimi.@`
- Type: `TXT`
- Record: `v=BIMI1; l=https://link/to/svg;`

## Final Thoughts

I should really test email deliverability more systematically on projects, but these DNS records are a good foundation. Set them up early and you'll avoid that awkward conversation later where the client asks why their contact form emails keep ending up in spam.

If anyone's got experience with VMC certificates for BIMI or has tips on DKIM implementation in TYPO3, I'd love to hear your thoughts. There's still a lot of nuance to email deliverability that catches me out occasionally.