---
name: marketing-pipeline-share-ai-content
description: Automated AI content pipeline for research, scriptwriting, and video generation using Claude, OpenAI, and Remotion
triggers:
  - how do I automate content creation with AI
  - generate blog posts and videos automatically
  - set up AI content pipeline with Claude and OpenAI
  - create automated marketing content workflow
  - build AI-powered content generation system
  - use Remotion for automated video rendering
  - scrape and research content with AI automation
  - generate multilingual content with AI pipeline
---

# Marketing Pipeline Share - AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the **Ultimate AI Content Pipeline**, a comprehensive TypeScript-based system that automates the entire content creation workflow from research and scriptwriting to video generation. The pipeline integrates Claude 3, OpenAI, and Remotion to create a complete content factory.

## What This Project Does

The Marketing Pipeline Share automates:

1. **Auto-Scan Research**: Crawls and analyzes real-time data from sources like TechCrunch, a16z, Twitter/X, LinkedIn
2. **AI Content Generation**: Creates diverse content formats (toplist, POV, case studies, how-tos) in multiple languages
3. **Video Rendering**: Automatically generates infographics and short-form videos using Remotion
4. **Multi-platform Optimization**: Outputs content optimized for Reels, TikTok, Shorts, and other platforms

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
# AI Provider Keys
ANTHROPIC_API_KEY=your_claude_api_key_here
OPENAI_API_KEY=your_openai_api_key_here

# Research/Scraping APIs
RAPIDAPI_KEY=your_rapidapi_key_here

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Remotion Configuration
REMOTION_AWS_ACCESS_KEY_ID=your_aws_access_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret_key

# Next.js Configuration
NEXT_PUBLIC_API_URL=http://localhost:3000
```

## Development Setup

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Run Remotion preview (for video development)
npm run remotion:preview
```

## Core Architecture

### 1. Content Research Module

The research module automatically crawls and analyzes content:

```typescript
// lib/research/scanner.ts
import { Anthropic } from '@anthropic-ai/sdk';

interface ResearchConfig {
  keyword: string;
  sources: string[];
  timeframe: '24h' | '7d' | '30d';
  language: 'en' | 'vi';
}

export async function scanContent(config: ResearchConfig) {
  const anthropic = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY,
  });

  // Crawl data from sources
  const rawData = await Promise.all(
    config.sources.map(source => crawlSource(source, config.keyword))
  );

  // Analyze with Claude
  const analysis = await anthropic.messages.create({
    model: 'claude-3-opus-20240229',
    max_tokens: 4096,
    messages: [{
      role: 'user',
      content: `Analyze this content and extract key insights for "${config.keyword}": ${JSON.stringify(rawData)}`
    }]
  });

  return {
    insights: analysis.content,
    sources: rawData,
    timestamp: new Date().toISOString(),
  };
}

async function crawlSource(source: string, keyword: string) {
  // Implementation for crawling specific sources
  const response = await fetch(`https://api.rapidapi.com/search`, {
    headers: {
      'X-RapidAPI-Key': process.env.RAPIDAPI_KEY!,
    },
  });
  
  return response.json();
}
```

### 2. Content Generation Module

Generate content in multiple formats and languages:

```typescript
// lib/content/generator.ts
import OpenAI from 'openai';
import { Anthropic } from '@anthropic-ai/sdk';

type ContentFormat = 'toplist' | 'pov' | 'case-study' | 'how-to';
type ToneOfVoice = 'expert' | 'friendly' | 'humorous';

interface ContentConfig {
  topic: string;
  format: ContentFormat;
  tone: ToneOfVoice;
  language: 'en' | 'vi';
  researchData: any;
}

export async function generateContent(config: ContentConfig) {
  const anthropic = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY,
  });

  const systemPrompt = buildSystemPrompt(config.format, config.tone);
  const userPrompt = buildUserPrompt(config.topic, config.researchData, config.language);

  const message = await anthropic.messages.create({
    model: 'claude-3-sonnet-20240229',
    max_tokens: 8192,
    system: systemPrompt,
    messages: [{
      role: 'user',
      content: userPrompt,
    }],
  });

  return {
    content: message.content,
    format: config.format,
    language: config.language,
    metadata: {
      tone: config.tone,
      generatedAt: new Date().toISOString(),
    },
  };
}

