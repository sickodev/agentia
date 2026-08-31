# HumanScope / Agentville 🏙️🤖

[![Java 21](https://img.shields.io/badge/Java-21%2B%20Loom-ED8B00?style=flat&logo=openjdk&logoColor=white)](https://openjdk.org/projects/loom/)
[![Spring Boot 3.3+](https://img.shields.io/badge/Spring%20Boot-3.3%2B-6DB33F?style=flat&logo=springboot&logoColor=white)](https://spring.io/projects/spring-boot)
[![React 18](https://img.shields.io/badge/React-18-61DAFB?style=flat&logo=react&logoColor=black)](https://react.dev/)
[![TypeScript](https://img.shields.io/badge/TypeScript-5.5-3178C6?style=flat&logo=typescript&logoColor=white)](https://www.typescriptlang.org/)
[![Vite](https://img.shields.io/badge/Vite-5.4-646CFF?style=flat&logo=vite&logoColor=white)](https://vitejs.dev/)
[![SQLite WAL](https://img.shields.io/badge/SQLite-WAL%20Mode-003B57?style=flat&logo=sqlite&logoColor=white)](https://www.sqlite.org/)
[![Tailwind CSS](https://img.shields.io/badge/Tailwind-3.4-38B2AC?style=flat&logo=tailwind-css&logoColor=white)](https://tailwindcss.com/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

> **Real-Time Autonomous Generative Multi-Agent Simulation Sandbox**  
> Inspired by Stanford's *Generative Agents* (Park et al., 2023), built with **Java 21 Virtual Threads (Project Loom)**, **Spring Boot 3.3**, **SQLite WAL**, and an interactive **2.5D Isometric Neo-Brutalist React Canvas**.

---

## 📖 Table of Contents

- [Overview](#-overview)
- [Key Features](#-key-features)
- [System Architecture](#-system-architecture)
- [Monorepo Structure](#-monorepo-structure)
- [Tech Stack](#-tech-stack)
- [Prerequisites](#-prerequisites)
- [Quickstart Guide](#-quickstart-guide)
  - [1. Clone and Navigate](#1-clone-and-navigate)
  - [2. Configure Environment Variables](#2-configure-environment-variables)
  - [3. Start the Backend](#3-start-the-backend)
  - [4. Start the Frontend](#4-start-the-frontend)
  - [5. Run via Docker Compose](#5-run-via-docker-compose)
- [Monorepo NPM Scripts](#-monorepo-npm-scripts)
- [API & WebSocket Protocol Overview](#-api--websocket-protocol-overview)
- [Automated Testing & Verification](#-automated-testing--verification)
- [Cloud Deployment](#-cloud-deployment)
- [License](#-license)

---

## 🌟 Overview

**Agentia** simulates 10–15+ autonomous AI citizens living in a 2.5D isometric town. Each agent perceives its surroundings, makes decisions driven by free-tier LLM cognition (Groq Llama-3 / Google Gemini 1.5 Flash), forms episodic memories, experiences dynamic moods, manages a wallet balance, and engages in emergent social and economic behaviors.

Unlike traditional single-threaded or GIL-bound Python multi-agent demos, Agentia leverages **Java 21 Project Loom Virtual Threads** to execute each agent's cognitive tick on a dedicated lightweight virtual thread with sub-millisecond scheduling and non-blocking I/O, streaming world state at 1 Hz via STOMP WebSockets to a neo-brutalist isometric web diorama.

---

## ✨ Key Features

- 🧵 **High-Concurrency Virtual Thread Engine:** 1 dedicated Virtual Thread per autonomous agent (`Executors.newVirtualThreadPerTaskExecutor()`), enabling 50+ concurrent agents with $< 150\text{ MB}$ JVM heap.
- 🧠 **Cost-Resilient LLM Cognition:** Built-in multi-tier cache using SHA-256 prompt hashing in SQLite (`llm_cache` table) paired with a Token Bucket rate limiter (30 RPM) and deterministic heuristic behavior trees for zero downtime during API rate limits.
- 🎨 **2.5D Isometric Neo-Brutalist Diorama:** Custom HTML5 2D Canvas rendering isometric buildings (Bakery, Market, Café, Town Hall, Workshop, Park, Residences) with 60 FPS linear interpolation (`lerp`) and warm gold dialogue bubbles.
- 🔍 **Instant Mind-Reading Inspector:** Click any stick-figure citizen to reveal their real-time thoughts, personality traits, occupation, live mood chip, wallet balance, and chronological episodic memory stream.
- ⚡ **God-Mode Dynamic Event Injection:** Administrator palette allowing live triggers of global environmental events (Market Crashes, Town Festivals, Thunderstorms, Bank Heists) that immediately ripple through agent decision-making.
- ⏪ **Deterministic Time-Travel DVR Replay:** Event-sourced tick snapshots stored in SQLite (`tick_snapshots` table) with an interactive scrubber bar to review historical simulation runs.

---

## 🏛️ System Architecture

```
+----------------------------------------------------------------------------------------------------+
|                                      SYSTEM ARCHITECTURE                                           |
+----------------------------------------------------------------------------------------------------+
|                                                                                                    |
|    +----------------------------------------------------+                                          |
|    |           React 18 + Vite Web Client               |                                          |
|    |  (2.5D Isometric Canvas, Zustand, STOMP/WS Client) |                                          |
|    +-------------------------+--------------------------+                                          |
|                              ^                                                                     |
|             STOMP / WebSocket | HTTP REST (God-Mode / Replay)                                       |
|                              v                                                                     |
|    +----------------------------------------------------+                                          |
|    |      Java 21 Spring Boot Application Core          |                                          |
|    |  +----------------------------------------------+  |                                          |
|    |  |       Simulation Coordinator / Tick Engine   |  |                                          |
|    |  |     (Phaser / Virtual Thread Task Executor)  |  |                                          |
|    |  +----------------------+-----------------------+  |                                          |
|    |                         |                                                                     |
|    |         +---------------+---------------+                                                     |
|    |         |                               |                                                     |
|    |         v                               v                                                     |
|    |  +---------------+             +-----------------+                                            |
|    |  | Agent Thread  | (N threads) | Environment     |                                            |
|    |  |   (Virtual)   |             | Agent (Virtual) |                                            |
|    |  +-------+-------+             +--------+--------+                                            |
|    |          |                              |                                                     |
|    |          +--------------+---------------+                                                     |
|    |                         |                                                                     |
|    |                         v                                                                     |
|    |  +----------------------------------------------+                                             |
|    |  |      In-Memory Concurrent State Map          |                                             |
|    |  |       (ConcurrentHashMap<ID, AgentState>)    |                                             |
|    |  +----------------------+-----------------------+                                             |
|    |                         |                                                                     |
|    |         +---------------+---------------+                                                     |
|    |         |                               |                                                     |
|    |         v                               v                                                     |
|    |  +---------------+             +-----------------+                                            |
|    |  | LLM Gateway   |             | SQLite Storage  |                                            |
|    |  |  - SHA Cache  |             | (WAL Mode,      |                                            |
|    |  |  - RateLimit  |             |  HikariCP Pool) |                                            |
|    |  +-------+-------+             +--------+--------+                                            |
|    |          |                              |                                                     |
|    +----------|------------------------------|-----------------------------------------------------+
|               v                              v                                                     |
|    +--------------------+          +--------------------+                                          |
|    | Groq / Gemini API  |          | SQLite Database    |                                          |
|    |  (Free Tier LLM)   |          |  (agentville.db)   |                                          |
|    +--------------------+          +--------------------+                                          |
+----------------------------------------------------------------------------------------------------+
```

---

## 📁 Monorepo Structure

```
agentia/
├── backend/                             # Java 21 Spring Boot Application
│   ├── pom.xml                          # Maven build definition
│   ├── Dockerfile                       # Multi-stage JDK 21 Alpine container
│   ├── README.md                        # Backend detailed documentation
│   └── src/
│       ├── main/
│       │   ├── java/com/agentia/        # Core engine, agents, LLM gateway, WebSocket
│       │   └── resources/               # application.properties, agents.json, schema.sql
│       └── test/                        # Automated unit, concurrency & persistence tests
├── frontend/                            # React 18 + TypeScript + Vite Application
│   ├── package.json                     # Frontend dependencies and scripts
│   ├── tsconfig.json                    # TypeScript strict compiler options
│   ├── vite.config.ts                   # Vite bundler configuration
│   ├── Dockerfile                       # Multi-stage Nginx static web container
│   ├── README.md                        # Frontend detailed documentation
│   └── src/
│       ├── components/                  # Isometric Canvas, Agent Drawer, God Mode, Replay
│       ├── hooks/                       # useSimulationSocket, useAgentStore
│       ├── types/                       # Simulation, Agent, Event, and Memory TypeScript models
│       ├── App.tsx                      # Main diorama layout shell
│       └── main.tsx                     # React DOM entrypoint
├── docker-compose.yml                   # One-command full-stack container orchestration
├── package.json                         # Root monorepo orchestration scripts
└── README.md                            # Monorepo root documentation
```

---

## 🛠️ Tech Stack

| Layer | Technologies |
|---|---|
| **Backend Engine** | Java 21 (Project Loom Virtual Threads), Spring Boot 3.3+, Spring WebSocket (STOMP) |
| **Persistence & Cache** | SQLite 3 (WAL mode), HikariCP, SHA-256 In-Memory / SQLite Prompt Cache |
| **Cognition & LLM** | Groq API (Llama-3-70B/8B), Google Gemini 1.5 Flash, Resilience4j / Bucket4j |
| **Frontend Web** | React 18, TypeScript 5.5, Vite 5.4, Tailwind CSS 3.4, Lucide React, Zustand |
| **Graphics & Rendering**| HTML5 Canvas 2D (2.5D Isometric Projection, Depth Sorting, Sub-pixel Lerp) |
| **DevOps & Containers** | Docker, Docker Compose, Multi-stage Alpine builds |

---

## 📋 Prerequisites

Before running the project locally, ensure you have the following installed:

- **Java Development Kit (JDK):** Version 21 or higher ([Eclipse Temurin](https://adoptium.net/) recommended)
- **Node.js:** Version 18.x or 20.x ([NodeJS.org](https://nodejs.org/))
- **NPM:** Version 9.x or higher
- **Maven:** Version 3.9+ (or use the included `./mvnw` wrapper)
- **LLM API Key (Free Tier):**
  - [Groq API Key](https://console.groq.com/) or
  - [Google Gemini API Key](https://aistudio.google.com/)
- *(Optional)* **Docker & Docker Compose** for containerized execution

---

## 🚀 Quickstart Guide

### 1. Clone and Navigate

```bash
git clone https://github.com/your-org/agentia.git
cd agentia/development
```

### 2. Configure Environment Variables

Create an `.env` file in the root or set environment variables in your terminal:

```bash
# LLM Providers (At least one required for generative cognition)
export GROQ_API_KEY="gsk_your_groq_api_key_here"
export GEMINI_API_KEY="AIzaSy_your_gemini_api_key_here"

# Optional overrides
export SIMULATION_TICK_RATE_MS=1000
export SPRING_PROFILES_ACTIVE=dev
```

### 3. Start the Backend

From the root directory:
```bash
npm run backend:dev
```
Or directly from `backend/`:
```bash
cd backend
./mvnw spring-boot:run
```
*The backend starts at `http://localhost:8080` with WebSocket STOMP endpoint at `ws://localhost:8080/ws-agentville`.*

### 4. Start the Frontend

In a separate terminal, from the root directory:
```bash
npm run frontend:dev
```
Or directly from `frontend/`:
```bash
cd frontend
npm install
npm run dev
```
*The frontend Vite dev server will launch at `http://localhost:5173` (or `http://localhost:3000`).*

### 5. Run via Docker Compose

To run both backend and frontend in isolated production containers with SQLite persistence:

```bash
docker compose up --build
```
Access the application:
- **Frontend Diorama:** `http://localhost:3000`
- **Backend API:** `http://localhost:8080/api/health`

---

## 📦 Monorepo NPM Scripts

The root `package.json` provides unified workspace commands:

| Command | Description |
|---|---|
| `npm run frontend:dev` | Launches the Vite frontend development server on port 5173 |
| `npm run frontend:build`| Type-checks with `tsc` and compiles the frontend for production into `frontend/dist` |
| `npm run backend:dev` | Starts the Spring Boot backend application using Maven wrapper |

---

## 🔌 API & WebSocket Protocol Overview

### REST Endpoints

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/api/health` | Service health status, JVM memory statistics, and virtual thread pool health |
| `GET` | `/api/agents` | Current states of all active agents (coordinates, mood, wallet, last action) |
| `GET` | `/api/agents/{id}/memories` | Last $N$ episodic memories for an agent (`?limit=10`) |
| `POST` | `/api/events/trigger` | Inject dynamic world event (`{"type": "FESTIVAL", "description": "..."}`) |
| `POST` | `/api/simulation/pause` | Pause live simulation tick clock |
| `POST` | `/api/simulation/resume` | Resume live simulation tick clock |
| `GET` | `/api/simulation/replay` | Retrieve historical tick range for DVR replay (`?from=1&to=100`) |

### WebSocket STOMP Topics

| Destination | Type | Description |
|---|---|---|
| `/topic/town-state` | Broker Broadcast | Broadcasts full simulation world state at 1 Hz tick rate |
| `/topic/events` | Broker Broadcast | Broadcasts real-time environmental events |
| `/topic/agent/{id}` | Broker Broadcast | Targeted agent dialogue and speech bubble updates |
| `/app/god-event` | App Inbound | Client triggers dynamic world event via WebSocket |
| `/app/agent-command` | App Inbound | User sends direct message or command to an agent |

---

## 🧪 Automated Testing & Verification

Run the complete verification test suite:

```bash
# Backend test suite (Concurrency, SQLite WAL, Rate Limiting, LLM Cache)
cd backend
./mvnw clean test

# Frontend type checking and bundle validation
cd ../frontend
npm run build
```

---

## ☁️ Cloud Deployment

- **Backend (Render / Fly.io / Railway):** Deploy the multi-stage `backend/Dockerfile` with persistent disk mounted at `/app/data` to retain `agentville.db`.
- **Frontend (Vercel / Netlify / Cloudflare Pages):** Deploy `frontend/` as a static SPA configured with `VITE_BACKEND_URL` and `VITE_WS_URL`.

---

## 📄 License

This project is licensed under the MIT License.
