---
name: marketing-pipeline-share-content-automation
description: Automate content creation from research to video generation using AI-powered pipeline with Claude, OpenAI, and Remotion
triggers:
  - how do I automate content creation with AI research and video generation
  - set up marketing pipeline for automated content from research to video
  - create automated content workflow with Claude and Remotion
  - generate AI-powered articles and videos from keywords automatically
  - build content automation pipeline with research crawling and video rendering
  - automate social media content from research to video with AI
  - use marketing pipeline share for content generation
  - configure AI content automation with multilingual support
---

# Marketing Pipeline Share Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with **marketing-pipeline-share**, an end-to-end content automation system that transforms keywords into finished articles and videos. The pipeline automatically researches trending topics, generates multilingual content with Claude/OpenAI, and renders videos with Remotion.

## What This Project Does

Marketing Pipeline Share automates the entire content creation workflow:

1. **Auto-Research**: Crawls recent news from TechCrunch, a16z, Twitter/X, LinkedIn (last 24h)
2. **AI Content Generation**: Creates articles in multiple formats (toplist, POV, case study, how-to) using Claude 3 or OpenAI
3. **Multilingual Output**: Generates content in Vietnamese and English simultaneously
4. **Video Rendering**: Automatically creates infographics and short videos using Remotion
5. **Platform Optimization**: Exports videos optimized for Reels, TikTok, Shorts

## Installation

### Prerequisites

```bash
node -v  # Requires Node.js 18+
npm -v   # or yarn/pnpm
```

### Clone and Setup

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
yarn install
# or
pnpm install
```

### Environment Configuration

Create a `.env.local` file in the root directory:

```bash
# AI Providers
ANTHROPIC_API_KEY=your_claude_api_key_here
OPENAI_API_KEY=your_openai_api_key_here

# Research/Data APIs
RAPIDAPI_KEY=your_rapidapi_key_here

# Optional: Video Rendering
REMOTION_LICENSE_KEY=your_remotion_license_here

# Database (if used)
DATABASE_URL=your_database_connection_string

# App Configuration
NEXT_PUBLIC_API_URL=http://localhost:3000
```

### Start Development Server

```bash
npm run dev
# or
yarn dev
# or
pnpm dev
```

The application will be available at `http://localhost:3000`

## Key Architecture

### Directory Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app router pages
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI provider integrations (Claude, OpenAI)
│   │   ├── research/    # Content research crawlers
│   │   ├── video/       # Remotion video rendering
│   │   └── utils/       # Helper functions
│   └── types/           # TypeScript type definitions
├── public/              # Static assets
└── remotion/           # Remotion video templates
```

## Core API Usage

### 1. Research & Content Scraping

```typescript
// lib/research/crawler.ts
import { crawlNews } from '@/lib/research/crawler';

interface ResearchOptions {
  keyword: string;
  sources?: ('techcrunch' | 'a16z' | 'twitter' | 'linkedin')[];
  timeRange?: '24h' | '7d' | '30d';
  maxResults?: number;
}

async function performResearch(options: ResearchOptions) {
  const { keyword, sources = ['techcrunch', 'a16z'], timeRange = '24h' } = options;
  
  try {
    const results = await crawlNews({
      query: keyword,
      sources,
      since: timeRange,
      limit: options.maxResults || 10
    });
    
    return {
      articles: results.articles,
      insights: results.insights,
      trends: results.trends
    };
  } catch (error) {
    console.error('Research failed:', error);
    throw error;
  }
}

// Usage
const research = await performResearch({
  keyword: 'AI automation',
  sources: ['techcrunch', 'twitter'],
  timeRange: '24h'
});
```

### 2. AI Content Generation with Claude

```typescript
// lib/ai/claude-generator.ts
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY
});

interface ContentRequest {
  topic: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'vi' | 'en';
  tone: 'expert' | 'friendly' | 'humorous';
  researchData: any;
}

async function generateContent(request: ContentRequest) {
  const { topic, format, language, tone, researchData } = request;
  
  const systemPrompt = buildSystemPrompt(format, language, tone);
  const userPrompt = buildUserPrompt(topic, researchData);
  
  const response = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    temperature: 0.7,
    system: systemPrompt,
    messages: [
      {
        role: 'user',
        content: userPrompt
      }
    ]
  });
  
  return {
    content: response.content[0].text,
    metadata: {
      model: response.model,
      tokens: response.usage
    }
  };
}

