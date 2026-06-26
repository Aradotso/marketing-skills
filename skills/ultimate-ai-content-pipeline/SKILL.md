---
name: ultimate-ai-content-pipeline
description: Automated content creation pipeline from research to video generation using Claude, OpenAI, and Remotion
triggers:
  - how do I automate content creation with AI
  - generate video content from text automatically
  - set up content pipeline with Claude and OpenAI
  - crawl news and create content automatically
  - how to use Ultimate AI Content Pipeline
  - create multilingual content with AI
  - automate social media video generation
  - build content research to video workflow
---

# Ultimate AI Content Pipeline Skill

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

Ultimate AI Content Pipeline is a comprehensive TypeScript-based content automation system that handles the entire workflow from research to video generation. It automatically crawls news sources (TechCrunch, a16z, Twitter, LinkedIn), generates multi-format content in multiple languages using Claude 3/OpenAI, and renders videos using Remotion.

**Key capabilities:**
- Auto-scan research from major news sources (24h data)
- Multi-format content generation (Toplist, POV, Case Study, How-to)
- Bilingual output (English/Vietnamese) with customizable tone
- Automatic video/infographic rendering via Remotion
- Next.js frontend for easy content scheduling

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

Create a `.env.local` file in the root directory:

```env
# AI API Keys
OPENAI_API_KEY=your_openai_key_here
ANTHROPIC_API_KEY=your_claude_key_here

# RapidAPI for news crawling
RAPIDAPI_KEY=your_rapidapi_key_here

# Remotion Configuration
REMOTION_LICENSE_KEY=your_remotion_license_here

# Optional: Database
DATABASE_URL=postgresql://user:password@localhost:5432/content_pipeline

# Next.js Configuration
NEXT_PUBLIC_API_URL=http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── crawler/     # News crawling logic
│   │   ├── content/     # Content generation
│   │   └── video/       # Remotion video rendering
│   └── utils/           # Helper functions
├── public/              # Static assets
├── remotion/            # Remotion video templates
└── package.json
```

## Core Components

### 1. News Crawler Module

```typescript
// src/lib/crawler/newsCrawler.ts
import axios from 'axios';

interface NewsSource {
  url: string;
  selector: string;
  type: 'techcrunch' | 'a16z' | 'twitter' | 'linkedin';
}

export async function crawlNews(keyword: string, timeframe: '24h' | '7d' = '24h') {
  const rapidApiHeaders = {
    'X-RapidAPI-Key': process.env.RAPIDAPI_KEY,
    'X-RapidAPI-Host': 'news-api14.p.rapidapi.com'
  };

  const response = await axios.get('https://news-api14.p.rapidapi.com/search', {
    params: {
      q: keyword,
      language: 'en',
      sortBy: 'publishedAt',
      timeframe: timeframe
    },
    headers: rapidApiHeaders
  });

  return response.data.articles;
}

export async function extractInsights(articles: any[]) {
  // Extract key insights, trends, and data points
  const insights = articles.map(article => ({
    title: article.title,
    summary: article.description,
    url: article.url,
    publishedAt: article.publishedAt,
    source: article.source.name
  }));

  return insights;
}
```

### 2. AI Content Generation

```typescript
// src/lib/ai/contentGenerator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY
});

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY
});

export type ContentFormat = 'toplist' | 'pov' | 'case-study' | 'how-to';
export type ContentTone = 'expert' | 'friendly' | 'humorous';
export type Language = 'en' | 'vi';

interface GenerateContentParams {
  keyword: string;
  insights: any[];
  format: ContentFormat;
  tone: ContentTone;
  language: Language;
}

export async function generateContentWithClaude(params: GenerateContentParams) {
  const { keyword, insights, format, tone, language } = params;

  const prompt = buildPrompt(keyword, insights, format, tone, language);

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

  return message.content[0].type === 'text' ? message.content[0].text : '';
}

export async function generateContentWithOpenAI(params: GenerateContentParams) {
  const { keyword, insights, format, tone, language } = params;

  const prompt = buildPrompt(keyword, insights, format, tone, language);

  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: 'You are an expert content writer specializing in marketing and technology.'
      },
      {
        role: 'user',
        content: prompt
      }
    ],
    temperature: 0.7,
    max_tokens: 4096
  });

  return completion.choices[0].message.content || '';
}

function buildPrompt(
  keyword: string,
  insights: any[],
  format: ContentFormat,
  tone: ContentTone,
  language: Language
): string {
  const insightsText = insights.map((ins, idx) => 
    `${idx + 1}. ${ins.title} - ${ins.summary} (Source: ${ins.source})`
  ).join('\n');

  const formatInstructions = {
    'toplist': 'Create a top 10 list format with clear rankings and explanations',
    'pov': 'Write from a unique perspective with personal insights and opinions',
    'case-study': 'Structure as a detailed case study with problem, solution, and results',
    'how-to': 'Create a step-by-step guide with actionable instructions'
  };

  const toneInstructions = {
    'expert': 'Use professional, authoritative language with industry terminology',
    'friendly': 'Use conversational, approachable language',
    'humorous': 'Include humor and wit while maintaining informativeness'
  };

  return `
