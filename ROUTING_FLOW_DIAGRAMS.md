# Priority-Based Routing System - Visual Flow

## System Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                    CONTAINER (Driver Interface)                  │
│  - Available Sections                                            │
│  - Current Location                                              │
│  - Status: IDLE / ACTIVE                                         │
└────────────────────┬────────────────────────────────────────────┘
                     │
                     ▼
        ┌────────────────────────┐
        │  CREATE ROUTE REQUEST  │
        │  (if sections > 0)     │
        └────────┬───────────────┘
                 │
                 ▼
┌────────────────────────────────────────────────────────────────┐
│              GREEDY PRODUCER SELECTION                          │
│  1. Calculate: efficiency = package_count / distance           │
│  2. Sort by efficiency (descending)                             │
│  3. Select until: ∑ sections ≤ available_sections              │
└────────────────┬───────────────────────────────────────────────┘
                 │
                 ▼
┌────────────────────────────────────────────────────────────────┐
│              ROUTING ALGORITHM                                  │
│                                                                  │
│  ┌──────────────┐              ┌──────────────┐                │
│  │   A* STAR    │              │  DIJKSTRA    │                │
│  │              │              │              │                │
│  │ f(n)=g(n)+h  │              │ Guaranteed   │                │
│  │ Faster       │              │ Shortest     │                │
│  └──────────────┘              └──────────────┘                │
│                                                                  │
│  Output: Ordered list of nodes to visit                         │
└────────────────┬───────────────────────────────────────────────┘
                 │
                 ▼
┌────────────────────────────────────────────────────────────────┐
│                  CREATE ROUTE & TASKS                           │
│                                                                  │
│  routes table:                                                  │
│    - route_id                                                   │
│    - container_id                                               │
│    - algorithm_used                                             │
│    - route_status: PLANNED                                      │
│                                                                  │
│  route_tasks table:                                             │
│    Task 1: PICKUP   | Producer A  | priority_order: 1          │
│    Task 2: PICKUP   | Producer B  | priority_order: 2          │
│    Task 3: DELIVERY | Dest X      | priority_order: 3          │
│    Task 4: DELIVERY | Dest Y      | priority_order: 4          │
│                                                                  │
│  Container status → ACTIVE                                      │
└────────────────┬───────────────────────────────────────────────┘
                 │
                 ▼
┌────────────────────────────────────────────────────────────────┐
│              PRIORITY QUEUE EXECUTION                           │
│                                                                  │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │ Priority | Task     | Location    | Status    | Action │   │
│  ├─────────────────────────────────────────────────────────┤   │
│  │    1     | PICKUP   | Producer A  | PENDING   | [GO]   │ ← UNLOCKED
│  │    2     | PICKUP   | Producer B  | PENDING   | Locked │   │
│  │    3     | DELIVERY | Dest X      | PENDING   | Locked │   │
│  │    4     | DELIVERY | Dest Y      | PENDING   | Locked │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                  │
│  RULE: Only priority 1 task can be started                      │
└────────────────┬───────────────────────────────────────────────┘
                 │
                 ▼ Driver clicks [GO]
┌────────────────────────────────────────────────────────────────┐
│                  TRANSACTION FLOW                               │
│                                                                  │
│  1. Task status → IN_PROGRESS                                   │
│  2. Create transaction (REQUESTED)                              │
│  3. Open transaction form                                       │
│                                                                  │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │  PICKUP Transaction                                       │  │
│  │  - Enter sections_used                                    │  │
│  │  - Submit                                                 │  │
│  │    → Inventory deducted                                   │  │
│  │    → Package status: LOADED                               │  │
│  │    → Container: used_sections += sections_used            │  │
│  │    → Transaction: APPROVED → COMPLETED                    │  │
│  └──────────────────────────────────────────────────────────┘  │
│                                                                  │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │  DELIVERY Transaction                                     │  │
│  │  - Enter sections_used                                    │  │
│  │  - Submit                                                 │  │
│  │    → Package status: DELIVERED                            │  │
│  │    → Container: used_sections -= sections_used            │  │
│  │    → Container: available_sections += sections_used       │  │
│  │    → Transaction: COMPLETED                               │  │
│  └──────────────────────────────────────────────────────────┘  │
└────────────────┬───────────────────────────────────────────────┘
                 │
                 ▼
