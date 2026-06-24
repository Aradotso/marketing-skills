---
name: marketing-pipeline-automation
description: Automated AI content pipeline for research, scriptwriting, video generation, and social media posting using Claude/OpenAI and Remotion
triggers:
  - automate my content creation pipeline
  - set up AI content generation with video rendering
  - create automated marketing content workflow
  - build content pipeline from research to video
  - generate social media content automatically
  - scrape news and create content with AI
  - set up remotion video generation pipeline
  - automate content research and posting
---

# Marketing Pipeline Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

Marketing Pipeline Automation is a complete AI-powered content creation system that automates the entire workflow from research to publication. It scrapes trending news from sources like TechCrunch, Twitter, and LinkedIn, generates content in multiple formats using Claude/OpenAI, renders videos with Remotion, and can auto-post to social platforms.

**Key capabilities:**
- Automated news research and data extraction
- Multi-format content generation (listicles, POV, case studies, how-tos)
- Bilingual support (English/Vietnamese)
- Automated video rendering with Remotion
- Social media integration for auto-posting

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

## Configuration

### Environment Variables

Create a `.env.local` file with the following required variables:

```bash
# AI Providers
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_connection

# Social Media APIs (optional)
FACEBOOK_ACCESS_TOKEN=your_token
LINKEDIN_API_KEY=your_key

# Remotion (for video rendering)
REMOTION_LICENSE_KEY=your_license
```

### Basic Configuration File

```typescript
// config/pipeline.config.ts
export const pipelineConfig = {
  research: {
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeframe: '24h',
    maxArticles: 10
  },
  content: {
    formats: ['toplist', 'pov', 'casestudy', 'howto'],
    languages: ['en', 'vi'],
    tones: ['expert', 'friendly', 'humorous']
  },
  video: {
    platform: ['reels', 'tiktok', 'shorts'],
    dimensions: {
      reels: { width: 1080, height: 1920 },
      shorts: { width: 1080, height: 1920 },
      tiktok: { width: 1080, height: 1920 }
    }
  }
};
```

## Core Usage Patterns

### 1. Research Phase - Automated News Scraping

```typescript
// lib/research/scraper.ts
import { ResearchService } from './services/research';

export async function scrapeLatestNews(keyword: string) {
  const researchService = new ResearchService({
    apiKey: process.env.RAPIDAPI_KEY!,
    sources: ['techcrunch', 'twitter', 'linkedin']
  });

  const articles = await researchService.fetchArticles({
    keyword,
    timeframe: '24h',
    maxResults: 10
  });

  const insights = await researchService.extractInsights(articles);
  
  return {
    articles,
    insights,
    trends: researchService.analyzeTrends(articles)
  };
}

// Usage
const newsData = await scrapeLatestNews('AI automation');
console.log(`Found ${newsData.articles.length} articles`);
```

### 2. Content Generation with AI

```typescript
// lib/content/generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

export class ContentGenerator {
  private anthropic: Anthropic;
  private openai: OpenAI;

  constructor() {
    this.anthropic = new Anthropic({
      apiKey: process.env.ANTHROPIC_API_KEY!
    });
    this.openai = new OpenAI({
      apiKey: process.env.OPENAI_API_KEY!
    });
  }

  async generateArticle(params: {
    keyword: string;
    format: 'toplist' | 'pov' | 'casestudy' | 'howto';
    language: 'en' | 'vi';
    tone: 'expert' | 'friendly' | 'humorous';
    researchData: any;
  }) {
    const prompt = this.buildPrompt(params);

    // Use Claude for content generation
    const message = await this.anthropic.messages.create({
      model: 'claude-3-5-sonnet-20241022',
      max_tokens: 4096,
      messages: [{
        role: 'user',
        content: prompt
      }]
    });

    const content = message.content[0].type === 'text' 
      ? message.content[0].text 
      : '';

    return {
      content,
      metadata: {
        format: params.format,
        language: params.language,
        wordCount: content.split(' ').length,
        generatedAt: new Date()
      }
    };
  }

  private buildPrompt(params: any): string {
    return `
      Create a ${params.format} article about "${params.keyword}" in ${params.language}.
      Tone: ${params.tone}
      
      Research data:
      ${JSON.stringify(params.researchData, null, 2)}
      
      Requirements:
      - Use data-backed insights
      - Include specific examples
      - Make it engaging and actionable
      - Optimize for ${params.language === 'vi' ? 'Vietnamese' : 'English'} audience
    `;
  }
}

// Usage
const generator = new ContentGenerator();
const article = await generator.generateArticle({
  keyword: 'AI content automation',
  format: 'howto',
  language: 'en',
  tone: 'expert',
  researchData: newsData
});
```

