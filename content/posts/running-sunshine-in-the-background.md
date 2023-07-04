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

Two fix this issue it is required to somehow run Sunshine independently of the logged in user, for which I had three ideas:
1. Running Sunshine as a systemd *system* service
2. Running Sunshine in a Docker container, I could deploy on one of my servers
3. Running Sunshine as a different user, that is always logged in

Regardless of which option I had picked, I would have always needed to run a display server (either X11 or Wayland with XWayland) along with Sunshine.

As most display servers require some kind of active session D-Bus, that rules out option 1 for me, because I did not want to first have to figure out how to run such a session in a system service.

Option 2 would arguably be the nicest one, as it provides almost complete separation from the host system.
However this makes it more difficult to allow access to a GPU (which is obviously required for games as well as accelerated streaming).
Furthermore, it also requires games and stores (like Steam) to be either installed in the container or to be on a volume, either mounted on the host or separated for it.
Installing them into the base image of the container would result in an absurdly huge Docker image.
Installing them into the container itself would make them ephemeral and would require reinstallation whenever the container restarts.
Installing them on a volume would work, however sharing it with the host would be likely to cause problems if different library versions are used.
Overall, I decided against this option, not only because of the difficulties, but also because all my servers are running on ``arm64``, making them basically unusable for gaming.

Therefore, in the end I picked option 3 and decided to create a separate user to be always logged in and be running Sunshine.
This sacrifices isolation from the system (which did cause some issues later), but for me that was okay, as I would only be using the game streaming for myself one trusted devices.
