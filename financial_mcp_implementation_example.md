# Implementing Your Own Financial Analysis Tool: Custom MCP Server Example

This guide demonstrates how to create a simple MCP server for specialized financial analysis needs. This example creates a tool for analyzing financial statements, performing ratio analysis, and generating forecasts - essential capabilities for FP&A professionals.

## Prerequisites

- Basic Node.js familiarity
- Node.js (v18+) installed
- Financial data needs that existing tools don't address well

## Step 1: Setup Project Structure

```bash
mkdir financial-analysis-mcp
cd financial-analysis-mcp
npm init -y
```

Install dependencies:

```bash
npm install @modelcontextprotocol/sdk axios typescript ts-node sqlite3 sqlite dotenv moment regression
npm install --save-dev @types/node nodemon
```

Create TypeScript configuration:

```bash
npx tsc --init
```

## Step 2: Define Your Tool's Purpose

For our example, we're creating a tool that:
1. Takes financial statement data (income statement, balance sheet, cash flow)
2. Calculates key financial ratios and trends
3. Performs variance analysis against historical data or budgets
4. Generates forecasts based on historical patterns
5. Returns structured data with insights about financial performance

## Step 3: Create Basic Directory Structure

```
financial-analysis-mcp/
├── src/
│   ├── server.ts              # Main server entry point
│   ├── tools/                 # Tool definitions
│   │   └── schema.ts          # Input/output schemas
│   ├── handlers/              # Tool handlers
│   │   ├── statements.ts      # Financial statement analysis
│   │   ├── ratios.ts          # Financial ratio calculations
│   │   └── forecasting.ts     # Financial forecasting logic
│   ├── services/              # External services
│   │   ├── alphavantage.ts    # AlphaVantage API integration
│   │   └── edgar.ts           # SEC EDGAR API integration
│   ├── database/              # Database management
│   │   └── db.ts              # SQLite storage
│   ├── models/                # Financial models
│   │   ├── ratio_models.ts    # Financial ratio definitions
│   │   └── forecast_models.ts # Forecasting algorithms
│   └── utils/                 # Utilities
│       ├── logger.ts          # Logging functions
│       └── calculators.ts     # Financial calculation helpers
├── data/                      # Local data storage
└── .env                       # Environment variables
```

## Step 4: Define Tool Schemas

Create `src/tools/schema.ts`:

```typescript
import { z } from 'zod';

// Input schema for analyzing financial statements
export const AnalyzeStatementsSchema = z.object({
  company_id: z.string().optional(),
  ticker: z.string().optional(),
  statements: z.object({
    income_statement: z.any().optional(),
    balance_sheet: z.any().optional(),
    cash_flow: z.any().optional()
  }).optional(),
  years: z.number().default(3),
  quarters: z.boolean().default(false),
  ratios: z.array(z.string()).optional(),
  comparison_benchmark: z.string().optional()
}).refine(data => data.company_id || data.ticker || data.statements, {
  message: "Either company_id, ticker, or statements must be provided"
});

// Input schema for financial forecasting
export const ForecastFinancialsSchema = z.object({
  company_id: z.string().optional(),
  ticker: z.string().optional(),
  statements: z.any().optional(),
  forecast_periods: z.number().default(4),
  period_type: z.enum(['quarter', 'year']).default('quarter'),
  method: z.enum(['linear', 'seasonal', 'growth_rate', 'monte_carlo']).default('seasonal'),
  variables: z.array(z.string()).optional(),
  scenarios: z.array(z.enum(['base', 'optimistic', 'pessimistic'])).optional(),
  confidence_interval: z.number().optional()
});

// Input schema for variance analysis
export const VarianceAnalysisSchema = z.object({
  actual_data: z.any(),
  comparison_data: z.any(), // Could be budget, forecast, or prior period
  comparison_type: z.enum(['budget', 'forecast', 'prior_period', 'industry_benchmark']).default('budget'),
  threshold_percent: z.number().default(5),
  detailed_factors: z.boolean().default(false)
});

// Input schema for financial risk analysis
export const RiskAnalysisSchema = z.object({
  financial_data: z.any(),
  risk_factors: z.array(z.string()).optional(),
  simulation_runs: z.number().default(1000),
  confidence_level: z.number().default(95),
  stress_test_scenarios: z.array(z.any()).optional()
});

// Tool definitions for MCP server
export const tools = [
  {
    name: 'analyze_statements',
    description: 'Analyze financial statements, calculate ratios and compare to benchmarks',
    inputSchema: AnalyzeStatementsSchema
  },
  {
    name: 'forecast_financials',
    description: 'Generate financial forecasts based on historical data',
    inputSchema: ForecastFinancialsSchema
  },
  {
    name: 'variance_analysis',
    description: 'Perform variance analysis between actual and budgeted/forecasted figures',
    inputSchema: VarianceAnalysisSchema
  },
  {
    name: 'risk_analysis',
    description: 'Analyze financial risk with Monte Carlo simulations and stress testing',
    inputSchema: RiskAnalysisSchema
  }
];
```

