# Bitcoin and Its So-Called Mysteries
### COVID-19 News Sentiment and Bitcoin Market Behaviour

A Python for Data Science and AI project testing whether pandemic-related news sentiment moved Bitcoin's price during the COVID-19 crisis.

---

## TL;DR

- Global pandemic news got **significantly more negative** during the crisis (mean tone −2.50 → −3.39, Welch t = 4.94, p < 0.001).
- Bitcoin's daily returns showed **no relationship** with daily changes in that sentiment (Pearson r = −0.043, p = 0.56), and no lagged relationship within ±10 days either.
- Both hypotheses are rejected. Bitcoin behaved as a **sentiment-resistant, semi-detached asset**, driven more by crypto-market dynamics than by the global news mood.

---

## Research Questions and Hypotheses

**RQ1 – How did the COVID-19 crisis affect public sentiment?**

| | Statement |
|---|---|
| H₀ | The COVID-19 crisis has no positive impact on public mood toward semi-detached financial assets like Bitcoin. |
| Hₐ | The COVID-19 crisis has a positive impact on public mood toward semi-detached financial assets like Bitcoin. |

**RQ2 – How did daily sentiment changes during the pandemic relate to Bitcoin's price?**

| | Statement |
|---|---|
| H₀ | Bitcoin price does not increase with positive sentiment. |
| Hₐ | Bitcoin price increases with positive sentiment. |

During COVID-19 the market was highly volatile. Testing RQ2 shows whether Bitcoin reacts emotionally like other assets or stays independent of public mood.

---

## Data

