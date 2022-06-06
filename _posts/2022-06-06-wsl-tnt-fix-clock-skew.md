---
title: Fix WSL2 clock skew issues
categories: [Tips & Tricks, WSL]
tags: [wsl,workaround,clockskew]
date: 2022-06-06 12:00:00 +0700
---

If you're using WSL2 on a laptop, there are [known](https://github.com/microsoft/WSL/issues/5324) [issues](https://github.com/microsoft/WSL/issues/8204) with clock skew that can happen when your laptop hibernates.  This can result in your WSL time and your Windows time getting out of sync and causes all kinds of issues inside Linux.

To resolve this there are two workarounds:

1. From inside WSL you can run the following command to re-sync with the hardware clock.

    ```console
    $ hwclock -s
    ```

2. Or from Windows you can just shutdown the WSL subsystem and it will automatically sync when it starts up the next time you use it.

    ```console
    C:> wsl --shutdown
    ```


Generally, I prefer #2 since it will be applied to all WSL environments in one go, and I can easily run it from Windows as part of my batch script that [updates Linux in all my WSL environment]({% post_url 2022-06-05-wsl-tnt-update-wsl-linux-env %}).  It does cause any existing WSL sessions to be terminated though, so if I've got a lot of sessions going I'll just go for the first option.

