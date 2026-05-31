---
name: marketing-pipeline-share-content-automation
description: Automated AI content pipeline from research to video generation using Claude, OpenAI, and Remotion
triggers:
  - automate content creation from research to video
  - generate marketing content with AI pipeline
  - create automated content workflow with Claude
  - build AI-powered content generation system
  - set up automatic video content from articles
  - use marketing pipeline for content automation
  - integrate AI research and video rendering
  - automate social media content production
---

# Marketing Pipeline Share - AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

Marketing Pipeline Share is a comprehensive AI-powered content automation system that handles the entire content lifecycle: from researching trending topics, generating multi-format articles (in multiple languages), to automatically rendering videos and infographics. It integrates Claude 3, OpenAI, web scraping APIs, and Remotion for video generation.

**Key capabilities:**
- Auto-scraping news from TechCrunch, a16z, Twitter/X, LinkedIn (last 24h)
- Multi-format content generation (Toplist, POV, Case Study, How-to)
- Bilingual support (English & Vietnamese) with customizable tone
- Automatic video/infographic rendering via Remotion
- Next.js web interface for content management

## Installation

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
pnpm install
# or
yarn install

# Set up environment variables
cp .env.example .env.local
```

### Required Environment Variables

```bash
# AI Services
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Web Scraping (RapidAPI)
RAPIDAPI_KEY=your_rapidapi_key

# Next.js Configuration
NEXT_PUBLIC_API_URL=http://localhost:3000

# Remotion Configuration (for video rendering)
REMOTION_LICENSE_KEY=your_remotion_license
```

### Run Development Server

```bash
npm run dev
# or
pnpm dev
# or
yarn dev
```

Access at `http://localhost:3000`

## Core Components

### 1. Research Module (Auto-Scan)

Automatically crawls and analyzes content from major news sources.

```typescript
// lib/research/crawler.ts
import { RapidAPI } from '@/lib/apis/rapidapi';

interface ResearchQuery {
  keyword: string;
  sources: ('techcrunch' | 'a16z' | 'twitter' | 'linkedin')[];
  timeframe: '24h' | '48h' | '7d';
}

export async function performResearch(query: ResearchQuery) {
  const rapidAPI = new RapidAPI(process.env.RAPIDAPI_KEY!);
  
  const results = await Promise.all(
    query.sources.map(source => 
      rapidAPI.scrapeSource(source, query.keyword, query.timeframe)
    )
  );
  
  return {
    insights: extractInsights(results),
    articles: results.flat(),
    trends: analyzeTrends(results)
  };
}

// Usage example
const research = await performResearch({
  keyword: 'AI automation',
  sources: ['techcrunch', 'twitter'],
  timeframe: '24h'
});
```

### 2. Content Generation with AI

Generate multi-format content using Claude or OpenAI.

```typescript
// lib/content/generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

interface ContentConfig {
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  researchData: any;
}

export async function generateContent(config: ContentConfig) {
  const anthropic = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY
  });

  const prompt = buildPrompt(config);
  
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [{
      role: 'user',
      content: prompt
    }]
  });

  return {
    content: message.content[0].text,
    metadata: {
      format: config.format,
      language: config.language,
      wordCount: countWords(message.content[0].text)
    }
  };
}

function buildPrompt(config: ContentConfig): string {
  const formatTemplates = {
    'toplist': 'Create a top 10 list about',
    'pov': 'Write a point-of-view article analyzing',
    'case-study': 'Develop a detailed case study on',
    'how-to': 'Write a comprehensive how-to guide for'
  };

  const toneModifiers = {
    'expert': 'Use professional, authoritative language.',
    'friendly': 'Use conversational, approachable language.',
    'humorous': 'Use engaging, witty language with humor.'
  };

  return `${formatTemplates[config.format]} the following research data:
  
${JSON.stringify(config.researchData, null, 2)}

Language: ${config.language === 'en' ? 'English' : 'Vietnamese'}
${toneModifiers[config.tone]}

Include data-backed insights and specific examples from the research.`;
}
```

### 3. Bilingual Content Generation

Generate content in both English and Vietnamese simultaneously.

```typescript
// lib/content/bilingual.ts
export async function generateBilingualContent(
  researchData: any,
  format: string,
  tone: string
) {
  const [englishContent, vietnameseContent] = await Promise.all([
    generateContent({
      format: format as any,
      language: 'en',
      tone: tone as any,
      researchData
    }),
    generateContent({
      format: format as any,
      language: 'vi',
      tone: tone as any,
      researchData
    })
  ]);

  return {
    en: englishContent,
    vi: vietnameseContent
  };
}
```

