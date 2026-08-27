# jiuwenswarm-vibestudio

**VibeStudio** — build full-stack web apps through natural language conversation, powered by WorkSwarm.

Describe what you want. A coordinated team of WorkSwarm agents writes the code, and a live in-browser preview appears immediately. No terminal, no configuration, no prior coding knowledge required.

**Requires:** a running WorkSwarm gateway (default `ws://localhost:19000`).

## Features

- Conversational app building with a live in-browser preview (Sandpack, bundled offline)
- Code drawer with file explorer and Monaco editor
- Inline Swarm activity feed showing each agent's work in real time
- 6 starter templates, quick-action buttons, agent clarification (HITL)
- ZIP download, chat export, undo/rewind, project persistence in `localStorage`
- Responsive mobile layout

## Quick Start

```bash
# 0. Start WorkSwarm (must be running)
jiuwenswarm-start            # gateway on ws://localhost:19000

# 1. Install deps
npm install

# 2. Configure env
cp .env.example .env         # VITE_JIUWENSWARM_URL defaults to ws://localhost:19000/v1/ws

# 3. Run
npm run dev                  # → http://localhost:5174
```

## Documentation

- [User Guide](docs/en/vibestudio/VibeStudioGuide.md)
- [Installation](docs/en/vibestudio/VibeStudioInstall.md)

## Development

```bash
npm run dev         # Vite dev server on :5174
npm test            # Vitest unit tests
npm run typecheck   # TypeScript check
npm run build       # production build → dist/
npm run preview     # serve the production build
```

## Repository layout

```
vibestudio/  The Vite/React app (src, tests, public, examples, configs)
docs/        User-facing docs (en + zh)
internal/    Working/design docs (not shipped)
```
