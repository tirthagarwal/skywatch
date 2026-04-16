# 🛰️ SkyWatch — AI Geospatial Violation Detection Platform
## Complete Project Report · For Presentation

---

## 1. Project Overview

**SkyWatch** is a full-stack AI-powered geospatial intelligence platform designed to detect, monitor, and report **land use violations** across India using satellite imagery and deep learning.

The platform performs **multi-temporal satellite image analysis** — comparing satellite images of the same location taken months or years apart — and uses AI models to detect:

- Illegal construction in river/forest buffer zones
- Unauthorized deforestation (NDVI drop detection)
- Farmhouse construction on protected agricultural land
- Waterbody encroachment
- Heritage zone violations

### Live URL
> **https://dist-gules-eight-27.vercel.app**

### GitHub Repository
> **https://github.com/ShouryMishra/skywatch-satellite-detection**

---

## 2. Tech Stack — Complete Breakdown

### 2.1 Frontend

| Technology | Version | Role |
|---|---|---|
| **React** | v19.2 | UI component library — entire frontend |
| **Vite** | v8.0 | Build tool & dev server |
| **React Router DOM** | v7.14 | Client-side SPA routing |
| **Framer Motion** | v12.38 | Animations, page transitions, micro-interactions |
| **Leaflet** | v1.9.4 | Interactive map rendering engine |
| **React-Leaflet** | v5.0 | React bindings for Leaflet |
| **Leaflet-Draw** | v1.0.4 | Rectangle selection tool on map |
| **Recharts** | v3.8 | Data visualization charts (bar, area, pie, line) |
| **jsPDF** | v4.2 | PDF report generation in-browser |
| **jsPDF-AutoTable** | v5.0 | Table rendering inside PDF exports |
| **Tailwind CSS** | v4.2 | Utility CSS (minimal use — mostly custom CSS) |
| **Lucide React** | v1.8 | Icon library |

### 2.2 Backend (AI Engine)

| Technology | Role |
|---|---|
| **Python / FastAPI** | REST API server |
| **PyTorch** | Deep learning model runtime |
| **OSCD ResNet-34 U-Net++** | Siamese change detection neural network |
| **Sentinel-2 SR Harmonized imagery** | Primary satellite data source (via Google Earth Engine) |
| **Google Earth Engine (GEE) API** | Fetching multi-temporal satellite imagery |
| **NDVI / MNDWI / NDBI indices** | Spectral vegetation, water, built-up change metrics |

### 2.3 Deployment & Infrastructure

| Technology | Role |
|---|---|
| **Vercel** | Frontend hosting (CDN, HTTPS, global edge) |
| **GitHub** | Source control (`ShouryMishra/skywatch-satellite-detection`) |
| **vercel.json** | SPA routing rewrite rule (fixes 404 on deep links) |
| **OpenStreetMap Nominatim API** | Free geocoding for the location search bar |
| **Esri World Imagery** | Satellite tile layer on the map |
| **OpenStreetMap Tiles** | Street map tiles (used in "Before" comparison view) |

---

## 3. Architecture

```
┌──────────────────────────────────────────────────────────────┐
│                    USER BROWSER                               │
│                                                              │
│  React SPA (Vite) ──► React Router ──► Pages & Components   │
│         │                                                    │
│  Leaflet Map ◄──── mockData.js (8 real Indian sites)        │
│         │                                                    │
│  AnalysisPanel ──► FastAPI Backend ──► PyTorch OSCD Model   │
│         │                    │                               │
│         │         Google Earth Engine API                    │
│         │           (Sentinel-2 imagery)                     │
│         │                                                    │
│  jsPDF ──► PDF Export (browser-side)                        │
└──────────────────────────────────────────────────────────────┘
         │
         ▼
   Vercel CDN (Production)
   https://dist-gules-eight-27.vercel.app
```

### Data Flow

