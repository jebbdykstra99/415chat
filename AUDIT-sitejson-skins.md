# Skin audit — `site.json` + `index.html` (ten factory clones)

**Role:** skin owner, read/report only  
**Date:** 2026-09-17  
**Home repo:** `jebbdykstra99/415chat` (this PR)  
**Scope:** `site.json`, `index.html`, and terms/privacy stubs on all ten factory clones. `bakasan` out of scope. No skin rewrites. No rail fixes. No news invented.

**Clones (exact GitHub repos, `main` at audit time):**

| Repo | `siteId` | `name` | CNAME / canonical |
|---|---|---|---|
| `jebbdykstra99/415chat` | `415chat` | `415chat` | `415chat.com` |
| `jebbdykstra99/samochat` | `samochat` | `samochat` | `samochat.com` |
| `jebbdykstra99/gpchat` | `gpchat` | `gpchat` | `gpchat.com` |
| `jebbdykstra99/gaichat` | `gaichat` | **`Gen AI`** | `gaichat.com` |
| `jebbdykstra99/bartchat` | `bartchat` | `bartchat` | `bartchat.com` |
| `jebbdykstra99/808chat` | `808chat` | `808chat` | `808chat.com` |
| `jebbdykstra99/buffettchat` | `buffettchat` | `buffettchat` | `buffettchat.com` |
| `jebbdykstra99/27chat` | `27chat` | `27chat` | `27chat.com` |
| `jebbdykstra99/popechat` | `popechat` | `popechat` | `popechat.com` |
| `jebbdykstra99/recruitchat` | `recruitchat` | `recruitchat` | `recruitchat.com` |

**Method:** read each clone’s `site.json` / `index.html` / `styles.css` `:root` / legal stubs / `robots.txt`. Factory contract taken from each clone’s `factory.js` (overlay allowlist, porch dwell, rail kinds, theme apply). HTTP GET (follow redirects) against official + outbound URLs on 2026-09-17. No Pages deploys were edited.

**Factory contract that matters for this audit**

- Overlay CTAs are allowlisted against exact `https://` strings on `rail.forecastPage`, `rail.forecastUrl`, `rail.alertsUrl`, and `rail.outbound[].url`. Mismatched overlay links are dropped silently.
- Known live `rail.kind` values: `nws-forecast`, `nws-cwf`, `bart-bsa`, `f1-calendar`. Anything else (including `porch`) does **not** fetch live cards.
- `applyTheme(site.theme)` writes CSS variables after `styles.css` loads. Hardcoded hex in CSS will not follow the skin theme.
- `applyRailChrome()` overwrites kicker / title / footer from `site.json` after boot. HTML chrome is what a user sees before JS, and what remains if `site.json` fails to load.
- Overlay UI exists only in some factory stamps (see matrix). `porch.dwellMs` is implemented on all ten.

---

## Top findings

1. **`808chat` News/rail HTML is still a vacation-package skin** while `site.json` is live NWS Honolulu. `#page-news` says “Suggested itineraries / Vacation packages”; the right-rail kicker says “This week’s packages.” Factory then paints “In the 808 / Room Brief.” Profile + explore copy still mentions packages.
2. **Rail overlay is only wired on three clones.** `415chat`, `samochat`, and `gaichat` have `factory.js` overlay + a `site.json` `overlay` block. The other seven factories have **no overlay renderer**. Adding `rail.overlay` on those skins would no-op. `808chat` is the nws-forecast sibling that never got the overlay stamp.
3. **`gaichat` overlay + outbound OpenAI URLs are unreachable from this audit** (`403` on `gpt-6-astra`, Agents API, Images 2.5). Overlay links *are* allowlisted against outbound (wiring OK); the destinations are not verified live.
4. **`gpchat` pins dummy Race+Post copy into the live room.** `rail.cmo` + three outbound Wikipedia cards claim a specific Spanish GP result / WDC table, and `sessionSeeds` are mixed into the live feed (factory does this on purpose). Not labeled dummy on the rail.
5. **Six skins still use the samochat Stories gradient** (`#12303a → #2a7a8c → #e07a3d`) in `.stories-text-card`, so Stories text cards keep a Santa Monica look after `site.json` theme applies.
6. **`bartchat` HTML footer still says “Dummy trends… Not official BART”** while `site.json` is `bart-bsa` with `meta: "Live · BART"`. FOUC / no-JS shows the dummy disclaimer over a live advisory rail.

