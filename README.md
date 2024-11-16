# FluxCD- Usage Of An Open Source GitOps Tool.

## Background
The main difference between regular DevOps and GitOps lies in how configuration management and deployment processes are handled.

In DevOps Process_

- There may be human intervention during the deployment process, and it is usually managed using CI/CD tools or manual steps.

and In GitOps Process_

- All configuration files are stored in Git, and any changes are automatically applied to the deployment system (such as FluxCD or ArgoCD) in the Kubernetes cluster.


**Does it make sense?** Let's try to understand it by an example_ 

**Case:** Imagine you have a Kubernetes cluster that hosts multiple applications, and you are responsible for managing deployments, configurations, and updates.

In a regular DevOps approach, CI/CD tools like GitHub Actions, GitLab CI, or CircleCI are used to automate parts of the deployment process. When a new change is pushed to the repository:
- The pipeline is triggered automatically (or manually, depending on configuration) whenever there is a change in the code or configuration files, such as a commit or pull request.
- The pipeline typically starts by building the application, running tests, and performing static analysis to ensure the quality of the code.
- Once the tests pass and the code is ready, the pipeline deploys the changes to the target environment (e.g., staging or production). This is usually done by applying Kubernetes manifests, Helm charts, or other deployment scripts.
- Once the pipeline finishes running, the changes are reflected in the deployed application, and the new version is live.
- You need to set credentials for the kubernetes cluster in the pipeline configurations.

What if, when you push a change to the repository and it gets deployed without using any pipeline, rather by the kubernetes itself?

In this case, GitOps is the answer. With GitOps, the process works like this:

- All configuration files (such as Kubernetes manifests, Helm charts, or application configuration) are stored in a Git repository.
- Tools like FluxCD or ArgoCD are set up to continuously monitor the Git repository. These tools act as the "automated operators" in your Kubernetes cluster.
- When a change is pushed to the Git repository (for example, a change to Kubernetes manifests or application configuration), the GitOps tool automatically detects the change.
- The GitOps tool syncs the new configuration with the Kubernetes cluster without requiring any manual steps or pipeline execution. This means the new changes are automatically deployed by Kubernetes itself once the GitOps tool updates the cluster.
- There is no need to manually configure or trigger a CI/CD pipeline. The Kubernetes cluster itself is always in sync with the configuration in the Git repository, ensuring that the desired state is automatically maintained.
- Since the GitOps tool handles the sync between the Git repository and Kubernetes, credentials are typically configured in the GitOps tool itself, not within a pipeline. 


## FluxCD Overview
FluxCD is a tool for GitOps that automatically synchronizes your Kubernetes clusters with configuration files stored in Git repositories. It is an open-source solution that enables continuous delivery by ensuring that the state of your cluster matches the configuration specified in the Git repository.


## Lets Directly Start Using FluxCD for Better Understanding

**Prerequisites:**
- Understanding of docker and kubernetes.
- Docker installed.
- A k8s cluster (docker-desktop or kind or any cloud is okay)
- Flux installed.
- kustomize installed.
- kubectx installed.
- kubectl installed.

### Step-1
As you have all these tools installed, lets create a kind cluster_
```
kind create cluster --name <cluster-name>
```

Check the cluster_
```
kind get cluster
```

To delete the cluster (not now obviously)
```
kind delete cluster --name <cluster-name>
```

### Step-2
Create a github repository, for my case the repository name is `fluxcd-best-practices` as I am going to follow the best practices for creating the complete project.

### Step-3
Generate a github access token.

### Step-4
Open terminal and set the access token in env_
```
export GITHUB_TOKEN=<github-token>
```

You can check the flux compatibility with your cluster by using the command below_
```
flux check --pre
```

Execute the flux Bootstrap Command_

```
flux bootstrap github \
  --token-auth \
  --owner=<github-username> \
  --repository=<repository-name> \
  --branch=main \
  --path=clusters/docker-desktop \
  --personal 
```
Clone the repository, you will see a project structure similar to this_

```
<!-- output structure  -->
├── clusters
│   └── docker-desktop
│       └── flux-system
│           ├── gotk-components.yaml
│           ├── gotk-sync.yaml
│           └── kustomization.yaml

```

### Step-5
Verify flux in k8s_
```
kubectl get deploy -n flux-system
```
Should see something like this_
```
NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
helm-controller           1/1     1            1           137m
kustomize-controller      1/1     1            1           137m
notification-controller   1/1     1            1           136m
source-controller         1/1     1            1           136m
```


