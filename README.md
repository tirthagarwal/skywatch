[README.md](https://github.com/user-attachments/files/26798593/README.md)
# 🛰️ SkyWatch — AI Satellite Change Detection Platform

> **Real-time illegal land-use change detection powered by Google Earth Engine + AI**

SkyWatch is a full-stack geospatial intelligence platform that automatically detects and reports illegal construction, deforestation, and environmental violations using Sentinel-2 satellite imagery and a trained Siamese U-Net++ deep learning model.

---

## ✨ Features

| Feature | Status |
|---|---|
| Interactive map with draw tools (select any area on Earth) | ✅ Live |
| Before/After satellite comparison with sync zoom | ✅ Live |
| Google Earth Engine — real Sentinel-2 imagery | ✅ Live |
| AI change detection (OSCD Siamese ResNet-34 U-Net++) | ✅ Live |
| Spectral indices — NDVI, MNDWI, NDBI | ✅ Live |
| Compliance engine — 4 violation rule types | ✅ Live |
| Government whitelist zones | ✅ Live |
| Firebase / Firestore persistent reporting | ✅ Ready |
| Violation PDF report export | 🔜 Next |

---

## 🏗️ Architecture

```
┌─────────────────────────────┐      ┌──────────────────────────────────┐
│   React Frontend (Vite)     │ ←──→ │   FastAPI Backend (Python)       │
│   - Leaflet map             │      │   - /analyze  → AI pipeline      │
│   - Before/After compare    │      │   - /violations → Firestore      │
│   - Analysis Panel          │      │   - /thumbnail → GEE tiles       │
│   - Violation overlay       │      │   - /gov-whitelist               │
└─────────────────────────────┘      └──────────┬───────────────────────┘
                                                 │
                              ┌──────────────────┼──────────────────┐
                              │                  │                  │
                    ┌─────────▼──────┐  ┌────────▼──────┐  ┌───────▼───────┐
                    │ Google Earth   │  │  OSCD Model   │  │   Firestore   │
                    │ Engine API     │  │  ResNet U-Net++│  │   (Firebase)  │
                    │ Sentinel-2 SR  │  │  26.1M params  │  │  Violations   │
                    └────────────────┘  └───────────────┘  └───────────────┘
```

---

## 🚀 Quick Start

### Prerequisites
- Node.js 18+
- Python 3.11+
- Google Earth Engine account (free non-commercial)

### 1. Clone & Install Frontend
```bash
git clone https://github.com/ShouryMishra/skywatch-satellite-detection.git
cd skywatch-satellite-detection
npm install
npm run dev
# → http://localhost:5173
```

### 2. Install Backend
```bash
cd backend
pip install fastapi uvicorn earthengine-api firebase-admin numpy Pillow python-dotenv requests python-dateutil torch torchvision segmentation-models-pytorch
```

### 3. Authenticate Google Earth Engine
```powershell
earthengine authenticate
# Opens browser → log in → Allow → paste code back
```

### 4. Configure Environment
Create `backend/.env`:
```env
GEE_PROJECT_ID=your-gcp-project-id
FIREBASE_CREDENTIALS_JSON=path/to/firebase-key.json
FIREBASE_PROJECT_ID=your-firebase-project-id
FRONTEND_ORIGIN=http://localhost:5173
```

### 5. Add Model Weights
Download `best_model.pth` and place it at:
```
backend/model/best_model.pth
```
> Model: OSCD-trained Siamese ResNet-34 U-Net++ (26.1M parameters, input: 12-band stacked Sentinel-2)

### 6. Start Backend
```bash
cd backend
python main.py
# → http://localhost:8000
```

---

## 🤖 AI Model Details

| Property | Value |
|---|---|
| Architecture | Siamese ResNet-34 U-Net++ (`segmentation-models-pytorch`) |
| Training Dataset | OSCD (Onera Satellite Change Detection) |
| Input | 12 channels: 6 Sentinel-2 bands × 2 (before + after stacked) |
| Output | Binary change mask (256×256) |
| Parameters | 26.1M |
| Bands used | B2, B3, B4 (RGB) · B8 (NIR) · B11, B12 (SWIR) |

---

## 🌍 Violation Detection Rules

| Rule | Trigger | Regulation |
|---|---|---|
| **Buffer Zone Encroachment** | Construction within 500m of water body | CRZ 2019, NGT |
| **Illegal Tree Felling** | NDVI drops >10% with built-up expansion | Forest Act 1927 |
| **Forest Encroachment** | Vegetation loss in forest grid cells | Forest Conservation Act |
| **Illegal Farmhouse** | Structure in agricultural/green zone | DGTCP Guidelines |

---

## 📡 API Endpoints

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/health` | Server status + GEE/Firebase connectivity |
| `POST` | `/analyze` | Run full AI pipeline on a bbox + date range |
| `GET` | `/thumbnail` | Get satellite tile URL for a location+date |
| `POST` | `/gov-whitelist` | Register approved construction zone |
| `GET` | `/violations` | List all detected violations |

---

## 📁 Project Structure

```
skywatch/
├── src/                        # React frontend
│   ├── pages/MapPage.jsx       # Main map interface
│   ├── components/             # UI components
│   └── index.css               # Design system
├── backend/
│   ├── main.py                 # FastAPI app + all endpoints
│   ├── gee_pipeline.py         # Google Earth Engine integration
│   ├── model_inference.py      # OSCD model inference
│   ├── indices.py              # NDVI/MNDWI/NDBI computation
│   ├── compliance_engine.py    # Violation detection rules
│   ├── firebase_db.py          # Firestore persistence
│   ├── training/               # Training data utilities
│   │   └── download_training_data.py
│   └── model/                  # Model weights (not in git)
│       └── best_model.pth
└── HANDOFF.md                  # Session continuity notes
```

---

## 🛠️ Tech Stack

**Frontend:** React · Vite · Leaflet · Framer Motion · Tailwind CSS

**Backend:** FastAPI · Python 3.11 · Uvicorn

**AI/ML:** PyTorch · Segmentation Models PyTorch · NumPy

**Geospatial:** Google Earth Engine API · Sentinel-2 SR Harmonized

**Database:** Firebase Firestore

---

## 📜 License
MIT
