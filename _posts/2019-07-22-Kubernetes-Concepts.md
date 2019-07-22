---
layout: post
author: junowa
---

This post is based on Kubernetes Concepts guide: https://kubernetes.io/fr/docs/concepts.

## Pods

### Overview

A Pod is a basic execution unit.

- pods that run a single container
- pods that run multiple containers

The multiple containers should form a single cohesive unit of service.  

Example of container design patterns :
- sidecar
- ambassador
- adapter

### Pods and controllers

A controller can create and manage pods for you.

Some examples of controllers:
- deployment
- statefulset
- daemonset

#### Pod Lifecycle

- pending: include time to dl image, before being scheduled
- running: pod bound to a node, all containers created
- succeeded: all pod terminated in sucess
- failed: at least one pod terminated in failure
- unknown: state can not be obtained (typically err in communicating bw host and pod)


##### Pod Conditions

A Pod has a PodStatus which is an array of conditions: **lastProbeTime, lastTransitionTime, message, reason, status, type**.
    
##### Pod Probes

A probe a a diagnostic preformed periodically by Kubelet on a container.
Kubelet calls a handler implemented by the container.

Type of handlers:
- ExecAction
- TCPSocketAction
- HTTPGetAction

Probe results:
- success
- failure
- unknown

The kubelet can optionally perform and react to two kinds of probes on running Containers:
- livenessProbe
- readinessProbe




 