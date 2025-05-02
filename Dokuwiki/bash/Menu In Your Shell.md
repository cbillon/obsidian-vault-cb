---
link: https://dev.to/vaiolabs_io/menu-in-your-shell-2a30
byline: Alex M. Schapelle
site: DEV Community
date: 2023-11-15T17:20
excerpt: Welcome gentle reader. My name is Silent-Mobius, also known as Alex M.
  Schapelle, your humanoid...
twitter: https://twitter.com/@
tags:
  - bash
  - linux
  - whiptail
  - script
  - software
  - coding
  - development
  - engineering
  - inclusive
  - community
slurped: 2024-10-20T08:35
title: Menu In Your Shell
---

[![Cover image for Menu In Your Shell](https://media.dev.to/dynamic/image/width=1000,height=420,fit=cover,gravity=auto,format=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F3ocuovwjhg16w9ihp7g2.png)](https://media.dev.to/dynamic/image/width=1000,height=420,fit=cover,gravity=auto,format=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F3ocuovwjhg16w9ihp7g2.png)

Welcome gentle reader. My name is Silent-Mobius, also known as Alex M. Schapelle, your humanoid cog-wheel.  
In today's issue I'd like to talk about something old yet somewhat useful especially if one seeks guided automation:

Terminal User Interface Menus AKA Dialog/Whiptail

Whiptail is a program that allows shell scripts to display dialog boxes to the user for informational purposes, or to get input from the user in a friendly way. Whiptail is included by default on Debian, which is a type of requirement

### [](https://dev.to/vaiolabs_io/menu-in-your-shell-2a30#the-motivation)The Motivation

I've delegated a task to one of my junior devops engineers, which to my surprise, have not heard of `whiptail` or `dialog`, thus I'd like to write for everyone a short tutorial on how to work with this tool.

### [](https://dev.to/vaiolabs_io/menu-in-your-shell-2a30#use-cases)Use Cases

Usually when working on software development project, `Bash`/`Shell` scripting is invoked to automate several manual tasks, such as:

- Library or tool installs
- Configuration setup
- Development environment setup
- So on. We can assume the use cases revolve around this idea of `automated setup`. It can be argued if this is the right tool or not for the task, yet many huge projects that prefer to stick to the tool that has the minimal pre requirements, on multiple platforms thus all of them come to conclusion to use `Shell` script with included binary tools.

`Whiptail` is included on most of Debian based system or on gnome2/xfce/mate/cinnamon desktop environments and can be used to setup project development environments and all mentioned above with guided menu that is terminal based.

### [](https://dev.to/vaiolabs_io/menu-in-your-shell-2a30#basics)Basics

#### [](https://dev.to/vaiolabs_io/menu-in-your-shell-2a30#basic-dialog)Basic Dialog

You don't actually need a script to display a basic dialog box from `whiptail`. Let's declare and then call a variable. The variable is merely a message of some sort that will be acknowledged with an `OK` button. No choices are offered, and no navigation is used.

At the command prompt, type the following information:

```


message="Whiptail: Tool Basics"
whiptail --msgbox --title "Intro to Whiptail" "$message" 25 80


```

 

- [+] `whiptail` is a cli command/tool that accepts sub commands
- [+] We'll discuss some of the sub-commands and will provide general suggested structure of the script from **my own personal view**
- [+] We'll include variables and data structures of shell script, thus understanding `shell` and its internals is somewhat of a requirement
- [+] Values `25` and `80` represent column measurements and they define the size of the interface window.

The output should look like this:

[![Image description](https://media.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F2pr8kqaf9lsi8tx64c92.png)](https://media.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F2pr8kqaf9lsi8tx64c92.png)

Let's go on and learn additional parts of `whiptail`

#### [](https://dev.to/vaiolabs_io/menu-in-your-shell-2a30#yesno-box)Yes/no Box

The simplest way to get input from the user is via a `Yes/no` box. This displays a dialog with two buttons labelled `Yes` and `No`

```


if whiptail --title "Example Dialog" --yesno "This is an example of a yes/no box." 29 80; then
    echo "User selected Yes, exit status was $?."
else
    echo "User selected No, exit status was $?."
fi


```

 

[![Image description](https://media.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fd3xsd1vbxo1r0uk2p11y.png)](https://media.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fd3xsd1vbxo1r0uk2p11y.png)

#### [](https://dev.to/vaiolabs_io/menu-in-your-shell-2a30#input-box)Input Box

When ever the direct input from the user required we can use `input box`. This displays a dialog with input area and two buttons accepting or canceling the input.

```


COLOR=$(whiptail --inputbox "What is your favorite Color?" 8 39 Blue --title "Example Dialog" 3>&1 1>&2 2>&3)
                                                                        # A trick to swap stdout and stderr.
# Again, you can pack this inside if, but it seems really long for some 80-col terminal users.
exitstatus=$?
if [ $exitstatus = 0 ]; then
    echo "User selected Ok and entered " $COLOR
else
    echo "User selected Cancel."
fi

echo "(Exit status was $exitstatus)"


```

 

When the user confirms their input, a list of the choices is printed to `stderr`. Yes, you read that correctly: `stderr`, and not `stdout`, which is where you pick up the user's input to consume it in the script. The way around this issue is to reverse the redirection so that the user's input goes to `stdout`.

Here is the phrase for doing that:

```


3>&1 1>&2 2>&3


```

 

Explanation:

- Create a file descriptor 3 that points to 1 (`stdout`)
- Redirect 1 (`stdout`) to 2 (`stderr`)
- Redirect 2 (`stderr`) to the 3 file descriptor, which is pointed to `stdout`

[![Image description](https://media.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fgmhyd70y17l2dmzeke63.png)](https://media.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fgmhyd70y17l2dmzeke63.png)

#### [](https://dev.to/vaiolabs_io/menu-in-your-shell-2a30#text-box)Text Box

A text box with contents of the given file inside. Add --scrolltext if the filename is longer than the window.

```


echo "Welcome to Bash $BASH_VERSION" > test_textbox
whiptail --textbox test_textbox 12 80


```

 

[![Image description](https://media.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fqmu7t9xoe6nlgimtijxr.png)](https://media.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fqmu7t9xoe6nlgimtijxr.png)

#### [](https://dev.to/vaiolabs_io/menu-in-your-shell-2a30#password-box)Password Box

A way to get a hidden password from the user is via a password box. This displays a dialog with two buttons labelled Ok and Cancel

```


PASSWORD=$(whiptail --passwordbox "please enter your secret password" 8 78 --title "password dialog" 3>&1 1>&2 2>&3)
                                                                        for some 80-col terminal users.
exitstatus=$?
if [ $exitstatus == 0 ]; then
    echo "User selected Ok and entered " $PASSWORD
else
    echo "User selected Cancel."
fi

echo "(Exit status was $exitstatus)"


```

 

#### [](https://dev.to/vaiolabs_io/menu-in-your-shell-2a30#menu)Menu

Whenever you want to present a list of options to the user, whiptail has several dialog types to choose from.

A menu should be used when you want the user to select one option from a list, such as for navigating a program.

```


whiptail --title "Menu example" --menu "Choose an option" 25 78 16 \
"<-- Back" "Return to the main menu." \
"Add User" "Add a user to the system." \
"Modify User" "Modify an existing user." \
"List Users" "List all users on the system." \
"Add Group" "Add a user group to the system." \
"Modify Group" "Modify a group and its list of members." \
"List Groups" "List all groups on the system."


```

 

The values given to --menu are:

- The text describing the menu ("Choose an option")
- The height of the dialog (25)
- The width of the dialog (78)
- The height of the menu list (16)

The rest of the values are a list of menu options in the format tag item, where tag is the name of the option which is printed to `stderr` when selected, and item is the description of the menu option.

If you are presenting a very long menu and want to make best use of the available screen, you can calculate the best box size by.

#### [](https://dev.to/vaiolabs_io/menu-in-your-shell-2a30#lists)Lists

At some point, you will want to present options to the user which would not be appropriate to place in a menu  
A check list allows a user to select one or more options from a list.

```


whiptail --title "Check list example" --checklist \
"Choose user's permissions" 20 78 4 \
"NET_OUTBOUND" "Allow connections to other hosts" ON \
"NET_INBOUND" "Allow connections from other hosts" OFF \
"LOCAL_MOUNT" "Allow mounting of local devices" OFF \
"REMOTE_MOUNT" "Allow mounting of remote devices" OFF


```

 

[![Image description](https://media.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F3ekh5ke5jltud1jxroia.png)](https://media.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F3ekh5ke5jltud1jxroia.png)

In case you prefer to use **single choice** in your lists the **--radiolist** is the suggested list for you.

```


whiptail --title "Radio list example" --radiolist \
"Choose user's permissions" 20 78 4 \
"NET_OUTBOUND" "Allow connections to other hosts" ON \
"NET_INBOUND" "Allow connections from other hosts" OFF \
"LOCAL_MOUNT" "Allow mounting of local devices" OFF \
"REMOTE_MOUNT" "Allow mounting of remote devices" OFF


```

 

[![Image description](https://media.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fgw9gaaerldbmqh1d9789.png)](https://media.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fgw9gaaerldbmqh1d9789.png)

#### [](https://dev.to/vaiolabs_io/menu-in-your-shell-2a30#progress-gauge)Progress Gauge

In cases where progress is required,one might use progress gauge

```


#!/bin/bash

NULL=/dev/null
PKG_LIST=(terminator mtr apt-transport-https  guake plank \ geany meld git gitg ethtool arp-scan nmap python3-nmap vlc \
netcat macchanger arp-scan nmap gnupg2 curl wget unrar make \ cmake ipython3 pipenv vim qemu-kvm libvirt-daemon \
plymouth plymouth-themes remmina glade terminator \
vlc virt-manager bash-completion vagrant darkslide)

{
    for (( pkg_num=0; pkg_num <= ${#PKG_LIST[@]}; pkg_num+=1))
    do
        dnf install -y ${PKG_LIST[pkg_num]} > $NULL 2>&1
    echo $pkg_num
    done
} |  whiptail --gauge "Please wait while we install $pkg..." 6 50 0



```

 

[![Image description](https://media.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fcjxjvfd8uw9vcxy3ment.png)](https://media.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fcjxjvfd8uw9vcxy3ment.png)

### [](https://dev.to/vaiolabs_io/menu-in-your-shell-2a30#structure)Structure

Usually when writing the shell scripts, if not guided, sysadmin/devops/developers tend to copy/paste logical list of commands without considering the fact if they will need to move the project somewhere else, or things like dependencies and so on, thus making the shell script unreadable or not understandable.  
Here below I am suggesting a C program like structure that is way more readable compared to what I've been taught:

```


#!/usr/bin/env bash 
############## Safe header start ############################
# Created by: Silent Mobius AKA Alex M. Schapelle
# Purpose: provide script structure suggestion
# Version: 0.1.0
# Update_date: 14.12.2023

set -o errexit
set -o pipefail

############## Safe header stop ############################

function main(){
  welcome
  options
}

function welcome(){
  whiptail --textbox "Welcome to MenuScript" 12 80
}

function options(){
OPTS=$(whiptail --title "Choose Option" --radiolist </span>
"Choose user's permissions" 20 78 4 </span>
"password_change" "Change your user password" ON </span>
"restart_network" "Restart the current network" OFF  3>&1 1>&2 2>&3 )

if [[ -z $OPTS ]];then
  echo "No Option Chosen"; exit 0
else
  for opt in ${OPTS}
    do
      case $opt in 
        "password_change")
            echo "Please change password"
            ;;
        "restart_network")
            echo "Restarting the network"
            ;;
        *)
            echo "Unsupported item $opt!" >&2
            exit 1
            ;;
      esac
    done
}

#####
# Main - _- _- _- _- _- _- Do Not Remove - _- _- _- _- _- _- _
#####
main "$@"

```

 

###   
[  
](https://dev.to/vaiolabs_io/menu-in-your-shell-2a30#summary)  
Summary  

Hope that it has been some what guiding article and that my intentions in regards to your all education evolution has been have been fruitful.  
What ever you do, please remember: Do Try To Have Some Fun.

As usually I could not have written it alone, thus here are some links I used to scribe this short tutorial.

- [wikibook](https://en.wikibooks.org/wiki/Bash_Shell_Scripting/Whiptail)
- [redhat portal for sysadmins](https://www.redhat.com/sysadmin/use-whiptail)
- [Gijs tech portal](https://gijs-de-jong.nl/posts/pretty-dialog-boxes-for-your-shell-scripts-using-whiptail/)