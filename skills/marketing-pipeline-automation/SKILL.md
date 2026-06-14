---
name: marketing-pipeline-automation
description: AI-powered content automation pipeline for research, script generation, and video creation using Claude/OpenAI and Remotion
triggers:
  - automate content creation from research to video
  - generate marketing content with AI pipeline
  - create videos from text automatically
  - scrape news and generate social media posts
  - build automated content workflow
  - set up AI content generation system
  - use Remotion for video rendering from content
  - integrate Claude API for content writing
---

# Marketing Pipeline Automation Skill

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

**Ultimate AI Content Pipeline** is a complete content automation system that transforms keywords into finished content pieces including articles, social media posts, and videos. The system:

- **Auto-crawls** recent news from TechCrunch, a16z, Twitter, LinkedIn
- **Generates content** in multiple formats (listicles, POV, case studies, how-tos) using Claude/OpenAI
- **Renders videos** automatically using Remotion
- **Supports multi-language** output (English/Vietnamese)
- **Provides flexible tone** customization (expert, friendly, humorous)

Built with Next.js and TypeScript, it integrates Claude 3, OpenAI, RapidAPI for scraping, and Remotion for video generation.

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

# Set up environment variables
cp .env.example .env.local
```

### Required Environment Variables

```bash
# AI Providers
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Data Sources
RAPIDAPI_KEY=your_rapidapi_key
TWITTER_BEARER_TOKEN=your_twitter_bearer_token

# Video Rendering (Remotion)
REMOTION_LICENSE_KEY=your_remotion_license_key

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# App Configuration
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

### Run Development Server

```bash
npm run dev
# Server runs on http://localhost:3000
```

## Core Architecture

The pipeline consists of three main stages:

1. **Research Stage**: Crawl and aggregate content from multiple sources
2. **Generation Stage**: Create content using AI based on scraped data
3. **Render Stage**: Convert content to video/images using Remotion

## Key Components

### 1. Research/Crawling Module

```typescript
// lib/research/crawler.ts
import axios from 'axios';

interface NewsSource {
  name: string;
  url: string;
  lastUpdated: Date;
}

export async function fetchRecentNews(keyword: string, hours: number = 24): Promise<NewsArticle[]> {
  const sources = [
    { name: 'TechCrunch', endpoint: process.env.TECHCRUNCH_API },
    { name: 'a16z', endpoint: process.env.A16Z_RSS_FEED }
  ];

  const articles: NewsArticle[] = [];
  
  for (const source of sources) {
    try {
      const response = await axios.get(source.endpoint, {
        params: {
          keyword,
          since: new Date(Date.now() - hours * 60 * 60 * 1000).toISOString()
        },
        headers: {
          'X-RapidAPI-Key': process.env.RAPIDAPI_KEY
        }
      });
      
      articles.push(...response.data.articles);
    } catch (error) {
      console.error(`Failed to fetch from ${source.name}:`, error);
    }
  }
  
  return articles;
}

// lib/research/analyzer.ts
export async function extractInsights(articles: NewsArticle[]): Promise<ContentInsights> {
  const insights = {
    keyTrends: [],
    statistics: [],
    quotes: [],
    sources: []
  };
  
  for (const article of articles) {
    // Extract key data points
    insights.statistics.push(...extractStats(article.content));
    insights.quotes.push(...extractQuotes(article.content));
    insights.sources.push({
      title: article.title,
      url: article.url,
      date: article.publishedAt
    });
  }
  
  return insights;
}
```

### 2. Content Generation with Claude/OpenAI

```typescript
// lib/generation/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

export interface ContentConfig {
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  targetAudience: string;
}

export async function generateContent(
  insights: ContentInsights,
  config: ContentConfig,
  provider: 'claude' | 'openai' = 'claude'
): Promise<GeneratedContent> {
  const prompt = buildPrompt(insights, config);
  
  if (provider === 'claude') {
    const message = await anthropic.messages.create({
      model: 'claude-3-5-sonnet-20241022',
      max_tokens: 4000,
      messages: [{
        role: 'user',
        content: prompt
      }],
      system: getSystemPrompt(config)
    });
    
    return parseClaudeResponse(message.content[0].text);
  } else {
    const completion = await openai.chat.completions.create({
      model: 'gpt-4-turbo-preview',
      messages: [
        { role: 'system', content: getSystemPrompt(config) },
        { role: 'user', content: prompt }
      ],
      temperature: 0.7,
    });
    
    return parseOpenAIResponse(completion.choices[0].message.content);
  }
}

function buildPrompt(insights: ContentInsights, config: ContentConfig): string {
  const { format, language, tone, targetAudience } = config;
  
  return `
