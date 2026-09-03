---
link: https://blog.gitbutler.com/signing-commits-in-git-explained
byline: Scott Chacon
site: GitButler
date: 2023-09-25T06:37
updated: 2023-09-25T06:37
excerpt: Why would you sign your commits in Git? How would you do it? How is it even done? Let's dig into signing commits in Git.
twitter: https://twitter.com/@gitbutler
tags:
  - git
slurped: 2026-08-24T07:44:00
title: Signing Commits in Git, Explained
---

I have a confession to make. I have written with authority about something I knew nothing about.

In Pro Git, there is a section on [signing your work](https://git-scm.com/book/en/v2/Git-Tools-Signing-Your-Work) that I wrote, explaining how to use GPG keys to sign commits in order to let others verify that it was actually you rather than just someone who ran:

```
$ git config author.email schacon@gmail.com
$ git config author.name Scott Chacon
$ git commit -m "LOL, I'm totes Scott!"
```

I have to admit that I think parts of that Pro Git section, now that I re-read it, may not even work properly.  I think the bang in the `user.signingKey` entry is not how you're actually supposed to do that. Embarrassingly, however, this page is one of the top search results from Google when you search for “git commit signing”. I may have misled a generation.

But the concept is cool.

At GitHub, we were never really majorly concerned with spoofing because you can’t push to a repository that you don’t have access to, so authentication and trust was more based on where the code lived than who was listed as the author.

However, it can be easily confusing and in 2012 (4 years after GitHub launched, as reference), Git got support for signing commits with a verifiable GPG key so you could run `git log --show-signature` and it would try to verify signatures with your local GPG toolchain.

When Ben and I wrote the second edition of [Pro Git](https://git-scm.com/book/en/v2) in 2014, this was still a relatively new and not widely used feature. GitHub didn’t support verifying commits for years, so not a massive number of people really used the feature in the wild. Setting up a GPG keychain and using it wasn’t something that was (or is) very simple to do, so it was (and is) pretty rare to see.

When [GitHub started supporting verifying GPG signatures](https://github.blog/2016-04-05-gpg-signature-verification/) in 2016, signing commits became more widespread, even evolving into companies requiring verified commits from employees contributing to corporate codebases. Centralizing this verification so that everyone working in the codebase didn’t have to setup and maintain GPG keyrings of everyone else made things way more accessible.

However, it still meant that everyone had to generate GPG keys and the toolchain to sign commits, which not all clients supported either, especially if they use libgit2 bindings rather than shelling out to the Git binary for committing.

So while concept was cool, most normal developers didn’t have a GPG toolchain up and running and most never saw the complexity tradeoff as worth investing in unless they were forced to do it by their employers.

On the other hand, most developers using Git in any fashion already had SSH keys setup. While some people use HTTPS for Git pushes and fetches, a lot already rely on the (arguably better, but that’s another blog post) SSH authentication mechanism, meaning they have an existing SSH key. But if we’re just verifying signatures of data using public/private key encryption and we’re already using SSH heavily, [why not use SSH instead of GPG to accomplish the same thing](https://www.agwa.name/blog/post/ssh_signatures)?

Well, it turns out that in 2019, OpenSSH 8.0 added support in `ssh-keygen` to sign and verify signatures on arbitrary data. This led to adding support for [SSH commit signing to Git 2.34](https://github.blog/2021-11-15-highlights-from-git-2-34/) in 2021. Now you can use SSH to do both authentication and signature verification without having to introduce a whole new toolchain.

And it turns out that it’s actually pretty easy to do. It can seem confusing, because now there are tons of ways to set up signing. There is GPG, there is S/MIME which we didn’t talk about, and now SSH, but there are different ways to get these keys, different ways to authenticate them, different verification options, etc.

If you want a quick start, what worked for me locally was running these two lines:

```
$ git config gpg.format ssh
$ git config user.signingKey ~/.ssh/id_rsa.pub
```

Now Git knows to use SSH to sign your commits and where your key is. Again, there are a hundred ways to configure signing, but this is probably the simplest that will work for most people.

Now when you run `git commit -S` it will add a signature to your commit and if you upload the key you specify on the `user.signingKey` config value to GitHub or GitLab as a signing key (something you could only do [starting about a year ago](https://github.blog/changelog/2022-08-23-ssh-commit-verification-now-supported/) on GitHub and a [little more recently on GitLab](https://gitlab.com/gitlab-org/gitlab/-/issues/343879)), they will see commits with the signature header and will try to verify it using signing keys on the account associated with the committer email address in the commit.

If it can’t be verified but it has a signature header, everyone will see an “Unverified” mark of shame. But if it can be verified, you’ll get a nice green “Verified” mark of awesomeness.



So now that you know a bit about the history of this feature and how to get it to work for you in a simple way, maybe you’re all set and you can stop reading. However, if you are interested in how exactly this works under the hood a bit, I’ll keep going.

Essentially I started down this path because one of our Alpha users asked in our Discord if we would support commit signing and I figured “how hard could it be?”.

After googling for it, finding my own article, reading it, getting confused and not getting it to work, I hung my head in shame, slapped myself in the face and got to work on figuring it out for real. The problem is that there isn’t a ton of technical documentation about how this works or how one might implement it, or even what GitHub is really doing in the background when trying to verify it.

I wanted to add this functionality to our GitButler client, which is [libgit2](https://libgit2.org/) based, so how do you sign commits using libgit2? Some libgit2 based clients do it, [some are trying](https://github.com/extrawurst/gitui/pull/1544), but most only support GPG and all of them shell out to GPG commands to get it done, none appear to be native implementations and none appear to support SSH signing.

So I started digging in. This is going to get a bit technical, so hopefully you know something about Git objects and whatnot.

Like a number of things I’ve tackled so far while writing a new Git client, I realized that there is not much technical documentation available and so I had to dig through the Git source itself to see what exactly was going on. It’s clear there is a header added to the commit, but it’s unclear how it’s generated exactly or what exact data it’s signing. Here is what the raw header with a signature looks like, as an example:

```
$ git cat-file -p 661d111
tree 1a26e6f16c6939729396c60b2f7bfcbed0365eca
parent 43beb0f5d5b70c7aad4e8a6dd627ae4ce33a2475
author Scott Chacon <schacon@gmail.com> 1695283752 +0200
committer Scott Chacon <schacon@gmail.com> 1695283752 +0200
gpgsig -----BEGIN SSH SIGNATURE-----
 U1NIU0lHAAAAAQAAADMAAAALc3NoLWVkMjU1MTkAAAAgsqPG/CsBdgJN
 MFhOpl5XyN+JOpmR8AAAADZ2l0AAAAAAAAAAZzaGE1MTIAAABTAAAAC3
 AAAAQJisbJ20L87DLj8bDklWLa6Nq2bD9LcCyCG5G0pxNOveQSeMPLjU
 eKrcYgCNgNv4vkkMJQyww=
 -----END SSH SIGNATURE-----

give branchCount a default
```

So, how to you get SSH to generate this signature data with a personal key?

I had to dig around the Git source code a bit, but it turns out that most of the signing and verifying logic is in a file called `[gpg-interface.c](https://github.com/git/git/blob/bcb6cae2966cc407ca1afc77413b3ef11103c175/gpg-interface.c)`. Even though we’re doing stuff with SSH, for legacy reasons everything is still referred to as "gpg". You’ll notice that the Git config setting is `gpg.format`, even though we’re then specifying that we’re using SSH and not GPG at all. In Git, "gpg" basically means "signing stuff".

So I found that in this file there is a function called `[sign_buffer_ssh()](https://github.com/git/git/blob/bcb6cae2966cc407ca1afc77413b3ef11103c175/gpg-interface.c#L1024-L1119)` that appears to do the actual signing work. In the middle of that is the important bit, which is constructing the command to fork/exec:

C

```
strvec_pushl(&signer.args, use_format->program,
         "-Y", "sign",
         "-n", "git",
         "-f", ssh_signing_key_file,
         NULL);
if (literal_ssh_key)
    strvec_push(&signer.args, "-U");
```

Farther up, the `use_format->program` part is defined as `ssh-keygen`, so basically we’re taking the data to sign, writing it to a tempfile, then running `ssh-keygen -Y sign` on it, something like this (the namespace is the string "git", which is what the `-n` is for):

```
$ ssh-keygen -Y sign -n git -f ~/.ssh/id_ed25519.pub -U Cargo.toml
Signing file Cargo.toml
Write signature to Cargo.toml.sig

$ cat Cargo.toml.sig
-----BEGIN SSH SIGNATURE-----
blahblahblah
-----END SSH SIGNATURE-----
```

Then Git reads this `.sig` file back in and injects it as a header to the commit data under the header name `gpgsig`.

Interestingly, Git headers generally don’t deal with newlines, so this header has a special format where it inserts a space at the beginning of each new line. So the signature has to be reformatted when it’s read back out to remove this extra data and get the correct signature data again.

But the next question is what exactly is it signing?

Well, it’s signing everything in the commit object data that would have been written if it hadn’t been signed. So basically in `[commit.c](https://github.com/git/git/blob/bcb6cae2966cc407ca1afc77413b3ef11103c175/commit.c#L1658)`, Git creates a buffer of the data that would have been written out as the commit object and if it’s supposed to sign it, it signs that data and then injects a new header into the middle of it.

Real quickly, I want to cover how to do this in `libgit2`, since it was what I was originally searching for and could not find.

We're using the [git2](https://docs.rs/git2/latest/git2/) Rust bindings, so for us writing a signed commit means generating a commit how you normally would but instead of running the `[commit()](https://docs.rs/git2/latest/git2/struct.Repository.html#method.commit)` function to write to the odb, you write the commit data as a buffer using `[commit_create_buffer()](https://docs.rs/git2/latest/git2/struct.Repository.html#method.commit_create_buffer)` and then convert that buffer into a string and sign it with an SSH key, which in our case means using `[SshSig.sign](https://docs.rs/ssh-key/0.6.1/ssh_key/struct.SshSig.html#method.sign)` in the `[ssh_key](https://docs.rs/ssh-key/0.6.1/ssh_key/)` crate:

RUST

```
 let sig = SshSig::sign(&key, "git", HashAlg::Sha512, buffer)?;
 sig.to_pem(LineEnding::default())
```

Then that signature PEM is added as a header to the final commit using the git2 `[commit_signed](https://docs.rs/git2/latest/git2/struct.Repository.html#method.commit_signed)` function, which writes to the Git odb and returns your `Oid`.

Anywho...

OK, so now we have a commit with an SSH generated signature header in it, signing all of the data in the commit _except itself_. Then you push that commit to GitHub and someone looks at it in the web UI. What exactly happens then?

Well, I don’t work at GitHub anymore, so I don’t know for sure, but I can guess what logically needs to happen at a high level.

First of all, GitHub would need to parse the commit object. It needs to do this anyways when displaying a list of commits or a single commit so that it can extract all the rest of the information: the author, the date, the commit message, etc. But if it sees a `gpgsig` header, that means that there is signature data in it, so it will need to try to verify the data.

For this it needs three things:

- the signature
- the data to verify
- the public signing key

**Getting the Signature**

As mentioned, to get the signature, we need to pull out the `gpgsig` data and remove spaces after each newline in the data to get the original signature back. Pretty straightforward.

**Getting the Data**

To get the data, we need to take all of the commit data and remove the `gpgsig` header to get the original buffer that we signed. In Git, this is done by the [parse_buffer_signed_by_header](https://github.com/git/git/blob/bcb6cae2966cc407ca1afc77413b3ef11103c175/commit.c#L1158) function in `commit.c`, if you want to see exactly the logic, but it’s also fairly straightforward - just remove the one header.

**Getting the Public Signing Key**

So the last thing we need is a public key that we trust belongs to that user in order to verify the signed data.

Now, locally this is rather complicated as you need to specify an `allowed_signers` file and have it formatted properly with everyone you want to verify signed commits for. This file isn’t easily shared or merged, I honestly don’t know how this is meant to be done except for very small, very technical teams. GitLab has a [nice simple document](https://docs.gitlab.com/ee/user/project/repository/signed_commits/ssh.html#verify-commits-locally) on how to do it, if you’re curious.

However, GitHub or GitLab, as a centralized server with clear user profiles and trusted user authentication, has no such complications. If an authenticated user uploads a public SSH or GPG key and has a verified email address, then GitHub can trust that this user (and thus this email address) can be verified by the public signing keys they uploaded. At this point GitHub becomes one massive `allowed_signers` file, arguably with much better security and trust then the one on your computer. So now we have this big secure central keyring that also happens to be where your commits are.

Very importantly, it’s not actually the `author` header of the commit data with which we use to look up the user to try signing keys from for verification. It’s actually the `committer` header. If we go back to our raw commit data example, we can see that in this case they’re exactly the same:

```
$ git cat-file -p 661d111
tree 1a26e6f16c6939729396c60b2f7bfcbed0365eca
parent 43beb0f5d5b70c7aad4e8a6dd627ae4ce33a2475
author Scott Chacon <schacon@gmail.com> 1695283752 +0200
committer Scott Chacon <schacon@gmail.com> 1695283752 +0200
gpgsig -----BEGIN SSH SIGNATURE-----
 U1NIU0lHAAAAAQAAADMAAAALc3NoLWVkMjU1MTkAAAAgsqPG/CsBdgJN
 MFhOpl5XyN+JOpmR8AAAADZ2l0AAAAAAAAAAZzaGE1MTIAAABTAAAAC3
 AAAAQJisbJ20L87DLj8bDklWLa6Nq2bD9LcCyCG5G0pxNOveQSeMPLjU
 eKrcYgCNgNv4vkkMJQyww=
 -----END SSH SIGNATURE-----

give branchCount a default
```

Most of the time this is true. These fields are only different if a commit is rebased or cherry picked or created by an agent, such as hitting the GitHub Merge Button™.

In fact, that’s sort of an interesting use case, so let’s look at that. Here is an example of raw merge commit data made by hitting the GitHub Merge Button™:

```
$ git cat-file -p e282f22e6d5fcd11ed656796b63cfb72e7c1d12e
tree a13329316515bf131de92274b6a6b3b7b62d15c8
parent 1f7796930fdc1aff30698b04343f294d068adfc0
parent 9c2534fd447918daa949e560ce2f957b5cc10b10
author Nikita Galaiko <nikita@galaiko.rocks> 1695283814 +0200
committer GitHub <noreply@github.com> 1695283814 +0200
gpgsig -----BEGIN PGP SIGNATURE-----

 wsBcBAABCAAQBQJlC/pmCRBK7hj4Ov3rIwAA83MIAKH0g6NNhVt3JjAHz4qo5jB2
 mvYGp/UzChRW3MmIESPJEXC+ssY1GddsmJlzXYL/QdBzS9+Yht17jluPMzztuCoW
 NrzW/Z8CtjvsP7Wmkx3YpEsZp20ORZv+4jjSCgLMYRmDtXhf410X9WOF0sOaA3fA
 QoicRqDTleZeJRMsWrSyKt02dIoyjZJluUKerXMp0ApqOkWGUcxVcgUR58SEC9CB
 hFK8ipBXd+gCwZzvl2OGKzCecPPE41moz+lU6FA6zsIS2pUEMeZGsRrpos0AfUGe
 waPQygqqGd84RyGRKEuwuRwoL56mLmO5uwraebGNUeKJeIxLSJj1OqrqlHQrikI=
 =JxuS
 -----END PGP SIGNATURE-----
```

So here we can see a few interesting things.

One is that there are two parents, as we would expect from a merge commit. The next is that the `gpgsig` field is in this case a PGP signature, not an SSH one. It’s also a bit interesting that GitHub does this signing automatically with merges and commits that it creates. We’ll return to that in a second. The third interesting thing is that the `author` and `committer` fields are different.

We didn’t rebase, but since we hit a Merge Button™, GitHub wrote the actual commit data, and in theory it verified that GitHub in fact wrote the commit and signed that work with a PGP key that can be independently verified. Which means that it can be safely inferred that Nikita hit that button because that’s how GitHub works - it wouldn't write his email address in there if he was not logged in and thus authenticated. It's actually pretty cool if you think about it.

Additionally, you can get GitHub’s PGP public key here: [https://github.com/web-flow.gpg](https://github.com/web-flow.gpg), which means you could use it to locally verify commits that were done on GitHub via `gpg` by importing this public key into your keyring like this:

```
$ curl https://github.com/web-flow.gpg | gpg --import
```

But I digress.

As the Germans say, “_Vertrauen ist gut, kontrolle ist besser._” Actually, I think it comes from something Lenin originally said, so maybe this was a bad example for me to have pulled up. Ignore that. But, let’s _kontrolle_ nonetheless.

Now we have everything we need: the data to verify, the signature that signed it, and the public keys to verify the signature. Locally, we would run `ssh-keygen` again with the user’s public key and the data we want to check. In Git, this is how that command is put together in [`verify_ssh_signed_buffer()`](https://github.com/git/git/blob/bcb6cae2966cc407ca1afc77413b3ef11103c175/gpg-interface.c#L557-L563) in `gpg-interface.c`:

C

```
 strvec_pushl(&ssh_keygen.args, "-Y", "verify",
                 "-n", "git",
                 "-f", ssh_allowed_signers,
                 "-I", principal,
                 "-s", buffer_file->filename.buf,
                 verify_time.buf,
                 NULL);
```

So it’s essentially a call to `ssh-keygen -Y verify` with the principle found in your `allowed_signers` file.

Presumably GitHub does something like this. I don’t know exactly how they do it since they are obviously not using an actual `allowed_signers` file, but there are a bunch of ways to replicate this functionality.

For example, we’re building GitButler in [Tauri](https://tauri.app/), which means all the backend stuff is in Rust, so we would be able to easily use the `[ssh_key::public::PublicKey::verify](https://docs.rs/ssh-key/0.6.1/ssh_key/public/struct.PublicKey.html#method.verify)` function of the `ssh-key` crate we’re already using for other things to be able to verify this data without shelling out to `ssh-keygen`.

So, to sum up, it's pretty cool to use SSH to sign your commits. It's not hard to do and GitHub/GitLab make the verification part really simple.

In fact,  you can set your GitHub account to be in "[vigilant mode](https://docs.github.com/en/authentication/managing-commit-signature-verification/displaying-verification-statuses-for-all-of-your-commits)" which I have to say is the coolest phrase the Hubbers have come up with since "Danger Zone". Once this mode is turned on, all commits that are not explicitly signed by one of your signing keys (SSH or PGP) and verified are explicitly marked as "Unverified".

Thanks for reading and I hope if you're trying to implement signing at a low level and stumbled across this article, this was useful to you.

## More on SSH Signing and GitHub

If you’re interested in more information around SSH signing and GitHub specifically, I can recommend this talk by Andy Feller at last years Git Merge.

## Commit Signing and GitButler

If you’re part of our GitButler Alpha, then you can start using commit signing with about 30 seconds of work by following our [Verifying Commits](https://docs.gitbutler.com/features/virtual-branches/verifying-commits) guide. If not, you can join our waiting list [here](https://n9tnn5agl93.typeform.com/to/i6aDKuNR?typeform-source=gitbutler.com).

[![Scott Chacon](https://blog.gitbutler.com/_next/image?url=%2Fprofiles%2Fschacon.jpg&w=256&q=75)](https://blog.gitbutler.com/author/schacon)

Written by [Scott Chacon](https://blog.gitbutler.com/author/schacon)

Scott Chacon is a co-founder of GitHub and GitButler, where he builds innovative tools for modern version control. He has authored Pro Git and spoken globally on Git and software collaboration.

### Table _of_ Contents