function buildSystemPrompt(format: ContentFormat, tone: ToneOfVoice): string {
  const toneGuides = {
    expert: 'Write with authority and deep expertise. Use industry terminology appropriately.',
    friendly: 'Write in a conversational, approachable style. Be helpful and encouraging.',
    humorous: 'Inject wit and humor while maintaining professionalism.',
  };

  const formatGuides = {
    'toplist': 'Create a numbered list format with clear headers and explanations.',
    'pov': 'Write from a first-person perspective sharing unique insights.',
    'case-study': 'Structure as: Problem, Solution, Results with specific metrics.',
    'how-to': 'Provide step-by-step instructions with actionable details.',
  };

  return `You are a professional content writer. ${toneGuides[tone]} ${formatGuides[format]}`;
}

function buildUserPrompt(topic: string, researchData: any, language: string): string {
  const langInstruction = language === 'vi' ? 'Write in Vietnamese.' : 'Write in English.';
  
  return `
    Create content about: ${topic}
    
    Research insights to incorporate:
    ${JSON.stringify(researchData.insights)}
    
    ${langInstruction}
    
    Ensure the content is:
    - Data-backed with statistics from the research
    - Current and trend-focused
    - Engaging and actionable
  `;
}
```

### 3. Video Generation with Remotion

Automate video creation from content:

```typescript
// remotion/compositions/ContentVideo.tsx
import { AbsoluteFill, Sequence, useCurrentFrame, useVideoConfig } from 'remotion';
import { interpolate } from 'remotion';

interface ContentVideoProps {
  title: string;
  points: string[];
  brandColor: string;
}

export const ContentVideo: React.FC<ContentVideoProps> = ({
  title,
  points,
  brandColor,
}) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  const titleOpacity = interpolate(frame, [0, 30], [0, 1], {
    extrapolateRight: 'clamp',
  });

  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a1a' }}>
      <Sequence from={0} durationInFrames={90}>
        <AbsoluteFill style={{ 
          justifyContent: 'center', 
          alignItems: 'center',
          opacity: titleOpacity,
        }}>
          <h1 style={{ 
            color: brandColor, 
            fontSize: 80,
            textAlign: 'center',
            padding: '0 100px',
          }}>
            {title}
          </h1>
        </AbsoluteFill>
      </Sequence>

      {points.map((point, index) => (
        <Sequence
          key={index}
          from={90 + (index * 120)}
          durationInFrames={120}
        >
          <PointSlide point={point} index={index + 1} color={brandColor} />
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};

const PointSlide: React.FC<{ point: string; index: number; color: string }> = ({
  point,
  index,
  color,
}) => {
  const frame = useCurrentFrame();
  const opacity = interpolate(frame, [0, 20], [0, 1]);
  const scale = interpolate(frame, [0, 20], [0.8, 1]);

  return (
    <AbsoluteFill style={{
      justifyContent: 'center',
      alignItems: 'center',
      opacity,
      transform: `scale(${scale})`,
    }}>
      <div style={{ maxWidth: '80%', textAlign: 'center' }}>
        <div style={{ 
          color, 
          fontSize: 120, 
          fontWeight: 'bold',
          marginBottom: 40,
        }}>
          {index}
        </div>
        <p style={{ 
          color: 'white', 
          fontSize: 48,
          lineHeight: 1.5,
        }}>
          {point}
        </p>
      </div>
    </AbsoluteFill>
  );
};
```

### 4. Render Video API

```typescript
// lib/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface RenderConfig {
  title: string;
  points: string[];
  format: 'reels' | 'tiktok' | 'shorts';
  brandColor?: string;
}

const FORMATS = {
  reels: { width: 1080, height: 1920, fps: 30 },
  tiktok: { width: 1080, height: 1920, fps: 30 },
  shorts: { width: 1080, height: 1920, fps: 30 },
};

export async function renderContentVideo(config: RenderConfig) {
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: (currentConfiguration) => currentConfiguration,
  });

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title: config.title,
      points: config.points,
      brandColor: config.brandColor || '#3B82F6',
    },
  });

  const formatConfig = FORMATS[config.format];

  const outputPath = path.join(
    process.cwd(),
    'public',
    'videos',
    `${Date.now()}-${config.format}.mp4`
  );

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: {
      title: config.title,
      points: config.points,
      brandColor: config.brandColor || '#3B82F6',
    },
    ...formatConfig,
  });

  return {
    path: outputPath,
    url: `/videos/${path.basename(outputPath)}`,
    format: config.format,
  };
}
```

## API Routes (Next.js)

### Content Generation Endpoint

```typescript
// pages/api/generate-content.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { scanContent } from '@/lib/research/scanner';
import { generateContent } from '@/lib/content/generator';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  try {
    const { keyword, format, tone, language } = req.body;

    // Step 1: Research
    const researchData = await scanContent({
      keyword,
      sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
      timeframe: '24h',
      language,
    });

    // Step 2: Generate content
    const content = await generateContent({
      topic: keyword,
      format,
      tone,
      language,
      researchData,
    });

    res.status(200).json({
      success: true,
      content,
      research: researchData,
    });
  } catch (error) {
    console.error('Content generation error:', error);
    res.status(500).json({ 
      error: 'Failed to generate content',
      details: error instanceof Error ? error.message : 'Unknown error',
    });
  }
}
```

### Video Rendering Endpoint

```typescript
// pages/api/render-video.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { renderContentVideo } from '@/lib/video/renderer';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  try {
    const { title, points, format, brandColor } = req.body;

    const video = await renderContentVideo({
      title,
      points,
      format: format || 'reels',
      brandColor,
    });

    res.status(200).json({
      success: true,
      video,
    });
  } catch (error) {
    console.error('Video rendering error:', error);
    res.status(500).json({ 
      error: 'Failed to render video',
      details: error instanceof Error ? error.message : 'Unknown error',
    });
  }
}
```

## Complete Pipeline Usage

```typescript
// Example: Full pipeline execution
import { scanContent } from '@/lib/research/scanner';
import { generateContent } from '@/lib/content/generator';
import { renderContentVideo } from '@/lib/video/renderer';

