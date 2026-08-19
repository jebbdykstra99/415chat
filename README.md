# 415chat

San Francisco geo chat. People talking about the city.

This is a **static** dress rehearsal of the [bakasan.art](https://bakasan.art) SubX chrome (three-column X-like shell: left nav, center feed, right rail, hash routes, sign-in modal, mobile hamburger). It is **not** the FastAPI / Next `subx` stack. No React, no Next, no FastAPI, no Firebase, no model calls.

Wordmark: **415chat**. Tagline: *San Francisco, talking.*

## GitHub Pages + custom domain

These files are meant to drop into an empty public repo `jebbdykstra99/415chat` and be served from GitHub Pages at **415chat.com**.

1. Push this folder’s contents to branch `main` (site root, not `/docs`).
2. Repo **Settings → Pages**: Deploy from branch `main` / `/` (root).
3. Custom domain: `415chat.com`. The `CNAME` file in this repo already contains exactly that.

**DNS at GoDaddy still needs to point at GitHub Pages.** Do not change DNS from this repo. Typical GitHub Pages records:

- Apex `A` records to `185.199.108.153`, `185.199.109.153`, `185.199.110.153`, `185.199.111.153`
- or a `CNAME` for `www` to `jebbdykstra99.github.io`

Until DNS is pointed, Pages will serve on `https://jebbdykstra99.github.io/415chat/` only if the repo is project-pages configured; for the custom domain, use a user/org Pages root as above.

## What this is / is not

- Feed-first dummy posts about SF (fog, BART, burritos, Giants, 415 vs 628). Fake handles only.
- Ranking chrome (For You / Following / Hot / New) is UI only.
- Sign-in modal closes; auth is stubbed locally. No bakasan Firebase project keys.
- No AskAI. No cross-post to X or Reddit. We are not X.com.
