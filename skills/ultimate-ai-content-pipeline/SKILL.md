---
name: ultimate-ai-content-pipeline
description: Automated content generation pipeline from research to video using Claude/OpenAI and Remotion
triggers:
  - automate content creation workflow
  - generate content from research to video
  - build AI content pipeline
  - create automated marketing content
  - setup content automation system
  - generate videos from written content
  - crawl news and generate articles
  - automate social media content creation
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the Ultimate AI Content Pipeline, a TypeScript-based system that automates the entire content creation workflow: from research and article generation to automated video rendering. It leverages Claude 3, OpenAI, web scraping, and Remotion for complete content automation.

## What This Project Does

Ultimate AI Content Pipeline is an all-in-one content automation system that:

- **Auto-scans and researches** trending topics from sources like TechCrunch, a16z, Twitter/X, and LinkedIn
- **Generates multi-format content** (toplist, POV, case studies, how-tos) in multiple languages using Claude/OpenAI
- **Renders videos automatically** using Remotion from written content
- **Optimizes for multiple platforms** (Reels, TikTok, Shorts)
- **Provides a Next.js dashboard** for managing the content pipeline

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
# AI Providers
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# Web Scraping (RapidAPI)
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_url

# Remotion (Video Rendering)
REMOTION_AWS_ACCESS_KEY_ID=your_aws_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret

# Next.js
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

## Key Architecture

The project is structured around a pipeline with these main stages:

1. **Research Stage** - Crawl and analyze news sources
2. **Content Generation Stage** - Use AI to create articles
3. **Video Rendering Stage** - Convert content to video using Remotion
4. **Publishing Stage** - Schedule and publish content

## Core API Usage Patterns

### 1. Research & Content Crawling

```typescript
// lib/research/crawler.ts
import axios from 'axios';

interface ResearchSource {
  url: string;
  type: 'techcrunch' | 'twitter' | 'linkedin' | 'a16z';
}

export async function crawlSource(source: ResearchSource): Promise<any> {
  const options = {
    method: 'GET',
    url: 'https://api.rapidapi.com/web-scraper',
    params: {
      url: source.url,
      extract: 'article'
    },
    headers: {
      'X-RapidAPI-Key': process.env.RAPIDAPI_KEY,
      'X-RapidAPI-Host': 'web-scraper.rapidapi.com'
    }
  };

  const response = await axios.request(options);
  return response.data;
}

export async function aggregateResearch(keyword: string): Promise<any[]> {
  const sources: ResearchSource[] = [
    { url: `https://techcrunch.com/search/${keyword}`, type: 'techcrunch' },
    { url: `https://a16z.com/?s=${keyword}`, type: 'a16z' }
  ];

  const results = await Promise.all(
    sources.map(source => crawlSource(source))
  );

  return results.filter(r => r !== null);
}
```

### 2. AI Content Generation with Claude

```typescript
// lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

interface ContentRequest {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  researchData: any[];
}

export async function generateContent(request: ContentRequest): Promise<string> {
  const prompt = buildPrompt(request);

  const message = await anthropic.messages.create({
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

function buildPrompt(request: ContentRequest): string {
  const formatInstructions = {
    'toplist': 'Create a numbered list article with detailed explanations',
    'pov': 'Write from a specific perspective or opinion piece',
    'case-study': 'Analyze a real-world example with data and insights',
    'how-to': 'Create a step-by-step tutorial guide'
  };

  return `
You are an expert content writer. Create a ${request.format} article about "${request.keyword}".

Language: ${request.language}
Tone: ${request.tone}
Format: ${formatInstructions[request.format]}

Research Data:
${JSON.stringify(request.researchData, null, 2)}

Requirements:
- Use data from the research to support your points
- Make it engaging and actionable
- Include relevant statistics and examples
- Optimize for SEO
- Add a compelling introduction and conclusion
`;
}
```

### 3. Content Generation with OpenAI (Alternative)

```typescript
// lib/ai/openai-generator.ts
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

export async function generateContentOpenAI(request: ContentRequest): Promise<string> {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: 'You are an expert content creator specializing in marketing and social media.'
      },
      {
        role: 'user',
        content: buildPrompt(request)
      }
    ],
    temperature: 0.7,
    max_tokens: 3000
  });

  return completion.choices[0].message.content || '';
}
```

### 4. Video Rendering with Remotion

```typescript
// lib/video/render.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  title: string;
  content: string;
  aspectRatio: '9:16' | '16:9' | '1:1';
  platform: 'reels' | 'tiktok' | 'shorts' | 'youtube';
}

