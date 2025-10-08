
# Implementation Plan: Chrome Extension + MCP Server Integration

**Branch**: `001-c-t-ph` | **Date**: 2025-09-30 | **Spec**: /specs/001-c-t-ph/spec.md
**Input**: Feature specification from `/specs/001-c-t-ph/spec.md`

## Execution Flow (/plan command scope)
```
1. Load feature spec from Input path
   → If not found: ERROR "No feature spec at {path}"
2. Fill Technical Context (scan for NEEDS CLARIFICATION)
   → Detect Project Type from file system structure or context (web=frontend+backend, mobile=app+api)
   → Set Structure Decision based on project type
3. Fill the Constitution Check section based on the content of the constitution document.
4. Evaluate Constitution Check section below
   → If violations exist: Document in Complexity Tracking
   → If no justification possible: ERROR "Simplify approach first"
   → Update Progress Tracking: Initial Constitution Check
5. Execute Phase 0 → research.md
   → If NEEDS CLARIFICATION remain: ERROR "Resolve unknowns"
6. Execute Phase 1 → architecture.md, component-design.md, ui-ux-mockup.md (if frontend), error-handling.md, contracts/, data-model.md, quickstart.md, agent-specific template file (e.g., `CLAUDE.md` for Claude Code, `.github/copilot-instructions.md` for GitHub Copilot, `GEMINI.md` for Gemini CLI, `QWEN.md` for Qwen Code or `AGENTS.md` for opencode).
7. Re-evaluate Constitution Check section
   → If new violations: Refactor design, return to Phase 1
   → Update Progress Tracking: Post-Design Constitution Check
8. Plan Phase 2 → Describe task generation approach (DO NOT create tasks.md)
9. STOP - Ready for /tasks command
```

**IMPORTANT**: The /plan command STOPS at step 7. Phases 2-4 are executed by other commands:
- Phase 2: /tasks command creates tasks.md
- Phase 3-4: Implementation execution (manual or via tools)

## Summary
Chrome Extension with MCP Server integration for web activity tracking, enabling users to capture events, logs, network traffic, and screenshots on specific domains, with MCP clients able to query and control the extension remotely.

## Technical Context
**Language/Version**: TypeScript 5.x
**Primary Dependencies**: WXT (Chrome Extension), xmcp (MCP Server), pnpm (monorepo)
**Storage**: Chrome storage API (local), server-side persistent storage
**Testing**: Vitest with WXT testing utilities
**Target Platform**: Chrome Extension (Manifest V3) + MCP Server (Node.js)
**Project Type**: Monorepo (chrome-extension + mcp-server)
**Performance Goals**: <5% performance overhead, <50MB memory per session, <1s response time
**Constraints**: Manifest V3 limitations, browser storage quotas, no hard limits on data capture
**Scale/Scope**: Single user sessions, domain-specific tracking, public MCP client access

## Constitution Check
*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

### Monorepo Structure ✅
- **Requirement**: Monorepo with pnpm workspaces and Turborepo
- **Compliance**: Project structure uses apps/ and packages/ directories
- **Status**: PASS

### Chrome Extension Development (WXT) ✅
- **Requirement**: WXT framework with Manifest V3
- **Compliance**: Extension uses WXT with entrypoints/ structure
- **Status**: PASS

### MCP Server Development (XMCP) ✅
- **Requirement**: xmcp framework with zod validation
- **Compliance**: Server uses xmcp with tools/ and resources/ directories
- **Status**: PASS

### Code Quality and Tooling ✅
- **Requirement**: TypeScript 5.x with strict options, Prettier, ESLint
- **Compliance**: All code uses TypeScript with monorepo-wide linting
- **Status**: PASS

### Testing (Vitest) ✅
- **Requirement**: Vitest with WXT testing utilities
- **Compliance**: Test files colocated with source code
- **Status**: PASS

### Browser APIs and Storage ✅
- **Requirement**: WXT's browser and storage APIs
- **Compliance**: Extension uses WXT's unified APIs
- **Status**: PASS

### Dependency Management ✅
- **Requirement**: pnpm with hoisted dependencies
- **Compliance**: Root package.json manages shared dependencies
- **Status**: PASS

## Project Structure

### Documentation (this feature)
```
specs/[###-feature]/
├── plan.md              # This file (/plan command output)
├── research.md          # Phase 0 output (/plan command)
├── architecture.md      # Phase 1 output (/plan command) - Overall system architecture
├── component-design.md  # Phase 1 output (/plan command) - Component design with sequence diagrams
├── ui-ux-mockup.md     # Phase 1 output (/plan command) - UI/UX mockups (if frontend/extension/mobile)
├── error-handling.md    # Phase 1 output (/plan command) - Error handling strategy
├── data-model.md        # Phase 1 output (/plan command)
├── quickstart.md        # Phase 1 output (/plan command)
├── contracts/           # Phase 1 output (/plan command)
└── tasks.md             # Phase 2 output (/tasks command - NOT created by /plan)
```

