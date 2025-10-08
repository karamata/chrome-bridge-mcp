# Error Handling Strategy

## Error Classification

### Severity Levels
- **CRITICAL**: System failure, requires immediate attention
- **ERROR**: Operation failed, user action needed
- **WARNING**: Potential issue, monitoring recommended
- **INFO**: Normal operation, informational messages

### Error Categories
- **Network Errors**: Connection issues, timeouts, sync failures
- **Storage Errors**: Quota exceeded, data corruption, access denied
- **Permission Errors**: Missing permissions, access denied
- **Validation Errors**: Invalid input, malformed data
- **Extension Errors**: Chrome API failures, manifest issues
- **MCP Errors**: Tool execution failures, validation errors
- **UI Errors**: Rendering issues, interaction failures

## Chrome Extension Error Handling

### Background Service Worker Errors

#### Connection Management
```typescript
class ConnectionErrorHandler {
  private maxRetries = 3
  private retryDelay = 1000 // 1 second

  async handleConnectionError(error: Error, context: string): Promise<void> {
    console.error(`[${context}] Connection error:`, error)

    // Attempt reconnection with exponential backoff
    for (let attempt = 1; attempt <= this.maxRetries; attempt++) {
      try {
        await this.reconnect()
        console.log(`[${context}] Reconnected successfully`)
        return
      } catch (retryError) {
        console.warn(`[${context}] Reconnection attempt ${attempt} failed`)
        if (attempt < this.maxRetries) {
          await this.delay(this.retryDelay * Math.pow(2, attempt - 1))
        }
      }
    }

    // Notify user of connection failure
    this.notifyUser('connection_failed', 'Unable to connect to MCP server')
  }

  private async reconnect(): Promise<void> {
    // Reconnection logic
  }

  private delay(ms: number): Promise<void> {
    return new Promise(resolve => setTimeout(resolve, ms))
  }

  private notifyUser(type: string, message: string): void {
    // Send message to popup UI
    chrome.runtime.sendMessage({
      type: 'notification',
      payload: { type, message }
    })
  }
}
```

#### Storage Error Handling
```typescript
class StorageErrorHandler {
  async handleStorageError(error: Error, operation: string): Promise<void> {
    console.error(`Storage error during ${operation}:`, error)

    if (error.message.includes('QUOTA_BYTES')) {
      // Handle quota exceeded
      await this.handleQuotaExceeded()
    } else if (error.message.includes('ACCESS_DENIED')) {
      // Handle permission errors
      await this.handlePermissionError()
    } else {
      // Generic storage error
      this.notifyUser('storage_error', 'Unable to save data locally')
    }
  }

  private async handleQuotaExceeded(): Promise<void> {
    try {
      // Clear old data
      await this.clearOldData()

      // Compress remaining data
      await this.compressData()

      this.notifyUser('storage_warning', 'Local storage optimized')
    } catch (error) {
      console.error('Failed to handle quota exceeded:', error)
      this.notifyUser('storage_critical', 'Unable to free up storage space')
    }
  }

  private async clearOldData(): Promise<void> {
    // Clear old session data
    const sessions = await chrome.storage.local.get('sessions')
    // Keep only recent sessions
  }

  private async compressData(): Promise<void> {
    // Compress stored data
  }

  private notifyUser(type: string, message: string): void {
    // Notify user
  }
}
```

#### Event Processing Errors
```typescript
class EventProcessingErrorHandler {
  async handleEventError(error: Error, event: any): Promise<void> {
    console.error('Event processing error:', error)

    // Log error with event context
    await this.logError('event_processing', {
      error: error.message,
      eventType: event.type,
      timestamp: event.timestamp,
      url: event.url
    })

    // Continue processing other events
    // Don't let single event failure stop entire session
  }

  private async logError(type: string, details: any): Promise<void> {
    // Log error for later analysis
    const errorLog = await chrome.storage.local.get('errorLog') || []
    errorLog.push({
      type,
      details,
      timestamp: Date.now()
    })

    // Keep only recent errors
    if (errorLog.length > 100) {
      errorLog.splice(0, errorLog.length - 100)
    }

    await chrome.storage.local.set({ errorLog })
  }
}
```

### Content Script Error Handling

#### DOM Event Errors
```typescript
class DOMEventErrorHandler {
  handleDOMError(error: Error, eventType: string): void {
    console.warn(`DOM event error for ${eventType}:`, error)

    // Continue listening for events
    // Don't let DOM errors break event capture

    // Send error to background for tracking
    chrome.runtime.sendMessage({
      type: 'dom_error',
      payload: {
        error: error.message,
        eventType,
        url: window.location.href,
        timestamp: Date.now()
      }
    })
  }
}
```

