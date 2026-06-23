---
name: ultimate-ai-content-pipeline
description: Automated content pipeline for research, scriptwriting, posting, and video generation using Claude/OpenAI and Remotion
triggers:
  - how do I automate content creation with AI research
  - set up automated content pipeline with video generation
  - use Claude and OpenAI for content automation
  - generate videos from text with Remotion integration
  - crawl news and create content automatically
  - build AI content pipeline with auto-posting
  - integrate Remotion for automated video rendering
  - scrape trending news and generate articles
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

Ultimate AI Content Pipeline is a complete content automation system that handles the entire workflow from research to publication. It automatically crawls news from sources like TechCrunch, a16z, Twitter/X, and LinkedIn, generates content in multiple formats using Claude 3 and OpenAI, and renders videos using Remotion. The system supports bilingual content (English/Vietnamese) and multiple content formats including toplists, POV articles, case studies, and how-to guides.

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
# AI API Keys
OPENAI_API_KEY=your_openai_key_here
ANTHROPIC_API_KEY=your_claude_key_here

# RapidAPI for News Crawling
RAPIDAPI_KEY=your_rapidapi_key_here

# Remotion Configuration (if applicable)
REMOTION_LICENSE_KEY=your_remotion_license_here

# Next.js Configuration
NEXT_PUBLIC_API_URL=http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/             # Core utilities
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── crawler/     # News crawling logic
│   │   └── video/       # Remotion video generation
│   ├── types/           # TypeScript type definitions
│   └── utils/           # Helper functions
├── public/              # Static assets
└── remotion/            # Remotion video templates
```

## Running the Application

```bash
# Development mode
npm run dev

# Production build
npm run build
npm start

# Render videos with Remotion
npm run remotion:render
```

## Core Features & Usage

### 1. Auto-Scan Research (News Crawling)

```typescript
import { crawlNews } from '@/lib/crawler/news-crawler';

interface NewsSource {
  source: 'techcrunch' | 'a16z' | 'twitter' | 'linkedin';
  timeframe: '24h' | '7d' | '30d';
  keywords?: string[];
}

async function fetchLatestNews(config: NewsSource) {
  const articles = await crawlNews({
    source: config.source,
    timeframe: config.timeframe,
    keywords: config.keywords,
  });
  
  return articles.map(article => ({
    title: article.title,
    content: article.content,
    url: article.url,
    publishedAt: article.publishedAt,
    insights: article.extractedInsights,
  }));
}

// Usage example
const news = await fetchLatestNews({
  source: 'techcrunch',
  timeframe: '24h',
  keywords: ['AI', 'machine learning', 'automation'],
});
```

### 2. AI Content Generation

```typescript
import { generateContent } from '@/lib/ai/content-generator';
import { Anthropic } from '@anthropic-ai/sdk';
import OpenAI from 'openai';

interface ContentConfig {
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi' | 'both';
  tone: 'professional' | 'friendly' | 'humorous';
  provider: 'claude' | 'openai';
}

async function createArticle(
  topic: string,
  researchData: any[],
  config: ContentConfig
) {
  const client = config.provider === 'claude' 
    ? new Anthropic({ apiKey: process.env.ANTHROPIC_API_KEY })
    : new OpenAI({ apiKey: process.env.OPENAI_API_KEY });

  const prompt = buildPrompt(topic, researchData, config);
  
  if (config.provider === 'claude') {
    const response = await client.messages.create({
      model: 'claude-3-opus-20240229',
      max_tokens: 4096,
      messages: [{
        role: 'user',
        content: prompt,
      }],
    });
    
    return response.content[0].text;
  } else {
    const response = await client.chat.completions.create({
      model: 'gpt-4-turbo-preview',
      messages: [{
        role: 'user',
        content: prompt,
      }],
      max_tokens: 4096,
    });
    
    return response.choices[0].message.content;
  }
}

function buildPrompt(
  topic: string,
  data: any[],
  config: ContentConfig
): string {
  const formatInstructions = {
    'toplist': 'Create a numbered list article with rankings and explanations',
    'pov': 'Write from a personal perspective with strong opinions',
    'case-study': 'Analyze specific examples with data and outcomes',
    'how-to': 'Provide step-by-step instructions with actionable tips',
  };

  return `
    Topic: ${topic}
    Format: ${config.format}
    Tone: ${config.tone}
    Language: ${config.language}
    
    ${formatInstructions[config.format]}
    
    Research Data:
    ${JSON.stringify(data, null, 2)}
    
    Generate comprehensive content that is data-backed and engaging.
  `;
}
```

### 3. Bilingual Content Generation

```typescript
import { translateContent } from '@/lib/ai/translator';

async function generateBilingualContent(
  englishContent: string,
  targetLanguage: 'vi' = 'vi'
) {
  const client = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY,
  });

  const response = await client.messages.create({
    model: 'claude-3-sonnet-20240229',
    max_tokens: 4096,
    messages: [{
      role: 'user',
      content: `Translate the following content to ${targetLanguage}, 
                maintaining the tone and style. Adapt idioms and 
                cultural references appropriately:
                
                ${englishContent}`,
    }],
  });

  return {
    en: englishContent,
    vi: response.content[0].text,
  };
}
```

### 4. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  template: 'infographic' | 'short-form' | 'reel';
  platform: 'tiktok' | 'youtube-shorts' | 'instagram-reels';
  content: {
    title: string;
    keyPoints: string[];
    visualData?: any;
  };
}

async function generateVideo(config: VideoConfig) {
  const bundleLocation = await bundle(
    path.join(process.cwd(), 'remotion/index.ts')
  );

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: config.template,
    inputProps: {
      title: config.content.title,
      keyPoints: config.content.keyPoints,
      visualData: config.content.visualData,
    },
  });

  const outputLocation = path.join(
    process.cwd(),
    'output',
    `${Date.now()}-${config.platform}.mp4`
  );

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: composition.defaultProps,
  });

  return outputLocation;
}

// Usage
const videoPath = await generateVideo({
  template: 'short-form',
  platform: 'tiktok',
  content: {
    title: '5 AI Trends in 2024',
    keyPoints: [
      'Multimodal AI goes mainstream',
      'AI agents automate workflows',
      'Open source LLMs catch up',
    ],
  },
});
```

