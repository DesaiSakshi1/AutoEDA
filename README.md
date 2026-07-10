AutoEDA — Automated Exploratory Data Analysis Platform

AutoEDA is a full-stack web application that lets you upload any tabular dataset (CSV or Excel) and instantly get a complete, professional Exploratory Data Analysis (EDA) report — summary statistics, missing-value analysis, correlations, distributions, outlier detection, feature importance, and AI-generated insights — without writing a single line of code.

It combines a React + TypeScript frontend with a Python FastAPI backend for heavier statistical computation, chart generation, and AI-powered narrative insights, while also supporting a lightweight client-side analysis mode when the backend isn't available.


Table of Contents


Features
Tech Stack
Architecture
Project Structure
Getting Started

Prerequisites
Frontend Setup
Backend Setup



Environment Variables
API Reference
Available Scripts
Testing
Deployment
Roadmap
License



Features


Drag-and-drop dataset upload — supports .csv, .xls, and .xlsx files up to 50 MB
Automatic column type detection — numeric, categorical, date, or empty
Overview dashboard — dataset shape, KPI cards (rows, columns, missing %, duplicates)
Descriptive statistics — mean, median, std, min/max, quartiles for every numeric column
Missing value analysis — column-wise missing counts with an interactive missing-data heatmap
Correlation analysis — correlation matrix with an interactive heatmap
Distribution charts — histograms/bar charts per column (numeric, categorical, and date)
Outlier detection — boxplots and flagged outlier records
Feature importance — model-based feature ranking against a user-selected target column
AI-generated insights — natural-language summary of key patterns, powered by OpenAI
Data cleaning actions — fix missing values, remove duplicates, flag outliers directly from the dashboard
Column deep-dive — click into any column for a focused, detailed view
Export options — download a polished PDF report, export cleaned data, or export to Excel/CSV
Light/dark theme toggle
Responsive landing page with feature highlights, tech stack showcase, and "how it works" walkthrough


Tech Stack

Frontend


React 18 + TypeScript + Vite
React Router v6 for routing
shadcn/ui + Radix UI primitives for accessible components
Tailwind CSS for styling
Recharts + Chart.js for data visualization
TanStack React Query for data fetching/caching
Framer Motion for animations
PapaParse (CSV) and SheetJS/xlsx (Excel) for client-side parsing
jsPDF + jspdf-autotable for client-side PDF generation
Axios for API communication
Vitest + Testing Library + Playwright for testing


Backend


Python 3 + FastAPI
Pandas / NumPy for data processing
Matplotlib / Seaborn for chart image generation
scikit-learn for feature importance modeling
ReportLab for PDF report generation
OpenAI API for AI-generated insights
Pytest + httpx for backend testing


Architecture

AutoEDA can run in two modes:


Full-stack mode (recommended) — The React frontend uploads the dataset to the FastAPI backend, which parses the file, runs the full statistical analysis in Pandas/NumPy, generates chart images with Matplotlib/Seaborn, computes feature importance with scikit-learn, calls the OpenAI API for narrative insights, and returns everything (plus a downloadable PDF) to the frontend for display. Each upload creates a server-side session (session_id) that subsequent requests (missing data, correlation, distribution, outliers, feature importance, insights, PDF export) reference.
Frontend-only mode — If no backend is reachable, the app falls back to a pure client-side analysis engine (src/lib/analyzeData.ts) that computes the same core statistics directly in the browser using PapaParse/XLSX, so the dashboard still works standalone.