Topic: ${keyword}

Recent Research Insights:
${insightsText}

Task: Generate a comprehensive article about "${keyword}" using the insights above.

Format: ${formatInstructions[format]}
Tone: ${toneInstructions[tone]}
Language: ${language === 'en' ? 'English' : 'Vietnamese'}

Requirements:
- Include data and statistics from the research
- Add credible sources and citations
- Make it engaging and SEO-friendly
- Length: 1500-2000 words
- Include a compelling headline and introduction
${language === 'vi' ? '- Write entirely in Vietnamese with natural phrasing' : ''}
  `.trim();
}
```

### 3. Bilingual Content Generation

```typescript
// src/lib/content/bilingualGenerator.ts
import { generateContentWithClaude, ContentFormat, ContentTone } from '../ai/contentGenerator';

export async function generateBilingualContent(
  keyword: string,
  insights: any[],
  format: ContentFormat,
  tone: ContentTone
) {
  // Generate both versions in parallel
  const [englishContent, vietnameseContent] = await Promise.all([
    generateContentWithClaude({
      keyword,
      insights,
      format,
      tone,
      language: 'en'
    }),
    generateContentWithClaude({
      keyword,
      insights,
      format,
      tone,
      language: 'vi'
    })
  ]);

  return {
    en: {
      content: englishContent,
      metadata: {
        language: 'en',
        format,
        tone,
        wordCount: englishContent.split(' ').length
      }
    },
    vi: {
      content: vietnameseContent,
      metadata: {
        language: 'vi',
        format,
        tone,
        wordCount: vietnameseContent.split(' ').length
      }
    }
  };
}
```

### 4. Remotion Video Generation

```typescript
// src/lib/video/videoGenerator.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  title: string;
  subtitle: string;
  keyPoints: string[];
  duration: number; // in seconds
  aspectRatio: '16:9' | '9:16' | '1:1';
}

export async function generateVideo(config: VideoConfig) {
  const compositionId = getCompositionForRatio(config.aspectRatio);
  
  // Bundle the Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: (config) => config
  });

  // Select composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: {
      title: config.title,
      subtitle: config.subtitle,
      keyPoints: config.keyPoints
    }
  });

  // Render video
  const outputLocation = path.resolve(
    `./public/videos/${Date.now()}-${compositionId}.mp4`
  );

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: {
      title: config.title,
      subtitle: config.subtitle,
      keyPoints: config.keyPoints
    }
  });

  return outputLocation;
}

function getCompositionForRatio(ratio: string): string {
  const compositions = {
    '16:9': 'YouTubeVideo',
    '9:16': 'ReelsVideo',
    '1:1': 'InstagramVideo'
  };
  return compositions[ratio as keyof typeof compositions];
}

export async function extractKeyPointsFromContent(content: string): Promise<string[]> {
  // Extract 5-7 key points from content for video
  const paragraphs = content.split('\n\n').filter(p => p.trim());
  const keyPoints = paragraphs
    .slice(0, 7)
    .map(p => p.split('.')[0])
    .filter(p => p.length > 20 && p.length < 200);
  
  return keyPoints.slice(0, 5);
}
```

### 5. Complete Pipeline Orchestration

```typescript
// src/lib/pipeline/orchestrator.ts
import { crawlNews, extractInsights } from '../crawler/newsCrawler';
import { generateBilingualContent } from '../content/bilingualGenerator';
import { generateVideo, extractKeyPointsFromContent } from '../video/videoGenerator';
import { ContentFormat, ContentTone } from '../ai/contentGenerator';

interface PipelineConfig {
  keyword: string;
  format: ContentFormat;
  tone: ContentTone;
  generateVideo: boolean;
  videoRatio?: '16:9' | '9:16' | '1:1';
}

export async function runContentPipeline(config: PipelineConfig) {
  console.log('🔍 Step 1: Crawling news sources...');
  const articles = await crawlNews(config.keyword, '24h');
  
  console.log('🧠 Step 2: Extracting insights...');
  const insights = await extractInsights(articles);
  
  console.log('✍️ Step 3: Generating bilingual content...');
  const content = await generateBilingualContent(
    config.keyword,
    insights,
    config.format,
    config.tone
  );
  
  let videoPath = null;
  if (config.generateVideo) {
    console.log('🎬 Step 4: Generating video...');
    const keyPoints = await extractKeyPointsFromContent(content.en.content);
    
    videoPath = await generateVideo({
      title: config.keyword,
      subtitle: `${config.format} - ${config.tone} tone`,
      keyPoints,
      duration: 30,
      aspectRatio: config.videoRatio || '16:9'
    });
  }
  
  console.log('✅ Pipeline complete!');
  
  return {
    content,
    insights,
    video: videoPath,
    timestamp: new Date().toISOString()
  };
}
```

## API Routes (Next.js)

