# <img src="https://github.com/user-attachments/assets/ac0839f0-da99-4f60-a1a3-3517c9cf70b1" alt="Logo" width="40" height="40" /> Google Maps Scraper

<p align="left">
Extract local business data from Google Maps into clean CSV/JSON for lead gen, market research, and ops.
</p>

<p align="center">
 <img width="800" height="300" alt="Image" src="https://github.com/user-attachments/assets/ebd5c2b9-170c-4e44-acf0-ff5789e4a3e1" />
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

import json, pandas as pd, time
from datetime import datetime
from playwright.sync_api import sync_playwright

def scrape(q="dentist", city="Austin,TX", n=20):
    with sync_playwright() as pw:
        b = pw.chromium.launch(headless=True); p = b.new_page()
        p.goto(f"https://www.google.com/maps/search/{q}+in+{city}"); p.wait_for_selector('[role="feed"]')
        data=[]; seen=set()
        for a in p.query_selector_all('a[href^="https://www.google.com/maps/place"]')[:n]:
            url=a.get_attribute("href"); 
            if not url or url in seen: continue; seen.add(url)
            p2=b.new_page(); p2.goto(url); tx=lambda s:(e:=p2.query_selector(s)) and e.inner_text().strip()
            data.append({"name":a.get_attribute("aria-label") or "", "url":url, "category":tx('[jslog*="breadcrumb"]') or "",
                        "rating":float((tx('span[aria-label*="stars"]') or "0").split()[0]), 
                        "reviews":int("".join(c for c in (tx('button[aria-label*="reviews"]')or"") if c.isdigit()) or 0),
                        "address":tx('button[data-item-id="address"]') or "", "phone":tx('button[data-item-id^="phone:tel:"]') or "",
                        "website":tx('a[data-item-id="authority"]') or "", "scraped_at":datetime.utcnow().isoformat()+"Z"}); p2.close(); time.sleep(.6)
        b.close(); return data

if __name__=="__main__": d=scrape(); pd.DataFrame(d).to_csv("places.csv",index=False); json.dump(d,open("places.json","w"),indent=2)

```

---

## Roadmap

- [ ]  Polygon/geojson targeting <br>
- [ ]  Sheets & Airtable exporters <br>
- [ ]  Phone/URL normalization helpers <br>
- [ ]  Proxy pool + retries <br>

---
## FAQ

Q: Does this bypass Google’s protections? <br>
A: No. It’s white-hat and throttled.

Q: Will this work for any country? <br>
A: Yes, but fields may vary by locale.

Q: How many results per run? <br>
A: Depends on query density, pagination, and throttling.

---
## License

MIT 
---

## Contact Us

Questions? Need a custom scraper or integrations?

 Website: https://www.bitbash.dev/ <br>
 Discord: https://discord.gg/zX7frTbx  <br>
 Telegram: https://t.me/devpilot1  <br>

---

<img width="1536" height="270" alt="Image" src="https://github.com/user-attachments/assets/fae69c91-6792-41ee-a073-50f46ba38788" />