## Step 5: Create External Service Integration

Create `src/services/alphavantage.ts`:

```typescript
import axios from 'axios';
import { logError, logInfo } from '../utils/logger';

// API Key should be in .env file
const API_KEY = process.env.ALPHA_VANTAGE_API_KEY;

export async function getIncomeStatement(ticker: string, period: 'annual' | 'quarterly' = 'annual'): Promise<any> {
  try {
    const response = await axios.get(
      `https://www.alphavantage.co/query?function=INCOME_STATEMENT&symbol=${ticker}&apikey=${API_KEY}`
    );
    
    if (response.data.hasOwnProperty('Error Message')) {
      throw new Error(response.data['Error Message']);
    }
    
    return period === 'annual' 
      ? response.data.annualReports 
      : response.data.quarterlyReports;
  } catch (error) {
    logError(`AlphaVantage API error: ${error.message}`);
    throw new Error(`Failed to retrieve income statement for ${ticker}`);
  }
}

export async function getBalanceSheet(ticker: string, period: 'annual' | 'quarterly' = 'annual'): Promise<any> {
  try {
    const response = await axios.get(
      `https://www.alphavantage.co/query?function=BALANCE_SHEET&symbol=${ticker}&apikey=${API_KEY}`
    );
    
    if (response.data.hasOwnProperty('Error Message')) {
      throw new Error(response.data['Error Message']);
    }
    
    return period === 'annual' 
      ? response.data.annualReports 
      : response.data.quarterlyReports;
  } catch (error) {
    logError(`AlphaVantage API error: ${error.message}`);
    throw new Error(`Failed to retrieve balance sheet for ${ticker}`);
  }
}

export async function getCashFlow(ticker: string, period: 'annual' | 'quarterly' = 'annual'): Promise<any> {
  try {
    const response = await axios.get(
      `https://www.alphavantage.co/query?function=CASH_FLOW&symbol=${ticker}&apikey=${API_KEY}`
    );
    
    if (response.data.hasOwnProperty('Error Message')) {
      throw new Error(response.data['Error Message']);
    }
    
    return period === 'annual' 
      ? response.data.annualReports 
      : response.data.quarterlyReports;
  } catch (error) {
    logError(`AlphaVantage API error: ${error.message}`);
    throw new Error(`Failed to retrieve cash flow statement for ${ticker}`);
  }
}

export async function getOverview(ticker: string): Promise<any> {
  try {
    const response = await axios.get(
      `https://www.alphavantage.co/query?function=OVERVIEW&symbol=${ticker}&apikey=${API_KEY}`
    );
    
    if (response.data.hasOwnProperty('Error Message')) {
      throw new Error(response.data['Error Message']);
    }
    
    return response.data;
  } catch (error) {
    logError(`AlphaVantage API error: ${error.message}`);
    throw new Error(`Failed to retrieve company overview for ${ticker}`);
  }
}

