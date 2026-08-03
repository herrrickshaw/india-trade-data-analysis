# India Trade &amp; Macro Data Analysis

A record of accessing India's official statistics via the **MoSPI (Ministry of Statistics and Programme Implementation)** data API/connector, plus TRADESTAT, RBI, and DGCI&amp;S sources — covering domestic macro indicators (CPI/WPI/GDP), currency &amp; reserves, and the descriptive trade-balance/HSN-chapter analysis. Snapshot of the latest published figures pulled on 2026-07-18.

**This is the descriptive half of a two-repo split.** The prescriptive companion — sector-by-sector and country-by-country recommendations, PLI scheme coverage checks, and policy-gap analysis — lives in [`india-trade-sector-policy-recommendations`](https://github.com/herrrickshaw/india-trade-sector-policy-recommendations). Both repos were originally one (`mospi-dataset-analysis`); they were split so the "what does the data show" analysis and the "what should be done about it" recommendations could be published, versioned, and read independently.

**Live site**: [herrrickshaw.github.io/india-trade-data-analysis](https://herrrickshaw.github.io/india-trade-data-analysis/) — every chart and JSON dataset below served directly via GitHub Pages, no build step.

This repo reads chronologically — each section was added as a new question came up, often building on the one before it. The **[five-year synthesis](#five-year-synthesis-trade-currency--policy-fy2021-22-to-fy2025-26)** near the end re-cuts every thread below to one consistent window if you want the short version first.

**Quick references, outside the chronological bulletins below:**
- [`reports/FY2025-26_sample_report.html`](https://herrrickshaw.github.io/india-trade-data-analysis/reports/FY2025-26_sample_report.html) / [`reports/FY2025-26_sample_report.pdf`](reports/FY2025-26_sample_report.pdf) — a standalone FY2025-26 snapshot: every figure re-derived from the datasets below, standardised to **US$ Million throughout** (PLI ₹ crore figures each converted at that scheme's own elapsed-period average rate — not one blanket rate), with a dedicated cross-source consistency-check log.
- [`charts/five_year_trade_currency_synthesis.html`](https://herrrickshaw.github.io/india-trade-data-analysis/charts/five_year_trade_currency_synthesis.html) / [`charts/five_year_trade_currency_synthesis.pdf`](charts/five_year_trade_currency_synthesis.pdf) — the five-year capstone (see below), also published as a standalone PDF.
- [`notebooks/mospi_live_reference.ipynb`](notebooks/mospi_live_reference.ipynb) — open in Colab — loads every dataset in this repo live from GitHub (plus the companion repo's PLI/policy datasets), two working live-scrape cells (RBI reserves, live USD/INR), and a tour of Python visualization toolkits (Seaborn, Plotly, Altair, Folium, missingno) alongside matplotlib.

**Contents:** [MoSPI dataset catalogue](#what-mospi-provides) · [Access workflow](#access-workflow) · [Session findings](#session-findings-as-of-2026-07-18) · [State CPI vs. WPI](#state-wise-cpi-vs-national-wpi-trends) · [CPI heatmap](#cpi-inflation-heatmap-choropleth) · [Forex &amp; currency](#rbi-foreign-exchange-reserves--currency-trend) · [Real exchange rate &amp; China comparison](#effective-buying-power--the-china-currency-manipulation-comparison) · [Trade balance](#trade-balance--is-india-an-export-surplus-country) · [HSN growth trends](#hsn-wise-historical-trends--which-imports-are-growing-fastest) · [Fertiliser/fuel → inflation](#imported-fertiliser--fuel-prices--diesel--cpiwpi) · [GDP growth](#gdp-growth-trend-with-revision-history) · [Five-year synthesis](#five-year-synthesis-trade-currency--policy-fy2021-22-to-fy2025-26) · [Known quirks](#known-quirks--caveats)

## What MoSPI provides

The connector fronts `api.mospi.gov.in` and exposes **25 datasets** covering 500+ indicators:

| Dataset | Covers |
|---|---|
| CPI | Consumer Price Index (retail inflation), Group/Item hierarchy, base years 2010/2012/2024 |
| WPI | Wholesale Price Index, base years 1993-94/2004-05/2011-12 |
| IIP | Index of Industrial Production (manufacturing/mining/electricity), monthly & annual |
| NAS | National Accounts Statistics (GDP, GVA, consumption, savings) |
| RBI | External sector: forex reserves, exchange rates, balance of payments, external debt |
| PLFS | Periodic Labour Force Survey (LFPR, WPR, unemployment rate, wages) |
| ASI | Annual Survey of Industries (factory-level financials) |
| ASUSE, EC | Informal sector / unincorporated enterprises, Economic Census |
| AISHE, UDISE | Higher education & school education statistics |
| GENDER, NFHS | Gender statistics, National Family Health Survey |
| ENVSTATS, MNRE | Environment statistics, renewable energy capacity |
| HCES, TUS | Household consumption expenditure, Time use survey |
| NSS75E–NSS80 | Various NSS survey rounds (education, disability, housing, telecom, land & livestock, AIDIS, AYUSH) |
| CPIALRL | CPI for Agricultural/Rural Labourers |

## Access workflow

Four steps, each depending on the previous:

```
list_datasets()                          -> identify the dataset
get_indicators(dataset)                  -> list available indicators
get_metadata(dataset, indicator_code...) -> valid filter values (years, states, categories...)
get_data(dataset, filters)                -> fetch the actual data
```

Filter codes are dataset-specific and not standardized (e.g. `indicator_code=3` means different things in different datasets) — always resolve them via `get_metadata` rather than guessing.

This repo currently documents that workflow as exercised through the MoSPI MCP connector inside Claude Code; it is not (yet) a standalone script, since reproducing it outside that environment would require reverse-engineering `api.mospi.gov.in`'s undocumented auth/request format from scratch.

## Session findings (as of 2026-07-18)

Every headline insight in this repo, pulled up to one place and ordered to match the sections below. Each linked section still carries its own full narrative, methodology, and **underlying data source** — this index is the "insights" half; the sections below hold both the fuller description *and* their data-source citation, kept separate from the summary here.

**Raw MoSPI connector snapshot** (the very first pull this session, not a derived insight):

| Series | Latest period returned | Value |
|---|---|---|
| CPI (base 2024, All India, Combined) | June 2026 | Index 107.00, **+4.38% YoY** (rural 4.74%, urban 3.92%; food & beverages +5.05%) |
| IIP (General, base 2011-12) | March 2026 | Index 173.2, **+4.1% YoY** |
| WPI (base 2011-12, overall) | April 2026 | Index 167 (Jan 157.6 → Feb 158.4 → Mar 160.8 → Apr 167) |
| RBI Foreign Exchange Reserves (Total, US$) | June 2025 | $698.1bn |

Raw responses: [`data/mospi_snapshot_2026-07-18.json`](https://herrrickshaw.github.io/india-trade-data-analysis/data/mospi_snapshot_2026-07-18.json).

**All derived insights, by section:**

- **[State-wise CPI vs. WPI](#state-wise-cpi-vs-national-wpi-trends)** — Telangana runs hottest at +6.36% YoY, NCT of Delhi coolest at +2.96%, against an All-India print of +4.38%. WPI broke a year of near-flat readings (~154–158) with a sharp run to 167 in April 2026.
- **[RBI forex reserves + currency trend](#rbi-foreign-exchange-reserves--currency-trend)** — Reserves hit a new all-time high of $728.5bn (27 Feb 2026), slid to a true trough of $666.9bn (26 Jun 2026, -8.5% peak-to-trough), then partially recovered to $675.2bn (10 Jul 2026). The rupee depreciated ~38% over the decade, then accelerated to +11.9% in the last 12 months, reaching ₹96.37 on 17 Jul 2026.
- **[Effective buying power & the China comparison](#effective-buying-power--the-china-currency-manipulation-comparison)** — Nominal rupee depreciation since Jan 2015 is +54.9%, but the real (PPP-adjusted) rate is +37.6% — effective buying power for dollar-priced goods down ~27%. Unlike China's 2019 currency-manipulator designation, RBI has been *selling* reserves to defend the rupee, not buying to suppress it.
- **[Trade balance](#trade-balance--is-india-an-export-surplus-country)** — No: India ran a merchandise deficit in every one of 8 fiscal years on record, widening from -$184.0bn (FY2018-19) to -$334.3bn (FY2025-26). The services surplus more than doubled ($82.1bn → $188.9bn), cutting the FY2024-25 overall gap to just -$94.6bn.
- **[HSN-wise historical trends](#hsn-wise-historical-trends--which-imports-are-growing-fastest)** — Mineral fuels (HS27) is the largest single import ($203.4bn) but grew only +21.2% over 8 years. Fertilisers (+118.9%), electrical machinery (+101.5%), and edible oils (+97.9%) are the fastest-growing imports. Electrical machinery exports quadrupled (+324.1%); gems & jewellery (HS71) is the only top-12 chapter shrinking, on either side (-29.9%).
- **[Imported fertiliser & fuel prices → CPI/WPI](#imported-fertiliser--fuel-prices--diesel--cpiwpi)** — Diesel passes through to WPI almost mechanically (r=0.92, +79.0% since April 2012); fertiliser subsidy holds Urea to just +13.8% despite the 2021-23 global price shock. CPI oils & fats swung from -18.17% to +21.24% year-on-year in the space of two years.
- **[GDP growth trend](#gdp-growth-trend-with-revision-history)** — FY2025-26 opens at +7% real growth (First Advance Estimate, expect revision). The COVID-19 year was revised from an initial -8% up to -6%; typical revisions run about ±0.75 percentage points, as much as 2pp.
- **[Five-year synthesis](#five-year-synthesis-trade-currency--policy-fy2021-22-to-fy2025-26)** — Imports grew nearly six times faster than exports over five years (+26.6% vs. +4.7%, a 5.7:1 ratio). A services surplus — not five years of PLI policy — absorbed roughly 88% of the widened goods deficit. China's import concentration and the USA's export concentration haven't meaningfully shifted despite scaled-up PLI outlays.

## State-wise CPI vs. national WPI trends

[`charts/cpi_wpi_state_trends.html`](https://herrrickshaw.github.io/india-trade-data-analysis/charts/cpi_wpi_state_trends.html) — open in a browser — charts Combined-sector CPI (base 2024) across 14 major states plus the All-India benchmark from January 2025 through June 2026, ranked by latest year-on-year inflation, alongside the national WPI trend over the same window (WPI has no state-level breakdown in MoSPI's data).

Headline: **Telangana runs hottest at +6.36% YoY**, **NCT of Delhi coolest at +2.96%**, against an All-India print of +4.38%. WPI broke a year of near-flat readings (~154–158) with a sharp run to 167 in April 2026.

Underlying data: [`data/cpi_statewise_trend_2025-01_to_2026-06.json`](https://herrrickshaw.github.io/india-trade-data-analysis/data/cpi_statewise_trend_2025-01_to_2026-06.json), [`data/wpi_national_trend_2025-01_to_2026-04.json`](https://herrrickshaw.github.io/india-trade-data-analysis/data/wpi_national_trend_2025-01_to_2026-04.json).

## CPI inflation heatmap (choropleth)

[`charts/cpi_india_heatmap.html`](https://herrrickshaw.github.io/india-trade-data-analysis/charts/cpi_india_heatmap.html) — open in a browser — a choropleth of India shaded by June 2026 CPI year-on-year inflation for the 13 states in this dataset (all other states/UTs shown in gray as "not tracked", not as low inflation). State boundaries come from amCharts' `amcharts4-geodata` (`india2023Low`, MIT-licensed), reprojected to SVG locally — no external map tiles or API calls at render time.

## RBI foreign exchange reserves + currency trend

[`charts/rbi_forex_reserves_trend.html`](https://herrrickshaw.github.io/india-trade-data-analysis/charts/rbi_forex_reserves_trend.html) — open in a browser — two segments on India's external sector. India's total forex reserves from Jan 2015 to the latest available reading. The MoSPI connector itself is confirmed still frozen at June 2025 (re-checked 2026-07-18, no change) — everything from June 2025 onward is a **real weekly series scraped directly from RBI's own site**, shown as a visually distinct dashed extension, never silently merged with the connector data.

**How the scrape works**: RBI's Weekly Statistical Supplement archive is fully enumerable — every release from 19 Sep 1998 to today sits at a sequential `rbi.org.in/scripts/WSSView.aspx?Id=N` URL, no API key or login needed (unlike `datagovindia`, which wraps data.gov.in and requires a free API key from that portal before anything — including search — will work). One quirk worth knowing: each release's *"as on"* reserve date is 7 days before its publish date, so taking the listed release date at face value mislabels every reading by a week.

That scrape changes the story from the previous version of this chart: reserves hit a **new all-time high of $728.5bn on 27 Feb 2026** (surpassing the $705.8bn Sep 2024 peak this bulletin previously called the record), then kept sliding — through a "West Asia conflict" shock and RBI rupee-defense intervention — to a **true trough of $666.9bn on 26 Jun 2026** (-8.5% peak-to-trough), before a partial recovery to **$675.2bn by 10 Jul 2026**, the latest reading anywhere, including RBI's own site as of this bulletin.

**New currency segment**: the rupee against the US dollar, same Jan 2015–Jul 2026 window, bridged past the connector's July 2025 endpoint with a scrape of RBI's separate **Reference Rate Archive** (`rbi.org.in/scripts/referenceratearchive.aspx`) — a date-range search form, not an enumerable ID archive like the reserves pages. (A dead end along the way: the WSS's own historical exchange-rate table, `PARAM1=6`, was discontinued in December 2012.) The rupee depreciated **~38% over the decade** (₹62.23 → ₹86.11, ~3.3%/year), then accelerated to **+11.9% in the last 12 months alone**, reaching ₹96.37 on 17 Jul 2026 — the same window as the reserves chart's "war shock" slide above.

Underlying data: [`data/rbi_forex_reserves_2015-01_to_2025-06.json`](https://herrrickshaw.github.io/india-trade-data-analysis/data/rbi_forex_reserves_2015-01_to_2025-06.json) (reserves) and [`data/rbi_usd_inr_exchange_rate_2015-01_to_2026-07.json`](https://herrrickshaw.github.io/india-trade-data-analysis/data/rbi_usd_inr_exchange_rate_2015-01_to_2026-07.json) (currency).

## Effective buying power & the China currency-manipulation comparison

[`charts/real_exchange_rate_and_currency_policy.html`](https://herrrickshaw.github.io/india-trade-data-analysis/charts/real_exchange_rate_and_currency_policy.html) — open in a browser — a follow-on to the currency segment, separating the rupee's nominal fall into "just India's higher inflation catching up" versus a genuine real (purchasing-power-adjusted) depreciation, using India's implied GDP deflator (from the GDP chart's own NAS data) against US CPI-U.

**Headline**: nominal depreciation since Jan 2015 is **+54.9%**, but the real (PPP-adjusted) rate still moved **+37.6%** — meaning a rupee's **effective buying power for dollar-priced goods has fallen ~27%** even after fully crediting India's own inflation. The last 12 months are almost entirely real, not inflationary: India's implied inflation this year (~0.9%) was *lower* than the US's, so PPP alone would have predicted the rupee strengthening — instead it fell 11.9% nominally (14.0% in real terms).

That's then read against the framework China was scrutinized under: the US Treasury's three-test "currency manipulator" criteria (trade surplus, current account surplus, and — the one that matters here — persistent net *purchases* of foreign currency to keep a currency artificially weak). China was formally designated a manipulator in Aug 2019, un-designated in Jan 2020. India's reserves data (from the segment above) shows the RBI doing the mechanical opposite of what tripped that third test — *selling* reserves to defend the rupee, not buying foreign currency to suppress it — the opposite motive from the one "manipulation" describes, even though the real depreciation is genuine. Cites Brookings' ["China's Currency Policy, Explained"](https://www.brookings.edu/articles/chinas-currency-policy-explained/) for the China policy context.

Underlying data: [`data/rupee_real_exchange_rate_2015_to_2026.json`](https://herrrickshaw.github.io/india-trade-data-analysis/data/rupee_real_exchange_rate_2015_to_2026.json).

## Trade balance — is India an export-surplus country?

[`charts/trade_balance_hsn_analysis.html`](https://herrrickshaw.github.io/india-trade-data-analysis/charts/trade_balance_hsn_analysis.html) — open in a browser — HSN (2-digit HS chapter) merchandise export and import data from the Commerce Ministry's own **TRADESTAT** database (`tradestat.commerce.gov.in`, not MoSPI/RBI), summed across all 98 tracked chapters to reconstruct India's national trade balance for FY2018-19 through FY2025-26.

**Verdict: no — in goods alone, or goods plus services.** India has run a merchandise trade deficit in **every one of the 8 fiscal years** on record here, widening from **-$184.0bn (FY2018-19) to -$334.3bn (FY2025-26)**. Exports covered 64.2% of the import bill in FY2018-19; they cover just **56.9%** now. **Mineral fuels (HS27, -$147.5bn)** and **pearls/gems/jewellery (HS71, -$81.1bn)** are the two largest deficit-driving chapters; **pharmaceuticals, vehicles, cereals and apparel** are the strongest surplus chapters. This closes the loop on the currency and reserves segments above: a persistent, widening goods deficit is exactly the kind of structural pressure that would push the rupee down over time, independent of any single shock — the backdrop the RBI has been spending reserves to lean against.

**How it was pulled**: TRADESTAT is a Laravel/Livewire app whose report URLs return `405` on a cold GET — visiting the search page first (to pick up a session cookie + CSRF token) then POSTing the form fields works, and each query returns two fiscal years at once, so 8 requests (4 per trade direction) covered the full 8-year window.

**Added: the services-trade offset.** Checked DPIIT and the Commerce Ministry for services data per request — **DPIIT doesn't publish services-trade statistics** (its main product is a goods import-monitoring system plus FDI data); the Commerce Ministry's own DGCI&S does track services exports via annual "Service Exports Reporting Form" PDF/Excel reports, but not as a queryable database. Used **RBI's Balance of Payments "Non-factor Services" invisibles series** (via the MoSPI connector's RBI dataset) instead, converted from ₹ Crore using fiscal-year-average exchange rates built from this repo's own currency segment. Result: India's services surplus **more than doubled**, from **$82.1bn (FY2018-19) to $188.9bn (FY2024-25)** — enough to cut the FY2024-25 overall gap from **-$283.5bn to just -$94.6bn**, and to nearly close it in the COVID year (-$14.1bn). Still a deficit every year — just roughly half the size the goods numbers alone suggest.

Underlying data: [`data/tradestat_hsn_export_import_2018-19_to_2025-26.json`](https://herrrickshaw.github.io/india-trade-data-analysis/data/tradestat_hsn_export_import_2018-19_to_2025-26.json) (goods, all 98 HS chapters) and [`data/services_trade_and_overall_balance_2015-16_to_2024-25.json`](https://herrrickshaw.github.io/india-trade-data-analysis/data/services_trade_and_overall_balance_2015-16_to_2024-25.json) (services + combined balance).

**Cross-validated (2026-07-18)** against three more official sources, per request: the Commerce Ministry's **Trade Intelligence and Analytics Portal** (`trade-analytics.commerce.gov.in`), **DGFT**'s trade-statistics hub, and a **PIB** press release. Imports match almost to the dollar ($776,013.65M on TIA Portal vs. $776,013.6M here); the largest export chapter (HS27) matches exactly ($55,871.2M both); PIB's official FY2025-26 exports ($441.78bn) and deficit ($333.19bn) match within a rounding error. One flagged discrepancy: TIA Portal's own export total reads ~2% ($9.3bn) higher than both this bulletin and PIB — likely a data-vintage difference on a still-provisional fiscal year, not a methodology error (this was independently confirmed later: DGCI&S's own bulletins are "dynamically revised... on the basis of late receipt of data," and TIA Portal continuously ingests revisions while a one-time pull or press release captures an earlier, less-revised snapshot). DGFT's "Import Export Data Bank" turned out to just be a link back to TRADESTAT itself, not an independent source. DPIIT's 295-page 2025-26 Annual Report doesn't publish trade-balance data (as already found), but its Industrial Entrepreneur Memorandum investment-intention filings independently corroborate the historical-trends chart below: proposed investment in Electricals Equipment and Fertilizers — this repo's two fastest-growing high-value import chapters — is also scaling up fastest in DPIIT's own, completely separate licensing data.

Underlying validation data: [`data/trade_data_cross_validation_2026-07-18.json`](https://herrrickshaw.github.io/india-trade-data-analysis/data/trade_data_cross_validation_2026-07-18.json).

## HSN-wise historical trends — which imports are growing fastest?

[`charts/hsn_historical_trends.html`](https://herrrickshaw.github.io/india-trade-data-analysis/charts/hsn_historical_trends.html) — open in a browser — the same TRADESTAT chapter-level data as the trade-balance bulletin above, re-cut to track each HS-2-digit chapter's own 8-year trajectory (FY2018-19 → FY2025-26) rather than just its latest-year snapshot. Ranks the top-12 import and top-12 export chapters by FY2025-26 value (independently-scaled sparklines per chapter), then re-ranks the same 12 import chapters by **CAGR** to separate "large" from "fast-growing."

**Headline: the biggest imports aren't the fastest-moving ones.** Mineral fuels (HS27) is by far the largest single import ($203.4bn) but has grown only **+21.2%** (2.8% CAGR) over eight years — largely a price story. The real acceleration is in **Fertilisers (HS31, +118.9%, 11.8% CAGR)** — plausibly global fertiliser price shocks since 2022 stacked on rising volumes — followed by **Electrical machinery (HS85, +101.5%, 10.5% CAGR)** and **Animal/vegetable fats & oils (HS15, +97.9%, 10.2% CAGR)**. Top 12 import chapters account for **81.5%** of all FY2025-26 imports. On the export side, **Electrical machinery (HS85) has quadrupled** (+324.1%, 22.9% CAGR, $12.7bn → $54.0bn) — by far the fastest mover on either side of the ledger — while **Pearls, gems & jewellery (HS71)** is the only top-12 chapter shrinking on either side (**-29.9%**) — a decline independently corroborated by NITI Aayog's Trade Watch, which separately flags several labour-intensive export sectors (including pearls/gems) as having lost global market share since 2015. See the [companion recommendations repo](https://github.com/herrrickshaw/india-trade-sector-policy-recommendations) for what to do about it.

Read against the trade-balance bulletin: the chapters driving the *deficit* (mineral fuels, gems/jewellery) are not the chapters driving its *momentum* — fertilisers, electronics components, and edible oils are where new import dependency is actually building.

Official validation note: the Commerce Ministry's own **TIA Portal user manual** confirms this repo's HS84/HS85 grouping and HS71 standalone treatment match DGCI&S's official 13-sector taxonomy (Electronic Goods = HS84-85; Gems and Jewellery = HS71 alone) — see the [companion repo's recommendations bulletin](https://github.com/herrrickshaw/india-trade-sector-policy-recommendations/blob/main/charts/sector_and_policy_recommendations.html) for the full table.

Underlying data: [`data/hsn_historical_trends_2018-19_to_2025-26.json`](https://herrrickshaw.github.io/india-trade-data-analysis/data/hsn_historical_trends_2018-19_to_2025-26.json).

## Bare-metal imports at 4-digit HS — splitting bullion out of chapter 71

The 2-digit chapter data above cannot separate gold from diamonds (both sit in HS71) or the copper value chain from its chapter aggregates — so this dataset pulls the key bare-metal 4-digit codes from the same TRADESTAT source, with full partner-country breakdowns, for FY2024-25 vs FY2025-26 (provisional): gold 7108, silver 7106, platinum-group 7110, diamonds 7102 (for chapter-71 context), the copper complex (ores 2603, blister 7402, refined cathodes 7403, scrap 7404), aluminium (unwrought 7601, scrap 7602), and unwrought nickel 7502 / lead 7801 / zinc 7901 / tin 8001.

**Headline: bare gold alone is 9.3% of India's entire FY2025-26 import bill** ($72.0bn, +24% YoY), second only to crude — with Switzerland ($20.1bn) and UAE ($15.3bn, the CEPA concessional channel) supplying half of it. Three patterns stand out: (1) **the platinum-group line collapsed -92%** ($5.4bn → $0.4bn) after customs closed the "platinum alloy" duty-arbitrage route, and that flow visibly snapped back into 7108 — the same bullion-arbitrage migration this project separately caught in gold compounds (HSN 28433000); (2) **sourcing is shifting to doré from mining countries** for domestic refining (Ghana +449%, Bolivia +245%, Brazil +538% YoY); (3) **silver imports tripled** to $12.1bn on the price rally plus solar-PV industrial demand. On the industrial side, the copper complex reached $14.7bn with concentrate (+54% YoY) growing fastest — new domestic smelting capacity substituting cathode imports with ore imports, i.e. import dependency moving down the value chain rather than shrinking.

Underlying data: [`data/metals_hsn4_import_2024-25_to_2025-26.json`](https://herrrickshaw.github.io/india-trade-data-analysis/data/metals_hsn4_import_2024-25_to_2025-26.json).

## Imported fertiliser & fuel prices → diesel → CPI/WPI

[`charts/fertiliser_fuel_price_transmission.html`](https://herrrickshaw.github.io/india-trade-data-analysis/charts/fertiliser_fuel_price_transmission.html) — open in a browser — correlates the MoSPI connector's own WPI item-level series for diesel (HSD) and fertilisers (Urea, DAP) against WPI and CPI headline/food inflation, 2012–2026, to test the import-price pass-through described in five published studies (cited in full inside the bulletin: James Hamilton's IJCB framing paper, Bhattacharya & Gupta's NIPFP food-inflation SVAR, Mishra & Roy's India Policy Forum paper, a JETIR 2025 survey, and an MBA thesis on India's import composition).

**Headline: diesel passes through to WPI almost mechanically (r=0.92); fertiliser mostly doesn't, because subsidy absorbs it.** Diesel (WPI HSD) is up **+79.0%** since April 2012, moving in near lockstep with WPI headline. **Urea is up only +13.8%** over the same window despite the 2021–23 global fertiliser shock — the subsidy regime holds it down almost regardless of the border price — while **DAP (smaller, less consistent subsidy) is up +38.4%**, roughly midway between the two. Against CPI, the picture is more textured: same-year correlations between these input costs and CPI headline/food/oils-and-fats are weak or inconsistent (r between -0.52 and +0.29) at annual resolution — not because the transmission described in the literature isn't real, but because (per Bhattacharya & Gupta's SVAR) it operates on a 1–4 month lag and fades within the year, which annual year-on-year correlation washes out. The one series that *does* show a dramatic, real swing is **CPI oils & fats**, whose monthly print went from **-18.17% (Jun 2023) to +21.24% (Aug 2025)** — matching Mishra & Roy's finding that edible oils are the one food category with near-1:1 global pass-through, and this repo's own trade data showing edible-oil imports (HS15) up 97.9% over the same window.

Underlying data: [`data/fertiliser_fuel_price_transmission_2012_to_2026.json`](https://herrrickshaw.github.io/india-trade-data-analysis/data/fertiliser_fuel_price_transmission_2012_to_2026.json).

## GDP growth trend, with revision history

[`charts/gdp_growth_trend.html`](https://herrrickshaw.github.io/india-trade-data-analysis/charts/gdp_growth_trend.html) — open in a browser — real (constant-price) GDP growth by fiscal year, FY2012-13 through FY2025-26, from the MoSPI connector's NAS dataset. Bars show the latest revised estimate for each year; a tick marks where the original First Advance Estimate landed, so the gap between tick and bar visualizes how much each year's number moved as later data came in.

Checked after last bulletin flagged the NAS `getNasIndicatorList` metadata endpoint as broken (upstream 500) — that endpoint is **still** down, but the actual `get_data` call works fine and returns real figures through FY2025-26's First Advance Estimate. Went looking at data.gov.in/apis first for a GDP source, per request; every economic dataset found there (forex reserves, GDP, Index of Eight Core Industries) turned out to be a static one-off Rajya Sabha Q&A release or a Ministry file frozen years ago despite a recent "Updated On" timestamp — none usable. The connector's own NAS data turned out to be the better, more current source already sitting unused.

Headline: **FY2025-26 opens at +7% real growth** (First Advance Estimate only — expect revision). The COVID-19 year (FY2020-21) was revised from an initial **-8%** up to **-6%**; FY2023-24 saw the largest upward revision of any year, from **+7%** to **+9%**. Across years with more than one estimate, the typical revision is about **±0.75 percentage points**, as much as **2pp** in either direction.

Underlying data: [`data/gdp_growth_rate_2012-13_to_2025-26.json`](https://herrrickshaw.github.io/india-trade-data-analysis/data/gdp_growth_rate_2012-13_to_2025-26.json).

## Five-year synthesis: trade, currency &amp; policy, FY2021-22 to FY2025-26

[`charts/five_year_trade_currency_synthesis.html`](https://herrrickshaw.github.io/india-trade-data-analysis/charts/five_year_trade_currency_synthesis.html) (open in a browser) / [`charts/five_year_trade_currency_synthesis.pdf`](charts/five_year_trade_currency_synthesis.pdf) (standalone PDF) — a capstone that re-cuts every dataset already collected above to one consistent five-year window, rather than the mix of 8-year (HSN), 14-year (WPI/CPI), and decade-plus (currency/reserves) windows used elsewhere in this repo. No new data collection — every figure is re-derived from the JSON files already cited above (plus the companion repo's country-priority datasets for the closing verdict).

**Headline: imports grew nearly six times faster than exports, and a services surplus — not five years of PLI policy — absorbed most of the difference.** Imports rose **+26.6%** ($613.1bn → $776.0bn, FY2021-22 → FY2025-26) against export growth of just **+4.7%** ($422.0bn → $441.7bn) — a growth ratio of roughly **5.7:1**. Over the same span the rupee's FY-average rate slid **+13.7%** (74.50 → 84.68 through FY2024-25) and has since accelerated to a **96.37** spot rate by mid-July 2026. The goods deficit widened by **$92.5bn** (-$191.0bn → -$283.5bn, FY2021-22 → FY2024-25), but the services surplus very nearly doubled (**+75.6%**, $107.6bn → $188.9bn) — covering an estimated **88% of that extra goods shortfall** and holding the overall gap's five-year growth to just $11.1bn. GDP growth ran 10%→8%→9%→6%→7% across the window (softest in FY2024-25); forex reserves grew 18.7% start-of-FY-to-latest, slower than the import bill.

Laid alongside the PLI policy timeline compressed to the same five years — Auto (₹25,938cr, Sep 2021), Textiles (₹10,683cr, Sep 2021), White Goods (all 85 companies selected by 2024), the 2023 expansion to 14 sectors (₹1.97 lakh crore), and Specialty Steel round 1.2 (₹11,887cr, MoUs Feb 2026) — the picture is consistent with the [companion repo's](https://github.com/herrrickshaw/india-trade-sector-policy-recommendations) country-concentration findings: **China's grip on India's electronic-component imports (~62%) and the USA's grip on India's fastest-growing exports (11 of 12 chapters) haven't meaningfully shifted** in the same window that PLI outlays scaled up. The policy response is real; it hasn't yet bent the two growth curves that matter most. For explicit, tiered recommendations on what to do about this, see the [Sector &amp; Policy Recommendations bulletin](https://github.com/herrrickshaw/india-trade-sector-policy-recommendations/blob/main/charts/sector_and_policy_recommendations.html) in the companion repo.

Underlying data: [`data/five_year_trade_currency_synthesis_2026-07-18.json`](https://herrrickshaw.github.io/india-trade-data-analysis/data/five_year_trade_currency_synthesis_2026-07-18.json).

## Known quirks / caveats

- **RBI forex reserves lag hard** — requesting 2025/2026 explicitly still only returned data through June 2025. Don't assume this series is current to the same month.
- **WPI's April 2026 jump** (+~6% over one quarter) and **IIP's February 2026 dip** (index fell from 169.9 in Jan to 158.8 in Feb, then rose to 173.2 in March) both look unusually steep/non-monotonic — worth cross-checking against an official MoSPI/RBI press release before treating either as fact.
- **NAS (GDP) indicator list** hit an upstream `500` from `api.mospi.gov.in` during this session and silently fell back to a bundled local indicator list (`_source: local_definitions_fallback`) — the live GDP endpoint may be intermittently flaky.
- **IIP's `get_data` requires an explicit `type` param** (`General`/`Sectoral`/`Use-based category`) — omitting it errors with `missing_required: type`, even though `category_code` alone looks sufficient.
- Large hierarchical metadata payloads (CPI, WPI) can exceed a single response's size limit and need paging/streaming rather than one bulk pull.
- **DGCI&S's own foreign trade bulletins are provisional and "dynamically revised... on the basis of late receipt of data"** (confirmed from the Commerce Ministry's own March 2024 Monthly Bulletin) — a live portal figure (like TIA Portal's) running a few percent higher than a one-time TRADESTAT pull from earlier in the fiscal year is expected data-vintage drift, not necessarily an error in either source.
- **Growth-rate conventions differ by portal** — TIA Portal's own user manual defines four distinct conventions (Monthly = vs. previous month, Quarterly = vs. previous quarter, Yearly = vs. previous year, YTD = year-to-date vs. same period prior year). This repo's own "% growth" figures are year-on-year on full-fiscal-year endpoints unless stated otherwise — always check which convention a cited figure uses before comparing it across sources.

## Companion repo

[`india-trade-sector-policy-recommendations`](https://github.com/herrrickshaw/india-trade-sector-policy-recommendations) — the prescriptive half of this split: PLI scheme coverage (import- and export-side), import-dependency policy gaps, country-by-country trade-relationship history, and an explicit tiered sector/policy recommendations bulletin cross-checked against NITI Aayog's Trade Watch Quarterly framework.

<!-- 
DATA LIBRARY LINK - Add this section to every repo README.md
This snippet provides discovery and documentation links.
-->

## 📊 Data Discovery

This repository is part of the **Global Data Library** — a unified catalog of 10,528 datasets across 40+ repositories.

### Quick Links

- **[Global Data Library README](.ruflo/DATA_LIBRARY_README.md)** — Full catalog, search API, and usage examples
- **[Data Library Python Interface](.ruflo/data-library/data_library.py)** — Query datasets programmatically
- **[Repository Scanner](.ruflo/data-library/repo_scanner.py)** — Reindex all repos to update the catalog

### Datasets in This Repository

The data catalog automatically inventories all datasets in this repo. To find your data:

```python
from data_library import DataLibrary

lib = DataLibrary()

# Search this repo's datasets
results = lib.search("", source="<repo-name>")

# Get dataset details
dataset = lib.get("<dataset_id>")
print(f"Rows: {dataset['row_count']}")
print(f"Freshness: {dataset['freshness_hours']} hours old")
print(f"Storage: {dataset['storage_tier']}")
```

### Browse the Full Catalog

**Market Coverage** (5 markets, 21,279 symbols):
- India (NSE/BSE): 2,364 instruments
- US (NASDAQ/NYSE): 7,442 instruments
- Europe (17 exchanges): 1,214 instruments
- Japan (TSE): 3,709 instruments
- Korea (KRX): 2,768 instruments

**Government Sources** (30+ ministries):
- MOSPI: 25 datasets (GDP, CPI, trade, agri, power)
- SEBI: 151,928 XBRL results + IPO pipeline
- PIB: 25+ ministry announcements
- DGFT: India trade data (monthly)
- Agmarknet: 300+ mandi prices (daily)
- NSE/MCX: Real-time derivatives chains

See [Global Data Library README](.ruflo/DATA_LIBRARY_README.md) for complete documentation.

### Finding Data Across All Repos

```python
# Find India OHLCV data (might be in multiple repos)
lib.search("india ohlcv", market="india")

# Get the fastest/freshest version
optimal = lib.get_optimal("india ohlcv", latency="<100ms", freshness="<1day")
# Returns: {"storage_tier": "cassandra", "path": "..."}

# Check data gaps
gaps = lib.gaps("india", date_from="2026-01-01")

# See which collectors are stale
status = lib.collectors_status()
```

---
