# COURTSIDE V11 Hybrid API — Railway deployment candidate

This package **preserves the original V10.3 source and website** and adds versioned V11 endpoints, normalized PostgreSQL prediction tables, a provider registry, and an **experimental** conservative ensemble. The public website continues to rank with the **unchanged V10.3 baseline**, until a V11 candidate proves superior on held-out games. No fabricated improvements or live results.

## Deploy
1. Create a **private GitHub repository** and upload this folder's contents, not the ZIP itself. Never commit credentials.
2. Create a Railway project with a PostgreSQL service and a web service from the repo. Set the web service **root directory to `/`**, and set `DATABASE_URL` to the Postgres service's connection URL.
3. Set `COURTSIDE_SESSION_KEY` to a long random value and `COURTSIDE_PASSWORD_HASH` to a Werkzeug password hash. Generate both locally with Python; do not place the plain password in Railway variables. Optional: `BALLDONTLIE_API_KEY`, `ODDS_API_KEY`. Premium provider availability does **not** mean it is fully integrated.
4. Deploy; Railway's Procfile starts gunicorn. Add a Railway-generated HTTPS domain. Verify `/health`, `/api/v11/status`, and `/api/v11/best-props`.
5. Log in as admin to use Analyze a Prop, Data & Performance, and refresh actions. Test a refresh and a subsequent redeploy. The current refresh job is in-process and should later be moved to a dedicated Railway worker or scheduler.

## API
- `GET /api/v11/status` public: model version and provider configuration (no keys)
- `GET /api/v11/best-props?limit=10` public: current-date ranked V10.3 probabilities; no threshold
- `GET /api/v11/projections?player=...` public: existing V10.3 projections
- `GET /api/v11/performance` public: only settled, stored V11 records; may be empty
- `POST /api/v11/candidate` admin: experimental ensemble alongside original V10.3 result; never silently replaces rankings

## Accuracy and data provenance
The supplied historical `backtest_results.csv` contains continuous PRA predictions and actuals, not timestamped sportsbook prop probabilities. It **cannot** establish an improved prop hit rate or ROI. Before promoting V11: collect time-stamped pregame lines and predictions, settle outcomes, avoid leakage, use rolling walk-forward splits, compare Brier/log loss and calibration against V10.3, and test betting ROI after vig. Do not interpret empty current-date slate as a bug if source lines are unavailable.

## Known limitations / security
- `v11_predictions` and `v11_provider_runs` schemas are ready, but ingestion and automatic settlement into normalized tables are **not yet wired**. Existing V10.3 CSV snapshot persistence remains in use.
- Premium BALLDONTLIE adapter is a **configuration scaffold only**, not a functioning live integration.
- Password changes require updating `COURTSIDE_PASSWORD_HASH` and redeploying; no self-service password change yet.
- The website is public-view, with admin-only write endpoints; use HTTPS and enable rate limiting, CSRF tokens, and proper user management before a broad production launch. Current Origin check is not a complete CSRF defense.
- Large model CSVs are bundled for compatibility and should be migrated to a normalized database/worker pipeline later.
- Local integration tests require Railway/PostgreSQL credentials. Syntax and unit checks alone are not a live deployment test.
