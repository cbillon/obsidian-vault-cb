---
link: https://tylercipriani.com/blog/2022/05/14/git-diffs-deep-deception/
byline: Tyler Cipriani
excerpt: |-
  But try to understand Try to understand Try try try to
  understand Git’s a magic command.
slurped: 2026-06-01T08:40
title: Explaining git diff to myself
tags:
  - git
---

> But try to understand  
> Try to understand  
> Try try try to understand  
> Git’s a magic command.
> 
> – [Heart  💕](https://www.youtube.com/watch?v=Ps7tVvQHLyo)

Once upon a time, I believed git was storing diffs somewhere. But then I learned I was wrong.

It’s challenging to wield git’s clunky interface when you have a broken mental model of its internals. Learning more about what’s happening inside git transformed me into a more effective git user.

In this post, I’ll attempt to explain all the deep details of `git diff` to my past self.

## 📍 Git add makes blobs

We can add files to repos using `git add`. But behind the [porcelain](https://git-scm.com/book/en/v2/Git-Internals-Plumbing-and-Porcelain), git’s busy compressing and storing this file deep in its bowels. Git terms the results of this process a “blob.”

Git stores blobs (among other things) inside the `.git/objects` directory.

```
$ git init
Initialized empty Git repository in /tmp/bar/.git/
$ echo "Hi, I'm blob" > foo
$ git add foo
$ tree .git/objects/
.git/objects/
└── 26
  └── 45aab142ef6b135a700d037e75cd9f1f1c94dc
```

But what’s in a blob? And why is this blob stored as `./26/45aab142ef6b135a700d037e75cd9f1f1c94dc`?

## 🗃️ Git stores things by their hash

Why did `git add foo` store the contents of `foo` as `2645aab142ef6b135a700d037e75cd9f1f1c94dc`?

Git mapped our file to a number via a hash function.

**A hash function maps data to a unique number** ([mostly](https://shattered.io/))—whenever the data changes, the hash function’s output changes dramatically.

[SHA1](https://en.wikipedia.org/wiki/SHA-1) is the hash function git uses by default. And when we `git add foo` git applies SHA1 to the contents of `foo`—`Hi, I'm blob\n`—and that spits out `2645aab142ef6b135a700d037e75cd9f1f1c94dc`.

Blobs are all about content. The filename “foo” doesn’t matter at all! We could have named the file “🌈”—git still would have stored it in the same place. If the file contents are EXACTLY the same, then the hash will be exactly the same.

## 🌱 Git commit creates commits and trees

You already know `git commit` creates a commit, but what is a commit?

A commit is a type of object. Git uses the word “object” to mean: a commit, a folder or directory (tree), a file (blob), or a tag. Git stores objects in its object database—everything inside the `.git/objects` directory.

```
$ git commit -m 'Initial Commit'
[main (root-commit) 0644991] Initial Commit
1 file changed, 1 insertion(+)
create mode 100644 foo
$ tree .git/objects/
.git/objects/
├── 06
│   └── 449913ac0e43b73bfbd3141f5643a4db6d47f8
├── 26
│   └── 45aab142ef6b135a700d037e75cd9f1f1c94dc
└── 41
  └── 81320a57137264d436b2ef861c31f430256bf4
```

After our commit, the object database has three objects: `06449913`, `2645aab1`, and `4181320a`.

So now we’ve established that one of these three objects is our blob (`2645aab1`)—let’s see if we can suss out the others.

## ✨ The magic command

The magic command to learn about any object is `git cat-file -p`. We can use that command to find out more about our mystery objects:

```
$ git cat-file -p 06449913ac0e43b73bfbd3141f5643a4db6d47f8
tree 4181320a57137264d436b2ef861c31f430256bf4
author Tyler Cipriani <tcipriani@wikimedia.org> 1652310544 -0600
committer Tyler Cipriani <tcipriani@wikimedia.org> 1652310544 -0600

Initial Commit
```

This object (`06449913`) appears to be our commit. A commit is metadata compressed and stored inside git’s object database.

Some of the metadata is obvious, but then there’s a tree. And that tree points to our other mystery object, `418132`. Let’s see what we can learn about our last remaining mystery object using our magic command:

```
$ git cat-file -p 4181320a57137264d436b2ef861c31f430256bf4
100644 blob 2645aab142ef6b135a700d037e75cd9f1f1c94dc    foo
```

So a **tree is an object that stores a directory listing of objects by their SHA1s**. And a **commit is an object that points at a tree** by recording the tree’s SHA1!

Commits point to trees, and trees point to blobs and other trees. Neat!

## 📈 Git’s dependency graph

So if we graphed the state of dependencies in our object database, we’d get something like this:

![Simple git repo’s object dependency graph](http://photos.tylercipriani.com/2022-05-15_git-merkle-1.png)

The commit incorporates our tree, which includes our blob—everything depends on our blob!

So if we change even a single bit inside a single file: git will notice—everything is entirely traceable from the commit down to the bit level. We get this for free by hashing objects and including those hashes in other objects.

This is the whole concept of a [Merkle Directed Acyclic Graph](https://en.wikipedia.org/wiki/Merkle_tree) (Merkle DAG)!

## 🍔 So, where’s the diff?

When we type `git diff`, git presents us a diff. We know there are blobs and trees and commits—so where’s the diff!?

**Git doesn’t store diffs anywhere at all!** It derives diffs from what’s stored in the object database.

```
$ echo "I'm ALSO blob" > baz
$ git add baz
$ git commit -m 'Add baz'
$ tree .git/objects/
.git/objects/
├── 06
│   └── 449913ac0e43b73bfbd3141f5643a4db6d47f8
├── 26
│   └── 45aab142ef6b135a700d037e75cd9f1f1c94dc
├── 41
│   └── 81320a57137264d436b2ef861c31f430256bf4
├── 95
│   └── 42599fac463c434456c0a16b13e346787f25da
├── 9b
│   └── 2716e4540c11e8d590e906dd8fa5a75904810a
└── e6
   └── 5a7344c46cebe61d052de6e30d33636e1cd0b4
```

We made a new commit, and now we have three new objects. We added a new file (blob), which made our directory different (tree), and we committed it (commit).

Our graph now looks like this:

![Simple git repo’s updated object dependency graph](http://photos.tylercipriani.com/2022-05-15_git-merkle-2.png)

You might be surprised by a few things in the graph:

- Our new commit stores its parent commit as metadata
- Our new tree points to our old blob, and our NEW blob

So now what happens when we try git diff:

```
$ git diff 064499..e65a73
diff --git a/baz b/baz
new file mode 100644
index 0000000..9b2716e
--- /dev/null
+++ b/baz
@@ -0,0 +1 @@
+I'm ALSO blob
```

Git compares the two commits, finds their trees, sees a new blob in the second commit, and shows you the diff of `/dev/null` and `baz`.

No diffs. Just Merkle DAGs. And now you know.

---

Thanks to [Joe Swanson](https://github.com/joescottswanson) for providing excellent early feedback on this post. And thanks to [Kostah Harlan](https://www.kostaharlan.net/) for reading an early draft of this post and making it less terrible. <3