Create a ${format} article in ${language} with a ${tone} tone for ${targetAudience}.

Based on the following research insights:
Trends: ${insights.keyTrends.join(', ')}
Statistics: ${JSON.stringify(insights.statistics)}
Key Quotes: ${insights.quotes.map(q => `"${q.text}" - ${q.source}`).join('\n')}

Requirements:
- Include data-backed claims with sources
- Format for social media engagement
- ${language === 'vi' ? 'Write in natural Vietnamese' : 'Write in clear English'}
- Optimize for ${format} structure
`;
}

function getSystemPrompt(config: ContentConfig): string {
  const toneInstructions = {
    expert: 'Use professional, authoritative language with industry jargon where appropriate.',
    friendly: 'Use conversational, approachable language that builds rapport.',
    humorous: 'Include witty remarks and light humor while maintaining credibility.'
  };
  
  return `You are an expert content creator specializing in ${config.format} articles.
${toneInstructions[config.tone]}
Always cite sources and use data to support claims.`;
}
```

### 3. Multi-language Content Generation

```typescript
// lib/generation/multilingual.ts
export async function generateBilingualContent(
  insights: ContentInsights,
  baseConfig: Omit<ContentConfig, 'language'>
): Promise<{ en: GeneratedContent; vi: GeneratedContent }> {
  const [englishContent, vietnameseContent] = await Promise.all([
    generateContent(insights, { ...baseConfig, language: 'en' }),
    generateContent(insights, { ...baseConfig, language: 'vi' })
  ]);
  
  return {
    en: englishContent,
    vi: vietnameseContent
  };
}
```

### 4. Video Rendering with Remotion

```typescript
// remotion/compositions.tsx
import { Composition } from 'remotion';
import { ContentVideo } from './ContentVideo';

export const RemotionRoot: React.FC = () => {
  return (
    <>
      <Composition
        id="ContentVideo"
        component={ContentVideo}
        durationInFrames={450} // 15 seconds at 30fps
        fps={30}
        width={1080}
        height={1920} // Vertical format for Reels/TikTok
        defaultProps={{
          title: '',
          points: [],
          style: 'modern'
        }}
      />
    </>
  );
};

// remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, interpolate, Sequence } from 'remotion';

interface ContentVideoProps {
  title: string;
  points: string[];
  style: 'modern' | 'minimal' | 'bold';
}

export const ContentVideo: React.FC<ContentVideoProps> = ({ title, points, style }) => {
  const frame = useCurrentFrame();
  
  const titleOpacity = interpolate(frame, [0, 30], [0, 1], {
    extrapolateRight: 'clamp',
  });
  
  return (
    <AbsoluteFill style={{ backgroundColor: '#000' }}>
      <div style={{ 
        padding: 60,
        opacity: titleOpacity 
      }}>
        <h1 style={{ 
          color: '#fff', 
          fontSize: 72,
          fontWeight: 'bold',
          marginBottom: 40
        }}>
          {title}
        </h1>
      </div>
      
      {points.map((point, index) => (
        <Sequence
          key={index}
          from={60 + index * 90}
          durationInFrames={90}
        >
          <PointSlide text={point} index={index} />
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};

const PointSlide: React.FC<{ text: string; index: number }> = ({ text, index }) => {
  const frame = useCurrentFrame();
  const opacity = interpolate(frame, [0, 15], [0, 1]);
  const translateY = interpolate(frame, [0, 15], [50, 0]);
  
  return (
    <AbsoluteFill style={{ 
      justifyContent: 'center',
      padding: 60 
    }}>
      <div style={{
        opacity,
        transform: `translateY(${translateY}px)`,
        color: '#fff',
        fontSize: 48,
        lineHeight: 1.5
      }}>
        <span style={{ color: '#00D9FF', fontWeight: 'bold' }}>
          {index + 1}.
        </span> {text}
      </div>
    </AbsoluteFill>
  );
};
```

### 5. Video Render API

