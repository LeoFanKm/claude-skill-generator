# Internal Scraping Tool + API Infrastructure - Technical Architecture

**Version**: 1.0
**Date**: 2025-11-13
**Status**: 🎯 Ready for Implementation
**Purpose**: Internal tool with API capability (commercialization deferred)
**Timeline**: 5 weeks (MVP)

---

## 🎯 Executive Summary

### What We're Building

A **general-purpose web scraping engine** for Claudate.com's internal operations team, implementing the intelligent 5-phase methodology from the yfe404/web-scraper project. The system is designed with API capability but remains **internal-only** with no immediate commercialization.

### Core Value Proposition

**Empower the operations team to extract data from any website** without writing code, using AI-guided strategy selection and progressive intelligence gathering.

### Key Innovation

**AI-first scraping strategy** that automatically determines the optimal extraction method:
1. **API Discovery** (fastest, most reliable)
2. **Sitemap + API** (optimal balance)
3. **Sitemap + HTML Parsing** (fallback)
4. **Browser Automation** (last resort)

Based on the proven methodology from https://github.com/yfe404/web-scraper

---

## 🏗️ System Architecture

### High-Level Overview

```
┌─────────────────────────────────────────────────────────────┐
│                   User Interface Layer                        │
├─────────────────────────────────────────────────────────────┤
│                                                               │
│  ┌──────────────────────────────────────────────────────┐   │
│  │      Internal Dashboard (Cloudflare Pages)            │   │
│  │  • Task Creator (no-code interface)                   │   │
│  │  • Strategy Recommender (AI-powered)                  │   │
│  │  • Job Monitor (real-time status)                     │   │
│  │  • Data Explorer (query results)                      │   │
│  └──────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
                           ↓
┌─────────────────────────────────────────────────────────────┐
│                    Strategy Layer (AI)                        │
├─────────────────────────────────────────────────────────────┤
│                                                               │
│  ┌──────────────────────────────────────────────────────┐   │
│  │         Strategy Recommender (Worker + AI)            │   │
│  │  Phase 1: Reconnaissance (analyze target)             │   │
│  │  Phase 2: Discovery (find APIs, sitemaps)             │   │
│  │  Phase 3: Strategy Selection (AI recommendation)      │   │
│  └──────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
                           ↓
┌─────────────────────────────────────────────────────────────┐
│                  Execution Layer (Workers)                    │
├─────────────────────────────────────────────────────────────┤
│                                                               │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐         │
│  │ API Scraper │  │ HTML Parser │  │  Browser    │         │
│  │  (Fast)     │  │  (Medium)   │  │ Automation  │         │
│  │             │  │             │  │   (Slow)    │         │
│  └─────────────┘  └─────────────┘  └─────────────┘         │
│                                                               │
│  Strategy Priority: API > Sitemap+API > Sitemap+HTML > Browser│
└─────────────────────────────────────────────────────────────┘
                           ↓
┌─────────────────────────────────────────────────────────────┐
│                      Storage Layer                            │
├─────────────────────────────────────────────────────────────┤
│                                                               │
│  ┌──────────────────────┐  ┌──────────────────────┐         │
│  │  D1 Database         │  │  R2 Storage          │         │
│  │  • Tasks metadata    │  │  • Raw HTML/JSON     │         │
│  │  • Job status        │  │  • Extracted data    │         │
│  │  • Schedules         │  │  • Media files       │         │
│  └──────────────────────┘  └──────────────────────┘         │
└─────────────────────────────────────────────────────────────┘
                           ↓
┌─────────────────────────────────────────────────────────────┐
│               API Layer (Internal + Future)                   │
├─────────────────────────────────────────────────────────────┤
│                                                               │
│  ┌──────────────────────────────────────────────────────┐   │
│  │            REST API (Worker)                          │   │
│  │  POST /api/scrape/analyze (strategy recommendation)   │   │
│  │  POST /api/scrape/execute (run scraping job)         │   │
│  │  GET  /api/scrape/jobs/:id (check job status)        │   │
│  │  GET  /api/scrape/data/:id (retrieve results)        │   │
│  │                                                        │   │
│  │  🔒 Auth: Internal only (Clerk + API keys)           │   │
│  │  📊 Rate Limit: Generous (internal use)              │   │
│  └──────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

---

## 📁 File Structure

```
internal-scraper/
├── packages/
│   ├── dashboard/                    # Frontend UI
│   │   ├── src/
│   │   │   ├── pages/
│   │   │   │   ├── index.tsx         # Task list
│   │   │   │   ├── create.tsx        # Create new task
│   │   │   │   ├── jobs/[id].tsx     # Job details
│   │   │   │   └── data/[id].tsx     # Data explorer
│   │   │   ├── components/
│   │   │   │   ├── TaskForm.tsx      # No-code task creator
│   │   │   │   ├── StrategyRecommender.tsx
│   │   │   │   ├── JobMonitor.tsx
│   │   │   │   └── DataTable.tsx
│   │   │   └── lib/
│   │   │       └── api.ts            # API client
│   │   └── package.json
│   │
│   ├── workers/
│   │   ├── api/                      # Main API worker
│   │   │   ├── src/
│   │   │   │   ├── index.ts
│   │   │   │   ├── routes/
│   │   │   │   │   ├── analyze.ts    # POST /api/scrape/analyze
│   │   │   │   │   ├── execute.ts    # POST /api/scrape/execute
│   │   │   │   │   ├── jobs.ts       # GET /api/scrape/jobs/:id
│   │   │   │   │   └── data.ts       # GET /api/scrape/data/:id
│   │   │   │   ├── middleware/
│   │   │   │   │   ├── auth.ts       # Clerk + API key auth
│   │   │   │   │   └── rate-limit.ts # Rate limiting
│   │   │   │   └── types.ts
│   │   │   └── wrangler.toml
│   │   │
│   │   ├── strategy-recommender/     # AI strategy worker
│   │   │   ├── src/
│   │   │   │   ├── index.ts
│   │   │   │   ├── reconnaissance.ts # Phase 1: Analyze site
│   │   │   │   ├── discovery.ts      # Phase 2: Find APIs/sitemaps
│   │   │   │   ├── selector.ts       # Phase 3: Choose strategy
│   │   │   │   └── ai-engine.ts      # Gemini integration
│   │   │   └── wrangler.toml
│   │   │
│   │   ├── scrapers/
│   │   │   ├── api-scraper/          # Strategy 1: Pure API
│   │   │   │   ├── src/
│   │   │   │   │   ├── index.ts
│   │   │   │   │   └── extractors/
│   │   │   │   │       ├── json.ts
│   │   │   │   │       ├── xml.ts
│   │   │   │   │       └── graphql.ts
│   │   │   │   └── wrangler.toml
│   │   │   │
│   │   │   ├── html-parser/          # Strategy 2: HTML parsing
│   │   │   │   ├── src/
│   │   │   │   │   ├── index.ts
│   │   │   │   │   ├── cheerio-engine.ts
│   │   │   │   │   └── selector-builder.ts
│   │   │   │   └── wrangler.toml
│   │   │   │
│   │   │   └── browser-automation/   # Strategy 3: Puppeteer
│   │   │       ├── src/
│   │   │       │   ├── index.ts
│   │   │       │   └── browser-client.ts
│   │   │       └── wrangler.toml
│   │   │
│   │   └── orchestrator/             # Job orchestration
│   │       ├── src/
│   │       │   ├── index.ts
│   │       │   ├── scheduler.ts      # Cron jobs
│   │       │   ├── queue-handler.ts  # Queue processing
│   │       │   └── retry-logic.ts    # Error handling
│   │       └── wrangler.toml
│   │
│   └── shared/                       # Shared types
│       ├── types.ts
│       ├── constants.ts
│       └── utils.ts
│
├── database/
│   ├── migrations/
│   │   ├── 001_init_scraper.sql
│   │   ├── 002_add_strategy_history.sql
│   │   └── 003_add_api_keys.sql
│   └── schema.sql
│
├── docs/
│   ├── METHODOLOGY.md                # 5-phase methodology explained
│   ├── API_REFERENCE.md              # API documentation
│   ├── STRATEGY_GUIDE.md             # When to use each strategy
│   └── EXAMPLES.md                   # Common use cases
│
└── scripts/
    ├── deploy.sh                     # Deployment script
    └── seed-examples.ts              # Seed example tasks
