---
name: marketing-pipeline-ai-content-automation
description: Automate content creation from research to video generation using AI (Claude/OpenAI) and Remotion for Vietnamese and English marketing content
triggers:
  - how do I automate content creation with AI
  - generate marketing content automatically
  - create videos from blog posts with Remotion
  - scrape news and generate content with Claude
  - build an AI content pipeline
  - automate social media content generation
  - research and write blog posts automatically
  - create multilingual marketing content with AI
---

# Marketing Pipeline AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This project provides a complete AI-powered content automation pipeline that handles research, content generation, and video rendering. It crawls news sources (TechCrunch, a16z, X, LinkedIn), generates content in multiple formats and languages using Claude/OpenAI, and automatically renders videos using Remotion.

## Installation

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
pnpm install
```

## Environment Configuration

Create a `.env.local` file with the following variables:

```bash
# AI Services
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_url

# Next.js
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

## Key Features

### 1. Auto-Research Content Scraping

The system automatically crawls recent news from multiple sources:

```typescript
// lib/research/scraper.ts
interface ResearchSource {
  name: string;
  url: string;
  selector: string;
}

export async function scrapeNewsSource(
  keyword: string,
  timeframe: '24h' | '7d' = '24h'
): Promise<ArticleData[]> {
  const sources: ResearchSource[] = [
    { name: 'TechCrunch', url: 'https://techcrunch.com', selector: '.article' },
    { name: 'a16z', url: 'https://a16z.com/blog', selector: '.post' }
  ];

  const articles: ArticleData[] = [];
  
  for (const source of sources) {
    const data = await fetchAndParse(source, keyword, timeframe);
    articles.push(...data);
  }

  return articles;
}

// Usage
const researchData = await scrapeNewsSource('AI automation', '24h');
```

### 2. AI Content Generation

Generate content in multiple formats using Claude or OpenAI:

```typescript
// lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';

interface ContentConfig {
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  researchData: ArticleData[];
}

export async function generateContent(config: ContentConfig): Promise<string> {
  const anthropic = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY,
  });

  const prompt = buildPrompt(config);

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [
      {
        role: 'user',
        content: prompt,
      },
    ],
  });

  return message.content[0].type === 'text' ? message.content[0].text : '';
}

function buildPrompt(config: ContentConfig): string {
  const formatInstructions = {
    'toplist': 'Create a numbered list format with detailed explanations',
    'pov': 'Write from a unique perspective with strong opinion',
    'case-study': 'Analyze with real examples and data points',
    'how-to': 'Provide step-by-step actionable instructions',
  };

  return `
Format: ${config.format}
Language: ${config.language}
Tone: ${config.tone}
Instructions: ${formatInstructions[config.format]}

Research Data:
${config.researchData.map(d => `- ${d.title}: ${d.summary}`).join('\n')}

Generate comprehensive content based on this research.
`;
}
```

### 3. OpenAI Alternative

```typescript
// lib/ai/openai-generator.ts
import OpenAI from 'openai';

export async function generateWithOpenAI(
  config: ContentConfig
): Promise<string> {
  const openai = new OpenAI({
    apiKey: process.env.OPENAI_API_KEY,
  });

  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are a ${config.tone} content writer creating ${config.format} content in ${config.language}.`,
      },
      {
        role: 'user',
        content: buildPrompt(config),
      },
    ],
    temperature: 0.7,
  });

  return completion.choices[0].message.content || '';
}
```

### 4. Video Generation with Remotion

```typescript
// lib/video/remotion-renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  content: string;
  title: string;
  format: 'reels' | 'tiktok' | 'shorts';
}

export async function renderContentVideo(
  config: VideoConfig
): Promise<string> {
  const dimensions = {
    reels: { width: 1080, height: 1920 },
    tiktok: { width: 1080, height: 1920 },
    shorts: { width: 1080, height: 1920 },
  };

  // Bundle the Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.join(__dirname, '../../remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  // Get composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title: config.title,
      content: config.content,
      ...dimensions[config.format],
    },
  });

  // Render video
  const outputPath = path.join(__dirname, `../../output/${Date.now()}.mp4`);
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: {
      title: config.title,
      content: config.content,
    },
  });

  return outputPath;
}
```

### 5. Remotion Video Component

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, Sequence, useCurrentFrame, useVideoConfig } from 'remotion';
import React from 'react';

interface ContentVideoProps {
  title: string;
  content: string;
}

export const ContentVideo: React.FC<ContentVideoProps> = ({ title, content }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  const opacity = Math.min(1, frame / 30);
  const contentPoints = content.split('\n').filter(Boolean);

  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a2e' }}>
      <Sequence from={0} durationInFrames={60}>
        <AbsoluteFill style={{ 
          justifyContent: 'center', 
          alignItems: 'center',
          opacity 
        }}>
          <h1 style={{ 
            color: 'white', 
            fontSize: 60, 
            textAlign: 'center',
            padding: '0 40px'
          }}>
            {title}
          </h1>
        </AbsoluteFill>
      </Sequence>

      {contentPoints.map((point, index) => (
        <Sequence key={index} from={60 + index * 90} durationInFrames={90}>
          <AbsoluteFill style={{
            justifyContent: 'center',
            alignItems: 'center',
            padding: '0 60px',
          }}>
            <div style={{ 
              color: 'white', 
              fontSize: 40,
              lineHeight: 1.6 
            }}>
              {point}
            </div>
          </AbsoluteFill>
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};
```

## Complete Pipeline Example

