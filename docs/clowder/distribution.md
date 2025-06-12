---
id: clowder-distribution
slug: /docs/clowder/distribution
title: Aimage and Model Distribution
sidebar_position: 6
---

# Distribution

Distribution is the process of moving aimage and model components between different systems.
Specifically, Clowder is concerned with distributing aimages, or their attendant components,
from centralized repositories to worker nodes.

Distribution requires a recognized protocol.

Clowder understands distribution to and from the following protocols:

* OCI - the preferred native format
* ollama - similar to but not identical to OCI
* Huggingface - a custom format

![Distribution formats](image/distribution_formats.png)

The rest of this document describes how Clowder handles distribution for each of these protocols, and especially how it stores the data locally.

Clowder has a goal to use highly-efficient storage for all distributed data, and to minimize duplication of data across different protocols. This means that, for example, if a model is available in both OCI and ollama formats, Clowder will store the model layers only once, regardless of the protocol used to access it.

All artifacts are stored in a content-addressable format, meaning that the data is stored based on its content rather than its name or location. This allows for efficient deduplication and storage management. All storage follows the [OCI v1 layout](https://specs.opencontainers.org/image-spec/image-layout/), which is a well-defined structure for local content-addressable storage.

More details about internal storage can be found in the [Clowder Storage](./storage.md) document.

## Protocols 

### OCI
Using OCI artifacts, Clowder can pull layers from or push to any OCI-compliant registry. Authentication follows the OCI distribution spec’s authentication, including support for anonymous access.

Blobs are replicated 1:1 between the local cache in `blobs/sha256/<digest>` and the remote registry as blobs. On pull, they are digest-verified before saving; on push, verification is up to the remote registry.

The name of the aimage is recorded in the root `index.json`.

### ollama

[ollama.com/library](http://ollama.com/library) is mostly compliant with OCI, functioning more as a subset. Clowder can pull model components from ollama.com registry using OCI distribution protocol. It is expected that, over time, the ollama protocol either will expand to meet OCI, or will diverge.

Blobs are replicated 1:1 between the local cache in `blobs/sha256/<digest>` and the remote registry as blobs. On pull, they are digest-verified before saving; on push, verification is up to the remote registry.

The name of the aimage is recorded in the root `index.json`.

Local storage does <u>not</u> replicate how ollama does local storage, which is similar to OCI v1-layout but not identical. Clowder is true to the OCI v1 layout spec.

### HuggingFace
HuggingFace has both git (deprecated) and https APIs. Clowder constructs the native https URL for any given resource and downloads it.

Note that huggingface does not appear to publish the API explicitly, but rather infer it from the source to their SDK. Local storage will follow the Clowder storage format, rather than the huggingface format.

Local storage does <u>not</u> replicate how huggingface library does local storage. It is built around the model, rather than content addressable, which can lead to significant duplication.


## Future Considerations

Using the root `index.json` file only scales moderately, and requires careful management
to ensure no race conditions. To avoid these issues, some software, like
[containerd](https://github.com/containerd/containerd) uses a boltdb to map names to root
manifests, while keeping the OCI v1 layout spec for the actual storage;
we may consider the same.
