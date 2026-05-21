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
  sh -c "bundle install --quiet && bundle exec jekyll serve --host 0.0.0.0 --baseurl /oceansprayfl"

# Browse: http://localhost:4000/oceansprayfl/   (trailing slash matters)
```

Management:

```bash
podman logs -f oceanspray-preview
podman stop oceanspray-preview
podman start oceanspray-preview
podman rm -f oceanspray-preview
```

URL gotcha: with `baseurl: /oceansprayfl`, the site is at
`/oceansprayfl/` **with** the trailing slash. No slash → 404.
Goes away once the custom domain is attached.

Full Podman recipe and rationale: `infra/oceanspray-site/README.md`.

## Repo-specific rules

- **`baseurl` is `/oceansprayfl`.** Don't blank it until DNS
  is on Pages.
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

For this site there's an extra wrinkle — it's a **business**
site for a venture with Adam. Anything that affects positioning,
pricing, service claims, or warranty language should get an extra
"is Adam aware?" beat from Sheehan before shipping. Don't push
business-substantive copy on Sheehan's nod alone unless he
confirms Adam is in the loop.

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