### 3. Video Generation with Remotion

```typescript
// lib/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

export class VideoRenderer {
  async renderContentVideo(params: {
    content: string;
    platform: 'reels' | 'tiktok' | 'shorts';
    outputPath: string;
  }) {
    // Bundle the Remotion project
    const bundleLocation = await bundle({
      entryPoint: path.resolve('./remotion/index.ts'),
      webpackOverride: (config) => config,
    });

    // Get composition
    const composition = await selectComposition({
      serveUrl: bundleLocation,
      id: 'ContentVideo',
      inputProps: {
        content: params.content,
        platform: params.platform
      }
    });

    // Render video
    await renderMedia({
      composition,
      serveUrl: bundleLocation,
      codec: 'h264',
      outputLocation: params.outputPath,
      inputProps: {
        content: params.content,
        platform: params.platform
      }
    });

    return {
      videoPath: params.outputPath,
      duration: composition.durationInFrames / composition.fps,
      dimensions: {
        width: composition.width,
        height: composition.height
      }
    };
  }
}

// Usage
const renderer = new VideoRenderer();
const video = await renderer.renderContentVideo({
  content: article.content,
  platform: 'reels',
  outputPath: './output/video.mp4'
});
```

### 4. Remotion Video Component

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, Sequence, useCurrentFrame, useVideoConfig } from 'remotion';

