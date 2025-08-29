# <img src="https://github.com/user-attachments/assets/ac0839f0-da99-4f60-a1a3-3517c9cf70b1" alt="Logo" width="40" height="40" /> Google Maps Scraper

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

---

## Python Code Example

```python

# 2) scrape.py
import json, time, pandas as pd
from datetime import datetime
from playwright.sync_api import sync_playwright

def normalize_place(p):
    p["scraped_at"] = datetime.utcnow().isoformat() + "Z"
    return p

def parse_place_card(card):
    name = card.get_attribute("aria-label") or card.inner_text().split("\n")[0]
    url = card.get_attribute("href") or ""
    return {"name": name, "url": url}

def scrape(query="dentist", city="Austin, TX", max_results=100):
    rows, seen = [], set()
    with sync_playwright() as pw:
        b = pw.chromium.launch(headless=True)
        page = b.new_page()
        page.goto(f"https://www.google.com/maps/search/{query}+in+{city}")
        page.wait_for_selector('[role="feed"]', timeout=20000)

        cards = page.query_selector_all('a[href^="https://www.google.com/maps/place"]')
        for a in cards[:max_results]:
            base = parse_place_card(a)
            if base["url"] in seen: continue
            seen.add(base["url"])

            # open detail in new tab
            page2 = b.new_page()
            page2.goto(base["url"])
            page2.wait_for_timeout(1500)

            def tx(sel):
                el = page2.query_selector(sel)
                return el.inner_text().strip() if el else ""

            row = {
                "place_id": page2.url.split("!3m1!")[0][-27:] if "!3m1!" in page2.url else "",
                "name": base["name"],
                "category": tx('[jslog*="breadcrumb"]') or "",
                "rating": float((tx('span[aria-label*="stars"]') or "0").split()[0] or 0),
                "reviews_count": int(''.join([c for c in tx('button[aria-label*="reviews"]') if c.isdigit()] or "0") or 0),
                "address": tx('button[data-item-id="address"]') or "",
                "phone": tx('button[data-item-id^="phone:tel:"]') or "",
                "website": tx('a[data-item-id="authority"]') or "",
                "lat": None, "lng": None,
                "url": page2.url
            }
            rows.append(normalize_place(row))
            page2.close()
            time.sleep(0.8)  # throttle

        b.close()
    return rows

if __name__ == "__main__":
    data = scrape(query="dentist", city="Austin, TX", max_results=40)
    pd.DataFrame(data).to_csv("output/places.csv", index=False)
    with open("output/places.json","w",encoding="utf-8") as f:
        json.dump(data, f, ensure_ascii=False, indent=2)
    print(f"Wrote {len(data)} rows to output/")
```

#  Run
``` bash
mkdir -p output
python scrape.py

``` 

## Examples

examples/sample.csv <br?
examples/sample.json

## Roadmap

 Polygon/geojson targeting <br>
 Sheets & Airtable exporters <br>
 Phone/URL normalization helpers <br>
 Proxy pool + retries <br>
 Docker image + CI <br>
 Basic QA dashboard (errors, throughput) <br>

## FAQ

Q: Does this bypass Google’s protections? <br>
A: No. It’s white-hat and throttled.

Q: Will this work for any country? <br>
A: Yes, but fields may vary by locale.

Q: How many results per run? <br>
A: Depends on query density, pagination, and throttling.

## License

MIT 






