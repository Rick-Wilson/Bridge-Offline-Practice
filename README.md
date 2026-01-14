# Bridge Bidding Practice

A Vue.js web application for practicing bridge bidding using PBN (Portable Bridge Notation) files with pre-recorded auctions and commentary.

## Features

### Current Functionality

- **PBN File Support**: Load practice deals from standard PBN 2.1 format files
  - Bundled sample files (Cue-bid, Drury conventions)
  - Import your own PBN files via file picker

- **Interactive Bidding Practice**
  - Practice bidding from the student seat (specified in PBN)
  - Guided prompts from PBN commentary tell you when to bid
  - Immediate feedback on incorrect bids
  - Auto-plays bids that don't require user input

- **Hand Display**
  - Traditional bridge layout (North at top, South at bottom)
  - Suit symbols with proper colors (red for hearts/diamonds)
  - Opponent hands hidden during practice, revealed after auction

- **Auction Display**
  - Standard 4-column auction table (W-N-E-S)
  - Turn indicator shows when user input is needed
  - Visual feedback for correct/incorrect bids

- **Bidding Box**
  - Level selection (1-7)
  - Strain buttons with suit symbols
  - Pass, Double, Redouble options
  - Invalid bids automatically disabled

- **Session Tracking**
  - Correct/incorrect bid counter
  - Deal navigation (previous/next/select)
  - Full commentary displayed after auction completes

## Getting Started

### Prerequisites

- Node.js 20+
- npm

### Installation

```bash
npm install
```

### Development

```bash
npm run dev
```

Open http://localhost:5173 in your browser.

### Production Build

```bash
npm run build
```

## Usage

1. Click a bundled file button (e.g., "Cue-bid") or upload your own PBN file
2. Read the prompt text that appears above the bidding box
3. Select your bid using the bidding box
4. If incorrect, the correct bid is shown - click "Continue" to proceed
5. After the auction completes, review the full commentary
6. Click "Next Deal" or use navigation to continue practicing

## PBN File Format

The app reads standard PBN files with these key tags:

- `[Deal]` - Hand distributions for all four players
- `[Dealer]` - Which seat deals
- `[Vulnerable]` - Vulnerability status
- `[Auction]` - The complete bidding sequence
- `[Student]` - Which seat the user practices (typically "S")
- `[Contract]` - Final contract
- Commentary in `{...}` blocks with `[BID xxx]` markers for prompts

## Future Goals

### Student Progress Tracking
- Save practice session results locally
- Track accuracy per convention/topic
- Review history of completed deals
- Identify areas needing more practice

### Teacher Integration
- Export student results for instructor review
- Aggregate statistics across multiple sessions
- Identify common bidding errors
- Assign specific practice sets to students

### Additional Features
- PWA support for offline mobile use
- Full auction replay mode (step through without user input)
- Multiple bidding system support
- Custom convention card configuration

## Tech Stack

- Vue 3 (Composition API)
- Vite
- Plain CSS (scoped styles)

## License

GNU General Public License v3.0
