---
name: profile
description: Find and fix performance bottlenecks using profiling tools and systematic analysis
license: MIT
compatibility: opencode
metadata:
  tags: performance, optimization
---

## What I do

Identify where time or memory is actually being spent using profiling tools, then fix the real bottlenecks — not assumed ones.

## Steps

1. **Define the target and get a baseline**:
   - What is slow? Be specific: "the `/api/orders` endpoint takes 2.3s", "the build takes 4 minutes", "this function allocates 800MB"
   - Record the baseline metric before touching anything

2. **Profile, don't guess** — use the right tool:

   | Platform | Time profiler | Memory profiler |
   |---|---|---|
   | Node.js | `node --prof` + `node --prof-process`, or `clinic flame` | `node --inspect` heap snapshot in Chrome DevTools |
   | Browser | Chrome DevTools → Performance tab → record | Chrome DevTools → Memory → heap snapshot |
   | Python | `python -m cProfile -o out.prof script.py` + `snakeviz out.prof` | `memory-profiler`: `@profile` decorator |
   | Go | `go test -cpuprofile cpu.prof ./...` + `go tool pprof cpu.prof` | `go test -memprofile mem.prof` |
   | Rust | `cargo flamegraph` | `heaptrack` |
   | Database | `EXPLAIN ANALYZE <query>` | — |

3. **Read the profiling output** — find the top 3 hotspots:
   - CPU profile: look for functions with highest **self time** (not total time)
   - Memory profile: look for allocation sites with highest **retained size**
   - DB explain plan: look for `Seq Scan` on large tables and high `actual rows` vs `estimated rows`

4. **Investigate each hotspot**:
   - N+1 query: a DB call inside a loop — replace with a batched query
   - Unnecessary recomputation: same calculation done on every request — cache or move outside the loop
   - Blocking I/O: synchronous call that could be async or parallelized
   - Missing index: `Seq Scan` on a filtered column — add an index
   - Large allocation: creating a big object that could be reused or streamed

5. **Fix the highest-impact issue first**, then measure again before fixing the next

6. **Confirm improvement** — compare against the baseline from step 1

## Reading a flame graph

```
A flame graph shows call stacks, width = time spent

Wide bars at the top of a stack = hot functions (where time is actually spent)
Wide bars at the bottom = called often, but time is spent in their children

Example reading:
  handleRequest (90%)
    └─ queryOrders (70%)          ← wide bar → bottleneck is here
        └─ db.query (68%)
    └─ formatResponse (15%)
    └─ auth.verify (5%)

Action: investigate why queryOrders/db.query takes 68% of request time.
        Run EXPLAIN ANALYZE on the query. Result: Seq Scan on orders(200k rows).
        Fix: add index on orders(user_id).
```

## Example: EXPLAIN ANALYZE output

```sql
EXPLAIN ANALYZE SELECT * FROM orders WHERE user_id = '123';

-- Before index:
Seq Scan on orders  (cost=0.00..4200.00 rows=1 width=120)
                    (actual time=0.042..89.3 ms rows=47 loops=1)
-- 89ms, full table scan

-- After: CREATE INDEX idx_orders_user_id ON orders(user_id);
Index Scan using idx_orders_user_id on orders  (cost=0.43..12.5 rows=1 width=120)
                                               (actual time=0.021..0.18 ms rows=47 loops=1)
-- 0.18ms — 500x faster
```

## Rules

- Measure before and after every change — "it feels faster" is not a result
- Do not optimize code that is not in the measured hot path
- Prefer algorithmic improvements (O(n²) → O(n log n)) over micro-optimizations (shaving 2μs from a function called once)
- Document in the commit: what was profiled, what the hotspot was, and the measured before/after improvement
- If the profiler shows a hotspot in a third-party library, investigate whether there's a configuration option or alternative before rewriting