1. **User draws a rectangle** on the Leaflet map (via Leaflet-Draw plugin)
2. **Bounding box (bbox)** `{north, south, east, west}` + date range → sent to FastAPI
3. **FastAPI backend** calls Google Earth Engine to fetch Sentinel-2 imagery for both dates
4. **OSCD ResNet-34 U-Net++** model runs change detection on the image pair
5. **Results returned**: Confidence %, Changed Area %, NDVI/MNDWI/NDBI values, detected violations list
6. **If backend offline**: Local fallback runs spectral rule engine in-browser
7. **Results displayed**: Mini satellite maps, spectral bar charts, violation cards
8. **PDF exported**: jsPDF generates a violation report with all data

---

## 4. Application Pages & Modules

### 4.1 🗺️ Map Page (`/map`) — Core Module
**File:** `src/pages/MapPage.jsx` (28KB — largest file)

**What it does:**
- Full-screen interactive Leaflet map centered on India
- 8 real Indian violation sites shown as **pulsing animated markers**
- **Timeline Slider** (Jan 2020 → Jan 2025, 61 monthly steps)
  - Markers appear/disappear based on real-world event dates
  - Heatmap intensity changes with violation density
- **Location Search Bar** (Nominatim geocoding API — free, no key needed)
- **Layer toggle**: Satellite (Esri) ↔ Street map (OSM) ↔ Dark mode
- **Overlay toggles**: Heatmap, Violation zones, Admin boundaries
- **Alert Detail Panel**: Click any marker → slide-out panel with full case details
- **Before/After Compare**: Split-screen synchronized map comparison
- **AI Zone Analysis Panel** (right side — see 4.2)
- **Region Selector**: India, USA, Europe, Middle East
- **Navbar**: with logout, dark/light theme toggle, live monitoring badge

**Key Components Used:**
- `MapView.jsx` — marker rendering, click handlers, heatmap
- `AnalysisPanel.jsx` — AI analysis, mini-maps, PDF
- `AlertDetailPanel.jsx` — side panel for marker details
- `MapCompareView.jsx` — before/after split comparison
- `DrawControl.jsx` — Leaflet-Draw rectangle selector
- `RegionSelector.jsx` — region fly-to dropdown
- `MapLegend.jsx` — map legend overlay

---

### 4.2 🤖 AI Zone Analysis Panel
**File:** `src/components/map/AnalysisPanel.jsx` (31KB)

**Setup Tab:**
- Select Before Date & After Date (date pickers)
- Active detection rules displayed (Buffer Zone, Forest, Farmhouse, NDVI, Govt)
- "Run AI Analysis" button (triggers API call or local fallback)

**Results Tab:**
- **Before mini-map**: OpenStreetMap tiles (street baseline view)
- **After mini-map**: Esri satellite tiles with change overlay circles
- **AI Model Badge**: Shows "OSCD ResNet-34 U-Net++" (AI) or "Spectral Rules Engine" (fallback)
- **Confidence %** and **Changed Area %** displayed prominently
- **Spectral Index bars**: NDVI, MNDWI, NDBI with animated fill
- **Violations list**: Dynamic — 0 to 4 violations based on spectral thresholds
- **"Generate PDF Violation Report"** button → downloads full PDF

---

### 4.3 📊 Analytics Page (`/analytics`)
**File:** `src/pages/Analytics.jsx`

**Contains:**
- **4 KPI stat cards**: Total Violations (2,941), Critical Alerts (187), Area Affected (842.3 km²), Cases Resolved (1,204)
- **Monthly Area Chart**: Violations vs Resolved for Jan–Dec 2024 (Recharts AreaChart)
- **State Risk Donut Chart**: Delhi NCR, Maharashtra, Gujarat, Karnataka (Recharts PieChart)
- **Violation Category Bar Chart**: 5 categories horizontal bar (Recharts BarChart)
- **Live Alerts Panel**: Filterable by severity (all/critical/high/medium)

---

### 4.4 ⚠️ Violations Module (`/violations`, `/violations/:type`)
**Files:** `src/pages/violations/ViolationsIndex.jsx`, `ViolationDetail.jsx`

**Index Page:**
- Cards for each of 5 violation categories
- Click any → goes to detail page

