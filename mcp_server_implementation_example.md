# Implementing Your Own Research Tool: Custom MCP Server Example

This guide demonstrates how to create a simple MCP server for specialized research needs. This example creates a tool for analyzing academic paper citations and relationships - something useful for literature reviews.

## Prerequisites

- Basic Node.js familiarity
- Node.js (v18+) installed
- A research need that existing tools don't address

## Step 1: Setup Project Structure

```bash
mkdir citation-network-mcp
cd citation-network-mcp
npm init -y
```

Install dependencies:

```bash
npm install @modelcontextprotocol/sdk axios cheerio typescript ts-node sqlite3 sqlite dotenv
npm install --save-dev @types/node nodemon
```

Create TypeScript configuration:

```bash
npx tsc --init
```

## Step 2: Define Your Tool's Purpose

For our example, we're creating a tool that:
1. Takes an academic paper DOI or URL
2. Extracts its citations
3. Builds a citation network showing relationships
4. Returns structured data about the paper's place in the research landscape

## Step 3: Create Basic Directory Structure

```
citation-network-mcp/
├── src/
│   ├── server.ts              # Main server entry point
│   ├── tools/                 # Tool definitions
│   │   └── schema.ts          # Input/output schemas
│   ├── handlers/              # Tool handlers
│   │   └── citation.ts        # Citation analysis logic
│   ├── services/              # External services
│   │   ├── crossref.ts        # CrossRef API integration
│   │   └── semantic.ts        # Semantic Scholar integration
│   ├── database/              # Database management
│   │   └── db.ts              # SQLite storage
│   └── utils/                 # Utilities
│       └── logger.ts          # Logging functions
├── data/                      # Local data storage
└── .env                       # Environment variables
```

## Step 4: Define Tool Schemas

Create `src/tools/schema.ts`:

```typescript
import { z } from 'zod';

// Input schema for analyzing a paper
export const AnalyzePaperSchema = z.object({
  doi: z.string().optional(),
  url: z.string().optional(),
  depth: z.number().default(1),
  max_citations: z.number().default(20)
}).refine(data => data.doi || data.url, {
  message: "Either DOI or URL must be provided"
});

// Input schema for citation network
export const CitationNetworkSchema = z.object({
  seed_papers: z.array(z.string()),
  depth: z.number().default(2),
  min_citations: z.number().default(5),
  max_nodes: z.number().default(50)
});

// Input schema for research front detection
export const ResearchFrontSchema = z.object({
  topic: z.string(),
  start_year: z.number().optional(),
  end_year: z.number().optional(),
  min_papers: z.number().default(10)
});

// Tool definitions for MCP server
export const tools = [
  {
    name: 'analyze_paper',
    description: 'Analyze a scholarly paper and its citation relationships',
    inputSchema: AnalyzePaperSchema
  },
  {
    name: 'build_citation_network',
    description: 'Build a citation network from seed papers',
    inputSchema: CitationNetworkSchema
  },
  {
    name: 'detect_research_front',
    description: 'Identify emerging research fronts in a topic area',
    inputSchema: ResearchFrontSchema
  }
];
```

## Step 5: Create External Service Integration

Create `src/services/crossref.ts`:

```typescript
import axios from 'axios';
import { logError, logInfo } from '../utils/logger';

interface CrossRefWork {
  DOI: string;
  title: string[];
  author: any[];
  published: {
    'date-parts': number[][];
  };
  reference: any[];
  // Additional fields as needed
}

export async function getPaperByDoi(doi: string): Promise<any> {
  try {
    const response = await axios.get(`https://api.crossref.org/works/${doi}`);
    return response.data.message;
  } catch (error) {
    logError(`CrossRef API error: ${error.message}`);
    throw new Error(`Failed to retrieve paper with DOI ${doi}`);
  }
}

export async function getPaperCitations(doi: string, limit: number = 20): Promise<any[]> {
  try {
    const response = await axios.get(
      `https://api.crossref.org/works/${doi}/citations`,
      { params: { rows: limit } }
    );
    
    return response.data.message.items || [];
  } catch (error) {
    logError(`CrossRef citations API error: ${error.message}`);
    return [];
  }
}
```

## Step 6: Create Handler Implementation

Create `src/handlers/citation.ts`:

```typescript
import { McpToolCallContext, formatSuccessResult, ErrorCode, McpError } from '@modelcontextprotocol/sdk';
import { getPaperByDoi, getPaperCitations } from '../services/crossref';
import { storeAnalysisResult, getPreviousAnalysis } from '../database/db';
import { logInfo, logError } from '../utils/logger';

