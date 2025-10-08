# Research: Chrome Extension + MCP Server Integration

## Chrome Extension Data Capture Research

### Decision: WXT Framework with Manifest V3
**Rationale**: WXT provides TypeScript-first development, simplified browser APIs, and built-in Manifest V3 support. It's the constitutionally mandated framework for this project.
**Alternatives considered**: Plain Chrome Extension API development (more complex), Plasmo (less TypeScript-focused)

### Decision: Event Capture Strategy
**Rationale**: Use content scripts for DOM event listeners and background script for cross-tab communication. Content scripts can capture click/input events directly, while background scripts handle network interception and storage.
**Alternatives considered**: Background script only (limited DOM access), injection scripts (less maintainable)

### Decision: Network Traffic Capture
**Rationale**: Use Chrome's `webRequest` API in background script with filtering for XHR/fetch/WebSocket. Sensitive requests (auth, passwords) will be excluded through URL pattern matching.
**Alternatives considered**: Service Workers (limited in Manifest V3), proxy approach (overkill)

### Decision: Screenshot Capture
**Rationale**: Use `chrome.tabs.captureVisibleTab()` API with base64 encoding. Screenshots will be triggered periodically and on significant events.
**Alternatives considered**: HTML canvas capture (less reliable), third-party libraries (unnecessary overhead)

## MCP Server Integration Research

### Decision: XMCP Framework
**Rationale**: XMCP provides structured tool/prompt/resource management with zod validation. It's the constitutionally mandated framework for this project.
**Alternatives considered**: Raw MCP SDK implementation (more manual work), custom server implementation (reinventing wheel)

### Decision: Real-time Sync Strategy
**Rationale**: WebSocket connection between extension and MCP server for real-time data sync. Fallback to HTTP polling for reliability.
**Alternatives considered**: HTTP only (less real-time), server-sent events (less flexible for bidirectional communication)

### Decision: MCP Tool Design
**Rationale**: Tools for session management, data querying, and extension control. Each tool will have specific zod schemas for input validation.
**Alternatives considered**: Generic CRUD tools (less type-safe), REST-like API (not MCP idiomatic)

## Storage Strategy Research

### Decision: Browser Storage (Local) + Server Storage
**Rationale**: Browser storage for current session data (cleared when new session starts), server storage for persistent data and historical access.
**Alternatives considered**: Only browser storage (limited by quotas), IndexedDB (complex for this use case)

### Decision: Data Structure
**Rationale**: Session-based organization with events, logs, network requests, and screenshots as separate arrays within each session. Timestamp-based indexing for querying.
**Alternatives considered**: Flat structure (harder to query), database-style (overkill for browser context)

## Security & Privacy Research

### Decision: Sensitive Data Filtering
**Rationale**: URL pattern matching to exclude auth endpoints, password fields, and common PII patterns. User-configurable domain whitelist for capture scope.
**Alternatives considered**: Capture everything (privacy risk), manual filtering (error-prone)

### Decision: Access Control
**Rationale**: Public access model as specified in requirements - any MCP client can access data. Session isolation by domain.
**Alternatives considered**: Authentication system (beyond scope), encryption (not required by spec)

## Performance & Scalability Research

### Decision: Event Debouncing
**Rationale**: Debounce rapid events (like typing) to capture final state rather than every keystroke. Batch network requests for efficiency.
**Alternatives considered**: Capture everything (performance impact), sampling (might miss important events)

### Decision: Data Compression
**Rationale**: JSON compression for network transfer, efficient data structures in browser. Limit screenshot frequency and resolution.
**Alternatives considered**: No compression (bandwidth issues), binary formats (less compatible)

## Testing Strategy Research

### Decision: Vitest with WXT Testing Utilities
**Rationale**: Constitutionally mandated testing framework. WXT provides specialized utilities for extension testing.
**Alternatives considered**: Jest (larger bundle size), Cypress (overkill for unit tests)

### Decision: Test Structure
**Rationale**: Unit tests for utilities, integration tests for extension flows, contract tests for MCP tools. Mock browser APIs for reliable testing.
**Alternatives considered**: Only unit tests (insufficient), end-to-end only (slow and flaky)

## Conclusion

All major technical decisions align with the project constitution and feature requirements. No significant blockers identified. The chosen stack (WXT + XMCP) provides a solid foundation for implementing the Chrome Extension + MCP Server integration feature.