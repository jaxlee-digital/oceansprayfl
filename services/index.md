---
layout: page
title: Services
subtitle: "Polyurethane injection, done right — for seawalls, slabs, and metal buildings."
permalink: /services/
---

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
