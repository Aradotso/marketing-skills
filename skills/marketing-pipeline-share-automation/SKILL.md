---
name: marketing-pipeline-share-automation
description: Automated AI content pipeline for research, scriptwriting, and video generation using Claude, OpenAI, and Remotion
triggers:
  - how do I automate content creation with AI
  - set up automated marketing content pipeline
  - generate blog posts and videos from keywords automatically
  - use Claude and OpenAI for content automation
  - create auto-research content system
  - build AI-powered content workflow
  - implement Remotion video generation pipeline
  - automate social media content with AI
---

# Marketing Pipeline Share - AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

Complete automation system for content creation: from research and scriptwriting to automated video generation. Leverages Claude 3, OpenAI, and Remotion to transform keywords into full content assets.

## What This Project Does

Marketing Pipeline Share is an end-to-end content automation system that:

- **Auto-researches** trending topics from TechCrunch, a16z, Twitter/X, LinkedIn (last 24h)
- **Generates multi-format content** (listicles, POV pieces, case studies, how-tos) in English and Vietnamese
- **Renders videos automatically** using Remotion for Reels, TikTok, YouTube Shorts
- **Creates infographics** from written content
- **Provides API integration** with OpenAI, Anthropic Claude, and RapidAPI

Built with Next.js + TypeScript, optimized for content creators and marketers.

## Installation

### Prerequisites

```bash
# Node.js 18+ required
node --version

# Package manager
npm --version
# or
yarn --version
```

### Setup Steps

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
yarn install

# Copy environment template
cp .env.example .env.local
```

### Environment Configuration

Create `.env.local` with these variables:

```bash
# AI API Keys
OPENAI_API_KEY=your_openai_key_here
ANTHROPIC_API_KEY=your_claude_key_here

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key_here
TWITTER_API_KEY=your_twitter_api_key_here

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Remotion (Video Generation)
REMOTION_LICENSE_KEY=your_remotion_license_key

# Next.js
NEXT_PUBLIC_API_URL=http://localhost:3000
```

### Start Development Server

```bash
npm run dev
# or
yarn dev
```

Access at `http://localhost:3000`

## Core API Usage

### 1. Research Module - Auto-Scan Content

```typescript
import { ResearchService } from '@/services/research';

// Initialize research service
const researcher = new ResearchService({
  openaiKey: process.env.OPENAI_API_KEY!,
  rapidApiKey: process.env.RAPIDAPI_KEY!,
});

// Auto-scan trending topics
async function scanTrendingTopics(keyword: string) {
  const results = await researcher.scanSources({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeframe: '24h',
    language: 'en',
  });

  return {
    articles: results.articles,
    insights: results.extractedInsights,
    statistics: results.dataPoints,
  };
}

// Example usage
const aiTrends = await scanTrendingTopics('AI automation');
console.log(aiTrends.insights);
```

### 2. Content Generation - Multi-Format

```typescript
import { ContentGenerator } from '@/services/content';

const generator = new ContentGenerator({
  claudeKey: process.env.ANTHROPIC_API_KEY!,
  openaiKey: process.env.OPENAI_API_KEY!,
});

// Generate blog post in multiple formats
async function generateContent(config: {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  tone: 'expert' | 'friendly' | 'humorous';
  languages: string[];
}) {
  const content = await generator.create({
    keyword: config.keyword,
    format: config.format,
    tone: config.tone,
    languages: config.languages,
    researchData: await scanTrendingTopics(config.keyword),
  });

  return {
    title: content.title,
    body: content.body,
    metadata: content.metadata,
    translations: content.translations,
  };
}

// Example: Generate listicle in English and Vietnamese
const article = await generateContent({
  keyword: 'AI content tools',
  format: 'toplist',
  tone: 'friendly',
  languages: ['en', 'vi'],
});
```

### 3. Video Generation with Remotion

```typescript
import { VideoRenderer } from '@/services/video';
import { Composition } from 'remotion';

const renderer = new VideoRenderer({
  licenseKey: process.env.REMOTION_LICENSE_KEY!,
});

// Render video from content
async function renderContentVideo(content: {
  title: string;
  points: string[];
  images: string[];
}) {
  const video = await renderer.render({
    composition: 'ContentVideo',
    props: {
      title: content.title,
      points: content.points,
      images: content.images,
      duration: 60, // seconds
    },
    format: {
      width: 1080,
      height: 1920, // Vertical for Reels/TikTok
      fps: 30,
    },
    output: 'video.mp4',
  });

  return video.outputPath;
}

// Example usage
const videoPath = await renderContentVideo({
  title: 'Top 5 AI Tools for 2026',
  points: article.body.split('\n').slice(0, 5),
  images: article.metadata.images,
});
```

## API Routes Pattern

### POST /api/research