Preview lock, `robots` `Disallow: /`, `noindex, nofollow`, and no GTM hold on all ten. That constraint is intact.

---

## Ranked findings

### P0 — broken or actively misleading

| ID | Repos | Finding |
|---|---|---|
| P0-1 | **`808chat`** | `index.html` News + right-rail chrome is leftover package-tour copy (“Suggested itineraries”, “Vacation packages”, “This week’s packages”). `site.json` `rail.kind` is `nws-forecast` with `meta: "Live · Oahu"` and an official HFO gridpoint. Factory overwrites kicker/title/footer after boot, so the first paint and the `#news` heading flash the old product. Profile still: “Dummy packages, not a booking engine.” Explore search placeholder still includes “packages.” `site.json` `trends[]` are still dummy vacation packages (unused on the live NWS rail, but the package brand is sitting in config). |
| P0-2 | **`gpchat`** | Dummy-as-live on a live rail. `rail.kind` is `f1-calendar` (Jolpica/OpenF1). `rail.cmo` hardcodes “Spanish GP Madrid · Madring” / `state: "Race+Post"` / next Azerbaijan 24–26 Sep. Three `outbound` cards state a concrete result (“Antonelli wins Spanish GP — VER +4.351s”, WDC table) as `Race+Post · Final`. Factory merges outbound onto live cards (`3 + pins`). `sessionSeeds` are **injected into the live feed**, including pre-race “lights out” / “who are you sitting with” copy next to a Race+Post CMO. Wikipedia URLs themselves `200`. The claim, not the host, is the bug. |
| P0-3 | **`bartchat`** | Live rail + dummy chrome. `site.json` `kind: "bart-bsa"`, `meta: "Live · BART"`, `forecastPage` official advisories (`200`). `index.html` footer still: “Dummy trends for dress rehearsal. Not official BART.” News `h1` is “Trending on the lines.” Factory overwrites footer to the official-advisories line after boot; first paint lies. |

### P1 — overlay / rail wiring, stale brand, destination holes

