---
title: "[Fix] Internet not working on WSL when VPN is connected"
description : "Issue with WSL when VPN is connected"
date: 2024-12-22T10:50:41+05:30
tags: ["wsl", "vpn"]
draft: false
---

## Scenario 

I am using Windows Subsystem for Linux (WSL) for my development work and I have a VPN client installed on my Windows machine. When I connect to the VPN, the internet stops working on WSL. However, the internet works fine on the Windows machine, but not on WSL.

## Solution
Create a file `%UserProfile%\.wslconfig` with the content below, and then restart wsl. 

```ini
[wsl2]
networkingMode=mirrored
dnsTunneling=true
```

Run `wsl.exe --shutdown`to restart WSL.

**Ref**: https://superuser.com/questions/1630487/no-internet-connection-ubuntu-wsl-while-vpn

## Tip
Launching the WSL with root user always

Execute this in the Powershell
```powershell
ubuntu config --default-user root
```
