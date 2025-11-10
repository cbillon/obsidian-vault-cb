---
link: https://blog.rohitlokhande.in/how-to-work-with-strings-bytes-runes-and-characters-in-go
byline: Rohit Lokhande
site: Tech Blog
date: 2025-11-07T19:11
excerpt: Learn how to handle strings, bytes, runes, and characters in Go, focusing on UTF-8 encoding and text processing intricacies
slurped: 2025-11-09T09:17
title: Strings and Characters in Go Explained
tags:
  - go
  - string
---

When you first start working with text in Go, everything seems simple until you meet emojis, accented characters, and UTF-8 encoding. Then suddenly, len("🙂") doesn't equal 1 anymore.

To truly understand how Go handles text, we need to dive into strings, bytes, and runes—three closely connected concepts that define how Go represents and processes data in memory.

Before we dive deep into how strings, bytes, and runes work in Go, we first need to understand the unsung hero that powers them all: **UTF-8**.

Every piece of text you see, whether it’s “Hello“, “नमस्ते“, or “🙂“, is nothing more than numbers stored in memory.  
But how does the computer know which number means what character?  
That’s where character encoding comes in.

## The Problem From Letters to Numbers

In the early days, computers used a very limited language called ASCII, which could only represent 128 characters, mostly English letters, digits, and symbols. That worked fine until people wanted to write in other languages or include emojis.

ASCII couldn’t handle words like “ありがとう“ or “नमस्ते“ and definitely not emojis like “🙂“.

We needed something global—a universal system that could represent every character from every language.

That’s when Unicode came to the rescue.

### Unicode

**Unicode** is like a massive multilingual dictionary. It assigns a unique number called a **code point** to every character on Earth.

**Examples:**

|Character|Unicode Code Point|Meaning|
|---|---|---|
|A|U+0041|Latin Capital A|
|क|U+0915|Hindi Ka|
|你|U+4F60|Chinese “You”|
|🙂|U+1F642|Smiling Face Emoji|

Now every character has an identity. But there’s still a problem: we need to store these numbers in memory as bytes (0s and 1s). To achieve this, UTF-8 comes into play.

## UTF-8 How Go Stores Text

UTF-8 (Unicode Transformation Format - 8-bit) is a way to store Unicode characters as bytes.  
It’s a variable-length encoding, meaning

- Some characters take 1 byte (like English letters)
    
- Some take 2, 3, or 4 bytes
    

### In Short

> UTF-8 is the bridge between human language and machine memory, and Go builds that bridge right into its foundation.

## Why UTF-8 instead of UTF-16 or other?

1. **Simplicity:**
    
    UTF-8 is the most common encoding on the web and in Unix systems, so it fits Go’s design philosophy of simplicity and practicality.
    
2. **Compatibility:**
    
    ASCII characters are the same in UTF-8, making it backward compatible and efficient for English text.
    
3. **Efficiency:**
    
    Most programming identifiers, file names, and JSON data are mostly ASCII — UTF-8 uses 1 byte for them, while UTF-16 would use 2 bytes.
    
4. **Interoperability:**
    
    UTF-8 is the standard encoding for most APIs, web data, and Linux systems, making Go programs more portable.
    

## What Exactly Is a String in Go?

In Go, a string is more than just text. It’s a read-only slice of bytes, a sequence of raw data stored in UTF-8 format.

That means:

- A string is immutable (you can’t change it after creation).
    
- It stores bytes, not characters.
    
- It’s UTF-8 encoded, meaning every character can take 1 to 4 bytes.
    

**Example:**

```
s := "Hello"
fmt.Println(s)
fmt.Println(len(s)) // 5 bytes
```

Here `len(s)` returns 5 because "Hello" uses one byte per character. It's simple ASCII code. But not all text is that simple.

## Strings and UTF-8 Encoding

Go supports UTF-8 natively, meaning it can represent any character from any language, including emojis, but they may take more than one byte.

**Example:**

```
s := "A🙂"
fmt.Println(len(s)) // 5
```

Why 5 bytes?

- “A“ → 1 byte
    
- “🙂“ → 4 bytes (because it’s a Unicode character)
    

So the total is 5 bytes, not 2 characters.

> `len(s)` gives the number of bytes, not characters. For plain English text, bytes = characters, because ASCII characters use 1 byte each.

## Bytes — The Raw Data Layer

A byte in Go is just an alias for uint8, representing the raw binary data behind every string.