export const ContentVideo: React.FC<{
  content: string;
  platform: string;
}> = ({ content, platform }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  const contentLines = content.split('\n').filter(line => line.trim());

  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a1a' }}>
      {contentLines.map((line, index) => (
        <Sequence
          key={index}
          from={index * fps * 2}
          durationInFrames={fps * 2}
        >
          <AbsoluteFill
            style={{
              justifyContent: 'center',
              alignItems: 'center',
              padding: 40
            }}
          >
            <div
              style={{
                fontSize: 48,
                color: 'white',
                textAlign: 'center',
                fontWeight: 'bold',
                opacity: frame < 15 ? frame / 15 : 1
              }}
            >
              {line}
            </div>
          </AbsoluteFill>
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};
```

### 5. Complete Pipeline Orchestration

```typescript
// lib/pipeline/orchestrator.ts
import { scrapeLatestNews } from '../research/scraper';
import { ContentGenerator } from '../content/generator';
import { VideoRenderer } from '../video/renderer';
import { SocialMediaPublisher } from '../social/publisher';

export class ContentPipeline {
  private generator: ContentGenerator;
  private renderer: VideoRenderer;
  private publisher: SocialMediaPublisher;

  constructor() {
    this.generator = new ContentGenerator();
    this.renderer = new VideoRenderer();
    this.publisher = new SocialMediaPublisher();
  }

  async runFullPipeline(params: {
    keyword: string;
    format: string;
    language: string;
    platforms: string[];
    autoPublish: boolean;
  }) {
    console.log('Step 1: Research phase...');
    const researchData = await scrapeLatestNews(params.keyword);

    console.log('Step 2: Content generation...');
    const article = await this.generator.generateArticle({
      keyword: params.keyword,
      format: params.format as any,
      language: params.language as any,
      tone: 'expert',
      researchData
    });

    console.log('Step 3: Video rendering...');
    const videos = await Promise.all(
      params.platforms.map(platform =>
        this.renderer.renderContentVideo({
          content: article.content,
          platform: platform as any,
          outputPath: `./output/${platform}_${Date.now()}.mp4`
        })
      )
    );

    console.log('Step 4: Publishing...');
    if (params.autoPublish) {
      const results = await this.publisher.publishToMultiplePlatforms({
        content: article.content,
        videos,
        platforms: params.platforms
      });
      return { article, videos, publishResults: results };
    }

    return { article, videos };
  }
}

// Usage
const pipeline = new ContentPipeline();
const result = await pipeline.runFullPipeline({
  keyword: 'AI marketing automation',
  format: 'toplist',
  language: 'en',
  platforms: ['reels', 'tiktok'],
  autoPublish: false
});
```

### 6. API Route Example (Next.js)

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { ContentPipeline } from '@/lib/pipeline/orchestrator';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const { keyword, format, language, platforms, autoPublish } = body;

    const pipeline = new ContentPipeline();
    const result = await pipeline.runFullPipeline({
      keyword,
      format,
      language,
      platforms,
      autoPublish: autoPublish || false
    });

    return NextResponse.json({
      success: true,
      data: result
    });
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

### 7. Frontend Component Example

```typescript
// components/ContentGenerator.tsx
'use client';

import { useState } from 'react';

export default function ContentGenerator() {
  const [keyword, setKeyword] = useState('');
  const [format, setFormat] = useState('toplist');
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState(null);

  const handleGenerate = async () => {
    setLoading(true);
    try {
      const response = await fetch('/api/generate', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          keyword,
          format,
          language: 'en',
          platforms: ['reels', 'tiktok'],
          autoPublish: false
        })
      });

      const data = await response.json();
      setResult(data);
    } catch (error) {
      console.error('Generation failed:', error);
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="p-6 max-w-2xl mx-auto">
      <h1 className="text-2xl font-bold mb-4">AI Content Generator</h1>
      
      <div className="space-y-4">
        <input
          type="text"
          value={keyword}
          onChange={(e) => setKeyword(e.target.value)}
          placeholder="Enter keyword..."
          className="w-full p-2 border rounded"
        />
        
        <select
          value={format}
          onChange={(e) => setFormat(e.target.value)}
          className="w-full p-2 border rounded"
        >
          <option value="toplist">Top List</option>
          <option value="pov">Point of View</option>
          <option value="casestudy">Case Study</option>
          <option value="howto">How-To Guide</option>
        </select>

        <button
          onClick={handleGenerate}
          disabled={loading || !keyword}
          className="w-full p-3 bg-blue-600 text-white rounded disabled:opacity-50"
        >
          {loading ? 'Generating...' : 'Generate Content'}
        </button>
      </div>

      {result && (
        <div className="mt-6 p-4 bg-gray-100 rounded">
          <h2 className="font-bold mb-2">Result:</h2>
          <pre className="whitespace-pre-wrap text-sm">
            {JSON.stringify(result, null, 2)}
          </pre>
        </div>
      )}
    </div>
  );
}
```

## CLI Commands

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Render video manually (if CLI exposed)
npm run render -- --input content.json --output video.mp4

# Run full pipeline (custom script)
npm run pipeline -- --keyword "AI tools" --format toplist
```

## Troubleshooting

### API Rate Limits

```typescript
// lib/utils/rateLimit.ts
export class RateLimiter {
  private queue: Array<() => Promise<any>> = [];
  private processing = false;

  async add<T>(fn: () => Promise<T>, delay = 1000): Promise<T> {
    return new Promise((resolve, reject) => {
      this.queue.push(async () => {
        try {
          const result = await fn();
          await new Promise(r => setTimeout(r, delay));
          resolve(result);
        } catch (error) {
          reject(error);
        }
      });
      this.process();
    });
  }

  private async process() {
    if (this.processing || this.queue.length === 0) return;
    this.processing = true;
    while (this.queue.length > 0) {
      const fn = this.queue.shift();
      if (fn) await fn();
    }
    this.processing = false;
  }
}
```

### Video Rendering Fails

Check Remotion dependencies and ensure FFMPEG is installed:

```bash
# Install FFMPEG (macOS)
brew install ffmpeg

# Install FFMPEG (Ubuntu)
sudo apt-get install ffmpeg

# Verify installation
ffmpeg -version
```

### Memory Issues with Large Content

```typescript
// Optimize for large batches
const results = await Promise.allSettled(
  batches.map(batch => pipeline.runFullPipeline(batch))
);
```

### Claude/OpenAI API Errors

```typescript
// Add retry logic
async function callAIWithRetry(fn: () => Promise<any>, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      await new Promise(r => setTimeout(r, 2000 * (i + 1)));
    }
  }
}
```
