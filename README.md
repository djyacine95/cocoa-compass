# Cocoa Compass: What Makes Craft Dark Chocolate Great?

## Authors

| Name | Role | Contact / GitHub |
|------|------|------------------|
| Yacine Djeddi | Sole author (data collection, storage, analysis, visualization, and interface) | yacine.djeddi@sjsu.edu / @djyacine95|

This is a solo project, so Yacine Djeddi wrote both the Data Collection and Storage Plan and the Data Analysis and Visualization Plan below.

---

## Project Description

This project analyzes 2,530 expert reviews of plain dark chocolate bars (2006–2021) from the Flavors of Cacao database to find out what separates a great bar from an average one. I examine how cocoa percentage, bean origin, manufacturer location, ingredients, and review year relate to the 1–5 rating, following the course workflow of Prepare → Analyze → Visualize → Predict → Evaluate Responsibly. Statistical tests (t-test, chi-square, ANOVA, correlation) quantify these relationships, and I also mine the tasting notes to see which flavors go with high ratings. The findings are told through five or more static visualizations and an interactive Plotly dashboard with a world map of bean origins and filters for cocoa percent, region, and year. Finally, I critique my work for bias, including a U.S.-heavy sample, subjective ratings, and inconsistent labels, with an optional model that predicts ratings as an extension.

---

## Project Outline / Plan

### Timeline

| Milestone | Deliverable | Due | Weight |
|-----------|-------------|-----|--------|
| 0 | Dataset selection & quality audit | Week 3 | 5% |
| 1 | Cleaned data & statistics | Week 6 | 10% |
| 2 | Static visualization story | Week 10 | 15% |
| 3 | Interactive dashboard prototype | Week 12 | 15% |
| Final | Complete report & 8-minute presentation | Week 16 | 25% |

### Repository Structure

```
.
├── README.md
├── LICENSE
├── .gitignore
├── requirements.txt
├── data/
│   ├── raw/                 # chocolate.csv, exactly as downloaded
│   └── processed/           # chocolate_clean.csv
├── docs/
│   ├── data_dictionary.md
│   ├── quality_audit.md
│   └── cleaning_log.md
├── notebooks/               # exploration and analysis notebooks
├── src/
│   ├── clean_data.py        # reproducible cleaning pipeline
│   ├── statistics.py        # descriptive + inferential analysis
│   └── visualizations.py    # static figure generation
├── figures/                 # PNG/PDF static visualizations
├── dashboard/
│   └── interactive_dashboard.html
└── report/                  # final report and presentation slides
```

---

### Interface Plan

The interface is an **interactive dashboard** that anyone, including people who have never written code, can open in a browser to explore what makes a bar of chocolate score well.

**Technology:** Plotly (Python) exported to a self-contained HTML file, so it can be shared, opened offline, and optionally deployed to GitHub Pages. Dash or Streamlit is the fallback if I need more advanced filtering.

**Layout (wireframe):**

```
+-------------------------------------------------------------------+
|  Cocoa Compass: Craft Dark Chocolate Explorer        [About Data] |
+------------------+------------------------------------------------+
|  FILTERS         |  KPI CARDS                                     |
|                  |  [ Bars ]  [ Avg Rating ]  [ Avg Cocoa % ]     |
|  Cocoa %  |-o--o-|------------------------------------------------+
|  Year     |-o--o-|  World map: average rating by bean origin      |
|  Maker location  |  (hover for country, # bars, avg rating)       |
|  Bean origin     |                                                |
|  Ingredients     +------------------------------------------------+
|  [ ] Has lecithin|  Cocoa % vs Rating scatter (hover: bar, maker, |
|  [ ] Has vanilla |  notes)           |  Top flavor notes (bar)    |
|                  |                   |                            |
|  [Reset]         |                   |                            |
+------------------+-------------------+----------------------------+
```

**Planned interactions:**
- Range sliders for cocoa percentage and review year; multi-select dropdowns for maker location and bean origin; checkboxes for ingredients (e.g., lecithin, vanilla)
- Hover tooltips showing bar name, maker, origin, cocoa %, rating, and tasting notes
- Zoom and pan on the scatter plot and map; click legend items to show or hide groups
- KPI cards, map, scatter, and flavor-note chart that all update together when a filter changes
- A "Reset filters" control and a short "About the data" panel explaining the 1–5 rating scale

**Design and accessibility principles:**
- Colorblind-friendly palette (seaborn `colorblind` / ColorBrewer); the map uses a single-hue sequential scale, and color is never the only way to tell groups apart
- Clear titles, axis labels, and units on every chart; high-contrast text
- Responsive layout that remains usable on a phone or tablet
- Keyboard-reachable controls and descriptive alt text for exported images