export async function renderContentVideo(config: VideoConfig): Promise<string> {
  const compositionId = 'ContentVideo';
  const bundleLocation = await bundle(
    path.join(process.cwd(), 'remotion/index.ts')
  );

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: {
      title: config.title,
      content: config.content,
      aspectRatio: config.aspectRatio
    }
  });

  const outputLocation = path.join(
    process.cwd(), 
    'public/videos', 
    `${Date.now()}.mp4`
  );

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: {
      title: config.title,
      content: config.content,
      aspectRatio: config.aspectRatio
    }
  });

  return outputLocation;
}
```

### 5. Remotion Video Component

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig, interpolate } from 'remotion';
import React from 'react';

interface ContentVideoProps {
  title: string;
  content: string;
  aspectRatio: '9:16' | '16:9' | '1:1';
}

export const ContentVideo: React.FC<ContentVideoProps> = ({ 
  title, 
  content, 
  aspectRatio 
}) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  const opacity = interpolate(
    frame,
    [0, 30],
    [0, 1],
    { extrapolateRight: 'clamp' }
  );

  const scale = interpolate(
    frame,
    [0, 30],
    [0.8, 1],
    { extrapolateRight: 'clamp' }
  );

  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a1a' }}>
      <div
        style={{
          opacity,
          transform: `scale(${scale})`,
          padding: '60px',
          color: 'white',
          fontFamily: 'Arial, sans-serif'
        }}
      >
        <h1 style={{ fontSize: '48px', marginBottom: '30px' }}>
          {title}
        </h1>
        <p style={{ fontSize: '24px', lineHeight: '1.6' }}>
          {content.substring(0, 200)}...
        </p>
      </div>
    </AbsoluteFill>
  );
};
```

### 6. Complete Pipeline Orchestration

```typescript
// lib/pipeline/orchestrator.ts
import { aggregateResearch } from '../research/crawler';
import { generateContent } from '../ai/content-generator';
import { renderContentVideo } from '../video/render';

interface PipelineConfig {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  languages: ('en' | 'vi')[];
  platforms: ('reels' | 'tiktok' | 'shorts')[];
  tone: 'expert' | 'friendly' | 'humorous';
}

export async function runContentPipeline(config: PipelineConfig) {
  console.log(`🚀 Starting pipeline for: ${config.keyword}`);

  // Stage 1: Research
  console.log('📡 Stage 1: Gathering research data...');
  const researchData = await aggregateResearch(config.keyword);

  // Stage 2: Content Generation
  console.log('🧠 Stage 2: Generating content...');
  const contents = await Promise.all(
    config.languages.map(language =>
      generateContent({
        keyword: config.keyword,
        format: config.format,
        language,
        tone: config.tone,
        researchData
      })
    )
  );

  // Stage 3: Video Rendering
  console.log('🎬 Stage 3: Rendering videos...');
  const videos = await Promise.all(
    config.platforms.map((platform, index) =>
      renderContentVideo({
        title: config.keyword,
        content: contents[0], // Use primary language
        aspectRatio: platform === 'youtube' ? '16:9' : '9:16',
        platform
      })
    )
  );

  console.log('✅ Pipeline complete!');
  
  return {
    research: researchData,
    contents,
    videos
  };
}
```

## Next.js API Routes