async function runFullPipeline(keyword: string) {
  // 1. Research phase
  console.log('Starting research...');
  const research = await scanContent({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter'],
    timeframe: '24h',
    language: 'en',
  });

  // 2. Content generation (English)
  console.log('Generating English content...');
  const contentEN = await generateContent({
    topic: keyword,
    format: 'toplist',
    tone: 'expert',
    language: 'en',
    researchData: research,
  });

  // 3. Content generation (Vietnamese)
  console.log('Generating Vietnamese content...');
  const contentVI = await generateContent({
    topic: keyword,
    format: 'toplist',
    tone: 'friendly',
    language: 'vi',
    researchData: research,
  });

  // 4. Extract key points for video
  const points = extractTopPoints(contentEN.content, 5);

  // 5. Render videos for multiple platforms
  console.log('Rendering videos...');
  const videos = await Promise.all([
    renderContentVideo({
      title: keyword,
      points,
      format: 'reels',
      brandColor: '#E1306C',
    }),
    renderContentVideo({
      title: keyword,
      points,
      format: 'tiktok',
      brandColor: '#000000',
    }),
    renderContentVideo({
      title: keyword,
      points,
      format: 'shorts',
      brandColor: '#FF0000',
    }),
  ]);

  return {
    research,
    content: {
      en: contentEN,
      vi: contentVI,
    },
    videos,
  };
}

function extractTopPoints(content: any, count: number): string[] {
  // Parse content and extract main points
  // Implementation depends on content structure
  return [];
}
```

## Frontend Integration

```typescript
// components/ContentGenerator.tsx
'use client';

import { useState } from 'react';

