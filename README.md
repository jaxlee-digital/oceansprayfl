# oceanspray-site

Marketing site for **Ocean Spray FL** - Jekyll site intended for
GitHub Pages, mirroring the structure of `jaxlee-site`.

## What it is

- **Live URL (planned):** `https://jaxlee-digital.github.io/oceansprayfl`
- **Custom domain:** `oceansprayfl.com` (not yet attached - set
  `baseurl: ""` in `_config.yml` when it is)
- **Repo (planned):** `https://github.com/jaxlee-digital/oceansprayfl`
- **Local clone:** `~/.openclaw/workspace/oceanspray-site`
- **Engine:** Jekyll via the `github-pages` gem
- **Theme:** custom (lives in repo: `_layouts`, `_includes`, `assets/css/main.scss`)

## Positioning

Refocused away from general spray-foam insulation toward:

1. **Seawall stabilization** (sole service focus, polyurethane injection)

Concrete lifting and metal building insulation have been **removed** from
the site. Seawall-adjacent slab leveling is still mentioned as an
add-on within seawall jobs, but not promoted as a standalone service.

## Structure

```
oceanspray-site/
├── _config.yml
├── Gemfile
├── README.md
├── index.md                    # home (uses _layouts/home.html)
├── about.md
├── contact.md
├── service-area.md
├── services/
│   └── index.md                # services index page
├── blog/
│   └── index.html              # blog archive (paginated)
├── _services/                  # service deep-dive collection
│   ├── seawall-stabilization.md
│   ├── concrete-lifting.md
│   └── metal-building-insulation.md
├── _posts/
│   ├── 2026-02-13-iguanas-damaging-swfl-seawalls.md
│   ├── 2025-12-04-ground-control-seawall-foam-injection.md
│   ├── 2025-08-31-red-iron-metal-building-project.md
│   └── 2025-07-24-foam-jacking-vs-mudjacking.md
├── _data/
│   ├── nav.yml                 # main navigation
│   └── services.yml            # home-page service cards
├── _layouts/
│   ├── default.html            # site shell (header, footer, CTA strip)
│   ├── home.html
│   ├── page.html
│   ├── post.html
│   ├── service.html
│   └── archive.html
├── _includes/
│   └── post-card.html
└── assets/
    └── css/
        └── main.scss           # custom stylesheet (no external theme)
```

## Local preview

Ruby is **not** installed on the host. Previews run in a Podman
container:

```bash
cd ~/.openclaw/workspace/oceanspray-site

mkdir -p .bundle && cat > .bundle/config <<EOF
---
BUNDLE_PATH: "vendor/bundle"
EOF

podman run -d --name oceanspray-preview \
  --userns=keep-id \
  -v "$PWD":/site:Z \
  -w /site \
  -p 127.0.0.1:4000:4000 \
  -e HOME=/tmp \
  docker.io/library/ruby:3.2 \
  sh -c "bundle install --quiet && bundle exec jekyll serve --host 0.0.0.0 --baseurl /oceansprayfl"

# First run: ~60s for bundle install. Then:
# http://localhost:4000/oceansprayfl/   (trailing slash required)

# Stop / start / remove:
podman stop oceanspray-preview
podman start oceanspray-preview
podman rm -f oceanspray-preview
```

Full notes (why each flag, host-Ruby alternative, gotchas) in
`infra/oceanspray-site/README.md`.

## Deploy

GitHub Pages **default branch build** (no Actions workflow). Push to
the default branch and Pages rebuilds within 1-2 minutes.

## Required asset additions

The site references images that **do not exist yet** in `assets/`.
Add these before going live (placeholder gradients render until then):

- `assets/images/hero-seawall.jpg` - wide hero image of a seawall /
  coastal site (used in `_layouts/home.html` background)
- `assets/images/services/seawall-cover.jpg` - service card / detail
- `assets/images/services/concrete-cover.jpg` - service card / detail
- `assets/images/services/metal-building-cover.jpg` - service card / detail
- `assets/favicon.ico` - favicon

Plus optional post cover images for each `_posts/*.md` entry
(`cover:` front-matter field).

Recommended source: pull from the existing
`oceansprayfl.com` Squarespace site (Sheehan owns those images),
or replace with new on-site photos as they're taken.

## Contact info

Pulled from the existing oceansprayfl.com Squarespace contact page.
Stored in `_config.yml` under `business:`. Update there to change
everywhere on the site.

- Phone: (239) 444-7792
- Email: Adam@OceanSprayFL.com
- HQ: Bonita Springs, FL

## Custom domain handoff (later)

When `oceansprayfl.com` gets pointed at GitHub Pages:

1. Add a `CNAME` file at the repo root containing `oceansprayfl.com`.
2. Set `baseurl: ""` in `_config.yml` (currently `/oceansprayfl`).
3. Update `url:` in `_config.yml` to `https://oceansprayfl.com`.
4. Configure DNS (A records or CNAME at the registrar).
5. Enable HTTPS in GitHub Pages settings once DNS propagates.

## How to pause / edit / delete

- **Pause publishing:** GitHub → repo Settings → Pages → set Source
  to "None".
- **Edit content:** push commits to the default branch; rebuild is
  automatic.
- **Take it offline entirely:** archive or delete the repo.

## Related

- `infra/oceanspray-site/README.md` - operational doc (deploy, DNS, etc.)
- `personal/business/ocean-spray-fl.md` - business context, pivot notes
- Sister site: `jaxlee-site/` - same architecture, different domain
