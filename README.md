<div align="center">

# The Virtual Chemist

**A browser-based chemistry game for discovering molecules by combining atoms.**

[![Live demo](https://img.shields.io/badge/LIVE%20DEMO-Play%20now-1d1b2f?style=for-the-badge&logo=render&logoColor=white)](https://virtual-chemist.onrender.com)
[![API docs](https://img.shields.io/badge/API%20DOCS-Explore-7257d5?style=for-the-badge&logo=swagger&logoColor=white)](https://virtual-chemist.onrender.com/docs)
[![Python](https://img.shields.io/badge/Python-3.12%2B-3776ab?style=for-the-badge&logo=python&logoColor=white)](https://www.python.org/)
[![FastAPI](https://img.shields.io/badge/FastAPI-0.115-009688?style=for-the-badge&logo=fastapi&logoColor=white)](https://fastapi.tiangolo.com/)

**Deployed website:** [https://virtual-chemist.onrender.com](https://virtual-chemist.onrender.com)

<img src="./docs/virtual-chemist-banner.svg" alt="Pastel atom orbits around a molecule illustration with the message: Connect. Combine. Discover molecules." width="100%">

</div>

Virtual Chemist pairs an interactive chemistry lab with a FastAPI backend. Combine atoms to discover compounds, save discoveries to your profile, and search the molecule collection using natural-language queries.

## Explore the lab

| 🧪 **Experiment** | 🔎 **Discover** | 🧑‍🔬 **Keep exploring** |
| --- | --- | --- |
| Drag atoms together with mouse or touch, or try webcam hand control. | Match combinations to molecule recipes and search descriptions with vector similarity. | Create an account, track discoveries, and get hints for your next experiment. |

Accounts use passwords and one-time email codes. PostgreSQL stores users, molecules, and discoveries; pgvector powers molecule search; Redis-compatible storage handles OTPs and sessions.

## How it works

```mermaid
flowchart LR
    Browser["Browser UI<br/>HTML · CSS · JavaScript"] -->|HTTP + JSON| API["FastAPI"]
    API --> DB[("PostgreSQL<br/>+ pgvector")]
    API --> Cache[("Redis / Valkey<br/>OTP + sessions")]
    classDef client fill:#fff,stroke:#1d1b2f,stroke-width:2px,color:#1d1b2f
    classDef service fill:#ffd35c,stroke:#1d1b2f,stroke-width:2px,color:#1d1b2f
    classDef storage fill:#b9a2ff,stroke:#1d1b2f,stroke-width:2px,color:#1d1b2f
    class Browser client
    class API service
    class DB,Cache storage
```

The app creates its database tables and views at startup, enables the PostgreSQL `vector` extension, and seeds the molecule collection.

## Technology

| Area | Tools |
| --- | --- |
| Frontend | HTML, CSS, JavaScript, Canvas |
| Backend | Python 3.12+, FastAPI, Uvicorn |
| Relational database | PostgreSQL, SQLAlchemy |
| Vector search | pgvector, 384-dimensional hashed text embeddings |
| Sessions and OTPs | Redis-compatible Redis / Valkey |
| Request validation | Pydantic |

## Run locally

### Prerequisites

- Python 3.12 or newer
- PostgreSQL with the `vector` extension available
- Redis or a Redis-compatible service

### Setup

1. Clone the repository and enter its directory.
2. Create and activate a virtual environment:

   ```powershell
   python -m venv .venv
   .\.venv\Scripts\Activate.ps1
   ```

   On macOS or Linux, activate it with:

   ```bash
   source .venv/bin/activate
   ```

3. Install dependencies:

   ```bash
   pip install -r requirements.txt
   ```

4. Create a `.env` file in the project root and configure the local services:

   ```dotenv
   DATABASE_URL=postgresql://postgres:postgres@localhost:5432/Chemist
   REDIS_URL=redis://localhost:6379/0
   OTP_DEV_MODE=true
   ```

5. Start the server:

   ```bash
   uvicorn app.main:app --reload
   ```

Open [http://localhost:8000](http://localhost:8000) to use the app. Interactive API documentation is available at [http://localhost:8000/docs](http://localhost:8000/docs).

With `OTP_DEV_MODE=true`, the login response includes the one-time code for local development. Do not enable this setting in a public deployment.

## API overview

All API routes are under `/api`. Routes marked **Bearer token** require the session token returned after OTP verification.

| Method | Route | Purpose | Authentication |
| --- | --- | --- | --- |
| `POST` | `/api/auth/register` | Create an account | None |
| `POST` | `/api/auth/login` | Check credentials and send a one-time code | None |
| `POST` | `/api/auth/verify-otp` | Verify the code and create a session | None |
| `GET` | `/api/auth/me` | Get the current account | Bearer token |
| `POST` | `/api/auth/logout` | End the current session | Bearer token |
| `POST` | `/api/chemist/combine` | Evaluate an atom combination | Bearer token |
| `GET` | `/api/chemist/profile` | Get the account's discoveries | Bearer token |
| `GET` | `/api/chemist/search?q=...` | Search molecules | Bearer token |
| `GET` | `/api/chemist/hint` | Get a discovery hint | Bearer token |

For request schemas and interactive testing, see the [OpenAPI documentation](https://virtual-chemist.onrender.com/docs).

## Deploy to Render

The root-level [`render.yaml`](./render.yaml) defines a Render Blueprint for the web service, PostgreSQL database, and Redis-compatible Key Value instance.

1. In Render, choose **New > Blueprint** and connect this repository's `main` branch.
2. Review the Blueprint resources and provide the requested SMTP settings.
3. Deploy the Blueprint.

The deployed app must use a real SMTP provider so that sign-in codes can be delivered. Set `SMTP_SERVER`, `SMTP_PORT`, `SMTP_USER`, and `SMTP_PASS` in Render; never commit secrets. Render's free web instances block outbound SMTP on ports 25, 465, and 587, so use a provider and port supported by Render (often 2525), or select a suitable paid web-service plan. `OTP_DEV_MODE` is disabled by the Blueprint.

**Free-tier limitations:** the web service may spin down when idle, free Postgres databases expire after 30 days, and free Key Value data is not persistent. These resources are intended for demos and experiments, not production workloads.

## Project layout

```text
app/
  models/       Database models
  repositories/ Database queries
  routers/      HTTP API routes
  schemas/      Request and response models
  services/     Authentication, chemistry, email, and search logic
  static/       Browser application
  database.py   PostgreSQL setup and views
  main.py       FastAPI application
render.yaml     Render Blueprint
requirements.txt
```

## Security notes

- Keep `.env` and all credentials private; `.env` is excluded by `.gitignore`.
- Set `OTP_DEV_MODE=false` for any public deployment.
- Configure SMTP secrets through the hosting provider's environment settings.
- Use managed PostgreSQL and Redis-compatible services for deployments; do not rely on local development defaults.