### 4. Video Generation with Remotion

Automatically render videos from generated content.

```typescript
// lib/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  content: string;
  aspectRatio: '16:9' | '9:16' | '1:1';
  platform: 'reels' | 'tiktok' | 'shorts' | 'general';
}

export async function renderContentVideo(config: VideoConfig) {
  const compositionId = 'ContentVideo';
  const bundleLocation = await bundle(
    path.join(process.cwd(), 'remotion/index.ts')
  );

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: {
      content: config.content,
      aspectRatio: config.aspectRatio
    }
  });

  const outputLocation = path.join(
    process.cwd(),
    'public/videos',
    `output-${Date.now()}.mp4`
  );

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: {
      content: config.content,
      aspectRatio: config.aspectRatio
    }
  });

  return outputLocation;
}

// Usage
const videoPath = await renderContentVideo({
  content: generatedContent.content,
  aspectRatio: '9:16',
  platform: 'reels'
});
```

### 5. Complete Pipeline Workflow

End-to-end content automation pipeline.

```typescript
// lib/pipeline/workflow.ts
export async function runContentPipeline(
  keyword: string,
  options: {
    format: 'toplist' | 'pov' | 'case-study' | 'how-to';
    generateVideo?: boolean;
    platforms?: ('reels' | 'tiktok' | 'shorts')[];
  }
) {
  // Step 1: Research
  console.log('📡 Starting research phase...');
  const research = await performResearch({
    keyword,
    sources: ['techcrunch', 'twitter', 'linkedin'],
    timeframe: '24h'
  });

  // Step 2: Generate bilingual content
  console.log('🧠 Generating content...');
  const content = await generateBilingualContent(
    research,
    options.format,
    'expert'
  );

  // Step 3: Generate videos (optional)
  let videos = {};
  if (options.generateVideo && options.platforms) {
    console.log('🎬 Rendering videos...');
    
    const videoPromises = options.platforms.map(async (platform) => {
      const aspectRatio = platform === 'reels' || platform === 'tiktok' 
        ? '9:16' 
        : '16:9';
      
      const videoPath = await renderContentVideo({
        content: content.en.content,
        aspectRatio,
        platform
      });
      
      return { platform, path: videoPath };
    });

    const renderedVideos = await Promise.all(videoPromises);
    videos = Object.fromEntries(
      renderedVideos.map(v => [v.platform, v.path])
    );
  }

  return {
    research: {
      insights: research.insights,
      articleCount: research.articles.length
    },
    content: {
      english: content.en,
      vietnamese: content.vi
    },
    videos
  };
}
```

## API Routes (Next.js)

### Content Generation Endpoint

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline/workflow';

export async function POST(request: NextRequest) {
  try {
    const { keyword, format, generateVideo, platforms } = await request.json();

    const result = await runContentPipeline(keyword, {
      format,
      generateVideo,
      platforms
    });

    return NextResponse.json({
      success: true,
      data: result
    });
  } catch (error) {
    return NextResponse.json({
      success: false,
      error: error.message
    }, { status: 500 });
  }
}
```

### Research Endpoint

```typescript
// app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { performResearch } from '@/lib/research/crawler';

export async function POST(request: NextRequest) {
  try {
    const { keyword, sources, timeframe } = await request.json();

    const research = await performResearch({
      keyword,
      sources: sources || ['techcrunch', 'twitter'],
      timeframe: timeframe || '24h'
    });

    return NextResponse.json({
      success: true,
      data: research
    });
  } catch (error) {
    return NextResponse.json({
      success: false,
      error: error.message
    }, { status: 500 });
  }
}
```

## Frontend Usage Example

```typescript
// components/ContentGenerator.tsx
'use client';

import { useState } from 'react';

