---
name: marketing-pipeline-share-automation
description: Automated AI content pipeline from research to video generation using Claude, OpenAI, and Remotion
triggers:
  - how do I set up the marketing content automation pipeline
  - automate content creation from research to video
  - use AI to generate social media content and videos
  - create automated content workflow with Claude and OpenAI
  - generate videos from blog posts automatically
  - set up content research and video pipeline
  - build automated marketing content system
  - crawl news and generate content with AI
---

# Marketing Pipeline Share Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This project is an end-to-end AI-powered content automation system that handles research, content generation, and video rendering. It crawls news sources (TechCrunch, a16z, Twitter, LinkedIn), uses Claude/OpenAI to generate multi-format content in multiple languages, and automatically renders videos using Remotion.

## What It Does

The Ultimate AI Content Pipeline automates:
- **Research Phase**: Auto-crawls news from major sources in the last 24h
- **Content Generation**: Creates articles in multiple formats (Toplist, POV, Case Study, How-to) using Claude 3 or OpenAI
- **Multi-language Support**: Generates content in English and Vietnamese simultaneously
- **Video Rendering**: Converts written content to videos/infographics via Remotion
- **Platform Optimization**: Exports videos optimized for Reels, TikTok, Shorts

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

### Environment Setup

Create a `.env.local` file in the root directory:

```bash
# AI Provider Keys
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_url

# Remotion (for video rendering)
REMOTION_LICENSE_KEY=your_remotion_key

# Next.js
NEXT_PUBLIC_API_URL=http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/           # Next.js app directory
│   ├── components/    # React components
│   ├── lib/
│   │   ├── ai/       # AI integration (Claude, OpenAI)
│   │   ├── crawler/  # News crawling logic
│   │   ├── content/  # Content generation
│   │   └── video/    # Remotion video rendering
│   └── types/        # TypeScript type definitions
├── remotion/         # Remotion video templates
└── public/           # Static assets
```

## Core APIs and Usage

### 1. News Crawling & Research

```typescript
import { crawlNews } from '@/lib/crawler/news-crawler';
import { analyzeTrends } from '@/lib/crawler/trend-analyzer';

// Crawl news from multiple sources
async function researchTopic(keyword: string) {
  const sources = ['techcrunch', 'a16z', 'twitter', 'linkedin'];
  
  const newsData = await crawlNews({
    keyword,
    sources,
    timeRange: '24h',
    maxResults: 50
  });
  
  // Analyze and extract insights
  const insights = await analyzeTrends(newsData);
  
  return {
    articles: newsData,
    insights,
    keyStats: insights.statistics
  };
}
```

### 2. AI Content Generation with Claude

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

async function generateContent(
  research: any,
  format: 'toplist' | 'pov' | 'case-study' | 'how-to',
  language: 'en' | 'vi'
) {
  const prompt = buildPrompt(research, format, language);
  
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [
      {
        role: 'user',
        content: prompt
      }
    ],
    temperature: 0.7
  });
  
  return message.content[0].text;
}

function buildPrompt(research: any, format: string, language: string) {
  const formatInstructions = {
    'toplist': 'Create a numbered list article with rankings',
    'pov': 'Write from a unique perspective or viewpoint',
    'case-study': 'Analyze a specific example with data',
    'how-to': 'Create step-by-step tutorial format'
  };
  
  return `
Based on this research data:
${JSON.stringify(research.insights, null, 2)}

Create a ${format} article in ${language === 'vi' ? 'Vietnamese' : 'English'}.
${formatInstructions[format]}

Include:
- Data-backed insights
- Recent statistics (within 24h)
- Actionable takeaways
- Engaging headline

Output format: Markdown
  `.trim();
}
```

### 3. OpenAI Alternative

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateWithOpenAI(research: any, config: any) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: 'You are an expert content creator specializing in marketing and social media.'
      },
      {
        role: 'user',
        content: buildPrompt(research, config.format, config.language)
      }
    ],
    temperature: 0.7,
    max_tokens: 4000
  });
  
  return completion.choices[0].message.content;
}
```

### 4. Multi-language Content Generation

