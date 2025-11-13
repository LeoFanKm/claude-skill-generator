# 修订版执行计划 - 基于新优先级

**版本**: 2.0 (修订版)
**日期**: 2025-11-13
**状态**: 🎯 基于用户反馈更新

---

## 🎯 新优先级顺序

根据用户反馈，优先级已调整为：

| 优先级 | 项目 | 类型 | 时间线 | 投资 | 原因 |
|--------|------|------|--------|------|------|
| **🥇 第一** | **Scraper API** | 基础设施 | 5 周 | $18K | 技术基础，支持其他项目 |
| **🥈 第二** | **病毒式内容仪表板** | 内部工具 | 4 周 | $20K | 依赖 Scraper，内部验证 |
| **🥉 第三** | **OnlyFans 插件** | 商业产品 | 6 周 | $45K | 商业化产品，最后实施 |

---

## 💡 战略优势分析

### 为什么这个顺序更好？

#### 1. **技术架构合理性** ✅

```
第一阶段: Scraper API (基础设施层)
    ↓
第二阶段: 病毒式内容仪表板 (应用层)
    ↓ (复用 Scraper)
第三阶段: OnlyFans 插件 (商业产品层)
```

**优势**:
- Scraper 提供数据采集能力
- 病毒式仪表板使用 Scraper 采集 Twitter/Reddit/TikTok 数据
- OnlyFans 插件可以复用 Scraper 的智能策略推荐能力

#### 2. **降低风险** ✅

| 阶段 | 风险等级 | 验证内容 |
|------|----------|----------|
| **Scraper** | 🟢 低 | 技术可行性、智能策略是否有效 |
| **病毒式仪表板** | 🟢 低 | Scraper 性能、内部团队使用反馈 |
| **OnlyFans 插件** | 🟡 中 | 已验证技术栈，降低商业化风险 |

#### 3. **内部验证循环** ✅

```
Week 1-5:  构建 Scraper
           ↓
Week 6-9:  构建病毒式仪表板，实战测试 Scraper
           ↓ (发现 bugs，优化性能)
Week 10:   Scraper 已经过实战验证，成熟稳定
           ↓
Week 11-16: 构建 OnlyFans 插件，复用已验证的技术
```

#### 4. **复用性最大化** ✅

**Scraper 核心能力可复用于**:
- ✅ 病毒式内容仪表板 (Twitter/Reddit/TikTok 采集)
- ✅ OnlyFans 插件 (可选：竞品分析、内容趋势采集)
- ✅ 未来任何需要数据采集的项目

---

## 📅 详细时间线 (顺序执行，单开发者)

### 总时间: 15 周 (约 3.75 个月)

```
┌─────────────────────────────────────────────────────────────┐
│ Phase 1: Scraper API (Weeks 1-5)                             │
│ ████████████████████████████████████████████                │
│ • Week 1-2: 基础设施 + 侦察/发现引擎                          │
│ • Week 3: AI 策略推荐                                        │
│ • Week 4: 执行引擎 (API/HTML/Browser)                        │
│ • Week 5: Dashboard UI + 测试                                │
│ 里程碑: ✅ Scraper API 内部可用                               │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│ Phase 2: 病毒式内容仪表板 (Weeks 6-9)                        │
│ ████████████████████████████████                            │
│ • Week 6: 集成 Scraper，构建数据采集 workers                 │
│ • Week 7: 病毒性评分算法 + AI 内容分析                       │
│ • Week 8: Dashboard UI (实时 feed、分析图表)                │
│ • Week 9: 测试 + 优化 + 内部发布                             │
│ 里程碑: ✅ 病毒式内容仪表板上线                               │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│ Phase 3: OnlyFans 插件 (Weeks 10-15)                        │
│ ████████████████████████████████████████████████████        │
│ • Week 10-11: 浏览器扩展基础 (MV3, content script)          │
│ • Week 12-13: AI 引擎 + 用户分析                             │
│ • Week 14: Popup UI + 测试                                   │
│ • Week 15: Beta 测试 + 准备发布                              │
│ 里程碑: ✅ OnlyFans 插件 MVP 完成                            │
└─────────────────────────────────────────────────────────────┘
```

