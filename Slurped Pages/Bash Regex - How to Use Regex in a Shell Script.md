---
link: https://kodekloud.com/blog/regex-shell-script/
byline: Hemanta Sundaray
site: DevOps Blog
date: 2023-10-24T15:40
excerpt: In this blog post, we understand what regex is, along with some common metacharacters.
twitter: https://twitter.com/@kodekloud1
slurped: 2025-12-20T16:36
title: "Bash Regex: How to Use Regex in a Shell Script"
tags:
  - bash
  - regex
---

#### Highlights

- ****Regex Power in Bash:**** Regular expressions let you search, validate, and transform text efficiently - perfect for automation in shell scripts.
- ****Two Core Components:****
- - __Ordinary characters__ match themselves (e.g., `abc`).
    - __Metacharacters__ add flexibility (e.g., `^`, `$`, `.`, `[]`, `*`, `+`, `?`, `()`, `|`).
- ****Practical Bash Usage:**** Use regex directly with the `[[ ... ]]` test construct and the `=~` operator - no need for external tools like `grep` or `sed`.
- ****Capturing Groups:**** Extract specific parts of text with parentheses `()` and retrieve matches using the special `BASH_REMATCH` array.
- ****Hands-On Example:**** A short Bash script demonstrates how to extract a domain name from a URL with regex.
- ****Maintainability Tip:**** Store regex patterns in variables for cleaner, more readable scripts - especially useful for complex expressions.
- ****Skill Booster:**** Mastering regex in Bash equips you to handle ****data extraction, validation, and transformation**** in your scripts with ease.

Regular expressions (regex) are a powerful tool for defining patterns within text. These patterns serve as robust mechanisms for searching, matching, and manipulating text, significantly reducing the amount of code and effort required to perform complex text-processing tasks. When used within Bash scripts, regex can help automate and streamline a variety of operations. Specifically, regex assists in:

- **Data extraction**: Pulling necessary information from text.
- **Data validation**: Ensuring the data follows a certain format or set of rules.
- **Data transformation**: Altering data into a desired format or structure.

In this blog post, **we'll briefly touch upon what regex is, explore some common regex metacharacters, and demonstrate how to use regex inside Bash scripts**. Whether you’re a system admin, a programmer, or someone curious about improving your scripting skills, knowing how to use regex in shell scripts is a valuable skill. So, without further ado, let’s dive in!

## Prerequisite

To try out the scripts in this blog post, you need access to **a Bash shell**. You also need **a text editor**, such as "nano" or "vim", which come pre-installed by default in many Unix-like operating systems.

For the purpose of this blog post, I'll be using KodeKloud’s [Ubuntu playground](https://kodekloud.com/playgrounds/playground-ubuntu-22-04?ref=kodekloud.com), which lets you access a pre-installed Ubuntu operating system in just one click. Best of all, you won't need to go through the hassle of installing any additional software— everything you need is already set up and ready to use.

## Create a Script File

Let’s start by creating a Bash script file named `demo.sh`. This is where we'll place and run the scripts we're about to write in the upcoming sections. To create it, run the following command:

```
nano /usr/local/bin/demo.sh
```

**Note**: While you're free to create the `demo.sh` file in any directory of your choice, we're placing it in the `/usr/local/bin` directory for a specific reason. In most Linux distributions, this directory is included in the system's command path. This means we can run our script without making it executable.

