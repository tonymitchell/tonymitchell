---
title: Apply updates to multiple WSL Linux environments quickly
categories: [Tips & Tricks, WSL]
tags: [wsl,apt]
date: 2022-06-05 12:00:00 +0700
---

You can update your default WSL environment with this command (assuming it uses the APT packaging system ):

```console
C:> wsl -u root -- bash -c "apt-get update && apt-get -y upgrade && apt-get autoremove"
```

If you have multiple, you can specify it on the command line with the Distro (`-d`) parameter. In the example below I'm updating 3 of my WSL environments in one go from a batch file. 

```batchfile
wsl -u root -d Ubuntu -- bash -c "apt-get update && apt-get -y upgrade && apt-get autoremove"
wsl -u root -d Debian -- bash -c "apt-get update && apt-get -y upgrade && apt-get autoremove"
wsl -u root -d kali-linux -- bash -c "apt-get update && apt-get -y upgrade && apt-get autoremove"
```
{: file="update_wsl.cmd" }

From here you could choose to run this batch file manually as needed, or automatically with the Windows scheduler.  

I find this approach makes it nice and easy to quickly keep all your WSL environments updated.


## A note about escaping special characters...

You may be wondering why I'm calling bash when any command passed to WSL would have been executed by the bash shell anyway. I'm doing this to allow me to pass the command in double-quotes (") to avoid having to escape any characters that are special to the Windows command shell.

For example, if I just tried the following it would result in an error since Windows would interpret the `&&` and then try to run apt-get on the Windows side:

```
wsl -u root -- apt-get update && apt-get -y upgrade && apt-get autoremove
```

In the above version, you will have to escape any special windows characters such as pipes `|` and conditionals `&&` to avoid them being handled by the Windows shell instead.

```
wsl -u root -- apt-get update ^&^& apt-get -y upgrade ^&^& apt-get autoremove
```

This option is perfectly fine, however I'd prefer to not have to think about which characters I have to escape for Windows vs. WSL, and just write my WSL command more naturally.

We can workaround that by running bash explicitly and passing our command in a quoted string.

```
wsl -u root -- bash -c "apt-get update && apt-get -y upgrade && apt-get autoremove"
```

