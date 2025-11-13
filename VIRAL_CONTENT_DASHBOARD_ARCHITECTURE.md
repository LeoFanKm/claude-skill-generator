# Viral Content Intelligence Dashboard - Technical Architecture

**Version**: 1.0
**Date**: 2025-11-13
**Status**: 🎯 Ready for Implementation
**Purpose**: Internal tool for tracking viral marketing material
**Timeline**: 4 weeks (MVP)

---

## 🎯 Executive Summary

### What We're Building

An **internal dashboard** for the Claudate.com team to monitor and analyze viral content trends across Twitter, Reddit, and TikTok in real-time. This tool identifies emerging viral marketing materials, tracks their propagation, and provides actionable insights for marketing strategy.

### Core Use Cases

1. **Trend Detection**: Automatically identify rising content before it peaks
2. **Competitive Intelligence**: Monitor what content resonates in our space (AI, productivity, SaaS)
3. **Content Inspiration**: Discover viral formats, hooks, and narratives
4. **Timing Optimization**: Learn when to post for maximum reach
5. **Campaign Planning**: Data-driven decision making for marketing initiatives

### Key Differentiation

**Internal tool first** - No user-facing complexity, no billing, no support burden. Pure utility for Claudate team to make better marketing decisions.

---

## 🏗️ System Architecture

### High-Level Overview

```
┌─────────────────────────────────────────────────────────────┐
│                   Data Collection Layer                       │
├─────────────────────────────────────────────────────────────┤
│                                                               │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │   Twitter    │  │   Reddit     │  │   TikTok     │      │
│  │   Scraper    │  │   Scraper    │  │   Scraper    │      │
│  │  (Worker)    │  │  (Worker)    │  │  (Worker)    │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
│         │                 │                  │               │
│         └─────────────────┴──────────────────┘               │
│                           ↓                                   │
└─────────────────────────────────────────────────────────────┘
                           ↓
┌─────────────────────────────────────────────────────────────┐
│                  Processing & Analysis Layer                  │
├─────────────────────────────────────────────────────────────┤
│                                                               │
│  ┌──────────────────────────────────────────────────────┐   │
│  │             Virality Scorer (Worker)                  │   │
│  │  • Engagement rate calculation                        │   │
│  │  • Growth velocity detection                          │   │
│  │  • Anomaly detection (sudden spikes)                  │   │
│  └──────────────────────────────────────────────────────┘   │
│                           ↓                                   │
│  ┌──────────────────────────────────────────────────────┐   │
│  │           Content Analyzer (AI + Worker)              │   │
│  │  • Topic extraction (Gemini API)                      │   │
│  │  • Sentiment analysis                                 │   │
│  │  • Format classification                              │   │
│  │  • Hook identification                                │   │
│  └──────────────────────────────────────────────────────┘   │
│                           ↓                                   │
└─────────────────────────────────────────────────────────────┘
                           ↓
┌─────────────────────────────────────────────────────────────┐
│                      Storage Layer                            │
├─────────────────────────────────────────────────────────────┤
│                                                               │
│  ┌──────────────────────┐  ┌──────────────────────┐         │
│  │  D1 Database         │  │  R2 Storage          │         │
│  │  • Content metadata  │  │  • Media files       │         │
│  │  • Metrics history   │  │  • Screenshots       │         │
│  │  • Analysis results  │  │  • Thumbnails        │         │
│  └──────────────────────┘  └──────────────────────┘         │
│                                                               │
└─────────────────────────────────────────────────────────────┘
                           ↓
┌─────────────────────────────────────────────────────────────┐
│                    Presentation Layer                         │
├─────────────────────────────────────────────────────────────┤
│                                                               │
│  ┌──────────────────────────────────────────────────────┐   │
│  │         Dashboard UI (Cloudflare Pages)               │   │
│  │  • Real-time feed (trending content)                  │   │
│  │  • Analytics charts (engagement, growth)              │   │
│  │  • Search & filters (platform, topic, date)           │   │
│  │  • Saved items (bookmarks, collections)               │   │
│  └──────────────────────────────────────────────────────┘   │
│                                                               │
└─────────────────────────────────────────────────────────────┘
```

---

## 📁 File Structure

