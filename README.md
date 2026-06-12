# yossi-islo-promo

A single-page static promo site for the coupon code **YOSSI150** on [islo.dev](https://islo.dev) — Incredibuild's secure, persistent cloud sandbox platform for AI coding agents.

- `index.html` — fully self-contained (inline CSS, one small click-to-copy script, IBM Plex Mono via Google Fonts). No build step, no dependencies.
- `.nojekyll` — disables Jekyll processing on GitHub Pages.

## Deploy to GitHub Pages

### 1. Create the repo and push (gh CLI)

From this directory (already a git repo with one commit):

```bash
gh repo create yossi-islo-promo --public --source=. --push
```

That creates `https://github.com/USERNAME/yossi-islo-promo` and pushes `main`.

<details>
<summary>Manual alternative (without gh)</summary>

1. Create an empty public repo named `yossi-islo-promo` on github.com (no README/license).
2. ```bash
   git remote add origin https://github.com/USERNAME/yossi-islo-promo.git
   git push -u origin main
   ```
</details>

### 2. Enable GitHub Pages

Either via the UI:

1. Open the repo on GitHub → **Settings** → **Pages**.
2. Under **Build and deployment**, set **Source** to **Deploy from a branch**.
3. Branch: **main**, folder: **/(root)** → **Save**.

Or via the gh CLI:

```bash
gh api -X POST repos/USERNAME/yossi-islo-promo/pages \
  -f "source[branch]=main" -f "source[path]=/"
```

### 3. Visit the site

After a minute or two, the site is live at:

```
https://USERNAME.github.io/yossi-islo-promo/
```

(Replace `USERNAME` with your GitHub username. Check deploy status under the repo's **Actions** tab or Settings → Pages.)

## Notes

- All product claims on the page are sourced from islo.dev, docs.islo.dev, and launch press coverage (PR Newswire, DevOps.com, The New Stack).
- The crabbox.sh section reflects the public OpenClaw repo (`openclaw/crabbox`), where islo.dev is documented as a delegated-run provider.
