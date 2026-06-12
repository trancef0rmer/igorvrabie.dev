+++
title = "I use Hyprland. My mouse is decorative."
date = 2026-06-12

[taxonomies]
tags = ["linux", "hyprland", "cachyos", "homelab"]
+++

I didn't plan this. Nobody plans this. You start with GNOME, like a normal person. Then you read one Reddit post about tiling window managers and six months later you're writing Lua configs at 2am and explaining to your girlfriend — just kidding, nobody who runs Hyprland has a girlfriend — why the taskbar is now a vertical strip of icons on the left side of the screen, actually it's called a bar, and no, the mouse still works, I just don't use it.

Hyprland is a Wayland compositor. It tiles your windows automatically, meaning you never overlap anything, everything is always visible, and your workflow becomes embarrassingly efficient once you get past the part where nothing works and you cry.

The config file is called `hyprland.conf`. It is 400 lines long. I know what every line does. I have broken it at least 40 times. Once I ended up with no keyboard input and had to SSH in from my phone to fix it. The irony of using a keyboard-only window manager and losing keyboard access was not lost on me.

## The keybindings

SUPER+Q closes a window. SUPER+RETURN opens a terminal. SUPER+W changes wallpaper, which triggers pywal, which recolors my entire desktop to match the wallpaper, which is genuinely the most impressive thing my setup does and absolutely nobody cares.

SUPER+1 through SUPER+9 switches workspaces. I use all nine. Each one has a purpose. I don't remember what most of them are for. There are three terminals open on workspace 3 and I'm afraid to close them.

## The mouse situation

The mouse is plugged in. It works. I use it exclusively for gaming and for those moments when a website decides to render a datepicker that requires clicking a specific pixel. I resent those websites personally.

Everything else is keyboard. SUPER+arrows to move focus. SUPER+SHIFT+arrows to move windows. Rofi for launching apps — type two letters, hit enter, done. I haven't touched a taskbar since 2023 and I feel fine.

## Waybar

The bar at the top shows the time, the workspace number, network status, battery, and the current song. It's written in JSON and CSS. I have three themes. I switch between them with a script. The script is called `waybarLayout.sh` and it is the most useful thing I have ever written, which says something about my priorities.

## Was it worth it?

Absolutely not. I spent two weeks configuring something I could have ignored entirely. My productivity did not improve. My understanding of Wayland, compositing, IPC sockets, and why X11 is held together with string improved enormously.

I would do it again. I'm probably going to do it again when Hyprland 2.0 breaks everything.

btw, I use CachyOS.
