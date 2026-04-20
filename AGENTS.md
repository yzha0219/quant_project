# AI Agent Instructions

## Project Overview

Quantitative trading strategy backtesting platform with FastAPI backend and Vue 3 frontend.

## Quick Start

### Backend
```bash
cd quant_backend
pip install -r requirements.txt
python run.py
# Runs on http://localhost:8000, API at /api/v1
```

### Frontend
```bash
cd quant_frontend
npm install
npm run dev
# Runs on http://localhost:3000
```

## Architecture

| Layer | Technology | Port |
|-------|------------|------|
| Backend API | FastAPI + uvicorn | 8000 |
| Frontend | Vue 3 + Vite | 3000 |
| Data Source | akshare (China stocks) | - |

## Key Components

### Backend Services
- [data_service.py](quant_backend/app/services/data_service.py) — Stock data fetching via akshare
- [indicator_service.py](quant_backend/app/services/indicator_service.py) — Technical indicators (MA, RSI, MACD, etc.)
- [backtest_service.py](quant_backend/app/services/backtest_service.py) — Strategy backtesting engine
- [optimizer_service.py](quant_backend/app/services/optimizer_service.py) — Strategy parameter optimization

### Frontend Components
- [BacktestForm.vue](quant_frontend/src/components/BacktestForm.vue) — Backtest configuration form
- [OptimizerPanel.vue](quant_frontend/src/components/OptimizerPanel.vue) — Strategy optimization UI
- [ComprehensiveCharts.vue](quant_frontend/src/components/ComprehensiveCharts.vue) — Results visualization

## API Endpoints

- `GET /api/v1/stock/{symbol}` — Fetch stock data
- `GET /api/v1/indicators/{symbol}` — Calculate technical indicators
- `POST /api/v1/backtest` — Run backtest
- `POST /api/v1/optimize` — Optimize strategy parameters

## Development Notes

- Backend proxy configured in [vite.config.js](quant_frontend/vite.config.js) — rewrites `/api` → `/api/v1`
- CORS allows all origins in development ([config.py](quant_backend/app/core/config.py))
- SSL optional; set `USE_HTTPS=true` and provide certs in `ssl/` directory