**Usability testing:** I will have at least three peers try the prototype for Milestone 3, record their feedback, and keep a short revision log of what I changed and why.

---

### Data Collection and Storage Plan
*(Written by Yacine Djeddi)*

**Data source.** I use the Chocolate Bar Ratings dataset, part of the TidyTuesday project (week 3, 2022-01-18):
- Dataset page: <https://github.com/rfordatascience/tidytuesday/tree/master/data/2022/2022-01-18>
- Direct CSV: <https://raw.githubusercontent.com/rfordatascience/tidytuesday/master/data/2022/2022-01-18/chocolate.csv>

The underlying ratings come from *Flavors of Cacao*, compiled by Brady Brelinski of the Manhattan Chocolate Society. The database focuses on plain dark chocolate, and the ratings reflect flavor only, not health benefits, social missions, or organic status. The CSV is downloaded once and stored unmodified in `data/raw/`; it is small enough to commit directly. I will cite the original source in my report.

**Data dictionary.** All 10 columns are documented in `docs/data_dictionary.md`:

| Column | Description |
|--------|-------------|
| ref | Reference ID; the highest numbers were the last entries made (not unique per bar) |
| company_manufacturer | Manufacturer name |
| company_location | Manufacturer location |
| review_date | Year the bar was reviewed |
| country_of_bean_origin | Country (or region) where the beans were grown; "Blend" for multiple origins |
| specific_bean_origin_or_bar_name | Specific bean or bar name |
| cocoa_percent | Cocoa percentage, stored as text (e.g., "70%") |
| ingredients | Number of ingredients plus codes: B = Beans, S = Sugar, S* = other sweetener, C = Cocoa Butter, V = Vanilla, L = Lecithin, Sa = Salt |
| most_memorable_characteristics | Comma-separated tasting notes (texture, flavor, overall impression) |
| rating | Expert rating on a 1–5 scale |

**Initial quality audit (Milestone 0).** Results from my first pass over the raw file, to be written up in `docs/quality_audit.md`:

- **Size:** 2,530 rows × 10 columns
- **Data types:** 3 numeric (`ref`, `review_date`, `rating`) and 7 text columns
- **Missing values:** only `ingredients` has gaps, with 87 missing (3.4%); every other column is complete
- **Duplicates:** 0 fully duplicated rows
- **Coverage:** 580 manufacturers, 67 manufacturer locations, 62 bean-origin labels, reviews from 2006 to 2021
- **Obvious problems:**
  - `cocoa_percent` is text with a "%" sign (range 42%–100%)
  - `ref` repeats (up to 10 bars share one value), so it cannot serve as a row key
  - `ingredients` formatting is inconsistent (e.g., `"5- B,S,C,V,L"` vs `"5-B,S,C,V,Sa"`)
  - 156 bars (6.2%) have the origin "Blend", which is not a country
  - Overlapping or non-country labels: "Sao Tome" vs "Sao Tome & Principe", "Congo" vs "DR Congo", regions such as Sumatra and Sulawesi among origins, and "Amsterdam", "Scotland", and "Wales" among manufacturer locations
  - Ratings take only 12 discrete values between 1.0 and 4.0 (none above 4.0, despite the 1–5 scale)
  - The sample is U.S.-heavy: 44.9% of bars come from U.S. manufacturers

**Cleaning plan (Milestone 1).** Every decision is logged in `docs/cleaning_log.md` with a justification:

1. Convert `cocoa_percent` to a numeric `cocoa_pct` column so it can be analyzed.
2. Add my own unique `row_id`, since `ref` is not unique.
3. Parse `ingredients` into `n_ingredients` plus yes/no columns for each ingredient code, using a pattern that tolerates the spacing differences.
4. Keep the 87 rows with missing `ingredients` and mark them "Unknown"; exclude them only from ingredient-specific analyses rather than discarding otherwise good reviews.
5. Standardize place names (merge "Sao Tome" variants, map Amsterdam → Netherlands, Scotland/Wales → U.K.) and add a clean `country` column so the world map works; keep the original columns untouched.
6. Keep "Blend" as its own category but exclude it from country-level comparisons and the map.
7. Split `most_memorable_characteristics` into individual lower-cased, trimmed tasting terms for flavor analysis.
8. Keep extreme values (e.g., 21 bars at 100% cocoa, ratings as low as 1.0) as legitimate data, but flag them.
9. Validate the output: correct data types, expected value ranges, and no unexpected missing values.

