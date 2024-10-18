---
id: exercise2
title: "Exercise 2: Using Helm"
previous_page: exercise1
next_page: exercise3
---
<link rel="stylesheet" href="/assets/css/styles.css">

# Exercise 2: Using Helm

<img src="assets/images/helm_logo.svg" alt="helm logo" width="100"/>
<br />

In this exercise we will install helm, start a local kubernetes cluster, use helm to install istio, use helm to apply istio configuration, and test sending different %s of traffic to two versions of a service via istio.

## Procedure

1. Install helm.
   ```bash
   brew install helm
   ```
     - Add the istio helm repository to your helm configuration. This will allow you to install istio with helm.
       ```
       helm repo add istio https://istio-release.storage.googleapis.com/charts
       ```
1. Create a new kubernetes cluster with kind.
   ```bash
   kind create cluster --name exercise2
   ```
1. Use helm to install istio in the kubernetes cluster.
     - Create a namespace for istio.
       ```bash
       kubectl --context kind-exercise2 create namespace istio-system
       ```
     - Install the istio base helm chart.
       ```
       helm --kube-context kind-exercise2 install istio-base istio/base -n istio-system
       ```
     - Install the istiod helm chart.
       ```bash
       helm --kube-context kind-exercise2 install istiod istio/istiod -n istio-system --wait
       ```
     - Verify the istio components are running.
       ```bash
       kubectl --context kind-exercise2 get pods -n istio-system
       ```
       You should see an istiod pod.
     - Enable automatic istio-proxy sidecar injection when new pods are brought up.
       ```bash
       kubectl --context kind-exercise1 label namespace default istio-injection=enabled
       ```
1. Create two services, service-a and service-b, using deployment.yaml files in this repo. In order to mimic a blue-green deployment we will actually create two deployments for service-b: service-b-v1 and service-b-v2.
```
kubectl --context kind-exercise2 apply -f ~/Development/traffic-istio-workshop/exercises/helm-exercise/k8s/service-a.yaml
kubectl --context kind-exercise2 apply -f ~/Development/traffic-istio-workshop/exercises/helm-exercise/k8s/service-b-v1.yaml
kubectl --context kind-exercise2 apply -f ~/Development/traffic-istio-workshop/exercises/helm-exercise/k8s/service-b-v2.yaml
```
> **_NOTE:_** Helm can be used to manage any kubernetes resource, including deployments. We'll keep applying our deployments with `kubectl` for now because we're mostly interested in the interaction between helm and istio.
    - Confirm that the pods are running
      ```bash
      kubectl --context kind-exercise2 get pods -o wide
      ```
1. Familiarize yourself with the helm chart we'll use in this example.
     - 
