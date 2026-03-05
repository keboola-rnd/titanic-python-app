# 🚢 RMS Titanic — Voyage Dashboard

Interactive data app built for **Keboola Data Apps**. FastAPI serves a self-contained vanilla JS dashboard that visualises Titanic passenger data straight from Keboola Storage.

---

## Repository structure

```
titanic-js-app/
├── app.py          ← FastAPI backend + full HTML/CSS/JS frontend (inlined)
├── pyproject.toml  ← Python dependencies + Keboola entrypoint
├── .env            ← Local credentials (never commit!)
└── .gitignore
```

---

## Tech stack

| Layer | Technology |
|---|---|
| Server | Python 3.10 · FastAPI · Uvicorn |
| Data | Pandas · keboola-storage-client |
| Frontend | Vanilla JS (ES modules) · pure SVG charts · CSS animations |
| Hosting | Keboola Data Apps |

No Streamlit. No Node.js. No build step.

---

## Features

- ⚓ **Survival analysis** — by class, gender and age group
- 💰 **Fare heatmap** — 200 passengers sorted by ticket price
- 🌍 **Animated voyage map** — Southampton → Cherbourg → Queenstown → sinking point
- 📋 **Passenger explorer** — server-side pagination, filtering, sorting and full-text search
- ✨ **Animated starfield** + ocean waves background
- 🔄 **Graceful fallback** — built-in sample data if Keboola is not connected

---

## REST API

All data is served by FastAPI. The JS frontend calls these endpoints at runtime.

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/health` | Liveness check + row count |
| GET | `/api/stats` | KPI numbers (total, survivors, avg age, avg fare) |
| GET | `/api/by-class` | Survival breakdown by passenger class |
| GET | `/api/by-gender` | Survival breakdown by sex |
| GET | `/api/by-port` | Boarding port breakdown |
| GET | `/api/by-age-group` | Survival rate by age group |
| GET | `/api/heatmap?n=200` | Fare heatmap sample (max 500) |
| GET | `/api/passengers` | Paginated, filtered and sorted passenger table |

### `/api/passengers` query params

| Param | Default | Values |
|-------|---------|--------|
| `q` | `""` | Full-text search across name, hometown, destination |
| `survived` | `all` | `all` / `survived` / `lost` |
| `cls` | `all` | `all` / `1` / `2` / `3` |
| `page` | `1` | Page number |
| `per_page` | `50` | 1–200 |
| `sort_by` | `PassengerId` | Any column name |
| `sort_dir` | `asc` | `asc` / `desc` |

Interactive docs available at `/docs` (Swagger UI) when running locally.

---

## Local development

### 1. Clone & install

```bash
git clone https://github.com/your-org/titanic-js-app.git
cd titanic-js-app
pip install -e .
```

### 2. Configure credentials

Copy `.env` and fill in your values:

```bash
KBC_URL=https://connection.keboola.com
KBC_TOKEN=your-keboola-api-token
TABLE_ID=in.c-titanic.passengers
```

### 3. Run

```bash
uvicorn app:app --reload --port 8080
```

Open [http://localhost:8080](http://localhost:8080) — the dashboard loads immediately.
If no Keboola credentials are set, the app falls back to built-in sample data automatically.

---

## Deploy to Keboola

1. Push this repo to GitHub (`.env` is gitignored — never committed).
2. In your Keboola project go to **Components → Data Apps → Create**.
3. Select **Git repository**, paste your GitHub URL, set branch to `main`.
4. Keboola reads the entrypoint from `pyproject.toml` automatically:
   ```
   uvicorn app:app --host 0.0.0.0 --port 8080
   ```
5. Under **Secrets** add:
   ```
   KBC_URL    = https://connection.keboola.com
   KBC_TOKEN  = <your-api-token>
   TABLE_ID   = in.c-titanic.passengers
   ```
6. Optionally set up an **Input Mapping** — map your Titanic table to `passengers.csv` for faster reads without API calls.
7. Click **Deploy** 🚀

---

## Data loading priority

The app tries three sources in order:

1. **Mounted CSV** — `/data/in/tables/passengers.csv` injected by Keboola Input Mapping (fastest)
2. **Storage API** — fetched via `keboola-storage-client` using `KBC_TOKEN`
3. **Built-in sample data** — 20 hardcoded passengers for local demo / testing

---

## Expected table columns

```
PassengerId, Survived, Pclass, Name, Sex, Age, SibSp, Parch,
Ticket, Fare, Cabin, Embarked, WikiId, Name_wiki, Age_wiki,
Hometown, Boarded, Destination, Lifeboat, Body, Class
```

Optional columns (`Lifeboat`, `Hometown`, `Destination`, `Age_wiki`) are handled gracefully — missing columns are simply hidden.

Port codes are normalised automatically: `S` → Southampton, `C` → Cherbourg, `Q` → Queenstown.