---

## 🚀 Phase 1: Scraper API (Weeks 1-5)

### Week 1-2: 基础设施 + 核心引擎

**目标**: 搭建基础设施，实现侦察和发现引擎

#### Day 1-3: 基础设施
- [ ] 创建 monorepo 结构 (pnpm workspace)
- [ ] 配置 Cloudflare (D1, R2, Workers)
- [ ] 数据库迁移 (运行所有 SQL migrations)
- [ ] 设置 CI/CD (GitHub Actions + wrangler)

**交付物**:
```
internal-scraper/
├── packages/
│   ├── workers/
│   │   ├── strategy-recommender/
│   │   └── api/
│   └── dashboard/
├── database/
│   └── migrations/ (已运行)
└── wrangler.toml (已配置)
```

#### Day 4-7: 侦察引擎 (Reconnaissance)
- [ ] 实现 `ReconnaissanceEngine` 类
  - [ ] robots.txt 检查
  - [ ] sitemap 发现
  - [ ] 框架检测 (Next.js, Nuxt, React, etc.)
  - [ ] JavaScript 重度检测
- [ ] 编写单元测试 (5+ 测试网站)
- [ ] 部署到 Cloudflare Workers

**交付物**:
```typescript
// 可以调用的 API
POST /api/scrape/reconnaissance
Body: { "targetUrl": "https://example.com" }
Response: {
  "hasAPI": true,
  "hasSitemap": true,
  "detectedFramework": "Next.js",
  "estimatedComplexity": "medium"
}
```

#### Day 8-10: 发现引擎 (Discovery)
- [ ] 实现 `DiscoveryEngine` 类
  - [ ] API 端点发现
  - [ ] Sitemap 解析
  - [ ] RSS/Atom feed 发现
- [ ] 集成测试
- [ ] 部署

**交付物**:
```typescript
POST /api/scrape/discovery
Response: {
  "apiEndpoints": [
    { "url": "/api/v1/products", "method": "GET", "responseType": "json" }
  ],
  "sitemaps": [
    { "url": "/sitemap.xml", "totalUrls": 1247 }
  ]
}
```

---

### Week 3: AI 策略推荐

**目标**: 实现 AI 驱动的策略选择器

#### Day 1-3: 策略选择器
- [ ] 实现 `StrategySelector` 类
- [ ] 集成 Gemini API (通过 OpenRouter)
- [ ] 构建 AI prompt 模板
- [ ] 测试策略推荐准确性

#### Day 4-5: 实现指南生成器
- [ ] 为每种策略生成实现指南
  - [ ] API 策略: 端点列表
  - [ ] HTML 策略: CSS 选择器生成
  - [ ] Browser 策略: 操作步骤
- [ ] 测试 10+ 不同网站

**交付物**:
```typescript
POST /api/scrape/analyze
Body: {
  "targetUrl": "https://example.com",
  "dataSchema": {
    "title": "text",
    "price": "number",
    "image": "url"
  }
}
Response: {
  "recommendation": {
    "strategy": "sitemap_api",
    "confidence": 0.87,
    "reasoning": "Site has sitemap with 1,247 URLs and JSON API endpoints...",
    "estimatedSpeed": "fast",
    "implementation": {
      "endpoints": ["/api/v1/products"],
      "selectors": {
        "title": "h1.product-title",
        "price": "span.price",
        "image": "img.product-img"
      }
    }
  }
}
```

---

### Week 4: 执行引擎

**目标**: 实现 3 种 scraper workers

#### Day 1-2: API Scraper
- [ ] 实现 `api-scraper` worker
- [ ] 支持 JSON/XML/GraphQL
- [ ] 测试 5+ API endpoints

#### Day 3: HTML Parser
- [ ] 实现 `html-parser` worker
- [ ] 使用 Cheerio 解析 HTML
- [ ] CSS 选择器提取

