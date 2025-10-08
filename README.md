# Chrome Bridge MCP

Chrome Bridge MCP is a web activity tracking system consisting of a Chrome extension that captures user interactions, console logs, network traffic, and screenshots, paired with an MCP server that stores this data and provides tools for AI clients to query and control the extension.

## Components

### Chrome Extension
- Captures user interactions (clicks, inputs, navigation)
- Records console logs, network traffic, and screenshots
- Works independently with local storage (IndexedDB)
- Supports JSON export and server synchronization
- Session-based tracking per domain

### MCP Server
- Receives and stores extension data persistently
- Provides MCP protocol interface for AI clients
- Offers query tools for sessions, events, logs, network data
- Enables remote control commands to extension
- Includes web interface for data management

## Key Features

- Real-time web activity capture
- Offline extension operation
- Server synchronization when connected
- AI client integration via MCP protocol
- Session management and data export

## Technology Stack

- **Chrome Extension**: WXT framework with TypeScript 5.x
- **MCP Server**: xmcp framework with TypeScript 5.x
- **Storage**: IndexedDB (local), SQLite/PostgreSQL (server)
- **Package Manager**: pnpm with monorepo structure
- **Testing**: Vitest

## Getting Started

1. Clone the repository
2. Install dependencies: `pnpm install`
3. Build the extension: `pnpm build`
4. Start the MCP server: `pnpm dev`

## License

MIT