```typescript
interface ContentRequest {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  tone: 'expert' | 'friendly' | 'humorous';
}

async function generateMultilingualContent(request: ContentRequest) {
  // Research phase
  const research = await researchTopic(request.keyword);
  
  // Generate in parallel
  const [englishContent, vietnameseContent] = await Promise.all([
    generateContent(research, request.format, 'en'),
    generateContent(research, request.format, 'vi')
  ]);
  
  return {
    en: {
      content: englishContent,
      metadata: {
        wordCount: englishContent.split(' ').length,
        format: request.format,
        sources: research.articles.length
      }
    },
    vi: {
      content: vietnameseContent,
      metadata: {
        wordCount: vietnameseContent.split(' ').length,
        format: request.format,
        sources: research.articles.length
      }
    }
  };
}
```

### 5. Video Rendering with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import { webpackOverride } from './remotion.config';

async function renderContentVideo(content: string, config: any) {
  // Prepare video data from content
  const videoData = parseContentForVideo(content);
  
  // Bundle Remotion composition
  const bundleLocation = await bundle({
    entryPoint: './remotion/index.ts',
    webpackOverride
  });
  
  // Select composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: videoData
  });
  
  // Render video
  const outputLocation = `./output/video-${Date.now()}.mp4`;
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: videoData
  });
  
  return outputLocation;
}

