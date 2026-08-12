# JobLock website

Two static pages. No build step, no dependencies, no external requests — the
CSS is inline and the only image is an inline SVG, so the whole thing works
offline and cannot leak a visitor's IP to a third party.

| File | Becomes |
|---|---|
| `index.html` | `/` — short landing page |
| `privacy.html` | `/privacy` — the privacy policy |
| `terms.html` | `/terms` — the terms of service |
| `vercel.json` | Clean URLs (`.html` stripped) and a few security headers |

The clean URLs come **only** from `vercel.json`. Without that file, `/privacy`
is a 404 and the links break — which is exactly what happened once already.

---

## Replace these things first

1. **`privacy@thejoblockapp.com`** — an alias that actually forwards to you. Apple
   requires a working contact method on the policy, and this address will be
   scraped. Set it up before publishing; a forwarding alias can be killed and
   replaced, your personal inbox cannot.
2. **`Jack Flickinger`** — swap for the LLC's name once it exists. Until then
   the individual name is correct, and it should match whatever the App Store
   listing says.
3. **`YOUR_STATE`** in `terms.html` — the state whose law governs and whose
   courts hear disputes. Appears twice, in the *Governing law* section.

Also keep the **"Last updated"** date honest. Change it when the substance
changes, not on every typo — a policy whose date moves for no reason is worse
than one that doesn't move at all.

---

## Deploying to Vercel

**Option A — from the dashboard (no CLI).**

