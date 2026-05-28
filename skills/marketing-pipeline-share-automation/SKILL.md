---
name: marketing-pipeline-share-automation
description: Full-stack AI content pipeline for automated research, script generation, Facebook posting, and video rendering using Claude/OpenAI and Remotion
triggers:
  - automate content creation with AI research and video generation
  - set up marketing pipeline for automated posting
  - generate videos from AI-written content automatically
  - crawl news sources and create content with Claude
  - build content automation system with Remotion
  - create AI-powered marketing pipeline with auto-posting
  - configure content pipeline from research to video
  - use marketing-pipeline-share for automated content workflow
---

# Marketing Pipeline Share Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with **marketing-pipeline-share**, a complete AI-powered content automation system that handles research (web crawling), script generation (Claude/OpenAI), Facebook auto-posting, and video rendering (Remotion). It's designed for marketers and content creators who want to automate their entire content workflow from keyword input to published video.

## What This Project Does

Marketing Pipeline Share is a TypeScript/Next.js application that automates the complete content creation pipeline:

1. **Auto-Research**: Crawls news sources (TechCrunch, a16z, Twitter, LinkedIn) for recent data
2. **AI Content Generation**: Uses Claude 3 or OpenAI to generate articles in multiple formats (toplist, POV, case study, how-to)
3. **Multi-language Support**: Generates content in both English and Vietnamese
4. **Auto-Posting**: Publishes directly to Facebook pages
5. **Video Rendering**: Converts content to videos/infographics using Remotion for Reels/TikTok/Shorts

## Installation

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
yarn install

# Copy environment template
cp .env.example .env.local
```

## Configuration

Set up your environment variables in `.env.local`:

```bash
# AI Provider APIs
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research/Crawling APIs
RAPIDAPI_KEY=your_rapidapi_key
RAPIDAPI_HOST=news-api-host

# Facebook Integration
FACEBOOK_PAGE_ACCESS_TOKEN=your_fb_page_token
FACEBOOK_PAGE_ID=your_page_id

# Remotion Configuration
REMOTION_CLOUDRUN_URL=your_cloudrun_url
REMOTION_API_KEY=your_remotion_key

# Application
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

## Key Architecture

```typescript
// Typical project structure
src/
├── app/                    # Next.js app directory
│   ├── api/               # API routes
│   │   ├── research/      # News crawling endpoints
│   │   ├── generate/      # AI content generation
│   │   ├── post/          # Facebook posting
│   │   └── render/        # Video rendering
│   └── dashboard/         # UI components
├── lib/
│   ├── ai/                # AI provider integrations
│   ├── crawlers/          # Web scraping utilities
│   ├── facebook/          # FB API integration
│   └── remotion/          # Video rendering logic
└── types/                 # TypeScript definitions
```

## Core Usage Patterns

### 1. Research & Content Generation Flow

```typescript
// lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

export interface ContentRequest {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  researchData?: any[];
}

export class ContentGenerator {
  private claude: Anthropic;
  private openai: OpenAI;

  constructor() {
    this.claude = new Anthropic({
      apiKey: process.env.ANTHROPIC_API_KEY,
    });
    this.openai = new OpenAI({
      apiKey: process.env.OPENAI_API_KEY,
    });
  }

  async generateWithClaude(request: ContentRequest): Promise<string> {
    const prompt = this.buildPrompt(request);
    
    const message = await this.claude.messages.create({
      model: 'claude-3-5-sonnet-20241022',
      max_tokens: 4096,
      messages: [{
        role: 'user',
        content: prompt
      }]
    });

    return message.content[0].type === 'text' 
      ? message.content[0].text 
      : '';
  }

  async generateWithOpenAI(request: ContentRequest): Promise<string> {
    const prompt = this.buildPrompt(request);
    
    const completion = await this.openai.chat.completions.create({
      model: 'gpt-4-turbo',
      messages: [{
        role: 'user',
        content: prompt
      }],
      temperature: 0.7,
    });

    return completion.choices[0].message.content || '';
  }

  private buildPrompt(request: ContentRequest): string {
    const { keyword, format, language, tone, researchData } = request;
    
    let prompt = `Create a ${format} article about "${keyword}" in ${language}.`;
    prompt += `\nTone: ${tone}`;
    
    if (researchData && researchData.length > 0) {
      prompt += `\n\nRecent research data:\n`;
      researchData.forEach((item, idx) => {
        prompt += `${idx + 1}. ${item.title}\n${item.summary}\n`;
      });
    }
    
    return prompt;
  }
}
```

