---
id: exercise2
title: "Exercise 2: Using Helm"
previous_page: exercise1
next_page: exercise3
---
<link rel="stylesheet" href="assets/css/styles.css">

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
       kubectl --context kind-exercise2 label namespace default istio-injection=enabled
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
1. Familiarize yourself with the helm chart we'll use in this example: [https://github.com/michaelfinch/traffic-istio-workshop/tree/main/exercises/helm-exercise/helm/istio-config](https://github.com/michaelfinch/traffic-istio-workshop/tree/main/exercises/helm-exercise/helm/istio-config).
> **_TEST YOUR KNOWLEDGE:_** What do you think will happen when we apply this helm chart?
> <br /><br />Istio `VirtualServices` are documented [here](https://istio.io/latest/docs/reference/config/networking/virtual-service/).
> <br />Istio `DestinationRules` are documented [here](https://istio.io/latest/docs/reference/config/networking/destination-rule/).
1. See how the helm chart will be rendered. The `helm template` command takes the values.yaml file and applies it to the files in the templates/ directory.
   ```bash
   helm --kube-context kind-exercise2 template istio-config ~/Development/traffic-istio-workshop/exercises/helm-exercise/helm/istio-config
   ```
1. Install the helm chart we'll use in this example.
   ```bash
   helm --kube-context kind-exercise2 install istio-config ~/Development/traffic-istio-workshop/exercises/helm-exercise/helm/istio-config
   ```
     - Verify that the chart was installed.
       ```bash
       helm --kube-context kind-exercise2 list
       ```
1. Verify that the `VirtualService` and `DestinationRule` were applied in istio.
     - Show the `VirtualService`.
       ```bash
       kubectl --context kind-exercise2 get virtualservice service-b -o yaml
       ```
     - Show the `DestinationRule`.
       ```bash
       kubectl --context kind-exercise2 get destinationrule service-b -o yaml
       ```
1. Check the envoy config and verify the 99%/1% traffic split for the two service-b deployments.
   ```bash
   istioctl --context kind-exercise2 proxy-config routes $(kubectl --context kind-exercise2 get pod -l app=service-a -o jsonpath='{.items[0].metadata.name}') -o json | jq '.. | objects | select(has("weightedClusters")) | .weightedClusters'
   ```
   The output should be:
   ```json
   {
     "clusters": [
       {
         "name": "outbound|80|v1|service-b.default.svc.cluster.local",
         "weight": 99
       },
       {
         "name": "outbound|80|v2|service-b.default.svc.cluster.local",
         "weight": 1
       }
     ]
   }
   ```
1. Send a bunch of curls from service-a to service-b and confirm that ~99% of them go to service-b-v1.
   ```bash
   for i in {1..20}; do
     kubectl --context kind-exercise2 exec -it $(kubectl --context kind-exercise2 get pod -l app=service-a -o jsonpath='{.items[0].metadata.name}') -- curl service-b.default.svc.cluster.local:80
     echo
   done
   ```
1. Manually update the weights in the [values.yaml](https://github.com/michaelfinch/traffic-istio-workshop/blob/main/exercises/helm-exercise/helm/istio-config/values.yaml) file on your laptop to give both versions of service-b 50% weight.
1. Apply the updated helm chart.
   ```bash
   helm --kube-context kind-exercise2 upgrade istio-config ~/Development/traffic-istio-workshop/exercises/helm-exercise/helm/istio-config -f ~/Development/traffic-istio-workshop/exercises/helm-exercise/helm/istio-config/values.yaml
   ```
1. Verify the change took effect.
     - Show the `VirtualService`.
       ```bash
       kubectl --context kind-exercise2 get virtualservice service-b -o yaml
       ```
     - Check the envoy config and verify the 50/50 traffic split for the two service-b deployments.
       ```bash
       istioctl --context kind-exercise2 proxy-config routes $(kubectl --context kind-exercise2 get pod -l app=service-a -o jsonpath='{.items[0].metadata.name}') -o json | jq '.. | objects | select(has("weightedClusters")) | .weightedClusters'
       ```
       The output should be:
       ```json
       {
         "clusters": [
           {
             "name": "outbound|80|v1|service-b.default.svc.cluster.local",
             "weight": 50
           },
           {
             "name": "outbound|80|v2|service-b.default.svc.cluster.local",
             "weight": 50
           }
         ]
       }
       ```
1. Send a bunch of curls from service-a to service-b and confirm that they are evenly split across service-b-v1 and service-b-v2.
   ```bash
   for i in {1..20}; do
     kubectl --context kind-exercise2 exec -it $(kubectl --context kind-exercise2 get pod -l app=service-a -o jsonpath='{.items[0].metadata.name}') -- curl service-b.default.svc.cluster.local:80
     echo
   done
   ```

<br />
> **_TEST YOUR KNOWLEDGE:_** Modify the [exercise2 helm chart](https://github.com/michaelfinch/traffic-istio-workshop/tree/main/exercises/helm-exercise/helm/istio-config) on your laptop to do the following:
> 1. Make 100% of traffic route to service-b-v1.
> 2. Using the [VirtualService](https://istio.io/latest/docs/reference/config/networking/virtual-service/) docs, enable request mirroring so that all requests to service-b-v1 are mirrored to service-b-v2.
> 3. Apply the helm changes.
> 4. Tail the logs of the service-b-v2 pod.
> 5. Send some requests from service-a to service-b. The requests should all hit service-b-v1, but you should also see logs on the service-b-v2 pod from the mirrored requests.
> <details>
> <summary>Solution</summary>
> <p>One way to implement this is with the following <code class="language-plaintext highlighter-rouge">values.yaml</code>:
> <pre>
> istio:
>   virtualService:
>     enabled: true
>     name: service-b
>     hosts:
>       - service-b
>     http:
>       - route:
>           - destination:
>               host: service-b
>               subset: v1
>             weight: 100          # <-- 100% weight
>           - destination:
>               host: service-b
>               subset: v2
>             weight: 0            # <-- 0% weight
>         mirror:                  # <-- added this stanza
>           host: service-b
>           subset: v2
>   destinationRule:
>     enabled: true
>     name: service-b
>     host: service-b
>     subsets:
>       - name: v1
>         labels:
>           version: v1
>       - name: v2
>         labels:
>           version: v2
> </pre>
> </p>
> </details>

<br />
<br />
> **_TEST YOUR KNOWLEDGE:_** Update the [exercise2 helm chart](https://github.com/michaelfinch/traffic-istio-workshop/tree/main/exercises/helm-exercise/helm/istio-config) on your laptop to do the following:
> 1. Using the [Authorization Policy](https://istio.io/latest/docs/reference/config/security/authorization-policy/) docs, add an access control that blocks requests from service-a to service-b.
> 2. Apply the helm changes.
> 3. Send some requests from service-a to service-b. You should get `RBAC: access denied` errors.
> 3. Modify the access control to _allow_ requests from service-a to service-b, apply it, and send some test requests.
> <details>
> <summary>Hint</summary>
> <p>Start by creating the file <code class="language-plaintext highlighter-rouge">templates/authorizationpolicy.yaml</code>. To make things simple at first you can create it without any templating logic or variable interpolation. Then once it looks good you can introduce variables and templating expressions to make the chart more dynamic.</p>
> </details>
> <details>
> <summary>Solution</summary>
> <p>One way to implement this is with the following <code class="language-plaintext highlighter-rouge">values.yaml</code>:
> <pre>
> istio:
>   virtualService:
>     enabled: true
>     name: service-b
>     hosts:
>       - service-b
>     http:
>       - route:
>           - destination:
>               host: service-b
>               subset: v1
>             weight: 100
>           - destination:
>               host: service-b
>               subset: v2
>             weight: 0
>   destinationRule:
>     enabled: true
>     name: service-b
>     host: service-b
>     subsets:
>       - name: v1
>         labels:
>           version: v1
>       - name: v2
>         labels:
>           version: v2
>   policy:                        # <-- added this stanza
>     enabled: true
>     action: ALLOW # Use DENY to block access
>     source:
>       namespaces:
>         - default
>       principals:
>         - cluster.local/ns/default/sa/default
>     destination:
>       hosts:
>         - service-b
> </pre>
> <br />
> and the following <code class="language-plaintext highlighter-rouge">templates/authorizationpolicy.yaml</code>:
> <br />
> <br />
> <pre>
> &#123;&#123;- if .Values.istio.policy.enabled &#125;&#125;
> apiVersion: security.istio.io/v1beta1
> kind: AuthorizationPolicy
> metadata:
>   name: allow-from-service-a
>   namespace: default
> spec:
>   action: &#123;&#123; .Values.istio.policy.action &#125;&#125;
>   rules:
>   - from:
>     - source:
>         namespaces:
>         &#123;&#123;- toYaml .Values.istio.policy.source.namespaces | nindent 12 &#125;&#125;
>         principals:
>         &#123;&#123;- toYaml .Values.istio.policy.source.principals | nindent 12 &#125;&#125;
>     to:
>     - operation:
>         hosts:
>         &#123;&#123;- toYaml .Values.istio.policy.destination.hosts | nindent 12 &#125;&#125;
> &#123;&#123;- end &#125;&#125;
> </pre>
> </p>
> </details>

<br />
