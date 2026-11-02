# Kubernetes Todo App --- Helm Charts

A Kubernetes deployment project for a Django-based Todo application,
packaged and deployed with Helm. The project demonstrates how to manage
a multi-component application on a local Kubernetes cluster, configure
workloads through Helm values, and organize application and database
resources using a parent chart and a MySQL subchart.

## Project Overview

The project deploys a Todo application and its MySQL database to a
Kubernetes cluster created with **kind**.

The application is packaged as a Helm chart named `todoapp`. MySQL is
maintained as a dependent subchart, allowing both components to be
deployed and managed as part of one Helm release.

The chart configuration is designed to keep deployment settings in
`values.yaml` files rather than hard-coding them in Kubernetes
manifests.

## Technologies

-   **Kubernetes** --- container orchestration
-   **kind** --- local Kubernetes cluster
-   **Helm** --- Kubernetes package management and templating
-   **Docker** --- container images
-   **Django / Python** --- application
-   **MySQL** --- relational database
-   **Bash** --- deployment automation

## Architecture

The deployment contains two main application components:

-   **Todo application** --- runs in Kubernetes using a Deployment and
    is exposed through a Kubernetes Service.
-   **MySQL** --- runs as a StatefulSet and uses persistent storage for
    database data.

The `todoapp` Helm chart includes the MySQL chart as a dependency. Helm
templates generate Kubernetes resources using values supplied in the
parent chart and subchart configuration.

``` text
                         kind cluster
┌──────────────────────────────────────────────────────────────┐
│                                                              │
│  ┌──────────────────────────┐     ┌────────────────────────┐  │
│  │       todoapp chart      │     │     mysql subchart     │  │
│  │                          │     │                        │  │
│  │  ┌────────────────────┐  │     │  ┌──────────────────┐  │  │
│  │  │    Deployment      │  │     │  │   StatefulSet    │  │  │
│  │  └────────────────────┘  │     │  └──────────────────┘  │  │
│  │  ┌────────────────────┐  │     │  ┌──────────────────┐  │  │
│  │  │      Service       │  │     │  │ Persistent Data  │  │  │
│  │  └────────────────────┘  │     │  └──────────────────┘  │  │
│  │  ┌────────────────────┐  │     │                        │  │
│  │  │ HPA / RBAC /       │  │     └────────────────────────┘  │
│  │  │ Service Account    │  │                                 │
│  │  └────────────────────┘  │                                 │
│  └──────────────────────────┘                                 │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

## Helm Chart Configuration

The project uses Helm templates and values files to make deployment
settings configurable.

### Todo application chart

The parent `todoapp` chart is designed to manage configuration such as:

-   Kubernetes namespace
-   Container image repository and tag
-   Secret values and environment variables
-   Resource requests and limits
-   Deployment rolling update strategy
-   Node affinity
-   Horizontal Pod Autoscaler settings
-   Persistent Volume and Persistent Volume Claim settings
-   Service account and RBAC configuration

### MySQL subchart

The MySQL subchart manages database-related resources and configuration,
including:

-   Kubernetes namespace
-   Database secrets
-   StatefulSet replica count
-   Container image repository and tag
-   Persistent Volume Claim storage request
-   Resource requests and limits
-   Node affinity and tolerations

### Chart dependency

The MySQL chart is configured as a dependency of the `todoapp` chart.
This allows Helm to manage the application and database charts together
as a single release.

## Repository Structure

The repository is organized around the cluster configuration, deployment
script, and Helm chart.

``` text
.
├── cluster.yml             # kind cluster configuration
├── bootstrap.sh            # Deployment and prerequisite setup
├── helm-chart/
│   └── todoapp/
│       ├── Chart.yaml      # Parent chart metadata and dependencies
│       ├── values.yaml     # Default application chart values
│       ├── charts/
│       │   └── mysql/      # MySQL dependency chart
│       └── templates/      # Kubernetes resource templates
├── INSTRUCTION.md          # Validation instructions
└── output.log              # Captured Kubernetes resource output
```

*The tree above illustrates the expected main components. Consult the
repository for the exact current file and template names.*

## Prerequisites

Install the following tools before deploying the project:

-   Docker
-   kubectl
-   kind
-   Helm

Make sure Docker is running and that the tools are available in your
shell:

``` bash
docker --version
kubectl version --client
kind version
helm version
```

## Deployment

### 1. Clone the repository

``` bash
git clone https://github.com/VadymMelnyk/devops_todolist_kubernetes_task_12_helm_charts.git
cd devops_todolist_kubernetes_task_12_helm_charts
```

### 2. Run the bootstrap script

The repository includes a `bootstrap.sh` script intended to automate
prerequisite setup and deployment.

``` bash
chmod +x bootstrap.sh
./bootstrap.sh
```

Review the script before running it to understand which cluster and
Kubernetes resources it creates.

### 3. Check the cluster

``` bash
kubectl cluster-info
kubectl get nodes -o wide
```

### 4. Check deployed resources

``` bash
kubectl get all,cm,secret,ing -A
```

The repository includes an `output.log` file with captured resource
information from the deployment.

## Helm Validation

You can inspect the Helm release and chart configuration with:

``` bash
helm list -A
helm dependency list ./helm-chart/todoapp
```

To validate chart rendering locally:

``` bash
helm lint ./helm-chart/todoapp
helm template todoapp ./helm-chart/todoapp
```

To inspect Kubernetes resources after deployment:

``` bash
kubectl get pods -A
kubectl get services -A
kubectl get deployments,statefulsets -A
kubectl get pvc,pv -A
kubectl get hpa -A
```

For detailed task-specific validation steps, see
[`INSTRUCTION.md`](INSTRUCTION.md).

## Key Concepts Practiced

This project provided hands-on practice with:

-   Creating and configuring a local Kubernetes cluster using kind.
-   Packaging Kubernetes manifests as reusable Helm charts.
-   Using Helm templates and `values.yaml` to parameterize Kubernetes
    resources.
-   Organizing a parent chart with a dependent MySQL subchart.
-   Deploying stateless workloads with Kubernetes Deployments.
-   Managing database workloads with StatefulSets and persistent
    storage.
-   Configuring Kubernetes Secrets and injecting secret values as
    container environment variables.
-   Defining resource requests and limits.
-   Configuring rolling updates, node affinity, and tolerations.
-   Configuring Horizontal Pod Autoscaling.
-   Managing Service Accounts and RBAC resources.
-   Automating cluster setup and application deployment with Bash.

## Useful Commands

  Purpose                              Command
  ------------------------------------ --------------------------------------------------
  List cluster nodes                   `kubectl get nodes -o wide`
  List Helm releases                   `helm list -A`
  List pods in all namespaces          `kubectl get pods -A`
  List services in all namespaces      `kubectl get services -A`
  List persistent volumes and claims   `kubectl get pv,pvc -A`
  List autoscalers                     `kubectl get hpa -A`
  View application pod logs            `kubectl logs <pod-name> -n <namespace>`
  Describe a pod                       `kubectl describe pod <pod-name> -n <namespace>`

