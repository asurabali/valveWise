# ValveWise

Water distribution network failure and resilience optimizer. ValveWise reads EPANET `.inp` networks, simulates pipe bursts and stuck valves, finds structural weak points, and decides where to place a limited budget of isolation valves to reduce the demand that gets cut off.

> **Scope:** isolation and optimization are connectivity models. Optional pressure-driven hydraulic states come from the vendored OWA-EPANET 2.2 engine. Outputs are not engineering or regulatory predictions.

## Repository layout

```
valvewise/
├── backend/    C engine + CLI, test networks, tests, benchmarks, vendored EPANET
└── frontend/   Browser UI (vanilla JS, no build step) + Node server that runs the backend
```

| Folder | What is in it |
|---|---|
| `backend/*.c, *.h` | Graph, hash map, queue, heap, parser, BFS/Tarjan isolation, four optimizers, JSON output, EPANET bridge, CLI (`main.c`) |
| `backend/data/` | Test networks (`.inp`) and companion `.valves` files |
| `backend/tests/` | Unit, property, fuzz, hand-network and hydraulics tests (`bash tests/run_tests.sh`) |
| `backend/bench/` | Benchmark matrix and results |
| `backend/third_party/epanet/` | Vendored OWA-EPANET 2.2 (`src/`, `include/`, MIT license) |
| `frontend/js/`, `index.html`, `*.css` | UI: adapter, SVG graph view, result panels |
| `frontend/server/server.js` | Zero-dependency server: static files, `GET /api/networks`, `POST /api/run` |
| `frontend/fixtures/` | Saved backend outputs used for the offline demo |

## Quick start

Requires `gcc` + `make` (Windows: MSYS2 MinGW, use `mingw32-make`) and Node 18+.

```bash
make -C backend                  # build the engine
node frontend/server/server.js   # start the UI on http://localhost:3000
```

Open `http://localhost:3000` (not `index.html` directly), pick a network and press **Run simulation**.

```bash
cd backend && bash tests/run_tests.sh      # backend tests
node --test frontend/tests/*.test.js       # frontend tests
```

Set `PORT=8080` for another port, or `VALVEWISE_BIN=/path/to/valvewise` to use a custom binary.

## How the pieces connect

`frontend/js/main.js` → `POST /api/run` → `server.js` spawns `backend/valvewise --input ... --output result.json` → JSON → `frontend/js/adapter.js` normalizes it → panels render it. Backend JSON field changes are logged in `backend/CONTRACT_CHANGES.md`.

## Docs

- [`backend/README.md`](backend/README.md): algorithms, CLI flags, JSON output, complexity
- [`frontend/README.md`](frontend/README.md): UI modules and limits
- [`frontend/DEMO.md`](frontend/DEMO.md): five-minute demo script

Vendored OWA-EPANET is MIT licensed (`backend/third_party/epanet/LICENSE`).
