---
tags:
  - ai
link: https://nelson.cloud/local-text-summarization-with-ollama-and-python-is-just-string-manipulation/
---
# Local Text Summarization With Ollama and Python Is Just String Manipulation

2025-08-24 · Updated 2026-01-25 · 3 min · 620 words  
[Python](https://nelson.cloud/categories/python/) [AI](https://nelson.cloud/categories/ai/) [Ollama](https://nelson.cloud/categories/ollama/) 

Table of Contents

- [Reading a Single File Into Ollama](https://nelson.cloud/local-text-summarization-with-ollama-and-python-is-just-string-manipulation/#reading-a-single-file-into-ollama)
- [Reading Multiple Files into Ollama](https://nelson.cloud/local-text-summarization-with-ollama-and-python-is-just-string-manipulation/#reading-multiple-files-into-ollama)
- [Dealing with Context Limits](https://nelson.cloud/local-text-summarization-with-ollama-and-python-is-just-string-manipulation/#dealing-with-context-limits)
- [References](https://nelson.cloud/local-text-summarization-with-ollama-and-python-is-just-string-manipulation/#references)

Table of Contents

- [Reading a Single File Into Ollama](https://nelson.cloud/local-text-summarization-with-ollama-and-python-is-just-string-manipulation/#reading-a-single-file-into-ollama)
- [Reading Multiple Files into Ollama](https://nelson.cloud/local-text-summarization-with-ollama-and-python-is-just-string-manipulation/#reading-multiple-files-into-ollama)
- [Dealing with Context Limits](https://nelson.cloud/local-text-summarization-with-ollama-and-python-is-just-string-manipulation/#dealing-with-context-limits)
- [References](https://nelson.cloud/local-text-summarization-with-ollama-and-python-is-just-string-manipulation/#references)

I’ve used LLMs before through an interface (e.g. [ChatGPT](https://chatgpt.com/), [Gemini](https://gemini.google.com/app), etc) but when I was trying to run an LLM locally I was overthinking how it worked.

Basically, it comes down to this: You pass in a string, and you get a string in return. That’s it.

So if we want to run an LLM locally using Python to summarize files, we build strings with Python and pass them into Ollama. If you want to read in files, open them in Python and concatenate the text with your prompt string. Then pass in the prompt string to Ollama.

Python is just a bridge between you and Ollama.

I’ve included some basic examples. The examples assume you have Ollama installed locally.

## [Reading a Single File Into Ollama](https://nelson.cloud/local-text-summarization-with-ollama-and-python-is-just-string-manipulation/#reading-a-single-file-into-ollama)

This is straightforward. Open a file and concatenate the text with a prompt which gets passed into Ollama:

```python
import ollama

# open a single file
file = open("path/to/file.txt")

# read it and concatenate to the prompt
prompt = f'Can you summarize this file for me? {file.read()}'

# pass in the prompt to Ollama
response = ollama.chat(
    model='gpt-oss:20b',
    messages=[
        {
            'role': 'user',
            'content': prompt
        }
    ]
)

print(response.message['content'])
```

## [Reading Multiple Files into Ollama](https://nelson.cloud/local-text-summarization-with-ollama-and-python-is-just-string-manipulation/#reading-multiple-files-into-ollama)

If you want to pass in multiple files in one prompt, you have to read and concatenate the files into a string, which you then concatenate into the prompt itself.

```python
import ollama

# open several textfiles
file = open("path/to/file.txt")
file2 = open("path/to/file2.txt")

# concatenate the textfiles into a single string
input = file.read() + file2.read()

# concatenate into the prompt
prompt = f'Can you summarize the following text for me? {input}'

# pass in the prompt to Ollama
response = ollama.chat(
    model='gpt-oss:20b',
    messages=[
        {
            'role': 'user',
            'content': prompt
        }
    ]
)

print(response.message['content'])
```

## [Dealing with Context Limits](https://nelson.cloud/local-text-summarization-with-ollama-and-python-is-just-string-manipulation/#dealing-with-context-limits)

If you want to read in multiple files but the files are huge, you may exceed the context limits of your model. You can still concatenate files/strings where possible but you can circumvent this by creating several chats. You’re basically running the code more than once, which means you can use a loop.

```python
import ollama

# open several textfiles and store them in a `files` list.
files = []
files.append(open("path/to/file.txt"))
files.append(open("path/to/file2.txt"))

# run a chat for each file in the list
for file in files:
    prompt = f'Can you summarize the following text for me? {file.read()}'

    # pass in the prompt to Ollama
    response = ollama.chat(
        model='gpt-oss:20b',
        messages=[
            {
                'role': 'user',
                'content': prompt
            }
        ]
    )

    print(response.message['content'])
```

The code above gives us separate summaries. But what if we want a single summary of all the files involved and they each exceed context limits? We can create a summary of each file, then create a summary of the summaries! It’s all string concatenation when you really think about it.

```python
import ollama

# save summaries in a list to summarize later on
summaries = []

# open several textfiles and store them in a `files` list.
files = []
files.append(open("path/to/file.txt"))
files.append(open("path/to/file2.txt"))

# run a chat for each file in the list
for file in files:
    prompt = f'Can you summarize the following text for me? {file.read()}'

    # pass in the prompt to Ollama
    response = ollama.chat(
        model='gpt-oss:20b',
        messages=[
            {
                'role': 'user',
                'content': prompt
            }
        ]
    )

    # append the summary of the file to the summaries list for later
    summaries.append(response.message['content'])


# create a single string from our list of summaries
summaries_string = "\n".join(summaries)

# start a final chat to summarize the summaries
prompt = f'Can you summarize the following text for me? {summaries_string}'

response = ollama.chat(
    model='gpt-oss:20b',
    messages=[
        {
            'role': 'user',
            'content': prompt
        }
    ]
)

print(response.message['content'])
```

That should be enough to get started. Thinking about all of this as just string manipulation made it “click” for me.

## [References](https://nelson.cloud/local-text-summarization-with-ollama-and-python-is-just-string-manipulation/#references)

- [https://ollama.com/download/](https://ollama.com/download/)
- [https://ollama.com/library/gpt-oss](https://ollama.com/library/gpt-oss)
- [https://github.com/ollama/ollama-python/tree/main/examples](https://github.com/ollama/ollama-python/tree/main/examples)