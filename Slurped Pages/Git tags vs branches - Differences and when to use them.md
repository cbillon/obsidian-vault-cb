---
link: https://circleci.com/blog/git-tags-vs-branches/
byline: Jacob Schmitt
site: CircleCI
date: 2023-03-08T14:00
excerpt: Learn the differences between Git tags and Git branches, when to use them, and how they can provide more efficiency and control in your CI/CD pipeline.
twitter: https://twitter.com/@
slurped: 2026-01-06T10:17
title: "Git tags vs branches: Differences and when to use them"
tags:
  - git
---

## This article covers:

- [What is a Git tag?](https://circleci.com/blog/git-tags-vs-branches/#what-is-a-git-tag)
- [What is a Git branch?](https://circleci.com/blog/git-tags-vs-branches/#what-is-a-git-branch)
- [Differences between Git tags and branches](https://circleci.com/blog/git-tags-vs-branches/#differences-between-git-tags-and-branches)
- [Using Git tags and branches in a CI/CD pipeline](https://circleci.com/blog/git-tags-vs-branches/#using-git-tags-and-branches-in-a-cicd-pipeline)
- [Using Git tags and branches with CircleCI](https://circleci.com/blog/git-tags-vs-branches/#using-git-tags-and-branches-with-circleci)
- [Branch and tag filters](https://circleci.com/blog/git-tags-vs-branches/#branch-and-tag-filters)
- [Monitor tagged workflows](https://circleci.com/blog/git-tags-vs-branches/#monitor-tagged-workflows)
- [Conclusion](https://circleci.com/blog/git-tags-vs-branches/#conclusion)

Tags and branches are two key Git concepts that allow developers to work on different versions of a project simultaneously. Both play an important role in organizing development work so that changes are easier to track and manage. Each serves a different purpose in the development process.

- Tags are markers for significant points in the project timeline
- Branches are separate lines of development that will eventually be merged back into the main application code

Git is one of the most [widely used](https://survey.stackoverflow.co/2022/#version-control-version-control-system-prof) version control systems. When paired with a continuous integration and delivery (CI/CD) pipeline, Git allows development teams to rapidly iterate and deliver new features while keeping software stable and deployable.

In this article, you’ll learn the differences and similarities between Git tags and branch development and how both can keep your development team more organized and efficient.

## What is a Git tag?

A [tag](https://book.git-scm.com/docs/git-tag) is an object referencing a specific commit within the project history, similar to chapter markers in a book. You can create tags to point to a new version release, a significant codebase change, or any other event development teams may want to reference. Many development teams use [semantic versioning](https://semver.org/) to communicate the types of changes that occur between one tag and another.

Read more: [The Path to Platform Engineering](https://circleci.com/resources/platform-engineering-ebook/)

Once a tag has been created, no further commits can be added to it, but you can reference it easily using the tag name. This allows you to switch to the tagged version of the codebase to see what the code looked like when the tag was created and, if necessary, revert to that specific codebase version.

### Working with the git tag command

The `git tag` command allows you to create, list, delete, and verify tags. Here’s how you can use the `git tag` command to specify a new version of your application code:

First, create a new tag called `v1.0.0` for the latest commit in your repository. Run this command:

```
git tag v1.0.0
```

List all the tags in your repository by running this:

```
git tag
```

The `git tag` command can also verify a specific tag. The next command verifies the `v1.0` tag and displays the details, including the tagger, date, and message.

```
git tag -v v1.0.0
```

Finally, the `git tag` command can delete tags. To delete the `v1.0` tag, run:

```
git tag -d v1.0.0
```

Deleting a tag does not affect the code in your repository — it just removes the tag label.

      ![](https://images.ctfassets.net/il1yandlcjgk/4FebFXsAEK4BPmyg0NWOYa/6d7c80084a6f781ba20cc50175cc7696/2024-07-10-confident-commit-newsletter-logo.png?w=511&fm=webp&q=60)

Actionable insights from 15 million+ datapoints

## What is a Git branch?

A [branch](https://book.git-scm.com/docs/gitglossary) is an independent development line containing a pointer — or the branch head — to the most recent commit in the code.

A single Git repository can track many branches, but your working tree centers on just one. The head points to the tip (the latest commit) of a branch.

A branch is automatically generated when you create a repository for your code. This default branch is called `main`. Writing code and committing your work automatically records a commit in the working branch.

### Working with the git branch command

The `git branch` command lets you create, list, rename, and delete branches. You cannot use it to switch between branches.

Here is an example of how to use `git branch`.

First, create a new branch called `dev`. Run:

```
git branch dev
```

Next, switch to the development branch. Run:

```
git switch dev
```

Now, make changes to your code and commit them to the dev branch using the usual `git add` and `git commit` commands.

To merge your changes from the dev branch into the main branch, run:

```
git merge dev
```

Using the `git branch` command displays a list of all the branches in your repository, with an asterisk next to the branch you are currently on:

```
git branch
```

You can also use the `git branch` command to rename a branch. To rename the dev branch to development, run:

```
git branch -m dev development
```

Finally, the `git branch` command can delete a branch. To delete the development branch, run:

```
git branch -d development
```

You cannot delete a branch you are currently on. You must switch to a different branch first.

Tags and branches are both used for version control of your code base. Their functions complement each other, and they are designed to be used together.

A branch is often used for new features and fixing bugs. It allows you to work without impacting the main codebase. Once your work is complete, you can incorporate your changes into the application by merging the branch back into the main codebase. This allows multiple people to work on different aspects of the project simultaneously. It also provides a way to experiment with new ideas without risking the main codebase’s stability.

Unlike branches, tags are not intended for ongoing development. Tags mark a specific point in the repository’s history to give developers an easy way to reference important milestones in the development timeline.

### When to use branches

Imagine that you want to add a new feature to a software project’s codebase but you aren’t sure if the new feature will work as expected. You want to experiment without affecting the main codebase.

In this scenario, use a Git branch to create a separate line of development for the new feature. This avoids affecting the main codebase. Once the feature is complete and tested, you can merge the branch back into the main codebase. Depending on your organization’s process, you may choose to delete the branch to prevent clutter.

### When to use tags

Now, suppose that you complete and test the new feature. You want to release the new software version to the users. In this case, use a Git tag to mark the current state of the codebase as a new version release.

You can name the tag to reflect the version number (such as `v1.2.3`) and include a brief description of the changes in the release. This allows you to easily reference the specific version of the released codebase. It is also easy to roll back to the previous version if necessary.

Tags and branches are important tools for keeping your development process organized and efficient. Not only do they make it easier for development teams to manually coordinate and review their progress, but they can also help automate, orchestrate, and monitor development processes in a [continuous integration and delivery (CI/CD) pipeline](https://circleci.com/blog/what-is-a-ci-cd-pipeline/).

For example, a CI/CD tool like CircleCI can be automated to build and test the code in a new feature branch to make sure it is ready to integrate into the main codebase. The production branch remains stable and developers can add new features quickly and safely without breaking existing functionality.

Every CI/CD pipeline is different. There are several [branching strategies](https://circleci.com/blog/you-are-what-you-git-how-your-vcs-branching-model-affects-your-delivery-cadence/) you can use during development, including:

- The Git flow model, which uses two main branches (develop and main) and several supporting branches for different stages of development.
- The feature branch model, which uses a separate branch for each new feature being developed.
- The trunk-based model, which uses a single main branch for all development. It uses temporary branches for small, frequent code check-ins into the main branch.

The best strategy for your project depends on your team’s specific needs and goals. The most effective DevOps teams create faster feedback loops and improve team velocity by making smaller, more frequent changes directly to the main line. As such, [trunk-based development](https://circleci.com/blog/trunk-vs-feature-based-dev/) can be the best fit for teams that want to get the most value out of their continuous integration pipelines.

There are many ways that branches and tags can help manage features, testing, deployment, and releases from your CircleCI pipeline.

With [CircleCI](https://circleci.com/product/), you can use tags as triggers for your pipelines. You can specify tags that automatically trigger a pipeline to run when they are added to a commit. This can be useful for several things, including automated tests or deploying your code to production.

You can also use tags to track the progress of your pipeline runs. You can use a tag to indicate that a certain step in the pipeline is complete. This helps you track the overall progress of the pipeline and identify potential issues. You can also use tags to filter for certain workflows in the [Insights dashboard](https://circleci.com/docs/insights/), giving you actionable metrics on build speed and performance that you can use to optimize your team’s delivery process.

## Branch and tag filters

With CircleCI’s [branch and tag filters](https://circleci.com/docs/configuration-reference/#jobfilters), you can specify the branches and tags to include in a pipeline. You can use branch filters to run a workflow (or a specific job within a workflow) only for changes on specific branches. This is useful for restricting certain jobs — such as running a subset of your tests or deploying your code to production — to branches dedicated to those activities.

You can also use Git tags to automatically execute a workflow in CircleCI using the `filters` key in your CircleCI configuration. This allows you to specify which tags should trigger the workflow.

To use branch and tag filters for workflows in CircleCI, [specify the filters in the workflows section of your CircleCI configuration](https://circleci.com/docs/workflows/#using-contexts-and-filtering-in-your-workflows). Using branch and tag filters in CircleCI you can fine-tune your pipeline and control when and how your changes are built, tested, and deployed.

## Monitor tagged workflows

The CircleCI Insights dashboard allows you to monitor workflows and track the performance of your builds.

To access the Insights endpoint, go to the CircleCI web UI and click the **Insights** tab. This brings up a dashboard of [important metrics](https://circleci.com/blog/how-to-measure-devops-success-4-key-metrics/) showing the performance of your pipelines over time. There is also data about resource and credit consumption that you can use to predict and control costs.

      ![Insights dashboard](app://images.ctfassets.net/il1yandlcjgk/16PSYJLU5Ikk2eAOOZkq5g/1bf438c94f02817b70b9c90aabfb52dd/tags-vs-branches-image3.png?w=1003&fm=webp&q=60)

You can use the filters to narrow down the data, showing a specific tag’s triggered pipelines.

      ![Pipelines filtered by tag](app://images.ctfassets.net/il1yandlcjgk/9Jf7hgWcPhCdir4GfR2hA/5070ae100d5afed658c2f1011034adfb/tags-vs-branches-image4.png?w=1003&fm=webp&q=60)

This allows you to track information such as [success rate and workflow duration](https://circleci.com/docs/insights-glossary/#general-metrics) or review the most recent workflow runs for a given tag.

      ![Summary metrics](app://images.ctfassets.net/il1yandlcjgk/1B1KWA8514n0iYAODv7rNk/3495efbc84550d43be166cc6b060fe7c/tags-vs-branches-image1.png?w=1003&fm=webp&q=60)

## Conclusion

Many development teams that use Git frequently use both Git branches and Git tags. Branches allow you to code features or fix bugs without impacting the main code branch. Tags are essential for marking a point in time in your code, such as a new release of your application.

Branches and tags work together to optimize your workflow. With CircleCI, they can also add flexibility and functionality to your CI/CD process. You can use them to orchestrate and streamline builds, tests, and deployments and to monitor the performance of high priority builds.

Learn more about how you can use effective branching and tagging strategies with best-in-class CI/CD. To streamline your software delivery, get started with your [free CircleCI account](https://circleci.com/signup/) today.