┌────────────────────────────────────────────────────────────────┐
│          DYNAMIC PRIORITY RECALCULATION                         │
│                                                                  │
│  After transaction completion:                                  │
│    1. Mark task as COMPLETED                                    │
│    2. Get remaining PENDING tasks                               │
│    3. For each task, calculate:                                 │
│                                                                  │
│       priority_score = α*(deadline_urgency)                     │
│                      + β*(distance_to_current_location)         │
│                      + γ*(task_type_weight)                     │
│                                                                  │
│       Where:                                                    │
│         α = 0.5 (deadline weight)                               │
│         β = 0.3 (distance weight)                               │
│         γ = 0.2 (task type: PICKUP=0.4, DELIVERY=0.6)          │
│                                                                  │
│    4. Sort tasks by priority_score (ascending)                  │
│    5. Update priority_order in DB                               │
│                                                                  │
│  New priority queue (example after task 1 completed):           │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │ Priority | Task     | Location    | Status    | Action │   │
│  ├─────────────────────────────────────────────────────────┤   │
│  │    1     | DELIVERY | Dest X      | PENDING   | [GO]   │ ← NOW UNLOCKED
│  │    2     | PICKUP   | Producer B  | PENDING   | Locked │   │
│  │    3     | DELIVERY | Dest Y      | PENDING   | Locked │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                  │
│  (Priority re-ordered based on distance/urgency after pickup)   │
└────────────────┬───────────────────────────────────────────────┘
                 │
                 ▼
         ┌───────────────┐
         │  More tasks?  │
         └───┬───────┬───┘
             │       │
         YES │       │ NO
             │       │
             ▼       ▼
     ┌──────────┐  ┌────────────────────┐
     │ Continue │  │ Complete Route     │
     │ Loop     │  │ - Route: COMPLETED │
     └──────────┘  │ - Container: IDLE  │
                   └────────────────────┘
```

## State Machine Diagrams

### Package Status Flow
```
AVAILABLE
    ↓ (route created)
REQUESTED
    ↓ (transaction approved)
APPROVED
    ↓ (pickup completed)
LOADED
    ↓ (in transit)
IN_TRANSIT
    ↓ (delivery completed)
DELIVERED
```

### Task Status Flow
```
PENDING
    ↓ (driver clicks GO)
IN_PROGRESS
    ↓ (transaction completed)
COMPLETED
```

### Transaction Status Flow
```
REQUESTED
    ↓ (form submitted)
APPROVED
    ↓ (inventory/capacity updated)
COMPLETED
```

### Route Status Flow
```
PLANNED
    ↓ (first task started)
ACTIVE
    ↓ (all tasks completed)
COMPLETED
```

## Data Flow Example

### Scenario: Container with 4 available sections

**Step 1: Route Creation**
- Producers: A (2 packages), B (3 packages), C (1 package)
- Distance from container: A=10km, B=5km, C=20km
- Efficiency: A=0.2, B=0.6, C=0.05
- Selection order: B → A (total 5 packages, 5 sections)
- Algorithm: A* chosen

**Step 2: Tasks Created**
```
1. PICKUP   | Producer B | Package X | priority: 1
2. PICKUP   | Producer B | Package Y | priority: 2
3. PICKUP   | Producer B | Package Z | priority: 3
4. PICKUP   | Producer A | Package M | priority: 4
5. PICKUP   | Producer A | Package N | priority: 5
6. DELIVERY | Dest 1     | Package X | priority: 6
7. DELIVERY | Dest 2     | Package Y | priority: 7
...
```

**Step 3: Execution**
- Driver sees task 1 (PICKUP Producer B, Package X)
- Clicks GO
- Transaction form: enters 1 section
- Submit:
  - Package X: AVAILABLE → LOADED
  - Container: used=1, available=3
  - Task 1: COMPLETED

**Step 4: Priority Recalculation**
- Remaining tasks evaluated
- Dest 1 (delivery for Package X) becomes high priority (package already loaded)
- New priority order updates

**Step 5: Loop**
- Next task unlocked
- Driver clicks GO
- Continue until all tasks completed

---

## Algorithm Complexity

### Dijkstra
- **Time:** O((V + E) log V) where V=nodes, E=edges
- **Space:** O(V)
- **Best for:** Small graphs, guaranteed shortest path

### A*
- **Time:** O(E) average, O(V²) worst
- **Space:** O(V)
- **Best for:** Large graphs, heuristic available

### Greedy Producer Selection
- **Time:** O(n log n) for sorting
- **Space:** O(n)
- **Approximation:** Near-optimal for knapsack variant

### Priority Recalculation
- **Time:** O(n log n) per recalculation
- **Frequency:** After each transaction
- **Tasks:** Typically < 50 per route

---

## Key Design Decisions

1. **Why strict priority queue?**
   - Prevents driver decision paralysis
   - Ensures optimal execution order
   - Maintains data consistency

2. **Why transaction-based approval?**
   - Audit trail for all operations
   - Rollback capability
   - State machine enforcement

3. **Why dynamic re-routing?**
   - Adapts to real-time changes
   - Optimizes remaining route
   - Handles contingencies

4. **Why separate task types?**
   - Different validation logic
   - Clear state transitions
   - Simplified UI flows

---

## Performance Considerations

- **Route creation:** ~100-500ms for 20 nodes
- **Priority recalc:** ~10-50ms for 10 tasks
- **Transaction complete:** ~50-100ms (DB updates + recalc)
- **Dashboard load:** ~200-400ms (tasks + map data)

**Optimization strategies:**
- Index on `route_id, priority_order`
- Cache graph edges in `distance_time_matrix`
- Batch DB updates in transactions
- Debounce priority recalculations