### 2. News Crawling/Research System

```typescript
// lib/crawlers/news-crawler.ts
import axios from 'axios';

export interface NewsArticle {
  title: string;
  url: string;
  source: string;
  publishedAt: string;
  summary: string;
}

export class NewsCrawler {
  private rapidApiKey: string;
  private rapidApiHost: string;

  constructor() {
    this.rapidApiKey = process.env.RAPIDAPI_KEY!;
    this.rapidApiHost = process.env.RAPIDAPI_HOST!;
  }

  async crawlRecentNews(keyword: string, hours: number = 24): Promise<NewsArticle[]> {
    try {
      const response = await axios.get('https://api.rapidapi.com/v1/news/search', {
        params: {
          q: keyword,
          from: this.getTimeRange(hours),
          sortBy: 'publishedAt',
          language: 'en'
        },
        headers: {
          'X-RapidAPI-Key': this.rapidApiKey,
          'X-RapidAPI-Host': this.rapidApiHost
        }
      });

      return response.data.articles.map((article: any) => ({
        title: article.title,
        url: article.url,
        source: article.source.name,
        publishedAt: article.publishedAt,
        summary: article.description || ''
      }));
    } catch (error) {
      console.error('News crawling error:', error);
      return [];
    }
  }

  private getTimeRange(hours: number): string {
    const date = new Date();
    date.setHours(date.getHours() - hours);
    return date.toISOString();
  }

  async crawlMultipleSources(keyword: string): Promise<NewsArticle[]> {
    // Crawl from TechCrunch, a16z, etc.
    const sources = ['techcrunch', 'a16z-blog'];
    const allArticles: NewsArticle[] = [];

    for (const source of sources) {
      const articles = await this.crawlRecentNews(`${keyword} site:${source}`, 24);
      allArticles.push(...articles);
    }

    return allArticles;
  }
}
```

### 3. API Route Example - Generate Content

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { ContentGenerator } from '@/lib/ai/content-generator';
import { NewsCrawler } from '@/lib/crawlers/news-crawler';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const { keyword, format, language, tone, useResearch, aiProvider } = body;

    let researchData = [];
    
    if (useResearch) {
      const crawler = new NewsCrawler();
      researchData = await crawler.crawlMultipleSources(keyword);
    }

    const generator = new ContentGenerator();
    let content: string;

    if (aiProvider === 'claude') {
      content = await generator.generateWithClaude({
        keyword,
        format,
        language,
        tone,
        researchData
      });
    } else {
      content = await generator.generateWithOpenAI({
        keyword,
        format,
        language,
        tone,
        researchData
      });
    }

    return NextResponse.json({
      success: true,
      content,
      researchCount: researchData.length
    });
  } catch (error: any) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

### 4. Facebook Auto-Posting