function buildSystemPrompt(format: string, language: string, tone: string): string {
  const prompts = {
    toplist: `You are an expert content writer creating engaging toplist articles in ${language}. Use a ${tone} tone.`,
    'pov': `You are a thought leader sharing unique perspectives in ${language}. Use a ${tone} tone.`,
    'case-study': `You are a business analyst creating detailed case studies in ${language}. Use a ${tone} tone.`,
    'how-to': `You are an instructional writer creating step-by-step guides in ${language}. Use a ${tone} tone.`
  };
  
  return prompts[format] || prompts.toplist;
}

function buildUserPrompt(topic: string, researchData: any): string {
  return `
Create comprehensive content about: ${topic}

Research Data:
${JSON.stringify(researchData, null, 2)}

Requirements:
- Include data-backed insights
- Reference recent trends (last 24h)
- Add actionable takeaways
- Optimize for social media sharing
  `;
}

// Usage
const article = await generateContent({
  topic: 'AI Content Automation Trends 2024',
  format: 'toplist',
  language: 'vi',
  tone: 'expert',
  researchData: research
});
```

### 3. Multilingual Content Generation

```typescript
// lib/ai/multilingual.ts
async function generateMultilingualContent(topic: string, researchData: any) {
  const languages = ['vi', 'en'];
  
  const contentPromises = languages.map(async (lang) => {
    return {
      language: lang,
      content: await generateContent({
        topic,
        format: 'toplist',
        language: lang as 'vi' | 'en',
        tone: 'expert',
        researchData
      })
    };
  });
  
  const results = await Promise.all(contentPromises);
  
  return results.reduce((acc, { language, content }) => {
    acc[language] = content;
    return acc;
  }, {} as Record<string, any>);
}

// Usage
const multilingualArticle = await generateMultilingualContent(
  'AI Marketing Automation',
  research
);

console.log(multilingualArticle.vi.content); // Vietnamese version
console.log(multilingualArticle.en.content); // English version
```

### 4. Video Generation with Remotion

```typescript
// lib/video/render.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  title: string;
  content: string[];
  style: 'infographic' | 'slideshow' | 'motion';
  platform: 'reels' | 'tiktok' | 'youtube-shorts';
}

const platformSpecs = {
  'reels': { width: 1080, height: 1920, fps: 30 },
  'tiktok': { width: 1080, height: 1920, fps: 30 },
  'youtube-shorts': { width: 1080, height: 1920, fps: 30 }
};

async function renderVideo(config: VideoConfig, outputPath: string) {
  const { title, content, style, platform } = config;
  const specs = platformSpecs[platform];
  
  // Bundle Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.join(process.cwd(), 'remotion/index.ts'),
    webpackOverride: (config) => config
  });
  
  // Get composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: style,
    inputProps: {
      title,
      content,
      ...specs
    }
  });
  
  // Render video
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: {
      title,
      content,
      ...specs
    }
  });
  
  return outputPath;
}

// Usage
const videoPath = await renderVideo({
  title: 'Top 5 AI Tools 2024',
  content: [
    'Tool 1: ChatGPT - Conversational AI',
    'Tool 2: Midjourney - Image Generation',
    'Tool 3: Claude - Advanced Reasoning',
    'Tool 4: Runway - Video AI',
    'Tool 5: ElevenLabs - Voice AI'
  ],
  style: 'infographic',
  platform: 'reels'
}, './output/ai-tools-video.mp4');

console.log('Video rendered:', videoPath);
```

### 5. Complete Pipeline Workflow

```typescript
// lib/pipeline/workflow.ts
interface PipelineConfig {
  keyword: string;
  contentFormat: 'toplist' | 'pov' | 'case-study' | 'how-to';
  languages: ('vi' | 'en')[];
  generateVideo: boolean;
  videoConfig?: {
    style: 'infographic' | 'slideshow' | 'motion';
    platform: 'reels' | 'tiktok' | 'youtube-shorts';
  };
}