| ID | Repos | Finding |
|---|---|---|
| P1-1 | **`808chat`**, **`bartchat`**, **`gpchat`**, **`buffettchat`**, **`27chat`**, **`popechat`**, **`recruitchat`** | **Overlay factory missing.** Those seven `factory.js` stamps have `porchDwellMs` but no `overlayEnabled` / `railOfficialUrls` / `#rail-overlay`. Overlay cannot be turned on from `site.json` alone. The three nws/live siblings that *do* have overlay code are `415chat`, `samochat`, `gaichat`. `808chat` is the forecast skin that never received the overlay block (`enabled` / `ms` / allowlisted links). `gpchat` and `bartchat` are the other live rails with no overlay config. |
| P1-2 | **`gaichat`** | Overlay is enabled (`ms: 8000`) and both CTAs **are** allowlisted against `outbound[].url` (exact match). Destinations: Anthropic Fable/Mythos `200` (title looks real in this environment); OpenAI `gpt-6-astra` / Agents API / Images 2.5 returned **`403`** from this audit UA (browser UA too). `https://grok.x.ai` redirects to `https://x.ai/` then `403`. Overlay will show the OpenAI CTA; click success is unverified. `kind` is `"porch"` (not a live fetch kind) — rail is outbound + porch only. |
| P1-3 | **`gaichat`**, **`808chat`**, **`gpchat`**, **`buffettchat`**, **`popechat`**, **`recruitchat`** | **Stories text-card gradient is still samochat.** `.stories-text-card` hardcodes `linear-gradient(160deg, #12303a 0%, #2a7a8c 55%, #e07a3d 130%)`. That is `samochat` `--nav-bg` / `--teal` / `--accent`. `site.json` theme apply cannot override it. After Stories v0 stamp these six skins keep a Santa Monica “overwritten look” on text stories. Contrast: `415chat`, `bartchat`, `27chat` retinted the gradient to their own tokens. |
| P1-4 | **`gaichat`** | Identity split. `siteId` / domain / legal title = `gaichat`. Wordmark / `<title>` / OG / auth heading = **`Gen AI`**. `index.html` description and `terms.html` still say **“Not grokchat.”** Intentional distancing, but the public name and the repo/domain/legal name are three different strings. |
| P1-5 | Almost all except `415chat` | **HTML rail chrome ≠ `site.json` rail chrome** (FOUC / no-JS). Factory `applyRailChrome()` hides this after a successful boot. Mismatches: `samochat` “What’s happening in SAMO” vs kicker `On samo`; `gpchat` “What’s happening on the grid” / “Trending in the paddock” vs `Race+Post`; `gaichat` News “In the lab · Policy desk” / “Dummy cards, not advice” / footer “Porch only” vs curated-tool footer; `buffettchat` News kicker “From the letter” vs `In this year's letter`; `27chat` “Listening now / on the wall” / “Works in the room” vs `The work didn't stop`; `recruitchat` “Curated outbound” / “Official desks, not ingest” vs `The room, not the funnel`; plus P0-1 and P0-3. |
| P1-6 | **`gaichat`**, **`buffettchat`**, **`27chat`**, **`popechat`**, **`recruitchat`** | `rail.kind: "porch"` is **not** a factory live kind. Behavior today: no NWS/BART/F1 fetch; `outbound` (if any) + porch card. Same as omitting `kind` and keeping `porch`. The string is leftover / misleading for the next stamp and is inconsistent with `415chat` / `samochat` / `808chat` / `bartchat` / `gpchat`. |
| P1-7 | **`recruitchat`** | `trends[]` use **`href`**, not `url`. Factory `renderTrendCard` / explore cards read `url` only. Those six DOL/BLS/USAJOBS/Indeed/CareerOneStop links never paint. Rail only shows the one `outbound` USAJOBS card (`url` key, `200`). If anyone later falls back to `TRENDS`, the links stay dead. |
| P1-8 | **`gaichat`** | `maxCards: 8` + porch on → factory `railNwsSlots()` is `max - 1` = **7**. Eight outbound cards are configured; the eighth (Grok) is dropped. Overlay still points at cards 1–2. |

### P2 — inconsistency, cache-bust, legal stubs, unused tokens

| ID | Repos | Finding |
|---|---|---|
| P2-1 | All ten | `privacy.html` says **“Google is off until that provider is enabled.”** Every `index.html` still ships the Google button and a modal note that mentions Google. Factory will rewrite the note on boot; the privacy stub stays wrong. |
| P2-2 | All ten except where noted | Legal `styles.css?v=` lags `index.html`. Examples: `415chat` index `v=14` vs terms/privacy `v=7`; `samochat` `12` vs `6`; `gpchat` `14` vs `7`; `bartchat` `13` vs `5`; `808chat` / `buffettchat` / `27chat` / `popechat` `13` vs `7`; `recruitchat` `19` vs `9`. `gaichat` is aligned (`15` / `15`). |
| P2-3 | All ten | Cache-bust versions are not a fleet. `site.json` query ranges `v=2`…`v=36`. `factory.js` ranges `v=26`…`v=38`. `gpchat` is farthest ahead (`site.json?v=36`, `factory.js?v=38`). Not a functional bug unless a file was edited without bumping the matching query. Preload `site.json?v=` matches `data-site` on every clone. |
| P2-4 | **`808chat`**, **`gpchat`**, **`27chat`** | Extra `theme` keys (`lava`, `plumeria`, `gold`, and on `27chat` a duplicate `delay`) are written to `:root` by `applyTheme` but **never read** in `styles.css` (`var(--lava)` / `var(--plumeria)` / `var(--gold)` have no hits on those skins). Dead tokens, not a mismatch. `bartchat` `--delay` *is* used (badge / liked / errors) — that extra key is real. |
| P2-5 | **`415chat`**, **`samochat`** | Dummy `trends[]` are not painted while `kind` is set (factory skips `TRENDS` for live kinds). `415chat` still has a dummy BART card with `meta: "Live in the city"`. Harmless today; would become a dummy-as-live card if `kind` were removed. `samochat` explore `places[].url` are live; two `santamonica.com` URLs `403`’d from this UA (WAF-like). |
| P2-6 | All ten | `stories.enabled: true` with `maxSeconds: 15`, `maxBytes: 8000000` on every skin. Stories tray is factory-injected (no `#stories` node in `index.html`). Consistent, not a bug. |
| P2-7 | All ten | `porch.dwellMs: 9000` on every skin (factory default is also 9000). Overlay `ms` is `7000` on `415chat` / `samochat`, `8000` on `gaichat`, absent elsewhere. Porch prompts/options are skin-sane. |
| P2-8 | All ten | `redditSr` equals `siteId`. Used only for Reddit share `sr=`. Fine. `seedNote` (“sample — not mixed into the live feed”) is honest **except** `gpchat`, which adds “sessionSeeds pin at the top of the live room” (see P0-2). |
| P2-9 | All non-`415chat` factories | `factory.js` still defaults `SITE_ID` / localStorage keys / site.json-failure boot to **`415chat` / “San Francisco, talking.”** Not a `site.json` bug; if `site.json` 404s (wrong cache-bust), the room identity leaks to 415. Skin files themselves do not contain cross-brand `415chat` strings. |
| P2-10 | **`27chat`** | “Live Through This” in places/topics/seed is the Hole album title, not a live-rail claim. Do not treat as dummy-as-live. |