```

**Total Estimated Code**: ~4,200 lines TypeScript

---

## 🗄️ Database Schema

```sql
-- migrations/001_init_scraper.sql

-- Scraping tasks (user-defined configurations)
CREATE TABLE scraper_tasks (
  id TEXT PRIMARY KEY,
  name TEXT NOT NULL,
  description TEXT,
  target_url TEXT NOT NULL,

  -- Strategy
  strategy TEXT CHECK(strategy IN ('auto', 'api', 'sitemap_api', 'sitemap_html', 'browser')),
  strategy_config TEXT, -- JSON: API endpoints, selectors, etc.

  -- Schedule
  schedule_type TEXT CHECK(schedule_type IN ('manual', 'interval', 'cron')),
  schedule_config TEXT, -- JSON: cron expression or interval

  -- Data schema (what to extract)
  data_schema TEXT NOT NULL, -- JSON: { "field_name": "selector/api_path" }

  -- Metadata
  created_by TEXT NOT NULL, -- User ID from Clerk
  created_at INTEGER NOT NULL,
  updated_at INTEGER NOT NULL,
  is_active BOOLEAN DEFAULT TRUE
);

CREATE INDEX idx_tasks_user ON scraper_tasks(created_by);
CREATE INDEX idx_tasks_active ON scraper_tasks(is_active);

