# YOSSI150 — Campaign Brief

## 1. The campaign in one paragraph

YOSSI150 is a personal promo-code campaign for **Islo** (islo.dev), Incredibuild's cloud sandbox platform for AI coding agents ("Everything coding agents need."). A simple landing page hosted free on GitHub Pages serves as Yossi's shareable hub: it explains Islo in one screen and funnels visitors to islo.dev/app.islo.dev with the code **YOSSI150**, which gives an **extra $150 USD in Islo bonus credits when buying credits** (Paddle is Islo's confirmed live payment processor; billing is prepaid credit packs). New users also get $50 free signup credits, no card required. At islo.dev's own worked example of $2.46 per 8-hour overnight agent session, the $150 bonus alone funds roughly 60 such sessions, or about 2,100 CPU-hours at $0.07/CPU-hour. One honest caveat up front: **the offer is defined, but the redemption mechanism is not built yet.** No promo-code feature exists on islo.dev or in the app bundle today, and a Paddle discount alone cannot grant bonus credits (it only reduces price — see Section 3), so Islo must build or configure the redemption path before this campaign can launch.

## 2. Why a personal landing page + link

- **Attribution.** A plain "check out islo.dev" mention is invisible in analytics. A personal link with UTM parameters (e.g., `https://islo.dev/?utm_source=yossi&utm_medium=landing&utm_campaign=yossi150`) tags every visitor, so signups from Yossi's audience are distinguishable from organic traffic.
- **Trust.** A human-named code ("YOSSI150") signals a personal recommendation rather than a generic sale ("SALE10"). People redeem codes from a person they follow at a higher rate because the code itself carries the endorsement. *(This is standard growth-marketing reasoning, not an Islo-specific measured fact.)*
- **Shareability.** One short, stable URL works everywhere: the last slide of a conference talk, a LinkedIn bio, a podcast host reading it aloud, a QR code on a sticker. The page can change; the link never has to.
- **Zero infra cost.** GitHub Pages is free static hosting with HTTPS — no server, no build pipeline, no maintenance bill. For a single-page funnel that's all you need.
- **Measurability.** The funnel has two measurable ends: UTM-tagged traffic arriving at islo.dev (web analytics) and YOSSI150 redemptions reported by the chosen redemption path. Comparing the two gives a real conversion rate. *(Whether islo.dev runs PostHog specifically was not verified in research — see Section 7.)*

## 3. How to deliver +$150 in credits

Islo's app bundle confirms Paddle Billing is live (live client token + three credit-pack price IDs: `PaddleCreditPack10/50/100`). But there is a critical mechanic to get right first:

> **A Paddle discount reduces the price paid. It cannot grant bonus credits.** "Extra $150 in Islo credits" is an Islo-side balance top-up, so it requires logic in Islo's own backend — Paddle alone can't deliver this offer.

Two honest implementation paths, plus one trap to avoid:

### Path A — Islo-side promo-code field (simplest; Paddle untouched)
Islo adds a "promo code" input in the app (e.g., on the billing page). Redeeming `YOSSI150` while buying credits credits the account with +$150 directly, on top of the purchased pack. No Paddle configuration is required; redemptions are counted in Islo's own database.

### Path B — Paddle-integrated: customData + webhook → credits top-up
Tie the bonus to a credit-pack purchase. The landing page / app passes the code through checkout as custom data:

```js
Paddle.Checkout.open({
  items: [{ priceId: "pri_01kqs1kg4557tnvkywvj97yrqp", quantity: 1 }],
  customData: { promoCode: "YOSSI150" }
});
```

Paddle passes `customData` through to transaction webhooks. Islo's backend listens for `transaction.completed`, reads `data.custom_data.promoCode`, and when it equals `YOSSI150`, tops the buyer's balance up by +$150 in bonus credits (idempotently, keyed on transaction id).

Optional variant: also create a `YOSSI150` discount in Paddle purely as a tracking trigger (e.g., a token 1% percentage discount via **Catalog > Discounts** or `POST /discounts`, with `enabled_for_checkout: true` and `restrict_to` the three pack price IDs) so the code can be *typed* at checkout and the webhook handler keys off the discount id instead of customData. That gets Paddle-side redemption counts (`times_used`) for free, but note the discount itself is still only a price reducer — the +$150 still comes from Islo's webhook handler.

### Path C — what NOT to do: a flat $150-off Paddle discount
A `flat` $150 discount is **not the same offer**: it means the customer *pays $150 less*, not *gets $150 more* — and on packs priced at $10/$50/$100 a $150-off discount exceeds the pack price entirely, which is nonsensical. Don't ship this and call it "+$150 in credits."

### How customers redeem it
- **Path A**: type `YOSSI150` into Islo's promo-code field (once built). The landing-page link can carry `?coupon=YOSSI150` so the app can pre-fill it.
- **Path B**: the code rides along invisibly via `customData` (pre-applied from the landing-page CTA), or — with the tracking-discount variant — typed at checkout, which requires Paddle's "display discount field" checkout setting (`showAddDiscounts`) to be on; whether Islo's current checkout shows that field is untested, but it's irrelevant unless the typed-code variant is chosen.

### Launch checklist
- Build one redemption path from Section 3 and verify a real account receives +$150 bonus credits on top of a purchased credit pack.
- Confirm `?coupon=YOSSI150&utm_source=yossi&utm_medium=landing&utm_campaign=yossi150` survives the islo.dev → app.islo.dev signup/billing flow.
- Test the GitHub Pages landing page on desktop and mobile, including both copy-code buttons and all CTAs.
- Push the final `index.html` to `main`; GitHub Pages should update `https://zozo123.github.io/yossi-islo-promo/` within a minute or two.

## 4. Five use cases (one per persona)

All grounded in capabilities shown on islo.dev; where research was thin for a persona, it's marked.

1. **Software Developer — overnight feature work.** Kick off a coding agent in a persistent Islo computer before logging off; the agent works against real services on localhost (Postgres, Redis, FastAPI, Next.js — straight from Islo's homepage demo) and you resume the exact session in the morning: "Same filesystem. Same history. Same context." Homepage worked example: an 8-hour overnight session costs $2.46.
2. **DevOps — secret hygiene for agent fleets.** Run agents behind Islo's security gateway: agents only ever see placeholder credentials, real secrets are injected at egress with a per-request audit trail, and deny policies block exfiltration and banned hosts. That turns "an agent leaked a token" from an incident into a non-event with a log line.
3. **SWE — 24/7 support-ticket triage.** One of Islo's three named homepage use cases: agents pick up Linear/Jira tickets around the clock, reproduce the issue in a real Linux VM, and open PRs. The team reviews PRs in the morning instead of triaging tickets at midnight.
4. **MLE — sandboxed code interpreters / untrusted execution.** Islo's homepage lists untrusted code execution (user plugins, code interpreters) as a core use case — relevant when an ML product lets users or models execute arbitrary code. *(Research found no ML-specific Islo features — no GPU claims, no training-workflow docs — so beyond "isolated execution for model-generated code," this one is plausible-generic, not site-verified.)*
5. **AI SWE / agent engineer — agent verification.** Islo's "Ship code you can trust" use case: run your agents' output in real, isolated VMs ("not docker-in-docker") to verify behavior before merging, with large API responses trimmed before they hit agent context. SDKs in Python (`uv add islo`), TypeScript (`npm install @islo-labs/sdk`), and Go make it scriptable into agent pipelines.

## 5. crabbox.sh, from zero

**Crabbox** (crabbox.sh, GitHub: openclaw/crabbox, MIT-licensed, free) is an open-source "remote software testing and execution control plane" from Peter Steinberger's OpenClaw project. The problem: developers — and especially fleets of AI coding agents — running many test suites melt a local machine ("Too many agents, too many test suites, one very tired Mac"). The fix: keep editing locally, but run the heavy commands remotely. One command — `crabbox run -- pnpm test` — leases a cloud machine, syncs your uncommitted working tree (tracked/non-ignored files only), runs the command, streams output back, and releases the machine: *lease, sync, run, release*.

Mechanics: a Go CLI (it's Go + TypeScript, not Rust — the crab is just branding), a Broker (Cloudflare Worker with a "Fleet" Durable Object holding credentials, leases, and spend caps), and throwaway SSH runners. Crabbox is **not** a sandbox itself; isolation comes from the underlying provider. It supports 40+ providers across three modes: brokered (AWS/Azure/GCP/Hetzner), direct SSH, and **delegated** — where a sandbox provider handles everything. **Islo is a documented delegated provider** (docs/providers/islo.md): crabbox drives Islo's Go SDK/HTTP API (api.islo.dev) with an `ISLO_API_KEY`, no SSH or broker needed. That's the campaign tie-in: crabbox users are one config block away from running on Islo. A LinkedIn post by Yossi Eliaz announces "Happy to introduce capsules to crabbox.sh" (capsules = replayable bundles of failing CI runs), but the post body wasn't fetched, so the exact nature of the Incredibuild/Islo contribution is unverified. Install: `brew install openclaw/tap/crabbox`. Status: v0.29.0, ~607 GitHub stars. Honest notes: research found no paid/hosted crabbox service (you pay your own cloud providers, with broker-enforced spend caps), and versions before v0.12.0 had a critical secret-leak CVE (CVE-2026-8634, CVSS 9.3), fixed in v0.12.0.

## 6. Campaign examples

### (a) Conference talk — last slide
QR code + URL on the closing slide:
> **Try Islo — persistent, secure computers for your coding agents.**
> `yossieliaz.github.io/islo` → code **YOSSI150** = extra **$150 in Islo bonus credits**
> Plus $50 free credits on signup, no card required
> An 8-hour overnight agent session: $2.46.

*(Replace the URL with the actual GitHub Pages address once published.)*

### (b) LinkedIn / X post template
> My coding agents don't run on my laptop anymore. They run on Islo — real Linux VMs that stay warm 24/7, with a security gateway so the agent never sees a real secret.
>
> Kicked off an agent before bed, woke up to PRs. The 8-hour session cost $2.46.
>
> Start with $50 free credits (no card), and my code **YOSSI150** gets you an extra **$150 in Islo bonus credits** when buying credits: <landing-page-link-with-UTMs>

### (c) Newsletter / podcast mention blurb
> This episode is supported by my pick of the month: **Islo** (islo.dev), from the team behind Incredibuild. It gives AI coding agents persistent, isolated cloud computers — real VMs, not docker-in-docker — with a security gateway that swaps placeholder credentials for real secrets only at egress. Sign up for $50 in free credits, no card required, and use code **YOSSI150** for an extra **$150 in Islo bonus credits** when buying credits: <landing-page-link>.

*(Do not publish any of these until the +$150 redemption mechanism from Section 3 is actually live — the code does nothing today.)*

## 7. Measuring it

Two independent counters, compared monthly:

1. **Top of funnel — UTM traffic.** Every campaign link carries `?utm_source=yossi&utm_medium=<channel>&utm_campaign=yossi150` (vary `utm_medium`: `talk`, `linkedin`, `podcast`). These land in islo.dev's web analytics. *(The brief assumes PostHog; research did not verify which analytics tool islo.dev runs — confirm with the Islo team and that UTM params survive the islo.dev → app.islo.dev signup hop.)*
2. **Bottom of funnel — YOSSI150 redemptions.** Where these are counted depends on the Section 3 path: with Path A, redemptions live in Islo's own database (count of +$150 grants); with Path B, Islo's `transaction.completed` webhook handler counts them (and the optional tracking-discount variant adds Paddle's `times_used` on the discount object plus transaction reports filtered by discount). Either way, YOSSI150 redemptions are countable exactly.

Visits-with-UTM ÷ YOSSI150 redemptions = the campaign's visit-to-paid conversion rate, per channel. If visits are high but redemptions are zero, check first that the redemption mechanism is actually live and reachable (Section 3) before blaming the copy.

---

### Flagged unverified claims (kept honest in the text above)
- The YOSSI150 offer is now defined (extra $150 USD in Islo bonus credits, per Yossi), but **no redemption mechanism exists yet** — no promo/coupon feature anywhere on islo.dev or in the app bundle (grep-verified absence). One of the Section 3 paths must be built before launch.
- Whether Islo's live Paddle checkout displays a discount-code field is untested — relevant only if the optional typed-code tracking-discount variant of Path B is chosen; irrelevant under Path A or customData-only Path B.
- islo.dev's analytics stack (PostHog assumed in the brief) was not verified by research.
- Credit-pack denominations ($10/$50/$100) are inferred from Paddle config key names, not confirmed prices.
- Free/Team/Enterprise tier details come from press coverage, not islo.dev itself.
- The MLE use case is plausible-generic; no ML-specific Islo features were found.
- The exact nature of the Incredibuild/Islo contribution to crabbox "capsules" (Yossi's LinkedIn post) is unverified; crabbox star/release counts may be slightly stale.