async function runContentPipeline(config: PipelineConfig) {
  const { keyword, contentFormat, languages, generateVideo, videoConfig } = config;
  
  console.log(`🚀 Starting pipeline for: ${keyword}`);
  
  // Step 1: Research
  console.log('📡 Researching...');
  const research = await performResearch({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter'],
    timeRange: '24h'
  });
  
  // Step 2: Generate Content
  console.log('🧠 Generating content...');
  const contentResults = await Promise.all(
    languages.map(async (lang) => ({
      language: lang,
      content: await generateContent({
        topic: keyword,
        format: contentFormat,
        language: lang,
        tone: 'expert',
        researchData: research
      })
    }))
  );
  
  // Step 3: Extract key points for video
  let videoPath = null;
  if (generateVideo && videoConfig) {
    console.log('🎬 Rendering video...');
    const englishContent = contentResults.find(r => r.language === 'en');
    const keyPoints = extractKeyPoints(englishContent?.content.content || '');
    
    videoPath = await renderVideo({
      title: keyword,
      content: keyPoints,
      style: videoConfig.style,
      platform: videoConfig.platform
    }, `./output/${keyword.replace(/\s+/g, '-')}-video.mp4`);
  }
  
  return {
    research,
    articles: contentResults,
    video: videoPath,
    metadata: {
      keyword,
      generatedAt: new Date().toISOString(),
      format: contentFormat
    }
  };
}

function extractKeyPoints(content: string, maxPoints: number = 5): string[] {
  // Simple extraction - in production, use more sophisticated NLP
  const lines = content.split('\n').filter(line => line.trim().length > 0);
  const points = lines
    .filter(line => /^\d+\./.test(line.trim()) || line.includes('•'))
    .slice(0, maxPoints);
  
  return points.map(p => p.replace(/^\d+\.\s*/, '').replace(/^•\s*/, '').trim());
}

// Usage: Complete automation
const result = await runContentPipeline({
  keyword: 'AI Marketing Automation 2024',
  contentFormat: 'toplist',
  languages: ['vi', 'en'],
  generateVideo: true,
  videoConfig: {
    style: 'infographic',
    platform: 'reels'
  }
});

console.log('✅ Pipeline complete!');
console.log('Articles:', result.articles.length);
console.log('Video:', result.video);
```

## Common Patterns

### Pattern 1: Scheduled Content Generation

```typescript
// lib/scheduler/cron.ts
import cron from 'node-cron';

// Run daily at 9 AM
cron.schedule('0 9 * * *', async () => {
  const topics = ['AI trends', 'Marketing automation', 'Social media'];
  
  for (const topic of topics) {
    try {
      await runContentPipeline({
        keyword: topic,
        contentFormat: 'toplist',
        languages: ['vi', 'en'],
        generateVideo: true,
        videoConfig: { style: 'infographic', platform: 'reels' }
      });
    } catch (error) {
      console.error(`Failed for ${topic}:`, error);
    }
  }
});
```

### Pattern 2: Batch Processing with Queue

```typescript
// lib/queue/processor.ts
interface QueueItem {
  id: string;
  keyword: string;
  config: PipelineConfig;
  status: 'pending' | 'processing' | 'completed' | 'failed';
}

class ContentQueue {
  private queue: QueueItem[] = [];
  private processing = false;
  
  async add(keyword: string, config: PipelineConfig) {
    const item: QueueItem = {
      id: Date.now().toString(),
      keyword,
      config,
      status: 'pending'
    };
    
    this.queue.push(item);
    this.processNext();
    
    return item.id;
  }
  
  private async processNext() {
    if (this.processing || this.queue.length === 0) return;
    
    this.processing = true;
    const item = this.queue.find(i => i.status === 'pending');
    
    if (!item) {
      this.processing = false;
      return;
    }
    
    item.status = 'processing';
    
    try {
      await runContentPipeline(item.config);
      item.status = 'completed';
    } catch (error) {
      item.status = 'failed';
      console.error(`Queue item ${item.id} failed:`, error);
    }
    
    this.processing = false;
    this.processNext();
  }
}

export const contentQueue = new ContentQueue();
```

### Pattern 3: API Route Integration (Next.js)

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline/workflow';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const { keyword, format = 'toplist', languages = ['vi', 'en'] } = body;
    
    if (!keyword) {
      return NextResponse.json(
        { error: 'Keyword is required' },
        { status: 400 }
      );
    }
    
    const result = await runContentPipeline({
      keyword,
      contentFormat: format,
      languages,
      generateVideo: false
    });
    
    return NextResponse.json({
      success: true,
      data: result
    });
    
  } catch (error) {
    console.error('API error:', error);
    return NextResponse.json(
      { error: 'Content generation failed' },
      { status: 500 }
    );
  }
}
```