export async function handleAnalyzePaper(ctx: McpToolCallContext) {
  try {
    const { doi, url, depth, max_citations } = ctx.args;
    let paperDoi = doi;
    
    // If only URL is provided, try to extract DOI from URL
    if (!paperDoi && url) {
      paperDoi = extractDoiFromUrl(url);
      if (!paperDoi) {
        throw new McpError(
          ErrorCode.InvalidArgument,
          'Could not extract DOI from URL'
        );
      }
    }
    
    // Check if we have a cached result
    const cachedResult = await getPreviousAnalysis(paperDoi);
    if (cachedResult) {
      return formatSuccessResult(cachedResult);
    }
    
    // Fetch paper metadata
    const paperData = await getPaperByDoi(paperDoi);
    
    // Fetch citations up to requested depth
    const citations = await fetchCitationsRecursive(paperDoi, depth, max_citations);
    
    // Build citation graph
    const citationGraph = buildCitationGraph(paperData, citations);
    
    // Analyze paper importance metrics
    const metrics = analyzeMetrics(paperData, citations);
    
    // Create result
    const result = {
      paper_info: {
        doi: paperDoi,
        title: paperData.title?.join(' '),
        authors: formatAuthors(paperData.author || []),
        year: paperData.published?.['date-parts']?.[0]?.[0] || null
      },
      citation_count: citations.length,
      citation_graph: citationGraph,
      metrics: metrics,
      analysis_timestamp: new Date().toISOString()
    };
    
    // Store result for future use
    await storeAnalysisResult(paperDoi, result);
    
    return formatSuccessResult(result);
  } catch (error) {
    logError(`analyze_paper handler error: ${error.message}`);
    if (error instanceof McpError) {
      throw error;
    }
    throw new McpError(
      ErrorCode.InternalError,
      `Analysis failed: ${error.message}`
    );
  }
}

// Helper functions
function extractDoiFromUrl(url: string): string | null {
  // Implementation to extract DOI from URL
  const doiMatch = url.match(/10\.\d{4,}\/[^\s]*/);
  return doiMatch ? doiMatch[0] : null;
}

async function fetchCitationsRecursive(doi: string, depth: number, limit: number): Promise<any[]> {
  // Implementation of recursive citation fetching
  // This is simplified - you'd need proper implementation
  if (depth <= 0) return [];
  
  const directCitations = await getPaperCitations(doi, limit);
  
  if (depth === 1) return directCitations;
  
  // For depth > 1, get citations of citations
  // In a real implementation, you'd need to handle rate limits
  // and avoid duplicate work
  
  return directCitations;
}

function buildCitationGraph(paper: any, citations: any[]): any {
  // Implementation to create a graph representation
  return {
    nodes: [
      { id: paper.DOI, type: 'seed', year: paper.published?.['date-parts']?.[0]?.[0] },
      ...citations.map(c => ({ 
        id: c.DOI, 
        type: 'citation', 
        year: c.published?.['date-parts']?.[0]?.[0]
      }))
    ],
    links: citations.map(c => ({
      source: c.DOI,
      target: paper.DOI
    }))
  };
}

function analyzeMetrics(paper: any, citations: any[]): any {
  // Implementation to calculate metrics like citation velocity, etc.
  // This would be more sophisticated in a real implementation
  return {
    h_index_estimate: 0, // Would calculate properly
    citation_velocity: 0, // Would calculate properly
    research_front_score: 0 // Would calculate properly
  };
}

function formatAuthors(authors: any[]): string[] {
  return authors.map(author => 
    `${author.family || ''}, ${author.given || ''}`.trim()
  );
}

// Add other handler implementations (build_citation_network, detect_research_front)
```

## Step 7: Create Database Management

Create `src/database/db.ts`:

```typescript
import sqlite3 from 'sqlite3';
import { open, Database } from 'sqlite';
import { logInfo, logError } from '../utils/logger';