### Source Code (repository root)
```
apps/
├── chrome-extension/
│   ├── entrypoints/
│   │   ├── background.ts
│   │   ├── content.ts
│   │   ├── popup.html
│   │   └── options.html
│   ├── components/
│   │   ├── SessionControls.ts
│   │   ├── DataViewer.ts
│   │   └── ExportControls.ts
│   ├── services/
│   │   ├── SessionManager.ts
│   │   ├── EventCapture.ts
│   │   ├── NetworkMonitor.ts
│   │   ├── ScreenshotCapture.ts
│   │   └── MCPSync.ts
│   ├── types/
│   │   ├── events.ts
│   │   ├── session.ts
│   │   └── mcp.ts
│   ├── utils/
│   │   ├── storage.ts
│   │   ├── validation.ts
│   │   └── filtering.ts
│   └── tests/
│       ├── unit/
│       ├── integration/
│       └── e2e/
├── mcp-server/
│   ├── src/
│   │   ├── tools/
│   │   │   ├── SessionTools.ts
│   │   │   ├── DataTools.ts
│   │   │   └── ControlTools.ts
│   │   ├── resources/
│   │   │   ├── SessionResource.ts
│   │   │   └── DataResource.ts
│   │   ├── prompts/
│   │   │   ├── AnalysisPrompts.ts
│   │   │   └── ControlPrompts.ts
│   │   ├── services/
│   │   │   ├── SessionService.ts
│   │   │   ├── DataService.ts
│   │   │   └── ExtensionService.ts
│   │   ├── types/
│   │   │   ├── mcp.ts
│   │   │   └── session.ts
│   │   └── utils/
│   │       ├── validation.ts
│   │       └── storage.ts
│   └── tests/
│       ├── unit/
│       └── integration/
└── web-interface/
    ├── src/
    │   ├── components/
    │   │   ├── SessionManager.tsx
    │   │   ├── DataViewer.tsx
    │   │   └── FilterControls.tsx
    │   ├── pages/
    │   │   ├── Dashboard.tsx
    │   │   └── SessionView.tsx
    │   ├── services/
    │   │   ├── APIClient.ts
    │   │   └── WebSocketClient.ts
    │   └── types/
    │       └── api.ts
    └── tests/
        ├── unit/
        └── integration/

packages/
├── shared/
│   ├── types/
│   │   ├── events.ts
│   │   ├── session.ts
│   │   └── network.ts
│   ├── utils/
│   │   ├── validation.ts
│   │   ├── filtering.ts
│   │   └── serialization.ts
│   └── constants/
│       ├── events.ts
│       └── storage.ts
└── testing/
    ├── fixtures/
    ├── mocks/
    └── utils/

tests/
├── contract/
├── integration/
└── e2e/
```

**Structure Decision**: Monorepo with three main applications (chrome-extension, mcp-server, web-interface) and shared packages. The chrome-extension uses WXT framework structure, mcp-server uses xmcp framework structure, and web-interface provides the management UI.

## Phase 0: Outline & Research
1. **Extract unknowns from Technical Context** above:
   - For each NEEDS CLARIFICATION → research task
   - For each dependency → best practices task
   - For each integration → patterns task

2. **Generate and dispatch research agents**:
   ```
   For each unknown in Technical Context:
     Task: "Research {unknown} for {feature context}"
   For each technology choice:
     Task: "Find best practices for {tech} in {domain}"
   ```

3. **Consolidate findings** in `research.md` using format:
   - Decision: [what was chosen]
   - Rationale: [why chosen]
   - Alternatives considered: [what else evaluated]

**Output**: research.md with all NEEDS CLARIFICATION resolved

## Phase 1: Design & Contracts
*Prerequisites: research.md complete*

1. **Generate architecture overview** → `architecture.md`:
   - High-level system architecture for entire repository
   - Technology stack and framework choices
   - Deployment architecture and infrastructure
   - Integration patterns between components
   - **Include architecture diagram using Mermaid syntax**

2. **Generate component design** → `component-design.md`:
   - Detailed component breakdown for entire system
   - Component responsibilities and interfaces
   - Data flow between components
   - **Include sequential diagrams using Mermaid syntax**
   - Component interaction patterns

