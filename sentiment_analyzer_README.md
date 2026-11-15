# Google Reviews Sentiment Analyzer

A fully offline, single-file HTML tool for analysing customer reviews. Paste any text reviews and get instant sentiment scores, charts, keyword extraction, and per-review breakdowns — no API key needed, no server, no installation.

---

## Getting Started

1. Download `sentiment_analyzer.html`
2. Open it in any modern browser
3. Paste your reviews into the text area (separate each review with a blank line)
4. Click **Analyze Sentiment**

No npm, no Node.js, no account required.

---

## How to Input Reviews

**From Google Maps (manual):**
1. Open the business on Google Maps
2. Scroll to the Reviews section
3. Copy review text
4. Paste into the text area — use a blank line between each review
5. Click **Analyze Sentiment**

**From Google Places API (optional):**
1. Enter the Google Place ID for the business
2. Enter your Google Places API key
3. Click **Fetch Reviews** — up to 5 reviews are fetched automatically

> Note: direct browser-to-Google API calls are blocked by CORS when running from a local file. The manual paste method always works; the API fetch works reliably when the page is served from a web server.

**Sample data:**
Click **Load Sample Reviews** to run the analyser immediately on 8 pre-loaded restaurant reviews.

---

## Output Sections

| Section | What it shows |
|---|---|
| Overall Sentiment Ring | Animated circular gauge displaying the compound score (−1 to +1) and overall label |
| KPI Tiles | Total reviews, % Positive, % Neutral, % Negative |
| Sentiment Distribution | Doughnut chart: Positive / Neutral / Negative counts |
| Per-Review Scores | Bar chart: individual compound score for each review, colour-coded |
| Top Positive Keywords | Most frequent positive words across all reviews with occurrence count |
| Top Negative Keywords | Most frequent negative words across all reviews with occurrence count |
| Individual Review Cards | Each review with its label badge, compound score, and a score progress bar |

---

## Sentiment Engine

The analyser uses a **VADER-style** algorithm — the same approach used in academic sentiment analysis — implemented entirely in JavaScript with no external dependencies.

### Step 1 — Lexicon lookup

Each word is matched against an AFINN-inspired lexicon of ~130 review-domain words, scored from −5 (most negative) to +5 (most positive).

| Score | Example words |
|---|---|
| +5 | excellent, outstanding, perfect, amazing, fantastic, phenomenal |
| +4 | great, awesome, best, love, terrific, fabulous, impressive |
| +3 | good, nice, happy, satisfied, friendly, helpful, delicious, professional |
| +2 | comfortable, convenient, affordable, beautiful, efficient, reliable |
| +1 | fair, decent, fine, okay, satisfactory, solid |
| −1 | average, mediocre, bland, forgettable, generic |
| −2 | disappointing, poor, overpriced, delayed, uncomfortable, underwhelming |
| −3 | bad, rude, unprofessional, dirty, frustrating, broken |
| −4 | terrible, awful, horrible, hate, worst, appalling, pathetic |
| −5 | fraud, scam, ripoff |

### Step 2 — Negation handling

A negation window of **3 words** is checked before each scored word. If a negation is found, the score is multiplied by **−0.74** (partial reversal, not full flip).

Negation words detected: `not, no, never, neither, cannot, can't, won't, don't, doesn't, didn't, isn't, aren't, wasn't, weren't, haven't, hasn't, hadn't, wouldn't, couldn't, shouldn't, without, barely, hardly, scarcely, lack, lacking`

### Step 3 — Intensifier scaling

A window of **2 words** before each scored word is checked for intensifiers. If found, the word's score is multiplied by the intensifier's weight.

| Multiplier | Words |
|---|---|
| ×1.5 | extremely, absolutely, utterly, exceptionally |
| ×1.4 | completely, incredibly |
| ×1.3 | very, super, seriously |
| ×1.2 | really, definitely, truly, particularly, especially, deeply |
| ×1.1 | quite, rather |
| ×0.8 | fairly, almost |
| ×0.7 | somewhat |
| ×0.6 | slightly |
| ×0.5 | barely |

### Step 4 — Compound score

All adjusted word scores are summed, then normalised using the VADER formula:

```
compound = sum / sqrt(sum² + 15)
```

This maps any sum to a value between **−1.0 and +1.0**.

### Classification thresholds

| Label | Compound score |
|---|---|
| Positive | ≥ 0.05 |
| Neutral | > −0.05 and < 0.05 |
| Negative | ≤ −0.05 |

### Overall score

The overall compound score shown in the ring gauge is the **arithmetic mean** of all individual review compound scores.

---

## Keyword Extraction

After scoring, all lexicon words found across all reviews are aggregated by polarity:

- **Positive keywords** — all words with a lexicon score > 0, sorted by total occurrence count
- **Negative keywords** — all words with a lexicon score < 0, sorted by total occurrence count

Up to 12 keywords are shown per category with their occurrence count.

---

## Charts

**Sentiment Distribution (doughnut)**
Counts of Positive, Neutral, and Negative reviews. Colour-coded green / amber / red.

**Per-Review Sentiment Scores (bar)**
One bar per review on the x-axis, compound score (−1 to +1) on the y-axis. Bars are coloured green (positive), amber (neutral), or red (negative).

---

## Tech Stack

| Technology | Purpose |
|---|---|
| HTML5 + CSS3 | Structure, layout, CSS custom properties, dark mode |
| Vanilla JavaScript (ES6+) | Sentiment engine, keyword extraction, chart rendering |
| [Chart.js 4.4](https://www.chartjs.org/) | Doughnut and bar charts — loaded from CDN |

No other dependencies. The CDN call for Chart.js is the only network request. All analysis runs locally in the browser.

---

## Browser Requirements

| Browser | Support |
|---|---|
| Chrome 90+ | Recommended |
| Edge 90+ | Recommended |
| Firefox 90+ | Supported |
| Safari 15+ | Supported |

---

## Limitations

- The lexicon covers ~130 words focused on the restaurant and service domain. Industry-specific jargon not in the lexicon scores as neutral.
- Sarcasm and irony are not detected (e.g. "Oh great, another hour wait" scores positive).
- Negation handling is window-based (3 words), not grammatical, so complex sentence structures may be misclassified.
- The Google Places API integration is subject to CORS restrictions when the file is opened locally. Manual paste always works.
- The Google Places API returns a maximum of 5 reviews per place per request.

---

## License

MIT — free to use, modify, and distribute.
