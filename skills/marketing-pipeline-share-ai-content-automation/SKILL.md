---
name: marketing-pipeline-share-ai-content-automation
description: Automated content creation pipeline with AI research, scriptwriting, multi-format output, and video generation using Claude/OpenAI and Remotion
triggers:
  - automate content creation with AI research and video generation
  - set up marketing pipeline with automatic article writing
  - generate social media content from trending topics automatically
  - build AI content automation with Claude and OpenAI
  - create automated video content from text using Remotion
  - implement content research and generation pipeline
  - automate multi-language content creation workflow
  - set up end-to-end marketing content automation
---

# Marketing Pipeline Share - AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This project is an all-in-one AI-powered content automation pipeline that researches trending topics, generates multi-format articles in multiple languages, and automatically creates video content. It integrates Claude 3, OpenAI, web scraping for real-time research, and Remotion for video rendering.

## What It Does

The Marketing Pipeline Share automates the entire content creation workflow:

1. **Auto-Research**: Crawls recent articles from TechCrunch, a16z, Twitter/X, LinkedIn (last 24h)
2. **AI Content Generation**: Creates articles in multiple formats (Top List, POV, Case Study, How-to) using Claude/OpenAI
3. **Multi-language Output**: Generates content in both English and Vietnamese with customizable tone
4. **Video Generation**: Automatically renders videos and infographics from written content using Remotion
5. **Platform Optimization**: Exports content optimized for Reels, TikTok, Shorts

## Installation

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
yarn install

# Set up environment variables
cp .env.example .env
```

## Configuration

Create a `.env` file in the root directory with the following variables:

```bash
# AI Services
OPENAI_API_KEY=your_openai_key_here
ANTHROPIC_API_KEY=your_claude_key_here

# Web Scraping (RapidAPI)
RAPIDAPI_KEY=your_rapidapi_key_here

# Database (if using)
DATABASE_URL=your_database_connection_string

# Next.js
NEXT_PUBLIC_APP_URL=http://localhost:3000

# Remotion (Video Rendering)
REMOTION_AWS_ACCESS_KEY_ID=your_aws_access_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret_key
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── services/         # Core services
│   │   ├── research/     # Web scraping & research
│   │   ├── ai/          # AI content generation
│   │   └── video/       # Video rendering
│   ├── lib/             # Utilities and helpers
│   └── types/           # TypeScript type definitions
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Key API Services

### 1. Research Service

The research service crawls and analyzes recent content from multiple sources.

```typescript
// src/services/research/scraper.ts
import axios from 'axios';

interface ResearchResult {
  title: string;
  url: string;
  summary: string;
  publishedAt: Date;
  source: string;
}

export async function researchTopic(
  keyword: string,
  sources: string[] = ['techcrunch', 'a16z', 'twitter']
): Promise<ResearchResult[]> {
  const results: ResearchResult[] = [];
  
  for (const source of sources) {
    try {
      const data = await scrapeSource(source, keyword);
      results.push(...data);
    } catch (error) {
      console.error(`Failed to scrape ${source}:`, error);
    }
  }
  
  return results.filter(
    (r) => new Date(r.publishedAt) > new Date(Date.now() - 24 * 60 * 60 * 1000)
  );
}

async function scrapeSource(
  source: string,
  keyword: string
): Promise<ResearchResult[]> {
  const response = await axios.get(
    `https://api.rapidapi.com/v1/${source}/search`,
    {
      params: { q: keyword, limit: 10 },
      headers: {
        'X-RapidAPI-Key': process.env.RAPIDAPI_KEY!,
        'X-RapidAPI-Host': `${source}-api.rapidapi.com`,
      },
    }
  );
  
  return response.data.results.map((item: any) => ({
    title: item.title,
    url: item.url,
    summary: item.description || item.content?.substring(0, 200),
    publishedAt: new Date(item.published_at),
    source: source,
  }));
}
```

### 2. AI Content Generation

Generate articles using Claude or OpenAI based on research data.

```typescript
// src/services/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