### 5. End-to-End Pipeline

```typescript
import { runContentPipeline } from '@/lib/pipeline';

async function automatedContentCreation(keyword: string) {
  try {
    // Step 1: Research
    const research = await crawlNews({
      source: 'techcrunch',
      timeframe: '24h',
      keywords: [keyword],
    });

    // Step 2: Generate content
    const article = await createArticle(keyword, research, {
      format: 'toplist',
      language: 'both',
      tone: 'professional',
      provider: 'claude',
    });

    // Step 3: Create bilingual versions
    const bilingualContent = await generateBilingualContent(
      article.en || article,
      'vi'
    );

    // Step 4: Generate video
    const videoPath = await generateVideo({
      template: 'infographic',
      platform: 'youtube-shorts',
      content: {
        title: `Top Insights: ${keyword}`,
        keyPoints: extractKeyPoints(bilingualContent.en),
      },
    });

    // Step 5: Return complete package
    return {
      article: bilingualContent,
      video: videoPath,
      metadata: {
        keyword,
        createdAt: new Date().toISOString(),
        sources: research.map(r => r.url),
      },
    };
  } catch (error) {
    console.error('Pipeline failed:', error);
    throw error;
  }
}

function extractKeyPoints(content: string): string[] {
  // Simple extraction logic - enhance as needed
  const lines = content.split('\n').filter(line => 
    line.match(/^[\d•-]\.?\s+/) // Match numbered or bulleted lists
  );
  return lines.slice(0, 5).map(line => 
    line.replace(/^[\d•-]\.?\s+/, '').trim()
  );
}
```

## API Routes (Next.js)

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { automatedContentCreation } from '@/lib/pipeline';

export async function POST(request: NextRequest) {
  try {
    const { keyword, format, language } = await request.json();

    if (!keyword) {
      return NextResponse.json(
        { error: 'Keyword is required' },
        { status: 400 }
      );
    }

    const result = await automatedContentCreation(keyword);

    return NextResponse.json({
      success: true,
      data: result,
    });
  } catch (error) {
    return NextResponse.json(
      { error: error.message },
      { status: 500 }
    );
  }
}
```

## Common Patterns

### Content Scheduling

```typescript
import { schedulePost } from '@/lib/scheduler';

interface ScheduleConfig {
  platform: 'facebook' | 'linkedin' | 'twitter';
  content: string;
  scheduledTime: Date;
  media?: string[];
}

async function scheduleContent(config: ScheduleConfig) {
  return await schedulePost({
    platform: config.platform,
    content: config.content,
    scheduledTime: config.scheduledTime,
    media: config.media,
  });
}
```

### Batch Processing

```typescript
async function batchGenerateContent(keywords: string[]) {
  const results = await Promise.allSettled(
    keywords.map(keyword => 
      automatedContentCreation(keyword)
    )
  );

  return results.map((result, index) => ({
    keyword: keywords[index],
    status: result.status,
    data: result.status === 'fulfilled' ? result.value : null,
    error: result.status === 'rejected' ? result.reason : null,
  }));
}
```

## Troubleshooting

### API Rate Limits

```typescript
import { RateLimiter } from '@/lib/utils/rate-limiter';

const limiter = new RateLimiter({
  maxRequests: 50,
  windowMs: 60000, // 1 minute
});

async function rateLimitedAPICall(fn: () => Promise<any>) {
  await limiter.waitForToken();
  return await fn();
}
```

### Error Handling

```typescript
class PipelineError extends Error {
  constructor(
    message: string,
    public stage: string,
    public originalError?: Error
  ) {
    super(message);
    this.name = 'PipelineError';
  }
}

async function safeContentGeneration(keyword: string) {
  try {
    return await automatedContentCreation(keyword);
  } catch (error) {
    if (error.message.includes('rate limit')) {
      console.log('Rate limited, waiting 60s...');
      await new Promise(resolve => setTimeout(resolve, 60000));
      return await automatedContentCreation(keyword);
    }
    throw new PipelineError(
      'Content generation failed',
      'generation',
      error
    );
  }
}
```

### Video Rendering Issues

```typescript
// Ensure Remotion dependencies are installed
// Check memory availability for rendering
async function renderWithRetry(config: VideoConfig, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await generateVideo(config);
    } catch (error) {
      console.log(`Render attempt ${i + 1} failed:`, error);
      if (i === maxRetries - 1) throw error;
      await new Promise(resolve => setTimeout(resolve, 5000));
    }
  }
}
```

## Best Practices

1. **Always use environment variables** for API keys
2. **Implement rate limiting** for external API calls
3. **Cache research results** to avoid redundant crawling
4. **Use TypeScript types** for better code safety
5. **Handle bilingual content** with proper encoding (UTF-8)
6. **Monitor API costs** especially for Claude/OpenAI usage
7. **Test video rendering** locally before production deployment
8. **Implement retry logic** for network-dependent operations
