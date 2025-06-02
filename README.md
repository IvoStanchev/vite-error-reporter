# Vite Error Reporter

A comprehensive error reporting plugin for Vite that captures **all** build, development, and runtime errors and sends them to a remote server for monitoring and debugging.

## 🚀 Features

- **Comprehensive Error Capture**: Captures all types of Vite errors including build, syntax, import, CSS, TypeScript, and plugin errors
- **15+ Error Hooks**: Monitors every Vite lifecycle event and error handling mechanism
- **Intelligent Classification**: Automatically categorizes errors by type and severity
- **Configurable Error Types**: Enable/disable specific error types as needed
- **Rate Limiting**: Prevents error spam with configurable limits
- **Retry Logic**: Automatic retry with exponential backoff for network failures
- **Custom Handlers**: Add custom error processing and filtering
- **TypeScript Support**: Full TypeScript definitions included
- **Framework Agnostic**: Works with React, Vue, Svelte, or vanilla JavaScript
- **Zero Configuration**: Works out of the box with sensible defaults

## 📦 Installation

```bash
npm install vite-error-reporter
# or
yarn add vite-error-reporter
# or
pnpm add vite-error-reporter
```

## 🔧 Basic Usage

Add the plugin to your `vite.config.js` or `vite.config.ts`:

```javascript
import { defineConfig } from 'vite';
import { viteErrorReporter } from 'vite-error-reporter';

export default defineConfig({
  plugins: [
    // ... your other plugins
    viteErrorReporter({
      errorServerUrl: 'https://your-error-server.com/api/errors'
    })
  ]
});
```

## ⚙️ Configuration

### Basic Configuration

```javascript
viteErrorReporter({
  // Required: URL of your error reporting server
  errorServerUrl: 'https://your-error-server.com/api/errors',
  
  // Optional: Enable console logging (default: true)
  enableConsoleLogging: true,
  
  // Optional: Include stack traces (default: true)
  enableStackTraces: true,
  
  // Optional: Include code frames (default: true)
  enableCodeFrames: true
})
```

### Advanced Configuration

```javascript
viteErrorReporter({
  errorServerUrl: 'https://your-error-server.com/api/errors',
  
  // Error filtering
  enabledErrorTypes: ['vite-syntax-error', 'vite-import-error'], // Only these types
  disabledErrorTypes: ['vite-hmr-error'], // Exclude these types
  ignorePatterns: [/node_modules/, /\.test\./], // Ignore files matching patterns
  
  // Rate limiting
  maxErrorsPerMinute: 50,
  maxErrorsPerType: 10,
  
  // Environment settings
  enableInProduction: false,
  enableInDevelopment: true,
  
  // Custom handlers
  onError: async (error) => {
    console.log('Custom error handler:', error);
  },
  onErrorSent: (error) => {
    console.log('Error sent successfully:', error.type);
  },
  onErrorFailed: (error, reason) => {
    console.error('Failed to send error:', reason);
  },
  
  // Transform errors before sending
  transformError: (error) => {
    return {
      ...error,
      customField: 'custom value'
    };
  },
  
  // Filter errors before sending
  shouldReportError: (error) => {
    return error.severity !== 'low';
  }
})
```

## 🎯 Error Types Captured

The plugin captures **50+ different error types** across all Vite operations:

### Build & Compilation Errors
- `vite-build-error` - General build failures
- `vite-syntax-error` - JavaScript/TypeScript syntax errors
- `vite-jsx-error` - JSX syntax and structure errors
- `vite-typescript-error` - TypeScript type checking errors
- `vite-transform-error` - File transformation errors

### Module & Import Errors
- `vite-import-error` - Module import failures
- `vite-resolve-error` - Module resolution failures
- `vite-dependency-error` - Dependency loading errors
- `vite-load-error` - File loading errors

### Asset & Styling Errors
- `vite-css-error` - CSS preprocessing errors
- `vite-sass-error` - Sass/SCSS compilation errors
- `vite-asset-error` - Asset processing errors
- `vite-image-error` - Image optimization errors

### Server & Network Errors
- `vite-dev-server-error` - Development server errors
- `vite-hmr-error` - Hot Module Replacement errors
- `vite-websocket-error` - WebSocket connection errors
- `vite-network-error` - Network connectivity errors

