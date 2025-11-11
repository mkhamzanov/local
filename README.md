# local

[![Repo size](https://img.shields.io/github/repo-size/mkhamzanov/local)](https://github.com/mkhamzanov/local)
[![Last commit](https://img.shields.io/github/last-commit/mkhamzanov/local)](https://github.com/mkhamzanov/local/commits)
[![Top language](https://img.shields.io/github/languages/top/mkhamzanov/local)](https://github.com/mkhamzanov/local)

English first, Russian below.

--- English — summary and navigation

Purpose
This repository contains notebooks and utilities for LTV (Lifetime Value) analysis and an experimental metric notebook (Kolmogorov/Khamzanov). The goal is: reproducible data generation, aggregation (ARPU/LTV), modeling/forecasting, and visualizations. Notebooks hold the experiments; scripts provide reusable code.

Quick repo layout (recommended)
- notebooks/
  - ltv_prediction.ipynb — full pipeline (generate synthetic payments → aggregate → compute ARPU/LTV → models + plots)
  - kolmogorov_khamzanov_metric.ipynb — experimental metric / larger notebook (see docs/)
- scripts/
  - ltv/
    - ltv_utils.py — generator + transform utilities (reusable functions)
    - ltv_forecast.py — runnable script to run the full forecast flow and save plot
- docs/
  - REFERENCE_LTV.md — reference for LTV functions and expected inputs/outputs
  - (future) REFERENCE_KOLMOGOROV.md — doc for Kolmogorov/Khamzanov notebook (to be added after notebook review)
- assets/plots/
  - ltv_forecast.png — example output (keeps binary artifacts out of notebooks)
- README.md — this file
- LICENSE — (if present) license file

Why this structure
- Keep notebooks for exploration / figures.
- Keep reusable code as small scripts/modules under scripts/ so they can be imported in notebooks and CI/tests.
- Move generated figures to assets/plots/ to track visuals without bloating notebooks.

Detailed navigation & how-to

1) Run the full forecast (recommended)
- Option A — from notebook:
  - Open notebooks/ltv_prediction.ipynb and run cells top-to-bottom.
- Option B — programmatically (preferred for reproducibility):
  - Use scripts/ltv/ltv_forecast.py which imports scripts/ltv/ltv_utils.py and:
    - generates synthetic payments (or reads your dataset)
    - aggregates → computes ARPU and LTV
    - fits polynomial/log model and an alternative power fit
    - saves the figure to assets/plots/ltv_forecast.png

Example (local):
- Ensure Python 3.8+ and these packages installed: numpy, pandas, scikit-learn, scipy, matplotlib, plotly (optional)
- Run:
  python -m pip install -r requirements.txt  # if you add requirements
  python scripts/ltv/ltv_forecast.py

2) Files and scripts (detailed)
- scripts/ltv/ltv_utils.py
  - generate_days_since_reg(num_payments, horizon_days=365)
    - returns sorted days offsets with exponential decay sampling (numpy array)
    - input: integer num_payments
  - generate_synthetic_payments(num_users=1000, date_from="2023-06-01", date_to="2024-01-01", amount_min=1.0, amount_max=4.0)
    - returns pandas.DataFrame with columns: user_id, reg_date, payment_date, days_since_reg, amount_in_usd
    - used to reproduce the synthetic dataset present in the notebook
  - transform_data(df)
    - aggregates by days_since_reg, computes amount_in_usd (sum), user_count (nunique), ARPU, cumulative LTV, sigma (1/sqrt(user_count))
    - returns aggregated DataFrame for modeling/visualization

- scripts/ltv/ltv_forecast.py
  - run_forecast(df, save_path="assets/plots/ltv_forecast.png", max_days=365)
    - calls transform_data → fits LinearRegression on PolynomialFeatures(log1p(days)) → tries curve_fit on power function a*x^b + c → plots observed LTV + predictions and saves the plot
  - main runs generate_synthetic_payments() then run_forecast()

- notebooks/ltv_prediction.ipynb
  - notebook flow: imports → data generation (synthetic) → transform_data → model fitting + curve_fit → matplotlib + plotly visualizations → example saved image
  - Recommended: convert notebook logic to scripts/ltv when you want reusable code (we provide ltv_utils.py/ltv_forecast.py example)

- notebooks/kolmogorov_khamzanov_metric.ipynb
  - experimental metric notebook. It is large — docs/REFERENCE_KOLMOGOROV.md will be created after a full review to document functions, parameters, and intended use.

3) How to contribute
- Small PRs for:
  - moving notebooks → notebooks/
  - adding scripts → scripts/ltv/
  - adding docs → docs/
  - adding tests/ → tests/
- Prefer: one logical change per PR, include tests or a short example invocation.

4) Tests & CI (suggestions)
- Add minimal unit tests for:
  - generate_days_since_reg: deterministic behavior with seeded RNG
  - transform_data: given a tiny DataFrame, aggregated outputs match expected numbers
- Add GitHub Actions workflow in .github/workflows/ to run pytest and linting.

5) Files I propose to add in this PR
- README.md (this file)
- docs/REFERENCE_LTV.md — short reference for the LTV utilities
- scripts/ltv/ltv_utils.py — utilities
- scripts/ltv/ltv_forecast.py — runnable script
(File contents are shown below — copy them into the branch readme-scripts-docs)

6) Next steps (how to push and open PR)
- Commands to run locally:
  git checkout -b readme-scripts-docs
  # add files as shown in this README
  git add README.md scripts/ltv/ltv_utils.py scripts/ltv/ltv_forecast.py docs/REFERENCE_LTV.md
  git commit -m "Add README, LTV scripts and reference docs; reorganize for reproducibility"
  git push origin readme-scripts-docs
- Open PR:
  - Title: "Add README, LTV scripts and docs; reorganize repo"
  - Body: see suggested PR description at the bottom of this README (or paste the PR body provided in the assistant response)
  - Use GitHub UI or CLI: gh pr create --title "..." --body "..."

--- Русский — краткая инструкция и навигация

Назначение
Репозиторий содержит ноутбуки и утилиты для анализа LTV и экспериментальную реализацию метрик. Цель — воспроизводимая генерация данных, агрегация (ARPU/LTV), моделирование/прогноз и визуализация.

Структура (подробно)
- notebooks/ — ноутбуки: ltv_prediction.ipynb, kolmogorov_khamzanov_metric.ipynb
- scripts/ — модули и скрипты:
  - scripts/ltv/ltv_utils.py — генерация и трансформация данных
  - scripts/ltv/ltv_forecast.py — запуск пайплайна и сохранение графика
- docs/ — справочники: REFERENCE_LTV.md, (позже) REFERENCE_KOLMOGOROV.md
- assets/plots/ — сохранённые графики

Как запустить (коротко)
1. Установите зависимости: numpy,pandas,scikit-learn,scipy,matplotlib
2. Запустите:
   python scripts/ltv/ltv_forecast.py
   — результат будет сохранён как assets/plots/ltv_forecast.png

Контрибьютинг
- Делайте маленькие PR'ы с фокусом.
- Добавляйте тесты для утилит и примеры запуска.

PR (готовая информация)
- Title: "Add README, LTV scripts and docs; reorganize repo"
- Body (short): "Add a human-friendly bilingual README, reusable LTV scripts (generate/transform/forecast), and reference docs. Move towards a clearer notebooks+scripts layout and add assets/plots for outputs."

If you want — I will open the PR for you after you confirm. If you want me to open it immediately, reply: "Open PR now" and I will proceed to create commits on branch readme-scripts-docs and open the PR.