#### Screenshot Capture Errors
```typescript
class ScreenshotErrorHandler {
  async handleScreenshotError(error: Error): Promise<void> {
    console.error('Screenshot capture error:', error)

    if (error.message.includes('tabs.captureVisibleTab')) {
      // Handle permission issues
      this.notifyUser('screenshot_permission', 'Unable to capture screenshots')
    } else {
      // Generic screenshot error
      this.notifyUser('screenshot_error', 'Screenshot capture failed')
    }
  }

  private notifyUser(type: string, message: string): void {
    // Notify user via background script
  }
}
```

### Popup UI Error Handling

#### User Action Errors
```typescript
class UIErrorHandler {
  handleUserActionError(error: Error, action: string): void {
    console.error(`UI error during ${action}:`, error)

    // Show user-friendly error message
    this.showErrorToast(this.getUserFriendlyMessage(error))

    // Log error for debugging
    this.logError('ui_action', {
      action,
      error: error.message,
      timestamp: Date.now()
    })
  }

  private getUserFriendlyMessage(error: Error): string {
    if (error.message.includes('network')) {
      return 'Network connection issue. Please check your internet connection.'
    }
    if (error.message.includes('permission')) {
      return 'Permission denied. Please check extension permissions.'
    }
    return 'An unexpected error occurred. Please try again.'
  }

  private showErrorToast(message: string): void {
    // Show toast notification
    const toast = document.createElement('div')
    toast.className = 'error-toast'
    toast.textContent = message
    document.body.appendChild(toast)

    setTimeout(() => {
      toast.remove()
    }, 5000)
  }
}
```

## MCP Server Error Handling

### Tool Execution Errors

#### Validation Errors
```typescript
class ToolValidationHandler {
  handleValidationError(error: z.ZodError, toolName: string): MCPResponse {
    console.warn(`Validation error for tool ${toolName}:`, error)

    return {
      success: false,
      error: {
        code: 'VALIDATION_ERROR',
        message: `Invalid input parameters for ${toolName}`,
        details: error.errors,
        suggestion: this.getValidationSuggestion(error.errors)
      }
    }
  }

  private getValidationSuggestion(errors: z.ZodError[]): string {
    // Provide helpful suggestions based on validation errors
    if (errors.some(e => e.code === 'invalid_type')) {
      return 'Please check parameter types and try again.'
    }
    if (errors.some(e => e.code === 'too_small')) {
      return 'Please provide all required parameters.'
    }
    return 'Please review your input parameters.'
  }
}
```

#### Execution Errors
```typescript
class ToolExecutionHandler {
  async handleExecutionError(error: Error, toolName: string, params: any): Promise<MCPResponse> {
    console.error(`Execution error for tool ${toolName}:`, error)

    // Log error for debugging
    await this.logError('tool_execution', {
      toolName,
      params,
      error: error.message,
      timestamp: Date.now()
    })

    return {
      success: false,
      error: {
        code: 'EXECUTION_ERROR',
        message: `Failed to execute ${toolName}`,
        details: error.message,
        retry: this.shouldRetry(error)
      }
    }
  }

  private shouldRetry(error: Error): boolean {
    // Determine if error is retryable
    if (error.message.includes('timeout')) return true
    if (error.message.includes('network')) return true
    if (error.message.includes('connection')) return true
    return false
  }
}
```

### Resource Access Errors

#### Session Not Found
```typescript
class SessionResourceHandler {
  handleSessionNotFound(sessionId: string): MCPResponse {
    console.warn(`Session not found: ${sessionId}`)

    return {
      success: false,
      error: {
        code: 'SESSION_NOT_FOUND',
        message: `Session ${sessionId} not found`,
        suggestion: 'Please check the session ID or list available sessions'
      }
    }
  }
}
```

#### Permission Errors
```typescript
class ResourcePermissionHandler {
  handlePermissionError(resource: string, action: string): MCPResponse {
    console.warn(`Permission error for ${resource}: ${action}`)

    return {
      success: false,
      error: {
        code: 'PERMISSION_DENIED',
        message: `Permission denied for ${action} on ${resource}`,
        suggestion: 'Please check your access permissions'
      }
    }
  }
}
```

### Connection Errors

