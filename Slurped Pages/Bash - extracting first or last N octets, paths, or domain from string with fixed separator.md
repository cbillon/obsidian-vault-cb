---
link: https://fabianlee.org/2023/10/09/bash-extracting-first-or-last-n-octets-paths-or-domain-from-string-with-fixed-separator/
site: "Fabian Lee : Software Engineer"
excerpt: "When parsing a string that is divided by a separator char, getting the
  first N values OR last N values is a common scenario when dealing with: IP
  address separated by periods, e.g. “10.11.12.13” File path separated by
  forward slash “/tmp/myfolder/subpath1/subpath2/subpath3” Fully qualified
  domain separated by periods “sub1.sub2.my.domain.com” Getting first N values
  Getting the first ... Bash: extracting first or last N octets, paths, or
  domain from string with fixed separator"
twitter: "https://twitter.com/Fabian Lee : Software Engineer"
tags:
  - slurp/array
  - slurp/cut
  - slurp/first
  - slurp/folder
  - slurp/fqdn
  - slurp/grep
  - slurp/ip
  - slurp/last
  - slurp/octet
  - slurp/parse
  - slurp/path
  - slurp/subdomain
slurped: 2025-03-02T10:03
title: "Bash: extracting first or last N octets, paths, or domain from string
  with fixed separator"
---

![](https://fabianlee.org/wp-content/uploads/2018/10/gnu-logo.gif)When parsing a string that is divided by a separator char, getting the first N values OR last N values is a common scenario when dealing with:

- IP address separated by periods, e.g. “10.11.12.13”
- File path separated by forward slash “/tmp/myfolder/subpath1/subpath2/subpath3”
- Fully qualified domain separated by periods “sub1.sub2.my.domain.com”

## Getting first N values

Getting the first N values is relatively simple because we know the exact indexes of the values we need. Although we could use awk or arrays (as shown in the NOTES at the bottom of this page), the simplest method is ‘cut’.

Get the first 3 octets of an IP address, returning “10.11.12”

IP="10.11.12.13"
echo "$IP" | cut -d. -f1-3

Get the first 3 paths of a folder path, returning “/tmp/myfolder/subpath1”

fpath="/tmp/myfolder/subpath1/subpath2/subpath3"
echo "$fpath" | cut -d/ -f1-4

Get the first 3 most specific domain names, returning “sub1.sub2.my”

fulldomain="sub1.sub2.my.domain.com"
echo "$fulldomain" | cut -d. -f1-3

## Getting last N values

Getting the last N values is a bit trickier than the first N.  Although we could use ‘cut’ and ‘awk’, it would require precalculated counts of the separator char.  So instead we rely of getting the last elements of an array, where the length of an array can be fetched with the syntax ${#arrayname[@]} and a range of elements can be fetched using ${arrayname[@]:begin:end}

Get the last 3 octets of an IP address, returning “11.12.13”

IP="10.11.12.13"
IFS=. read -ra iparray <<< "$IP"
echo ${iparray[@]:${#iparray[@]}-3:${#iparray[@]}-1} | tr ' ' '.'

Get the last 3 paths of a folder, returning “/subpath1/subpath2/subpath3”

fpath="/tmp/myfolder/subpath1/subpath2/subpath3"
IFS=/ read -ra folderarray <<< "$fpath"
echo -en "/" ; echo ${folderarray[@]:${#folderarray[@]}-3:${#folderarray[@]}-1} | tr ' ' '/'

Get the last 3 domain names from a FQDN, returning “my.domain.com”

fulldomain="sub1.sub2.my.domain.com"
IFS=. read -ra domainarray <<< "$fulldomain"
echo ${domainarray[@]:${#domainarray[@]}-3:${#domainarray[@]}-1} | tr ' ' '.'

REFERENCES

[man cut](https://www.man7.org/linux/man-pages/man1/cut.1.html)

[man grep](https://www.man7.org/linux/man-pages/man1/grep.1.html)

[gnu.org, printf and awk](https://www.gnu.org/software/gawk/manual/html_node/Printf-Examples.html)

[stackoverflow, split string by separator char into array](https://stackoverflow.com/questions/60746331/simplest-way-to-split-a-csv-string-into-an-array-that-works-for-both-bash-zsh)

NOTES

Below are three different ways of getting the first 3 octets of an IP address, returning “10.11.12”.  Using ‘cut’ is the easiest method.

IP="10.11.12.13"
# method 1: using cut
echo "$IP" | cut -d. -f1-3
# method2: using awk
echo "$IP" | awk -F. '{ printf "%s.%s.%s\n", $1,$2,$3 }'
# method 3: using array
IFS=. read -ra iparray <<< "$IP"
echo ${iparray[@]:0:3} | tr ' ' '.'

Below are three different ways of getting the first 3 paths of a folder, returning “/tmp/myfolder/subpath1”.  Using ‘cut’ is the easiest method.

fpath="/tmp/myfolder/subpath1/subpath2/subpath3"
# method 1: using cut
echo "$fpath" | cut -d/ -f1-4
# method2: using awk
echo "$fpath" | awk -F/ '{ printf "%s/%s/%s/%s\n", $1,$2,$3,$4 }'
# method 3: using array
IFS=/ read -ra folderarray <<< "$fpath"
echo -en "/" ; echo ${folderarray[@]:0:4} | tr ' ' '/'

Below are three different ways of getting the first 3 most specific domain names, returning “sub1.sub2.my”.  Using ‘cut’ is the easiest method.

fulldomain="sub1.sub2.my.domain.com"
# method 1: using cut
echo "$fulldomain" | cut -d. -f1-3
# method2: using awk
echo "$fulldomain" | awk -F. '{ printf "%s.%s.%s\n", $1,$2,$3 }'
# method 3: using array
IFS=. read -ra domainarray <<< "$fulldomain"
echo ${domainarray[@]:0:3} | tr ' ' '.'