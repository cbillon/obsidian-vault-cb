---
link: https://joshtronic.com/2023/03/12/parsing-arguments-in-shell-script/
byline: Josh Sherman
excerpt: |-
  Posted 12-Mar-2023
              
              in Command-line Interface
              
                tagged
                
                  Shell Script
slurped: 2026-01-22T09:53
title: Parsing arguments in shell script
tags:
  - bash
---

I write a good amount of shell scripts, but I tend to use arguments pretty sparingly. Because of this, my implementation to handle said arguments tends to be pretty weak. Some good examples are the arguments needing to be passed in a specific order and a lack of using long or shorthand arguments.

While this works out well enough, especially if you only have a single argument, things can always be better.

With a new script needing to be written last week, I set out to learn how to improve parsing of command-line arguments in shell scripts. In doing so, I ended up learning some new stuff, like the build-in `shift` to help iterate through the argument vector.

I've also embraced starting my script by writing up a help section that tells you a bit about the script as well as demonstrates the usage.

Let's say we wanted to write a script that accepted a required argument to specify the environment we're using, and an optional argument that tells the script to perform an action, like deleting data, we can start by writing a help function like this:

```
function usage {
  echo ""
  echo "Usage: $0 -e|--env <dev|qa|prod> [-d|--delete]"
  echo ""
  echo "List out stuff in environment, optionally delete it."
  echo ""
  echo "Options:"
  echo "  -d, --delete     Delete the stuff."
  exit 1
}
```

Not much to it, but it not only helps a new user to the script understand what it's about, but also sets the stage for the code we're going to write to parse the referenced arguments.

This is where `shift` comes in. Instead of just interrogating `$1`, `$2`, and such, we're going to loop through the argument vector. Depending on if the argument will accept a value or not, we'll use `shift` to remove the first item from the array. This also moves the rest of the items up to the previous index, so `$3` becomes `$2`, `$2` becomes `$1`, and so on.

For arguments that accept values, we'll want to use `shift` twice and for everything else, we will use `shift` once. If the `--help` argument is specified, or if the argument isn't recognized at all, we'll present the user with our help information:

```
while [[ $# -gt 0 ]]; do
  case $1 in
    -e|--env)
      ENV="$2"
      shift
      shift
      ;;
    -d|--delete)
      DELETE=true
      shift
      ;;
    -h|--help)
      usage
      shift
      ;;
    *)
      echo "Invalid option: $1"
      usage
      ;;
  esac
done
```

In parsing through the arguments, we created a couple of variables that represents the information. You can change those as you see fit, or even expand the `case` statement to handle even more options.

Even though we're done parsing the arguments, we still have a bit of work to do. One of the arguments, `--env` is not only required, but it also only accepts a fixed set of values.

We can simply sanity check that the variable we set above, `$ENV`, is in fact set. If so, we can do an additional sanity check to ensure that it matches of the expected set of values:

```
if [ -z "$ENV" ]; then
  echo "Missing required option: -e|--env"
  usage
elif [[ ! "$ENV" =~ ^(dev|qa|prod)$ ]]; then
  echo "Invalid environment: $ENV"
  usage
fi
```

At this point, we've successfully parsed the command-line arguments passed to our script, and ensured that the necessary arguments are there and properly set.

The arguments can also be passed in any order, since we're looping through the argument vector and not expecting anything in a certain position.

If we were to put everything together, it would look like this:

```
#!/usr/bin/env bash
# Function to print usage instructions
function usage {
echo ""
echo "Usage: $0 -e|--env <dev|qa|prod> [-d|--delete]"
echo ""
echo "List out stuff in environment, optionally delete it."
echo ""
echo "Options:"
echo "  -d, --delete     Delete the stuff."
exit 1
}
# Parse command line arguments
while [[ $# -gt 0 ]]; do
case $1 in
-e|--env)
ENV="$2"
shift
shift
;;
-d|--delete)
DELETE=true
shift
;;
-h|--help)
usage
shift
;;
*)
echo "Invalid option: $1"
usage
;;
esac
done
# Check if our environment is set and valid
if [ -z "$ENV" ]; then
echo "Missing required option: -e|--env"
usage
elif [[ ! "$ENV" =~ ^(dev|qa|prod)$ ]]; then
echo "Invalid environment: $ENV"
usage
fi
# List out stuff in environment
```