# cs151-halma

Halma implemented in **Game Description Language (GDL)** with a browser-based standalone player. The game rules are encoded as logic in each HTML file. The UI only renders the board and submits moves to the engine.

Rules are summarized in [HALMA_RULES.md](HALMA_RULES.md), based on the Martin Gardner article in [halma_game.pdf](halma_game.pdf).

## Game modes

| File | Players | Pieces | Start | Goal |
|------|---------|--------|-------|------|
| [Halma.html](Halma.html) | 2 (red, blue) | 19 each | Top-left vs bottom-right | Fill the opposite corner yard |
| [Halma4.html](Halma4.html) | 4 (red, green, blue, yellow) | 13 each | All four corners | Fill the diagonally opposite corner yard |

Turn order:

- **2-player:** red → blue → red …
- **4-player:** red → green → blue → yellow → red …

## Run locally

From this directory, start a simple HTTP server

```bash
cd cs151-halma
python3 -m http.server 8000
```

Then open:

- 2-player: http://localhost:8000/Halma.html  
- 4-player: http://localhost:8000/Halma4.html  

Each page links to the other mode in the header.

## How to play (UI)

- **Drag and drop** a piece belonging to the current player onto an empty legal destination.
- **Click-to-move:** click your piece, then click the destination.
- **Move history:** use the arrow keys to step backward and forward through prior positions.

Invalid moves are rejected; the destination cell flashes briefly.

## Project layout

```
cs151-halma/
├── Halma.html          # 2-player standalone game + embedded GDL
├── Halma4.html         # 4-player standalone game + embedded GDL
├── Halma_files/
│   ├── epilog.js       # Epilog / GDL engine
│   └── standalone.js   # Gamemaster-style standalone player
├── HALMA_RULES.md      # Human-readable rules summary
├── halma_game.pdf      # Source article (Scientific American, 1971)
└── README.md
```

## GDL / game encoding

Each HTML file embeds a full game description in a hidden `<textarea id="library">`. That block defines, in GDL/Epilog:

- `role(...)` — players  
- `init(...)` — starting board and control  
- `legal(...)` — legal moves (steps and hop chains)  
- `move(...) :: ...` — state transitions  
- `goal(...)` / `terminal` — scoring and end conditions  
- Static facts — board coordinates, adjacency, hops, `succ/2` for the move counter  

The standalone runtime (`Halma_files/standalone.js`) loads the library, builds the initial state, and calls `perform(action)` when you move. The page overrides `renderstate` and `renderactions` to draw the 16×16 grid and handle drag-and-drop; move validity is checked with `findlegalp` for a single proposed move, which keeps the UI responsive.

## Winning

A player wins when all of their pieces occupy their **target yard** (the yard diagonally opposite their start). The game also ends after 401 plies via `terminal :- step(401)`. Partial progress is reflected in per-role goal scores (0, 25, 50, 75, 100).

Partnership rules from the article are described in [HALMA_RULES.md](HALMA_RULES.md) but are **not** implemented in `Halma4.html` (individual play only).

## Development notes

- **Branches:** feature work has included drag-and-drop UI (`feature/drag-drop`) and four-player mode (`feature/four-players` on remote). Main playable files are `Halma.html` and `Halma4.html` on `main`.
- **Hop chains:** multi-hop sequences are encoded as bounded chains (up to 6 hops) as a single `move(R1,C1,R2,C2)` action.
- **Performance:** avoid `findlegals` in the UI; use `findlegalp` for the one move being attempted.
