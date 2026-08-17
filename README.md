# GSC Anomaly Detection

Three Google Colab notebooks that find anomalies in your Google Search Console data. Upload a CSV export, pick a metric, run the cells, get back a chart of flagged days plus a CSV you can hand to a client or drop into a report.

No local setup, no Python environment to manage, no cost. Everything runs in the browser on Google's hardware.

## The notebooks

| Notebook | Method | Best at | Weak spots |
|---|---|---|---|
| [`isolation-forest.ipynb`](isolation-forest.ipynb) | Isolation Forest | Multi-dimensional weirdness (clicks, impressions, CTR, position together), fast first-pass sniff test, global outliers | Gradual changes, anything seasonal |
| [`local-outlier-factor.ipynb`](local-outlier-factor.ipynb) | Local Outlier Factor | Weirdness relative to peers, keyword-level data, points that are odd within their own group, A/B comparisons | Global awareness, datasets with wildly disparate performance |
| [`forecast-residual.ipynb`](forecast-residual.ipynb) | STL decomposition + residual z-scores | Time series with real seasonality (including holidays), gradual and subtle shifts, SERP volatility | Short date ranges, can be persnickety to tune |

Start with Isolation Forest if you just want to know whether anything looks broken. Move to the others once you know what kind of weird you are chasing.

## Getting your data

Each notebook expects a CSV with daily rows and these columns:

```
Date, Clicks, Impressions, CTR, Position
```

That is exactly the shape of a Search Console export. In GSC, go to **Performance**, set your date range, open the **Dates** tab, and hit **Export → Download CSV**. Column names are normalized automatically, so capitalization and stray whitespace are fine, as are comma-separated numbers and percent-formatted CTR.

How much history you need:

* **Isolation Forest / LOF**: a couple of months is workable, more is better.
* **Forecast residual, weekly seasonality**: 2 to 3 months minimum.
* **Forecast residual, yearly seasonality**: at least 2 years, no way around it.

## Running one

1. Open the notebook in Colab: **File → Open notebook → GitHub**, then paste this repo's URL.
2. **File → Save a copy in Drive.** You want your own copy so your edits stick.
3. Run the cells top to bottom.
4. When prompted, upload your GSC CSV.
5. Adjust the configuration cell and re-run from that point if the results feel too noisy or too quiet.

**One gotcha:** the LOF notebook pins numpy to 1.26.4 in cell 1 to avoid a compatibility break. After that cell finishes you must go **Runtime → Restart runtime**, then skip cell 1 and start from cell 2. The other notebooks do not need a restart.

## Tuning

Every notebook has a single configuration cell. These are the knobs worth touching:

**Local Outlier Factor**

* `PRIMARY_METRIC`: which column to treat as the headline metric.
* `N_NEIGHBORS` (default 20): how much context each point is judged against. Lower (10 to 15) is more sensitive to local patterns and flags more. Higher (25 to 30) takes a broader view and flags less.
* `CONTAMINATION` (default 0.05): roughly what share of days you expect to be anomalous.

There is a parameter sweep cell near the end that runs a grid of both values and prints how many anomalies each combination catches. Use it before you commit to a setting.

**Forecast residual**

* `METRIC`: the column to decompose.
* `COUNTRY` (default `US`): drives the holiday calendar.
* `SEASONAL_PERIOD`: `7` for day-of-week effects, `365` for time-of-year effects. Blogs, news, and B2B usually want 7. Ecommerce, travel, and anything with a real annual cycle wants 365, but only if you have the history for it.
* `SIGMA_THRESHOLD` (default 3): how many standard deviations from the expected value counts as an anomaly. Drop to 2 for more hits, raise to 4 for only the extremes.

## What you get back

Each notebook renders its charts inline and then exports two CSVs, which download automatically:

* a full export with every date, the model's score, the anomaly flag, and a positive/negative classification
* an anomalies-only file for when you just want the list

The forecast-residual notebook also exports the decomposed trend, seasonal, and residual components alongside an `expected` column, which is useful when someone asks "what should traffic have been that day?"

## Reading the output

Both anomaly types matter. **Negative anomalies** are the ones that get attention, but **positive anomalies** are worth just as much investigation. An unexplained spike is often a tracking bug, a scraper, or a SERP feature you did not know you had.

A flagged day is a question, not an answer. Cross-reference against your own timeline before drawing conclusions: deploys, algorithm updates, migrations, campaign launches, outages, GSC reporting delays. The model has no idea any of those happened.

Also worth knowing: anomaly detection and forecasting use the same underlying models. The difference is entirely in how you apply them.

## Your data stays yours

These notebooks process everything inside your own Colab runtime. Your CSV is read into memory, analyzed, and written back out as a download. Nothing is transmitted to any external service. The only outbound network traffic is `pip` fetching packages.

One thing to watch, though: **clear your outputs before sharing a copy of a notebook you have run.** Charts and printed tables embed your actual traffic numbers directly in the `.ipynb` file, so a notebook shared after a run carries that data with it. Use **Edit → Clear all outputs**, then save, before you share a link, email the file, or commit it to a public repo.

The same applies to Colab's link sharing. Setting a notebook to "Anyone with the link" exposes whatever is currently sitting in its output cells.

## Where this came from

These notebooks were built for two talks:

* [Stop Losing SEO Traffic](https://speakerdeck.com/samtorres/mozcon-nyc-2025-stop-losing-seo-traffic), Mozcon NYC 2025
* [When Traffic Gets Weird: ML for Anomalies & Forecasts](https://speakerdeck.com/samtorres/when-traffic-gets-weird-ml-for-anomalies-and-forecasts), Tech SEO Connect 2025

The decks cover the reasoning behind picking one model over another, which is the part that does not fit in a notebook comment.

## License & contributions

Released under the [MIT License](LICENSE). Free to use, copy, modify, and redistribute, including commercially. Provided as is, with no warranty.

If you extend one of these or find a case where a model misbehaves, open an issue or a PR.
