# Overview

## Introduction

`copy-pr-bot` is a GitHub application that automates the pull request testing strategy described in the CircleCI blog post entitled [_Triggering trusted CI jobs on untrusted forks_](https://circleci.com/blog/triggering-trusted-ci-jobs-on-untrusted-forks/).

The post describes a process for testing pull requests from public forks that ensures the changes are vetted first before triggering a CI workflow.

The gist of the process is that users can configure their CI system to trigger on pushes to certain prefixed branches (e.g. `pull-request/`) within a source repository instead of triggering on pull request events.

Then, when a pull-request is opened, a repository member with `write` permissions or greater can inspect the changes of the pull request to ensure that they are both legitimate and benign.

Assuming the changes meet that criteria, the repository member can then push the changes to the source repository on a prefixed branch, like `pull-request/123`, to trigger a CI workflow.

Since the commit SHA of the branch within the source repository matches the commit SHA on the pull request, any workflow statuses reported as a result of the branch workflow will also be reported back to the pull request.

The `copy-pr-bot` GitHub application provides some automation to simplify this process.

## Automation

The automation works like this:

!!! note

    Bolded terms are defined in the [glossary](#glossary) section below.

- If a pull request is opened by a **trusted user** and contains only **trusted changes**, the pull request's code will automatically be copied to a `pull-request/` prefixed branch in the **source repository** (e.g. `pull-request/123`)
  - Any subsequent **trusted changes** that are pushed to the pull request will also be copied to the branch in the **source repository** automatically
- If a pull request is opened by an **untrusted user** _or_ contains **untrusted changes**, the application will leave a comment on the pull request indicating that the changes need to be vetted first before they are copied to the **source repository**
  - After the changes are vetted, the **vetter** can leave an `/ok to test` comment on the pull request to have it copied to the **source repository**
  - Once a pull request contains an **untrusted change**, every subsequent commit will require an `/ok to test` comment to copy the latest changes to the **source repository**

An `/ok to test` comment will receive a 👍 reaction from `copy-pr-bot` if the pull request code was successfully copied to the **source repository**.

A pull request's corresponding `pull-request/*` branch is deleted when the pull request is merged or closed.

For more details on the vetters' responsibilities, check out [this FAQ](./faqs.md#what-are-my-responsibilities-as-a-vetter).

## Installation

Before installing the GitHub application, open a pull request to add your organization to the allow list in [src/orgs.ts](https://github.com/nv-gha-runners/copy-pr-bot/blob/main/src/orgs.ts).

The application will immediately uninstall itself from any organizations that are not on this list.

Once the pull request has been merged, install the `copy-pr-bot` application to your GitHub organization with this link: <https://github.com/apps/copy-pr-bot>.

Upon installation, `copy-pr-bot` won't do anything unless the [configuration file described below](#configuration) is committed and has `enabled: true` set. Therefore, it is okay to install the `copy-pr-bot` GitHub application on _all_ repositories in an organization by default if desired.

## Configuration

The `copy-pr-bot` application can be configured by adding a `.github/copy-pr-bot.yaml` file to a repository. This file path must be used exactly. Paths like `.github/copy-pr-bot.yml` (note `.yml` vs. `.yaml`) will not work.

Additionally, the configuration file will not take effect until it is committed to the default branch of a given repository.

The snippet below shows an example configuration:

```yaml
enabled: true
additional_trustees:
  - nvidia_employee_gh_username_1
  - nvidia_employee_gh_username_2
  - non_nvidia_employee_gh_username_1 # non NVIDIA employees on this list are ignored
additional_vetters:
  - nvidia_employee_gh_username_3
  - nvidia_employee_gh_username_4
  - non_nvidia_employee_gh_username_2 # non NVIDIA employees on this list are ignored
auto_sync_draft: false
auto_sync_ready: true
```

The values for the `additional_trustees` and `additional_vetters` keys are a list of GitHub usernames for NVIDIA employees. Any non-NVIDIA employee usernames from this list are ignored. To determine whether a username belongs to an NVIDIA employee, the usernames are compared against the NVIDIA Enterprise Member List (see the [glossary](#glossary) for information about this list).

The `auto_sync_draft` and `auto_sync_ready` keys control whether or not the bot automaticallly pushes the branch for a PR that is in the draft state or ready for review state, respectively. If `auto_sync_ready` is not set, it defaults to `true`. If `auto_sync_draft` is not set, it defaults to the value of `auto_sync_ready`.

Once the application is configured, you can begin creating workflows using `on: push` events:

```yaml
name: pull request workflow
on:
  push:
    branches:
      - "pull-request/[0-9]+"
```

## Glossary

- **NVIDIA Enterprise Member List**: an aggregated list of all of the members of each organization in the NVIDIA GitHub Enterprise account. These users are NVIDIA employees
- **Source Repository**: a repository like `rapidsai/cudf`. This is in contrast to a forked repository, like `gh_user/cudf`
- **Trusted Change**: a commit that is both signed and from a trusted user
- **Trusted User**: an organization member _or_ a member that is listed in the `additional_trustees` configuration list (see note below)
- **Untrusted Change**: a commit that is not a trusted change (according to the definition above)
- **Untrusted User**: a user that is not a trusted user (according to the definition above)
- **Vetter**: a user that has permissions to leave an `/ok to test` comment on a pull request. This is a user with `write` access (or greater) for a particular repository _or_ a member that is listed in the `additional_vetters` configuration list (see note below)

!!! note

    Any usernames specified in the `additional_trustees` or `additional_vetters` lists will be ignored if they are not also a part of the NVIDIA Enterprise Member List (described above).

## Limitations

### Large Number of Pull Request Commits

If a pull request has 250 commits or greater, it will automatically be treated as untrusted.

This is due to the limitations describe in GitHub's API docs here: <https://docs.github.com/en/rest/pulls/pulls?apiVersion=2022-11-28#list-commits-on-a-pull-request>.

In this scenario, the `copy-pr-bot` application is unable to retrieve all of the commits on the pull request, so it is treated as untrusted.

To work around this limitation, squash the commits in your pull request or have a vetter comment `/ok to test` to have the pull request code manually copied to the source repository.

## FAQs

For frequently asked questions and troubleshooting tips, view the [FAQs](./faqs.md).

## Contributing

View the source repository for information about contributing to the application: <https://github.com/nv-gha-runners/copy-pr-bot>.
