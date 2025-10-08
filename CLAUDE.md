# chrome-bridge-mcp Development Guidelines

Auto-generated from all feature plans. Last updated: 2025-09-30

## Active Technologies
- TypeScript 5.x + WXT (Chrome Extension), XMCP (MCP Server), pnpm (monorepo) (001-c-t-ph)
- TypeScript 5.x + WXT (Chrome Extension), xmcp (MCP Server), pnpm (monorepo) (001-c-t-ph)
- Chrome storage API (local), server-side persistent storage (001-c-t-ph)

## Project Structure
```
backend/
frontend/
tests/
```

## Commands
npm test [ONLY COMMANDS FOR ACTIVE TECHNOLOGIES][ONLY COMMANDS FOR ACTIVE TECHNOLOGIES] npm run lint

## Code Style
TypeScript 5.x: Follow standard conventions

## Development Approach
**Hybrid: Contract-First + Bottom-Up**
- Phase 1: Define contracts (OpenAPI specs, TypeScript types, mock servers)
- Phase 2: Parallel implementation (Frontend & Backend tracks guided by contracts)
- Phase 3: Integration & validation (Contract compliance, E2E tests)

**Key Principles**:
- All APIs must comply with OpenAPI specifications
- TypeScript types generated from contracts
- Mock servers for parallel development
- Contract testing with Pact
- Type safety throughout the codebase

## Recent Changes
- 001-c-t-ph: Added TypeScript 5.x + WXT (Chrome Extension), xmcp (MCP Server), pnpm (monorepo)
- 001-c-t-ph: Added TypeScript 5.x + WXT (Chrome Extension), XMCP (MCP Server), pnpm (monorepo)

<!-- MANUAL ADDITIONS START -->
<!-- MANUAL ADDITIONS END -->