export async function getIndustryAverages(industry: string): Promise<any> {
  // In a real implementation, this would fetch industry average data
  // This is a placeholder as Alpha Vantage doesn't directly provide this
  try {
    // Placeholder for industry data
    return {
      // Sample industry averages
      profit_margin: 0.15,
      current_ratio: 1.8,
      debt_to_equity: 1.2,
      return_on_assets: 0.08,
      return_on_equity: 0.14
    };
  } catch (error) {
    logError(`Industry data error: ${error.message}`);
    throw new Error(`Failed to retrieve industry averages for ${industry}`);
  }
}
```

## Step 6: Create Ratio Calculation Models

Create `src/models/ratio_models.ts`:

```typescript
export interface FinancialRatio {
  name: string;
  category: 'liquidity' | 'profitability' | 'solvency' | 'efficiency' | 'valuation';
  calculate: (financialData: any) => number | null;
  benchmark?: number;
  description: string;
}

export const financialRatios: Record<string, FinancialRatio> = {
  current_ratio: {
    name: 'Current Ratio',
    category: 'liquidity',
    description: 'Measures the company\'s ability to pay short-term obligations',
    calculate: (data) => {
      if (!data.balance_sheet || !data.balance_sheet[0]) return null;
      const bs = data.balance_sheet[0];
      return parseFloat(bs.totalCurrentAssets) / parseFloat(bs.totalCurrentLiabilities);
    },
    benchmark: 2.0
  },

  quick_ratio: {
    name: 'Quick Ratio',
    category: 'liquidity',
    description: 'Measures the company\'s ability to pay short-term obligations with liquid assets',
    calculate: (data) => {
      if (!data.balance_sheet || !data.balance_sheet[0]) return null;
      const bs = data.balance_sheet[0];
      const quickAssets = parseFloat(bs.totalCurrentAssets) - parseFloat(bs.inventory || 0);
      return quickAssets / parseFloat(bs.totalCurrentLiabilities);
    },
    benchmark: 1.0
  },

  gross_margin: {
    name: 'Gross Margin',
    category: 'profitability',
    description: 'Percentage of revenue available after direct costs',
    calculate: (data) => {
      if (!data.income_statement || !data.income_statement[0]) return null;
      const is = data.income_statement[0];
      return parseFloat(is.grossProfit) / parseFloat(is.totalRevenue);
    },
    benchmark: 0.4
  },

  net_profit_margin: {
    name: 'Net Profit Margin',
    category: 'profitability',
    description: 'Percentage of revenue that is net income',
    calculate: (data) => {
      if (!data.income_statement || !data.income_statement[0]) return null;
      const is = data.income_statement[0];
      return parseFloat(is.netIncome) / parseFloat(is.totalRevenue);
    },
    benchmark: 0.1
  },

  return_on_assets: {
    name: 'Return on Assets',
    category: 'profitability',
    description: 'How efficiently assets are used to generate earnings',
    calculate: (data) => {
      if (!data.income_statement || !data.income_statement[0] || !data.balance_sheet || !data.balance_sheet[0]) return null;
      const is = data.income_statement[0];
      const bs = data.balance_sheet[0];
      return parseFloat(is.netIncome) / parseFloat(bs.totalAssets);
    },
    benchmark: 0.05
  },

  return_on_equity: {
    name: 'Return on Equity',
    category: 'profitability',
    description: 'How efficiently equity is used to generate earnings',
    calculate: (data) => {
      if (!data.income_statement || !data.income_statement[0] || !data.balance_sheet || !data.balance_sheet[0]) return null;
      const is = data.income_statement[0];
      const bs = data.balance_sheet[0];
      return parseFloat(is.netIncome) / parseFloat(bs.totalShareholderEquity);
    },
    benchmark: 0.15
  },

  debt_to_equity: {
    name: 'Debt to Equity',
    category: 'solvency',
    description: 'Relative proportion of debt to equity used to finance assets',
    calculate: (data) => {
      if (!data.balance_sheet || !data.balance_sheet[0]) return null;
      const bs = data.balance_sheet[0];
      return parseFloat(bs.totalLiabilities) / parseFloat(bs.totalShareholderEquity);
    },
    benchmark: 1.5
  },

  interest_coverage: {
    name: 'Interest Coverage',
    category: 'solvency',
    description: 'Ability to pay interest on outstanding debt',
    calculate: (data) => {
      if (!data.income_statement || !data.income_statement[0]) return null;
      const is = data.income_statement[0];
      if (!is.interestExpense || parseFloat(is.interestExpense) === 0) return null;
      return parseFloat(is.ebit || is.operatingIncome) / parseFloat(is.interestExpense);
    },
    benchmark: 3.0
  },

  asset_turnover: {
    name: 'Asset Turnover',
    category: 'efficiency',
    description: 'Efficiency of asset utilization to generate revenue',
    calculate: (data) => {
      if (!data.income_statement || !data.income_statement[0] || !data.balance_sheet || !data.balance_sheet[0]) return null;
      const is = data.income_statement[0];
      const bs = data.balance_sheet[0];
      return parseFloat(is.totalRevenue) / parseFloat(bs.totalAssets);
    },
    benchmark: 0.5
  },

  inventory_turnover: {
    name: 'Inventory Turnover',
    category: 'efficiency',
    description: 'How many times inventory is sold and replaced in a period',
    calculate: (data) => {
      if (!data.income_statement || !data.income_statement[0] || !data.balance_sheet || !data.balance_sheet[0]) return null;
      const is = data.income_statement[0];
      const bs = data.balance_sheet[0];
      if (!bs.inventory || parseFloat(bs.inventory) === 0) return null;
      return parseFloat(is.costOfGoods || is.costofGoodsAndServicesSold) / parseFloat(bs.inventory);
    },
    benchmark: 4.0
  }
};

