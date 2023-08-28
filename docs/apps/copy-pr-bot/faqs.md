# FAQs

## Why is this GitHub application necessary?

This application is necessary to ensure that pull requests from public repository forks are properly vetted before any CI workflows run on them.

Running workflows on pull requests from public forks without vetting them first is a security risk. This becomes especially important for CI machines that are self-hosted.

For additional details about the motivation behind this application, view the onboarding page [here](../../onboarding/index.md).

## What are my responsibilities as a vetter?

[Vetters](./index.md#glossary) are responsible for ensuring that the changes introduced by a pull request are both legitimate and benign.

If a commit introduces changes that seem suspicious (running network scans, uploading/downloading/executing suspicious files, trying to access secrets, etc.), reach out to an organization or repository administrator for additional guidance.

If the changes are legitimate and benign, an `/ok to test` comment can be left on the pull request to copy its code to a branch in the source repository so that a CI workflow can begin.

Every subsequent commit on an untrusted pull request will also need to be vetted and will require an additional `/ok to test` comment.

## Why did I receive a comment that my pull request requires additional validation?

!!! note

    For definitions of the bolded terms below, view the [glossary](./index.md#glossary).

If any of the criteria below are true, a pull request is deemed untrusted and will require an `/ok to test` comment from a **vetter** for every subsequent commit that is pushed:

- The pull request was opened by an **untrusted user**
- The pull request contains commits from an unknown or **untrusted user**
- The pull request contains 250 commits or greater
- The pull request contains unsigned commits (see [this](https://docs.github.com/en/authentication/managing-commit-signature-verification/about-commit-signature-verification) GitHub documentation about signing commits)

In the event that a **trusted user** accidentally pushes an unsigned commit, they may rebase their changes locally to sign the commit(s) in question and force push an update to their pull request. Assuming all of the rebased changes are **trusted changes**, the pull request will no longer require an `/ok to test` comment.

## Why didn't my `/ok to test` comment work?

If an `/ok to test` comment didn't work as expected, check the following items:

- Ensure that the author of the `/ok to test` comment is a **vetter** as defined in the [glossary](./index.md#glossary)
- Ensure that the `/ok to test` comment matches this regular expression: `^/ok(ay)? to test$`. Whitespace is trimmed.

A successful `/ok to test` comment will receive a 👍 reaction from `copy-pr-bot`.

## The application isn't working correctly. How can I troubleshoot?

Here are a few things to check if the application isn't working as expected:

- Ensure the application is installed on the repository in question. An organization or repository administrator will typically have to verify this.
- Ensure the `copy-pr-bot.yaml` file is committed to the default branch. The application won't start working until the file is on the default branch of the repository.
- Ensure the `copy-pr-bot.yaml` file has the correct file name and is in the correct directory. It should be in `.github/copy-pr-bot.yaml` exactly. Using a `.yml` file extension will not work.

## Can I ignore `pull-request/*` branches locally?

Yes, you can avoid fetching `pull-request/*` branches locally by running the `git` command below, where `upstream` in `remote.upstream.fetch` is the `git` remote name corresponding to the source repository:

```sh
git config \
  --global \
  --add "remote.upstream.fetch" \
  '^refs/heads/pull-request/*'
```

Note that this `git` configuration option requires `git` version `2.29` or greater to support negative refspecs ([source](https://github.blog/2020-10-19-git-2-29-released/#user-content-negative-refspecs)).

## Will the application clean up branches that it creates?

Yes. Anytime a pull request is closed or merged, it's corresponding branch in the source repository is also deleted.
