# kind-argocd-bootstrap

In this project you will find an example of how to create and bootstrap and KinD cluster with ArgoCD.

Leveraging the ArgoCD concept of [App of Apps](https://argoproj.github.io/argo-cd/operator-manual/declarative-setup/#app-of-apps),
you will be able to install a number of kubernetes manifests quickly, safely and repeatedly.


## Project structure

The project structure should be similar to this:

```bash
.
├── README.md
├── applications
│   ├── istio-app
│   ├── istio-operator-app
│   └── master-app
├── argocd-bootstrap
│   ├── kustomization.yaml
│   ├── master-app.yaml
│   ├── repositories
│   └── resource.customizations
└── resources
    └── cluster.yaml
``` 

The `resources` folder contains the specifications for an kind Cluster.
The `argocd-bootstrap` folder contains the `kustomization` required to install ArgoCD and bootstrap the cluster. It also
contains the `master-app.yaml` file that is nothing else than the app-of-apps responsible for installing all the _other_ kubernetes manifests.
`applications` is the folder that contains the `master-app` itself plus all the other apps that you might want to install on your cluster.

## Creating the cluster

In order to create the cluster run:

`kind create cluster --config=resources/cluster.yaml`

## Bootstrap the cluster

`kubectl apply -k argocd-bootstrap`

If you then want to access ArgoCD via a load balancer you have to then issue:

`kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'`
