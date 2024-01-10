---
title: "Projects"
draft: false
toc: true
---

## [cheeky-imp](https://git.jcm.re/cheeky-imp/cheeky-imp)

cheeky-imp is a set of tools for hooking into Vulkan applications.
It supports extracting and replacing textures, buffers (e.g. models) and shaders,
as well as an expressive rule-based language for defining custom behaviour.
It also allows loading plugins, with the following plugins available so far:
- [cheeky-imgui](https://git.jcm.re/cheeky-imp/cheeky-imgui) for rendering overlayed GUIs with [Dear ImGUI](https://github.com/ocornut/imgui)
- [cheeky-input-x11](https://git.jcm.re/cheeky-imp/cheeky-input-x11) for receiving input from X11 (and sending it to the overlay)
- [cheeky-dbus](https://git.jcm.re/cheeky-imp/cheeky-dbus) for making arbitrary DBus calls
- [cheeky-datahook](https://git.jcm.re/cheeky-imp/cheeky-datahook) for extracting and injecting arbitrary data (e.g. positions, matrices) from and into shaders

---

## [cheeky-companion](https://git.jcm.re/cheeky-imp/cheeky-companion)

A cheeky-imp plugin for injecting a controllable companion into the game, which can move around freely.

---

## [chocobotpp](https://git.jcm.re/jcm/chocobotpp)

A simple Discord bot made using [D++](https://dpp.dev/) with the following features:
- simple economy system
- reminders
- some minigames
- some fun commands
- Christmas presents
- a (WIP) web dashboard
- simple custom commands
- freely definable command aliases
