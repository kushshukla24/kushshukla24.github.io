---
title: "Memory available but not free in Linux"
description : "An interesting scenario and concept of Linux where memory is shown to be available but not free in Linux"
date: 2024-12-23T11:50:41+05:30
tags: ["linux", "ram", "memory"]
draft: false
---

## Scenario 

I was configuring a client application on a linux machine. The application does a pre-flight check related to the memory and failed with the message that 
"The memory is insufficient. Application requires 2GB of memory while only 1.6GB is available". 

However, when I checked the memory on the machine using `free -m` command, it showed that there is 12GB of memory is available, but only 1.6GB is free. 

# Investigation and Solution

On simple google search, I got directed to this link https://unix.stackexchange.com/questions/415814/memory-runs-full-over-time-high-buffer-cache-usage-low-available-memory
which simply explained that "Linux borrows unused memory for disk caching. This makes it looks like you are low on memory, but you are not." 

It also gave the solution to clear the cache using the below command.
```bash
echo 3 > /proc/sys/vm/drop_caches 
```
**Ref**: https://unix.stackexchange.com/questions/87908/how-do-you-empty-the-buffers-and-cache-on-a-linux-system
**Ref**: https://www.linuxatemyram.com/

This worked for me and the application was able to proceed with the installation.

