# PatientTwin AI — Build Specification

Cardiac-risk digital twin for the Happiest Health Digital Twin Challenge 2026 (Unstop).
This file is the single source of truth for the build. Read it fully before writing any code, and re-read the relevant section before each stage.

---

## 0. How this file is used

- The developer is a solo student building this over about four days. Time is short, so correctness and a working end-to-end demo matter more than extra features.
- Build in the stages listed in `PROMPTS.md`. After each stage: run it, show real output, and stop for review.
- If anything in this spec is ambiguous or seems wrong, say so and propose a default. Do not silently invent behaviour.
- Anything marked **MUST** is required. **SHOULD** is wanted if time allows. **MAY** is optional.

---

## 1. What we are building (one paragraph)

A virtual patient ("digital twin") that holds a static EHR-style profile and a continuously updating stream of wearable-style data (resting heart rate, HRV, sleep, steps, BP, glucose). A machine-learning model that ingests **both** streams estimates the probability of a **high-risk cardiac event or anomaly in the next 72 hours**. Each new day of data updates the twin's stored state, refreshes the model output, shows before/after, and can trigger alerts and simulated notifications to an emergency contact and a doctor. A polished doctor-facing dashboard presents all of this.

All data is synthetic or public. Nothing here is medical advice or a validated clinical tool.

---

## 2. Contest rules this build must satisfy (from the official brief)

1. The prototype **MUST** fuse two streams:
   - **Static/historical:** simulated EHR (demographics, past diagnoses, lab results, optional genetic markers).
   - **Dynamic/real-time:** simulated or open-source wearable/IoT time series (heart rate, HRV, sleep, steps, CGM).
2. A working algorithmic model **MUST** ingest both streams and predict an adverse event.
3. A conceptual **doctor dashboard** MUST show how a doctor would interact with the virtual patient.
4. Only anonymized, open-source or synthetic data. No real patient data, ever.
5. Submission is a **public GitHub repo**; the root folder name is `TeamName_CollegeName`.
6. The README **MUST** contain: team and college details; project title; problem statement and healthcare use case; technical stack and AI/ML model details; licence details.
7. The repo/submission **MUST** also have: an architecture diagram (PDF/PPT), a presentation (PDF/PPT) with project details and outcome, and a demo video of at least 20 minutes (recorded by the developer, not by you).

Your job covers the code, README, architecture diagram file, a first-draft presentation, and the video script. The developer records the video.

---

## 3. Guardrails (non-negotiable)

1. **Never invent results.** Every number in the README, docs and UI (metrics, counts, rates) must come from files produced by running the code (`models/metrics.json`, etc.). If a result does not exist yet, write `TBD (run src/train.py)`.
2. **No data leakage.** Features at prediction time `t` may only use data from days `<= t`. Split train/validation/test **by patient**, never by random row. Personal baselines used in features must be computed only from days before the feature window.
3. **Be honest about the labels.** Event labels come from our simulator. Say so in the README, docs and UI "About" panel. Never write "predicts heart attacks". Use "modelled probability of a high-risk cardiac event on simulated data".
4. **Do not retrain** when new data arrives in the app. Update the patient window and run inference with the saved model.
5. **Reproducibility.** Fixed seeds from `config.yaml`. One command per stage. Pinned versions in `requirements.txt`.
6. **No real patient data, no secrets, no real SMS/email sending.** Notifications are simulated previews written to a log.
7. **Fictional patients only.** Fictional names, clearly tagged "Simulated".
8. **Do not add features not in this spec** (no LLM features, no cloud deployment, no real device integration) unless asked.
9. **Keep code readable.** Small modules, docstrings, type hints. The developer must be able to explain every file in a viva.
10. **Never leave placeholders that look real.** No fake team names, fake licences, fake links. Use clearly marked `[FILL IN]` fields in the README.

---

## 4. Environment

- Python 3.11 or 3.12. Work inside a virtual environment (`python -m venv .venv`).
- `requirements.txt` (pin the exact versions you test with):
  `streamlit`, `pandas`, `numpy`, `scikit-learn`, `plotly`, `joblib`, `pyyaml`, `scipy`, `matplotlib`, `pytest`, `python-pptx` (for the draft presentation).
