---
title: "Pod gets stuck at the Terminating State"
description : "An interesting scenario where pod gets stuck at terminating state and even the force delete does not deletes the pod."
date: 2024-12-23T11:50:41+05:30
tags: ["linux", "ram", "memory"]
draft: true
---

## Scenario 



# Investigation and Solution

On simple google search, I got directed to this link [https://www.linuxatemyram.com/](https://stackoverflow.com/questions/35453792/pods-stuck-in-terminating-status]


It also gave the solution to clear the cache using the below command.
```bash
kubectl delete pod <PODNAME> --grace-period=0 --force --namespace <NAMESPACE>
```

**Ref**: https://stackoverflow.com/questions/35453792/pods-stuck-in-terminating-status

This worked for me and the application was able to proceed with the installation.
