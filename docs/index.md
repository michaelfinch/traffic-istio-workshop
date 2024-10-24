---
id: overview
title: "Overview"
next_page: exercise1
---
<link rel="stylesheet" href="assets/css/styles.css">

# Overview
The goal of this workshop is to gain hands-on experience with some common Istio concepts and tooling. It is a self-guided process that you can work on individually or in pairs. All of the steps are run on your local machine.

During this workshop you will:
* Use [kind](https://kind.sigs.k8s.io/) to install a kubernetes cluster locally.
* Install [Istio](https://istio.io/) within a kubernetes cluster.
* Use [Helm](https://helm.sh/) charts to create and manage Istio resources.
* Use [ArgoCD](https://argo-cd.readthedocs.io/en/stable/) to apply Helm charts from a web UI.

You will complete a series of exercises that build upon one another and have you:
* Send traffic between two kubernetes services.
* Create two versions of a kubernetes service, and control the % of traffic that goes to each one.
* Setup an auth policy to allow/deny traffic from one service to another.
* Roll back an istio change with ArgoCD.
* Create two kubernetes clusters and send traffic between them using gateways.

If you have significant Istio experience and are already familiar with all of this then feel free to pair with someone who has less experience.

## Before You Begin
Please add your github username [here](https://docs.google.com/document/d/1gPU5n6rKzuSSm3Gg8cVlh8vtMEKU-dttfVKfXjMR8F4/edit?tab=t.0). For one of the exercises in the workshop you will push commits to this repo, and in order to do that we first have to add you as a collaborator.

<br />