```
viral-content-dashboard/
├── workers/
│   ├── scraper-twitter/              # Twitter scraping worker
│   │   ├── src/
│   │   │   ├── index.ts              # Worker entry point
│   │   │   ├── twitter-client.ts     # Twitter API wrapper
│   │   │   ├── scheduler.ts          # Cron job handler
│   │   │   └── types.ts              # TypeScript interfaces
│   │   ├── wrangler.toml
│   │   └── package.json
│   │
│   ├── scraper-reddit/               # Reddit scraping worker
│   │   ├── src/
│   │   │   ├── index.ts
│   │   │   ├── reddit-client.ts      # Reddit API wrapper
│   │   │   └── subreddit-monitor.ts  # Track specific subreddits
│   │   └── wrangler.toml
│   │
│   ├── scraper-tiktok/               # TikTok scraping worker
│   │   ├── src/
│   │   │   ├── index.ts
│   │   │   ├── tiktok-client.ts      # TikTok unofficial API
│   │   │   └── hashtag-monitor.ts    # Track hashtags
│   │   └── wrangler.toml
│   │
│   ├── analyzer/                     # Content analysis worker
│   │   ├── src/
│   │   │   ├── index.ts
│   │   │   ├── virality-scorer.ts    # Calculate virality score
│   │   │   ├── ai-analyzer.ts        # Gemini API integration
│   │   │   ├── topic-extractor.ts    # Topic extraction
│   │   │   └── format-classifier.ts  # Classify content format
│   │   └── wrangler.toml
│   │
│   └── api/                          # API for frontend
│       ├── src/
│       │   ├── index.ts
│       │   ├── routes/
│       │   │   ├── content.ts        # GET /api/content
│       │   │   ├── trending.ts       # GET /api/trending
│       │   │   ├── search.ts         # GET /api/search
│       │   │   └── analytics.ts      # GET /api/analytics
│       │   └── middleware/
│       │       └── auth.ts           # Internal auth only
│       └── wrangler.toml
│
├── dashboard/                        # Frontend (Cloudflare Pages)
│   ├── src/
│   │   ├── pages/
│   │   │   ├── index.tsx             # Home/trending feed
│   │   │   ├── analytics.tsx         # Analytics dashboard
│   │   │   ├── search.tsx            # Search interface
│   │   │   └── saved.tsx             # Bookmarked content
│   │   ├── components/
│   │   │   ├── ContentCard.tsx       # Content item display
│   │   │   ├── TrendChart.tsx        # Virality chart
│   │   │   ├── PlatformFilter.tsx    # Filter by platform
│   │   │   └── MetricsPanel.tsx      # Key metrics display
│   │   ├── lib/
│   │   │   ├── api.ts                # API client
│   │   │   └── utils.ts              # Helpers
│   │   └── styles/
│   │       └── globals.css
│   ├── public/
│   ├── package.json
│   └── next.config.js                # Next.js config
│
├── database/
│   ├── migrations/
│   │   ├── 001_init_viral_content.sql
│   │   └── 002_add_indexes.sql
│   └── schema.sql                    # Complete schema
│
├── shared/
│   ├── types.ts                      # Shared TypeScript types
│   └── constants.ts                  # App-wide constants
│
└── docs/
    ├── API.md                        # API documentation
    ├── SCRAPING_STRATEGY.md          # Scraping methodology
    └── DEPLOYMENT.md                 # Deployment guide
```

**Total Estimated Code**: ~2,800 lines TypeScript

---

## 🗄️ Database Schema

### D1 Database Structure

