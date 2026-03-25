# Strategic C Chess

> A fully-featured chess engine built in C for UCI EECS 22L, featuring an AI opponent, local multiplayer, and a custom GUI.

**Team 7 — PretendGINEERS:**  
Mharlo Borromeo · Tuaha Khan · Jack Lu · Calvin Nguyen · Mervin Nguyen · Peter Nguyen

**Version:** v2024.04.29 · **Course:** UC Irvine EECS 22L S24

---

## Table of Contents

- [Overview](#overview)
- [System Requirements](#system-requirements)
- [Installation](#installation)
- [Building from Source](#building-from-source)
- [Running the Game](#running-the-game)
- [Gameplay](#gameplay)
- [Project Structure](#project-structure)
- [API Reference](#api-reference)
- [Error Reference](#error-reference)
- [Packaging a Release](#packaging-a-release)
- [License](#license)

---

## Overview

Chess-pionage is a chess game written in C, offering both single-player (vs. AI) and local two-player modes. The engine handles the full chess ruleset — including castling, en passant, and pawn promotion — and ships with a text-based UI with bitmap-rendered pieces in the GUI build.

**Features:**

- AI opponent (`aiTurn`) driven by legal move generation
- Local two-player mode
- Full rules: castling, en passant, pawn promotion
- Coordinate-based move input (`A2 A4` notation)
- Check and checkmate detection
- Custom GUI with bitmap piece rendering

---

## System Requirements

| Component | Minimum |
|-----------|---------|
| CPU | x86_64, ≥ 1 GHz |
| RAM | 1 GB |
| Disk | 500 MB |
| OS | Linux (RHEL 7 / RHEL 8) |

---

## Installation

### Pre-built binary (no compilation required)

1. Download `Chess.tar.gz`.
2. Extract the archive:
   ```bash
   gtar xvzf Chess.tar.gz
   ```
3. Run the executable:
   ```bash
   cd chess/bin
   ./chesspionage
   ```

### Uninstalling

```bash
cd Downloads/chess
rm -f *
```

> ⚠️ Confirm there are no other files in the directory before running `rm -f *`.

---

## Building from Source

```bash
cd src
make
```

The compiled binary is placed in `bin/`.

---

## Running the Game

```bash
cd bin
./chess
```

On launch, the game prompts you to select a mode and, if playing vs. computer, a color:

```
Welcome to Chesspionage!
Please select from the following options:
1. Player vs Player
2. Player vs Computer

Select Your Team Color:
1. White
2. Black
```

---

## Gameplay

**Objective:** Force the opponent's king into checkmate — a position where every possible move leaves the king in check.

**Making a move:** Enter the start and destination coordinates when prompted:

```
Enter the coordinates of the piece to move (x0 y0): B2 B4
```

**Special moves supported:**
- **Castling** — King moves two squares toward a rook; the rook jumps to the other side. Neither piece may have moved previously, no pieces may be between them, and the king cannot pass through or land on an attacked square.
- **En Passant** — A pawn that advances two squares past an opposing pawn may be captured diagonally on the immediately following turn only.
- **Pawn Promotion** — When a pawn reaches the opposite back rank, the player selects a replacement piece (Queen, Rook, Bishop, or Knight).

**Piece reference:**

| Piece | Count | Movement |
|-------|-------|----------|
| Pawn | 8 | Forward 1–2 squares; captures diagonally |
| Rook | 2 | Any number of squares horizontally or vertically |
| Bishop | 2 | Any number of squares diagonally |
| Knight | 2 | L-shape: 2 squares + 1 perpendicular |
| Queen | 1 | Any direction, any number of squares |
| King | 1 | One square in any direction |

---

## Project Structure

```
Chess-pionage/
├── bin/              # Compiled executable and bitmap assets (*.bmp)
├── doc/
│   ├── README
│   ├── INSTALL
│   ├── COPYRIGHT
│   └── Chess_UserManual.pdf
├── src/              # All C source and header files
└── Makefile
```

---

## API Reference

All core functions operate on a `struct Board *board` representing the 8×8 game state.

### `initializeBoard(struct Board *board)`
Allocates and populates the board with pieces in their starting positions. Called once at game start. Performs a memory allocation check on completion.

### `printBoard(struct Board *board)`
Renders the current board state to the terminal using algebraic notation (`K`, `Q`, `R`, `B`, `N`, `P` for white; lowercase for black). May optionally highlight valid moves for a selected piece.

### `setPiece(struct Board *board, int file, int rank, pieceType piece, pieceColor color)`
Places a single piece of a given type and color at the specified file/rank. Used internally by `initializeBoard`.

### `isValid(struct Board *board, Move *move, pieceColor color)` → `int`
Returns `1` if the proposed move conforms to the moving piece's legal moveset, the destination is in-bounds, and the destination is not occupied by a friendly piece. Returns `0` otherwise. Called in a loop inside `movePiece` until a valid move is entered.

### `movePiece(struct Move *move, struct Board *board)`
Moves the piece to its destination and sets the vacated square to `NULL`. Relies on `isValid` for legality checking.

### `pieceMove(struct Board *board, Move *move, pieceColor color)`
Updates the game state for a given piece movement. Each piece type has a dedicated variant (`pawnMove`, `bishopMove`, `queenMove`, etc.).

### `capture(struct Board *board, struct Move *move)`
Removes the captured piece and updates the board. Legality is pre-validated by `isValid`.

### `isCheck(struct Board *board, pieceColor color)` → `int`
Returns `1` if the king of the given color is currently under attack. Used as a precondition by `isCheckmate`.

### `isCheckmate(struct Board *board)` → `int`
Returns `1` if the current player's king is in check with no legal escapes, ending the game. Returns `0` if play continues. Called after every move.

### `isCastle(Move *move, struct Board *board, pieceColor color)` → `int`
Returns `1` if the castling maneuver is legal (neither piece has moved, no pieces between them, king not passing through check). Returns `0` otherwise.

### `isPromotion(Move *move, struct Board *board)` → `int`
Returns `1` if a pawn has reached the opponent's back rank and is eligible for promotion. Supports promotion to Queen, Rook, Bishop, or Knight.

### `promotePawn(struct Board *board, int promoteChoice)`
Removes the pawn, clears its square, and places the chosen piece type at the same position.

### `generateLegalMoves(struct Board *board, pieceColor color, Move moves[], int *moveCount)`
Populates `moves[]` with all legal moves for the given color and updates `*moveCount`. Used by the AI.

### `aiTurn(struct Board *board, pieceColor color)`
Executes the AI's move for the given color. Selects a move from the output of `generateLegalMoves`.

---

## Error Reference

| Message | Cause |
|--------|-------|
| `King not found on board` | King is missing from the board state — indicates an illegal capture or bad initialization |
| `Computer cannot move` | AI generated zero legal moves without triggering checkmate — indicates a logic error |
| `Invalid coordinates` | Input coordinates are out of bounds (e.g. `-1 -1`) |
| `No piece at that square` | Selected starting square is empty |
| `Not your piece` | Selected piece belongs to the opponent |
| `Can't put a piece there` | Destination coordinates are out of bounds (e.g. `9 9`) |
| `Invalid move` | Move passed coordinate checks but failed `isValid` |

---

## Packaging a Release

Full project archive:

```bash
tar -zcvf Chess_V1.0.tar.gz ./doc ./bin ./src ./Makefile
```

Documentation-only archive:

```bash
tar -zcvf Chess_V1.0_docs.tar.gz \
    ./doc/README \
    ./doc/COPYRIGHT \
    ./doc/Chess_UserManual.pdf \
    ./doc/INSTALL \
    ./Makefile \
    ./bin/*.bmp
```

---

## License

Permission to use, copy, modify, and distribute this software for any purpose with or without fees is hereby granted.

**THE SOFTWARE IS PROVIDED "AS IS."** All authors disclaim all warranties, express or implied. In no event shall any author be liable for any direct, indirect, or consequential damages arising from the use or performance of this software.

See [`doc/COPYRIGHT`](doc/COPYRIGHT) for the full license text.
