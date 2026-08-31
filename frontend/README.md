# Agentia Frontend 🎨🏙️

[![React 18](https://img.shields.io/badge/React-18.3-61DAFB?style=flat&logo=react&logoColor=black)](https://react.dev/)
[![TypeScript](https://img.shields.io/badge/TypeScript-5.5-3178C6?style=flat&logo=typescript&logoColor=white)](https://www.typescriptlang.org/)
[![Vite](https://img.shields.io/badge/Vite-5.4-646CFF?style=flat&logo=vite&logoColor=white)](https://vitejs.dev/)
[![Tailwind CSS](https://img.shields.io/badge/Tailwind-3.4-38B2AC?style=flat&logo=tailwind-css&logoColor=white)](https://tailwindcss.com/)
[![Zustand](https://img.shields.io/badge/Zustand-State%20Store-443E38?style=flat)](https://github.com/pmndrs/zustand)
[![STOMP.js](https://img.shields.io/badge/STOMP.js-WebSocket%20Client-blue?style=flat)](https://github.com/stomp-js/stompjs)

> **Neo-Brutalist 2.5D Isometric Multi-Agent Town Sandbox**  
> A high-performance, real-time web client rendering autonomous AI citizen simulations with sub-pixel movement interpolation, instant mind-reading inspection, God-Mode dynamic event injection, and historical DVR scrubbing.

---

## 📖 Table of Contents

- [Visual Identity & Art Direction](#-visual-identity--art-direction)
- [Key Features](#-key-features)
- [Project Structure](#-project-structure)
- [Design System & Tokens](#-design-system--tokens)
- [2.5D Isometric Canvas Engine](#-25d-isometric-canvas-engine)
- [State Management & WebSockets](#-state-management--websockets)
- [Environment Configuration](#-environment-configuration)
- [Getting Started](#-getting-started)
- [Available Scripts](#-available-scripts)
- [Component Architecture](#-component-architecture)
- [Production Deployment](#-production-deployment)

---

## 🎨 Visual Identity & Art Direction

The frontend adopts a **Light-Mode Minimalist Neo-Brutalist 2.5D Isometric Diorama** aesthetic:
- **Perspective:** Isometric miniature world viewed from an elevated angle on a warm cream ground (`#FDF8EF`).
- **Buildings:** Color-blocked isometric structures with thick 2–3px black outlines, vibrant flat fills, and architectural props (awnings, chimneys, fountains, benches, market stalls).
- **Citizens:** Expressive stick-figure citizens with line limbs, circle heads, occupation accessories (chef hats, badges, berets), and floating warm gold speech bubbles.
- **UI Chrome:** Floating neo-brutalist cards featuring thick 2px black borders, hard 4px offset box shadows (`shadow-[4px_4px_0px_0px_#000]`), bold uppercase sans-serif typography, and tactile physical button press interactions (`active:translate-x-[3px] active:translate-y-[3px]`).

---

## ✨ Key Features

- 🏙️ **HTML5 2.5D Isometric Canvas:** Custom renderer converting grid coordinates to isometric screen space with automatic depth sorting (painter's algorithm).
- 🏃 **60 FPS Smooth Movement (`lerp`):** Sub-pixel linear interpolation between 1-second server tick coordinates eliminates jerky grid steps.
- 💬 **Warm Gold Speech Bubbles:** Real-time dialogue floating directly above speaking agents with auto-dismiss timers.
- 🔍 **Interactive Agent Inspector Drawer:** Instant slide-over displaying:
  - Stick avatar & occupation badge
  - Personality traits & active objective
  - Monospace wallet balance & live mood chip (Happy, Anxious, Tired, Energetic, Neutral)
  - Thought card showing current internal monologue
  - Chronological episodic memory dots timeline
- ⚡ **God-Mode Event Palette:** 2×2 color-blocked modal with tactile preset cards (Market Crash, Town Festival, Thunderstorm, Bank Heist), custom event prompt input, and instant broadcast trigger.
- ⏪ **Time-Travel Replay DVR Scrubber:** Scrub back and forth across past simulation ticks without triggering new LLM calls.
- ⚡ **Resilient STOMP WebSocket Client:** Automatic reconnect and topic subscriptions for real-time 1 Hz state synchronization.

---

## 📁 Project Structure

```
frontend/
├── index.html                   # HTML5 document root
├── package.json                 # Frontend dependencies and scripts
├── tsconfig.json                # TypeScript strict configuration
├── tsconfig.node.json           # Vite Node configuration
├── vite.config.ts               # Vite configuration and aliases
├── tailwind.config.js           # Neo-brutalist theme tokens and shadows
├── postcss.config.js            # PostCSS configuration
├── Dockerfile                   # Multi-stage production container
└── src/
    ├── assets/                  # Icons, fonts, and static textures
    ├── components/              # UI components
    │   ├── canvas/
    │   │   ├── TownCanvas.tsx   # 2.5D Isometric Canvas renderer
    │   │   ├── IsometricGrid.ts # Isometric coordinate transforms
    │   │   ├── BuildingRender.ts# Isometric building drawings & props
    │   │   └── AgentRender.ts   # Stick figures, accessories & speech bubbles
    │   ├── inspector/
    │   │   ├── AgentDrawer.tsx  # Agent inspection slide-out panel
    │   │   ├── MemoryStream.tsx # Episodic memory timeline dots
    │   │   └── MoodChip.tsx     # Color-coded mood badge
    │   ├── godmode/
    │   │   └── GodModeModal.tsx # 2x2 dynamic event injection modal
    │   ├── controls/
    │   │   ├── EventTicker.tsx  # Live scrolling event & dialogue feed
    │   │   ├── StatusBar.tsx    # Tick counter, agent count & connection pill
    │   │   └── ReplayBar.tsx    # DVR timeline scrubber and playback controls
    │   └── common/
    │       └── Button.tsx       # Neo-brutalist tactile button
    ├── hooks/
    │   ├── useSimulationSocket.ts # STOMP over SockJS connection manager
    │   └── useKeyboardShortcuts.ts# Space (pause), ESC (dismiss), G (god-mode), R (replay)
    ├── store/
    │   └── useAgentStore.ts     # Zustand store for simulation state, agents, and replay
    ├── types/
    │   └── simulation.ts        # TypeScript definitions (Agent, Action, Event, Memory, Tick)
    ├── App.tsx                  # Root layout shell
    ├── App.css                  # Custom neo-brutalist utility classes
    ├── index.css                # Tailwind directives and base styling
    ├── main.tsx                 # React DOM mount point
    └── vite-env.d.ts            # Vite client type definitions
```

---

## 🎨 Design System & Tokens

### Building Color Palette

| Building | Role / Activity | Hex Fill | Props & Features |
|---|---|---|---|
| **Bakery** | Food & Social Hub | `#FFE600` (Canary Yellow) | Awning, bread display, warm signage |
| **Market** | Commerce & Trading | `#FF6B6B` (Vivid Coral) | Striped awning, fruit & vegetable stalls |
| **Park** | Leisure & Relaxation | `#00E599` (Electric Mint) | Fountain, benches, lollipop trees, fences |
| **Café** | Social Gossip & Coffee | `#C4B5FD` (Soft Lavender) | Outdoor tables, chairs, chalk menu board |
| **Town Hall** | Politics & Governance | `#7DD3FC` (Sky Blue) | Grand columns, clock tower, fluttering flag |
| **Workshop** | Crafting & Innovation | `#F472B6` (Hot Pink) | Work easel, tools, craft bench |
| **Residential** | Rest & Sleep | `#FB923C` (Warm Orange) | Row houses, brick chimneys, picket fences |

### Neo-Brutalist CSS Tokens

```css
/* Card Surface */
.neo-card {
  background-color: #FFFFFF;
  border: 2px solid #000000;
  box-shadow: 4px 4px 0px 0px #000000;
}

/* Tactile Button Press */
.neo-button {
  border: 2px solid #000000;
  box-shadow: 3px 3px 0px 0px #000000;
  transition: transform 0.08s ease-in-out, box-shadow 0.08s ease-in-out;
}
.neo-button:hover {
  transform: translateY(-2px);
  box-shadow: 5px 5px 0px 0px #000000;
}
.neo-button:active {
  transform: translate(3px, 3px);
  box-shadow: 0px 0px 0px 0px #000000;
}

/* Gold Speech Bubble */
.neo-speech-bubble {
  background-color: #FEF3C7; /* amber-100 */
  border: 2px solid #000000;
  box-shadow: 3px 3px 0px 0px #000000;
}
```

---

## 📐 2.5D Isometric Canvas Engine

### Coordinate Transformation
Converts discrete simulation grid coordinates $(x, y)$ to continuous isometric screen coordinates $(sx, sy)$:

$$sx = \text{originX} + (x - y) \cdot \frac{\text{tileWidth}}{2}$$

$$sy = \text{originY} + (x + y) \cdot \frac{\text{tileHeight}}{2}$$

### Smooth Linear Interpolation (`lerp`)
During each animation frame, agent positions smoothly transition toward the latest tick position:

$$x_{\text{render}}(t) = x_{\text{prev}} + (x_{\text{target}} - x_{\text{prev}}) \cdot \min\left(1.0, \frac{t - t_{\text{tick}}}{T_{\text{interval}}}\right)$$

### Depth Sorting
All buildings and agent sprites are depth-sorted by their composite grid index $y + x$ before rendering (Painter's Algorithm), ensuring foreground entities naturally occlude background structures.

---

## 🔄 State Management & WebSockets

### Centralized Store (`useAgentStore`)
Manages reactive application state using **Zustand**:
- `agents`: Map of active agents indexed by unique `agent_id`.
- `selectedAgentId`: Currently inspected agent (opens drawer).
- `activeWorldEvent`: Current global event (e.g. Market Crash).
- `isReplayMode`: Flag indicating live simulation vs. historical DVR mode.
- `replayTick`: Current scrubbed tick index.
- `isPaused`: Pause/resume toggle.

### STOMP Connection Lifecycle (`useSimulationSocket`)
- Connects to `ws://localhost:8080/ws-agentville` over SockJS/STOMP.
- Subscribes to:
  - `/topic/town-state` $\rightarrow$ Full 1 Hz tick state updates.
  - `/topic/events` $\rightarrow$ Dynamic environment events.
  - `/topic/agent/{id}` $\rightarrow$ Targeted speech bubbles and interactions.
- Publishes to:
  - `/app/god-event` $\rightarrow$ Sends user-triggered event payload.

---

## ⚙️ Environment Configuration

Create a `.env` or `.env.local` file in `frontend/`:

```env
# Backend REST API Base URL
VITE_BACKEND_URL=http://localhost:8080

# Backend STOMP WebSocket Endpoint
VITE_WS_URL=http://localhost:8080/ws-agentville

# Default Canvas Tile Dimensions
VITE_TILE_WIDTH=64
VITE_TILE_HEIGHT=32
```

---

## 🚀 Getting Started

### 1. Install Dependencies

```bash
cd frontend
npm install
```

### 2. Launch Development Server

```bash
npm run dev
```
Open `http://localhost:5173` in your browser.

### 3. Type-Check and Build

```bash
npm run build
```
Generates production-ready optimized assets in the `dist/` directory.

### 4. Preview Production Build

```bash
npm run preview
```

---

## ⌨️ Keyboard Shortcuts

| Key | Action |
|---|---|
| <kbd>Space</kbd> | Pause / Resume live simulation ticks |
| <kbd>G</kbd> | Open / Close God-Mode Event Injection Modal |
| <kbd>R</kbd> | Toggle Time-Travel DVR Replay Bar |
| <kbd>Esc</kbd> | Dismiss Agent Inspector Drawer or Active Modal |

---

## 🚢 Production Deployment

### Multi-Stage Dockerfile
```dockerfile
# Build Stage
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Serve Stage
FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

### Static Hosting Platforms
Deploy seamlessly to:
- **Vercel:** Root directory `frontend`, build command `npm run build`, output directory `dist`.
- **Netlify:** Base directory `frontend`, publish directory `dist`.
- **Cloudflare Pages:** Framework preset `Vite`.

---

## 📄 License

This frontend is part of the HumanScope / Agentville simulation suite licensed under the MIT License.