```sql
-- migrations/001_init_viral_content.sql

-- Main content table
CREATE TABLE viral_content (
  id TEXT PRIMARY KEY,
  platform TEXT NOT NULL CHECK(platform IN ('twitter', 'reddit', 'tiktok')),
  platform_id TEXT NOT NULL, -- Original post ID from platform
  author_username TEXT NOT NULL,
  author_followers INTEGER,
  content_text TEXT,
  content_type TEXT CHECK(content_type IN ('text', 'image', 'video', 'link', 'mixed')),
  url TEXT NOT NULL,
  media_urls TEXT, -- JSON array of media URLs

  -- Engagement metrics (snapshot)
  likes INTEGER DEFAULT 0,
  shares INTEGER DEFAULT 0,
  comments INTEGER DEFAULT 0,
  views INTEGER DEFAULT 0,

  -- Virality metrics (calculated)
  virality_score REAL DEFAULT 0.0,
  growth_velocity REAL DEFAULT 0.0, -- Engagement per hour
  peak_engagement INTEGER DEFAULT 0,

  -- AI analysis results
  topics TEXT, -- JSON array ["AI", "productivity", "SaaS"]
  sentiment TEXT CHECK(sentiment IN ('positive', 'negative', 'neutral')),
  content_format TEXT, -- "thread", "meme", "tutorial", "announcement", etc.
  hook_type TEXT, -- "question", "stat", "story", "controversy", etc.
  key_insights TEXT, -- AI-generated summary

  -- Timestamps
  posted_at INTEGER NOT NULL,
  discovered_at INTEGER NOT NULL,
  analyzed_at INTEGER,

  -- Flags
  is_trending BOOLEAN DEFAULT FALSE,
  is_saved BOOLEAN DEFAULT FALSE,

  UNIQUE(platform, platform_id)
);

CREATE INDEX idx_viral_platform ON viral_content(platform);
CREATE INDEX idx_viral_trending ON viral_content(is_trending, virality_score DESC);
CREATE INDEX idx_viral_posted_at ON viral_content(posted_at DESC);
CREATE INDEX idx_viral_score ON viral_content(virality_score DESC);

-- Metrics history (time-series data)
CREATE TABLE content_metrics_history (
  id TEXT PRIMARY KEY,
  content_id TEXT NOT NULL REFERENCES viral_content(id) ON DELETE CASCADE,

  -- Metrics snapshot
  likes INTEGER NOT NULL,
  shares INTEGER NOT NULL,
  comments INTEGER NOT NULL,
  views INTEGER NOT NULL,

  -- Calculated
  engagement_rate REAL, -- (likes + shares + comments) / views
  growth_since_last REAL, -- % increase since last snapshot

  recorded_at INTEGER NOT NULL
);

CREATE INDEX idx_metrics_content ON content_metrics_history(content_id, recorded_at DESC);

-- Trending topics (aggregated view)
CREATE TABLE trending_topics (
  id TEXT PRIMARY KEY,
  topic_name TEXT NOT NULL,
  platform TEXT,

  -- Aggregates
  content_count INTEGER DEFAULT 0,
  total_engagement INTEGER DEFAULT 0,
  avg_virality_score REAL DEFAULT 0.0,

  -- Trend direction
  trend_direction TEXT CHECK(trend_direction IN ('rising', 'stable', 'falling')),

  -- Time window
  window_start INTEGER NOT NULL,
  window_end INTEGER NOT NULL,

  updated_at INTEGER NOT NULL
);

CREATE INDEX idx_topics_platform ON trending_topics(platform, avg_virality_score DESC);

-- Saved collections (for team members to bookmark interesting content)
CREATE TABLE saved_collections (
  id TEXT PRIMARY KEY,
  name TEXT NOT NULL,
  description TEXT,
  created_by TEXT, -- Team member email/ID
  created_at INTEGER NOT NULL
);

CREATE TABLE saved_collection_items (
  collection_id TEXT NOT NULL REFERENCES saved_collections(id) ON DELETE CASCADE,
  content_id TEXT NOT NULL REFERENCES viral_content(id) ON DELETE CASCADE,
  notes TEXT,
  added_at INTEGER NOT NULL,

  PRIMARY KEY (collection_id, content_id)
);

-- Scraper status (track scraper health)
CREATE TABLE scraper_status (
  platform TEXT PRIMARY KEY,
  last_run INTEGER NOT NULL,
  status TEXT CHECK(status IN ('success', 'error', 'running')),
  error_message TEXT,
  items_scraped INTEGER DEFAULT 0,
  updated_at INTEGER NOT NULL
);
```

**Storage Pattern**:
- **D1**: Metadata + metrics (<1KB per row, fast queries)
- **R2**: Media files (images, videos, thumbnails) + raw JSON dumps

---

## 🔍 Data Collection Strategy

### Twitter Scraping

**Approach**: Use Twitter API v2 (Free tier: 1,500 tweets/month) + Apify Twitter Scraper (backup)

```typescript
// workers/scraper-twitter/src/twitter-client.ts

import { TwitterApi } from 'twitter-api-v2';

interface TwitterScraperConfig {
  keywords: string[];
  hashtags: string[];
  accounts: string[];
  minLikes: number;
  minRetweets: number;
}

class TwitterScraper {
  private client: TwitterApi;
  private config: TwitterScraperConfig;

  constructor(bearerToken: string, config: TwitterScraperConfig) {
    this.client = new TwitterApi(bearerToken);
    this.config = config;
  }

  async scrapeRecentTweets(): Promise<Tweet[]> {
    const tweets: Tweet[] = [];

    // Search by keywords
    for (const keyword of this.config.keywords) {
      const searchResults = await this.client.v2.search(keyword, {
        max_results: 100,
        'tweet.fields': ['public_metrics', 'created_at', 'author_id'],
        'user.fields': ['username', 'public_metrics'],
        expansions: ['author_id', 'attachments.media_keys'],
        'media.fields': ['url', 'preview_image_url']
      });

      for await (const tweet of searchResults) {
        if (this.meetsThreshold(tweet)) {
          tweets.push(this.transformTweet(tweet));
        }
      }
    }

    // Monitor specific accounts
    for (const account of this.config.accounts) {
      const userTweets = await this.getUserRecentTweets(account);
      tweets.push(...userTweets);
    }

    return tweets;
  }

  private meetsThreshold(tweet: any): boolean {
    const metrics = tweet.public_metrics;
    return (
      metrics.like_count >= this.config.minLikes &&
      metrics.retweet_count >= this.config.minRetweets
    );
  }

  private transformTweet(raw: any): Tweet {
    return {
      platform: 'twitter',
      platformId: raw.id,
      authorUsername: raw.author?.username || 'unknown',
      authorFollowers: raw.author?.public_metrics?.followers_count || 0,
      contentText: raw.text,
      contentType: this.detectContentType(raw),
      url: `https://twitter.com/${raw.author?.username}/status/${raw.id}`,
      mediaUrls: raw.attachments?.media?.map((m: any) => m.url) || [],
      likes: raw.public_metrics.like_count,
      shares: raw.public_metrics.retweet_count,
      comments: raw.public_metrics.reply_count,
      views: raw.public_metrics.impression_count || 0,
      postedAt: new Date(raw.created_at).getTime()
    };
  }

  private detectContentType(tweet: any): ContentType {
    if (tweet.attachments?.media) {
      const hasVideo = tweet.attachments.media.some((m: any) => m.type === 'video');
      const hasImage = tweet.attachments.media.some((m: any) => m.type === 'photo');
      if (hasVideo && hasImage) return 'mixed';
      if (hasVideo) return 'video';
      if (hasImage) return 'image';
    }
    if (tweet.entities?.urls?.length > 0) return 'link';
    return 'text';
  }
}

