# Scraper 工具开发指南 - 非技术人员版

**目标**: 基于开源项目 yfe404/web-scraper 二次开发，形成 Claudate 内部的 Scraper 工具

**给开发者的话**: 这份文档是为不懂技术的项目管理者准备的，请按照步骤逐一完成，每个步骤都有明确的验收标准。

---

## 📋 项目概述

### 我们要做什么？

将开源项目 **yfe404/web-scraper**（一个 Claude Code Skill）的**方法论和知识**转化为：
1. **Claudate 内部的实际工具**（紧耦合版本）
2. 使用 Cloudflare 技术栈（符合公司规范）
3. 未来可以解耦成独立产品

### 为什么这样做？

yfe404/web-scraper **不是一个可以直接运行的应用**，而是：
- 📚 **知识库**：教 Claude AI 如何智能地抓取网页
- 📖 **方法论**：5 阶段智能抓取流程
- 💡 **最佳实践**：策略选择、性能优化

我们需要把这些**知识和方法**转化为**真正的代码和工具**。

---

## 🎯 最终目标

### 阶段 1: Claudate 集成版 (Week 1-5)

**紧耦合版本特征**:
- ✅ 集成到 Claudate.com 现有项目中
- ✅ 使用 Claudate 的 Clerk 认证
- ✅ 共享 Claudate 的 D1 数据库
- ✅ 只有 Claudate 团队成员可以访问
- ✅ 从 Claudate 控制面板访问（`/scraper` 路由）

**交付物**:
```
claudate.com/
├── app/
│   ├── scraper/              # 新增：Scraper 功能模块
│   │   ├── page.tsx          # 主页面（任务列表）
│   │   ├── create/page.tsx   # 创建任务页面
│   │   └── jobs/[id]/page.tsx # 任务详情页面
│   └── api/
│       └── scraper/          # 新增：Scraper API 路由
│           ├── analyze/route.ts
│           ├── execute/route.ts
│           └── jobs/[id]/route.ts
├── workers/
│   └── scraper/              # 新增：Scraper Workers
│       ├── reconnaissance.ts
│       ├── discovery.ts
│       ├── strategy.ts
│       └── executor.ts
└── database/
    └── migrations/
        └── 005_scraper.sql   # 新增：Scraper 数据表
```

### 阶段 2: 独立产品化 (未来，Week 6+)

**解耦后特征**:
- ✅ 独立的代码仓库
- ✅ 独立的域名（如 scraper.claudate.com）
- ✅ 独立的认证系统
- ✅ API 对外开放（收费）

**暂时不考虑，先完成阶段 1**

---

## 📝 给另一个 Claude Code 终端的完整提示词

### 如何使用这份指南？

1. **打开新的 Claude Code 终端**
2. **打开 Claudate 项目文件夹**
3. **将下面的"完整提示词"复制粘贴给 Claude**
4. **Claude 会自动执行所有步骤**

---

## 🤖 完整提示词（复制粘贴给新的 Claude Code）

```
# Scraper 工具开发任务

你好！我需要你基于开源项目 yfe404/web-scraper 的方法论，为 Claudate.com 开发一个内部 Scraper 工具。

## 项目背景

**源项目**: https://github.com/yfe404/web-scraper
- 这是一个 Claude Code Skill（知识库），不是可运行的应用
- 包含智能网页抓取的 5 阶段方法论
- 我们需要将其方法论转化为实际的 Cloudflare Workers 应用

**技术栈要求** (必须符合 Claudate 规范):
- **框架**: Next.js 14+ (App Router)
- **云服务**: Cloudflare (Workers, D1, R2, KV, Pages)
- **语言**: TypeScript
- **认证**: Clerk (Claudate 现有)
- **数据库**: Cloudflare D1 (SQLite)
- **存储**: Cloudflare R2 (大文件)
- **AI**: Google Gemini 2.5 Flash (通过 OpenRouter)

## 任务目标

实现一个**与 Claudate 紧耦合的内部 Scraper 工具**，包含：

### 核心功能
1. **智能策略推荐** (基于 yfe404 方法论)
   - 侦察阶段：分析目标网站
   - 发现阶段：查找 API、sitemap
   - 策略阶段：AI 推荐最优抓取策略

2. **No-code 任务创建**
   - 团队成员无需写代码即可创建抓取任务
   - 可视化配置数据结构

3. **多策略执行引擎**
   - API 抓取（最快）
   - HTML 解析（中等）
   - 浏览器自动化（最慢，备用）

4. **任务监控和数据管理**
   - 实时任务状态
   - 数据预览和导出

## 实施计划

### Phase 1: 数据库设计 (Day 1)

创建数据库迁移文件：

**文件**: `database/migrations/005_scraper_tables.sql`

```sql
-- Scraper 任务配置表
CREATE TABLE IF NOT EXISTS scraper_tasks (
  id TEXT PRIMARY KEY,
  name TEXT NOT NULL,
  description TEXT,
  target_url TEXT NOT NULL,

  -- 策略配置
  strategy TEXT CHECK(strategy IN ('auto', 'api', 'sitemap_api', 'sitemap_html', 'browser')),
  strategy_config TEXT, -- JSON

  -- 数据结构定义
  data_schema TEXT NOT NULL, -- JSON: {"field": "selector/path"}

  -- 调度
  schedule_type TEXT CHECK(schedule_type IN ('manual', 'interval', 'cron')),
  schedule_config TEXT, -- JSON

  -- 元数据
  created_by TEXT NOT NULL, -- Clerk user ID
  created_at INTEGER NOT NULL,
  updated_at INTEGER NOT NULL,
  is_active BOOLEAN DEFAULT TRUE
);

