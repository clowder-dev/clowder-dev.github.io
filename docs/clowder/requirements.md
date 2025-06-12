---
id: clowder-requirements
slug: /docs/clowder/requirements
title: Hardware Requirements
sidebar_position: 5
---

## Hardware Requirements

Most software runtimes have 2 sets of requirements: the platform and the resources.

For example, running on `linux/amd64` or `darwin/arm64`, with a minimum of 2 CPUs and 4 GB of RAM, and 10 GB of disk space.

Most of the Clowder platform components, such as the controller, storage manager, and load balancer, have these kinds of requirements. They can run on any physical or virtual machine that meets the platform requirements.

The worker nodes, however, have additional requirements. They need to run an AI runtime engine, on which inference is performed. These runtimes have their own requirements, which are typically more stringent than the platform requirements.

The worker node _can_ run using solely a regular CPU and its memory, but this is not recommended. The performance will be poor, and the inference will take a long time. It is highly recommended to run the worker nodes on machines with dedicated hardware accelerators, such as GPUs or TPUs.

Clowder is built to support the following inference engines:

* CPU, both amd64/x86_64 and arm64/aarch64
* Nvidia GPUs, using CUDA and TensorRT
* AMD GPUs, using ROCm
* RISC-V specialized processing units