// Cron trigger (every 30 minutes)
export default {
  async scheduled(event: ScheduledEvent, env: Env, ctx: ExecutionContext) {
    const scraper = new TwitterScraper(env.TWITTER_BEARER_TOKEN, {
      keywords: ['AI tools', 'Claude AI', 'productivity', 'no-code'],
      hashtags: ['#AITools', '#ProductivityHacks', '#SaaS'],
      accounts: ['OpenAI', 'AnthropicAI', 'levelsio', 'danshipper'],
      minLikes: 100,
      minRetweets: 20
    });

    const tweets = await scraper.scrapeRecentTweets();

    // Store in D1
    await ctx.waitUntil(storeTweets(env.DB, tweets));

    // Trigger analysis worker
    for (const tweet of tweets) {
      await env.ANALYZER_QUEUE.send({ contentId: tweet.id });
    }
  }
};
```

**Rate Limits**: 1 request/30min = 48 requests/day = ~4,800 tweets/day (within free tier)

### Reddit Scraping

**Approach**: Use Reddit API (free, no auth required for public data) + PRAW fallback

```typescript
// workers/scraper-reddit/src/reddit-client.ts

interface RedditScraperConfig {
  subreddits: string[];
  sortBy: 'hot' | 'top' | 'rising' | 'new';
  timeFilter: 'hour' | 'day' | 'week';
  minUpvotes: number;
  minComments: number;
}

class RedditScraper {
  private baseURL = 'https://www.reddit.com';
  private config: RedditScraperConfig;

  constructor(config: RedditScraperConfig) {
    this.config = config;
  }

  async scrapeSubreddits(): Promise<RedditPost[]> {
    const posts: RedditPost[] = [];

    for (const subreddit of this.config.subreddits) {
      const url = `${this.baseURL}/r/${subreddit}/${this.config.sortBy}.json?t=${this.config.timeFilter}&limit=100`;

      const response = await fetch(url, {
        headers: {
          'User-Agent': 'Claudate-Viral-Content-Tracker/1.0'
        }
      });

      if (!response.ok) {
        console.error(`Failed to fetch r/${subreddit}: ${response.status}`);
        continue;
      }

      const data = await response.json();

      for (const child of data.data.children) {
        const post = child.data;
        if (this.meetsThreshold(post)) {
          posts.push(this.transformPost(post, subreddit));
        }
      }
    }

    return posts;
  }

  private meetsThreshold(post: any): boolean {
    return (
      post.ups >= this.config.minUpvotes &&
      post.num_comments >= this.config.minComments &&
      !post.stickied // Exclude pinned posts
    );
  }

  private transformPost(raw: any, subreddit: string): RedditPost {
    return {
      platform: 'reddit',
      platformId: raw.id,
      authorUsername: raw.author,
      authorFollowers: 0, // Reddit doesn't expose this easily
      contentText: raw.title + '\n\n' + (raw.selftext || ''),
      contentType: this.detectContentType(raw),
      url: `https://www.reddit.com${raw.permalink}`,
      mediaUrls: this.extractMediaUrls(raw),
      likes: raw.ups,
      shares: 0, // Reddit doesn't track shares
      comments: raw.num_comments,
      views: 0, // Reddit doesn't expose view count publicly
      postedAt: raw.created_utc * 1000,
      subreddit: subreddit
    };
  }