```typescript
// lib/facebook/post-manager.ts
import axios from 'axios';

export interface FacebookPost {
  message: string;
  link?: string;
  imageUrl?: string;
}

export class FacebookPostManager {
  private pageAccessToken: string;
  private pageId: string;

  constructor() {
    this.pageAccessToken = process.env.FACEBOOK_PAGE_ACCESS_TOKEN!;
    this.pageId = process.env.FACEBOOK_PAGE_ID!;
  }

  async publishPost(post: FacebookPost): Promise<string> {
    try {
      const endpoint = `https://graph.facebook.com/v18.0/${this.pageId}/feed`;
      
      const response = await axios.post(endpoint, {
        message: post.message,
        link: post.link,
        access_token: this.pageAccessToken
      });

      return response.data.id;
    } catch (error: any) {
      throw new Error(`Facebook posting failed: ${error.response?.data?.error?.message || error.message}`);
    }
  }

  async publishPhoto(post: FacebookPost): Promise<string> {
    if (!post.imageUrl) {
      throw new Error('Image URL required for photo post');
    }

    const endpoint = `https://graph.facebook.com/v18.0/${this.pageId}/photos`;
    
    const response = await axios.post(endpoint, {
      url: post.imageUrl,
      caption: post.message,
      access_token: this.pageAccessToken
    });

    return response.data.id;
  }

  async schedulePost(post: FacebookPost, publishTime: Date): Promise<string> {
    const endpoint = `https://graph.facebook.com/v18.0/${this.pageId}/feed`;
    
    const response = await axios.post(endpoint, {
      message: post.message,
      link: post.link,
      published: false,
      scheduled_publish_time: Math.floor(publishTime.getTime() / 1000),
      access_token: this.pageAccessToken
    });

    return response.data.id;
  }
}
```

### 5. Video Rendering with Remotion

```typescript
// lib/remotion/video-renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import { webpackOverride } from './webpack-override';

export interface VideoConfig {
  title: string;
  content: string[];
  duration: number;
  format: 'reels' | 'tiktok' | 'shorts';
}

export class VideoRenderer {
  async renderContentVideo(config: VideoConfig): Promise<string> {
    const bundled = await bundle({
      entryPoint: './src/remotion/index.ts',
      webpackOverride: webpackOverride,
    });

    const composition = await selectComposition({
      serveUrl: bundled,
      id: 'ContentVideo',
      inputProps: {
        title: config.title,
        content: config.content,
        format: config.format
      },
    });

    const outputLocation = `./output/video-${Date.now()}.mp4`;

    await renderMedia({
      composition,
      serveUrl: bundled,
      codec: 'h264',
      outputLocation,
      inputProps: {
        title: config.title,
        content: config.content,
        format: config.format
      },
    });

    return outputLocation;
  }

  getAspectRatio(format: string): { width: number; height: number } {
    switch (format) {
      case 'reels':
      case 'tiktok':
      case 'shorts':
        return { width: 1080, height: 1920 }; // 9:16
      default:
        return { width: 1920, height: 1080 }; // 16:9
    }
  }
}
```

### 6. Complete Pipeline Example

```typescript
// lib/pipeline/content-pipeline.ts
import { ContentGenerator } from '@/lib/ai/content-generator';
import { NewsCrawler } from '@/lib/crawlers/news-crawler';
import { FacebookPostManager } from '@/lib/facebook/post-manager';
import { VideoRenderer } from '@/lib/remotion/video-renderer';

export class ContentPipeline {
  private generator: ContentGenerator;
  private crawler: NewsCrawler;
  private fbManager: FacebookPostManager;
  private videoRenderer: VideoRenderer;

  constructor() {
    this.generator = new ContentGenerator();
    this.crawler = new NewsCrawler();
    this.fbManager = new FacebookPostManager();
    this.videoRenderer = new VideoRenderer();
  }

  async executeFullPipeline(keyword: string) {
    console.log('Step 1: Researching...');
    const researchData = await this.crawler.crawlMultipleSources(keyword);

    console.log('Step 2: Generating content...');
    const content = await this.generator.generateWithClaude({
      keyword,
      format: 'toplist',
      language: 'en',
      tone: 'expert',
      researchData
    });

    console.log('Step 3: Rendering video...');
    const videoPath = await this.videoRenderer.renderContentVideo({
      title: keyword,
      content: content.split('\n').filter(line => line.trim()),
      duration: 30,
      format: 'reels'
    });

    console.log('Step 4: Posting to Facebook...');
    const postId = await this.fbManager.publishPost({
      message: `New content about ${keyword}!`,
      link: `https://example.com/videos/${videoPath}`
    });

    return {
      success: true,
      researchCount: researchData.length,
      content,
      videoPath,
      postId
    };
  }
}
```

### 7. API Route - Full Pipeline

```typescript
// app/api/pipeline/execute/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { ContentPipeline } from '@/lib/pipeline/content-pipeline';

