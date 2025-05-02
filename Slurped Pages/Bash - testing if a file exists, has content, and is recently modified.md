---
link: https://fabianlee.org/2023/05/10/bash-testing-if-a-file-exists-has-content-and-is-recently-modified/
site: "Fabian Lee : Software Engineer"
excerpt: "If you need to test for a file’s existence, content size, and whether it was recently modified, the ‘find‘ utility can provide this functionality in a single call. One scenario for this usage might be the cached results from a remote service call (database, REST service, etc).  If fetching these results was a relatively costly ... Bash: testing if a file exists, has content, and is recently modified"
twitter: "https://twitter.com/Fabian Lee : Software Engineer"
tags:
  - slurp/cached
  - slurp/exists
  - slurp/find
  - slurp/gnu
  - slurp/length
  - slurp/local
  - slurp/locally
  - slurp/modified
  - slurp/size
  - bash
  - file
  - modified
slurped: 2025-03-02T10:08
title: "Bash: testing if a file exists, has content, and is recently modified"
---

![](https://fabianlee.org/wp-content/uploads/2018/10/gnu-logo.gif)If you need to test for a file’s existence, content size, and whether it was recently modified, the ‘[find](https://manpages.org/find)‘ utility can provide this functionality in a single call.

One scenario for this usage might be the cached results from a remote service call (database, REST service, etc).  If fetching these results was a relatively costly operation, you may want download the results only if the local cache was older than a certain threshold.

Below is an example where an http request to a remote site is cached locally, and re-downloaded if older than 24 hours.

cachefile=/tmp/cached_file.html

# test for existence, non-zero content, and modified in last day
find $cachefile -mtime -1 -size +0b 2>/dev/null | grep .
if [ $? -ne 0 ]; then
  echo "$cachefile needs to be downloaded"
  wget -q https://fabianlee.org/ -O $cachefile
  echo "DONE downloaded $cachefile"
else
  echo "$cachefile already exists, has content size, and was last modified in the last day"
fi

The first time you run this logic, it will download the locally cached file.  The second time, it will see the file exists and is recently modified and will not need to do the remote download.

Here is the full [test_cached_file.sh](https://github.com/fabianlee/blogcode/blob/master/bash/test_cached_file.sh) script on github.

REFERENCES

[man find](https://manpages.org/find)

[fabianlee github, script for this article](https://github.com/fabianlee/blogcode/blob/master/bash/test_cached_file.sh)

[stackoverflow, using grep at end of find to get 0/1 final result](https://serverfault.com/questions/225798/can-i-make-find-return-non-0-when-no-matching-files-are-found)

NOTES

An alternate way of checking for file size and modification date, but requires more setup.

# create file that looks like it was modified 1 day ago
agedfile=$(mktemp)
touch -d "1 days ago" $agedfile

if [[ ! -f "$cachefile" || $(stat -c%s $cachefile) -eq 0 || "$cachefile" -ot "$agedfile" ]]; then
  echo "file does not exist, or is zero content, or older. Need to redownload"
fi