interface ContentRequest {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'professional' | 'friendly' | 'humorous';
  researchData: any[];
}

export async function generateContent(
  request: ContentRequest,
  provider: 'claude' | 'openai' = 'claude'
): Promise<string> {
  const prompt = buildPrompt(request);
  
  if (provider === 'claude') {
    const message = await anthropic.messages.create({
      model: 'claude-3-5-sonnet-20241022',
      max_tokens: 4096,
      messages: [{
        role: 'user',
        content: prompt,
      }],
    });
    
    return message.content[0].type === 'text' 
      ? message.content[0].text 
      : '';
  } else {
    const completion = await openai.chat.completions.create({
      model: 'gpt-4-turbo-preview',
      messages: [{
        role: 'user',
        content: prompt,
      }],
      max_tokens: 4096,
    });
    
    return completion.choices[0]?.message?.content || '';
  }
}

function buildPrompt(request: ContentRequest): string {
  const formatInstructions = {
    'toplist': 'Create a top 10 list format with numbered items',
    'pov': 'Write from a personal perspective with strong opinions',
    'case-study': 'Analyze as a detailed case study with data and insights',
    'how-to': 'Write as a step-by-step tutorial guide',
  };
  
  const toneInstructions = {
    'professional': 'Use formal, expert tone with industry terminology',
    'friendly': 'Use conversational, approachable language',
    'humorous': 'Include witty observations and light humor',
  };
  
  return `
You are an expert content writer specializing in marketing and technology.

Topic: ${request.keyword}
Format: ${formatInstructions[request.format]}
Language: ${request.language === 'en' ? 'English' : 'Vietnamese'}
Tone: ${toneInstructions[request.tone]}

Research Data:
${request.researchData.map((r, i) => `
${i + 1}. ${r.title}
   Source: ${r.source}
   Summary: ${r.summary}
   URL: ${r.url}
`).join('\n')}

Requirements:
- Use the research data to create an original, insightful article
- Include specific data points and examples from the research
- Make it engaging and actionable for the target audience
- Length: 1500-2000 words
- Include a compelling headline and subheadings
- Add a clear call-to-action at the end

Generate the complete article now:
`;
}
```

### 3. Multi-Language Content Generation

Generate content in both English and Vietnamese simultaneously.

```typescript
// src/services/ai/multi-lang-generator.ts
import { generateContent, ContentRequest } from './content-generator';

interface MultiLangContent {
  en: string;
  vi: string;
  metadata: {
    keyword: string;
    format: string;
    generatedAt: Date;
  };
}

export async function generateMultiLanguageContent(
  request: Omit<ContentRequest, 'language'>
): Promise<MultiLangContent> {
  const [enContent, viContent] = await Promise.all([
    generateContent({ ...request, language: 'en' }),
    generateContent({ ...request, language: 'vi' }),
  ]);
  
  return {
    en: enContent,
    vi: viContent,
    metadata: {
      keyword: request.keyword,
      format: request.format,
      generatedAt: new Date(),
    },
  };
}
```

### 4. Video Generation with Remotion

Render videos from generated content using Remotion.

```typescript
// src/services/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  content: string;
  format: 'reels' | 'tiktok' | 'shorts';
  aspectRatio: '9:16' | '16:9' | '1:1';
}

export async function renderContentVideo(
  config: VideoConfig,
  outputPath: string
): Promise<string> {
  const compositionId = getCompositionId(config.format);
  const bundleLocation = await bundle(
    path.join(process.cwd(), 'remotion/index.ts')
  );
  
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: {
      content: config.content,
      aspectRatio: config.aspectRatio,
    },
  });
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: {
      content: config.content,
      aspectRatio: config.aspectRatio,
    },
  });
  
  return outputPath;
}

function getCompositionId(format: string): string {
  const compositionMap = {
    'reels': 'InstagramReels',
    'tiktok': 'TikTokVideo',
    'shorts': 'YouTubeShorts',
  };
  return compositionMap[format] || 'InstagramReels';
}
```

### 5. Remotion Video Template

```typescript
// remotion/compositions/Reels.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig } from 'remotion';
import React from 'react';