-- Scraper 任务执行记录
CREATE TABLE IF NOT EXISTS scraper_jobs (
  id TEXT PRIMARY KEY,
  task_id TEXT NOT NULL REFERENCES scraper_tasks(id) ON DELETE CASCADE,

  -- 执行状态
  status TEXT CHECK(status IN ('pending', 'running', 'completed', 'failed', 'cancelled')),
  strategy_used TEXT,

  -- 时间
  started_at INTEGER,
  completed_at INTEGER,
  duration_ms INTEGER,

  -- 结果
  items_extracted INTEGER DEFAULT 0,
  items_failed INTEGER DEFAULT 0,
  data_location TEXT, -- R2 对象 key

  -- 错误处理
  error_message TEXT,
  retry_count INTEGER DEFAULT 0,

  -- 元数据
  triggered_by TEXT,
  created_at INTEGER NOT NULL
);

-- 策略分析历史
CREATE TABLE IF NOT EXISTS scraper_strategy_analyses (
  id TEXT PRIMARY KEY,
  task_id TEXT REFERENCES scraper_tasks(id) ON DELETE CASCADE,
  target_url TEXT NOT NULL,

  -- 发现结果
  has_api BOOLEAN DEFAULT FALSE,
  api_endpoints TEXT, -- JSON
  has_sitemap BOOLEAN DEFAULT FALSE,
  sitemap_url TEXT,

  -- AI 推荐
  recommended_strategy TEXT NOT NULL,
  confidence_score REAL,
  reasoning TEXT,
  estimated_performance TEXT,

  analyzed_at INTEGER NOT NULL
);

-- 创建索引
CREATE INDEX IF NOT EXISTS idx_scraper_tasks_user ON scraper_tasks(created_by);
CREATE INDEX IF NOT EXISTS idx_scraper_jobs_task ON scraper_jobs(task_id, created_at DESC);
CREATE INDEX IF NOT EXISTS idx_scraper_jobs_status ON scraper_jobs(status);
```

**验收标准**:
- [ ] 文件创建在正确位置
- [ ] 运行迁移成功: `wrangler d1 execute DB --file=database/migrations/005_scraper_tables.sql`
- [ ] 所有表创建成功

---

### Phase 2: 侦察引擎 (Reconnaissance) (Day 2-3)

**目标**: 实现第一阶段 - 分析目标网站

**文件**: `workers/scraper/reconnaissance.ts`

```typescript
/**
 * 侦察引擎 - 分析目标网站结构
 * 基于 yfe404/web-scraper 方法论的 Phase 1
 */

export interface ReconnaissanceResult {
  url: string;
  hasAPI: boolean;
  hasSitemap: boolean;
  hasRobotsTxt: boolean;
  detectedFramework: string | null;
  isJavaScriptHeavy: boolean;
  estimatedComplexity: 'low' | 'medium' | 'high';
  timestamp: number;
}