```typescript
// src/app/api/pipeline/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline/orchestrator';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const { keyword, format, tone, generateVideo, videoRatio } = body;

    // Validate input
    if (!keyword || !format || !tone) {
      return NextResponse.json(
        { error: 'Missing required fields' },
        { status: 400 }
      );
    }

    // Run pipeline
    const result = await runContentPipeline({
      keyword,
      format,
      tone,
      generateVideo: generateVideo || false,
      videoRatio: videoRatio || '16:9'
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

## Usage Examples

### Basic Content Generation

```typescript
import { runContentPipeline } from './lib/pipeline/orchestrator';

// Generate a toplist article about AI trends
const result = await runContentPipeline({
  keyword: 'AI trends 2024',
  format: 'toplist',
  tone: 'expert',
  generateVideo: false
});

console.log('English content:', result.content.en.content);
console.log('Vietnamese content:', result.content.vi.content);
```

### Full Pipeline with Video

```typescript
// Generate content + video for social media
const result = await runContentPipeline({
  keyword: 'Marketing automation',
  format: 'how-to',
  tone: 'friendly',
  generateVideo: true,
  videoRatio: '9:16' // For Reels/TikTok
});

console.log('Video saved to:', result.video);
```

### Frontend Integration

```typescript
// src/components/ContentGenerator.tsx
'use client';

import { useState } from 'react';

export default function ContentGenerator() {
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState(null);

  const handleGenerate = async (e: React.FormEvent) => {
    e.preventDefault();
    setLoading(true);

    const formData = new FormData(e.target as HTMLFormElement);
    
    const response = await fetch('/api/pipeline', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        keyword: formData.get('keyword'),
        format: formData.get('format'),
        tone: formData.get('tone'),
        generateVideo: formData.get('generateVideo') === 'on',
        videoRatio: formData.get('videoRatio')
      })
    });

    const data = await response.json();
    setResult(data);
    setLoading(false);
  };

  return (
    <form onSubmit={handleGenerate}>
      <input name="keyword" placeholder="Enter keyword" required />
      <select name="format" required>
        <option value="toplist">Top List</option>
        <option value="pov">Point of View</option>
        <option value="case-study">Case Study</option>
        <option value="how-to">How To</option>
      </select>
      <select name="tone" required>
        <option value="expert">Expert</option>
        <option value="friendly">Friendly</option>
        <option value="humorous">Humorous</option>
      </select>
      <label>
        <input type="checkbox" name="generateVideo" />
        Generate Video
      </label>
      <select name="videoRatio">
        <option value="16:9">16:9 (YouTube)</option>
        <option value="9:16">9:16 (Reels/TikTok)</option>
        <option value="1:1">1:1 (Instagram)</option>
      </select>
      <button type="submit" disabled={loading}>
        {loading ? 'Generating...' : 'Generate Content'}
      </button>
      
      {result && (
        <div>
          <h3>Results:</h3>
          <div>{JSON.stringify(result, null, 2)}</div>
        </div>
      )}
    </form>
  );
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

# Run tests
npm test

# Render video (standalone)
npm run remotion:render
```

## Common Patterns

### Scheduled Content Generation

```typescript
// Use node-cron for scheduled content
import cron from 'node-cron';
import { runContentPipeline } from './lib/pipeline/orchestrator';

// Run every day at 9 AM
cron.schedule('0 9 * * *', async () => {
  const topics = ['AI news', 'Marketing trends', 'Tech updates'];
  
  for (const topic of topics) {
    await runContentPipeline({
      keyword: topic,
      format: 'toplist',
      tone: 'expert',
      generateVideo: true,
      videoRatio: '16:9'
    });
  }
});
```

### Batch Processing

```typescript
async function batchGenerateContent(keywords: string[]) {
  const results = await Promise.all(
    keywords.map(keyword =>
      runContentPipeline({
        keyword,
        format: 'pov',
        tone: 'friendly',
        generateVideo: false
      })
    )
  );
  
  return results;
}
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement retry logic with exponential backoff
async function withRetry<T>(
  fn: () => Promise<T>,
  maxRetries = 3
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error: any) {
      if (error.status === 429 && i < maxRetries - 1) {
        const delay = Math.pow(2, i) * 1000;
        await new Promise(resolve => setTimeout(resolve, delay));
        continue;
      }
      throw error;
    }
  }
  throw new Error('Max retries exceeded');
}

// Usage
const content = await withRetry(() =>
  generateContentWithClaude(params)
);
```

### Memory Issues with Video Rendering

```typescript
// Process videos sequentially instead of parallel
async function renderVideosSequentially(configs: VideoConfig[]) {
  const results = [];
  
  for (const config of configs) {
    const video = await generateVideo(config);
    results.push(video);
    
    // Clean up memory
    if (global.gc) {
      global.gc();
    }
  }
  
  return results;
}
```

### Missing API Keys

```typescript
// Validate environment variables on startup
function validateEnv() {
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

validateEnv();
```

## Performance Optimization

```typescript
// Cache research results to avoid redundant API calls
import NodeCache from 'node-cache';

const cache = new NodeCache({ stdTTL: 3600 }); // 1 hour cache

export async function getCachedNews(keyword: string) {
  const cacheKey = `news:${keyword}`;
  const cached = cache.get(cacheKey);
  
  if (cached) {
    return cached;
  }
  
  const news = await crawlNews(keyword);
  cache.set(cacheKey, news);
  return news;
}
```