  private detectContentType(post: any): ContentType {
    if (post.is_video) return 'video';
    if (post.post_hint === 'image') return 'image';
    if (post.post_hint === 'link') return 'link';
    if (post.selftext) return 'text';
    return 'mixed';
  }

  private extractMediaUrls(post: any): string[] {
    const urls: string[] = [];
    if (post.url) urls.push(post.url);
    if (post.preview?.images) {
      urls.push(...post.preview.images.map((img: any) => img.source.url));
    }
    return urls;
  }
}

// Cron trigger (every 20 minutes for high-traffic subreddits)
export default {
  async scheduled(event: ScheduledEvent, env: Env, ctx: ExecutionContext) {
    const scraper = new RedditScraper({
      subreddits: [
        'ChatGPT',
        'ClaudeAI',
        'ArtificialIntelligence',
        'entrepreneur',
        'SaaS',
        'Productivity',
        'nocode',
        'indiehackers'
      ],
      sortBy: 'hot',
      timeFilter: 'day',
      minUpvotes: 50,
      minComments: 10
    });

    const posts = await scraper.scrapeSubreddits();
    await ctx.waitUntil(storePosts(env.DB, posts));

    for (const post of posts) {
      await env.ANALYZER_QUEUE.send({ contentId: post.id });
    }
  }
};
```

**Rate Limits**: Reddit allows ~60 requests/minute without auth. 1 request/20min = very conservative.

### TikTok Scraping

**Approach**: Use unofficial TikTok API (tiktok-scraper npm package) or Apify TikTok Scraper

```typescript
// workers/scraper-tiktok/src/tiktok-client.ts

import TikTokScraper from 'tiktok-scraper';

interface TikTokScraperConfig {
  hashtags: string[];
  keywords: string[];
  minLikes: number;
  minViews: number;
}

class TikTokScraper {
  private config: TikTokScraperConfig;

  constructor(config: TikTokScraperConfig) {
    this.config = config;
  }

  async scrapeHashtags(): Promise<TikTokVideo[]> {
    const videos: TikTokVideo[] = [];

    for (const hashtag of this.config.hashtags) {
      try {
        const results = await TikTokScraper.hashtag(hashtag, {
          number: 50,
          sessionList: [] // Can add session cookies for better reliability
        });

        for (const video of results.collector) {
          if (this.meetsThreshold(video)) {
            videos.push(this.transformVideo(video, hashtag));
          }
        }
      } catch (error) {
        console.error(`Failed to scrape #${hashtag}:`, error);
      }
    }

    return videos;
  }

  private meetsThreshold(video: any): boolean {
    return (
      video.diggCount >= this.config.minLikes &&
      video.playCount >= this.config.minViews
    );
  }

  private transformVideo(raw: any, hashtag: string): TikTokVideo {
    return {
      platform: 'tiktok',
      platformId: raw.id,
      authorUsername: raw.authorMeta.name,
      authorFollowers: raw.authorMeta.fans,
      contentText: raw.text,
      contentType: 'video',
      url: `https://www.tiktok.com/@${raw.authorMeta.name}/video/${raw.id}`,
      mediaUrls: [raw.videoUrl],
      likes: raw.diggCount,
      shares: raw.shareCount,
      comments: raw.commentCount,
      views: raw.playCount,
      postedAt: raw.createTime * 1000,
      hashtag: hashtag
    };
  }
}

// Cron trigger (every 1 hour - TikTok is less time-sensitive for B2B content)
export default {
  async scheduled(event: ScheduledEvent, env: Env, ctx: ExecutionContext) {
    const scraper = new TikTokScraper({
      hashtags: ['AItools', 'productivity', 'techtools', 'SaaS', 'entrepreneur'],
      keywords: ['AI', 'Claude', 'ChatGPT', 'productivity'],
      minLikes: 500,
      minViews: 10000
    });

    const videos = await scraper.scrapeHashtags();
    await ctx.waitUntil(storeVideos(env.DB, videos));

    for (const video of videos) {
      await env.ANALYZER_QUEUE.send({ contentId: video.id });
    }
  }
};
```

**Note**: TikTok is the riskiest platform (unofficial API, rate limits, IP blocks). Consider Apify as paid alternative (~$50/mo for reliable scraping).

---

## 📊 Virality Scoring Algorithm

```typescript
// workers/analyzer/src/virality-scorer.ts

interface ViralityFactors {
  engagementRate: number;     // (likes + shares + comments) / views
  growthVelocity: number;      // Engagement per hour since posting
  authorInfluence: number;     // Follower count (normalized)
  contentQuality: number;      // AI-assessed quality (0-1)
  novelty: number;             // How unique/fresh the content is (0-1)
  recency: number;             // Time decay factor (0-1)
}

