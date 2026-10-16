# PatientTwin AI — Prompts for Antigravity (agy)

Use these in order. Each prompt is self-contained and tells the agent to read `SPEC.md` first. Do one stage at a time, run it yourself, and commit before moving on.

---

## Before you start (10 minutes)

1. Create the project folder named `TeamName_CollegeName` (use your real team and college names) and open it in Antigravity.
2. Put `SPEC.md` and this file in the folder root. Run `git init` and make a first commit.
3. If your Antigravity version supports a rules file (such as `AGENTS.md`), save the **Guardrails block** below into it. If not, paste the block at the start of each stage.
4. Prefer the agent's planning mode (if your version has one) so you can read its plan before it writes code. Review the plan against `SPEC.md`; reject plans that add features not in the spec.
5. Download the Framingham CSV (search "Framingham heart study dataset"; check the licence on the page you use) and place it at `data/raw/framingham.csv`. If you cannot get it quickly, skip it; the spec has a fallback.
6. Work in a virtual environment. Never let the agent install packages globally.

### Guardrails block (paste as rules or at the top of each stage)

```
You are building the project described in SPEC.md. Read SPEC.md fully before doing anything.

Rules:
- Never invent results. Every number in docs, README or UI must come from files produced by running the code. If a result does not exist, write TBD.
- No data leakage: features at time t use only days <= t. Split by patient, never by row.
- Event labels come from our simulator; say so honestly. Never claim the model predicts heart attacks or is clinically validated.
- Do not retrain when new data arrives in the app. Run inference with the saved pipeline.
- Use fixed seeds from config.yaml. Pin package versions in requirements.txt.
- No real patient data, no real SMS/email sending, no secrets, no fake team names, licences or URLs. Use [FILL IN].
- Do not add features that are not in SPEC.md. If something is unclear, ask or propose a default and say so.
- Keep code small, readable, typed and documented. I must be able to explain every file.
- After each stage: run the code, show me the real output, list what you built, and stop. Do not start the next stage.
```

---

## Stage 0 — Project skeleton

```
Read SPEC.md fully. Stage 0 only.

Create the folder structure from section 5, a pinned requirements.txt, config.yaml (seed, n_patients=3000, n_days=120, event-rate parameters, alert thresholds from section 10.2, cooldown_days, ack_timeout, paths), src/config.py that loads it, .streamlit/config.toml with a light base theme matching the design tokens in section 12.2, a README.md skeleton with the headings from section 14 (all content [FILL IN] for now), an MIT LICENSE with [FILL IN] for the copyright holder, and a .gitignore (venv, __pycache__, twin.db, logs, data/raw if licence requires).

Create the virtual environment instructions, install dependencies, and run a trivial `pytest` that passes. Show me the tree and the output. Stop.
```

## Stage 1 — Baseline risk and synthetic cohort

```
Read SPEC.md sections 6 and 3. Stage 1 only.

Implement src/baseline_risk.py (Framingham-trained logistic regression if data/raw/framingham.csv exists, otherwise the documented fallback formula with a warning; inspect the real column names first and tell me what you found) and src/simulate.py exactly as in section 6: static profiles, daily wearable series with personal norms and AR(1) noise, real deterioration episodes with 3 to 5 day prodromes where each signal participates with probability ~0.7, benign look-alike episodes at ~1.5x the event frequency, recovery exclusion, ~3% missing values, symptom_flag on some event days, and the windows file with 72-hour labels and a by-patient split column.

Tune the event rate in config.yaml so 2%-6% of eligible windows are positive and print the real rate. Print: patients, days, events, benign episodes, windows, positives per split, and the Framingham baseline AUC if used. Save a few diagnostic plots (example normal week, example prodrome week) to docs/screenshots/. Write a test that the same seed gives the same data. Show me the real numbers. Stop.
```

## Stage 2 — Features and model

