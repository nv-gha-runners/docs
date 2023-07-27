# Overview

## Introduction

The `nvidia-runners` GitHub application provides NVIDIA with the necessary permissions to provision self-hosted runners to a particular GitHub organization. Upon a successful installation, it will create a runner group that will need to be configured as described below.

## Installation

Before installing the GitHub application, open a pull request to add your organization to the allow list in [src/orgs.ts](https://github.com/nv-gha-runners/rapids-runners/blob/main/src/orgs.ts).

The application will immediately uninstall itself from any organizations that are not on this list.

This ensures that organizations who aren't intended to use this application will not send it additional webhook requests and therefore increase our compute load.

Once the pull request has been merged, install the `nvidia-runners` application to your GitHub organization with this link: <https://github.com/apps/nvidia-runners>.

## Configuration

Upon installation, the application will create the `rapids-runners` runner group in the installed organization.

!!! note

    The `rapids-runners` group will eventually be renamed to `nvidia-runners`.

    All existing organizations will be updated automatically when this change occurs.

By default, the runner group will be:

- Usable on selected repositories (though no repositories will be selected)
- Enabled for public repositories

To configure which repositories have access to the runner group, follow the GitHub documentation [here](https://docs.github.com/en/actions/hosting-your-own-runners/managing-self-hosted-runners/managing-access-to-self-hosted-runners-using-groups#changing-which-repositories-can-access-a-runner-group).
