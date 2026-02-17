# Halifax Scotiabank Centre Artist Demand Model

## Purpose
Build a practical, explainable forecasting model for 10,000–12,000 seat arena configurations at Scotiabank Centre that outputs:
- **Demand Score (0–100)**
- **Confidence Rating (Low / Medium / High)**
- **Projected Ticket Range**
- **Risk Profile (Low / Medium / High)**
- **Key Demand Drivers**

---

## Step 1) Weighted Variable Framework

### Category weights (top-level)
Use category-level weighting first, then variable-level scoring inside each category.

| Category | Weight | Why this weight is rational in Halifax |
|---|---:|---|
| 1. Historical Market Data | **30%** | Local proof of demand is the strongest signal in a secondary market with unique regional behavior. |
| 2. Artist Momentum Indicators | **25%** | Momentum captures current relevance and demand velocity, especially for younger demos and digital-first discovery. |
| 3. Routing & Tour Context | **15%** | Atlantic routing friction and tour logic can materially raise or suppress actual conversion. |
| 4. Economic & Local Context | **10%** | Macro and local calendar effects influence walk-up and discount risk, but are less artist-specific. |
| 5. Audience Fit | **20%** | Halifax’s strong regional loyalty and format preferences make fit a major conversion driver. |

Total = 100%

### Variable design by category
Score each variable on a **0–100 normalized scale**.

#### 1) Historical Market Data (30%)
- **Prior Halifax sales for artist (or nearest comp set)** – 35% of category
- **Comparable genre performance in Halifax (last 24 months)** – 20%
- **Average sell-through rate (same capacity band)** – 20%
- **Days-to-sell curve / pace** – 15%
- **Realized price tier strength (gross per cap relative to target)** – 10%

#### 2) Artist Momentum Indicators (25%)
- **Spotify monthly listeners trend (not just level)** – 20%
- **Canadian streaming share / concentration** – 20%
- **Social follower growth rate (90-day)** – 15%
- **Recent sell-through in similar secondary markets** – 30%
- **Canada chart/radio chart presence (rolling 6 months)** – 15%

#### 3) Routing & Tour Context (15%)
- **East Coast routing efficiency (distance + load-in feasibility)** – 30%
- **Production scale fit for venue economics** – 20%
- **Competing dates in Atlantic Canada / Northeast corridor window** – 30%
- **Cannibalization risk from nearby major markets timing** – 20%

#### 4) Economic & Local Context (10%)
- **Seasonality index (weather + holiday periods)** – 30%
- **Same-week event competition index** – 25%
- **Consumer confidence / discretionary spend proxy** – 20%
- **University calendar alignment (students in/out of market)** – 25%

#### 5) Audience Fit (20%)
- **Demo alignment with Halifax ticket buyers (age/genre propensity)** – 35%
- **Crossover appeal beyond core fan base** – 20%
- **Atlantic Canada radio support / spins** – 25%
- **Brand familiarity / nostalgia strength in region** – 20%

---

## Step 2) Scoring Formula and Normalization

### 2.1 Variable normalization (0–100)
Use one of these methods per metric type:

1. **Min-max normalization** (for bounded historical ranges)

\[
N_i = 100 \times \frac{X_i - P10_i}{P90_i - P10_i}
\]

- Clip below 0 and above 100.
- Use **P10/P90** instead of min/max to reduce outlier distortion.

2. **Directional normalization** for “lower is better” variables (e.g., competing dates):

\[
N_i = 100 - \left(100 \times \frac{X_i - P10_i}{P90_i - P10_i}\right)
\]

3. **Binary or rule-based variables** (e.g., routing viability):
- 100 = excellent condition
- 70 = workable
- 40 = weak
- 10 = poor

### 2.2 Category score
For each category \(c\):

\[
CategoryScore_c = \sum_{i=1}^{k} (N_i \times w_{i|c})
\]

Where \(w_{i|c}\) sums to 1 within each category.

### 2.3 Final Demand Score (0–100)

\[
DemandScore = \sum_{c=1}^{5} (CategoryScore_c \times W_c)
\]

Where top-level weights are:
- Historical 0.30
- Momentum 0.25
- Routing 0.15
- Economic 0.10
- Audience Fit 0.20

### 2.4 Confidence Rating
Use **data completeness + signal agreement**:

- **Data Coverage %** = populated variables / total variables
- **Signal Variance** = standard deviation across category scores
- **Comparable Depth** = count of valid comparable shows

Rule set:
- **High Confidence**: Coverage \(\ge 85\%\), comparables \(\ge 8\), and category SD \(\le 15\)
- **Medium Confidence**: Coverage 65–84%, comparables 4–7, or SD 16–25
- **Low Confidence**: Coverage \(< 65\%\), comparables \(< 4\), or SD \(> 25\)

### 2.5 Ticket Range projection
Use capacity-constrained range:

\[
ProjectedTickets = Capacity \times (DemandScore / 100) \times AdjustmentFactor
\]

Where AdjustmentFactor is a confidence/risk modifier:
- High confidence, low risk: 0.98–1.05
- Medium confidence: 0.90–1.00
- Low confidence/high risk: 0.75–0.92

For 10,000–12,000 configurations, output a **range**:
- **Low case** = base projection \(\times 0.90\)
- **Expected case** = base projection
- **High case** = base projection \(\times 1.10\)