export class ReconnaissanceEngine {
  async analyze(targetUrl: string): Promise<ReconnaissanceResult> {
    const result: ReconnaissanceResult = {
      url: targetUrl,
      hasAPI: false,
      hasSitemap: false,
      hasRobotsTxt: false,
      detectedFramework: null,
      isJavaScriptHeavy: false,
      estimatedComplexity: 'low',
      timestamp: Date.now()
    };

    // 1. 检查 robots.txt
    try {
      const robotsUrl = new URL('/robots.txt', targetUrl).href;
      const robotsResponse = await fetch(robotsUrl);

      if (robotsResponse.ok) {
        result.hasRobotsTxt = true;
        const robotsTxt = await robotsResponse.text();

        // 从 robots.txt 查找 sitemap
        const sitemapMatch = robotsTxt.match(/Sitemap:\s*(.+)/i);
        if (sitemapMatch) {
          result.hasSitemap = true;
        }
      }
    } catch (e) {
      console.log('Failed to fetch robots.txt:', e);
    }

    // 2. 检查常见 sitemap 位置
    if (!result.hasSitemap) {
      result.hasSitemap = await this.checkCommonSitemapPaths(targetUrl);
    }

    // 3. 获取首页并分析
    try {
      const homeResponse = await fetch(targetUrl);
      const html = await homeResponse.text();

      // 检测框架
      result.detectedFramework = this.detectFramework(html);

      // 检测是否 JavaScript 重度
      result.isJavaScriptHeavy = this.isJavaScriptHeavy(html);

      // 查找 API 端点线索
      result.hasAPI = this.hasAPIHints(html);
    } catch (e) {
      console.error('Failed to fetch homepage:', e);
    }

    // 4. 评估复杂度
    result.estimatedComplexity = this.estimateComplexity(result);

    return result;
  }