#### Day 4-5: Browser Automation
- [ ] 实现 `browser-automation` worker
- [ ] 集成 Cloudflare Browser Rendering API
- [ ] 测试 JavaScript-heavy 网站

**交付物**:
```typescript
POST /api/scrape/execute
Body: {
  "taskId": "task-123",
  "strategy": "api"
}
Response: {
  "jobId": "job-456",
  "status": "pending"
}

// 稍后查询
GET /api/scrape/jobs/job-456
Response: {
  "status": "completed",
  "itemsExtracted": 1247,
  "duration": 8234, // ms
  "dataLocation": "r2://scraper-data/job-456.json"
}
```

---

### Week 5: Dashboard UI + 测试

**目标**: 构建内部 dashboard，端到端测试

#### Day 1-3: Dashboard UI
- [ ] 使用 Next.js + Tailwind 构建
- [ ] 任务创建界面 (no-code)
- [ ] 任务监控页面
- [ ] 数据浏览器

**界面设计**:
```
┌─────────────────────────────────────────────────────┐
│ Internal Scraper Dashboard                          │
├─────────────────────────────────────────────────────┤
│                                                     │
│ [+ Create New Task]   [View All Jobs]   [Docs]     │
│                                                     │
│ Active Tasks (5)                                    │
│ ┌─────────────────────────────────────────────┐   │
│ │ Product Prices - Example.com                │   │
│ │ Strategy: API | Last Run: 2 hours ago       │   │
│ │ Status: ✅ Success (1,247 items)            │   │
│ │ [Run Now] [View Data] [Edit]                │   │
│ └─────────────────────────────────────────────┘   │
│                                                     │
│ Recent Jobs (10)                                    │
│ • job-456: Completed in 8s (1,247 items) ✅       │
│ • job-455: Failed (Network error) ❌              │
│ • job-454: Completed in 145s (3,421 items) ✅     │
│                                                     │
└─────────────────────────────────────────────────────┘
```

#### Day 4-5: 端到端测试
- [ ] 测试 10+ 真实网站
- [ ] 性能基准测试
- [ ] Bug 修复
- [ ] 文档编写

**测试网站列表**:
1. E-commerce (Shopify 站点)
2. 新闻网站 (含 RSS)
3. SaaS 产品页面
4. API-first 网站
5. JavaScript-heavy SPA
6. 静态生成网站
7. 多语言网站
8. 需要认证的网站
9. 分页内容网站
10. 动态加载内容网站

**Week 5 交付里程碑**: ✅ **Scraper API 完全可用**

---

## 🎨 Phase 2: 病毒式内容仪表板 (Weeks 6-9)

### Week 6: 集成 Scraper + 数据采集

**目标**: 复用 Scraper API 构建平台特定的 scrapers

#### Day 1-2: Twitter Scraper
- [ ] 使用 Scraper API 分析 Twitter
- [ ] 实现 Twitter API 客户端
- [ ] 配置 cron (每 30 分钟)
- [ ] 测试数据采集

**实现方式**:
```typescript
// 复用 Scraper 的策略推荐能力
const recommendation = await scraper.analyze('https://api.twitter.com/2/tweets/search/recent');

// 基于推荐实现 Twitter scraper
if (recommendation.strategy === 'api') {
  // 使用 Twitter API v2
  const tweets = await scraper.execute({
    strategy: 'api',
    endpoint: 'https://api.twitter.com/2/tweets/search/recent',
    params: { query: 'AI tools', max_results: 100 }
  });
}
```

#### Day 3: Reddit Scraper
- [ ] Reddit API 集成
- [ ] 监控目标 subreddits
- [ ] Cron 配置 (每 20 分钟)

#### Day 4-5: TikTok Scraper
- [ ] TikTok unofficial API 或 Apify
- [ ] Hashtag 监控
- [ ] Cron 配置 (每小时)

**Week 6 交付物**: ✅ 3 个平台数据采集 workers 运行中

---

