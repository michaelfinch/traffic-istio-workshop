---
id: exercise1
title: "Exercise 1: Using Istio"
previous_page: overview
next_page: exercise2
---
<link rel="stylesheet" href="/assets/css/styles.css">

# Exercise 1: Using Istio

<img src="assets/images/istio_logo.png" alt="istio logo" width="200"/>
<br />

In this exercise we will start a local kubernetes cluster, install istio, deploy two kubernetes services, and test sending traffic between the services via istio.

## Procedure

1. Checkout the [traffic-istio-workshop](https://github.com/michaelfinch/traffic-istio-workshop/tree/main) repo in the ~/Development directory on your laptop. This repo contains files we will use in this exercise.
   ```bash
   cd ~/Development
   git clone git@github.com:michaelfinch/traffic-istio-workshop.git
   ```

2. Install [kind](https://kind.sigs.k8s.io/). kind is a tool for running local Kubernetes clusters using Docker.
   ```bash
   brew install kind
   ```

3. Install [istioctl](https://istio.io/latest/docs/reference/commands/istioctl/). istioctl is a tool for interacting with Istio.
   ```bash
   brew install istioctl
   ```

4. Create a Kubernetes cluster with kind.
   ```bash
   kind create cluster --name exercise1
   ```

    - Verify the cluster exists.
      ```bash
      kind get clusters
      ```
    - Verify kubernetes is running in the cluster.
      ```bash
      kubectl cluster-info --context kind-exercise1
      ```
      The output should look like this:
      ```
      Kubernetes control plane is running at https://127.0.0.1:54508
      CoreDNS is running at https://127.0.0.1:54508/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

      To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
      ```
> **_NOTE:_**  kind will automatically set your kubernetes context to `kind-exercise1`. You can confirm this by running `kubectl config current-context`. Regardless, we will always manually specify the context for every command in this workshop to ensure we're using the correct one.
1. Install istio in the kubernetes cluster.
```
istioctl --context kind-exercise1 install --set profile=demo -y
```
The `--set profile=demo` allows us to test out a bunch of istio functionality without having to manually enable each component.
    - Verify the istio components are running.
      ```
      kubectl --context kind-exercise1 get pods -n istio-system
      ```
      You should see an ingress gateway pod, an egress gateway pod, and an istiod pod (the control plane).
    - Enable automatic istio-proxy sidecar injection when new pods are brought up.
      ```
      kubectl --context kind-exercise1 label namespace default istio-injection=enabled
      ```
1. Create two services, service-a and service-b, using deployment.yaml files in this repo.
```
kubectl --context kind-exercise1 apply -f ~/Development/traffic-istio-workshop/exercises/istio-exercise/k8s/service-a.yaml
kubectl --context kind-exercise1 apply -f ~/Development/traffic-istio-workshop/exercises/istio-exercise/k8s/service-b.yaml
```
    - Confirm that the pods are running
      ```
      kubectl --context kind-exercise1 get pods -o wide
      ```
> **_TEST YOUR KNOWLEDGE:_**  inspect the deployment files `service-a.yaml` and `service-b.yaml`. What do these services do?
1. Use curl to send a request from service-a to service-b via istio.
```
kubectl --context kind-exercise1 exec -it $(kubectl --context kind-exercise1 get pod -l app=service-a -o jsonpath='{.items[0].metadata.name}') -- curl service-b.default.svc.cluster.local:80
```
The output should be:
```
Hello from Service B
```
     - If you get an error about `curl` not being found, it's still being installed. Wait a minute and try again.

1. Inspect istio.
     - Get an overview of the istio mesh.
       ```
       istioctl --context kind-exercise1 proxy-status
       ```
     - Show the envoy config for the service-a pod.
       ```
       istioctl --context kind-exercise1 proxy-config all $(kubectl --context kind-exercise1 get pod -l app=service-a -o jsonpath='{.items[0].metadata.name}')
       ```
     - Show the cluster, route, and endpoint for service-b in service-a's envoy config.
       ```
       istioctl --context kind-exercise1 proxy-config all $(kubectl --context kind-exercise1 get pod -l app=service-a -o jsonpath='{.items[0].metadata.name}') | grep -i service-b
       ```
     - Show the envoy config for the service-a pod in JSON format, which is identical to the x/config_dump output from envoy's admin interface.
       ```
       istioctl --context kind-exercise1 proxy-config all $(kubectl --context kind-exercise1 get pod -l app=service-a -o jsonpath='{.items[0].metadata.name}') -o json | less
       ```
     - Curl the envoy admin interface directly on the service-a pod.
       ```
       kubectl --context kind-exercise1 exec -it $(kubectl --context kind-exercise1 get pod -l app=service-a -o jsonpath='{.items[0].metadata.name}') -- curl -s http://localhost:15000/clusters | grep healthy
       ```
> **_TEST YOUR KNOWLEDGE:_**  Can you send a curl from the service-a pod to the `/version` path of the istio control plane (istiod) to confirm connectivity?
> <details>
> <summary>Hint 1</summary>
> <p>grep for <code class="language-plaintext highlighter-rouge">istiod</code> in the envoy config.</p>
> </details>
> <details>
> <summary>Hint 2</summary>
> <p>istiod listens on two ports, one for XDS traffic over gRPC (15010) and the other to expose monitoring info over HTTP (15014).</p>
> </details>
> <details>
> <summary>Solution</summary>
> <p><code class="language-plaintext highlighter-rouge">kubectl --context kind-exercise1 exec -it $(kubectl  --context kind-exercise1 get pod -l app=service-a -o jsonpath='{.items[0].metadata.name}') -- curl -sS istiod.istio-system:15014/version</code></p>
> </details>
