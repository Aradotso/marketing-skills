---
name: marketing-pipeline-automation
description: Automated AI content pipeline for research, scripting, publishing, and video generation using Claude/OpenAI and Remotion
triggers:
  - how do I automate content creation with this pipeline
  - set up the marketing automation system
  - generate content from research to video
  - configure the AI content pipeline
  - create automated social media content
  - use the content generation workflow
  - integrate Claude and OpenAI for content
  - render videos from content automatically
---

# Marketing Pipeline Automation Skill

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to help developers use the **Ultimate AI Content Pipeline** - an end-to-end automated content generation system that handles research, scripting, publishing, and video creation using AI (Claude 3, OpenAI) and Remotion for video rendering.

## What This Project Does

The Marketing Pipeline automates the entire content creation workflow:

1. **Auto-Research**: Crawls fresh data from TechCrunch, a16z, Twitter/X, LinkedIn (last 24h)
2. **AI Content Generation**: Creates multi-format content (toplists, POV, case studies, how-tos) in multiple languages
3. **Video Rendering**: Automatically generates infographics and short-form videos using Remotion
4. **Multi-Platform Output**: Exports content optimized for Reels, TikTok, Shorts

Built with Next.js and TypeScript, integrating OpenAI, Anthropic Claude, RapidAPI, and Remotion.

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

```bash
# OpenAI Configuration
OPENAI_API_KEY=your_openai_key_here
OPENAI_MODEL=gpt-4-turbo-preview

# Anthropic Claude Configuration
ANTHROPIC_API_KEY=your_anthropic_key_here
CLAUDE_MODEL=claude-3-opus-20240229

# RapidAPI for Research Scraping
RAPIDAPI_KEY=your_rapidapi_key_here

# Database (if applicable)
DATABASE_URL=your_database_url_here

# Remotion Configuration
REMOTION_AWS_ACCESS_KEY_ID=your_aws_access_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret_key

# Next.js Configuration
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

Reference environment variables in your code:

```typescript
const openaiKey = process.env.OPENAI_API_KEY;
const claudeKey = process.env.ANTHROPIC_API_KEY;
```

## Running the Application

```bash
# Development mode
npm run dev

# Production build
npm run build
npm start

# Render videos with Remotion
npm run remotion:render
```

## Core API Patterns

### 1. Research & Data Crawling

```typescript
// lib/research/crawler.ts
import { RapidAPIClient } from '@/lib/api/rapidapi';

interface ResearchSource {
  source: string;
  title: string;
  content: string;
  url: string;
  publishedAt: Date;
}

export async function crawlRecentNews(
  keyword: string,
  sources: string[] = ['techcrunch', 'a16z']
): Promise<ResearchSource[]> {
  const rapidAPI = new RapidAPIClient(process.env.RAPIDAPI_KEY!);
  
  const results: ResearchSource[] = [];
  
  for (const source of sources) {
    const data = await rapidAPI.fetchNews({
      source,
      keyword,
      timeframe: '24h'
    });
    
    results.push(...data.articles.map(article => ({
      source,
      title: article.title,
      content: article.content,
      url: article.url,
      publishedAt: new Date(article.published)
    })));
  }
  
  return results;
}
```

### 2. AI Content Generation with Claude

```typescript
// lib/ai/claude-generator.ts
import Anthropic from '@anthropic-ai/sdk';

interface ContentConfig {
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  targetAudience: string;
}

export async function generateContent(
  research: ResearchSource[],
  config: ContentConfig
): Promise<string> {
  const anthropic = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY
  });
  
  const prompt = buildPrompt(research, config);
  
  const message = await anthropic.messages.create({
    model: process.env.CLAUDE_MODEL || 'claude-3-opus-20240229',
    max_tokens: 4096,
    messages: [{
      role: 'user',
      content: prompt
    }]
  });
  
  return message.content[0].type === 'text' 
    ? message.content[0].text 
    : '';
}

function buildPrompt(research: ResearchSource[], config: ContentConfig): string {
  const researchContext = research.map(r => 
    `Source: ${r.source}\nTitle: ${r.title}\n${r.content}`
  ).join('\n\n---\n\n');
  
  return `
Based on the following research from the last 24 hours, create a ${config.format} article in ${config.language}.

Tone: ${config.tone}
Target Audience: ${config.targetAudience}

Research Data:
${researchContext}

Generate comprehensive, data-backed content that is trending and insightful.
`;
}
```

### 3. OpenAI Alternative Implementation

```typescript
// lib/ai/openai-generator.ts
import OpenAI from 'openai';

