# Fleet Management System (FMS)

> **Production-grade autonomous robot fleet coordination platform**  
> Built at **Bharat Forge (Kalyani Group)** · Robotics Division · Pune, India

> ⚠️ **Note:** This repository contains documentation only. The source code lives on the organization's private GitHub. No proprietary code is included here.

---

## Overview

FMS is a full-stack **Fleet Management System** for coordinating autonomous mobile robots (AMRs) in a live manufacturing environment. It handles real-time task assignment, multi-agent path planning, traffic conflict resolution, mission lifecycle management, and operator visibility — all through a unified backend and dashboard.

The system is built on a **TypeScript/Node.js backend** with an **Express + MQTT + Redis + MongoDB** core, and a **React + Three.js frontend** for real-time 3D warehouse visualization and fleet analytics.

**Version:** Fleet Management System v3.1 — MongoDB Persistence

---

## My Contributions

This was a collaborative project. My specific areas of ownership:

### 1. Traffic Management & Multi-Agent Path Planning
- Designed and implemented the **graph-based traffic management module** — nodes represent physical positions (stations, intersections, charging bays), edges represent traversable paths with weights
- Implemented **conflict detection** to prevent two robots from reserving the same node simultaneously
- Built **dynamic re-routing** logic that triggers when a conflict or blockage is detected mid-execution
- Integrated multiple planning solvers: **Prioritized Planning, CBS (Conflict-Based Search), ECBS/EECBS, ICBS, SIPP, and RHCR (Rolling Horizon Cooperative A\*)**
- Built traffic **reservation and blockage management** — robots reserve path segments ahead of time and release them on completion
- Implemented **replanning saga** — event-driven replan orchestration when execution deviates from plan

### 2. Analytics Dashboards
- Built the **Robot Analytics Dashboard** — per-robot metrics including uptime, mission completion rate, battery trends, and task throughput
- Built the **Dispatch Analytics Dashboard** — order fulfillment rates, dispatch queue depth, avg. dispatch time, and failure categorization
- Both dashboards use **Recharts** for data visualization and **TanStack React Query** for live data fetching with background refresh

### 3. WMS–FMS Integration
- Designed and implemented the integration layer between the **Warehouse Management System (WMS)** and FMS — enabling transport tasks triggered from WMS to be automatically dispatched as robot missions in FMS
- Built the `/api/v1/wms` and `/api/v1/transport` API endpoints that serve as the bridge between the two systems — receiving inbound task requests from WMS, validating them, and translating them into FMS mission commands
- Handled end-to-end task lifecycle synchronization — mission status updates in FMS propagate back to WMS so warehouse operators have consistent visibility across both systems

### 4. Frontend Development
- Built and maintained multiple dashboard pages including the **Warehouse view**, **Order management**, and **Robot chat/control UI**
- Implemented real-time UI updates using **Socket.IO client** and **MQTT client** — robot state changes and mission events reflect in the UI without manual refresh
- Integrated **xterm.js** for the in-dashboard SSH terminal, allowing operators to directly access robot agents from the browser
- Built reusable React components using **shadcn/Radix UI** with Tailwind CSS, and used **Zustand** for shared fleet state across pages

### 5. Backend APIs
- Designed and built REST API endpoints across multiple resource groups including `/robots`, `/missions`, `/dispatch`, `/transport`, and `/orders`
- Implemented request validation using **Joi** and **express-validator**, and structured error handling with consistent response schemas
- Added **Winston-based structured logging** across API layers for request tracing and failure diagnosis

---

## System Architecture

```
┌──────────────────────────────────────────────────────────────────┐
│                        React Frontend (Vite)                      │
│                                                                  │
│  /dashboard/robots    /dashboard/analytics   /dashboard/map      │
│  /dashboard/missions  /dashboard/dispatch-analytics              │
│  /dashboard/warehouse /dashboard/order       /dashboard/robotchat│
│                                                                  │
│  Three.js 3D Map · Recharts · Zustand · TanStack Query · MQTT    │
└───────────────────────────┬──────────────────────────────────────┘
                            │ REST + WebSocket + MQTT (ws)
┌───────────────────────────▼──────────────────────────────────────┐
│                    Node.js / Express Backend                      │
│                                                                  │
│  ┌──────────────┐  ┌──────────────┐  ┌────────────────────────┐ │
│  │Fleet Module  │  │ Mission Saga │  │  Traffic Manager       │ │
│  │(Command Bus  │  │(Replan Saga) │  │  (Graph · CBS · SIPP)  │ │
│  │ Query Bus    │  └──────────────┘  └────────────────────────┘ │
│  │ Event Bus)   │                                               │
│  └──────────────┘  ┌──────────────┐  ┌────────────────────────┐ │
│                    │  Event Store  │  │  Execution Monitor     │ │
│                    │  (MongoDB)    │  │  (Heartbeat · State)   │ │
│                    └──────────────┘  └────────────────────────┘ │
│                                                                  │
│  ┌────────────────────────────────────────────────────────────┐  │
│  │              Persistence & Caching Layer                    │  │
│  │   MongoDB (fleet state · missions · maps · event store)    │  │
│  │   Redis   (live robot state · pub/sub · session · cache)   │  │
│  └────────────────────────────────────────────────────────────┘  │
└───────────────────────────┬──────────────────────────────────────┘
                            │ MQTT (tcp)
         ┌──────────────────┼──────────────────┐
         ▼                  ▼                  ▼
     Robot Agent A      Robot Agent B      Robot Agent C
      (ROS2 / AMR)      (ROS2 / AMR)      (ROS2 / AMR)
```