```
Read SPEC.md sections 7 and 8. Stage 2 only.

Implement src/features.py (single code path used by training and the app, explicit FEATURE_COLUMNS, no leakage, personal-norm features computed strictly before the window) and src/train.py: by-patient 70/15/15 split, the three baselines, logistic regression and a gradient-boosting model in sklearn Pipelines, calibration on validation, thresholds chosen on validation only and frozen in models/thresholds.json, a single test-set evaluation, lead-time analysis, the robustness re-runs (two extra seeds and 1.5x noise), permutation importance, plots, metrics.json, model_card.md.

Write tests/test_features_no_leakage.py and tests/test_split_by_patient.py. Run everything and show me metrics.json in full. Tell me honestly how easy the task is and whether the label looks too trivial (for example one feature separating the classes almost perfectly). If it is trivial, propose a fix to the simulator and stop. Do not tune on the test set.
```

## Stage 3 — Twin engine, replay and demo patients

```
Read SPEC.md section 9. Stage 3 only.

Implement src/db.py (schema as specified), src/twin.py (create_patient, append_reading with validation, refresh_state returning previous/current/change/tier/drivers/alerts, history), src/make_demo.py (five demo patients from the TEST split only, with fictional names, saved to data/processed/demo_patients.json) and src/replay.py (stream a patient's days through append_reading + refresh_state, with step/jump support). Tier and alert evaluation can be a stub that calls src/alerts.py once it exists in Stage 4; for now return the model probability and a probability-only tier.

Write tests/test_twin_update.py and tests/test_replay.py (replay equals batch scoring). Run a command-line replay of the deteriorating patient and print day, probability and tier. Show me the output. Stop.
```

## Stage 4 — Alerts and simulated notifications

```
Read SPEC.md section 10. Stage 4 only.

Implement src/alerts.py (tiers from model probability and thresholds.json, safety-net rules from config, own-baseline z-score check, plain-language reasons, cooldown, escalation with ack timeout, acknowledge) and src/notifier.py (contact and doctor message previews labelled SIMULATED — NOT SENT, written to the alerts table and logs/notifications.log; the Critical message includes the emergency-services line). Wire it into twin.refresh_state.

Write tests/test_alert_rules.py with boundary tests. Replay the deteriorating patient and show the first alert day, the tier, the reasons and both message previews, and how many days before the event it fired. Replay the steady and drifting patients and show how many alerts they produced (should be few). Stop.
```

## Stage 5 — UI foundation (design system only)

```
Read SPEC.md section 12 carefully, especially 12.1 to 12.5 and 12.10. Stage 5 only.

Build the design system, not the full screens: ui/styles.css with all tokens, the ECG-paper background, typography with bundled fonts and fallbacks, panel styles, tier badges, mono labels, hairline section headers, print styles; ui/theme.py with the Plotly template; ui/components.py with HTML builders for the top bar, hero risk panel (number, change chip, tier badge, risk ribbon, pulse-line SVG), patient row, signal-chart header, alert item and message bubble. Hide Streamlit's default header/footer/menu and override defaults in one commented block.

Create a temporary page `ui_preview.py` that shows every component in the steady, watch, urgent and critical states with sample values (clearly fake, labelled preview). Use the browser tool if available to take screenshots at 1440x900 and 1024x768 and save them to docs/screenshots/. Review the screenshots critically against SPEC section 12 (not default Streamlit, no emoji, no purple gradients, readable, tier not colour-only) and fix what you find. Show me the screenshots. Stop.
```

## Stage 6 — Full dashboard

```
Read SPEC.md section 12 again and review the Stage 5 components. Stage 6 only.

Build app.py and ui/views_ward.py: top bar with tabs, virtual ward left rail (five demo patients, status glyph, 7-day risk sparkline), hero panel, 3x2 signal-strip small multiples (14 days, own-norm band, model-window highlight, faint thresholds, direct labels, no legends), twin timeline with play/pause/step/speed and a then-vs-now toggle, the "Why this risk" driver bars, and the alert feed with the latest simulated message preview and Acknowledge. Add the manual-entry drawer. Live replay uses st.fragment(run_every=...) with st.session_state, with a step button fallback. Everything reads from the real twin database and model; no hard-coded demo numbers.

Test the three key flows by running them: (1) steady patient, (2) drifting patient triggers Watch, (3) deteriorating patient escalates to Critical, shows the banner and previews, and Acknowledge works. Take screenshots of each at 1440x900 and 1280x720 and save to docs/screenshots/. Fix any overlap, clipping or illegible text. Show me the screenshots and list anything that still looks generic. Stop.
```

