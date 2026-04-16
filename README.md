# Bridge
Bridge -> Weighted multi-factor matching engine pairing isolated elderly immigrants with multilingual young volunteers. Python · FastAPI · PostgreSQL · React.
# Bridge

**Language-first matching engine connecting elderly immigrants with multilingual young volunteers.**

Bridge pairs isolated elderly immigrants with young volunteers who speak their language — not just people who live nearby. A Slovene grandmother in Queens, a German-speaking grandfather in Yorkville, a French-speaking widow on the Upper West Side: each can find a young person who shares their mother tongue and wants to spend time with them in person.

The product is a weighted multi-factor matching engine with a coordinator-facing web interface. Pilot partner: [Dorot](https://www.dorotusa.org/) NYC.

---

## Why this exists

Chronic social isolation among older adults carries health risks comparable to smoking 15 cigarettes a day. For elderly immigrants, that isolation is compounded by a language barrier that cuts them off from most English-speaking volunteer networks. Existing platforms match on location and availability. They don't treat language fluency as a first-class signal.

Multilingual young people — students, second-generation immigrants, language learners — are an underused supply side for this problem. Bridge exists to connect the two.

## The thesis

A weighted multi-factor matching engine that treats language fluency as a first-class signal will produce measurably stronger intergenerational connections than proximity-based matching alone. Specifically:

- Higher repeat-pairing rates
- Longer average connection duration
- Greater reduction in UCLA Loneliness Scale scores over time

The pilot is designed to generate the data to test this.

---

## How matching works

Each potential pairing receives a composite score in `[0, 1]` computed from six factors:

| Factor              | Weight | What it measures                                         |
|---------------------|--------|----------------------------------------------------------|
| Language overlap    | 0.35   | Best shared language, scored by minimum fluency level    |
| Availability match  | 0.25   | Whether volunteer's recurring schedule covers the request|
| Repeat-pairing bias | 0.20   | History of successful past connections between the pair  |
| Geographic proximity| 0.10   | Distance between neighborhood centroids                  |
| Activity fit        | 0.05   | Requested activity vs. volunteer preferences             |
| Feedback history    | 0.05   | Volunteer's average rating across past connections       |

Every match persists its full per-factor breakdown as JSONB. Coordinators can see exactly why a pairing was proposed, and the data becomes the foundation for empirically tuning weights as the pilot runs.

See [`CLAUDE.md`](./CLAUDE.md) §4 for the full algorithm specification.

---

## Stack

| Layer       | Choice                                  |
|-------------|-----------------------------------------|
| Backend     | Python 3.11, FastAPI, SQLAlchemy 2.0    |
| Database    | PostgreSQL 15                           |
| Migrations  | Alembic                                 |
| Frontend    | React 18, Vite, TypeScript              |
| UI          | Tailwind CSS, shadcn/ui                 |
| Testing     | pytest, Vitest                          |
| Type checks | mypy --strict, tsc                      |
| Dev env     | Docker Compose                          |

---

## Getting started

**Prerequisites:** Docker, Python 3.11+, Node 20+.

```bash
# Clone and enter
git clone https://github.com/<you>/bridge.git
cd bridge

# Environment
cp .env.example .env

# Start Postgres
docker compose up -d

# Backend
cd backend
pip install -e ".[dev]"
alembic upgrade head
python -m app.seed          # load fixtures
uvicorn app.main:app --reload

# Frontend (separate terminal)
cd frontend
npm install
npm run dev
```

API docs live at `http://localhost:8000/docs`. The coordinator UI runs at `http://localhost:5173`.

---

## Project layout

```
bridge/
├── CLAUDE.md              # Project spec and engineering conventions
├── backend/
│   ├── app/
│   │   ├── api/           # FastAPI routers
│   │   ├── services/      # Business logic
│   │   │   └── matching/  # Core algorithm
│   │   ├── repositories/  # Database queries
│   │   ├── models/        # SQLAlchemy models
│   │   └── schemas/       # Pydantic request/response types
│   └── tests/
└── frontend/
    └── src/
        ├── api/           # Generated from OpenAPI
        ├── pages/
        └── components/
```

Layering rule: `api/ → services/ → repositories/ → models/`. Never skip layers, never reach upward. See [`CLAUDE.md`](./CLAUDE.md) §2 for detail.

---

## Roadmap

**Phase 1 — MVP (Fall 2026)**
Coordinator-mediated web tool. Dorot staff enters elderly requests; the system ranks volunteer matches; coordinator makes the introduction offline.

**Phase 2 — Pilot (Fall/Winter 2026–27)**
15–20 elderly participants, 15–20 Collegiate School student volunteers. Instrumented with UCLA Loneliness Scale, connection frequency, and repeat-pairing metrics.

**Phase 3 — Analysis**
Compare language-matched vs. non-matched pairs on retention and loneliness outcomes. Seek academic collaboration (Columbia or NYU: psychology, gerontology, computational social science).

**Phase 4 — Expansion**
Other NYC community organizations serving immigrant elderly populations. Later, other cities with high elderly immigrant density.

Explicit non-goals for v1: public marketplace, consumer mobile apps, in-app messaging, AI companionship features. Bridge facilitates human-to-human contact; it does not replace it.

---

## Privacy

Elderly users' data is subject to stricter handling than the volunteer side. No street addresses are stored — location is represented at the neighborhood level. No PII appears in logs. All data is encrypted in transit, and production database connections require SSL. Final retention and consent policies are being developed with Dorot.

---

## Contributing

This is an early-stage research and pilot project. Not currently accepting outside contributions, but feedback, questions, and academic collaboration inquiries are welcome via issues.

---

## License

To be determined before public release.

## Acknowledgments

Dorot NYC, for the partnership and for the volunteer work that surfaced the problem in the first place.