---

## Key Features

| Feature | Description |
|---|---|
| **Multi-Agent Path Planning** | Graph-based routing with CBS, ECBS, ICBS, SIPP, RHCR solvers |
| **Traffic Conflict Detection** | Node reservation system prevents simultaneous path conflicts |
| **Dynamic Replanning** | Saga-driven replan on deviation, blockage, or robot failure |
| **Mission Lifecycle** | Create → Execute → Pause → Resume → Cancel → Abort with full timeline |
| **Real-time Fleet Monitoring** | Live position, battery, heartbeat, and status per robot |
| **3D Warehouse Visualization** | Three.js + React Three Fiber map rendering with point-cloud support |
| **Navigation Graph Editor** | Interactive editor for nodes, paths, zones, and traffic configs |
| **Robot Analytics Dashboard** | Per-robot KPIs: uptime, completion rate, battery trends |
| **Dispatch Analytics Dashboard** | Order fulfillment rate, queue depth, avg dispatch time |
| **MQTT Communication** | Bidirectional low-latency robot ↔ backend messaging |
| **SSH Terminal Service** | In-dashboard terminal access to robot agents |
| **WMS / MES Integration** | API hooks for warehouse and manufacturing execution systems |

---

## Tech Stack

### Backend
| Category | Technology |
|---|---|
| Runtime | Node.js >= 18, TypeScript |
| Framework | Express |
| Database | MongoDB + Mongoose, GridFS (maps/point-clouds) |
| Cache & Pub/Sub | Redis via ioredis |
| Messaging | MQTT |
| Real-time | Socket.IO |
| Architecture | Command Bus · Query Bus · Event Bus · Sagas · Event Store |
| Auth & Security | JWT, Helmet, CORS, Rate Limiting |
| Validation | Joi, express-validator |
| Logging | Winston |
| Testing | Jest + Supertest |
| Terminal | ssh2 |

### Frontend
| Category | Technology |
|---|---|
| Framework | React 18 + TypeScript + Vite |
| Styling | Tailwind CSS + shadcn/Radix UI |
| 3D Rendering | Three.js + React Three Fiber + Drei |
| State | Zustand |
| Data Fetching | TanStack React Query |
| Charts | Recharts |
| Animations | Framer Motion |
| MQTT Client | mqtt.js |
| Terminal UI | xterm.js |
| Export | XLSX |

---

## Fleet Planning Module

The planning engine supports multiple multi-agent path planning algorithms selectable at runtime:

| Solver | Description |
|---|---|
| **Prioritized Planning** | Fast, simple — plans robots in priority order |
| **CBS** | Conflict-Based Search — optimal, finds inter-agent conflicts and resolves |
| **ECBS / EECBS** | Enhanced CBS with suboptimality bound for speed-quality tradeoff |
| **ICBS** | Improved CBS with cardinal conflict detection |
| **SIPP** | Safe Interval Path Planning — handles dynamic obstacles |
| **RHCR** | Rolling Horizon Cooperative A\* — scales to large fleets with a receding horizon |

Replanning is handled by a **Saga** — an event-driven orchestrator that listens for execution deviations (robot stuck, path blocked, mission failed) and triggers a replan with updated world state.

---

## API Overview

All endpoints are mounted under `/api/v1`.

### Fleet Module (`/api/v1/fleet`)