- Local-only. The finished demo **MUST** run with no internet connection (bundle fonts and assets, no CDN calls at runtime).
- Commands the README must document:
  ```
  python -m venv .venv
  # Windows: .venv\Scripts\activate    macOS/Linux: source .venv/bin/activate
  pip install -r requirements.txt
  python -m src.simulate        # generate synthetic cohort
  python -m src.train           # train, evaluate, save model + metrics
  python -m src.make_demo       # pick demo patients from the test split
  streamlit run app.py          # launch dashboard
  pytest                        # run tests
  ```

---

## 5. Repository layout

The root folder is named `TeamName_CollegeName` (placeholder until the developer fills it in). Inside:

```
TeamName_CollegeName/
  app.py                      # Streamlit entry point (thin; calls ui/)
  config.yaml                 # seeds, cohort size, thresholds, paths
  requirements.txt
  README.md
  LICENSE                     # MIT unless the developer says otherwise
  AGENTS.md                   # optional: guardrails copy for the agent
  .streamlit/config.toml      # base theme + server options
  assets/
    fonts/                    # bundled font files + their licences
    pulse.svg                 # optional
  data/
    README.md                 # sources, licences, how to download Framingham
    raw/                      # framingham.csv goes here (not committed if licence forbids)
    synthetic/                # generated: static.csv, wearable_daily.csv, events.csv
    processed/                # generated: windows.csv (features + labels + split)
  models/
    fusion_pipeline.joblib
    baseline_risk.joblib      # if Framingham used
    metrics.json
    thresholds.json
    model_card.md
    plots/                    # roc.png, pr.png, calibration.png, confusion.png, importance.png
  src/
    config.py                 # loads config.yaml
    baseline_risk.py          # Framingham model or documented fallback formula
    simulate.py               # synthetic cohort generator
    features.py               # 7-day window features
    train.py                  # splits, baselines, models, calibration, metrics, plots
    twin.py                   # twin state engine
    db.py                     # SQLite helpers and schema
    alerts.py                 # tiers, rules, escalation, cooldown
    notifier.py               # simulated message previews + log
    replay.py                 # scenario replay of held-out patients
    whatif.py                 # model what-if simulation
    make_demo.py              # selects demo patients from test split
  ui/
    styles.css                # the design system (section 12)
    theme.py                  # python-side tokens, plotly template
    components.py             # HTML builders: hero, strip, feed, chips
    views_ward.py             # main monitoring view
    views_doctor.py           # printable doctor summary view
    views_about.py            # data, model, limits
  tests/
    test_features_no_leakage.py
    test_split_by_patient.py
    test_alert_rules.py
    test_twin_update.py
    test_replay.py
  docs/
    make_architecture.py      # generates docs/architecture.pdf
    architecture.pdf
    presentation_outline.md
    make_presentation.py      # draft .pptx via python-pptx
    video_script.md
    screenshots/              # UI screenshots for README and slides
```

---

## 6. Data specification

All paths and parameters come from `config.yaml` (seed, `n_patients`, `n_days`, event rates, thresholds).

### 6.1 Static EHR (`data/synthetic/static.csv`), one row per patient

| column | type / range | notes |
|---|---|---|
| patient_id | string | `P0001`... |
| display_name | string | fictional, Indian first name + initial, tagged "Simulated" |
| age | int, 30–80 | skewed to 45–65 |
| sex | M/F | |
| smoker | 0/1 | |
| cigs_per_day | int | 0 if non-smoker |
| diabetes | 0/1 | |
| prior_hypertension | 0/1 | |
| prior_cad | 0/1 | prior coronary disease |
| bp_meds | 0/1 | |
| total_chol | mg/dL, 140–300 | lab result |
| hba1c | %, 4.8–10 | lab result, higher if diabetes |
| bmi | 18–40 | |
| family_history | 0/1 | stand-in for the brief's optional "genetic markers"; label it as a simulated flag |
| baseline_risk | 0–1 | from `baseline_risk.py` |
| emergency_contact_name / relation | string | fictional |
| doctor_name | string | fictional |