Browser (React SPA)
   │
   ├── /api/*  (Vite dev proxy → http://127.0.0.1:8000)
   │
   ▼
FastAPI Backend
   ├── routers/eda.py         → upload, summary, missing, correlation, distribution,
   │                             outliers, feature-importance, insights, export/pdf
   ├── services/parser.py     → CSV/Excel parsing
   ├── services/analysis.py   → core statistical analysis
   ├── services/visualization.py → Matplotlib/Seaborn chart images (base64)
   ├── services/feature_importance.py → scikit-learn model-based ranking
   ├── services/insights.py   → OpenAI-powered narrative insights
   ├── services/export_pdf.py → ReportLab PDF report builder
   └── store.py                → in-memory session storage

Project Structure

insight-navigator/
├── src/
│   ├── components/
│   │   ├── dashboard/        # TabOverview, TabStatistics, TabMissing, TabCorrelation,
│   │   │                     # TabCharts, TabFeatureImportance, TabReport, AIInsights,
│   │   │                     # DataCleaning, ExportMenu, ColumnDrawer, KPICards
│   │   ├── landing/          # Navbar, HeroSection, FeaturesSection, HowItWorks,
│   │   │                     # TechStack, StatsBar, Footer
│   │   └── ui/                # shadcn/ui component library
│   ├── pages/                 # Landing, Dashboard, Index, NotFound
│   ├── lib/
│   │   ├── analyzeData.ts     # client-side EDA analysis engine (fallback mode)
│   │   ├── api.ts              # Axios client for the FastAPI backend
│   │   ├── generateData.ts     # sample/demo data generator
│   │   ├── pdfCharts.ts         # chart helpers for client-side PDF export
│   │   └── theme.tsx            # light/dark theme provider
│   ├── hooks/                  # use-toast, use-mobile
│   └── App.tsx                  # app root + routing
├── backend/
│   ├── main.py                  # FastAPI app entrypoint
│   ├── config.py                 # env config (file size limits, OpenAI settings)
│   ├── store.py                   # in-memory session store
│   ├── routers/eda.py              # all API endpoints
│   ├── services/
│   │   ├── parser.py                # CSV/Excel parsing
│   │   ├── analysis.py               # statistics, correlation, missing data, outliers
│   │   ├── visualization.py           # chart image generation
│   │   ├── feature_importance.py       # scikit-learn feature ranking
│   │   ├── insights.py                  # OpenAI-based insight generation
│   │   └── export_pdf.py                 # PDF report builder
│   ├── tests/                             # pytest test suite
│   └── requirements.txt
├── public/
├── dist/                          # production build output
├── vite.config.ts                  # dev server + /api proxy config
├── tailwind.config.ts
└── package.json

Getting Started

Prerequisites


Node.js 18+ and npm
Python 3.10+
An OpenAI API key (optional — only required for AI-generated insights)


Frontend Setup

bash# Install dependencies
npm install

# Start the development server (http://localhost:5000)
npm run dev

The Vite dev server proxies any request to /api/* through to the FastAPI backend at http://127.0.0.1:8000, so make sure the backend is running alongside the frontend for full functionality.

Backend Setup

bashcd backend

# Create and activate a virtual environment (recommended)
python -m venv venv
source venv/bin/activate        # on Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Copy environment template and add your OpenAI key
cp .env.example .env

# Run the API server (http://127.0.0.1:8000)
python main.py

The backend also exposes a health check at GET /health.

Environment Variables

Create a backend/.env file (see backend/.env.example):

VariableDescriptionDefaultOPENAI_API_KEYAPI key used to generate AI insights(empty)OPENAI_MODELOpenAI model used for insight generationgpt-4o-mini

Optionally, set VITE_API_URL in the frontend environment to point to a non-default backend URL (defaults to /api, proxied to http://127.0.0.1:8000).

API Reference

All endpoints are served under the FastAPI app (proxied via /api in development).

MethodEndpointDescriptionPOST/uploadUpload a CSV/Excel file, optionally with a target_column; returns full analysis, chart images, AI insights, and a session_idGET/summaryColumn stats, categorical breakdown, dtypes, and shape for a sessionGET/missingMissing-value breakdown and heatmap imageGET/correlationCorrelation matrix and heatmap imageGET/distribution/{column}Distribution chart for a specific columnGET/outliersOutlier records and boxplot imagesPOST/feature-importanceCompute feature importance against a chosen target columnGET/insightsRetrieve the AI-generated narrative insights for a sessionGET/export/pdfDownload a formatted PDF EDA report for a sessionGET/healthBackend health check

Available Scripts

CommandDescriptionnpm run devStart the Vite development servernpm run buildProduction build (output in dist/)npm run build:devDevelopment-mode buildnpm run previewPreview the production build locallynpm run lintRun ESLintnpm run testRun the Vitest test suitenpm run test:watchRun Vitest in watch mode

Backend tests (from the backend/ directory):

bashpytest

Testing


Frontend: Vitest + React Testing Library for unit/component tests; Playwright config included for end-to-end testing.
Backend: Pytest with httpx-based API tests covering the analysis pipeline and endpoints (backend/tests/).


Deployment

The frontend builds to a static site (npm run build → dist/) and can be hosted on any static host (Vercel, Netlify, GitHub Pages, etc.). The backend is a standard FastAPI app and can be deployed on any ASGI-compatible host (Render, Railway, Fly.io, a VPS with uvicorn/gunicorn, etc.). When deploying separately, set VITE_API_URL on the frontend to the backend's public URL and update CORS settings in backend/main.py accordingly.

Roadmap


Persistent storage for sessions (currently in-memory)
User authentication and saved report history
Support for additional file formats (JSON, Parquet)
Multi-file / multi-dataset comparison