### Week 7: 病毒性评分 + AI 分析

**目标**: 实现病毒性评分算法和内容分析

#### Day 1-3: 病毒性评分器
- [ ] 实现 `ViralityScorer` 类
- [ ] 计算参与度、增长速度、作者影响力
- [ ] 历史数据跟踪 (每小时快照)

#### Day 4-5: AI 内容分析器
- [ ] 使用 Gemini 提取主题
- [ ] 情感分析
- [ ] 内容格式分类 (thread, meme, tutorial, etc.)
- [ ] Hook 类型识别

**Week 7 交付物**: ✅ 每条内容都有病毒性评分 (0-100) 和 AI 洞察

---

### Week 8: Dashboard UI

**目标**: 构建实时病毒内容 feed

#### Day 1-3: 首页 (Trending Feed)
- [ ] 内容卡片组件 (平台徽章、病毒性评分、指标)
- [ ] 过滤器 (平台、主题、时间)
- [ ] 排序 (病毒性、参与度、最新)
- [ ] 实时更新 (WebSocket 或轮询)

#### Day 4-5: 分析页面
- [ ] 趋势图表 (病毒性随时间变化)
- [ ] 热门主题排行榜
- [ ] 平台对比分析
- [ ] 导出功能 (CSV/JSON)

**Week 8 交付物**: ✅ 功能完整的 dashboard UI

---

### Week 9: 测试 + 优化 + 发布

**目标**: 完善并内部发布

#### Day 1-2: 内部测试
- [ ] 团队成员测试 (3-5 人)
- [ ] 收集反馈
- [ ] Bug 修复

#### Day 3-4: 性能优化
- [ ] 数据库查询优化 (添加索引)
- [ ] 缓存策略 (KV for hot data)
- [ ] Worker 性能调优

#### Day 5: 内部发布
- [ ] 编写使用文档
- [ ] 团队培训 (如何使用)
- [ ] 正式上线

**Week 9 里程碑**: ✅ **病毒式内容仪表板内部上线**

**成功指标**:
- ✅ 跟踪 5,000+ 内容项
- ✅ 每天新增 200+ 项
- ✅ 3+ 团队成员日活
- ✅ 80%+ 趋势预测准确率

---

## 🎮 Phase 3: OnlyFans 插件 (Weeks 10-15)

**注**: 此阶段在 Week 10 开始，此时 Scraper 已经过 **实战验证** (通过病毒式仪表板的使用)

### Week 10-11: 浏览器扩展基础

**目标**: 实现 Chrome Extension MV3 基础架构

#### Week 10: Content Script + DOM 集成
- [ ] Manifest V3 配置
- [ ] Content script 注入
- [ ] OnlyFans DOM observer
- [ ] UI 注入 (AI 助手面板)
- [ ] Message passing (content ↔ background)

#### Week 11: Background Service Worker
- [ ] Service worker 设置
- [ ] IndexedDB 存储管理
- [ ] Chrome Storage API 集成
- [ ] 消息路由

**Week 11 交付物**: ✅ 扩展可以注入 OnlyFans 并显示 UI

---

### Week 12-13: AI 引擎 + 用户分析

**目标**: 实现核心 AI 功能

#### Week 12: AI 引擎
- [ ] Gemini API 集成
- [ ] 用户分析 prompts
- [ ] 回复生成 prompts
- [ ] 主题建议 prompts
- [ ] 测试 AI 质量

**复用 Scraper 的 AI 能力**:
```typescript
// Scraper 已经有成熟的 AI prompt 工程
import { AIEngine } from '@internal-scraper/ai-engine';

// 复用相同的 Gemini 集成
const aiEngine = new AIEngine(env.GEMINI_API_KEY);
const insights = await aiEngine.analyze(userAnalysisPrompt);
```

#### Week 13: 用户分析器
- [ ] 行为模式分析
- [ ] 主题提取
- [ ] 最佳响应时间计算
- [ ] 参与度评分

**Week 13 交付物**: ✅ AI 可以分析粉丝并生成建议

