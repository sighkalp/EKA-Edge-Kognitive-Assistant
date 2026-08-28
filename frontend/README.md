# Frontend Dashboard

Architecture Version: 1.1 — Architecture Standardized
Last Updated: 2026-08-29

**Responsibility**: Provide the user-facing mission dashboard and client-side interaction layer.

---

## Overview

The frontend is a Next.js application built with React and TypeScript. It serves as the user interface for mission monitoring and operations, but it is not the application backend.

The frontend is responsible for:
- visualizing mission state and events
- presenting alerts and safety status
- allowing user queries and operator actions
- rendering experiment progress and timelines
- consuming FastAPI responses over REST and WebSocket

**Stack**:
- Next.js
- React
- TypeScript
- Axios
- WebSocket

---

## Ownership Model

```text
Frontend (Next.js/React/TypeScript)
        |
        | Axios requests to FastAPI
        | WebSocket stream from FastAPI
        v
FastAPI Backend
```

This repository standard explicitly keeps Next.js as a frontend-only layer. It does not use Next.js API Routes as the application backend.

---

## Local Development

```bash
cd frontend
npm install
npm run dev
```

Development URL:
- http://localhost:3000

---

## Production Build

```bash
cd frontend
npm run build
npm start
```

This distinguishes development mode (`npm run dev`) from production mode (`npm run build` + `npm start`).

---

## Key UI Areas

### Mission dashboard
- live experiment overview
- procedure progress
- current step context
- status banners and alert summaries

### Video and monitoring view
- object detection overlays
- current mission camera feed
- system status indicators

### Knowledge interface
- user questions
- grounded responses from the backend
- source references

### Timeline and operational log
- mission events
- experiment transitions
- alerts and warnings history

---

## Data Access

Use Axios for REST access to FastAPI endpoints.

```ts
const response = await axios.get('http://localhost:8000/api/mission/mission_001/state');
```

Use WebSocket for real-time updates from the backend.

```ts
const socket = new WebSocket('ws://localhost:8000/ws/mission/mission_001');
```

---

## Testing

```bash
cd frontend
npm test
```

Recommended stack:
- Vitest
- React Testing Library

---

## Deployment

The frontend is deployed in Docker as a Next.js application and communicates with the FastAPI backend and PostgreSQL + pgvector data layer.

---

## Standardization Notes

- Next.js is frontend-only.
- The backend remains Python + FastAPI.
- No backend logic is implemented in Next.js API Routes.
- The frontend consumes the backend contract rather than directly invoking internal modules.
