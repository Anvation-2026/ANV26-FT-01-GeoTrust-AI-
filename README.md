# GeoTrust AI

**Does anyone actually trade from this address?**

GeoTrust AI is a single-page web app that checks a company's declared address against four physical signals (zoning, foot traffic, tenant density and transaction volume). It returns a trust score from 0 to 100 and explains the reasons behind it.

It runs entirely in the browser. There is no backend, no build step and no API key.

---

## Table of contents

- [Overview](#overview)
- [Key features](#key-features)
- [How it works](#how-it-works)
- [Scoring rules](#scoring-rules)
- [Built-in demo cases](#built-in-demo-cases)
- [Getting started](#getting-started)
- [Using your own data (CSV)](#using-your-own-data-csv)
- [Project structure](#project-structure)
- [Tech stack](#tech-stack)
- [Customisation](#customisation)
- [Deployment](#deployment)
- [Accessibility and performance](#accessibility-and-performance)
- [Limitations](#limitations)
- [Roadmap](#roadmap)
- [License](#license)

---

## Overview

A registered address is not proof of a real business. Shell companies can share a single door, claim hundreds of daily transactions from a place nobody visits, and pass a paperwork check. GeoTrust AI compares what a company *claims* with what the street *shows*, and makes the result easy to review.

**Intended users:** banks and fintechs (onboarding), marketplaces (seller checks), and registries or regulators (filing review).

## Key features

| Feature | Description |
| --- | --- |
| **Trust score (0 to 100)** | A rule-based score with a clear verdict: credible, shared location, or ghost-address risk. |
| **Explainable results** | Every score lists the supporting signals and the risks or contradictions behind it. |
| **Signal inspector** | Change industry, building type, foot traffic, tenant count and transactions, and watch the verdict update live. |
| **Street scene** | An animated SVG: dots inside the building are tenants, moving dots on the street are foot traffic. |
| **24-hour pulse chart** | A smooth SVG chart comparing foot traffic with claimed transaction volume. |
| **Real map** | An OpenStreetMap view with a marker on the exact location. No API key needed. |
| **CSV upload** | Load your own company list. Each row becomes a case you can check. |
| **Address lookup** | Rows with no coordinates are located from their address using OpenStreetMap's free Nominatim search. |
| **Dark and light themes** | Follows the system setting, can be switched manually, and is remembered. |
| **Responsive and accessible** | Works from phone to desktop, supports keyboard use and reduced-motion settings. |

## How it works

1. **Pick an address.** Choose a demo case, type a company name, or upload a CSV.
2. **Gather signals.** Zoning, foot traffic, tenant density and transaction volume.
3. **Fuse and score.** Rules compare the claim to the street and produce a score from 0 to 100.
4. **Explain.** The page lists what supports the score and what contradicts it.

## Scoring rules

Every company starts at **100 points**. Rules add or remove points, and the final score is limited to the range 0 to 100.

| Rule | Condition | Points |
| --- | --- | --- |
| Zoning mismatch | Industry is logistics and building is residential | −45 |
| Ghost density | More than 30 entities at the address and foot traffic below 20 | −50 |
| Shared hub | More than 25 entities and foot traffic of 50 or more | −35 |
| High volume, no activity | More than 500 transactions per day and foot traffic below 25 | −30 |
| Zoning match | Industry is logistics and building is industrial | +5 |

**Verdicts**

| Score | Verdict |
| --- | --- |
| 75 to 100 | High confidence: credible trading location |
| 40 to 74 | Moderate confidence: ambiguous shared location |
| 0 to 39 | Low confidence: high-risk ghost address |
| Not in dataset | A name that is not in the demo cases or your CSV scores 0 |

The rules shown on the page and the rules used for scoring come from the same `RULES` object in the code, so they cannot drift apart.

## Built-in demo cases

| Case | Foot traffic | Entities | Transactions/day | Score | Verdict |
| --- | --- | --- | --- | --- | --- |
| Apex Logistics Warehouse Hub | 85 | 2 | 450 | 100 | Credible |
| OmniTech Co-Working Suite 402 | 55 | 32 | 180 | 65 | Shared location |
| Virtual Capital & Freight Corp | 5 | 48 | 900 | 0 | Ghost address |

The company names and addresses in these cases are fictional examples.

## Getting started

No installation is needed.

**Option 1: open the file**

Double-click `index.html`, or open it in any modern browser.

**Option 2: run a local server (recommended)**

```bash
# Python 3
python3 -m http.server 8000
# then open http://localhost:8000
```

An internet connection is needed for the map, the address lookup and the Google Fonts. The rest of the page works offline, and fonts fall back to system fonts.

## Using your own data (CSV)

Click **Sample CSV** in the demo to download a ready-made file, then upload it with **Upload company CSV**.

Column headers are matched by keyword, so exact names are not required.

| Data | Header keywords recognised | Notes |
| --- | --- | --- |
| Company name | `company`, `name`, `entity` | Required. Rows without a name are skipped. |
| Address | `address`, `location` | Used for the map caption and address lookup. |
| Industry | `industry`, `sector` | Mapped to logistics, finance or software. |
| Building type | `building`, `zoning` | Mapped to residential, industrial or commercial. |
| Foot traffic | `foot`, `activity`, `pedestrian` | 0 to 100. |
| Tenants | `tenant`, `density` | 1 to 60. |
| Transactions | `transaction`, `tx` | 0 to 1000 per day. |
| Latitude | `lat` | Optional. Exact position when provided. |
| Longitude | `lng`, `lon` | Optional. Exact position when provided. |

Example:

```csv
company,address,industry,building,foot_traffic,co_tenants,tx_per_day,lat,lng
Northwind Freight Depot,"Harbour Road 12, Wuerzburg",Logistics,industrial,78,3,520,49.8003,9.9102
Harborline Cargo Ltd,"Flat 9, Elm Court, Dublin",Transport,residential,6,41,780,53.3441,-6.2675
```

If a numeric column is missing, the page fills in a sensible default so the case still runs. If `lat` and `lng` are missing, the address is looked up automatically. For the most accurate pin, include coordinates.

## Project structure

```
.
├── index.html   # The entire app: markup, styles and scripts
└── README.md
```

Inside `index.html`, the code is organised in clear sections:

| Section | Purpose |
| --- | --- |
| Tokens and base styles | Design variables, light and dark themes, typography |
| Components | Glass cards, buttons, rings, chips, charts |
| HTML sections | Landing, Problem, How it works, Demo, Features, Impact, Team |
| `RULES`, `calculateScore`, `verdictFor` | The scoring engine |
| `PRESETS` | The three demo cases |
| `buildScene`, `drawChart`, `showMap` | Street scene, 24-hour chart and map |
| `parseCSV`, `recordsFromCSV` | CSV reading and column matching |
| Scroll behaviour | Reveal animations, progress bar, section tracking |

## Tech stack

- **HTML5, CSS3 and vanilla JavaScript.** No frameworks, no dependencies, no build tools.
- **SVG** for the street scene, the chart, the icons and the animated background.
- **OpenStreetMap** embed for the map, and **Nominatim** for address lookup (both free, no key).
- **Google Fonts:** Bricolage Grotesque and Instrument Sans, with system-font fallbacks.

## Customisation

**Team members.** Edit the `TEAM` list near the top of the script:

```js
const TEAM = [
  { name: 'Your Name', role: 'Your role' }
];
```

**Scoring rules.** Change the points or labels in the `RULES` object. The rules list on the page updates automatically. Thresholds live in `calculateScore`.

**Demo cases.** Edit or add entries in `PRESETS` (name, address, coordinates, industry, building, activity, density, transactions).

**Colours and fonts.** Change the CSS variables under `:root`, `[data-theme="dark"]` and `[data-theme="light"]`.

## Deployment

Because the app is a static file, it can be hosted anywhere:

- **GitHub Pages:** push `index.html` to a repository and enable Pages.
- **Netlify or Vercel:** drag the folder in, or connect the repository.
- **Any web server:** upload `index.html`.

## Accessibility and performance

- Semantic landmarks, labelled controls and visible keyboard focus.
- Live regions announce score changes.
- `prefers-reduced-motion` is respected, and animations are switched off.
- Colour is never the only signal: verdicts also appear as text.
- Single file with no heavy libraries, so the page loads quickly.

## Limitations

- **This is a demonstration.** Foot traffic, tenant counts and transaction volumes are inputs you set or supply. The app does not collect real mobility or transaction data.
- The scoring is a transparent set of rules, not a trained machine-learning model.
- The demo cases use fictional companies, so their map pins are approximate.
- The address lookup uses Nominatim's public service, which has usage limits. It is suitable for small lists, not bulk geocoding.
- Scores support human review. They should not be the only basis for a compliance or legal decision.

## Roadmap

- Connect real data sources for foot traffic and registry records.
- Batch scoring: run every CSV row at once and export the results.
- Adjustable rule weights in the interface.
- Multi-language support.

## License

Add your chosen licence here (for example, MIT) and include a `LICENSE` file in the repository.

---

Map data © [OpenStreetMap contributors](https://www.openstreetmap.org/copyright).
