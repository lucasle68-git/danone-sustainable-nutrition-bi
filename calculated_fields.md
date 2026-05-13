# Calculated Fields — Formulas, Weightings, Rationale

All indices are normalised to a **0–100 scale** for cross-country comparability. Weightings are not arbitrary, each is grounded in WHO malnutrition frameworks, FAIRR ESG benchmarks, or established development-economics literature (Sachs & Warner 2001 on the resource curse, etc.).

---

## 1. Malnutrition Severity Index

**Purpose:** Aggregate a single nutrition-crisis severity score from five WHO indicators.

**Formula (conceptual):**
```
MSI = 0.30 × Undernourishment_norm
    + 0.25 × Stunting_norm
    + 0.20 × SevereFoodInsecurity_norm
    + 0.15 × SevereWasting_norm
    + 0.10 × Anemia_Women_norm
```

**Rationale:**
- **Undernourishment (30%)**: primary SDG 2.1 target; weighted highest.
- **Stunting (25%)**: proxy for irreversible developmental damage in under-5s.
- **Severe food insecurity (20%)**: multidimensional access measure.
- **Severe wasting (15%)**: acute malnutrition signal.
- **Anemia in women (10%)**: hidden hunger and intergenerational transmission.

---

## 2. Resource Dependency Risk Score

**Purpose:** Identify countries whose economies are structurally vulnerable to commodity shocks.

**Formula (conceptual):**
```
RDR = 0.60 × min(ResourceRents_pct_GDP / 30, 1) × 100
    + 0.40 × (1 − clamp(NetSavings_pct, 0, 0.20) / 0.20) × 100
```

**Rationale:**
- **Resource rents (60%)**, extraction dependence; normalised to a 30%-of-GDP threshold where Sachs–Warner "resource curse" effects become severe.
- **Adjusted net savings (40%, inverted)**, captures capital depletion. Negative savings (countries consuming wealth) score maximum risk; ≥20% (adequate reinvestment) scores zero.

---

## 3. Agricultural Productivity Index

**Purpose:** Assess supply-chain readiness for dairy distribution and processing.

**Formula (conceptual):**
```
API = 0.50 × CerealYield_normalised
    + 0.50 × AgValueAddedPerWorker_normalised
```

**Rationale:**
- **Cereal yield (50%)**: proxies climate suitability and irrigation infrastructure.
- **Value added per worker (50%)**: proxies mechanisation and cold-chain capacity.
- Equal weighting: high yield without processing capacity limits scalability; high mechanisation in unsuitable climates faces production constraints.

---

## 4. Sustainability Risk Score

**Purpose:** Identify economies whose growth model is structurally unsustainable (SDG 12.2 alignment).

**Formula (conceptual):** identical structure to *Resource Dependency Risk* but interpreted at the macroeconomic-strategy level rather than the supply-chain-risk level.

**Rationale:** Higher scores indicate vulnerability to commodity shocks and inadequate economic diversification, both red flags for long-term partnership viability.

---

## 5. Sustainable Nutrition Opportunity Index *(primary KPI)*

**Purpose:** The composite the entire dashboard is built around, a single number expressing where Danone should look first.

**Formula (conceptual):**
```
SNOI = 0.45 × MalnutritionSeverityIndex
     + 0.30 × DairyConsumptionGap_normalised
     + 0.25 × (100 − SustainabilityRiskScore)
```

**Rationale:**
- **Malnutrition Severity (45%)**: anchors selection in social need, aligned to Danone's mission and SDG 2.
- **Dairy Consumption Gap (30%)**: market-growth headroom.
- **Sustainability (25%, inverted risk)**: favours diversified economies, ensuring SDG 12 alignment.

A high score = a market where nutritional intervention delivers maximum social impact within commercially and environmentally sustainable boundaries.

---

## Why Weighted Composites Over Raw Indicators?

Three reasons:

1. **Decision compression.** Executives cannot rank 33 indicators across 35 countries by inspection. A composite forces explicit trade-off articulation.
2. **Auditable assumptions.** Weights are visible and challengeable, better than implicit ranking heuristics.
3. **SDG operationalisation.** SDGs are aspirations until they're metrics. Weighted indices translate framework into action.

**Caveat:** any composite hides information. The dashboard always preserves drill-down to the underlying indicators so that the composite is a *summary view*, not a black box.
