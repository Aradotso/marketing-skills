---
name: marketing-pipeline-share-content-automation
description: Automate content creation from research to video generation using AI-powered pipeline with Claude, OpenAI, and Remotion
triggers:
  - automate content creation pipeline
  - generate AI content from research to video
  - create automated marketing content with AI
  - build content pipeline with Claude and OpenAI
  - auto-generate blog posts and videos from keywords
  - set up AI content automation system
  - create content workflow with Remotion videos
  - automate research and content generation
---

# Marketing Pipeline Share - Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

Marketing Pipeline Share is an end-to-end automated content creation system that transforms keywords into complete content pieces including articles, infographics, and videos. The pipeline automatically researches current news from sources like TechCrunch, a16z, Twitter/X, and LinkedIn, generates content in multiple formats and languages (English/Vietnamese), and renders videos using Remotion.

**Key capabilities:**
- Auto-scrape and research latest news (24h data)
- Generate content in multiple formats (Toplist, POV, Case Study, How-to)
- Support bilingual content (EN/VN) with customizable tone
- Auto-render videos and infographics via Remotion
- Next.js-based UI for easy workflow management

## Installation

### Prerequisites

- Node.js 18+ and npm/yarn/pnpm
- API keys for:
  - OpenAI (GPT-4/GPT-3.5)
  - Anthropic Claude (Claude 3)
  - RapidAPI (for web scraping)

### Setup

```bash
# Clone repository
git clone https://github.com/pennydinh/marketing-pipeline-share.git
cd marketing-pipeline-share

# Install dependencies
npm install
# or
pnpm install

# Set up environment variables
cp .env.example .env.local
```

### Environment Configuration

Create `.env.local` with the following:

```bash
# AI Services
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_anthropic_key

# Research/Scraping
RAPIDAPI_KEY=your_rapidapi_key

# Database (if using)
DATABASE_URL=your_database_connection_string

# Remotion (Video Generation)
REMOTION_AWS_ACCESS_KEY_ID=your_aws_access_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret_key

# Application
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

### Run Development Server

```bash
npm run dev
# or
pnpm dev

# Open http://localhost:3000
```

## Core Components

### 1. Research Module (Auto-Scan)

The research module crawls and analyzes content from multiple sources:

```typescript
// lib/research/scanner.ts
import { OpenAI } from 'openai';

interface ResearchConfig {
  keyword: string;
  sources: string[];
  timeRange: '24h' | '7d' | '30d';
  language: 'en' | 'vi';
}

export async function performResearch(config: ResearchConfig) {
  const { keyword, sources, timeRange } = config;
  
  // Fetch data from sources
  const rawData = await Promise.all(
    sources.map(source => fetchFromSource(source, keyword, timeRange))
  );
  
  // Analyze and extract insights
  const insights = await analyzeData(rawData);
  
  return {
    keyword,
    sources: rawData,
    insights,
    timestamp: new Date().toISOString()
  };
}

async function fetchFromSource(
  source: string, 
  keyword: string, 
  timeRange: string
) {
  const response = await fetch('/api/scrape', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ source, keyword, timeRange })
  });
  
  return response.json();
}
```

### 2. Content Generation

Generate content in multiple formats using Claude or OpenAI:

```typescript
// lib/content/generator.ts
import Anthropic from '@anthropic-ai/sdk';

interface ContentConfig {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  researchData: any;
}

export async function generateContent(config: ContentConfig) {
  const anthropic = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY,
  });

  const prompt = buildPrompt(config);
  
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [
      {
        role: 'user',
        content: prompt
      }
    ]
  });

  return parseContent(message.content);
}

function buildPrompt(config: ContentConfig): string {
  const { keyword, format, language, tone, researchData } = config;
  
  const formatInstructions = {
    'toplist': 'Create a numbered list article with rankings',
    'pov': 'Write from a unique perspective or opinion',
    'case-study': 'Analyze a real-world example with data',
    'how-to': 'Create a step-by-step tutorial guide'
  };
  
  return `
You are a professional content writer. Create ${language} content about: ${keyword}

Format: ${formatInstructions[format]}
Tone: ${tone}
Research Data: ${JSON.stringify(researchData)}

Requirements:
- Use recent data and insights from research
- Include statistics and sources
- Make it engaging and actionable
- Structure with clear headings and sections
`;
}
```

### 3. Video Generation with Remotion

```typescript
// lib/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  title: string;
  sections: Array<{
    heading: string;
    content: string;
    duration: number;
  }>;
  style: 'infographic' | 'talking-points' | 'slides';
}