export function calculateAllRatios(financialData: any): Record<string, number | null> {
  const results: Record<string, number | null> = {};
  
  for (const [key, ratio] of Object.entries(financialRatios)) {
    results[key] = ratio.calculate(financialData);
  }
  
  return results;
}

export function calculateSelectedRatios(financialData: any, ratioKeys: string[]): Record<string, number | null> {
  const results: Record<string, number | null> = {};
  
  for (const key of ratioKeys) {
    if (financialRatios[key]) {
      results[key] = financialRatios[key].calculate(financialData);
    }
  }
  
  return results;
}

export function compareRatiosToIndustry(
  companyRatios: Record<string, number | null>,
  industryRatios: Record<string, number>
): Record<string, {value: number | null, industry: number, difference: number | null, percentageDifference: number | null}> {
  const results: Record<string, {value: number | null, industry: number, difference: number | null, percentageDifference: number | null}> = {};
  
  for (const [key, ratio] of Object.entries(companyRatios)) {
    if (ratio !== null && industryRatios[key] !== undefined) {
      const difference = ratio - industryRatios[key];
      const percentageDifference = (difference / industryRatios[key]) * 100;
      
      results[key] = {
        value: ratio,
        industry: industryRatios[key],
        difference,
        percentageDifference
      };
    }
  }
  
  return results;
}
```

## Step 7: Create Financial Statement Analysis Handler

Create `src/handlers/statements.ts`:

```typescript
import { McpToolCallContext, formatSuccessResult, ErrorCode, McpError } from '@modelcontextprotocol/sdk';
import { getIncomeStatement, getBalanceSheet, getCashFlow, getOverview, getIndustryAverages } from '../services/alphavantage';
import { storeAnalysisResult, getPreviousAnalysis } from '../database/db';
import { logInfo, logError } from '../utils/logger';
import { calculateAllRatios, calculateSelectedRatios, compareRatiosToIndustry } from '../models/ratio_models';
import moment from 'moment';

