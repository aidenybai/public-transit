## General Rules

- MUST: Use TypeScript interfaces over types.
- MUST: Use arrow functions over function declarations.
- MUST: Never comment unless absolutely necessary.
  - If the code is a hack, prefix with // HACK: reason
- MUST: Use kebab-case for files.
- MUST: Use descriptive names for variables (avoid shorthands).
- MUST: Do not type cast ("as") unless absolutely necessary.
- MUST: Remove unused code and don't repeat yourself.
- MUST: Put all magic numbers in constants using `SCREAMING_SNAKE_CASE` with unit suffixes (`_MS`, `_PX`).

## Project Structure

Single-package pnpm project with dual build targets:

- `src/server.ts` — Node.js WebSocket relay server
- `src/client.ts` — Browser WebSocket client + agent provider
- `src/connection.ts` — Node.js connection helpers (auto-start server, connect to existing)
- `src/protocol.ts` — Shared constants and TypeScript interfaces
- `src/utils.ts` — Internal utilities
- `src/index.ts` — Re-exports everything

## Build

```bash
pnpm build   # production build with tsup
pnpm dev     # watch mode
```

## Key Commands

- **Install**: `pnpm install`
- **Build**: `pnpm build`
- **Dev**: `pnpm dev`
