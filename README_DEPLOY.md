# COURTSIDE V10.3 — Railway public-view deployment

This repository retains the original V10.3 model source and data and adds public read-only views, password-protected admin actions, and PostgreSQL persistence of model CSV artifacts.

## Deploy
1. Create a **private** GitHub repository, upload this project (including `app/`), and connect it to a Railway project.
2. Add a Railway **PostgreSQL** service; expose its `DATABASE_URL` to the web service using a Railway variable reference.
3. Generate a password hash locally with `python -c "from werkzeug.security import generate_password_hash; import getpass; print(generate_password_hash(getpass.getpass()))"` after installing requirements. Set `COURTSIDE_PASSWORD_HASH` to that result in Railway variables. Never set a plaintext password.
4. Set `COURTSIDE_SESSION_KEY` to a randomly generated 64-character hex secret (e.g. `python -c "import secrets;print(secrets.token_hex(32))"`).
5. Deploy and generate a Railway public domain under Networking. HTTPS is provided by Railway. Open `/` for public picks, `/login` for admin access.
6. Change admin password by replacing `COURTSIDE_PASSWORD_HASH` in Railway variables and redeploying. Existing sessions remain active until session secret rotates; rotate `COURTSIDE_SESSION_KEY` to sign everyone out.

## Important limitations
- This is a deployable **candidate**, not a live deployment. Railway and external API connectivity must be tested in your own account.
- Original V10.3 desktop code remains unchanged. The browser's public tabs are Dashboard, Best Props, High Probability, and All Scored Props. Analyze a Prop and Data & Performance are admin-only.
- Database stores CSV snapshots (including historical data), not a fully normalized prediction schema. CSV files are seeded from bundled data on first boot and restored from PostgreSQL on subsequent boots. After admin refreshes, CSV artifacts are saved back to PostgreSQL. Do not run more than one web replica or background refresh at once; overlapping writes may overwrite updates.
- On Railway's ephemeral filesystem, files not in `persistence.FILES` do not persist. Admin refresh is asynchronous and may be interrupted by redeploys. Production hardening should add durable job queues, monitoring, rate limiting, CSRF protection, and a normalized relational schema.
- Public player projections may include stale historical data; only dated today props appear in the daily ranking. No claims of live availability without an actual successful feed refresh.
- Verify third-party API licenses allow public redistribution before enabling public-facing predictions based on their feeds.
- Do not expose `DATABASE_URL`, API tokens, or the original source repository publicly.