  private async checkCommonSitemapPaths(baseUrl: string): Promise<boolean> {
    const paths = ['/sitemap.xml', '/sitemap_index.xml', '/sitemap1.xml'];

    for (const path of paths) {
      try {
        const url = new URL(path, baseUrl).href;
        const response = await fetch(url, { method: 'HEAD' });
        if (response.ok) return true;
      } catch (e) {
        // Continue to next path
      }
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
    // 移除所有脚本和样式标签
    const contentWithoutScripts = html
      .replace(/<script[\s\S]*?<\/script>/gi, '')
      .replace(/<style[\s\S]*?<\/style>/gi, '')
      .replace(/<[^>]+>/g, '')
      .trim();

    // 如果内容少于 500 字符，很可能是 SPA
    return contentWithoutScripts.length < 500;
  }

  private hasAPIHints(html: string): boolean {
    const apiPatterns = [
      /\/api\/v?\d+\//i,
      /\/graphql/i,
      /\/rest\/v?\d+\//i,
      /["']https?:\/\/[^"']+\/api\//i
    ];

    return apiPatterns.some(pattern => pattern.test(html));
  }

  private estimateComplexity(result: ReconnaissanceResult): 'low' | 'medium' | 'high' {
    let score = 0;

    if (result.hasAPI) score -= 2;
    if (result.hasSitemap) score -= 1;
    if (result.isJavaScriptHeavy) score += 3;
    if (result.detectedFramework) score += 1;

    if (score <= 0) return 'low';
    if (score <= 2) return 'medium';
    return 'high';
  }
}
```

**验收标准**:
- [ ] 文件创建成功
- [ ] 可以分析任意 URL
- [ ] 正确检测 sitemap、框架、复杂度
- [ ] 测试至少 5 个不同网站

**测试命令**:
```typescript
// 在 workers 中创建测试
const engine = new ReconnaissanceEngine();
const result = await engine.analyze('https://example.com');
console.log(result);
```

---

### Phase 3: 发现引擎 (Discovery) (Day 4-5)

**文件**: `workers/scraper/discovery.ts`

```typescript
/**
 * 发现引擎 - 查找 API 端点和数据源
 * 基于 yfe404/web-scraper 方法论的 Phase 2
 */

export interface APIEndpoint {
  url: string;
  method: string;
  responseType: 'json' | 'xml' | 'html';
  requiresAuth: boolean;
  sampleResponse?: any;
}

export interface Sitemap {
  url: string;
  totalUrls: number;
  urls: string[];
  isSitemapIndex: boolean;
}

export interface DiscoveryResult {
  apiEndpoints: APIEndpoint[];
  sitemaps: Sitemap[];
  timestamp: number;
}

export class DiscoveryEngine {
  async discover(targetUrl: string): Promise<DiscoveryResult> {
    const result: DiscoveryResult = {
      apiEndpoints: [],
      sitemaps: [],
      timestamp: Date.now()
    };

    // 1. 发现 API 端点
    result.apiEndpoints = await this.discoverAPIs(targetUrl);

    // 2. 发现 sitemaps
    result.sitemaps = await this.discoverSitemaps(targetUrl);

    return result;
  }

  private async discoverAPIs(baseUrl: string): Promise<APIEndpoint[]> {
    const endpoints: APIEndpoint[] = [];

    // 常见 API 路径
    const apiPaths = [
      '/api',
      '/api/v1',
      '/api/v2',
      '/graphql',
      '/rest/v1',
      '/__data.json',
      '/_next/data'
    ];

    for (const path of apiPaths) {
      try {
        const url = new URL(path, baseUrl).href;
        const response = await fetch(url);

        if (response.ok) {
          const contentType = response.headers.get('content-type') || '';

          endpoints.push({
            url,
            method: 'GET',
            responseType: this.detectResponseType(contentType),
            requiresAuth: false,
            sampleResponse: contentType.includes('json') ? await response.json() : null
          });
        }
      } catch (e) {
        // API 不存在，继续
      }
    }

    return endpoints;
  }

  private async discoverSitemaps(baseUrl: string): Promise<Sitemap[]> {
    const sitemaps: Sitemap[] = [];

    // 1. 从 robots.txt 查找
    try {
      const robotsUrl = new URL('/robots.txt', baseUrl).href;
      const robotsResponse = await fetch(robotsUrl);

      if (robotsResponse.ok) {
        const robotsTxt = await robotsResponse.text();
        const matches = robotsTxt.matchAll(/Sitemap:\s*(.+)/gi);

        for (const match of matches) {
          const sitemapUrl = match[1].trim();
          const sitemap = await this.parseSitemap(sitemapUrl);
          if (sitemap) sitemaps.push(sitemap);
        }
      }
    } catch (e) {
      console.log('Failed to check robots.txt');
    }

    // 2. 检查常见路径
    const commonPaths = ['/sitemap.xml', '/sitemap_index.xml'];
    for (const path of commonPaths) {
      try {
        const url = new URL(path, baseUrl).href;
        const sitemap = await this.parseSitemap(url);
        if (sitemap && !sitemaps.some(s => s.url === url)) {
          sitemaps.push(sitemap);
        }
      } catch (e) {
        // Continue
      }
    }

    return sitemaps;
  }

  private async parseSitemap(url: string): Promise<Sitemap | null> {
    try {
      const response = await fetch(url);
      if (!response.ok) return null;

      const xml = await response.text();
      const urlMatches = xml.matchAll(/<loc>(.*?)<\/loc>/g);
      const urls: string[] = [];

      for (const match of urlMatches) {
        urls.push(match[1]);
      }

      return {
        url,
        totalUrls: urls.length,
        urls: urls.slice(0, 100), // 前 100 个
        isSitemapIndex: xml.includes('<sitemapindex')
      };
    } catch (e) {
      return null;
    }
  }

  private detectResponseType(contentType: string): 'json' | 'xml' | 'html' {
    if (contentType.includes('json')) return 'json';
    if (contentType.includes('xml')) return 'xml';
    return 'html';
  }
}
```

**验收标准**:
- [ ] 可以发现 API 端点
- [ ] 可以发现并解析 sitemap
- [ ] 测试 5+ 不同网站

---

### Phase 4: 策略选择器 (AI-Powered) (Day 6-8)

**文件**: `workers/scraper/strategy.ts`

```typescript
/**
 * 策略选择器 - AI 驱动的最优策略推荐
 * 基于 yfe404/web-scraper 方法论的 Phase 3
 */

import type { ReconnaissanceResult } from './reconnaissance';
import type { DiscoveryResult } from './discovery';

export interface StrategyRecommendation {
  strategy: 'api' | 'sitemap_api' | 'sitemap_html' | 'browser';
  confidence: number;
  reasoning: string;
  estimatedSpeed: 'fast' | 'medium' | 'slow';
  estimatedCost: 'low' | 'medium' | 'high';
  implementation: {
    endpoints?: string[];
    selectors?: Record<string, string>;
  };
}

export class StrategySelector {
  private apiKey: string;

  constructor(apiKey: string) {
    this.apiKey = apiKey;
  }

  async recommend(
    targetUrl: string,
    recon: ReconnaissanceResult,
    discovery: DiscoveryResult,
    dataSchema: Record<string, string>
  ): Promise<StrategyRecommendation> {
    // 构建 AI prompt
    const prompt = this.buildPrompt(targetUrl, recon, discovery, dataSchema);

    // 调用 Gemini API
    const response = await this.callGemini(prompt);

    // 解析 AI 响应
    return this.parseResponse(response, discovery);
  }

  private buildPrompt(
    targetUrl: string,
    recon: ReconnaissanceResult,
    discovery: DiscoveryResult,
    dataSchema: Record<string, string>
  ): string {
    return `
你是一个网页抓取策略专家。分析以下网站并推荐最优抓取策略。

**目标 URL**: ${targetUrl}

**侦察结果**:
- 有 API: ${recon.hasAPI}
- 有 Sitemap: ${recon.hasSitemap}
- 检测到框架: ${recon.detectedFramework || '未知'}
- JavaScript 重度: ${recon.isJavaScriptHeavy}
- 复杂度: ${recon.estimatedComplexity}

**发现结果**:
- 发现 ${discovery.apiEndpoints.length} 个 API 端点
${discovery.apiEndpoints.map(e => `  - ${e.url} (${e.responseType})`).join('\n')}
- 发现 ${discovery.sitemaps.length} 个 Sitemap
${discovery.sitemaps.map(s => `  - ${s.url} (${s.totalUrls} URLs)`).join('\n')}

**要提取的数据结构**:
${JSON.stringify(dataSchema, null, 2)}

**可用策略** (按优先级):
1. **api**: 直接使用 API（最快、最可靠）
2. **sitemap_api**: Sitemap + API 组合（最优平衡）
3. **sitemap_html**: Sitemap + HTML 解析（中等速度）
4. **browser**: 浏览器自动化（最慢、最后手段）

**任务**:
推荐一个最适合的策略。考虑：
- 速度 (API > Sitemap+API > Sitemap+HTML > Browser)
- 可靠性 (API 最可靠)
- 复杂度 (越简单越好)
- 数据可用性 (能否获取所有需要的字段)

返回 JSON 格式:
{
  "strategy": "api|sitemap_api|sitemap_html|browser",
  "confidence": 0.85,
  "reasoning": "解释为什么选择这个策略",
  "estimatedSpeed": "fast|medium|slow",
  "estimatedCost": "low|medium|high"
}
`;
  }

  private async callGemini(prompt: string): Promise<string> {
    const response = await fetch('https://openrouter.ai/api/v1/chat/completions', {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${this.apiKey}`,
        'Content-Type': 'application/json',
        'HTTP-Referer': 'https://claudate.com',
        'X-Title': 'Claudate Scraper'
      },
      body: JSON.stringify({
        model: 'google/gemini-2.0-flash-exp:free',
        messages: [{ role: 'user', content: prompt }],
        temperature: 0.3
      })
    });

    if (!response.ok) {
      throw new Error(`Gemini API error: ${response.status}`);
    }

    const data = await response.json();
    return data.choices[0].message.content;
  }

  private parseResponse(
    aiResponse: string,
    discovery: DiscoveryResult
  ): StrategyRecommendation {
    try {
      // 尝试解析 JSON
      const jsonMatch = aiResponse.match(/\{[\s\S]*\}/);
      if (!jsonMatch) throw new Error('No JSON found');

      const parsed = JSON.parse(jsonMatch[0]);

      // 添加实现细节
      const implementation: any = {};
      if (parsed.strategy === 'api' || parsed.strategy === 'sitemap_api') {
        implementation.endpoints = discovery.apiEndpoints.map(e => e.url);
      }

      return {
        strategy: parsed.strategy,
        confidence: parsed.confidence,
        reasoning: parsed.reasoning,
        estimatedSpeed: parsed.estimatedSpeed,
        estimatedCost: parsed.estimatedCost,
        implementation
      };
    } catch (e) {
      // 如果解析失败，返回默认策略
      console.error('Failed to parse AI response:', e);

      // 简单规则备选
      if (discovery.apiEndpoints.length > 0) {
        return {
          strategy: 'api',
          confidence: 0.7,
          reasoning: '发现了 API 端点，直接使用 API 是最快的方法',
          estimatedSpeed: 'fast',
          estimatedCost: 'low',
          implementation: {
            endpoints: discovery.apiEndpoints.map(e => e.url)
          }
        };
      } else if (discovery.sitemaps.length > 0) {
        return {
          strategy: 'sitemap_html',
          confidence: 0.6,
          reasoning: '发现了 sitemap，可以用来获取所有页面 URL',
          estimatedSpeed: 'medium',
          estimatedCost: 'medium',
          implementation: {}
        };
      } else {
        return {
          strategy: 'browser',
          confidence: 0.5,
          reasoning: '未发现 API 或 sitemap，需要使用浏览器自动化',
          estimatedSpeed: 'slow',
          estimatedCost: 'high',
          implementation: {}
        };
      }
    }
  }
}
```

**验收标准**:
- [ ] 可以调用 Gemini API
- [ ] AI 返回有效的策略推荐
- [ ] 如果 AI 失败，有合理的备选逻辑
- [ ] 测试 5+ 不同网站

---

### Phase 5: API 路由 (Day 9-10)

**文件**: `app/api/scraper/analyze/route.ts`

```typescript
import { auth } from '@clerk/nextjs';
import { NextResponse } from 'next/server';
import { ReconnaissanceEngine } from '@/workers/scraper/reconnaissance';
import { DiscoveryEngine } from '@/workers/scraper/discovery';
import { StrategySelector } from '@/workers/scraper/strategy';

export async function POST(request: Request) {
  // 验证用户登录
  const { userId } = auth();
  if (!userId) {
    return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
  }

  try {
    const body = await request.json();
    const { targetUrl, dataSchema } = body;

    if (!targetUrl || !dataSchema) {
      return NextResponse.json(
        { error: 'Missing targetUrl or dataSchema' },
        { status: 400 }
      );
    }

    // Phase 1: 侦察
    const reconEngine = new ReconnaissanceEngine();
    const recon = await reconEngine.analyze(targetUrl);

    // Phase 2: 发现
    const discoveryEngine = new DiscoveryEngine();
    const discovery = await discoveryEngine.discover(targetUrl);

    // Phase 3: 策略推荐
    const selector = new StrategySelector(process.env.OPENROUTER_API_KEY!);
    const recommendation = await selector.recommend(
      targetUrl,
      recon,
      discovery,
      dataSchema
    );

    // 保存分析结果到 D1
    const analysisId = crypto.randomUUID();
    await env.DB.prepare(`
      INSERT INTO scraper_strategy_analyses (
        id, target_url, has_api, api_endpoints, has_sitemap,
        recommended_strategy, confidence_score, reasoning,
        estimated_performance, analyzed_at
      ) VALUES (?, ?, ?, ?, ?, ?, ?, ?, ?, ?)
    `).bind(
      analysisId,
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

    return NextResponse.json({
      reconnaissance: recon,
      discovery,
      recommendation
    });
  } catch (error: any) {
    console.error('Analysis error:', error);
    return NextResponse.json(
      { error: error.message },
      { status: 500 }
    );
  }
}
```

**验收标准**:
- [ ] API 路由创建成功
- [ ] 需要 Clerk 认证
- [ ] 可以分析任意 URL
- [ ] 返回完整的侦察、发现、策略推荐结果
- [ ] 结果保存到 D1 数据库

**测试**:
```bash
# 使用 curl 测试 (需要先登录 Claudate 获取 session token)
curl -X POST http://localhost:3000/api/scraper/analyze \
  -H "Content-Type: application/json" \
  -H "Cookie: __session=YOUR_SESSION_TOKEN" \
  -d '{
    "targetUrl": "https://example.com",
    "dataSchema": {
      "title": "text",
      "price": "number"
    }
  }'
```

---

### Phase 6: UI 界面 (Day 11-13)

**文件**: `app/scraper/page.tsx`

```typescript
'use client';

import { useState } from 'react';
import { useRouter } from 'next/navigation';

export default function ScraperPage() {
  const router = useRouter();
  const [analyzing, setAnalyzing] = useState(false);
  const [result, setResult] = useState<any>(null);

  const [form, setForm] = useState({
    targetUrl: '',
    dataSchema: {
      // 默认示例
      title: 'text',
      price: 'number'
    }
  });

  const handleAnalyze = async () => {
    setAnalyzing(true);
    setResult(null);

    try {
      const response = await fetch('/api/scraper/analyze', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(form)
      });

      if (!response.ok) {
        throw new Error('分析失败');
      }

      const data = await response.json();
      setResult(data);
    } catch (error: any) {
      alert('错误: ' + error.message);
    } finally {
      setAnalyzing(false);
    }
  };

  return (
    <div className="container mx-auto p-8">
      <h1 className="text-3xl font-bold mb-8">Scraper 工具</h1>

      {/* 输入表单 */}
      <div className="bg-white rounded-lg shadow p-6 mb-8">
        <h2 className="text-xl font-semibold mb-4">分析网站</h2>

        <div className="mb-4">
          <label className="block text-sm font-medium mb-2">
            目标 URL
          </label>
          <input
            type="url"
            className="w-full border rounded px-3 py-2"
            placeholder="https://example.com"
            value={form.targetUrl}
            onChange={(e) => setForm({ ...form, targetUrl: e.target.value })}
          />
        </div>

        <div className="mb-4">
          <label className="block text-sm font-medium mb-2">
            数据结构 (JSON)
          </label>
          <textarea
            className="w-full border rounded px-3 py-2 font-mono text-sm"
            rows={6}
            value={JSON.stringify(form.dataSchema, null, 2)}
            onChange={(e) => {
              try {
                const schema = JSON.parse(e.target.value);
                setForm({ ...form, dataSchema: schema });
              } catch (e) {
                // 无效 JSON，暂不更新
              }
            }}
          />
          <p className="text-xs text-gray-500 mt-1">
            示例: {"{"}"title": "text", "price": "number"{"}"}
          </p>
        </div>

        <button
          onClick={handleAnalyze}
          disabled={!form.targetUrl || analyzing}
          className="bg-blue-600 text-white px-6 py-2 rounded hover:bg-blue-700 disabled:opacity-50"
        >
          {analyzing ? '分析中...' : '开始分析'}
        </button>
      </div>

      {/* 分析结果 */}
      {result && (
        <div className="bg-white rounded-lg shadow p-6">
          <h2 className="text-xl font-semibold mb-4">分析结果</h2>

          {/* 侦察结果 */}
          <div className="mb-6">
            <h3 className="font-medium mb-2">侦察阶段</h3>
            <div className="bg-gray-50 rounded p-4 text-sm">
              <p>✓ 有 API: {result.reconnaissance.hasAPI ? '是' : '否'}</p>
              <p>✓ 有 Sitemap: {result.reconnaissance.hasSitemap ? '是' : '否'}</p>
              <p>✓ 检测到框架: {result.reconnaissance.detectedFramework || '无'}</p>
              <p>✓ 复杂度: {result.reconnaissance.estimatedComplexity}</p>
            </div>
          </div>

          {/* 发现结果 */}
          <div className="mb-6">
            <h3 className="font-medium mb-2">发现阶段</h3>
            <div className="bg-gray-50 rounded p-4 text-sm">
              <p>✓ 发现 {result.discovery.apiEndpoints.length} 个 API 端点</p>
              {result.discovery.apiEndpoints.map((ep: any, i: number) => (
                <p key={i} className="ml-4 text-xs text-gray-600">
                  - {ep.url} ({ep.responseType})
                </p>
              ))}
              <p className="mt-2">✓ 发现 {result.discovery.sitemaps.length} 个 Sitemap</p>
              {result.discovery.sitemaps.map((sm: any, i: number) => (
                <p key={i} className="ml-4 text-xs text-gray-600">
                  - {sm.url} ({sm.totalUrls} URLs)
                </p>
              ))}
            </div>
          </div>

          {/* 策略推荐 */}
          <div className="mb-6">
            <h3 className="font-medium mb-2">推荐策略</h3>
            <div className="bg-blue-50 rounded p-4">
              <div className="flex items-center justify-between mb-2">
                <span className="font-semibold text-lg">
                  {result.recommendation.strategy.toUpperCase()}
                </span>
                <span className="text-sm text-gray-600">
                  置信度: {(result.recommendation.confidence * 100).toFixed(0)}%
                </span>
              </div>
              <p className="text-sm mb-2">{result.recommendation.reasoning}</p>
              <div className="flex gap-4 text-xs text-gray-600">
                <span>速度: {result.recommendation.estimatedSpeed}</span>
                <span>成本: {result.recommendation.estimatedCost}</span>
              </div>
            </div>
          </div>

          <button
            onClick={() => {
              // TODO: 创建任务
              alert('创建任务功能待实现');
            }}
            className="bg-green-600 text-white px-6 py-2 rounded hover:bg-green-700"
          >
            创建抓取任务
          </button>
        </div>
      )}
    </div>
  );
}
```

**验收标准**:
- [ ] 页面可以访问: `http://localhost:3000/scraper`
- [ ] 只有登录用户可以访问（Clerk 保护）
- [ ] 可以输入 URL 和数据结构
- [ ] 点击分析后显示侦察、发现、策略推荐结果
- [ ] UI 美观、易用

---

### Phase 7: 测试和文档 (Day 14-15)

**创建测试文档**: `docs/SCRAPER_TESTING.md`

```markdown
# Scraper 工具测试指南

## 测试网站列表

请使用以下网站测试 Scraper 的智能分析能力：

1. **E-commerce (Shopify)**
   - URL: https://www.allbirds.com
   - 预期策略: sitemap_html
   - 原因: 有 sitemap，无公开 API

2. **新闻网站**
   - URL: https://techcrunch.com
   - 预期策略: sitemap_api
   - 原因: 有 sitemap 和 RSS (类似 API)

3. **API-first 网站**
   - URL: https://api.github.com
   - 预期策略: api
   - 原因: 纯 API

4. **JavaScript SPA**
   - URL: https://react.dev
   - 预期策略: browser
   - 原因: React SPA，内容动态加载

5. **静态网站**
   - URL: https://example.com
   - 预期策略: sitemap_html
   - 原因: 简单静态网站

## 测试步骤

1. **登录 Claudate**
   - 访问 http://localhost:3000
   - 使用 Clerk 登录

2. **访问 Scraper**
   - 导航到 http://localhost:3000/scraper

3. **分析网站**
   - 输入测试 URL
   - 定义数据结构，例如:
     ```json
     {
       "title": "text",
       "description": "text",
       "image": "url"
     }
     ```
   - 点击"开始分析"

4. **验证结果**
   - 检查侦察结果是否正确
   - 检查发现结果是否找到 API/Sitemap
   - 检查推荐策略是否合理
   - 置信度应该 >0.6

## 预期性能

- 分析时间: <5 秒
- 成功率: >90%
- AI 准确率: >80%
```

**验收标准**:
- [ ] 所有 5 个测试网站都能成功分析
- [ ] 策略推荐符合预期
- [ ] 性能达标（<5 秒）
- [ ] 文档完整清晰

---

## 最终交付清单

完成以上所有步骤后，你应该有：

### ✅ 代码文件
- [ ] `database/migrations/005_scraper_tables.sql` - 数据库表
- [ ] `workers/scraper/reconnaissance.ts` - 侦察引擎
- [ ] `workers/scraper/discovery.ts` - 发现引擎
- [ ] `workers/scraper/strategy.ts` - 策略选择器
- [ ] `app/api/scraper/analyze/route.ts` - API 路由
- [ ] `app/scraper/page.tsx` - UI 界面

### ✅ 功能验证
- [ ] 数据库表创建成功
- [ ] 侦察引擎工作正常（测试 5+ 网站）
- [ ] 发现引擎工作正常（测试 5+ 网站）
- [ ] 策略选择器工作正常（测试 5+ 网站）
- [ ] API 路由可以调用
- [ ] UI 界面可以访问和使用
- [ ] 需要 Clerk 登录（未登录会跳转）

### ✅ 文档
- [ ] `docs/SCRAPER_TESTING.md` - 测试指南
- [ ] 代码注释完整
- [ ] 每个函数都有 JSDoc

### ✅ 性能指标
- [ ] 分析时间 <5 秒
- [ ] AI 推荐准确率 >80%
- [ ] 无重大 bugs

---

## 后续阶段 (暂不实施)

完成 Phase 1-7 后，未来可以扩展：

- **执行引擎**: 实现实际的抓取功能（API/HTML/Browser）
- **任务调度**: Cron jobs 自动执行
- **数据导出**: CSV/JSON 导出
- **API 对外开放**: 独立产品化

但现在先专注完成 **智能分析和策略推荐** 这个核心功能。

---

## 需要帮助？

如果遇到问题：
1. 检查 console.log 输出
2. 检查数据库是否正确创建
3. 检查 API keys 是否配置正确
4. 查看 Cloudflare Workers 日志

祝开发顺利！🚀
```

---

## 💡 如何使用这份提示词？

### 第 1 步：准备工作

1. **打开 Claudate 项目文件夹**
2. **确保项目已经正确配置**:
   - Clerk 认证已设置
   - Cloudflare D1 数据库已创建
   - OpenRouter API key 已配置
3. **确认你在正确的分支**（如 `main` 或 `development`）

### 第 2 步：打开新的 Claude Code 终端

1. 打开一个**全新的 Claude Code 会话**
2. 在新会话中打开 **Claudate 项目文件夹**
3. 确保 Claude 可以访问所有文件

### 第 3 步：粘贴提示词

将上面的 **完整提示词**（从 "# Scraper 工具开发任务" 开始到最后）**完整复制粘贴** 给新的 Claude。

### 第 4 步：Claude 开始执行

Claude 会：
1. ✅ 阅读并理解任务
2. ✅ 按照 Phase 1-7 逐步执行
3. ✅ 创建所有必要的文件
4. ✅ 实现所有功能
5. ✅ 进行测试

### 第 5 步：验收

开发完成后，你需要验证：

1. **访问 UI**: http://localhost:3000/scraper
2. **测试分析功能**: 输入一个 URL（如 https://example.com）
3. **检查结果**: 应该看到侦察、发现、策略推荐三个部分
4. **验证准确性**: 推荐的策略应该合理

---

## 🎯 预期时间线

如果 Claude 全速工作：
- **Day 1-3**: 数据库 + 侦察引擎 + 发现引擎 ✅
- **Day 4-8**: 策略选择器（AI 集成）✅
- **Day 9-10**: API 路由 ✅
- **Day 11-13**: UI 界面 ✅
- **Day 14-15**: 测试和文档 ✅

**总计**: 约 2 周（10-15 个工作日）

---

## ✅ 成功标准

开发完成后，你应该能够：

1. **登录 Claudate**
2. **访问 `/scraper` 页面**
3. **输入任意 URL**（如 https://github.com）
4. **点击"开始分析"**
5. **看到完整的分析结果**:
   - 侦察阶段: 是否有 API、Sitemap、框架等
   - 发现阶段: 发现的 API 端点和 Sitemap
   - 策略推荐: AI 推荐的最优策略 + 理由
6. **策略推荐应该合理**（如有 API 就推荐 API 策略）

---

## 🚀 下一步

完成这个基础版本后，我们可以继续：

1. **Phase 2**: 实现执行引擎（实际抓取数据）
2. **Phase 3**: 添加任务调度（定时抓取）
3. **Phase 4**: 数据管理和导出
4. **Phase 5**: 解耦成独立产品

但现在先专注于 **智能分析和策略推荐** 这个核心功能！

---

**准备好了吗？**

复制上面的提示词，粘贴给新的 Claude Code，开始开发！🎉
