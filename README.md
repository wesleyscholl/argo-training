# Getting Started With Argo Workflows

## Overview

### What is Argo Workflows?

Argo Workflows is an open source container-native workflow engine for orchestrating parallel jobs on Kubernetes. Argo Workflows is implemented as a Kubernetes CRD (Custom Resource Definition).

### Argo Workflows Basic Concepts - Templates

Argo Workflows uses templates to define the structure of a workflow. Templates are reusable and can be shared across workflows. Templates can be of different types, such as container, steps, DAG, script, resource, suspend, and HTTP. Argo Workflows are written in YAML format.

### Workflow

A workflow is a sequence of steps where each step is a container.

### Template Types

- **Container**: A container template that uses a Docker image to run a container.

- **Steps**: Steps are a collection of templates that run in sequence or in parallel. Use case: Run a set of tasks in sequence or in parallel. 

- **DAG**: A directed acyclic graph (DAG) is a collection of tasks with the restriction that it has no cycles. A DAG template is a collection of tasks that can run in parallel or sequentially.

- **Script**: A script template is a pod that runs an inline script, including bash, python, perl, ruby, etc.

- **Resource**: A resource template can interact with Kubernetes resources, such as creating, deleting, or updating resources.

- **Suspend**: A suspend template is a special template that suspends a workflow until a specified time or until it is manually resumed.

- **HTTP**: An HTTP template is a pod that runs an HTTP request.

### Argo Workflows Template Example

```yaml
apiVersion: argoproj.io/v1alpha1
kind: WorkflowTemplate # Kind of resource
metadata:
  name: template-types # Name of the workflow template
spec:
    entrypoint: main # Template entrypoint of the workflow
    templates:
    - name: main
      steps: # Run each of the templates in step sequence
      - - name: run-dag
          template: print-dag
      - - name: run-suspend
          template: run-suspend
      - - name: run-container
          template: whalesay
        - name: run-script # Runs in parallel with run-container
          template: run-script
      - - name: run-http
          template: run-http
      - - name: run-resource
          template: run-resource

    - name: run-http
      http: # Run an HTTP request
        url: https://google.com
        method: GET # HTTP method
        successCondition: "response.body contains \"google\"" # If the response body contains "google"

    - name: run-suspend # Suspend the workflow for 10 seconds
      suspend: # Can be resumed manually
        duration: "10s" # Duration to suspend

    - name: run-resource # Run a Kubernetes resource
      resource:
        action: create # Create a resource
        manifest: | # Manifest file to create a workflow
          apiVersion: argoproj.io/v1alpha1
          kind: Workflow # Kind of resource
          metadata:
            generateName: new-workflow- # Generate a name
          spec:
            entrypoint: main # Entry point of the workflow
            templates:
              - name: main
                container:
                  image: argoproj/argosay:v2 # Docker image
        setOwnerReference: true # Set owner reference
      serviceAccountName: argo # Service account name

    - name: run-script
      script:
        # Run a python script
        image: python:alpine3.6
        command: [python]
        source: | # Inline python script - source code
            print("hello world")

    - name: print-dag
      dag:
        tasks:
        - name: A
          template: whalesay # Run in parallel with B and C
        - name: B
          template: whalesay # Run in parallel with A and C
        - name: C
          template: whalesay # Run in parallel with A and B
        - name: D
          dependencies: [A, B, C] # D runs after A and B and C
          template: whalesay

    - name: whalesay
      container: # Run a container
        image: docker/whalesay:latest # Docker image
        command: [cowsay] # Command to run
        args: ["hello world"] # Command line arguments
```