### Plugin & Framework Errors
- `vite-plugin-error` - Plugin execution errors
- `vite-rollup-error` - Rollup bundling errors
- `vite-esbuild-error` - ESBuild transformation errors

[See full list of error types](./src/config.ts)

## 🔧 Enable/Disable Error Types

You can selectively enable or disable specific error types:

```javascript
viteErrorReporter({
  errorServerUrl: 'https://your-server.com/api/errors',
  
  // Only capture these error types
  enabledErrorTypes: [
    'vite-syntax-error',
    'vite-import-error',
    'vite-build-error'
  ],
  
  // OR exclude specific error types
  disabledErrorTypes: [
    'vite-hmr-error',        // Ignore HMR errors
    'vite-css-error'         // Ignore CSS errors
  ]
})
```

## 📊 Error Data Structure

Each error report includes comprehensive debugging information:

```typescript
interface ErrorReportData {
  type: string;              // Error type classification
  message: string;           // Error message
  plugin?: string;           // Which Vite plugin detected the error
  file?: string;             // File path where error occurred
  line?: number;             // Line number
  column?: number;           // Column number
  stack?: string;            // Stack trace
  frame?: string;            // Code frame showing error context
  severity?: 'low' | 'medium' | 'high' | 'critical';
  category?: 'syntax' | 'build' | 'runtime' | 'network' | 'asset';
  timestamp: string;         // ISO timestamp
  environment: string;       // 'development' | 'production'
  source: string;            // Which hook captured the error
}
```

## 🏗️ Error Server Setup

Create a simple Express server to receive error reports:

```javascript
const express = require('express');
const app = express();

app.use(express.json());

app.post('/api/errors', (req, res) => {
  const errorData = req.body;
  
  // Log to console
  console.error('🚨 Vite Error:', errorData);
  
  // Store in database, send to monitoring service, etc.
  // await saveToDatabase(errorData);
  // await sendToSlack(errorData);
  
  res.status(200).json({ received: true });
});

app.listen(3001, () => {
  console.log('Error server running on port 3001');
});
```

## 🎨 Integration Examples

### With Sentry

```javascript
import * as Sentry from '@sentry/node';

viteErrorReporter({
  errorServerUrl: 'http://localhost:3001/api/errors',
  onError: async (error) => {
    Sentry.captureException(new Error(error.message), {
      tags: {
        errorType: error.type,
        severity: error.severity
      },
      extra: error
    });
  }
})
```

### With Custom Logging

```javascript
viteErrorReporter({
  errorServerUrl: 'http://localhost:3001/api/errors',
  transformError: (error) => ({
    ...error,
    projectName: 'my-awesome-project',
    buildId: process.env.BUILD_ID,
    userId: getCurrentUserId()
  }),
  shouldReportError: (error) => {
    // Only report high severity errors in production
    if (process.env.NODE_ENV === 'production') {
      return error.severity === 'high' || error.severity === 'critical';
    }
    return true;
  }
})
```

## 🔍 Debugging

Enable detailed console logging to see what errors are being captured:

```javascript
viteErrorReporter({
  errorServerUrl: 'http://localhost:3001/api/errors',
  enableConsoleLogging: true,  // See all captured errors
  enableStackTraces: true,     // Include full stack traces
  enableCodeFrames: true       // Include code context
})
```

## 📈 Performance

The plugin is designed for minimal performance impact:

- **Lazy Error Processing**: Errors are processed asynchronously
- **Rate Limiting**: Prevents overwhelming your error server
- **Efficient Parsing**: Optimized error message parsing
- **Memory Management**: Automatic cleanup of error counters

## 🤝 Contributing

Contributions are welcome! Please read our [Contributing Guide](CONTRIBUTING.md) for details.

## 📄 License

MIT License - see [LICENSE](LICENSE) file for details.

## 🆘 Support

- [GitHub Issues](https://github.com/vite-error-reporter/vite-error-reporter/issues)
- [Documentation](https://github.com/vite-error-reporter/vite-error-reporter#readme)
- [Examples](https://github.com/vite-error-reporter/vite-error-reporter/tree/main/examples)

---

**Made with ❤️ for the Vite community**