export async function handleAnalyzeStatements(ctx: McpToolCallContext) {
  try {
    const { 
      company_id, 
      ticker, 
      statements, 
      years = 3, 
      quarters = false, 
      ratios = [],
      comparison_benchmark 
    } = ctx.args;
    
    let financialData: any = {
      income_statement: null,
      balance_sheet: null,
      cash_flow: null,
      overview: null
    };
    
    // If statements provided directly, use those
    if (statements) {
      financialData = {
        income_statement: statements.income_statement || null,
        balance_sheet: statements.balance_sheet || null,
        cash_flow: statements.cash_flow || null
      };
    }
    // Otherwise fetch from API using ticker
    else if (ticker) {
      // Check cache first
      const cachedResult = await getPreviousAnalysis(ticker);
      if (cachedResult) {
        // Check if cache is fresh (less than 24 hours old)
        const timestamp = new Date(cachedResult.timestamp);
        const oneDayAgo = new Date();
        oneDayAgo.setDate(oneDayAgo.getDate() - 1);
        
        if (timestamp > oneDayAgo) {
          return formatSuccessResult(cachedResult);
        }
      }
      
      // Fetch statements
      const period = quarters ? 'quarterly' : 'annual';
      
      // Make API calls in parallel
      const [incomeStatementData, balanceSheetData, cashFlowData, overviewData] = await Promise.all([
        getIncomeStatement(ticker, period),
        getBalanceSheet(ticker, period),
        getCashFlow(ticker, period),
        getOverview(ticker)
      ]);
      
      // Limit to requested number of years
      financialData = {
        income_statement: incomeStatementData.slice(0, years),
        balance_sheet: balanceSheetData.slice(0, years),
        cash_flow: cashFlowData.slice(0, years),
        overview: overviewData
      };
    }
    // Handle custom company data by ID
    else if (company_id) {
      // In a real implementation, this would fetch from your custom database
      throw new McpError(
        ErrorCode.NotImplemented,
        "Custom company data retrieval not implemented"
      );
    } else {
      throw new McpError(
        ErrorCode.InvalidArgument,
        "Either ticker, company_id, or statements must be provided"
      );
    }
    
    // Calculate financial ratios
    let ratioResults;
    if (ratios && ratios.length > 0) {
      ratioResults = calculateSelectedRatios(financialData, ratios);
    } else {
      ratioResults = calculateAllRatios(financialData);
    }
    
    // Get industry comparison data if benchmark specified
    let industryComparison = null;
    if (comparison_benchmark && financialData.overview) {
      const industry = financialData.overview.Industry;
      const industryRatios = await getIndustryAverages(industry);
      industryComparison = compareRatiosToIndustry(ratioResults, industryRatios);
    }
    
    // Calculate historical trends (YoY growth)
    const trends = calculateTrends(financialData);
    
    // Generate financial health summary
    const healthSummary = generateHealthSummary(ratioResults, trends);
    
    // Prepare analysis result
    const analysisResult = {
      company_info: {
        ticker: ticker || null,
        name: financialData.overview?.Name || null,
        sector: financialData.overview?.Sector || null,
        industry: financialData.overview?.Industry || null
      },
      statement_periods: financialData.income_statement?.map((stmt: any) => stmt.fiscalDateEnding) || [],
      financial_ratios: ratioResults,
      industry_comparison: industryComparison,
      trends: trends,
      health_summary: healthSummary,
      timestamp: new Date().toISOString()
    };
    
    // Store result for future use if we have a ticker
    if (ticker) {
      await storeAnalysisResult(ticker, analysisResult);
    }
    
    return formatSuccessResult(analysisResult);
  } catch (error) {
    logError(`analyze_statements handler error: ${error.message}`);
    if (error instanceof McpError) {
      throw error;
    }
    throw new McpError(
      ErrorCode.InternalError,
      `Financial analysis failed: ${error.message}`
    );
  }
}

