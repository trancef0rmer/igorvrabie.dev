+++
title = "Ditching Trakt, or: how to rob a dead integration"
date = 2026-07-31
draft = true

[taxonomies]
tags = ["selfhosted", "homelab", "plex"]
+++

I made my Trakt account in 2020, back when I was scrobbling everything from Kodi. Every movie, every episode, every rating — logged automatically, sitting in the cloud, someone else's problem. That was the deal. It was a good deal right up until it wasn't.

At some point Trakt switched to email-link login. No more password — they email you a link, you click it, you're in. Fine, modern, whatever. Except my account, `letsgettrakt`, was made back when I signed up for everything with fake details and an email I no longer control. No email on file means no link. No link means no login. Six years of history, and I was standing outside the door with no key.

"That's fine," said Claudiu. "We'll just export it through the API."

Reader, we would not just export it through the API.

## Three walls

The modern way to pull your data off Trakt is a tool called `traktexport`. It needs a Trakt API application — a client ID and secret you generate from your account. So I went to make one, and Trakt informed me, politely, that creating API applications now requires **VIP**. The paid tier. To get my own data out, I'd have to pay the company I was trying to leave, for the privilege of leaving.

Wall one.

Fine. The browser. I made a brand new Trakt account — a fresh one, created for the sole purpose of rescuing the old one, which is a sentence I did not expect to write — and confirmed I could actually see my old history rendered right there on the page. If the browser can see it, the browser has a token, and if it has a token I can borrow it. Except the new Trakt web app proxies everything through its own backend behind httpOnly cookies. There's no bearer token sitting in the network tab to steal. The data was on the screen and completely out of reach.

Wall two.

Then I remembered PlexTraktSync. Months ago I'd run a little container that kept Plex and Trakt in step, and it authenticates with its own OAuth token — one belonging to the old account. It was still on the ThinkPad. We dug it out, pulled the refresh token, and asked Trakt for a fresh one.

```
400 invalid_grant: session not found
```

Revoked. When the account got locked, every session died with it. (Also, briefly, Trakt's Cloudflare answered `403 error 1010` on everything until Claudiu remembered a Python script has to lie about its User-Agent to pass as a browser. Small thing. Cost us ten minutes and some dignity.)

Wall three. Three dead ends, no data, and a growing suspicion I'd have to eat the loss.

## The corpse had pockets

Here's the thing about PlexTraktSync: it caches. In the same folder as that dead token was a file called `trakt_cache.sqlite`. Twenty-eight megabytes. A `requests-cache` database — every API response the tool had ever fetched, pickled and stored so it wouldn't have to ask Trakt twice.

Every response it had ever fetched. Including the last full sync, June 3rd, of my entire library.

```
594 watched movies
105 watched shows (2,687 episodes)
1,776 episode ratings
watchlist, collection, the lot
```

All of it, sitting in `_content` blobs inside pickled HTTP responses, TMDB IDs and timestamps intact. The export I couldn't buy at any price was already on my own disk, left there by the very integration that had died. We read the pickle by hand — parsing the byte-length opcodes directly instead of unpickling, because you don't execute a pickle you found lying in a cache — and dumped the JSON. Now we just needed somewhere to put it.

Enter YamTrack. Self-hosted, open source, a single container on the Pi. It tracks movies, shows, anime, games, books — everything Trakt did and a few things it didn't — except it runs on my hardware, answers to nobody, and has no VIP tier, no login links, and no mechanism by which it could ever lock me out of my own account. This is the salvation. This is where six years of history goes to be safe.

We converted the cache into a CSV YamTrack could swallow. Then its SQLite database locked up mid-import. Then it locked up again. YamTrack runs a web server, a worker, and a scheduler, and all three wanted to write to one file at the exact moment I was shoving 3,800 rows through it. Claudiu said it'd be fine. It was not fine, twice. We moved it to Postgres and it went in clean: 697 movies, 204 shows, 2,689 episodes, ratings and watchlist. Six years, recovered.

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
