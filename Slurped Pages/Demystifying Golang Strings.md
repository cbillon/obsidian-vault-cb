---
link: https://medium.com/@andreiboar/demystifying-golang-strings-05981b84f1a7
byline: Andrei Boar
site: Medium
date: 2024-07-15T06:50
excerpt: "Demystifying Golang Strings This post discusses Golang strings: their design, and how runes and bytes fit into the picture. It’s gonna be a long article, so let's get started! Declaring our first …"
twitter: https://twitter.com/@zuzuleinen
slurped: 2025-11-26T11:38
title: Demystifying Golang Strings
tags:
  - go
  - string
---

This post discusses Golang strings: their design, and how runes and bytes fit into the picture.

It’s gonna be a long article, so let's get started!

## Declaring our first string

Let’s declare our first string:

```
  package main

  import "fmt"

  func main() {  
      str := "Hello André!"

      fmt.Println(str)  
}

```
Apart from my first name being in French, there’s nothing weird here. The syntax is a string literal. Some languages allow strings in single quotes, but in Go, strings are enclosed with double quotes only.

To understand what a string is in Go let’s jump to the definition in the documentation and elaborate from there:

// string is the set of all strings of 8-bit bytes, conventionally but not  
// necessarily representing UTF-8-encoded text. A string may be empty, but  
// not nil. Values of string type are immutable.  
type string string

## A string is a slice of bytes

Let’s go through each index of the string and display the value at that index:
```
  package main

  import "fmt"

  func main() {  
      str := "Hello André!"

      for i := range 13 {  
       fmt.Print(str[i], " ")  
      }  
      fmt.Println()  
  }
  
```


The output doesn’t contain any characters, just numbers:

72 101 108 108 111 32 65 110 100 114 195 169 33

These numbers represent bytes in decimal notation. Let’s transform these bytes in hexadecimal by using the `%x` verb:

```
  package main

  import "fmt"

  func main() {  
      str := "Hello André!"

      for i := range 13 {  
         fmt.Printf("%x ", str[i])  
      }  
      fmt.Println()  
  }
  
```

48 65 6c 6c 6f 20 41 6e 64 72 c3 a9 21

The first byte, 48, or `0x48` how is usually written in hexadecimal notation represents the letter `H`.

