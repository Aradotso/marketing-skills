---
name: marketing-pipeline-ai-content-automation
description: Automate content creation from research to video generation using AI (Claude/OpenAI) and Remotion for multi-platform marketing content
triggers:
  - how do I automate content creation with AI
  - set up automated marketing content pipeline
  - generate blog posts and videos from keywords automatically
  - create AI-powered content workflow with Claude and OpenAI
  - automate research and scriptwriting for marketing content
  - build content automation system with Remotion video rendering
  - configure AI content pipeline for social media
  - set up automated news aggregation and content generation
---

# Marketing Pipeline AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to help developers use the **Ultimate AI Content Pipeline** - a TypeScript-based system that automates the entire content creation workflow from research and scriptwriting to automatic video generation and publishing.

## What This Project Does

The Marketing Pipeline automates:

1. **Auto-Research**: Crawls and analyzes real-time data from sources like TechCrunch, a16z, Twitter/X, LinkedIn
2. **AI Content Generation**: Creates multi-format content (toplist, POV, case study, how-to) in multiple languages using Claude/OpenAI
3. **Video Rendering**: Automatically generates infographics and short-form videos using Remotion
4. **Multi-Platform Optimization**: Exports content optimized for Reels, TikTok, Shorts, and blog platforms

Built with Next.js, TypeScript, Remotion, and integrated with OpenAI, Anthropic Claude, and RapidAPI.

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

```bash
# AI Provider API Keys
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# Research & Data APIs
RAPIDAPI_KEY=your_rapidapi_key
TWITTER_BEARER_TOKEN=your_twitter_bearer

# Content Sources (Optional)
TECHCRUNCH_API_KEY=your_tc_key
A16Z_RSS_FEED=https://a16z.com/feed/

# Video Rendering
REMOTION_LICENSE_KEY=your_remotion_key

# Database & Storage
DATABASE_URL=your_database_url
STORAGE_BUCKET=your_storage_bucket

# Application
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

## Running the Application

```bash
# Development mode
npm run dev

# Production build
npm run build
npm start

# Run Remotion renderer separately
npm run remotion:dev
```

## Core Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── research/    # Auto-research crawlers
│   │   ├── content/     # Content generation logic
│   │   └── video/       # Remotion video rendering
│   ├── hooks/           # React hooks
│   └── utils/           # Utilities
├── remotion/            # Remotion video templates
├── public/              # Static assets
└── prisma/              # Database schema
```

## Key API Patterns

### 1. Research Automation

```typescript
// src/lib/research/auto-scanner.ts
import { searchNews } from './news-crawler';
import { analyzeInsights } from '../ai/analyzer';

interface ResearchResult {
  sources: Source[];
  insights: Insight[];
  trending: string[];
}

export async function autoResearch(
  keyword: string,
  timeframe: '24h' | '7d' | '30d' = '24h'
): Promise<ResearchResult> {
  // Crawl multiple sources
  const [techcrunch, twitter, linkedin] = await Promise.all([
    searchNews('techcrunch', keyword, timeframe),
    searchNews('twitter', keyword, timeframe),
    searchNews('linkedin', keyword, timeframe),
  ]);

  const sources = [...techcrunch, ...twitter, ...linkedin];

  // AI-powered insight extraction
  const insights = await analyzeInsights(sources, {
    model: 'claude-3-sonnet',
    focus: ['trends', 'data-points', 'expert-opinions'],
  });

  return {
    sources,
    insights,
    trending: extractTrendingTopics(sources),
  };
}
```

### 2. AI Content Generation

```typescript
// src/lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

interface ContentConfig {
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  provider: 'claude' | 'openai';
}

export async function generateContent(
  keyword: string,
  research: ResearchResult,
  config: ContentConfig
): Promise<string> {
  const prompt = buildPrompt(keyword, research, config);

  if (config.provider === 'claude') {
    const response = await anthropic.messages.create({
      model: 'claude-3-5-sonnet-20241022',
      max_tokens: 4096,
      messages: [
        {
          role: 'user',
          content: prompt,
        },
      ],
    });

    return response.content[0].type === 'text' 
      ? response.content[0].text 
      : '';
  } else {
    const response = await openai.chat.completions.create({
      model: 'gpt-4-turbo-preview',
      messages: [
        {
          role: 'system',
          content: 'You are an expert content creator specializing in marketing.',
        },
        {
          role: 'user',
          content: prompt,
        },
      ],
      temperature: 0.7,
    });

    return response.choices[0].message.content || '';
  }
}

function buildPrompt(
  keyword: string,
  research: ResearchResult,
  config: ContentConfig
): string {
  const formatInstructions = {
    'toplist': 'Create a numbered list article with rankings and explanations',
    'pov': 'Write from a unique perspective with strong opinions',
    'case-study': 'Analyze real examples with data and outcomes',
    'how-to': 'Provide step-by-step actionable instructions',
  };

  return `
