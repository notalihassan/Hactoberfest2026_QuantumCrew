# 🥬 FoodMatch: Urban Food Desert Supply Chain Matchmaker

> AI-assisted marketplace that connects urban farms and community gardens with corner stores in food deserts, with verifiable onchain trust.

![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)
![Hacktoberfest 2026](https://img.shields.io/badge/Hacktoberfest-2026-blueviolet)

Built for **Hacktoberfest 2026 Hack Day, Challenge 3**.

---

## Table of Contents

1. [The Problem](#the-problem)
2. [Our Solution](#our-solution)
3. [Sponsor Tracks at a Glance](#sponsor-tracks-at-a-glance)
4. [Architecture](#architecture)
5. [Tech Stack](#tech-stack)
6. [Gemma 4 Integration (Gemini API)](#gemma-4-integration-gemini-api)
7. [Snowflake and CoCo](#snowflake-and-coco)
8. [Solana Agent Registry](#solana-agent-registry)
9. [GitHub Copilot Usage](#github-copilot-usage)
10. [Getting Started](#getting-started)
11. [Environment Variables](#environment-variables)
12. [API Reference](#api-reference)
13. [Project Structure](#project-structure)
14. [Testing](#testing)
15. [Deployment](#deployment)
16. [Demo Script](#demo-script)
17. [Contributing](#contributing)
18. [License](#license)

---

## The Problem

In low-income urban food deserts, corner stores want to sell fresh fruit and vegetables, but commercial suppliers skip them because order volumes are small. At the same time, community gardens and urban farms throw away large amounts of surplus produce because they have no sales channel and no spare hours to find nearby buyers.

## Our Solution

FoodMatch closes that gap with four ideas:

- **AI listings:** a farmer uploads a photo; the model checks quality, grades it, and writes the listing automatically.
- **Demand prediction:** Snowflake data shows which neighborhoods face the biggest fresh food deficit on which days of the week.
- **Smart matching:** listings are ranked against stores by neighborhood deficit and distance.
- **Trust layer:** buyer and seller agents have onchain identities on Solana, and every completed deal updates a portable reputation score.

## Sponsor Tracks at a Glance

| Track | How we use it | Where to look |
|---|---|---|
| Google Gemma 4 | Produce image analysis, auto descriptions, conversational price bargaining | `backend/market/services/ai_service.py` |
| Snowflake CoCo | SQL for predicting weekly fresh food deficit per neighborhood | `sql/`, `backend/market/services/demand_service.py` |
| Solana Agent Registry | Agent identity and onchain reputation and feedback | `backend/market/services/registry_service.py` |
| GitHub Copilot | Boilerplate, mock data, routing helpers, debugging | [Copilot section](#github-copilot-usage) |

## Architecture

```mermaid
flowchart LR
    subgraph Client
        UI[React SPA<br/>Farmer · Store · Map · Chat]
    end

    subgraph Backend[Django REST API]
        API[Views and Serializers]
        AI[ai_service]
        DEM[demand_service]
        MAT[matching_service]
        REG[registry_service]
        DB[(SQLite / Postgres)]
    end

    UI --> API
    API --> AI
    API --> DEM
    API --> MAT
    API --> REG
    API --> DB

    AI --> GEM[Gemini API<br/>Gemma 4]
    DEM --> SNOW[Snowflake<br/>CoCo SQL]
    REG --> SOL[Solana Devnet<br/>Agent Registry]
```

**Core flow**

1. Farmer uploads a produce photo.
2. Gemma 4 analyzes it and returns a structured listing (item, grade, description, suggested price).
3. The listing is saved under the farmer's verified agent identity.
4. The demand service scores nearby neighborhoods using Snowflake data.
5. The matching service ranks stores by deficit and distance.
6. Buyer and seller agents negotiate in chat.
7. On acceptance, feedback is recorded on Solana and the trust badge updates.

## Tech Stack

| Layer | Technology |
|---|---|
| Frontend | React, Vite, Tailwind CSS, React Router, Axios, Leaflet |
| Backend | Python 3.12, Django, Django REST Framework |
| AI | Gemma 4 through the Gemini API (`google-genai`) |
| Data | Snowflake, SQLite (dev) / Postgres (prod) |
| Blockchain | Solana devnet, Agent Registry |
| DevOps | Docker, docker-compose, nginx, AWS EC2 |

## Gemma 4 Integration (Gemini API)

Gemma 4 is called through the Gemini API using the `google-genai` SDK. The exact model ID is read from the `GEMMA_MODEL_ID` environment variable.

- **Model used:** `<FILL IN: exact Gemma 4 model ID>`
- **Image analysis:** produce photo in, strict JSON out (`item`, `grade`, `freshness_notes`, `description`, `suggested_price_per_kg`). Output is validated before it is saved.
- **Negotiation agent:** the seller agent replies to buyer offers with a floor price guardrail.
- **Code location:** `backend/market/services/ai_service.py`

**Sample request flow**

```
Photo upload → ai_service.analyze_produce() → Gemini API (Gemma 4)
            → JSON validation → Listing saved → Response to UI
```

> 📸 Add a screenshot of the AI result card here: `docs/screenshots/ai-result.png`

## Snowflake and CoCo

We use a free dataset from the Snowflake Marketplace to estimate the fresh food deficit per neighborhood and per day of the week.

- **Dataset:** `<FILL IN: dataset name and provider>`
- **SQL:** `sql/demand.sql`, written with the help of CoCo
- **Service:** `backend/market/services/demand_service.py` runs the query and returns `[{day, score}]`. A cached fallback keeps the demo working if Snowflake is unreachable.

**CoCo prompts used**

```
<FILL IN: the exact prompt(s) you gave CoCo>
```

> 📸 Add a screenshot of CoCo and the query result here: `docs/screenshots/coco.png`

## Solana Agent Registry

Every buyer agent and seller agent has a verifiable onchain identity. After each accepted deal, feedback is submitted so reputation is portable across the marketplace.

All Solana logic lives in one adapter, `registry_service.py`, with three functions: `register_agent`, `submit_feedback`, `get_reputation`.

| Agent | Role | Address | Registration tx |
|---|---|---|---|
| `<name>` | Seller | `<address>` | [`<signature>`](https://explorer.solana.com/tx/SIGNATURE?cluster=devnet) |
| `<name>` | Buyer | `<address>` | [`<signature>`](https://explorer.solana.com/tx/SIGNATURE?cluster=devnet) |

**Feedback transaction example:** `<FILL IN: signature link>`

## GitHub Copilot Usage

Document where Copilot helped. Judges look for specific prompts and concrete outcomes.

| # | Blocker or task | Exact Copilot prompt | Result |
|---|---|---|---|
| 1 | `<e.g. spatial route boilerplate>` | `<paste prompt>` | `<what it produced / what you changed>` |
| 2 | `<e.g. mock inventory data>` | `<paste prompt>` | `<result>` |
| 3 | `<e.g. debugging an error>` | `<paste prompt>` | `<result>` |

> 📸 Add screenshots or chat logs here: `docs/screenshots/copilot-*.png`

## Getting Started

### Prerequisites

- Python 3.12+
- Node.js 20+
- Docker and docker-compose (optional but recommended)
- Accounts: Google AI Studio (Gemini API key), Snowflake, a funded Solana devnet wallet

### Option A: Docker

```bash
git clone https://github.com/<your-username>/foodmatch.git
cd foodmatch
cp .env.example .env        # then fill in your keys
docker compose up --build
```

- Frontend: http://localhost:5173
- API: http://localhost:8000/api

### Option B: Local development

**Backend**

```bash
cd backend
python -m venv venv
source venv/bin/activate          # Windows: venv\Scripts\activate
pip install -r requirements.txt
cp ../.env.example ../.env        # fill in your keys
python manage.py migrate
python manage.py seed_demo        # creates demo farmers, stores, listings
python manage.py runserver
```

**Frontend**

```bash
cd frontend
npm install
npm run dev
```

## Environment Variables

Copy `.env.example` to `.env` and fill in the values. **Never commit `.env`.**

| Variable | Description |
|---|---|
| `DJANGO_SECRET_KEY` | Django secret key |
| `DEBUG` | `True` for development, `False` in production |
| `ALLOWED_HOSTS` | Comma-separated hosts |
| `CORS_ALLOWED_ORIGINS` | Frontend origin(s) |
| `GEMINI_API_KEY` | Google AI Studio API key |
| `GEMMA_MODEL_ID` | Exact Gemma 4 model ID |
| `SF_ACCOUNT`, `SF_USER`, `SF_PASSWORD`, `SF_WAREHOUSE`, `SF_DATABASE`, `SF_SCHEMA` | Snowflake connection |
| `SOLANA_RPC_URL` | Solana RPC (devnet) |
| `SOLANA_PAYER_KEYPAIR` | Funded devnet wallet key |
| `SOLANA_REGISTRY_PROGRAM_ID` | Agent Registry program ID |
| `MATCH_WEIGHT_DEFICIT`, `MATCH_WEIGHT_DISTANCE` | Matching score weights |

## API Reference

Base path: `/api`

| Method | Endpoint | Description |
|---|---|---|
| `POST` | `/listings/` | Upload photo and quantity; returns AI-generated listing |
| `GET` | `/listings/` | List all listings |
| `GET` | `/demand/?zip=` | Weekly deficit scores for a ZIP code |
| `GET` | `/matches/<listing_id>/` | Ranked stores for a listing |
| `POST` | `/chat/<deal_id>/` | Send a message; returns the seller agent reply |
| `POST` | `/deals/<id>/accept/` | Accept a deal; returns the feedback transaction |
| `POST` | `/agents/register/` | Register an agent onchain |
| `GET` | `/health/` | Health check |

## Project Structure

```
foodmatch/
├── backend/
│   ├── core/                  # Django project settings
│   ├── market/
│   │   ├── models.py          # Agent, Listing, Deal, ChatMessage
│   │   ├── serializers.py
│   │   ├── views.py
│   │   └── services/
│   │       ├── ai_service.py
│   │       ├── demand_service.py
│   │       ├── matching_service.py
│   │       └── registry_service.py
│   ├── requirements.txt
│   └── manage.py
├── frontend/
│   └── src/
│       ├── pages/             # FarmerUpload, StoreDashboard, MapView, Chat
│       ├── components/        # TrustBadge, ...
│       └── api.js
├── sql/                       # Snowflake / CoCo queries
├── docs/                      # Screenshots and extra docs
├── docker-compose.yml
├── .env.example
├── LICENSE
└── README.md
```

## Testing

```bash
cd backend
pytest
```

Tests cover the matching logic, the AI response parser, and the API endpoints. External services (Gemini, Snowflake, Solana) are mocked.

## Deployment

The target is a single AWS EC2 instance running docker-compose, with nginx serving the React build and proxying `/api` to Django (gunicorn). See [`DEPLOY.md`](DEPLOY.md) for step by step instructions.

## Demo Script

About 3 minutes:

1. Farmer uploads a produce photo and the AI grades it and writes the listing.
2. Open the store dashboard and show the weekly deficit chart.
3. Show the matched listings and the map.
4. Negotiate the price in chat.
5. Accept the deal and open the **Solana Explorer** link to show the onchain feedback.

> 🎥 Demo video: `<FILL IN: link>`
> 🌐 Live demo: `<FILL IN: link>`

## Contributing

Contributions are welcome.

1. Fork the repo
2. Create a branch: `git checkout -b feature/your-feature`
3. Commit your changes with a clear message
4. Open a pull request

## License

Released under the [MIT License](LICENSE).
