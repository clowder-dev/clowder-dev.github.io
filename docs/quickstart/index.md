---
id: quickstart
slug: /quickstart
title: Quick Start
sidebar_position: 1
---

Welcome to the Clowder quickstart guide.

## Requirements

What do you need to run Clowder?

Clowder runs on top of Kubernetes, so you will need a Kubernetes cluster. You can use any Kubernetes provider, such as:

- [Official Kubernetes](https://kubernetes.io/)
- [Minikube](https://minikube.sigs.k8s.io/docs/)
- [Kind](https://kind.sigs.k8s.io/)
- [K3s](https://k3s.io/)
- [OpenShift](https://www.openshift.com/)

As well as managed Kubernetes services, such as:

- [Amazon EKS](https://aws.amazon.com/eks/)
- [Google GKE](https://cloud.google.com/kubernetes-engine)
- [Azure AKS](https://azure.microsoft.com/en-us/services/kubernetes-service/)

Because Clowder runs inference on AI models, you will want hardware to run your model. That can be both a CPU and a GPU as well
as dedicated inference processors.

If you want to run on an [0xide sled](https://oxide.computer/), you will need to deploy a Kubernetes cluster to it.
The good folks at Ainekko have created
[a single-binary Kubernetes installer and controller for 0xide sleds](https://github.com/aifoundry-org/oxide-controller).

## Deployment

Once you have your Kubernetes cluster up and running, and credentials to access it,
deploy Clowder to the Kubernetes cluster.

You can deploy Clowder using either `helm` or `kubectl`. The recommended way is to use `helm`, but you can also use `kubectl` directly.

### With helm

The easiest way to install Clowder is using [helm](https://helm.sh/) package manager:

```sh
helm install clowder oci://docker.io/aifoundryorg/clowder
```

### With kubectl

Otherwise you can use kubernetes cli directly. For this to work you should clone
this repository first:

```sh
git clone https://github.com/clowder-dev/clowder.git
cd clowder
```

Then deploy all the parts with a single command:

```sh
kubectl apply -f k8s/
```

## Local Access

The Clowder API is exposed using a single Kubernetes `Service`. If you want it exposed to your local
machine, use `kubectl port-forward`:

```sh
# Start this and leave running in a separate terminal window.
kubectl port-forward svc/nekko-lb-svc 3090:3090
```

## Use it!

For our examples, we will assumg you have forwarded the Clowder API to your local machine on port `3090`, per the above instructions.
If you are running it elsewhere, you will need to adjust the API URL accordingly.

Let's download a model and run inference on it. For access, we need the authentication token for the Clowder API.
Unless configured otherwise, the default token is `nekko-admin-token`. Since we did a default quickstart installation, we can use this token to
access the API.

First, download the model. We will use the [SmolLM2-135M-Instruct](https://huggingface.co/unsloth/SmolLM2-135M-Instruct-GGUF) model from Hugging Face.

Let's get a list of physical nodes to decide on which node to deploy our model with runtime:

```sh
kubectl get nodes
NAME                                 STATUS   ROLES                       AGE   VERSION
ainekko-control-plane-0-10ahb6ro     Ready    control-plane,etcd,master   85m   v1.32.4+k3s1
ainekko-worker-1747986577-r7qql3ie   Ready    <none>                      85m   v1.32.4+k3s1
```

Mark that the worker node is called `ainekko-worker-1747986577-r7qql3ie`. We will use this node to deploy our model.
In the future, you will be able to:

* use the Clowder API to get a list of nodes
* select a node based on its labels
* select a node based on its capabilities, such as GPU or CPU
* tell Clowder to automatically select a node for you

For now, we will use the node name directly.

```sh
curl -H "Authorization: Bearer nekko-admin-token" \
  -X POST \
  --data '{"modelUrl": "hf:///unsloth//SmolLM2-135M-Instruct-GGUF/SmolLM2-135M-Instruct-Q4_K_M.gguf", "modelAlias": "smol", "nodeName": "ainekko-worker-1747986577-r7qql3ie", "credentials": "YOUR_HUGGING_FACE_TOKEN"}' \
  -i \
  http://localhost:3090/api/v1/workers
```

This downloads the model and starts a worker runtime pod, let's check:

```sh
curl -H "Authorization: Bearer nekko-admin-token" http://localhost:3090/api/v1/workers/list
```

This gives the result:

```json
{"count":1,"status":"success","workers":[{"name":"","model_name":"smol","model_alias":"smol"}]}
```

We can now use `http://localhost:3090/v1/chat_completions` as we would any
OpenAI API compatible chat completions endpoint.

### Web UI

By default, Open WebUI client app is deployed on the cluster.

Expose it locally with:

```sh
# Start this and leave it running in a separate terminal window.
kubectl port-forward svc/open-webui 4080:4080
```

Now we can open the UI at [http://localhost:4080](http://localhost:4080), select the model
and have a chat.