export default function ContentGenerator() {
  const [keyword, setKeyword] = useState('');
  const [format, setFormat] = useState<'toplist' | 'pov' | 'case-study' | 'how-to'>('toplist');
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState<any>(null);

  async function handleGenerate() {
    setLoading(true);
    try {
      const response = await fetch('/api/generate-content', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          keyword,
          format,
          tone: 'expert',
          language: 'en',
        }),
      });

      const data = await response.json();
      setResult(data);

      // Optionally trigger video rendering
      if (data.success) {
        await renderVideo(data.content);
      }
    } catch (error) {
      console.error('Generation failed:', error);
    } finally {
      setLoading(false);
    }
  }

  async function renderVideo(content: any) {
    const points = content.content[0].text.split('\n').slice(0, 5);
    
    await fetch('/api/render-video', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        title: keyword,
        points,
        format: 'reels',
      }),
    });
  }

  return (
    <div className="max-w-4xl mx-auto p-6">
      <input
        type="text"
        value={keyword}
        onChange={(e) => setKeyword(e.target.value)}
        placeholder="Enter topic keyword..."
        className="w-full p-3 border rounded mb-4"
      />
      
      <select
        value={format}
        onChange={(e) => setFormat(e.target.value as any)}
        className="w-full p-3 border rounded mb-4"
      >
        <option value="toplist">Top List</option>
        <option value="pov">Point of View</option>
        <option value="case-study">Case Study</option>
        <option value="how-to">How-To Guide</option>
      </select>

      <button
        onClick={handleGenerate}
        disabled={loading || !keyword}
        className="w-full bg-blue-600 text-white p-3 rounded disabled:opacity-50"
      >
        {loading ? 'Generating...' : 'Generate Content & Video'}
      </button>

      {result && (
        <div className="mt-6 p-4 bg-gray-50 rounded">
          <h3 className="font-bold mb-2">Generated Content</h3>
          <pre className="whitespace-pre-wrap">{JSON.stringify(result, null, 2)}</pre>
        </div>
      )}
    </div>
  );
}
```

## Common Patterns

### Pattern 1: Batch Content Generation

```typescript
async function generateBatchContent(keywords: string[]) {
  const results = await Promise.allSettled(
    keywords.map(async (keyword) => {
      const research = await scanContent({
        keyword,
        sources: ['techcrunch'],
        timeframe: '24h',
        language: 'en',
      });

      return generateContent({
        topic: keyword,
        format: 'toplist',
        tone: 'expert',
        language: 'en',
        researchData: research,
      });
    })
  );

  return results.map((result, index) => ({
    keyword: keywords[index],
    status: result.status,
    data: result.status === 'fulfilled' ? result.value : null,
    error: result.status === 'rejected' ? result.reason : null,
  }));
}
```

### Pattern 2: Schedule Content Publishing

```typescript
import { scheduleJob } from 'node-schedule';

function scheduleContentGeneration(keywords: string[], cronExpression: string) {
  return scheduleJob(cronExpression, async () => {
    console.log('Running scheduled content generation...');
    
    for (const keyword of keywords) {
      try {
        const pipeline = await runFullPipeline(keyword);
        // Save to database or publish
        await saveContent(pipeline);
      } catch (error) {
        console.error(`Failed for ${keyword}:`, error);
      }
    }
  });
}

// Run daily at 9 AM
scheduleContentGeneration(['AI trends', 'Marketing automation'], '0 9 * * *');
```

## Troubleshooting

### API Rate Limits

```typescript
// lib/utils/rateLimiter.ts
class RateLimiter {
  private queue: Array<() => Promise<any>> = [];
  private processing = false;
  private delayMs: number;

  constructor(requestsPerMinute: number) {
    this.delayMs = 60000 / requestsPerMinute;
  }

  async enqueue<T>(fn: () => Promise<T>): Promise<T> {
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
      const fn = this.queue.shift();
      if (fn) {
        await fn();
        await new Promise(resolve => setTimeout(resolve, this.delayMs));
      }
    }

    this.processing = false;
  }
}

export const anthropicLimiter = new RateLimiter(50); // 50 requests per minute
export const openaiLimiter = new RateLimiter(60);
```

### Error Handling for Video Rendering

```typescript
async function renderWithRetry(config: RenderConfig, maxRetries = 3) {
  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      return await renderContentVideo(config);
    } catch (error) {
      console.error(`Render attempt ${attempt} failed:`, error);
      
      if (attempt === maxRetries) {
        throw new Error(`Failed after ${maxRetries} attempts: ${error}`);
      }
      
      // Exponential backoff
      await new Promise(resolve => setTimeout(resolve, Math.pow(2, attempt) * 1000));
    }
  }
}
```

### Memory Management for Large Batches

```typescript
async function processBatchInChunks<T>(
  items: T[],
  processor: (item: T) => Promise<any>,
  chunkSize = 5
) {
  const results = [];
  
  for (let i = 0; i < items.length; i += chunkSize) {
    const chunk = items.slice(i, i + chunkSize);
    const chunkResults = await Promise.all(chunk.map(processor));
    results.push(...chunkResults);
    
    // Allow garbage collection between chunks
    if (global.gc) {
      global.gc();
    }
  }
  
  return results;
}
```

This skill provides comprehensive guidance for working with the Marketing Pipeline Share project, enabling AI agents to help developers automate content creation workflows effectively.