// Define database path
const DB_PATH = './data/citation_analysis.db';

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
      CREATE TABLE IF NOT EXISTS paper_analysis (
        doi TEXT PRIMARY KEY,
        result TEXT NOT NULL,
        timestamp TEXT NOT NULL
      );
      
      CREATE TABLE IF NOT EXISTS citation_networks (
        network_id TEXT PRIMARY KEY,
        seed_papers TEXT NOT NULL,
        result TEXT NOT NULL,
        timestamp TEXT NOT NULL
      );
    `);
    
    logInfo('Citation database initialized successfully');
  } catch (error) {
    logError(`Database initialization failed: ${error.message}`);
    throw error;
  }
}

// Store a paper analysis result
export async function storeAnalysisResult(doi: string, result: any): Promise<void> {
  try {
    await db.run(
      `INSERT OR REPLACE INTO paper_analysis
       (doi, result, timestamp)
       VALUES (?, ?, ?)`,
      doi,
      JSON.stringify(result),
      new Date().toISOString()
    );
  } catch (error) {
    logError(`Failed to store analysis: ${error.message}`);
    throw error;
  }
}

// Get a previous paper analysis
export async function getPreviousAnalysis(doi: string): Promise<any | null> {
  try {
    const row = await db.get(
      `SELECT * FROM paper_analysis WHERE doi = ?`,
      doi
    );
    
    if (!row) return null;
    
    // Parse the result JSON
    const result = JSON.parse(row.result);
    
    // Check if the result is still fresh (less than 1 week old)
    const timestamp = new Date(row.timestamp);
    const oneWeekAgo = new Date();
    oneWeekAgo.setDate(oneWeekAgo.getDate() - 7);
    
    if (timestamp < oneWeekAgo) {
      // Result is too old, return null to force a refresh
      return null;
    }
    
    return result;
  } catch (error) {
    logError(`Failed to get previous analysis: ${error.message}`);
    return null;
  }
}

// Add other database functions for citation networks and research fronts
```

## Step 8: Create Main Server File

Create `src/server.ts`:

```typescript
import { McpServerBuilder, onToolCall } from '@modelcontextprotocol/sdk';
import { tools } from './tools/schema';
import { initializeDatabase } from './database/db';
import { handleAnalyzePaper } from './handlers/citation';
import { logInfo, logError } from './utils/logger';
import * as dotenv from 'dotenv';

// Load environment variables
dotenv.config();

async function startServer() {
  try {
    // Initialize database
    await initializeDatabase();
    
    // Build MCP server
    const server = new McpServerBuilder()
      .withName('citation-network-mcp')
      .withDescription('Citation network analysis for academic research')
      .withTools(tools)
      .withToolCallHandler(
        onToolCall('analyze_paper', handleAnalyzePaper)
        // Add handlers for other tools:
        // .onToolCall('build_citation_network', handleBuildCitationNetwork)
        // .onToolCall('detect_research_front', handleDetectResearchFront)
      )
      .build();
    
    // Start server
    server.listen();
    logInfo('Citation Network MCP server started and listening');
  } catch (error) {
    logError(`Failed to start server: ${error.message}`);
    process.exit(1);
  }
}

startServer();
```

## Step 9: Add Script to Package.json

Update your `package.json` to include:

```json
{
  "scripts": {
    "build": "tsc",
    "start": "node build/server.js",
    "dev": "nodemon --watch 'src/**/*.ts' --exec 'ts-node' src/server.ts"
  }
}
```

## Step 10: Running Your MCP Server

1. Start development server:
```bash
npm run dev
```

2. To connect to Claude or other AI tools, register your MCP server:
```json
{
  "stdio_servers": [
    {
      "name": "citation-network-mcp",
      "command": "node",
      "args": ["/absolute/path/to/citation-network-mcp/build/server.js"]
    }
  ]
}
```

## Example Usage in Research

Once configured, you can use your custom MCP server like this:

```python
# Example of using the tool in a Python-based workflow
response = use_mcp_tool(
    server_name="citation-network-mcp",
    tool_name="analyze_paper",
    arguments={
        "doi": "10.1038/s41586-021-03819-2",
        "depth": 2,
        "max_citations": 30
    }
)

# Process and visualize the citation network
import networkx as nx
import matplotlib.pyplot as plt

# Build network from response
G = nx.DiGraph()
for node in response["citation_graph"]["nodes"]:
    G.add_node(node["id"], year=node["year"], type=node["type"])
    
for link in response["citation_graph"]["links"]:
    G.add_edge(link["source"], link["target"])
    
# Visualize
plt.figure(figsize=(12, 8))
pos = nx.spring_layout(G)
nx.draw(G, pos, with_labels=True)
plt.title(f"Citation Network: {response['paper_info']['title']}")
plt.show()
```

## Extending Your Server

To make your server even more useful for research:

1. **Add a visualization tool** that generates citation maps as images
2. **Implement field detection** to categorize papers into research areas
3. **Create a co-author network analysis** to identify research communities
4. **Add temporal analysis** to track how citations evolve over time

## Conclusion

This example demonstrates how researchers can create custom MCP servers for specialized research needs. The citation network analyzer provides a concrete example of extending AI capabilities with domain-specific tools.

By following this pattern, you can create tools for your specific research domain that go beyond what general-purpose AI tools offer.