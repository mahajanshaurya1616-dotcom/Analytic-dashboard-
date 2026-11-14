# Trailhead Analytics

A custom, full-stack analytics dashboard built from scratch (no Power BI / Tableau).
It pulls raw **sales**, **marketing spend** and **customer feedback** data from REST APIs,
cleans it on a Node/Express backend, and serves it to a **React + TailwindCSS** front-end
that uses **Axios** to fetch the data and **Recharts** to render interactive charts.

## How data flows

```
 Sales API ─┐
 Marketing ─┼─► connectors.js (Axios, cached) ─► clean.js ─► analytics.js ─► GET /api/dashboard ─► React (Axios) ─► charts
 Feedback ──┘      fetch raw JSON               fix types,     KPIs, trends,      one JSON payload        filters + click-to-filter
                                                dedupe, drop    breakdowns
                                                bad rows
```

## Features

- Three upstream REST sources fetched in parallel, with a short server-side cache
- Data cleaning: string-money to numbers, inconsistent casing/whitespace, duplicate orders, missing values, invalid dates and ratings
- KPIs: revenue, orders, average order value, marketing spend, ROAS, cost per lead, average rating
- Interactive charts: monthly revenue vs spend, revenue by region (click a bar to filter everything), spend and leads by channel, sentiment donut
- Date-range and region filters, refresh button, loading and error states
- Data-quality footer showing how many rows were kept after cleaning

## Tech stack

| Layer | Tools |
|-------|-------|
| Front-end | React 18, Vite, TailwindCSS, Axios, Recharts |
| Back-end | Node.js, Express, Axios |

## Getting started

Requires Node.js 18+.

```bash
git clone <your-repo-url>
cd analytics-dashboard
npm run install:all
cp backend/.env.example backend/.env
npm run dev
```

- Front-end: http://localhost:5173
- API: http://localhost:4000/api/dashboard

## Using real data

The backend ships with built-in mock sources (deliberately messy) at `/mock-sources/*`
so the project works out of the box. To use live APIs, set these in `backend/.env`:

```
SALES_API_URL=https://your-service.example.com/sales
MARKETING_API_URL=https://your-service.example.com/marketing
FEEDBACK_API_URL=https://your-service.example.com/feedback
```

Each endpoint should return a JSON array (or `{ "data": [...] }`) with these fields.
If your APIs use different names, adjust the mapping in `backend/src/clean.js`.

| Source | Fields |
|--------|--------|
| Sales | `order_id, date, region, product, units, revenue` |
| Marketing | `date, channel, spend, impressions, clicks, leads` |
| Feedback | `id, date, region, product, rating (1-5), comment` |

## API

`GET /api/dashboard?from=YYYY-MM-DD&to=YYYY-MM-DD&region=North`

Returns `kpis`, `trend`, `byRegion`, `byChannel`, `topProducts`, `sentiment`,
`ratingDistribution`, `recentFeedback`, `regions` and a `quality` report.

## Project structure

```
analytics-dashboard/
├── backend/
│   ├── server.js            # Express app and routes
│   └── src/
│       ├── mockSources.js   # Simulated upstream REST APIs
│       ├── connectors.js    # Fetches + caches the raw data
│       ├── clean.js         # Cleaning rules
│       └── analytics.js     # KPI and chart aggregations
└── frontend/
    └── src/
        ├── App.jsx          # Layout, filters, data fetching
        ├── api.js           # Axios client
        └── components/      # KpiStrip, Charts, Tables, Panel
```

## Deploying

- Front-end: `npm run build` then host `frontend/dist` (Vercel, Netlify). Set `VITE_API_URL` to your API's URL.
- Back-end: deploy `backend/` to Render, Railway or Fly.io and set the environment variables above.

## Ideas to extend

- Persist cleaned data in PostgreSQL and schedule refreshes
- Add CSV export, drill-down pages per product, or user login
- Add unit tests for `clean.js` and `analytics.js`
