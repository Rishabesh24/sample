# Green Home Score - Project Walkthrough

Welcome to the Green Home Score project! This document outlines the architecture, the implementation phases we've completed, and a guide on how everything fits together.

## Overview 1

The Green Home Score is a sophisticated platform that calculates a property's energy efficiency based on 7 strict criteria:
1. **EPC Rating** (25%)
2. **Energy Consumption Intensity (ECI)** (20%)
3. **Insulation** (15%)
4. **Heating System** (15%)
5. **Windows & Glazing** (10%)
6. **Renewable Energy** (10%)
7. **Smart Energy Management** (5%)

## Architecture

The project is structured with a clear separation of concerns, ensuring that the frontend remains highly responsive while complex calculations and machine learning tasks are handled securely on the backend.

### Frontend (Next.js App Router)
- **UI Framework:** React with TypeScript, Tailwind CSS, Lucide React, and Recharts.
- **State Management:** Handled locally via `src/lib/context.tsx`.
- **Key Pages:**
  - `/`: Dashboard displaying the Green Home Score, component breakdowns, and provisional points.
  - `/assess`: A 4-step wizard to collect property data and trigger calculations.
  - `/assistant`: An interactive AI chat interface.
  - `/recommendations`: An interactive simulator for retrofit options.

### Backend (Next.js Route Handlers)
- **API Routes (`/api/*`):** Serve as the secure bridge between the frontend and external services.
- **Scoring Engine:** Housed securely in `src/lib/scoringEngine.ts`. This ensures that calculation logic (and any adjustments to weights or formulas) is authoritative and cannot be manipulated by the client.
- **Database (Supabase):** Configured to store properties, assessments, simulation jobs, and retrofit catalogues using strict Row Level Security (RLS) policies.

### Simulation Worker (Python)
- **Role:** Handles complex physics-based building energy simulations via EnergyPlus.
- **API (FastAPI):** Exposes an endpoint (`/api/jobs/run`) that Next.js calls to offload heavy calculations. 
- **Structure:** Located in the `worker/` directory, complete with a `Dockerfile` for easy containerization.

## Phase 1 Highlights: Frontend & Logic

In Phase 1, we established the core user experience and strict adherence to the scoring formulas:
- **Strict Calculations:** Implemented the exact mathematical formulas provided for ECI, Insulation, Renewables, etc.
- **Incomplete Assessments:** If any of the 7 components are missing data, the final Green Score is explicitly set to `null` and displayed as "Unavailable".
- **Provisional Points:** Instead of a final score, the dashboard clearly distinguishes and displays the sum of successfully calculated components as "Provisional Points".
- **Zero vs. Null:** Missing data does not count as a 0 score; it is securely tracked as `null` to prevent skewed averages.

## Phase 2 Highlights: Backend & Intelligence

In Phase 2, we built the backend architecture while seamlessly integrating it into the working frontend:
- **Database Migrations:** Created robust Supabase SQL migrations (`supabase/migrations/20261011000000_initial_schema.sql`) to handle all necessary data models and enforce Row Level Security.
- **Server-Side APIs:** Built routes for `assessments`, `properties`, `simulations`, `recommendations`, and an AI `assistant`.
- **Graceful Fallbacks:** The frontend attempts to use the Next.js APIs. If credentials (like `SUPABASE_URL` or `OPENAI_API_KEY`) are missing, the system gracefully falls back to local "Demo Mode" calculations—ensuring the UI never breaks.
- **AI Integration:** The `/api/assistant` route safely contacts OpenAI strictly using server-side keys, and prompts the LLM to explain the assessment without fabricating financial numbers or scores.

## Setup & Configuration

To transition the project from "Demo Mode" to a fully live backend, follow these steps:

1. **Environment Variables:**
   Copy `.env.example` to `.env.local` and configure your keys:
   ```env
   NEXT_PUBLIC_SUPABASE_URL=...
   NEXT_PUBLIC_SUPABASE_ANON_KEY=...
   OPENAI_API_KEY=...
   WORKER_API_URL=http://localhost:8000
   ```

2. **Database Initialization:**
   If using the Supabase CLI, run:
   ```bash
   supabase start
   supabase db push
   ```

3. **Running the Python Worker:**
   Open a separate terminal instance:
   ```bash
   cd worker
   python -m venv venv
   source venv/bin/activate
   pip install -r requirements.txt
   uvicorn main:app --reload
   ```

4. **Running the Next.js Frontend:**
   ```bash
   npm install
   npm run dev
   ```

## Missing Data / Future Phases

- **EnergyPlus Executable:** The Python worker is currently stubbed to handle requests gracefully. A full EnergyPlus installation and EPW weather files are required on the host system to run true physics simulations.
- **Machine Learning (Recommendations):** The recommendations API currently returns a `pending_unconfigured` state, as we are awaiting the ML training dataset and defined ranking formulas. We chose not to invent arbitrary financial payback periods.
