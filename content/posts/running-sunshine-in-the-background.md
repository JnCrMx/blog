---
title: "Running Sunshine in the Background"
date: 2023-01-24T12:38:11+01:00
draft: false
---

[Sunshine](https://github.com/LizardByte/Sunshine) is an open source server implementation for NVIDIA's game streaming protocol.
Its client-side pendant is [Moonlight](https://moonlight-stream.org/) and supports a lot of different devices, making it very useful.
Together they allow for a pretty well-working game streaming experience, especially on local network.

However, it has kind of annoyed me that I had to be logged in on my PC in order to use Sunshine (as it is per-default running as a systemd user service).
Furthermore, the GNOME Screenlock will also effect Sunshine, if it activates after the PC has been left without interaction for too long.
