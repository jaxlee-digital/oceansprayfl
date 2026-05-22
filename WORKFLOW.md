# WORKFLOW: oceanspray-site

Operating manual for the Ocean Spray FL marketing site at
`jaxlee-digital.github.io/oceansprayfl` (custom domain
`oceansprayfl.com` planned, not yet attached). Read this first
before publishing or deploying anything to this repo.

## TL;DR

- **Repo:** `jaxlee-digital/oceansprayfl` (GitHub)
- **Local path:** `~/.openclaw/workspace/oceanspray-site/` — this is
  its **own git repo**, nested inside the workspace but not part of
  the workspace repo. **Always target it explicitly with `git -C
  /home/sheehan/.openclaw/workspace/oceanspray-site …`.** `cd` does
  not persist across exec calls in the OpenClaw gateway, so `cd` +
  `git status` will silently run against the workspace repo and
  miss this site's changes.
- **Live (current):** https://jaxlee-digital.github.io/oceansprayfl
- **Live (planned):** https://oceansprayfl.com (DNS still on
  Squarespace; flip pending — see `infra/oceanspray-site/README.md`
  open work)
- **Engine:** Jekyll via `github-pages` gem, default-branch build
- **Theme:** custom (in-repo)
- **Baseurl:** `/oceansprayfl` (blank it only when custom domain
  is attached)
- **Identity:** `jaxlee-digital` GitHub account
- **Business positioning:** seawall stabilization (lead),
  concrete lifting (cross-sell), metal building insulation
  (secondary). **Not** general spray-foam insulation.

Full background: `infra/oceanspray-site/README.md`.
Business context: `personal/business/ocean-spray-fl.md`.

## Common jobs

### Publish a blog post

1. Read `skills/jekyll-publish-post/SKILL.md`.
2. Draft into `_drafts/<slug>.md`.
3. Preview locally (Podman — see below).
4. Show Sheehan; wait for explicit approval.
5. Promote draft → `_posts/YYYY-MM-DD-<slug>.md`.
6. Commit + push via `skills/git-safe-commit/` and
   `skills/jekyll-deploy/`.

### Edit a service page

Service pages live in `_services/` (rendered from `services/`
permalinks). Same flow as any layout/content change: edit,
preview, show diff, get approval, commit, push, verify.

### Update business info (phone, email, service area)

These come from `_config.yml` under `business:`. Touching this
file requires a full preview pass — a YAML typo here breaks
the whole site build.

### Update `llms.txt`

