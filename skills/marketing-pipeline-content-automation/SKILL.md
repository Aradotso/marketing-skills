---
name: marketing-pipeline-content-automation
description: Automated AI content pipeline for research, scriptwriting, posting, and video generation using Claude/OpenAI and Remotion
triggers:
  - automate content creation with AI research and video generation
  - set up automated marketing content pipeline
  - generate video content from text using Remotion
  - create content from keyword to published post automatically
  - build AI-powered content workflow with Claude and OpenAI
  - scrape trending news and generate social media content
  - auto-generate multilingual marketing content
  - create content pipeline with research and video rendering
---

# Marketing Pipeline Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the Ultimate AI Content Pipeline - a comprehensive TypeScript-based system that automates the entire content creation workflow from keyword research through news scraping, AI content generation (using Claude/OpenAI), and automatic video rendering with Remotion.

## What This Project Does

The marketing-pipeline-share project is an end-to-end content automation system that:

- **Auto-scans and researches**: Crawls real-time data from TechCrunch, a16z, Twitter/X, LinkedIn for trending topics
- **AI content generation**: Creates articles in multiple formats (Toplist, POV, Case Study, How-to) using Claude 3 or OpenAI
- **Multilingual support**: Generates content simultaneously in English and Vietnamese with customizable tone
- **Video rendering**: Automatically converts text content into videos/infographics using Remotion
- **Multi-platform optimization**: Exports videos optimized for Reels, TikTok, Shorts

## Installation

### Prerequisites

```bash
# Node.js 18+ required
node --version

# Install dependencies
npm install
# or
pnpm install
# or
yarn install
```

### Environment Configuration

Create a `.env.local` file in the project root:

```bash
# AI Provider Keys
ANTHROPIC_API_KEY=your_claude_key_here
OPENAI_API_KEY=your_openai_key_here

# Research/Scraping APIs
RAPIDAPI_KEY=your_rapidapi_key_here

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Remotion License (for video rendering)
REMOTION_LICENSE_KEY=your_remotion_license

# Optional: Social Media Auto-posting
FACEBOOK_ACCESS_TOKEN=your_fb_token
LINKEDIN_ACCESS_TOKEN=your_linkedin_token
```

### Development Server

```bash
npm run dev
# or
pnpm dev
# or
yarn dev
```

Access the application at `http://localhost:3000`

## Key Architecture

### Project Structure

```
marketing-pipeline-share/
├── src/
│   ├── app/              # Next.js App Router pages
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI service integrations
│   │   ├── scraper/     # News scraping logic
│   │   ├── video/       # Remotion video rendering
│   │   └── utils/       # Helper functions
│   ├── types/           # TypeScript type definitions
│   └── config/          # Configuration files
├── remotion/            # Remotion video templates
├── public/              # Static assets
└── package.json
```

## Core Functionality

### 1. Content Research & Scraping

```typescript
// src/lib/scraper/newsScaper.ts
import { TrendingSources } from '@/types/research';

interface NewsArticle {
  title: string;
  url: string;
  source: string;
  publishedAt: Date;
  summary: string;
}

export async function scrapeNews(
  keyword: string,
  sources: TrendingSources[] = ['techcrunch', 'a16z', 'twitter']
): Promise<NewsArticle[]> {
  const rapidApiKey = process.env.RAPIDAPI_KEY;
  
  const articles: NewsArticle[] = [];
  
  for (const source of sources) {
    const response = await fetch(
      `https://api.rapidapi.com/news/${source}?q=${encodeURIComponent(keyword)}`,
      {
        headers: {
          'X-RapidAPI-Key': rapidApiKey!,
          'X-RapidAPI-Host': 'news-data.p.rapidapi.com'
        }
      }
    );
    
    const data = await response.json();
    articles.push(...data.articles);
  }
  
  return articles.filter(article => 
    isWithinLast24Hours(article.publishedAt)
  );
}

function isWithinLast24Hours(date: Date): boolean {
  const now = new Date();
  const hoursDiff = (now.getTime() - date.getTime()) / (1000 * 60 * 60);
  return hoursDiff <= 24;
}
```

### 2. AI Content Generation

```typescript
// src/lib/ai/contentGenerator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

type ContentFormat = 'toplist' | 'pov' | 'case-study' | 'how-to';
type Language = 'en' | 'vi';
type Tone = 'expert' | 'friendly' | 'humorous';

interface ContentConfig {
  keyword: string;
  format: ContentFormat;
  language: Language;
  tone: Tone;
  researchData: string; // Scraped news insights
}

// Using Claude
export async function generateWithClaude(config: ContentConfig): Promise<string> {
  const anthropic = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY,
  });

  const systemPrompt = buildSystemPrompt(config.format, config.tone);
  const userPrompt = buildUserPrompt(config.keyword, config.researchData, config.language);

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    system: systemPrompt,
    messages: [
      {
        role: 'user',
        content: userPrompt,
      },
    ],
  });

  return message.content[0].type === 'text' ? message.content[0].text : '';
}

