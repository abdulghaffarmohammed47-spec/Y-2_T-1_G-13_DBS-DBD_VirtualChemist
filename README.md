# The Virtual Chemist

A FastAPI chemistry game with PostgreSQL-backed molecule discoveries, Redis-backed sessions and OTPs, and an interactive browser frontend.

## Run locally

1. Create a virtual environment and install dependencies with `pip install -r requirements.txt`.
2. Set `DATABASE_URL` and `REDIS_URL` in a local `.env` file.
3. Configure SMTP with `SMTP_SERVER`, `SMTP_PORT`, `SMTP_USER`, and `SMTP_PASS` to deliver login codes. For local-only development, `OTP_DEV_MODE=true` returns the code in the login response.
4. Start the server with `uvicorn app.main:app --reload`.

The app is served at `/`; the API is available under `/api`.

## Deploy to Render

The included `render.yaml` provisions a free web service, PostgreSQL database, and Redis-compatible Key Value instance. In Render, create a Blueprint from this repository and provide `SMTP_SERVER`, `SMTP_PORT`, `SMTP_USER`, and `SMTP_PASS` when prompted. Keep `OTP_DEV_MODE` disabled in deployment. Render's free web instances block outbound SMTP on ports 25, 465, and 587, so use an SMTP provider/port that Render allows (commonly 2525), or choose a paid web-service plan.

Free Render web services spin down when idle, free Postgres databases expire after 30 days, and free Key Value data is not persistent. These free resources are suitable for demos, not production.