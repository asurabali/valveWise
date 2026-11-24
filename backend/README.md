# ValveWise backend

C11 engine and CLI. Parses EPANET `.inp` files, models pipe bursts and stuck-open valves, and optimizes valve placement under a budget.

## Source map

| File | Role |
|---|---|
| `graph.c/h` | Adjacency-list network graph |
| `hashmap.c/h` | Open-addressing hash table (djb2): string IDs to dense indices |
| `queue.c/h` | Circular integer queue for BFS |
| `heap.c/h` | Binary heap for ranking critical links |
| `inp_parser.c/h` | EPANET-style `.inp` parser (+ external `.valves` files) |
| `isolation.c/h` | BFS reachability, isolation boundaries, stuck valves, iterative Tarjan (bridges, articulation points) |
| `optimizer.c/h` | Greedy, random, topology and exhaustive valve placement |
| `hydraulics.c/h` | Bridge to OWA-EPANET pressure-driven states (baseline, after failure, after isolation, after optimization) |
| `json_output.c/h` | JSON serializer |
| `main.c` | CLI |

## Build and run

```bash
make            # Windows MinGW: mingw32-make
./valvewise --input data/sample.inp --output out.json --verbose                       # full run
./valvewise --input data/sample.inp --fail P5 --budget 3 --output out.json            # one failure
./valvewise --input data/sample.inp --fail P3,P7 --stuck V1 --budget 5 --output out.json
./valvewise --input data/net_image3.inp --valves data/net_image3.valves --output out.json
```

Flags: `--input F`, `--output F`, `--fail P1,P2`, `--stuck V1`, `--valves FILE`, `--budget N`, `--seed N`, `--mode full|rank|reach`, `--hydraulics on|off`, `--min-pressure N` (psi, default 20), `--verbose`.

Tests: `bash tests/run_tests.sh`. Benchmarks: `bash bench/run.sh`.

## Optimizers

| Algorithm | Idea | Cost |
|---|---|---|
| Greedy | Each round add the valve with the best marginal disruption reduction. When no single valve helps (isolating a failed segment needs valves on several sides at once) it looks ahead over groups of 2..remaining valves and takes the group with the best gain per valve. Lookahead is capped at 100,000 evaluations per round. | O(B·C·S·(V+E)) plus lookahead |
| Random | 100 random placements, reports mean and best | O(100·S·(V+E)) |
| Topology | Bridges first, then highest endpoint-degree product | O(C log C + S·(V+E)) |
| Exhaustive | All C(C,B) combinations, skipped above 500,000 | O(C(C,B)·S·(V+E)) |

`C` = candidate links, `S` = failure scenarios.

## JSON output

Top-level keys: `graph`, `reachability`, `isolation`, `structural`, `ranked_links`, `optimization` (`greedy`, `random`, `topology`, `exhaustive`), `explanations`, `performance`, `hydraulics`. Existing fields are never renamed or retyped; additions are logged in `CONTRACT_CHANGES.md`. On failure the CLI exits non-zero and writes `{"error": "..."}` to `--output`.

## Portability notes

`main.c` defines `_POSIX_C_SOURCE` for `strdup`, and the Makefile passes `-D_POSIX_C_SOURCE=200809L -D_DEFAULT_SOURCE` to the vendored EPANET sources (`fdopen`, `mkstemp`). Without these, 64-bit Linux/macOS builds truncate pointers and segfault.