export async function renderContentVideo(
  config: VideoConfig,
  outputPath: string
) {
  // Bundle Remotion composition
  const bundleLocation = await bundle(
    path.join(process.cwd(), 'remotion/index.ts')
  );

  // Get composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title: config.title,
      sections: config.sections,
      style: config.style
    }
  });

  // Render video
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: {
      title: config.title,
      sections: config.sections,
      style: config.style
    }
  });

  return outputPath;
}
```

### 4. Pipeline Orchestration

Complete workflow from keyword to published content:

```typescript
// lib/pipeline/orchestrator.ts
export interface PipelineConfig {
  keyword: string;
  contentFormat: 'toplist' | 'pov' | 'case-study' | 'how-to';
  languages: Array<'en' | 'vi'>;
  generateVideo: boolean;
  autoPublish: boolean;
}

export async function runContentPipeline(config: PipelineConfig) {
  const pipeline = {
    status: 'running',
    steps: [] as any[]
  };

  try {
    // Step 1: Research
    pipeline.steps.push({ step: 'research', status: 'running' });
    const researchData = await performResearch({
      keyword: config.keyword,
      sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
      timeRange: '24h',
      language: config.languages[0]
    });
    pipeline.steps[0].status = 'completed';
    pipeline.steps[0].data = researchData;

    // Step 2: Generate content for each language
    const contents = [];
    for (const language of config.languages) {
      pipeline.steps.push({ 
        step: `content-${language}`, 
        status: 'running' 
      });
      
      const content = await generateContent({
        keyword: config.keyword,
        format: config.contentFormat,
        language,
        tone: 'expert',
        researchData
      });
      
      contents.push({ language, content });
      pipeline.steps[pipeline.steps.length - 1].status = 'completed';
      pipeline.steps[pipeline.steps.length - 1].data = content;
    }

    // Step 3: Generate video (optional)
    if (config.generateVideo) {
      pipeline.steps.push({ step: 'video', status: 'running' });
      
      const videoPath = await renderContentVideo({
        title: config.keyword,
        sections: contents[0].content.sections,
        style: 'infographic'
      }, `./output/${Date.now()}-video.mp4`);
      
      pipeline.steps[pipeline.steps.length - 1].status = 'completed';
      pipeline.steps[pipeline.steps.length - 1].data = { videoPath };
    }

    // Step 4: Auto-publish (optional)
    if (config.autoPublish) {
      pipeline.steps.push({ step: 'publish', status: 'running' });
      
      const publishResults = await publishContent(contents);
      
      pipeline.steps[pipeline.steps.length - 1].status = 'completed';
      pipeline.steps[pipeline.steps.length - 1].data = publishResults;
    }

    pipeline.status = 'completed';
    return pipeline;

  } catch (error) {
    pipeline.status = 'failed';
    pipeline.error = error.message;
    throw error;
  }
}
```

## API Routes

### Create Pipeline Job

```typescript
// app/api/pipeline/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline/orchestrator';

export async function POST(request: NextRequest) {
  try {
    const config = await request.json();
    
    // Validate config
    if (!config.keyword) {
      return NextResponse.json(
        { error: 'Keyword is required' },
        { status: 400 }
      );
    }

    // Run pipeline
    const result = await runContentPipeline(config);

    return NextResponse.json(result);
  } catch (error) {
    return NextResponse.json(
      { error: error.message },
      { status: 500 }
    );
  }
}
```

### Get Pipeline Status

```typescript
// app/api/pipeline/[id]/route.ts
import { NextRequest, NextResponse } from 'next/server';

export async function GET(
  request: NextRequest,
  { params }: { params: { id: string } }
) {
  const pipelineId = params.id;
  
  // Fetch pipeline status from database
  const pipeline = await db.pipeline.findUnique({
    where: { id: pipelineId },
    include: { steps: true }
  });

  if (!pipeline) {
    return NextResponse.json(
      { error: 'Pipeline not found' },
      { status: 404 }
    );
  }

  return NextResponse.json(pipeline);
}
```

## Usage Examples

### Basic Content Generation

```typescript
import { runContentPipeline } from '@/lib/pipeline/orchestrator';

// Simple blog post generation
const result = await runContentPipeline({
  keyword: 'AI Marketing Trends 2024',
  contentFormat: 'toplist',
  languages: ['en', 'vi'],
  generateVideo: false,
  autoPublish: false
});

console.log('Content generated:', result);
```

### Full Pipeline with Video

```typescript
// Complete automation with video rendering
const fullPipeline = await runContentPipeline({
  keyword: 'Top 10 SaaS Tools for Startups',
  contentFormat: 'toplist',
  languages: ['en'],
  generateVideo: true,
  autoPublish: true
});

console.log('Pipeline completed:', fullPipeline.status);
console.log('Video path:', fullPipeline.steps.find(s => s.step === 'video').data.videoPath);
```

### Custom Research Sources

```typescript
import { performResearch } from '@/lib/research/scanner';

const customResearch = await performResearch({
  keyword: 'blockchain adoption',
  sources: ['techcrunch', 'coindesk', 'twitter'],
  timeRange: '7d',
  language: 'en'
});