`llms.txt` is a plain-text summary of the site for LLM crawlers
(spec: https://llmstxt.org/). Lives at the repo root, served at
`/llms.txt`. Hand-curated, not generated.

**Why we ship it:** Chrome Lighthouse's *Agentic Browsing* audit
category explicitly checks for `llms.txt` at the domain root as a
discoverability signal. Cheap win, no downside.
Ref: https://developer.chrome.com/docs/lighthouse/agentic-browsing/scoring

Structure (Markdown, but served as text):

```markdown
# Ocean Spray FL

> One-line positioning: seawall stabilization (lead), concrete
> lifting (cross-sell), metal building insulation (secondary).
> Serving SW Florida.

Optional short paragraph of context.

## Services

- [Seawall stabilization](https://oceansprayfl.com/seawall-stabilization/): polyurethane injection behind failing seawalls.
- [Driveway lifting](https://oceansprayfl.com/driveway-lifting/): foam-jacking sunken slabs.
- [Pool deck lifting](https://oceansprayfl.com/pool-deck-lifting/)
- [Warehouse slab lifting](https://oceansprayfl.com/warehouse-slab-lifting/)

## About

- [About](https://oceansprayfl.com/about/)
- [Service area](https://oceansprayfl.com/service-area/)
- [Contact](https://oceansprayfl.com/contact/)

## Optional

- [Privacy policy](https://oceansprayfl.com/privacy-policy/)
- [Terms of service](https://oceansprayfl.com/terms-of-service/)
```

Rules:

- **Hand-edit only.** Don't auto-generate from `_config.yml` or the
  sitemap. The point is a curated, opinionated index.
- **Front matter required** so Jekyll doesn't try to process it as
  HTML. Use `--- \n permalink: /llms.txt \n sitemap: false \n ---`
  at the top, then the Markdown body. (Pages will still serve it
  with `Content-Type: text/plain` because of the `.txt` extension.)
- **Absolute URLs only** in the link list. LLM crawlers may fetch
  this without site context.
- **Update when a service page is added, removed, or renamed.**
  Routine blog posts do not need an `llms.txt` change.
- **Verify after deploy:**
  ```bash
  curl -sI https://oceansprayfl.com/llms.txt | head -3
  curl -s  https://oceansprayfl.com/llms.txt | head -20
  ```
  Expect `200` and `Content-Type: text/plain`.

Optional companion: `llms-full.txt` for the expanded, full-content
version. Skip unless Sheehan wants it; the short `llms.txt` covers
the standard use case.

### Agentic browsing readiness (Lighthouse)

Chrome ships an *Agentic Browsing* category in Lighthouse that
scores how machine-friendly the site is. No weighted 0-100; it
reports a pass ratio across deterministic checks. Worth tracking
as the agentic web tooling matures.

What it audits:

- **`llms.txt`** at the domain root. Covered above.
- **WebMCP tool registration.** Declarative (HTML) and imperative
  (JS) tools surfaced via the WebMCP API. Not implementing this
  yet for Ocean Spray — the site is brochureware, no forms or
  actions worth exposing as agent tools. Revisit if we add a
  quote/booking flow.
- **Accessibility tree quality.** Every interactive element needs
  a programmatic name, valid roles, valid parent-child structure,
  and must not be hidden from the a11y tree while interactive.
  This is just "good semantic HTML + ARIA" — same hygiene the
  regular Lighthouse a11y audit already enforces.
- **Cumulative Layout Shift (CLS).** Images need `width`/`height`,
  no late-injected content above the fold. Already on the
  performance audit, doubly relevant here.

How to run it:

```bash
# Headless Lighthouse against the live site, agentic category only
podman run --rm --network host \
  -v /tmp:/out:Z \
  docker.io/femtopixel/google-lighthouse \
  https://oceansprayfl.com/ \
  --only-categories=agentic-browsing \
  --output=json --output-path=/out/lh-agentic.json \
  --chrome-flags="--headless --no-sandbox"
```

Then `jq '.categories["agentic-browsing"]' /out/lh-agentic.json` to
see the pass ratio. Category id may shift while the feature is
still emerging — confirm with `--list-all-audits` if it errors.

When to run:

- Before custom-domain cutover — baseline.
- After any structural change (new service page, layout edit, new
  third-party script).
- Quarterly otherwise.

Do not chase a perfect ratio. WebMCP is the big unchecked box and
we're skipping it on purpose for now. Track the result in
`infra/oceanspray-site/README.md` so we can see drift over time.

### Submit sitemap to Google

Sitemap is auto-generated by the `jekyll-sitemap` plugin at
`/sitemap.xml`. URLs:

- Custom domain (current): https://oceansprayfl.com/sitemap.xml
- Pages fallback: https://jaxlee-digital.github.io/oceansprayfl/sitemap.xml

Steps:

1. Sanity check the live sitemap responds 200 and has entries:
   ```bash
   curl -sI https://oceansprayfl.com/sitemap.xml | head -1
   curl -s https://oceansprayfl.com/sitemap.xml | grep -c '<loc>'
   ```
2. Open Google Search Console: https://search.google.com/search-console
3. Select property `oceansprayfl.com` (or add + verify it first via
   DNS TXT if not present — registrar is wherever DNS lives post
   Squarespace cutover).
4. Left nav → **Sitemaps**.
5. Under "Add a new sitemap," enter: `sitemap.xml`
6. Submit. Status should flip to **Success** within a few minutes
   (sometimes hours for first submission).
7. Re-submit after any structural change (new service page, new
   blog post category, URL renames). Routine post additions do not
   require re-submission; Google re-reads on its own crawl.

### Cut over to custom domain (oceansprayfl.com)

Pending task. When ready:

1. Confirm Squarespace email forwarding survives or migrate
   the mailbox.
2. Add `CNAME` file with `oceansprayfl.com`.
3. Set `baseurl: ""` in `_config.yml`.
4. Update `url:` to `https://oceansprayfl.com`.
5. Update DNS (Squarespace → Pages — A records to GitHub's IPs
   + CNAME for `www`).
6. Wait for Pages to provision Let's Encrypt cert.
7. Verify https + http→https redirect.
8. Cancel Squarespace billing on confirmation from Adam.

Coordinate with Adam on timing. See open work in
`infra/oceanspray-site/README.md`.

## Local preview (Podman)

Ruby is **not** installed on the host. Preview runs in a
container:

```bash
cd ~/.openclaw/workspace/oceanspray-site

# One-time bundler config (if .bundle/config doesn't exist):
mkdir -p .bundle && cat > .bundle/config <<EOF
---
BUNDLE_PATH: "vendor/bundle"
EOF

# Start preview (detached, localhost-only):
podman run -d --name oceanspray-preview \
  --userns=keep-id \
  -v "$PWD":/site:Z \
  -w /site \
  -p 127.0.0.1:4000:4000 \
  -e HOME=/tmp \
  docker.io/library/ruby:3.2 \
  sh -c "bundle install --quiet && bundle exec jekyll serve --host 0.0.0.0"

# Browse: http://localhost:4000/   (no baseurl, custom domain is live)
```

Management:

```bash
podman logs -f oceanspray-preview
podman stop oceanspray-preview
podman start oceanspray-preview
podman rm -f oceanspray-preview
```

Full Podman recipe and rationale: `infra/oceanspray-site/README.md`.

## Accessibility & standards — hard requirement

Same baseline as the rest of Sheehan's web work: **WCAG 2.2 AA
minimum on everything user-facing.** Full checklist lives in
`jaxlee-site/AGENT.md` under "Accessibility & standards" — do not
duplicate it here, read that file and apply it.

Quick reminders specific to this site:

- **Service pages** are the conversion path. Every CTA button must
  be reachable by keyboard and have a programmatic name (no
  icon-only buttons without `aria-label`).
- **Phone/email links** in templates must keep visible text plus
  `tel:` / `mailto:` hrefs. Don't hide the number behind an icon.
- **Hero images and service photos** need real `alt` text describing
  the work shown, not "hero image" or "photo". Decorative
  background imagery gets `alt=""` + `aria-hidden="true"`.
- **Forms** (contact form, quote request if added) need visible
  labels, not placeholder-only inputs.
- **Color contrast** — the brand palette skews light; verify text
  on tinted backgrounds with a contrast checker before shipping.
- **Run `skills/a11y-audit/`** before any deploy that touches
  templates, layouts, or new pages. Routine blog posts skip the
  audit unless they introduce new components.

If a pattern can't ship accessibly, don't ship it. Flag and propose
an accessible alternative.

## Repo-specific rules

- **`baseurl` is empty.** Custom domain is attached. Do not
  reintroduce `/oceansprayfl` — it will break every link.
- **Internal links** must use `{{ site.baseurl }}/path/`.
- **Business info is in `_config.yml`** under `business:`.
  Templates reference it as `site.business.<field>`. Single
  source of truth — don't duplicate phone/email/etc. in
  individual pages.
- **GitHub Pages plugins only.** Allowed: `jekyll-feed`,
  `jekyll-seo-tag`, `jekyll-sitemap`, `jekyll-paginate`.
- **Positioning matters.** Do not add general residential
  spray-foam insulation content. The pivot away from that is
  intentional — see `personal/business/ocean-spray-fl.md`.

## SELinux + Podman preview gotcha

The Podman preview container mounts the site with `:Z`, which assigns
an SELinux `container_file_t` label so only this container can read
the files. The OpenClaw `edit` tool writes files through a temp path
and the result lands with `user_tmp_t` instead. The container then
fails the next rebuild with:

```
Error: Permission denied @ rb_sysopen - /site/assets/css/main.scss
```

After editing any file in this repo while the preview container is
running, restore the label:

```bash
SITE=/home/sheehan/.openclaw/workspace/oceanspray-site
# Check
stat -c %C "$SITE/assets/css/main.scss"
# Fix (use any file the container already reads as the reference)
chcon --reference="$SITE/Gemfile" "$SITE/assets/css/main.scss"
```

For a batch of edits, sweep the whole repo (skips _site, vendor,
.git, .jekyll-cache):

```bash
SITE=/home/sheehan/.openclaw/workspace/oceanspray-site
find "$SITE" \
  -path "$SITE/_site" -prune -o \
  -path "$SITE/vendor" -prune -o \
  -path "$SITE/.git" -prune -o \
  -path "$SITE/.jekyll-cache" -prune -o \
  -type f -print 2>/dev/null \
  | while read f; do
      ctx=$(stat -c %C "$f" 2>/dev/null)
      if echo "$ctx" | grep -q "user_tmp_t"; then
        chcon --reference="$SITE/Gemfile" "$f"
      fi
    done
```

Does not affect git or the live deploy. Only matters for the local
Podman preview during an editing session.

## Git ops — location matters

**Use `git -C <absolute-path>` for every git command.** `cd` does
not persist between exec calls in the OpenClaw gateway — each
command re-enters from `workdir`, so `cd oceanspray-site && git …`
actually runs git from the workspace root. The workspace repo is
a separate git repo that tracks this whole tree as content, so a
wrong-path `git add -A` will silently stage workspace files.

Right pattern:

```bash
SITE=/home/sheehan/.openclaw/workspace/oceanspray-site
git -C "$SITE" status
git -C "$SITE" add <files>
git -C "$SITE" commit -m "..."
git -C "$SITE" push origin main
```

Sanity check before any add/commit:

```bash
git -C "$SITE" rev-parse --show-toplevel
# must print: /home/sheehan/.openclaw/workspace/oceanspray-site
```

If it prints the workspace path instead, you're targeting the
wrong repo — stop and re-check the `-C` path.

## Git auth — required setup

This repo uses a per-repo credential helper. Without it, the
first push fails with `could not read Username for
'https://github.com'`. Verify:

```bash
git -C ~/.openclaw/workspace/oceanspray-site config --get-all credential.helper
# expected: /home/sheehan/.openclaw/bin/git-credential-jaxlee
```

If missing, set it **per-repo only** (never globally):

```bash
git -C ~/.openclaw/workspace/oceanspray-site config \
  credential.helper /home/sheehan/.openclaw/bin/git-credential-jaxlee
```

Full identity policy: `personal/security/identities.md`.

## Approval gate

Same as jaxlee-site: drafts only until approved, only finished
work goes up, explicit OK required before push.

**Always send a screenshot of the local preview before asking for
deploy approval.** Sheehan needs to see the rendered result, not
just a diff or a description. Pattern:

1. Make changes, restart preview if `_config.yml` was touched.
2. Confirm the rebuild was clean (`podman logs --tail 10 oceanspray-preview`).
3. Take a screenshot of the affected page(s):

   ```bash
   mkdir -p /tmp/oceanspray-preview-shots && chmod 777 /tmp/oceanspray-preview-shots
   podman run --rm --network host --user 0:0 \
     -v /tmp/oceanspray-preview-shots:/out:Z \
     docker.io/zenika/alpine-chrome --no-sandbox \
     --hide-scrollbars --window-size=1280,900 \
     --screenshot=/out/home.png \
     http://localhost:4000/
   cp /tmp/oceanspray-preview-shots/home.png \
     /home/sheehan/.openclaw/workspace/oceanspray-rebrand-preview.png
   ```

4. Attach via `MEDIA:/home/sheehan/.openclaw/workspace/<file>.png`
   in the reply.
5. Wait for explicit "push" before committing.
6. Clean up the workspace-root preview image after push.

## Where things live

- `_posts/` — blog posts
- `_services/` — service-page content (collection)
- `_areas/` — service-area pages (collection)
- `_data/` — structured data (testimonials, FAQs, etc. if used)
- `_layouts/`, `_includes/`, `assets/` — theme
- `services/`, `driveway-lifting/`, `pool-deck-lifting/`,
  `seawall-stabilization/`, `warehouse-slab-lifting/` —
  permalinked service URLs
- `about.md`, `contact.md`, `privacy-policy.md`,
  `terms-of-service.md`, `service-area.md` — top-level pages
- `_config.yml` — site + business config
- `llms.txt` — curated site index for LLM crawlers
  (https://llmstxt.org/)

## Verify deploy

```bash
curl -sI https://jaxlee-digital.github.io/oceansprayfl/ | head -5
curl -s https://jaxlee-digital.github.io/oceansprayfl/sitemap.xml | grep -m1 lastmod
```

After custom-domain cutover, swap the host.

## Related

- `infra/oceanspray-site/README.md` — full infra notes
- `personal/business/ocean-spray-fl.md` — business context
- `skills/jekyll-publish-post/SKILL.md`
- `skills/jekyll-deploy/SKILL.md`
- `skills/git-safe-commit/SKILL.md`
