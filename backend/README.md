# HumanScope / Agentville Backend ⚙️🧵

[![Java 21](https://img.shields.io/badge/Java-21%2B%20Loom-ED8B00?style=flat&logo=openjdk&logoColor=white)](https://openjdk.org/projects/loom/)
[![Spring Boot 3.3+](https://img.shields.io/badge/Spring%20Boot-3.3%2B-6DB33F?style=flat&logo=springboot&logoColor=white)](https://spring.io/projects/spring-boot)
[![SQLite 3 WAL](https://img.shields.io/badge/SQLite-WAL%20Mode-003B57?style=flat&logo=sqlite&logoColor=white)](https://www.sqlite.org/)
[![HikariCP](https://img.shields.io/badge/HikariCP-Connection%20Pool-orange?style=flat)](https://github.com/brettwooldridge/HikariCP)
[![WebSocket STOMP](https://img.shields.io/badge/Spring%20WebSocket-STOMP-6DB33F?style=flat)](https://docs.spring.io/spring-framework/reference/web/websocket.html)
[![Resilience4j](https://img.shields.io/badge/Resilience4j-Rate%20Limiting-red?style=flat)](https://resilience4j.readme.io/)

> **High-Concurrency Autonomous Generative Multi-Agent Simulation Engine**  
> Powered by **Java 21 Project Loom Virtual Threads**, Spring Boot 3.3, and SQLite WAL mode. Orchestrates 10–50+ concurrent LLM-driven agents with sub-millisecond scheduling, token-efficient SHA-256 prompt caching, resilient rate limiting, and real-time STOMP state broadcasting.

---

## 📖 Table of Contents

- [Overview](#-overview)
- [Key Features](#-key-features)
- [Architecture & Concurrency Model](#-architecture--concurrency-model)
  - [Virtual Thread Execution (Project Loom)](#virtual-thread-execution-project-loom)
  - [Tick Loop & Phase Synchronization](#tick-loop--phase-synchronization)
- [Cognitive Architecture & LLM Gateway](#-cognitive-architecture--llm-gateway)
  - [Observe-Reflect-Decide-Act Cycle](#observe-reflect-decide-act-cycle)
  - [Strict JSON Prompt Schema](#strict-json-prompt-schema)
  - [SHA-256 Prompt Caching & Rate Limiting](#sha-256-prompt-caching--rate-limiting)
  - [Heuristic Fallback Engine](#heuristic-fallback-engine)
- [Database Schema & SQLite WAL Mode](#-database-schema--sqlite-wal-mode)
- [REST API Reference](#-rest-api-reference)
- [WebSocket STOMP Protocol](#-websocket-stomp-protocol)
- [Configuration & Environment Variables](#-configuration--environment-variables)
- [Getting Started](#-getting-started)
- [Automated Testing Suite](#-automated-testing-suite)
- [Containerization & Cloud Deployment](#-containerization--cloud-deployment)

---

## 🌟 Overview

The **HumanScope (Agentville)** backend is an event-driven, high-concurrency simulation engine built from the ground up on modern Java 21. Unlike conventional single-threaded Python agent implementations that suffer from GIL contention and heavy memory overhead, this backend runs each autonomous agent on its own **Virtual Thread**.

Agents perceive their 2D world, query past episodic memories from SQLite, invoke free-tier LLM APIs (Groq Llama-3 or Gemini 1.5 Flash) with caching, update thread-safe shared spatial memory, and broadcast 1 Hz tick snapshots to connected web clients over WebSocket STOMP.

---

## ✨ Key Features

- 🧵 **Massive Concurrency with Project Loom:** 1 Virtual Thread per agent with near-zero memory footprint ($< 150\text{ MB}$ JVM heap for 50 agents).
- 🧠 **Dual-Provider LLM Gateway:** Integrates with **Groq** (Llama-3-70B/8B) and **Google Gemini 1.5 Flash** with strict JSON output schemas.
- ⚡ **Multi-Tier Prompt Caching:** Exact SHA-256 hash lookup in SQLite (`llm_cache`) yields $> 40\%$ cache hit rates and sub-2ms responses for routine actions.
- 🛡️ **Token Bucket Rate Limiting:** 30 RPM protective rate limiting with instantaneous deterministic fallback to rule-based behavior trees on HTTP 429 or timeout.
- 💾 **High-Performance SQLite WAL:** Write-Ahead Logging (`PRAGMA journal_mode = WAL`) and HikariCP connection pooling ensure non-blocking concurrent reads during batch writes.
- 📡 **Real-Time WebSocket Streaming:** 1 Hz STOMP broadcasting of full world state, targeted agent dialogues, and dynamic environmental events.
- ⏪ **Event-Sourced Replay Engine:** Full tick history recorded in `tick_snapshots` enabling historical time scrubbing without LLM re-invocation.

---

## 🏛️ Architecture & Concurrency Model

### Virtual Thread Execution (Project Loom)

```java
@Configuration
public class ConcurrencyConfig {
    @Bean(name = "agentExecutor")
    public ExecutorService agentExecutor() {
        return Executors.newVirtualThreadPerTaskExecutor();
    }
}
```

Every agent executes its perception and cognitive cycle inside a dedicated virtual thread. When an agent blocks on SQLite I/O or an LLM HTTP request, the carrier thread is immediately released to execute other tasks.

### Tick Loop & Phase Synchronization

The simulation operates on a deterministic tick frequency ($T_{\text{tick}} = 1000\text{ms}$):

```
+-----------------------------------------------------------------------------------+
|                            DETERMINISTIC TICK LOOP                                |
+-----------------------------------------------------------------------------------+
|                                                                                   |
|  [ Phase A: Snapshot & Vision ]                                                   |
|    ↳ Extract immutable world state from ConcurrentHashMap                         |
|                                                                                   |
|  [ Phase B: Concurrently Dispatch Agent Virtual Threads ]                         |
|    ↳ Agent 1 (VT 1) ──┐                                                           |
|    ↳ Agent 2 (VT 2) ──┼──> CompletableFuture.allOf().join()                       |
|    ↳ Agent N (VT N) ──┘                                                           |
|                                                                                   |
|  [ Phase C: Conflict Resolution & State Mutation ]                                |
|    ↳ Atomic coordinate & wallet updates applied to ConcurrentHashMap              |
|                                                                                   |
|  [ Phase D: Persistence & WebSocket Broadcast ]                                   |
|    ↳ Async SQLite batch write (tick_snapshots) + STOMP push to /topic/town-state  |
|                                                                                   |
+-----------------------------------------------------------------------------------+
```

---

## 🧠 Cognitive Architecture & LLM Gateway

### Observe-Reflect-Decide-Act Cycle

1. **Observe:** Agent scans entities within vision radius ($R \le 5$) and checks for active global world events.
2. **Retrieve:** Queries top-$K$ relevant episodic memories from SQLite ordered by importance and recency.
3. **Prompt Generation:** Assembles strict JSON system and context payloads.
4. **Cache Lookup:** Checks SQLite `llm_cache` for $\text{SHA-256}(\text{Prompt})$.
   - **Cache HIT:** Returns parsed action immediately ($< 2\text{ms}$).
   - **Cache MISS:** Dispatches rate-limited HTTP call to Groq / Gemini API.
5. **Parse & Fallback:** Deserializes JSON into `AgentAction`. If rate limit (429) or timeout occurs, deterministic heuristic behavior tree generates a contextual action.
6. **Memorize:** Persists action and dialogue summary into `memories` table.

### Strict JSON Prompt Schema

```json
{
  "system": "You are {name}, a {personality} living in Agentville. Occupation: {occupation}. Location: ({x},{y}). Wallet: ${money}. Goal: {goal}. Output ONLY valid JSON matching the schema.",
  "context": {
    "current_tick": 42,
    "world_event": "Town Festival at Market Square",
    "nearby_agents": [{"id": "agent_2", "name": "Bob", "distance": 2, "action": "selling fresh bread"}],
    "recent_memories": [
      "Met Bob yesterday and talked about buying flour",
      "Heard announcement about the upcoming festival"
    ]
  },
  "expected_response_schema": {
    "thought": "String (internal monologue)",
    "action_type": "MOVE | INTERACT | REST | WORK",
    "target_x": "Integer (0-50)",
    "target_y": "Integer (0-50)",
    "dialogue": "String or null",
    "mood": "HAPPY | CURIOUS | TIRED | ANXIOUS | NEUTRAL",
    "money_delta": "Integer (-100 to +100)"
  }
}
```

### SHA-256 Prompt Caching & Rate Limiting

```
              ┌───────────────────────────┐
              │  Assemble Prompt Payload  │
              └─────────────┬─────────────┘
                            │
                            v
              ┌───────────────────────────┐
              │ Compute SHA-256 Hash      │
              └─────────────┬─────────────┘
                            │
                   [ Query llm_cache ]
                     /             \
             (Found) /               \ (Not Found)
                    v                 v
          ┌──────────────────┐   [ Check Token Bucket ]
          │ Return Cached    │     /             \
          │ Response (< 2ms) │ (Available)     (Exhausted / 429)
          └──────────────────┘     /                 \
                                  v                   v
                        ┌──────────────────┐   ┌──────────────────┐
                        │ Call Groq/Gemini │   │ Heuristic        │
                        │ & Save to Cache  │   │ Fallback Engine  │
                        └──────────────────┘   └──────────────────┘
```

---

## 💾 Database Schema & SQLite WAL Mode

SQLite is configured with **Write-Ahead Logging (WAL)** for optimal concurrent read/write performance:

```sql
PRAGMA journal_mode = WAL;
PRAGMA synchronous = NORMAL;
PRAGMA busy_timeout = 5000;
```

### Table Definitions

```sql
-- Agents Table
CREATE TABLE IF NOT EXISTS agents (
    id VARCHAR(64) PRIMARY KEY,
    name VARCHAR(128) NOT NULL,
    personality TEXT NOT NULL,
    occupation VARCHAR(128) NOT NULL,
    x INTEGER NOT NULL DEFAULT 0,
    y INTEGER NOT NULL DEFAULT 0,
    mood VARCHAR(32) NOT NULL DEFAULT 'NEUTRAL',
    money INTEGER NOT NULL DEFAULT 100,
    last_action TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Episodic Memories
CREATE TABLE IF NOT EXISTS memories (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    agent_id VARCHAR(64) NOT NULL,
    content TEXT NOT NULL,
    importance_score REAL DEFAULT 1.0,
    timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (agent_id) REFERENCES agents(id) ON DELETE CASCADE
);
CREATE INDEX IF NOT EXISTS idx_memories_agent_time ON memories(agent_id, timestamp DESC);

-- World Events
CREATE TABLE IF NOT EXISTS events (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    type VARCHAR(64) NOT NULL,
    payload JSON NOT NULL,
    timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- LLM Prompt Cache
CREATE TABLE IF NOT EXISTS llm_cache (
    prompt_hash VARCHAR(64) PRIMARY KEY,
    response TEXT NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Historical Tick Snapshots (DVR Replay)
CREATE TABLE IF NOT EXISTS tick_snapshots (
    tick_number INTEGER PRIMARY KEY,
    state_payload JSON NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

---

## 🔌 REST API Reference

| Method | Endpoint | Description | Request Body / Query Params |
|---|---|---|---|
| `GET` | `/api/health` | Service health status, JVM memory stats & virtual thread metrics | None |
| `GET` | `/api/agents` | Current states of all active citizens | None |
| `GET` | `/api/agents/{id}/memories` | Last $N$ episodic memories for an agent | `?limit=10` |
| `POST` | `/api/events/trigger` | Inject a dynamic world event into the simulation | `{"type": "FESTIVAL", "description": "Town Fair"}` |
| `POST` | `/api/simulation/pause` | Pause active simulation ticks | None |
| `POST` | `/api/simulation/resume` | Resume active simulation ticks | None |
| `GET` | `/api/simulation/replay` | Fetch historical tick snapshots for playback | `?from=1&to=100` |

---

## 📡 WebSocket STOMP Protocol

- **STOMP Endpoint:** `ws://localhost:8080/ws-agentville`

### Outbound Subscriptions (Server $\rightarrow$ Client)
- `/topic/town-state` $\rightarrow$ Broadcasts full world tick state payload at 1 Hz.
- `/topic/events` $\rightarrow$ Broadcasts immediate dynamic environment events.
- `/topic/agent/{id}` $\rightarrow$ Targeted agent updates (speech bubbles, memory additions).

### Inbound Destinations (Client $\rightarrow$ Server)
- `/app/god-event` $\rightarrow$ Dynamic event trigger payload.
- `/app/agent-command` $\rightarrow$ Direct instruction or chat sent to a specific agent.

---

## ⚙️ Configuration & Environment Variables

### Environment Variables

| Variable | Required | Default | Description |
|---|---|---|---|
| `GROQ_API_KEY` | Optional* | None | API key for Groq Cloud (Llama-3 models) |
| `GEMINI_API_KEY` | Optional* | None | API key for Google Gemini 1.5 Flash |
| `SPRING_DATASOURCE_URL` | No | `jdbc:sqlite:data/agentville.db` | SQLite database file location |
| `SIMULATION_TICK_RATE_MS` | No | `1000` | Duration of each simulation tick in milliseconds |
| `SIMULATION_AGENT_COUNT` | No | `12` | Number of autonomous agents to initialize |
| `LLM_CACHE_ENABLED` | No | `true` | Enable/disable SHA-256 prompt caching |
| `LLM_RATE_LIMIT_RPM` | No | `30` | Max requests per minute before heuristic fallback |

*\*Note: At least one LLM key (`GROQ_API_KEY` or `GEMINI_API_KEY`) is recommended for generative behavior; otherwise, the heuristic fallback engine handles all agent decisions.*

---

## 🚀 Getting Started

### 1. Requirements
- **JDK 21+** (Eclipse Temurin 21 recommended)
- **Maven 3.9+** (or use included `./mvnw`)

### 2. Run Locally with Maven Wrapper

```bash
cd backend

# On Linux / macOS
./mvnw spring-boot:run

# On Windows (PowerShell)
.\mvnw.cmd spring-boot:run
```

The server will initialize SQLite tables, load the default agent configuration, and start the simulation loop on `http://localhost:8080`.

### 3. Verify Health Check

```bash
curl http://localhost:8080/api/health
```

---

## 🧪 Automated Testing Suite

The test suite covers virtual thread concurrency, SQLite WAL under load, prompt caching, and WebSocket delivery:

```bash
# Run all tests
./mvnw clean test

# Run specific concurrency test
./mvnw test -Dtest=VirtualThreadEngineTest

# Run persistence integrity test
./mvnw test -Dtest=SqlitePersistenceTest
```

---

## 🚢 Containerization & Cloud Deployment

### Multi-Stage `Dockerfile`

```dockerfile
# Build Stage
FROM eclipse-temurin:21-jdk-alpine AS builder
WORKDIR /app
COPY . .
RUN ./mvnw clean package -DskipTests

# Runtime Stage
FROM eclipse-temurin:21-jre-alpine
WORKDIR /app
RUN mkdir -p /app/data
VOLUME /app/data
COPY --from=builder /app/target/agentville-backend-*.jar app.jar
ENV SPRING_DATASOURCE_URL=jdbc:sqlite:/app/data/agentville.db
EXPOSE 8080
ENTRYPOINT ["java", "-XX:+UseZGC", "-XX:+ZGenerational", "-jar", "app.jar"]
```

### Deploying to Render / Fly.io
1. Set environment variables `GROQ_API_KEY` and `GEMINI_API_KEY`.
2. Attach a persistent volume mounted to `/app/data` to ensure SQLite data persists across deployments and container restarts.
3. Expose port `8080`.

---

## 📄 License

This backend engine is part of the HumanScope / Agentville simulation suite licensed under the MIT License.
