---
name: marketing-pipeline-ai-content-automation
description: Automated AI content pipeline for research, script generation, and video creation using Claude/OpenAI and Remotion
triggers:
  - create automated content pipeline with AI
  - set up AI content generation workflow
  - automate content research and video generation
  - build marketing content automation system
  - generate AI-powered marketing videos
  - create content from research to video automatically
  - set up Claude and OpenAI content pipeline
  - automate social media content creation with AI
---

# Marketing Pipeline AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the **Ultimate AI Content Pipeline**, a comprehensive TypeScript-based system that automates content creation from research to video generation. The pipeline crawls news sources, generates multi-format content using Claude/OpenAI, and renders videos automatically using Remotion.

## What This Project Does

The Marketing Pipeline is an end-to-end content automation system that:

- **Auto-scans research sources** (TechCrunch, a16z, Twitter/X, LinkedIn) for trending topics
- **Generates content** in multiple formats (toplist, POV, case study, how-to) using AI
- **Supports bilingual output** (English and Vietnamese) with customizable tone
- **Renders videos and infographics** automatically using Remotion
- **Optimizes for social platforms** (Reels, TikTok, Shorts)

## Installation

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

## Environment Configuration

Create a `.env.local` file in the project root:

```bash
# AI Provider Keys
ANTHROPIC_API_KEY=your_claude_api_key_here
OPENAI_API_KEY=your_openai_api_key_here

# Research/Crawling APIs
RAPIDAPI_KEY=your_rapidapi_key_here

# Remotion (Video Rendering)
REMOTION_LICENSE_KEY=your_remotion_license_key_here

# Next.js Configuration
NEXT_PUBLIC_API_URL=http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/             # Core libraries
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── crawler/     # Content crawling modules
│   │   ├── generator/   # Content generation logic
│   │   └── video/       # Remotion video rendering
│   └── utils/           # Utility functions
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Core API Usage

### 1. Research & Content Crawling

```typescript
import { crawlNewsSource } from '@/lib/crawler/news-crawler';
import { analyzeContent } from '@/lib/ai/content-analyzer';

// Crawl recent news from specified sources
async function gatherResearch(keyword: string, sources: string[] = ['techcrunch', 'a16z']) {
  const articles = await Promise.all(
    sources.map(source => 
      crawlNewsSource({
        source,
        keyword,
        timeRange: '24h',
        limit: 10
      })
    )
  );
  
  // Flatten and analyze
  const allArticles = articles.flat();
  const insights = await analyzeContent(allArticles, {
    provider: 'claude', // or 'openai'
    model: 'claude-3-sonnet-20240229'
  });
  
  return { articles: allArticles, insights };
}
```

### 2. AI Content Generation

```typescript
import { generateContent } from '@/lib/generator/content-generator';
import { ContentFormat, ContentTone, Language } from '@/lib/types';

// Generate content using Claude or OpenAI
async function createArticle(topic: string, researchData: any) {
  const content = await generateContent({
    provider: 'claude', // or 'openai'
    apiKey: process.env.ANTHROPIC_API_KEY,
    prompt: {
      topic,
      format: ContentFormat.POV, // TOPLIST, CASE_STUDY, HOW_TO
      tone: ContentTone.PROFESSIONAL, // FRIENDLY, HUMOROUS
      language: Language.BILINGUAL, // ENGLISH, VIETNAMESE
      researchData
    },
    maxTokens: 4000
  });
  
  return content;
}
```

### 3. Multi-Format Content Generation

```typescript
import { ContentPipeline } from '@/lib/generator/pipeline';

const pipeline = new ContentPipeline({
  anthropicKey: process.env.ANTHROPIC_API_KEY,
  openaiKey: process.env.OPENAI_API_KEY
});

// Generate multiple formats from single topic
async function generateMultiFormat(keyword: string) {
  const formats = ['toplist', 'pov', 'case-study', 'how-to'];
  
  const results = await pipeline.generateBatch({
    keyword,
    formats,
    languages: ['en', 'vi'],
    includeResearch: true,
    sources: ['techcrunch', 'linkedin', 'twitter']
  });
  
  return results; // Array of generated content in all formats
}
```

### 4. Video Rendering with Remotion

```typescript
import { renderVideo } from '@/lib/video/renderer';
import { VideoTemplate } from '@/lib/video/templates';

// Render video from generated content
async function createContentVideo(content: any) {
  const videoConfig = {
    template: VideoTemplate.INFOGRAPHIC, // or SHORTS, REELS
    content: {
      title: content.title,
      keyPoints: content.keyPoints,
      images: content.generatedImages,
      duration: 60 // seconds
    },
    style: {
      aspectRatio: '9:16', // For TikTok/Reels
      branding: {
        logo: '/logo.png',
        colors: ['#FF6B6B', '#4ECDC4']
      }
    }
  };
  
  const videoPath = await renderVideo(videoConfig, {
    outputFormat: 'mp4',
    quality: 'high',
    codec: 'h264'
  });
  
  return videoPath;
}
```

## Common Workflow Patterns

### Complete Content Pipeline

```typescript
import { ContentAutomationPipeline } from '@/lib/pipeline';

