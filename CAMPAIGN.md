# YOSSI150 — Campaign Brief

## 1. The campaign in one paragraph

YOSSI150 is a personal promo-code campaign for **Islo** (islo.dev), Incredibuild's cloud sandbox platform for AI coding agents ("Everything coding agents need."). A simple landing page hosted free on GitHub Pages serves as Yossi's shareable hub: it explains Islo in one screen and funnels visitors to islo.dev/app.islo.dev with the code **YOSSI150**, which gives a discount at Paddle checkout (Paddle is Islo's confirmed live payment processor; billing is prepaid credit packs). One honest caveat up front: **the "150" in the code name is just a label — it does not define the discount.** The actual discount type and amount (e.g., 15% off, $15 off a credit pack) must be explicitly configured when the discount is created in Paddle (Section 3). Note also that as of this writing, no promo-code mechanism is visible on islo.dev or in the app bundle, so creating this discount in Paddle (and confirming the checkout shows a discount field) is a prerequisite, not an existing feature.

## 2. Why a personal landing page + link

- **Attribution.** A plain "check out islo.dev" mention is invisible in analytics. A personal link with UTM parameters (e.g., `https://islo.dev/?utm_source=yossi&utm_medium=landing&utm_campaign=yossi150`) tags every visitor, so signups from Yossi's audience are distinguishable from organic traffic.
- **Trust.** A human-named code ("YOSSI150") signals a personal recommendation rather than a generic sale ("SALE10"). People redeem codes from a person they follow at a higher rate because the code itself carries the endorsement. *(This is standard growth-marketing reasoning, not an Islo-specific measured fact.)*
- **Shareability.** One short, stable URL works everywhere: the last slide of a conference talk, a LinkedIn bio, a podcast host reading it aloud, a QR code on a sticker. The page can change; the link never has to.
- **Zero infra cost.** GitHub Pages is free static hosting with HTTPS — no server, no build pipeline, no maintenance bill. For a single-page funnel that's all you need.
- **Measurability.** The funnel has two measurable ends: UTM-tagged traffic arriving at islo.dev (web analytics) and YOSSI150 redemptions reported by Paddle. Comparing the two gives a real conversion rate. *(Whether islo.dev runs PostHog specifically was not verified in research — see Section 7.)*

## 3. How to set YOSSI150 up in Paddle

Islo's app bundle confirms Paddle Billing is live (live client token + three credit-pack price IDs: `PaddleCreditPack10/50/100`). Setup requires Islo's Paddle dashboard access.

### Dashboard steps
1. Go to **Paddle > Catalog > Discounts > New discount** (live: vendors.paddle.com; sandbox: sandbox-vendors.paddle.com).
2. **Description** (internal only, required): e.g., "Yossi personal promo".
3. **Type & Amount**: choose `percentage` (amount is a string, "0.01"–"100") or `flat` (requires a `currency_code`). **Decide the real discount here** — e.g., 15% off.
4. **Code**: `YOSSI150` (1–32 alphanumeric chars).
5. Leave **Checkout discount code** (`enabled_for_checkout`) on so customers can redeem it at checkout.
6. Optional: **Usage limit** (`usage_limit`), **Expiration date** (`expires_at`, RFC 3339), **Product restriction** (`restrict_to` — e.g., limit to the three credit-pack price IDs: `pri_01kqs1kg4557tnvkywvj97yrqp`, `pri_01kqs1m0c8jw45n5v2ywvt4mng`, `pri_01kqs1mgt5vppe8eej2kp10s5m`). `recur` matters only for subscriptions; Islo bills prepaid credit packs, so it can stay off.

### Minimal API example
```bash
curl -X POST https://api.paddle.com/discounts \
  -H "Authorization: Bearer $PADDLE_API_KEY" \   # needs discount.write scope
  -H "Content-Type: application/json" \
  -d '{
    "description": "Yossi personal promo - 15% off credit packs",
    "type": "percentage",
    "amount": "15",
    "code": "YOSSI150",
    "enabled_for_checkout": true,
    "restrict_to": ["pri_01kqs1kg4557tnvkywvj97yrqp",
                    "pri_01kqs1m0c8jw45n5v2ywvt4mng",
                    "pri_01kqs1mgt5vppe8eej2kp10s5m"]
  }'
```
Response returns an id like `dsc_01hv...` with status `active`. (Use sandbox-api.paddle.com to test first.)

### How customers apply it
- **Typed at checkout**: works if `enabled_for_checkout: true` AND the "display discount field on the checkout" option is on in Paddle > Checkout > Checkout settings (per-checkout: `showAddDiscounts`). **Unverified whether Islo's current checkout shows this field** — confirm with a test purchase.
- **Pre-applied via Paddle.js** (Islo-side change): `Paddle.Checkout.open({ discountCode: "YOSSI150", items: [...] })` — or `discountId`, but never both. Also `Paddle.Checkout.updateCheckout({ discountId })` and `data-discount-code` HTML attributes.
- **Pre-applied via hosted checkout URL**: append `?discount_code=YOSSI150` (takes precedence over `?discount_id=...`). This is the best option for the landing-page CTA: the button can deep-link with the discount already applied, so visitors never need to type anything. *(Requires Islo to use/expose hosted checkout URLs; unverified for their setup.)*

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
> `yossieliaz.github.io/islo` → code **YOSSI150** at checkout
> $50 in free credits on signup · no card required
> An 8-hour overnight agent session: $2.46.

*(Replace the URL with the actual GitHub Pages address once published.)*

### (b) LinkedIn / X post template
> My coding agents don't run on my laptop anymore. They run on Islo — real Linux VMs that stay warm 24/7, with a security gateway so the agent never sees a real secret.
>
> Kicked off an agent before bed, woke up to PRs. The 8-hour session cost $2.46.
>
> Start with $50 free credits (no card), and use my code **YOSSI150** when you buy credits: <landing-page-link-with-UTMs>

### (c) Newsletter / podcast mention blurb
> This episode is supported by my pick of the month: **Islo** (islo.dev), from the team behind Incredibuild. It gives AI coding agents persistent, isolated cloud computers — real VMs, not docker-in-docker — with a security gateway that swaps placeholder credentials for real secrets only at egress. Sign up for $50 in free credits, no card required, and use code **YOSSI150** at checkout for a discount: <landing-page-link>.

*(In all three, state the actual discount — "15% off" or similar — once it's configured in Paddle; don't imply "150" means 150 of anything.)*

## 7. Measuring it

Two independent counters, compared monthly:

1. **Top of funnel — UTM traffic.** Every campaign link carries `?utm_source=yossi&utm_medium=<channel>&utm_campaign=yossi150` (vary `utm_medium`: `talk`, `linkedin`, `podcast`). These land in islo.dev's web analytics. *(The brief assumes PostHog; research did not verify which analytics tool islo.dev runs — confirm with the Islo team and that UTM params survive the islo.dev → app.islo.dev signup hop.)*
2. **Bottom of funnel — Paddle redemptions.** Paddle tracks redemptions per discount code (`times_used` on the discount object, plus transaction reports filtered by discount), so YOSSI150 purchases are countable exactly.

Visits-with-UTM ÷ YOSSI150 redemptions = the campaign's visit-to-paid conversion rate, per channel. If visits are high but redemptions are zero, check first whether the checkout actually exposes the discount (Section 3 caveat) before blaming the copy.

---

### Flagged unverified claims (kept honest in the text above)
- The discount amount behind YOSSI150 does not exist yet; "15% off" is a placeholder until configured in Paddle.
- No promo/coupon mechanism currently exists anywhere on islo.dev or in the app bundle (grep-verified absence); whether Islo's live Paddle checkout displays a discount-code field is untested.
- Whether Islo uses hosted Paddle checkout URLs (needed for `?discount_code=` deep links) is unverified.
- islo.dev's analytics stack (PostHog assumed in the brief) was not verified by research.
- Credit-pack denominations ($10/$50/$100) are inferred from Paddle config key names, not confirmed prices.
- Free/Team/Enterprise tier details come from press coverage, not islo.dev itself.
- The MLE use case is plausible-generic; no ML-specific Islo features were found.
- The exact nature of the Incredibuild/Islo contribution to crabbox "capsules" (Yossi's LinkedIn post) is unverified; crabbox star/release counts may be slightly stale.