---

### Week 14: Popup UI + 模板引擎

**目标**: 完善扩展 UI

#### Day 1-3: Popup 界面
- [ ] React 设置页面
- [ ] 分析仪表板
- [ ] 模板管理器
- [ ] 模式选择器 (破冰/调情/促活/促成交)

#### Day 4-5: 模板引擎
- [ ] 默认模板库 (每种模式 5+ 模板)
- [ ] 个性化变量替换
- [ ] 自定义模板创建

**Week 14 交付物**: ✅ 功能完整的 MVP

---

### Week 15: Beta 测试 + 发布准备

**目标**: 验证并准备发布

#### Day 1-2: 内部测试
- [ ] 在真实 OnlyFans 账户上测试
- [ ] Bug 修复
- [ ] UI 抛光

#### Day 3-4: Beta 测试
- [ ] 招募 10-20 OnlyFans creators
- [ ] 发放测试版本
- [ ] 收集反馈
- [ ] 迭代改进

#### Day 5: 发布准备
- [ ] 编写隐私政策
- [ ] 创建 Chrome Web Store 列表
- [ ] 准备营销材料
- [ ] 设置支付集成 (Stripe)

**Week 15 里程碑**: ✅ **OnlyFans 插件 MVP 完成**

---

## 💰 投资明细 (顺序执行，单开发者)

### 开发成本

| 阶段 | 周数 | 费率 | 总成本 |
|------|------|------|--------|
| **Phase 1: Scraper** | 5 周 | $100/hr × 40hr/wk | $20,000 |
| **Phase 2: 病毒式仪表板** | 4 周 | $100/hr × 40hr/wk | $16,000 |
| **Phase 3: OnlyFans 插件** | 6 周 | $100/hr × 40hr/wk | $24,000 |
| **总开发成本** | **15 周** | - | **$60,000** |

### 其他成本

| 类别 | 成本 | 备注 |
|------|------|------|
| **设计** | $3,000 | UI/UX (主要用于 OnlyFans 插件) |
| **法律** | $2,000 | 隐私政策、ToS (主要用于 OnlyFans) |
| **营销** | $5,000 | OnlyFans 插件营销材料 |
| **基础设施** | $1,000 | Cloudflare、域名 (前 3 个月) |
| **缓冲** | $7,000 | 10% 应急费用 |
| **其他总计** | **$18,000** | |

### 总投资

**总计**: **$78,000** (15 周，单开发者)

---

## ⚡ 加速选项 (2 开发者并行)

如果您想更快完成，可以并行开发：

### 并行时间线 (9 周总计)

```
Weeks 1-5:
  Dev 1: Scraper API                    ████████████ 完成

Weeks 6-9:
  Dev 1: 病毒式仪表板                    ████████ 完成
  Dev 2: OnlyFans 插件 (前期)           ████████ 部分完成

Weeks 10-11:
  Dev 2: OnlyFans 插件 (收尾)           ████ 完成
```

**时间节省**: 6 周 → 4 周节省 (从 15 周减少到 11 周)
**额外成本**: +$24K (Dev 2 在 weeks 6-11 工作)
**总投资**: $102K (vs $78K 顺序执行)

---

## 📊 成本效益对比

### 方案 A: 顺序执行 (推荐) ⭐

| 指标 | 数值 |
|------|------|
| **总投资** | $78K |
| **时间线** | 15 周 (3.75 个月) |
| **风险** | 🟢 低 (逐步验证) |
| **资源** | 1 开发者 |
| **首次收入** | Month 4 (OnlyFans 插件上线) |

**适合**:
- ✅ 预算有限 (<$100K)
- ✅ 单一技术创始人
- ✅ 稳健优先

### 方案 B: 部分并行

| 指标 | 数值 |
|------|------|
| **总投资** | $102K |
| **时间线** | 11 周 (2.75 个月) |
| **风险** | 🟡 中 (需要协调) |
| **资源** | 2 开发者 (后期) |
| **首次收入** | Month 3 (提前 1 个月) |