3. **Generate UI/UX mockup design** → `ui-ux-mockup.md` (if frontend/extension/mobile):
   - User interface mockups in simple markdown format
   - User experience flow diagrams
   - Screen layouts and navigation patterns
   - Visual design guidelines
   - **Skip this file if backend-only project**

4. **Generate error handling strategy** → `error-handling.md`:
   - Error handling patterns for both frontend and backend
   - Error classification and severity levels
   - Error propagation strategies
   - User-facing error messages and recovery flows
   - Logging and monitoring strategies

5. **Extract entities from feature spec** → `data-model.md`:
   - Entity name, fields, relationships
   - Validation rules from requirements
   - State transitions if applicable
   - **Generate ERD diagram using Mermaid syntax**

6. **Generate API contracts** from functional requirements:
   - For each user action → endpoint
   - Use standard REST/GraphQL patterns
   - Output OpenAPI/GraphQL schema to `/contracts/`

7. **Generate contract tests** from contracts:
   - One test file per endpoint
   - Assert request/response schemas
   - Tests must fail (no implementation yet)

8. **Extract test scenarios** from user stories:
   - Each story → integration test scenario
   - Quickstart test = story validation steps

9. **Update agent file incrementally** (O(1) operation):
   - Run `.specify/scripts/bash/update-agent-context.sh claude`
     **IMPORTANT**: Execute it exactly as specified above. Do not add or remove any arguments.
   - If exists: Add only NEW tech from current plan
   - Preserve manual additions between markers
   - Update recent changes (keep last 3)
   - Keep under 150 lines for token efficiency
   - Output to repository root

**Output**: architecture.md, component-design.md, ui-ux-mockup.md (if applicable), error-handling.md, data-model.md (with ERD), /contracts/*, failing tests, quickstart.md, agent-specific file

## Phase 2: Task Planning Approach
*This section describes what the /tasks command will do - DO NOT execute during /plan*

## Task Generation Strategy

The `/tasks` command will generate tasks.md by analyzing:
1. **Architecture dependencies** from `architecture.md`
2. **Component implementation order** from `component-design.md`
3. **UI/UX implementation priorities** from `ui-ux-mockup.md`
4. **Error handling implementation** from `error-handling.md`
5. **Data model dependencies** from `data-model.md`
6. **Contract requirements** from `/contracts/`
7. **Test scenarios** from `quickstart.md`

Tasks will be ordered by:
- Infrastructure setup first
- Core architecture components
- Data layer implementation
- API/contract implementation
- Error handling integration
- UI/UX implementation
- Integration testing
- End-to-end validation

**Task Generation Strategy**:
- Load `.specify/templates/tasks-template.md` as base
- Generate tasks from Phase 1 design docs (architecture, component-design, ui-ux-mockup, error-handling, data-model, contracts, quickstart)
- Each contract → contract test task [P]
- Each entity → model creation task [P]
- Each story → integration test task [P]
- Each component → implementation task [S]

**Ordering Strategy**:
- TDD order: Tests before implementation
- Dependency order: Architecture → Components → Models → Services → UI
- Mark [P] for parallel execution (independent files)

**Estimated Output**: 25-30 numbered, ordered tasks in tasks.md

**Ready for /tasks command execution**: All Phase 1 artifacts complete and validated

**IMPORTANT**: This phase is executed by the /tasks command, NOT by /plan

## Phase 3+: Future Implementation
*These phases are beyond the scope of the /plan command*

**Phase 3**: Task execution (/tasks command creates tasks.md)  
**Phase 4**: Implementation (execute tasks.md following constitutional principles)  
**Phase 5**: Validation (run tests, execute quickstart.md, performance validation)

## Complexity Tracking
*Fill ONLY if Constitution Check has violations that must be justified*

| Violation | Why Needed | Simpler Alternative Rejected Because |
|-----------|------------|-------------------------------------|
| [e.g., 4th project] | [current need] | [why 3 projects insufficient] |
| [e.g., Repository pattern] | [specific problem] | [why direct DB access insufficient] |


## Progress Tracking
*This checklist is updated during execution flow*

**Phase Status**:
- [x] Phase 0: Research complete (/plan command)
- [x] Phase 1: Design complete (/plan command)
- [ ] Phase 2: Task planning complete (/plan command - describe approach only)
- [ ] Phase 3: Tasks generated (/tasks command)
- [ ] Phase 4: Implementation complete
- [ ] Phase 5: Validation passed

**Gate Status**:
- [x] Initial Constitution Check: PASS
- [x] Post-Design Constitution Check: PASS
- [x] All NEEDS CLARIFICATION resolved
- [x] Complexity deviations documented

---
*Based on Constitution v1.0.0 - See `/memory/constitution.md`*
