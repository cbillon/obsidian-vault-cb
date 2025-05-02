---
link: https://fabianlee.org/2023/01/05/bash-awk-to-extract-nth-match-from-file-based-on-line-separator/
site: "Fabian Lee : Software Engineer"
excerpt: "If you need to extract the Nth occurrence of a match in a file (given definitive block separators), awk provides a convenient way to express the extraction. For example, a chained pem certificate will have multiple certification definitions with unique starting and ending marker lines.  Here is how you would extract the second certificate. awk ... Bash: awk to extract Nth match from file based on line separator"
twitter: "https://twitter.com/Fabian Lee : Software Engineer"
tags:
  - slurp/
  - bash
  - extract
slurped: 2025-03-02T10:14
title: "Bash: awk to extract Nth match from file based on line separator"
---

![](https://fabianlee.org/wp-content/uploads/2018/10/gnu-logo.gif)If you need to extract the Nth occurrence of a match in a file (given definitive block separators), awk provides a convenient way to express the extraction.

For example, a chained pem certificate will have multiple certification definitions with unique starting and ending marker lines.  Here is how you would extract the second certificate.

awk '/-----BEGIN CERTIFICATE-----/&&++k==2,/-----END CERTIFICATE-----/' my-chained.pem

As another example, in a [multi-document yaml](https://yaml.org/spec/1.0/#id2561718) file, three dashes isolated on a single line indicate an independent document.  Because the beginning and ending markers are the same, if you want to isolate the second match use the command below.

# second match isolated
awk '/---/&&++k==2,/FOOBAR/' my-multidoc.yaml | tail -n+2 | awk '//&&++k==1,/---/'

‘FOOBAR’ above does not hold any special meaning, it is simply line content that is unlikely to be found (and therefore captures all the way to the end of file).  

### Example Script

I’ve posted an example script on github, [awk_nth_match.sh](https://github.com/fabianlee/blogcode/blob/master/bash/awk_nth_match.sh)

wget https://raw.githubusercontent.com/fabianlee/blogcode/master/bash/awk_nth_match.sh
chmod +x awk_nth_match.sh

# illustrate use of awk to pull Nth occurence
./awk_nth_match.sh

REFERENCES

[yaml,org, multi-document yaml](https://yaml.org/spec/1.0/#id2561718)

[github fabianlee, example using awk to pull nth match](https://github.com/fabianlee/blogcode/blob/master/bash/awk_nth_match.sh)