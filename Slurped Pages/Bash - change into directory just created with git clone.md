---
link: https://fabianlee.org/2023/10/13/bash-change-into-directory-just-created-with-git-clone/
site: "Fabian Lee : Software Engineer"
excerpt: "The first action you typically take after “git clone” is to change
  into the newly created directory.  This can be accomplished at the Bash shell
  in a couple of ways. Here is the git URL we will use as an example for this
  article. # can be https or ssh, can end in .git or ... Bash: change into
  directory just created with git clone"
twitter: "https://twitter.com/Fabian Lee : Software Engineer"
tags:
  - slurp/cd
  - slurp/clone
  - slurp/git
slurped: 2025-03-02T10:02
title: "Bash: change into directory just created with git clone"
---

![](https://fabianlee.org/wp-content/uploads/2018/10/gnu-logo.gif)The first action you typically take after “git clone” is to change into the newly created directory.  This can be accomplished at the Bash shell in a couple of ways.

Here is the git URL we will use as an example for this article.

# can be https or ssh, can end in .git or not
my_git_repo=https://github.com/fabianlee/blogcode.git

The easiest way is to call [basename](https://man7.org/linux/man-pages/man1/basename.1.html) on the [special underscore variable](https://tldp.org/LDP/Bash-Beginners-Guide/html/sect_03_02.html#sect_03_02_05) (last parameter of last command) stripping any .git suffix.

git clone $my_git_repo && cd $(basename $_ .git)

The second way requires that we specify a directory name parameter for the clone command, which creates a clean last parameter.

git clone $my_git_repo my_new_dir && cd $_

REFERENCES

[tldp.org, special variables for Bash](https://tldp.org/LDP/Bash-Beginners-Guide/html/sect_03_02.html#sect_03_02_05)

[stackoverflow.com, clone and change into directory](https://stackoverflow.com/questions/56414849/how-do-i-git-clone-and-cd-into-the-folder-i-just-made-in-one-line)

[unix.stackexchange, cd automatically after git clone](https://unix.stackexchange.com/questions/97920/how-to-cd-automatically-after-git-clone)

NOTES

In a similar fashion, you can download and unarchive a file

wget -Nq https://github.com/smallstep/cli/releases/download/v0.25.0/step_linux_0.25.0_amd64.tar.gz && tar xfz $(basename $_)