-- Scraping jobs (individual execution instances)
CREATE TABLE scraper_jobs (
  id TEXT PRIMARY KEY,
  task_id TEXT NOT NULL REFERENCES scraper_tasks(id) ON DELETE CASCADE,

  -- Execution
  status TEXT CHECK(status IN ('pending', 'running', 'completed', 'failed', 'cancelled')),
  strategy_used TEXT, -- Actual strategy used (may differ from task.strategy if auto)
  started_at INTEGER,
  completed_at INTEGER,
  duration_ms INTEGER,

  -- Results
  items_extracted INTEGER DEFAULT 0,
  items_failed INTEGER DEFAULT 0,
  data_location TEXT, -- R2 object key where results are stored

  -- Error handling
  error_message TEXT,
  retry_count INTEGER DEFAULT 0,
  max_retries INTEGER DEFAULT 3,

  -- Metadata
  triggered_by TEXT, -- 'manual', 'schedule', 'api'
  created_at INTEGER NOT NULL
);

CREATE INDEX idx_jobs_task ON scraper_jobs(task_id, created_at DESC);
CREATE INDEX idx_jobs_status ON scraper_jobs(status);

-- Strategy analysis history (AI recommendations)
CREATE TABLE strategy_analyses (
  id TEXT PRIMARY KEY,
  task_id TEXT REFERENCES scraper_tasks(id) ON DELETE CASCADE,
  target_url TEXT NOT NULL,

  -- Discovery results
  has_api BOOLEAN DEFAULT FALSE,
  api_endpoints TEXT, -- JSON array of discovered endpoints
  has_sitemap BOOLEAN DEFAULT FALSE,
  sitemap_url TEXT,
  has_robots_txt BOOLEAN DEFAULT FALSE,

  -- AI recommendation
  recommended_strategy TEXT NOT NULL,
  confidence_score REAL, -- 0-1
  reasoning TEXT, -- AI's explanation
  estimated_performance TEXT, -- "fast", "medium", "slow"

  analyzed_at INTEGER NOT NULL
);

CREATE INDEX idx_analyses_task ON strategy_analyses(task_id);

-- Extracted data samples (for preview/debugging)
CREATE TABLE data_samples (
  id TEXT PRIMARY KEY,
  job_id TEXT NOT NULL REFERENCES scraper_jobs(id) ON DELETE CASCADE,
  sample_data TEXT NOT NULL, -- JSON: first 10 items
  created_at INTEGER NOT NULL
);

-- API keys (for future external API access)
CREATE TABLE api_keys (
  id TEXT PRIMARY KEY,
  key_hash TEXT NOT NULL UNIQUE,
  key_prefix TEXT NOT NULL, -- First 8 chars for display
  name TEXT,

  -- Permissions
  user_id TEXT NOT NULL, -- Owner
  scopes TEXT NOT NULL, -- JSON array: ["read", "write", "execute"]

  -- Usage
  usage_count INTEGER DEFAULT 0,
  last_used_at INTEGER,

  -- Limits
  rate_limit_per_hour INTEGER DEFAULT 100,
  is_active BOOLEAN DEFAULT TRUE,

  created_at INTEGER NOT NULL,
  expires_at INTEGER
);

CREATE INDEX idx_api_keys_hash ON api_keys(key_hash);
CREATE INDEX idx_api_keys_user ON api_keys(user_id);
```

**Storage Pattern**:
- **D1**: Task configs, job status, API keys (<1KB per row)
- **R2**: Extracted data (JSON), raw HTML, large datasets
- **KV**: Rate limiting counters, caching

---

## 🧠 5-Phase Methodology Implementation

### Phase 1: Interactive Reconnaissance

**Purpose**: Analyze target website to understand structure

```typescript
// workers/strategy-recommender/src/reconnaissance.ts

interface ReconnaissanceResult {
  url: string;
  hasAPI: boolean;
  hasSitemap: boolean;
  hasRobotsTxt: boolean;
  detectedFramework: string | null;
  isJavaScriptHeavy: boolean;
  estimatedComplexity: 'low' | 'medium' | 'high';
}

