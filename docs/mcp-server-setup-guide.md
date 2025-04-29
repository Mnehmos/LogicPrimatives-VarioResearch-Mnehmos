# Technical Implementation Guide: Setting Up MCP Servers for Research Agents

This guide provides practical instructions for implementing the MCP servers that power the Logic Primitives research framework. It serves as a technical companion to the conceptual guide in `research-agents-guide.md`.

## Prerequisites

- Node.js 18.0.0 or later
- A Gemini API key (for LLM-dependent primitives)
- VS Code with Roo Code or similar extension

## Setting Up the Core MCP Servers

### 1. Logic MCP Primitives Server

The `logic-mcp-primitives` server is the cornerstone of the research framework, providing the core cognitive operations.

#### Installation

1. Clone or create the project directory:
   ```bash
   mkdir -p logic-mcp-primitives
   cd logic-mcp-primitives
   ```

2. Initialize the project:
   ```bash
   npm init -y
   ```

3. Install dependencies:
   ```bash
   npm install @modelcontextprotocol/sdk @google/generative-ai sqlite sqlite3 zod axios dotenv uuid
   npm install --save-dev typescript ts-node nodemon @types/node jest ts-jest @types/jest eslint prettier
   ```

4. Create TypeScript configuration:
   ```bash
   npx tsc --init
   ```

5. Update `tsconfig.json` with appropriate settings:
   ```json
   {
     "compilerOptions": {
       "target": "ES2020",
       "module": "commonjs",
       "outDir": "./build",
       "rootDir": "./src",
       "strict": true,
       "esModuleInterop": true,
       "skipLibCheck": true,
       "forceConsistentCasingInFileNames": true
     }
   }
   ```

#### Project Structure

Create the following directory structure:

```
logic-mcp-primitives/
├── src/
│   ├── server.ts              # Main server entry point
│   ├── handlers/              # Primitive handlers
│   │   └── index.ts           # Exports all handler functions
│   ├── llm/
│   │   └── llm_service.ts     # LLM interaction service
│   ├── state/
│   │   └── db_manager.ts      # SQLite state management
│   ├── tools/
│   │   └── index.ts           # Tool definitions and schemas
│   ├── utils/
│   │   ├── id_generator.ts    # Unique ID generation
│   │   ├── logging.ts         # Logging utilities
│   │   └── cache_manager.ts   # Cache management
│   ├── validation/
│   │   └── validator.ts       # Input validation utilities
│   └── __tests__/            # Test files
├── build/                    # Compiled output
├── context_history.db        # SQLite database (created at runtime)
└── .env                     # Environment variables
```

#### Core Server Implementation

1. Create `src/server.ts`:

```typescript
import { McpServerBuilder, onToolCall } from '@modelcontextprotocol/sdk';
import { tools } from './tools';
import { initializeDatabase } from './state/db_manager';
import { initializeLLMService } from './llm/llm_service';
import { handleObserve, handleDefine, handleDistinguish, handleSequence, 
  handleCompare, handleInfer, handleReflect, handleAsk, handleSynthesize, 
  handleDecide, handleAdapt, handleRetrieveObservation, handleRetrieveArtifact,
  handleListContextHistory } from './handlers';
import { logInfo, logError } from './utils/logging';

async function startServer() {
  try {
    // Initialize database
    await initializeDatabase();
    
    // Initialize LLM service
    initializeLLMService();
    
    // Build MCP server
    const server = new McpServerBuilder()
      .withName('logic-mcp-primitives')
      .withDescription('Cognitive primitives for structured reasoning')
      .withTools(tools)
      .withToolCallHandler(
        onToolCall('observe', handleObserve)
          .onToolCall('define', handleDefine)
          .onToolCall('distinguish', handleDistinguish)
          .onToolCall('sequence', handleSequence)
          .onToolCall('compare', handleCompare)
          .onToolCall('infer', handleInfer)
          .onToolCall('reflect', handleReflect)
          .onToolCall('ask', handleAsk)
          .onToolCall('synthesize', handleSynthesize)
          .onToolCall('decide', handleDecide)
          .onToolCall('adapt', handleAdapt)
          .onToolCall('retrieve_observation', handleRetrieveObservation)
          .onToolCall('retrieve_artifact', handleRetrieveArtifact)
          .onToolCall('list_context_history', handleListContextHistory)
      )
      .build();
    
    // Start server
    server.listen();
    logInfo('Logic MCP Primitives server started and listening');
  } catch (error) {
    logError(`Failed to start server: ${error}`);
    process.exit(1);
  }
}

startServer();
```