| Method | Endpoint | Description |
|---|---|---|
| `POST` | `/fleet/missions` | Create a new mission |
| `GET` | `/fleet/missions` | List all missions |
| `GET` | `/fleet/missions/:id` | Get mission detail |
| `GET` | `/fleet/missions/:id/timeline` | Full mission event timeline |
| `POST` | `/fleet/missions/:id/execute` | Execute a mission |
| `POST` | `/fleet/missions/:id/pause` | Pause execution |
| `POST` | `/fleet/missions/:id/resume` | Resume execution |
| `POST` | `/fleet/missions/:id/abort` | Abort mission |
| `POST` | `/fleet/missions/:id/cancel` | Cancel mission |
| `POST` | `/fleet/robots` | Register a robot |
| `GET` | `/fleet/robots` | List all robots |
| `GET` | `/fleet/robots/:id` | Get robot detail |
| `PATCH` | `/fleet/robots/:id` | Update robot |
| `POST` | `/fleet/robots/assign` | Assign robot to mission |
| `GET` | `/fleet/overview` | Fleet-wide overview |
| `GET` | `/fleet/statistics` | Fleet statistics |
| `POST` | `/fleet/planning/replan` | Trigger replan |
| `PUT` | `/fleet/planning/solver` | Switch planning solver |
| `GET` | `/fleet/traffic/zones` | Get traffic zones |
| `GET` | `/fleet/traffic/conflicts` | Get active conflicts |

### Other API Groups
`/robots` · `/missions` · `/maps` · `/graphs` · `/navigation` · `/orders` · `/dispatch` · `/transport` · `/stations` · `/zones` · `/pointclouds` · `/wms` · `/costmaps`

---

## Frontend Pages

| Route | Description |
|---|---|
| `/dashboard` | Main fleet overview |
| `/dashboard/robots` | Robot list and live status |
| `/dashboard/map` | Interactive 3D map + graph editor |
| `/dashboard/missions` | Mission management |
| `/dashboard/analytics` | Robot analytics dashboard |
| `/dashboard/dispatch-analytics` | Dispatch analytics dashboard |
| `/dashboard/warehouse` | Warehouse view |
| `/dashboard/order` | Order management |
| `/dashboard/order/mission` | Mission orders |
| `/dashboard/order/orders` | Order queue |
| `/dashboard/robotchat` | Robot control / chat UI |
| `/dashboard/settings` | System settings |
| `/dashboard/documentation` | In-app docs |

---

## Running Locally

### Prerequisites
- Node.js >= 18
- MongoDB running locally or via Atlas
- Redis running locally
- MQTT broker (e.g. Mosquitto)

### Backend
```bash
git clone <repo>
cd fleet-management-system
cp .env.example .env   # fill in your values
npm install
npm run dev            # starts on :3001
```

### Frontend
```bash
cd client
cp .env.example .env   # fill in VITE_ vars
npm install
npm run start          # starts on :8080
```

### Key Environment Variables
```env
# Backend
PORT=3001
DB_URL=mongodb://localhost:27017/fleet_management
MQTT_BROKER_URL=mqtt://localhost
REDIS_HOST=localhost
REDIS_PORT=6379
JWT_SECRET=your_secret

# Frontend
VITE_API_IP_ADDRESS=127.0.0.1
VITE_API_PORT=3001
VITE_MQTT_BROKER_IP=127.0.0.1
VITE_MQTT_PORT=9001
```

---

## Design Decisions

**Why CBS-family solvers over simple prioritized planning?**
Prioritized planning is fast but suboptimal — a lower-priority robot can get permanently blocked by a higher-priority one. CBS finds and resolves conflicts at a constraint level, producing collision-free paths without arbitrary priority hierarchies. RHCR extends this to large fleets by only planning a rolling time window ahead.

**Why Redis for live robot state?**
Robot state (position, battery, status) is written at high frequency by multiple services. Redis atomic operations make conflict-free concurrent writes straightforward, with configurable TTLs per state type — 5 min for robot state, 2 hours for map cache.

**Why MQTT over REST for robot communication?**
REST polling adds latency and overhead. MQTT's pub/sub model lets robots push state changes instantly and receive commands with minimal overhead — critical for real-time path coordination.

**Why an Event Store?**
Every mission and fleet event is persisted to MongoDB as an immutable event log. This gives full audit trail, enables timeline reconstruction for any mission, and makes replanning easier — the planner replays events to reconstruct current world state.

---

## Related Projects

- [Warehouse Management System](https://github.com/avnisingh07/wms) — Sister project handling QR-triggered inbound transport workflows in the same manufacturing environment

---

## Contributors

| Name | Role | Contribution |
|---|---|---|
| **Avni Singh** | AI/ML Intern | Traffic management, multi-agent path planning, analytics dashboards, WMS–FMS integration, frontend pages, backend APIs |
| **Lakshay Sharma** | AI/ML Intern | Frontend, map visualization |
| **Sumit Singh** | Robotics Software Engineer | Core backend architecture, robot integration |

---

## Author

**Avni Singh** — Software Engineering Intern, Bharat Forge (Kalyani Group)
[linkedin.com/in/avnisingh07](https://linkedin.com/in/avnisingh07) · [avnisinghrana1@gmail.com](mailto:avnisinghrana1@gmail.com) · [github.com/avnisingh07](https://github.com/avnisingh07)