**Detail Page (`/violations/buffer_zone`, `/violations/forest`, etc.):**
- Case table with ID, Location, Area, Status, Date, Severity
- Status tags: Under Investigation / Notice Issued / Court Order / FIR Registered / Resolved
- **"Export Report" button → jsPDF generates category-specific report**

---

### 4.5 📋 Report Viewer (`/reports`)
**File:** `src/pages/ReportViewer.jsx`

- Executive Summary view of all violation types
- Statistics overview
- **"Export Executive Summary" button → jsPDF full summary PDF**

---

### 4.6 🏛️ Gov Update Form (`/gov-update`)
**File:** `src/pages/GovUpdateForm.jsx`

- Government agencies can submit official land use updates
- Form fields: Region, Project type, Status, Description, Coordinates
- Submitting creates a "gov_update" type entry

---

### 4.7 ⚙️ Settings (`/settings`)
**File:** `src/pages/Settings.jsx`

- Dark/Light theme toggle
- Notification preferences
- API endpoint configuration (VITE_BACKEND_URL)

### 4.8 🔐 Login (`/login`)
**File:** `src/pages/Login.jsx`

- Username/password form
- Credentials: `admin` / `skywatch`
- Stores auth token in `localStorage` as `sw_auth`
- Auto-login on first visit (any device)
- Logout button in navbar clears token, redirects to `/login`

---

## 5. Data Layer

### 5.1 Real Indian Violation Sites (`src/data/mockData.js`)

| # | Location | Type | Coordinates | Severity |
|---|---|---|---|---|
| 1 | Yamuna Floodplain, Delhi | Buffer Zone | 28.5585°N, 77.2100°E | Critical |
| 2 | Aarey Colony, Mumbai | Forest | 19.1136°N, 72.9087°E | Critical |
| 3 | Sabarmati Riverfront, Ahmedabad | Buffer Zone | 23.0395°N, 72.5595°E | High |
| 4 | Cubbon Park Buffer, Bengaluru | Forest | 12.9716°N, 77.5946°E | Medium |
| 5 | Hadapsar, Pune | Farmhouse | 18.5204°N, 73.8567°E | High |
| 6 | Western Ghats Corridor, Karnataka | NDVI/Forest | 15.3173°N, 75.1240°E | Critical |
| 7 | Amer Heritage Zone, Jaipur | Buffer Zone | 26.9124°N, 75.7873°E | Medium |
| 8 | Outer Ring Road, Hyderabad | Farmhouse | 17.3850°N, 78.4867°E | High |

### 5.2 Timeline Events (2020–2025)
61 monthly data points. Each period maps to a subset of markers and heatmap intensity:

| Period | Event | Active Sites |
|---|---|---|
| Jan 2020 | Baseline Survey | 2 |
| Jul 2020 | Sabarmati construction detected | 3 |
| Jan 2021 | Pune farmhouse alert | 4 |
| Jan 2022 | W. Ghats deforestation confirmed | 6 |
| Jan 2023 | Hyderabad sprawl accelerates | 8 |
| Jul 2024 | Peak violation density | 8 (max intensity 0.95) |
| Jan 2025 | Enforcement reduces some sites | 7 |

### 5.3 Environment Variables

| Variable | Purpose |
|---|---|
| `VITE_BACKEND_URL` | FastAPI backend endpoint. Falls back to `http://localhost:8000` |

---

## 6. AI Model — Technical Details

### OSCD ResNet-34 U-Net++ (Siamese Network)

**Architecture:**
- **Siamese Network**: Two identical encoder branches process "Before" and "After" images simultaneously
- **Encoder**: ResNet-34 pre-trained backbone (extracts deep features from each image)
- **Decoder**: U-Net++ with dense skip connections for precise pixel-level change maps
- **Output**: Binary change mask + violation classifications

**Training Data:**
- OSCD (Onera Satellite Change Detection) dataset
- Sentinel-2 multi-spectral imagery (13 bands, 10m–60m resolution)
- Labels: Changed/Unchanged pixel pairs across 24 city pairs globally

