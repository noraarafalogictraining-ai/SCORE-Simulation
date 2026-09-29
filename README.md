# SCORE Simulation

SCORE Performance Management Simulation (NextWave Digital, Commercial, Version 18), packaged as a Vite + React app that deploys to Vercel.

## Run locally

```bash
npm install
npm run dev      # http://localhost:5173
npm run build    # production build in dist/
```

## Deploy to Vercel

1. In Vercel, choose **Add New → Project** and import this GitHub repository.
2. Vercel detects **Vite** automatically: build command `npm run build`, output directory `dist`. No environment variables are needed.
3. Click **Deploy**.

## Project layout

```
index.html               host page
src/main.jsx             React entry (StrictMode)
src/App.jsx              renders <ScoreSimulation height="100vh" />
src/ScoreSimulation.jsx  the simulation component (default export) + SCORE_HTML (named export)
```

> `source/index.html`, `scripts/build-jsx.mjs` and `examples/`, mentioned below, are not in this repository yet. Until they are added, content edits are made in the `SIM_CONFIG` block inside the `SCORE_HTML` string in `src/ScoreSimulation.jsx`.

---

## Component reference

This package holds the SCORE Performance Management Simulation (NextWave Digital, Commercial, Version 18) as a React component.

The component runs the **complete, tested simulation unchanged** inside an isolated, same-origin frame. Everything behaves exactly as in the standalone HTML build:
- scoring (Black Mirror)
- branching
- Team Health
- Reflection
- scoreboard and awards
- saved participant state

The simulation's styles, hash routing and globals never leak into your app, and your app's CSS never breaks the simulation.

### Contents

```
ScoreSimulation.jsx      the component (default export) + SCORE_HTML (named export)
source/index.html        editable source of the simulation (same as the standalone build)
scripts/build-jsx.mjs    regenerates ScoreSimulation.jsx from source/index.html
examples/vite-App.jsx    Vite / CRA usage
examples/next-page.jsx   Next.js App Router usage (client-only)
```

### Install

Copy `ScoreSimulation.jsx` into your components folder. The only peer dependency is React 17 or later; the component was tested with React 18 in StrictMode. There are no other packages.

```jsx
import ScoreSimulation from "./ScoreSimulation";

export default function App() {
  return <ScoreSimulation height="100vh" />;
}
```

**Next.js:** the component uses browser-only APIs, so load it with `dynamic(..., { ssr: false })`. See `examples/next-page.jsx`.

### Props

| Prop | Default | Description |
|---|---|---|
| `height` | `"100vh"` | CSS height of the simulation frame |
| `initialView` | none | Start view: `home`, `company`, `hub`, `employee-omar`…, `r0`–`r4`, `h2`, `reflection` (locked stages stay locked) |
| `onReady` | none | `callback(api)` receiving the simulation's debug API (`config`, `store`, `go(view)`, `systems`…) |
| `className`, `style`, `title` | none | Passed to the `<iframe>` |

### Behaviour notes

- **Saved state:**
  - Participant data is stored in the host origin's `localStorage` under `score-sim::nextwave::commercial::v3`.
  - It survives page reloads and remounts.
  - Clear it with "Reset participant data" in the simulation footer.
- **Navigation:**
  - The browser back button moves between simulation pages.
  - Your app's URL is never changed.
- **Network:** none. There is no AI service and no API keys. The only outbound request is Google Fonts, which falls back to system fonts if blocked.
- **Content Security Policy:** if your app sets a CSP, allow `frame-src blob:`.
- **Bundle size:** about 1 MB, mostly embedded photos. Lazy-load the route that shows it, e.g. `React.lazy` or Next.js `dynamic`.

### Editing content

All content and rules live in the `SIM_CONFIG` block of `source/index.html`. To edit:

1. Edit `SIM_CONFIG` in `source/index.html`. Open the file directly in a browser to check the result.
2. Run `node scripts/build-jsx.mjs` to regenerate the component.
3. Run a journey to confirm.

Keep these rules:
- **Stage ids:** keep `r0`–`r4`, `h2` and `reflection`, and keep the saved-state shape. If the shape changes, bump `meta.storageKey`.
- **Hidden fields:** the following exist for scoring and the facilitator debrief only. Never render them to participants:
  - `managerChallenge`
  - `coreTension`
  - employee-specific `potentialBiases`
  - `rounds.r3.evidence[id].coreJudgment`

  `developmentAreas` appear only in Looking Ahead, labelled "Development Evidence".
- **API keys:** never add API keys. The optional AI checker (`aiAssist`) stays `false` until a secure server-side service exists.

### Verified

The component was tested in a React 18 StrictMode host app. The tests covered:
- rendering, and the host page staying untouched
- routing to Company Profile, the Commercial Hub and the employee pages
- browser back inside the frame, with the host URL unchanged
- reference dialogs keeping typed contract work
- state persisting across a full host-page reload
- the `onReady` API

The embedded document is byte-identical to the standalone Version 18 build, which passes the full participant journey regression (final index 96) and the hidden-interpretation scan.

A native React rewrite (components and hooks instead of the embedded engine) would be a separate project and would need full regression testing against this version.
