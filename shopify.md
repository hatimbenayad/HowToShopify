# The Complete Guide to Building Shopify Online Store 2.0 Themes: From Scratch to Production

> **Purpose of this Document:**  
> This file is a self-contained, technically rigorous, end-to-end blueprint for designing and developing a production-ready **Shopify Online Store 2.0 (OS 2.0)** theme.  
> Whenever you feed this document to an AI assistant or human developer, it provides all the architectural rules, directory structures, Liquid syntax, schema constraints, JSON template mechanics, AJAX APIs, performance guidelines, and common pitfalls needed to build any Shopify theme without missing steps.

---

## Table of Contents
1. [Core Principles & Architectural Tenets](#1-core-principles--architectural-tenets)
2. [Online Store 2.0 Directory Structure](#2-online-store-20-directory-structure)
3. [The Core Layout: `layout/theme.liquid`](#3-the-core-layout-layoutthemeliquid)
4. [Global Design System & Settings Schema](#4-global-design-system--settings-schema)
5. [Sections & The Section Schema Engine](#5-sections--the-section-schema-engine)
   - [Section Anatomy](#section-anatomy)
   - [The Schema Dictionary (Valid Setting Types)](#the-schema-dictionary-valid-setting-types)
   - [Section Blocks & Theme Editor Interactivity](#section-blocks--theme-editor-interactivity)
   - [Presets vs. Section Groups (`enabled_on` / `disabled_on`)](#presets-vs-section-groups-enabled_on--disabled_on)
6. [Section Groups: Header and Footer Architecture](#6-section-groups-header-and-footer-architecture)
7. [JSON Templates Engine (`templates/*.json`)](#7-json-templates-engine-templatesjson)
8. [Snippets & Reusable Components](#8-snippets--reusable-components)
9. [Shopify Liquid: Essential Objects, Filters & Tags](#9-shopify-liquid-essential-objects-filters--tags)
10. [JavaScript, Cart APIs & Section Rendering](#10-javascript-cart-apis--section-rendering)
11. [Product Catalog Data & CSV Specifications](#11-product-catalog-data--csv-specifications)
12. [Common Pitfalls & Fatal Bugs to Avoid](#12-common-pitfalls--fatal-bugs-to-avoid)
13. [Verification & Pre-Flight Checklist](#13-verification--pre-flight-checklist)
14. [Master Blueprint Prompt for Future AI Conversations](#14-master-blueprint-prompt-for-future-ai-conversations)

---

## 1. Core Principles & Architectural Tenets

When building a modern Shopify theme, adhere to these non-negotiable standards:

1. **Pure Standards, Zero Build-Step Dependency:**  
   Write clean, native Liquid, semantic HTML5, modern vanilla CSS (using CSS Custom Properties), and lightweight vanilla ES6+ JavaScript. Avoid frameworks like React, Vue, jQuery, or bloated CSS frameworks unless strictly required by the client. The theme must run out-of-the-box when zipped and uploaded to any Shopify store.

2. **Online Store 2.0 Everywhere:**  
   Every template (`index`, `product`, `collection`, `cart`, `page`, `search`, `404`) must be a `.json` template, not a legacy `.liquid` template. This enables merchants to add, reorder, configure, and remove sections freely on any page.

3. **Section-Scoped Styles & Encapsulation:**  
   Styles should be scoped using `#shopify-section-{{ section.id }}` or unique, predictable BEM class prefixes. Sections must never bleed layout styles into other sections.

4. **Accessibility (WCAG AA) & Mobile-First:**  
   - All interactive controls (drawers, carousels, modals, accordion disclosures) must support full keyboard navigation (`Tab`, `Esc`, `Enter`, `Space`).
   - Visually hidden labels (`.sr-only`) must exist on icon-only buttons.
   - Every image must output explicit `alt` text and responsive `srcset`.
   - Layouts must be tested from **360px up to 1920px+** without horizontal overflow.

5. **Internationalization (`locales/`):**  
   Never hardcode merchant-facing or customer-facing strings. Use the Liquid translation filter `{{ 'general.some_key' | t }}` with definitions located in `locales/en.default.json`.

---

## 2. Online Store 2.0 Directory Structure

Shopify enforces a strict, flat root structure. Subdirectories inside these folders are **not** permitted by Shopify's theme engine.

```
my-theme/
├── assets/                  # CSS stylesheets, JS bundles, static SVGs, images
│   ├── theme.css            # Global resets, design tokens, typography, utilities
│   ├── theme.js             # Global interactive logic (drawer, cart event dispatcher)
│   └── hero-slideshow.js    # Section-specific script
├── config/
│   ├── settings_schema.json # Defines global merchant theme settings (colors, fonts, etc.)
│   └── settings_data.json   # Current merchant configuration values & theme presets
├── layout/
│   ├── theme.liquid         # Primary HTML shell wrapping all templates
│   └── password.liquid      # Minimal layout for the password-protected gate page
├── locales/
│   ├── en.default.json      # English translations for customer-facing text
│   └── en.default.schema.json # Translations for Theme Customizer setting labels
├── sections/                # Reusable section components & section groups
│   ├── header-group.json    # Header section group (announcement bar + main header)
│   ├── footer-group.json    # Footer section group (main footer)
│   ├── announcement-bar.liquid
│   ├── header.liquid
│   ├── hero-slideshow.liquid
│   ├── collection-list.liquid
│   ├── product-carousel.liquid
│   ├── product-compare.liquid
│   ├── main-product.liquid
│   ├── main-collection.liquid
│   ├── main-cart.liquid
│   ├── main-page.liquid
│   ├── main-search.liquid
│   ├── main-404.liquid
│   └── footer.liquid
├── snippets/                # Reusable partials (rendered via {% render %})
│   ├── icon.liquid          # Unified SVG icon dictionary
│   ├── product-card.liquid  # Reusable product grid/carousel card
│   ├── price.liquid         # Formatted pricing & sale strikethrough logic
│   └── badge.liquid         # Discount, tag, and inventory status badges
└── templates/               # OS 2.0 JSON layout configurations & customer liquid files
    ├── index.json           # Homepage structure
    ├── product.json         # Product details page structure
    ├── collection.json      # Collection page structure
    ├── cart.json            # Cart page structure
    ├── page.json            # Static page structure
    ├── search.json          # Search results page structure
    ├── 404.json             # 404 Not Found page structure
    └── customers/           # Traditional Liquid templates required for customer accounts
        ├── account.liquid
        ├── activate_account.liquid
        ├── addresses.liquid
        ├── login.liquid
        ├── order.liquid
        ├── register.liquid
        └── reset_password.liquid
```

---

## 3. The Core Layout: `layout/theme.liquid`

`layout/theme.liquid` is the master container. Missing mandatory Liquid tags here will break Shopify analytics, third-party apps, or cause theme upload rejection.

### Complete Reference `layout/theme.liquid`:

```liquid
<!doctype html>
<html class="no-js" lang="{{ request.locale.iso_code }}">
  <head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width,initial-scale=1">
    <meta name="theme-color" content="{{ settings.color_accent | default: '#000000' }}">
    <link rel="canonical" href="{{ canonical_url }}">
    <link rel="preconnect" href="https://cdn.shopify.com" crossorigin>

    {%- if settings.favicon != blank -%}
      <link rel="icon" type="image/png" href="{{ settings.favicon | image_url: width: 32, height: 32 }}">
    {%- endif -%}

    <title>
      {{ page_title }}
      {%- if current_tags %} &ndash; tagged "{{ current_tags | join: ', ' }}"{% endif -%}
      {%- if current_page != 1 %} &ndash; Page {{ current_page }}{% endif -%}
      {%- unless page_title contains shop.name %} &ndash; {{ shop.name }}{% endunless -%}
    </title>

    {%- if page_description -%}
      <meta name="description" content="{{ page_description | escape }}">
    {%- endif -%}

    {%- render 'meta-tags' -%}

    <!-- CRITICAL MANDATORY TAG: Injects Shopify analytics, app scripts, preview bar -->
    {{ content_for_header }}

    <!-- Dynamic Theme CSS Tokens Generated From config/settings_schema.json -->
    <style>
      :root {
        --color-accent: {{ settings.color_accent | default: '#7B0F91' }};
        --color-bg: {{ settings.color_bg | default: '#F7F8FA' }};
        --color-surface: {{ settings.color_surface | default: '#FFFFFF' }};
        --color-text: {{ settings.color_text | default: '#111111' }};
        --color-muted: {{ settings.color_muted | default: '#6B7280' }};
        --color-border: {{ settings.color_border | default: '#D9DCE1' }};
        --color-sale: {{ settings.color_sale | default: '#E11D1D' }};
        --color-dark: {{ settings.color_dark | default: '#33373D' }};
        
        --radius-sm: {{ settings.radius_sm | default: 4 }}px;
        --radius-lg: {{ settings.radius_lg | default: 16 }}px;
        
        --font-body-family: {{ settings.type_body_font.family }}, {{ settings.type_body_font.fallback_families }};
        --font-body-weight: {{ settings.type_body_font.weight }};
        --font-heading-family: {{ settings.type_header_font.family }}, {{ settings.type_header_font.fallback_families }};
        --font-heading-weight: {{ settings.type_header_font.weight }};
      }
    </style>

    {{ 'theme.css' | asset_url | stylesheet_tag }}

    <script>
      document.documentElement.className = document.documentElement.className.replace('no-js', 'js');
      window.ShopifyTheme = {
        routes: {
          cart_add_url: '{{ routes.cart_add_url }}',
          cart_change_url: '{{ routes.cart_change_url }}',
          cart_update_url: '{{ routes.cart_update_url }}',
          cart_url: '{{ routes.cart_url }}',
          predictive_search_url: '{{ routes.predictive_search_url }}'
        },
        moneyFormat: {{ shop.money_format | json }}
      };
    </script>
    <script src="{{ 'theme.js' | asset_url }}" defer="defer"></script>
  </head>

  <body class="template-{{ template.name | handle }}">
    <!-- Accessible Skip to Content Link -->
    <a class="skip-to-content-link sr-only" href="#MainContent">
      {{ 'accessibility.skip_to_text' | t | default: 'Skip to content' }}
    </a>

    <!-- Header Section Group -->
    {% sections 'header-group' %}

    <!-- CRITICAL MANDATORY TAG: Renders page content from templates/*.json -->
    <main id="MainContent" class="content-for-layout focus-none" role="main" tabindex="-1">
      {{ content_for_layout }}
    </main>

    <!-- Footer Section Group -->
    {% sections 'footer-group' %}
  </body>
</html>
```

---

## 4. Global Design System & Settings Schema

Global settings live in `config/settings_schema.json`. These create the controls in **Shopify Admin → Theme Settings** (the gear icon).

### Standard Structure of `config/settings_schema.json`:

```json
[
  {
    "name": "theme_info",
    "theme_name": "Ignite",
    "theme_version": "1.0.0",
    "theme_author": "Developer",
    "theme_documentation_url": "https://example.com/docs",
    "theme_support_url": "https://example.com/support"
  },
  {
    "name": "Colors",
    "settings": [
      {
        "type": "color",
        "id": "color_accent",
        "label": "Accent Color",
        "default": "#7B0F91"
      },
      {
        "type": "color",
        "id": "color_bg",
        "label": "Page Background",
        "default": "#F7F8FA"
      },
      {
        "type": "color",
        "id": "color_surface",
        "label": "Card Surface",
        "default": "#FFFFFF"
      },
      {
        "type": "color",
        "id": "color_text",
        "label": "Main Text",
        "default": "#111111"
      },
      {
        "type": "color",
        "id": "color_muted",
        "label": "Muted Text",
        "default": "#6B7280"
      },
      {
        "type": "color",
        "id": "color_border",
        "label": "Borders",
        "default": "#D9DCE1"
      },
      {
        "type": "color",
        "id": "color_sale",
        "label": "Sale / Alert",
        "default": "#E11D1D"
      }
    ]
  },
  {
    "name": "Typography",
    "settings": [
      {
        "type": "font_picker",
        "id": "type_header_font",
        "label": "Heading Font",
        "default": "oswald_n7"
      },
      {
        "type": "font_picker",
        "id": "type_body_font",
        "label": "Body Font",
        "default": "inter_n4"
      }
    ]
  },
  {
    "name": "Corner Radiuses",
    "settings": [
      {
        "type": "range",
        "id": "radius_sm",
        "label": "Small Elements (Buttons, Inputs, Badges)",
        "min": 0,
        "max": 16,
        "step": 2,
        "unit": "px",
        "default": 4
      },
      {
        "type": "range",
        "id": "radius_lg",
        "label": "Large Containers (Hero, Cards, Modals)",
        "min": 0,
        "max": 32,
        "step": 2,
        "unit": "px",
        "default": 16
      }
    ]
  },
  {
    "name": "Social Media",
    "settings": [
      { "type": "url", "id": "social_twitter_link", "label": "X / Twitter" },
      { "type": "url", "id": "social_facebook_link", "label": "Facebook" },
      { "type": "url", "id": "social_instagram_link", "label": "Instagram" }
    ]
  }
]
```

---

## 5. Sections & The Section Schema Engine

### Section Anatomy
Every file in `sections/*.liquid` has up to three distinct layers:
1. **Liquid / HTML Markup**: Renders the dynamic DOM.
2. **CSS / Scoped Styling**: Enclosed in `<style>` tags or loaded via `<link>`/`theme.css`.
3. **`{% schema %}` Block**: Pure JSON describing the merchant controls.

```liquid
{%- comment -%} sections/hero-slideshow.liquid {%- endcomment -%}

<section 
  id="shopify-section-{{ section.id }}" 
  class="hero-slideshow" 
  data-autoplay="{{ section.settings.autoplay }}"
>
  <div class="hero-slideshow__container">
    {%- for block in section.blocks -%}
      <div class="hero-slideshow__slide" {{ block.shopify_attributes }}>
        <h2>{{ block.settings.heading | escape }}</h2>
        {%- if block.settings.button_link != blank -%}
          <a href="{{ block.settings.button_link }}" class="btn">
            {{ block.settings.button_label | escape }}
          </a>
        {%- endif -%}
      </div>
    {%- endfor -%}
  </div>
</section>

{% schema %}
{
  "name": "Hero slideshow",
  "tag": "section",
  "class": "section-hero",
  "settings": [
    {
      "type": "checkbox",
      "id": "autoplay",
      "label": "Enable autoplay",
      "default": true
    }
  ],
  "blocks": [
    {
      "type": "slide",
      "name": "Slide",
      "limit": 5,
      "settings": [
        {
          "type": "text",
          "id": "heading",
          "label": "Heading",
          "default": "Tech Redefined"
        },
        {
          "type": "url",
          "id": "button_link",
          "label": "Button link"
        },
        {
          "type": "text",
          "id": "button_label",
          "label": "Button text",
          "default": "SHOP NOW"
        }
      ]
    }
  ],
  "presets": [
    {
      "name": "Hero slideshow",
      "blocks": [
        { "type": "slide" }
      ]
    }
  ]
}
{% endschema %}
```

---

### The Schema Dictionary (Valid Setting Types)

> ⚠️ **CRITICAL WARNING:** Using an invalid setting type (e.g., `"collection_picker"` or `"product_picker"`) causes Shopify to **reject the schema entirely**, silently hiding the section from the Theme Customizer!

#### 1. Basic Content Inputs
- `text`: Single-line plain text.
- `textarea`: Multi-line plain text.
- `richtext`: HTML formatting with bold, italic, links, lists.
- `inline_richtext`: HTML formatting allowed, but stripped of block `<p>` wrappers (ideal for titles/headings).
- `number`: Numeric input.
- `checkbox`: Boolean true/false toggle.
- `radio`: Radio buttons (uses `"options"` array of `{ "value": "x", "label": "X" }`).
- `range`: Slider with `"min"`, `"max"`, `"step"`, `"unit"`, and `"default"`.
- `select`: Dropdown menu (uses `"options"` array).

#### 2. Media & Visual Inputs
- `color`: Hex color selector.
- `color_background`: CSS gradient / background selector.
- `color_scheme`: Links to theme color schemes.
- `image_picker`: Shopify CDN image picker.
- `video`: Native hosted MP4/WebM video picker.
- `video_url`: YouTube / Vimeo URL input (requires `"accept": ["youtube", "vimeo"]`).
- `font_picker`: Shopify font library picker.

#### 3. Shopify Commerce Resource Pickers (Exact Syntax)
| Setting Type | Valid Type Name | DO NOT USE (Common Mistake) | Notes |
| :--- | :--- | :--- | :--- |
| Single Collection | `"collection"` | `"collection_picker"` | Returns a `collection` handle or object |
| Multiple Collections | `"collection_list"` | `"collections"` | Returns an array of collection objects |
| Single Product | `"product"` | `"product_picker"` | Returns a `product` handle or object |
| Multiple Products | `"product_list"` | `"products"` | Returns an array of product objects |
| Navigation Menu | `"link_list"` | `"menu_picker"` | Returns a `linklist` handle |
| Page Picker | `"page"` | `"page_picker"` | Returns a `page` object |
| Blog Picker | `"blog"` | `"blog_picker"` | Returns a `blog` object |
| Article Picker | `"article"` | `"article_picker"` | Returns an `article` object |
| URL / Link | `"url"` | `"link"` | Can link to external URLs or internal routes |
| Metaobject | `"metaobject"` | `"metaobject_picker"` | Requires `"metaobject_type"` |
| Metaobject List | `"metaobject_list"` | — | Requires `"metaobject_type"` |

#### 4. Structural / Informational Elements
- `header`: Visual separator in the customizer.
- `paragraph`: Descriptive text for the merchant.

---

### Section Blocks & Theme Editor Interactivity

#### The Mandatory `{{ block.shopify_attributes }}`
Whenever iterating over `section.blocks`, you **must** output `{{ block.shopify_attributes }}` on the block's outer HTML wrapper:

```liquid
{%- for block in section.blocks -%}
  <div class="my-block-element" {{ block.shopify_attributes }}>
    <!-- content -->
  </div>
{%- endfor -%}
```
**Why this matters:** When the merchant clicks a block in the Shopify Theme Editor sidebar, Shopify's customizer JS injects `data-shopify-editor-block` attributes to scroll to and highlight that element in real-time. Without `{{ block.shopify_attributes }}`, live-preview block selection is broken.

---

### Presets vs. Section Groups (`enabled_on` / `disabled_on`)

1. **How Sections Appear in "Add section":**  
   Shopify only lists a section in the Theme Customizer's **Add section** menu if the section schema contains a `"presets"` array:
   ```json
   "presets": [
     {
       "name": "Featured Products",
       "settings": { ... },
       "blocks": [ ... ]
     }
   ]
   ```

2. **Restricting Where Sections Can Be Added (`enabled_on` / `disabled_on`):**  
   To prevent body sections (like Product Compare) from being added to the header, or to prevent Header/Footer sections from being added to body templates:
   ```json
   "enabled_on": {
     "templates": ["index", "product", "collection"],
     "groups": ["header"]
   }
   ```
   Or restrict:
   ```json
   "disabled_on": {
     "groups": ["header", "footer"]
   }
   ```

---

## 6. Section Groups: Header and Footer Architecture

In OS 2.0, the Header and Footer are decoupled from `layout/theme.liquid` into **Section Groups**:
- `sections/header-group.json`
- `sections/footer-group.json`

### `sections/header-group.json`:
```json
{
  "name": "Header Group",
  "type": "header",
  "sections": {
    "announcement-bar": {
      "type": "announcement-bar",
      "settings": {}
    },
    "header": {
      "type": "header",
      "settings": {}
    }
  },
  "order": [
    "announcement-bar",
    "header"
  ]
}
```

### `sections/footer-group.json`:
```json
{
  "name": "Footer Group",
  "type": "footer",
  "sections": {
    "footer": {
      "type": "footer",
      "settings": {}
    }
  },
  "order": [
    "footer"
  ]
}
```

In `layout/theme.liquid`, render them using the `{% sections %}` tag (plural):
```liquid
{% sections 'header-group' %}

<main id="MainContent">{{ content_for_layout }}</main>

{% sections 'footer-group' %}
```

---

## 7. JSON Templates Engine (`templates/*.json`)

Every page template is represented by a JSON file that defines which sections render, their configuration, their blocks, and their order.

### Anatomy of `templates/index.json`:
```json
{
  "sections": {
    "hero": {
      "type": "hero-slideshow",
      "settings": {
        "autoplay": true
      },
      "blocks": {
        "slide-1": {
          "type": "slide",
          "settings": {
            "heading": "Next-Gen Audio",
            "button_label": "SHOP NOW"
          }
        }
      },
      "block_order": ["slide-1"]
    },
    "category-grid": {
      "type": "collection-list",
      "settings": {
        "heading": "Shop by Category"
      }
    },
    "featured-products": {
      "type": "product-carousel",
      "settings": {
        "heading": "Best Sellers"
      }
    }
  },
  "order": [
    "hero",
    "category-grid",
    "featured-products"
  ]
}
```

### Special Page Templates (`product.json`, `collection.json`, `cart.json`)
For dedicated pages, create a "main" section (e.g. `sections/main-product.liquid`, `sections/main-collection.liquid`) and reference it in the template:

```json
{
  "sections": {
    "main": {
      "type": "main-product",
      "settings": {}
    },
    "recommendations": {
      "type": "product-carousel",
      "settings": {
        "heading": "You May Also Like"
      }
    }
  },
  "order": [
    "main",
    "recommendations"
  ]
}
```

---

## 8. Snippets & Reusable Components

Snippets are stored in `snippets/*.liquid` and rendered using the `{% render %}` tag.

### Rules for Snippets:
1. **Never use `{% include %}`:** It is deprecated, slow, and causes variable scope leakage.
2. **Explicit Variable Passing:** Snippets do not automatically inherit parent loop or assign variables unless explicitly passed:
   ```liquid
   {%- render 'product-card', product_card_product: product, show_vendor: true -%}
   ```
3. **Keep Them Pure:** Snippets should contain minimal business logic and return reusable UI.

### Production Reference: `snippets/price.liquid`
```liquid
{%- comment -%}
  Renders a product's formatted price, handling sales, strikethroughs, and variable variant prices.
  Usage:
  {% render 'price', product: product, use_variant: false %}
{%- endcomment -%}

{%- assign target = product.selected_or_first_available_variant | default: product -%}
{%- assign compare_at_price = target.compare_at_price -%}
{%- assign price = target.price | default: 1999 -%}
{%- assign on_sale = false -%}

{%- if compare_at_price > price -%}
  {%- assign on_sale = true -%}
{%- endif -%}

<div class="price-container {% if on_sale %}price--on-sale{% endif %}">
  {%- if product.price_varies and use_variant != true -%}
    <span class="price__from">{{ 'products.product.price.from_price_html' | t | default: 'From' }}</span>
  {%- endif -%}

  <span class="price__regular font-semibold">
    {{ price | money }}
  </span>

  {%- if on_sale -%}
    <s class="price__compare text-muted">
      {{ compare_at_price | money }}
    </s>
  {%- endif -%}
</div>
```

---

## 9. Shopify Liquid: Essential Objects, Filters & Tags

### Essential Liquid Objects
| Object | Key Properties / Usage |
| :--- | :--- |
| `product` | `product.title`, `product.vendor`, `product.price`, `product.compare_at_price`, `product.images`, `product.variants`, `product.options_with_values`, `product.tags`, `product.metafields` |
| `collection` | `collection.title`, `collection.products`, `collection.all_products_count`, `collection.featured_image` |
| `cart` | `cart.item_count`, `cart.total_price`, `cart.items`, `cart.note` |
| `routes` | `routes.cart_url`, `routes.cart_add_url`, `routes.search_url`, `routes.account_url`, `routes.all_products_collection_url` |
| `request` | `request.page_type` (e.g. 'index', 'product', 'collection', 'cart', '404'), `request.locale.iso_code` |
| `settings` | Global values from `config/settings_schema.json` |

### Critical Liquid Filters
- `image_url: width: 600, height: 600, crop: 'center'` — Efficient CDN image resizer.
- `image_tag: loading: 'lazy', fetchpriority: 'auto', alt: '...'` — Generates responsive `<img>` tags.
- `money` or `money_with_currency` — Formats integers into store currency (e.g., `1999` → `$19.99`).
- `asset_url | stylesheet_tag` — Enqueues CSS from `assets/`.
- `asset_url | script_tag` — Enqueues JS with automatic CDN URLs.
- `t` — Pulls translation string from `locales/en.default.json`.
- `json` — Escapes strings/objects into safe JSON for inline `<script>` tags.

---

## 10. JavaScript, Cart APIs & Section Rendering

Shopify stores do not require heavy backend servers for interaction. The frontend communicates with Shopify via standard AJAX endpoints.

### 1. AJAX Add to Cart (`POST /cart/add.js`)
```javascript
async function addToCart(variantId, quantity = 1) {
  try {
    const response = await fetch(window.ShopifyTheme.routes.cart_add_url, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Accept': 'application/json'
      },
      body: JSON.stringify({
        items: [{ id: Number(variantId), quantity: Number(quantity) }]
      })
    });
    
    if (!response.ok) {
      const errorData = await response.json();
      throw new Error(errorData.description || 'Failed to add item to cart');
    }

    const itemData = await response.json();
    
    // Dispatch global custom event so header bubble and cart drawers update
    document.dispatchEvent(new CustomEvent('cart:updated', { detail: { item: itemData } }));
    return itemData;
  } catch (error) {
    console.error('Cart Error:', error);
    alert(error.message);
  }
}
```

### 2. Live Cart State Fetcher (`GET /cart.js`)
```javascript
async function refreshCartBubble() {
  const response = await fetch(window.ShopifyTheme.routes.cart_url + '.js');
  const cart = await response.json();
  
  const bubble = document.querySelector('.header__cart-count');
  if (bubble) {
    bubble.textContent = cart.item_count;
    bubble.classList.toggle('hidden', cart.item_count === 0);
  }
}
document.addEventListener('cart:updated', refreshCartBubble);
```

### 3. Shopify Section Rendering API
To re-render a section without reloading the page (e.g. updating cart drawers or facet filters):
```javascript
async function updateCartSection() {
  const response = await fetch('/?sections=cart-drawer');
  const sections = await response.json();
  const html = sections['cart-drawer'];
  document.querySelector('#cart-drawer-container').innerHTML = html;
}
```

---

## 11. Product Catalog Data & CSV Specifications

When importing demo products for testing or client onboarding, Shopify requires an exact **47-column CSV**.

### Critical CSV Header Columns:
```csv
Handle,Title,Body (HTML),Vendor,Standardized Product Type,Custom Product Type,Tags,Published,Option1 Name,Option1 Value,Option2 Name,Option2 Value,Option3 Name,Option3 Value,Variant SKU,Variant Grams,Variant Inventory Tracker,Variant Inventory Qty,Variant Inventory Policy,Variant Fulfillment Service,Variant Price,Variant Compare At Price,Variant Requires Shipping,Variant Taxable,Variant Barcode,Image Src,Image Position,Image Alt Text,Gift Card,SEO Title,SEO Description,Google Shopping / Google Product Category,Google Shopping / Gender,Google Shopping / Age Group,Google Shopping / MPN,Google Shopping / Condition,Google Shopping / Custom Product,Google Shopping / Custom Label 0,Google Shopping / Custom Label 1,Google Shopping / Custom Label 2,Google Shopping / Custom Label 3,Google Shopping / Custom Label 4,Variant Image,Variant Weight Unit,Variant Tax Code,Cost per item,Status
```

### The Fatal CSV Offset Bug:
- If a data row has 46 or 48 fields due to missing or extra commas, all subsequent columns shift!
- For example, if `Variant SKU` is omitted without a placeholder comma, the string `"true"` from `Variant Requires Shipping` shifts into `Variant Price`, causing the import error:  
  **`"true" is not a valid price`**.
- **Rule:** Always generate CSVs using Python's `csv.DictWriter` or an automated CSV builder to guarantee exact field counts.

---

## 12. Common Pitfalls & Fatal Bugs to Avoid

| Pitfall / Mistake | Consequence | Correct Pattern |
| :--- | :--- | :--- |
| `"type": "collection_picker"` in schema | **Section disappears** from Theme Customizer | Use `"type": "collection"` |
| `"type": "product_picker"` in schema | **Section disappears** from Theme Customizer | Use `"type": "product"` |
| Missing `"presets"` in section schema | Section cannot be added in Theme Customizer | Always provide `"presets": [{ "name": "..." }]` for dynamic sections |
| Missing `{{ block.shopify_attributes }}` | Block selection in theme editor does not highlight or auto-scroll | Place `{{ block.shopify_attributes }}` on the outer container of every block loop |
| Missing `{{ content_for_header }}` | Analytics, app blocks, and preview mode break | Must be inside `<head>` in `theme.liquid` |
| Missing `{{ content_for_layout }}` | Page body content fails to render | Must be inside `<main>` in `theme.liquid` |
| Using `{% include %}` | Deprecated; causes variable pollution and warnings | Always use `{% render 'snippet-name' %}` |
| Referencing undefined object like `{{ email }}` in reset password | `shopify theme check` fails with `UndefinedObject` | Use static wording or verified objects |
| Subdirectories inside `/sections/` or `/assets/` | Shopify rejects theme upload | Keep folder structure strictly flat |
| Including `.git`, `node_modules`, or root zip files in theme archive | Theme upload fails size checks or file structure check | Only package: `assets`, `config`, `layout`, `locales`, `sections`, `snippets`, `templates` |

---

## 13. Verification & Pre-Flight Checklist

Before delivering or uploading any Shopify theme:

1. **Run Theme Check (Zero Offenses):**
   ```bash
   shopify theme check
   ```
   Must output: `XX files inspected with no offenses found. 0 errors, 0 warnings.`

2. **Verify All Sections in Editor:**
   - Confirm every section in `sections/*.liquid` intended for body pages has a `"presets"` key.
   - Confirm setting types match the valid Shopify setting dictionary.

3. **Accessibility Audit:**
   - Tab through all navigation items; ensure focus outlines are visible.
   - Verify mobile drawers trap focus and close on `Escape`.

4. **Verify ZIP Archive Structure:**
   The ZIP file must unpack with the standard theme directories (`assets`, `config`, `layout`, `locales`, `sections`, `snippets`, `templates`) directly at the root of the archive (not nested inside a secondary subfolder).

---

## 14. Master Blueprint Prompt for Future AI Conversations

*Copy and paste the prompt below into any new AI conversation whenever you want to start building a new Shopify theme from scratch:*

```markdown
I want to build a high-performance, production-ready Shopify Online Store 2.0 theme.
I have attached the technical reference guide `HOWTOSHOPIFY.md` which defines all Shopify OS 2.0 rules, directory structures, schema setting types, Liquid syntax, and validation requirements.

Here is the theme concept and aesthetic vision:
- Theme Name: [Insert Name, e.g. "Vanguard"]
- Industry / Niche: [e.g. Luxury Watches / Minimalist Apparel / High-Tech Hardware]
- Design Language: [e.g. Dark mode, glassmorphism accents, neon highlights, condensed bold headings]
- Key Custom Sections Needed:
  1. Header with announcement bar, sticky navigation, and search drawer
  2. Hero banner / slideshow with dynamic overlays
  3. Featured collection list with hover interactions
  4. Product showcase / carousel with variant preview chips and AJAX cart
  5. Interactive comparison matrix or feature highlight table
  6. Multi-column footer with newsletter capture and currency/language selectors

Workflow to follow:
1. Work in iterative cycles. Do not write full code until we confirm the initial specification.
2. Ensure all schemas strictly follow `HOWTOSHOPIFY.md` (no invalid setting types like `collection_picker`).
3. Guarantee that every addable section includes `presets`.
4. Ensure full Theme Check compliance with zero errors.

Please begin by asking any clarifying questions and summarizing your implementation roadmap.
```