class ViralityScorer {
  /**
   * Calculate virality score (0-100)
   *
   * Formula:
   * Score = (ER * 30) + (GV * 25) + (AI * 15) + (CQ * 15) + (N * 10) + (R * 5)
   *
   * Weights:
   * - Engagement Rate: 30% (most important)
   * - Growth Velocity: 25% (trending indicator)
   * - Author Influence: 15% (reach potential)
   * - Content Quality: 15% (AI assessment)
   * - Novelty: 10% (uniqueness)
   * - Recency: 5% (time decay)
   */
  calculateScore(content: Content, factors: ViralityFactors): number {
    const score =
      this.normalizeEngagementRate(factors.engagementRate) * 30 +
      this.normalizeGrowthVelocity(factors.growthVelocity) * 25 +
      this.normalizeAuthorInfluence(factors.authorInfluence) * 15 +
      factors.contentQuality * 15 +
      factors.novelty * 10 +
      factors.recency * 5;

    return Math.min(100, Math.max(0, score));
  }

  private normalizeEngagementRate(rate: number): number {
    // Typical good engagement: 3-5%
    // Viral: >10%
    // Scale to 0-1 with sigmoid
    return 1 / (1 + Math.exp(-10 * (rate - 0.05)));
  }

  private normalizeGrowthVelocity(velocity: number): number {
    // velocity = total_engagement / hours_since_post
    // 100 engagements/hour = good
    // 1000/hour = viral
    return Math.min(1, velocity / 1000);
  }

  private normalizeAuthorInfluence(followers: number): number {
    // Scale logarithmically
    // 1K followers = 0.3
    // 10K = 0.5
    // 100K = 0.7
    // 1M+ = 1.0
    return Math.min(1, Math.log10(followers) / 6);
  }

  async analyzeContent(content: Content, db: D1Database): Promise<ViralityFactors> {
    // Calculate engagement rate
    const totalEngagement = content.likes + content.shares + content.comments;
    const engagementRate = content.views > 0 ? totalEngagement / content.views : 0;

    // Calculate growth velocity
    const hoursSincePost = (Date.now() - content.postedAt) / (1000 * 60 * 60);
    const growthVelocity = hoursSincePost > 0 ? totalEngagement / hoursSincePost : 0;

    // Normalize author influence
    const authorInfluence = content.authorFollowers;

    // AI quality assessment (using Gemini)
    const { contentQuality, novelty } = await this.assessWithAI(content);

    // Recency factor (exponential decay)
    const recency = Math.exp(-hoursSincePost / 24); // Decays over 24 hours

    return {
      engagementRate,
      growthVelocity,
      authorInfluence,
      contentQuality,
      novelty,
      recency
    };
  }

  private async assessWithAI(content: Content): Promise<{ contentQuality: number; novelty: number }> {
    // Use Gemini API to assess content quality and novelty
    const prompt = `
Analyze this social media content and rate it on two dimensions:

**Content Text**:
${content.contentText}

**Questions**:
1. Content Quality (0-1): How well-crafted, valuable, or engaging is this content?
   - 0.0-0.3: Low quality (spam, low effort, unclear)
   - 0.4-0.6: Medium quality (decent, informative)
   - 0.7-1.0: High quality (exceptional, highly valuable)

2. Novelty (0-1): How unique, fresh, or unexpected is this content?
   - 0.0-0.3: Generic (commonly seen, repetitive)
   - 0.4-0.6: Moderately unique (some fresh angle)
   - 0.7-1.0: Highly novel (unique insight, fresh perspective)

Return only JSON: {"contentQuality": 0.8, "novelty": 0.6}
`;

    // Call Gemini API (omitted for brevity, see OnlyFans plugin doc for implementation)
    const response = await callGeminiAPI(prompt);
    return JSON.parse(response);
  }
}
```

**Virality Thresholds**:
- **0-30**: Low virality (normal content)
- **31-50**: Moderate virality (gaining traction)
- **51-70**: High virality (trending)
- **71-90**: Very high virality (viral)
- **91-100**: Extreme virality (mega-viral, rare)

---

## 🎨 Dashboard UI Design

### Home Page (Trending Feed)

```tsx
// dashboard/src/pages/index.tsx

import React, { useState, useEffect } from 'react';
import { ContentCard } from '../components/ContentCard';
import { PlatformFilter } from '../components/PlatformFilter';
import { TrendChart } from '../components/TrendChart';

interface Content {
  id: string;
  platform: 'twitter' | 'reddit' | 'tiktok';
  authorUsername: string;
  contentText: string;
  url: string;
  likes: number;
  shares: number;
  comments: number;
  views: number;
  viralityScore: number;
  topics: string[];
  postedAt: number;
}

