+++
title = "fontconfig 2.18 broke my terminal icons and I'm not okay"
date = 2026-06-14

[taxonomies]
tags = ["linux", "cachyos", "debugging", "fontconfig"]
+++

It started with an upgrade. `cachy-update`, routine stuff. Somewhere in the 400MB of packages was `fontconfig 2.18.1`. I didn't notice. Nobody warns you about fontconfig.

A restart later, my terminal icons were smaller. Not missing — smaller. Powerline glyphs, status icons, the little symbols that make a terminal look like you know what you're doing. All there, just... shrunk.

First suspect: Alacritty font config. I had `monospace` set as the font family — lazy, but it always worked. Claudiu (that's what I call the AI) checked the config, confirmed `monospace` was resolving to `Noto Sans Mono`. Seemed fine.

Then we went down the rabbit hole.

Switched to `JetBrainsMono Nerd Font`. Font looked wrong. Switched to `Iosevka Nerd Font` — the typeface looked right, similar to Noto, but icons were still off. Then we discovered via `fc-match` that `monospace` was always resolving to `Noto Sans Mono`. Iosevka just looked familiar. We spent twenty minutes switching fonts when the font was never the problem.

Then we went after fontconfig itself. Created `~/.config/fontconfig/fonts.conf`. Added alias rules to make `JetBrainsMono Nerd Font` the preferred fallback for Nerd Font glyph ranges — `fc-match` confirmed it was winning. Icons still small. Switched the alias to `Symbols Nerd Font Mono` — specifically designed as a fallback font, should have worked. Still small.

Changing the fallback provider didn't help because the provider was never the problem. fontconfig 2.18 changed how it scales fallback font glyphs. It didn't matter which Nerd Font was serving the glyphs — they all came out smaller. It's not just us — there are open issues on the fontconfig GitLab: [#530](https://gitlab.freedesktop.org/fontconfig/fontconfig/-/work_items/530) ("Nerd Font glyphs render smaller in waybar after upgrade to 2.18") and [#534](https://gitlab.freedesktop.org/fontconfig/fontconfig/-/work_items/534) ("Upgrade to 2.18 makes fonts render wrong on X11"). Same symptoms, same environment, same fix.

The fix nobody wants to admit is the answer: downgrade.

```bash
sudo downgrade fontconfig
sudo downgrade lib32-fontconfig
```

Pick `2.17.1`. Then open `/etc/pacman.conf` and add both to `IgnorePkg`:

```ini
IgnorePkg = fontconfig lib32-fontconfig
```

Icons back. Problem solved.

Now — before we sat down to write this, we asked ourselves: how did we even get here? One command answered it:

```bash
grep fontconfig /var/log/pacman.log
```

`fontconfig` upgraded from `2.17.1` to `2.18.1` on June 6th. Eight days before we started debugging. That command takes two seconds. We should have run it first — before switching fonts, before touching fontconfig rules, before any of it. We didn't. I called Claudiu some names along the way. He said "noted" and kept going. I'm still not sure if that's professionalism or a threat.

Is the fix elegant? No. Is it correct? Also no. Does it work? Yes.

fontconfig 2.18 is sitting in the Arch Linux Archive now, thinking about what it did. My icons are the right size. My `monospace` resolves to Noto Sans Mono. Everything is fine.

Until the next upgrade.
