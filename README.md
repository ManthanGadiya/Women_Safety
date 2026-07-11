# Women Safety Risk Zone Prediction System

A full-stack machine learning web application that predicts crime risk zones in real time to enhance women's safety. Uses a Random Forest classifier trained on 20+ years of Chicago crime data to categorize geographic areas as **Low**, **Medium**, or **High** risk at any given time of day.

## Features

### Live Interactive Map
Click anywhere on the map to get an instant risk assessment. A color-coded heatmap overlay (green = Low, yellow = Medium, red = High) shows risk levels across the city — powered by CartoDB light tiles with no API key required.

### Time Machine
Slide through hours 0–23 to see how crime risk changes throughout the day. The ML model recalculates predictions dynamically based on historical crime patterns for that exact hour (e.g., higher risk at 2 AM vs. noon).

### Safe Route Planning
Get turn-by-turn navigation that avoids high-risk zones. Powered by the OSRM routing API — draws realistic curved paths along the actual street network with written navigation steps below the map.

### Smart SOS System
- **Manual SOS** button opens a WhatsApp deep link (`wa.me`) with your live Google Maps coordinates pre-drafted
- **Auto-SOS** triggers automatically when you enter a high-risk zone
- Backend logs a simulated SMS delivery to your emergency contacts

### Emergency Contacts
Manage your emergency contacts from the profile page with an interactive Edit/Save toggle.

### Analytics Dashboard
Dynamic charts (Recharts) showing crime distribution by day of week and crime type. Filter by Area Scope (Whole City / 5km Radius) and Time Scope (This Week / Last Month / Custom Date) — charts mathematically scale data based on your selection.

## Tech Stack

| Layer | Technology | Why |
|---|---|---|
| Backend | **FastAPI** (Python 3.10), Uvicorn | High-performance async server, auto Swagger docs |
| ML | **Scikit-Learn RandomForestClassifier** | Handles non-linear data well, resists overfitting |
| Data | **Pandas, NumPy, Joblib** | Data cleaning, feature engineering, model serialization |
| Database | **SQLite** | Lightweight, file-based, no separate server setup needed |
| Frontend | **React 19, Vite 8, React Router 7** | Fast component-based UI with instant HMR |
| Maps | **Leaflet.js, React-Leaflet 5** | Open-source, no paid API keys required |
| Charts | **Recharts 3** | Customizable charting for analytics |
| Routing | **OSRM** (Open Source Routing Machine) | Real road-network routing, free to use |
| Icons | **Lucide React** | Consistent modern icons |
| SOS | **WhatsApp `wa.me` deep links** | Opens pre-drafted message with live coordinates |
| Styling | Vanilla CSS with glassmorphism | Light theme with gradient backgrounds |

## Project Structure

```
├── backend/
│   ├── main.py                  # FastAPI server — all REST endpoints
│   ├── database.py              # SQLite database init & schema
│   ├── data_preparation.py      # ML training pipeline (data cleaning, feature extraction, model training)
│   ├── PROJECT_DETAILS.md       # Comprehensive documentation
│   ├── women_safety.db          # SQLite database (users, emergency contacts)
│   └── models/
│       └── risk_model.pkl       # Trained RandomForestClassifier
├── frontend/
│   ├── src/
│   │   ├── main.jsx             # React entry point
│   │   ├── index.css            # Global styles (glassmorphism, light theme)
│   │   ├── App.jsx              # Router + Navbar + global alert system
│   │   └── pages/
│   │       ├── LiveMapPage.jsx  # Interactive map, risk prediction, SOS, safe routes
│   │       ├── AnalyticsPage.jsx # Crime trend charts with filters
│   │       └── ProfilePage.jsx  # User profile & emergency contacts
│   ├── package.json
│   └── vite.config.js
├── data/
│   ├── grid_risk.csv            # Pre-computed geographic grid risk levels
│   └── crimes.csv               # Reference path to original dataset
├── main.py
└── README.md
```

## ML Model Details

The model in `backend/data_preparation.py`:

1. **Data Cleaning** — Drops rows missing lat/lon, keeps `Date`, `Primary Type`, `Latitude`, `Longitude`
2. **Feature Engineering** — Extracts `Hour`, `DayOfWeek`, `Month` from the timestamp
3. **Spatial Binning** — Rounds lat/lon to 2 decimal places (~1.1 km grid cells)
4. **Risk Labeling** — Uses **50th percentile** (Low/Medium boundary) and **85th percentile** (Medium/High boundary) of crime counts per grid-hour
5. **Training** — RandomForestClassifier with 100 trees, max depth 10, trained on `[GridLat, GridLon, Hour]`
6. **Outputs** — `risk_model.pkl` (model) and `grid_risk.csv` (pre-computed grid data)

**Upgrade History:**
- Initially 33% of the city was flagged "High Risk" — recalibrated to use the **85th percentile** so only genuinely dangerous areas show red
- Nearest Safe Zone now computes genuine Euclidean distance to the nearest Low-Risk grid instead of random placement

## Setup

### Prerequisites
- Python 3.10+
- Node.js 18+

### Backend

```bash
python -m venv venv
.\venv\Scripts\Activate.ps1   # Windows
source venv/bin/activate       # macOS/Linux

pip install fastapi uvicorn joblib pandas numpy scikit-learn

cd backend
uvicorn main:app --host 0.0.0.0 --port 8000 --reload
```

API docs at `http://localhost:8000/docs`.

### Frontend

```bash
cd frontend
npm install
npx vite
```

Opens at `http://localhost:5173`.

## API Endpoints

### Risk & Navigation

| Method | Endpoint | Description |
|---|---|---|
| POST | `/api/predict_risk` | Predict risk level for a location. Body: `{ lat, lon, datetime }` |
| GET | `/api/grid_data?hour=14` | Get grid risk heatmap data (optional hour filter) |
| POST | `/api/safe_route` | Get safe route coordinates. Body: `{ start_lat, start_lon, end_lat, end_lon }` |

### SOS

| Method | Endpoint | Description |
|---|---|---|
| POST | `/api/trigger_sos` | Trigger SOS — logs simulated SMS to emergency contacts. Body: `{ user_id, lat, lon }` |

### User Management

| Method | Endpoint | Description |
|---|---|---|
| POST | `/api/register` | Register new user. Body: `{ name, email, phone, password }` |
| POST | `/api/login` | Login. Body: `{ email, password }` |
| GET | `/api/contacts/{user_id}` | Get emergency contacts for a user |
| POST | `/api/contacts` | Add emergency contact. Body: `{ user_id, name, phone, relation }` |

## Data Source

The ML model was trained on the [Chicago Crime Dataset](https://data.cityofchicago.org/Public-Safety/Crimes-2001-to-Present/ijzp-q8t2) (2001–Present) from the Chicago Data Portal (~348M+ rows). Pre-computed `grid_risk.csv` and `risk_model.pkl` are included so the app works out of the box without retraining.

To retrain from scratch, update the path in `data_preparation.py` to point to the actual dataset, then run:

```bash
python backend/data_preparation.py
```
