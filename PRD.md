# Product Requirements Document (PRD)
# TripGuard — Smart Trip Intelligence

**Version:** 1.0
**Date:** March 2, 2026
**Status:** Active Development

---

## 1. Executive Summary

TripGuard is a context-aware trip intelligence web application that helps users plan trips by analyzing timing, risks, and logistics in real time. Users enter a destination, departure time, and transport mode, and the app returns a comprehensive assessment — including risk warnings, smart suggestions, and AI-powered reasoning — to help them arrive on time and avoid common travel pitfalls.

The application features a specialized **Flight Intelligence** mode that detects airport destinations and provides live flight status tracking, delay alerts, and gate/terminal information for users catching flights or picking someone up.

---

## 2. Problem Statement

Travelers frequently misjudge timing when heading to destinations, especially when multiple variables are involved — venue closing hours, traffic patterns, airport security wait times, and flight delays. Existing map and travel apps show estimated travel time but don't provide holistic, context-aware advice that considers the purpose of the trip, real-time conditions, and actionable alternatives.

**TripGuard fills this gap** by combining route estimation with intelligent analysis, delivering proactive warnings and suggestions rather than just directions.

---

## 3. Target Users

| Persona | Description |
|---|---|
| **Daily Commuter** | Needs to arrive at appointments, events, or venues on time and wants awareness of timing risks. |
| **Air Traveler** | Catching a flight and wants to know if their departure plan gives enough buffer for check-in, security, and boarding — especially when flights are delayed. |
| **Airport Pickup Driver** | Picking up a passenger and wants real-time arrival status so they can time their departure optimally. |
| **Event-Goer** | Heading to a restaurant, concert, gym, or park and wants to ensure they arrive before closing or event start. |

---

## 4. Core Features

### 4.1 Trip Planning Form

- **Starting Location**: Search-based input with autocomplete (via OpenStreetMap/Nominatim) and "Use My Location" GPS detection.
- **Destination**: Search-based input with autocomplete, interactive Leaflet map selection, and optional structured address fields (address, city, state, postal code).
- **Event / Purpose** (optional): Free-text field to describe the reason for travel (e.g., "Book signing", "Doctor appointment").
- **Departure Date & Time**: Date picker and time input with automatic timezone detection.
- **Transport Mode**: Selection among driving, transit, walking, and cycling — each with appropriate speed estimates.
- **Notes** (optional): Free-text field for additional context (max 500 characters).

### 4.2 Smart Airport Detection

When the user enters a destination containing airport keywords (e.g., "JFK", "Airport", IATA codes), the app automatically:

1. Detects the airport context.
2. Prompts a **Flight Dialog** modal with two modes:
   - **Catching a Flight**: Captures airline, flight number, destination city, and scheduled departure time.
   - **Airport Pickup**: Captures airline, flight number, and expected landing time.
3. Displays a **Flight Context Banner** in the form summarizing captured flight details, editable via an inline edit button.

### 4.3 AI-Powered Trip Analysis

Upon form submission, the backend:

1. **Generates trip context**: Calculates distance (Haversine formula with 1.3x road multiplier, or coordinate-based), estimated travel time (based on transport mode speed), and estimated arrival time.
2. **Checks Redis cache**: Avoids redundant AI calls for identical plans (10-minute TTL).
3. **Calls TinyFish AI API** (if configured): Sends the trip context to a web-browsing AI agent that searches Google for real-time venue hours, flight statuses, and conditions. Falls back to deterministic mock analysis if the API key is not set.
4. **Returns structured results**: Risk assessment, suggestions, reasoning, and an overall status.

### 4.4 Analysis Results Dashboard

Results are displayed in a structured panel with the following sections:

| Section | Description |
|---|---|
| **Status Banner** | Color-coded indicator — **Good** (green), **Caution** (amber), or **High Risk** (red) — with a one-line summary. |
| **Context Summary ("At a Glance")** | Key metrics: travel time, distance (km & miles), departure/arrival times, venue hours (or flight info in flight mode). |
| **Risk Assessment** | List of identified risks, each categorized as `info`, `warning`, or `danger`, with a title and detailed description. |
| **Smart Suggestions** | Actionable tips categorized as `timing`, `alternative`, or `tip`. |
| **AI Reasoning** | Natural language explanation of the analysis logic — what the AI considered and why it reached its conclusion. |

### 4.5 Flight Intelligence

In flight mode, the analysis focuses on:

- **Live flight status**: On-time, delayed, cancelled, diverted.
- **Expected vs. actual departure/arrival time comparison**: Highlights discrepancies prominently.
- **Airport buffer calculation**: Whether the user has enough time for check-in, security, and boarding (2–3 hour buffer for departures; 20–40 min post-landing buffer for pickups).
- **Gate, terminal, and baggage claim information**.
- **Departure adjustment suggestions**: If a flight is delayed, suggests the user can leave later.

### 4.6 Session Memory & Agent State

Using Redis, the app maintains:

- **Session context**: Recent searches, last transport mode, last location (1-hour TTL).
- **Agent workflow state**: Query count, recent destinations, last analysis status (30-minute TTL). This enables progressively context-aware responses across multiple queries.

### 4.7 Theme Support

- Light and dark mode toggle available in the header.
- Themed with a "Sunset Journey" warm gradient palette (orange-to-violet).

---

## 5. Technical Architecture

### 5.1 Stack

