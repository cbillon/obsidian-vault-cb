---
link: https://dev.to/moukoublen/unicode-emojis-and-a-bit-of-golang-3ced
byline: Kostas Stamatakis
site: DEV Community
date: 2024-10-04T20:16
excerpt: Lately, I have had an issue with my Fedora Linux installation displaying emojis in the OS UI and...
twitter: https://twitter.com/@
tags:
  - go
  - emoji
  - unicode
slurped: 2025-07-25T10:53
title: Unicode, Emojis, and a bit of Golang
---

Lately, I have had an issue with my Fedora Linux installation displaying emojis in the OS UI and browsers. This issue led me to investigate a bit around the font config project, but to test my configurations and fonts, I needed to produce emojis from all Unicode versions, which eventually led me to write a Golang "script" to print all the emojis and some information about their internals.

Throughout this trip, I deep-dived into the internals of emojis, their binary representations, and some of the weird/cute decisions made by the Unicode Standard regarding emojis.

But first, let's take a quick step back and summarise some glossary.

## [](https://dev.to/moukoublen/unicode-emojis-and-a-bit-of-golang-3ced#encoding-or-character-encoding)Encoding (or Character Encoding)

We could describe encoding as the "mapping" or "translation" between a letter of a language and the binary representation of this letter. For example, the traditional ASCII encoding maps the letter `a` to `0x61` hex (`0b01100001` binary). Examples of encodings are the Microsoft ([Windows 125x](https://en.wikipedia.org/wiki/Windows_code_page)) or ISO ([ISO/IEC 8859](https://en.wikipedia.org/wiki/ISO/IEC_8859)) 8-bit code pages.

In these fixed 8-bit code pages, the minimum "amount" of information used is 8-bit (1 byte), which means they can contain 256 different letters/characters. Different code pages were created by reusing the 256 binary codes to support many languages. So, having a text file with these 3 bytes written on it `[0xD0, 0xE5, 0xF2]` reads as "Πες" using the Greek ISO 8859-7, or "Ðåò" using the western ISO 8859-7 (same bytes, interpreted differently based on the code page).

At some point, having many different code pages didn't scale well as the technology progressed. So, we needed something that could fit all languages (and more) and be unified across systems.

[ fast forward, leaving a lot of history and standards out, to the present ]

### [](https://dev.to/moukoublen/unicode-emojis-and-a-bit-of-golang-3ced#unicode-standard)Unicode Standard

The [Unicode](https://home.unicode.org/) Standard was designed to support all of the world's writing systems that can be digitized. So, using the above example, in the Unicode standards, the Greek letter "Π" has the code `0x03A0` while the Latin capital letter eth "Ð" has the code `0x00D0` and no longer collide. Unicode Standard has versions, and at the time of writing, the latest version is [16.0](https://www.unicode.org/versions/Unicode16.0.0/) ([spec](https://www.unicode.org/versions/Unicode16.0.0/core-spec/)).

But wait a minute, what is this "code point"?

### [](https://dev.to/moukoublen/unicode-emojis-and-a-bit-of-golang-3ced#unicode-code-points)Unicode Code Points

In the Unicode Standard, every "letter," control character, emoji, and every defined item in general has a unique binary value called a "[code point](https://www.unicode.org/versions/Unicode16.0.0/core-spec/chapter-2/#G25564)". The standard defines all the code points, and each code point contains pure code/binary information. The hexadecimal format for each code point is usually written with a `U+` prefix. For example, the Greek Small Letter Omega (ω) code point is `U+03C9`.

So how do we actually encode those code points?

### [](https://dev.to/moukoublen/unicode-emojis-and-a-bit-of-golang-3ced#unicode-encoding-forms-and-encoding-schemes)Unicode Encoding Forms and Encoding Schemes

The first part of encoding Code Points into bytes is the [Encoding Fomrs](https://www.unicode.org/versions/Unicode16.0.0/core-spec/chapter-2/#G13708). According to the standard:

> encoding forms specify how each integer (code point) for a Unicode character is to be expressed as a sequence of one or more code units.

Encoding Forms use the term "code unit" to refer to the smallest unit of data used to represent a Unicode code point within a particular encoding.

The Unicode Standard defines three different Encoding Forms:

- [UTF-32](https://www.unicode.org/versions/Unicode16.0.0/core-spec/chapter-2/#G18421). Fixed length code unit per code point. Size per code point: one 32bits code unit (4 bytes).
- [UTF-16](https://www.unicode.org/versions/Unicode16.0.0/core-spec/chapter-2/#G11153). Variable length code units per code point. Size per code point: one or two 16bit code units (2~4 bytes).
- [UTF-8](https://www.unicode.org/versions/Unicode16.0.0/core-spec/chapter-2/#G11165). Variable length code units per code point. Size per code point: one to four 8bit code units (1~4 bytes).

This means a single code point or a sequence of code points might be encoded differently depending on the encoding form used.

The layer that takes care of the actual binary serialization in Unicode is called [Encoding Schemes](https://www.unicode.org/versions/Unicode16.0.0/core-spec/chapter-2/#G19273) and takes care of all the low-level details (such as endianness). [Table 2-4](https://www.unicode.org/versions/Unicode16.0.0/core-spec/chapter-2/#G18461) of the Unicode Spec:  

```
|Encoding Scheme| Endian Order                | BOM Allowed? |
| ------------- | ----------------------------| ------------ |
| UTF-8         | N/A                         | yes          |
| UTF-16        | Big-endian or little-endian | yes          |
| UTF-16BE      | Big-endian                  | no           |
| UTF-16LE      | Little-endian               | no           |
| UTF-32        | Big-endian or little-endian | yes          |
| UTF-32BE      | Big-endian                  | no           |
| UTF-32LE      | Little-endian               | no           |
```

 

Note: Almost all modern programming languages, os, and filesystems use Unicode (with one of its encoding schemes) as their native encoding. Java and .NET use UTF-16, whereas Golang uses UTF-8 as internal string encoding (that means when we create any string in memory, it is encoded in Unicode with the mentioned encoding form)

## [](https://dev.to/moukoublen/unicode-emojis-and-a-bit-of-golang-3ced#emoji)Emoji

The Unicode Standard also defines code points for emojis (a lot of them), and (after some mix-up with version number), the version of the Emoji "standard" progresses in parallel with the Unicode Standard. At the time of writing, we have Emoji "16.0" and Unicode Standard "16.0".

Examples:  
⛄ Snowman Without Snow (U+26C4)  
🥰 Smiling Face with Smiling Eyes and Three Hearts (U+1F970)

### [](https://dev.to/moukoublen/unicode-emojis-and-a-bit-of-golang-3ced#emoji-modifiers-and-join)Emoji Modifiers and Join

Unicode defines modifiers that could follow an emoji's base code point, such as variation and skin tone (we will not explore the variation part).

We have six skin tone modifiers (following the [Fitzpatrick scale](https://en.wikipedia.org/wiki/Fitzpatrick_scale)) called EMOJI MODIFIER FITZPATRICK TYPE-X (where x is 1 to 6), and they affect all human emojis.

Light Skin Tone (Fitzpatrick Type-1-2) (`U+1F3FB`)  
Medium-Light Skin Tone (Fitzpatrick Type-3) (`U+1F3FC`)  
Medium Skin Tone (Fitzpatrick Type-4) (`U+1F3FD`)  
Medium-Dark Skin Tone (Fitzpatrick Type-5) (`U+1F3FE`)  
Dark Skin Tone (Fitzpatrick Type-6) (`U+1F3FF`)

So, for example, like all human emojis, the baby emoji 👶 (`U+1F476`), when not followed by a skin modifier, appears in a neutral yellow color. In contrast, when a skin color modifier follows it, it changes accordingly.  
👶 `U+1F476`  
👶🏿 `U+1F476 U+1F3FF`  
👶🏾 `U+1F476 U+1F3FE`  
👶🏽 `U+1F476 U+1F3FD`  
👶🏼 `U+1F476 U+1F3FC`  
👶🏻 `U+1F476 U+1F3FB`

#### [](https://dev.to/moukoublen/unicode-emojis-and-a-bit-of-golang-3ced#joining-emojis-together)Joining emojis together

The most strange but **cute** decision of the Emoji/Unicode Standard is that some emojis have been defined by joining others together using the [Zero Width Joiner](https://en.wikipedia.org/wiki/Zero-width_joiner) without a standalone code point.

So, for example, when we combine:  
White Flag 🏳️ (`U+1F3F3 U+FE0F`) +  
Zero Width Joiner (`U+200D`) +  
Rainbow 🌈 (`U+1F308`)

It appears as Rainbow Flag 🏳️‍🌈 (`U+1F3F3 U+FE0F U+200D U+1F308`)

Or, 👨🏽 + 🚀 => 👨🏽‍🚀  
Or even, 👩🏼 + ❤️ + 💋 + 👨🏾 => 👩🏼‍❤️‍💋‍👨🏾

It's like squeezing emojis together, and then, poof 💥, a new emoji appears. How cute is that?

---

I wanted to create a Markdown table with all emojis, and the Unicode emoji sequence tables are the source of truth for that.

[https://unicode.org/Public/emoji/16.0/emoji-sequences.txt](https://unicode.org/Public/emoji/16.0/emoji-sequences.txt)  
[https://unicode.org/Public/emoji/16.0/emoji-zwj-sequences.txt](https://unicode.org/Public/emoji/16.0/emoji-zwj-sequences.txt)

So I created a Golang parser ([here](https://gist.github.com/moukoublen/ce99590ea668f826b6511f58f3e9843a)) that fetches and parses those sequence files, generates each emoji when a range is described in the sequence file, and prints a markdown table with some internal information for each one (like the **parts** in case it joined, or the **base** + **skin tone**, etc.).

You can find the markdown table [here](https://gist.github.com/moukoublen/c9db7ec09b1028ce23bb3b0bbf0f1a72).

The last column of this table is in this format `<utf-8 byte length>:<unicode code points length>`.

## [](https://dev.to/moukoublen/unicode-emojis-and-a-bit-of-golang-3ced#golang-unicode-and-rune)Golang, Unicode and Rune

```
str := "⌚"
len([]rune(str)) // 1
len([]byte(str)) // 3
```

 

As we discussed, Golang internal string encoding is UTF-8, which means that, for example, for clock emoji ⌚ the byte length is 3 (because the UTF-8 produces 3 bytes to "write" this code point), and the code point length is 1.

> Golang rune == Unicode Code Point

But in the case of joined emoji -even if it "appears" as one- we have many code points (runes) and even more bytes.  

```
str := "👩🏽‍❤️‍💋‍👨🏻"
len([]rune(str)) // 10
len([]byte(str)) // 35
```

 

And the reason is that:  

```
👩🏽‍❤️‍💋‍👨🏻 : 👩🏼 + ZWJ + ❤️ + ZWJ + 💋 + ZWJ + 👨🏾

👩🏼  : 1F469 1F3FC // 👩 + skin tone modifier [2 code points]
ZWJ : 200D // [1 code points] * 3
❤️  : 2764 FE0F // ❤ + VS16 for emoji-style [2 code points]
💋  : 1F48B // [1 code point]
👨🏾  : 1F468 1F3FE // 👨 + skin tone modifier [2 code points]
```

 

😉

---

It is worth mentioning that how we see emojis depends on our system font and which versions of emoji this font supports.

I don't know the exact internals of font rendering and how it can render the joined fonts correctly. Perhaps it will be a future post.

Til then, cheers 👋