# <img src="https://github.com/user-attachments/assets/ac0839f0-da99-4f60-a1a3-3517c9cf70b1" alt="Logo" width="40" height="30" /> Google Maps Scraper

<p align="center">
Extract local business data from Google Maps into clean CSV/JSON for lead gen, market research, and ops.
</p>

<p align="center">
 <img width="1400" height="550" alt="Image" src="https://github.com/user-attachments/assets/ebd5c2b9-170c-4e44-acf0-ff5789e4a3e1" />
</p>

---

## Table of Contents
1. [Overview](#overview)
2. [Features](#features)
3. [What You Can Do](#what-you-can-do)
4. [Architecture](#architecture)
5. [Workflow](#workflow)
6. [Data Schema](#data-schema)
7. [Quick Start](#quick-start)
8. [Examples](#examples)
9. [Responsible Use](#responsible-use)
10. [Roadmap](#roadmap)
11. [FAQ](#faq)
12. [License](#license)
13. [Contact](#contact)

---

## Overview
**Google Maps Scraper** collects structured data for places (name, category, rating, reviews, address, phone, website, coordinates, hours, etc.) from search results and place detail pages.  
Built for **lead generation**, **territory planning**, **competitor scans**, and **list enrichment**.

> White-hat design: no CAPTCHA/anti-bot bypass included; obey rate limits.

---

## Features

| # | Feature | What It Does | Why It Matters |
|---|---|---|---|
| 1 | **Keyword + Geo Targeting** | Search by query, city, radius, or polygon | Precise local data pulls |
| 2 | **Rich Place Schema** | name, category, rating, reviews_count, price_level, address, phone, website, plus_code, lat, lng, hours | Ready for CRM/BI |
| 3 | **Pagination & Deep Fetch** | Iterates result pages and opens details | Higher completeness |
| 4 | **Dedupe by place_id/url** | Removes duplicates | Clean lists |
| 5 | **CSV/JSON Export** | Writes to `/output` | Easy imports |
| 6 | **Optional Enrichment** | Normalize phones, resolve websites, basic email discover (optional) | Better outreach hit-rate |
| 7 | **Configurable Throttling** | Rate limiting & backoff | Safer scraping |
| 8 | **Pluggable Proxies (opt.)** | Rotating proxy pool support | Fewer blocks |

---

## What You Can Do
- Build **B2B lead lists** (e.g., “dentists in Austin within 15km”)
- Audit **categories/ratings** for competitive scans
- Plan **door-to-door** or **inside sales** routes using coordinates
- Enrich existing lists with **phone/website/hours**

---

## Architecture
<p align="center">
 <img width="1200" height="450" alt="Image" src="https://github.com/user-attachments/assets/a62aa1c0-1f3b-4ea4-a6f4-167fe61050cd" />
</p>

**Flow (high level):**  
Input (query + geo) → Target Builder (grid/tiles or polygon) → Fetch Layer (search → result cards → detail pages) → Parser → Normalizer → Dedupe (place_id/url) → Validator → Exporters (CSV/JSON/Sheets/Airtable/Webhook) → Observability (logs/metrics)

---

## Workflow
<p align="center">
  <img alt="Workflow" src="assets/workflow.png" width="1536" height="500">
</p>

1) Input query/geo → 2) Build targets → 3) Rate limit on → 4) Search results crawl → 5) Open place detail → 6) Parse fields → 7) Normalize → 8) Dedupe → 9) Validate → 10) Export

---

## Data Schema

### Table
| Field | Type | Example |
|---|---|---|
| place_id | string | ChIJN1t_tDeuEmsRUsoyG83frY4 |
| name | string | “Acme Dental Clinic” |
| category | string | “Dentist” |
| rating | number | 4.6 |
| reviews_count | integer | 128 |
| price_level | integer \| null | 2 |
| address | string | “123 Main St, Austin, TX 78701” |
| city | string | “Austin” |
| state | string | “TX” |
| postal_code | string | “78701” |
| country | string | “US” |
| phone | string | “+1 512-555-1234” |
| website | string | “https://acmedental.com” |
| plus_code | string \| null | “PR5F+7Q Austin, Texas” |
| lat | number | 30.2672 |
| lng | number | -97.7431 |
| opening_hours | object \| null | `{ "mon":"09:00-17:00", ... }` |
| status | string | “OPEN_NOW” |
| url | string | Place share URL |
| scraped_at | string(ISO) | “2025-08-29T10:30:00Z” |

### JSON Schema (`/schema/place.schema.json`)
```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "title": "Place",
  "type": "object",
  "required": ["place_id", "name", "lat", "lng", "url", "scraped_at"],
  "properties": {
    "place_id": {"type":"string"},
    "name": {"type":"string"},
    "category": {"type":"string"},
    "rating": {"type":"number"},
    "reviews_count": {"type":"integer"},
    "price_level": {"type":["integer","null"]},
    "address": {"type":"string"},
    "city": {"type":"string"},
    "state": {"type":"string"},
    "postal_code": {"type":"string"},
    "country": {"type":"string"},
    "phone": {"type":"string"},
    "website": {"type":"string"},
    "plus_code": {"type":["string","null"]},
    "lat": {"type":"number"},
    "lng": {"type":"number"},
    "opening_hours": {"type":["object","null"]},
    "status": {"type":"string"},
    "url": {"type":"string"},
    "scraped_at": {"type":"string", "format":"date-time"}
  }
}
