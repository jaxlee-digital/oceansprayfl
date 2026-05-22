# WORKFLOW: oceanspray-site

Operating manual for the Ocean Spray FL marketing site at
`oceansprayfl.com`. Read this first before publishing or deploying
anything to this repo.

## TL;DR

- **Repo:** `jaxlee-digital/oceansprayfl` (GitHub)
- **Local path:** `~/.openclaw/workspace/oceanspray-site/` — its
  **own git repo**, nested inside the workspace. Use
  `git -C /home/sheehan/.openclaw/workspace/oceanspray-site …`
  for every git command (see `skills/git-safe-commit/`).
- **Live:** https://oceansprayfl.com (custom domain attached; `www`
  + `oceanspraydoam.us` + `oceansprayfoamfl.com` 301 to canonical)
- **Pages fallback:** https://jaxlee-digital.github.io/oceansprayfl/
- **Engine:** Jekyll via `github-pages` gem, default-branch build
- **Theme:** custom (in-repo)
- **Baseurl:** `""` (empty; custom domain is live)
- **Identity:** `jaxlee-digital` GitHub account, per-repo
  credential helper `/home/sheehan/.openclaw/bin/git-credential-jaxlee`
- **Business positioning:** seawall stabilization (lead),
  concrete lifting (cross-sell), metal building insulation
  (secondary). **Not** general spray-foam insulation.

Full background: `infra/oceanspray-site/README.md`.
Business context: `personal/business/ocean-spray-fl.md`.

## Common jobs

| Job | Skill / section |
|---|---|
| Publish a blog post | `skills/jekyll-publish-post/` |
| Edit a service / content page | edit → preview → screenshot → approve → push (`skills/jekyll-deploy/`) |
| Update business info (`_config.yml`) | Same flow, **always preview** — YAML typo breaks the whole build |
| Update `llms.txt` | `skills/agentic-browsing/` |
| Audit accessibility | `skills/a11y-audit/` |
| Shoot preview screenshots | `skills/jekyll-screenshot-preview/` |
| Submit sitemap to Google | See "Submit sitemap" section below |

### Submit sitemap to Google

1. Sanity check the live sitemap:
   ```bash
   curl -sI https://oceansprayfl.com/sitemap.xml | head -1
   curl -s https://oceansprayfl.com/sitemap.xml | grep -c '<loc>'
   ```
2. Open Google Search Console: https://search.google.com/search-console
3. Select property `oceansprayfl.com` (add + verify via DNS TXT
   if not present).
4. Left nav → **Sitemaps** → "Add a new sitemap" → `sitemap.xml`.
5. Re-submit after structural changes (new service page, URL
   renames). Routine posts: Google re-reads on its own crawl.

## Local preview (Podman)

Ruby is **not** installed on the host. Preview runs in a container.

```bash
SITE=/home/sheehan/.openclaw/workspace/oceanspray-site

# One-time bundler config:
mkdir -p "$SITE/.bundle" && [ -f "$SITE/.bundle/config" ] || cat > "$SITE/.bundle/config" <<EOF
---
BUNDLE_PATH: "vendor/bundle"
EOF

# Start preview (detached, localhost-only):
podman run -d --name oceanspray-preview \
  --userns=keep-id \
  -v "$SITE":/site:Z \
  -w /site \
  -p 127.0.0.1:4000:4000 \
  -e HOME=/tmp \
  docker.io/library/ruby:3.2 \
  sh -c "bundle install --quiet && bundle exec jekyll serve --host 0.0.0.0"

# Browse: http://localhost:4000/
```

Management:

```bash
podman logs -f oceanspray-preview
podman stop oceanspray-preview
podman start oceanspray-preview
podman rm -f oceanspray-preview
```

Full Podman recipe and rationale: `infra/oceanspray-site/README.md`.

### SELinux + Podman preview gotcha

The Podman preview container mounts the site with `:Z`, assigning
an SELinux `container_file_t` label. The OpenClaw `edit` tool
writes files through a temp path and they land with `user_tmp_t`,
which the container can't read. The next rebuild fails with:

