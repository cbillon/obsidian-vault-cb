---
link: https://gagor.pro/2024/05/remove-password-from-pdf-documents/
byline: Tom
site: Tom's Blog
date: 2024-05-27T02:00
excerpt: Learn how to remove passwords from PDF documents using the qpdf tool on Linux, making it easier to read protected files on devices like the Pocketbook Touch HD.
tags:
  - password
  - document
slurped: 2026-02-24T10:50
title: Remove password from PDF documents
---

I use a Pocketbook Touch HD first gen. to read ebooks and I bought it in 2017 (oh dear!). It’s a really good device, despite being terribly slow according to current standards and battery have probably only 20% capacity comparing to the day I bought it. Anyway, it outperformed any of my smartphones on it’s longevity. I don’t use it that often anymore but it happen to me to read some stuff that I find on the Internet.

Recently I received a doc which I wanted to read but it was password protected PDF. Reader prompted me to share the password but it was unable to decrypt/read PDF, prompting again and again as if password was wrong. As it was a lengthy text which I preferred to read lying in the bed than in front of PC, I wanted to remove it.

There’s a nice tool for that, called `qpdf`[1](#fn:1). You can install it like that:

```
apt update
apt install -y qpdf
```

Then just run[2](#fn:2):

```
qpdf -password=<document-password> -decrypt /path/to/secured.pdf unprotected.pdf
```

Now I could send it to my Pocketbook and read 😄

---
