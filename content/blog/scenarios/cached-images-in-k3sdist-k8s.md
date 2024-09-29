---
title: "Cached Images in K3sdist K8s"
description : "Finding and Pushing the Cached Images in K3S distribution of Kubernetes to Docker Registry"
date: 2024-09-29T10:50:41+05:30
tags: ["ctr", "k3s", "k8s", "containerd"]
draft: false
---

## Scenario 

I have a Kubernetes cluster (K3S distribution) which I use for development of my application. I work only on a subset of services and thereby for un-worked services I am comfortable using stale tags. All the tags are pushed to private docker registry but for  space optimization reasons, we have a policy in place to delete any tags which are created more than 3 months ago. 

I was comfortable working with my setup and now would like to setup similar environment for my colleague. So, I setup a new VM and installed the K3S distribution of the K8S. On deploying my application on this new cluster, most of the un-worked services are failing because of `ImagePullBackoff` error.


## Investigation
My development K8S cluster had cached docker images for the un-worked services and thereby even on pod restart/application re-deployment the cached images are used (given that `imagePullPolicy: Never`) and it works. 
 
Now on the new cluster, I don't have these cached images and thereby the pods are failing.

I need to find the cached images and push them to the private registry so that I can use them on the new cluster.

## Solution

`ctr` is command-line client shipped as part of the containerd project. If you have containerd running on a machine, chances are the ctr binary is also present there. On the K8S cluster (K3S distribution), we have the `containerd` running, hence we have this CLI already available.

#### Steps to list and push cached images in Kubernetes Cluster

1. SSH into the Kubernetes node and find the socket file for containerd

`find / -type s -name "*.sock"`

2. List all the namespace 

`ctr --address /run/k3s/containerd/containerd.sock namespace ls`

3. List all the images with filter

`ctr --address /run/k3s/containerd/containerd.sock --namespace k8s.io image ls -q | grep -q <filter>` 

4. Pick the image and push

`ctr --address /run/k3s/containerd/containerd.sock --namespace k8s.io image push <selected_image>`


