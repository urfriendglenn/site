+++
title = "Certified Kubernetes Administrator: Scheduling"
date = "2024-09-24T21:51:21-05:00"
author = "glewis"
authorTwitter = "" #do not include @
cover = ""
tags = ["cka", "kubernetes"]
keywords = ["cka", "kubernetes"]
description = ""
showFullContent = false
readingTime = false
hideComments = false
+++
# Manual Scheduling
## How Scheduling Works
Every pod has a field called `nodeName` that is blank by default\
The scheduler looks through all pods and searches for pods without this field set\
Those pods are candidates for being scheduled\
It then assigns the pod to the node based on the scheduling algorithm\
It then assigns the node by setting the value to the `nodeName` field\
Nodes can be assigned statically in the definition file as well\
Kubernetes will not allow the value change of the `nodeName` field to an existing pod, so a binding object must be created then posted to the node's binding API:\
`pod-bind-definition.yaml`:
```
apiVersion: v1
kind: Binding
metadata:
	name: nginx
target:
	apiVersion: v1
	kind: Node
	name:
```
