# Modeling Core Customer Activation for a Photos App (Synthetic Data)

This repo documents an end-to-end workflow for thinking about **Core customer activation** and a **short-form video strategy** for a cloud photo product similar to Amazon Photos.

All data is **synthetic** – it mimics the structure of real logs without exposing anything proprietary.

---

## Notebook index

1. **Part A – Core activation setup & synthetic data**

   *Colab:*  
   [Modeling Core Customer Activation – Part A]([https://colab.research.google.com/drive/18p5muzKLaigfUEKSWUgrqeMj5tDBk6aK?usp=sharing])

   - Define the **business context** and the `Core_90d` target.
   - Specify the `photos_first_login_features` schema (one row per customer).
   - Simulate a realistic synthetic dataset.

2. **Part B – Core activation modeling & insights**

   *Colab:*  
   [Modeling Core Customer Activation – Part B]([(https://colab.research.google.com/drive/1mCBabpIj7njjK5UlbnLX9c1ErYTqY_Z3?usp=sharing)])

   - Train baseline models (logistic regression, random forest) to predict `is_core_90d`.
   - Identify acquisition + early-usage drivers of Core activation.
   - Frame the video product decision:

     > “Should we build a short-form video feed for Photos customers?  
     > If yes, should it live inside Photos or as a standalone app?”

3. **Part C – Video strategy simulation (integrated vs standalone)**

   *Colab:*  
   [Video Strategy Simulation – Part C](https://colab.research.google.com/drive/1HI7hT58J0tFLY7F9JwlacAwloVw7Vvw2#scrollTo=MT-bM5NtAxKs)

   - Define **priors** for video adoption & impact per segment.
   - Simulate integrated vs standalone scenarios on a 24M-active base.
   - Run sensitivity analysis over the standalone priors.

The rest of this doc ties these notebooks into a single narrative.

---

## 1. Business context

We model a **cloud photo storage product** roughly analogous to Amazon Photos, integrated with:

- **1P clients:** Android, iOS, Desktop, PhotosWeb  
- **2P surfaces:** Echo Show, Fire TV, Alexa

Core logic:

> If we **delight customers**, we get **retention**, which drives **profit**.

Operationally that means turning new signups into **Core customers** who:

- have uploaded photo or video content, and  
- regularly launch our apps across key platforms.

Because real production logs are confidential, Part A builds a **synthetic dataset** with:

- the same entity structure (one row per customer),
- similar distributions for attribution, platforms, engagement,
- realistic definitions for “First Login”, “Acquired”, “Engaged”, “Core”.

---

## 2. Modeling goal: `Core_90d` activation

**Entity**  
`customer_id` – one row per customer.

**Start time**  
`first_login_ts` – first-ever interaction that counts as Photos usage.

- Mobile: first app launch or upload.  
- Web: first launch on a “Your Photos” page.  
- Android has pre-/post-`SignInSuccess` logic to mirror production.

**Target**  
`is_core_90d` – binary flag:

- `1` if the customer satisfies the Core definition within 90 days of first login.  
- `0` otherwise.

The synthetic table `photos_first_login_features` includes:

- **Attribution**

  - `attrib_channel_type` – Marketing, Alexa, Sharing, Clickstream, BizDev, Other…  
  - `attrib_client_tier` – 1P / 2P / 3P / Other.

- **Entry platform**

  - `entry_client` – Android, iOS, Desktop, PhotosWeb.

- **Lead flags**

  - group sharing, wakescreen, prints, Snapfish, retail prints, etc.

- **Early 7-day behavior**

  - `n_upload_events_7d`, `n_app_launch_7d`, `n_photo_view_events_7d`,  
    `n_notifications_opened_7d`, `n_distinct_platforms_7d`.

- **Outcomes / states**

  - Acquired, Engaged, Core, churn proxies, revenue hooks.

See Part A for the schema and generation code.

---

## 3. Baseline modeling of `Core_90d` (Part B)

Before touching video, we need to understand **what currently drives Core activation**.

### 3.1 Models

We train two baseline models on the synthetic dataset:

1. **Logistic regression**

   - Features: attribution, 1P/2P tier, entry platform, lead flags, early 7-day behavior.
   - Output: calibrated probability for `is_core_90d`.

2. **Random forest**

   - Same feature set; captures non-linearities and interactions.
   - Used mainly for lift ranking and feature importance.

**Test metrics (illustrative)**

- Logistic: accuracy ≈ 0.70, ROC AUC ≈ 0.73–0.74  
- Random forest: accuracy ≈ 0.71, ROC AUC ≈ 0.74–0.75

(Exact numbers in Part B.)

### 3.2 Drivers of `Core_90d`

From logistic coefficients and RF importances:

- **Strong positive drivers**

  - `prime_status_at_first_login_Prime`  
  - `lead_any` (any lead source present)  
  - `n_distinct_platforms_7d`  
  - `n_upload_events_7d`, `n_app_launch_7d`  
  - `n_notifications_opened_7d`

- **Negative drivers**

  - `entry_client_PhotosWeb` (web-only entry)  
  - Low-intent channels like generic Clickstream / Other / some BizDev.

Lift-by-decile curves show the models rank customers sensibly: top score deciles have much higher `Core_90d` rates than bottom ones.

---

## 4. Segmentation of the Photos base

Using early behavior + platform + Prime, we cluster customers (K-means) into four segments:

1. **Cluster 0 – Power Prime viewers**

   - Very high launches & photo views.  
   - Strong Prime skew, mobile-heavy.  
   - Highest baseline `Core_90d`.

2. **Cluster 2 – Prime mid-engaged**

   - Moderate launches & views.  
   - Mostly Prime, lower intensity than cluster 0.

3. **Cluster 3 – Mixed mid-engaged**

   - Mix of Prime and NonPrime.  
   - Mid-tier engagement.

4. **Cluster 1 – Low-engaged NonPrime / Trial**

   - Low launches & views.  
   - Lowest baseline `Core_90d`.

These clusters are the units we target with any video surface, and the units used in simulation.

(See Part C for the cluster summary tables and “behavioral profile by cluster” plot.)

---

## 5. Video product decision

The key product question framed in Part B:

> **“Should we build a short-form video feed for Photos customers?  
> If yes, should it be integrated into the existing Photos app or launched as a standalone app?”**

We care about:

- impact on **`Core_90d`**, not just raw views or watch time,
- how that impact scales across segments under realistic adoption.

Part C uses the synthetic Photos dataset + clusters to compare two strategies:

1. **Integrated video feed** inside Photos.  
2. **Standalone video app** sharing storage and identity but with its own entry point.

---

## 6. Priors for video behavior (Part C)

We have no real “Photos video feed” data, so we must define **priors** – explicit starting assumptions, per segment, that we are willing to update later when experiment data arrives.


### 6.1 Adoption priors

For each cluster \(c\), define:

- $p_{c,\text{int}}$ – probability that a user in cluster \(c\) adopts **integrated** video (inside Photos).
- $p_{c,\text{stand}}$ – probability that a user in cluster \(c\) adopts the **standalone** video app.



| cluster | segment_label               | `p_adopt_integrated` | `p_adopt_standalone` |
|--------:|----------------------------|----------------------:|----------------------:|
| 0       | Power Prime viewers        | 0.70                  | 0.50                  |
| 2       | Prime mid-engaged          | 0.50                  | 0.35                  |
| 3       | Mixed mid-engaged          | 0.40                  | 0.25                  |
| 1       | Low-engaged NonPrime/Trial | 0.20                  | 0.10                  |

Interpretation:

- Integrated is easier to “stumble into”, so adoption is higher everywhere.  
- Low-engaged NonPrime / Trial are hardest to move → most conservative priors.

### 6.2 Usage priors (sessions & minutes)

For each cluster \(c\) and mode \(m\):

- $\lambda_{c,m}$ – average video sessions per 7 days.
- $\mu_c$ – average minutes per session.


| cluster | segment_label               | `mean_sessions_integrated` | `mean_sessions_standalone` | `mean_minutes_per_session` |
|--------:|----------------------------|---------------------------:|----------------------------:|----------------------------:|
| 0       | Power Prime viewers        | 7.0                        | 6.0                         | 4.0                         |
| 2       | Prime mid-engaged          | 5.0                        | 4.0                         | 3.5                         |
| 3       | Mixed mid-engaged          | 4.0                        | 3.0                         | 3.0                         |
| 1       | Low-engaged NonPrime/Trial | 2.0                        | 1.5                         | 2.5                         |

These are calibrated against real app-launch frequencies in the synthetic data:  
Power Prime users already launch the app ~30+ times per week, so 6–7 video sessions is an aggressive but plausible subset.

### 6.3 Impact priors (uplift in `Core_90d`)

For each cluster \(c\) and mode \(m\):

- \(b_c\): baseline `Core_90d` rate for cluster \(c\).  
- \(\delta_{c,m}\): uplift in `Core_90d` probability if a user adopts video in mode \(m\).

Values are on the order of a few percentage points and are loosely calibrated to similar features (memories, notifications, wakescreen). They are **not** assumed to double retention.

These priors are what we stress-test in the simulation; they are not “truth”.

---



## 7. Simulation design

We simulate high-engagement segments (clusters 0, 2, 3) on a hypothetical **24M active** base to compare:

- an **integrated** video feed inside Photos, and  
- a **standalone** video app.

### 7.1 Per-user data-generating process

For each user *i* in cluster `c(i)` and mode `m ∈ {int, stand}`:

1. **Video adoption**

The user adopts the video surface with probability `p_{c(i),m}`:

`A_i^{(m)} ~ Bernoulli(p_{c(i),m})`.

2. **Video sessions in 7 days**

Weekly video sessions follow a Poisson law with rate `λ_{c(i),m}`:

`S_i^{(m)} ~ Poisson(λ_{c(i),m})`.

3. **Video watch minutes in 7 days**

Total minutes watched in 7 days:

`T_i^{(m)} = S_i^{(m)} · μ_{c(i)} · exp(ε_i)`, with `ε_i ~ Normal(0, σ²)`.

4. **Core_90d probability with video**

Baseline Core_90d rate for the cluster is `b_{c(i)}`.  
If the user adopts video (`A_i^{(m)} = 1`), we add a mode-specific uplift `δ_{c(i),m}` and clip into `[0, 0.99]`:

`p_i^{(m)} = min( max( b_{c(i)} + δ_{c(i),m} · A_i^{(m)}, 0 ), 0.99 )`.

5. **Core_90d outcome**

The Core outcome is drawn as:

`Y_i^{(m)} ~ Bernoulli(p_i^{(m)})`.

---

### 7.2 Scenario-level Core_90d metrics

For a single simulation run with `N_sim` users under mode *m*:

- **Core_90d rate with video**

`R_m = (1 / N_sim) · Σ_i Y_i^{(m)}`.

- **Baseline Core_90d rate (no video)**

`R_0 = Σ_c π_c · b_c`, where `π_c` is the fraction of users in cluster *c*.

- **Uplift in Core_90d**

`U_m = R_m − R_0`.

- **Incremental Core users on the high-engagement base**

`ΔCore_m = U_m · N_high`,

where `N_high` is the number of high-engagement users (the share of the 24M base in clusters 0, 2, 3).

---

### 7.3 Monte Carlo procedure

We repeat the simulation `S` times to get a distribution, not a single point estimate.

For each mode `m ∈ {int, stand}` and run `s = 1,…,S`:

1. Simulate adoption, usage, and `Y_i^{(m)}` for all users.  
2. Compute `ΔCore_m^{(s)}`.

We then summarize:

- Mean uplift:  
  `\overline{ΔCore_m} = (1 / S) · Σ_s ΔCore_m^{(s)}`.
- Standard deviation of `{ΔCore_m^{(s)}}`.  
- Percentiles p5, p50, p95 of `{ΔCore_m^{(s)}}`.

We do this for both **integrated** and **standalone** and compare the resulting distributions.

---

## 8. Base results: integrated vs standalone

Using the priors in Section 6, the Monte Carlo results on a 24M active base are:

### 8.1 Integrated video (Photos feed)

Mean incremental Core customers:  
`\overline{ΔCore_int} ≈ 306,000`.

5th–95th percentile: roughly 224k–391k.

### 8.2 Standalone video app

Mean incremental Core customers:  
`\overline{ΔCore_stand} ≈ 248,000`.

5th–95th percentile: roughly 168k–336k.

### 8.3 Difference (standalone − integrated)

Mean difference:  
`\overline{ΔCore_stand} − \overline{ΔCore_int} ≈ −58,000`.

Empirical probability that standalone wins:  
`P(ΔCore_stand > ΔCore_int) ≈ 0`.

**Takeaway:** under these priors, an integrated video feed creates roughly **60k more Core customers** than a standalone app on a 24M active base, and standalone essentially never wins in the simulations.

---

## 9. Sensitivity analysis over standalone priors

We stress-test the **standalone** priors while keeping integrated fixed.

Let:

- `α` = multiplier on standalone adoption,  
- `β` = multiplier on standalone uplift.

For each cluster *c*:

- Adjusted adoption: `p'_{c,stand} = α · p_{c,stand}`.  
- Adjusted uplift: `δ'_{c,stand} = β · δ_{c,stand}`.

For each grid point `(α, β)` in a range such as `[0.8, 1.6] × [0.8, 1.6]`:

1. Re-run the standalone simulation using `p'` and `δ'`.  
2. Compute mean incremental standalone uplift  
   `\overline{ΔCore_stand(α,β)}`.  
3. Compare to integrated via  

   `D(α,β) = \overline{ΔCore_stand(α,β)} − \overline{ΔCore_int}`.

We visualize `D(α,β)` as a heatmap:

- `D(α,β) < 0` (red) → standalone worse than integrated.  
- `D(α,β) > 0` (blue) → standalone better.

### 9.1 What the heatmap shows

- Around `(α,β) = (1.0, 1.0)` (base priors), `D(α,β) ≈ −58k`: integrated clearly wins.
- Even up to `α ≈ 1.2` or `β ≈ 1.2`, the map stays red: modest improvements in standalone adoption or uplift are not enough.
- Standalone only becomes clearly better in the **top-right** of the grid where both are true:
  - adoption is about 40–60% higher than base (`α ≈ 1.4–1.6`), **and**
  - uplift per adopter is about 30–50% higher (`β ≈ 1.3–1.5`).

**Plain language:** for standalone to be the better bet, it has to **massively beat expectations** on both adoption and impact on Core. Integrated is the base-rate winner across most reasonable parameter settings.

---

## 10. Verification and limitations

### 10.1 Verification

Sanity checks:

- If we set `p_int = p_stand` and `δ_int = δ_stand`, the integrated and standalone simulations produce nearly identical distributions.
- If we set all `p_{c,m} = 0`, both scenarios collapse to the same baseline `R_0`.
- Uniformly scaling all `δ_{c,m}` shifts uplift roughly linearly, as expected.
- The qualitative shape of the heatmap is stable for reasonable `S` (number of runs).

### 10.2 Limitations

- Priors are assumptions, informed by analogous features, not measured video data.
- The simulation optimizes only for `Core_90d`; it does **not** explicitly model:
  - revenue per Core user,
  - long-term brand or ecosystem effects,
  - platform-specific economics (Fire TV vs mobile),
  - cannibalization of other surfaces.

This is a decision aid, not a substitute for real experiments.

---

## 11. Recommended path

Given the analysis:

1. **Phase 0 – Integrated feed inside Photos**

   - Launch a lightweight short-form video surface within Photos.  
   - Start with high-engagement clusters (0 and 2: Power Prime & Prime mid-engaged).  
   - Instrument adoption, session depth, and incremental `Core_90d` vs controls.

2. **Phase 1 – Decide on standalone**

   - From experiments, estimate empirical `(α̂, β̂)` and locate that point on the sensitivity heatmap.  
   - If it lands in the blue region (standalone adoption/uplift ≥ ~1.4–1.5× priors), a standalone app is rational.  
   - Otherwise, the base-rate bet is to keep video integrated and continue optimizing that surface.

3. **Phase 2 – Scale the winner**

   - If integrated dominates, double-down on video as a Core driver inside Photos and across 2P surfaces (Echo Show, Fire TV, Alexa).  
   - If standalone proves exceptional, invest in its own growth loops and brand, while sharing the same storage and identity backend.

Together, this README and the three Colab notebooks form a transparent, reproducible argument for why an integrated video feed is the default winner under realistic assumptions, and what conditions must be true for a standalone app to actually be the better bet.
