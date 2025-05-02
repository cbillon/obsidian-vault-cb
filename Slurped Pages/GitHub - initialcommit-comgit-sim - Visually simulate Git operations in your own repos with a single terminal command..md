---
link: https://github.com/initialcommit-com/git-sim
byline: initialcommit-com
site: GitHub
excerpt: Visually simulate Git operations in your own repos with a single terminal command. - initialcommit-com/git-sim
twitter: https://twitter.com/@github
slurped: 2025-02-21T13:18
title: "GitHub - initialcommit-com/git-sim: Visually simulate Git operations in your own repos with a single terminal command."
tags:
  - git
  - simulation
  - sim
---

## git-sim

[](https://github.com/initialcommit-com/git-sim#git-sim)

[![git-sim-logo-with-tagline-1440x376p45](https://user-images.githubusercontent.com/49353917/232990611-58d0693f-69c0-45c8-b51d-cd540793d18c.gif)](https://user-images.githubusercontent.com/49353917/232990611-58d0693f-69c0-45c8-b51d-cd540793d18c.gif)

[![GitHub license](https://camo.githubusercontent.com/31b7b0e455fd7f8ce089617358f162426e17a27c10767ff81607adab33e9f541/68747470733a2f2f696d672e736869656c64732e696f2f6769746875622f6c6963656e73652f696e697469616c636f6d6d69742d636f6d2f6769742d73696d)](https://github.com/initialcommit-com/git-sim/blob/main/LICENSE) [![GitHub tag](https://camo.githubusercontent.com/07f9c678da51660af4001f6feae469ca241b23277c32b32148d6e7b5939c82aa/68747470733a2f2f696d672e736869656c64732e696f2f6769746875622f762f72656c656173652f696e697469616c636f6d6d69742d636f6d2f6769742d73696d)](https://img.shields.io/github/v/release/initialcommit-com/git-sim) [![Downloads](https://camo.githubusercontent.com/a54eb0c17985541c7db394d52d8a95bff5b7fde6b076a61806203af239d4ef5c/68747470733a2f2f7374617469632e706570792e746563682f62616467652f6769742d73696d)](https://pepy.tech/project/git-sim) [![Contributors](https://camo.githubusercontent.com/d5bca66912f895bccf074b6b0f14b2f908a2f3e060edc62f043b0a93b71ef56b/68747470733a2f2f696d672e736869656c64732e696f2f6769746875622f636f6e7472696275746f72732f696e697469616c636f6d6d69742d636f6d2f6769742d73696d)](https://github.com/initialcommit-com/git-sim/graphs/contributors) [![Share](https://camo.githubusercontent.com/c8c09a37598b01bbfb8a923c2a5d11f19f4ae6a9d03931505edb686b11107db6/68747470733a2f2f696d672e736869656c64732e696f2f747769747465722f75726c3f6c6162656c3d53686172652675726c3d6874747073253341253246253246747769747465722e636f6d253246696e6974636f6d6d6974)](https://twitter.com/intent/tweet?text=Check%20out%20%23gitsim%20%2D%20a%20tool%20to%20visualize%20%23Git%20operations%20in%20your%20local%20repos%20with%20a%20single%20terminal%20command,%20by%20%40initcommit!%20https%3A%2F%2Fgithub%2Ecom%2Finitialcommit%2Dcom%2Fgit%2Dsim)

Visually simulate Git operations in your own repos with a single terminal command.

This generates an image (default) or video visualization depicting the Git command's behavior.

Command syntax is based directly on Git's command-line syntax, so using git-sim is as familiar as possible.

Example: `$ git-sim merge <branch>`

[![git-sim-merge_04-22-23_21-04-32_cropped](https://user-images.githubusercontent.com/49353917/233821875-a7bb640d-10be-4433-a8fb-bd25646eeff4.jpg)](https://user-images.githubusercontent.com/49353917/233821875-a7bb640d-10be-4433-a8fb-bd25646eeff4.jpg)

Check out the [git-sim release blog post](https://initialcommit.com/blog/git-sim) for the full scoop!

## Support git-sim

[](https://github.com/initialcommit-com/git-sim#support-git-sim)

Git-Sim is Free and Open-Source Software (FOSS). Your support will help me work on it (and other Git projects) full time!

- [Sponsor Git-Sim on GitHub](https://github.com/sponsors/initialcommit-com)
- [Support Git-Sim via Patreon](https://patreon.com/user?u=92322459)

## Use cases

[](https://github.com/initialcommit-com/git-sim#use-cases)

- Visualize Git commands to understand their effects on your repo before actually running them
- Prevent unexpected working directory and repository states by simulating before running
- Share visualizations (jpg/png image or mp4/webm video) of your Git commands with your team, or the world
- Save visualizations as a part of your team documentation to document workflow and prevent recurring issues
- Create static Git diagrams (jpg/png) or dynamic animated videos (mp4/webm) to speed up content creation
- Help visual learners understand how Git commands work
- Combine with bundled command [git-dummy](https://github.com/initialcommit-com/git-dummy) to generate a dummy Git repo and then simulate operations on it

## Features

[](https://github.com/initialcommit-com/git-sim#features)

- Run a one-liner git-sim command in the terminal to generate a custom Git command visualization (.jpg) from your repo
- Supported commands: `add`, `branch`, `checkout`, `cherry-pick`, `clean`, `clone`, `commit`, `config`, `fetch`, `init`, `log`, `merge`, `mv`, `pull`, `push`, `rebase`, `remote`, `reset`, `restore`, `revert`, `rm`, `stash`, `status`, `switch`, `tag`
- Generate an animated video (.mp4) instead of a static image using the `--animate` flag (note: significant performance slowdown, it is recommended to use `--low-quality` to speed up testing and remove when ready to generate presentation-quality video)
- Color commits by parameter, such as author with the `--color-by=author` option
- Choose between dark mode (default) and light mode
- Specify output formats of either jpg, png, mp4, or webm
- Combine with bundled command [git-dummy](https://github.com/initialcommit-com/git-dummy) to generate a dummy Git repo and then simulate operations on it
- Animation only: Add custom branded intro/outro sequences if desired
- Animation only: Speed up or slow down animation speed as desired

## Quickstart

[](https://github.com/initialcommit-com/git-sim#quickstart)

Note: If you prefer to install git-sim with Docker, skip steps (1) and (2) here and jump to the [Docker installation](https://github.com/initialcommit-com/git-sim#docker-installation) section below, then come back here to step (3).

1. **Install Manim and its dependencies for your OS / environment:**
    
    - [Install Manim on Windows](https://docs.manim.community/en/stable/installation/windows.html)
    - [Install Manim on MacOS](https://docs.manim.community/en/stable/installation/macos.html)
    - [Install Manim on Linux](https://docs.manim.community/en/stable/installation/linux.html)
    - [Install Manim in Conda](https://docs.manim.community/en/stable/installation/conda.html)
2. Install `git-sim`:
    

Note: For MacOS, it is recommended to **NOT** use the system Python to install Git-Sim, and instead use [Homebrew](https://brew.sh/) to install a version of Python to work with Git-Sim. Virtual environments should work too.

3. Browse to the Git repository you want to simulate Git commands in:

4. Run the program:

$ git-sim [global options] <subcommand> [subcommand options]

Optional: If you don't have an existing Git repo to simulate commands on, use the bundled [git-dummy](https://github.com/initialcommit-com/git-dummy) command to generate a dummy Git repo with the desired number of branches and commits to simulate operations on with git-sim:

$ git-dummy --name="dummy-repo" --branches=3 --commits=10
$ cd dummy-repo
$ git-sim [global options] <subcommand> [subcommand options]

Or if you want to do it all in a single command:

$ git-dummy --no-subdir --branches=3 --commits=10 && git-sim [global options] <subcommand> [subcommand options]

5. Simulated output will be created as a `.jpg` file. Output files are named using the subcommand executed combined with a timestamp, and by default are stored in a subdirectory called `git-sim_media/`. The location of this subdirectory is customizable using the command line flag `--media-dir=path/to/output`. Note that when the `--animate` global flag is used, render times will be much longer and a `.mp4` video output file will be produced.
    
6. For convenience, environment variables can be set for any global command-line option available in git-sim. All environment variables start with `git_sim_` followed by the name of the option.
    

For example, the `--media-dir` option can be set as an environment variable like:

$ export git_sim_media_dir=~/Desktop

Similarly, the `--speed` option can be set like:

Boolean flags can be set like:

$ export git_sim_light_mode=true

In general:

$ export git_sim_option_name=option_value

Explicitly specifying options at the command-line takes precedence over the corresponding environment variable values.

7. See global help for list of global options/flags and subcommands:

8. See subcommand help for list of options/flags for a specific subcommand:

$ git-sim <subcommand> -h

## Requirements

[](https://github.com/initialcommit-com/git-sim#requirements)

- Python 3.7 or greater
- Pip (Package manager for Python)
- [Manim (Community version)](https://www.manim.community/)

## Commands

[](https://github.com/initialcommit-com/git-sim#commands)

Basic usage is similar to Git itself - `git-sim` takes a familiar set of subcommands including "add", "branch", "checkout", "cherry-pick", "clean", "clone", "commit", "config", "fetch", "init", "log", "merge", "mv", "pull", "push", "rebase", "remote", "reset", "restore", "revert", "rm", "stash", "status", "switch", "tag" along with corresponding options.

$ git-sim [global options] <subcommand> [subcommand options]

The `[global options]` apply to the overarching `git-sim` simulation itself, including:

`-n <number>`: Number of commits to display from each branch head.  
`--all`: Display all local branches in the log output.  
`--animate`: Instead of outputting a static image, animate the Git command behavior in a .mp4 video.  
`--color-by author`: Color commits by parameter, such as author.  
`--invert-branches`: Invert positioning of branches by reversing order of multiple parents where applicable.  
`--hide-merged-branches`: Hide commits from merged branches, i.e. only display mainline commits.  
`--media-dir`: The path at which to store the simulated output media files.  
`-d`: Disable the automatic opening of the image/video file after generation. Useful to avoid errors in console mode with no GUI.  
`--light-mode`: Use a light mode color scheme instead of default dark mode.  
`--reverse, -r`: Display commit history in the reverse direction.  
`--img-format`: Output format for the image file, i.e. `jpg` or `png`. Default output format is `jpg`.  
`--stdout`: Write raw image data to stdout while suppressing all other program output.  
`--output-only-path`: Only output the path to the generated media file to stdout. Useful for other programs to ingest.  
`--quiet, -q`: Suppress all output except errors.  
`--highlight-commit-messages`: Make commit message text bigger and bold, and hide commit ids.  
`--style`: Graphical style of the output image or animated video, i.e. `clean` (default) or `thick`.

Animation-only global options (to be used in conjunction with `--animate`):

`--video-format`: Output format for the video file, i.e. `mp4` or `webm`. Default output format is `mp4`.  
`--speed=n`: Set the multiple of animation speed of the output simulation, `n` can be an integer or float, default is 1.5.  
`--low-quality`: Render the animation in low quality to speed up creation time, recommended for non-presentation use.  
`--show-intro`: Add an intro sequence with custom logo and title.  
`--show-outro`: Add an outro sequence with custom logo and text.  
`--title=title`: Custom title to display at the beginning of the animation.  
`--logo=logo.png`: The path to a custom logo to use in the animation intro/outro.  
`--outro-top-text`: Custom text to display above the logo during the outro.  
`--outro-bottom-text`: Custom text to display below the logo during the outro.  
`--font`: Font family used to display rendered text.

The `[subcommand options]` are like regular Git options specific to the specified subcommand (see below for a full list).

The following is a list of Git commands that can be simulated and their corresponding options/flags.

### git add

[](https://github.com/initialcommit-com/git-sim#git-add)

Usage: `git-sim add <file 1> <file 2> ... <file n>`

- Specify one or more `<file>` as a _modified_ working directory file, or an untracked file
- Simulated output will show files being moved to the staging area
- Note that simulated output will also show the most recent 5 commits on the active branch

[![git-sim-add_01-05-23_22-07-40](https://user-images.githubusercontent.com/49353917/210940814-7e8dc318-6116-4e56-b415-bc547401a56a.jpg)](https://user-images.githubusercontent.com/49353917/210940814-7e8dc318-6116-4e56-b415-bc547401a56a.jpg)

### git branch

[](https://github.com/initialcommit-com/git-sim#git-branch)

Usage: `git-sim branch <new branch name>`

- Specify `<new branch name>` as the name of the new branch to simulate creation of
- Simulated output will show the newly create branch ref along with most recent 5 commits on the active branch

[![git-sim-branch_01-05-23_22-13-17](https://user-images.githubusercontent.com/49353917/210941509-2a42a7a4-2168-4f62-913f-3f6fe74a0684.jpg)](https://user-images.githubusercontent.com/49353917/210941509-2a42a7a4-2168-4f62-913f-3f6fe74a0684.jpg)

### git checkout

[](https://github.com/initialcommit-com/git-sim#git-checkout)

Usage: `git-sim checkout [-b] <branch>`

- Checks out `<branch>` into the working directory, i.e. moves `HEAD` to the specified `<branch>`
- The `-b` flag creates a new branch with the specified name `<branch>` and checks it out, assuming it doesn't already exist

[![git-sim-checkout_04-09-23_21-46-04](https://user-images.githubusercontent.com/49353917/230827836-e9f23a0e-2576-4716-b2fb-6327d3cf9b22.jpg)](https://user-images.githubusercontent.com/49353917/230827836-e9f23a0e-2576-4716-b2fb-6327d3cf9b22.jpg)

### git cherry-pick

[](https://github.com/initialcommit-com/git-sim#git-cherry-pick)

Usage: `git-sim cherry-pick <commit>`

- Specify `<commit>` as a ref (branch name/tag) or commit ID to cherry-pick onto the active branch
- Supports editing the cherry-picked commit message with: `$ git-sim cherry-pick <commit> -e "Edited commit message"`

[![git-sim-cherry-pick_01-05-23_22-23-08](https://user-images.githubusercontent.com/49353917/210942811-fa5155b1-4c6f-4afc-bea2-d39b4cd594aa.jpg)](https://user-images.githubusercontent.com/49353917/210942811-fa5155b1-4c6f-4afc-bea2-d39b4cd594aa.jpg)

### git clean

[](https://github.com/initialcommit-com/git-sim#git-clean)

Usage: `git-sim clean`

- Simulated output will show untracked files being deleted
- Since this is just a simulation, no need to specify `-i`, `-n`, `-f` as in regular Git
- Note that simulated output will also show the most recent 5 commits on the active branch

[![git-sim-clean_04-09-23_22-05-54](https://user-images.githubusercontent.com/49353917/230830043-779e7230-f439-461a-a408-b19b263e86e4.jpg)](https://user-images.githubusercontent.com/49353917/230830043-779e7230-f439-461a-a408-b19b263e86e4.jpg)

### git clone

[](https://github.com/initialcommit-com/git-sim#git-clone)

Usage: `git-sim clone <url>`

- Clone the remote repo from `<url>` (web URL or filesystem path) to a new folder in the current directory
- Output will report if clone operation is successful and show log of local clone

[![git-sim-clone_04-09-23_21-51-53](https://user-images.githubusercontent.com/49353917/230828521-80c8d2d1-2a31-46bb-aeed-746f0441c86e.jpg)](https://user-images.githubusercontent.com/49353917/230828521-80c8d2d1-2a31-46bb-aeed-746f0441c86e.jpg)

### git commit

[](https://github.com/initialcommit-com/git-sim#git-commit)

Usage: `git-sim commit -m "Commit message"`

- Simulated output will show the new commit added to the tip of the active branch
- Specify a commit message with the `-m` option
- HEAD and the active branch will be moved to the new commit
- Simulated output will show files in the staging area being included in the new commit
- Supports amending the last commit with: `$ git-sim commit --amend -m "Amended commit message"`

[![git-sim-commit_01-05-23_22-10-21](https://user-images.githubusercontent.com/49353917/210941149-d83677a1-3ab7-4880-bc0f-871b1f150087.jpg)](https://user-images.githubusercontent.com/49353917/210941149-d83677a1-3ab7-4880-bc0f-871b1f150087.jpg)

### git config

[](https://github.com/initialcommit-com/git-sim#git-config)

Usage: `git-sim config [--list] <section.option> <value>`

- Simulated output describes the specified configuration change
- Use `--list` or `-l` to display all configuration

[![git-sim-config_04-16-24_08-34-34](https://private-user-images.githubusercontent.com/49353917/322908713-c123e7a7-1fff-4f5c-b4a2-1e34ea2a4d80.jpg?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3NDAxNDQyMjEsIm5iZiI6MTc0MDE0MzkyMSwicGF0aCI6Ii80OTM1MzkxNy8zMjI5MDg3MTMtYzEyM2U3YTctMWZmZi00ZjVjLWI0YTItMWUzNGVhMmE0ZDgwLmpwZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFWQ09EWUxTQTUzUFFLNFpBJTJGMjAyNTAyMjElMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjUwMjIxVDEzMTg0MVomWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPTdjZmQxZjMyNDFhNTI5ZTQ2ZWJkMDIwNjVhMzE5NzE2ODljZmMwNDU0ZjQ0YTZjNjE4N2Y5MjI1ZTk0NTQwMTQmWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0In0.rWuo8e6vxHp8ylzVi4u09hZzBMDISsb_wgZu-0kcTJM)](https://private-user-images.githubusercontent.com/49353917/322908713-c123e7a7-1fff-4f5c-b4a2-1e34ea2a4d80.jpg?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3NDAxNDQyMjEsIm5iZiI6MTc0MDE0MzkyMSwicGF0aCI6Ii80OTM1MzkxNy8zMjI5MDg3MTMtYzEyM2U3YTctMWZmZi00ZjVjLWI0YTItMWUzNGVhMmE0ZDgwLmpwZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFWQ09EWUxTQTUzUFFLNFpBJTJGMjAyNTAyMjElMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjUwMjIxVDEzMTg0MVomWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPTdjZmQxZjMyNDFhNTI5ZTQ2ZWJkMDIwNjVhMzE5NzE2ODljZmMwNDU0ZjQ0YTZjNjE4N2Y5MjI1ZTk0NTQwMTQmWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0In0.rWuo8e6vxHp8ylzVi4u09hZzBMDISsb_wgZu-0kcTJM)

### git fetch

[](https://github.com/initialcommit-com/git-sim#git-fetch)

Usage: `git-sim fetch <remote> <branch>`

- Fetches the specified `<branch>` from the specified `<remote>` to the local repo

[![git-sim-fetch_04-09-23_21-47-59](https://user-images.githubusercontent.com/49353917/230828090-acae8979-4097-43a8-96ea-525890e0e0a8.jpg)](https://user-images.githubusercontent.com/49353917/230828090-acae8979-4097-43a8-96ea-525890e0e0a8.jpg)

### git init

[](https://github.com/initialcommit-com/git-sim#git-init)

Usage: `git-sim init`

- Simulated output describes the initialized `.git/` directory and it's contents

[![git-sim-init_04-16-24_08-34-47](https://private-user-images.githubusercontent.com/49353917/322908611-2abb1a4a-3022-4353-a828-2d337baa8383.jpg?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3NDAxNDQyMjEsIm5iZiI6MTc0MDE0MzkyMSwicGF0aCI6Ii80OTM1MzkxNy8zMjI5MDg2MTEtMmFiYjFhNGEtMzAyMi00MzUzLWE4MjgtMmQzMzdiYWE4MzgzLmpwZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFWQ09EWUxTQTUzUFFLNFpBJTJGMjAyNTAyMjElMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjUwMjIxVDEzMTg0MVomWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPWU1OGE0YmRhYzdmMjkwNjVhZDRiZjk0YzY0NGZjYWFjYzFjYzhmMGRiNzAzYTkxNTYxNDk3ODRjOWZhMzRlNDImWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0In0.nMKPvmgWcJkXGBOlIopF_IeSodXCzTGkW37dD2DbejE)](https://private-user-images.githubusercontent.com/49353917/322908611-2abb1a4a-3022-4353-a828-2d337baa8383.jpg?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3NDAxNDQyMjEsIm5iZiI6MTc0MDE0MzkyMSwicGF0aCI6Ii80OTM1MzkxNy8zMjI5MDg2MTEtMmFiYjFhNGEtMzAyMi00MzUzLWE4MjgtMmQzMzdiYWE4MzgzLmpwZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFWQ09EWUxTQTUzUFFLNFpBJTJGMjAyNTAyMjElMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjUwMjIxVDEzMTg0MVomWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPWU1OGE0YmRhYzdmMjkwNjVhZDRiZjk0YzY0NGZjYWFjYzFjYzhmMGRiNzAzYTkxNTYxNDk3ODRjOWZhMzRlNDImWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0In0.nMKPvmgWcJkXGBOlIopF_IeSodXCzTGkW37dD2DbejE)

### git log

[](https://github.com/initialcommit-com/git-sim#git-log)

Usage: `git-sim log [-n <number>] [--all]`

- Simulated output will show the most recent 5 commits on the active branch by default
- Use `-n <number>` to set number of commits to display from each branch head
- Set `--all` to display all local branches in the log output

[![git-sim-log_01-05-23_22-02-39](https://user-images.githubusercontent.com/49353917/210940300-aadd14c6-72ab-4529-a1be-b494ed5dd4c9.jpg)](https://user-images.githubusercontent.com/49353917/210940300-aadd14c6-72ab-4529-a1be-b494ed5dd4c9.jpg)

### git merge

[](https://github.com/initialcommit-com/git-sim#git-merge)

Usage: `git-sim merge <branch> [-m "Commit message"] [--no-ff]`

- Specify `<branch>` as the branch name to merge into the active branch
- If desired, specify a commit message with the `-m` option
- Simulated output will depict a fast-forward merge if possible
- Otherwise, a three-way merge will be depicted
- To force a merge commit when a fast-forward is possible, use `--no-ff`
- If merge fails due to merge conflicts, the conflicting files are displayed

[![git-sim-merge_01-05-23_09-44-46](https://user-images.githubusercontent.com/49353917/210942030-c7229488-571a-4943-a1f4-c6e4a0c8ccf3.jpg)](https://user-images.githubusercontent.com/49353917/210942030-c7229488-571a-4943-a1f4-c6e4a0c8ccf3.jpg)

### git mv

[](https://github.com/initialcommit-com/git-sim#git-mv)

Usage: `git-sim mv <file> <new file>`

- Specify `<file>` as file to update name/path
- Specify `<new file>` as new name/path of file
- Simulated output will show the name/path of the file being updated
- Note that simulated output will also show the most recent 5 commits on the active branch

[![git-sim-mv_04-09-23_22-05-13](https://user-images.githubusercontent.com/49353917/230829978-0a64dbe2-d974-4cef-9c6e-ed26e987342f.jpg)](https://user-images.githubusercontent.com/49353917/230829978-0a64dbe2-d974-4cef-9c6e-ed26e987342f.jpg)

### git pull

[](https://github.com/initialcommit-com/git-sim#git-pull)

Usage: `git-sim pull [<remote> <branch>]`

- Pulls the specified `<branch>` from the specified `<remote>` to the local repo
- If `<remote>` and `<branch>` are not specified, the active branch is pulled from the default remote
- If merge conflicts occur, they are displayed in a table

[![git-sim-pull_04-09-23_21-50-15](https://user-images.githubusercontent.com/49353917/230828298-455c0a9d-cf94-499e-9e35-623e7b218772.jpg)](https://user-images.githubusercontent.com/49353917/230828298-455c0a9d-cf94-499e-9e35-623e7b218772.jpg)

### git push

[](https://github.com/initialcommit-com/git-sim#git-push)

Usage: `git-sim push [<remote> <branch>]`

- Pushes the specified `<branch>` to the specified `<remote>` and displays the local result
- If `<remote>` and `<branch>` are not specified, the active branch is pushed to the default remote
- If the push fails due to remote changes that don't exist in the local repo, a message is included telling the user to pull first, along with color coding which commits need to be pulled

[![git-sim-push_04-21-23_13-41-57](https://user-images.githubusercontent.com/49353917/233731005-51fd7887-ae14-4ceb-a5d5-e5aed79e9fd8.jpg)](https://user-images.githubusercontent.com/49353917/233731005-51fd7887-ae14-4ceb-a5d5-e5aed79e9fd8.jpg)

### git rebase

[](https://github.com/initialcommit-com/git-sim#git-rebase)

Usage: `git-sim rebase <new-base>`

- Specify `<new-base>` as the branch name to rebase the active branch onto

[![git-sim-rebase_01-05-23_09-53-34](https://user-images.githubusercontent.com/49353917/210942598-4ff8d1e6-464d-48f3-afb9-f46f7ec4828c.jpg)](https://user-images.githubusercontent.com/49353917/210942598-4ff8d1e6-464d-48f3-afb9-f46f7ec4828c.jpg)

### git remote

[](https://github.com/initialcommit-com/git-sim#git-remote)

Usage: `git-sim remote [add|rename|remove|get-url|set-url] [<remote>] [<url>]`

- Simulated output will show remotes being added, renamed, removed, modified as indicated
- Running `git-sim remote` with no options will list all existing remotes and their details

[![git-sim-remote_04-16-24_08-40-37](https://private-user-images.githubusercontent.com/49353917/322910417-ebaff04c-d5b6-4691-97b3-60bb502ba444.jpg?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3NDAxNDQyMjEsIm5iZiI6MTc0MDE0MzkyMSwicGF0aCI6Ii80OTM1MzkxNy8zMjI5MTA0MTctZWJhZmYwNGMtZDViNi00NjkxLTk3YjMtNjBiYjUwMmJhNDQ0LmpwZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFWQ09EWUxTQTUzUFFLNFpBJTJGMjAyNTAyMjElMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjUwMjIxVDEzMTg0MVomWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPWZlMjgwMmFhYzNhMDc3YTZmMWM0NzczZTQ0MGFhNDVkZmU2Yzg0ZTA2ZjEwNDZmODhiMjE3YmU5ZmJiYWQ2MTgmWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0In0.n7hqvLudVNCoFer4bmDXHfcC10byKll4HKVo60TBPWU)](https://private-user-images.githubusercontent.com/49353917/322910417-ebaff04c-d5b6-4691-97b3-60bb502ba444.jpg?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3NDAxNDQyMjEsIm5iZiI6MTc0MDE0MzkyMSwicGF0aCI6Ii80OTM1MzkxNy8zMjI5MTA0MTctZWJhZmYwNGMtZDViNi00NjkxLTk3YjMtNjBiYjUwMmJhNDQ0LmpwZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFWQ09EWUxTQTUzUFFLNFpBJTJGMjAyNTAyMjElMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjUwMjIxVDEzMTg0MVomWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPWZlMjgwMmFhYzNhMDc3YTZmMWM0NzczZTQ0MGFhNDVkZmU2Yzg0ZTA2ZjEwNDZmODhiMjE3YmU5ZmJiYWQ2MTgmWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0In0.n7hqvLudVNCoFer4bmDXHfcC10byKll4HKVo60TBPWU)

### git reset

[](https://github.com/initialcommit-com/git-sim#git-reset)

Usage: `git-sim reset <reset-to> [--mixed|--soft|--hard]`

- Specify `<reset-to>` as any commit id, branch name, tag, or other ref to simulate reset to from the current HEAD (default: `HEAD`)
- As with a normal git reset command, default reset mode is `--mixed`, but can be specified using `--soft`, `--hard`, or `--mixed`
- Simulated output will show branch/HEAD resets and resulting state of the working directory, staging area, and whether any file changes would be deleted by running the actual command

[![git-sim-reset_01-05-23_22-15-49](https://user-images.githubusercontent.com/49353917/210941835-80f032d2-4f06-4032-8dd0-98c8a2569049.jpg)](https://user-images.githubusercontent.com/49353917/210941835-80f032d2-4f06-4032-8dd0-98c8a2569049.jpg)

### git restore

[](https://github.com/initialcommit-com/git-sim#git-restore)

Usage: `git-sim restore <file 1> <file 2> ... <file n>`

- Specify one or more `<file>` as a _modified_ working directory file, or staged file
- Simulated output will show files being moved back to the working directory or discarded changes
- Note that simulated output will also show the most recent 5 commits on the active branch

[![git-sim-restore_01-05-23_22-09-14](https://user-images.githubusercontent.com/49353917/210941009-e6bf7271-ce9b-4e41-9a0b-24cc4b8d3b15.jpg)](https://user-images.githubusercontent.com/49353917/210941009-e6bf7271-ce9b-4e41-9a0b-24cc4b8d3b15.jpg)

### git revert

[](https://github.com/initialcommit-com/git-sim#git-revert)

Usage: `git-sim revert <to-revert>`

- Specify `<to-revert>` as any commit id, branch name, tag, or other ref to simulate revert for
- Simulated output will show the new commit which reverts the changes from `<to-revert>`
- Simulated output will include the next 4 most recent commits on the active branch

[![git-sim-revert_01-05-23_22-16-59](https://user-images.githubusercontent.com/49353917/210941979-6db8b55c-2881-41d8-9e2e-6263b1dece13.jpg)](https://user-images.githubusercontent.com/49353917/210941979-6db8b55c-2881-41d8-9e2e-6263b1dece13.jpg)

### git rm

[](https://github.com/initialcommit-com/git-sim#git-rm)

Usage: `git-sim rm <file 1> <file 2> ... <file n>`

- Specify one or more `<file>` as a _tracked_ file
- Simulated output will show files being removed from Git tracking
- Note that simulated output will also show the most recent 5 commits on the active branch

[![git-sim-rm_04-09-23_22-01-29](https://user-images.githubusercontent.com/49353917/230829899-f5d688ea-bc8e-46f9-a54a-55d251c8915d.jpg)](https://user-images.githubusercontent.com/49353917/230829899-f5d688ea-bc8e-46f9-a54a-55d251c8915d.jpg)

### git stash

[](https://github.com/initialcommit-com/git-sim#git-stash)

Usage: `git-sim stash [push|pop|apply] <file>`

- Specify one or more `<file>` as a _modified_ working directory file, or staged file
- If no `<file>` is specified, all available files will be included
- Simulated output will show files being moved in/out of the Git stash
- Note that simulated output will also show the most recent 5 commits on the active branch

[![git-sim-stash_01-05-23_22-11-18](https://user-images.githubusercontent.com/49353917/210941254-69c80b63-5c06-411a-a36a-1454b2906ee8.jpg)](https://user-images.githubusercontent.com/49353917/210941254-69c80b63-5c06-411a-a36a-1454b2906ee8.jpg)

### git status

[](https://github.com/initialcommit-com/git-sim#git-status)

Usage: `git-sim status`

- Simulated output will show the state of the working directory, staging area, and untracked files
- Note that simulated output will also show the most recent 5 commits on the active branch

[![git-sim-status_01-05-23_22-06-28](https://user-images.githubusercontent.com/49353917/210940685-735665e2-fa12-4043-979c-54c295b13800.jpg)](https://user-images.githubusercontent.com/49353917/210940685-735665e2-fa12-4043-979c-54c295b13800.jpg)

### git switch

[](https://github.com/initialcommit-com/git-sim#git-switch)

Usage: `git-sim switch [-c] <branch>`

- Switches the checked-out branch to `<branch>`, i.e. moves `HEAD` to the specified `<branch>`
- The `-c` flag creates a new branch with the specified name `<branch>` and switches to it, assuming it doesn't already exist

[![git-sim-switch_04-09-23_21-42-43](https://user-images.githubusercontent.com/49353917/230827783-a8740ace-b66f-4cac-b94e-5d101d27e0b5.jpg)](https://user-images.githubusercontent.com/49353917/230827783-a8740ace-b66f-4cac-b94e-5d101d27e0b5.jpg)

### git tag

[](https://github.com/initialcommit-com/git-sim#git-tag)

Usage: `git-sim tag <new tag name>`

- Specify `<new tag name>` as the name of the new tag to simulate creation of
- Simulated output will show the newly create tag ref along with most recent 5 commits on the active branch

[![git-sim-tag_01-05-23_22-14-18](https://user-images.githubusercontent.com/49353917/210941647-79376ff7-2941-42b3-964a-b1d3a404a4fe.jpg)](https://user-images.githubusercontent.com/49353917/210941647-79376ff7-2941-42b3-964a-b1d3a404a4fe.jpg)

## Video animation examples

[](https://github.com/initialcommit-com/git-sim#video-animation-examples)

$ git-sim --animate reset HEAD^

GitSim.reset.mp4

$ git checkout main
$ git-sim --animate merge dev

GitSim.merge.mp4

$ git checkout dev
$ git-sim --animate rebase main

GitSim.rebase.mp4

$ git checkout main
$ git-sim --animate cherry-pick dev

GitSim.cherry-pick.mp4

## Basic command examples

[](https://github.com/initialcommit-com/git-sim#basic-command-examples)

Simulate the output of the git log command:

$ cd path/to/git/repo
$ git-sim log

Simulate the output of the git status command:

Simulate adding a file to the Git staging area:

$ git-sim add filename.ext

Simulate restoring a file from the Git staging area:

$ git-sim restore filename.ext

Simulate creating a new commit based on currently staged changes:

$ git-sim commit -m "Commit message"

Simulate stashing all working directory and staged changes:

Simulate creating a new Git branch:

$ git-sim branch new-branch-name

Simulate creating a new Git tag:

$ git-sim tag new-tag-name

Simulate a hard reset of the current branch HEAD to the previous commit:

$ git-sim reset HEAD^ --hard

Simulate reverting the changes in an older commit:

Simulate merging a branch into the active branch:

Simulate rebasing the active branch onto a new base:

Simulate cherry-picking a commit from another branch onto the active branch:

$ git-sim cherry-pick 0ae641

## Command examples with extra options/flags

[](https://github.com/initialcommit-com/git-sim#command-examples-with-extra-optionsflags)

Use light mode for white background and black text, instead of the default black background with white text:

$ git-sim --light-mode status

Animate the simulated output as a .mp4 video file:

$ git-sim --animate add filename.ext

Add an intro and outro with custom text and logo (must include `--animate`):

$ git-sim --animate --show-intro --show-outro --outro-top-text="My Git Repo" --outro-bottom-text="Thanks for watching!" --logo=path/to/logo.png status

Customize the output image/video directory location:

$ git-sim --media-dir=path/to/output status

Optionally, set the environment variable `git_sim_media_dir` to set a global default media directory, to be used if no `--media-dir` is provided. Simulated output images/videos will be placed in this location, in subfolders named with the corresponding repo's name.

$ export git_sim_media_dir=path/to/media/directory
$ git-sim status

Note: `--media-dir` takes precedence over the environment variable. If you set the environment variable and still provide the argument, you'll find the media in the path provided by `--media-dir`.

Generate output video in low quality to speed up rendering time (useful for repeated testing, must include `--animate`):

$ git-sim --animate --low-quality status

## Installation

[](https://github.com/initialcommit-com/git-sim#installation)

See **Quickstart** section for details on installing manim and other dependencies. Then run:

## Docker installation

[](https://github.com/initialcommit-com/git-sim#docker-installation)

1. Clone down the git-sim repository:

$ git clone https://github.com/initialcommit-com/git-sim.git

2. Browse into the `git-sim` folder and build the Docker image:

$ docker build -t git-sim .

3. Run git-sim commands as follows:
    - Windows: `docker run --rm -v %cd%:/usr/src/git-sim git-sim [global options] <subcommand> [subcommand options]`
    - MacOS / Linux: `docker run --rm -v $(pwd):/usr/src/git-sim git-sim [global options] <subcommand> [subcommand options]`

Optional: On MacOS / Linux / or GitBash in Windows, create an alias for the long docker command so your can run it as a normal `git-sim` command. To do so add the following line to your `.bashrc` or equivalent, then restart your terminal:

git-sim() { docker run --rm -v $(pwd):/usr/src/git-sim git-sim "$@"; }

This will enable you to run git-sim subcommands as [described above](https://github.com/initialcommit-com/git-sim#commands).

## Learn More

[](https://github.com/initialcommit-com/git-sim#learn-more)

Learn more about this tool on the [git-sim project page](https://initialcommit.com/tools/git-sim).

## Authors

[](https://github.com/initialcommit-com/git-sim#authors)

**Jacob Stopak** - on behalf of [Initial Commit](https://initialcommit.com/)