class ReconnaissanceEngine {
  async analyze(targetUrl: string): Promise<ReconnaissanceResult> {
    const results: ReconnaissanceResult = {
      url: targetUrl,
      hasAPI: false,
      hasSitemap: false,
      hasRobotsTxt: false,
      detectedFramework: null,
      isJavaScriptHeavy: false,
      estimatedComplexity: 'low'
    };

    // Check robots.txt
    const robotsUrl = new URL('/robots.txt', targetUrl).href;
    const robotsResponse = await fetch(robotsUrl);
    if (robotsResponse.ok) {
      results.hasRobotsTxt = true;
      const robotsTxt = await robotsResponse.text();

      // Check for sitemap in robots.txt
      const sitemapMatch = robotsTxt.match(/Sitemap:\s*(.+)/i);
      if (sitemapMatch) {
        results.hasSitemap = true;
      }
    }

    // Check common sitemap locations
    if (!results.hasSitemap) {
      results.hasSitemap = await this.checkSitemap(targetUrl);
    }

    // Fetch homepage and analyze
    const homepageResponse = await fetch(targetUrl);
    const html = await homepageResponse.text();

    // Detect framework
    results.detectedFramework = this.detectFramework(html);

    // Check if JavaScript-heavy (SPA)
    results.isJavaScriptHeavy = this.isJavaScriptHeavy(html);

    // Look for API hints
    results.hasAPI = await this.discoverAPIs(targetUrl, html);

    // Estimate complexity
    results.estimatedComplexity = this.estimateComplexity(results);

    return results;
  }

  private async checkSitemap(baseUrl: string): Promise<boolean> {
    const commonPaths = [
      '/sitemap.xml',
      '/sitemap_index.xml',
      '/sitemap-index.xml',
      '/sitemap1.xml'
    ];

    for (const path of commonPaths) {
      const url = new URL(path, baseUrl).href;
      const response = await fetch(url, { method: 'HEAD' });
      if (response.ok) return true;
    }

    return false;
  }

  private detectFramework(html: string): string | null {
    if (html.includes('__NEXT_DATA__')) return 'Next.js';
    if (html.includes('__nuxt')) return 'Nuxt.js';
    if (html.includes('ng-version')) return 'Angular';
    if (html.includes('data-reactroot') || html.includes('data-reactid')) return 'React';
    if (html.includes('data-v-')) return 'Vue.js';
    return null;
  }

  private isJavaScriptHeavy(html: string): boolean {
    // Check if page is mostly empty without JS
    const contentWithoutScripts = html
      .replace(/<script[\s\S]*?<\/script>/gi, '')
      .replace(/<style[\s\S]*?<\/style>/gi, '')
      .replace(/<[^>]+>/g, '')
      .trim();

    // If content is less than 500 chars, likely SPA
    return contentWithoutScripts.length < 500;
  }

  private async discoverAPIs(baseUrl: string, html: string): Promise<boolean> {
    // Look for common API patterns in HTML
    const apiPatterns = [
      /\/api\/v?\d+\//i,
      /\/graphql/i,
      /\/rest\/v?\d+\//i,
      /["']https?:\/\/[^"']+\/api\//i
    ];

    for (const pattern of apiPatterns) {
      if (pattern.test(html)) return true;
    }

    // Check common API endpoints
    const commonEndpoints = [
      '/api',
      '/api/v1',
      '/graphql',
      '/rest/v1'
    ];

    for (const endpoint of commonEndpoints) {
      const url = new URL(endpoint, baseUrl).href;
      try {
        const response = await fetch(url, { method: 'HEAD' });
        if (response.ok) return true;
      } catch (e) {
        // Endpoint doesn't exist
      }
    }

    return false;
  }

  private estimateComplexity(results: ReconnaissanceResult): 'low' | 'medium' | 'high' {
    let score = 0;

    if (results.hasAPI) score -= 2; // APIs make it easier
    if (results.hasSitemap) score -= 1;
    if (results.isJavaScriptHeavy) score += 3;
    if (results.detectedFramework) score += 1;

    if (score <= 0) return 'low';
    if (score <= 2) return 'medium';
    return 'high';
  }
}
```

### Phase 2: Automatic Discovery

**Purpose**: Find available APIs, sitemaps, and data sources

```typescript
// workers/strategy-recommender/src/discovery.ts

interface DiscoveryResult {
  apiEndpoints: APIEndpoint[];
  sitemaps: Sitemap[];
  dataFeeds: DataFeed[];
}

interface APIEndpoint {
  url: string;
  method: string;
  responseType: 'json' | 'xml' | 'html';
  requiresAuth: boolean;
  sampleResponse?: any;
}

class DiscoveryEngine {
  async discover(targetUrl: string): Promise<DiscoveryResult> {
    const result: DiscoveryResult = {
      apiEndpoints: [],
      sitemaps: [],
      dataFeeds: []
    };

    // Discover APIs
    result.apiEndpoints = await this.discoverAPIs(targetUrl);

    // Discover sitemaps
    result.sitemaps = await this.discoverSitemaps(targetUrl);

    // Discover RSS/Atom feeds
    result.dataFeeds = await this.discoverDataFeeds(targetUrl);

    return result;
  }