export default function HomePage() {
  const [contents, setContents] = useState<Content[]>([]);
  const [platform, setPlatform] = useState<string>('all');
  const [sortBy, setSortBy] = useState<'virality' | 'engagement' | 'recent'>('virality');
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    loadTrendingContent();
  }, [platform, sortBy]);

  const loadTrendingContent = async () => {
    setLoading(true);
    const params = new URLSearchParams({
      platform: platform === 'all' ? '' : platform,
      sort: sortBy,
      limit: '50'
    });

    const response = await fetch(`/api/trending?${params}`);
    const data = await response.json();
    setContents(data.contents);
    setLoading(false);
  };

  return (
    <div className="dashboard-container">
      <header className="dashboard-header">
        <h1>Viral Content Intelligence</h1>
        <p className="subtitle">Real-time tracking of trending content across Twitter, Reddit, and TikTok</p>
      </header>

      <div className="controls">
        <PlatformFilter
          selected={platform}
          onChange={setPlatform}
        />

        <div className="sort-selector">
          <label>Sort by:</label>
          <select value={sortBy} onChange={(e) => setSortBy(e.target.value as any)}>
            <option value="virality">Virality Score</option>
            <option value="engagement">Total Engagement</option>
            <option value="recent">Most Recent</option>
          </select>
        </div>

        <button className="refresh-btn" onClick={loadTrendingContent}>
          🔄 Refresh
        </button>
      </div>

      <div className="stats-overview">
        <StatCard label="Tracked Content" value="1,247" change="+12%" />
        <StatCard label="Trending Now" value="87" change="+5%" />
        <StatCard label="Avg Virality" value="64.3" change="+2.1" />
        <StatCard label="New Today" value="342" change="+18%" />
      </div>

      {loading ? (
        <div className="loading">Loading trending content...</div>
      ) : (
        <div className="content-grid">
          {contents.map((content) => (
            <ContentCard key={content.id} content={content} />
          ))}
        </div>
      )}
    </div>
  );
}

const StatCard: React.FC<{ label: string; value: string; change: string }> =
  ({ label, value, change }) => (
    <div className="stat-card">
      <div className="stat-label">{label}</div>
      <div className="stat-value">{value}</div>
      <div className={`stat-change ${change.startsWith('+') ? 'positive' : 'negative'}`}>
        {change}
      </div>
    </div>
  );
```

### Content Card Component

```tsx
// dashboard/src/components/ContentCard.tsx

import React from 'react';

interface ContentCardProps {
  content: Content;
}

export const ContentCard: React.FC<ContentCardProps> = ({ content }) => {
  const platformIcon = {
    twitter: '𝕏',
    reddit: '🔴',
    tiktok: '🎵'
  }[content.platform];

  const viralityColor =
    content.viralityScore >= 70 ? '#10b981' :
    content.viralityScore >= 50 ? '#f59e0b' :
    '#6b7280';

  return (
    <div className="content-card">
      <div className="card-header">
        <span className="platform-badge">{platformIcon} {content.platform}</span>
        <div className="virality-badge" style={{ backgroundColor: viralityColor }}>
          {content.viralityScore.toFixed(0)}
        </div>
      </div>

      <div className="card-body">
        <p className="author">@{content.authorUsername}</p>
        <p className="content-text">{truncate(content.contentText, 200)}</p>

        <div className="topics">
          {content.topics.map((topic) => (
            <span key={topic} className="topic-tag">{topic}</span>
          ))}
        </div>
      </div>

      <div className="card-footer">
        <div className="metrics">
          <span>❤️ {formatNumber(content.likes)}</span>
          <span>🔄 {formatNumber(content.shares)}</span>
          <span>💬 {formatNumber(content.comments)}</span>
          {content.views > 0 && <span>👁️ {formatNumber(content.views)}</span>}
        </div>

        <div className="actions">
          <button onClick={() => window.open(content.url, '_blank')}>
            View Original
          </button>
          <button onClick={() => saveContent(content.id)}>
            Save
          </button>
        </div>
      </div>

      <div className="card-timestamp">
        {formatRelativeTime(content.postedAt)}
      </div>
    </div>
  );
};

function truncate(text: string, length: number): string {
  return text.length > length ? text.slice(0, length) + '...' : text;
}

function formatNumber(num: number): string {
  if (num >= 1000000) return (num / 1000000).toFixed(1) + 'M';
  if (num >= 1000) return (num / 1000).toFixed(1) + 'K';
  return num.toString();
}