## Configuration

### AI Provider Selection

```typescript
// lib/config/ai-provider.ts
export const AI_CONFIG = {
  provider: process.env.AI_PROVIDER || 'claude', // 'claude' or 'openai'
  models: {
    claude: 'claude-3-5-sonnet-20241022',
    openai: 'gpt-4-turbo-preview'
  },
  maxTokens: {
    claude: 4096,
    openai: 4096
  },
  temperature: 0.7
};

async function generateWithProvider(request: ContentRequest) {
  const provider = AI_CONFIG.provider;
  
  if (provider === 'claude') {
    return await generateContent(request);
  } else {
    return await generateWithOpenAI(request);
  }
}
```

### Research Source Configuration

```typescript
// lib/config/research.ts
export const RESEARCH_CONFIG = {
  sources: {
    techcrunch: {
      enabled: true,
      apiKey: process.env.RAPIDAPI_KEY,
      endpoint: 'https://techcrunch.com/api/v1'
    },
    a16z: {
      enabled: true,
      rssUrl: 'https://a16z.com/feed/'
    },
    twitter: {
      enabled: Boolean(process.env.TWITTER_BEARER_TOKEN),
      bearerToken: process.env.TWITTER_BEARER_TOKEN
    }
  },
  defaultTimeRange: '24h',
  maxResultsPerSource: 10
};
```

## Troubleshooting

### Issue: API Rate Limits

```typescript
// lib/utils/rate-limiter.ts
class RateLimiter {
  private requests: number[] = [];
  private maxRequests: number;
  private windowMs: number;
  
  constructor(maxRequests = 10, windowMs = 60000) {
    this.maxRequests = maxRequests;
    this.windowMs = windowMs;
  }
  
  async acquire() {
    const now = Date.now();
    this.requests = this.requests.filter(time => now - time < this.windowMs);
    
    if (this.requests.length >= this.maxRequests) {
      const oldestRequest = this.requests[0];
      const waitTime = this.windowMs - (now - oldestRequest);
      await new Promise(resolve => setTimeout(resolve, waitTime));
    }
    
    this.requests.push(now);
  }
}

export const claudeLimiter = new RateLimiter(50, 60000); // 50 req/min
export const openaiLimiter = new RateLimiter(60, 60000); // 60 req/min

// Usage in generate function
await claudeLimiter.acquire();
const response = await anthropic.messages.create({...});
```

### Issue: Video Rendering Timeout

```typescript
// Increase timeout for large videos
await renderMedia({
  composition,
  serveUrl: bundleLocation,
  codec: 'h264',
  outputLocation: outputPath,
  timeoutInMilliseconds: 300000, // 5 minutes
  chromiumOptions: {
    headless: true,
    gl: 'angle' // Better compatibility
  }
});
```

### Issue: Memory Leaks in Long-Running Processes

```typescript
// Use proper cleanup
process.on('SIGTERM', async () => {
  console.log('Shutting down gracefully...');
  // Close database connections
  // Cancel pending operations
  process.exit(0);
});

// Limit concurrent operations
import pLimit from 'p-limit';
const limit = pLimit(3); // Max 3 concurrent pipelines

const results = await Promise.all(
  topics.map(topic => limit(() => runContentPipeline(topic)))
);
```

### Issue: Missing Environment Variables

```typescript
// lib/utils/env-validator.ts
const requiredEnvVars = [
  'ANTHROPIC_API_KEY',
  'OPENAI_API_KEY',
  'RAPIDAPI_KEY'
];

export function validateEnv() {
  const missing = requiredEnvVars.filter(key => !process.env[key]);
  
  if (missing.length > 0) {
    throw new Error(
      `Missing required environment variables: ${missing.join(', ')}\n` +
      'Please check your .env.local file.'
    );
  }
}

// Call at app startup
validateEnv();
```

## Best Practices

1. **Always use environment variables** for API keys and secrets
2. **Implement rate limiting** to avoid API quota exhaustion
3. **Cache research results** to reduce redundant API calls
4. **Use TypeScript strict mode** for better type safety
5. **Handle errors gracefully** with proper logging
6. **Monitor token usage** to control AI API costs
7. **Batch similar requests** to optimize performance
8. **Store generated content** in a database for reuse

This skill enables comprehensive automation of content marketing workflows using state-of-the-art AI and video rendering technologies.