---

## Per-skin matrix

| Repo | kind | porch dwell | overlay in `site.json` | overlay in `factory.js` | overlay links allowlisted | outbound | maxCards | stories | HTML name vs `site.json` | `:root` vs `theme` tokens |
|---|---|---|---|---|---|---|---|---|---|---|
| `415chat` | `nws-forecast` | 9000 | on / 7000 | yes | yes (NWS page = `forecastPage`) | 0 | default 3 | on | match | match |
| `samochat` | `nws-forecast` | 9000 | on / 7000 | yes | yes (NWS page + NDBC buoy = outbound) | 1 | default 3 | on | match | match |
| `gpchat` | `f1-calendar` | 9000 | **absent** | **no** | n/a | 3 pins | 3 (ignored; F1 uses `3+pins`) | on | match | match + unused `gold` |
| `gaichat` | **`porch`** | 9000 | on / 8000 | yes | yes vs outbound | 8 (7 paint) | 8 | on | **`Gen AI` vs `gaichat`** | match |
| `bartchat` | `bart-bsa` | 9000 | **absent** | **no** | n/a | 0 | default 3 | on | match | match; `--delay` used |
| `808chat` | `nws-forecast` | 9000 | **absent** | **no** | n/a | 0 | default 3 | on | name match; **package chrome** | match; unused `lava`/`plumeria` |
| `buffettchat` | **`porch`** | 9000 | **absent** | **no** | n/a | 1 letters archive | default 3 | on | match | match |
| `27chat` | **`porch`** | 9000 | **absent** | **no** | n/a | 0 (porch only) | default 3 | on | match | match; unused `gold` |
| `popechat` | **`porch`** | 9000 | **absent** | **no** | n/a | 1 Vatican News | default 3 | on | match | match |
| `recruitchat` | **`porch`** | 9000 | **absent** | **no** | n/a | 1 USAJOBS | default 3 | on | match | match |

---

## Identity / domain sanity

All ten: `siteId` == repo == CNAME host (minus `.com`). `redditSr` matches. Canonical + OG URL use `https://{cname}/`.

The only wordmark exception is **`gaichat`**: public name `Gen AI`, legal/back-links `gaichat`, domain `gaichat.com` (P1-4).

`index.html` `<title>`, `.brand-title`, `.brand-sub`, and auth heading match `site.json` `name` + `tagline` on every skin (gaichat included: both say `Gen AI`). Compose placeholders match `site.json` on every skin. Compose avatars are local (`415`, `SM`, `GP`, `GA`, `BC`, `808`, `B`, `27`, `PC`, `RC`).

No clone’s `site.json` / `index.html` / legal stubs contain another clone’s `*chat` siteId, except **`gaichat` → grokchat** (intentional “not that flagship” copy).

---

## Rail / porch / overlay detail

### Live rails (honest `Live ·` meta)

