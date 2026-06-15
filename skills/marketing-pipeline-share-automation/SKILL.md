---
name: marketing-pipeline-share-automation
description: AI-powered content automation pipeline from research to script writing to video generation using Claude, OpenAI, and Remotion
triggers:
  - automate content creation with AI
  - generate marketing content automatically
  - create videos from text with Remotion
  - scrape news and generate articles
  - build AI content pipeline
  - automate social media content workflow
  - generate multilingual marketing content
  - create AI-powered content system
---

# Marketing Pipeline Share Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

Marketing Pipeline Share is a comprehensive TypeScript-based content automation system that orchestrates the entire content creation workflow: from research and data collection to script generation and video rendering. It leverages Claude 3, OpenAI APIs, and Remotion to transform a single keyword into publication-ready content across multiple formats and languages.

**Key capabilities:**
- Auto-crawl real-time news from TechCrunch, a16z, Twitter, LinkedIn
- Generate content in multiple formats (toplist, POV, case study, how-to)
- Bilingual content creation (English & Vietnamese)
- Automatic video rendering with Remotion
- Multi-platform optimization (Reels, TikTok, Shorts)

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

## Configuration

Create a `.env.local` file in the root directory:

```env
# AI Provider APIs
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# Research & Scraping APIs
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_url

# Application Settings
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

### Environment Variables Reference

| Variable | Purpose | Required |
|----------|---------|----------|
| `OPENAI_API_KEY` | OpenAI GPT models for content generation | Yes |
| `ANTHROPIC_API_KEY` | Claude 3 for advanced content creation | Yes |
| `RAPIDAPI_KEY` | Access to news scraping APIs | Yes |
| `DATABASE_URL` | Store generated content and metadata | Optional |

## Running the Application

```bash
# Development mode
npm run dev

# Production build
npm run build
npm start

# Remotion video rendering
npm run render
```

## Core Architecture

### 1. Research Pipeline (Auto-Scan)

The research module crawls recent content from multiple sources:

```typescript
// lib/research/crawler.ts
import { RapidAPIClient } from './rapid-api-client';

interface ResearchConfig {
  keyword: string;
  sources: string[];
  timeRange: '24h' | '7d' | '30d';
  language?: 'en' | 'vi';
}

export async function performResearch(config: ResearchConfig) {
  const client = new RapidAPIClient(process.env.RAPIDAPI_KEY!);
  
  const results = await Promise.all(
    config.sources.map(source => 
      client.searchNews({
        query: config.keyword,
        source: source,
        timeRange: config.timeRange
      })
    )
  );
  
  // Aggregate and deduplicate results
  const aggregated = aggregateResults(results);
  
  // Extract insights using AI
  const insights = await extractInsights(aggregated, config.language);
  
  return {
    rawData: aggregated,
    insights: insights,
    sources: config.sources,
    timestamp: new Date()
  };
}

async function extractInsights(data: any[], language?: string) {
  const prompt = `Analyze the following news data and extract key insights, trends, and data points:
  
${JSON.stringify(data, null, 2)}

Language: ${language || 'en'}
Format: Return structured insights with categories, trends, and supporting data.`;

  // Use Claude for deep analysis
  const response = await callClaudeAPI(prompt);
  return parseInsights(response);
}
```

### 2. Content Generation

Generate content in multiple formats using AI:

```typescript
// lib/content/generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

interface ContentConfig {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  tone: 'expert' | 'friendly' | 'humorous';
  language: 'en' | 'vi';
  researchData: any;
}

export class ContentGenerator {
  private anthropic: Anthropic;
  private openai: OpenAI;
  
  constructor() {
    this.anthropic = new Anthropic({
      apiKey: process.env.ANTHROPIC_API_KEY
    });
    this.openai = new OpenAI({
      apiKey: process.env.OPENAI_API_KEY
    });
  }
  
  async generate(config: ContentConfig) {
    const systemPrompt = this.buildSystemPrompt(config);
    const userPrompt = this.buildUserPrompt(config);
    
    // Use Claude for Vietnamese and complex formats
    if (config.language === 'vi' || config.format === 'case-study') {
      return await this.generateWithClaude(systemPrompt, userPrompt);
    }
    
    // Use OpenAI for English and simpler formats
    return await this.generateWithOpenAI(systemPrompt, userPrompt);
  }
  
  private async generateWithClaude(system: string, user: string) {
    const message = await this.anthropic.messages.create({
      model: 'claude-3-5-sonnet-20241022',
      max_tokens: 4096,
      system: system,
      messages: [{
        role: 'user',
        content: user
      }]
    });
    
    return this.parseClaudeResponse(message);
  }
  