**Storage and reproducibility.**
- `src/clean_data.py` reads `data/raw/chocolate.csv` and writes `data/processed/chocolate_clean.csv`, so the cleaned file can always be regenerated from the raw file.
- Raw data is read-only; all changes happen in code, never by hand in a spreadsheet.
- Dependencies are pinned in `requirements.txt`, and random seeds are fixed for any sampling or modeling.
- Work is done on feature branches and merged through pull requests with clear commit messages.

**Ethical note.** The dataset contains the names of real small businesses and craft makers. I analyze patterns across groups and avoid presenting any single company's ratings as a verdict on its quality.

---

### Data Analysis and Visualization Plan
*(Written by Yacine Djeddi)*

**Descriptive statistics.** For `rating`, `cocoa_pct`, and `review_date` I will report mean, median, standard deviation, min, max, and quartiles. I will also report counts and average ratings by bean origin, manufacturer location, ingredient count, and review year.

**Inferential analysis (at least one test required; I plan several):**

| Question | Test |
|----------|------|
| Does cocoa percentage go up or down with rating? | Pearson and Spearman correlation (Spearman because ratings are discrete) |
| Are bars made in the same country where the beans were grown (317 bars) rated differently from all others (2,213)? | Independent-samples Welch's t-test |
| Is the share of high-rated bars (rating ≥ 3.5) different across cocoa bands (<70%, 70–74%, 75%+)? | Chi-square test of independence, with Cramér's V as effect size |
| Do average ratings differ across the 22 bean origins with at least 30 reviews? | One-way ANOVA with Tukey post-hoc, plus Kruskal-Wallis as a non-parametric check |

For each test I will state the hypotheses, check assumptions, report the p-value and an effect size, and interpret the result in plain language, including whether a statistically significant result is large enough to matter in practice.

**Static visualizations (Milestone 2, at least five):**

| # | Chart | Purpose / Rationale |
|---|-------|---------------------|
| 1 | Histogram of ratings | Shows the overall shape; bins aligned to the 12 discrete values |
| 2 | Scatter plot with trend line: cocoa % vs rating | Shows the relationship between two numeric variables, with jitter to handle overlapping points |
| 3 | Horizontal bar chart: average rating by bean origin (≥ 30 reviews), with confidence intervals | Easy comparison across categories, sorted for readability |
| 4 | Box plot: rating by cocoa band or ingredient count | Shows spread, median, and outliers for each group |
| 5 | Line chart: average rating and number of reviews by year | Shows change over time and flags years with few reviews |
| 6 | Bar chart: most frequent tasting notes among top-rated vs lowest-rated bars | Connects the numbers to flavor |

I may add a world map of average rating by bean origin as an advanced visualization for bonus points. Charts will be arranged as a narrative (the overall picture → what predicts a good bar → what a good bar tastes like) and exported at 300 dpi to `figures/`.

**Accessibility check.** I will use a colorblind-safe palette, verify the charts with a colorblind simulator, add direct labels or markers so color is not the only encoding, and keep text large and high-contrast.

**Bias critique.** For each chart I will note what might mislead the viewer, for example:
- Ratings come from one reviewing organization, so they reflect one panel's taste and not consumers in general
- The database covers plain dark chocolate only, so conclusions do not apply to milk chocolate or flavored bars
- U.S. manufacturers make up about 45% of the sample, and a few origins dominate, so averages for small groups are unreliable
- Review counts vary a lot by year, which makes year-to-year trends noisy
- Bar charts of averages can hide the spread, and truncated axes can exaggerate small differences
- Ratings are discrete, so histogram bin choices can change the apparent pattern
- Correlation is not causation; cocoa percentage, ingredients, origin, and maker are tangled together
- Inconsistent place names and the "Blend" category affect map and country-level results

**Optional ML extension (bonus).** If time allows, I will predict whether a bar is high-rated (≥ 3.5) from cocoa percentage, ingredients, bean origin, and manufacturer location using logistic regression and a Random Forest with a stratified train/test split and a fixed random seed. I will report accuracy, precision, recall, and a confusion matrix against a simple "always predict the majority class" baseline, show feature importance, and check performance across subgroups (e.g., U.S. vs non-U.S. makers) to see whether the model works equally well for all of them.

**Final report and presentation (Week 16).** The report covers the executive summary, data preparation, EDA findings, visualization story, dashboard description, bias and ethical considerations, and conclusions. The 8-minute presentation covers what I did, what I found, what I recommend, and what I learned.

---

## Getting Started

```bash
git clone <https://github.com/djyacine95/cocoa-compass>
cd <cocoa-compass>
python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate
pip install -r requirements.txt
python src/clean_data.py
```

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details. The dataset belongs to its original creators (see Data Source above) and is not covered by this license.
