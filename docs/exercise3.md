---
id: exercise3
title: "Exercise 3: Using ArgoCd"
previous_page: exercise2
next_page: exercise4
---
<link rel="stylesheet" href="assets/css/styles.css">

# Exercise 3: Using ArgoCd

<img src="assets/images/argocd_logo.png" alt="argocd logo" width="200"/>
<br />

In this exercise we will start a local kubernetes cluster, use helm to install istio, install argocd, and use argocd to apply our kubernetes resources.

## Procedure

1. Create a new kubernetes cluster with kind.
   ```bash
   kind create cluster --name exercise3
   ```
1. Use helm to install istio in the kubernetes cluster.
     - Create a namespace for istio.
       ```bash
       kubectl --context kind-exercise3 create namespace istio-system
       ```
     - Install the istio base helm chart.
       ```
       helm --kube-context kind-exercise3 install istio-base istio/base -n istio-system
       ```
     - Install the istiod helm chart.
       ```bash
       helm --kube-context kind-exercise3 install istiod istio/istiod -n istio-system --wait
       ```
     - Verify the istio components are running.
       ```bash
       kubectl --context kind-exercise3 get pods -n istio-system
       ```
       You should see an istiod pod.
     - Enable automatic istio-proxy sidecar injection when new pods are brought up.
       ```bash
       kubectl --context kind-exercise3 label namespace default istio-injection=enabled
       ```
1. Install argocd.
   ```bash
   kubectl --context kind-exercise3 create namespace argocd
   kubectl --context kind-exercise3 apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
   ```
     - Verify the argocd components are running.
       ```bash
       kubectl --context kind-exercise3 get pods -n argocd
       ```
       There should be ~7 pods running.
1. Access the argocd UI.
     - Run the following in a dedicated terminal window to make the UI accessible from your browser. It will hang indefinitely.
       ```bash
       kubectl --context kind-exercise3 port-forward svc/argocd-server -n argocd 8080:443
       ```
> **_NOTE:_** If your browser complains about an insecure connection, tell it to proceed anyway.
     - Get the argocd admin password.
       ```bash
       kubectl --context kind-exercise3 get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 --decode
       ```
     - Log in at [https://localhost:8080](https://localhost:8080) with the username `admin` and the password from the previous step.
1. Use argocd to apply the service-a and service-b deployment.yaml files.
     - In the argocd UI, click `NEW APP`. Set the following fields, and then click `CREATE`.
         - Application: workshop-deployment
         - Project Name: default
         - Sync policy: manual
         - Repository url: https://github.com/michaelfinch/traffic-istio-workshop
         - Path: exercises/argocd-exercise/k8s
         - Cluster url: https://kubernetes.default.svc
         - Namespace: default
     - Go back to the argocd homepage and click the `SYNC` button for the `workshop-deployment` application, followed by the `SYNCHRONIZE` button.
     - Verify that the service-a and service-b pods come up.
       ```bash
       kubectl --context kind-exercise3 get pods -o wide
       ```
1. Use argocd to apply the istio helm charts for this exercise.
     - In the argocd UI, click `NEW APP`. Set the following fields, and then click `CREATE`.
         - Application: workshop-istio
         - Project Name: default
         - Sync policy: manual
         - Repository url: https://github.com/michaelfinch/traffic-istio-workshop
         - Path: exercises/argocd-exercise/helm/istio-config
> **_NOTE:_**  After specifying the repository and path, you'll notice that the UI will automatically start populating some of the form fields if you scroll down.
         - Cluster url: https://kubernetes.default.svc
         - Namespace: default
     - Go back to the argocd homepage and click the `SYNC` button for the `workshop-istio` application, followed by the `SYNCHRONIZE` button.
     - Verify that the virtual service for service-b was created.
       ```bash
       kubectl --context kind-exercise3 get virtualservice service-b -o yaml
       ```

<br />

> **_EXPLORE:_** Take some time to explore the argocd UI.

<br />