| Repo | kind | Official endpoints probed | Notes |
|---|---|---|---|
| `415chat` | `nws-forecast` | NWS grid + forecast page `200` | Overlay headline “Live SF Bay forecast” is honest NWS, not dummy trends. |
| `samochat` | `nws-forecast` | NWS grid + page + NDBC 46221 `200` | Overlay two links, both allowlisted. |
| `808chat` | `nws-forecast` | HFO grid + Honolulu page `200` | **No overlay.** HTML still packages (P0-1). |
| `bartchat` | `bart-bsa` | BART BSA API + advisories page `200` | Public demo key `MW9S-E7SL-26DU-VV8V`. No overlay. Dummy HTML footer (P0-3). |
| `gpchat` | `f1-calendar` | Jolpica next.json `200`; OpenF1 `/v1` root `400` (factory calls `/sessions`, not the bare root) | Wikipedia 2026 GP pages `200`. CMO/outbound/sessionSeeds are the dummy-as-live problem (P0-2). No overlay. |

### Porch-only / outbound rooms

| Repo | Porch | Outbound probed |
|---|---|---|
| `gaichat` | Claude / GPT / Gemini / Grok | Anthropic + Google blog `200`; OpenAI three URLs `403`; `cursor.com` `200`; `grok.x.ai` → `x.ai` `403` |
| `buffettchat` | Buy / Hold / Sell | Berkshire letters `200` |
| `27chat` | A verse / A riff | none |
| `popechat` | Blessing / Question | Vatican News `200` |
| `recruitchat` | A role / A hire | USAJOBS outbound `200`; unused `trends[].href` DOL/BLS `200`; Indeed `403`; CareerOneStop `200` with browser UA |

Porch dwell is uniformly 9000 ms and implemented on all ten factories. No broken `dwellMs` types.

### Overlay allowlist (clones that have overlay)

| Repo | Link | Allowlisted against |
|---|---|---|
| `415chat` | `forecast.weather.gov/MapClick.php?lat=37.7749&lon=-122.4194` | `forecastPage` |
| `samochat` | same pattern for 34.0195, −118.4912 | `forecastPage` |
| `samochat` | NDBC 46221 | `outbound[0].url` |
| `gaichat` | Anthropic Fable/Mythos 5.1 | `outbound[0].url` |
| `gaichat` | OpenAI GPT-6 Astra | `outbound[1].url` |

No overlay link failed the exact-string allowlist. There is no “configured but dropped” overlay CTA in the fleet. The miss is skins that never got an overlay block, plus destinations that do not verify (P1-1, P1-2).

---

## Theme: `site.json` vs `styles.css`

Standard tokens (`bg`, `surface`, `nav-*`, `text*`, `accent*`, `teal`, `silver`, `border`) **match** `:root` on all ten. `applyTheme` will not flash a different palette for those keys.

The “overwritten look” is **not** a JSON/CSS variable mismatch. It is hardcoded hex that ignores variables:

- **P1-3** — samochat Stories gradient on six skins.
- Component leftovers that are less severe: several skins still hardcode rail snippet/meta greys (`#b8c5cf`, `#8aa0b0`) instead of `var(--text-muted)`. Visible, but they do not re-skin the clone as 415/samo the way the Stories gradient does.
- `415chat` / `bartchat` / `27chat` Stories gradients were retinted to local nav/accent. Those three are the model.

Unused extra tokens: `808chat` `lava` / `plumeria`; `gpchat` `gold`; `27chat` `gold` (and `delay` aliased to gold). `bartchat` `delay` is used.

---

## Dummy seed / trends vs “live”

| Repo | Seed policy | Live-claim risk |
|---|---|---|
| `415chat` | `seedNote` honest; seed not in live feed | Dummy trend `meta: "Live in the city"` unused while `kind` is set. Rail `Live · SF Bay` is real NWS. |
| `samochat` | honest seedNote | Rail `Live · Santa Monica` is real NWS. |
| `gpchat` | **`sessionSeeds` pin on the live feed** | **P0-2.** Dummy race weekend + CMO/outbound Final. |
| `gaichat` | honest seedNote | Rail meta is “Gen AI · This room” (not Live). Overlay does not say Live. |
| `bartchat` | honest seedNote | Rail `Live · BART` is real BSA. HTML footer still says dummy (P0-3). |
| `808chat` | honest seedNote | Rail `Live · Oahu` is real NWS. `trends[]` are dummy packages, unused on rail, still in file (P0-1). |
| `buffettchat` | honest | meta “This room.” OK. |
| `27chat` | honest | “Live Through This” is a record title (P2-10). |
| `popechat` | honest | meta “From the source.” OK. |
| `recruitchat` | honest | meta “This room.” Unused trends claim “outbound” but use `href` (P1-7). |

