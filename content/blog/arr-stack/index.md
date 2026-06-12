+++
title = "The *arr stack runs itself. I wish."
date = 2026-06-12

[taxonomies]
tags = ["homelab", "selfhosted", "devops"]
+++

That's what they say on the forums. "Set it and forget it." "It just works." Reddit threads full of people casually mentioning their setup like it took an afternoon. It did not take an afternoon.

I run the full stack on a Raspberry Pi 5. Prowlarr finds things. Sonarr and Radarr grab them. qBittorrent downloads them. Bazarr adds subtitles. Everything talks to everything else through a Traefik reverse proxy with OAuth2 and CrowdSec because apparently I hate having free time. It's beautiful, in theory.

In practice, I've spent more time fixing it than watching anything it downloaded. This is fine. I've learned more from breaking this stack than from any course I've ever paid for. Which is also fine, because the courses cost money and the Pi just cost sleep.

## The Sonarr incident

One morning — not a particularly special morning — Sonarr stopped working. Not "slow" stopped. Not "needs a restart" stopped. Database corruption stopped. Just a clean, confident refusal to start. SQLite, it turns out, does not enjoy being written to over a network mount at speed. The journal gets confused. Pages go missing. The database develops opinions.

Who knew. Everyone, apparently. There are entire wiki pages about this. I found them after the fact, which is the correct order of operations.

The fix involved stopping everything, copying the database locally, running `.recover` in the SQLite shell, moving it back, fixing permissions, restarting, and checking the logs with the energy of someone who has already accepted their fate. It worked. I didn't tell anyone. I opened a beer and watched something on YouTube instead of on my perfectly functioning media server.

The lesson: keep your SQLite databases on local storage. The lesson I actually applied: added a cron job that backs up all the databases nightly. Progress.

## The NAS mount problem

The media lives on a NAS, mounted over CIFS. Samba. Old reliable. On reboot, systemd would helpfully start all the *arr containers before the NAS was actually mounted — because it thought the mount was fine. The automount stub exists. `mountpoint -q /mnt/samba` returns true. The directory is there. Everything looks great. Nothing works.

This is a beautiful example of a system doing exactly what you told it to do, and not at all what you wanted.

The fix: a custom `wait-for-nas.service` that loops on `findmnt -t cifs /mnt/samba` until something real shows up. If the NAS isn't there after an hour, we give up and log a failure. Practical. Slightly embarrassing. Runs at 4am when the power blinks and you wake up to find qBittorrent seeding nothing into the void.

## The CrowdSec situation

CrowdSec is a security tool that reads your logs, detects attacks, and shares threat intelligence with the community. Great concept. I added it to the stack. It ran happily for weeks, detecting nothing, sharing nothing, quietly existing.

Turns out `acquis.yaml` — the config that tells CrowdSec which logs to read — was pointing to `/does/not/exist`. An actual path I had copy-pasted from an example and never corrected. The security layer was blind. Watching nothing. Protecting nothing. Feeling good about itself.

Fixed now. Pointing at the actual Traefik log. CrowdSec has since detected several things and I feel considerably safer, or at least more informed about how many bots are trying to log into my Raspberry Pi.

## The upside

It's cheaper than Netflix. Nobody can cancel my subscription. The content library is exactly what I want, not what an algorithm thinks I want. I control the subtitles. I control the quality. I control the infrastructure — loosely, chaotically, but I control it.

Plex runs on a separate container on a Synology NAS. The Pi handles the downloading, the NAS handles the serving. I also have a Samsung 65" OLED in the living room, which is arguably the most important piece of infrastructure in this whole setup.

I have personally watched maybe four things on it. Horror mostly, hard sci-fi when I'm feeling optimistic. Oppenheimer is still in the queue. Has been for a while. I built a home media server, debugged it at 4am, wrote a blog post about it at 5:30am on a Friday — but somehow there's no time to watch a three-hour film about the guy who built the thing that could have ended the world.

The rest is just there, quietly existing at `/mnt/samba`, waiting for a version of me with more free time.

Worth it. Ask me again after the next reboot.