interface ReelsProps {
  content: string;
  aspectRatio: '9:16' | '16:9' | '1:1';
}

export const InstagramReels: React.FC<ReelsProps> = ({ content, aspectRatio }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  
  const opacity = Math.min(1, frame / (fps / 2));
  const sections = content.split('\n\n').filter(Boolean);
  const currentSection = Math.floor(frame / (fps * 3)) % sections.length;
  
  return (
    <AbsoluteFill
      style={{
        backgroundColor: '#000',
        justifyContent: 'center',
        alignItems: 'center',
        padding: 40,
      }}
    >
      <div
        style={{
          fontSize: 48,
          color: '#fff',
          textAlign: 'center',
          opacity,
          lineHeight: 1.4,
          fontWeight: 'bold',
        }}
      >
        {sections[currentSection]}
      </div>
    </AbsoluteFill>
  );
};
```

## Complete Content Pipeline Workflow

```typescript
// src/services/pipeline/content-pipeline.ts
import { researchTopic } from '../research/scraper';
import { generateMultiLanguageContent } from '../ai/multi-lang-generator';
import { renderContentVideo } from '../video/renderer';

interface PipelineConfig {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  tone: 'professional' | 'friendly' | 'humorous';
  generateVideo: boolean;
  videoFormat?: 'reels' | 'tiktok' | 'shorts';
}

export async function runContentPipeline(config: PipelineConfig) {
  console.log(`Starting content pipeline for: ${config.keyword}`);
  
  // Step 1: Research
  console.log('Step 1: Researching topic...');
  const researchData = await researchTopic(config.keyword);
  console.log(`Found ${researchData.length} relevant articles`);
  
  // Step 2: Generate Content
  console.log('Step 2: Generating multi-language content...');
  const content = await generateMultiLanguageContent({
    keyword: config.keyword,
    format: config.format,
    tone: config.tone,
    researchData,
  });
  
  // Step 3: Generate Video (if requested)
  let videoPath: string | null = null;
  if (config.generateVideo && config.videoFormat) {
    console.log('Step 3: Rendering video...');
    videoPath = await renderContentVideo(
      {
        content: content.en,
        format: config.videoFormat,
        aspectRatio: '9:16',
      },
      `output/video-${Date.now()}.mp4`
    );
    console.log(`Video rendered: ${videoPath}`);
  }
  
  return {
    content,
    videoPath,
    researchSources: researchData.length,
    completedAt: new Date(),
  };
}
```

## API Routes (Next.js)

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/services/pipeline/content-pipeline';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    
    const result = await runContentPipeline({
      keyword: body.keyword,
      format: body.format || 'toplist',
      tone: body.tone || 'professional',
      generateVideo: body.generateVideo || false,
      videoFormat: body.videoFormat || 'reels',
    });
    
    return NextResponse.json({
      success: true,
      data: result,
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

## Frontend Component Example

```typescript
// src/components/ContentGenerator.tsx
'use client';

import { useState } from 'react';