### Generate Content Endpoint

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

  try {
    const { keyword, format, languages, platforms, tone } = req.body;

    const result = await runContentPipeline({
      keyword,
      format: format || 'toplist',
      languages: languages || ['en', 'vi'],
      platforms: platforms || ['reels', 'tiktok'],
      tone: tone || 'expert'
    });

    res.status(200).json(result);
  } catch (error) {
    console.error('Pipeline error:', error);
    res.status(500).json({ 
      error: 'Pipeline failed', 
      message: error instanceof Error ? error.message : 'Unknown error'
    });
  }
}
```

### Research Endpoint

```typescript
// pages/api/research.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { aggregateResearch } from '@/lib/research/crawler';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  try {
    const { keyword } = req.body;
    const data = await aggregateResearch(keyword);
    res.status(200).json({ data });
  } catch (error) {
    res.status(500).json({ error: 'Research failed' });
  }
}
```

## Frontend Usage Example

```typescript
// components/ContentGenerator.tsx
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
          languages: ['en', 'vi'],
          platforms: ['reels', 'tiktok'],
          tone: 'expert'
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
    <div className="p-6">
      <input
        type="text"
        value={keyword}
        onChange={(e) => setKeyword(e.target.value)}
        placeholder="Enter keyword..."
        className="border p-2 rounded w-full mb-4"
      />
      <button
        onClick={handleGenerate}
        disabled={loading}
        className="bg-blue-500 text-white px-6 py-2 rounded"
      >
        {loading ? 'Generating...' : 'Generate Content'}
      </button>

      {result && (
        <div className="mt-6">
          <h3 className="text-xl font-bold mb-2">Results:</h3>
          <p>Contents: {result.contents.length}</p>
          <p>Videos: {result.videos.length}</p>
        </div>
      )}
    </div>
  );
}
```

## Development Commands

```bash
# Run development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Render Remotion video locally
npx remotion render src/index.ts ContentVideo output.mp4

# Preview Remotion composition
npx remotion preview
```

## Common Patterns

### Pattern 1: Batch Content Generation

```typescript
async function batchGenerate(keywords: string[]) {
  const results = [];
  
  for (const keyword of keywords) {
    const result = await runContentPipeline({
      keyword,
      format: 'toplist',
      languages: ['en'],
      platforms: ['reels'],
      tone: 'expert'
    });
    
    results.push(result);
    
    // Rate limiting
    await new Promise(resolve => setTimeout(resolve, 2000));
  }
  
  return results;
}
```

### Pattern 2: Custom AI Prompt Templates

```typescript
const PROMPT_TEMPLATES = {
  viral: `Create viral content about {keyword} that:
- Hooks readers in first 3 seconds
- Uses emotional triggers
- Includes surprising statistics
- Ends with strong CTA`,
  
  educational: `Create educational content about {keyword} that:
- Explains complex concepts simply
- Uses real-world examples
- Provides actionable takeaways
- Includes visual descriptions`
};

function getPromptTemplate(type: keyof typeof PROMPT_TEMPLATES, keyword: string) {
  return PROMPT_TEMPLATES[type].replace('{keyword}', keyword);
}
```

## Troubleshooting

### API Rate Limits

If you encounter rate limits:

```typescript
// lib/utils/rate-limiter.ts
export async function withRateLimit<T>(
  fn: () => Promise<T>,
  delayMs: number = 1000
): Promise<T> {
  await new Promise(resolve => setTimeout(resolve, delayMs));
  return fn();
}
```

### Video Rendering Timeout

For long videos, increase timeout:

```typescript
await renderMedia({
  composition,
  serveUrl: bundleLocation,
  codec: 'h264',
  outputLocation,
  timeoutInMilliseconds: 120000, // 2 minutes
});
```

### Memory Issues with Large Research Data

Implement pagination:

```typescript
async function paginatedResearch(keyword: string, limit: number = 10) {
  const allResults = await aggregateResearch(keyword);
  return allResults.slice(0, limit);
}
```

### Claude/OpenAI Token Limits

Split content into chunks:

```typescript
function chunkContent(content: string, maxTokens: number = 3000): string[] {
  const words = content.split(' ');
  const chunks: string[] = [];
  let currentChunk: string[] = [];
  
  for (const word of words) {
    currentChunk.push(word);
    if (currentChunk.length >= maxTokens / 4) { // Rough estimate
      chunks.push(currentChunk.join(' '));
      currentChunk = [];
    }
  }
  
  if (currentChunk.length > 0) {
    chunks.push(currentChunk.join(' '));
  }
  
  return chunks;
}
```

This skill covers the complete workflow from research automation to video generation, enabling AI agents to effectively utilize this marketing automation pipeline.
