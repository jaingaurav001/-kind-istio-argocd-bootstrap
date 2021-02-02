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

`kubectl apply -k argocd-bootstrap/argocd-istio-bootstrap/`

NOTE: If you get an error like: error: unable to recognize "argocd-bootstrap/argocd-istio-bootstrap": no matches for kind "Application" in version "argoproj.io/v1alpha1” just run the command again, this happens because you’ve just installed the CRD Application and trying to install an instance of it.

## Access the ArgoCD
 
If you then want to access ArgoCD via a load balancer you have to then issue:

The configuration profile will set the ingress type to LoadBalancer, which is not working on a local cluster.

For the ingress gateway to accept incoming connections we have to change the type from LoadBalancer to NodePort and change the assigned port to 32000 (the port we forwarded during the cluster creation).

Apply the patch which does the above:
`kubectl patch service istio-ingressgateway -n istio-system --patch "$(cat resources/patch-ingressgateway-nodeport.yaml)"`

Edit the /etc/hosts file so to have something like:
`##
# Host Database
#
# localhost is used to configure the loopback interface
# when the system is booting.  Do not change this entry.
##

[...]
127.0.0.1 argocd.kube
[...]`

Then try to access in your browser http://argocd.kube. It should open the login page of ArgoCD.

## Get the password of your running Argo CD:
`kubectl get pods \
  -n argocd \
  -l app.kubernetes.io/name=argocd-server \
  -o "jsonpath={.items[0].metadata.name}"`