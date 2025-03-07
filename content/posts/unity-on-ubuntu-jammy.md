---
title: "How to install the Unity Desktop on Ubuntu 22.04"
date: 2024-01-28T21:07:47+01:00
draft: false
toc: true
tags:
  - ubuntu
  - ubuntu unity
  - linux
  - desktop environment
  - tutorial
---

> **TL;DR**
>
> You need to install `ubuntu-unity-desktop dbus-x11 mlocate unity-lens-music unity-lens-photos unity-lens-video indicator-notifications unity-tweak-tool compizconfig-settings-manager`
> and then **purge** `gtk3-nocsd libgtk3-nocsd0`.

## 1. Install Ubuntu 22.04

First of you, we of course need to [download](https://ubuntu.com/download/desktop) and [install](https://ubuntu.com/tutorials/install-ubuntu-desktop) the "normal" Ubuntu 22.04.
In this tutorial, I will go with the standard GNOME desktop, because it seems closest to Unity in terms of installed packages.

## 2. Update the system

Secondly, we want to update our system's packages to make sure they are on their latest version.

For that we can simply run `sudo apt update` and then `sudo apt upgrade`.

## 3. Install Unity

Thankfully there is an (almost) complete base package available in the Ubuntu repositories already,
that we can install as a baseline.
However this package alone will result in problems with GNOME when trying to login.
In order for these to be fixed, we will need to additionally install `dbus-x11`.

*A big "thank you" to this askUbuntu answer: https://askubuntu.com/a/1474820*

So, all in all we will install our packages with:
```bash
sudo apt install ubuntu-unity-desktop dbus-x11
```

During the installation the system might ask, if it should use `gdm` or `lightdm` as a *Display Manager*.
Picking `lightdm` will give the "true" Ubuntu Unity experience, however it can also lead to
graphical login being unavailable if anything goes wrong.
Therefore, for now, we will pick `gdm`, which is GNOME's default Display Manager.

We will change this setting later on using `sudo dpkg-reconfigure lightdm`.

## 4. Try out the Unity desktop

A this point we can log out and before logging in again change our desktop environment to "Unity".
Now, we should land in a clean and freshly installed Unity desktop.

However, there are still some minor quirks and problems, that we will fix in the next steps.

## 5. Improvements & Fixing problems

### 5.1. Fixing/Installing Lenses

By default, the Unity Lenses (for searching for pictures, videos and music) will not display
any search results.
This is because the required packages (including the Lenses themselves) have not been installed yet.

We can install them using
```bash
sudo apt install mlocate unity-lens-music unity-lens-photos unity-lens-video
```
(`mlocate` is using by the music and the video Lense for searching for files)

### 5.2. Removing extra title bars

By default the package [`gtk3-nocsd`](https://github.com/PCMan/gtk3-nocsd) will be installed.
This package is made to disable GTK3 client side decorations.
However, in Unity this causes some windows to be rendered with two title bars (and other visual artifacts),
which we want to avoid.

Therefore, we should remove this package and its child package `libgtk3-nocsd0`.
To do so, we need to use `apt purge`, which will also remove the configuration files for this package.
Otherwise, the system would still try to load the library this package adds, which would cause lots
and lots of errors messages everything.

```bash
sudo apt purge gtk3-nocsd libgtk3-nocsd0
```

**Note:** You might get some warnings that say
```plain
ERROR: ld.so: object 'libgtk3-nocsd.so.0' from LD_PRELOAD cannot be preloaded (cannot open shared object file): ignored
```
These warnings will go away after rebooting and can be ignored until then.

## 6. Additional Software

### 6.1. Notification indicator

By default, Unity only displays the current notification (somewhat reliably).
It does not have a builtin feature to display past notifications.

To "fix" this, we can simply install a notification indicator using
```bash
sudo apt install indicator-notifications
```

After relogging, we will have a little letterbox-like icon in the top right,
on which we can click to view (and remove) past notifications.

### 6.2. Installing additional Scopes

Unity Scopes (similarly to Lenses) provide additional services included in the search function
of the Unity Launcher.
The difference however is, that Scopes to not have their own tabs in the Launcher, but instead
show up only in the search results.

One useful scope we can install is the Calculator Scope, which allows us to type
calculations in the search bar and receive the results as a search results.

We can install it using
```bash
sudo apt install unity-scope-calculator
```

After relogging the calculator can be using by typing `result:1+2`, `calc:1+2`, `calculate:1+2` or `calculator:1+2`
in the search bar of the Unity Launcher.
**The Calculator Scope only works in the home (=first) tab of the Launcher.**

> ***Pro tip:** If you are unsure, which Scopes are installed and which keywords they use, you can have a look in `/usr/share/unity/scopes/`.*
>
> *Each Scope has a `.desktop` file in there.*
> *This file describes the Scope and -- most importantly -- mentions the keywords (as `Keywords=keyword1;keyword2;...;`),*
> *that can be used to activate it.*
>
> *Our Calculator Scope can for example be found in `/usr/share/unity/scopes/info/calculator.desktop`.*
> *Its keywords are written as `Keywords=calculator;result;calculate;calc;`.*

We could also install other Scopes, but for this tutorial, we will stick with the calculator.

### 6.3. Installing the Unity Tweak Tool

To get access to additional settings, you can install the *Unity Tweak Tool* using the following command:
```bash
sudo apt install unity-tweak-tool
```

### 6.4. Installing CompizConfig Settings Manager (very advanced)

Since the Unity Desktop is based on *Compiz*, we can install the
*CompizConfig Settings Manager* for a bunch of (very) advanced settings.

> ***Warning:* Changing those settings might break your desktop environment.**

```bash
sudo apt install compizconfig-settings-manager
```

## 7. Finishing touches

### 7.1. Switch to LightDM

After verifying that everything works fine, we can switch from `gdm` to `lightdm` for
the authentic Unity login experience.

To do so, we can simply run (make sure your terminal is maximized for this)
```bash
sudo dpkg-reconfigure lightdm
```
and then pick `lightdm` rather than `gdm` as our default Display Manager..

## 8. Enjoy your Unity Desktop! :sparkling_heart:
