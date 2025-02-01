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


### Workflow Variables and How to Use Them

Argo Workflows support variables that can be used in templates. Variables can be used to pass data between templates. Variables can be defined in the workflow spec and can be used in templates. Variables can be of different types, such as string, integer, boolean, object, array, or map.

#### How to access workflow variables in templates

To access workflow variables in templates, use the following syntax:

```yaml
{{workflow.parameters.<variable-name>}} # Access workflow parameters - From anywhere in the workflow

{{inputs.parameters.<variable-name>}} # Access input parameters - From within a template

{{outputs.parameters.<variable-name>}} # Access output parameters - From within a template
```


Steps and DAG tasks:
```yaml
{{steps.<step-name>.outputs.parameters.<variable-name>}} # Access step outputs

{{tasks.<task-name>.outputs.parameters.<variable-name>}} # Access task outputs

{{steps.<step-name>.outputs.artifacts.<artifact-name>}} # Access step artifacts

{{tasks.<task-name>.outputs.artifacts.<artifact-name>}} # Access task artifacts

{{tasks.<task-name>.outputs.result}} # Access task result

{{steps.<step-name>.outputs.result}} # Access step result
```


### Argo Workflows Variables Example

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Workflow
metadata:
  generateName: hello-world- # Generate a name
spec:
    entrypoint: main # Entry point of the workflow
    arguments: # Arguments to pass to the workflow
        parameters:
        - name: message # Name of the parameter
        value: hello world # Value of the parameter
    templates:
    - name: main
        steps:
        - - name: generate-message
            template: generate-message
            arguments: # Arguments to pass to the template
            parameters:
            - name: message # Name of the parameter
                value: "{{workflow.parameters.message}}" # Use workflow parameter
        - - name: print-message
            template: print-message
            arguments: # Arguments to pass to the template
            parameters:
            - name: message # Name of the parameter
                value: "{{steps.generate-message.outputs.parameters.message}}" # Use output of generate-message
    
    - name: generate-message
        inputs: # Input parameters
        parameters:
        - name: message # Name of the parameter
        outputs: # Output parameters
        parameters:
        - name: message # Name of the parameter
            valueFrom:
            parameter: message # Use input parameter
    
    - name: print-message
        inputs: # Input parameters
        parameters:
        - name: message # Name of the parameter
        container: # Run a container
        image: alpine:3.6 # Docker image
        command: [echo] # Command to run
        args: ["{{inputs.parameters.message}}"] # Use input parameter
```

### Argo Workflows - Types of Workflow Resources

Argo Workflows support different types of workflow resources, such as:

- **Workflow**: A workflow is a sequence of steps where each step is a container.

- **CronWorkflow**: A cron workflow is a workflow that runs on a schedule.

- **WorkflowTemplate**: A workflow template is a reusable workflow that can be shared across workflows.

- **ClusterWorkflowTemplate**: A cluster workflow template is a reusable workflow that can be shared across workflows at the cluster level.

#### Use Cases

- **Workflow**: Use a workflow to run a sequence of steps or parallel jobs. Can be used for CI/CD pipelines, data processing, machine learning, etc.

- **CronWorkflow**: Use a cron workflow to run a workflow on a schedule. Can be used for periodic tasks, backups, monitoring, etc.

- **WorkflowTemplate**: Use a workflow template to define a reusable workflow. Can be used to share workflows across teams or projects within the same namespace.

- **ClusterWorkflowTemplate**: Use a cluster workflow template to define a reusable workflow at the cluster level. Can be used to share workflows across the entire cluster.


### Argo Workflows - Workflow Resource Example

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Workflow
metadata:
  generateName: workflow-example- # Generate a name
spec:
    entrypoint: main # Entry point of the workflow
    templates:
    - name: main
      container: # Run a container
        image: alpine:3.6 # Docker image
        command: [echo] # Command to run
        args: ["This is a Workflow"] # Command line arguments
```

### Argo Workflows - CronWorkflow Resource Example

```yaml
apiVersion: argoproj.io/v1alpha1
kind: CronWorkflow
metadata:
  generateName: cron-workflow-example- # Generate a name
spec:
    schedule: "*/1 * * * *" # Schedule to run every minute
    workflowSpec:
    entrypoint: main # Entry point of the workflow
    templates:
    - name: main
      container: # Run a container
        image: alpine:3.6 # Docker image
        command: [echo] # Command to run
        args: ["This is a scheduled CronWorkflow"] # Command line arguments
```

### Argo Workflows - WorkflowTemplate Resource Example

```yaml
apiVersion: argoproj.io/v1alpha1
kind: WorkflowTemplate
metadata:
  name: workflow-template-example # Name of the workflow template
spec:
    entrypoint: main # Entry point of the workflow
    templates:
    - name: main
      container: # Run a container
        image: alpine:3.6 # Docker image
        command: [echo] # Command to run
        args: ["This is a WorkflowTemplate"] # Command line arguments
```

### Argo Workflows - ClusterWorkflowTemplate Resource Example

```yaml
apiVersion: argoproj.io/v1alpha1
kind: ClusterWorkflowTemplate
metadata:
  name: cluster-workflow-template-example # Name of the cluster workflow template
spec:
    entrypoint: main # Entry point of the workflow
    templates:
    - name: main
      container: # Run a container
        image: alpine:3.6 # Docker image
        command: [echo] # Command to run
        args: ["This is a ClusterWorkflowTemplate"] # Command line arguments
```