# Priority-Based Container Routing System

## Overview

A complete **container-driven, priority-based routing and transaction system** for delivery management. The system uses graph algorithms (Dijkstra, A*), greedy capacity selection, and dynamic priority recalculation to optimize delivery routes.

---

## Key Principles

1. **Container drives the system** - Routes are planned first based on container capacity
2. **Priority queue execution** - Tasks are executed in strict priority order
3. **Transaction-based approval** - Every pickup/delivery requires request-approval flow
4. **Dynamic re-routing** - Priorities recalculated after each transaction
5. **Driver only clicks GO** - All logic handled by backend

---

## Database Schema

### Core Tables

#### `location_nodes`
Graph nodes representing producers, destinations, and depots.
```sql
- node_id (PK)
- node_type: ENUM('PRODUCER', 'DESTINATION', 'DEPOT')
- reference_id: ID of producer/destination/depot
- latitude, longitude
```

#### `distance_time_matrix`
Graph edges with travel distances and times.
```sql
- from_node_id, to_node_id (composite PK)
- distance_km
- travel_time_minutes
```

#### `routes`
Route plans for containers.
```sql
- route_id (PK)
- container_id (FK)
- algorithm_used: ENUM('A_STAR', 'DIJKSTRA')
- route_status: ENUM('PLANNED', 'ACTIVE', 'COMPLETED')
```

#### `route_tasks` ⭐ CORE TABLE
Priority queue of pickup and delivery tasks.
```sql
- task_id (PK)
- route_id (FK)
- task_type: ENUM('PICKUP', 'DELIVERY')
- node_id (FK)
- package_id (FK)
- priority_order: INT (lower = higher priority)
- task_status: ENUM('PENDING', 'IN_PROGRESS', 'COMPLETED')
```

#### `transactions`
Request-approval flow for load/deliver operations.
```sql
- transaction_id (PK)
- task_id (FK)
- container_id (FK)
- package_id (FK)
- transaction_type: ENUM('LOAD', 'DELIVER')
- transaction_status: ENUM('REQUESTED', 'APPROVED', 'COMPLETED')
- sections_used
```

#### Updated: `packages`
```sql
- status: ENUM('AVAILABLE', 'REQUESTED', 'APPROVED', 'LOADED', 'IN_TRANSIT', 'DELIVERED')
- destination_id (NEW)
- required_sections (NEW)
```

#### Updated: `containers`
```sql
- status: ENUM('IDLE', 'ACTIVE')
- current_node_id (NEW)
- used_sections (NEW)
```

---

## Routing Algorithms

### 1. Dijkstra's Algorithm
**Location:** `server.js/utils/dijkstra.js`

**Use case:** Guaranteed shortest travel time (SLA-critical routes)

**Cost function:**
```
min ∑ travel_time
```

**Implementation:**
- Priority queue-based
- Returns: `{distance, path: [nodeIds]}`

---

### 2. A* Algorithm
**Location:** `server.js/utils/astar.js`

**Use case:** Faster route planning with heuristic

**Cost function:**
```
f(n) = g(n) + h(n)
  g(n) = actual travel_time so far
  h(n) = haversine_distance_to_goal (heuristic)
```

**Implementation:**
- Uses haversine distance as heuristic
- Returns: `{distance, path: [nodeIds]}`

---

### 3. Greedy Capacity-Aware Producer Selection
**Location:** `server.js/utils/greedyProducerSelection.js`

**Use case:** Knapsack-style producer selection before route creation

**Algorithm:**
1. Calculate efficiency for each producer: `package_count / distance`
2. Sort producers by efficiency (descending)
3. Greedily select until `used_sections ≤ available_sections`

**Constraint:**
```
∑ required_sections ≤ available_sections
```

---

### 4. Dynamic Priority Recalculation
**Location:** `server.js/utils/priorityCalculation.js`

**Use case:** Re-order pending tasks after each transaction

**Priority score (lower = higher priority):**
```
priority = α*(deadline_urgency) + β*(distance) + γ*(task_type)

  α = 0.5 (deadline weight)
  β = 0.3 (distance weight)
  γ = 0.2 (task type weight)
```

**Factors:**
- **Deadline urgency:** 0-1 (overdue = 1, >24h = 0.3)
- **Distance:** Normalized haversine distance
- **Task type:** PICKUP = 0.4, DELIVERY = 0.6

---

## API Routes

### Route Management

#### `GET /container/:container_id/routing`
**Container Routing Dashboard**
- Shows container capacity (total/used/available)
- Displays priority task queue
- Map visualization of route nodes
- GO button for top priority task only

#### `GET /container/:container_id/plan-route`
**Route Planning Page**
- Select algorithm (A* or Dijkstra)
- Shows available packages count
- Creates route on submit

#### `POST /container/:container_id/create-route`
**Create Route**
- Runs greedy producer selection
- Applies selected algorithm (A*/Dijkstra)
- Creates route + task queue
- Updates package status to REQUESTED

---

### Task Execution

#### `POST /task/:task_id/start`
**Start Task (GO button)**
- Verifies task is top priority
- Updates task status to IN_PROGRESS
- Creates transaction (REQUESTED)
- Redirects to transaction form

#### `GET /task/:task_id/transaction`
**Transaction Form**
- Shows task details
- Input: sections_used
- LOAD: pickup at producer
- DELIVER: delivery at destination

#### `POST /transaction/:transaction_id/complete`
**Complete Transaction**
- LOAD: Deduct inventory, update container capacity, set package LOADED
- DELIVER: Free container capacity, set package DELIVERED
- Mark task COMPLETED
- Recalculate priorities for remaining tasks
- If no tasks remain: complete route, set container IDLE

