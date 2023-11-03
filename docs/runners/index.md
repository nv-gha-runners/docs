# Overview

This page includes information about the self-hosted runners that are available from NVIDIA and how to use them.

## CPU Runners

The CPU labeled runners are backed by various EC2 instances and do not have any GPUs installed.

| Label               | EC2 Machine Type             |
| ------------------- | ---------------------------- |
| `linux-amd64-cpu4`  | `m5d.xlarge` ([specs][m5])   |
| `linux-amd64-cpu8`  | `m5d.2xlarge` ([specs][m5])  |
| `linux-amd64-cpu16` | `m5d.4xlarge` ([specs][m5])  |
| `linux-arm64-cpu4`  | `m6gd.xlarge` ([specs][m6])  |
| `linux-arm64-cpu8`  | `m6gd.2xlarge` ([specs][m6]) |
| `linux-arm64-cpu16` | `m6gd.4xlarge` ([specs][m6]) |

[m5]: https://aws.amazon.com/ec2/instance-types/m5/#Product_details
[m6]: https://aws.amazon.com/ec2/instance-types/m6g/#Product_Details

<!-- prettier-ignore-start -->
!!! info
    The CPU label names consist of the following components:

    ```text
    linux-amd64-cpu4
    ^     ^     ^  ^
    |     |     |  |
    |     |     |  CPU Core Count
    |     |     CPU Designator
    |     Architecture
    Operating System
    ```
<!-- prettier-ignore-end -->

## GPU Runners

The GPU labeled runners have the GPUs specified in the table below installed.

<!-- prettier-ignore-start -->
!!! warning "Important!"
    GPU jobs have two requirements. If these requirements aren't met, the GitHub Actions job will fail. See the [_Example_](#example) section below for an example.

    1. They must run in a container (i.e. `nvidia/cuda:12.0.0-base-ubuntu22.04`)
    2. They must set the `NVIDIA_VISIBLE_DEVICES: ${{ env.NVIDIA_VISIBLE_DEVICES }}` container environment variable
<!-- prettier-ignore-end -->

| Label                             | GPU    | Driver Version | # of GPUs |
| --------------------------------- | ------ | -------------- | --------- |
| `linux-amd64-gpu-v100-earliest-1` | `V100` | `470`          | `1`       |
| `linux-amd64-gpu-v100-latest-1`   | `V100` | `525`          | `1`       |
| `linux-arm64-gpu-a100-latest-1`   | `A100` | `525`          | `1`       |
| `linux-amd64-gpu-t4-latest-1`     | `T4`   | `525`          | `1`       |

<!-- prettier-ignore-start -->
!!! info
    The GPU label names consist of the following components:

    ```text
    linux-amd64-gpu-t4-latest-1
    ^     ^     ^   ^  ^      ^
    |     |     |   |  |      |
    |     |     |   |  |      Number of GPUs Available
    |     |     |   |  GPU Driver Version
    |     |     |   GPU Type
    |     |     GPU Designator
    |     Architecture
    Operating System
    ```
<!-- prettier-ignore-end -->

Due to limited GPU capacity and the overhead associated with manually rotating self-hosted runner labels when GPU drivers are updated, there are no driver-specific self-hosted runner labels (e.g. `linux-amd64-gpu-t4-525-1`).

Instead, the driver-version designators `earliest` and `latest` are used. The values of these designators represent the oldest and newest NVIDIA supported drivers at any given time.

The chart above will be kept up-to-date with the corresponding driver versions at any given time.

Supported organizations will be notified whenever these versions are scheduled to be updated.

## Example

The code snippet below shows how the labels above may be utilized in a GitHub Action workflow.

```yaml
name: Test Self Hosted Runners
on: push
jobs:
  job1_cpu:
    runs-on: linux-amd64-cpu8
    steps:
      - name: hello
        run: echo "hello"
  job2_gpu:
    runs-on: linux-amd64-gpu-v100-latest-1
    container: # GPU jobs must run in a container
      image: nvidia/cuda:12.0.0-base-ubuntu22.04
      env:
        NVIDIA_VISIBLE_DEVICES: ${{ env.NVIDIA_VISIBLE_DEVICES }} # GPU jobs must set this container env variable
    steps:
      - name: hello
        run: |
          echo "hello"
          nvidia-smi
```

For additional details on self-hosted runner usage, see the official GitHub Action documentation page [here](https://docs.github.com/en/actions/hosting-your-own-runners/using-self-hosted-runners-in-a-workflow).