```typescript
// pages/api/research.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { ResearchService } from '@/services/research';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  const { keyword, sources, timeframe } = req.body;

  try {
    const researcher = new ResearchService({
      openaiKey: process.env.OPENAI_API_KEY!,
      rapidApiKey: process.env.RAPIDAPI_KEY!,
    });

    const results = await researcher.scanSources({
      keyword,
      sources,
      timeframe,
    });

    res.status(200).json({ success: true, data: results });
  } catch (error) {
    res.status(500).json({ 
      error: 'Research failed', 
      message: error.message 
    });
  }
}
```

### POST /api/generate

```typescript
// pages/api/generate.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { ContentGenerator } from '@/services/content';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  const { keyword, format, tone, languages, researchData } = req.body;

  try {
    const generator = new ContentGenerator({
      claudeKey: process.env.ANTHROPIC_API_KEY!,
      openaiKey: process.env.OPENAI_API_KEY!,
    });

    const content = await generator.create({
      keyword,
      format,
      tone,
      languages,
      researchData,
    });

    res.status(200).json({ success: true, data: content });
  } catch (error) {
    res.status(500).json({ 
      error: 'Generation failed', 
      message: error.message 
    });
  }
}
```

### POST /api/render-video

```typescript
// pages/api/render-video.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { VideoRenderer } from '@/services/video';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  const { title, points, images, format } = req.body;

  try {
    const renderer = new VideoRenderer({
      licenseKey: process.env.REMOTION_LICENSE_KEY!,
    });

    const video = await renderer.render({
      composition: 'ContentVideo',
      props: { title, points, images },
      format: format || { width: 1080, height: 1920, fps: 30 },
    });

    res.status(200).json({ 
      success: true, 
      videoUrl: video.outputPath 
    });
  } catch (error) {
    res.status(500).json({ 
      error: 'Rendering failed', 
      message: error.message 
    });
  }
}
```

## Complete Pipeline Example

```typescript
// lib/pipeline.ts
import { ResearchService } from '@/services/research';
import { ContentGenerator } from '@/services/content';
import { VideoRenderer } from '@/services/video';

export class ContentPipeline {
  private researcher: ResearchService;
  private generator: ContentGenerator;
  private renderer: VideoRenderer;

  constructor() {
    this.researcher = new ResearchService({
      openaiKey: process.env.OPENAI_API_KEY!,
      rapidApiKey: process.env.RAPIDAPI_KEY!,
    });

    this.generator = new ContentGenerator({
      claudeKey: process.env.ANTHROPIC_API_KEY!,
      openaiKey: process.env.OPENAI_API_KEY!,
    });

    this.renderer = new VideoRenderer({
      licenseKey: process.env.REMOTION_LICENSE_KEY!,
    });
  }

  async execute(config: {
    keyword: string;
    format: 'toplist' | 'pov' | 'case-study' | 'how-to';
    tone: 'expert' | 'friendly' | 'humorous';
    languages: string[];
    generateVideo: boolean;
  }) {
    // Step 1: Research
    console.log('🔍 Researching trending topics...');
    const research = await this.researcher.scanSources({
      keyword: config.keyword,
      sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
      timeframe: '24h',
    });

    // Step 2: Generate Content
    console.log('✍️ Generating content...');
    const content = await this.generator.create({
      keyword: config.keyword,
      format: config.format,
      tone: config.tone,
      languages: config.languages,
      researchData: research,
    });

    // Step 3: Render Video (optional)
    let videoUrl = null;
    if (config.generateVideo) {
      console.log('🎬 Rendering video...');
      const video = await this.renderer.render({
        composition: 'ContentVideo',
        props: {
          title: content.title,
          points: content.body.split('\n').filter(p => p.trim()),
          images: content.metadata.images,
        },
        format: { width: 1080, height: 1920, fps: 30 },
      });
      videoUrl = video.outputPath;
    }

    return {
      research: {
        articles: research.articles.length,
        insights: research.extractedInsights,
      },
      content: {
        title: content.title,
        body: content.body,
        translations: content.translations,
      },
      video: videoUrl,
    };
  }
}

// Usage
const pipeline = new ContentPipeline();

const result = await pipeline.execute({
  keyword: 'AI automation tools 2026',
  format: 'toplist',
  tone: 'friendly',
  languages: ['en', 'vi'],
  generateVideo: true,
});

console.log('✅ Pipeline complete:', result);
```

## Frontend Integration

```typescript
// components/ContentCreator.tsx
import { useState } from 'react';

export function ContentCreator() {
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState(null);

  async function createContent(formData: {
    keyword: string;
    format: string;
    tone: string;
  }) {
    setLoading(true);

    try {
      // Step 1: Research
      const researchRes = await fetch('/api/research', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          keyword: formData.keyword,
          sources: ['techcrunch', 'twitter'],
          timeframe: '24h',
        }),
      });
      const research = await researchRes.json();

      // Step 2: Generate
      const contentRes = await fetch('/api/generate', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          keyword: formData.keyword,
          format: formData.format,
          tone: formData.tone,
          languages: ['en', 'vi'],
          researchData: research.data,
        }),
      });
      const content = await contentRes.json();

      // Step 3: Render video
      const videoRes = await fetch('/api/render-video', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          title: content.data.title,
          points: content.data.body.split('\n').slice(0, 5),
          images: content.data.metadata.images,
        }),
      });
      const video = await videoRes.json();

      setResult({ content: content.data, video: video.videoUrl });
    } catch (error) {
      console.error('Pipeline failed:', error);
    } finally {
      setLoading(false);
    }
  }

  return (
    <div>
      <h1>AI Content Pipeline</h1>
      {/* Form UI here */}
      {loading && <p>Processing...</p>}
      {result && (
        <div>
          <h2>{result.content.title}</h2>
          <p>{result.content.body}</p>
          {result.video && <video src={result.video} controls />}
        </div>
      )}
    </div>
  );
}
```

