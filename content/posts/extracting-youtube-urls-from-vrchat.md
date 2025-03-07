---
title: "Extracting YouTube URLs from VRChat with Proton"
date: 2024-04-26T23:43:04+02:00
draft: false
tags:
  - vrchat
  - game hacking
  - proton
  - wine
  - linux
  - youtube-dl
---

Some VRChat worlds play awesome music, but don't tell you what it is.

So here goes a (relatively) simple trick to find the YouTube URLs using Proton on Ubuntu
(or possibly any other Linux distro).

> **TL;DR**
>
> Set the launch options in Steam to `WINEDEBUG=trace+process %command%`.
>
> Run Steam from the terminal and `grep` for `yt-dlp.exe`.
> ```bash
> steam 2>&1 | grep yt-dlp.exe
> flatpak run com.valvesoftware.Steam 2>&1 | grep yt-dlp.exe
> ```
>
> *Enjoy~*

## Background

VRChat uses [yt-dlp](https://github.com/yt-dlp/yt-dlp) in order to extract music and videos
that get played ingame.

However, the program is not directly integrated, but instead downloaded as an `.exe` and then
run with the appropriate arguments whenever a YouTube-URL needs resolving.

Therefore, in order to find those URLs we simply need to monitor which processes VRChat launches.
A simple way to do this, is to make use of the existing debugging functionality of Wine.
For this we want to launch the game with the environment variable `WINEDEBUG` set to `trace+process`.
This will give detailed logs of all kinds of process-related calls the game makes.

Now, the only thing left is filtering the log output for any executions of `yt-dlp.exe`.

## Detailed Step-by-step guide

### 1. Make sure you are on Linux

Currently these instructions only work on Linux, but doing it on Windows should be even easier.
For example, the SysInternals [Process Monitor](https://learn.microsoft.com/en-us/sysinternals/downloads/procmon) will show you all kinds of events, including executed processes.

### 2. Add launch options

Go to *Steam > Library > VRChat > Gear Icon > Properties > General* and set the "Launch options"
to `WINEDEBUG=trace+process %command%`.

### 3. Exit Steam

Completely exit Steam, so we can restart it with logs streamed to the terminal in the next step.

### 4. Launch Steam and filter the logs

This step depends on how you installed Steam.

#### Snap or native (`.deb`) installation

Simply run Steam like this:
```bash
steam 2>&1 | grep yt-dlp.exe
```

#### Flatpak

Simply run Steam like this:
```bash
flatpak run com.valvesoftware.Steam 2>&1 | grep yt-dlp.exe
```

These commands will take all (`2>&1`) of Steam's output and filter it for any lines containing `yt-dlp.exe`.

### 5. Start the game and join the world with the video

### 6. Search for appropriate log lines containing the URLs

Look into your terminal and look for log lines containing YouTube-URLs.
They should look like this:
```txt
01a8:trace:process:NtCreateUserProcess L"\\??\\C:\\users\\steamuser\\AppData\\LocalLow\\VRChat\\VRChat\\Tools\\yt-dlp.exe" image L"C:\\users\\steamuser\\AppData\\LocalLow\\VRChat\\VRChat\\Tools\\yt-dlp.exe" cmdline L"--no-check-certificate --no-cache-dir --rm-cache-dir -f \"(mp4/best)[height<=?2160][height>=?64][width>=?64]\" --get-url \"https://youtu.be/...\" --exp-allow \"facebook.com,nicovideo.jp,soundcloud.com,twitch.tv,vimeo.com,youku.com,youtube.com,youtu.be,assets.vrchat.com,v-market.wo"... parent (nil) machine 0
```

### 7. Exit the game and Steam and then restart Steam normally

## Other possible methods/ideas:
- Monitor process creation using `auditd` & `auditctl -a task,always`
- Replace the `yt-dlp.exe` binary and log the arguments before executing the real binary
- Monitor process creation with Windows tools
- Be lucky and catch the process in System Monitor / Task Manager
