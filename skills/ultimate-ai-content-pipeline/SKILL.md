---
name: ultimate-ai-content-pipeline
description: Automated content creation pipeline from research to video generation using Claude/OpenAI APIs
triggers:
  - how do I set up the AI content pipeline
  - generate automated content with research and video
  - create content using Claude and OpenAI APIs
  - automate content research and video generation
  - use the marketing pipeline for content creation
  - set up automated content workflow with AI
  - integrate Claude API for content generation
  - create videos from AI-generated content
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection

A comprehensive TypeScript-based content automation system that handles the entire content lifecycle: auto-research from news sources, AI-powered content generation in multiple formats and languages, and automatic video rendering using Remotion.

## What It Does

This pipeline automates:
- **Research**: Crawls recent content from TechCrunch, a16z, Twitter/X, LinkedIn (24h window)
- **Content Generation**: Creates articles in multiple formats (Toplist, POV, Case Study, How-to) using Claude 3 or OpenAI
- **Multi-language**: Generates both English and Vietnamese versions simultaneously
- **Video Creation**: Auto-renders videos and infographics using Remotion
- **Multi-platform**: Optimizes output for Reels, TikTok, Shorts

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

## Configuration

Create a `.env.local` file in the root directory:

```env
# AI APIs
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research API (RapidAPI)
RAPIDAPI_KEY=your_rapidapi_key

# Optional: Custom endpoints
NEXT_PUBLIC_API_URL=http://localhost:3000
```

## Project Structure

```
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── research/     # Auto-research modules
│   │   ├── ai/           # AI content generation
│   │   └── video/        # Remotion video rendering
│   └── types/            # TypeScript types
├── public/               # Static assets
└── remotion/             # Remotion video templates
```

## Core Modules

### 1. Research Module

Auto-crawls and analyzes content from multiple sources:

```typescript
// src/lib/research/crawler.ts
import axios from 'axios';

interface ResearchResult {
  source: string;
  title: string;
  content: string;
  url: string;
  publishedAt: Date;
}

export async function crawlRecentNews(
  keyword: string,
  hours: number = 24
): Promise<ResearchResult[]> {
  const rapidApiKey = process.env.RAPIDAPI_KEY;
  
  const options = {
    method: 'GET',
    url: 'https://news-api.example.com/search',
    params: {
      q: keyword,
      time_range: `${hours}h`,
      sources: 'techcrunch,a16z,twitter,linkedin'
    },
    headers: {
      'X-RapidAPI-Key': rapidApiKey,
      'X-RapidAPI-Host': 'news-api.example.com'
    }
  };

  const response = await axios.request(options);
  return response.data.articles.map((article: any) => ({
    source: article.source.name,
    title: article.title,
    content: article.description,
    url: article.url,
    publishedAt: new Date(article.publishedAt)
  }));
}

export async function extractInsights(
  results: ResearchResult[]
): Promise<string> {
  const combinedContent = results
    .map(r => `${r.title}\n${r.content}`)
    .join('\n\n');
  
  return combinedContent;
}
```

### 2. AI Content Generation

Generate content using Claude or OpenAI:

```typescript
// src/lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

type ContentFormat = 'toplist' | 'pov' | 'case-study' | 'how-to';
type Language = 'en' | 'vi';

interface ContentRequest {
  keyword: string;
  format: ContentFormat;
  language: Language;
  researchData: string;
  tone?: 'expert' | 'friendly' | 'humorous';
}

export class ContentGenerator {
  private claude: Anthropic;
  private openai: OpenAI;

  constructor() {
    this.claude = new Anthropic({
      apiKey: process.env.ANTHROPIC_API_KEY,
    });
    this.openai = new OpenAI({
      apiKey: process.env.OPENAI_API_KEY,
    });
  }

  async generateWithClaude(request: ContentRequest): Promise<string> {
    const prompt = this.buildPrompt(request);
    
    const message = await this.claude.messages.create({
      model: 'claude-3-5-sonnet-20241022',
      max_tokens: 4096,
      messages: [
        {
          role: 'user',
          content: prompt
        }
      ]
    });

    return message.content[0].type === 'text' 
      ? message.content[0].text 
      : '';
  }

  async generateWithOpenAI(request: ContentRequest): Promise<string> {
    const prompt = this.buildPrompt(request);
    
    const completion = await this.openai.chat.completions.create({
      model: 'gpt-4-turbo-preview',
      messages: [
        {
          role: 'system',
          content: 'You are an expert content creator specializing in marketing content.'
        },
        {
          role: 'user',
          content: prompt
        }
      ],
      max_tokens: 4096
    });

    return completion.choices[0].message.content || '';
  }

  private buildPrompt(request: ContentRequest): string {
    const formatInstructions = {
      'toplist': 'Create a numbered list article with clear rankings',
      'pov': 'Write from a personal perspective sharing insights',
      'case-study': 'Analyze with data, examples, and outcomes',
      'how-to': 'Provide step-by-step instructions'
    };

    const toneMap = {
      'expert': 'professional and authoritative',
      'friendly': 'conversational and approachable',
      'humorous': 'engaging with light humor'
    };

    return `
