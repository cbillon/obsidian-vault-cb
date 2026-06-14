---
link: https://tylercipriani.com/blog/2024/09/29/subliminal-git-commits/
byline: Tyler Cipriani
excerpt: |-
  We’ve all seen Git commit hashes—the hard-to-remember hexadecimal
  strings that refer to Git commits.
slurped: 2026-06-01T08:17
title: Git commit hashes that spark joy
tags:
  - git
---

We’ve all seen Git commit hashes—the hard-to-remember hexadecimal strings that refer to Git commits.

But it’s rare to see Git commit hashes spark joy:

```
$ git log --oneline
deadbeef (HEAD -> main) Dead beef is beef. Live beef is cow. 🥁
c0ffeeee Late night debugging ☕
defacedd Fix vandalism
facade00 Provide simple interface class
c0deca11 Add function call
```

Creating memorable Git hashes is hard. Git creates hashes with SHA-1 or SHA-2, which are one-way functions—there’s no way to predict a hash.

So, to spread joy with Git history, we need to:

1. Brute-force funny Git commit hashes.
2. Keep Git usable – Git commit messages and metadata should stay usable by humans and machines.
3. Be fast – I need to find commit hashes quickly, without churning forever for a bit of silly fun.

## What can I spell with a git commit?

![O’RLY Insulting SHA-1 Collisions. (Copyright 2024 DenITDao via orlybooks)](https://photos.tylercipriani.com/2024-09-29_insulting-sha1-collisions.jpg)

With hexadecimal we can spell any word containing the set of letters `{A, B, C, D, E, F}`—`DEADBEEF` (a classic) or `ABBABABE` (for _Mama Mia_ aficionados).

This is because hexadecimal is a base-16 numbering system—a single “digit” represents 16 numbers:

```
Base-10: 0  1  2  3  4  5  6  7  8  9 10 11 12 13 16 15
Base-16: 0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f
```

Git hashes use lowercase letters and the numbers 0–9. Some numbers look like lowercase letters, which expands our palette of words—substituting `0` for `O`, `1` for `l`, and `5` for `s`.

I created a [script](https://gist.github.com/thcipriani/8f16f00f1c3f68e96d8c7e85c56f2900) that scours word lists for valid words and phrases.

With my script, I found some masterpieces: `dadb0d` (dad bod), `bada55` (bad ass), `0ff1oad` (offload), `0ddba11` (oddball), and the famous last words: `fee155afe` (feels safe).

## What is a Git commit hash?

Git commit hashes are no mystery.

A Git commit object is a compressed file that looks like:

```
$ < .git/objects/b8/da4201321b5d73297c353a36e09492d6abcdf2 zlib-flate -uncompress | cat -v
commit 195^@tree 66dd8a515582fba80ca1e1137a6763f3759ebd65
author Tyler Cipriani <REDACTED> 1760808267 -0600
committer Tyler Cipriani <REDACTED> 1760812806 -0600

Initial commit
```

Decompressing a Git commit object using `zlib-flate -uncompress` gives you the complete formatted commit object. Commit objects use the format:

```
commit [Length of commit message in bytes]\0[Commit message]`
```

The commit message includes:

- Pointers to other Git objects (parent commits, trees, or other commits)
- Author name, email, author time
- Committer name, email, commit time
- The commit message

Git stores commits as zlib-compressed files under `.git/objects`[1](https://tylercipriani.com/blog/2024/09/29/subliminal-git-commits/#fn1). Each filename is the SHA-1 (or SHA-2) of the object’s decompressed contents:

```
$ < .git/objects/b8/da4201321b5d73297c353a36e09492d6abcdf2 zlib-flate -uncompress | sha1sum
b8da4201321b5d73297c353a36e09492d6abcdf2  -
```

## Recreate a Git commit hash with Bash

Before I can make funny Git commit hashes, I need generate valid ones.

```
$ mkdir /tmp/git-repo && cd /tmp/git-repo
$ git init
Initialized empty Git repository in /tmp/git-repo/.git/
$ echo > README.md && git add README.md && git commit -m 'Initial commit'
[main (root-commit) 68ec0dd] Initial commit
 1 file changed, 1 insertion(+)
  create mode 100644 README.md
$ git log --oneline
68ec0dd (HEAD -> main) Initial commit
```

I created a Git repo with an empty `README.md` and one commit. The commit’s hash is `68ec0dd`.

```
$ git cat-file -p 68ec0dd > commit-msg
$ COMMIT_SIZE_BYTES="$(wc -c < commit-msg)"
$ printf 'commit %d\0' "$COMMIT_SIZE_BYTES" | \
    cat - commit-msg | \
    sha1sum
68ec0dd6dead532f18082b72beeb73bd828ee8fc  -
```

Here I recreate the commit hash `68ec0dd`.

- `git cat-file` – get the commit message
- `wc -c` – get the commit message length in bytes
- `printf 'commit %d\0%s'` – print a formatted Git commit object, complete with the commit message and length
- `sha1sum` – get the hash of the commit object

## Brute-forcing Git commit hashes, naïve attempt

My initial modest goal is:

1. Make a commit with a hash that starts `00`.
2. Leave our code, author, and committer untouched.
3. Keep the commit message the same, at least for humans.

To change the hash, I’ll need to change the hash input—the commit message. Modifying a commit message invisibly to users will be tricky.

Git pretty-prints commit messages when you run `git log`. Pretty printing trims any trailing `isspace()` characters:

- SPACE - `\u0020`
- TAB - `\u0009`

No one will see trailing spaces, but trailing spaces will change the commit hash.

To brute-force a Git commit hash, I’ll tweak the commit message in a loop, adding a random combination of space and tab characters until the commit hash starts with `00`:

```
#!/usr/bin/env bash
# git-hash-fiddler.sh
# ~~~~~~~~~~~~~~~~~~~
git cat-file -p HEAD > commit-msg
CALC_HASH() {
    COMMIT_SIZE_BYTES="$(wc -c < commit-msg)"
    printf 'commit %d\0' "$COMMIT_SIZE_BYTES" | \
        cat - commit-msg | \
        sha1sum | awk '{print $1}'
}
CURRENT_HASH=$(CALC_HASH)
SPACE_CHARS=$'\u0020\u0009'

# Check if the commit hash starts with 00
while [[ ! "$CURRENT_HASH" =~ ^00 ]]; do
    grep -o . <<< "$SPACE_CHARS" | shuf -n1 >> commit-msg
    CURRENT_HASH=$(CALC_HASH)
done

# Add the modified commit to the git object store
< commit-msg git hash-object -w -t commit --stdin

# Update the current commit
git update-ref HEAD "$CURRENT_HASH"
```

First, I get the current commit. Then, in the `while` loop, I append a random space character to the commit and compute the hash until the hash starts with `00`.

Once I find a commit that has a hash starting `00`, I use `git hash-object` to create a new Git commit object. Finally, I call `git update-ref` to point `HEAD` to my new commit hash.

```
$ git log --oneline
68ec0dd (HEAD -> main) Initial commit
$ time bash git-fiddler.sh
00ba5924057452af287a2be151061f973862b811

real    0m2.618s
user    0m3.130s
sys     0m1.430s
$ git log --oneline -1
00ba592 (HEAD -> main) Initial commit  
```

It took about two seconds to brute-force a Git commit hash starting with `00`.

Not bad.

Let’s go big. Let’s try for five leading `0`s:

```
$ time bash git-fiddler.sh
00000a2195aa3d9e85bc5b70e70807e23f2f64fb

real    118m37.577s
user    133m54.613s
sys     72m51.337s
```

Finding a commit hash starting with `00000` took **two hours** with my silly Bash script.

I may need to optimize.

## Brute-forcing Git commit hashes (before the heat death of the universe)

Fortunately, user `not-an-aardvark` created a tool for that—[lucky-commit](https://github.com/not-an-aardvark/lucky-commit).

`lucky_commit` does the same thing as our Bash script; it adds combinations of `TAB` and `SPACE` characters until it finds the Git hash with a prefix you specify.

But, unlike my Bash script, it’s fast:

```
$ time lucky_commit 00000

real    0m0.103s
user    0m0.323s
sys     0m0.037s
$ git log --oneline -1
00000e9 (HEAD -> main) Initial commit        
```

Finding a Git commit hash starting with `00000` takes 100ms (as opposed to **two hours** with my Bash script).

`lucky_commit` speeds up Git commit hashes in two ways:

- **Does less work** – Both SHA-1 and SHA-2 break commit messages in 64-byte chunks and then perform operations on those chunks one at a time. Each chunk affects subsequent chunks. By pre-computing the hash state before appending a fixed-length amount of padding, `lucky_commit` does a fraction of the bit twiddling of our shell script.
- **Uses more CPU** – Adding a fixed-length (48-bit) amount of padding to a commit using only two characters (`TAB` and `SPACE`), you have 248 variations (281 trillion possibilities). And you can split hashing into threads, doing more work in parallel.
- **Moves to the GPU** – If you have a GPU, you can do even more hashing operations in parallel. This is an option in `lucky_commit`.

## Use Git hooks to spread joy

All that’s left to do is to make all my commits `bada55` by abusing Git hooks.

```
$ cat > .git/hooks/post-commit && chmod +x .git/hooks/post-commit
#!/usr/bin/env bash
WORDS=(
    babb1e    # BABBLE
    bada55    # BADASS
    badc0de   # BADCODE: not even sorry.
    c0ffee    # COFFEE
    0ddba11   # ODDBALL
    5adface   # 😿
)
lucky_commit "${WORDS[RANDOM%${#WORDS[*]}]}"
printf '✨ %s ✨\n' "$(git log --one line -1)"
```

With the `post-commit` hook above, after I commit, `lucky_commit` fiddles with my commit message until my commit hash is a random prefix from the list.

Fast, automated Git joy:

```
$ git commit --amend --no-edit
✨ 0ddba116 Initial commit ✨
[main bd256a9] Initial commit
 Date: Mon Oct 20 17:58:32 2025 -0600
 1 file changed, 1 insertion(+)
 create mode 100644 README.md
$ git commit --amend --no-edit
✨ badc0de9 Initial commit ✨
[main facd923] Initial commit
 Date: Mon Oct 20 17:58:32 2025 -0600
 1 file changed, 1 insertion(+)
 create mode 100644 README.md
$ git commit --amend --no-edit
✨ babb1e6 Initial commit ✨
[main 441ec61] Initial commit
 Date: Mon Oct 20 17:58:32 2025 -0600
 1 file changed, 1 insertion(+)
 create mode 100644 README.md
```

---

1. Git also stores compressed objects under `.git/objects/pack`, but most objects start their lives as loose files.[↩︎](https://tylercipriani.com/blog/2024/09/29/subliminal-git-commits/#fnref1)