2. Create handler implementations for each primitive in `src/handlers/index.ts`. Here's an example for the `observe` primitive:

```typescript
import { McpToolCallContext, formatSuccessResult, ErrorCode, McpError } from '@modelcontextprotocol/sdk';
import { validateInput } from '../validation/validator';
import { storeContextItem } from '../state/db_manager';
import { generateId } from '../utils/id_generator';
import { logInfo, logError } from '../utils/logging';
import { ObserveInputSchema } from '../tools';
import * as fs from 'fs/promises';
import axios from 'axios';
import * as path from 'path';
import * as url from 'url';

export async function handleObserve(ctx: McpToolCallContext) {
  try {
    // Validate input
    const args = validateInput(ObserveInputSchema, ctx.args);
    const { context_id, source, format = 'text' } = args;
    
    // Generate unique ID for this observation
    const observation_id = generateId('obs');
    
    let observed_data = '';
    const metadata: any = { source, format };
    
    // Process source based on type
    if (typeof source === 'string') {
      if (source.startsWith('file://')) {
        // Handle file source
        try {
          const filePath = source.replace('file://', '');
          
          // Security check - validate path is safe
          const normalizedPath = path.normalize(filePath);
          if (normalizedPath.includes('..')) {
            throw new Error('Path traversal attempt detected');
          }
          
          observed_data = await fs.readFile(filePath, 'utf8');
        } catch (err) {
          metadata.error = `Failed to read file: ${err.message}`;
          observed_data = source; // Fallback to treating source as raw text
        }
      } else if (source.startsWith('http://') || source.startsWith('https://')) {
        // Handle URL source
        try {
          const response = await axios.get(source);
          observed_data = typeof response.data === 'string' 
            ? response.data 
            : JSON.stringify(response.data);
        } catch (err) {
          metadata.error = `Failed to fetch URL: ${err.message}`;
          observed_data = source; // Fallback to treating source as raw text
        }
      } else {
        // Treat as raw text
        observed_data = source;
      }
    } else {
      // Handle non-string source (convert to string)
      observed_data = JSON.stringify(source);
    }
    
    // Store observation in database
    await storeContextItem({
      artifact_id: observation_id,
      context_id,
      primitive_used: 'observe',
      input_args: args,
      output_result: {
        observation_id,
        observed_data,
        metadata
      },
      timestamp: new Date().toISOString()
    });
    
    // Return success
    return formatSuccessResult({
      observation_id,
      metadata
    });
  } catch (error) {
    logError(`observe handler error: ${error.message}`);
    if (error instanceof McpError) {
      throw error;
    }
    throw new McpError(ErrorCode.InternalError, `Failed to process observation: ${error.message}`);
  }
}
```

### 2. Database Manager Implementation

Create `src/state/db_manager.ts` for state persistence:

