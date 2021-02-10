# Jump App Gitops

## Introduction

*Jump App GitOps* is one of a set of repositories developed to generate a microservice based application, named _Jump App_. This repository includes an automated way of deploy _Jump App's_ microservices and all stuff around this application (Deployments, Services, Build Configs, Pipelines, etc), including optionally CI/CD or Service Mesh objects as well. 

As probably known, this automated "tool" is based on helm and tries to integrate the following solutions:

- Red Hat Openshift Container Platform 4 (\*Kubernetes)
- Multi programing language microservices (Javascript, Golang, Python and Java)
- GitOps solution, based on ArgoCD.
- CI/CD strategy, based Tekton.
- Service Mesh architecture, based on Istio.

This repository was created to include all automated procedures to achieve the following goals:

- Create required namespaces in Kubernetes (gitops-argocd, istio-system, jump-app-cicd, jump-app-dev, jump-app-pre and jump-app-pro)
- Install required ArgoCD objects (ArgoCD Server, Route, Rolebindings and Applications)
- Deploy CI/CD objects in jump-app-cicd namespace (Imagestreams, BuildConfigs, Tekton Pipelines, etc)
- Deploy _Jump App's_ microservices in each environment/namespace
- Create Service Mesh objects when Istio support is enabled.

_NOTE:_ It is important to know that it is possible to activate/deactivate features through variable _enabled_ defined for each sub-chart in the global _values.yaml_ file.

## Requisites

In order to start working with this repository, it is required:

- A Red Hat Openshift Container Platform Cluster +4.5
- ArgoCD Operator installed
- Red Hat Openshift Pipelines Operator installed
- Helm client installed in the local machine (Please follow https://helm.sh/docs/intro/install/ for more information)

Optional requirements when Service Mesh is activated:

- Red Hat Openshift Service Mesh Operator installed
- Kiali Operator installed provided by Red Hat
- Red Hat Openshift Jager Operator installed
- Service Mesh Control Plane Object with default gateways configured (*Please, find object examples in examples folder*)
- Service Mesh Member Roll Object configured (*Please, find object examples in examples folder*)

IMPORTANT: Quick Start section (_setup.sh_ script) include the procedure to install the previous requirements automatically with the exception of *Red Hat Openshift Container Platform Cluster +4.5* and *Helm* in your local machine.

## Multi Branch

This repository has a set of branches in order to manage different environments configuration files in ArgoCD. If you want to modify default values, it is required access to the specific branch, modify values.yaml file and push the file to the git repository.
 
- feature/jump-app-cicd -> Tekton chart with CI/CD configuration
- feature/jump-app-dev -> Jump App chart with DEV environment configuration
- feature/jump-app-pre -> Jump App chart with PRE environment configuration
- feature/jump-app-pro -> Jump App chart with PRO environment configuration

## Quick Start

_Jump App_ architecture contains three environments (dev, pre and pro) where the application is deployed automatically. If the priority is making use of this solution with three environments and not waste any time, the following procedure install _Jump App_ and configure CI/CD and GitOps solutions automatically:

- Download submodules

```$bash
git submodule update --remote
```

- Modify _appsDomain_ parameter (*When it is required)

```$bash
sh scripts/extra/update_charts_domain.sh apps.mydomain.com 
```

- Execute _setup.sh_ script for installing Operator

```$bash
oc login
sh ./scripts/setup.sh
```

**NOTE**: It is possible to deploy Red Hat Service Mesh solution passing the following parameter to _setup.sh_ script:

```$bash
sh ./scripts/setup.sh --servicemesh
```

**IMPORTANT**: By default, some namespaces will be created (_gitops-argocd_, _istio-system_, _jump-app-cicd_, _jump-app-dev_, _jump-app-pre_ and _jump-app-pro_). If it is required to modify their names, take special attention to modify associated variables and define the new names correctly.

### Custom Installation

When it is required to modify Jump App environments in order to avoid some environments, for example, it is required to modify *scripts/files/values-argocd.yaml* file in order to specify this requirement.


#### E.g. Deploy ArgoCD and CI/CD elements

```$bash
vi scripts/files/values-argocd.yaml

##
# Jump App ArgoCD Chart values
##

# Helm Repo GIT
helmRepoUrl: https://github.com/acidonper/jump-app-gitops.git

# ArgoCD apps definition
apps:
  jump-app-cicd:
    branch: feature/jump-app-cicd 
    enabled: true
  jump-app-pro:
    branch: feature/jump-app-pro
    enabled: false
  jump-app-pre:
    branch: feature/jump-app-pre
    enabled: false
  jump-app-dev:
    branch: feature/jump-app-dev 
    enabled: false
...
```

```$bash
oc login
sh ./scripts/setup.sh
```

#### E.g. Deploy ArgoCD, CI/CD elements and DEV environment

```$bash
vi scripts/files/values-argocd.yaml

##
# Jump App ArgoCD Chart values
##

# Helm Repo GIT
helmRepoUrl: https://github.com/acidonper/jump-app-gitops.git

# ArgoCD apps definition
apps:
  jump-app-cicd:
    branch: feature/jump-app-cicd 
    enabled: true
  jump-app-pro:
    branch: feature/jump-app-pro
    enabled: false
  jump-app-pre:
    branch: feature/jump-app-pre
    enabled: false
  jump-app-dev:
    branch: feature/jump-app-dev 
    enabled: true
...
```

```$bash
oc login
sh ./scripts/setup.sh
```

## ArgoCD

### Access Console

Once ArgoCD Server is installed, it is possible access ArgoCD Web UI follow next procedure:

- Obtain admin password and ArgoCD Server URL

```$bash
oc login
oc get secret argocd-cluster -o jsonpath='{.data.admin\.password}' -n gitops-argocd | base64 -d
oc get route argocd-server -n gitops-argocd
argocd-gitops-argocd.apps.<mydomain>
```

### ArgoCD CLI

- Auth ArgoCD Server using CLI

```$bash
oc port-forward service/argocd-server 8888:443
argocd login 127.0.0.1:8888 --username admin --password xxxx
```

- List ArgoCD Apps

```$bash
argocd app list
```

- Sync ArgoCD Apps current state

```$bash
argocd app sync jump-app-dev
```

## Charts Tests

### Jump App ArgoCD

```$bash
helm template ./charts/jump-app-argocd -f examples/local/values-jump-app-argocd.yaml --debug --namespace gitops-argocd
```

### Jump App CI/CD

```$bash
helm template ./charts/jump-app-cicd -f examples/local/values-jump-app-cicd.yaml --namespace jump-app-cicd
```

### Jump App

- DEV

```$bash
helm template ./charts/jump-app-micros -f examples/local/values-jump-app-dev.yaml --namespace jump-app-dev
```

## Author Information

Asier Cidon

asier.cidon@gmail.com