| Source | What | Access |
|---|---|---|
| [CryptoCompare API](https://min-api.cryptocompare.com/documentation/) (`histoday`) | Daily BTC/USD closing price | API key |
| [GDELT 2.0 Global Knowledge Graph](https://www.gdeltproject.org/data.html) (`gdelt-bq.gdeltv2.gkg_partitioned` on Google BigQuery) | Daily average `V2Tone` (news tone) of articles tagged `TAX_DISEASE_PANDEMIC`, `UN_LOCKDOWN` or `WB_2028_PANDEMICS` | Google Cloud project |

**Study periods**

| Period | Dates |
|---|---|
| Pre-crisis | 2019-01-01 → 2020-02-29 |
| Crisis | 2020-03-01 → 2020-09-01 |

---

## Method

1. **Collection** – 610 daily BTC closes and 511 days of GDELT pandemic-news tone. Neither dataset had missing values.
2. **Integration** – Inner join on date (keeps only days present in both) then `dropna()`, giving **511 observations**: 326 pre-crisis and 185 crisis.
3. **Preprocessing (crisis period)** – Daily percentage change in BTC close (returns) and first difference of sentiment. The first row (NaN) is dropped, leaving **184 observations**.
4. **Stationarity check** – Augmented Dickey-Fuller test on both transformed series.
5. **Hypothesis tests**
   - RQ1: Welch's two-sample t-test on sentiment, pre-crisis vs crisis.
   - RQ2: Pearson correlation between BTC returns and sentiment change, plus a cross-correlation function over lags −10 to +10 days.

---

## Exploratory Data Analysis

### Dataset summary

| Dataset | Rows | Range | Missing |
|---|---|---|---|
| BTC close price | 610 | 2019-01-01 → 2020-09-01 | 0 |
| GDELT pandemic sentiment | 511 | 2019-01-02 → 2020-09-01 | 0 |
| Merged | 511 | – | 0 |
| Pre-crisis subset | 326 | Jan 2019 → Feb 2020 | – |
| Crisis subset | 185 | Mar 2020 → Sep 2020 | – |

BTC rose from about $3,880 (Jan 2019) to $11,927 (1 Sep 2020). GDELT tone is negative on almost every day, which is typical of disease and lockdown news.

### BTC price vs. sentiment during the crisis

![BTC price vs crisis sentiment](figures/btc_price_vs_sentiment.png)

BTC crashed to about $4,900 in mid-March 2020 ("Black Thursday") and then recovered steadily to above $11,500 by August. Sentiment moved sideways and became more erratic over the summer, with deep negative spikes in late June and early July. The two lines show no shared trend.

### Stationarity (ADF test, crisis period)

| Series | ADF statistic | p-value | Lags | Result |
|---|---|---|---|---|
| BTC daily returns | −5.383 | 0.000004 | 12 | Stationary |
| Sentiment change (differenced) | −7.342 | 1.06 × 10⁻¹⁰ | 8 | Stationary |

Both series are stationary, so their mean and variance are stable over time. Any correlation, or lack of one, is not an artefact of shared trends.

---

## Results

### RQ1: Public mood during the crisis

| Period | Mean sentiment |
|---|---|
| Pre-crisis | **−2.4961** |
| Crisis | **−3.3912** |

Welch's t-test: **t = 4.9399, p < 0.001**

The difference is statistically significant, but in the opposite direction from Hₐ: sentiment became more negative. **We fail to reject H₀ (Hₐ is not supported).** Public mood did not improve because of the COVID-19 crisis.

### RQ2: Bitcoin price vs. sentiment

![Returns vs sentiment change](figures/returns_vs_sentiment_change.png)

**Pearson r = −0.0429, p = 0.5635**

The scatter is a shapeless cloud centred on zero. The single extreme point near −38% is the 12 March 2020 crash. Sentiment changes have no measurable linear relationship with BTC returns.

![Cross-correlation](figures/cross_correlation.png)

Cross-correlations for lags −10 to +10 days all lie inside the 95% confidence band (±0.145). Sentiment does not predict BTC returns a few days later, and returns do not lead sentiment either.

**We fail to reject H₀ (Hₐ is not supported).** Daily sentiment had little bearing on Bitcoin's price changes.

---

## Conclusions

1. ADF tests confirmed both datasets were stationary and suitable for statistical analysis.
2. Public sentiment changed sharply during the crisis and became more pessimistic.
3. Bitcoin's price did not correlate with sentiment, either on the same day or with a lag.
4. Bitcoin acted as a **sentiment-resistant or semi-detached asset**, driven more by internal market activity than by the media's emotional climate.

## Limitations

- GDELT tone measures general pandemic news, not opinions about Bitcoin specifically. RQ1 therefore tests overall public mood, not mood toward Bitcoin.
- The pre-crisis period has gaps (326 of 425 days) because pandemic-tagged news was sparse before 2020. Means for that period rest on fewer and possibly less representative days.
- Pearson correlation captures linear, same-scale relationships only. Non-linear or volatility-level effects (e.g. sentiment vs. absolute returns) were not tested.
- One crisis window (Mar–Sep 2020) and daily frequency only.

---

## Repository Contents

| File | Description |
|---|---|
| `Bitcoin_and_public_sentiment.ipynb` | Full pipeline: data collection, integration, preprocessing, ADF tests, hypothesis tests and charts |
| `Blue and White Modern Data Analysis Presentation (1).pdf` | Final presentation slides |
| `figures/` | Charts exported from the notebook |

## Reproducing the Analysis

The notebook was built for **Google Colab**.

1. Open `Bitcoin_and_public_sentiment.ipynb` in Colab.
2. Set `GCP_PROJECT_ID` to a Google Cloud project with BigQuery enabled and `CRYPTO_API_KEY` to a CryptoCompare API key.
3. Run all cells. The Colab auth cell will ask you to sign in to Google.

Dependencies: `pandas`, `numpy`, `requests`, `matplotlib`, `google-cloud-bigquery`, `statsmodels`, `scipy`.

> The GDELT query scans a large partitioned table. Check your BigQuery quota before running it.

## References

1. Leetaru, K. H., & Schrodt, P. (2014, May 29). *The world's largest event dataset now publicly available in BigQuery* [Blog post]. Google Cloud Platform.
2. The GDELT Project. (n.d.). *Global Database of Events, Language, and Tone (GDELT)* [Dataset]. https://www.gdeltproject.org/data.html
3. CryptoCompare Ltd. (n.d.). *CryptoCompare API: Cryptocurrency market data* [Web API]. Retrieved October 18, 2025, from https://min-api.cryptocompare.com/documentation/
4. Corbet, S., Larkin, C., & Lucey, B. (2020). The contagion effects of the COVID-19 pandemic: Evidence from gold and cryptocurrencies. *Finance Research Letters, 35*, 101554. https://doi.org/10.1016/j.frl.2020.101554