export default function ContentGenerator() {
  const [keyword, setKeyword] = useState('');
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState<any>(null);
  
  async function handleGenerate() {
    setLoading(true);
    try {
      const response = await fetch('/api/generate', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          keyword,
          format: 'toplist',
          tone: 'professional',
          generateVideo: true,
          videoFormat: 'reels',
        }),
      });
      
      const data = await response.json();
      setResult(data.data);
    } catch (error) {
      console.error('Generation failed:', error);
    } finally {
      setLoading(false);
    }
  }
  
  return (
    <div className="p-8">
      <h1 className="text-3xl font-bold mb-6">AI Content Generator</h1>
      
      <input
        type="text"
        value={keyword}
        onChange={(e) => setKeyword(e.target.value)}
        placeholder="Enter topic keyword..."
        className="w-full p-4 border rounded-lg mb-4"
      />
      
      <button
        onClick={handleGenerate}
        disabled={loading || !keyword}
        className="bg-blue-600 text-white px-6 py-3 rounded-lg disabled:opacity-50"
      >
        {loading ? 'Generating...' : 'Generate Content'}
      </button>
      
      {result && (
        <div className="mt-8 space-y-4">
          <div>
            <h2 className="text-xl font-bold mb-2">English Content</h2>
            <div className="p-4 bg-gray-100 rounded-lg whitespace-pre-wrap">
              {result.content.en}
            </div>
          </div>
          
          <div>
            <h2 className="text-xl font-bold mb-2">Vietnamese Content</h2>
            <div className="p-4 bg-gray-100 rounded-lg whitespace-pre-wrap">
              {result.content.vi}
            </div>
          </div>
          
          {result.videoPath && (
            <div>
              <h2 className="text-xl font-bold mb-2">Generated Video</h2>
              <p className="text-gray-600">{result.videoPath}</p>
            </div>
          )}
        </div>
      )}
    </div>
  );
}
```

## Common Patterns

### Batch Content Generation

```typescript
// Generate multiple articles at once
async function batchGenerate(keywords: string[]) {
  const results = await Promise.all(
    keywords.map((keyword) =>
      runContentPipeline({
        keyword,
        format: 'toplist',
        tone: 'professional',
        generateVideo: false,
      })
    )
  );
  
  return results;
}
```

### Content Scheduling

```typescript
// Schedule content generation for later
import { scheduleJob } from 'node-schedule';

scheduleJob('0 9 * * *', async () => {
  // Run daily at 9 AM
  const trendingTopics = await getTrendingTopics();
  await batchGenerate(trendingTopics);
});
```

### Custom Video Templates

```typescript
// remotion/compositions/CustomTemplate.tsx
export const CustomTemplate: React.FC<{ title: string; points: string[] }> = ({
  title,
  points,
}) => {
  const frame = useCurrentFrame();
  
  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a1a' }}>
      <div style={{ padding: 60 }}>
        <h1 style={{ fontSize: 72, color: '#fff', marginBottom: 40 }}>
          {title}
        </h1>
        {points.map((point, i) => (
          <div
            key={i}
            style={{
              fontSize: 36,
              color: '#fff',
              opacity: frame > (i + 1) * 30 ? 1 : 0,
              marginBottom: 20,
            }}
          >
            {i + 1}. {point}
          </div>
        ))}
      </div>
    </AbsoluteFill>
  );
};
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement rate limiting and retry logic
import pRetry from 'p-retry';

async function generateWithRetry(request: ContentRequest) {
  return pRetry(
    () => generateContent(request),
    {
      retries: 3,
      onFailedAttempt: (error) => {
        console.log(
          `Attempt ${error.attemptNumber} failed. Retrying...`
        );
      },
    }
  );
}
```

### Memory Issues with Video Rendering

```typescript
// Reduce memory usage by processing videos sequentially
async function renderVideosSequentially(configs: VideoConfig[]) {
  const results = [];
  for (const config of configs) {
    const result = await renderContentVideo(
      config,
      `output/video-${Date.now()}.mp4`
    );
    results.push(result);
    // Allow garbage collection between renders
    await new Promise((resolve) => setTimeout(resolve, 1000));
  }
  return results;
}
```

### Research Data Quality

```typescript
// Filter and validate research results
function validateResearchData(results: ResearchResult[]): ResearchResult[] {
  return results.filter((r) => {
    const hasValidContent = r.summary && r.summary.length > 50;
    const isRecent = new Date(r.publishedAt) > new Date(Date.now() - 48 * 60 * 60 * 1000);
    const hasValidUrl = r.url && r.url.startsWith('http');
    
    return hasValidContent && isRecent && hasValidUrl;
  });
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

# Run type checking
npm run type-check

# Render a single video composition
npm run remotion render
```

This skill enables AI coding agents to effectively utilize the Marketing Pipeline Share project for automated content creation, from research through video generation, with full TypeScript integration and Next.js deployment.