```typescript
import sqlite3 from 'sqlite3';
import { open, Database } from 'sqlite';
import { logInfo, logError } from '../utils/logging';

// Define database path
const DB_PATH = './context_history.db';

// Define context record interface
export interface ContextRecord {
  artifact_id: string;
  context_id: string;
  primitive_used: string;
  input_args: any;
  output_result: any;
  timestamp: string;
}

// Global database connection
let db: Database;

// Initialize database
export async function initializeDatabase(): Promise<void> {
  try {
    // Open database connection
    db = await open({
      filename: DB_PATH,
      driver: sqlite3.Database
    });
    
    // Create tables if they don't exist
    await db.exec(`
      CREATE TABLE IF NOT EXISTS context_history (
        artifact_id TEXT PRIMARY KEY,
        context_id TEXT NOT NULL,
        primitive_used TEXT NOT NULL,
        input_args TEXT NOT NULL,
        output_result TEXT NOT NULL,
        timestamp TEXT NOT NULL
      );
      
      CREATE INDEX IF NOT EXISTS idx_context_id ON context_history(context_id);
      CREATE INDEX IF NOT EXISTS idx_primitive_used ON context_history(primitive_used);
    `);
    
    logInfo('Database initialized successfully');
  } catch (error) {
    logError(`Database initialization failed: ${error.message}`);
    throw error;
  }
}

// Store a context item
export async function storeContextItem(record: ContextRecord): Promise<void> {
  try {
    await db.run(
      `INSERT INTO context_history
       (artifact_id, context_id, primitive_used, input_args, output_result, timestamp)
       VALUES (?, ?, ?, ?, ?, ?)`,
      record.artifact_id,
      record.context_id,
      record.primitive_used,
      JSON.stringify(record.input_args),
      JSON.stringify(record.output_result),
      record.timestamp
    );
  } catch (error) {
    logError(`Failed to store context item: ${error.message}`);
    throw error;
  }
}

// Get a context item by ID
export async function getContextItemById(artifactId: string): Promise<ContextRecord | null> {
  try {
    const row = await db.get(
      `SELECT * FROM context_history WHERE artifact_id = ?`,
      artifactId
    );
    
    if (!row) return null;
    
    return {
      ...row,
      input_args: JSON.parse(row.input_args),
      output_result: JSON.parse(row.output_result)
    };
  } catch (error) {
    logError(`Failed to get context item: ${error.message}`);
    throw error;
  }
}

// Get multiple context items by IDs
export async function getContextItemsByIds(artifactIds: string[]): Promise<ContextRecord[]> {
  try {
    if (!artifactIds.length) return [];
    
    const placeholders = artifactIds.map(() => '?').join(',');
    const rows = await db.all(
      `SELECT * FROM context_history WHERE artifact_id IN (${placeholders})`,
      ...artifactIds
    );
    
    return rows.map(row => ({
      ...row,
      input_args: JSON.parse(row.input_args),
      output_result: JSON.parse(row.output_result)
    }));
  } catch (error) {
    logError(`Failed to get context items: ${error.message}`);
    throw error;
  }
}

// Get recent context items by context ID
export async function getRecentContextItems(contextId: string, limit: number = 10): Promise<ContextRecord[]> {
  try {
    const rows = await db.all(
      `SELECT * FROM context_history 
       WHERE context_id = ? 
       ORDER BY timestamp DESC 
       LIMIT ?`,
      contextId,
      limit
    );
    
    return rows.map(row => ({
      ...row,
      input_args: JSON.parse(row.input_args),
      output_result: JSON.parse(row.output_result)
    }));
  } catch (error) {
    logError(`Failed to get recent context items: ${error.message}`);
    throw error;
  }
}
```

### 3. Integrating with Gemini API

Create `src/llm/llm_service.ts` for LLM integration:

```typescript
import { logInfo, logError } from '../utils/logging';
import { GoogleGenerativeAI, GenerativeModel, Part, EnhancedPromptParams } from '@google/generative-ai';

// Define the result interface
export interface LLMResult {
  content: string | null;
  error?: string;
}

// Global LLM service variables
let genAI: GoogleGenerativeAI;
const modelConfigs: Record<string, any> = {
  default: {
    temperature: 0.7,
    topP: 0.9,
    topK: 32,
    maxOutputTokens: 4096
  },
  infer: {
    temperature: 0.3,
    topP: 0.8,
    topK: 40,
    maxOutputTokens: 2048
  },
  reflect: {
    temperature: 0.5,
    topP: 0.9,
    topK: 32,
    maxOutputTokens: 4096
  },
  synthesize: {
    temperature: 0.7,
    topP: 0.9,
    topK: 32,
    maxOutputTokens: 8192
  }
};

// Initialize the LLM service
export function initializeLLMService(): void {
  try {
    const apiKey = process.env.LOGIC_MCP_GEMINI_API_KEY;
    
    if (!apiKey) {
      logError('Gemini API key not found. Set LOGIC_MCP_GEMINI_API_KEY environment variable.');
      throw new Error('API key missing');
    }
    
    genAI = new GoogleGenerativeAI(apiKey);
    logInfo('LLM service initialized successfully');
  } catch (error) {
    logError(`LLM service initialization failed: ${error.message}`);
    throw error;
  }
}

// Call the Gemini API
export async function callGeminiChatCompletion(
  messages: Array<{ role: string; content: string | Part[] }>,
  modelName: string = 'gemini-pro',
  primitiveType: string = 'default'
): Promise<LLMResult> {
  try {
    // Ensure service is initialized
    if (!genAI) {
      throw new Error('LLM service not initialized');
    }
    
    // Override model from environment variable if available
    const envModelOverride = process.env[`LOGIC_MCP_${primitiveType.toUpperCase()}_MODEL`];
    const finalModelName = envModelOverride || modelName;
    
    // Get model configuration
    const config = modelConfigs[primitiveType] || modelConfigs.default;
    
    // Get model
    const model = genAI.getGenerativeModel({ model: finalModelName });
    
    // Start chat
    const chat = model.startChat({
      history: messages.slice(0, -1).map(msg => ({
        role: msg.role === 'user' ? 'user' : 'model',
        parts: typeof msg.content === 'string' ? [{ text: msg.content }] : msg.content
      })),
      generationConfig: config
    });
    
    // Send message
    const lastMessage = messages[messages.length - 1];
    const content = typeof lastMessage.content === 'string' 
      ? lastMessage.content 
      : lastMessage.content;
    
    const result = await chat.sendMessage(content);
    const response = await result.response;
    const text = response.text();
    
    if (!text) {
      return { content: null, error: 'Empty response from LLM' };
    }
    
    return { content: text };
  } catch (error) {
    logError(`LLM call failed: ${error.message}`);
    return { content: null, error: `LLM call failed: ${error.message}` };
  }
}
```

