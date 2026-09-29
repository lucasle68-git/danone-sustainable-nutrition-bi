# Calculated Fields: Formulas, Weightings, Rationale

This page documents the Tableau calculated fields exactly as implemented in `dashboard/sustainable_nutrition_dashboard.twbx`. Weightings draw on WHO malnutrition frameworks, FAIRR ESG benchmarks and development-economics literature on the resource curse (Sachs & Warner, 2001).

**Missing values:** indicators are wrapped in `IFNULL(..., 0)` or `ZN()`, so a missing indicator contributes 0 to a composite. This keeps every country scoreable, but can understate need for countries with sparse survey data. Read the indices together with the completeness figures in [`data/data_dictionary.md`](../data/data_dictionary.md).

---

## 1. Malnutrition Severity Index

**Purpose:** one severity score from five nutrition indicators (all in %).

```
MSI = IFNULL([Prevalence of undernourishment], 0)            * 0.30
    + IFNULL([Prevalence of stunting, under 5], 0)           * 0.25
    + IFNULL([Prevalence of severe food insecurity], 0)      * 0.20
    + IFNULL([Prevalence of severe wasting, under 5], 0)     * 0.15
    + IFNULL([Prevalence of anaemia, women 15–49], 0)        * 0.10
```

**Scale:** a weighted average of percentages. Across the 35 countries it ranges from about 1 to 17 (higher = more severe).

**Rationale for weights:**
- Undernourishment (30%): primary SDG 2.1 target.
- Stunting (25%): irreversible developmental damage in under-5s.
- Severe food insecurity (20%): multidimensional access measure.
- Severe wasting (15%): acute malnutrition signal.
- Anaemia in women (10%): hidden hunger and intergenerational effects.

---

## 2. Dairy Consumption Gap

**Purpose:** market headroom for dairy, on a 0–100 scale (higher = bigger gap = more opportunity).

```
IF ISNULL([Milk Consumption (kg/capita/year)])      THEN 50   // neutral when unknown
ELSEIF [Milk Consumption] >= 200                    THEN 0
ELSEIF [Milk Consumption] >= 150                    THEN 25
ELSEIF [Milk Consumption] >= 100                    THEN 50
ELSEIF [Milk Consumption] >= 50                     THEN 75
ELSE 100
END
```

**Rationale:** banding makes the gap easy to explain to non-technical stakeholders and limits the influence of extreme values.

---

## 3. Resource Dependency Risk

**Purpose:** exposure to extractive-economy risk (higher = riskier). This is the risk measure used in the primary KPI.

```
RDR = IFNULL([Total natural resources rents (% of GDP)], 0)                  * 0.60
    + (100 − IFNULL([Adjusted net savings (% of GNI)], 50))                  * 0.40
```

**Rationale:**
- Resource rents (60%): dependence on extraction.
- Adjusted net savings (40%, inverted): whether an economy reinvests or depletes its wealth. A missing value is treated as a neutral 50.

---

## 4. Sustainability Risk Score

**Purpose:** a thresholded view of the same two drivers, used for the resource-sustainability dashboard.

```
SRS = ( ZN(AVG([Resource rents % GDP])) / 30 ) * 60
    + CASE adjusted net savings (average)
        <= 0%   → 40
        >= 20%  → 0
        between → ((20 − savings) / 20) * 40
```

**Rationale:** rents are scaled to a 30%-of-GDP threshold where resource-curse effects become severe; negative savings (an economy consuming its wealth) score the maximum 40.

---

## 5. Agricultural Productivity Index

**Purpose:** supply-chain readiness, on a 0–100 scale (higher = more productive).

```
API = IFNULL([Cereal yield], 0)
        / { MAX([Cereal yield]) } * 50
    + IFNULL([Agriculture value added per worker], 0)
        / { MAX([Agriculture value added per worker]) } * 50
```

Each component is scaled against the highest value in the dataset with a table-scoped LOD expression (`{ MAX(...) }`).

**Rationale:** equal weights, because high yield without processing capacity limits scale, and mechanisation without suitable climate limits production.

---

## 6. Sustainable Nutrition Opportunity Index (primary KPI)

**Purpose:** a single score for where Danone should look first.

```
SNOI = [Malnutrition Severity Index]             * 0.45
     + [Dairy Consumption Gap]                   * 0.30
     + (100 − [Resource Dependency Risk])        * 0.25
```

Computed per country-year and averaged over 2016–2020. Top 3: Tanzania 53.6, Senegal 52.9, Bangladesh 52.5.

**Rationale:**
- Malnutrition (45%): anchors selection in social need (SDG 2).
- Dairy gap (30%): market-growth headroom.
- Low resource dependency (25%): favours diversified economies (SDG 12).

Because the three inputs sit on different scales (the malnutrition index peaks around 17, the other two run to 100), the dairy gap and resource components drive most of the spread between countries. A future version would min–max normalise each input to 0–100 before weighting.

---

## 7. Quadrant Assignment (2×2 matrix)

```
IF   AVG(stunting) >= 25 AND AVG(resource rents) >= 6  THEN "Critical Opportunity"
ELSEIF AVG(stunting) >= 25                             THEN "Nutrition Focus"
ELSEIF AVG(resource rents) >= 2                        THEN "Sustainable Transition"
ELSE "Premium Markets"
END
```

**Rationale:** 25% stunting sits inside the WHO/UNICEF "high" prevalence band (20% to under 30%); the rent thresholds separate diversified economies (< 2%) from moderately (2–6%) and highly (≥ 6%) resource-dependent ones.

---

## Supporting fields

| Field | Purpose |
|---|---|
| Gender gap | Male minus female stunting rate (pp) |
| Milk consumption level | Bands consumption into Very High to Very Low |
| Production-Consumption Gap / Net Balance | Dairy surplus or deficit (1,000 tonnes) |
| Dairy Protein Gap from Daily Needs | Shortfall against a 10 g/day dairy-protein benchmark |
| Productivity Category | Bands the Agricultural Productivity Index |
| Dominant Resource Type | Largest resource-rent category per country (oil, gas, coal, mineral, forest) |

---

## Why weighted composites?

1. **Decision compression.** Executives cannot rank 33 indicators across 35 countries by eye; a composite forces explicit trade-offs.
2. **Auditable assumptions.** Weights are visible and can be challenged.
3. **SDG operationalisation.** Composites turn SDG goals into measurable screening criteria.

**Caveat:** any composite hides information, so every dashboard keeps drill-down to the underlying indicators.