Topic: ${keyword}
Format: ${config.format}
Language: ${config.language}
Tone: ${config.tone}

Research Data:
${JSON.stringify(research.insights, null, 2)}

Instructions: ${formatInstructions[config.format]}

Include specific data points and trending information from the research.
Make it engaging, actionable, and optimized for social sharing.
`;
}
```

### 3. Video Generation with Remotion

```typescript
// src/lib/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  template: 'infographic' | 'short-form' | 'story';
  platform: 'reels' | 'tiktok' | 'youtube-shorts';
  content: {
    title: string;
    points: string[];
    images?: string[];
  };
}

export async function generateVideo(config: VideoConfig): Promise<string> {
  const compositionId = getCompositionId(config.template, config.platform);
  
  // Bundle Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  // Select composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: config.content,
  });

  // Render video
  const outputPath = path.join(
    process.cwd(),
    'public',
    'videos',
    `${Date.now()}.mp4`
  );

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: config.content,
  });

  return outputPath;
}

function getCompositionId(
  template: string,
  platform: string
): string {
  const aspectRatios = {
    'reels': '9:16',
    'tiktok': '9:16',
    'youtube-shorts': '9:16',
  };

  return `${template}-${aspectRatios[platform]}`;
}
```

### 4. Complete Pipeline Workflow

```typescript
// src/lib/pipeline/orchestrator.ts
import { autoResearch } from '../research/auto-scanner';
import { generateContent } from '../ai/content-generator';
import { generateVideo } from '../video/renderer';
import { publishToPlatform } from '../publish/publisher';

interface PipelineConfig {
  keyword: string;
  contentFormats: ('blog' | 'video' | 'infographic')[];
  platforms: ('facebook' | 'twitter' | 'linkedin' | 'tiktok')[];
  aiProvider: 'claude' | 'openai';
  language: 'en' | 'vi';
}

export async function runContentPipeline(
  config: PipelineConfig
): Promise<void> {
  console.log(`🚀 Starting pipeline for: ${config.keyword}`);

  // Step 1: Research
  console.log('📡 Researching...');
  const research = await autoResearch(config.keyword, '24h');

  // Step 2: Generate Content
  console.log('🧠 Generating content...');
  const content = await generateContent(config.keyword, research, {
    format: 'toplist',
    language: config.language,
    tone: 'expert',
    provider: config.aiProvider,
  });

  const outputs = [];

  // Step 3: Create Formats
  if (config.contentFormats.includes('blog')) {
    outputs.push({
      type: 'blog',
      content: content,
    });
  }

  if (config.contentFormats.includes('video')) {
    console.log('🎬 Rendering video...');
    const videoPath = await generateVideo({
      template: 'short-form',
      platform: 'reels',
      content: {
        title: config.keyword,
        points: extractKeyPoints(content),
      },
    });

    outputs.push({
      type: 'video',
      path: videoPath,
    });
  }

  // Step 4: Publish
  console.log('📤 Publishing...');
  for (const platform of config.platforms) {
    await publishToPlatform(platform, outputs);
  }

  console.log('✅ Pipeline completed!');
}

function extractKeyPoints(content: string): string[] {
  // Extract bullet points or key sentences
  const lines = content.split('\n');
  return lines
    .filter(line => line.match(/^[\d\-\*]/))
    .slice(0, 5)
    .map(line => line.replace(/^[\d\-\*\.]\s*/, ''));
}
```

## Usage Examples

### Basic Content Generation

```typescript
// pages/api/generate.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { runContentPipeline } from '@/lib/pipeline/orchestrator';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  const { keyword } = req.body;

  try {
    await runContentPipeline({
      keyword,
      contentFormats: ['blog', 'video'],
      platforms: ['facebook', 'tiktok'],
      aiProvider: 'claude',
      language: 'en',
    });

    res.status(200).json({ success: true });
  } catch (error) {
    console.error('Pipeline error:', error);
    res.status(500).json({ error: 'Pipeline failed' });
  }
}
```

### React Component for UI

```typescript
// src/components/ContentGenerator.tsx
'use client';

import { useState } from 'react';

export default function ContentGenerator() {
  const [keyword, setKeyword] = useState('');
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState<any>(null);

  const handleGenerate = async () => {
    setLoading(true);
    try {
      const response = await fetch('/api/generate', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ keyword }),
      });

      const data = await response.json();
      setResult(data);
    } catch (error) {
      console.error('Error:', error);
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="p-6 max-w-2xl mx-auto">
      <h1 className="text-3xl font-bold mb-6">AI Content Pipeline</h1>
      
      <div className="space-y-4">
        <input
          type="text"
          value={keyword}
          onChange={(e) => setKeyword(e.target.value)}
          placeholder="Enter keyword (e.g., AI Marketing Trends)"
          className="w-full p-3 border rounded-lg"
        />
        
        <button
          onClick={handleGenerate}
          disabled={loading || !keyword}
          className="w-full bg-blue-600 text-white p-3 rounded-lg disabled:opacity-50"
        >
          {loading ? 'Generating...' : 'Generate Content'}
        </button>

        {result && (
          <div className="mt-6 p-4 bg-green-50 rounded-lg">
            <h3 className="font-semibold">Success!</h3>
            <p>Content generated and published</p>
          </div>
        )}
      </div>
    </div>
  );
}
```