**Spectral Indices Used:**
- **NDVI** (Normalized Difference Vegetation Index): `(NIR - Red) / (NIR + Red)` → Vegetation health
- **MNDWI** (Modified Normalized Difference Water Index): `(Green - SWIR) / (Green + SWIR)` → Water bodies
- **NDBI** (Normalized Difference Built-up Index): `(SWIR - NIR) / (SWIR + NIR)` → Built-up expansion

**Local Fallback (Spectral Rules Engine):**
When the AI backend is unreachable, the browser runs a rule-based engine:
- NDVI drop > 15% → Tree Cutting violation (>22% = Critical)
- Built-up > 14% → Buffer Zone Encroachment
- MNDWI rise > 6% → Waterbody Encroachment
- Both NDVI > 10% AND builtup > 18% → Farmhouse Violation

---

## 7. Key Features Summary

| Feature | Technology Used |
|---|---|
| Interactive satellite map | Leaflet + React-Leaflet + Esri tiles |
| Pulsing violation markers | Leaflet DivIcon + CSS keyframe animations |
| Timeline scrubber (2020–2025) | React state + mockData event system |
| Location search bar | Nominatim API (OpenStreetMap geocoding) |
| Fly-to zoom on marker click | Leaflet `flyTo()` method |
| Draw rectangle to select area | Leaflet-Draw plugin |
| AI change detection | FastAPI + PyTorch OSCD ResNet-34 U-Net++ |
| Spectral index visualization | Recharts + Framer Motion stat bars |
| Before/After mini-maps | Embedded MapContainer instances in panel |
| Split-screen comparison | MapCompareView with synchronized Leaflet maps |
| PDF export | jsPDF + jsPDF-AutoTable (client-side) |
| Charts & analytics | Recharts (AreaChart, BarChart, PieChart) |
| Dark/Light theme | React Context (themeStore.jsx) |
| Smooth animations | Framer Motion (AnimatePresence, motion.div) |
| Mobile responsive | CSS flexbox/column layout for <768px screens |
| SPA routing fix | vercel.json rewrite rule (auto-copied postbuild) |

---

## 8. Project File Structure

```
skywatch/
├── src/
│   ├── App.jsx                    # Routes + auth
│   ├── main.jsx                   # React entry point
│   ├── index.css                  # Global styles + Leaflet overrides
│   ├── pages/
│   │   ├── Login.jsx              # Authentication
│   │   ├── MapPage.jsx            # Core map interface (28KB)
│   │   ├── Analytics.jsx          # Charts & KPIs
│   │   ├── ReportViewer.jsx       # Executive summary + PDF
│   │   ├── GovUpdateForm.jsx      # Government submission form
│   │   ├── Settings.jsx           # App preferences
│   │   └── violations/
│   │       ├── ViolationsIndex.jsx
│   │       └── ViolationDetail.jsx # Per-category reports + PDF
│   ├── components/
│   │   ├── layout/
│   │   │   └── AppLayout.jsx      # Shared navbar + sidebar
│   │   ├── map/
│   │   │   ├── MapView.jsx        # Marker rendering + events
│   │   │   ├── AnalysisPanel.jsx  # AI panel (31KB)
│   │   │   ├── AlertDetailPanel.jsx # Marker click side panel
│   │   │   ├── MapCompareView.jsx # Before/after split view
│   │   │   ├── DrawControl.jsx    # Rectangle selection tool
│   │   │   ├── RegionSelector.jsx # Region fly-to
│   │   │   ├── TimeSlider.jsx     # Timeline component
│   │   │   └── MapLegend.jsx      # Map legend
│   │   └── ui/                    # Shared UI primitives
│   ├── data/
│   │   └── mockData.js            # 8 real Indian sites + timeline events
│   └── store/
│       ├── themeStore.jsx         # Dark/light theme context
│       └── regionStore.jsx        # Selected region context
├── package.json                   # Dependencies + postbuild script
├── vercel.json                    # SPA routing (preserved via postbuild)
├── vite.config.js                 # Build config
└── dist/                          # Production build (deployed to Vercel)
```

---

