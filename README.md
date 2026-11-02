# India trade and macro data analysis

Descriptive analysis of India's merchandise trade, currency, reserves and domestic macro indicators
(CPI, WPI, IIP, GDP), built from official sources (MoSPI data API, RBI, TRADESTAT, DGCI&S) and published as
static HTML charts plus the JSON datasets behind them. Everything is a snapshot pulled on 2026-07-18.
It is the descriptive half of a two-repo split; the prescriptive companion is
[`india-trade-sector-policy-recommendations`](https://github.com/herrrickshaw/india-trade-sector-policy-recommendations).
Both were originally one repo (`mospi-dataset-analysis`) and were split so "what the data shows" and
"what should be done" could be versioned and read independently.

> Status: archived snapshot (data as of 2026-07-18; no collector code, so it is not re-runnable). Figures are
> from official Government of India / RBI portals and are provisional where the source says so. Not
> investment advice.

**Live site**: [herrrickshaw.github.io/india-trade-data-analysis](https://herrrickshaw.github.io/india-trade-data-analysis/) —
every chart and JSON file is served directly from this repo via GitHub Pages, no build step.
For plain-language use with an AI assistant see [`HOW_TO_USE_WITH_AI.md`](HOW_TO_USE_WITH_AI.md).

## What it does
- Charts state-wise CPI (base 2024, 14 states + All India, Jan 2025 to Jun 2026) against national WPI, plus a choropleth of June 2026 CPI inflation.
- Reconstructs India's forex reserves (Jan 2015 onward) and USD/INR series, extending the MoSPI connector's frozen June-2025 endpoint with a direct scrape of RBI's Weekly Statistical Supplement and Reference Rate Archive.
- Separates nominal from real (PPP-adjusted) rupee depreciation and reads it against the US Treasury currency-manipulator tests applied to China.
- Rebuilds the national merchandise trade balance FY2018-19 to FY2025-26 from all 98 HS chapters in TRADESTAT, adds RBI's services surplus, and cross-validates against the TIA Portal, DGFT and PIB.
- Ranks the top-12 import and export HS chapters by size and by 8-year CAGR, and splits bullion and base metals out at 4-digit HS with partner-country detail.
- Tests imported fertiliser / diesel price pass-through to WPI and CPI (2012-2026), and charts real GDP growth with its revision history (FY2012-13 to FY2025-26).
- Re-cuts everything to one five-year window (FY2021-22 to FY2025-26) in a capstone synthesis and a standalone FY2025-26 report in US$ million throughout.

## Quick start
Nothing to install. Open the live site, or clone and open any chart in a browser.

```bash
git clone https://github.com/herrrickshaw/india-trade-data-analysis.git
cd india-trade-data-analysis
open charts/five_year_trade_currency_synthesis.html   # the capstone; start here
```

To load every dataset live in Python, open
[`notebooks/mospi_live_reference.ipynb`](https://colab.research.google.com/github/herrrickshaw/india-trade-data-analysis/blob/main/notebooks/mospi_live_reference.ipynb)
in Colab. It reads the JSON files from GitHub (plus the companion repo's PLI/policy datasets), has two working
live-scrape cells (RBI reserves, live USD/INR) and tours Seaborn, Plotly, Altair, Folium and missingno
alongside matplotlib. [`notebooks/repo_verification.ipynb`](notebooks/repo_verification.ipynb) is a 44-point
health check of both repos' Pages URLs and datasets.

## How it works
There is no pipeline code in the repo. Each bulletin was produced in a Claude Code session by pulling data
through the MoSPI MCP connector or by direct HTTP scrape, writing the result to `data/*.json`, and rendering a
self-contained HTML chart in `charts/`. The sections were added chronologically as new questions came up.

| Stage | How | Output |
|---|---|---|
| MoSPI pull | connector workflow `list_datasets -> get_indicators -> get_metadata -> get_data` against `api.mospi.gov.in` (25 datasets, 500+ indicators) | `data/mospi_snapshot_2026-07-18.json`, CPI/WPI/GDP/fertiliser JSON |
| RBI scrape | Weekly Statistical Supplement, enumerable `rbi.org.in/scripts/WSSView.aspx?Id=N` (19 Sep 1998 to date, no key); Reference Rate Archive date-range form for USD/INR | `data/rbi_forex_reserves_*.json`, `data/rbi_usd_inr_exchange_rate_*.json` |
| TRADESTAT pull | `tradestat.commerce.gov.in` (Laravel/Livewire): GET the search page for a session cookie + CSRF token, then POST; two fiscal years per query, 8 requests for the 8-year window | `data/tradestat_hsn_export_import_*.json`, `data/metals_hsn4_import_*.json` |
| Services offset | RBI BoP "Non-factor Services" via the connector, converted from Rs crore at FY-average rates from this repo's own currency series | `data/services_trade_and_overall_balance_*.json` |
| Cross-validation | TIA Portal, DGFT, PIB release, DPIIT annual report | `data/trade_data_cross_validation_2026-07-18.json` |
| Synthesis | no new collection; re-derived from the JSON above | `charts/five_year_trade_currency_synthesis.{html,md}`, `reports/FY2025-26_sample_report.html` |

Design notes: filter codes in the MoSPI API are dataset-specific (`indicator_code=3` means different things
in different datasets), so they are always resolved via `get_metadata`. The MoSPI workflow is only documented
as exercised through the connector; reproducing it standalone would require reverse-engineering the API's
undocumented auth/request format. Scraped extensions past the connector's June-2025 endpoint are drawn as a
dashed segment, never merged silently with connector data. The CPI choropleth uses amCharts `amcharts4-geodata`
(`india2023Low`, MIT) reprojected to SVG locally, with no map tiles or API calls at render time.

## Headline findings (as of 2026-07-18)
- **Connector snapshot**: CPI (base 2024, combined) June 2026 index 107.00, +4.38% YoY (rural 4.74%, urban 3.92%, food & beverages +5.05%); IIP (general, base 2011-12) March 2026 index 173.2, +4.1% YoY; WPI (base 2011-12) April 2026 index 167 (Jan 157.6, Feb 158.4, Mar 160.8); RBI total reserves June 2025 $698.1bn.
- **State CPI vs WPI**: Telangana hottest at +6.36% YoY, NCT of Delhi coolest at +2.96%, All-India +4.38%. WPI broke a year of near-flat readings (~154-158) with a run to 167 in April 2026.
- **Reserves and rupee**: all-time high $728.5bn on 27 Feb 2026 (above the $705.8bn Sep 2024 peak), trough $666.9bn on 26 Jun 2026 (-8.5% peak-to-trough), partial recovery to $675.2bn by 10 Jul 2026. Rupee depreciated ~38% over the decade (Rs 62.23 to 86.11, ~3.3%/yr), then +11.9% in the last 12 months to Rs 96.37 on 17 Jul 2026.
- **Real exchange rate**: nominal depreciation since Jan 2015 +54.9%, real (PPP-adjusted) +37.6%, effective buying power for dollar-priced goods down ~27%. The last 12 months were almost entirely real (14.0% real vs 11.9% nominal) since India's implied inflation (~0.9%) was below the US's. Unlike China's Aug 2019 manipulator designation (lifted Jan 2020), RBI has been *selling* reserves to defend the rupee, not buying to suppress it. Context from Brookings, ["China's Currency Policy, Explained"](https://www.brookings.edu/articles/chinas-currency-policy-explained/).
- **Trade balance**: merchandise deficit in every one of the 8 fiscal years, -$184.0bn (FY2018-19) to -$334.3bn (FY2025-26); export cover of imports fell from 64.2% to 56.9%. Largest deficit chapters: mineral fuels HS27 (-$147.5bn) and gems/jewellery HS71 (-$81.1bn); strongest surplus chapters: pharmaceuticals, vehicles, cereals, apparel. Services surplus more than doubled, $82.1bn (FY2018-19) to $188.9bn (FY2024-25), cutting the FY2024-25 overall gap from -$283.5bn to -$94.6bn (COVID year -$14.1bn).
- **Cross-validation**: imports match the TIA Portal almost to the dollar ($776,013.65M vs $776,013.6M); HS27 exports match exactly ($55,871.2M); PIB's FY2025-26 exports ($441.78bn) and deficit ($333.19bn) match within rounding. TIA Portal's export total reads ~2% ($9.3bn) higher, consistent with data-vintage drift on a provisional year. DGFT's "Import Export Data Bank" is a link back to TRADESTAT; DPIIT publishes no services-trade or trade-balance series, but its IEM filings independently show Electricals and Fertilizers investment intentions scaling fastest.
- **HSN trends (FY2018-19 to FY2025-26)**: mineral fuels HS27 is the largest import ($203.4bn) but grew only +21.2% (2.8% CAGR); fastest-growing imports are fertilisers HS31 (+118.9%, 11.8% CAGR), electrical machinery HS85 (+101.5%, 10.5% CAGR) and fats/oils HS15 (+97.9%, 10.2% CAGR); top-12 chapters are 81.5% of imports. Electrical machinery exports quadrupled (+324.1%, 22.9% CAGR, $12.7bn to $54.0bn); HS71 is the only top-12 chapter shrinking on either side (-29.9%), corroborated by NITI Aayog's Trade Watch. The HS84-85 / HS71 grouping matches DGCI&S's official 13-sector taxonomy per the TIA Portal manual.
- **4-digit metals (FY2024-25 vs FY2025-26)**: bare gold (7108) is 9.3% of the import bill ($72.0bn, +24% YoY), half from Switzerland ($20.1bn) and UAE ($15.3bn, CEPA channel); platinum-group (7110) collapsed -92% ($5.4bn to $0.4bn) after the "platinum alloy" duty-arbitrage route closed, mirroring the migration seen in gold compounds (HSN 28433000); dore sourcing from Ghana (+449%), Bolivia (+245%), Brazil (+538%); silver tripled to $12.1bn; copper complex $14.7bn with concentrate +54% YoY as smelting capacity substitutes cathode imports with ore.
- **Fertiliser/fuel pass-through (2012-2026)**: diesel (WPI HSD) +79.0% since April 2012, r=0.92 with WPI headline; urea only +13.8% (subsidy), DAP +38.4%. Same-year correlations with CPI are weak (r -0.52 to +0.29) because, per Bhattacharya & Gupta's NIPFP SVAR, transmission runs on a 1-4 month lag. CPI oils & fats swung from -18.17% (Jun 2023) to +21.24% (Aug 2025), matching Mishra & Roy's near-1:1 edible-oil pass-through. Other literature cited in the chart: Hamilton (IJCB), a JETIR 2025 survey, an MBA thesis on import composition.
- **GDP revisions**: FY2025-26 opens at +7% (First Advance Estimate); FY2020-21 was revised from -8% to -6%; FY2023-24 from +7% to +9%; typical revision ~0.75pp, up to 2pp. data.gov.in GDP/reserves datasets were found to be frozen one-off releases despite recent "Updated On" stamps.
- **Five-year synthesis (FY2021-22 to FY2025-26)**: imports +26.6% ($613.1bn to $776.0bn) vs exports +4.7% ($422.0bn to $441.7bn), a 5.7:1 ratio; FY-average rupee +13.7% (74.50 to 84.68 through FY2024-25); goods deficit widened $92.5bn (-$191.0bn to -$283.5bn) while the services surplus grew +75.6% ($107.6bn to $188.9bn), covering ~88% of the extra goods shortfall; GDP growth 10%, 8%, 9%, 6%, 7%; reserves +18.7%. PLI timeline in the same window: Auto Rs 25,938cr (Sep 2021), Textiles Rs 10,683cr (Sep 2021), White Goods (85 companies by 2024), 2023 expansion to 14 sectors (Rs 1.97 lakh crore), Specialty Steel 1.2 Rs 11,887cr (MoUs Feb 2026). China's ~62% share of electronic-component imports and the USA's lead in 11 of 12 fastest-growing export chapters have not shifted.

## Repository layout
| Path | Purpose |
|---|---|
| `index.html` | GitHub Pages landing page linking every chart, dataset and notebook |
| `charts/*.html` | Ten self-contained HTML bulletins (CPI/WPI, CPI heatmap, reserves, real exchange rate, trade balance, HSN trends, fertiliser/fuel, GDP, five-year synthesis) |
| `charts/five_year_trade_currency_synthesis.md` | Text extract of the capstone (replaced the former PDF) |
| `reports/FY2025-26_sample_report.html` | Standalone FY2025-26 snapshot in US$ million with a cross-source consistency log; PLI Rs crore figures converted at each scheme's own elapsed-period average rate |
| `reports/blog_metal_recycling_import_bill.html` | Blog post on metal recycling vs the import bill |
| `data/*.json` | 16 tracked datasets, one per bulletin, named by series and period (see [Data sources](#data-sources)) |
| `data/metal_recyclers_business_model_validation.json` | Company universe for the recycling post (Jain Resource Recycling listed as JAINREC.BO, Oct 2025) |
| `data/pib_index.sqlite` | Currently a 0-byte placeholder |
| `notebooks/` | `mospi_live_reference.ipynb` (Colab data loader + viz tour), `repo_verification.ipynb` (44-point health check) |
| `graphify-out/` | Generated knowledge-graph output; not hand-edited |
| `.github/workflows/quality-gates.yml` | gitleaks secret scan on push / PR |
| `CLAUDE.md` | Source-fetching rules for AI sessions |

## Data sources
| Source | What is taken | Access route | Licence / caveat |
|---|---|---|---|
| MoSPI data API (`api.mospi.gov.in`) via MCP connector | CPI, WPI, IIP, NAS (GDP), RBI external-sector series | connector only; auth format undocumented | RBI reserves frozen at June 2025 in the connector; NAS indicator-list endpoint returns 500 |
| RBI Weekly Statistical Supplement | Weekly forex reserves, 1998 to date | scraped, `WSSView.aspx?Id=N` | "as on" date is 7 days before publish date |
| RBI Reference Rate Archive | Daily USD/INR | scraped, date-range form | WSS historical FX table (`PARAM1=6`) discontinued Dec 2012 |
| TRADESTAT (`tradestat.commerce.gov.in`) | HS-2 and HS-4 exports/imports with partner countries, FY2018-19 to FY2025-26 | scraped, cookie + CSRF then POST | provisional and dynamically revised |
| TIA Portal, DGFT, PIB, DPIIT annual report | Cross-checks | manual | TIA Portal ingests revisions continuously |
| US CPI-U | Real exchange rate | public | |
| amCharts `amcharts4-geodata` | India state boundaries | bundled | MIT |

## Testing
No test suite. `notebooks/repo_verification.ipynb` checks that every Pages URL and dataset in both repos
resolves; the GitHub workflow runs a gitleaks scan.

## Limitations and known issues
- **RBI forex reserves lag hard** in the connector: requesting 2025/2026 still returns data only through June 2025.
- **WPI's April 2026 jump** (~+6% in a quarter) and **IIP's February 2026 dip** (169.9 Jan to 158.8 Feb, 173.2 Mar) look unusually steep; cross-check against an official MoSPI/RBI release before treating either as fact.
- **NAS indicator list** hit an upstream `500` and silently fell back to a bundled local list (`_source: local_definitions_fallback`); the live GDP endpoint may be flaky.
- **IIP `get_data` requires an explicit `type`** (`General` / `Sectoral` / `Use-based category`); omitting it errors with `missing_required: type`.
- Large hierarchical metadata payloads (CPI, WPI) can exceed one response and need paging.
- **DGCI&S bulletins are provisional** and "dynamically revised on the basis of late receipt of data"; a live portal figure a few percent above a one-time pull is vintage drift, not necessarily an error.
- **Growth-rate conventions differ by portal** (TIA Portal defines monthly, quarterly, yearly and YTD); this repo's "% growth" is year-on-year on full-fiscal-year endpoints unless stated.
- The PDF versions of the capstone and the FY2025-26 report were removed from the repo (replaced by the `.md` extract); `index.html` still links to them.
- `data/pib_index.sqlite` is empty (0 bytes).
- No collector code is tracked, so none of the datasets can be regenerated from this repo alone.

## Related repositories
- [india-trade-sector-policy-recommendations](https://github.com/herrrickshaw/india-trade-sector-policy-recommendations) — the prescriptive half: PLI coverage, import-dependency gaps, country-by-country trade history, tiered sector/policy recommendations.
- [india-forex-analysis](https://github.com/herrrickshaw/india-forex-analysis) — earlier Markdown-only analysis of reserves, trade deficit and debt.

## Licence
No licence file yet — all rights reserved. Source data remains subject to its publishers' terms; the map
geodata is MIT-licensed by amCharts.
