# Sales Lead Analyzer UI

A React + TypeScript frontend for the [Sales Lead Analyzer](https://github.com/dangerousLefty/sales-lead-analyzer) Spring Boot backend.

The UI lets a user submit an inbound customer message and view leads that were qualified by the backend's asynchronous qualification pipeline (local async, AWS SQS, or Apache Kafka, selected server-side).

## Features

- Submit inbound customer messages to the backend's `POST /api/v1/messages` endpoint
- Load qualified leads on mount from `GET /api/v1/leads`
- Refresh the leads list on demand after the asynchronous qualification completes
- Controlled input with disabled-when-empty submit button
- Loading, error, and success states for both the submit form and the leads list
- Typed API responses using TypeScript type aliases

## Tech Stack

- React 18
- TypeScript
- Vite (dev server and build tool)
- ESLint

## Prerequisites

- Node.js 20+
- The [Sales Lead Analyzer backend](https://github.com/dangerousLefty/sales-lead-analyzer) running on `http://localhost:8080` in any dispatch mode (`local`, `sqs`, or `kafka`)

## Getting Started

```bash
npm install
npm run dev
```

The dev server starts on `http://localhost:5173`.

## Available Scripts

| Script            | Purpose                                                |
| ----------------- | ------------------------------------------------------ |
| `npm run dev`     | Starts the Vite dev server with hot module reload      |
| `npm run build`   | Builds the production bundle into `dist/`              |
| `npm run preview` | Serves the production build locally for verification   |
| `npm run lint`    | Runs ESLint over the project                           |

## Configuration

The backend base URL is currently hard-coded to `http://localhost:8080` in `src/App.tsx`. For a production build this should move to an environment variable read via `import.meta.env.VITE_API_BASE_URL`.

## CORS

The Vite dev server runs on port 5173 and the backend on port 8080, which are different origins as far as the browser is concerned. The backend must therefore allow cross-origin requests from `http://localhost:5173`. The Sales Lead Analyzer backend includes a `CorsConfig` that grants this permission for local development.

## Architecture

The current UI is a single-component application:

- `App` owns all state: submit form state (`content`, `submitting`, `error`, `lastCreated`) and leads-list state (`leads`, `loadingLeads`, `leadsError`).
- `LeadsList` is a presentational component that receives the leads array via props and renders each item.

Data flow:

```
User types in textarea
        ↓
setContent(...)  →  App re-renders with new content
        ↓
User clicks Submit
        ↓
handleSubmit()  →  POST /api/v1/messages
        ↓
setLastCreated(response)  →  green success message renders
        ↓
Backend Kafka pipeline qualifies the message asynchronously
        ↓
User clicks Refresh
        ↓
fetchLeads()  →  GET /api/v1/leads  →  setLeads(...)  →  LeadsList re-renders
```

The refresh is manual by design so that the asynchronous qualification pipeline (which may take a few seconds) is observable end to end.

## Planned Improvements

- Move the backend base URL to an environment variable
- Split the submit form into its own component
- Extract API calls into a `src/api/` module
- Add React Router with a dedicated `/leads/:id` detail page
- Replace inline styles with Tailwind or a component library
- Add optimistic UI or polling so new leads appear without a manual refresh
- Add Vitest + React Testing Library coverage for the submit and refresh flows
- Add a production build stage in the backend's `docker-compose.yml`
