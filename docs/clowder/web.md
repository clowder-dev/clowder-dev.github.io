---
id: clowder-web
title: Clowder Web UI
sidebar_position: 8
---
Clowder is a system for deploying, managing, and operating AI models.
As such, its focus is on the backend of managing models, configurations, and deployments, and providing APIs for interacting with these models.
These APIs usually are integrated into other systems, such as web applications or command-line interfaces (CLIs), rather than
serving as a standalone user interface.

However, by default, Clowder does include the Open WebUI client app, a web-based user interface (UI) that allows users to interact with models.

The Open WebUI is exposed at port 4080 of the service `open-webui` in the `clowder` namespace.

You can expose it locally with:

```sh
# Start this and leave it running in a separate terminal window.
kubectl port-forward svc/open-webui 4080:4080
```

Whether exposed locally or accessed directly in the cluster, open it in your browser, for example locally at [http://localhost:4080](http://localhost:4080).
Open a browser to the address, select the model, and have a chat.