function parseContentForVideo(content: string) {
  // Extract key points, stats, headlines
  const sections = content.split('\n\n');
  
  return {
    title: sections[0].replace(/^#+ /, ''),
    slides: sections.slice(1, 6).map(section => ({
      text: section,
      duration: 5 // seconds
    })),
    style: {
      platform: 'reels', // or 'tiktok', 'shorts'
      aspectRatio: '9:16'
    }
  };
}
```

### 6. Complete Pipeline Example

```typescript
import { PipelineOrchestrator } from '@/lib/pipeline/orchestrator';

async function runContentPipeline(keyword: string) {
  const pipeline = new PipelineOrchestrator({
    aiProvider: 'claude', // or 'openai'
    videoEnabled: true,
    languages: ['en', 'vi']
  });
  
  try {
    // Step 1: Research
    console.log('🔍 Researching topic...');
    const research = await pipeline.research(keyword);
    
    // Step 2: Generate Content
    console.log('✍️ Generating content...');
    const content = await pipeline.generateContent({
      research,
      format: 'toplist',
      tone: 'expert',
      languages: ['en', 'vi']
    });
    
    // Step 3: Render Videos
    console.log('🎬 Rendering videos...');
    const videos = await pipeline.renderVideos(content, {
      platforms: ['reels', 'tiktok', 'shorts']
    });
    
    // Step 4: Package output
    return {
      content: content,
      videos: videos,
      metadata: {
        keyword,
        createdAt: new Date(),
        sources: research.articles.length
      }
    };
    
  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}

// Usage
runContentPipeline('AI marketing automation')
  .then(result => {
    console.log('✅ Pipeline complete!');
    console.log('Content pieces:', Object.keys(result.content).length);
    console.log('Videos:', result.videos.length);
  });
```

## Next.js API Routes

### Research Endpoint

```typescript
// src/app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { crawlNews } from '@/lib/crawler/news-crawler';

export async function POST(request: NextRequest) {
  const { keyword, sources } = await request.json();
  
  try {
    const results = await crawlNews({
      keyword,
      sources: sources || ['techcrunch', 'a16z'],
      timeRange: '24h'
    });
    
    return NextResponse.json(results);
  } catch (error) {
    return NextResponse.json(
      { error: 'Research failed' },
      { status: 500 }
    );
  }
}
```

### Content Generation Endpoint

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { generateMultilingualContent } from '@/lib/content/generator';

export async function POST(request: NextRequest) {
  const { keyword, format, tone } = await request.json();
  
  try {
    const content = await generateMultilingualContent({
      keyword,
      format,
      tone
    });
    
    return NextResponse.json(content);
  } catch (error) {
    return NextResponse.json(
      { error: 'Generation failed' },
      { status: 500 }
    );
  }
}
```

### Video Rendering Endpoint

```typescript
// src/app/api/render/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { renderContentVideo } from '@/lib/video/renderer';

export async function POST(request: NextRequest) {
  const { content, platform } = await request.json();
  
  try {
    const videoPath = await renderContentVideo(content, {
      platform,
      aspectRatio: platform === 'reels' ? '9:16' : '16:9'
    });
    
    return NextResponse.json({ videoPath });
  } catch (error) {
    return NextResponse.json(
      { error: 'Rendering failed' },
      { status: 500 }
    );
  }
}
```

## Common Patterns

### Error Handling

```typescript
class PipelineError extends Error {
  constructor(
    message: string,
    public stage: 'research' | 'generation' | 'rendering',
    public originalError?: Error
  ) {
    super(message);
    this.name = 'PipelineError';
  }
}

async function safeExecute<T>(
  fn: () => Promise<T>,
  stage: string
): Promise<T> {
  try {
    return await fn();
  } catch (error) {
    throw new PipelineError(
      `Failed at ${stage}`,
      stage as any,
      error as Error
    );
  }
}
```

### Rate Limiting

```typescript
import { Ratelimit } from '@upstash/ratelimit';
import { Redis } from '@upstash/redis';

const ratelimit = new Ratelimit({
  redis: Redis.fromEnv(),
  limiter: Ratelimit.slidingWindow(10, '1 h'),
});

async function checkRateLimit(identifier: string) {
  const { success, limit, reset, remaining } = 
    await ratelimit.limit(identifier);
    
  if (!success) {
    throw new Error(`Rate limit exceeded. Reset at ${reset}`);
  }
  
  return { remaining, reset };
}
```

### Caching Research Results

```typescript
import { Redis } from 'ioredis';

const redis = new Redis(process.env.REDIS_URL);

async function getCachedResearch(keyword: string) {
  const cacheKey = `research:${keyword}`;
  const cached = await redis.get(cacheKey);
  
  if (cached) {
    return JSON.parse(cached);
  }
  
  const fresh = await researchTopic(keyword);
  
  // Cache for 1 hour
  await redis.setex(cacheKey, 3600, JSON.stringify(fresh));
  
  return fresh;
}
```

## Development Commands

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Run Remotion preview
npm run remotion:preview

# Render a specific composition
npm run remotion:render -- ContentVideo output.mp4

# Type checking
npm run type-check

# Linting
npm run lint
```

## Configuration

### Remotion Config

```typescript
// remotion.config.ts
import { Config } from '@remotion/cli/config';

Config.setVideoImageFormat('jpeg');
Config.setOverwriteOutput(true);
Config.setConcurrency(4);
Config.setCodec('h264');
```

### AI Provider Selection

```typescript
// src/lib/config/ai-config.ts
export const aiConfig = {
  provider: process.env.AI_PROVIDER || 'claude',
  models: {
    claude: 'claude-3-5-sonnet-20241022',
    openai: 'gpt-4-turbo-preview'
  },
  maxTokens: 4096,
  temperature: 0.7
};
```

## Troubleshooting

### API Key Issues

```typescript
// Validate API keys on startup
function validateEnvironment() {
  const required = [
    'OPENAI_API_KEY',
    'ANTHROPIC_API_KEY',
    'RAPIDAPI_KEY'
  ];
  
  const missing = required.filter(key => !process.env[key]);
  
  if (missing.length > 0) {
    throw new Error(
      `Missing required environment variables: ${missing.join(', ')}`
    );
  }
}

validateEnvironment();
```

### Memory Issues with Video Rendering

```typescript
// Increase Node.js memory
// In package.json
{
  "scripts": {
    "remotion:render": "NODE_OPTIONS='--max-old-space-size=4096' remotion render"
  }
}
```

### Crawler Blocking

```typescript
// Add delays and user agents
const crawlerConfig = {
  delay: 2000, // ms between requests
  userAgent: 'Mozilla/5.0 (compatible; ContentBot/1.0)',
  maxRetries: 3,
  timeout: 10000
};

async function crawlWithRetry(url: string) {
  for (let i = 0; i < crawlerConfig.maxRetries; i++) {
    try {
      await new Promise(r => setTimeout(r, crawlerConfig.delay));
      return await fetch(url, {
        headers: { 'User-Agent': crawlerConfig.userAgent }
      });
    } catch (error) {
      if (i === crawlerConfig.maxRetries - 1) throw error;
    }
  }
}
```

### Content Quality Issues

```typescript
// Add validation layer
function validateGeneratedContent(content: string): boolean {
  const checks = {
    minLength: content.length > 500,
    hasHeading: /^#+ /.test(content),
    hasStats: /\d+%|\d+\s*(users|people|companies)/.test(content),
    notEmpty: content.trim().length > 0
  };
  
  return Object.values(checks).every(Boolean);
}

async function generateValidContent(research: any, config: any) {
  let attempts = 0;
  const maxAttempts = 3;
  
  while (attempts < maxAttempts) {
    const content = await generateContent(research, config.format, config.language);
    
    if (validateGeneratedContent(content)) {
      return content;
    }
    
    attempts++;
  }
  
  throw new Error('Failed to generate valid content');
}
```