Correlations should be plausible (older, diabetic, hypertensive, smoker patients have higher baseline risk) but with noise.

### 6.2 Baseline risk (`src/baseline_risk.py`)

- **Preferred:** if `data/raw/framingham.csv` exists, train a logistic regression (in a Pipeline) on the public Framingham heart-study data to predict its 10-year CHD label from the fields that overlap our static profile (age, sex, smoker, cigs per day, BP meds, diabetes, total cholesterol, BMI, and systolic BP if available). Report its own AUC on a held-out part. Use it to score each synthetic patient. Confirm the actual column names in the file; do not assume them.
- **Fallback:** if the file is absent, use a documented logistic formula with fixed coefficients, and print a warning. Label it "synthetic baseline score" everywhere.
- Output is a probability in [0, 1], stored as `baseline_risk`.

### 6.3 Wearable daily series (`data/synthetic/wearable_daily.csv`), one row per patient per day

Columns: `patient_id, day (0..n_days-1), date, resting_hr, hrv_rmssd, sleep_hours, deep_sleep_pct, steps, systolic, diastolic, glucose_fasting, symptom_flag`.

- `n_days` default 120, `n_patients` default 3000 (config). About 3% of values randomly missing (to force imputation realism).
- **Personal norms:** each patient has their own baseline for each signal, derived from the static profile plus noise (older, higher BMI, smoker, diabetic → higher resting HR, lower HRV, higher BP, higher glucose).
- **Normal variation:** AR(1) day-to-day noise, a weekly pattern in steps and sleep, small slow drift.
- **Deterioration episodes (the signal the model must find):**
  - Each patient has an `event_propensity` that rises with `baseline_risk`.
  - An **event** happens on day `E`. In the `L` days before it (`L` in {3,4,5}) a ramped prodrome appears: resting HR up (+4 to +14 bpm), HRV down (−10% to −35%), sleep down (−0.5 to −2.0 h), steps down (−10% to −45%), systolic up (+5 to +25 mmHg). Each signal participates with probability about 0.7, so the label is never a simple threshold on one signal. Ramp shape increases toward the event.
  - A **benign look-alike episode** (same shapes at about 60% of the magnitude, but no event) occurs about 1.5 times as often as events, to keep the task honest.
  - After an event, the next 7 days are recovery: exclude those days from windows.
  - Tune the event rate in `config.yaml` so that **2% to 6%** of eligible windows are positive. Print the actual rate.
- `symptom_flag` (0/1): rare, raised on some event days only. Used by the alert rules, **not** as a model feature (avoid label leakage).

### 6.4 Events (`data/synthetic/events.csv`)

`patient_id, event_day, prodrome_length, kind` where `kind` is `event`. Also store benign episodes in a separate `episodes.csv` for debugging.

### 6.5 Windows (`data/processed/windows.csv`)

One row per eligible `(patient_id, day t)` with at least 7 days of history. Label `y(t) = 1` if an event occurs on any of days `t+1, t+2, t+3` (the next 72 h at daily resolution). Include split column (`train/val/test`), assigned by patient.

---

## 7. Features (`src/features.py`)

For window days `t-6 .. t`, for each wearable signal (`resting_hr, hrv_rmssd, sleep_hours, deep_sleep_pct, steps, systolic, diastolic, glucose_fasting`):

- mean, min, max, std, last value, OLS slope over the window
- `delta_vs_own_norm`: window mean minus the patient's own median over days `t-36 .. t-7` (strictly before the window; needs a minimum of 14 valid days, else NaN)
- `z_vs_own_norm`: that delta divided by the patient's own robust spread (MAD) over the same earlier days
- count of missing days in the window

Plus static features (all columns in 6.1 except ids, names, contact fields) and `baseline_risk`.

`symptom_flag` and anything dated after `t` are **forbidden** as features. Provide `FEATURE_COLUMNS` as an explicit list and a test that fails if a forbidden column appears.

The same function builds features in training and in the app (single code path).

---

## 8. Modelling (`src/train.py`)

