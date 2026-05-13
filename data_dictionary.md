# Data Dictionary

Reference for the two datasets powering the dashboard, both stored in `/data/processed/`:

1. **`Danone_SDG_Data_CLEAN.xlsx`** — SDG indicators + FAO dairy metrics
2. **`GDP_Data_Cleaned.xlsx`** — GDP growth, GDP per capita, tax revenue

The two are joined inside Tableau using a relationship model — see *Join logic* below.

For per-column data types, non-null counts, and sample values, see the **`Data Dictionary`** sheet inside `Danone_SDG_Data_CLEAN.xlsx`.

---

## At a glance

| | File 1: SDG & Dairy | File 2: GDP & Macro |
| --- | --- | --- |
| Filename | `Danone_SDG_Data_CLEAN.xlsx` | `GDP_Data_Cleaned.xlsx` |
| Sheets | 5 (Main Data + 4 metadata sheets) | 1 |
| Rows × Cols | 180 × 36 | 175 × 6 |
| Granularity | One row per (Country × Year) | One row per (Country × Year) |
| Country coverage | 35 | 35 (same set) |
| Year coverage | 2016 – 2020 (Egypt: 2011 – 2020) | 2016 – 2020 |
| Source(s) | World Bank SDG Database; FAO Food Balances | World Bank Open Data (macro indicators) |

Row-count difference (180 vs 175): Egypt has 10 years of SDG data in File 1 but only 5 years in File 2, so 5 extra Egypt rows in File 1 are unmatched on the macro side. These rows are retained via outer-join logic in Tableau rather than dropped.

---

## Join logic

In Tableau's data model, the two files are connected via a **relationship** (not a pre-merged inner join) on three keys:

```
Country Name = Country Name
Country Code = Country Code
Year         = Year
```

Three-key joining defends against country-name collisions (e.g., "Korea, Rep." vs "Korea") and verifies ISO3 alignment.

**Why a Tableau relationship and not a pre-merge in Python/Excel?** A pre-merged inner join would have dropped countries with partial coverage in either file, shrinking the sample from 35 countries to roughly 25. The relationship model preserves every country and lets each visual decide which metrics it needs.

---

## File 1 — `Danone_SDG_Data_CLEAN.xlsx`

### Sheets

| Sheet | Purpose | Rows × Cols |
| --- | --- | --- |
| **Main Data** | The integrated country-year fact table used by the dashboard. | 180 × 36 |
| **SDG Classification** | Maps each indicator to its UN SDG, thematic category, source, and unit. | 33 × 5 |
| **Country Metadata** | Region, income-level, and ISO3 code lookup for the 35 countries. | 35 × 5 |
| **Key Indicators 2020** | Pre-filtered 2020 snapshot of six headline metrics — convenience view. | 36 × 8 |
| **Data Dictionary** | Per-column metadata: data type, non-null count, completeness %, sample values. | 36 × 5 |

### Column groups in `Main Data`

**Identifiers (3 cols).** `Country Name`, `Country Code` (ISO3), `Year`.

**SDG 2 — Zero Hunger (22 cols).**
- *Nutrition & Food Security* (18 cols) — undernourishment, food insecurity (moderate/severe), child stunting/wasting/underweight/overweight (each disaggregated by sex), anaemia in women of reproductive age, exclusive breastfeeding, plus four FAO milk metrics (Production, Food Supply, Consumption per capita, Protein Supply per capita per day).
- *Agricultural Production* (2 cols) — cereal yield (kg/hectare), agriculture value added per worker (constant 2010 US$).

**SDG 12 — Responsible Consumption & Production (8 cols).**
- *Natural Resources* (7 cols) — total natural resource rents (% of GDP), and disaggregated rents: oil, gas, coal, mineral, forest.
- *Macro sustainability* (1 col) — adjusted net savings excluding particulate emission damage (% of GNI).

> The full indicator-to-SDG mapping with units is in the **`SDG Classification`** sheet of the workbook.

---

## File 2 — `GDP_Data_Cleaned.xlsx`

Single sheet, 6 columns.

| Column | Type | Completeness | Notes |
| --- | --- | --- | --- |
| `Country Name` | text | 100% | Harmonised to match File 1. |
| `Country Code` | text | 100% | ISO 3166-1 alpha-3. |
| `Year` | int | 100% | 2016 – 2020. |
| `GDP growth (annual %)` | float | 100% | World Bank indicator `NY.GDP.MKTP.KD.ZG`. |
| `GDP per capita` | float | 100% | Current US$ per person; World Bank indicator `NY.GDP.PCAP.CD`. |
| `Tax revenue (% of GDP)` | float | 82.3% | World Bank indicator `GC.TAX.TOTL.GD.ZS` — gaps are countries that don't publish consolidated tax revenue annually. |