```typescript
// app/api/generate-content/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { scrapeNewsSource } from '@/lib/research/scraper';
import { generateContent } from '@/lib/ai/content-generator';
import { renderContentVideo } from '@/lib/video/remotion-renderer';

export async function POST(request: NextRequest) {
  try {
    const { keyword, format, language, tone, generateVideo } = await request.json();

    // Step 1: Research
    console.log('Starting research...');
    const researchData = await scrapeNewsSource(keyword, '24h');

    // Step 2: Generate Content
    console.log('Generating content...');
    const content = await generateContent({
      format,
      language,
      tone,
      researchData,
    });

    let videoPath = null;

    // Step 3: Generate Video (optional)
    if (generateVideo) {
      console.log('Rendering video...');
      videoPath = await renderContentVideo({
        content,
        title: `${keyword} - ${format}`,
        format: 'reels',
      });
    }

    return NextResponse.json({
      success: true,
      content,
      videoPath,
      researchSources: researchData.length,
    });

  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { error: 'Content generation failed' },
      { status: 500 }
    );
  }
}
```

## Frontend Integration

```typescript
// app/page.tsx
'use client';

import { useState } from 'react';

export default function Home() {
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState<any>(null);

  const handleGenerate = async (e: React.FormEvent<HTMLFormElement>) => {
    e.preventDefault();
    setLoading(true);

    const formData = new FormData(e.currentTarget);
    
    const response = await fetch('/api/generate-content', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        keyword: formData.get('keyword'),
        format: formData.get('format'),
        language: formData.get('language'),
        tone: formData.get('tone'),
        generateVideo: formData.get('video') === 'on',
      }),
    });

    const data = await response.json();
    setResult(data);
    setLoading(false);
  };

  return (
    <div className="container mx-auto p-8">
      <h1 className="text-4xl font-bold mb-8">AI Content Pipeline</h1>
      
      <form onSubmit={handleGenerate} className="space-y-4">
        <input
          name="keyword"
          placeholder="Enter keyword (e.g., AI automation)"
          className="w-full p-3 border rounded"
          required
        />

        <select name="format" className="w-full p-3 border rounded">
          <option value="toplist">Top List</option>
          <option value="pov">POV/Opinion</option>
          <option value="case-study">Case Study</option>
          <option value="how-to">How-to Guide</option>
        </select>

        <select name="language" className="w-full p-3 border rounded">
          <option value="en">English</option>
          <option value="vi">Tiếng Việt</option>
        </select>

        <select name="tone" className="w-full p-3 border rounded">
          <option value="expert">Expert</option>
          <option value="friendly">Friendly</option>
          <option value="humorous">Humorous</option>
        </select>

        <label className="flex items-center">
          <input type="checkbox" name="video" className="mr-2" />
          Generate Video
        </label>

        <button
          type="submit"
          disabled={loading}
          className="w-full bg-blue-600 text-white p-3 rounded hover:bg-blue-700"
        >
          {loading ? 'Generating...' : 'Generate Content'}
        </button>
      </form>

      {result && (
        <div className="mt-8 p-6 bg-gray-50 rounded">
          <h2 className="text-2xl font-bold mb-4">Result</h2>
          <p className="mb-2">Research sources: {result.researchSources}</p>
          <div className="whitespace-pre-wrap">{result.content}</div>
          {result.videoPath && (
            <p className="mt-4 text-green-600">Video: {result.videoPath}</p>
          )}
        </div>
      )}
    </div>
  );
}
```

## Running the Application

```bash
# Development mode
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Render video only
npm run remotion:render
```

## Troubleshooting

### API Rate Limits
If you encounter rate limits, implement caching:

```typescript
// lib/cache/redis-cache.ts
import Redis from 'ioredis';

const redis = new Redis(process.env.REDIS_URL);

export async function getCachedContent(key: string): Promise<string | null> {
  return await redis.get(key);
}

export async function setCachedContent(
  key: string,
  content: string,
  ttl: number = 3600
): Promise<void> {
  await redis.setex(key, ttl, content);
}
```

### Remotion Rendering Issues
Ensure ffmpeg is installed:

```bash
# macOS
brew install ffmpeg

# Ubuntu/Debian
sudo apt-get install ffmpeg

# Windows (use Chocolatey)
choco install ffmpeg
```

### Memory Issues During Video Rendering
Increase Node.js memory limit:

```bash
NODE_OPTIONS=--max-old-space-size=4096 npm run dev
```

## Advanced Patterns

### Batch Content Generation

```typescript
// lib/batch/batch-generator.ts
export async function batchGenerateContent(
  keywords: string[],
  config: Omit<ContentConfig, 'researchData'>
): Promise<Map<string, string>> {
  const results = new Map<string, string>();

  for (const keyword of keywords) {
    const researchData = await scrapeNewsSource(keyword, '24h');
    const content = await generateContent({
      ...config,
      researchData,
    });
    results.set(keyword, content);
    
    // Rate limiting
    await new Promise(resolve => setTimeout(resolve, 2000));
  }

  return results;
}
```

### Schedule Auto-Publishing

```typescript
// lib/scheduler/auto-publish.ts
import cron from 'node-cron';

export function scheduleContentGeneration() {
  // Run every day at 9 AM
  cron.schedule('0 9 * * *', async () => {
    const keywords = ['AI trends', 'marketing automation', 'content creation'];
    
    for (const keyword of keywords) {
      const researchData = await scrapeNewsSource(keyword, '24h');
      const content = await generateContent({
        format: 'toplist',
        language: 'vi',
        tone: 'expert',
        researchData,
      });
      
      // Auto-publish to your platform
      await publishToSocialMedia(content);
    }
  });
}
```

This skill covers the complete AI content automation pipeline from research to video generation, enabling agents to help users build powerful marketing content systems.
