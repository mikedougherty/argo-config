# ArgoCD Bootstrapping

_This document assumes an understanding of ArgoCD, its various objects & resources, and how the system operates to apply/synchronize cluster state. For more information, see the [official docs](https://argoproj.github.io/argo-cd/)._

The contents of this directory are used to set up ArgoCD, and maintain its configuration after being set up.

## Operating ArgoCD

### Initial Setup

To set up ArgoCD on a new k8s cluster:

1. Create the argocd namespace:
    ```
    kubectl apply -f bootstrap/argocd-ns.yaml
    ```
1. Install the basic argocd application (from the offical installation resource):
    ```
    kubectl -n argocd apply -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
    ```
1. Make sure you have an SSH key added to this repository as a deploy key. Put the private key into a kubernetes secret:
    ```
    cp my-key-file.id_rsa sshPrivateKey
    kubectl -n argocd create secret generic github-key --from-file=sshPrivateKey
    rm sshPrivateKey
    ```
1. Create the AppProject object for the bootstrap Application:
    ```
    kubectl -n argocd apply -f bootstrap/argocd-project.yaml
    ```
1. Create the bootstrap Application:
    ```
    kubectl -n argocd apply -f bootstrap/bootstrap-app.yaml
    ```
    Once ArgoCD discovers this object, it will self-configure all remaining components.

### Making Changes

From now on, configuring ArgoCD is the same as any other Argo-controlled application.
Make your changes to `bootstrap/argocd-app.yaml`, commit, and push to your branch.
Changes in 'master' will be picked up automatically.

## Components

### argocd-ns.yaml

The namespace (argocd) that argo does all of its work in.

### argocd-project.yaml

The argoproj.io/AppProject object that the self-managing argoproj.io/Application objects will be a part of.
This object configures the source repositories that further content can be pulled from and the namespaces that can be deployed to.
In this case, we allow content from this repository, and the official Argo Project helm repository. Only the `argocd` namespace is deployed to.

### bootstrap-app.yaml

This argoproj.io/Application is used to keep ArgoCD itself configured properly. It maintains the state defined in _this_ directory. This application is a part of the argocd AppProject defined by `argocd-project.yaml`.

### argocd-app.yaml

This argoproj.io/Application defines the Helm chart deployment of ArgoCD itself. This application is a part of the argocd AppProject defined by `argocd-project.yaml`.

Changes to ArgoCD's configuration should be made here. Primary configuration details include:
  - What repositories ArgoCD can pull from and what credentials should be used for them (for example, the `github-key` secret from the initial setup guide)
  - Ingress settings

### argo-config-app.yaml

This argoproj.io/Application is used to pull in all other AppProject and Application objects defined elsewhere in this repository. This application is a part of the argocd AppProject defined by `argocd-project.yaml`.

For more information on how this is used to make our cluster definitions extensible and scalable, see projects/README.md
