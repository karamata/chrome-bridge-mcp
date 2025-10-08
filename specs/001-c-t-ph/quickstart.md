# Quickstart Guide: Chrome Extension + MCP Server Integration

## Prerequisites

- Node.js 20.x
- pnpm package manager
- Chrome browser
- MCP client (Claude, GPT, Cursor, etc.)

## Installation

1. **Clone and setup the repository**:
   ```bash
   git clone <repository-url>
   cd chrome-bridge-mcp
   pnpm install
   ```

2. **Build all packages**:
   ```bash
   pnpm build
   ```

## Chrome Extension Setup

1. **Build the extension**:
   ```bash
   cd apps/chrome-extension
   pnpm build
   ```

2. **Load the extension in Chrome**:
   - Open Chrome and navigate to `chrome://extensions/`
   - Enable "Developer mode" in the top right
   - Click "Load unpacked"
   - Select the `apps/chrome-extension/.output/chrome-mv3` directory

3. **Verify extension installation**:
   - You should see the Chrome Bridge extension icon in the toolbar
   - Click the icon to open the popup interface

## MCP Server Setup

1. **Start the MCP server**:
   ```bash
   cd apps/mcp-server
   pnpm start
   ```

2. **Configure your MCP client**:
   - Add the MCP server to your client's configuration
   - Use the server address (e.g., `ws://localhost:3000` for WebSocket)

## Basic Usage

### Starting a Session

1. **Via Chrome Extension**:
   - Navigate to the domain you want to track
   - Click the extension icon
   - Enter a session title (optional)
   - Click "Start Session"

2. **Via MCP Client**:
   ```json
   {
     "tool": "start_session",
     "args": {
       "domain": "https://example.com",
       "title": "Test Session"
     }
   }
   ```

### Capturing Data

1. **The extension will automatically capture**:
   - Click events on elements
   - Input events in form fields
   - Navigation changes
   - Console logs (log, error, warn, info)
   - Network requests (excluding sensitive ones)
   - Screenshots (automatic and manual)

2. **Manual screenshot capture**:
   ```json
   {
     "tool": "capture_screenshot",
     "args": {
       "sessionId": "session-id-here"
     }
   }
   ```

### Querying Data

1. **List available sessions**:
   ```json
   {
     "tool": "list_sessions",
     "args": {
       "domain": "https://example.com",
       "status": "active"
     }
   }
   ```

2. **Get session data**:
   ```json
   {
     "tool": "get_session",
     "args": {
       "sessionId": "session-id-here",
       "includeEvents": true,
       "includeLogs": true,
       "includeNetwork": true,
       "includeScreenshots": true
     }
   }
   ```

3. **Query specific data types**:
   ```json
   {
     "tool": "query_events",
     "args": {
       "sessionId": "session-id-here",
       "type": "click",
       "limit": 50
     }
   }
   ```

### Stopping a Session

1. **Via Chrome Extension**:
   - Click the extension icon
   - Click "Stop Session"

2. **Via MCP Client**:
   ```json
   {
     "tool": "stop_session",
     "args": {
       "sessionId": "session-id-here"
     }
   }
   ```

### Exporting Data

1. **Export session as JSON**:
   ```json
   {
     "tool": "export_session",
     "args": {
       "sessionId": "session-id-here",
       "format": "json"
     }
   }
   ```

## Testing

### Running Tests

1. **Run all tests**:
   ```bash
   pnpm test
   ```

2. **Run tests for specific package**:
   ```bash
   cd apps/chrome-extension
   pnpm test
   ```

### Test Scenarios

1. **Session Management**:
   - Start and stop sessions
   - Verify session data capture
   - Test session persistence

2. **Data Capture**:
   - Event capture accuracy
   - Network request filtering
   - Screenshot capture

3. **MCP Integration**:
   - Tool execution
   - Data querying
   - Error handling

## Development

### Development Mode

1. **Start extension in development mode**:
   ```bash
   cd apps/chrome-extension
   pnpm dev
   ```

2. **Start MCP server in development mode**:
   ```bash
   cd apps/mcp-server
   pnpm dev
   ```

### Code Structure

```
apps/
├── chrome-extension/
│   ├── entrypoints/     # Extension entry points
│   ├── components/      # UI components
│   ├── utils/          # Utility functions
│   └── tests/          # Extension tests
├── mcp-server/
│   ├── src/
│   │   ├── tools/      # MCP tools
│   │   ├── prompts/    # MCP prompts
│   │   ├── resources/  # MCP resources
│   │   └── utils/      # Server utilities
│   └── tests/          # Server tests
└── web-interface/
    ├── src/
    │   ├── components/ # Web UI components
    │   ├── pages/      # Web UI pages
    │   └── services/   # API services
    └── tests/          # Web UI tests

packages/
├── shared-types/       # Shared TypeScript types
├── shared-utils/       # Shared utilities
└── storage/           # Storage utilities
```

## Troubleshooting

### Common Issues

1. **Extension not loading**:
   - Ensure you're using Chrome with Manifest V3 support
   - Check for errors in the extension manifest
   - Verify build completed successfully

2. **MCP server connection issues**:
   - Check if server is running on correct port
   - Verify WebSocket/HTTP configuration
   - Check client configuration

3. **Data not capturing**:
   - Ensure session is active
   - Check domain permissions
   - Verify content script injection

### Debug Mode

1. **Enable debug logging**:
   ```bash
   export DEBUG=chrome-bridge:*
   pnpm dev
   ```

2. **Chrome DevTools**:
   - Use extension popup DevTools for UI debugging
   - Use background script DevTools for service worker debugging
   - Use content script DevTools for page interaction debugging

## Configuration

### Environment Variables

Create a `.env` file in the repository root:

```env
# MCP Server Configuration
MCP_SERVER_PORT=3000
MCP_SERVER_HOST=localhost

# Chrome Extension Configuration
EXTENSION_ID=your-extension-id

# Development Configuration
NODE_ENV=development
DEBUG=chrome-bridge:*
```

### Customization

1. **Add new event types**: Update event capture logic in content scripts
2. **Modify data filtering**: Adjust network request filtering rules
3. **Custom MCP tools**: Add new tools in the MCP server
4. **UI customization**: Modify web interface components

## Contributing

1. **Follow the constitution**: Adhere to project constitution guidelines
2. **Write tests**: Ensure all new code has corresponding tests
3. **Update documentation**: Keep docs in sync with code changes
4. **Code style**: Use Prettier and ESLint formatting