export async function generateContentWithOpenAI(
  research: ResearchSource[],
  config: ContentConfig
): Promise<string> {
  const openai = new OpenAI({
    apiKey: process.env.OPENAI_API_KEY
  });
  
  const systemPrompt = `You are an expert content creator specializing in ${config.format} articles.
Write in a ${config.tone} tone for ${config.targetAudience}.`;
  
  const userPrompt = buildPrompt(research, config);
  
  const completion = await openai.chat.completions.create({
    model: process.env.OPENAI_MODEL || 'gpt-4-turbo-preview',
    messages: [
      { role: 'system', content: systemPrompt },
      { role: 'user', content: userPrompt }
    ],
    temperature: 0.7,
    max_tokens: 4000
  });
  
  return completion.choices[0].message.content || '';
}
```

### 4. Video Generation with Remotion

```typescript
// lib/video/render.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  title: string;
  content: string[];
  format: 'reels' | 'tiktok' | 'shorts';
}

export async function renderContentVideo(
  config: VideoConfig,
  outputPath: string
): Promise<string> {
  const compositionId = 'ContentVideo';
  
  // Bundle Remotion project
  const bundleLocation = await bundle(
    path.join(process.cwd(), 'src/remotion/index.ts')
  );
  
  // Get composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: {
      title: config.title,
      slides: config.content,
      aspectRatio: getAspectRatio(config.format)
    }
  });
  
  // Render video
  const outputLocation = path.join(outputPath, `${Date.now()}.mp4`);
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    envVariables: {
      AWS_ACCESS_KEY_ID: process.env.REMOTION_AWS_ACCESS_KEY_ID!,
      AWS_SECRET_ACCESS_KEY: process.env.REMOTION_AWS_SECRET_ACCESS_KEY!
    }
  });
  
  return outputLocation;
}

function getAspectRatio(format: VideoConfig['format']): [number, number] {
  switch (format) {
    case 'reels':
    case 'tiktok':
    case 'shorts':
      return [9, 16]; // Vertical
    default:
      return [16, 9]; // Horizontal
  }
}
```

### 5. Full Pipeline Orchestration

```typescript
// lib/pipeline/orchestrator.ts
import { crawlRecentNews } from '@/lib/research/crawler';
import { generateContent } from '@/lib/ai/claude-generator';
import { renderContentVideo } from '@/lib/video/render';

interface PipelineConfig {
  keyword: string;
  sources: string[];
  contentFormat: 'toplist' | 'pov' | 'case-study' | 'how-to';
  languages: ('en' | 'vi')[];
  videoFormats: ('reels' | 'tiktok' | 'shorts')[];
  tone: 'expert' | 'friendly' | 'humorous';
  targetAudience: string;
}

export async function runContentPipeline(config: PipelineConfig) {
  console.log(`Starting pipeline for keyword: ${config.keyword}`);
  
  // Step 1: Research
  console.log('Step 1: Crawling research data...');
  const research = await crawlRecentNews(config.keyword, config.sources);
  
  const results = [];
  
  // Step 2: Generate content for each language
  for (const language of config.languages) {
    console.log(`Step 2: Generating ${language} content...`);
    
    const content = await generateContent(research, {
      format: config.contentFormat,
      language,
      tone: config.tone,
      targetAudience: config.targetAudience
    });
    
    // Parse content into slides
    const slides = parseContentToSlides(content);
    
    // Step 3: Render videos for each format
    for (const format of config.videoFormats) {
      console.log(`Step 3: Rendering ${format} video in ${language}...`);
      
      const videoPath = await renderContentVideo(
        {
          title: `${config.keyword} - ${config.contentFormat}`,
          content: slides,
          format
        },
        `./output/videos/${language}`
      );
      
      results.push({
        language,
        format,
        content,
        videoPath
      });
    }
  }
  
  return results;
}

