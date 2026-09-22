# Stack Overflow Developer Survey 2025 — Analysis & Dashboard

## Overview

This project explores the Stack Overflow Developer Survey 2025 from two angles:

1. **A descriptive dashboard** of respondent demographics, employment status, geography, and programming language usage.
2. **A missingness analysis** examining *who leaves survey questions blank, and how those patterns of non-response relate to one another* — rather than only analyzing the answers respondents gave.

The second part was the more interesting question to me: most analyses of this dataset treat missing values as noise to be cleaned away. Here, missingness itself is treated as the subject of analysis — using a NaN indicator matrix and hierarchical clustering to find groups of questions that tend to go unanswered together.

## Repository structure

```
├── README.md
├── data/
│   ├── results.txt          # raw survey responses (not included - see note below)
│   └── schema.txt           # question text and metadata for each survey column
├── so_survey2025.ipynb      # cleaning, before/after comparison, CSV exports for the main dashboard
├── so_survey2025_nans.ipynb # missingness clustering analysis
└── so_survey2025_dash.pbix  # Power BI dashboard (2 pages)
```

**Note:** the raw dataset is not included in this repository due to its size. It's the publicly available Stack Overflow Developer Survey 2025 results file, downloadable directly from Stack Overflow's survey results page.

## Part 1 — Main dashboard (`so_survey2025.ipynb`)

A focused subset of core respondent attributes (age, country, education, employment, years coding, languages used) is cleaned and prepared for Power BI. Before dropping incomplete rows, the distributions of each variable are compared before and after removal — this checks whether listwise deletion introduces bias into the remaining sample. No substantial distributional shift was found across the variables checked.

Multi-select fields (`LanguageHaveWorkedWith`, `Employment`) are reshaped from semicolon-delimited strings into long-format bridge tables, allowing them to be related properly to the main respondent table in Power BI rather than handled as unstructured text.

**Dashboard highlights:**
- Respondent geography (choropleth map + top 15 countries)
- Age distribution
- Employment status breakdown *(respondents may report more than one status — see note on dashboard)*
- Most commonly used programming languages

## Part 2 — Missingness analysis (`so_survey2025_nans.ipynb`)

Rather than analyzing survey answers directly, this notebook analyzes the pattern of missing answers across the full 100+ column survey.

**Method:**
1. Columns with very high missingness driven by survey branching logic (e.g. questions only shown to respondents who use AI tools, or only to employed respondents) were identified and excluded where they would otherwise produce trivial or misleading clusters.
2. A binary "is this value missing?" indicator matrix was built for the remaining columns.
3. **Jaccard distance** was used to measure similarity between columns based on their missingness patterns — chosen over simple correlation because it directly measures *co-occurrence of missingness* (how often two columns are missing together, relative to how often either is missing at all), which better suits binary indicator data than a linear correlation coefficient.
4. Hierarchical clustering (average linkage) was applied to the resulting distance matrix, visualized as a dendrogram, and cut into 11 clusters.

**Key findings:**
- Missingness clusters largely reflect the survey's branching structure — e.g. all AI-related follow-up questions cluster tightly together, since they're only shown to respondents who indicated they use AI tools in the first place.
- The "Core Demographics" cluster is broader than its name suggests — it groups together columns that are *rarely* missing in general (including Stack Overflow usage questions like `SOAccount` and `SOVisitFreq`), not only demographic fields. This is a property of the Jaccard-based approach: columns with similarly low missingness rates cluster together even when thematically unrelated.
- The "Job & Comp" cluster notably contains the columns with the highest overall missingness rates in the dataset (`ProfessionalTech`, `ProfessionalCloud`, `ProfessionalQuestion`, `TimeAnswering` among the top 5).
- The top Jaccard-similarity pairs (e.g. `TimeAnswering`/`TimeSearching`, `WorkExp`/`ICorPM`) are near-perfectly linked largely because they share the same survey branching logic (shown only to employed respondents), rather than reflecting a specific behavioral choice to skip both.

**A limitation worth stating plainly:** this analysis identifies *association*, not *cause*. A cluster of jointly-missing columns tells us those questions tend to go unanswered together — it doesn't tell us why (survey fatigue, branching logic, or a genuine reluctance to answer) without further investigation into each specific case.

## Requirements

- Python 3, with `pandas`, `numpy`, `matplotlib`, `missingno`, `scipy`
- Power BI Desktop (to open the `.pbix` file)

## Viewing the dashboard

Open `so_survey2025_dash.pbix` in Power BI Desktop. The file contains two report pages: the main respondent overview, and the missingness cluster analysis.