1. **Split by patient** (70/15/15 train/val/test, stratified by whether the patient has any event, seed from config). A test asserts no patient appears in two splits.
2. **Baselines** (evaluated on validation and test):
   - always predict no event;
   - simple rule: resting-HR z-score above a cut-off;
   - logistic regression (class-weighted).
3. **Main model:** gradient boosting (`HistGradientBoostingClassifier` or similar), inside a Pipeline with imputation. Compare with logistic regression and keep the simpler one if the difference is small. Explain the choice in `model_card.md`.
4. **Calibration:** calibrate on the validation split (isotonic or sigmoid) and show a calibration plot.
5. **Thresholds** chosen **on validation only**, then frozen in `models/thresholds.json`:
   - `watch`: threshold at about 90% recall
   - `urgent`: threshold at about 80% recall
   - `critical`: threshold where precision is at least 50% if reachable, otherwise the 99th percentile of validation scores
   - Document exactly how each was chosen.
6. **Test set is used once**, after everything is frozen. Report: ROC-AUC, PR-AUC, precision, recall, F1 at each threshold, confusion matrix, Brier score, number of patients/windows/positives per split, positive rate.
7. **Lead time:** for test patients with events, report how many days before the event the model first crosses the `urgent` threshold (median and distribution).
8. **Robustness:** regenerate the cohort with two other seeds and with 1.5x noise; re-run and report how much AUC and recall move. Save to `metrics.json` under `robustness`.
9. **Explainability:** permutation importance on the validation set; also a per-patient "top drivers" function (feature contribution approximated by permuting/neutralising feature groups, or coefficients if linear). Group drivers into plain-language labels (e.g. "Resting heart rate vs own norm").
10. **Save:** `fusion_pipeline.joblib`, `metrics.json`, `thresholds.json`, `model_card.md`, plots. Record library versions in `metrics.json`.

`model_card.md` must include intended use, data, label definition, metrics (copied from `metrics.json`), limitations, and the line "Evaluated on simulated data; not clinically validated."

---

## 9. Twin engine (`src/twin.py`, `src/db.py`)

SQLite file `twin.db` (path in config). Tables:

- `patients(id, display_name, static_json, baseline_risk, contact_json, doctor_json, created_at)`
- `readings(id, patient_id, day, date, resting_hr, hrv_rmssd, sleep_hours, deep_sleep_pct, steps, systolic, diastolic, glucose_fasting, symptom_flag, source)`  where `source` is `replay` or `manual`
- `twin_state(patient_id, as_of_day, features_json, baseline_json, updated_at)`
- `predictions(id, patient_id, day, probability, tier, model_version, drivers_json, created_at)`
- `alerts(id, patient_id, day, tier, reasons_json, message_contact, message_doctor, status, escalated_at, acknowledged_at)`

Functions (pure, unit-testable):

- `create_patient(static)`, `append_reading(patient_id, reading)` → validates ranges, rejects impossible values with a clear message, stores the reading.
- `refresh_state(patient_id)` → builds the 7-day window and features using `features.py`, runs the fixed pipeline, stores the prediction, evaluates alerts, returns `{previous, current, change, tier, drivers, alerts}`.
- `history(patient_id)` → readings, predictions, alerts for charts.

Rules: appending the same `(patient_id, day)` twice updates rather than duplicates. Working with fewer than 7 days of data returns a clear "not enough history" state instead of a number.

### Replay (`src/replay.py`, `src/make_demo.py`)

`make_demo.py` picks **five demo patients from the test split only** (never training patients) and writes `data/processed/demo_patients.json`:

