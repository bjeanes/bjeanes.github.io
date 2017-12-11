# Covidence Engineering Site

[Deployed Site](http://covidence.engineering)

## Deploying

This site is hosted on GitHub pages but is not built by GitHub pages (due to
the use of [`jekyll-archives` plugin](https://github.com/jekyll/jekyll-archives)).
However, the deployment experience is equivalent; simply commit source changes
and push to GitHub or edit in-line in GitHub UI.

A BuildKite CI job will generate the site and push just the built site files to
the appropriate branch for GitHub to serve.

For more details on how this works, see [the initial
blog](http://covidence.engineering/deploy-directory-to-branch/) post on the
blog.