### Lets Understand the Bootstrap Command_
- `flux bootstrap github` => means we are bootstraping for github. 
- `--path=clusters/docker-desktop` => following the best practices. I am using `docker-desktop` k8s cluster. Soon I will explain the best practices in details.
- `--personal` => for personal github account, skip for org level repo.

### Contollers_
- **helm-controller:** responsible for managing and applying Helm charts in your Kubernetes cluster.
- **kustomize-controller:** responsible for applying resources defined using Kustomize in your Git repository.
- **notification-controller:** responsible for handling notifications within the FluxCD setup.
- **source-controller:** responsible for managing and monitoring the Git repositories that store your Kubernetes resources.

### Best Practice Project Structure for FluxCD
[Ways of structuring your repositories](https://fluxcd.io/flux/guides/repository-structure/)

```
├── apps
│   ├── base
│   ├── production 
│   └── staging
├── infrastructure
│   ├── base
│   ├── production 
│   └── staging
└── clusters
    ├── production
    └── staging # in our case we have docker-desktop cluster
```

### Detailed Description of Each Section

#### 1. apps directory:
The apps directory contains resources related to applications deployed across your clusters. It is further split into:

#### 2. clusters directory:
The clusters directory holds configurations specific to different clusters. For example, the docker-desktop cluster folder holds configurations for the apps and FluxCD sync for the docker-desktop cluster.

#### 3. infrastructure directory
The infrastructure directory in the FluxCD project structure is where you define and manage resources related to the underlying infrastructure of your clusters, such as monitoring tools, ingress controllers, logging systems, and other foundational services that all applications in your cluster rely on.

- **base**: This folder contains resources that are common across all clusters. For example, you might want to deploy cert-manager or set up namespaces for all clusters. These resources are not specific to a particular cluster but are shared by all.

- **staging/production**:  This folder contains cluster-specific resources. The resources here will be applied to the `docker-desktop` cluster.


### Example_

Suppose we need to create a namespace for each cluster and the namespace name is `app-ns1`. We want to keep it in the `apps` folder. As this is common, lets create a `namespace.yaml` file in `apps/base` folder.

```
# namespace.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: app-ns1
  labels:
    app: app-ns1
```

Create another file named `kustomization.yaml` in base folder to indicate the `namespace` file.
```
resources:
- namespace.yaml
```
This will make sure, whenever any change in pushed from the `namespace.yaml` file, the `kustomization.yaml` file will trace that and will notify the cluster.

In the `apps/docker-desktop` create another file named `kustomization.yaml` and link the `apps/base` config.
```
resources:
- ../base
```

Lets create another file inside `clusters/docker-desktop` folder named `apps.yaml`
```
# Kustomization custom resource to load resource located in the app directory

apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: app
  namespace: flux-system
spec:
  interval: 30s
  path: ./apps/docker-desktop
  prune: true
  sourceRef:
    kind: GitRepository
    name: flux-system
```

Push the changes to github.


## Check
```
kubectl get kustomization
kubectl get gitrepository
```

You should see a new namespace created in the cluster.

We installed nginx from the infrastructure. Hope you will understand how its configured and work.

My overall project structure following best practices is as below_
```
├── apps
│   ├── base
│   │   ├── kustomization.yaml
│   │   └── namespace.yaml
│   └── docker-desktop
│       └── kustomization.yaml
├── clusters
│   └── docker-desktop
│       ├── apps.yaml
│       ├── flux-system
│       │   ├── gotk-components.yaml
│       │   ├── gotk-sync.yaml
│       │   └── kustomization.yaml
│       └── infrastructure.yaml
├── infrastructure
│   ├── base
│   │   ├── kustomization.yaml
│   │   ├── nginx-depl.yaml
│   │   ├── nginx-ns.yaml
│   │   └── nginx-svc.yaml
│   └── docker-desktop
│       ├── kustomization.yaml
│       └── namespace.yaml
└── README.md
```

### PLEASE REVIEW THIS GITHUB REPO FOR BETTER UNDERSTANDING OF THE CODE AND CONFIGURATIONS

---
---

### kustomize installation on ubuntu
```
curl -s "https://raw.githubusercontent.com/kubernetes-sigs/kustomize/master/hack/install_kustomize.sh"  | bash
sudo mv kustomize /usr/local/bin
```

### Use
```
kustomize build apps/docker-desktop
```

### Check live change

```
watch kubectl get ns
```