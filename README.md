# Two Lips — Maximizing Divergence of Two Induced Paths
 
**Author:** Ahmet Berke Kandemir  
**Student ID:** 162235
 
---
 
## Problem Statement
 
Given an undirected graph with two special nodes **A** and **B**, find two **induced paths** from A to B such that the absolute difference of their lengths is **maximized**:
 
> **Score = |len(P1) − len(P2)|**
 
Constraints:
- Both paths must be **induced** (no chord edges — if two nodes are both on the path, the only edge allowed between them is the one actually traversed).
- The two paths must be **node-disjoint** except at A and B.
This is an NP-hard problem, so the solution uses heuristics within a **30-second time limit**.
 
---
 
## Approach Overview
 
The algorithm combines four techniques:
 
| Technique | Role |
|---|---|
| BFS | Shortest path finding & distance precomputation |
| DFS with backtracking | Main path search with pruning |
| Greedy iterative builder | Fast diverse path sampling |
| Segment surgery (`perturbPath`) | Local improvement on existing paths |
 
### Key Data Structure: `GrowState`
 
Efficient O(1) chord-check per node during path construction:
- `inPath[]` — boolean membership array
- `adjCount[v]` — number of current path nodes adjacent to `v`
A node `X` can be legally appended **only if** `adjCount[X] == 1`.  
This avoids scanning the entire path on every extension step.
 
---
 
## Three-Phase Execution
 
| Phase | Time Window | Strategy |
|---|---|---|
| **Phase 1** | 0 – 5s | Enumerate short A→B paths (direct, 2-hop, 3-hop up to 40), block their internals, search for the longest complement |
| **Phase 2** | 5 – 12s | Build random long paths first, then BFS the shortest complement — maximizes diversity |
| **Phase 3** | 12 – 29s | Fix the short path; repeatedly restart and apply surgery on the best long path found so far |
 
---
 
## 9 Sorting Strategies
 
During candidate selection, the algorithm rotates between strategies to suit different graph types:
 
1. Prefer nodes **far from B**
2. Prefer **low-degree** nodes
3. Prefer nodes **far from both** A and B
4. Random order
5–9. Combined / weighted variants
---
 
## Graph Density Adaptation
 
| Density | Avg Degree | DFS Branch Factor | Notes |
|---|---|---|---|
| Sparse | < 8 | 4 | Long paths achievable; best performance |
| Medium | 8 – 15 | 3 | `GrowState` pays off here |
| Dense | > 15 | 2 | Paths stay short; Phase 2 random sampling dominates |
 
---
 
## Files
 
| File | Description |
|---|---|
| `two_lips14.cpp` | Full C++ solution |
| `TwoLips_Kandemir_162235.docx` | Project report with detailed algorithm explanation |
 
---
 
## Building & Running
 
```bash
g++ -O2 -std=c++17 -o two_lips two_lips14.cpp
 
# Input format: N M A B, then M edges
echo "5 5 1 5
1 2
2 3
3 4
4 5
1 3" | ./two_lips
```
 
**Input:** `N M A B` on the first line, then `M` lines of edges `u v`.  
**Output:** Two induced paths from A to B.
 
---
 
## Algorithm Design Notes
 
- **Branching limit on dense graphs = 2:** Higher branching explodes the search space within milliseconds on dense instances.
- **Depth cutoff at N × 2/3:** Once two-thirds of nodes are covered, candidates become very constrained — reducing branching here saves time with minimal quality loss.
- **40 short candidates in Phase 1:** ~0.1s per candidate; more candidates reduced per-candidate time budget too aggressively.
- **Restart every 1.8s in Phase 3:** ~9 restarts over 17 seconds balances exploration vs. exploitation.
- **Surgery every 2.5s:** Each surgery call tries up to 100 segment replacements with mini-DFS; needs ~1.5s budget to be effective.
- **Timeout check every 400 DFS calls:** Amortizes the `clock()` overhead across millions of recursive calls.
