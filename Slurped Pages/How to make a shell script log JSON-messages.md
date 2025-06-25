---
link: https://stegard.net/2021/07/how-to-make-a-shell-script-log-json-messages/
slurped: 2025-06-05T16:28
title: How to make a shell script log JSON-messages – Øyvind Stegard
tags:
  - shell
  - json
  - log
---
If you have a shell script running in some environment where logs are expected to be formatted as JSON, it can be cumbersome to ensure all commands in the script output valid single line JSON-formatted messages, instead of just raw lines of text, which is what a shell script commonly does. Here I present a technique which can be used so that the script needs very little modifications to be able to output structured JSON instead of raw text lines.

We will setup a bash script so that its regular output is redirected to a JSON encoder [co-process](https://manpages.ubuntu.com/manpages/focal/man1/bash.1.html#shell%20grammar) automatically. This is setup at the beginning of the script, and subsequent commands’ output will automatically be wrapped as JSON-messages. It requires the [jq command](https://stegard.net/2021/03/command-line-jq-snippets/) to be present on the system where script runs.

```
#!/usr/bin/env bash

# 1 Make copies of shell's current stdout and stderr file
# descriptors:
exec 100>&1 200>&2

# 2 Define function which logs arguments or stdin as JSON-message to stdout:
log() {
    if [ "$1" = - ]; then
        jq -Rsc '{"@timestamp": now|strftime("%Y-%m-%dT%H:%M:%S%z"),
                  "message":.}' 1>&100 2>&200
    else
        jq --arg m "$*" -nc '{"@timestamp": now|strftime("%Y-%m-%dT%H:%M:%S%z"),
                              "message":$m}' 1>&100 2>&200
    fi
}

# 3 Start a co-process which transforms input lines to JSON messages:
coproc JSON_LOGGER { jq --unbuffered -Rc \
      '{"@timestamp": now|strftime("%Y-%m-%dT%H:%M:%S%z"),
        "message":.}' 1>&100 2>&200; }

# 4 Finally redirect shell's stdout/stderr to JSON logger
# co-process:
exec 1>&${JSON_LOGGER[1]} 2>&${JSON_LOGGER[1]}

# What follows is whatever you need your script to do

echo Hello brave world
echo '  testing "escaping" and white  space  '
echo >&2 this goes do stderr

uname

# If we want multiple output lines from a single command
# wrapped in a single JSON-message, we need to pipe it to the log function:
curl -sS --head https://api.github.com/|head -n 3|log -

# .. otherwise, each curl output line would become its own
# JSON-encoded message, which may not be desirable.
```

Output to a terminal with color support should look something like this:

```
{"@timestamp":"2021-07-01T14:55:15+02:00","message":"Hello brave world"}
{"@timestamp":"2021-07-01T14:55:15+02:00","message":"  testing \"escaping\" and white  space  "}
{"@timestamp":"2021-07-01T14:55:15+02:00","message":"this goes do stderr"}
{"@timestamp":"2021-07-01T14:55:15+02:00","message":"Linux"}
{"@timestamp":"2021-07-01T14:55:16+02:00","message":"HTTP/2 200 \r\nserver: GitHub.com\r\ndate: Thu, 01 Jul 2021 12:55:10 GMT\r"}
```

Jq will take care of all the necessary escaping and always produce valid single line JSON-structured messages, regardless of message payload.

## Notes

- You could make the above setup reusable, by putting the code in its own file and sourcing it at the beginning of scripts that need it.
- High precision timestamps cannot be generated natively in jq. If you need millisecond or higher precision on the log event timestamps, there are ways to do it, depending on the shell and command line tools available. If you have bash >= 5, you can modify the `log()` function so that timestamp string is generated using the expression:  
    `$(date +%Y-%m-%dT%H:%M:%S).${EPOCHREALTIME#*[.,]}$(date +%z`). You’ll need to pass this as an `--arg` to jq for each invocation and use it in the JSON template. Also, it is a bit harder to accomplish for the log transformer co-process, because ideally we’d like only a single persistent jq process running, for efficiency reasons and no pipeline buffering.
- You could easily expand the structured log messages by adding JSON fields to the jq templates. For example, you could add a `level` field, to indicate log level. Or a `hostname` field using the `$HOSTNAME` shell variable.
- Building on the previous point, you could create two separate JSON encoder functions, where one handles stderr messages and logs them at level ERROR, while another one logs regular stdout as level INFO. Then create two co-processes for stdout and stderr, with different jq JSON templates respectively. Finally, redirect main stdout to the first co-process and stderr to the second.
- For the log transformer co-process, be aware that pipeline buffering can have unfortunate effects, for instance missing the last log events before script exits. This is why jq is invoked with `--unbuffered`.