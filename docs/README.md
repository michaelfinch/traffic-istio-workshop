# Overview
The goal of this workshop is to gain hands-on experience with some common Istio concepts and tooling. It is a self-guided process that you can work on individually or in pairs. All of the steps are run on your local machine.

During this workshop you will:
* Use [kind](https://kind.sigs.k8s.io/) to install a kubernetes cluster locally.
* Install [Istio](https://istio.io/) within a kubernetes cluster.
* Use [Helm](https://helm.sh/) charts to create and manage Istio resources.
* Use [ArgoCD](https://argo-cd.readthedocs.io/en/stable/) to easily apply Helm charts.

You will complete a series of exercises that build upon one another and have you:
* Send traffic between two kubernetes services.
* Create two versions of a kubernetes service, and control the % of traffic that goes to each one.
* Setup an auth policy to allow/deny traffic from one service to another.
* Create two kubernetes clusters and send traffic between them using gateways.

If you have significant Istio experience and are already familiar with all of this then feel free to pair with someone who has less experience.

## Exercise 1: Istio
In this exercise we will start a local kubernetes cluster, install istio, deploy two kubernetes services, and test sending traffic between the services via istio.

1. Checkout the [traffic-istio-workshop](https://github.com/michaelfinch/traffic-istio-workshop/tree/main) repo in the ~/Development directory on your laptop. This repo contains files we will use in this exercise.
```
cd ~/Development
git clone git@github.com:michaelfinch/traffic-istio-workshop.git
```
1. Install [kind](https://kind.sigs.k8s.io/). kind is a tool for running local kubernetes clusters using Docker.
```
brew install kind
```
1. Install [istoctl](https://istio.io/latest/docs/reference/commands/istioctl/). istioctl is a tool for interacting with Istio.
```
brew install istioctl
```
1. Create a kubernetes cluster with kind.
```
kind create cluster --name exercise1
```
    1. Verify the cluster exists.
```
kind get clusters
```
    1. Verify kubernetes is running in the cluster.
```
kubectl cluster-info --context kind-exercise1
```
The output should look something like this
```
Kubernetes control plane is running at https://127.0.0.1:54508
CoreDNS is running at https://127.0.0.1:54508/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```
> **_NOTE:_**  kind will automatically set your kubernetes context to kind-exercise1. You can confirm this by running `kubectl config current-context`. Regardless, we will always manually specify the context for every command in this workshop to ensure we're using the correct one.
1. Install istio in the kubernetes cluster.
```
istioctl --context kind-exercise1 install --set profile=demo -y
```
The demo profile allows us to test out a bunch of istio functionality without having to manually enable each component.
    1. Verify the istio components are running.
```
kubectl --context kind-exercise1 get pods -n istio-system
```
    1. Enable automatic istio-proxy sidecar injection.
```
kubectl --context kind-exercise1 label namespace default istio-injection=enabled
```

## Exercise 2: Helm

## Exercise 3: ArgoCD

## Exercise 4: Multicluster Communication