Create a ${request.format} article about "${request.keyword}" in ${request.language === 'en' ? 'English' : 'Vietnamese'}.

Tone: ${toneMap[request.tone || 'expert']}
Format: ${formatInstructions[request.format]}

Research Data:
${request.researchData}

Requirements:
- Include data-backed insights from the research
- Use engaging headlines and subheadings
- Add relevant statistics and examples
- Optimize for SEO with natural keyword placement
- Length: 1500-2000 words
`;
  }
}
```

### 3. Bilingual Generation

Generate both English and Vietnamese simultaneously:

```typescript
// src/lib/ai/bilingual-generator.ts
import { ContentGenerator } from './content-generator';

export async function generateBilingualContent(
  keyword: string,
  format: ContentFormat,
  researchData: string
) {
  const generator = new ContentGenerator();

  const [englishContent, vietnameseContent] = await Promise.all([
    generator.generateWithClaude({
      keyword,
      format,
      language: 'en',
      researchData,
      tone: 'expert'
    }),
    generator.generateWithClaude({
      keyword,
      format,
      language: 'vi',
      researchData,
      tone: 'friendly'
    })
  ]);

  return {
    en: englishContent,
    vi: vietnameseContent
  };
}
```

### 4. Video Generation with Remotion

Render videos from generated content:

```typescript
// src/lib/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  title: string;
  content: string;
  format: 'reels' | 'tiktok' | 'shorts';
}

export async function renderContentVideo(config: VideoConfig) {
  const compositionId = 'ContentVideo';
  const bundleLocation = await bundle(
    path.join(process.cwd(), 'remotion', 'index.ts')
  );

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: {
      title: config.title,
      content: config.content,
      aspectRatio: getAspectRatio(config.format)
    }
  });

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
    inputProps: composition.defaultProps,
  });

  return outputPath;
}

function getAspectRatio(format: string): [number, number] {
  const ratios = {
    'reels': [9, 16],
    'tiktok': [9, 16],
    'shorts': [9, 16],
  };
  return ratios[format as keyof typeof ratios] || [16, 9];
}
```

### 5. Remotion Video Template

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig, interpolate } from 'remotion';
import React from 'react';

interface ContentVideoProps {
  title: string;
  content: string;
  aspectRatio: [number, number];
}

export const ContentVideo: React.FC<ContentVideoProps> = ({
  title,
  content,
  aspectRatio
}) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  const opacity = interpolate(frame, [0, 30], [0, 1], {
    extrapolateRight: 'clamp',
  });

  const scale = interpolate(frame, [0, 30], [0.8, 1], {
    extrapolateRight: 'clamp',
  });

  return (
    <AbsoluteFill
      style={{
        backgroundColor: '#1a1a2e',
        justifyContent: 'center',
        alignItems: 'center',
      }}
    >
      <div
        style={{
          opacity,
          transform: `scale(${scale})`,
          maxWidth: '80%',
          textAlign: 'center',
        }}
      >
        <h1
          style={{
            fontSize: 60,
            color: '#00d9ff',
            marginBottom: 40,
            fontWeight: 'bold',
          }}
        >
          {title}
        </h1>
        <p
          style={{
            fontSize: 24,
            color: '#ffffff',
            lineHeight: 1.6,
          }}
        >
          {content.substring(0, 200)}...
        </p>
      </div>
    </AbsoluteFill>
  );
};
```

## API Routes

### Create Content Pipeline

```typescript
// src/app/api/pipeline/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { crawlRecentNews, extractInsights } from '@/lib/research/crawler';
import { generateBilingualContent } from '@/lib/ai/bilingual-generator';
import { renderContentVideo } from '@/lib/video/renderer';

export async function POST(request: NextRequest) {
  try {
    const { keyword, format } = await request.json();

    // Step 1: Research
    const newsResults = await crawlRecentNews(keyword, 24);
    const researchData = await extractInsights(newsResults);

    // Step 2: Generate Content
    const content = await generateBilingualContent(
      keyword,
      format,
      researchData
    );

    // Step 3: Render Video
    const videoPath = await renderContentVideo({
      title: keyword,
      content: content.en,
      format: 'reels'
    });

    return NextResponse.json({
      success: true,
      data: {
        content,
        videoPath,
        sources: newsResults.map(r => r.url)
      }
    });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: (error as Error).message },
      { status: 500 }
    );
  }
}
```

## Usage Example

```typescript
// src/app/page.tsx
'use client';

