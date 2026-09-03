---
tags:
  - git
  - tag
---
Git tag immutable
steady state, push a git tag

How are tags immutable? They can be deleted and added again to point at a different commit id.
Setting aside the distributed nature of git, if I delete a tag and add it again to point at a different commit id, how can the tag be considered immutable? If I step away from my desk and come back and look at a commit id, I know it is exactly the same thing as before. I can’t say the same thing about a tag.

The v0.0.1 is just a label for the tag object itself.

E.g. if you clone git.git you can see a tag for v2.16.2, the latest release. That you can re-create point to something entirely different.

But if you do:

```
    $ git rev-parse v2.16.2
    86aabcca24951ccfb9392014c8a379992434a7df
    $ git rev-parse v2.16.2^{commit}
    ffa952497288d29d94b16675c6789ef83850def3
```

You can see that the annotated tag object (GPG signature and all) is really called 86aabcca24951ccfb9392014c8a379992434a7df, and that it points to a commit ffa952497288d29d94b16675c6789ef83850def3.

That 86aabcca... is the immutable part of tags, just as commit objects are immutable.

Note that this only applies to annotated tags, lightweight tags just point to the commit object themselves, so they're really just a sort of branch name that git treats differently in not ever advancing it to another commit.

Annotated tags, however, are stored as full objects in the Git database. They're checksummed; contain the tagger name, email, and date; have a tagging message; and **can be signed and verified with GNU Privacy Guard (GPG)**.

the commit hash doesn't verify who made the commit. Someone else may add a commit to your repo with your name. But if you signed with your key that only you have then anyone else can verify it with your public key that you sent them before.

An annotated tag is made by `git tag -a` and then entering the message when prompted or `git tag -m 'message'`.  A signed tag is made with `git tag -s` and optionally the `-a` or `-m` to include a message
  

In both cases the structure is the same, but the signed one has the gpg signature appended on the message field.

  

See the following sequence of commands for example:

[source](https://git-scm.com/docs/git-rev-parse)
If you want to make sure that the output actually names an object in your object database and/or can be used as a specific type of object you require, you can add the `^{type}` peeling operator to the parameter. For example, `git` `rev-parse` `"$VAR^{commit}"` will make sure `$VAR` names an existing object that is a commit-ish (i.e. a commit, or an annotated tag that points at a commit). To make sure that `$VAR` names an existing object of any type, `git` `rev-parse` `"$VAR^{object}"` can be used.


This experience highlighted an important point — not all Git tags are created equal. Depending on the type of tag used, you may or may not get access to helpful metadata such as:

- Who created the tag
- When it was created
- What the purpose or message behind it was
- Whether the tag is verifiable (i.e., cryptographically signed)

This metadata can be crucial when you’re working on a team, managing multiple environments, or trying to trace the history of a release. But **lightweight tags**? They give you none of that. They’re just simple pointers to a commit — quick and easy to create, but lacking context.

Types of Git Tags Git supports three main types of tags:

- Lightweight Tags — the simplest kind, essentially just a label for a commit.
- Annotated Tags — enriched with metadata like author, date, and a custom message.
- Signed Tags — like annotated tags, but with an added cryptographic signature for authenticity.

Let’s take a closer look at each of them and when to use which.

**Lightweight Tags:** A **lightweight tag** is the most basic type of Git tag. It’s essentially just a **named pointer** to a specific commit nothing more then that just a plain bookmark.

git tag  v1.1

Above command create a git l**ightweight** tag which just point to the commit without any metadata.

Press enter or click to view image in full size

LightWeight Tag

As we can see above, the tag simply points to a commit that was made way back on **June 20th**, even though the tag itself was created today for this demo.

**Annotated Tag:**This is a better way of tagging then lightweight tag.An annotated tag is a full Git object that stores rich metadata along with the tag. This includes:

- The tagger’s name and email
- The date the tag was created
- A message describing the tag.

git tag -a v1.0 -m "Release Final Version"

Above command which has a message as well creates an annotated tag.

Press enter or click to view image in full size

Annotated Tag

As shown in the example above, this tag contains **much more information** — including **when** the tag was created, **who** created it, and the **message** describing the purpose of the tag.

**Signed Tag:**This is a better version of annotated tag**.I**t includes a cryptographic signature. This signature verifies that the tag was created by you, and that it hasn’t been tampered with.

git tag -s v1.2 -m "Release Final Version with signed "

Above command creates a signed tag with message and signed by the developer who is relasing the version .

Press enter or click to view image in full size

Signed Tag

At first glance, a **signed tag** may look almost identical to an **annotated tag** — same message, same tagger info, same creation date. So you might wonder: **_what’s actually different here?_**

git tag -v v1.2  
object 1406f4f7b28223305f42f6531fcd49a159f27e92  
type commit  
tag v1.2  
tagger Rajesh Kumar  Dash <dashrajesh49@gmail.com> 1753469617 +0530  
  
Release Final Version with signed tag  
gpg: Signature made Sat Jul 26 00:23:37 2025 IST  
gpg:                using RSA key somekey  
gpg: Good signature from "rajesh (signed rsa) <dashrajesh49@gmail.com>" [ultimate]

So, what sets it apart is the **additional layer of information**: we can see **who signed** the tag, **when the signature was made**, and even details like the **key ID** and **email address** of the signer.

This makes it easy to **verify the authenticity** of the tag and trace it back to a trusted source.

After going through all this, you might be wondering: “**If signed tags offer the most features and security, why not just use them all the time?**”

- Use light weight tag when we need a quick reference to a commit for personal or temporary use.Avoid using lightweight tags for production or release workflows — they don’t carry any context or traceability
- Use Annoated tag when you need a tag that’s **traceable** normally preffered in production release; I use annotated tags  for all official releases and major milestones.
- Use signed tags when you’re publishing to an open-source project or distributing binaries. The **cryptographic signature ensures that others can verify the authenticity of your release**, **preventing anyone from mirroring your repository and pointing the same release tag to a malicious or broken commit.


**`git push --follow-tags`**
- [git push --follow-tags](https://stackoverflow.com/a/26438076/895245) will only push annotated tags
This is a sane option introduced in Git 1.8.3:

```
git push --follow-tags
```

It pushes both commits and only tags that are both:

- annotated
- reachable (an ancestor) from the pushed commits

This is sane because:

- you should only push annotated tags to the remote, and keep lightweight tags for local development to avoid tag clashes. See also: [What is the difference between an annotated and unannotated tag?](https://stackoverflow.com/questions/11514075)
- it won't push annotated tags on unrelated branches

It is for those reasons that `--tags` should be avoided.

Git 2.4 [has added](https://github.com/git/git/commit/61ca378275e83c48343c74a849ff0dcdef9abc91) the `push.followTags` option to turn that flag on by default which you can set with:

```
git config --global push.followTags true
```

or by adding `followTags = true` to the `[push]` section of your `~/.gitconfig` file.