  private async generateWithOpenAI(system: string, user: string) {
    const completion = await this.openai.chat.completions.create({
      model: 'gpt-4-turbo-preview',
      messages: [
        { role: 'system', content: system },
        { role: 'user', content: user }
      ],
      temperature: 0.7,
      max_tokens: 3000
    });
    
    return this.parseOpenAIResponse(completion);
  }
  
  private buildSystemPrompt(config: ContentConfig): string {
    const formatInstructions = {
      'toplist': 'Create a comprehensive top list with detailed explanations for each item.',
      'pov': 'Write from a unique perspective with strong opinions backed by research.',
      'case-study': 'Develop an in-depth case study with problem, solution, and results.',
      'how-to': 'Create a step-by-step tutorial with actionable instructions.'
    };
    
    return `You are an expert content creator specializing in ${config.format} format.
Tone: ${config.tone}
Language: ${config.language}
Task: ${formatInstructions[config.format]}

Use the provided research data to create data-backed, engaging content.
Include statistics, quotes, and real examples.`;
  }
  
  private buildUserPrompt(config: ContentConfig): string {
    return `Topic: ${config.keyword}

Research Data:
${JSON.stringify(config.researchData, null, 2)}

Create compelling ${config.format} content about this topic using the research insights.`;
  }
}
```

### 3. Bilingual Content Generation

Create parallel English and Vietnamese versions:

```typescript
// lib/content/bilingual.ts
export async function generateBilingualContent(
  keyword: string,
  format: string,
  researchData: any
) {
  const generator = new ContentGenerator();
  
  // Generate both versions in parallel
  const [englishContent, vietnameseContent] = await Promise.all([
    generator.generate({
      keyword,
      format: format as any,
      tone: 'expert',
      language: 'en',
      researchData
    }),
    generator.generate({
      keyword,
      format: format as any,
      tone: 'expert',
      language: 'vi',
      researchData
    })
  ]);
  
  return {
    en: englishContent,
    vi: vietnameseContent,
    metadata: {
      keyword,
      format,
      createdAt: new Date(),
      sources: researchData.sources
    }
  };
}
```

### 4. Video Generation with Remotion

Transform content into video format:

```typescript
// remotion/compositions/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig } from 'remotion';
import { interpolate } from 'remotion';

interface ContentVideoProps {
  title: string;
  points: string[];
  style: 'reels' | 'tiktok' | 'shorts';
}

export const ContentVideo: React.FC<ContentVideoProps> = ({
  title,
  points,
  style
}) => {
  const frame = useCurrentFrame();
  const { fps, durationInFrames } = useVideoConfig();
  
  const titleOpacity = interpolate(
    frame,
    [0, 30],
    [0, 1],
    { extrapolateRight: 'clamp' }
  );
  
  const dimensions = {
    reels: { width: 1080, height: 1920 },
    tiktok: { width: 1080, height: 1920 },
    shorts: { width: 1080, height: 1920 }
  };
  
  return (
    <AbsoluteFill style={{ backgroundColor: '#000' }}>
      <div style={{ 
        opacity: titleOpacity,
        padding: '60px',
        color: 'white'
      }}>
        <h1 style={{ 
          fontSize: '72px',
          fontWeight: 'bold',
          marginBottom: '40px'
        }}>
          {title}
        </h1>
        
        {points.map((point, index) => {
          const pointFrame = 60 + (index * 90);
          const pointOpacity = interpolate(
            frame,
            [pointFrame, pointFrame + 20],
            [0, 1],
            { extrapolateRight: 'clamp' }
          );
          
          return (
            <div 
              key={index}
              style={{
                opacity: pointOpacity,
                fontSize: '48px',
                marginBottom: '30px',
                transform: `translateX(${interpolate(
                  frame,
                  [pointFrame, pointFrame + 20],
                  [-50, 0],
                  { extrapolateRight: 'clamp' }
                )}px)`
              }}
            >
              {index + 1}. {point}
            </div>
          );
        })}
      </div>
    </AbsoluteFill>
  );
};
```

```typescript
// lib/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