// Helper function to calculate trends from financial data
function calculateTrends(financialData: any) {
  const trends: any = {
    revenue: { values: [], growth: [] },
    net_income: { values: [], growth: [] },
    gross_margin: { values: [], growth: [] },
    operating_margin: { values: [], growth: [] }
  };
  
  if (!financialData.income_statement || financialData.income_statement.length < 2) {
    return trends;
  }
  
  // Sort statements by date (oldest first)
  const statements = [...financialData.income_statement].sort((a: any, b: any) => {
    return moment(a.fiscalDateEnding).diff(moment(b.fiscalDateEnding));
  });
  
  // Extract values
  for (let i = 0; i < statements.length; i++) {
    const stmt = statements[i];
    const revenue = parseFloat(stmt.totalRevenue);
    const netIncome = parseFloat(stmt.netIncome);
    const grossProfit = parseFloat(stmt.grossProfit);
    const operatingIncome = parseFloat(stmt.operatingIncome || stmt.ebit);
    
    trends.revenue.values.push(revenue);
    trends.net_income.values.push(netIncome);
    trends.gross_margin.values.push(grossProfit / revenue);
    trends.operating_margin.values.push(operatingIncome / revenue);
    
    // Calculate growth (starting from second period)
    if (i > 0) {
      const prevRevenue = trends.revenue.values[i-1];
      const prevNetIncome = trends.net_income.values[i-1];
      const prevGrossMargin = trends.gross_margin.values[i-1];
      const prevOperatingMargin = trends.operating_margin.values[i-1];
      
      trends.revenue.growth.push((revenue - prevRevenue) / prevRevenue);
      trends.net_income.growth.push((netIncome - prevNetIncome) / Math.abs(prevNetIncome));
      trends.gross_margin.growth.push(trends.gross_margin.values[i] - prevGrossMargin);
      trends.operating_margin.growth.push(trends.operating_margin.values[i] - prevOperatingMargin);
    }
  }
  
  return trends;
}