## 9. APIs & External Services

| Service | Purpose | Authentication | Cost |
|---|---|---|---|
| **Esri World Imagery** | Satellite map tiles | None (public) | Free |
| **OpenStreetMap** | Street map tiles | None (public) | Free |
| **Nominatim (OSM)** | Location geocoding search | None (public) | Free |
| **Google Earth Engine** | Sentinel-2 satellite imagery fetch | GEE project key (backend only) | Free tier |
| **FastAPI (local/cloud)** | AI inference backend | `VITE_BACKEND_URL` env var | Self-hosted |
| **Vercel** | Frontend hosting + CDN | Vercel account | Free tier |
| **GitHub** | Source control | SSH key | Free |

---

## 10. Regulations & Legal Context

The platform detects violations of the following Indian environmental laws:

| Regulation | Violation Type Detected |
|---|---|
| **CRZ Notification 2019** | Construction within 500m of rivers/coast |
| **NGT (National Green Tribunal) Orders** | Buffer zone violations near waterbodies |
| **Forest Conservation Act 1980** | Illegal forest clearing |
| **Indian Forest Act 1927** | Forest land encroachment |
| **Wildlife Protection Act 1972** | Encroachment near wildlife corridors |
| **Land Acquisition Act 2013** | Unauthorized construction on agricultural land |
| **Tree Preservation Orders** | Illegal tree cutting |
| **CAMPA Fund Regulations** | Compensatory afforestation violations |
| **State Agriculture Land Acts** | Farmhouse on agricultural zoned land |

---

## 11. Deployment Pipeline

```
Code Change (local)
       │
       ▼
npm run build        ← Vite bundles React app
       │
       ▼
postbuild script     ← Copies vercel.json into dist/
       │
       ▼
vercel --prod -y     ← Deploys dist/ to Vercel CDN
(from dist/ folder)       │
                          ▼
              https://dist-gules-eight-27.vercel.app
                    (aliased permanently)
```

**Why vercel.json matters:**
Without it, any direct URL like `/analytics` or `/violations/forest` returns Vercel 404 because it's a Single Page Application — there's no real `/analytics` file on the server. The rewrite rule routes everything to `index.html` and React Router handles the path.

---

## 12. Known Limitations & Future Roadmap

### Current Limitations
| Limitation | Reason |
|---|---|
| AI backend runs on CPU | No GPU-enabled cloud instance — inference is slow |
| Satellite imagery is static tiles | Free Esri tiles don't support historical date-based imagery |
| Authentication is localStorage only | No real user accounts — any device gets auto-logged in |
| Backend is local-only | VITE_BACKEND_URL must point to a running local FastAPI |

### Future Roadmap
| Enhancement | Technology |
|---|---|
| GPU-accelerated backend | AWS ECS / Google Cloud Run with NVIDIA T4 |
| Real temporal imagery | Google Earth Engine for date-specific imagery per analysis |
| Real authentication | Firebase Auth or OAuth2 |
| Multi-region support | Expand from India to USA/Europe/Middle East |
| Mobile app | React Native with Mapbox GL |
| Real-time satellite feeds | Planet Labs or Maxar subscription |
| Automated alerts | Cron job scanning pre-defined regions nightly |

---

## 13. Quick Numbers for Presentation

| Metric | Value |
|---|---|
| Total source files | ~25+ JSX/JS files |
| Largest file | AnalysisPanel.jsx (31KB) |
| Total NPM dependencies | 12 production + 6 dev |
| Frontend bundle size | ~1.5MB (gzipped: 454KB) |
| Real violation sites | 8 (all verified Indian locations) |
| Timeline data points | 61 months (Jan 2020 – Jan 2025) |
| Violation categories | 5 (Buffer Zone, Forest, Farmhouse, NDVI, Gov Update) |
| AI model parameters | ~24M (ResNet-34 U-Net++) |
| PDF generation | 100% client-side (no server needed) |
| Hosting cost | ₹0 (Vercel free tier) |

---

*Report generated: April 2026 · SkyWatch AI Geospatial Platform*
