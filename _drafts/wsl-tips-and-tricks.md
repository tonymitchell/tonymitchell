---
title: WSL Tips & Tricks
categories: [Tips&Tricks, WSL]
tags: [tips, wsl]
---

## Tip 1: Fix WSL2 clock skew issues

If you're using WSL2 on a laptop, there are known issues with clock skew that can happen when your laptop hibernates. 

To resolve this there are two workarounds:

Inside WSL

```console
hwclock -s
```

And 

From Windows

```
wsl --shutdown
```


## Tip 2: Quickly apply updates to multiple WSL Linux environments

From Windows:

```batchfile
wsl -u root -d Ubuntu -- bash -c "apt-get update && apt-get -y upgrade && apt-get autoremove"
wsl -u root -d Debian -- bash -c "apt-get update && apt-get -y upgrade && apt-get autoremove"
wsl -u root -d kali-linux -- bash -c "apt-get update && apt-get -y upgrade && apt-get autoremove"
```
{: file="update_wsl.cmd" }


