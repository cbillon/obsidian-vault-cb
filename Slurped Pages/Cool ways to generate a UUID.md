---
link: https://bhoot.dev/2024/cool-ways-to-get-uuid/
byline: |-
  Jayesh Bhoot's Ghost
    Town
excerpt: OK, may be not cool enough for Linux veterans, but cool enough for me!
slurped: 2025-02-08T09:08
title: Cool ways to generate a UUID
---

OK, may be not cool enough for Linux veterans, but cool enough for me!

There are two ways, in the order of coolness:

1. On Linux only: `cat /proc/sys/kernel/random/uuid`
    
    ```
		    $ cat /proc/sys/kernel/random/uuid
    fa199476-f6f3-4a32-ab89-91d81d9cc319
    ```
    
2. On Linux and macOS: `uuidgen`
    
    ```
    $ uuidgen
    6180fa98-ddd7-4777-a40d-b548db1f3444
    ```
    
    On macOS, `uuidgen` gives a UUID in uppercase. You can pipe it to `tr` command to translate it to lowercase.
    
    ```
    # On macOS
    $ uuidgen
    58D8D68C-B64D-4E1E-AC0F-B00AFAC1E895
    
    $ uuidgen | tr "[:upper:]" "[:lower:]"
    919ae848-df9c-41fa-b59f-de4b933c927c
    ```