export async function renderContentVideo(
  content: any,
  outputPath: string,
  platform: 'reels' | 'tiktok' | 'shorts'
) {
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: (config) => config
  });
  
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title: content.title,
      points: content.keyPoints,
      style: platform
    }
  });
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: {
      title: content.title,
      points: content.keyPoints,
      style: platform
    }
  });
  
  return outputPath;
}
```

### 5. Complete Pipeline Orchestration

```typescript
// lib/pipeline/orchestrator.ts
export class ContentPipeline {
  async execute(keyword: string, options: PipelineOptions) {
    console.log(`Starting pipeline for: ${keyword}`);
    
    // Step 1: Research
    const research = await performResearch({
      keyword,
      sources: ['techcrunch', 'a16z', 'twitter'],
      timeRange: '24h',
      language: options.language
    });
    
    // Step 2: Generate Content
    const content = await generateBilingualContent(
      keyword,
      options.format,
      research
    );
    
    // Step 3: Render Videos (if requested)
    let videos = null;
    if (options.includeVideo) {
      videos = await Promise.all(
        ['reels', 'tiktok', 'shorts'].map(platform =>
          renderContentVideo(
            content.en,
            `./output/${keyword}-${platform}.mp4`,
            platform as any
          )
        )
      );
    }
    
    // Step 4: Save to database
    await saveToDatabase({
      keyword,
      content,
      videos,
      research: research.insights,
      metadata: {
        createdAt: new Date(),
        format: options.format,
        sources: research.sources
      }
    });
    
    return {
      content,
      videos,
      insights: research.insights
    };
  }
}

// Usage
const pipeline = new ContentPipeline();
const result = await pipeline.execute('AI Marketing Trends 2024', {
  format: 'toplist',
  language: 'en',
  includeVideo: true
});
```

## API Routes (Next.js)

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { ContentPipeline } from '@/lib/pipeline/orchestrator';

export async function POST(request: NextRequest) {
  try {
    const { keyword, format, includeVideo } = await request.json();
    
    const pipeline = new ContentPipeline();
    const result = await pipeline.execute(keyword, {
      format,
      language: 'en',
      includeVideo
    });
    
    return NextResponse.json(result);
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { error: 'Pipeline execution failed' },
      { status: 500 }
    );
  }
}
```

## Common Patterns

### Pattern 1: Scheduled Content Generation

```typescript
// lib/scheduler/cron.ts
import cron from 'node-cron';
import { ContentPipeline } from '@/lib/pipeline/orchestrator';

export function scheduleContentGeneration(topics: string[]) {
  // Run daily at 6 AM
  cron.schedule('0 6 * * *', async () => {
    for (const topic of topics) {
      try {
        const pipeline = new ContentPipeline();
        await pipeline.execute(topic, {
          format: 'toplist',
          language: 'en',
          includeVideo: true
        });
      } catch (error) {
        console.error(`Failed to generate content for ${topic}:`, error);
      }
    }
  });
}
```

### Pattern 2: Content Personalization

```typescript
// lib/personalization/customizer.ts
export async function personalizeContent(
  baseContent: any,
  audience: {
    industry: string;
    level: 'beginner' | 'intermediate' | 'expert';
    interests: string[];
  }
) {
  const customPrompt = `Adapt this content for:
Industry: ${audience.industry}
Level: ${audience.level}
Interests: ${audience.interests.join(', ')}

Original content:
${JSON.stringify(baseContent)}`;

  const anthropic = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY
  });
  
  const response = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 2048,
    messages: [{ role: 'user', content: customPrompt }]
  });
  
  return parseResponse(response);
}
```

## Troubleshooting

### Issue: API Rate Limits

```typescript
// lib/utils/rate-limiter.ts
import { Ratelimit } from '@upstash/ratelimit';
import { Redis } from '@upstash/redis';

const ratelimit = new Ratelimit({
  redis: Redis.fromEnv(),
  limiter: Ratelimit.slidingWindow(10, '1 m'),
});

export async function withRateLimit(key: string, fn: () => Promise<any>) {
  const { success, reset } = await ratelimit.limit(key);
  
  if (!success) {
    const waitTime = reset - Date.now();
    throw new Error(`Rate limit exceeded. Retry after ${waitTime}ms`);
  }
  
  return await fn();
}
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
    headless: true
  }
});
```

### Issue: Content Quality Inconsistency

```typescript
// Add validation layer
function validateContent(content: any): boolean {
  const checks = [
    content.title?.length > 10,
    content.body?.length > 500,
    content.keyPoints?.length >= 3,
    content.sources?.length > 0
  ];
  
  return checks.every(check => check === true);
}

// Regenerate if validation fails
let attempts = 0;
let content;

do {
  content = await generator.generate(config);
  attempts++;
} while (!validateContent(content) && attempts < 3);
```

## Best Practices

1. **Always validate research data** before feeding to AI models
2. **Cache research results** to avoid redundant API calls
3. **Use streaming responses** for real-time content generation feedback
4. **Implement retry logic** with exponential backoff for API calls
5. **Monitor token usage** to optimize costs
6. **Version control your prompts** for reproducibility
7. **Test video rendering locally** before deploying