// Helper function to generate health summary
function generateHealthSummary(ratios: Record<string, number | null>, trends: any) {
  // This would be more sophisticated in a real implementation
  // Here's a simple version for demonstration
  
  const summary = {
    liquidity: { score: 0, assessment: '' },
    profitability: { score: 0, assessment: '' },
    solvency: { score: 0, assessment: '' },
    growth: { score: 0, assessment: '' },
    overall: { score: 0, assessment: '' }
  };
  
  // Liquidity assessment
  if (ratios.current_ratio && ratios.quick_ratio) {
    const currentRatioScore = ratios.current_ratio >= 2.0 ? 2 : (ratios.current_ratio >= 1.0 ? 1 : 0);
    const quickRatioScore = ratios.quick_ratio >= 1.0 ? 2 : (ratios.quick_ratio >= 0.5 ? 1 : 0);
    summary.liquidity.score = (currentRatioScore + quickRatioScore) / 2;
    
    if (summary.liquidity.score >= 1.5) {
      summary.liquidity.assessment = "Strong liquidity position";
    } else if (summary.liquidity.score >= 0.5) {
      summary.liquidity.assessment = "Adequate liquidity, but monitor closely";
    } else {
      summary.liquidity.assessment = "Potential liquidity concerns";
    }
  }
  
  // Profitability assessment
  if (ratios.net_profit_margin && ratios.return_on_equity) {
    const marginScore = ratios.net_profit_margin >= 0.15 ? 2 : (ratios.net_profit_margin >= 0.05 ? 1 : 0);
    const roeScore = ratios.return_on_equity >= 0.15 ? 2 : (ratios.return_on_equity >= 0.08 ? 1 : 0);
    summary.profitability.score = (marginScore + roeScore) / 2;
    
    if (summary.profitability.score >= 1.5) {
      summary.profitability.assessment = "Strong profitability metrics";
    } else if (summary.profitability.score >= 0.5) {
      summary.profitability.assessment = "Average profitability performance";
    } else {
      summary.profitability.assessment = "Below average profitability";
    }
  }
  
  // Solvency assessment
  if (ratios.debt_to_equity && ratios.interest_coverage) {
    const debtEquityScore = ratios.debt_to_equity <= 1.0 ? 2 : (ratios.debt_to_equity <= 2.0 ? 1 : 0);
    const coverageScore = ratios.interest_coverage >= 3.0 ? 2 : (ratios.interest_coverage >= 1.5 ? 1 : 0);
    summary.solvency.score = (debtEquityScore + coverageScore) / 2;
    
    if (summary.solvency.score >= 1.5) {
      summary.solvency.assessment = "Strong solvency position with low debt risk";
    } else if (summary.solvency.score >= 0.5) {
      summary.solvency.assessment = "Moderate debt levels, manageable in normal conditions";
    } else {
      summary.solvency.assessment = "High debt burden may present challenges";
    }
  }
  
  // Growth assessment
  if (trends.revenue.growth.length > 0) {
    const recentRevenueGrowth = trends.revenue.growth[trends.revenue.growth.length - 1];
    const recentProfitGrowth = trends.net_income.growth[trends.net_income.growth.length - 1];
    
    const revenueScore = recentRevenueGrowth >= 0.1 ? 2 : (recentRevenueGrowth >= 0 ? 1 : 0);
    const profitScore = recentProfitGrowth >= 0.15 ? 2 : (recentProfitGrowth >= 0 ? 1 : 0);
    summary.growth.score = (revenueScore + profitScore) / 2;
    
    if (summary.growth.score >= 1.5) {
      summary.growth.assessment = "Strong growth trajectory";
    } else if (summary.growth.score >= 0.5) {
      summary.growth.assessment = "Modest growth performance";
    } else {
      summary.growth.assessment = "Growth concerns, may be stagnant or declining";
    }
  }
  
  // Overall assessment
  const scores = [
    summary.liquidity.score,
    summary.profitability.score,
    summary.solvency.score,
    summary.growth.score
  ].filter(score => score > 0);
  
  if (scores.length > 0) {
    summary.overall.score = scores.reduce((a, b) => a + b, 0) / scores.length;
    
    if (summary.overall.score >= 1.5) {
      summary.overall.assessment = "Strong financial health with good fundamentals";
    } else if (summary.overall.score >= 1.0) {
      summary.overall.assessment = "Good financial health, some areas for improvement";
    } else if (summary.overall.score >= 0.5) {
      summary.overall.assessment = "Average financial health, multiple areas need attention";
    } else {
      summary.overall.assessment = "Financial health concerns, significant improvement needed";
    }
  }
  
  return summary;
}
```

## Step 8: Create Database Manager

Create `src/database/db.ts`:

```typescript
import sqlite3 from 'sqlite3';
import { open, Database } from 'sqlite';
import { logInfo, logError } from '../utils/logger';

// Define database path
const DB_PATH = './data/financial_analysis.db';

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
      CREATE TABLE IF NOT EXISTS financial_analysis (
        ticker TEXT PRIMARY KEY,
        result TEXT NOT NULL,
        timestamp TEXT NOT NULL
      );
      
      CREATE TABLE IF NOT EXISTS financial_forecasts (
        forecast_id TEXT PRIMARY KEY,
        ticker TEXT NOT NULL,
        result TEXT NOT NULL,
        timestamp TEXT NOT NULL,
        FOREIGN KEY(ticker) REFERENCES financial_analysis(ticker)
      );
      
      CREATE TABLE IF NOT EXISTS variance_analyses (
        analysis_id TEXT PRIMARY KEY,
        description TEXT NOT NULL,
        result TEXT NOT NULL,
        timestamp TEXT NOT NULL
      );
    `);
    
    logInfo('Financial database initialized successfully');
  } catch (error) {
    logError(`Database initialization failed: ${error.message}`);
    throw error;
  }
}

// Store a financial analysis result
export async function storeAnalysisResult(ticker: string, result: any): Promise<void> {
  try {
    await db.run(
      `INSERT OR REPLACE INTO financial_analysis
       (ticker, result, timestamp)
       VALUES (?, ?, ?)`,
      ticker,
      JSON.stringify(result),
      new Date().toISOString()
    );
  } catch (error) {
    logError(`Failed to store analysis: ${error.message}`);
    throw error;
  }
}

// Get a previous financial analysis
export async function getPreviousAnalysis(ticker: string): Promise<any | null> {
  try {
    const row = await db.get(
      `SELECT * FROM financial_analysis WHERE ticker = ?`,
      ticker
    );
    
    if (!row) return null;
    
    // Parse the result JSON
    const result = JSON.parse(row.result);
    result.timestamp = row.timestamp;
    
    return result;
  } catch (error) {
    logError(`Failed to get previous analysis: ${error.message}`);
    return null;
  }
}
```