  private async discoverAPIs(baseUrl: string): Promise<APIEndpoint[]> {
    const endpoints: APIEndpoint[] = [];

    // Common API paths to check
    const apiPaths = [
      '/api',
      '/api/v1',
      '/api/v2',
      '/graphql',
      '/rest/v1',
      '/__data.json', // SvelteKit
      '/_next/data', // Next.js
      '/_nuxt/data' // Nuxt
    ];

    for (const path of apiPaths) {
      const url = new URL(path, baseUrl).href;

      try {
        const response = await fetch(url);
        if (response.ok) {
          const contentType = response.headers.get('content-type') || '';
          const responseType = this.detectResponseType(contentType);

          endpoints.push({
            url,
            method: 'GET',
            responseType,
            requiresAuth: response.status === 401 || response.status === 403,
            sampleResponse: responseType === 'json' ? await response.json() : null
          });
        }
      } catch (e) {
        // Endpoint doesn't exist or errored
      }
    }

    return endpoints;
  }

  private async discoverSitemaps(baseUrl: string): Promise<Sitemap[]> {
    const sitemaps: Sitemap[] = [];

    // Check robots.txt first
    const robotsUrl = new URL('/robots.txt', baseUrl).href;
    const robotsResponse = await fetch(robotsUrl);

    if (robotsResponse.ok) {
      const robotsTxt = await robotsResponse.text();
      const sitemapMatches = robotsTxt.matchAll(/Sitemap:\s*(.+)/gi);

      for (const match of sitemapMatches) {
        const sitemapUrl = match[1].trim();
        sitemaps.push(await this.parseSitemap(sitemapUrl));
      }
    }

    // Check common paths
    const commonPaths = ['/sitemap.xml', '/sitemap_index.xml'];
    for (const path of commonPaths) {
      const url = new URL(path, baseUrl).href;
      const response = await fetch(url);

      if (response.ok) {
        sitemaps.push(await this.parseSitemap(url));
      }
    }

    return sitemaps;
  }

  private async parseSitemap(url: string): Promise<Sitemap> {
    const response = await fetch(url);
    const xml = await response.text();

    // Parse XML to extract URLs
    const urlMatches = xml.matchAll(/<loc>(.*?)<\/loc>/g);
    const urls: string[] = [];

    for (const match of urlMatches) {
      urls.push(match[1]);
    }

    return {
      url,
      totalUrls: urls.length,
      urls: urls.slice(0, 100), // First 100 for preview
      isSitemapIndex: xml.includes('<sitemapindex')
    };
  }

  private detectResponseType(contentType: string): 'json' | 'xml' | 'html' {
    if (contentType.includes('json')) return 'json';
    if (contentType.includes('xml')) return 'xml';
    return 'html';
  }
}
```

### Phase 3: Strategy Recommendation (AI-Powered)

**Purpose**: Use AI to recommend optimal scraping strategy

```typescript
// workers/strategy-recommender/src/selector.ts

interface StrategyRecommendation {
  strategy: 'api' | 'sitemap_api' | 'sitemap_html' | 'browser';
  confidence: number; // 0-1
  reasoning: string;
  estimatedSpeed: 'fast' | 'medium' | 'slow';
  estimatedCost: 'low' | 'medium' | 'high';
  implementation: ImplementationGuide;
}

interface ImplementationGuide {
  endpoints?: string[];
  selectors?: Record<string, string>;
  actions?: BrowserAction[];
}

class StrategySelector {
  private aiEngine: AIEngine;

  constructor(aiEngine: AIEngine) {
    this.aiEngine = aiEngine;
  }

  async recommend(
    targetUrl: string,
    recon: ReconnaissanceResult,
    discovery: DiscoveryResult,
    dataSchema: DataSchema
  ): Promise<StrategyRecommendation> {
    // Build context for AI
    const prompt = this.buildRecommendationPrompt(
      targetUrl,
      recon,
      discovery,
      dataSchema
    );

    // Get AI recommendation
    const aiResponse = await this.aiEngine.analyze(prompt);
    const recommendation = this.parseAIResponse(aiResponse);

    // Add implementation details
    recommendation.implementation = await this.buildImplementationGuide(
      recommendation.strategy,
      discovery,
      dataSchema
    );

    return recommendation;
  }

