---
link: https://fabianlee.org/2023/08/27/bash-decoding-a-jwt-from-the-command-line-with-jq/
site: "Fabian Lee : Software Engineer"
excerpt: "Although jwt.io has become a common online destination for decoding JWT, this can also be done locally using jq. # populate JWT variable JWT=... # decode with jq utility echo $JWT | jq -R 'split(\".\") | .[0],.[1] | @base64d | fromjson' Attribution of credit goes to this gist. If you have not installed jq on ... Bash: decoding a JWT from the command line with jq"
twitter: "https://twitter.com/Fabian Lee : Software Engineer"
tags:
  - base64
  - jwt
  - jq
  - bash
slurped: 2025-03-02T10:05
title: "Bash: decoding a JWT from the command line with jq"
---

August 27, 2023

  

Categories: [Linux](https://fabianlee.org/category/linux/)  

![](https://fabianlee.org/wp-content/uploads/2018/10/gnu-logo.gif)Although [jwt.io](https://jwt.io/) has become a common online destination for decoding JWT, this can also be done locally using [jq](https://jqlang.github.io/jq/download/).

# populate JWT variable
JWT=...

# decode with jq utility
echo $JWT | jq -R 'split(".") | .[0],.[1] | @base64d | fromjson'

Attribution of credit goes to [this gist](https://gist.github.com/angelo-v/e0208a18d455e2e6ea3c40ad637aac53).

If you have not installed jq on Debian/Ubuntu, it is offered in the official repositories.

sudo apt install -y jq

For other OS or direct downloads, see the [jq official page](https://jqlang.github.io/jq/download/).

REFERENCES

[github.com gist, original gist](https://gist.github.com/angelo-v/e0208a18d455e2e6ea3c40ad637aac53)

[prefetch.net, command used in this article](https://prefetch.net/blog/2020/07/14/decoding-json-web-tokens-jwts-from-the-linux-command-line/)

[jq downloads/install](https://jqlang.github.io/jq/download/)