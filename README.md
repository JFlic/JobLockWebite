# JobLock website

Two static pages. No build step, no dependencies, no external requests — the
CSS is inline and the only image is an inline SVG, so the whole thing works
offline and cannot leak a visitor's IP to a third party.

| File | Becomes |
|---|---|
| `index.html` | `/` — short landing page |
| `privacy.html` | `/privacy` — the privacy policy |
| `vercel.json` | Clean URLs (`.html` stripped) and a few security headers |

---

## Replace these two things first

Both appear in `index.html` and `privacy.html`:

1. **`privacy@joblock.app`** — an alias that actually forwards to you. Apple
   requires a working contact method on the policy, and this address will be
   scraped. Set it up before publishing; a forwarding alias can be killed and
   replaced, your personal inbox cannot.
2. **`Jack Flickinger`** — swap for the LLC's name once it exists. Until then
   the individual name is correct, and it should match whatever the App Store
   listing says.

Also keep the **"Last updated"** date honest. Change it when the substance
changes, not on every typo — a policy whose date moves for no reason is worse
than one that doesn't move at all.

---

## Deploying to Vercel

**Option A — from the dashboard (no CLI).**

1. Push this repo to GitHub.
2. [vercel.com/new](https://vercel.com/new) → import the repo.
3. Set **Root Directory** to `website`. Leave framework as *Other*; there is
   no build command and no output directory.
4. Deploy. You get `https://<project>.vercel.app`, and the policy lives at
   `https://<project>.vercel.app/privacy`.

**Option B — from the terminal.**

```sh
npm i -g vercel
cd website
vercel --prod
```

### A custom domain

Vercel → Project → Settings → Domains. A real domain is worth the ~$12/yr
here: the App Store listing, the Family Controls request and the app's own
support link all point at this URL, and moving it later means editing all
three. `joblock.app` would make `privacy@joblock.app` real at the same time.

---

## Where this URL is needed

- **App Store Connect** → App Privacy → *Privacy Policy URL*. Required; you
  cannot submit without it.
- **The Family Controls (Distribution) request form** has a website field.
  This is a perfectly good answer to it.
- **App Store Connect → App Information → Support URL** can point at `/` too.

---

## Keeping it true

The policy describes what the app actually does today, checked against the
code rather than written from a template:

- no networking code anywhere in `lib/`
- screenshots read once and discarded, never retained (`ApplicationLogger`)
- ML Kit's text model runs on-device
- Screen Time selections are opaque tokens the app cannot resolve to app names
- the photo permission is full-library, and the app's restriction to
  post-session images is its own promise rather than something iOS enforces —
  the policy says so plainly, because a policy that overstates the guarantee
  is worse than one that admits the limit

**If any of that changes, this page has to change with it.** The two most
likely candidates are adding analytics and adding a backend for sync. Either
one turns "Data Not Collected" on the App Store listing into a false
declaration, which is a review rejection at best.