  private buildRecommendationPrompt(
    targetUrl: string,
    recon: ReconnaissanceResult,
    discovery: DiscoveryResult,
    dataSchema: DataSchema
  ): string {
    return `
You are an expert web scraping strategist. Analyze the following website and recommend the optimal scraping strategy.

**Target URL**: ${targetUrl}

**Reconnaissance Results**:
- Has API: ${recon.hasAPI}
- Has Sitemap: ${recon.hasSitemap}
- Framework: ${recon.detectedFramework || 'Unknown'}
- JavaScript-heavy: ${recon.isJavaScriptHeavy}
- Complexity: ${recon.estimatedComplexity}

**Discovery Results**:
- API Endpoints Found: ${discovery.apiEndpoints.length}
  ${discovery.apiEndpoints.map(e => `  - ${e.url} (${e.responseType})`).join('\n')}
- Sitemaps Found: ${discovery.sitemaps.length}
  ${discovery.sitemaps.map(s => `  - ${s.url} (${s.totalUrls} URLs)`).join('\n')}

**Data Schema to Extract**:
${JSON.stringify(dataSchema, null, 2)}

**Available Strategies** (in order of preference):
1. **API**: Use discovered APIs directly (fastest, most reliable)
2. **Sitemap + API**: Combine sitemap URLs with API calls (optimal balance)
3. **Sitemap + HTML**: Parse HTML from sitemap URLs (medium speed)
4. **Browser Automation**: Use headless browser (slowest, last resort)

**Your Task**:
Recommend ONE strategy that best fits this scenario. Consider:
- Speed (API > Sitemap+API > Sitemap+HTML > Browser)
- Reliability (API > others)
- Complexity (simpler is better)
- Data availability (can we get all required fields?)

Return JSON:
{
  "strategy": "api|sitemap_api|sitemap_html|browser",
  "confidence": 0.85,
  "reasoning": "Explain why this strategy is optimal",
  "estimatedSpeed": "fast|medium|slow",
  "estimatedCost": "low|medium|high"
}
`;
  }

  private async buildImplementationGuide(
    strategy: string,
    discovery: DiscoveryResult,
    dataSchema: DataSchema
  ): Promise<ImplementationGuide> {
    const guide: ImplementationGuide = {};

    switch (strategy) {
      case 'api':
        guide.endpoints = discovery.apiEndpoints.map(e => e.url);
        break;

      case 'sitemap_api':
      case 'sitemap_html':
        guide.endpoints = discovery.sitemaps.map(s => s.url);
        // Build CSS selectors for each field in dataSchema
        guide.selectors = await this.generateSelectors(dataSchema);
        break;

      case 'browser':
        guide.actions = await this.generateBrowserActions(dataSchema);
        break;
    }

    return guide;
  }

  private async generateSelectors(dataSchema: DataSchema): Promise<Record<string, string>> {
    // Use AI to suggest CSS selectors for each field
    const prompt = `
Given the following data schema, suggest CSS selectors:

${JSON.stringify(dataSchema, null, 2)}

For each field, provide a likely CSS selector.

Return JSON:
{
  "field_name": "css selector",
  ...
}
`;

    const response = await this.aiEngine.analyze(prompt);
    return JSON.parse(response);
  }
}
```

**Performance Benchmarks** (from yfe404 project):

| Strategy | Speed | Reliability | Complexity | Best For |
|----------|-------|-------------|------------|----------|
| **Pure API** | ⚡⚡⚡⚡⚡ 5x | ⭐⭐⭐⭐⭐ 5/5 | 🟢 Low | Structured data, modern sites |
| **Sitemap + API** | ⚡⚡⚡⚡ 4x | ⭐⭐⭐⭐ 4/5 | 🟡 Medium | Mixed content, URL discovery |
| **Sitemap + HTML** | ⚡⚡⚡ 3x | ⭐⭐⭐ 3/5 | 🟡 Medium | Static content, no API |
| **Browser Automation** | ⚡ 1x | ⭐⭐ 2/5 | 🔴 High | JavaScript-heavy, interactive |

---

## 🎨 Internal Dashboard UI

### Task Creator (No-Code Interface)

```tsx
// dashboard/src/pages/create.tsx

import React, { useState } from 'react';
import { StrategyRecommender } from '../components/StrategyRecommender';

export default function CreateTaskPage() {
  const [step, setStep] = useState<1 | 2 | 3>(1);
  const [taskData, setTaskData] = useState({
    name: '',
    targetUrl: '',
    dataSchema: {},
    strategy: 'auto',
    schedule: 'manual'
  });

  const [recommendation, setRecommendation] = useState(null);

  const handleAnalyze = async () => {
    const response = await fetch('/api/scrape/analyze', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        targetUrl: taskData.targetUrl,
        dataSchema: taskData.dataSchema
      })
    });

    const data = await response.json();
    setRecommendation(data);
    setStep(2);
  };

  return (
    <div className="create-task-page">
      <h1>Create Scraping Task</h1>

      {step === 1 && (
        <div className="step-basic-info">
          <h2>Step 1: Basic Information</h2>

          <div className="form-group">
            <label>Task Name</label>
            <input
              type="text"
              value={taskData.name}
              onChange={(e) => setTaskData({ ...taskData, name: e.target.value })}
              placeholder="e.g., Product Prices from Example.com"
            />
          </div>

          <div className="form-group">
            <label>Target URL</label>
            <input
              type="url"
              value={taskData.targetUrl}
              onChange={(e) => setTaskData({ ...taskData, targetUrl: e.target.value })}
              placeholder="https://example.com/products"
            />
          </div>

          <div className="form-group">
            <label>Data Schema (What to Extract)</label>
            <p className="help-text">Define fields you want to extract:</p>

            <DataSchemaBuilder
              schema={taskData.dataSchema}
              onChange={(schema) => setTaskData({ ...taskData, dataSchema: schema })}
            />
          </div>

          <button onClick={handleAnalyze} className="btn-primary">
            Analyze & Get Recommendation
          </button>
        </div>
      )}

      {step === 2 && recommendation && (
        <div className="step-strategy">
          <h2>Step 2: Strategy Recommendation</h2>

          <StrategyRecommender recommendation={recommendation} />

          <button onClick={() => setStep(3)} className="btn-primary">
            Continue
          </button>
        </div>
      )}

      {step === 3 && (
        <div className="step-schedule">
          <h2>Step 3: Schedule</h2>

          <div className="form-group">
            <label>When to run?</label>
            <select
              value={taskData.schedule}
              onChange={(e) => setTaskData({ ...taskData, schedule: e.target.value })}
            >
              <option value="manual">Manual (on-demand)</option>
              <option value="interval">Interval (every X hours)</option>
              <option value="cron">Cron (custom schedule)</option>
            </select>
          </div>

          <button onClick={handleCreate} className="btn-success">
            Create Task
          </button>
        </div>
      )}
    </div>
  );
}