export function ContentGenerator() {
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState(null);

  async function generateContent(formData: FormData) {
    setLoading(true);
    
    const response = await fetch('/api/generate', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        keyword: formData.get('keyword'),
        format: formData.get('format'),
        generateVideo: formData.get('generateVideo') === 'on',
        platforms: ['reels', 'tiktok']
      })
    });

    const data = await response.json();
    setResult(data.data);
    setLoading(false);
  }

  return (
    <div>
      <form action={generateContent}>
        <input name="keyword" placeholder="Enter keyword..." required />
        <select name="format">
          <option value="toplist">Top List</option>
          <option value="pov">POV</option>
          <option value="case-study">Case Study</option>
          <option value="how-to">How-to Guide</option>
        </select>
        <label>
          <input type="checkbox" name="generateVideo" />
          Generate Video
        </label>
        <button type="submit" disabled={loading}>
          {loading ? 'Generating...' : 'Generate Content'}
        </button>
      </form>

      {result && (
        <div>
          <h3>English Content</h3>
          <p>{result.content.english.content}</p>
          
          <h3>Vietnamese Content</h3>
          <p>{result.content.vietnamese.content}</p>
          
          {result.videos && (
            <div>
              <h3>Generated Videos</h3>
              {Object.entries(result.videos).map(([platform, path]) => (
                <video key={platform} src={path} controls />
              ))}
            </div>
          )}
        </div>
      )}
    </div>
  );
}
```

## Configuration Patterns

### Custom AI Provider Configuration

```typescript
// lib/config/ai-providers.ts
export const aiProviderConfig = {
  claude: {
    model: 'claude-3-5-sonnet-20241022',
    maxTokens: 4096,
    temperature: 0.7
  },
  openai: {
    model: 'gpt-4-turbo-preview',
    maxTokens: 4096,
    temperature: 0.7
  }
};

export function getAIProvider(provider: 'claude' | 'openai') {
  const config = aiProviderConfig[provider];
  
  if (provider === 'claude') {
    return new Anthropic({
      apiKey: process.env.ANTHROPIC_API_KEY
    });
  } else {
    return new OpenAI({
      apiKey: process.env.OPENAI_API_KEY
    });
  }
}
```

### Remotion Video Configuration

```typescript
// remotion/config.ts
export const videoConfig = {
  fps: 30,
  durationInFrames: 900, // 30 seconds
  width: 1080,
  height: 1920, // 9:16 for Reels/TikTok
  backgroundColor: '#ffffff'
};

export const platformPresets = {
  reels: { width: 1080, height: 1920 },
  tiktok: { width: 1080, height: 1920 },
  shorts: { width: 1080, height: 1920 },
  general: { width: 1920, height: 1080 }
};
```

## Troubleshooting

### API Rate Limits

```typescript
// lib/utils/rate-limiter.ts
export class RateLimiter {
  private queue: Array<() => Promise<any>> = [];
  private processing = false;
  
  async add<T>(fn: () => Promise<T>): Promise<T> {
    return new Promise((resolve, reject) => {
      this.queue.push(async () => {
        try {
          const result = await fn();
          resolve(result);
        } catch (error) {
          reject(error);
        }
      });
      
      if (!this.processing) {
        this.process();
      }
    });
  }
  
  private async process() {
    this.processing = true;
    
    while (this.queue.length > 0) {
      const fn = this.queue.shift()!;
      await fn();
      await new Promise(resolve => setTimeout(resolve, 1000)); // 1s delay
    }
    
    this.processing = false;
  }
}

// Usage
const limiter = new RateLimiter();
const research = await limiter.add(() => performResearch(query));
```

### Video Rendering Memory Issues

If video rendering fails with memory errors:

```typescript
// Render videos sequentially instead of parallel
async function renderVideosSequentially(configs: VideoConfig[]) {
  const results = [];
  
  for (const config of configs) {
    const video = await renderContentVideo(config);
    results.push(video);
    
    // Allow garbage collection between renders
    await new Promise(resolve => setTimeout(resolve, 2000));
  }
  
  return results;
}
```

### Claude API Timeout

```typescript
// Implement retry logic
async function generateContentWithRetry(config: ContentConfig, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await generateContent(config);
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      await new Promise(resolve => setTimeout(resolve, 2000 * (i + 1)));
    }
  }
}
```

## Common Workflows

### Daily Content Automation

```typescript
// scripts/daily-automation.ts
async function dailyContentGeneration() {
  const keywords = ['AI trends', 'tech innovation', 'marketing automation'];
  
  for (const keyword of keywords) {
    const result = await runContentPipeline(keyword, {
      format: 'toplist',
      generateVideo: true,
      platforms: ['reels', 'tiktok']
    });
    
    // Save to database or CMS
    await saveToDatabase(result);
  }
}

// Run with cron
// 0 9 * * * node scripts/daily-automation.ts
```

### Batch Processing

```typescript
async function batchProcess(keywords: string[]) {
  const results = await Promise.allSettled(
    keywords.map(keyword => 
      runContentPipeline(keyword, {
        format: 'how-to',
        generateVideo: false
      })
    )
  );
  
  return results.filter(r => r.status === 'fulfilled');
}
```
