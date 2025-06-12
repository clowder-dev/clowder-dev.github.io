---
id: clowder-storage
slug: /docs/clowder/storage
title: Aimage and Model Storage
sidebar_position: 7
---

Inference and finetuning require access to, and sometimes generation of, artifacts,
such as models and adapters.

These artifacts must be stored in a way that is accessible to consumers of the artifacts, specifically:

* runtime engines on worker nodes
* users for outputs

In addition to the usual requirements of storage for computing,
AI artifacts tend to be quite large, sometimes gigabytes or even tens of gigabytes in size.
This requires two considerations, to avoid both storage and network issues:

* Storage should try to avoid duplication.
* Storage should be reusable among processes.

## Storage Philosophy

The storage philosophy is composed of several parts.

1. External registries for user-provided artifacts
1. Streaming only for user-provided artifacts
1. Optimized local storage using OCI content-addressable storage (CAS)

### External Registries

Clowder relies on external registries to make models available. For example, when an operator wants to make a model available
to the system, they provide a reference to the model in an external registry. The clowder system will not store
a centralized local copy of the model for its system. Since clowder often is deployed within IT organizations,
artifact repositories are almost always available. Multiple pulls of the same artifacts are over low-latency
high-bandwidth LANs, and are not a concern.

In the case where no registry is available, we recommend that customers deploy their own registry,
such as [Harbor](https://goharbor.io) or [CNCF Distribution](https://github.com/distribution/distribution)
(formerly Docker Registry).

### Streaming only

Inference that does not have large artifacts that need to be uploaded, such as images or audio files, are streamed
directly from the user endpoint, via the API gateway to the inference engine on the target worker node. These are
not stored anywhere. If the worker node fails before completing the job, the inference is lost, and the user
must resubmit the entire job.

As part of this philosophy, the Clowder system does not store user context or other data, in order to create applications.
That is the responsibility of the developer _on top of_ the Clowder system, rather than inside it.

This also means that the system has no provision for storing larger artifacts, such as audio files for transacription,
or images or video for analysis.

The OpenAI API normally does this in 2 stages:

1. Upload artifact to a storage API endpoint, receive an artifact ID
1. Invoke inference using artifact ID

For Nekko, the user is expected to:

1. Upload artifact to _their own artifacts repository_.
1. Invoke inference using artifact URL.

From Nekko's perspective, access to an artifact for inference is the same as access to a model.

NOTE: This may change over time, should this be a common requirement.

### Optimized Local Storage

On each worker node where an AImage is run, the AImage is downloaded from the registry and stored locally.

These are stored locally using storage that complies with the [OCI v1 layout specification](https://github.com/opencontainers/image-spec/blob/main/image-layout.md). Specifically:

* A single directory is the primary entrypoint
* The directory has a single file called `index.json` which is a json file matching a `v1.Index`, with a manifest entry for each AImage downloaded, referencing the digest of the manifest for the AImage and an annotation for the name.
* The manifest contains references to:
  * The configuration file as a digest
  * Each layer in sequence, including its type. As of this writing, supported types are:
    * Model layer
    * EP layer
* All manifests, config and layers are stored in the directory under `blobs/sha256/` for sha256 or, if sha512 is used, `blobs/sha512`.

```
root/
root/index.json        <--- maps ollama3.2:70B to manifest 8161cf7fd23fa...
root/blobs/
root/blobs/sha256/
root/blobs/sha256/8161cf7fd23fa3facc563646085bbf19b89880f46d9c3422a8166fb382117c43  <--- manifest
root/blobs/sha256/ad5a1d7e5df19681dc93d731f89631ff1c57cc01d3cf65247084890ff9429115  <--- config
root/blobs/sha256/03886da590fdde984264a9045fb6294c6d0a1dbdf00687dca4310fc10491afb9  <--- model layer
root/blobs/sha256/caaaa06c155e12489c3cf3b02aabfeaa2826b83be8e3761dba93c04d5438b78f  <--- model layer
root/blobs/sha256/2304d6e290542f4e75fecc57e109691a06acc8850f981b96c16687e3267f25b0  <--- adapter layer
root/blobs/sha256/cfd51cb355a61905e50154c652b935e8e7043978fdd500f67ff09791ac1a9f83  <--- adapter layer
```

#### Media Types

Still to be determined are the media-types to use. There are two options:
* Use custom media types.
* Use container media types.

Custom media types comply with [OCI Artifacts](https://github.com/opencontainers/artifacts), and do a better job of describing the contents of the manifests, layers and configuration. On the other hand, some registries refuse to accept blobs and manifests with anything other than basic container media types: OCI index/docker manifest; OCI manifest/docker image; OCI/Docker config; image layers.
If possible, our strong preference is to use custom media types. Proposed are:

* `vnd.clowder.aimage.manifest.v1`
* `vnd.clowder.aimage.config.v1`
* `vnd.clowder.aimage.model.gguf.layer.v1`
* `vnd.clowder.aimage.model.onnx.layer.v1`
* `vnd.clowder.aimage.model.torchscript.layer.v1`
* `vnd.clowder.aimage.model.savedmodel.layer.v1`
* `vnd.clowder.aimage.model.transformers.layer.v1`
* `vnd.clowder.aimage.adapter.lora.layer.v1`

Note that this does not determine what format the model layers are in, and what format the adapter layers are in. Those should be part of the media type.

##### Standard types

Sample index:

```json
{
   "schemaVersion": 2,
   "manifests": [
      {
         "mediaType": "application/vnd.oci.image.manifest.v1+json",
         "size": 1522,
         "digest": "sha256:8161cf7fd23fa3facc563646085bbf19b89880f46d9c3422a8166fb382117c43",
         "annotations": {
            "io.containerd.image.namename": "nekko.ai/llama/3.1:70B",
            "org.opencontainers.image.created": "2021-04-21T11:33:57Z",
            "org.opencontainers.image.ref.name": "nekko.ai/llama/3.1:70B"
         }
      },
...
]
```

Sample manifest:

```json
{
   "mediaType": "application/vnd.oci.image.manifest.v1+json",
   "schemaVersion": 2,
   "config": {
      "mediaType": "application/vnd.oci.image.config.v1+json",
      "digest": "sha256:ad5a1d7e5df19681dc93d731f89631ff1c57cc01d3cf65247084890ff9429115",
      "size": 1930
   },
   "layers": [
      {
         "mediaType": "application/vnd.oci.image.layer.v1+gzip",
         "digest": "sha256:03886da590fdde984264a9045fb6294c6d0a1dbdf00687dca4310fc10491afb9",
         "size": 712863
      },
      {
         "mediaType": "application/vnd.oci.image.layer.v1+gzip",
         "digest": "sha256:caaaa06c155e12489c3cf3b02aabfeaa2826b83be8e3761dba93c04d5438b78f",
         "size": 791510
      },
      {
         "mediaType": "application/vnd.oci.image.layer.v1+gzip",
         "digest": "sha256:2304d6e290542f4e75fecc57e109691a06acc8850f981b96c16687e3267f25b0",
         "size": 5621574
      },
      {
         "mediaType": "application/vnd.oci.image.layer.v1+gzip",
         "digest": "sha256:cfd51cb355a61905e50154c652b935e8e7043978fdd500f67ff09791ac1a9f83",
         "size": 52099
      },
...
]
```


##### Custom Types

Sample index:

```json
{
   "schemaVersion": 2,
   "manifests": [
      {
         "mediaType": "application/vnd.clowder.aimage.manifest.v1+json",
         "size": 1522,
         "digest": "sha256:8161cf7fd23fa3facc563646085bbf19b89880f46d9c3422a8166fb382117c43",
         "annotations": {
            "io.containerd.image.namename": "nekko.ai/llama/3.1:70B",
            "org.opencontainers.image.created": "2021-04-21T11:33:57Z",
            "org.opencontainers.image.ref.name": "nekko.ai/llama/3.1:70B"
         }
      },
...
]
```

Sample manifest:

```json
{
   "mediaType": "application/vnd.clowder.aimage.manifest.v1+json",
   "schemaVersion": 2,
   "config": {
      "mediaType": "application/vnd.clowder.aimage.config.v1+json",
      "digest": "sha256:ad5a1d7e5df19681dc93d731f89631ff1c57cc01d3cf65247084890ff9429115",
      "size": 1930
   },
   "layers": [
      {
         "mediaType": "application/vnd.clowder.aimage.model.gguf.layer.v1+gzip",
         "digest": "sha256:03886da590fdde984264a9045fb6294c6d0a1dbdf00687dca4310fc10491afb9",
         "size": 712863
      },
      {
         "mediaType": "application/vnd.clowder.aimage.model.gguf.layer.v1+gzip",
         "digest": "sha256:caaaa06c155e12489c3cf3b02aabfeaa2826b83be8e3761dba93c04d5438b78f",
         "size": 791510
      },
      {
         "mediaType": "application/vnd.clowder.aimage.adapter.lora.layer.v1+gzip",
         "digest": "sha256:2304d6e290542f4e75fecc57e109691a06acc8850f981b96c16687e3267f25b0",
         "size": 5621574
      },
      {
         "mediaType": "application/vnd.clowder.aimage.adapter.lora.layer.v1+gzip",
         "digest": "sha256:cfd51cb355a61905e50154c652b935e8e7043978fdd500f67ff09791ac1a9f83",
         "size": 52099
      },
...
]
```


#### Format
From <u>distribution</u> and <u>storage</u> perspectives, Clowder does not impose any particular format. It can accept any format, provided that a media-type is available for it. For example:

| Model Format              | Media Type                                      |
|---------------------------|-------------------------------------------------|
| GGUF                      | `vnd.clowder.aimage.model.gguf.layer.v1`          |
| ONNX                      | `vnd.clowder.aimage.model.onnx.layer.v1`          |
| TorchScript               | `vnd.clowder.aimage.model.torchscript.layer.v1`   |
| SavedModel (TensorFlow)   | `vnd.clowder.aimage.model.savedmodel.layer.v1`    |
| Transformers (HuggingFace)| `vnd.clowder.aimage.model.transformers.layer.v1`  |

Since Clowder relies on `llama.cpp` as the underlying low-level runtime engine, loaded as a library, and `llama.cpp` works solely with GGUF, as of this writing, only GGUF format model layers can be run. This may change over time, due to one of: `llama.cpp` support for other formats; nekko model format conversion support; or nekko adoption of other low-level runtime engines.

## Open Questions

### Where do applications live?

Applications are run on separate infrastructure provided by the operator outside of the Nekko platform.
The Nekko platform exposes the OpenAI API, which is consumed by externally-running applications.
