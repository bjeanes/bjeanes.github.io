---
layout: post
author: bo
title: Deploy a sub-directory as a branch
subtitle: As an example, this blog is deployed to GitHub pages in this way...
tags:
  - git
  - github
  - shell
---

This post outlines a mechanism for using one branch as the source to generate a
compiled artifact into another branch. This is achieved without resorting to
moving files around a working directory and staging dirty changes away safely,
by leveraging some neat internals of Git.

### Why?

But first, why would someone want to do this?

In general, storing (deterministically) buildable artifacts in a repository can
be unneccessary and even problematic. For example: building a binary which
can't run on all platforms that might use the repository; or creating large
files which grow the repository size rapidly (e.g. JARs).

In the past, I have been in situations where using Git's object model to store
an immutable history of changes in compiled outputs has been useful, in
particular for analytical purposes.

However, this time, the reason is rather mundane: GitHub's Jekyll compilation
whitelists certain plugins (for good, security-related reasons) and I had set
up this blog using a theme and plugin combination that apparently is not
compatible with GitHub-built Jekyll projects.

### How

GitHub expects to find source files for GitHub Pages deployment in either the
`gh-pages` or `master` branch (depending on whether its building a project page
or a user/org page). All that was necessary was to generate the compiled files
into that appropriate branch.

However, I did not want it to be up to the person making changes to the blog to
have to remember to generate and commit the generated files. In addition, it was
important to me that the Git history of source changes was not conflated with
the adjacent need to store the compiled version.

That meant:

1. As much automated as possible
2. The compiled branch (`master`, in this case) should be nothing but the
   generated files and a separate commit history.

#### Understanding `git commit`

When you run a command like `git commit .`, Git is doing a few of things
of interest internally:

1. `git-write-tree` stores the current state of your (staged) files as a tree
2. `git-commit-tree` creates a commit for that tree (and, usually, a parent
   commit)
3. `git-update-ref` Updating a ref (branch) to point to that new commit

By dropping down a level and taking control of these primitives directly, we
can actually exercise a little bit more control over what is in the commit and
how that commit relates to the rest of the repository's history.

#### Writing a tree at a custom root

The key ingredient that we will leverage is the `--prefix` flag to
`git-write-tree`. This allows us to create a tree object rooted at an arbitrary
directory!

For example, compare the following:

```
$ git show $(git write-tree)
tree c97ce6b80599da62e34a684a245dec5df1228b8d

.buildkite/
.gitignore
CNAME
Gemfile
Gemfile.lock
_config.yml
_data/
_drafts/
assets/
bin/
index.html
pages/

$ git show $(git write-tree --prefix pages)
tree 8e479816fadad469d16bcb16c24413feb780765f

archive.html
category.html
tag.html
```

### Piecing it together

The main caveat to account for is that `git-write-tree` will only write staged
or committed files. I don't want to keep the built files in the source branch
so I have the build directory ignored, which means those files are neither
committed nor staged.

To work around this, a `bin/deploy` script temporarily stages the files for the
duration of the commit building. **NOTE**, because I am deploying to `master`
branch, I configured the repository's default branch to be `source` and the
`bin/deploy` script lived there, along with all the Jekyll source files.

Here's a simplified version of the script:


``` bash
#!/usr/bin/env bash

branch="master"
build_dir="dist"

# This can be substituted for any command which builds artifacts to a
# specific directory
bundle exec jekyll build --destination $build_dir

# Force the files to be added, regardless of .gitignore
git add -f $build_dir

# Write tree to an object
tree=$(git write-tree --prefix=$build_dir)

# Un-stage the build so it is back to being ignored
git reset -- $build_dir

# Create a new commit for that tree as a child of target branch's commit
commit=$(git commit-tree -p $branch -m "Deploy" $tree)

# Update the GitHub Pages' branch to the new commit
git update-ref refs/heads/$branch $commit

# Push the compiled branch to GitHub
git push origin $branch
```

### Automated builds

I used our existing [Buildkite](https://buildkite.com) CI setup to quickly add
a build which essentially just called `bin/deploy` to automatically deploy
changes we made to GitHub (much the way that GitHub would automatically deploy
changes if I were committing directly to `master`/`gh-pages`).

### Some changes for resilience

* I wanted to preserve committer/authorship metadata for the deploy commits to
  make it clear whose latest change is deployed. To handle this, the
  `bin/deploy` script takes these details from the commit being built and sets
  some env vars before we generate the deploy commit:

  ``` bash
  export GIT_AUTHOR_NAME=$(git show -s --format='%an' HEAD)
  export GIT_COMMITTER_NAME="$GIT_AUTHOR_NAME"
  export GIT_AUTHOR_EMAIL=$(git show -s --format='%ae' HEAD)
  export GIT_COMMITTER_EMAIL="$GIT_AUTHOR_EMAIL"

  commit=$(git commit-tree -p $branch -m "Deploy" $tree)
  ```

* I wanted to make it clear which commit had been deployed, so I augmented the
  commit creation step to record a description of the source commit in the commit
  message, including whether or not the working directory had uncommitted
  changes:

  ``` bash
  identifier=$(git describe --dirty --always) # e.g. "213c9af", "213c9a-dirty"
  commit=$(git commit-tree -p $branch -m "Deploy $identifier" $tree)
  ```

* For maximum re-usability, I have the script detect the target branch based on
  the type of repository (project or user/org):

  ``` bash
  # Determine which branch GitHub pages is built from, for this repository
  remote=$(git config remote.origin.url)
  if [[ $remote = *.github.io ]] || [[ $remote = *.github.io.git ]]; then
    branch=master
  else
    branch=gh-pages
  fi
  ```

* To still work in CI where a clean checkout has occurred without necessarily
  creating local branch refs, I use long-form names
  (`refs/remotes/origin/$branch` or `refs/heads/$branch`, as appropriate)

### Result

Feel free to have a look at the [repository for the
blog](https://github.com/covidence/covidence.github.io). The `master` branch
contains the history of deployed changes and the `source` branch contains the
history of source changes.

This blog post was deployed in this manner.

----

[Related gist](https://gist.github.com/nobuoka/d0f088df57d50e4cda1a)
