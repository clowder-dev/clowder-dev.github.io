---
id: clowder-concepts
slug: /docs/clowder/concepts
title: Clowder Core Concepts
sidebar_position: 3
---

# Clowder AI Concepts

The following are the core concepts behind the Clowder platform. They reflect how Clowder views the world of deployable, manageable, operable AI models.

## Model

A model is precisely what it means in general AI. Models can be modified or original, base or fine-tuned, open source or closed, proprietary or third-party.

Models are **pure**.

They do not include any kind of runtime engine or other executables. Models are analogous to data, and, in fact, really only are just data: architectural information and parameters. The parameters are the tensors for each layer: weight matrix and vector bias information, all of which is data. There are no executable binaries.

## External Parameter (EP) Layers

EP Layers are modifiers that are “layered” on top of a base model to modify, or fine-tune, it for runtime. For more information, read [our guide on fine-tuning](./fine-tuning.md).

## Configuration
Configuration is runtime information for running a model. It may include, among other things:

* Prompts to prepend to each query
* Filtering
* Security processing
* Webhooks to call
* Properties for storing queries and responses
* Deployment priority, for example, must it stay in memory and prevent other model from loading, or can it be preempted? This may fit better in a deployment.
* etc.

## Manifest
Manifest is a json file that contains references to a model in one or more model layers, zero or more EP Layers, and a configuration file. This single manifest represents the entire content of a single distributable and runnable AI model, an AImage; see the next section.

Two or more manifests can refer to the same model layer, adapter layer, or configuration.

## AImage

An aimage is the ultimate AI resource, the atomic unit of AI deployment and operation in Clowder. It is a deployable artifact that contains all the information needed to run a model, including its layers, configuration, and manifest.

An aimage is a runnable model. It is a combination of:

* 1 or more Model layers - required
* 0 or more EP layers - optional
* 1 Configuration - required, although it can be empty
* 1 Manifest - required, referenced by the aimage name, contains references to the model, layers and configuration

For example:
* Simplest aimage is `llama3.1:7B`, no layers and empty configuration
* Three distinct deployments of `llama3.1:7B`:
  * Base model, no EP layers, no configuration
  * Base model, 1 EP layer, configuration defining that it must run on nodes with specific GPUs
  * Base model, 2 EP layers, configuration setting prompts that must precede every query, and restricting access to certain user groups

An aimage is the basic <u>potentially</u> deployable artifact.

![AImage](./image/aimage.png)

## Deployment
A deployment is a running instance of an aimage. It is run by deploying it in line with its configuration, and is capable of responding to queries, i.e. performing inference. Each aimage can be deployed 0, 1 or many times. As long as it is deployed, it can respond to queries.

![Deployment](./image/deployment.png)

## Methods
Methods are functions, or queries, against a specific deployment, using API calls, for example, calls to a chat completion API. Methods are applied against _deployments_, not _models_. It is the deployed aimage that contains configuration controlling how methods are executed against a running model.

Note that the lower-level can be thought of as a special case of the higher-level. One can define an application that has no plugins, no context window, no history, no security controls, which means all queries are directly against the model.

From Clowder's perspective, every method call is simply an API call against a deployment, i.e.
a deployed AImage.

![Methods](./image/methods.png)

## Middleware
Middleware is specific software that sits between the user's inference API call and the model. It is transparent
to the user. The input to middleware is the user's API call, which performs some functions and then passes the API call, potentially modified, on to the model. The middleware receives the model's response, potentially performs further activities, and sends the response to the user.

Because middleware is transparent to the user, it can be stacked, or layered, one in front of the other.

Examples of middleware are:

* call logging
* auditing
* security checks
* language filters

## Registry
A registry is any location that does all of:

1. Stores one or more AImage or its components: manifest, configuration, model layers, EP layers.
1. Exposes access to the AImage and/or its components via a well-known API.

![Registry](./image/registry.png)

## Distribution
Distribution is the process of sending components of an AImage - model, EP layers, configuration and manifest - from one location to another, where at least one of the locations is a registry.

![Distribution](./image/distribution.png)

## Node
A node is an independent device running an operating system, whether bare metal, virtualized or containerized, that can run software.

Clowder recognizes different node roles. An actual node can fulfill any combination of one, two or all of the roles simultaneously. In the simplest deployment, a single node performs all of the roles. The choice of how to colocate them is a deployment-time choice.

Details of how the nodes interact is available in [Architecture](./architecture.md).

### Worker
Stateless node where actual inference, fine-tuning and training are run. Normally this is a node with specialized hardware for performing model calculations, but it does not have to be.

### Storage
Node that provides storage services to worker nodes. Worker nodes can be configured to pull aimages directly from remote registries, or can be restricted to functioning only against a local storage node.

For all intents and purposes, a storage node is a registry, as defined in this document.

### Controller
Node that provides operational API services to users, manages all other node types, distributes workloads to worker nodes and handles all management operations.

### Load Balancer
Node that provides load-balanced runtime API services to users. Whether doing simple inference or querying applications, these nodes provide the endpoint.

![Nodes](./image/nodes.png)

## Runtime Engine

A runtime engine is low-level software that is responsible for:

1. Taking a locally-available model in a format supported by that runtime engine
1. Loading the model into memory, normally of a GPU or similar specialized processor
1. Loading EP layers, if any, into memory
1. Performing inference requests against the model

The runtime engine does not handle:

* Retrieving the model or EP layers from remote storage to local disk
* Setting up additional prompts or applications around the model
* Exposing useful APIs to users
* Any operational functionality

The runtime engine is a low-level component that is responsible for the actual inference, and nothing else.

### Engines in use

The following runtime engines are in use in Clowder:

* [llama.cpp](https://github.com/ggerganov/llama.cpp) - for LLMs

 The following engines are broadly used and are under consideration in Clowder:

* [whisper.cpp](https://github.com/ggerganov/whisper.cpp) - for audio transcription
* [PyTorch](https://pytorch.org/) - for general AI models
* [TensorFlow Runtime](https://github.com/tensorflow/runtime) - for general AI models
* [TensorRT](https://developer.nvidia.com/tensorrt) - for Nvidia GPUs

## Storage Manager
A storage manager is a software component that is responsible for managing the local storage of AImages and their components on a worker node. Since worker nodes can have multiple deployments
of the same model layers, which are very large, the storage manager is responsible for ensuring that the storage is used efficiently and that the model layers are not duplicated unnecessarily,
and then provided to any deployment locally.