| Layer | Technology |
|---|---|
| **Frontend** | React 18, TypeScript, Vite, Tailwind CSS, shadcn/ui (Radix primitives) |
| **Backend** | Node.js, Express 5, TypeScript (tsx) |
| **Database** | PostgreSQL (via Drizzle ORM) — provisioned but not actively used for core features |
| **Caching / State** | Redis (ioredis) — optional; app gracefully degrades without it |
| **AI Engine** | TinyFish AI API (SSE-based web automation agent) |
| **Maps** | Leaflet + React-Leaflet, OpenStreetMap/Nominatim for geocoding |
| **Routing** | Wouter (client-side) |
| **Data Validation** | Zod (shared schemas between client and server) |
| **State Management** | TanStack React Query (server state), React Hook Form (form state) |

### 5.2 Project Structure

```
├── client/                  # Frontend React application
│   ├── src/
│   │   ├── components/      # UI components (PlanForm, LocationPicker, FlightDialog, result cards)
│   │   ├── hooks/           # Custom React hooks (useMobile, useToast)
│   │   ├── lib/             # Utilities (queryClient, cn helper)
│   │   ├── pages/           # Page components (Home, NotFound)
│   │   ├── App.tsx          # Root app with routing
│   │   └── main.tsx         # Entry point
│   └── index.html
├── server/                  # Backend Express application
│   ├── index.ts             # Server entry, middleware setup
│   ├── routes.ts            # API endpoints and business logic
│   ├── redis.ts             # Redis client and helper functions
│   ├── storage.ts           # Storage interface (memory-based)
│   ├── vite.ts              # Vite dev server middleware
│   └── static.ts            # Production static file serving
├── shared/                  # Shared types and schemas
│   └── schema.ts            # Zod schemas, TypeScript interfaces
├── script/
│   └── build.ts             # Production build script (Vite + esbuild)
├── vite.config.ts           # Vite configuration
├── drizzle.config.ts        # Drizzle ORM configuration
├── tsconfig.json            # TypeScript configuration
└── package.json             # Dependencies and scripts
```

### 5.3 API Endpoints

| Method | Endpoint | Description |
|---|---|---|
| `POST` | `/api/agent/analyze-plan` | Submit a trip plan for AI analysis. Returns risks, suggestions, reasoning, and status. |
| `GET` | `/api/redis-status` | Check Redis connection and configuration status. |
| `GET` | `/api/cache-test` | Diagnostic: verify Redis cache read/write operations. |
| `GET` | `/api/agent-memory-test` | Diagnostic: view current session memory and agent state. |

### 5.4 Data Flow

```
User Input → PlanForm (Zod validation)
  → POST /api/agent/analyze-plan
    → Cache check (Redis)
    → Context generation (distance, timing, venue hours)
    → TinyFish AI call (or mock fallback)
    → Session & agent state update (Redis)
    → AnalysisResult response
  → Results Dashboard (status, risks, suggestions, reasoning)
```

---

## 6. External Dependencies

| Service | Purpose | Required? |
|---|---|---|
| **TinyFish AI API** | Real-time web-browsing AI for venue hours, flight status, and contextual analysis | Optional — falls back to deterministic mock analysis |
| **Redis** | Caching, session memory, agent state | Optional — app works without it (no caching, no memory) |
| **OpenStreetMap / Nominatim** | Geocoding and location search autocomplete | Yes (free, no API key needed) |
| **PostgreSQL** | Database (provisioned via Drizzle ORM) | Provisioned but not actively used for core features |

---

## 7. Environment Variables

| Variable | Description | Required? |
|---|---|---|
| `TINYFISH_API_KEY` | API key for TinyFish AI agent | No (mock analysis used if absent) |
| `REDIS_HOST` | Redis server hostname | No (runs without cache) |
| `REDIS_PORT` | Redis server port (default: 6379) | No |
| `REDIS_PASSWORD` | Redis authentication password | No |
| `DATABASE_URL` | PostgreSQL connection string | Yes (for Drizzle ORM config) |
| `PORT` | Server port (default: 5000) | No |

---

## 8. Non-Functional Requirements

| Requirement | Detail |
|---|---|
| **Performance** | Cached responses return instantly; uncached AI analysis completes within 10–30 seconds. |
| **Reliability** | Graceful degradation: app functions without Redis (no caching) and without TinyFish API (mock analysis). |
| **Responsiveness** | Mobile-friendly responsive layout; adapts from two-column desktop to stacked mobile view. |
| **Accessibility** | Built on Radix UI primitives with semantic HTML and ARIA attributes. |
| **Security** | Input validation via Zod on both client and server; no user authentication required for core features. |
| **Theming** | Light and dark mode support with consistent design tokens. |

---

## 9. Future Considerations

- **User Accounts & History**: Persistent trip history with login, replacing IP-based session tracking.
- **Real-Time Traffic Integration**: Direct traffic API integration (e.g., Google Maps, HERE) for more accurate travel time estimates.
- **Push Notifications**: Alert users of flight status changes or timing risks as departure time approaches.
- **Multi-Stop Trips**: Support for itineraries with multiple destinations.
- **Weather Integration**: Direct weather API data for departure and destination conditions.
- **Sharing**: Allow users to share trip analysis results with others.

---

## 10. Success Metrics

| Metric | Target |
|---|---|
| **Analysis Accuracy** | 90%+ of timing assessments align with actual travel outcomes. |
| **Response Time** | Cached: < 200ms. AI-powered: < 15 seconds. |
| **User Engagement** | Average 2+ analyses per session (tracked via agent state `queryCount`). |
| **Graceful Degradation** | 100% uptime regardless of Redis or TinyFish availability. |
