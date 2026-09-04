# Publishing the Siege Architect website — 6 steps

This folder is a complete static website. Publishing it gets you three things Google asks
for, in one go:

| What | Why you need it | File |
| --- | --- | --- |
| **Privacy policy URL** | Google Play will not let you publish without a public HTTPS policy | `privacy.html` |
| **`app-ads.txt`** | AdMob uses it to verify you own this app's ad inventory; unverified inventory attracts fewer bids | `app-ads.txt` — **and** `root-domain/app-ads.txt`, which goes to a *different* repo (see step 6) |
| **A landing page + playable demo** | somewhere to send people, and where the two files above live anyway | `index.html`, `play/` |

**Before you start:** run `node scripts/build-site.mjs` from the repo root. It builds the
game into `site/play/` and copies the screenshots, the icon and the current privacy policy
into this folder. Nothing here is stale after that.

---

## Step 1 — Create a public repo on GitHub

Go to <https://github.com/new> and create a repository named **`siege-architect-site`**.

- **Public** (GitHub Pages on a free account only serves public repos).
- Do **not** add a README, .gitignore or licence — an empty repo is easier to push into.

## Step 2 — Push the contents of this folder

The simplest layout is: the *contents* of `site/` at the repo root.

```bash
cd "C:\Users\Anuj Gupta\source\repos\SiegeArchitect\site"
git init
git add -A
git commit -m "Siege Architect website"
git branch -M main
git remote add origin https://github.com/anujgupta-echocode/siege-architect-site.git
git push -u origin main
```

*(Or drag every file and folder in `site/` onto GitHub's "uploading an existing file" page.
Make sure the hidden **`.nojekyll`** file goes too — see step 4.)*

**Alternative:** if you would rather keep everything in the main project repo, push the whole
project and set Pages' source folder to **`/site`** in step 3 instead of `/ (root)`. Only do
this if the main repo is public — it contains no secrets (`store/keystore/` and
`store/release/` are gitignored), but check before you make it public.

## Step 3 — Turn on GitHub Pages

In the new repo: **Settings → Pages → Build and deployment**.

- **Source:** *Deploy from a branch*
- **Branch:** `main`, folder **`/ (root)`** (or `/site` if you took the alternative above)
- **Save.**

Wait about a minute; the page shows a green "Your site is live at…" banner when it is ready.

## Step 4 — Check the four URLs

Your site is at:

```
https://anujgupta-echocode.github.io/siege-architect-site/
```

Open all four in a **private/incognito window** — Play's reviewer is not logged in as you:

| URL | Expect |
| --- | --- |
| `…/siege-architect-site/` | the landing page, with screenshots |
| `…/siege-architect-site/privacy.html` | the privacy policy, no login prompt |
| `…/siege-architect-site/play/` | the game boots and the main menu appears |
| `…/siege-architect-site/app-ads.txt` | plain text, starting with a `#` comment (it will not *verify* from this path — step 6) |

If `/play/` shows a blank page, the usual cause is the missing **`.nojekyll`** file: GitHub
Pages runs Jekyll by default and silently drops files whose names begin with an underscore.
`.nojekyll` is in this folder and is written by `scripts/build-site.mjs`; it is hidden, so
check that it actually got committed (`git ls-files | grep nojekyll`).

## Step 5 — Paste the privacy URL into the Play Console

`https://anujgupta-echocode.github.io/siege-architect-site/privacy.html` goes into
**both** of these — Play checks them separately:

1. *App content → Privacy policy*
2. *Grow → Store presence → Main store listing → Privacy policy* (if your Console shows the
   field there)

Also put the site's root URL into *Store settings → Store listing contact → Website*.

## Step 6 — Give AdMob the site, and fix `app-ads.txt`

**The line is already written.** `app-ads.txt` in this folder now carries the live record
for publisher `pub-6236130271371525`:

```
google.com, pub-6236130271371525, DIRECT, f08c47fec0942fa0
```

There is nothing left to paste — but there *is* somewhere else to publish it.

> ### The one thing that will not work from this repo
>
> Google fetches app-ads.txt from the **domain root** — `https://<domain>/app-ads.txt` — and
> from nowhere else. A GitHub Pages **project** site serves this folder under a path, so the
> copy here ends up at `…github.io/siege-architect-site/app-ads.txt`, which is never fetched.
> No error appears; AdMob just keeps saying "unverified".

1. Publish the second copy, **`root-domain/app-ads.txt`**, to a repo named exactly
   **`anujgupta-echocode.github.io`** (a GitHub Pages *user* site — it is served from the
   root). Full instructions, including why the repo name is not negotiable:
   **`root-domain/README.md`**. The alternative is your own domain pointed at Pages.
2. Check `https://anujgupta-echocode.github.io/app-ads.txt` in a private window — plain text,
   HTTP 200.
3. In AdMob: *Apps → Siege Architect → App settings → **Developer website*** — enter the
   **root** `https://anujgupta-echocode.github.io/`, not the `/siege-architect-site/` path.
   AdMob crawls only the root of the domain you declare.
4. *Apps → app-ads.txt →* **Check for updates**. Verification can take ~24 hours.

Keep the two copies identical — `tests/ads/config.test.ts` fails if they diverge.

None of this blocks launch. Ads serve fine without a verified app-ads.txt; you simply get
fewer programmatic bids, which shows up as a slightly lower eCPM.

---

## Keeping it up to date

Re-run `node scripts/build-site.mjs` after any change to the game, the screenshots or the
privacy policy, then commit and push this folder again. The script only replaces the
generated parts (`play/`, `screenshots/`, `privacy.html`, `.nojekyll`, the two images) — your
hand-edited `index.html`, `app-ads.txt` and this README are never touched.

## What is in here

| Path | Generated? | Notes |
| --- | --- | --- |
| `index.html` | hand-written | landing page; the Play badge is a placeholder until the app is live |
| `privacy.html` | **generated** | verbatim copy of `store/privacy-policy.html` — edit that file, not this one |
| `app-ads.txt` | hand-written | **live** — `google.com, pub-6236130271371525, DIRECT, f08c47fec0942fa0` |
| `root-domain/` | hand-written | the same `app-ads.txt` again, for the `anujgupta-echocode.github.io` repo — the only place it verifies from (step 6) |
| `play/` | **generated** | `vite build` output; the game runs from a sub-path because `vite.config.ts` sets `base: './'` |
| `screenshots/` | **generated** | copied from `store/screenshots/` |
| `icon-512.png`, `feature-graphic.png` | **generated** | copied from `store/` |
| `.nojekyll` | **generated** | stops GitHub Pages running Jekyll over the build output |
| `README.md` | hand-written | this file |