export async function POST(request: NextRequest) {
  try {
    const { keyword } = await request.json();

    if (!keyword) {
      return NextResponse.json(
        { success: false, error: 'Keyword is required' },
        { status: 400 }
      );
    }

    const pipeline = new ContentPipeline();
    const result = await pipeline.executeFullPipeline(keyword);

    return NextResponse.json(result);
  } catch (error: any) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

## Running the Application

```bash
# Development
npm run dev

# Production build
npm run build
npm start

# Run specific pipeline steps
npm run research -- --keyword "AI trends"
npm run generate -- --keyword "AI trends" --format toplist
npm run render -- --input content.json
```

## Common Workflows

### Workflow 1: Research-Only Mode

```typescript
// Quick research without content generation
import { NewsCrawler } from '@/lib/crawlers/news-crawler';

const crawler = new NewsCrawler();
const articles = await crawler.crawlRecentNews('AI marketing', 48);
console.log(`Found ${articles.length} articles`);
```

### Workflow 2: Multi-Language Content

```typescript
// Generate content in both languages
const generator = new ContentGenerator();

const englishContent = await generator.generateWithClaude({
  keyword: 'Marketing automation',
  format: 'how-to',
  language: 'en',
  tone: 'expert'
});

const vietnameseContent = await generator.generateWithClaude({
  keyword: 'Marketing automation',
  format: 'how-to',
  language: 'vi',
  tone: 'friendly'
});
```

### Workflow 3: Scheduled Posting

```typescript
// Schedule content for future posting
const fbManager = new FacebookPostManager();
const publishTime = new Date();
publishTime.setHours(publishTime.getHours() + 24);

const postId = await fbManager.schedulePost({
  message: generatedContent,
  link: 'https://example.com/article'
}, publishTime);
```

## Troubleshooting

### API Key Issues

```typescript
// Verify API keys are loaded
if (!process.env.ANTHROPIC_API_KEY) {
  throw new Error('ANTHROPIC_API_KEY not found in environment');
}

// Test API connection
const testConnection = async () => {
  try {
    const claude = new Anthropic({ apiKey: process.env.ANTHROPIC_API_KEY });
    await claude.messages.create({
      model: 'claude-3-5-sonnet-20241022',
      max_tokens: 10,
      messages: [{ role: 'user', content: 'test' }]
    });
    console.log('✓ Claude API connected');
  } catch (error) {
    console.error('✗ Claude API failed:', error);
  }
};
```

### Rate Limiting

```typescript
// Implement retry logic with exponential backoff
async function withRetry<T>(
  fn: () => Promise<T>,
  maxRetries: number = 3
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error: any) {
      if (error.status === 429 && i < maxRetries - 1) {
        await new Promise(resolve => setTimeout(resolve, Math.pow(2, i) * 1000));
        continue;
      }
      throw error;
    }
  }
  throw new Error('Max retries exceeded');
}
```

### Facebook Posting Errors

```typescript
// Debug Facebook API errors
try {
  await fbManager.publishPost(post);
} catch (error: any) {
  if (error.message.includes('access token')) {
    console.error('Facebook token expired or invalid');
    // Refresh token logic here
  } else if (error.message.includes('rate limit')) {
    console.error('Facebook rate limit reached');
    // Queue for later
  } else {
    console.error('Unknown Facebook error:', error);
  }
}
```

### Video Rendering Issues

```typescript
// Check Remotion dependencies
import { getCompositions } from '@remotion/renderer';

const checkRemotionSetup = async () => {
  try {
    const compositions = await getCompositions('./src/remotion/index.ts');
    console.log('Available compositions:', compositions.map(c => c.id));
  } catch (error) {
    console.error('Remotion setup error:', error);
  }
};
```

This skill enables complete automation of content marketing workflows from research through video publication using modern AI tools and TypeScript.
