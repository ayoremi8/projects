# Modeling Core Customer Activation for a Photos App (Synthetic Data)

This repo documents an end-to-end workflow for thinking about **Core customer activation** and a **short-form video strategy** for a cloud photo product similar to Amazon Photos.

We deliberately use **synthetic data** that mimics the structure of real logs, so the logic is shareable.

---

## Notebook index

1. **Part A – Core activation setup & synthetic data**

   *Colab:*  
   [Modeling Core Customer Activation – Part A](https://colab.research.google.com/drive/18p5muzKLaigfUEKSWUgrqeMj5tDBk6aK#scrollTo=vdipBatXjW9x)

   - Define the **business context** and **Core_90d** target.
   - Specify the `photos_first_login_features` schema (one row per customer).
   - Simulate a realistic synthetic dataset.

2. **Part B – Core activation modeling & insights**

   *Colab:*  
   [Modeling Core Customer Activation – Part B](https://colab.research.google.com/drive/1mCBabpIj7njjK5UlbnLX9c1ErYTqY_Z3#scrollTo=JD1bbqfVpSzd)

   - Train baseline models (logistic, random forest) to predict `is_core_90d`.
   - Identify acquisition + early-usage drivers of Core activation.
   - Frame the video product decision:
     > “Integrated short-form video feed vs standalone video app?”

3. **Part C – Video strategy simulation (integrated vs standalone)**

   *Colab:*  
   [Video Strategy Simulation – Part C](https://colab.research.google.com/drive/1HI7hT58J0tFLY7F9JwlacAwloVw7Vvw2#scrollTo=MT-bM5NtAxKs)

   - Define priors for video adoption & impact per customer segment.
   - Simulate integrated vs standalone scenarios on a 24M active base.
   - Run sensitivity analysis over the standalone priors.

The rest of this document is the narrative that ties these notebooks together.

---

## 1. Business context

We model a **cloud photo storage product** similar to Amazon Photos, integrated with:

- **1P clients:** Android, iOS, Desktop, PhotosWeb  
- **2P surfaces:** Echo Show, Fire TV, Alexa

Core business logic:

> If we **delight customers**, we get **retention**, which drives **profit**.

Practically, that means turning new signups into **Core** customers who:

- have uploaded photo/video content, and  
- regularly launch our apps on key platforms.

Because true production logs are confidential, Part A works with a **synthetic dataset** that has:

- the same entity (one row per customer),
- similar distributions and correlations,
- realistic channel/platform definitions and behavior.

---

## 2. Modeling goal: Core_90d activation

**Entity:**  
`customer_id` (one row per customer).

**Start time:**  
`first_login_ts` – the first-ever interaction that counts as Photos usage:

- For mobile: first app launch or upload.  
- For Web: app launch on a “Your Photos” page.  
- Platform-specific nuances mirror the real system.

**Target:** `is_core_90d`

- `1` if the customer meets the **Core** definition within 90 days of first login.  
- `0` otherwise.

The synthetic table `photos_first_login_features` includes:

- **Attribution**
  - `attrib_channel_type` (Marketing, Alexa, Sharing, Clickstream, BizDev, Other, etc.)
  - `attrib_client_tier` (1P / 2P / 3P / Other)
- **Entry platform**
  - `entry_client` (Android, iOS, Desktop, PhotosWeb)
- **Lead types (any true/false flags)**
  - Group share, wakescreen, prints, Snapfish, retail prints, etc.
- **Early 7-day behavior**
  - Upload events, app launches, photo views, notifications opened,
  - distinct platforms used.
- **Outcomes / states**
  - Acquired, Engaged, Core, churn proxies, revenue hooks.

> See **Part A** for the full schema and synthetic generation workflow.

---

## 3. Baseline modeling of Core_90d (Part B)

Before thinking about video, we want to understand **what currently drives Core activation**.

### 3.1 Models

We train two baseline models on the synthetic dataset:

1. **Logistic regression**

   - Features: attribution channel, 1P/2P tier, entry platform, lead flags, early 7-day counts.
   - Output: calibrated probabilities for `is_core_90d`.

2. **Random Forest**

   - Same feature set, non-linear interactions.
   - Used primarily for **lift ranking** and feature importance.

Both are evaluated on a held-out test set.

**Key metrics (test set)**

- Logistic:
  - Accuracy ≈ 0.70
  - ROC AUC ≈ 0.73–0.74
- Random Forest:
  - Accuracy ≈ 0.71
  - ROC AUC ≈ 0.74–0.75

(See Part B: *metrics tables and confusion matrices*.)

### 3.2 Drivers of Core_90d

From the logistic coefficients and RF feature importances:

- **Strong positive drivers**
  - Prime at first login (`prime_status_at_first_login_Prime`)
  - Any lead signal (`lead_any`)
  - Number of distinct platforms in first 7 days (`n_distinct_platforms_7d`)
  - Upload events in first 7 days (`n_upload_events_7d`)
  - App launches in first 7 days (`n_app_launch_7d`)
  - Notifications opened in first 7 days

- **Negative drivers**
  - Web-only entry (`entry_client_PhotosWeb`)
  - Some low-intent channels (BizDev, generic Clickstream, Other)

Lift curves by decile show that the RF model ranks customers sensibly: top deciles have much higher Core_90d rates than the bottom ones.

> See Part B: *“Top drivers of Core_90d (model view)”* and *lift-by-decile charts*.

---

## 4. Segmentation of the Photos base

Using early behavior and platform/Prime, we cluster customers into four segments (K-means in Part B/C):

1. **Cluster 0 – Power Prime viewers**
   - High app launches and photo views.
   - Mostly Prime, mobile-heavy.
   - Highest baseline Core_90d.

2. **Cluster 2 – Prime mid-engaged**
   - Moderate launches/views.
   - Prime-heavy but less active than cluster 0.

3. **Cluster 3 – Mixed mid-engaged**
   - Mix of Prime/NonPrime, mobile + Web.
   - Mid-tier engagement.

4. **Cluster 1 – Low-engaged NonPrime/Trial**
   - Low launches/views.
   - Lower baseline Core_90d.

These clusters are the **units we target** with any new video surface and the units we use in the simulation.

> See Part C: *cluster summary tables and “Behavioral profile by cluster” plot.*

---

## 5. Video product decision (Part B → Part C)

The new question, framed in Part B:

> **“Should we build a short-form video feed for Photos customers?  
> If yes, should it be integrated into the existing Photos app or launched as a standalone app?”**

This is not a pure UX preference question. We care specifically about:

- impact on **Core_90d**, and
- how that impact scales across segments.

Part C uses the existing synthetic Photos dataset and the cluster structure to explore two paths:

1. **Integrated video feed** inside Photos:  
   A new feed surface reachable from the main app.

2. **Standalone video app**:  
   A separate app that shares storage and account but has its own entry point and growth loop.

---

## 6. Priors for video (Part C)

We have no production data for “Photos video feed” or “Photos standalone video app”.  
So we define **priors**: explicit, per-segment starting assumptions for video behavior.

### 6.1 Adoption priors

For each cluster \(c\), we specify:

- \(p_{c,\mathrm{int}}\): probability of adopting integrated video.  
- \(p_{c,\mathrm{stand}}\): probability of adopting standalone.

| cluster | segment_label                | p_adopt_integrated | p_adopt_standalone |
|--------:|-----------------------------|-------------------:|-------------------:|
| 0       | Power Prime viewers         | 0.70               | 0.50               |
| 2       | Prime mid-engaged           | 0.50               | 0.35               |
| 3       | Mixed mid-engaged           | 0.40               | 0.25               |
| 1       | Low-engaged NonPrime/Trial  | 0.20               | 0.10               |

Interpretation:

- Integrated video is assumed **easier to sample** than installing a new app → higher adoption in every segment.
- Low-engaged NonPrime/Trial are the hardest to move → most conservative priors.

A sanity check chart compares:

- baseline Core_90d rate (%),
- `p_adopt_integrated`,
- `p_adopt_standalone`,

per cluster, to ensure we’re not assuming adoption wildly out of sync with retention.

### 6.2 Usage priors (sessions & minutes)

For each cluster \(c\) and mode \(m\), we assume:

- \(\lambda_{c,m}\): mean video sessions per 7 days (Poisson mean).  
- \(\mu_c\): mean minutes per session (same for both modes).

| cluster | segment_label                | mean_sessions_integrated | mean_sessions_standalone | mean_minutes_per_session |
|--------:|-----------------------------|-------------------------:|--------------------------:|--------------------------:|
| 0       | Power Prime viewers         | 7.0                      | 6.0                       | 4.0                       |
| 2       | Prime mid-engaged           | 5.0                      | 4.0                       | 3.5                       |
| 3       | Mixed mid-engaged           | 4.0                      | 3.0                       | 3.0                       |
| 1       | Low-engaged NonPrime/Trial  | 2.0                      | 1.5                       | 2.5                       |

Comparison against real behavior:

- Power Prime currently launch the app ~30+ times/week → 6–7 video sessions/week is ambitious but within the realm of their existing usage.
- Mid-engaged segments see 4–7 launches/week → video sessions are modeled as a subset/redistribution of that behavior.

### 6.3 Impact priors (uplift in Core_90d)

For each cluster and mode we define:

- \(\delta_{c,\mathrm{int}}\): uplift in Core_90d probability if a user adopts integrated video.  
- \(\delta_{c,\mathrm{stand}}\): uplift in Core_90d probability if a user adopts standalone.

Magnitudes are in the range of a few percentage points, loosely calibrated to:

- observed lifts from memories, notifications, and wakescreen features,
- the idea that a new high-engagement habit can move Core_90d but won’t double it.

These priors are **stress-tested** in the sensitivity analysis rather than treated as ground truth.

---

## 7. Simulation design

We simulate high-engagement segments (clusters 0, 2, 3) on a hypothetical **24M active** base.

### 7.1 Per-user simulation

For each user \(i\) in cluster \(c(i)\), and mode \(m \in \{\mathrm{int}, \mathrm{stand}\}\):

1. **Video adoption**

   \[
   A_i^{(m)} \sim \mathrm{Bernoulli}\left(p_{c(i),m}\right)
   \]

2. **Video sessions**

   \[
   S_i^{(m)} \sim \mathrm{Poisson}\left(\lambda_{c(i),m}\right)
   \]

3. **Video minutes**

   \[
   T_i^{(m)} = S_i^{(m)} \cdot \mu_{c(i)} \cdot \exp(\varepsilon_i),
   \quad \varepsilon_i \sim \mathcal{N}(0,\sigma^2)
   \]

4. **Core_90d with video**

   Baseline Core_90d rate for the cluster is \(b_{c(i)}\).  
   If video is adopted, probability is shifted by \(\delta_{c(i),m}\):

   \[
   p_i^{(m)} = \min\!\left( \max\!\left( b_{c(i)} + \delta_{c(i),m} \cdot A_i^{(m)},\, 0 \right),\, 0.99 \right)
   \]

   Then:

   \[
   Y_i^{(m)} \sim \mathrm{Bernoulli}\left(p_i^{(m)}\right)
   \]

5. **Scenario Core_90d rate**

   For a simulation run with \(N_\text{sim}\) users:

   \[
   R_m = \frac{1}{N_\text{sim}} \sum_{i=1}^{N_\text{sim}} Y_i^{(m)}
   \]

   Baseline (no video) Core_90d is:

   \[
   R_0 = \sum_{c} \pi_c\, b_c
   \]

   where \(\pi_c\) is the share of users in cluster \(c\).

6. **Uplift and incremental Core_90d users**

   Uplift in Core_90d rate:

   \[
   U_m = R_m - R_0
   \]

   Incremental Core_90d customers on the 24M base:

   \[
   \Delta \text{Core}_m = U_m \cdot N_{\text{high}}
   \]

   where \(N_{\text{high}}\) is the number of high-engagement customers in the population (approx. 24M × share in clusters 0,2,3).

### 7.2 Monte Carlo procedure

For each mode \(m\):

1. For runs \(s = 1,\dots,S\):
   - Simulate adoption, usage, and Core outcomes.
   - Compute \(\Delta \text{Core}_m^{(s)}\).

2. Summarize across runs:
   - Mean uplift: \(\overline{\Delta \text{Core}_m}\).
   - Standard deviation.
   - Percentiles (p5, p50, p95).

We repeat this for **integrated** and **standalone**, then compare distributions.

---

## 8. Base results: integrated vs standalone

Using the priors in Section 6:

- **Integrated video (Photos feed)**

  - Mean incremental Core_90d users (24M base):  
    \(\overline{\Delta \text{Core}_\mathrm{int}} \approx 306{,}000\)
  - 5th–95th percentile: ~224k – 391k.

- **Standalone video app**

  - Mean incremental Core_90d users (24M base):  
    \(\overline{\Delta \text{Core}_\mathrm{stand}} \approx 248{,}000\)
  - 5th–95th percentile: ~168k – 336k.

- **Difference (standalone − integrated)**

  - Mean: \(\overline{\Delta \text{Core}_\mathrm{stand}} - \overline{\Delta \text{Core}_\mathrm{int}} \approx -58{,}000\).
  - 5th–95th percentile: roughly −185k to +68k.
  - Empirical probability that standalone beats integrated under these priors: ~0%.

**Interpretation**

> On these priors, an integrated video feed in Photos is expected to create roughly **60k more Core_90d customers** than a standalone app on a 24M active base, and standalone almost never wins in the simulated draws.

---

## 9. Sensitivity analysis over standalone priors

To ensure we’re not hard-coding the answer, we stress-test the **standalone** priors while holding integrated fixed.

For multipliers:

- \(\alpha \in [0.8, 1.6]\): adoption multiplier  
  \(p_{c,\mathrm{stand}}' = \alpha \cdot p_{c,\mathrm{stand}}\)
- \(\beta \in [0.8, 1.6]\): uplift multiplier  
  \(\delta_{c,\mathrm{stand}}' = \beta \cdot \delta_{c,\mathrm{stand}}\)

For each grid point \((\alpha, \beta)\):

1. Re-run the standalone simulation with \(p'\) and \(\delta'\).
2. Compute mean incremental standalone uplift \(\overline{\Delta \text{Core}_\mathrm{stand}(\alpha,\beta)}\).
3. Compute the **difference vs integrated**:

   \[
   D(\alpha,\beta) =
   \overline{\Delta \text{Core}_\mathrm{stand}(\alpha,\beta)} -
   \overline{\Delta \text{Core}_\mathrm{int}}
   \]

We plot \(D(\alpha,\beta)\) as a heatmap:

- **Red**: \(D(\alpha,\beta) < 0\) → standalone worse than integrated.  
- **Blue**: \(D(\alpha,\beta) > 0\) → standalone better.

**Key observations from the heatmap**

- Around \((\alpha,\beta) = (1.0,1.0)\) (base priors), standalone is clearly worse (≈ −58k).
- Even at \(\alpha\) or \(\beta\) up to ~1.2, the plot stays red → modest improvements in standalone adoption *or* uplift are not enough.
- Standalone only starts to reliably beat integrated in the **top-right corner** where:
  - adoption is ~40–60% higher than assumed (`α ≈ 1.4–1.6`), **and**
  - uplift per adopter is ~30–50% higher than assumed (`β ≈ 1.3–1.5`).

**Plain language**

> For a standalone app to be the better bet, it needs to **dramatically outperform** reasonable expectations on both adoption *and* its impact on Core_90d. Integrated is the base-rate winner across a wide, plausible region of parameter space.

---

## 10. Verification and limitations

### 10.1 Verification

- **Implementation sanity**
  - If we set \(p_{c,\mathrm{int}} = p_{c,\mathrm{stand}}\) and \(\delta_{c,\mathrm{int}} = \delta_{c,\mathrm{stand}}\), the integrated and standalone simulations produce indistinguishable results.
  - If we set all \(p_{c,m} = 0\), both scenarios reduce to baseline Core_90d.
  - Increasing all \(\delta_{c,m}\) scales uplift roughly as expected.

- **Robustness**
  - The heatmap shows integrated dominating across a wide band of \((\alpha,\beta)\).
  - Standalone only wins under *very* optimistic assumptions about its adoption and impact.

### 10.2 Limitations

- Priors, while reasoned, are still priors; they’re calibrated on analogous features, not real video usage.
- Simulation focuses on **Core_90d**; it does not explicitly model:
  - revenue per Core customer,
  - long-term brand halo or ecosystem value,
  - platform-specific economics (e.g., Fire TV vs mobile).

The simulation is a **decision aid**, not a substitute for running an actual experiment.

---

## 11. Recommended path

Given the analysis:

1. **Phase 0 – Integrated feed inside Photos**

   - Launch a lightweight short-form video surface inside Photos.
   - Start with high-engagement clusters (0 and 2: Power Prime, Prime mid-engaged).
   - Instrument adoption, sessions, and incremental Core_90d vs controls.

2. **Phase 1 – Decide on standalone**

   - Map observed adoption/uplift against the sensitivity heatmap:
     - If metrics land in the **“blue” region** (standalone-equivalent adoption and uplift ≥ ~1.4–1.5× our priors), a standalone app becomes rational.
     - If not, the base-rate bet is to keep video integrated and continue optimizing that surface.

3. **Phase 2 – Scale the winning pattern**

   - If integrated dominates, treat video as a Core-driver inside Photos + 2P surfaces (Echo Show, Fire TV).
   - If standalone proves exceptional, invest in its own growth loops, while still leveraging shared storage and identity.

---

This document + the three Colab notebooks together form a **transparent, reproducible argument** for why an integrated video feed is the default choice under realistic assumptions, and what conditions would need to be true for a standalone app to be the better bet.
