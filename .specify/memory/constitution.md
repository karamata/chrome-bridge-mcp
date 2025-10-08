<!-- Sync Impact Report -->
<!-- Version change: None → 1.0.0 -->
<!-- Modified principles: All principles replaced with Chrome Extension & MCP Server specific guidelines -->
<!-- Added sections: All sections are new additions -->
<!-- Removed sections: Previous generic template sections -->
<!-- Templates requiring updates: ✅ plan-template.md -->
<!-- Follow-up TODOs: None -->

# Chrome Bridge MCP Constitution

## Core Principles

### I. Monorepo Structure
The project is a monorepo managed with `pnpm` workspaces and orchestrated by Turborepo. All code resides within distinct sub-projects under the `packages/` or `apps/` directory. The two primary applications are `apps/chrome-extension` (built with WXT) and `apps/mcp-server` (built with xmcp). Shared utilities or libraries should be placed in `packages/`.

### II. Chrome Extension Development (WXT)
The Chrome extension **MUST** be developed using the **WXT** framework. All entrypoints (background scripts, content scripts, popups) **MUST** be defined within the `apps/chrome-extension/entrypoints/` directory, following WXT's file-based routing conventions. The extension will target Chrome with Manifest V3.

### III. MCP Server Development (XMCP)
The MCP server **MUST** be developed using the **xmcp** framework. All tools, prompts, and resources **MUST** be defined within their respective directories (`tools/`, `prompts/`, `resources/`) inside the `apps/mcp-server/src/` directory. All tool/prompt/resource schemas **MUST** use `zod` for validation. The server will be configured via `xmcp.config.ts` for either HTTP or STDIO transport as required by the feature plan.

### IV. Code Quality and Tooling
All code **MUST** be written in TypeScript with the strictest possible compiler options. The entire monorepo **MUST** be formatted using Prettier and linted with ESLint according to the root-level configurations. No code should be committed that does not pass linting and type-checking.

### V. Testing (Vitest)
All new logic, including xmcp tools, UI components, and utility functions, **MUST** be accompanied by unit tests. **Vitest** is the designated testing framework for the entire monorepo. WXT's testing utilities (`wxt/testing`) **MUST** be used for testing extension-specific components. Test files must be colocated with the source code they are testing.

### VI. Browser APIs and Storage
Interaction with browser APIs within the Chrome extension **MUST** utilize WXT's unified `browser` object (`import { browser } from 'wxt/browser'`). For client-side storage, the extension **MUST** use WXT's simplified storage API (`import { storage } from 'wxt/storage'`) to ensure type-safety and consistency.

### VII. Dependency Management
Dependencies **MUST** be managed using `pnpm`. Shared dependencies across the `chrome-extension` and `mcp-server` packages should be hoisted to the root `package.json` wherever possible to ensure version consistency and reduce duplication.

## Additional Constraints

- **Node.js Version**: The project standard is Node.js 20.x.
- **Package Manager**: `pnpm` is the only package manager to be used.
- **Manifest Version**: The Chrome Extension will strictly target Manifest V3.

## Development Workflow

1. **Branching**: All new features or fixes must be developed in a feature branch (e.g., `feature/name-of-feature`).
2. **Specification**: New features must begin with the `specify` CLI workflow (`/specify`, `/plan`, `/tasks`).
3. **Code Style**: Run `pnpm format` before committing to ensure code consistency.
4. **Pull Requests**: All code must be submitted via a Pull Request to the `main` branch. PRs must pass all CI checks (linting, testing, building) before being merged.

## Governance
This constitution is the supreme source of truth for the project's architecture and development practices. Any technical plan, task, or implementation that contradicts these principles is considered invalid. AI agents **MUST** strictly adhere to this constitution in all generated plans and code. Amendments to this document require a Pull Request and approval from the project maintainers.

**Version**: 1.0.0 | **Ratified**: 2023-10-27 | **Last Amended**: 2025-09-29