#### WebSocket Errors
```typescript
class WebSocketErrorHandler {
  handleWebSocketError(error: Error): void {
    console.error('WebSocket error:', error)

    // Notify connected clients
    this.broadcastError({
      type: 'connection_error',
      message: 'WebSocket connection error',
      timestamp: Date.now()
    })

    // Attempt reconnection
    this.scheduleReconnection()
  }

  private broadcastError(error: any): void {
    // Broadcast error to all connected clients
  }

  private scheduleReconnection(): void {
    // Schedule reconnection attempt
    setTimeout(() => {
      this.reconnect()
    }, 5000)
  }

  private reconnect(): void {
    // Reconnection logic
  }
}
```

## Web Interface Error Handling

### API Error Handling

#### Network Errors
```typescript
class APIErrorHandler {
  async handleNetworkError(error: Error, endpoint: string): Promise<void> {
    console.error(`API error for ${endpoint}:`, error)

    // Show user-friendly error
    this.showNetworkError()

    // Attempt retry for temporary failures
    if (this.isRetryable(error)) {
      await this.retryRequest(endpoint)
    }
  }

  private showNetworkError(): void {
    // Show network error notification
    const notification = document.createElement('div')
    notification.className = 'network-error'
    notification.innerHTML = `
      <div class="error-content">
        <h3>Connection Error</h3>
        <p>Unable to connect to the server. Please check your connection.</p>
        <button onclick="this.parentElement.parentElement.remove()">Dismiss</button>
      </div>
    `
    document.body.appendChild(notification)
  }

  private isRetryable(error: Error): boolean {
    return error.message.includes('timeout') ||
           error.message.includes('network') ||
           error.message.includes('connection')
  }
}
```

#### Data Display Errors
```typescript
class DataDisplayErrorHandler {
  handleDataError(error: Error, dataType: string): void {
    console.error(`Data display error for ${dataType}:`, error)

    // Show error in data view
    this.showErrorInView(dataType, error.message)

    // Fallback to empty state
    this.showEmptyState(dataType)
  }

  private showErrorInView(dataType: string, message: string): void {
    const container = document.querySelector(`[data-type="${dataType}"]`)
    if (container) {
      container.innerHTML = `
        <div class="error-state">
          <h3>Error Loading ${dataType}</h3>
          <p>${message}</p>
          <button onclick="location.reload()">Retry</button>
        </div>
      `
    }
  }

  private showEmptyState(dataType: string): void {
    const container = document.querySelector(`[data-type="${dataType}"]`)
    if (container) {
      container.innerHTML = `
        <div class="empty-state">
          <h3>No ${dataType} Available</h3>
          <p>There might be an issue loading the data.</p>
        </div>
      `
    }
  }
}
```

## Error Recovery Strategies

### Automatic Recovery

#### Connection Recovery
```typescript
class ConnectionRecovery {
  private recoveryAttempts = 0
  private maxRecoveryAttempts = 5

  async recoverConnection(): Promise<boolean> {
    while (this.recoveryAttempts < this.maxRecoveryAttempts) {
      try {
        await this.testConnection()
        console.log('Connection recovered successfully')
        this.recoveryAttempts = 0
        return true
      } catch (error) {
        this.recoveryAttempts++
        console.warn(`Recovery attempt ${this.recoveryAttempts} failed`)
        await this.delay(1000 * this.recoveryAttempts)
      }
    }

    return false
  }

  private async testConnection(): Promise<void> {
    // Test connection logic
  }

  private delay(ms: number): Promise<void> {
    return new Promise(resolve => setTimeout(resolve, ms))
  }
}
```

#### Data Recovery
```typescript
class DataRecovery {
  async recoverSessionData(sessionId: string): Promise<boolean> {
    try {
      // Try to recover from local backup
      const localData = await this.getLocalBackup(sessionId)
      if (localData) {
        await this.restoreData(sessionId, localData)
        return true
      }

      // Try to recover from server
      const serverData = await this.getServerData(sessionId)
      if (serverData) {
        await this.restoreData(sessionId, serverData)
        return true
      }

      return false
    } catch (error) {
      console.error('Data recovery failed:', error)
      return false
    }
  }

  private async getLocalBackup(sessionId: string): Promise<any> {
    // Get backup from local storage
  }

  private async getServerData(sessionId: string): Promise<any> {
    // Get data from server
  }

  private async restoreData(sessionId: string, data: any): Promise<void> {
    // Restore data
  }
}
```

### User-Guided Recovery