const DataSchemaBuilder: React.FC<{
  schema: any;
  onChange: (schema: any) => void;
}> = ({ schema, onChange }) => {
  const [fields, setFields] = useState<Array<{ name: string; type: string }>>([
    { name: '', type: 'text' }
  ]);

  const addField = () => {
    setFields([...fields, { name: '', type: 'text' }]);
  };

  const updateField = (index: number, key: string, value: string) => {
    const updated = [...fields];
    updated[index] = { ...updated[index], [key]: value };
    setFields(updated);

    // Convert to schema object
    const schemaObj = updated.reduce((acc, field) => {
      if (field.name) {
        acc[field.name] = field.type;
      }
      return acc;
    }, {} as any);

    onChange(schemaObj);
  };

  return (
    <div className="schema-builder">
      {fields.map((field, index) => (
        <div key={index} className="field-row">
          <input
            type="text"
            placeholder="Field name (e.g., title)"
            value={field.name}
            onChange={(e) => updateField(index, 'name', e.target.value)}
          />

          <select
            value={field.type}
            onChange={(e) => updateField(index, 'type', e.target.value)}
          >
            <option value="text">Text</option>
            <option value="number">Number</option>
            <option value="url">URL</option>
            <option value="date">Date</option>
          </select>

          <button onClick={() => {
            setFields(fields.filter((_, i) => i !== index));
          }}>
            Remove
          </button>
        </div>
      ))}

      <button onClick={addField} className="btn-secondary">
        + Add Field
      </button>
    </div>
  );
};
```

---

## 🔌 API Endpoints (Internal + Future)

### POST /api/scrape/analyze

**Purpose**: Analyze a website and get strategy recommendation

```typescript
// workers/api/src/routes/analyze.ts

export async function handleAnalyze(request: Request, env: Env): Promise<Response> {
  const { targetUrl, dataSchema } = await request.json();

  // Phase 1: Reconnaissance
  const reconEngine = new ReconnaissanceEngine();
  const recon = await reconEngine.analyze(targetUrl);

  // Phase 2: Discovery
  const discoveryEngine = new DiscoveryEngine();
  const discovery = await discoveryEngine.discover(targetUrl);

  // Phase 3: Strategy Recommendation
  const selector = new StrategySelector(env.AI_ENGINE);
  const recommendation = await selector.recommend(
    targetUrl,
    recon,
    discovery,
    dataSchema
  );

  // Store analysis
  await env.DB.prepare(`
    INSERT INTO strategy_analyses (
      id, target_url, has_api, api_endpoints, has_sitemap,
      recommended_strategy, confidence_score, reasoning,
      estimated_performance, analyzed_at
    ) VALUES (?, ?, ?, ?, ?, ?, ?, ?, ?, ?)
  `).bind(
    generateId(),
    targetUrl,
    recon.hasAPI,
    JSON.stringify(discovery.apiEndpoints),
    recon.hasSitemap,
    recommendation.strategy,
    recommendation.confidence,
    recommendation.reasoning,
    recommendation.estimatedSpeed,
    Date.now()
  ).run();

  return jsonResponse({
    reconnaissance: recon,
    discovery,
    recommendation
  });
}
```

### POST /api/scrape/execute

**Purpose**: Execute a scraping job

```typescript
// workers/api/src/routes/execute.ts

