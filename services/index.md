---
layout: page
title: Services
subtitle: "Polyurethane injection, done right. For seawalls, slabs, and metal buildings."
permalink: /services/
description: >-
  Ocean Spray Foam services: seawall stabilization, concrete lifting
  (driveways, pool decks, warehouse slabs), and metal building
  insulation across Southwest Florida.
---

Polyurethane injection is the technology behind everything we do.
Same resin, applied three ways, across Southwest Florida.

<div class="services-grid__list" style="display:grid; grid-template-columns:repeat(auto-fit, minmax(280px, 1fr)); gap:1.5rem; list-style:none; padding:0;">
{% for svc in site.data.services %}
  <a class="service-card" href="{{ svc.url | relative_url }}" style="display:block; background:#fff; border:1px solid var(--c-line); border-radius:14px; overflow:hidden; box-shadow:var(--shadow-sm); color:inherit; text-decoration:none;">
    <div class="service-card__cover" style="height:180px; background-image:url('{{ svc.cover | relative_url }}'); background-size:cover; background-position:center;"></div>
    <div class="service-card__body" style="padding:1.5rem;">
      <h3>{{ svc.title }}</h3>
      <p>{{ svc.blurb }}</p>
      <p class="service-card__more" style="font-weight:600; color:var(--c-brand);">Learn more →</p>
    </div>
  </a>
{% endfor %}
</div>

## Service × city directory

Quick-reference index for our team. Every city we serve, every
service we run there, one click away. Use these pages before going
out to a quote so you've got the local context and the diagnostic
breakdown in front of you.

<div class="service-matrix" markdown="0">

  <div class="service-matrix__col">
    <h3>Seawall stabilization</h3>
    <ul>
      {% for r in site.data.regions %}
        {% if r.weight.seawall and r.weight.seawall != "skip" %}
          <li><a href="{{ '/seawall-stabilization/' | append: r.slug | append: '/' | relative_url }}">{{ r.name }}{% if r.weight.seawall == "flagship" %} <span class="matrix-tag">★ flagship</span>{% endif %}</a></li>
        {% endif %}
      {% endfor %}
    </ul>
  </div>

  <div class="service-matrix__col">
    <h3>Pool deck lifting</h3>
    <ul>
      {% for r in site.data.regions %}
        {% if r.weight.pool_deck and r.weight.pool_deck != "skip" %}
          <li><a href="{{ '/pool-deck-lifting/' | append: r.slug | append: '/' | relative_url }}">{{ r.name }}{% if r.weight.pool_deck == "flagship" %} <span class="matrix-tag">★ flagship</span>{% endif %}</a></li>
        {% endif %}
      {% endfor %}
    </ul>
  </div>

  <div class="service-matrix__col">
    <h3>Driveway lifting</h3>
    <ul>
      {% for r in site.data.regions %}
        {% if r.weight.driveway and r.weight.driveway != "skip" %}
          <li><a href="{{ '/driveway-lifting/' | append: r.slug | append: '/' | relative_url }}">{{ r.name }}{% if r.weight.driveway == "flagship" %} <span class="matrix-tag">★ flagship</span>{% endif %}</a></li>
        {% endif %}
      {% endfor %}
    </ul>
  </div>

  <div class="service-matrix__col">
    <h3>Warehouse slab lifting</h3>
    <ul>
      {% for r in site.data.regions %}
        {% if r.weight.warehouse and r.weight.warehouse != "skip" %}
          <li><a href="{{ '/warehouse-slab-lifting/' | append: r.slug | append: '/' | relative_url }}">{{ r.name }}{% if r.weight.warehouse == "flagship" %} <span class="matrix-tag">★ flagship</span>{% endif %}</a></li>
        {% endif %}
      {% endfor %}
    </ul>
  </div>

</div>

## What employees should pull before a quote

Each city × service page covers:

- **Local soil and seawall conditions** specific to that market
- **The diagnostic split** — when to recommend restoration vs replacement
- **Typical cost range** for the work in that area
- **Timeline and disruption** comparison vs the alternative
- **Common FAQs** customers ask about that service in that city
- **Neighborhoods and ZIP codes** we cover

Bring it up on your phone in the truck before you knock.
