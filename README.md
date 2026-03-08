# API Gateway

A production-grade API Gateway built with Node.js, Express, Redis, and MongoDB.

## Features

- **Sliding Window Rate Limiting** — powered by Redis sorted sets and Lua scripts for atomic operations
- **JWT Authentication** — verifies tokens and propagates user identity to downstream services
- **Request Proxying** — forwards valid requests to downstream services with tracing headers
- **Request Logging** — logs every request asynchronously to MongoDB
- **Dashboard API** — query stats, logs, and active rate limits

## Tech Stack

- Node.js (ES Modules)
- Express
- Redis (ioredis)
- MongoDB (Mongoose)
- http-proxy-middleware

## Getting Started

### Prerequisites

- Node.js v18+
- MongoDB running locally
- Redis running locally

### Installation

\`\`\`bash
git clone https://github.com/yourusername/api-gateway.git
cd api-gateway
npm install
\`\`\`

Copy the example env file and fill in your values:
\`\`\`bash
cp .env.example .env
\`\`\`

### Running

Terminal 1 — start the dummy downstream service:
\`\`\`bash
node downstream.mjs
\`\`\`

Terminal 2 — start the gateway:
\`\`\`bash
npm run start:dev
\`\`\`

## Dashboard Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | /dashboard/stats | Total requests, blocked requests, top IPs |
| GET | /dashboard/logs | Paginated request logs with filters |
| GET | /dashboard/logs/:ip | All activity from a specific IP |
| GET | /dashboard/rate-limits | Who is currently rate limited |

## How Rate Limiting Works

Each IP or authenticated user gets a Redis sorted set. Every request adds a timestamped entry. On each new request the gateway:

1. Removes entries older than the window
2. Counts remaining entries
3. Rejects if over the limit, allows and adds entry if under

All four steps run atomically via a Lua script — preventing race conditions under high load.