---

## Countries (35)

Both files cover the same 35-country set, grouped by region:

| Region | Countries |
| --- | --- |
| Europe | France, Germany, Italy, Netherlands, Sweden, United Kingdom |
| Sub-Saharan Africa | Congo (DRC), Ethiopia, Nigeria, Senegal, South Africa, Tanzania |
| Southeast Asia | Indonesia, Malaysia, Philippines, Thailand, Vietnam |
| Middle East & North Africa | Egypt, Morocco, Saudi Arabia, Türkiye, United Arab Emirates |
| Latin America | Brazil, Colombia, Mexico, Peru |
| East Asia | China, Japan, Korea (Rep.) |
| South Asia | Bangladesh, India, Pakistan |
| North America | Canada, United States |
| Oceania | Australia |

Income mix: 16 High-Income, 9 Upper-Middle, 8 Lower-Middle, 4 Low-Income (World Bank classification).

---

## Data completeness — honest accounting

Completeness varies meaningfully across columns. Worth knowing before any interpretation:

| Indicator group (file) | Typical completeness | Driver |
| --- | --- | --- |
| Identifiers, Year (both files) | 97 – 100% | One country missing ISO3 in File 1. |
| GDP growth, GDP per capita (File 2) | 100% | Annual, well-reported. |
| Tax revenue (File 2) | 82% | Not every country publishes consolidated tax revenue every year. |
| Natural resource rents (File 1, SDG 12) | 96 – 97% | Annual World Bank reporting. |
| FAO milk metrics (File 1) | 92% | Annual, FAOSTAT. |
| Adjusted net savings (File 1) | 92% | Annual, World Bank. |
| Anaemia in women (File 1) | 76% | Modelled, available most years. |
| Undernourishment (File 1) | 78% | Annual but with lags for some countries. |
| Food insecurity, moderate/severe (File 1) | 55 – 56% | FAO Voices of the Hungry survey, biennial. |
| **Child anthropometrics** — stunting, wasting, underweight, overweight (incl. sex disaggregations) | **19 – 24%** | DHS/MICS surveys, not run annually per country. |
| Cereal yield (File 1) | 58% | Annual but missing for newer years in several countries. |
| Agriculture value added per worker (File 1) | 18% | Sparse reporting. |
| Exclusive breastfeeding (File 1) | 9% | Survey-driven, rarely repeated. |

**Overall completeness for File 1 ≈ 55.8%. File 2 ≈ 97% on GDP metrics, 82% on tax revenue.**

Missing values were **deliberately not imputed.** The gaps are systematic — countries with the weakest statistical infrastructure also tend to have the worst nutrition outcomes, so imputation would mask exactly the cases most relevant to the analysis. The dashboard is positioned as a **strategic screening tool**, not an operational planning system.

---

## Data sources

| Source | URL | Contributes to |
| --- | --- | --- |
| World Bank — Sustainable Development Goals Database | <https://databank.worldbank.org/source/sustainable-development-goals-(sdgs)> | 29 indicators in File 1 (nutrition, agriculture, resource sustainability). |
| FAO — Food Balances Statistics | <https://www.fao.org/faostat/en/#data/FBS> | 4 dairy indicators in File 1 (Production, Food Supply, Consumption per capita, Protein Supply per capita per day). |
| World Bank Open Data — World Development Indicators | <https://databank.worldbank.org/source/world-development-indicators> | All 3 macro indicators in File 2 (GDP growth, GDP per capita, Tax revenue). |

All sources are open and reproducible.

---

## How these datasets are used in the dashboard

| Dashboard page | File 1 columns used | File 2 columns used |
| --- | --- | --- |
| 1 · Executive Summary | All 5 composite indices (computed as Tableau calculated fields). | — |
| 2 · Nutrition Security Crisis | Undernourishment, stunting (incl. sex disaggregation), food insecurity, anaemia. | GDP per capita (for the stunting-vs-GDP scatter). |
| 3 · Dairy & Protein Gaps | Milk Consumption, Milk Production, Milk Protein Supply, Stunting. | — |
| 4 · Agricultural Capacity | Cereal yield, Agriculture value added per worker, Stunting. | — |
| 5 · Resource Sustainability | All resource-rent columns, Adjusted net savings, Milk Food Supply. | — |
| 6 · Strategic Connection | Composite indices + Total resource rents. | GDP per capita (for tier context in the Priority Ranking Table). |

The five composite indices built on top of File 1 (Malnutrition Severity Index, Sustainable Nutrition Opportunity Index, etc.) are documented in [`/docs/calculated_fields.md`](../docs/calculated_fields.md).