// Full automation: Research → Generate → Render
async function runFullPipeline(keyword: string) {
  const pipeline = new ContentAutomationPipeline({
    anthropicKey: process.env.ANTHROPIC_API_KEY,
    openaiKey: process.env.OPENAI_API_KEY,
    rapidApiKey: process.env.RAPIDAPI_KEY
  });
  
  // Step 1: Research
  const research = await pipeline.research({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter'],
    depth: 'comprehensive'
  });
  
  // Step 2: Generate Content
  const content = await pipeline.generate({
    research,
    formats: ['pov', 'toplist'],
    languages: ['en', 'vi'],
    tone: 'professional'
  });
  
  // Step 3: Render Videos
  const videos = await pipeline.renderVideos({
    content,
    templates: ['reels', 'shorts'],
    watermark: true
  });
  
  return { research, content, videos };
}
```

### Scheduled Content Generation

```typescript
import { CronJob } from 'cron';
import { ContentScheduler } from '@/lib/scheduler';

// Schedule daily content generation
const scheduler = new ContentScheduler({
  pipeline: new ContentAutomationPipeline({ /* config */ })
});

// Run every day at 9 AM
const dailyJob = new CronJob('0 9 * * *', async () => {
  const topics = await scheduler.getTrendingTopics();
  
  for (const topic of topics) {
    const result = await runFullPipeline(topic);
    await scheduler.publishContent(result, {
      platforms: ['facebook', 'linkedin', 'twitter']
    });
  }
});

dailyJob.start();
```

### Custom AI Provider Configuration

```typescript
import { AIProviderFactory } from '@/lib/ai/provider-factory';

// Switch between Claude and OpenAI dynamically
const aiProvider = AIProviderFactory.create({
  primary: {
    type: 'claude',
    apiKey: process.env.ANTHROPIC_API_KEY,
    model: 'claude-3-sonnet-20240229'
  },
  fallback: {
    type: 'openai',
    apiKey: process.env.OPENAI_API_KEY,
    model: 'gpt-4-turbo-preview'
  },
  rateLimits: {
    requestsPerMinute: 50,
    tokensPerMinute: 100000
  }
});

const generated = await aiProvider.generate({
  prompt: 'Write a marketing article about...',
  maxTokens: 2000
});
```

## Development Commands

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Render Remotion video (CLI)
npm run remotion:render

# Type checking
npm run type-check

# Linting
npm run lint
```

## API Routes (Next.js)

### Research Endpoint

```typescript
// app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { gatherResearch } from '@/lib/crawler/news-crawler';

export async function POST(req: NextRequest) {
  const { keyword, sources } = await req.json();
  
  const result = await gatherResearch(keyword, sources);
  
  return NextResponse.json(result);
}
```

### Content Generation Endpoint

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { generateContent } from '@/lib/generator/content-generator';

export async function POST(req: NextRequest) {
  const { topic, format, language, research } = await req.json();
  
  const content = await generateContent({
    provider: 'claude',
    apiKey: process.env.ANTHROPIC_API_KEY,
    prompt: { topic, format, language, researchData: research }
  });
  
  return NextResponse.json({ content });
}
```

### Video Render Endpoint

```typescript
// app/api/video/render/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { renderVideo } from '@/lib/video/renderer';

export async function POST(req: NextRequest) {
  const { content, template } = await req.json();
  
  const videoPath = await renderVideo({
    template,
    content,
    style: { aspectRatio: '9:16' }
  });
  
  return NextResponse.json({ videoPath });
}
```

## Troubleshooting

### API Rate Limiting

```typescript
import { RateLimiter } from '@/lib/utils/rate-limiter';

const limiter = new RateLimiter({
  maxRequests: 50,
  windowMs: 60000 // 1 minute
});

async function safeAPICall(fn: Function) {
  await limiter.waitForToken();
  try {
    return await fn();
  } catch (error) {
    if (error.status === 429) {
      // Wait and retry
      await new Promise(resolve => setTimeout(resolve, 5000));
      return await fn();
    }
    throw error;
  }
}
```

### Video Rendering Memory Issues

```typescript
// Reduce memory usage for large video renders
const videoConfig = {
  // ... other config
  remotionConfig: {
    concurrency: 1, // Render one frame at a time
    imageFormat: 'jpeg', // Use JPEG instead of PNG
    scale: 0.8, // Reduce resolution slightly
    enforceAudioTrack: false
  }
};
```

### Content Quality Control

```typescript
import { validateContent } from '@/lib/utils/content-validator';

async function generateQualityContent(topic: string) {
  let attempts = 0;
  const maxAttempts = 3;
  
  while (attempts < maxAttempts) {
    const content = await generateContent({ topic });
    
    const validation = validateContent(content, {
      minWords: 500,
      requireSources: true,
      checkGrammar: true,
      checkPlagiarism: true
    });
    
    if (validation.passed) return content;
    
    attempts++;
    // Refine prompt based on validation errors
  }
  
  throw new Error('Failed to generate quality content');
}
```

### Crawler Blocking Prevention

```typescript
import { ProxyRotator } from '@/lib/crawler/proxy';
import { UserAgentRotator } from '@/lib/crawler/user-agent';

const crawler = {
  proxy: new ProxyRotator({ /* proxy list */ }),
  userAgent: new UserAgentRotator(),
  
  async fetch(url: string) {
    return fetch(url, {
      headers: {
        'User-Agent': this.userAgent.getRandom(),
      },
      // Use proxy if available
      ...(this.proxy.getNext() && { agent: this.proxy.getNext() })
    });
  }
};
```

## Best Practices

1. **Always validate environment variables** before running the pipeline
2. **Implement retry logic** for API calls to handle rate limits
3. **Cache research results** to avoid redundant crawling
4. **Monitor token usage** for Claude/OpenAI to control costs
5. **Use webhook callbacks** for long-running video renders
6. **Implement content moderation** before auto-publishing
7. **Version control your prompts** for consistent output quality
8. **Test video renders** with shorter durations first