## Configuration

### Custom Tone Presets

```typescript
// config/tones.ts
export const TONE_PRESETS = {
  expert: {
    systemPrompt: 'Write as an industry expert with data-backed insights',
    temperature: 0.7,
    maxTokens: 2000,
  },
  friendly: {
    systemPrompt: 'Write in a conversational, approachable style',
    temperature: 0.9,
    maxTokens: 1500,
  },
  humorous: {
    systemPrompt: 'Write with wit and humor while staying informative',
    temperature: 1.0,
    maxTokens: 1800,
  },
};
```

### Video Templates

```typescript
// remotion/compositions.ts
import { Composition } from 'remotion';
import { ContentVideo } from './ContentVideo';
import { InfographicVideo } from './InfographicVideo';

export const RemotionRoot = () => {
  return (
    <>
      <Composition
        id="ContentVideo"
        component={ContentVideo}
        durationInFrames={1800}
        fps={30}
        width={1080}
        height={1920}
      />
      <Composition
        id="InfographicVideo"
        component={InfographicVideo}
        durationInFrames={900}
        fps={30}
        width={1080}
        height={1080}
      />
    </>
  );
};
```

## Common Patterns

### Error Handling

```typescript
async function safeExecute<T>(
  operation: () => Promise<T>,
  fallback: T
): Promise<T> {
  try {
    return await operation();
  } catch (error) {
    console.error('Operation failed:', error);
    return fallback;
  }
}

// Usage
const content = await safeExecute(
  () => generator.create({ keyword, format, tone }),
  { title: 'Fallback Title', body: '', metadata: {} }
);
```

### Rate Limiting

```typescript
import pLimit from 'p-limit';

const limit = pLimit(3); // Max 3 concurrent API calls

async function batchGenerate(keywords: string[]) {
  const promises = keywords.map(keyword =>
    limit(() => generator.create({ keyword, format: 'toplist' }))
  );

  return Promise.all(promises);
}
```

### Caching Research Results

```typescript
import NodeCache from 'node-cache';

const cache = new NodeCache({ stdTTL: 3600 }); // 1 hour

async function cachedResearch(keyword: string) {
  const cacheKey = `research:${keyword}`;
  
  const cached = cache.get(cacheKey);
  if (cached) return cached;

  const results = await researcher.scanSources({ keyword });
  cache.set(cacheKey, results);
  
  return results;
}
```

## Troubleshooting

### Issue: API Rate Limits

**Solution:** Implement exponential backoff:

```typescript
async function retryWithBackoff<T>(
  fn: () => Promise<T>,
  maxRetries = 3
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      const delay = Math.pow(2, i) * 1000;
      await new Promise(resolve => setTimeout(resolve, delay));
    }
  }
  throw new Error('Max retries exceeded');
}
```

### Issue: Video Rendering Timeout

**Solution:** Use background jobs:

```typescript
import { Queue } from 'bullmq';

const videoQueue = new Queue('video-rendering');

async function queueVideoRender(props: any) {
  await videoQueue.add('render', props, {
    attempts: 3,
    backoff: { type: 'exponential', delay: 2000 },
  });
}
```

### Issue: Memory Leaks with Large Content

**Solution:** Stream processing:

```typescript
import { pipeline } from 'stream/promises';

async function streamGenerate(keyword: string) {
  const stream = await generator.createStream({ keyword });
  const chunks: string[] = [];
  
  await pipeline(
    stream,
    async function* (source) {
      for await (const chunk of source) {
        chunks.push(chunk);
        yield chunk;
      }
    }
  );
  
  return chunks.join('');
}
```

### Issue: Missing Environment Variables

**Solution:** Validation on startup:

```typescript
// lib/validateEnv.ts
const requiredEnvVars = [
  'OPENAI_API_KEY',
  'ANTHROPIC_API_KEY',
  'RAPIDAPI_KEY',
  'REMOTION_LICENSE_KEY',
];

export function validateEnv() {
  const missing = requiredEnvVars.filter(key => !process.env[key]);
  
  if (missing.length > 0) {
    throw new Error(
      `Missing required environment variables: ${missing.join(', ')}`
    );
  }
}

// Call in _app.tsx or startup script
validateEnv();
```

This skill enables AI coding agents to effectively implement and extend the Marketing Pipeline Share automation system for comprehensive content creation workflows.