1. Push this repo to GitHub.
2. [vercel.com/new](https://vercel.com/new) → import `JFlic/JobLockWebite`.
3. Leave **Root Directory** as `./` — the HTML sits at the repo root. Leave
   framework as *Other*; there is no build command and no output directory.
4. Deploy. You get `https://<project>.vercel.app`, and the policy lives at
   `https://<project>.vercel.app/privacy`.

**Option B — from the terminal.**

```sh
npm i -g vercel
vercel --prod
```

### The custom domain

`thejoblockapp.com`, registered through Vercel. Because Vercel is both the
registrar and the DNS host, there are no A or CNAME records to copy by hand —
Project → Settings → Domains → add `thejoblockapp.com` and it wires itself up.
Add `www.thejoblockapp.com` too and set it to redirect to the apex.

The App Store listing, the Family Controls request and the app's own support
link all point at this URL, so moving it later means editing all three.

---

## Email on the domain

**Vercel does not provide an email service** — its own docs say so. It hosts
the DNS records, and something else has to host the mailbox.

Two addresses, both landing in one inbox:

| Address | Used by |
|---|---|
| `support@thejoblockapp.com` | App Store Connect support contact, users reporting bugs |
| `privacy@thejoblockapp.com` | `privacy.html` and `terms.html` |

This costs nothing if you want it to. The only real question is whether you
need to *send* from the address or merely *receive* at it.

| Option | Cost | Sends? | Lives in |
|---|---|---|---|
| **Zoho Mail** free tier | $0 | **yes** | Zoho webmail + Zoho app |
| **ImprovMX** free tier | $0 | no, forwards only | your Gmail |
| **Cloudflare** Email Routing | $0 | no, forwards only | your Gmail |
| **iCloud+** Custom Email Domain | $0.99/mo | **yes** | Apple Mail |

Forwarding-only is *fine for App Store review* — Apple checks that the
address works, not which server your replies leave from. The cost of
forwarding is that replies go out from your personal Gmail, which exposes
that address to every user who writes in. That is a polish problem, not a
compliance one.

**Zoho Mail's Forever Free plan** is the pick if free matters: real
send-and-receive on a custom domain, up to 5 users and 5 GB each, one domain.
The catch is no IMAP/POP on the free tier, so it will not connect to Apple
Mail — you live in Zoho's webmail or their app. Sign up, verify the domain,
then add the MX and SPF records Zoho prints in Vercel → Domains →
`thejoblockapp.com`. Zoho's record values differ by datacenter region, so
copy theirs rather than any you find in a blog post.

**ImprovMX** is the zero-friction free option: keep DNS on Vercel, add its MX
records plus `v=spf1 include:spf.improvmx.com ~all`, point both addresses at
your Gmail, done in five minutes. Capped at 500 forwards/day, which is not a
real limit here.

### Cloudflare Email Routing, with the site still on Vercel

Also free, and the sturdier of the two forwarders — but Cloudflare requires
the whole zone: *"You must be using Cloudflare DNS to use Email Service."*
There is no partial or CNAME-only mode for it. So the nameservers move to
Cloudflare and Vercel keeps serving the site over records you recreate there.

1. Add `thejoblockapp.com` to Cloudflare as a new site, Free plan.
2. In **Vercel → Domains → `thejoblockapp.com` → Nameservers**, switch to
   Cloudflare's two assigned nameservers. Vercel is the registrar, so this is
   done in Vercel, not anywhere else. Allow up to 48 hours, usually far less.
3. Recreate the site records in Cloudflare's DNS tab:

   | Type | Name | Value | Proxy |
   |---|---|---|---|
   | `A` | `@` | `76.76.21.21` | **DNS only (grey)** |
   | `CNAME` | `www` | `cname.vercel-dns.com` | **DNS only (grey)** |

   > **The grey cloud is not optional.** A proxied (orange) record hides the
   > real answer from Vercel, which blocks its certificate challenge — the
   > symptom is a broken padlock or a redirect loop, not an obvious DNS error.

4. Cloudflare → **Email** → Email Routing → enable. It adds its own MX, SPF
   and DKIM records for you. Route `support@` and `privacy@` to your Gmail
   and confirm the verification mail Cloudflare sends.

That is receiving handled, free and permanently. To *reply* as the address
rather than from Gmail, add it in Gmail under Settings → Accounts → **Send
mail as**, using `smtp.gmail.com` port 587 with a Google App Password
(requires 2-Step Verification). Gmail mails a confirmation code to
`support@thejoblockapp.com`, Cloudflare forwards it to you, and from then on
replies leave with the right From address. Add `include:_spf.google.com` to
the SPF record if you do this.

**iCloud+** is worth the dollar only if you want this in Apple Mail on the
phone you already carry, and it is free at the margin if you already pay for
iCloud storage. iCloud.com → Mail → Settings → Custom Email Domain, then add
the records Apple prints (MX to `mx01`/`mx02.mail.icloud.com`, an SPF TXT, a
verification TXT, and a DKIM CNAME). Three addresses per domain.

Vercel has **DNS Presets** on the domain screen that fill in the records for
common providers automatically — check it before typing anything by hand.
Whichever you pick, send a test message *and reply to it* before putting the
address in front of Apple.

---

## Where this URL is needed

- **App Store Connect** → App Privacy → *Privacy Policy URL*. Required; you
  cannot submit without it. → `https://thejoblockapp.com/privacy`
- **App Store Connect** → App Information → *Support URL*. Also required.
  → `https://thejoblockapp.com`
- **App Store Connect** → App Information → *License Agreement*. Optional —
  Apple's standard EULA is the default. → `https://thejoblockapp.com/terms`
- **The Family Controls (Distribution) request form** has a website field.
  → `https://thejoblockapp.com`
- **Inside the app itself.** Guideline 5.1.1(i): *"All apps must include a link
  to their privacy policy in the App Store Connect metadata field and within
  the app in an easily accessible manner."* A URL in App Store Connect alone is
  not enough — Settings needs a row that opens `https://thejoblockapp.com/privacy`.
  As of this writing `lib/` contains no such link.

---

## When JobLock stops being free

Nothing about the domain or the Family Controls entitlement changes — that
approval is per bundle ID and does not care what the app costs. Three other
things do change, in rough order of how badly they bite.

**1. The privacy label, if you use a paywall SDK.** This is the expensive one.

| How you charge | What happens to "Data Not Collected" |
|---|---|
| StoreKit 2 directly | Survives. Apple processes the payment; you receive no personal data and run no third-party code. |
| RevenueCat, Superwall, Adapty, etc. | **Dies.** These SDKs phone home with a user ID and device info, so you must declare *Purchases* and *Identifiers*, and `privacy.html` stops being true — it currently claims no networking code, no analytics SDKs, and nothing leaving the phone. |

Choosing StoreKit 2 keeps every claim on this site intact. Choosing a paywall
SDK means rewriting the privacy policy, not just amending it.

**2. `terms.html` needs real purchase terms.** The *What it costs* section
says the app is free and contains no purchases, and the liability cap is
written as "the amount you paid for it, which is zero". Both become false the
day you ship a price. Replace them with the actual price, what the purchase
unlocks, and Apple's refund process — **do not promise refunds yourself**,
they run through Apple.

**3. Auto-renewable subscriptions add hard disclosure requirements.** A
one-time non-consumable unlock is much lighter; a subscription obliges you to
show, *inside the app* before the purchase, the subscription title, its
length, its price per period, and functional links to both the privacy policy
and the Terms of Use. The metadata needs the same links — Terms of Use goes in
the App Description if you use Apple's standard EULA, or in the EULA field in
App Store Connect if you use `terms.html`. Missing these is the usual cause of
a Guideline 3.1.2 rejection.

If you only ever ship a one-time unlock via StoreKit 2, the total damage is
one rewritten section of `terms.html`.

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
