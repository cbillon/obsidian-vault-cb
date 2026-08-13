
In my many years experience, I've never regretted learning a 'plan B' in case 'plan A' gets disrupted or worse, never materializes.

Petr Skoda
 in the end "selling" GPL3 licensed software is weird, we should be selling support for plugins
 I like when open-source software is available for direct download for free and "sold" in app stores as a form of donation to project.
 You can sell commercial support only if it provides some value to customers - such as easier communication with devs, faster bug fixing, better help/docs, longer support, etc. In case of Marketplace it seems that they are not willing to give sellers any tools that would help with that - they just rely on forcing everything through contracts/lawyers/trademarks.

[Immutability version is foundational](https://blog.packagist.com/immutable-versions-on-packagist/) 
Our next goal for immutability is now to provide immutable artifacts for each version. We intend to accomplish this step in the next months with a combination of hosting artifacts ourselves and publishing info for verification on our transparency log. We still urgently need additional corporate sponsors to push forward with this work: contact us at sponsoring@packagist.org.

We strongly recommend other Composer repository maintainers introduce immutability on their package versions too, to help bring these improvements to the entire PHP ecosystem.

## [**A real deletion model, with reasons**](https://blog.packagist.com/immutable-versions-on-packagist/#a-real-deletion-model-with-reasons)

Immutability raises an obvious question. If a published version can never be rewritten, how do you deal with one that genuinely needs to come down, for example because a release accidentally contained secret information? Until now Packagist.org had exactly one blunt tool: a version could be removed entirely, with no recovery, no record of why, and no way to differentiate between a maintainer tidying up their own package and an administrative takedown.

Alongside immutability we are introducing soft-deletion with explicit reasons. Instead of erasing a version entirely, only a record of the version remains, not the release artifact or code, and the record is marked with the reason the version was taken down:

- **No longer found in the source repository.** When a version disappears from the repository, for example because its tag was deleted, Packagist.org marks it accordingly. Dev versions are permanently removed after a day, as before, since branches come and go. Stable version metadata is stored indefinitely, so a tag that briefly vanishes can be recovered as long as it is unchanged, rather than being permanently lost to immutability.
- **Deleted by maintainer.** A maintainer removed the version through the per-version delete on the package page.
- **Removed by admin.** An administrator took the version down, with an optional public reason shown on the package page, e.g. after manual review of automatically flagged malware releases.
- **Hidden by admin.** An administrative takedown with no public trace, used for spam waves and selective takedowns. The version stays visible to maintainers and admins, and is hidden from everyone else.

Soft-deleted versions are removed from the metadata Composer reads, so Composer can no longer resolve a version that has been pulled, regardless of the reason. On the package page, soft-deleted versions appear grayed out with a per-reason explanation.

![](https://storage.ghost.io/c/c4/73/c4739028-7475-4e92-ab07-856192bf3472/content/images/2026/07/data-src-image-3ab80a88-f6e1-4010-9147-763ce02d9d0e.png)

Where recovery makes sense, the version shows a Recover button. Maintainers can recover their own deletions and versions that went missing from the repository. Versions removed or hidden by an administrator are recoverable by administrators only.

![](https://storage.ghost.io/c/c4/73/c4739028-7475-4e92-ab07-856192bf3472/content/images/2026/07/data-src-image-4d0d8773-c33d-4cb9-919a-94e6b75bd215.png)

## [#**Transparency log**](https://blog.packagist.com/immutable-versions-on-packagist/#transparency-log)

Every one of these actions is recorded on the public transparency log for the package: soft-deletions, recoveries, and blocked reference changes all leave a permanent public entry.. The result is a system where anyone can understand, after the fact, what happened to a version and why, instead of discovering that a version was silently rewritten or deleted.


Packagist does not store any source code. It is a metadata index that points to Git repositories. Nils Adermann describes this in the May 27, 2026 update as a deliberate strength:
>   
> Composer's model of distributing packages straight from git tags keeps the artifact directly tied to its source rather than to a separately-uploaded build.

This is a fundamental architectural decision. Hypothetical full access to the servers of Packagist.org is not, by itself, enough to poison packages. npm and PyPI, which host their artifacts centrally, do not have this natural separation.