function parseContentToSlides(content: string): string[] {
  // Split content into video-friendly slides
  const sections = content.split(/\n#{1,3}\s/);
  return sections
    .filter(s => s.trim().length > 0)
    .map(s => s.trim().substring(0, 200)); // Limit slide text
}
```

## Next.js API Route Example

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline/orchestrator';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    
    const {
      keyword,
      sources = ['techcrunch', 'a16z'],
      contentFormat = 'toplist',
      languages = ['en', 'vi'],
      videoFormats = ['reels', 'tiktok'],
      tone = 'expert',
      targetAudience = 'marketers and content creators'
    } = body;
    
    if (!keyword) {
      return NextResponse.json(
        { error: 'Keyword is required' },
        { status: 400 }
      );
    }
    
    const results = await runContentPipeline({
      keyword,
      sources,
      contentFormat,
      languages,
      videoFormats,
      tone,
      targetAudience
    });
    
    return NextResponse.json({
      success: true,
      results
    });
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { error: 'Pipeline execution failed' },
      { status: 500 }
    );
  }
}
```

## Frontend Usage Example

```typescript
// app/components/ContentGenerator.tsx
'use client';

import { useState } from 'react';

export function ContentGenerator() {
  const [keyword, setKeyword] = useState('');
  const [loading, setLoading] = useState(false);
  const [results, setResults] = useState<any[]>([]);
  
  const handleGenerate = async () => {
    setLoading(true);
    
    try {
      const response = await fetch('/api/generate', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          keyword,
          contentFormat: 'toplist',
          languages: ['en', 'vi'],
          videoFormats: ['reels'],
          tone: 'expert'
        })
      });
      
      const data = await response.json();
      setResults(data.results);
    } catch (error) {
      console.error('Generation failed:', error);
    } finally {
      setLoading(false);
    }
  };
  
  return (
    <div className="p-6">
      <input
        type="text"
        value={keyword}
        onChange={(e) => setKeyword(e.target.value)}
        placeholder="Enter keyword..."
        className="border p-2 rounded"
      />
      
      <button
        onClick={handleGenerate}
        disabled={loading}
        className="ml-2 bg-blue-500 text-white px-4 py-2 rounded"
      >
        {loading ? 'Generating...' : 'Generate Content'}
      </button>
      
      {results.map((result, i) => (
        <div key={i} className="mt-4 border p-4 rounded">
          <h3>{result.language} - {result.format}</h3>
          <p className="mt-2">{result.content.substring(0, 200)}...</p>
          <a href={result.videoPath} className="text-blue-500">
            Download Video
          </a>
        </div>
      ))}
    </div>
  );
}
```

## Common Workflows

### Simple Content Generation

```typescript
import { crawlRecentNews } from '@/lib/research/crawler';
import { generateContent } from '@/lib/ai/claude-generator';

const research = await crawlRecentNews('AI automation', ['techcrunch']);
const content = await generateContent(research, {
  format: 'toplist',
  language: 'en',
  tone: 'expert',
  targetAudience: 'marketers'
});

console.log(content);
```

### Batch Video Generation

```typescript
const keywords = ['AI marketing', 'content automation', 'video generation'];

for (const keyword of keywords) {
  await runContentPipeline({
    keyword,
    sources: ['techcrunch', 'a16z'],
    contentFormat: 'how-to',
    languages: ['en'],
    videoFormats: ['reels', 'tiktok'],
    tone: 'friendly',
    targetAudience: 'small business owners'
  });
}
```

## Troubleshooting

### API Rate Limits

If you encounter rate limits:

```typescript
// lib/utils/rate-limiter.ts
export async function withRetry<T>(
  fn: () => Promise<T>,
  maxRetries = 3,
  delay = 1000
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error: any) {
      if (error.status === 429 && i < maxRetries - 1) {
        await new Promise(resolve => setTimeout(resolve, delay * (i + 1)));
        continue;
      }
      throw error;
    }
  }
  throw new Error('Max retries exceeded');
}
```

### Missing Environment Variables

Always validate environment variables at startup:

```typescript
// lib/config/validate.ts
export function validateEnv() {
  const required = [
    'OPENAI_API_KEY',
    'ANTHROPIC_API_KEY',
    'RAPIDAPI_KEY'
  ];
  
  const missing = required.filter(key => !process.env[key]);
  
  if (missing.length > 0) {
    throw new Error(`Missing environment variables: ${missing.join(', ')}`);
  }
}
```

### Remotion Rendering Failures

Check AWS credentials and permissions:

```typescript
if (!process.env.REMOTION_AWS_ACCESS_KEY_ID) {
  console.warn('AWS credentials not configured - video rendering disabled');
}
```

## Best Practices

1. **Always use environment variables** for API keys and secrets
2. **Implement rate limiting** when working with external APIs
3. **Cache research data** to avoid redundant API calls
4. **Use TypeScript interfaces** for type safety across the pipeline
5. **Handle errors gracefully** at each pipeline stage
6. **Monitor API costs** especially for OpenAI/Claude usage
7. **Optimize video rendering** by batching operations