// Using OpenAI
export async function generateWithOpenAI(config: ContentConfig): Promise<string> {
  const openai = new OpenAI({
    apiKey: process.env.OPENAI_API_KEY,
  });

  const systemPrompt = buildSystemPrompt(config.format, config.tone);
  const userPrompt = buildUserPrompt(config.keyword, config.researchData, config.language);

  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      { role: 'system', content: systemPrompt },
      { role: 'user', content: userPrompt },
    ],
    temperature: 0.7,
    max_tokens: 4096,
  });

  return completion.choices[0].message.content || '';
}

function buildSystemPrompt(format: ContentFormat, tone: Tone): string {
  const formatInstructions = {
    'toplist': 'Create a numbered list article with clear rankings and explanations.',
    'pov': 'Write from a unique perspective with personal insights and opinions.',
    'case-study': 'Present a detailed case study with data, analysis, and outcomes.',
    'how-to': 'Write a step-by-step tutorial with actionable instructions.',
  };

  const toneInstructions = {
    'expert': 'Use professional, authoritative language with industry terminology.',
    'friendly': 'Write in a conversational, approachable style.',
    'humorous': 'Add wit and light humor while maintaining informativeness.',
  };

  return `You are an expert content creator. ${formatInstructions[format]} ${toneInstructions[tone]} Use the provided research data to create data-backed, trending content.`;
}

function buildUserPrompt(keyword: string, researchData: string, language: Language): string {
  const langInstruction = language === 'vi' 
    ? 'Write in Vietnamese.' 
    : 'Write in English.';
  
  return `
Keyword: ${keyword}

Recent Research Data:
${researchData}

${langInstruction}

Create a comprehensive, engaging article that incorporates the latest insights from the research data. Include specific examples, statistics, and actionable takeaways.
  `.trim();
}
```

### 3. Video Generation with Remotion

```typescript
// src/lib/video/renderVideo.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  content: string;
  title: string;
  format: 'reels' | 'tiktok' | 'shorts'; // 9:16 aspect ratio
  duration?: number; // in seconds
}

export async function renderContentVideo(config: VideoConfig): Promise<string> {
  const { content, title, format, duration = 30 } = config;

  // Bundle Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  // Select composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title,
      content,
      format,
    },
  });

  // Output path
  const outputLocation = path.join(
    process.cwd(),
    'public',
    'videos',
    `${Date.now()}-${format}.mp4`
  );

  // Render video
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: {
      title,
      content,
      format,
    },
  });

  return outputLocation;
}
```

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, Sequence, useCurrentFrame, useVideoConfig } from 'remotion';
import React from 'react';

interface ContentVideoProps {
  title: string;
  content: string;
  format: 'reels' | 'tiktok' | 'shorts';
}

export const ContentVideo: React.FC<ContentVideoProps> = ({ title, content, format }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  const opacity = Math.min(1, frame / (fps * 0.5));

  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a1a' }}>
      <Sequence from={0} durationInFrames={fps * 2}>
        <AbsoluteFill
          style={{
            justifyContent: 'center',
            alignItems: 'center',
            opacity,
          }}
        >
          <h1 style={{ color: 'white', fontSize: 60, textAlign: 'center', padding: 40 }}>
            {title}
          </h1>
        </AbsoluteFill>
      </Sequence>
      
      <Sequence from={fps * 2} durationInFrames={fps * 8}>
        <AbsoluteFill style={{ padding: 60, justifyContent: 'center' }}>
          <p style={{ color: 'white', fontSize: 32, lineHeight: 1.6 }}>
            {content}
          </p>
        </AbsoluteFill>
      </Sequence>
    </AbsoluteFill>
  );
};
```

### 4. Complete Pipeline Workflow

```typescript
// src/lib/pipeline/contentPipeline.ts
import { scrapeNews } from '@/lib/scraper/newsScaper';
import { generateWithClaude } from '@/lib/ai/contentGenerator';
import { renderContentVideo } from '@/lib/video/renderVideo';

interface PipelineConfig {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  generateVideo: boolean;
  videoFormat?: 'reels' | 'tiktok' | 'shorts';
}

interface PipelineResult {
  content: string;
  videoPath?: string;
  researchSources: number;
}

export async function runContentPipeline(
  config: PipelineConfig
): Promise<PipelineResult> {
  // Step 1: Research & Scraping
  console.log('🔍 Researching trending news...');
  const articles = await scrapeNews(config.keyword);
  
  const researchData = articles
    .map(article => `- ${article.title} (${article.source}): ${article.summary}`)
    .join('\n');

  // Step 2: AI Content Generation
  console.log('✍️ Generating content with AI...');
  const content = await generateWithClaude({
    keyword: config.keyword,
    format: config.format,
    language: config.language,
    tone: config.tone,
    researchData,
  });

  const result: PipelineResult = {
    content,
    researchSources: articles.length,
  };

  // Step 3: Video Generation (optional)
  if (config.generateVideo && config.videoFormat) {
    console.log('🎬 Rendering video...');
    const videoPath = await renderContentVideo({
      content: content.substring(0, 500), // First 500 chars for video
      title: config.keyword,
      format: config.videoFormat,
    });
    
    result.videoPath = videoPath;
  }

  console.log('✅ Pipeline complete!');
  return result;
}
```

