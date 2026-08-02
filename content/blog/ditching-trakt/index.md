+++
title = "Ditching Trakt, or: how to rob a dead integration"
date = 2026-07-31
draft = false

[taxonomies]
tags = ["selfhosted", "homelab", "plex"]
+++

I made my Trakt account in 2020, back when I was scrobbling everything from Kodi. Every movie, every episode, every rating — logged automatically, sitting in the cloud, someone else's problem. That was the deal. It was a good deal right up until it wasn't.

At some point Trakt switched to email-link login. No more password — they email you a link, you click it, you're in. Fine, modern, whatever. Except my account was made back when I signed up for everything with fake details and an email I no longer control. No email on file means no link. No link means no login. Six years of history, and I was standing outside the door with no key.

"That's fine," said Claudiu. "We'll just export it through the API."

We never exported it through the API.

## Three walls

With the account itself sealed shut, the question got simpler: could I reach the data at all, from anywhere? So I went to the browser and opened my old profile in an incognito window — signed into nothing — just to see whether any of it was even public. It was: my history sitting right there on the page, readable by anyone who knew the URL. I even spun up a brand new Trakt account, one created for the sole purpose of rescuing the old one, and it could see the rest. Progress. Except seeing it isn't having it. To pull a profile out of Trakt programmatically — even a public one — you need an API token, and creating the app to get one now requires **VIP**. It didn't used to. I'd made Trakt API apps years ago, for free, back when poking at the API was half the fun of it. Somewhere along the way it turned into VIP — and not just *called* VIP, actually walled off behind the paid tier. So to export my own data I'd have to pay the company I was trying to leave, for the privilege of leaving.

Wall one.

I tried the human route too — emailed Trakt support, laid out the whole locked-out mess, asked nicely for a hand getting into my own account. No answer. Nothing as I was writing this, and most probably nothing as you're reading it.

So if I can't mint my own token, I'll borrow the one the browser's already using. If the browser can see it, the browser has a token. Except the Trakt web app proxies everything through its own backend behind httpOnly cookies — there's no bearer token sitting in the network tab to steal. The data was on the screen and completely out of reach.

Wall two.

Then something nagged at me. "Wait. I was running some shit for exactly this, years back — some little sync tool, Plex to Trakt, mirrored my scrobbles so I never had to think about it. Hang on, let me check the ThinkPad."

"What was it called?" Claudiu asked.

"No idea. Something obvious." It was PlexTraktSync — an open-source tool whose whole job is to mirror what you watch in Plex up to Trakt so the two stay in step, which means it carries its own Trakt login, an OAuth token. And the one it was holding belonged to the old account. Still sitting there on the ThinkPad, months later, like a key left under a mat.

"That's a live credential," said Claudiu, entirely too pleased. "Pull the refresh token, ask Trakt for a fresh one."

So we did.

```
400 invalid_grant: session not found
```

Revoked. When the account got locked, every session died with it. (Also, briefly, Trakt's Cloudflare answered `403 error 1010` on everything until Claudiu remembered a Python script has to lie about its User-Agent to pass as a browser. Small thing. Cost us ten minutes and some dignity.)

Wall three. Three dead ends, no data, and a growing suspicion I'd have to eat the loss.

## The corpse had pockets

I'd have eaten it. Claudiu wouldn't. While I was drafting the eulogy for six years of history, he was still poking around inside that dead container, and he pointed at something I'd walked straight past.

Here's the thing about PlexTraktSync: it caches. In the same folder as that dead token was a file called `trakt_cache.sqlite`. Twenty-eight megabytes. A `requests-cache` database — every API response the tool had ever fetched, pickled and stored so it wouldn't have to ask Trakt twice.

Every response it had ever fetched. Including the last full sync, June 3rd, of my entire library.

```
594 watched movies
105 watched shows (2,687 episodes)
1,776 episode ratings
watchlist, collection, the lot
```

All of it, sitting in `_content` blobs inside pickled HTTP responses, TMDB IDs and timestamps intact. The export I couldn't buy at any price was already on my own disk, left there by the very integration that had died. I told Claudiu to just unpickle the thing and be done. He refused — you don't execute a pickle you found lying in a cache, same way you don't autorun a USB stick you found in a parking lot — and read it out by hand instead, parsing the length-prefixed byte strings straight from the opcode stream. Slower, paranoid, correct. We dumped the JSON. Now we just needed somewhere to put it.

That somewhere is YamTrack — the point where this stops being a heist and turns into a housewarming. It's a self-hosted, open-source media tracker: movies, shows, anime, games, books, the lot — everything Trakt did, plus a few things it never bothered with. The difference is where it lives: one container on my own Pi, in my own living room. No VIP tier, no login links, no email address it can lose. No backend that can revoke a session, raise the rent, or decide one morning that six years of my own history now costs extra. It answers to me and nobody else. This is where it all goes — and this time, safe means mine.

We converted the cache into a CSV YamTrack could swallow. Then its SQLite database locked up mid-import. Then it locked up again. YamTrack runs a web server, a worker, and a scheduler, and all three wanted to write to one file at the exact moment I was shoving 3,800 rows through it. Claudiu said it'd be fine. It was not fine. It was not fine the second time either, at which point I called him a piece of useless metal — he noted it, the way he always does, and moved the whole thing to Postgres. Went in clean: 697 movies, 204 shows, 2,689 episodes, ratings and watchlist. Six years, recovered.

## The part where Claudiu marks the wrong show

Plex handles the going-forward part, minus one detail: live scrobbling is a Plex Pass feature, and I don't have Plex Pass. Not because of the money — because of the principle. You don't pay a subscription for something you can build yourself out of open source. So we bridged it through Tautulli, which watches Plex and can run a script when something gets marked watched. The script tells YamTrack what I saw.

The first version fired on every episode and passed no arguments at all, because Claudiu had put them in the notifier's *body* field when the script agent reads them from *subject*. Silent. Did nothing. Beautifully.

We fixed that. Then I watched an episode of Grizzy & the Lemmings — a wordless cartoon about a bear terrorizing some lemmings, which I put on when I'm sad, and we are not going to unpack that here — to test the bridge, and YamTrack cheerfully recorded:

```
Marked episode as played: Medium S03E13
```

Not Grizzy. *Medium.* A show I have never watched, a season and episode I've never seen. It turns out Tautulli hands you the *show's* external IDs when you ask about an episode, and YamTrack read the show's TVDB number as if it were an episode's, landing somewhere completely unrelated. Claudiu had trusted the IDs. Claudiu was, once again, confidently wrong.

The real fix makes the bridge resolve each episode's actual ID itself, and refuse to scrobble at all when it can't — better a gap than a lie. I watched an episode of Primal to confirm. This time it marked Primal. The right episode. On its own. I didn't touch a thing.

## What I should have done years ago

The lesson isn't "back up your Trakt account," although, sure, back up your Trakt account. The lesson is that the export I spent an evening failing to obtain never mattered, because the data was never really Trakt's to withhold. It was cached on my own hardware the entire time, by a tool I'd forgotten I was running. I just had to stop asking the company for permission and start reading my own disk.

So it's self-hosted now. YamTrack on the Pi, Postgres underneath, Tautulli feeding it every time I finish something. Nobody emails me a link, nobody revokes my session, nobody gates six years of my own history behind a subscription.

That's the whole thing, really. Don't keep your data on someone else's cloud. The cloud is a computer you're renting, owned by people who can change the locks, raise the rent, or one day email a login link to an address you lost in 2020. Build it yourself. Host it yourself. Own it yourself.

Mine's in the living room. It has never once asked me to prove I'm me.