---

## Preview / robots / GTM (constraint check)

| Control | All ten? |
|---|---|
| `index.html` `<meta name="robots" content="noindex, nofollow">` | yes |
| Preview banner “Preview only. Accounts are not open.” | yes |
| Legal pages: preview banner + `noindex, nofollow` | yes |
| `robots.txt` `User-agent: *` / `Disallow: /` | yes |
| GTM / gtag / `googletagmanager` | **absent** (no hits) |
| Terms/privacy titled “(Preview)” and namespaced to the skin | yes (`gaichat` legal title uses `gaichat`, not `Gen AI`) |

Do not lift these in a follow-up skin pass unless product asks.

---

## Factory stamp drift (context only — do not rewrite from this PR)

`factory.js` is **not** one binary. SHA / size at audit:

| Repo | SHA-256 prefix | bytes | overlay UI | notes |
|---|---|---|---|---|
| `415chat` | `3239c728b461` | 178750 | yes | Stories thumbs stamp |
| `samochat` | `2c2f8005013f` | 179127 | yes | |
| `gaichat` | `4df5d1a8fc3a` | 179314 | yes | |
| `gpchat` | `f005b215434f` | 183670 | **no** | sessionSeeds + F1 CMO; newest cache-bust |
| `808chat` | `b941a646b55d` | 170743 | **no** | |
| `bartchat` | `0738f478357f` | 170748 | **no** | |
| `buffettchat` | `d1e601df2962` | 170762 | **no** | |
| `27chat` | `461e46c1b86d` | 170736 | **no** | |
| `popechat` | `2fe1220419ac` | 170600 | **no** | |
| `recruitchat` | `2d2de45815bb` | 170346 | **no** | smallest / oldest of the porch set |

Skin-owner work cannot enable overlay on the seven without a factory stamp. This report does not open PRs on those repos.

---

## What looks healthy

- `415chat` identity, overlay allowlist, NWS URLs, theme tokens, and HTML rail chrome line up. Strongest reference skin for forecast + overlay.
- `samochat` overlay allowlist (forecast + buoy) is the two-link reference. Theme tokens match. Stories gradient is native samo, not leftover.
- Porch options/prompts are on-niche everywhere; `dwellMs` is consistent.
- `buffettchat` / `popechat` outbound hosts resolve (`200`) and do not claim “Live.”
- `stories.enabled` shape is fleet-wide.
- Preview / robots / no-GTM lock is fleet-wide.
- No cross-skin `siteId` leaks in `site.json` or `index.html` (except gaichat’s explicit grokchat denial).

---

## Suggested next owners (not this PR)

Report only. Suggested split if someone picks this up later:

1. Skin HTML/JSON: `808chat` package chrome + dummy package trends; `bartchat` dummy footer/h1; FOUC kickers; `gaichat` name/`grokchat` copy; `recruitchat` `href` → `url` if those cards should exist; drop or label `gpchat` CMO/outbound/sessionSeeds (do **not** invent a replacement result).
2. Factory stamp: overlay renderer onto the seven; decide whether `kind: "porch"` is a real kind or should be omitted.
3. Theme: retint `.stories-text-card` on the six samo-gradient skins to local `nav-bg` / `teal` / `accent`.
4. Legal: Google-off sentence vs Google button; cache-bust legal CSS.

Do not treat HTTP `403` as a confirmed dead page (OpenAI / Indeed / some `santamonica.com` paths look like bot/WAF). Treat `200` official NWS / BART / Jolpica / Berkshire / Vatican / USAJOBS as confirmed.

---

## Audit artifacts

Clone SHAs were `origin/main` at fetch time on 2026-09-17. This file is the only deliverable on `415chat`. No other repos were pushed or PR’d.
