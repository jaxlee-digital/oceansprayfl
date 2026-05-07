---
layout: page
title: Service Area
subtitle: "Southwest Florida. Collier, Lee, and Hendry counties."
permalink: /service-area/
---

Ocean Spray FL is based in **Bonita Springs**, on the seam between
Lee and Collier counties. From there we cover most of coastal and
inland SWFL.

<figure class="embed-map">
  <iframe
    src="https://www.google.com/maps/d/u/8/embed?mid=1JMST0fg9GaeIW0YBW5wjjJTJib73tGI&ehbc=2E312F"
    title="Ocean Spray FL service area map"
    width="100%"
    height="480"
    style="border:0;border-radius:6px;"
    loading="lazy"
    referrerpolicy="no-referrer-when-downgrade"
    allowfullscreen>
  </iframe>
</figure>

## Cities we serve

{% for city in site.business.service_area %}- {{ city }}
{% endfor %}

## Counties

{% for county in site.business.counties %}- {{ county }}
{% endfor %}

If your property is nearby but not on the list, call us anyway.
We travel for the right project, and we'll let you know if it's a
fit before anyone wastes a trip.

## Why local matters

Coastal soil conditions in SWFL are not the same as the inland
mainland or other parts of Florida. The mix of sandy fill, tidal
fluctuation, and seasonal storm surge creates a specific kind of
void-and-erosion pattern behind seawalls and beneath slabs.
Working in the region every day means we know what we're looking at
when we get there.

[Request a free assessment →]({{ '/contact/' | relative_url }})
