---
title: Home
layout: home
nav_order: 1
toc: false
---

# ScalpelSpace Docs

<p class="ds-home-lead">Datasheets, design files and protocol references for
ScalpelSpace hardware.</p>

## Products

One datasheet per hardware release, with specifications, pinouts, design
files and release history.

{% include datasheet/product_cards.html %}

[All products]({% link products/index.md %})

## Protocols

Shared communication protocols and message schemes used across ScalpelSpace
products.

<div class="ds-catalog">
{%- assign protocols = site.pages | where: "parent", "Protocols" | sort: "nav_order" %}
{%- for p in protocols %}
<a class="ds-catalog-card" href="{{ p.url | relative_url }}">
<span class="ds-kicker">Protocol</span>
<span class="ds-catalog-card__name">{{ p.title }}</span>
{%- if p.summary %}
<span class="ds-catalog-card__tagline">{{ p.summary }}</span>
{%- endif %}
</a>
{%- endfor %}
</div>

## Firmware and software

Firmware, drivers and libraries are documented in each project's repository
on [GitHub](https://github.com/scalpelspace). Each datasheet links to the
repositories for that product.
