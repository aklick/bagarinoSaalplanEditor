# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a single-file HTML seating plan editor (Saalplan-Editor) for creating and managing venue seating layouts. The application is entirely self-contained in `SaalplanEditor.html` with no build process or external dependencies beyond the jsPDF CDN library.

## Architecture

### Single-Page Application Structure
- **Language**: Vanilla JavaScript (ES6+) with HTML5 Canvas
- **No build tools**: The entire application exists in one HTML file
- **Persistence**: Uses browser LocalStorage for automatic saving
- **External dependencies**: jsPDF library (loaded via CDN) for PDF export

### Core Components

The application is built around a single `SeatPlanEditor` class (starts at line 584) that manages:

1. **Canvas State Management**
   - Seats array containing all seat objects with properties: `x, y, block, row, seat, priceCategory, size, shape`
   - Selection tracking using a Set
   - Undo history (max 50 states)
   - Background image support

2. **Operating Modes** (controlled via `mode` property):
   - `info`: Display seat information on click (default)
   - `add`: Place new seat blocks (triggered by modal confirmation)
   - `select`: Select individual seats or use lasso selection
   - `move`: Drag selected seats to new positions

3. **Event System**
   - Mouse events: `mousedown`, `mousemove`, `mouseup`, `click` for canvas interaction
   - Keyboard shortcuts: `Enter` (exit move mode), `Escape` (cancel selection/return to info mode)

### Key Features

- **Seat Block Creation**: Modal-based configuration (line 689-709) for creating structured seat layouts with:
  - Horizontal or vertical row orientation
  - Configurable row/seat numbering (including start numbers and positions)
  - Three price categories with color coding
  - Canvas boundary validation before placement

- **Selection System**:
  - Lasso selection (drag rectangle)
  - Click to toggle individual seats
  - Selected seats can be moved, deleted, or have categories changed

- **Data Persistence**:
  - Auto-saves to LocalStorage after each modification
  - Import/Export JSON with metadata
  - PDF export with legend

- **Visual Elements**:
  - Row labels rendered on both sides (line 979-1038)
  - Background image support with scaling
  - Color-coded seats by price category (defined at line 600-604)

## State Flow

1. User opens modal to add seats → fills configuration → clicks "OK - Platzieren"
2. Mode switches to `add`, modal closes
3. User clicks canvas position
4. `addSeatsAtPosition()` validates boundaries, creates seat grid, saves state
5. Renders seats, updates stats, saves to LocalStorage
6. Automatically returns to `info` mode

## Data Structures

### Seat Object
```javascript
{
  x: number,           // Canvas X coordinate
  y: number,           // Canvas Y coordinate
  block: string,       // Block identifier (e.g., "Block A")
  row: number,         // Row number
  seat: number,        // Seat number within row
  priceCategory: "1"|"2"|"3",
  size: number,        // Diameter/width in pixels
  shape: "circle"|"square"
}
```

### Export Format
```javascript
{
  seats: Array<Seat>,
  metadata: {
    created: ISO8601_timestamp,
    totalSeats: number
  }
}
```

## Testing & Running

Simply open `SaalplanEditor.html` in a modern web browser. No server or build process required.

## Important Implementation Notes

- Canvas coordinates are relative to the canvas element, not the viewport (rect calculation at line 672-673)
- Boundary checking prevents seats from being placed outside the 1200x800 canvas (line 792-795)
- Row labels are grouped by `block_row` key and calculated from all seats in that row (line 979-1038)
- Background images are stored in LocalStorage as data URLs (line 636-641)
- Undo system uses deep cloning via JSON parse/stringify (line 1164)