```typescript
// lib/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

export async function renderContentVideo(
  content: GeneratedContent,
  outputPath: string
): Promise<string> {
  const compositionId = 'ContentVideo';
  const bundleLocation = await bundle(
    path.join(process.cwd(), 'remotion/index.ts')
  );
  
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: {
      title: content.title,
      points: content.keyPoints,
      style: 'modern'
    },
  });
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: {
      title: content.title,
      points: content.keyPoints,
      style: 'modern'
    },
  });
  
  return outputPath;
}

// Render multiple aspect ratios
export async function renderMultipleFormats(
  content: GeneratedContent
): Promise<{ reels: string; tiktok: string; youtube: string }> {
  const formats = {
    reels: { width: 1080, height: 1920 },
    tiktok: { width: 1080, height: 1920 },
    youtube: { width: 1920, height: 1080 }
  };
  
  const results = {};
  
  for (const [platform, dimensions] of Object.entries(formats)) {
    const outputPath = path.join(
      process.cwd(),
      'output',
      `${content.id}_${platform}.mp4`
    );
    
    results[platform] = await renderContentVideo(content, outputPath);
  }
  
  return results as any;
}
```

## Complete Pipeline Implementation

```typescript
// lib/pipeline/full-pipeline.ts
import { fetchRecentNews, extractInsights } from '../research';
import { generateBilingualContent } from '../generation';
import { renderMultipleFormats } from '../video';

export interface PipelineConfig {
  keyword: string;
  contentFormat: 'toplist' | 'pov' | 'case-study' | 'how-to';
  tone: 'expert' | 'friendly' | 'humorous';
  targetAudience: string;
  generateVideo: boolean;
}

export async function runFullPipeline(config: PipelineConfig) {
  console.log('🔍 Step 1: Research - Fetching recent news...');
  const articles = await fetchRecentNews(config.keyword, 24);
  
  console.log(`📊 Found ${articles.length} articles. Extracting insights...`);
  const insights = await extractInsights(articles);
  
  console.log('✍️ Step 2: Generating bilingual content...');
  const content = await generateBilingualContent(insights, {
    format: config.contentFormat,
    tone: config.tone,
    targetAudience: config.targetAudience
  });
  
  let videos = null;
  if (config.generateVideo) {
    console.log('🎬 Step 3: Rendering videos...');
    videos = {
      en: await renderMultipleFormats(content.en),
      vi: await renderMultipleFormats(content.vi)
    };
  }
  
  console.log('✅ Pipeline complete!');
  
  return {
    insights,
    content,
    videos
  };
}
```

## API Routes (Next.js)

```typescript
// app/api/pipeline/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runFullPipeline } from '@/lib/pipeline/full-pipeline';

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
    
    const result = await runFullPipeline(config);
    
    return NextResponse.json({
      success: true,
      data: result
    });
    
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { error: 'Pipeline execution failed', details: error.message },
      { status: 500 }
    );
  }
}

// app/api/content/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { generateContent } from '@/lib/generation/content-generator';

export async function POST(request: NextRequest) {
  const { insights, config, provider } = await request.json();
  
  const content = await generateContent(insights, config, provider);
  
  return NextResponse.json({ content });
}

// app/api/video/render/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { renderContentVideo } from '@/lib/video/renderer';

export async function POST(request: NextRequest) {
  const { content, outputPath } = await request.json();
  
  const videoPath = await renderContentVideo(content, outputPath);
  
  return NextResponse.json({ videoPath });
}
```

## CLI Commands (if applicable)

```bash
# Run the full pipeline
npm run pipeline -- --keyword "AI automation" --format toplist --tone expert

# Generate content only (no video)
npm run generate -- --keyword "AI trends" --no-video

# Render video from existing content
npm run render:video -- --content-id abc123 --formats reels,tiktok,youtube

# Development
npm run dev          # Start Next.js dev server
npm run build        # Build for production
npm run start        # Start production server

# Remotion commands
npm run remotion:preview   # Preview Remotion compositions
npm run remotion:render    # Render a specific composition
```

## Common Patterns

### Pattern 1: Scheduled Content Generation

```typescript
// lib/scheduler/content-scheduler.ts
import cron from 'node-cron';
import { runFullPipeline } from '../pipeline/full-pipeline';

export function scheduleContentGeneration(
  schedule: string, // cron format
  configs: PipelineConfig[]
) {
  cron.schedule(schedule, async () => {
    console.log('⏰ Scheduled content generation started');
    
    for (const config of configs) {
      try {
        await runFullPipeline(config);
        console.log(`✅ Generated content for: ${config.keyword}`);
      } catch (error) {
        console.error(`❌ Failed for ${config.keyword}:`, error);
      }
    }
  });
}

// Usage: Run daily at 9 AM
scheduleContentGeneration('0 9 * * *', [
  {
    keyword: 'AI trends',
    contentFormat: 'toplist',
    tone: 'expert',
    targetAudience: 'marketers',
    generateVideo: true
  }
]);
```

### Pattern 2: Content Batch Processing