```
Error: Permission denied @ rb_sysopen - /site/assets/css/main.scss
```

Restore the label after editing while the preview is running:

```bash
SITE=/home/sheehan/.openclaw/workspace/oceanspray-site
stat -c %C "$SITE/assets/css/main.scss"
chcon --reference="$SITE/Gemfile" "$SITE/assets/css/main.scss"
```

Batch sweep (skips `_site`, `vendor`, `.git`, `.jekyll-cache`):

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

Does not affect git or the live deploy.

## Accessibility & standards

**WCAG 2.2 AA on everything user-facing.** Full checklist:
`jaxlee-site/AGENT.md` "Accessibility & standards". Audit tool:
`skills/a11y-audit/`.

Site-specific notes:

- Service pages are the conversion path — CTAs reachable by
  keyboard, programmatic names on icon-only buttons.
- Phone/email links keep visible text + `tel:` / `mailto:` hrefs.
- Hero / service photos need descriptive `alt`; decorative imagery
  uses `alt=""` + `aria-hidden="true"`.
- Forms need visible labels, not placeholder-only inputs.
- Brand palette skews light — verify contrast on tinted backgrounds.

Run `skills/a11y-audit/` before any deploy that touches templates,
layouts, or new pages. Routine blog posts skip the audit unless
they introduce new components.

## Repo-specific rules

- **`baseurl` is empty.** Custom domain is attached. Do not
  reintroduce `/oceansprayfl` — it breaks every link.
- **Internal links** must use `{{ site.baseurl }}/path/`.
- **Business info is in `_config.yml`** under `business:`.
  Templates reference it as `site.business.<field>`. Single source
  of truth — don't duplicate phone/email in individual pages.
- **GitHub Pages plugins only.** Allowed: `jekyll-feed`,
  `jekyll-seo-tag`, `jekyll-sitemap`, `jekyll-paginate`.
- **Positioning matters.** Do not add general residential
  spray-foam insulation content. The pivot away is intentional —
  see `personal/business/ocean-spray-fl.md`.

## Approval gate

- Drafts only until approved. Finished work goes up, explicit OK
  required before push.
- **Send desktop + tablet + mobile screenshots** with every visual
  change. Helper + viewports: `skills/jekyll-screenshot-preview/`.
- Acceptable approval phrases: "publish it", "ship it", "push",
  "looks good". Ambiguous → ask.

## Where things live

- `_posts/` — blog posts
- `_services/` — service-page collection (rendered at `/services/<slug>/`)
- `_areas/` — service-area collection (rendered at `/areas/<slug>/`)
- `_data/` — structured data (testimonials, FAQs, etc. if used)
- `_layouts/`, `_includes/`, `assets/` — theme
- `seawall-stabilization/`, `driveway-lifting/`, `pool-deck-lifting/`,
  `warehouse-slab-lifting/` — customer-facing service pages
  (top-level dirs with their own permalinks; `_services/` collection
  is the cross-service detail layer)
- `about.md`, `contact.md`, `privacy-policy.md`,
  `terms-of-service.md`, `service-area.md` — top-level pages
- `_config.yml` — site + business config
- `llms.txt` — curated site index for LLM crawlers
  (https://llmstxt.org/)

## Verify deploy

See `skills/jekyll-deploy/`. Quick site-specific checks:

```bash
curl -sI https://oceansprayfl.com/ | head -5
curl -s https://oceansprayfl.com/sitemap.xml | grep -m1 lastmod
curl -sI https://oceansprayfl.com/llms.txt | head -3   # if llms.txt was touched
```

## Related

- `infra/oceanspray-site/README.md` — full infra notes
- `personal/business/ocean-spray-fl.md` — business context
- `skills/jekyll-publish-post/SKILL.md`
- `skills/jekyll-deploy/SKILL.md`
- `skills/git-safe-commit/SKILL.md`
- `skills/jekyll-screenshot-preview/SKILL.md`
- `skills/a11y-audit/SKILL.md`
- `skills/agentic-browsing/SKILL.md`