**适合**:
- ✅ 预算充足 ($100K+)
- ✅ 想要加速但保持验证循环
- ✅ 平衡风险和速度

---

## ✅ 关键验证点

### Scraper API 验证 (Week 5)

**成功标准**:
- ✅ 可以分析 10+ 不同类型网站
- ✅ AI 策略推荐准确率 >80%
- ✅ API scraper 比 browser 快 40-60x
- ✅ Dashboard UI 易于使用 (内部团队反馈)

**如果失败**: 重新评估，可能需要额外 1-2 周

---

### 病毒式仪表板验证 (Week 9)

**成功标准**:
- ✅ 每天采集 200+ 新内容
- ✅ 团队实际使用 (3+ 日活)
- ✅ 识别到 5+ 可操作洞察/周
- ✅ Scraper 性能稳定 (通过实战测试)

**如果失败**: OnlyFans 插件可能需要调整 scraper 部分

---

### OnlyFans 插件验证 (Week 15)

**成功标准**:
- ✅ 10+ beta 测试者
- ✅ 4.0+ 平均评分
- ✅ <5% 错误率
- ✅ AI 建议被采纳率 >40%

**如果失败**: 延长 beta 测试，迭代改进

---

## 📈 收入时间线 (方案 A)

```
Month 1-3: $0 (开发阶段)
           └─ Scraper 完成 (Month 1.25)
           └─ 病毒式仪表板完成 (Month 2.25)

Month 4:   $694 MRR (OnlyFans 插件上线)
           └─ 10 Creator tier + 3 Pro + 1 Agency

Month 6:   $8,470 MRR
           └─ 100 Creator + 20 Pro + 5 Agency

Month 9:   $20,000 MRR (盈亏平衡: 累计收入 = $78K 投资)

Month 12:  $34,380 MRR ($412K ARR)
           └─ 500 Creator + 100 Pro + 20 Agency
```

**盈亏平衡**: Month 9 (从 OnlyFans 插件上线算起是 5-6 个月)

---

## 🎯 下一步行动

### 本周 (Week 0)

**决策**:
- [ ] 确认顺序执行方案 (推荐)
- [ ] 确认预算 ($78K)
- [ ] 确认时间线 (15 周可接受)

**准备**:
- [ ] 招募/分配开发者 (1 名全栈)
- [ ] 设置 Cloudflare 账户
- [ ] 注册域名 (如需要)
- [ ] 准备开发环境

### Week 1 (第一天)

**启动 Scraper API 开发**:
- [ ] 创建 repo + monorepo 结构
- [ ] 配置 wrangler.toml
- [ ] 运行数据库迁移
- [ ] 开始编写 reconnaissance 引擎

---

## 📚 参考文档

所有技术规范文档仍然有效：

1. **INTERNAL_SCRAPER_API_ARCHITECTURE.md** (2,200 行) - 详细技术规范
2. **VIRAL_CONTENT_DASHBOARD_ARCHITECTURE.md** (1,800 行) - 详细技术规范
3. **ONLYFANS_PLUGIN_ARCHITECTURE.md** (2,400 行) - 详细技术规范
4. **MASTER_IMPLEMENTATION_ROADMAP.md** (1,500 行) - 原始战略分析

---

## ✅ 修订版计划状态

**状态**: 🎯 **已更新，准备执行**

**变更**:
- ✅ 优先级重新排序 (Scraper → 病毒式 → OnlyFans)
- ✅ 时间线调整为顺序执行
- ✅ 强调技术验证循环
- ✅ 复用策略明确化

**优势**:
- ✅ 更稳健的技术路径
- ✅ 降低商业化风险
- ✅ 更好的内部验证
- ✅ 技术复用最大化

---

**准备好开始了！** 🚀

**下一步**: 确认方案 → 招募开发者 → Week 1 启动 Scraper API 开发

---

**文档版本**: 2.0 (修订版)
**创建日期**: 2025-11-13
**修订原因**: 基于用户反馈调整优先级