## Stage 7 — Doctor view, About tab, what-if

```
Read SPEC.md sections 11 and 12.6 items 10 to 12. Stage 7 only.

Implement ui/views_doctor.py (one-page printable chart-style summary: patient header, static profile, baseline risk, current risk and trend, last 14 days of signals, alert history, top drivers; @media print styles), ui/views_about.py (data sources and licences, real vs synthetic, model description, real metrics read from models/metrics.json, limitations, not-medical-advice statement) and src/whatif.py with the what-if drawer (resting HR delta, sleep, steps, systolic) showing a ghost marker on the risk ribbon and the label "Model simulation — not advice".

Verify print output (browser print to PDF) looks clean. Save screenshots to docs/screenshots/. Show me them. Stop.
```

## Stage 8 — Documentation and exports

```
Read SPEC.md section 14. Stage 8 only.

Fill README.md from real outputs only (metrics from models/metrics.json, screenshots from docs/screenshots/). Leave team, college, licence holder, Framingham URL and video link as [FILL IN]. Write data/README.md, docs/make_architecture.py (generate docs/architecture.pdf and architecture.png), docs/presentation_outline.md, docs/make_presentation.py producing a first-draft docs/PatientTwin_presentation.pptx (about 18 to 22 slides, real numbers and real screenshots only), and docs/video_script.md (22 to 25 minutes with timings and exactly what to click during the demo).

List every claim in the README that depends on a number and the file it came from. Stop.
```

## Stage 9 — Independent audit

```
Act as a sceptical reviewer of this repository against SPEC.md. Do not fix anything yet; report only.

Check and report with evidence (file and line):
1. Any data leakage in features, splits, thresholds or demo patient selection (demo patients must be test-split only).
2. Any number in README, docs, slides or UI that is not traceable to a generated file.
3. Any place that claims to predict heart attacks or implies clinical validity.
4. Whether the model can be reproduced from a clean clone with the documented commands (run them in a fresh venv).
5. Whether any placeholder looks like real information (team names, links, licences).
6. Whether the offline demo works with the network disabled.
7. Whether every item in the SPEC section 15 checklist is true.

End with a prioritised fix list.
```

## Stage 10 — Fix, polish, rehearse

```
Fix the issues from the audit in priority order, re-run all tests, and re-run the full command sequence from a clean clone. Then do one UI polish pass: compare screenshots against SPEC section 12.11 and tighten spacing, alignment, hierarchy and copy. Finally, run through docs/video_script.md as a dry run: list each demo step, run it, and tell me any step that fails or looks weak. Show me the final screenshots and test output.
```

---

## Tips for working with the agent

- **Commit after every stage.** If a stage goes wrong, roll back instead of arguing with the agent.
- **If it adds things you did not ask for**, say: "Remove anything not in SPEC.md and explain what you removed."
- **If metrics look suspiciously perfect** (AUC above about 0.97), assume the simulator or features leak; ask the agent to show which features separate the classes.
- **If the UI still looks generic after Stage 6**, give it one specific critique at a time with a screenshot ("the risk number is not dominant enough", "the signal strip has too many gridlines") rather than "make it nicer".
- **Read the generated code** for `features.py`, `simulate.py` and `alerts.py` yourself. These are the files you will be asked about.
- **You record the video.** The agent can write the script but cannot do the recording or the voice-over.
- **Before submitting:** fill every `[FILL IN]`, confirm the Phase 1 deadline and format on the Unstop page, make the repo public, and open it in a private window to check that everything loads.