## Remotion Video Template Example

```typescript
// remotion/compositions/ShortForm.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig } from 'remotion';

interface ShortFormProps {
  title: string;
  points: string[];
}

export const ShortForm: React.FC<ShortFormProps> = ({ title, points }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  const opacity = Math.min(1, frame / 30);
  const currentPoint = Math.floor(frame / (fps * 2));

  return (
    <AbsoluteFill
      style={{
        backgroundColor: '#1a1a1a',
        justifyContent: 'center',
        alignItems: 'center',
        padding: 40,
      }}
    >
      <div style={{ opacity, textAlign: 'center', color: 'white' }}>
        <h1 style={{ fontSize: 48, marginBottom: 40 }}>{title}</h1>
        
        {points[currentPoint] && (
          <p style={{ fontSize: 32, lineHeight: 1.5 }}>
            {points[currentPoint]}
          </p>
        )}
      </div>
    </AbsoluteFill>
  );
};
```

## Common Patterns

### Rate Limiting AI Requests

```typescript
// src/lib/utils/rate-limiter.ts
export class RateLimiter {
  private queue: Array<() => Promise<any>> = [];
  private running = 0;
  private maxConcurrent: number;

  constructor(maxConcurrent = 3) {
    this.maxConcurrent = maxConcurrent;
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
    if (this.running >= this.maxConcurrent || this.queue.length === 0) {
      return;
    }

    this.running++;
    const fn = this.queue.shift();
    
    if (fn) {
      await fn();
      this.running--;
      this.process();
    }
  }
}

// Usage
const limiter = new RateLimiter(3);
const result = await limiter.add(() => generateContent(keyword, research, config));
```

### Caching Research Results

```typescript
// src/lib/utils/cache.ts
import { Redis } from '@upstash/redis';

const redis = new Redis({
  url: process.env.UPSTASH_REDIS_URL,
  token: process.env.UPSTASH_REDIS_TOKEN,
});

export async function cachedResearch(
  keyword: string,
  timeframe: string
): Promise<ResearchResult> {
  const cacheKey = `research:${keyword}:${timeframe}`;
  
  // Try cache first
  const cached = await redis.get(cacheKey);
  if (cached) {
    return cached as ResearchResult;
  }

  // Perform research
  const result = await autoResearch(keyword, timeframe as any);

  // Cache for 1 hour
  await redis.set(cacheKey, result, { ex: 3600 });

  return result;
}
```

## Troubleshooting

### API Key Issues

```typescript
// src/lib/utils/validate-env.ts
export function validateEnvironment() {
  const required = [
    'OPENAI_API_KEY',
    'ANTHROPIC_API_KEY',
    'RAPIDAPI_KEY',
  ];

  const missing = required.filter(key => !process.env[key]);

  if (missing.length > 0) {
    throw new Error(
      `Missing required environment variables: ${missing.join(', ')}\n` +
      'Please check your .env.local file'
    );
  }
}
```

### Video Rendering Failures

```typescript
// Check Remotion logs
export async function safeRenderVideo(config: VideoConfig): Promise<string | null> {
  try {
    return await generateVideo(config);
  } catch (error) {
    console.error('Video rendering failed:', error);
    
    // Fallback: create static image instead
    if (error.message.includes('composition not found')) {
      console.log('Falling back to static image generation');
      return await generateStaticImage(config);
    }
    
    return null;
  }
}
```

### Memory Issues with Large Content

```typescript
// Process in chunks for large content
export async function generateLargeContent(
  keywords: string[],
  config: ContentConfig
): Promise<string[]> {
  const results = [];
  const chunkSize = 5;

  for (let i = 0; i < keywords.length; i += chunkSize) {
    const chunk = keywords.slice(i, i + chunkSize);
    const chunkResults = await Promise.all(
      chunk.map(keyword => 
        generateContent(keyword, research, config)
      )
    );
    results.push(...chunkResults);
    
    // Allow garbage collection
    await new Promise(resolve => setTimeout(resolve, 1000));
  }

  return results;
}
```

## Performance Optimization

- Use caching for research results (1-hour TTL recommended)
- Implement rate limiting for AI API calls
- Process video rendering asynchronously with job queues
- Use streaming responses for real-time content generation
- Batch multiple content requests when possible
- Monitor API usage and implement cost controls

## Security Best Practices

- Never commit API keys to version control
- Use environment variables for all secrets
- Implement request validation and sanitization
- Set up rate limiting on API endpoints
- Use proper CORS configuration for production
- Encrypt sensitive data in database