import { useState } from 'react';

export default function Home() {
  const [keyword, setKeyword] = useState('');
  const [format, setFormat] = useState<ContentFormat>('toplist');
  const [result, setResult] = useState<any>(null);
  const [loading, setLoading] = useState(false);

  const handleGenerate = async () => {
    setLoading(true);
    try {
      const response = await fetch('/api/pipeline', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ keyword, format })
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
    <div className="container mx-auto p-8">
      <h1 className="text-4xl font-bold mb-8">AI Content Pipeline</h1>
      
      <div className="space-y-4">
        <input
          type="text"
          value={keyword}
          onChange={(e) => setKeyword(e.target.value)}
          placeholder="Enter keyword..."
          className="w-full p-3 border rounded"
        />
        
        <select
          value={format}
          onChange={(e) => setFormat(e.target.value as ContentFormat)}
          className="w-full p-3 border rounded"
        >
          <option value="toplist">Top List</option>
          <option value="pov">POV Article</option>
          <option value="case-study">Case Study</option>
          <option value="how-to">How-To Guide</option>
        </select>
        
        <button
          onClick={handleGenerate}
          disabled={loading}
          className="w-full p-3 bg-blue-600 text-white rounded hover:bg-blue-700"
        >
          {loading ? 'Generating...' : 'Generate Content'}
        </button>
      </div>

      {result && (
        <div className="mt-8 space-y-4">
          <div className="p-4 border rounded">
            <h2 className="text-2xl font-bold mb-2">English Version</h2>
            <div dangerouslySetInnerHTML={{ __html: result.data.content.en }} />
          </div>
          
          <div className="p-4 border rounded">
            <h2 className="text-2xl font-bold mb-2">Vietnamese Version</h2>
            <div dangerouslySetInnerHTML={{ __html: result.data.content.vi }} />
          </div>
          
          <div className="p-4 border rounded">
            <h2 className="text-2xl font-bold mb-2">Generated Video</h2>
            <video controls src={result.data.videoPath} className="w-full" />
          </div>
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

# Render videos (Remotion)
npm run remotion:render
```

## Common Patterns

### Scheduled Content Generation

```typescript
// src/lib/scheduler.ts
import cron from 'node-cron';
import { crawlRecentNews, extractInsights } from './research/crawler';
import { generateBilingualContent } from './ai/bilingual-generator';

export function scheduleContentGeneration() {
  // Run every day at 9 AM
  cron.schedule('0 9 * * *', async () => {
    const keywords = ['AI trends', 'Marketing automation', 'Content creation'];
    
    for (const keyword of keywords) {
      const news = await crawlRecentNews(keyword, 24);
      const insights = await extractInsights(news);
      const content = await generateBilingualContent(keyword, 'toplist', insights);
      
      // Save to database or publish
      console.log(`Generated content for: ${keyword}`);
    }
  });
}
```

### Custom Video Templates

```typescript
// remotion/templates/InfographicVideo.tsx
import { AbsoluteFill, Sequence } from 'remotion';
import React from 'react';

export const InfographicVideo: React.FC<{
  stats: Array<{ label: string; value: string }>;
}> = ({ stats }) => {
  return (
    <AbsoluteFill>
      {stats.map((stat, i) => (
        <Sequence key={i} from={i * 60} durationInFrames={60}>
          <AbsoluteFill style={{ 
            justifyContent: 'center', 
            alignItems: 'center' 
          }}>
            <h2 style={{ fontSize: 48 }}>{stat.label}</h2>
            <p style={{ fontSize: 72, fontWeight: 'bold' }}>{stat.value}</p>
          </AbsoluteFill>
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};
```

## Troubleshooting

### API Rate Limits

```typescript
// src/lib/utils/rate-limiter.ts
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

// Usage
const limiter = new RateLimiter(10); // 10 requests per minute
await limiter.add(() => generator.generateWithClaude(request));
```

### Video Rendering Timeout

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
  },
});
```

### Memory Issues with Large Content

```typescript
// Process content in chunks
function chunkContent(content: string, maxChunkSize: number = 2000): string[] {
  const chunks: string[] = [];
  for (let i = 0; i < content.length; i += maxChunkSize) {
    chunks.push(content.slice(i, i + maxChunkSize));
  }
  return chunks;
}

// Generate content in smaller pieces
const chunks = chunkContent(researchData);
const results = await Promise.all(
  chunks.map(chunk => generator.generateWithClaude({ ...request, researchData: chunk }))
);
const finalContent = results.join('\n\n');
```