function formatRelativeTime(timestamp: number): string {
  const seconds = Math.floor((Date.now() - timestamp) / 1000);
  if (seconds < 60) return `${seconds}s ago`;
  if (seconds < 3600) return `${Math.floor(seconds / 60)}m ago`;
  if (seconds < 86400) return `${Math.floor(seconds / 3600)}h ago`;
  return `${Math.floor(seconds / 86400)}d ago`;
}
```

---

## 🚀 Implementation Roadmap

### Week 1: Foundation

**Days 1-2**: Infrastructure Setup
- [ ] Initialize monorepo (pnpm workspace)
- [ ] Create D1 database + run migrations
- [ ] Set up R2 bucket
- [ ] Configure wrangler.toml for all workers

**Days 3-5**: Twitter Scraper
- [ ] Implement Twitter API client
- [ ] Build scraper logic
- [ ] Add cron scheduling
- [ ] Test and verify data collection

### Week 2: Reddit + TikTok

**Days 1-3**: Reddit Scraper
- [ ] Implement Reddit API client
- [ ] Build scraper for target subreddits
- [ ] Add cron scheduling
- [ ] Test data collection

**Days 4-5**: TikTok Scraper
- [ ] Implement TikTok scraper (unofficial API or Apify)
- [ ] Build hashtag monitoring
- [ ] Add cron scheduling
- [ ] Test (may need proxy/IP rotation)

### Week 3: Analysis & Scoring

**Days 1-3**: Virality Scorer
- [ ] Implement scoring algorithm
- [ ] Integrate Gemini API for quality assessment
- [ ] Build metrics history tracking
- [ ] Test scoring accuracy

**Days 4-5**: Content Analyzer
- [ ] Implement topic extraction
- [ ] Build format classifier
- [ ] Add sentiment analysis
- [ ] Test analysis results

### Week 4: Dashboard UI

**Days 1-3**: Frontend Core
- [ ] Set up Next.js + Tailwind
- [ ] Build trending feed page
- [ ] Create content card component
- [ ] Implement filtering and sorting

**Days 4-5**: Polish & Deploy
- [ ] Add analytics page
- [ ] Implement search functionality
- [ ] Deploy to Cloudflare Pages
- [ ] Test end-to-end

---

## 💰 Cost Estimation

| Resource | Usage | Cost/month |
|----------|-------|------------|
| **Twitter API** | Free tier | $0 |
| **Reddit API** | Free | $0 |
| **TikTok Scraping** | Apify | $49 |
| **Gemini API** | 500K tokens | $2.50 |
| **D1 Database** | 1M reads, 100K writes | $0.75 |
| **R2 Storage** | 50 GB | $0.75 |
| **Workers** | 1M requests | $0.30 |
| **Pages** | Unlimited | $0 |
| **Total** | Internal tool | **~$54/month** |

**Very affordable for internal use** - Less than $700/year.

---

## 📈 Success Metrics

### Internal Use KPIs

| Metric | Target (Month 1) | Target (Month 3) |
|--------|------------------|------------------|
| **Content Tracked** | 5,000+ items | 50,000+ items |
| **Daily Active Items** | 200+ new/day | 500+ new/day |
| **Trending Accuracy** | 60%+ viral prediction | 80%+ viral prediction |
| **Team Engagement** | 3+ daily active users | 5+ daily active users |
| **Actionable Insights** | 5+ per week | 15+ per week |

### Qualitative Success

- Team uses dashboard daily for content inspiration
- Marketing campaigns informed by viral trends
- Faster response to trending topics (24-48 hours)
- Better understanding of audience preferences

---

## 🔄 Future API Productization (Phase 2)

**If we decide to commercialize later**, the API could offer:

```typescript
// Example public API endpoints (Phase 2)

// GET /api/v1/trending
// Returns trending content across platforms
interface TrendingAPI {
  platform?: 'twitter' | 'reddit' | 'tiktok';
  topic?: string;
  minScore?: number;
  limit?: number;
}

// GET /api/v1/content/:id
// Get detailed analysis for specific content

// GET /api/v1/topics/trending
// Get trending topics across all platforms

// POST /api/v1/track
// Track specific keywords/hashtags for user
```

**Pricing** (if commercialized):
- **Free**: 100 API calls/day
- **Starter** ($49/mo): 10K calls/day
- **Pro** ($199/mo): 100K calls/day
- **Enterprise** ($999/mo): Unlimited + custom tracking

**Projected Revenue** (Year 1, if commercialized):
- 1,000 users × $49 avg = **$49K MRR** = $588K ARR

**But this is deferred** - Focus on internal use first!

---

## ✅ Document Status

**Current Status**: 🎯 **Ready for Implementation**

**Completeness**:
- [x] Architecture defined
- [x] Scraping strategy documented
- [x] Virality algorithm specified
- [x] Database schema complete
- [x] UI design provided
- [x] Timeline established
- [x] Costs estimated

**Next Action**: Begin Week 1 (Infrastructure Setup)

---

**Document Version**: 1.0
**Created**: 2025-11-13
**Lines**: ~1,800
**Estimated Read Time**: 45 minutes

---

**Ready to build viral content intelligence!** 🚀
