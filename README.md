# Tekton Pipelines Demo

This repository contains a demonstration of Tekton Pipelines, showcasing various Tekton tasks, pipelines, and associated configurations.

## Table of Contents
- [Introduction](#introduction)
- [Setup](#setup)
- [Files and Configuration](#files-and-configuration)
  - [Event Listener](#event-listener)
  - [Goodbye World Task](#goodbye-world-task)
  - [Hello Goodbye Pipeline](#hello-goodbye-pipeline)
  - [Hello World Task](#hello-world-task)
  - [RBAC Configuration](#rbac-configuration)
- [Running the Pipeline](#running-the-pipeline)
- [Contributing](#contributing)
- [License](#license)

## Introduction

This project demonstrates how to use Tekton Pipelines to orchestrate tasks in a Kubernetes environment. The example includes tasks for saying "Hello" and "Goodbye" and a pipeline that runs these tasks sequentially.

## Setup

To set up and run this demo, ensure you have a Kubernetes cluster and Tekton Pipelines installed. Follow these steps to apply the configurations:

1. Apply the RBAC configuration:
   ```bash
   kubectl apply -f rbac.yaml
   
2. Apply the tasks and pipeline configurations:
```bash
kubectl apply -f hello-world.yaml
kubectl apply -f goodbye-world.yaml
kubectl apply -f hello-goodbye-pipeline.yaml
```
3. Apply the EventListener:
```bash
kubectl apply -f event-listener.yaml
```
## Files and Configuration

### Event Listener
```YAML
apiVersion: triggers.tekton.dev/v1beta1
kind: EventListener
metadata:
  name: hello-listener
spec:
  serviceAccountName: tekton-robot
  triggers:
    - name: hello-trigger 
      bindings:
      - ref: hello-binding
      template:
        ref: hello-template
```
The EventListener is responsible for triggering the pipeline run based on certain events.
![Alt text](https://tekton.dev/images/TriggerFlow.svg)

### Goodbye World Task

```yaml
apiVersion: tekton.dev/v1beta1
kind: Task
metadata:
  name: goodbye
spec:
  params:
  - name: username
    type: string
  steps:
    - name: goodbye
      image: ubuntu
      script: |
        #!/bin/bash
        echo "Goodbye $(params.username)!"
```
The Goodbye World Task prints a goodbye message.

### Hello Goodbye Pipeline
```yaml
apiVersion: tekton.dev/v1beta1
kind: Pipeline
metadata:
  name: hello-goodbye
spec:
  params:
  - name: username
    type: string
  tasks:
    - name: hello
      taskRef:
        name: hello
    - name: goodbye
      runAfter:
        - hello
      taskRef:
        name: goodbye
      params:
      - name: username
        value: $(params.username)
```
The Hello Goodbye Pipeline runs the "hello" task followed by the "goodbye" task.

### Hello World Task
```yaml
apiVersion: tekton.dev/v1beta1
kind: Task
metadata:
  name: hello
spec:
  steps:
    - name: echo
      image: alpine
      script: |
        #!/bin/sh
        echo "Hello World"
```
The Hello World Task prints a hello message.
### RBAC Configuration
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: tekton-robot
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: triggers-example-eventlistener-binding
subjects:
- kind: ServiceAccount
  name: tekton-robot
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: tekton-triggers-eventlistener-roles
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: triggers-example-eventlistener-clusterbinding
subjects:
- kind: ServiceAccount
  name: tekton-robot
  namespace: default
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: tekton-triggers-eventlistener-clusterroles
```
This configuration sets up the necessary role bindings and service account for Tekton to run the pipelines.

### Running the Pipeline
To run the pipeline, create a PipelineRun resource or trigger it through the EventListener:
```yaml
apiVersion: tekton.dev/v1beta1
kind: PipelineRun
metadata:
  name: hello-goodbye-run
spec:
  pipelineRef:
    name: hello-goodbye
  params:
  - name: username
    value: "Tekton"
```
Apply the PipelineRun:

```bash
kubectl apply -f hello-goodbye-pipeline-run.yaml
```
### Contributing
Contributions are welcome! Please submit a pull request or open an issue to discuss your idea.

### License
This project is licensed under the MIT License.

```Code
You can further customize this README file as needed. Would you like me to commit this README file to your repository?
```
