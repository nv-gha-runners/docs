# Overview

The onboarding steps for non-NVIDIA GitHub organizations are:

1. Review [this](https://docs.github.com/en/actions/managing-workflow-runs/approving-workflow-runs-from-public-forks) GitHub documentation and follow the links to enable required workflow approvals for pull requests from **all outside contributors**.
   - This will help prevent abuse of our runners (e.g. crypto mining). **Organizations who do not enable this option may have their runner access revoked**. Setting this option at the organization level only affects new repositories. Therefore, the setting will need to be explicitly enabled for existing repositories as well.
1. Review and install the [`nvidia-runners`](../apps/nvidia-runners/index.md) GitHub application
   - Wait for confirmation from NVIDIA's GitHub Actions team before moving to the final step
1. View the [runners](../runners/index.md) page and start using the self-hosted runners!
