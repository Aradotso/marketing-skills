---
name: marketing-pipeline-share-ai-content
description: Automated AI content pipeline for research, scriptwriting, and video generation using Claude, OpenAI, and Remotion
triggers:
  - automate my content creation workflow
  - generate videos from article content automatically
  - research and write marketing content with AI
  - create automated content pipeline with Claude
  - build AI-powered content generation system
  - auto-generate social media videos from text
  - set up content automation with Remotion
  - crawl news and generate content automatically
---

# Marketing Pipeline Share - AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the Ultimate AI Content Pipeline, a comprehensive TypeScript-based system that automates content creation from research through video generation. The pipeline integrates Claude 3, OpenAI, web scraping, and Remotion video rendering to create an end-to-end content factory.

## What This Project Does

Marketing Pipeline Share is a complete content automation system that:

- **Auto-scans research**: Crawls news sources (TechCrunch, a16z, Twitter/X, LinkedIn) for fresh data
- **Generates content**: Uses Claude/OpenAI to write articles in multiple formats (listicles, POV, case studies, how-tos)
- **Multi-language support**: Creates content in both English and Vietnamese with customizable tone
- **Video rendering**: Automatically generates videos and infographics from written content using Remotion
- **Platform optimization**: Exports videos optimized for Reels, TikTok, Shorts

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

### Environment Configuration

Create a `.env.local` file in the root directory:

```bash
# AI Provider Keys
OPENAI_API_KEY=your_openai_key_here
ANTHROPIC_API_KEY=your_claude_key_here

# RapidAPI for web scraping
RAPIDAPI_KEY=your_rapidapi_key_here

# Remotion License (if using Remotion Cloud)
REMOTION_LICENSE_KEY=your_remotion_key_here

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Next.js Configuration
NEXT_PUBLIC_API_URL=http://localhost:3000
```

### Development Server

```bash
npm run dev
# or
yarn dev
# or
pnpm dev
```

The application will be available at `http://localhost:3000`

## Core Architecture

### Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app router pages
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI provider integrations
│   │   ├── scraper/     # Web scraping modules
│   │   ├── video/       # Remotion video generation
│   │   └── utils/       # Utility functions
│   └── types/           # TypeScript type definitions
├── remotion/            # Remotion video compositions
└── public/              # Static assets
```

## Key Components & Usage

### 1. Content Research Module

```typescript
// src/lib/scraper/newsScanner.ts
import axios from 'axios';

interface NewsSource {
  url: string;
  keywords: string[];
  dateRange: '24h' | '7d' | '30d';
}

export async function scanNews(sources: NewsSource[]) {
  const headers = {
    'X-RapidAPI-Key': process.env.RAPIDAPI_KEY,
    'X-RapidAPI-Host': 'news-api.rapidapi.com'
  };

  const results = await Promise.all(
    sources.map(async (source) => {
      const response = await axios.get(
        'https://news-api.rapidapi.com/search',
        {
          params: {
            q: source.keywords.join(' OR '),
            time: source.dateRange,
            lang: 'en'
          },
          headers
        }
      );
      return response.data;
    })
  );

  return results.flat();
}

// Usage example
const techNews = await scanNews([
  {
    url: 'techcrunch.com',
    keywords: ['AI', 'startup', 'funding'],
    dateRange: '24h'
  }
]);
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

interface ContentConfig {
  topic: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  researchData: any[];
}

export async function generateContent(config: ContentConfig) {
  const { topic, format, language, tone, researchData } = config;

  const systemPrompt = `You are an expert content writer specializing in ${format} articles.
Tone: ${tone}
Language: ${language}
Use the following research data to write a comprehensive, data-backed article.`;

  const userPrompt = `Topic: ${topic}

Research Data:
${JSON.stringify(researchData, null, 2)}

Write a ${format} article that:
1. Uses specific data points and statistics
2. Includes real examples and case studies
3. Provides actionable insights
4. Is engaging and well-structured`;

  // Using Claude for content generation
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [
      {
        role: 'user',
        content: `${systemPrompt}\n\n${userPrompt}`
      }
    ]
  });

  return message.content[0].type === 'text' 
    ? message.content[0].text 
    : '';
}