1. **Steady:** low risk, no event, calm signals.
2. **Drifting:** a benign look-alike episode; risk rises but no event.
3. **Deteriorating:** a real event; the prodrome is visible and the alert fires before the event day.
4. **High baseline, stable:** high baseline risk but calm signals (shows that baseline alone doesn't mean alert).
5. **Young, noisy:** low baseline risk with a noisy benign episode (shows false-alarm control).

`replay.py` streams a chosen patient's days one by one through `append_reading` + `refresh_state` so the demo behaves exactly like live input. Supports play/pause/step/speed and "jump to day".

---

## 10. Alerts and notifications (`src/alerts.py`, `src/notifier.py`)

### 10.1 Tiers

| Tier | Label (UI) | Glyph | Trigger (any of) |
|---|---|---|---|
| Normal | STEADY | ● | probability below `watch` threshold and no rule hits |
| Watch | WATCH | ◆ | probability ≥ `watch`; or rule Watch; or own-baseline z-score ≥ 3 on any key signal |
| Urgent | URGENT | ▲ | probability ≥ `urgent`; or rule Urgent; or Watch confirmed on the next day |
| Critical | CRITICAL | ■ | probability ≥ `critical`; or rule Critical; or any symptom with an Urgent-level vital |

The tier is the highest tier reached. Every alert stores **reasons in plain language** (e.g. "Resting heart rate is 11 bpm above this patient's usual", "Model probability 34% is above the urgent threshold").

### 10.2 Safety-net rules (illustrative; the developer must cite guidelines in the README before final)

| Signal | Watch | Urgent | Critical |
|---|---|---|---|
| Systolic BP (mmHg) | 130–139 | 140–179 | ≥ 180 |
| Diastolic BP (mmHg) | 80–89 | 90–119 | ≥ 120 |
| Resting HR (bpm) | <50 or 101–110 | 111–130 | >130 or <40 |
| Fasting glucose (mg/dL) | <70 or 141–180 | <54 or 181–300 | >300 |
| Symptom flag | none | symptom with normal vitals | symptom with an Urgent-level vital |

Put thresholds in `config.yaml`, not hard-coded. Add a code comment: "Illustrative values; verify against AHA/ACC and ADA before use."

### 10.3 Behaviour

- **Cooldown:** the same tier for the same patient does not re-alert within `cooldown_days` (config, default 1 simulated day). A tier increase always alerts.
- **Recheck before Urgent from a single borderline Watch** (rules only, not model-driven).
- **Escalation:** Urgent notifies the emergency contact first; if not acknowledged within `ack_timeout` (config; 2 minutes real time in demo mode), also prepare the doctor message. Critical notifies both immediately.
- **Acknowledge** button sets `acknowledged_at` and stops escalation.
- **Notifier:** builds two message previews (contact: plain language; doctor: concise clinical summary with the last 7 days). Writes to the `alerts` table and `logs/notifications.log`. **Never sends anything.** Every preview is labelled "SIMULATED — not sent".
- Always include in Critical messages the line: "If the patient has chest pain, trouble breathing or faints, call emergency services now."

---

## 11. What-if (`src/whatif.py`) — SHOULD

Given the current window, let the user change: resting HR delta, sleep hours, steps, systolic BP. Recompute features and run the same fixed pipeline. Show the new probability next to the current one. Label: "Model simulation on simulated data — not advice".

---

## 12. UI design system — this is not a default Streamlit look

The dashboard must look like a designed clinical product, not a Streamlit template and not generic "AI app" styling. Implement with Streamlit plus a real CSS file, custom HTML components and a custom Plotly template.

### 12.1 Concept: "Ink & Pulse"

A calm, premium clinical chart. Warm paper, deep ink text, one vivid signal colour used sparingly for risk, serif numerals for the important number, and a faint ECG-paper grid in the background as a quiet cardiac motif. Think a well-designed medical journal crossed with a ward monitor, not a crypto dashboard.

### 12.2 Design tokens (put in `ui/styles.css` as CSS variables; mirror in `ui/theme.py`)

```
--paper:        #F5F1EA;   /* page */
--paper-2:      #EDE7DC;   /* recessed areas */
--card:         #FBF9F5;   /* panels */
--ink:          #1B1F2A;   /* main text */
--ink-2:        #5C6170;   /* secondary text */
--line:         #D9D2C5;   /* hairlines */
--steady:       #1F7A6D;   /* teal */
--watch:        #D9902B;   /* amber */
--urgent:       #E4472B;   /* vermilion */
--critical:     #A3141F;   /* deep crimson */
--grid-minor:   rgba(180, 60, 40, 0.05);
--grid-major:   rgba(180, 60, 40, 0.09);
--radius:       6px;       /* small, restrained radii */
```

- **Dark "ward mode" (SHOULD):** `--paper #0F1318`, `--card #171C23`, `--ink #E8E4DC`, `--ink-2 #9AA0AC`, `--line #2A313B`; same tier colours lightened for contrast. Toggle in the top bar.
- Contrast: body text at least 4.5:1. Never rely on colour alone; tiers always use glyph + label + colour.

### 12.3 Typography (bundle locally; all are SIL OFL licensed; include licence files)

- **Display / big numerals:** Fraunces (variable) — used for the risk number, section titles and patient name.
- **Body / UI:** IBM Plex Sans.
- **Data / labels:** IBM Plex Mono with tabular numerals for values, timestamps, units.
- Fallback stacks if files are missing: `Georgia, serif`; `system-ui, sans-serif`; `ui-monospace, Consolas, monospace`. The app must still look intentional with fallbacks and must never crash on missing fonts.
- Scale: risk number 88–120 px; section titles 22–26 px; body 15 px; labels 11–12 px uppercase with 0.08em tracking. Minimum text size 12 px.

### 12.4 Background and surfaces

- Page background: `--paper` plus an ECG-paper grid, applied to the page only (never behind dense text; panels sit on solid `--card`):
  ```
  background-image:
    linear-gradient(var(--grid-minor) 1px, transparent 1px),
    linear-gradient(90deg, var(--grid-minor) 1px, transparent 1px),
    linear-gradient(var(--grid-major) 1px, transparent 1px),
    linear-gradient(90deg, var(--grid-major) 1px, transparent 1px);
  background-size: 8px 8px, 8px 8px, 40px 40px, 40px 40px;
  ```
- Panels: 1 px `--line` border, no heavy shadows, no glassmorphism, no gradients except the tier ribbon. Section headers use a small uppercase mono label with a hairline rule, like a printed chart.

### 12.5 Layout (desktop 1440×900 first; stacks cleanly below 1100 px)

```
┌──────────────────────────────────────────────────────────────────────────┐
│ TOP BAR: wordmark + glyph | patient chip | "SIMULATED DATA" tag | as-of    │
│          time | mode: [Replay | Manual] | theme toggle | tabs: Ward · Doctor · About
├────────────┬───────────────────────────────────────────┬─────────────────┤
│ VIRTUAL    │ HERO: risk numeral, change chip, tier,      │ WHY THIS RISK   │
│ WARD       │ pulse line, risk ribbon                     │ driver bars     │
│ 5 patients │ ─────────────────────────────────────────── │ ─────────────── │
│ status dot │ SIGNAL STRIP: small multiples, 14 days      │ ALERT FEED      │
│ mini risk  │ (HR, HRV, sleep, steps, BP, glucose)        │ timeline +      │
│ sparkline  │ ─────────────────────────────────────────── │ message preview │
│            │ TWIN TIMELINE: scrubber + play controls     │ + Acknowledge   │
└────────────┴───────────────────────────────────────────┴─────────────────┘
```

Left rail about 260 px, right rail about 340 px, centre fluid. Left-aligned editorial layout, not centred-everything.

### 12.6 Components

1. **Top bar:** slim, 56 px. Wordmark "PatientTwin" in Fraunces with a tiny ECG-glyph. A persistent tag `SIMULATED DATA · EDUCATIONAL PROTOTYPE` in mono caps. Tabs styled as underlined text, not default Streamlit tabs.
2. **Virtual ward (left rail):** a list of five fictional patients. Each row: name, age/sex, tier glyph, a 7-day risk sparkline, current risk %. Selected row has an ink left-border. Clicking switches the twin.
3. **Hero risk panel:**
   - Label (mono caps): `PROBABILITY OF HIGH-RISK CARDIAC EVENT · NEXT 72 H`.
   - The number in Fraunces, huge (e.g. `18%`; show `<1%` rather than 0%).
   - Change chip next to it: `▲ +6.2 pts vs yesterday`, with the prior value in small text ("was 11.8%").
   - Tier badge: glyph + word + one-sentence plain reason.
   - **Pulse line:** an inline SVG ECG trace that scrolls horizontally. Speed follows the patient's resting HR for the day; irregularity and colour intensify with tier (steady teal, critical crimson). Honour `prefers-reduced-motion` (static trace).
   - **Risk ribbon** (instead of a speedometer gauge): a thin horizontal bar split into the four tier zones using the frozen thresholds, with a marker at the current probability and a ghost marker at yesterday's.
4. **Signal strip (small multiples):** six compact charts in a 3×2 grid. Each shows the last 14 days with: the patient's own-norm band shaded softly, the 7-day model window highlighted, faint guideline threshold lines, and today's point emphasised. Header per chart: signal name, latest value in mono, and "+9 vs own norm" in the tier colour when notable. No chart borders, no gridlines except a faint baseline, no legends (direct labels), transparent background.
5. **Twin timeline:** a horizontal scrubber across days with ticks for alerts and the event day (revealed only in replay review mode), play / pause / step / speed controls, and a "then vs now" comparison toggle that ghosts the previous state.
6. **Why this risk (right rail):** diverging horizontal bars of top drivers in plain language ("Resting heart rate is 9 bpm above usual", "Sleep down 1.4 h"), red for up-risk, teal for down-risk, each with a signed percentage-point contribution.
7. **Alert feed (right rail):** vertical timeline, newest first. Each item: tier glyph, time, headline, reasons. The latest alert expands to show the **simulated message preview** styled as a phone message bubble for the emergency contact and a compact clinical note for the doctor, both stamped `SIMULATED — NOT SENT`. `Acknowledge` button and an escalation countdown when relevant.
8. **Critical state:** a full-width crimson strip at the top reading "Seek emergency care" copy, crimson panel border on the hero, a subtle pulse on the alert card only. Calm everywhere else; no flashing.
9. **Manual entry:** a collapsible drawer ("Log today's reading") with the daily fields, symptom checkboxes and a Submit button. Submitting visibly updates the hero, strip and feed.
10. **What-if panel (SHOULD):** sliders in a side drawer, shows a ghost marker on the ribbon and "Model simulation — not advice".
11. **Doctor view (tab):** a one-page chart-style summary: patient header, static profile, baseline risk, current risk and trend, last 14 days of signals, alert history, and the model's top drivers. Print-friendly CSS (`@media print`: white background, no grid, no controls). This is the "conceptual doctor dashboard" the brief asks for.
12. **About tab:** data sources and licences, what is real vs synthetic, model description, real metrics from `metrics.json`, limitations, and the not-medical-advice statement.

### 12.7 Motion

Short, purposeful: number count-up on change (400 ms), ribbon marker slides (300 ms), alert card fades in. Pulse line is the only continuous animation. All motion disabled under `prefers-reduced-motion`.

### 12.8 Copy

Plain, precise, calm. No exclamation marks, no emoji, no "AI-powered" slogans. Examples: "Resting heart rate is 9 bpm above this patient's usual." "Model probability crossed the urgent threshold." Units always shown.

### 12.9 Plotly template (`ui/theme.py`)

Custom template: transparent paper/plot background, Plex Sans text, 1.5 px lines in `--ink`, own-norm band as a low-opacity fill, event markers as small diamonds, hover labels styled to the paper theme, no default Plotly blue palette, no modebar, no gridlines except a faint horizontal reference.

### 12.10 Streamlit implementation notes

- `st.set_page_config(layout="wide", page_title="PatientTwin", initial_sidebar_state="collapsed")`.
- Load `ui/styles.css` once via `st.markdown("<style>…</style>", unsafe_allow_html=True)`. Hide the default header, footer and main menu via CSS. Reduce default block padding.
- Build the hero, ribbon, cards and feed as HTML strings in `ui/components.py` (escape all dynamic text). Use `st.components.v1.html` for the animated SVG pulse if CSS animation inside markdown is unreliable.
- Streamlit's internal CSS selectors (`data-testid`) can change between versions: **pin the Streamlit version** and keep all selector overrides in one clearly commented block.
- Live replay: use `st.fragment(run_every=…)` (or a step button as fallback) with state in `st.session_state`.
- Bundle fonts under `assets/fonts` and reference them with `@font-face` (use `server.enableStaticServing` or base64 embedding). Verify the app looks right with the network disabled.
- Do **not** use: default `st.metric` styling, default Streamlit blue, emoji icons, purple/blue gradient hero, centred-everything layout, stock card grids with big shadows, default Plotly colours.

### 12.11 UI acceptance checks

- Screenshots at 1440×900, 1280×720 and 1024×768 saved to `docs/screenshots/` (use the browser tool if available) and reviewed for overlap, clipping and legibility.
- Tier is distinguishable without colour (glyph + label).
- The three key states look good: steady, drifting (Watch), critical.
- With fonts removed the fallback looks acceptable; with the network off nothing breaks.
- A first-time viewer can say what the patient's risk is, whether it went up or down, and why, within five seconds.

---

## 13. Tests (`tests/`)

- `test_features_no_leakage.py`: changing any value after day `t` leaves features at `t` unchanged; forbidden columns absent from `FEATURE_COLUMNS`.
- `test_split_by_patient.py`: no patient appears in more than one split.
- `test_alert_rules.py`: boundary values for each rule, tier precedence, cooldown, escalation timer, acknowledge.
- `test_twin_update.py`: appending a reading stores it, produces a new prediction, previous vs current are reported, duplicate day updates not duplicates, fewer than 7 days returns "not enough history".
- `test_replay.py`: replaying a demo patient produces the same predictions as batch scoring on the same windows.

All tests must pass with `pytest` from a clean clone.

---

## 14. Documentation deliverables

1. **README.md** with these headings, in this order, filled from real outputs:
   Team details and college `[FILL IN]` · Project title · Problem statement and healthcare use case · What makes this a digital twin · Data (real vs synthetic) · Technical stack · AI/ML model details (with real metrics from `metrics.json`) · Architecture (embed the diagram image) · How to run · Screenshots · Alerts and safety · Limitations and responsible AI · Future scope · Open-source licence and attributions · Demo video link `[FILL IN]`.
2. **docs/architecture.pdf**: generated by `docs/make_architecture.py` (matplotlib or similar). Data sources → feature builder → fusion model → twin state (SQLite) → alert engine → notifier → dashboard. Also export a PNG for the README.
3. **docs/presentation_outline.md** and a first-draft **`docs/PatientTwin_presentation.pptx`** via `python-pptx` (about 18–22 slides: problem, use case, brief, data, simulator, architecture, features, model, results, calibration, robustness, twin loop, alerts, dashboard screenshots, doctor view, impact, limitations, future scope). The developer will polish and may export PDF. Do not put numbers in slides that are not in `metrics.json`.
4. **docs/video_script.md**: a 22–25 minute script with timings (hook and problem 0–2, use case and brief 2–5, data 5–9, architecture 9–12, model 12–16, live demo 16–21, impact 21–23, limits and future 23–25), including what to click during the demo.
5. **data/README.md**: sources, licences, download instructions for Framingham (URL placeholder `[FILL IN]` — do not invent a URL), and how the synthetic data is made.
6. **LICENSE**: MIT by default; attribution list for libraries, fonts and datasets.

---

## 15. Definition of done

- [ ] `python -m src.simulate`, `python -m src.train`, `python -m src.make_demo`, `streamlit run app.py` all run from a clean clone.
- [ ] Both data streams feed one model; the model is saved and used by the app without retraining.
- [ ] Real metrics saved and shown; robustness numbers present; limitations stated.
- [ ] Twin state persists in SQLite; before/after shown on every update.
- [ ] Five demo patients from the test split replay correctly; the deteriorating patient triggers an alert before the event day.
- [ ] Alerts with plain-language reasons, simulated notifications, acknowledge and escalation work.
- [ ] Doctor view exists and prints cleanly.
- [ ] UI meets section 12.11, offline.
- [ ] All tests pass.
- [ ] README, architecture PDF, draft presentation and video script exist with no invented numbers and no fake placeholders.
- [ ] Repo contains no secrets and no raw data the licence forbids.