// Use research for manual content creation
const content = await generateContent({
  keyword: 'blockchain adoption',
  format: 'case-study',
  language: 'en',
  tone: 'expert',
  researchData: customResearch
});
```

## Common Patterns

### Batch Content Generation

```typescript
async function generateMultipleTopics(keywords: string[]) {
  const results = await Promise.allSettled(
    keywords.map(keyword => 
      runContentPipeline({
        keyword,
        contentFormat: 'how-to',
        languages: ['en', 'vi'],
        generateVideo: true,
        autoPublish: false
      })
    )
  );

  return results.map((result, index) => ({
    keyword: keywords[index],
    status: result.status,
    data: result.status === 'fulfilled' ? result.value : null,
    error: result.status === 'rejected' ? result.reason : null
  }));
}
```

### Scheduled Content Pipeline

```typescript
import cron from 'node-cron';

// Run daily at 9 AM
cron.schedule('0 9 * * *', async () => {
  const trendingTopics = await fetchTrendingTopics();
  
  for (const topic of trendingTopics) {
    await runContentPipeline({
      keyword: topic,
      contentFormat: 'pov',
      languages: ['en'],
      generateVideo: true,
      autoPublish: true
    });
  }
});
```

### Error Handling and Retry Logic

```typescript
async function runPipelineWithRetry(
  config: PipelineConfig,
  maxRetries = 3
) {
  let lastError;
  
  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      return await runContentPipeline(config);
    } catch (error) {
      lastError = error;
      console.log(`Attempt ${attempt} failed:`, error.message);
      
      if (attempt < maxRetries) {
        // Exponential backoff
        await new Promise(resolve => 
          setTimeout(resolve, Math.pow(2, attempt) * 1000)
        );
      }
    }
  }
  
  throw lastError;
}
```

## Troubleshooting

### API Rate Limits

If you encounter rate limit errors:

```typescript
// Add rate limiting middleware
import rateLimit from 'express-rate-limit';

const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // limit each IP to 100 requests per windowMs
  message: 'Too many requests, please try again later'
});

// Apply to API routes
app.use('/api/', limiter);
```

### Video Rendering Failures

Common Remotion issues:

```typescript
// Check Remotion configuration
// remotion.config.ts
import { Config } from '@remotion/cli/config';

Config.setVideoImageFormat('jpeg');
Config.setOverwriteOutput(true);
Config.setConcurrency(2); // Reduce if memory issues

// Increase timeout for long renders
Config.setTimeoutInMilliseconds(300000); // 5 minutes
```

### Memory Issues with Large Content

```typescript
// Stream large content processing
import { pipeline } from 'stream/promises';

async function processLargeContent(content: string) {
  // Split into chunks
  const chunks = content.match(/.{1,1000}/g) || [];
  
  const processed = [];
  for (const chunk of chunks) {
    const result = await processChunk(chunk);
    processed.push(result);
    
    // Allow garbage collection
    await new Promise(resolve => setImmediate(resolve));
  }
  
  return processed.join('');
}
```

### Debug Mode

Enable verbose logging:

```typescript
// lib/utils/logger.ts
export function createLogger(module: string) {
  return {
    debug: (msg: string, data?: any) => {
      if (process.env.DEBUG === 'true') {
        console.log(`[${module}] ${msg}`, data || '');
      }
    },
    error: (msg: string, error?: any) => {
      console.error(`[${module}] ERROR: ${msg}`, error);
    }
  };
}

// Usage
const logger = createLogger('pipeline');
logger.debug('Starting research phase', { keyword: 'AI trends' });
```

### Database Connection Issues

```typescript
// lib/db/client.ts
import { PrismaClient } from '@prisma/client';

const globalForPrisma = global as unknown as { prisma: PrismaClient };

export const prisma =
  globalForPrisma.prisma ||
  new PrismaClient({
    log: process.env.NODE_ENV === 'development' 
      ? ['query', 'error', 'warn'] 
      : ['error'],
  });

if (process.env.NODE_ENV !== 'production') {
  globalForPrisma.prisma = prisma;
}

// Test connection
export async function testDatabaseConnection() {
  try {
    await prisma.$connect();
    console.log('✅ Database connected');
    return true;
  } catch (error) {
    console.error('❌ Database connection failed:', error);
    return false;
  }
}
```

## Best Practices

1. **Always use environment variables** for API keys and sensitive data
2. **Implement rate limiting** to avoid hitting API quotas
3. **Cache research results** to reduce redundant API calls
4. **Queue video rendering** for heavy processing tasks
5. **Monitor token usage** for AI APIs to control costs
6. **Validate user input** before starting pipeline
7. **Store generated content** in database for reuse
8. **Set up proper error tracking** (Sentry, LogRocket, etc.)
