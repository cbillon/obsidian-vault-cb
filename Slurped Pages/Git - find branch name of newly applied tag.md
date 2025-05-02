---
link: https://fabianlee.org/2023/09/21/git-find-branch-name-of-newly-applied-tag/
site: "Fabian Lee : Software Engineer"
excerpt: "If you are within the context of a CI/CD tool, you may run into the
  scenario where a newly applied git tag has initiated a pipeline action.
  Depending on the tool, the pipeline will provide you with either a SHA of the
  last commit and/or the tag name – but not the branch where the ... Git: find
  branch name of newly applied tag"
twitter: "https://twitter.com/Fabian Lee : Software Engineer"
tags:
  - slurp/action
  - slurp/github
  - slurp/gitlab
  - slurp/pipeline
  - slurp/resolve
  - slurp/sha
  - slurp/sha256
  - slurp/tag
slurped: 2025-03-02T10:00
title: "Git: find branch name of newly applied tag"
---

![](https://fabianlee.org/wp-content/uploads/2016/09/gitlogo.png)If you are within the context of a CI/CD tool, you may run into the scenario where a newly applied git tag has initiated a pipeline action.

Depending on the tool, the pipeline will provide you with either a SHA of the last commit and/or the tag name – but not the branch where the tag was applied.  This is because a git tag is tied to a commit, not a specific branch.

However, if the tag was applied as the latest commit on a branch (a likely scenario for triggering a pipeline), then you can use the following commands to resolve back to a branch name.

# pipeline may already provide the git SHA
# but if not, you can lookup by tag name
tag_name=v1.0.0
tag_sha=$(git rev-list -1 $tag_name)

# find branch name
git for-each-ref | grep ^$tag_sha | grep origin | head -n1 | sed "s/.*\///"

This will NOT work if:

- There has been a more recent commit to this branch
- You have a shallow fetch/clone, that does not have the branch/tag history

## GitLab pipeline

The CI_COMMIT_SHA and CI_COMMIT_TAG are part of the [predefined variables](https://docs.gitlab.com/ee/ci/variables/predefined_variables.html).  But you will need to set the variable GIT_DEPTH=0 or the project CI/CD settings to “git clone” to see the origin refs, because by default it only does a shallow git fetch.

simple-resolve-branch-on-tag:
  rules:
    - if: $CI_COMMIT_TAG
  stage: .pre
  # small alpine image that has git/grep/sed installed
  image: registry.gitlab.com/gitlab-pipeline-helpers/docker-alpine-with-envsubst:1.0.6
  variables:
    GIT_DEPTH: 0
  script: |
    echo "CI_COMMIT_TAG = $CI_COMMIT_TAG"
    echo "CI_COMMIT_SHA = $CI_COMMIT_SHA"
    echo "CI_COMMIT_SHORT_SHA = $CI_COMMIT_SHORT_SHA"
    echo "candidates for branch name"
    git for-each-ref | grep ^$CI_COMMIT_SHA | grep origin | grep -v HEAD | head -n1 | sed "s/.*\///"
    branch_name=$(git for-each-ref | grep ^$CI_COMMIT_SHA | grep origin | grep -v HEAD | head -n1 | sed "s/.*\///")
    echo tag $CI_COMMIT_TAG is on branch $branch_name

Here is a full [example .gitlab-ci.yml](https://gitlab.com/gitlab-pipeline7091038/gitlab-pipeline-lifecycle-example/-/blob/main/.gitlab-ci.yml).

The [docker-alpine-with-envsubst](https://gitlab.com/gitlab-pipeline-helpers/docker-alpine-with-envsubst) image is one I built using an alpine-based image that ensures I have the git, grep, and sed utilties (~36Mb).

## GitHub Action

The GITHUB_SHA and GITHUB_REF_NAME are part of the [predefined variables](https://docs.github.com/en/actions/learn-github-actions/variables).  But you will need to set the ‘fetch-depth’ of actions/checkout to 0 to see the origin refs, because by default it only does a shallow git fetch.

...
      - name: Check out repository code
        uses: actions/checkout@v4
        with:
          fetch-depth: 0
...
      - name: calculate branch name IF tag
        if: github.ref_type == 'tag'
        run: |
          echo "GITHUB_REF_NAME = $GITHUB_REF_NAME"
          echo "GITHUB_SHA = $GITHUB_SHA"
          branch_name=`git for-each-ref | grep ^$GITHUB_SHA | grep origin | grep -v HEAD | head -n1 | sed "s/.*\///"`
          echo tag $GITHUB_REF_NAME is on branch $branch_name

Here is a full example, [github-actions-buildOCI.yml](https://github.com/fabianlee/tiny-tools-multi-arch/blob/master/.github/workflows/github-actions-buildOCI.yml)

REFERENCES

[stackoverflow, finding source branch when creating tag in gitlab using GitLab workflows](https://stackoverflow.com/questions/62033350/find-source-branch-when-creating-tag-in-gitlab-using-gitlab-ci-yml)

[stackoverflow, use git rev-list as a general solution for both annotated and lighweight tags (show-ref only works for lighweight)](https://stackoverflow.com/questions/1862423/how-to-tell-which-commit-a-tag-points-to-in-git/1862542#1862542)

[gitlab, predefined variables](https://docs.gitlab.com/ee/ci/variables/predefined_variables.html)

[github, predefined GitHub Action variables](https://docs.github.com/en/actions/learn-github-actions/variables)

[stackoverflow, exhaustive search on tags for every branch](https://stackoverflow.com/questions/32166548/how-to-list-all-tags-within-a-certain-git-branch)

[Gitlab issue, checking for empty variables and non-existent variables in pipeline](https://gitlab.com/gitlab-org/gitlab/-/issues/220562)

[GitLab source, docker-alpine-with-envsubst image](https://gitlab.com/gitlab-pipeline-helpers/docker-alpine-with-envsubst)

NOTES

**‘show-ref’ only works for lightweight tags (not annotated)  
**

lightweight_tag_name=v1.0.0
lightweight_tag_sha=$(git show-ref --dereference $tag_name | head -n1 | cut -d' ' -f1)