```typescript
// lib/batch/batch-processor.ts
export async function processBatch(keywords: string[]) {
  const results = await Promise.allSettled(
    keywords.map(keyword => 
      runFullPipeline({
        keyword,
        contentFormat: 'pov',
        tone: 'friendly',
        targetAudience: 'general audience',
        generateVideo: false
      })
    )
  );
  
  const successful = results.filter(r => r.status === 'fulfilled');
  const failed = results.filter(r => r.status === 'rejected');
  
  return {
    totalProcessed: keywords.length,
    successful: successful.length,
    failed: failed.length,
    results: successful.map(r => r.value)
  };
}
```

### Pattern 3: Custom Content Templates

```typescript
// lib/templates/custom-template.ts
export interface ContentTemplate {
  name: string;
  structure: string[];
  requiredElements: string[];
  style: object;
}

const templates: Record<string, ContentTemplate> = {
  viral_toplist: {
    name: 'Viral Top List',
    structure: [
      'hook',
      'introduction',
      'items (5-7)',
      'data_visualization',
      'conclusion',
      'cta'
    ],
    requiredElements: ['statistics', 'expert_quotes', 'sources'],
    style: {
      tone: 'engaging',
      emoji: true,
      formatting: 'bold_headers'
    }
  },
  thought_leadership: {
    name: 'Thought Leadership POV',
    structure: [
      'controversial_statement',
      'context',
      'argument',
      'counterargument',
      'resolution',
      'takeaway'
    ],
    requiredElements: ['original_insight', 'industry_data'],
    style: {
      tone: 'authoritative',
      emoji: false,
      formatting: 'professional'
    }
  }
};

export function applyTemplate(
  content: GeneratedContent,
  templateName: string
): GeneratedContent {
  const template = templates[templateName];
  // Apply template structure and styling
  return enhanceWithTemplate(content, template);
}
```

## Troubleshooting

### Issue: API Rate Limits

```typescript
// lib/utils/rate-limiter.ts
import pLimit from 'p-limit';

const limit = pLimit(3); // Max 3 concurrent API calls

export async function fetchWithRateLimit<T>(
  fn: () => Promise<T>
): Promise<T> {
  return limit(fn);
}

// Usage
const articles = await Promise.all(
  sources.map(source => 
    fetchWithRateLimit(() => fetchFromSource(source))
  )
);
```

### Issue: Video Rendering Timeout

```typescript
// Increase timeout for long videos
await renderMedia({
  composition,
  serveUrl: bundleLocation,
  codec: 'h264',
  outputLocation: outputPath,
  timeoutInMilliseconds: 300000, // 5 minutes
  chromiumOptions: {
    headless: true,
    args: ['--no-sandbox', '--disable-setuid-sandbox']
  }
});
```

### Issue: Memory Issues with Large Datasets

```typescript
// Process in chunks
async function processLargeDataset(items: any[], chunkSize: number = 10) {
  const results = [];
  
  for (let i = 0; i < items.length; i += chunkSize) {
    const chunk = items.slice(i, i + chunkSize);
    const chunkResults = await processChunk(chunk);
    results.push(...chunkResults);
    
    // Clear memory between chunks
    if (global.gc) global.gc();
  }
  
  return results;
}
```

### Issue: Claude/OpenAI API Errors

```typescript
// lib/utils/ai-retry.ts
export async function callWithRetry<T>(
  fn: () => Promise<T>,
  maxRetries: number = 3,
  delayMs: number = 1000
): Promise<T> {
  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      return await fn();
    } catch (error) {
      if (attempt === maxRetries) throw error;
      
      console.warn(`Attempt ${attempt} failed, retrying...`);
      await new Promise(resolve => setTimeout(resolve, delayMs * attempt));
    }
  }
  
  throw new Error('Max retries exceeded');
}

// Usage
const content = await callWithRetry(() => 
  generateContent(insights, config)
);
```

## Performance Optimization

```typescript
// Cache research results
import NodeCache from 'node-cache';

const cache = new NodeCache({ stdTTL: 3600 }); // 1 hour

export async function fetchWithCache(keyword: string) {
  const cacheKey = `news_${keyword}`;
  const cached = cache.get(cacheKey);
  
  if (cached) {
    console.log('📦 Using cached results');
    return cached;
  }
  
  const articles = await fetchRecentNews(keyword);
  cache.set(cacheKey, articles);
  
  return articles;
}
```

This skill provides comprehensive coverage of the marketing pipeline automation system, enabling AI agents to effectively assist developers in implementing automated content generation workflows.