When you convert a string to a byte slice, you’re seeing its internal UTF-8 byte representation.

**Example:**

```
s := "Hi"
b := []byte(s)
fmt.Println(b) // [72 105]
```

Each number here is the ASCII code for the character:

- H → 72
    
- I → 105
    

Think of bytes as the DNA of your string — the smallest building blocks.

## Runes — The Character View

While bytes represent raw data, runes represent characters specifically, as Unicode code points.

**In Go:**

```
type rune = int32
```

So each rune can represent one Unicode character, no matter how many bytes it takes.

Let’s understand with the example below.

```
s := "A🙂"
r := []rune(s)
fmt.Println(r) // [65 128578]
```

Here,

- A → Unicode 65
    
- 🙂 → Unicode 128578
    

And if we get bytes from the same string, we get the output below.

```
s := "A🙂"
b := []byte(s1)
fmt.Println(b) // [65 240 159 153 130]
```

Here in the above example:

- A → 65
    
- and the other bytes are for emojis.
    

So, a rune is what Go uses to correctly handle multilingual text and emojis.

## String, Bytes, and Runes - The Comparison

|Type|Underlying Type|Represents|Use Case|
|---|---|---|---|
|string|Read-only slice of bytes|UTF-8 encoded text|Standard text data|
|[]byte|Slice of uint8|Raw binary data|File I/O, networking, encryption|
|[]rune|Slice of int32|Unicode code points|Character-level manipulation|

## Iterating Over Strings

When you use a for loop to range over a string, Go automatically decodes UTF-8 and gives you each rune, not each byte.

```
s := "Go🙂"
for i, r := range s {
    fmt.Printf("%d: %c\n", i, r)
}
```

**Output:**

```
0: G
1: o
3: 🙂
```

Notice how the emoji starts at index 3, not 2, because the emoji is 4 bytes long. This is Go’s built-in way of helping you iterate over characters safely, even for complex text.

## Common Pitfalls

1. `len(s)` **Gives Bytes, Not Characters**
    
    ```
     s := "🙂🙂🙂"
     fmt.Println(len(s))                 // 12
     fmt.Println(utf8.RuneCountInString(s)) // 3
    ```
    
    Use `utf8.RuneCountInString` from the unicode / utf8 package when you want the character count, not byte count.
    

**Strings Are Immutable**

You can’t modify a string directly

```
s := "Go"
s[0] = 'N' // ❌ compile-time error
```

Instead, convert to a slice, modify and convert back:

```
b := []byte(s)
b[0] = 'N'
s = string(b)
fmt.Prinln(s) // "NO"
```

## Understanding the Memory Representation

Here’s how Go internally stores and interprets text

```
String: "Go🙂"

Bytes: [71 111 240 159 153 130]
Runes: [71 111 128578]

'G'   = 1 byte
'o'   = 1 byte
'🙂'  = 4 bytes
```

## When to Use What

|**Use Case**|**Best Type**|
|---|---|
|Normal text processing|string|
|Raw binary I/O (files, sockets, hashing)|[]byte|
|Character-by-Character operations|[]rune|
|Counting or slicing Unicode text|utf8.RuneCountInString|

## Why This Matters?

Understanding strings, bytes, and runes helps you:

- Avoid bugs with Unicode text.
    
- Handle emojis and multilingual input correctly.
    
- Optimize performance when dealing with files or network data.
    
- Build a mental model of Go’s memory representation.
    

This is one of those small topics that quietly separates beginner Go programmers from intermediate ones.

### Quick Recap

|Concept|Meaning|
|---|---|
|String|Immutable sequence of bytes (UTF-8 encoded)|
|Byte|Represents one raw byte of data (uint8)|
|Rune|Represents a single Unicode code point (int32)|
|Tip|Always remember: 1 character is not equal to 1 byte in UTF-8|

## Conclusion

In conclusion, working with text in Go involves understanding the intricate relationship between strings, bytes, and runes, all of which are underpinned by UTF-8 encoding. This system allows Go to efficiently handle a wide range of characters, from simple ASCII to complex Unicode symbols like emojis. By grasping these concepts, you can effectively manage text data, avoid common pitfalls, and ensure your applications are robust and capable of handling multilingual and emoji-rich content. This knowledge is crucial for developing efficient and reliable Go programs, setting apart beginner programmers from those with a deeper understanding of the language's text processing capabilities.