### 4. Running and Maintenance Scripts

Add these to your `package.json`:

```json
{
  "scripts": {
    "build": "tsc",
    "start": "node build/server.js",
    "dev": "nodemon --watch 'src/**/*.ts' --exec 'ts-node' src/server.ts",
    "test": "jest",
    "lint": "eslint 'src/**/*.ts'",
    "format": "prettier --write 'src/**/*.ts'"
  }
}
```

### 5. Environment Configuration

Create a `.env` file for API keys:

```
LOGIC_MCP_GEMINI_API_KEY=your_gemini_api_key_here
LOGIC_MCP_INFER_MODEL=gemini-pro
LOGIC_MCP_REFLECT_MODEL=gemini-pro
LOGIC_MCP_SYNTHESIZE_MODEL=gemini-pro
```

## Connecting to Roo Code

To use your MCP server with Roo Code or similar extensions:

1. Create or edit the MCP settings file:
   ```json
   {
     "stdio_servers": [
       {
         "name": "logic-mcp-primitives",
         "command": "node",
         "args": ["/absolute/path/to/logic-mcp-primitives/build/server.js"],
         "env": {
           "LOGIC_MCP_GEMINI_API_KEY": "your_gemini_api_key_here"
         }
       }
     ]
   }
   ```

2. Locate this file in the correct configuration directory:
   - For most extensions: `~/.mcp/mcp_settings.json`
   - For some: Check specific documentation for the configuration location

3. Restart the extension or editor to pick up the new MCP server configuration

## Additional MCP Servers

### Brave Search Server

For web search capabilities:

```typescript
// Abbreviated implementation
import { McpServerBuilder, onToolCall } from '@modelcontextprotocol/sdk';
import axios from 'axios';

async function handleBraveWebSearch(ctx) {
  const { query, count = 10, offset = 0 } = ctx.args;
  
  const response = await axios.get('https://api.search.brave.com/res/v1/web/search', {
    params: { q: query, count, offset },
    headers: { 'X-Subscription-Token': process.env.BRAVE_API_KEY }
  });
  
  return formatSuccessResult(response.data);
}

const server = new McpServerBuilder()
  .withName('brave-search')
  .withDescription('Brave Search API')
  .withTools(braveSearchTools)
  .withToolCallHandler(
    onToolCall('brave_web_search', handleBraveWebSearch)
  )
  .build();

server.listen();
```

## Testing Your Implementation

1. Create basic tests in the `__tests__` directory:

```typescript
// src/__tests__/observe.test.ts
import { handleObserve } from '../handlers';
import { initializeDatabase } from '../state/db_manager';

beforeAll(async () => {
  // Set up test database
  process.env.DB_PATH = ':memory:';
  await initializeDatabase();
});

test('handleObserve processes raw text correctly', async () => {
  const mockContext = {
    args: {
      context_id: 'test-context-123',
      source: 'This is raw test data'
    }
  };
  
  const result = await handleObserve(mockContext as any);
  
  expect(result).toHaveProperty('observation_id');
  expect(result).toHaveProperty('metadata');
  expect(result.metadata.format).toBe('text');
});
```

2. Run tests:
```bash
npm test
```

## Deploying to Production

For production deployment:

1. Set up proper environment variable management
2. Consider containerization with Docker
3. Implement proper logging with rotation
4. Add monitoring for server health

## Troubleshooting

Common issues and solutions:

| Issue | Solution |
|-------|----------|
| API key errors | Verify environment variables are properly set |
| Database errors | Check SQLite file permissions and path |
| Memory issues | Consider implementing pagination for large datasets |
| Connection timeouts | Add proper timeout handling and retries |

## Next Steps

Once your core implementation is working:

1. Implement additional primitive handlers
2. Add more robust error handling
3. Create logging and monitoring
4. Develop additional MCP servers for specialized data sources

By following this guide, you'll have a functional implementation of the Logic MCP Primitives server that powers the research framework described in the conceptual guide.