// Usage example
const article = await generateContent({
  topic: 'AI in Marketing Automation 2024',
  format: 'toplist',
  language: 'en',
  tone: 'expert',
  researchData: techNews
});
```

### 3. Bilingual Content Generation

```typescript
// src/lib/ai/bilingualGenerator.ts
export async function generateBilingualContent(
  topic: string,
  researchData: any[]
) {
  const [englishContent, vietnameseContent] = await Promise.all([
    generateContent({
      topic,
      format: 'toplist',
      language: 'en',
      tone: 'expert',
      researchData
    }),
    generateContent({
      topic,
      format: 'toplist',
      language: 'vi',
      tone: 'friendly',
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

```typescript
// remotion/compositions/ArticleVideo.tsx
import { AbsoluteFill, useCurrentFrame, interpolate } from 'remotion';

interface ArticleVideoProps {
  title: string;
  points: string[];
  duration: number;
}

export const ArticleVideo: React.FC<ArticleVideoProps> = ({
  title,
  points,
  duration
}) => {
  const frame = useCurrentFrame();
  const opacity = interpolate(frame, [0, 30], [0, 1]);

  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a1a' }}>
      <div style={{ 
        opacity, 
        padding: '60px',
        color: 'white',
        fontSize: '48px',
        fontWeight: 'bold'
      }}>
        {title}
      </div>
      <div style={{ padding: '60px', color: 'white' }}>
        {points.map((point, index) => (
          <div key={index} style={{ 
            marginBottom: '20px',
            fontSize: '32px',
            opacity: interpolate(
              frame,
              [30 * (index + 1), 30 * (index + 2)],
              [0, 1]
            )
          }}>
            • {point}
          </div>
        ))}
      </div>
    </AbsoluteFill>
  );
};
```

```typescript
// src/lib/video/renderVideo.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

export async function renderArticleVideo(
  articleContent: string,
  outputPath: string
) {
  // Parse article into video-friendly format
  const videoData = parseArticleForVideo(articleContent);

  const compositionId = 'ArticleVideo';
  const bundleLocation = await bundle({
    entryPoint: path.join(process.cwd(), 'remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: videoData,
  });

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: videoData,
  });

  return outputPath;
}

function parseArticleForVideo(content: string) {
  const lines = content.split('\n');
  const title = lines[0].replace(/^#\s*/, '');
  const points = lines
    .filter(line => line.trim().startsWith('-') || line.trim().startsWith('•'))
    .map(line => line.replace(/^[-•]\s*/, ''))
    .slice(0, 5);

  return {
    title,
    points,
    duration: 30 * points.length
  };
}
```

### 5. Complete Pipeline Integration

```typescript
// src/lib/pipeline/contentPipeline.ts
import { scanNews } from '../scraper/newsScanner';
import { generateBilingualContent } from '../ai/bilingualGenerator';
import { renderArticleVideo } from '../video/renderVideo';

interface PipelineConfig {
  keywords: string[];
  sources: string[];
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  generateVideo: boolean;
}

export async function runContentPipeline(config: PipelineConfig) {
  console.log('🔍 Step 1: Scanning news sources...');
  const researchData = await scanNews([
    {
      url: config.sources[0],
      keywords: config.keywords,
      dateRange: '24h'
    }
  ]);

  console.log('✍️ Step 2: Generating bilingual content...');
  const content = await generateBilingualContent(
    config.keywords.join(' '),
    researchData
  );

  let videoPath = null;
  if (config.generateVideo) {
    console.log('🎬 Step 3: Rendering video...');
    videoPath = await renderArticleVideo(
      content.en,
      `./output/video-${Date.now()}.mp4`
    );
  }

  console.log('✅ Pipeline complete!');
  
  return {
    content,
    videoPath,
    researchData: researchData.length
  };
}

// Usage example
const result = await runContentPipeline({
  keywords: ['AI', 'Marketing', 'Automation'],
  sources: ['techcrunch.com'],
  format: 'toplist',
  generateVideo: true
});
```

### 6. API Route Example (Next.js)

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline/contentPipeline';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const { keywords, sources, format, generateVideo } = body;

    if (!keywords || !Array.isArray(keywords)) {
      return NextResponse.json(
        { error: 'Keywords array is required' },
        { status: 400 }
      );
    }

    const result = await runContentPipeline({
      keywords,
      sources: sources || ['techcrunch.com'],
      format: format || 'toplist',
      generateVideo: generateVideo ?? true
    });

    return NextResponse.json(result);
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { error: 'Failed to generate content' },
      { status: 500 }
    );
  }
}
```

### 7. Frontend Component Example

```typescript
// src/components/ContentGenerator.tsx
'use client';

import { useState } from 'react';

export default function ContentGenerator() {
  const [keywords, setKeywords] = useState('');
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState<any>(null);

  const handleGenerate = async () => {
    setLoading(true);
    try {
      const response = await fetch('/api/generate', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          keywords: keywords.split(',').map(k => k.trim()),
          format: 'toplist',
          generateVideo: true
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
    <div className="p-6 max-w-4xl mx-auto">
      <h1 className="text-3xl font-bold mb-6">AI Content Generator</h1>
      
      <div className="mb-4">
        <label className="block mb-2 font-semibold">
          Keywords (comma-separated):
        </label>
        <input
          type="text"
          value={keywords}
          onChange={(e) => setKeywords(e.target.value)}
          className="w-full p-3 border rounded"
          placeholder="AI, Marketing, Automation"
        />
      </div>

      <button
        onClick={handleGenerate}
        disabled={loading}
        className="bg-blue-600 text-white px-6 py-3 rounded hover:bg-blue-700 disabled:opacity-50"
      >
        {loading ? 'Generating...' : 'Generate Content'}
      </button>

      {result && (
        <div className="mt-6">
          <h2 className="text-2xl font-bold mb-4">Results:</h2>
          <div className="space-y-4">
            <div className="p-4 bg-gray-100 rounded">
              <h3 className="font-semibold">English Content:</h3>
              <pre className="whitespace-pre-wrap">{result.content.en}</pre>
            </div>
            <div className="p-4 bg-gray-100 rounded">
              <h3 className="font-semibold">Vietnamese Content:</h3>
              <pre className="whitespace-pre-wrap">{result.content.vi}</pre>
            </div>
            {result.videoPath && (
              <div className="p-4 bg-green-100 rounded">
                <h3 className="font-semibold">Video Generated:</h3>
                <p>{result.videoPath}</p>
              </div>
            )}
          </div>
        </div>
      )}
    </div>
  );
}
```

## Common Patterns

### Content Scheduling

```typescript
// src/lib/scheduler/contentScheduler.ts
import cron from 'node-cron';
import { runContentPipeline } from '../pipeline/contentPipeline';

export function scheduleContentGeneration(
  schedule: string, // cron format: '0 9 * * *'
  config: any
) {
  cron.schedule(schedule, async () => {
    console.log('Running scheduled content generation...');
    try {
      await runContentPipeline(config);
    } catch (error) {
      console.error('Scheduled generation failed:', error);
    }
  });
}

// Generate content daily at 9 AM
scheduleContentGeneration('0 9 * * *', {
  keywords: ['AI', 'Technology'],
  sources: ['techcrunch.com'],
  format: 'toplist',
  generateVideo: true
});
```

### Batch Processing

```typescript
export async function batchGenerateContent(topics: string[]) {
  const results = [];
  
  for (const topic of topics) {
    const result = await runContentPipeline({
      keywords: [topic],
      sources: ['techcrunch.com'],
      format: 'toplist',
      generateVideo: false
    });
    results.push({ topic, ...result });
    
    // Rate limiting
    await new Promise(resolve => setTimeout(resolve, 5000));
  }
  
  return results;
}
```

## Troubleshooting

### API Rate Limits

```typescript
// src/lib/utils/rateLimiter.ts
export class RateLimiter {
  private queue: Array<() => Promise<any>> = [];
  private processing = false;
  private delay: number;

  constructor(requestsPerMinute: number) {
    this.delay = 60000 / requestsPerMinute;
  }

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
      this.process();
    });
  }

  private async process() {
    if (this.processing || this.queue.length === 0) return;
    
    this.processing = true;
    const fn = this.queue.shift()!;
    await fn();
    await new Promise(resolve => setTimeout(resolve, this.delay));
    this.processing = false;
    this.process();
  }
}
```

### Video Rendering Memory Issues

For large video renders, increase Node.js memory:

```bash
NODE_OPTIONS="--max-old-space-size=4096" npm run dev
```

### Error Handling

```typescript
// src/lib/utils/errorHandler.ts
export class PipelineError extends Error {
  constructor(
    message: string,
    public stage: 'research' | 'generation' | 'video',
    public originalError?: any
  ) {
    super(message);
    this.name = 'PipelineError';
  }
}

export async function safePipelineRun(config: any) {
  try {
    return await runContentPipeline(config);
  } catch (error) {
    if (error instanceof PipelineError) {
      console.error(`Pipeline failed at ${error.stage}:`, error.message);
    } else {
      console.error('Unexpected error:', error);
    }
    throw error;
  }
}
```

## Build & Deployment

```bash
# Production build
npm run build

# Start production server
npm start

# Remotion video rendering server
npm run remotion:server
```

## Best Practices

1. **Always use environment variables** for API keys
2. **Implement rate limiting** when calling external APIs
3. **Cache research data** to avoid redundant API calls
4. **Monitor token usage** for AI providers to control costs
5. **Use TypeScript strictly** for type safety across the pipeline
6. **Test video compositions** locally before batch rendering

This skill enables comprehensive automation of content marketing workflows from research to video delivery.