---

## Complete Process Flow

### Step 1: Container Dashboard
Navigate to: `/allContainers` → click **Route** button

Shows:
- Available sections
- Current route (if any)
- Priority task queue

### Step 2: Create Route
Click **Create New Route**

Backend:
1. Run greedy producer selection
2. Apply A* or Dijkstra
3. Create `routes` record
4. Populate `route_tasks` with PICKUP + DELIVERY tasks
5. Assign priority_order
6. Update container status to ACTIVE

### Step 3: Priority Queue Display
Dashboard shows:

| Priority | Task     | Location    | Status     | Action |
|----------|----------|-------------|------------|--------|
| 1        | PICKUP   | Producer A  | PENDING    | **GO** |
| 2        | PICKUP   | Producer B  | PENDING    | Locked |
| 3        | DELIVERY | Destination | PENDING    | Locked |

**Only priority 1 has GO button.**

### Step 4: Pickup Transaction
Driver clicks **GO**:
1. Task status → IN_PROGRESS
2. Transaction REQUESTED created
3. Form opens: enter sections_used
4. Submit:
   - Inventory deducted
   - Package status → LOADED
   - Transaction APPROVED → COMPLETED
   - Container capacity updated

### Step 5: Dynamic Re-Routing
After pickup:
1. Task marked COMPLETED
2. `recalculatePriorities()` runs on remaining tasks
3. Next task unlocked (becomes priority 1)

### Step 6: Delivery Transaction
At destination:
1. Click **GO** on delivery task
2. Form opens
3. Submit:
   - Package status → DELIVERED
   - Container capacity freed
   - Transaction COMPLETED

### Step 7: Route Completion
When task queue empty:
- Route status → COMPLETED
- Container status → IDLE
- New routing allowed

---

## UI Pages

### 1. Container Routing Dashboard
**Path:** `/container/:container_id/routing`

**Features:**
- Container capacity stats
- Priority task queue table
- Leaflet map with route nodes
- GSAP entrance animations

### 2. Plan Route
**Path:** `/container/:container_id/plan-route`

**Features:**
- Algorithm selection (A* / Dijkstra)
- Shows available packages
- Modern card layout with GSAP

### 3. Transaction Form
**Path:** `/task/:task_id/transaction`

**Features:**
- Task details
- Sections input
- Auto-updates inventory/capacity
- GSAP interactions

---

## Navigation Links

**Added to existing pages:**
- `/allContainers` → Each row has **Route** button
- `/container/:id` → **Open Routing Dashboard** button

---

## Setup & Installation

### 1. Run Schema Script
```bash
node server.js/scripts/create_routing_schema.js
```

Creates:
- `location_nodes`
- `distance_time_matrix`
- `routes`
- `route_tasks`
- `transactions`
- Updates `packages` and `containers` tables

### 2. Populate Test Data
```bash
node server.js/scripts/populate_test_routing_data.js
```

Creates:
- 5 destinations
- 10 sample packages

### 3. Start Server
```bash
npm run start
```

Server runs on: `http://localhost:5001`

---

## Testing Flow

1. **Navigate to Containers:**
   `http://localhost:5001/allContainers`

2. **Open Routing Dashboard:**
   Click **Route** button for any container

3. **Create Route:**
   - Click **Create New Route**
   - Select A* or Dijkstra
   - Click **Create Route**

4. **Execute Tasks:**
   - Priority queue appears
   - Click **GO** on task #1
   - Fill transaction form
   - Submit
   - Repeat for next tasks

5. **Observe Re-Routing:**
   - After each transaction, priorities update
   - Map markers show nodes
   - Tasks complete in order

---

## File Structure

```
server.js/
├── utils/
│   ├── dijkstra.js          # Dijkstra algorithm
│   ├── astar.js             # A* algorithm
│   ├── haversine.js         # Distance calculation
│   ├── greedyProducerSelection.js
│   └── priorityCalculation.js
├── scripts/
│   ├── create_routing_schema.js
│   └── populate_test_routing_data.js
├── views/
│   └── routing/
│       ├── containerRoutingDashboard.ejs
│       ├── planRoute.ejs
│       └── transactionForm.ejs
└── server.js (updated with API routes)
```

---

## Technical Summary

**For AI Agent / Viva:**

> "The system is container-driven and priority-based. Routes are created first using **Dijkstra** or **A*** on a graph model, selecting producers only if container capacity allows maximum package collection via **greedy knapsack algorithm**. Route execution is handled through a **strict priority task queue**, where each pickup or delivery generates a **request–approval transaction**. After every transaction, priorities are **recalculated dynamically** using weighted scoring (deadline urgency, distance, task type), ensuring correct ordering and feasibility. Drivers interact only through a dashboard that exposes the **next priority task** (GO button), while all logic, validation, and routing decisions are handled by the **backend**."

---

## Notes

- ✅ Existing routes and APIs remain **unchanged**
- ✅ New system runs **independently** alongside current pages
- ✅ Database structure is **backward compatible**
- ✅ All algorithms are **modular** and **testable**
- ✅ Transaction state machine enforces **strict sequencing**
- ✅ Priority queue ensures **only top task is actionable**

---

## Support

Server running at: `http://localhost:5001`

Key routes:
- `/allContainers` - Container list with Route buttons
- `/container/:id/routing` - Priority queue dashboard
- `/container/:id/plan-route` - Create new route