### 5. API Route Example (Next.js)

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline/contentPipeline';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    
    const {
      keyword,
      format = 'toplist',
      language = 'en',
      tone = 'expert',
      generateVideo = false,
      videoFormat = 'reels',
    } = body;

    if (!keyword) {
      return NextResponse.json(
        { error: 'Keyword is required' },
        { status: 400 }
      );
    }

    const result = await runContentPipeline({
      keyword,
      format,
      language,
      tone,
      generateVideo,
      videoFormat,
    });

    return NextResponse.json({
      success: true,
      data: result,
    });
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { error: 'Pipeline execution failed', details: error.message },
      { status: 500 }
    );
  }
}
```

### 6. Frontend Component Example

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
        body: JSON.stringify({
          keyword,
          format: 'toplist',
          language: 'en',
          tone: 'expert',
          generateVideo: true,
          videoFormat: 'reels',
        }),
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
    <div className="p-8">
      <h1 className="text-3xl font-bold mb-6">AI Content Pipeline</h1>
      
      <div className="space-y-4">
        <input
          type="text"
          value={keyword}
          onChange={(e) => setKeyword(e.target.value)}
          placeholder="Enter keyword (e.g., AI Marketing)"
          className="w-full p-3 border rounded"
        />
        
        <button
          onClick={handleGenerate}
          disabled={loading || !keyword}
          className="bg-blue-600 text-white px-6 py-3 rounded disabled:opacity-50"
        >
          {loading ? 'Generating...' : 'Generate Content'}
        </button>

        {result && (
          <div className="mt-6 p-6 bg-gray-100 rounded">
            <h2 className="text-xl font-semibold mb-4">Results</h2>
            <p className="mb-2">Research Sources: {result.data.researchSources}</p>
            <div className="bg-white p-4 rounded">
              <pre className="whitespace-pre-wrap">{result.data.content}</pre>
            </div>
            {result.data.videoPath && (
              <div className="mt-4">
                <video controls src={result.data.videoPath} className="w-full max-w-md" />
              </div>
            )}
          </div>
        )}
      </div>
    </div>
  );
}
```

## Common Patterns

### Batch Content Generation

```typescript
// Generate multiple pieces of content for different keywords
async function batchGenerate(keywords: string[]) {
  const results = await Promise.allSettled(
    keywords.map(keyword =>
      runContentPipeline({
        keyword,
        format: 'toplist',
        language: 'en',
        tone: 'expert',
        generateVideo: false,
      })
    )
  );

  return results
    .filter(r => r.status === 'fulfilled')
    .map((r: any) => r.value);
}
```

### Scheduled Content Generation

```typescript
// Using node-cron or similar scheduler
import cron from 'node-cron';

// Run every day at 9 AM
cron.schedule('0 9 * * *', async () => {
  const trendingKeywords = await getTrendingKeywords();
  await batchGenerate(trendingKeywords);
});
```

## Troubleshooting

### API Rate Limits

```typescript
// Add rate limiting and retry logic
async function fetchWithRetry(url: string, options: any, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      const response = await fetch(url, options);
      if (response.status === 429) {
        await new Promise(resolve => setTimeout(resolve, 2000 * (i + 1)));
        continue;
      }
      return response;
    } catch (error) {
      if (i === maxRetries - 1) throw error;
    }
  }
}
```

### Video Rendering Memory Issues

```typescript
// For large-scale video rendering, use queue system
import Bull from 'bull';

const videoQueue = new Bull('video-rendering', {
  redis: { host: '127.0.0.1', port: 6379 },
});

videoQueue.process(async (job) => {
  return await renderContentVideo(job.data);
});

// Add job to queue instead of direct rendering
await videoQueue.add({ content, title, format });
```

### Environment Variables Not Loading

Ensure `.env.local` is in the root directory and restart the dev server:

```bash
# Kill all node processes
pkill node

# Restart
npm run dev
```

## Build & Deploy

```bash
# Build for production
npm run build

# Start production server
npm start

# For video rendering in production, ensure ffmpeg is installed
apt-get install ffmpeg
```

This skill provides comprehensive coverage for automating marketing content creation workflows using AI research, generation, and video rendering capabilities.