## Step 9: Create Main Server File

Create `src/server.ts`:

```typescript
import { McpServerBuilder, onToolCall } from '@modelcontextprotocol/sdk';
import { tools } from './tools/schema';
import { initializeDatabase } from './database/db';
import { handleAnalyzeStatements } from './handlers/statements';
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
      .withName('financial-analysis-mcp')
      .withDescription('Financial statement analysis, ratio calculation, and forecasting')
      .withTools(tools)
      .withToolCallHandler(
        onToolCall('analyze_statements', handleAnalyzeStatements)
        // Add handlers for other tools:
        // .onToolCall('forecast_financials', handleForecastFinancials)
        // .onToolCall('variance_analysis', handleVarianceAnalysis)
        // .onToolCall('risk_analysis', handleRiskAnalysis)
      )
      .build();
    
    // Start server
    server.listen();
    logInfo('Financial Analysis MCP server started and listening');
  } catch (error) {
    logError(`Failed to start server: ${error.message}`);
    process.exit(1);
  }
}

startServer();
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
      "name": "financial-analysis-mcp",
      "command": "node",
      "args": ["/absolute/path/to/financial-analysis-mcp/build/server.js"],
      "env": {
        "ALPHA_VANTAGE_API_KEY": "your_api_key_here"
      }
    }
  ]
}
```

## Example Usage in FP&A Workflows

Once configured, you can use your custom MCP server within your FP&A process:

```
# Example of using the tool in a financial analysis workflow

# 1. Analyze Apple's financial statements
response = use_mcp_tool(
    server_name="financial-analysis-mcp",
    tool_name="analyze_statements",
    arguments={
        "ticker": "AAPL",
        "years": 5,
        "quarters": false,
        "comparison_benchmark": "industry"
    }
)

# 2. Extract key insights for your executive summary
liquidity_assessment = response["health_summary"]["liquidity"]["assessment"]
profitability_assessment = response["health_summary"]["profitability"]["assessment"]
current_ratio = response["financial_ratios"]["current_ratio"]
profit_margin = response["financial_ratios"]["net_profit_margin"]

# 3. Generate a report like:
"""
# Financial Analysis: Apple Inc.

## Key Performance Indicators
- Current Ratio: 1.8 (Industry avg: 1.6)
- Net Profit Margin: 25.3% (Industry avg: 15.2%)
- Return on Equity: 45.1% (Industry avg: 18.7%)

## Financial Health Assessment
- Liquidity: Strong liquidity position
- Profitability: Excellent profit generation capabilities
- Solvency: Low debt risk with strong coverage

## Growth Trends
- Revenue: 8.3% YoY growth
- Net Income: 12.1% YoY growth
- Margins: Stable with slight improvements

## Recommendations
Based on the financial analysis, consider increasing R&D investments 
while maintaining the strong liquidity position. The company has 
capacity for additional strategic acquisitions if aligned with 
growth objectives.
"""
```

## Extending For Advanced FP&A Needs

To make your server even more useful for financial planning:

1. **Add a forecasting tool** that projects financial statements based on historical patterns
2. **Implement scenario analysis** with different economic assumptions
3. **Create cash flow forecasting** features to aid in capital planning
4. **Add budget variance analysis** to compare actual vs planned performance

## Conclusion

This example demonstrates how FP&A professionals can create custom MCP servers for specialized financial analysis needs. The financial statement analyzer provides a concrete example of extending AI capabilities with domain-specific financial tools.

By following this pattern, you can create tools for your specific financial analysis needs that go beyond what general-purpose AI tools and traditional Excel models offer.