For an in-depth guide on how to make a Bash script file executable, check out our blog post: [How to Make a Bash Script File Executable in Linux](https://kodekloud.com/blog/make-bash-script-file-executable-linux/).

## What are Regular Expressions?

At their core, regular expressions are **symbolic notations used to identify patterns within text**. In regex, characters are categorized into two main types:

### 1. Ordinary Characters

Ordinary characters are the simplest form in regex. They **represent exact characters in the text**. For example, the regular expression `abc` would match the characters `abc` in a string, nothing more, nothing less.

### 2. Metacharacters

Metacharacters are characters that have a special meaning when used in a regex pattern. Unlike ordinary characters, which only match themselves, metacharacters can match a range of characters or a position or act as modifiers to alter the behavior of part of your regex pattern. They are the building blocks that allow you to specify the rules for the types of strings you want to match or manipulate.

For example, consider the regex pattern `ho*`. In this pattern, `h` and `o` are ordinary characters that match the characters `h` and `o` in a string, respectively, while `*` is a metacharacter. The metacharacter `*` matches zero or more occurrences of the preceding character, which in this case is `o`. So, this pattern would match `h`, `ho`, `hoo`, `hooo`, and so on.

Here’s a breakdown of common metacharacters and their functions:

- **`^`** (caret): Matches the start of a string.
- **`$`** (dollar): Matches the end of a string.
- **`.`** (dot): Matches any single character except a newline character.
- **`[]`** (square brackets): Defines a character class, matching any one character within the brackets.
- **`{}`** (curly brackets): Specifies a specific quantity of characters to match.
- **`-`** (hyphen): Specifies a range of characters when used within square brackets.
- **`?`** (question mark): Makes the preceding character optional, matching zero or one occurrence.
- **`*`** (asterisk): Matches zero or more occurrences of the preceding character.
- **`+`** (plus): Matches one or more occurrences of the preceding character.
- **`()`** (parentheses): Groups expressions together.
- **`|`** (pipe): Indicates an OR condition between two expressions.
- **`\`** (backslash): Escapes a metacharacter, allowing it to be matched as a literal character.

With a good understanding of these metacharacters, you can create complex search patterns suited to your text-processing requirements. Note that regex syntax is a vast topic in itself, and covering it completely is outside the scope of this blog post.

**In Bash scripts, regular expressions can be used directly within the `[[ ... ]]` test construct by using the `=~` operator.** This setup allows you to evaluate and compare strings against patterns defined by regular expressions. When the `=~` operator is used, the string to the right of the operator is considered a regular expression, and the string to the left of the operator is the string to be matched against that regular expression.

**Note**: When we say "used directly," we mean that this method does not require calling external programs or utilities such as `grep`, `sed`, or `awk`. Instead, the `=~` operator within the `[[ ... ]]` construct provides a straightforward and built-in way to work with regular expressions in Bash.

Now, let’s illustrate this with a practical example:

Below, we have a script designed to extract the domain name from a given [URL](https://developer.mozilla.org/en-US/docs/Learn/Common_questions/Web_mechanics/What_is_a_URL?ref=kodekloud.com).

```
#!/bin/bash

url=$1

if [[ "$url" =~ ^https?://([^/]+) ]]; then
    domain=${BASH_REMATCH[1]}
    echo "Domain name: $domain"
else
    echo "Invalid URL"
fi
```

In the script above, the line `url=$1` is our starting point. Here, we are assigning the value of the first argument passed to the script to the variable `url`. Whenever this script is called, it's expected that the user provides a URL as the first argument. This URL is then stored in the `url` variable, which is subsequently used in the regex matching operation within the `[[ ... ]]` construct.

Note that `$1` is called a positional parameter and stands for the first argument passed to the script. To understand what positional parameters are and how they are used to parse command line options and arguments, check out our blog post: [How to Use Bash Getopts With Examples](https://kodekloud.com/blog/bash-getopts/#parsing-command-line-options-using-positional-parameters).

Following this, we have an `if-else` block that handles the logic of matching the provided URL against a regular expression pattern to extract the domain name.

Inside the `[[ ... ]]` construct, we have an expression in the form of `"$url" =~ ^https?://([^/]+)`. On the left-hand side of the `=~` operator, we have the `url` variable, which holds the URL string to be matched. On the right-hand side is the regular expression `^https?://([^/]+)`, which defines the pattern the URL should match.

Let's break down the regular expression:

- `^https?://` is looking for strings that start with `http://` or `https://`, where the `s` is optional due to the `?` metacharacter.
- `([^/]+)` is a capturing group that matches one or more characters that are not a forward slash, representing the domain part of the URL.

Once the expression `[[ "$url" =~ ^https?://([^/]+) ]]` evaluates to true—indicating that the URL matches the regex pattern—Bash populates a special array named `BASH_REMATCH` with the results of the regex matching operation. This array is instrumental in accessing the segments of the string that matched the pattern, and it’s structured as follows:

- `BASH_REMATCH[0]` will contain the entire string that matched the regex pattern. In the context of the above script, it would hold the entire URL if the URL conforms to the pattern specified by the regex.
- `BASH_REMATCH[1]`, `BASH_REMATCH[2]`, `BASH_REMATCH[3]`, and so on, will contain the portions of the string that matched any capturing groups in the regex pattern, in the order that they appear in the pattern. Capturing groups are denoted by parentheses `( )` in the regex pattern and are used to capture a particular portion of the matched text.

In our script, the regex pattern `^hhttps?://([^/]+)` contains one capturing group `([^/]+)`, which aims to capture the domain part of the URL. Therefore, `BASH_REMATCH[1]` will contain the portion of the string that matched this capturing group—specifically, the domain name of the URL.

The line `domain=${BASH_REMATCH[1]}` in the script extracts the domain name from the `BASH_REMATCH` array, specifically from the first capturing group of the regex match. This is how we obtain the domain name from the URL, which is then echoed to the console with the line `echo "Domain name: $domain"`.

Furthermore, if you prefer, you can save the regex pattern in a variable and use that variable within the expression. This can make your script more readable and maintainable, especially when dealing with complex regex patterns.

```
#!/bin/bash

url=$1
regex_pattern="^https?://([^/]+)"

if [[ "$url" =~ $regex_pattern ]]; then
    domain=${BASH_REMATCH[1]}
    echo "Domain name: $domain"
else
    echo "Invalid URL"
fi
```

In this modified version of the script, we first define the `regex_pattern` variable to hold the regular expression pattern. Then, inside the `[[ ... ]]` construct, we use the `regex_pattern` variable to the right of the `=~` operator.

By doing this, we maintain a clean, organized script that's easier to read and modify. This method of storing the regex pattern in a variable is particularly useful when dealing with complex or lengthy regex patterns.

Now that we have our script ready, let's see it in action. First, we need to add it to the `demo.sh` script file. Run the following command to open `demo.sh` using the "nano" text editor:

```
nano /usr/local/bin/demo.sh
```

This will open the "nano" editor with a new, blank file called `demo.sh`. Now, add the following script to the editor: 

```
#!/bin/bash

url=$1  

if [[ "$url" =~ ^https?://([^/]+) ]]; then
    domain=${BASH_REMATCH[1]}
    echo "Domain name: $domain"
else
    echo "Invalid URL"
fi
```

You can do this by simply copying and pasting the script into the nano editor.

Once you have pasted the script, you will need to save the changes. To do this, press `ctrl + o`. This will prompt you to confirm the file name to which the changes should be written. Just press `enter` to confirm. To exit the "nano" editor, press `ctrl + x`.

Now, [run the script](https://kodekloud.com/blog/linux-run-sh-script/) using the following command:

```
bash demo.sh https://kodekloud.com/blog
```

Here, `https://kodekloud.com/blog` is specified as a URL and is passed as an argument to the `demo.sh` script.

Upon running this script, you will see the domain name `kodekloud.com` printed out on the terminal, as shown below:

![](https://lh3.googleusercontent.com/uPVSMsS7N84PcRnI65F7x4LhC6W2G2PP-VFXUcG53V2xmzFqmOpJovMs-KqRqVV32dLLitTfwmF9MA407fo-EGSw6ukL06BCcqwUae1dk0aK0aRw5TCRyDDQEIlwQvpfAfpHciJPsU0F-7yqivuk9w)

Now, you know how to use regex in a shell script. 

Want to master Linux skills and get certified? Check out our [Linux Learning Path](https://kodekloud.com/learning-path/linux/?ref=kodekloud.com):

[

Linux Learning Path | Kodekloud

Uncover our expert-designed Linux learning path. Master Linux administration and development with a proven study roadmap and resources.

![](https://kodekloud.com/blog/content/images/icon/67a4a75e8c572bff61235ddd_Dark-20Favicon-20KK-57.png)

![](https://kodekloud.com/blog/content/images/thumbnail/64d1ca611b95aeb3f0f60e7c_Learing-20Path-20Share-20preview-20-4--13.webp)

](https://kodekloud.com/learning-path/linux/?ref=kodekloud.com)

## Conclusion

In this blog post, we briefly explored what regex is, along with some common metacharacters. We then learned how to use regex within Bash scripts using the `[[ ... ]]` construct and the `=~` operator.

As you continue your scripting journey, keep experimenting with regex patterns and integrate them into your scripts. In no time, you'll find yourself tackling complex text-processing challenges with ease.

Looking to build a solid foundation in shell and Bash scripting or take your existing skills to the next level? Check out these courses from KodeKloud: 

- [Shell Scripts for Beginners](https://kodekloud.com/courses/shell-scripts-for-beginners/?ref=kodekloud.com): In this course, you'll dive into the practical world of Linux shell scripting. Regardless of your programming experience, you'll master fundamental scripting concepts such as variables, loops, and control logic. Throughout the course, you'll get plenty of hands-on experience using our comprehensive labs. Not only that, you'll also receive immediate feedback on your scripts, which will help you improve and refine them. 
- [Advanced Bash Scripting](https://kodekloud.com/courses/advance-bash-scripting/?ref=kodekloud.com): In this course, you'll start with fundamentals like variables, functions, and parameter expansions and then dive deeper into streams, input/output redirection, and command-line utilities like `awk` and `sed`. You'll master arrays for data manipulation and storage and learn best practices to create robust scripts.

## FAQ

#### Q1: What are regular expressions used for in Bash scripts?

Regex helps with ****data extraction, validation, and transformation****, making your scripts more powerful and reducing the need for lengthy code.

#### Q2: Do I need external tools like `grep` or `sed` to use regex in Bash?

Not always. Bash supports regex directly with the `[[ ... ]]` construct and the `=~` operator. However, tools like `grep`, `awk`, or `sed` are still useful for advanced processing.

#### Q3: What is `BASH_REMATCH`?

`BASH_REMATCH` is a special Bash array that stores the results of a regex match:

- `[0]` contains the full match.
- `[1]`, `[2]`, etc., contain values from capturing groups.

#### Q4: How do capturing groups work in Bash regex?

Parentheses `()` in regex create capturing groups. They let you extract specific portions of text (e.g., extracting the domain name from a URL).

#### Q5: Can regex patterns be stored in variables?

Yes. Storing regex in variables improves ****readability and maintainability****, especially for long or complex patterns.

#### Q6: Are regex patterns the same across all tools?

Mostly similar, but there are slight syntax differences between ****Bash, grep, sed, and Perl-compatible regex****. Always test your patterns in the tool you’re using.

#### Q7: Is learning regex really worth it for scripting?

Absolutely. Regex is a ****universal skill**** - once learned, it applies not only in Bash but also in Python, JavaScript, databases, and many other technologies.