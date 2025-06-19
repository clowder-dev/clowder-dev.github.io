---
id: clowder-api
title: Clowder API
sidebar_position: 9
---

Clowder provides a RESTful API for interacting with the platform. The API allows you to manage models, deployments, and configurations, as well as perform inference on deployed models.

There are two distinct categories of API endpoints:

* Inference
* Operational

## Inference API

The inference API is used to interact with deployed models and perform inference. It allows you to send queries to models and receive responses.

The inference API is compatible with the [OpenAI API](https://platform.openai.com/docs/api-reference/introduction) and supports the same endpoints and parameters. It also augments it with additional endpoints for usage with Clowder.

Inference API endpoints are used by end-users directly or by AI applications to interact with deployed models.

Read the official Clowder Inference API documentation for all of the endpoints.

## Operational API

The operational API is used to manage models, deployments, and configurations. It allows you to create, update, and delete models, deployments, and configurations, as well as retrieve information about them.

Operational API endpoints are used by administrators and developers to manage the Clowder platform.

Read the official Clowder Operational API documentation for all of the endpoints.