#### Error Resolution UI
```typescript
class ErrorResolutionUI {
  showErrorResolution(error: Error, context: string): void {
    const resolution = this.getResolution(error, context)

    const modal = document.createElement('div')
    modal.className = 'error-resolution-modal'
    modal.innerHTML = `
      <div class="modal-content">
        <h3>${resolution.title}</h3>
        <p>${resolution.description}</p>
        <div class="resolution-steps">
          ${resolution.steps.map(step => `<p>${step}</p>`).join('')}
        </div>
        <div class="modal-actions">
          <button onclick="this.closest('.error-resolution-modal').remove()">Close</button>
          ${resolution.action ? `<button onclick="${resolution.action}">${resolution.actionLabel}</button>` : ''}
        </div>
      </div>
    `

    document.body.appendChild(modal)
  }

  private getResolution(error: Error, context: string): any {
    // Return resolution steps based on error type
    if (error.message.includes('quota')) {
      return {
        title: 'Storage Space Full',
        description: 'Your browser storage is full. Please clear some data.',
        steps: [
          'Click "Clear Local Data" in settings',
          'Export important sessions first',
          'Reduce screenshot frequency'
        ],
        action: 'clearStorage()',
        actionLabel: 'Clear Data'
      }
    }

    // Default resolution
    return {
      title: 'Operation Failed',
      description: 'An error occurred while processing your request.',
      steps: [
        'Please try again',
        'If the problem persists, restart the extension',
        'Contact support if needed'
      ],
      action: 'location.reload()',
      actionLabel: 'Reload'
    }
  }
}
```

## Error Monitoring and Logging

### Centralized Error Logging
```typescript
class ErrorLogger {
  private errorBuffer: ErrorLog[] = []
  private maxBufferSize = 100

  async logError(error: Error, context: string, details: any = {}): Promise<void> {
    const errorLog: ErrorLog = {
      id: this.generateErrorId(),
      timestamp: Date.now(),
      message: error.message,
      stack: error.stack,
      context,
      details,
      severity: this.getErrorSeverity(error)
    }

    this.errorBuffer.push(errorLog)

    // Keep buffer size manageable
    if (this.errorBuffer.length > this.maxBufferSize) {
      this.errorBuffer = this.errorBuffer.slice(-this.maxBufferSize)
    }

    // Send to server for monitoring
    await this.sendToServer(errorLog)

    // Console logging
    this.logToConsole(errorLog)
  }

  private generateErrorId(): string {
    return `error_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`
  }

  private getErrorSeverity(error: Error): 'low' | 'medium' | 'high' | 'critical' {
    if (error.message.includes('critical')) return 'critical'
    if (error.message.includes('quota') || error.message.includes('permission')) return 'high'
    if (error.message.includes('timeout') || error.message.includes('network')) return 'medium'
    return 'low'
  }

  private async sendToServer(errorLog: ErrorLog): Promise<void> {
    try {
      // Send error to monitoring server
    } catch (error) {
      console.warn('Failed to send error to server:', error)
    }
  }

  private logToConsole(errorLog: ErrorLog): void {
    console.group(`Error [${errorLog.severity}] ${errorLog.context}`)
    console.error(errorLog.message)
    console.error('Stack:', errorLog.stack)
    console.error('Details:', errorLog.details)
    console.groupEnd()
  }
}
```

### Error Analytics
```typescript
class ErrorAnalytics {
  private errorCounts: Map<string, number> = new Map()
  private errorTrends: ErrorTrend[] = []

  async trackError(error: Error, context: string): Promise<void> {
    const errorKey = `${context}:${error.message}`

    // Increment error count
    const currentCount = this.errorCounts.get(errorKey) || 0
    this.errorCounts.set(errorKey, currentCount + 1)

    // Track trend
    this.errorTrends.push({
      timestamp: Date.now(),
      context,
      errorType: error.constructor.name,
      message: error.message
    })

    // Keep only recent trends
    if (this.errorTrends.length > 1000) {
      this.errorTrends = this.errorTrends.slice(-1000)
    }

    // Check for error spikes
    await this.checkErrorSpikes()
  }

  private async checkErrorSpikes(): Promise<void> {
    // Analyze error trends for spikes
    const recentErrors = this.errorTrends.filter(
      trend => Date.now() - trend.timestamp < 60000 // Last minute
    )

    if (recentErrors.length > 10) {
      // Error spike detected
      await this.notifyErrorSpike(recentErrors)
    }
  }

  private async notifyErrorSpike(errors: ErrorTrend[]): Promise<void> {
    console.warn('Error spike detected:', errors.length, 'errors in last minute')

    // Send notification for monitoring
  }
}
```

This comprehensive error handling strategy ensures that all components of the system can gracefully handle errors, provide meaningful feedback to users, and maintain system stability even when unexpected issues occur.