export async function handleExecute(request: Request, env: Env): Promise<Response> {
  const { taskId } = await request.json();

  // Load task configuration
  const task = await env.DB.prepare(`
    SELECT * FROM scraper_tasks WHERE id = ?
  `).bind(taskId).first();

  if (!task) {
    return jsonResponse({ error: 'Task not found' }, 404);
  }

  // Create job
  const jobId = generateId();
  await env.DB.prepare(`
    INSERT INTO scraper_jobs (
      id, task_id, status, triggered_by, created_at
    ) VALUES (?, ?, 'pending', 'manual', ?)
  `).bind(jobId, taskId, Date.now()).run();

  // Send to appropriate scraper worker based on strategy
  const strategy = task.strategy === 'auto'
    ? await determineStrategy(task)
    : task.strategy;

  await env.SCRAPER_QUEUE.send({
    jobId,
    taskId,
    strategy,
    config: JSON.parse(task.strategy_config)
  });

  return jsonResponse({
    jobId,
    status: 'pending',
    message: 'Job queued for execution'
  });
}
```

### GET /api/scrape/jobs/:id

**Purpose**: Check job status

```typescript
// workers/api/src/routes/jobs.ts

export async function handleGetJob(request: Request, env: Env): Promise<Response> {
  const url = new URL(request.url);
  const jobId = url.pathname.split('/').pop();

  const job = await env.DB.prepare(`
    SELECT * FROM scraper_jobs WHERE id = ?
  `).bind(jobId).first();

  if (!job) {
    return jsonResponse({ error: 'Job not found' }, 404);
  }

  return jsonResponse({
    id: job.id,
    taskId: job.task_id,
    status: job.status,
    strategy: job.strategy_used,
    itemsExtracted: job.items_extracted,
    itemsFailed: job.items_failed,
    duration: job.duration_ms,
    startedAt: job.started_at,
    completedAt: job.completed_at,
    error: job.error_message,
    dataLocation: job.data_location
  });
}
```

### GET /api/scrape/data/:id

**Purpose**: Retrieve extracted data

```typescript
// workers/api/src/routes/data.ts

export async function handleGetData(request: Request, env: Env): Promise<Response> {
  const url = new URL(request.url);
  const jobId = url.pathname.split('/').pop();

  // Get job to find data location
  const job = await env.DB.prepare(`
    SELECT data_location FROM scraper_jobs WHERE id = ? AND status = 'completed'
  `).bind(jobId).first();

  if (!job || !job.data_location) {
    return jsonResponse({ error: 'Data not found' }, 404);
  }

  // Fetch from R2
  const object = await env.R2_BUCKET.get(job.data_location);

  if (!object) {
    return jsonResponse({ error: 'Data not found in storage' }, 404);
  }

  const data = await object.json();

  return jsonResponse({
    jobId,
    itemCount: data.length,
    data
  });
}
```

---

## 🚀 Implementation Roadmap

### Week 1: Core Infrastructure

- [ ] Set up monorepo + D1 + R2
- [ ] Implement reconnaissance engine
- [ ] Implement discovery engine
- [ ] Test on 5 sample websites

### Week 2: Strategy Selection

- [ ] Integrate Gemini API
- [ ] Build strategy selector
- [ ] Implement recommendation logic
- [ ] Test AI accuracy

### Week 3: Scrapers

- [ ] Build API scraper worker
- [ ] Build HTML parser worker
- [ ] Build browser automation worker (Cloudflare Browser Rendering)
- [ ] Test all strategies

### Week 4: Dashboard UI

- [ ] Build task creator
- [ ] Build job monitor
- [ ] Build data explorer
- [ ] Deploy to Pages

### Week 5: Polish & Testing

- [ ] End-to-end testing
- [ ] Performance optimization
- [ ] Documentation
- [ ] Internal launch

---

## 💰 Cost Estimation

| Resource | Usage | Cost/month |
|----------|-------|------------|
| **Gemini API** | 2M tokens | $10 |
| **D1** | 5M reads, 500K writes | $3.25 |
| **R2** | 100 GB | $1.50 |
| **Workers** | 10M requests | $0.50 |
| **Browser Rendering** | 1K pages | $5 |
| **Total** | Internal use | **~$20/month** |

**Extremely affordable** for internal tool.

---

## ✅ Document Status

**Status**: 🎯 Ready for Implementation

**Next**: Begin Week 1 (Core Infrastructure)

---

**Document Version**: 1.0
**Created**: 2025-11-13
**Lines**: ~2,200

---

**Let's build intelligent scraping!** 🚀