Indeed, if we check how the [letter H](https://www.compart.com/en/unicode/U+0048) is represented in UTF-8 we see that it is done by using the `0X48` byte:

Press enter or click to view image in full size

Strings, like everything that has to be stored electronically, are just a [bunch of bytes](https://medium.com/@andreiboar/fundamentals-of-i-o-in-go-c893d3714deb#:~:text=Everything%20is%20a%20bunch%20of%20bytes).

Because a string in Go is a slice of bytes, indexing it gives the individual byte, not a character like in other languages.

## A string is not a []byte

Strings in Go are typically referred to as **“**_slices of bytes”_, but they are not really a `[]byte` type. If we check [how a string is implemented](https://github.com/golang/go/blob/master/src/runtime/string.go#L242) in Go:
```

  type stringStruct struct {  
     str unsafe.Pointer  
     len int  
  }

```

we notice that the data structure is similar to a slice but is missing the `cap` field. If you’re unfamiliar with Go slices' internal structure, you can check my older article: [Demystifying Golang Slices](https://medium.com/@andreiboar/demystifying-golang-slices-83ffe3550db5).

You can call slice functions on strings, but since they don’t have `cap` field, we cannot grow them with `append`:
```
  package main

  import "fmt"

  func main() {  
   str := "Hello André!"

   // We can do this  
   for _, v := range str {  
    fmt.Println(v)  
   }

   fmt.Println(len(str)) // We can do this  
   fmt.Println(str[0:3]) // We can do this  
   fmt.Println(str[0]) // We can do this  
   dst := make([]byte, 3)  
   copy(dst, str) // We can do this

   str = append(str, "a") // We cannot do this  
  }

```

Strings are so similar to a `[]byte` that many functions from the [strings](https://pkg.go.dev/strings) package have an equivalent in the [bytes](https://pkg.go.dev/bytes) package. So if you have a `[]byte` representing a string you don’t need to convert it to string, just use the equivalent function from the bytes package.

Despite the similarity, a string is not a `[]byte`. As the saying goes, _“All models are wrong, but some are useful.”_ it’s still helpful to think of strings as slices of bytes because it will help you remember that indexing them by a key will give you a byte or calling `len` on them will give you the number of bytes.

Strings are just slices of bytes without capacity.

## Strings are read-only

In the previous section, I showed you the similarity between strings and slices. We found out we can do many slice operations on strings, but we cannot call `append` because strings miss the `cap` field.

Another thing you cannot do on strings is to change them by the index:

```
  package main

  import "fmt"

  func main() {  
      str := "Hello André!"

      fmt.Println(str[2]) // Even though you can do this  
      str[2] = byte(72)  // You cannot do this  
  }
  
```

You can read an individual byte from a string by indexing, but you cannot change it.

This change is perfectly OK when dealing with regular slices:

```
  package main

  import "fmt"

  func main() {  
      str := []byte("Hello André!") // Here we convert a string to a []byte

     fmt.Println(str[2]) // You can do this  
      str[2] = byte('a')  // You can fo this

      fmt.Println(str) // [72 101 97 108 111 32 87 111 114 108 100 33]  
  }
  
```

I hope by now it is clear why we can call strings as read-only slices of bytes.

And just because they are read-only and you cannot use `append` it doesn’t mean you cannot create new strings from other strings:

```
  package main

  import (  
      "fmt"  
      "strings"  
  )

  func main() {  
      str := "Hello André!"  
      str += " What a fine day!"  
      str = str + " A fine day indeed!"

      var b strings.Builder  
      b.WriteString("Hello André!")  
      b.WriteString(" What a fine day!")

      fmt.Println(str)        // Hello André! What a fine day! A fine day                                                          
      fmt.Println(b.String()) // Hello André! What a fine day!  
  }
  
```

The `+=` helps us achieve the append functionality.

Because strings are immutable, new memory is allocated on every `+=`. To avoid lots of allocations you can opt for the more efficient `[strings.Builder](https://pkg.go.dev/strings#Builder)` or even `[bytes.Buffer](https://pkg.go.dev/bytes#Buffer)`because they have internal buffers that optimize memory allocations, but do that only when you deal with many concatenations, usually `+=` is the perfect choice.

## The zero value of a string is an empty string

Regular slices can be nil. Strings in Go, even though they share a similar design with them cannot be `nil`:

```
  package main

  import "fmt"

  func main() {  
   var (  
    sb  []byte  
    str string  
   )

   fmt.Println(sb == nil)  // true  
   fmt.Println(str == "")  // true  
   fmt.Println(str == nil) // Cannot convert 'nil' to type 'string'

   _ = []byte(nil) // possible  
   _ = string(nil) // Cannot convert 'nil' to type 'string'  
  }
  
```

So, we say the zero value of a string is the empty string `“”` . It is never `nil`, nor can it be converted to `nil`.

## A string is not necessarily UTF-8 encoded text

Because strings are just slices of bytes, the compiler lets us convert a `[]byte` to a `string`:
```
  package main

  import "fmt"

  func main() {  
      str := string([]byte{72, 101, 108, 108, 111, 32, 65, 110, 100, 114, 195,     169, 33})

      fmt.Println(str) // Hello André!  
  }
  
```


This means that we can convert any `[]byte` to a string including a slice of which bytes that don’t represent UTF-8 encoded text:

```
  package main

  import "fmt"

  func main() {  
      str := string([]byte{255})

      fmt.Println(str)  
  }
  
```
 This outputs:

�

because it couldn't be decoded using UTF-8. That’s why the docs state that a _“string is the set of all strings of 8-bit bytes, conventionally but not necessarily representing UTF-8-encoded text”._

## Ranging over a string gives you a rune, not a byte

> Wait what? You just told me that a string is a slice of bytes. Why don’t I get a byte when ranging over a string? And what the hell is a **rune**?

you might ask.

Patience my friend. Everything will be revealed to you in due time, I promise.

You might have noticed that so far I ranged over 13 when I wanted to index the string:

for i := range 13

But since we can range over a string, let’s do that now:

```
  package main

  import "fmt"

  func main() {  
      str := "Hello André!"

      for k, v := range str {  
         fmt.Printf("%d %v %T\n", k, v, v)  
      }  
  }
  
```

In the above code, we display the key, value, and type of each value on a new line. If you’re not familiar with these verbs you can check the [documentation](https://pkg.go.dev/fmt#hdr-Printing).

Now look at the result and see if something is off there:

0 72   int32  
1 101  int32  
2 108  int32  
3 108  int32  
4 111  int32  
5 32   int32  
6 65   int32  
7 110  int32  
8 100  int32  
9 114  int32  
10 233 int32  
12 33  int32

Look at each key and see if you notice something.  
Look at the type of each value and see if something is not right.

OK, spoiler time!

If you look carefully at each key you’ll notice that key **11** is missing. However, we just did `range str` so where is it? You get every key if you range over regular slices, so why not here?

The type of each value is `int32` but a byte is a `uint8`, not `int32`:

// byte is an alias for uint8 and is equivalent to uint8 in all ways. It is  
// used, by convention, to distinguish byte values from 8-bit unsigned  
// integer values.  
type byte = uint8

So what the hell is this `int32`? Indexing a string gives us a byte. Shouldn’t the range over a string in Go give us bytes?

Well, it doesn't. **Ranging over a string in Go gives us runes, not bytes.** And indeed a rune is an alias for `int32`, that’s why we get int32 in our program:

// rune is an alias for int32 and is equivalent to int32 in all ways. It is  
// used, by convention, to distinguish character values from integer values.  
type rune = int32

## So, what is actually a rune?

A rune represents a Unicode code point. I’ll spare you the details how we got from ASCII to Unicode and UTF-8. Joel Spolsky has a nice history [here](https://www.joelonsoftware.com/2003/10/08/the-absolute-minimum-every-software-developer-absolutely-positively-must-know-about-unicode-and-character-sets-no-excuses/) and you can also check this video by Leetcoder:

Each character is represented by a code point in Unicode. For example, `H` is represented by `U+0048` code point where `U+` means Unicode and `0048` is a hexadecimal number:

Press enter or click to view image in full size

Because code point was a bit of a mouthful, Go team came up with rune that essentially represents the same thing.

Go uses UTF-8, which is the most popular standard that takes all Unicode codepoints and encodes them in byte sequences. For example, the `U+0048` will be encoded to `0x48`.

UTF-8 is a **variable-length** character encoding, which means that not all characters are stored using the same number of bytes. UTF-8 encodes codepoints from 1 to 4 bytes. `H` takes 1 byte, but `é` takes 2 bytes:

```
  package main

  import (  
      "fmt"  
      "unicode/utf8"  
  )

  func main() {  
      str := "Hello André!"

      for _, v := range str {  
         fmt.Printf("%U %d (%v)\n", v, utf8.RuneLen(v), string(v))  
      }  
  }
    
```

In above program we range over `str`, which means we get a rune on every iteration and we display on 3 columns: the Unicode code point(provided by `%U` verb), the bytes required for it, and the string value:

U+0048 1 (H)  
U+0065 1 (e)  
U+006C 1 (l)  
U+006C 1 (l)  
U+006F 1 (o)  
U+0020 1 ( )  
U+0041 1 (A)  
U+006E 1 (n)  
U+0064 1 (d)  
U+0072 1 (r)  
U+00E9 2 (é)  
U+0021 1 (!)

As we can see `é` requires 2 bytes, which are `0xC3 0xA9.` That’s why 11 was missing from our range above because ranging over the string gives us a rune on each iteration and for each rune, we get its starting key:

Press enter or click to view image in full size

The values in red are the keys for each rune. Notice `é requiring 2 bytes of storage`

So when I think about strings in Go, I have in my mind this 3-layer image:

Press enter or click to view image in full size

The string, the code points and the bytes

The string is on the high level. The code points(runes) that make up those characters are on the second level. At the lowest level, we have the bytes that encode those code points.

From now on, I’ll stop using the code points terminology and use just runes instead.

As you can see in our example, the number of runes is not necessarily equal to the number of bytes. Remember, a rune can take 1, 2, 3, or 4 bytes.

That’s why `len(str)` is not a good idea to use when you want to count the runes in a string. Because `len(str)` will give you the right answer only when you have 1-byte runes.

Use [RuneCountInString](https://pkg.go.dev/unicode/utf8#RuneCountInString) when counting the number of runes of a string:

```
  package main

  import (  
      "fmt"  
      "unicode/utf8"  
  )

  func main() {  
      str := "Hello André!" // 12 characters

      fmt.Println(len(str))                    // 13 because we have 13 bytes  
      fmt.Println(utf8.RuneCountInString(str)) // 12 because we have 12 runes  
  }
  
```

You should have a read on the [unicode/utf8](https://pkg.go.dev/unicode/utf8) package. It’s quite small and there are some useful methods in there.

So remember not to use `len(str)` to count the number of runes in a string. `len` on a string will give you the number of bytes, not the number of runes.

## Runes are just numbers

Because `rune` represent a Unicode code point, which is just a hexadecimal number with a `U+` prefix added to it, **runes are just numbers**.

This is good to remember because for example sometimes you might want to compare them:

```
  package main

  import "fmt"

  func main() {  
      fmt.Println(isBetween('B', 'A', 'D'))  
  }

  // isBetween checks if x is between a and b  
  func isBetween(x, a, b rune) bool {  
      return x > a && x < b  
  }
  
```

Now, just because they are numbers it doesn’t mean you always use their integer version. We also have [rune literals](https://go.dev/ref/spec#hex_byte_value:~:text=2%20*%201i%20%3D%3D%200.25i-,Rune%20literals,-A%20rune%20literal), allowing you to create runes in different ways.

Bellow we have 3 ways of creating the same rune
```
  package main

  import "fmt"

  func main() {  
      runeFromChar := 'A'   
      runeFromUnicode := '\u0041'  
      runFromByteVal := '\x41'

      fmt.Println(runeFromChar)    // 65  
      fmt.Println(runeFromUnicode) // 65  
      fmt.Println(runFromByteVal)  // 65  
 }
 
```


Most of the time, you’ll work with the first option, where you have the actual character enclosed in single quotes.

Because a string is composed of runes, you can also convert a `[]rune` or a single `rune` to a string:

```
  package main

  import "fmt"

  func main() {  
      runes := []rune{72, 101, 108, 108, 111, 32, 65, 110, 100, 114, 233, 33}

      fmt.Println(string(runes)) // Hello André!

      runes = []rune{72}  
      fmt.Println(string(runes)) // H  
  }
  
```

## Conclusion

That was it! I hope you enjoyed this article as much as I had fun writing it!

I hope by now, strings in Go are much clearer for you, and you feel more confident about working with them.

Let’s recap some of the things we just learned:

- a string is a read-only slice of bytes
- indexing a string gives the individual byte, not a character
- a string cannot be nil, just empty string `“”`
- strings in Go are usually, but not necessarily UTF-8 encoded text
- ranging over strings gives you runes, not bytes
- `rune` is just a short-hand for Unicode code point
- a rune is just an `int32` and a byte is just an `uint8`