### 2.6 Risk Profile
Risk is based on downside drivers:
- High discount dependence
- Heavy weather exposure period
- Weak routing logic
- Volatile momentum metrics
- Low audience fit

Rule of thumb:
- **Low Risk**: Demand \(\ge 70\), confidence medium/high, no major red flags
- **Medium Risk**: Demand 55–69 or 1–2 major red flags
- **High Risk**: Demand \(<55\) or 3+ major red flags

### 2.7 Excel / Power BI implementation notes
- Keep one row per artist-date-market candidate.
- Columns:
  - Raw variables
  - Normalized variables
  - Variable weighted scores
  - Category totals
  - Demand score
  - Confidence, risk, ticket range
- Use parameter cells for all weights so you can tune without rewriting formulas.

---

## Step 3) Decision Bands and Booking Strategy

| Demand Score Band | Tier Label | Booking Implication |
|---|---|---|
| **85–100** | Arena Headliner Lock | Full arena configuration, premium pricing ladder, early onsale, heavier media burst to maximize yield rather than awareness. |
| **70–84** | Strong Play | Book arena with disciplined holdbacks, dynamic pricing, medium marketing ramp, and tight presale targeting. |
| **55–69** | Configurable Risk | Consider reduced-capacity configuration, stronger partner support, cautious guarantee structure, and milestone-based ad spend. |
| **40–54** | Theatre Play | Shift to smaller room or defer; only proceed with exceptional strategic rationale (e.g., brand relationship). |
| **<40** | Pass | Do not place in arena cycle; monitor for future momentum change. |

---

## Step 4) Predictive Refinement / Learning Loop

### 4.1 Post-show data ingestion
After each event, record:
- Final paid tickets
- Walk-up %
- Average ticket price and discounting depth
- Marketing spend by channel
- Weather severity index at onsale and show week
- Lead-time and pacing checkpoints (T-90/T-60/T-30/T-7)

### 4.2 Error measurement
For each show:

\[
ForecastError\% = \frac{ActualTickets - ForecastTickets}{ForecastTickets}
\]

Track by segment:
- Genre
- Day-of-week
- Season
- Routing archetype
- New vs repeat artist

### 4.3 Weight update method (practical)
Start with your expert weights above. Every quarter:
1. Regress actual sell-through on normalized variables.
2. Convert standardized coefficients into updated suggested weights.
3. Blend old and new weights:

\[
NewWeight = 0.7 \times OldWeight + 0.3 \times DataSuggestedWeight
\]

4. Apply guardrails:
- No category shifts > +/-5 points per quarter
- Keep total category weights = 100%
- Require minimum sample size before changing sub-weights

### 4.4 Continuous improvement operating cadence
- **Monthly**: calibration review (bias by genre, season, promoter type)
- **Quarterly**: formal re-weighting cycle
- **Bi-annually**: rebuild normalization bands (P10/P90) using latest 24-month window
- **Annually**: revisit band thresholds and risk policy based on realized margin outcomes

This turns the model from static scorecard to a **learning forecasting system**.

---

## Step 5) Sample Simulation (Illustrative)

Assume 11,000 sellable seats.

### A) Mid-tier country artist
- Historical: 72
- Momentum: 68
- Routing: 80
- Economic/local: 62
- Audience fit: 78

Weighted Demand Score:
\[
(72\times0.30)+(68\times0.25)+(80\times0.15)+(62\times0.10)+(78\times0.20)=72.4
\]

**Output**
- Demand Score: **72** (Strong Play)
- Confidence: **High** (strong comps in Atlantic + stable pace history)
- Ticket range: **7,200–8,800**
- Risk: **Low–Medium**
- Why: Country over-indexes in regional markets; routing works; momentum is healthy but not explosive.

### B) Viral hip hop act
- Historical: 48
- Momentum: 92
- Routing: 58
- Economic/local: 60
- Audience fit: 64

Weighted Demand Score:
\[
(48\times0.30)+(92\times0.25)+(58\times0.15)+(60\times0.10)+(64\times0.20)=64.9
\]

**Output**
- Demand Score: **65** (Configurable Risk)
- Confidence: **Medium** (limited local comp depth, high volatility)
- Ticket range: **5,800–7,800**
- Risk: **Medium–High**
- Why: Massive digital heat but weaker local proof and potentially fragile conversion if pricing is aggressive.

### C) Legacy rock act (declining streams, strong brand)
- Historical: 84
- Momentum: 45
- Routing: 74
- Economic/local: 66
- Audience fit: 82

Weighted Demand Score:
\[
(84\times0.30)+(45\times0.25)+(74\times0.15)+(66\times0.10)+(82\times0.20)=70.5
\]

**Output**
- Demand Score: **71** (Strong Play)
- Confidence: **High** (deep historical analogs)
- Ticket range: **7,000–8,600**
- Risk: **Medium**
- Why: Brand and nostalgia offset digital decline; pricing and on-sale timing are crucial.

---

## Practical deployment checklist
1. Build template in Excel/Power BI with parameterized weights.
2. Backfill 24–36 months of Halifax + similar-market history.
3. Run 10–20 recent shows through backtest.
4. Compare predicted vs actual and calibrate thresholds.
5. Use model in weekly pipeline meetings with a one-page artist viability brief.

This gives you a Halifax-specific internal demand engine with transparent assumptions, auditable outputs, and a built-in learning loop.
