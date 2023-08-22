# Overview

## Introduction

This page outlines some important preliminary information that users will need to know about testing pull requests with NVIDIA's self-hosted runners.

Once this information has been reviewed, follow the links at the bottom of this page to find out more about the onboarding process.

## Pull Request Testing

For security reasons, GitHub does not currently recommend using self-hosted runners with the `pull_request` or `pull_request_target` events on public repositories.

Therefore, these event types have been disabled entirely on NVIDIA's self-hosted runners.

If a self-hosted runner determines that the job it's about to run has been triggered by one of these disabled events, it will exit immediately before the job can run.

To safely test pull requests despite this limitation, NVIDIA has adopted a process known as _marking code as trusted by pushing upstream_.

The gist of this process is that users can configure GitHub Actions to trigger on pushes to certain prefixed branches (e.g. `pull-request/`) of a source repository instead of triggering on pull request events.

Then, when a pull request is opened, a repository member with `write` permissions or greater can inspect the changes of the pull request to ensure that they are both legitimate and benign.

Assuming the changes meet this criteria, the repository member can then push the changes to the source repository on a prefixed branch, like `pull-request/123`, to trigger a GitHub Actions workflow.

Since the commit SHA of the branch within the source repository matches the commit SHA on the pull request, any workflows statuses reported as a result of the branch workflow will also be reported back to the pull request.

Complete details about this strategy can be found in the CircleCI blog post entitled [_Triggering trusted CI jobs on untrusted forks_](https://circleci.com/blog/triggering-trusted-ci-jobs-on-untrusted-forks/).

### Automating this Process

Since it's not practical to expect repository members to manually copy the source code for every pull request into the source repository, NVIDIA has created a GitHub application, called [`copy-pr-bot`](../apps/copy-pr-bot/index.md), to help automate this process.

The application will automatically copy pull request code from trusted pull requests into the source repository so that GitHub Action workflows can begin without any intervention.

Any untrusted pull requests will need to be vetted and receive an `/ok to test` comment by a trusted user before the application will copy its code to the source repository for a workflow to begin.

You can view the `copy-pr-bot` application page [here](../apps/copy-pr-bot/index.md) for more details on how the application works and for the definitions of trusted and untrusted pull requests.

### Example

The result of the process described above means that GitHub Action workflows that are triggered by `pull_request` and `pull_request_target` events will not run:

<!-- prettier-ignore-start -->
!!! failure "Incorrect: self-hosted runners will not run jobs from this workflow"
    ``` yaml
    name: pull request workflow
    on:
      pull_request: # or `pull_request_target`
    ```
<!-- prettier-ignore-end -->

Instead, configure your pull request workflows to run on `pull-request/*` branches like this:

<!-- prettier-ignore-start -->
!!! success "Correct"
    ``` yaml
    name: pull request workflow
    on:
      push:
        branches:
          - "pull-request/[0-9]+"
    ```
<!-- prettier-ignore-end -->

### Future Plans

Once GitHub addresses the security concerns associated with using self-hosted runners with `pull_request`-type events, we will remove this restriction from our self-hosted runners and update this documentation accordingly.

## Next Steps

GitHub organizations that are a member of the NVIDIA GitHub Enterprise account can go [here](./nvidia.md) to proceed with onboarding instructions.

GitHub Organizations that are not part of the NVIDIA GitHub Enterprise account can go [here](./non-nvidia.md).
