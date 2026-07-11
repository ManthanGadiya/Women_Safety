# Women Safety Risk Zone Prediction System

A full-stack machine learning web application that predicts crime risk zones in real time to enhance women's safety. Uses a Random Forest classifier trained on historical Chicago crime data to categorize geographic areas as **Low**, **Medium**, or **High** risk.

## Features

- **Live Interactive Map** — Click anywhere to get a risk assessment with heatmap overlay
- **Time Machine** — Slide through hours (0–23) to see how risk changes by time of day
- **Safe Route Planning** — Turn-by-turn navigation that avoids high-risk zones (powered by OSRM)
- **Smart SOS** — Manual SOS + auto-trigger in high-risk zones; sends WhatsApp deep link with live location
- **Emergency Contacts** — Manage contacts from your profile page
- **Analytics Dashboard** — Charts for crime trends, distribution, and threat levels

## Tech Stack

| Layer | Technology |
|---|---|
| Backend | FastAPI (Python 3.10), Uvicorn |
| ML | Scikit-Learn RandomForestClassifier, Pandas, NumPy |
| Database | SQLite |
| Frontend | React 19, Vite, React Router 7 |
| Maps | Leaflet.js, React-Leaflet 5 |
| Charts | Recharts 3 |
| Routing API | OSRM (Open Source Routing Machine) |
| Styling | Vanilla CSS with glassmorphism |

## Project Structure

```
├── backend/
│   ├── main.py                 # FastAPI server & REST endpoints
│   ├── database.py             # SQLite setup
│   ├── data_preparation.py     # ML model training pipeline
│   ├── models/risk_model.pkl   # Trained Random Forest model
│   └── women_safety.db         # SQLite database
├── frontend/
│   ├── src/
│   │   ├── App.jsx             # Router & layout
│   │   ├── pages/
│   │   │   ├── LiveMapPage.jsx # Map, risk prediction, SOS, routes
│   │   │   ├── AnalyticsPage.jsx # Charts & crime trends
│   │   │   └── ProfilePage.jsx # Profile & emergency contacts
│   │   └── main.jsx            # Entry point
│   ├── package.json
│   └── vite.config.js
├── data/
│   ├── grid_risk.csv            # Pre-computed grid risk levels
│   └── crimes.csv               # (reference to original dataset path)
├── main.py
└── README.md
```

## Setup

### Prerequisites
- Python 3.10+
- Node.js 18+

### Backend

```bash
# Create and activate virtual environment
python -m venv venv
.\venv\Scripts\Activate.ps1   # Windows
source venv/bin/activate       # macOS/Linux

# Install dependencies
pip install fastapi uvicorn joblib pandas numpy scikit-learn

# Start the server
cd backend
uvicorn main:app --host 0.0.0.0 --port 8000 --reload
```

API docs available at `http://localhost:8000/docs`.

### Frontend

```bash
cd frontend
npm install
npx vite
```

Opens at `http://localhost:5173`.

## API Endpoints

| Method | Endpoint | Purpose |
|---|---|---|
| POST | `/api/predict_risk` | Predict risk for a (lat, lon, datetime) |
| GET | `/api/grid_data` | Get grid risk heatmap data |
| POST | `/api/safe_route` | Get safe route coordinates |
| POST | `/api/trigger_sos` | Trigger SOS alert |
| POST | `/api/register` | Register user |
| POST | `/api/login` | Login |
| GET | `/api/contacts/{user_id}` | Get emergency contacts |
| POST | `/api/contacts` | Add emergency contact |

## Data Source

The ML model was trained on Chicago crime data from the [Chicago Data Portal](https://data.cityofchicago.org/Public-Safety/Crimes-2001-to-Present/ijzp-q8t2). Pre-computed risk data (`grid_risk.csv`) and the trained model (`risk_model.pkl`) are included so the app works out of the box.
