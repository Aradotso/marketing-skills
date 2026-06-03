---
name: marketing-pipeline-share-ai-content
description: Automated AI content pipeline for research, scriptwriting, and video generation using Claude/OpenAI and Remotion
triggers:
  - automate content creation pipeline
  - generate AI video content from research
  - crawl news and create marketing content
  - build automated content workflow
  - create videos from AI-written scripts
  - research to video content automation
  - setup AI marketing pipeline
  - generate multilingual content automatically
---

# Marketing Pipeline Share - AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

Marketing Pipeline Share is an end-to-end automated content creation system that transforms keywords into finished content and videos. The pipeline includes:

1. **Auto-research**: Crawls recent news from TechCrunch, a16z, Twitter/X, LinkedIn
2. **AI content generation**: Uses Claude 3 or OpenAI to create articles in multiple formats (toplist, POV, case study, how-to)
3. **Multilingual output**: Simultaneous English and Vietnamese content
4. **Video rendering**: Automatically generates videos and infographics using Remotion

Built with TypeScript, Next.js, and integrates with OpenAI, Anthropic (Claude), RapidAPI, and Remotion.

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

# Setup environment variables
cp .env.example .env.local
```

## Configuration

### Environment Variables

Create `.env.local` with the following required variables:

```bash
# AI Providers
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_url

# Next.js
NEXT_PUBLIC_API_URL=http://localhost:3000
```

### API Provider Selection

The system supports both OpenAI and Claude. Configure your preferred provider in the content generation settings.

## Key Components

### 1. Research Module

The research module crawls and analyzes recent content from multiple sources:

```typescript
// lib/research/crawler.ts
interface ResearchSource {
  name: string;
  url: string;
  selector?: string;
}

export async function crawlSources(
  keyword: string,
  sources: ResearchSource[]
): Promise<ResearchData[]> {
  const results = await Promise.all(
    sources.map(async (source) => {
      const response = await fetch(source.url, {
        headers: {
          'X-RapidAPI-Key': process.env.RAPIDAPI_KEY!,
        },
      });
      
      const data = await response.json();
      return parseSourceData(data, keyword);
    })
  );
  
  return results.flat();
}

export function extractInsights(data: ResearchData[]): Insight[] {
  // Extract key insights, statistics, and trends
  return data.map(item => ({
    title: item.title,
    summary: item.summary,
    source: item.source,
    publishedAt: item.publishedAt,
    relevanceScore: calculateRelevance(item, keyword),
  }));
}
```

### 2. AI Content Generation

Generate content using Claude or OpenAI with various formats:

```typescript
// lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

interface ContentConfig {
  provider: 'claude' | 'openai';
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'professional' | 'friendly' | 'humorous';
  insights: Insight[];
}

export async function generateContent(
  config: ContentConfig
): Promise<GeneratedContent> {
  const prompt = buildPrompt(config);
  
  if (config.provider === 'claude') {
    const anthropic = new Anthropic({
      apiKey: process.env.ANTHROPIC_API_KEY,
    });
    
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
    
    return parseClaudeResponse(message);
  } else {
    const openai = new OpenAI({
      apiKey: process.env.OPENAI_API_KEY,
    });
    
    const completion = await openai.chat.completions.create({
      model: 'gpt-4-turbo-preview',
      messages: [
        {
          role: 'system',
          content: 'You are an expert content creator.',
        },
        {
          role: 'user',
          content: prompt,
        },
      ],
    });
    
    return parseOpenAIResponse(completion);
  }
}

function buildPrompt(config: ContentConfig): string {
  const formatInstructions = {
    'toplist': 'Create a numbered list article highlighting top items',
    'pov': 'Write from a specific point of view with personal insights',
    'case-study': 'Analyze a specific example with data and outcomes',
    'how-to': 'Provide step-by-step instructions',
  };
  
  return `
Create a ${config.format} article in ${config.language} with a ${config.tone} tone.

Research Insights:
${config.insights.map(i => `- ${i.title}: ${i.summary}`).join('\n')}

${formatInstructions[config.format]}

Include:
- Engaging headline
- Introduction
- Main content with data-backed insights
- Conclusion with actionable takeaways
`;
}
```

### 3. Multilingual Content

Generate simultaneous English and Vietnamese versions:

```typescript
// lib/ai/multilingual.ts
export async function generateMultilingualContent(
  config: Omit<ContentConfig, 'language'>
): Promise<{ en: GeneratedContent; vi: GeneratedContent }> {
  const [enContent, viContent] = await Promise.all([
    generateContent({ ...config, language: 'en' }),
    generateContent({ ...config, language: 'vi' }),
  ]);
  
  return { en: enContent, vi: viContent };
}
```

### 4. Video Generation with Remotion

Transform content into videos:

```typescript
// lib/video/remotion-renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';

interface VideoConfig {
  content: GeneratedContent;
  template: 'infographic' | 'talking-points' | 'short-form';
  platform: 'reels' | 'tiktok' | 'shorts';
}

export async function renderContentVideo(
  config: VideoConfig
): Promise<string> {
  const compositionId = getCompositionId(config.template);
  
  // Bundle Remotion project
  const bundleLocation = await bundle({
    entryPoint: './src/video/index.ts',
    webpackOverride: (config) => config,
  });
  
  // Get composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: {
      title: config.content.title,
      sections: config.content.sections,
      platform: config.platform,
    },
  });
  
  // Render video
  const outputLocation = `./output/${Date.now()}.mp4`;
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: composition.props,
  });
  
  return outputLocation;
}

function getCompositionId(template: string): string {
  const dimensions = {
    reels: { width: 1080, height: 1920 },
    tiktok: { width: 1080, height: 1920 },
    shorts: { width: 1080, height: 1920 },
  };
  
  return `${template}-video`;
}
```

### 5. Complete Pipeline

Execute the full pipeline from keyword to video:

```typescript
// lib/pipeline/executor.ts
export async function runContentPipeline(
  keyword: string,
  options: PipelineOptions
): Promise<PipelineResult> {
  // Step 1: Research
  const sources = getResearchSources(options.niche);
  const researchData = await crawlSources(keyword, sources);
  const insights = extractInsights(researchData);
  
  // Step 2: Generate Content
  const content = await generateMultilingualContent({
    provider: options.aiProvider || 'claude',
    format: options.format || 'toplist',
    tone: options.tone || 'professional',
    insights,
  });
  
  // Step 3: Generate Video (optional)
  let videos: { en?: string; vi?: string } = {};
  
  if (options.generateVideo) {
    videos.en = await renderContentVideo({
      content: content.en,
      template: options.videoTemplate || 'infographic',
      platform: options.platform || 'reels',
    });
    
    videos.vi = await renderContentVideo({
      content: content.vi,
      template: options.videoTemplate || 'infographic',
      platform: options.platform || 'reels',
    });
  }
  
  return {
    keyword,
    insights,
    content,
    videos,
    createdAt: new Date(),
  };
}
```

## API Routes

### Next.js API Endpoints

```typescript
// pages/api/pipeline/run.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { runContentPipeline } from '@/lib/pipeline/executor';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }
  
  try {
    const { keyword, options } = req.body;
    
    if (!keyword) {
      return res.status(400).json({ error: 'Keyword is required' });
    }
    
    const result = await runContentPipeline(keyword, options);
    
    return res.status(200).json(result);
  } catch (error) {
    console.error('Pipeline error:', error);
    return res.status(500).json({ 
      error: 'Pipeline execution failed',
      details: error.message,
    });
  }
}
```

### Research Endpoint

```typescript
// pages/api/research/crawl.ts
export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  const { keyword, sources } = req.body;
  
  const data = await crawlSources(keyword, sources);
  const insights = extractInsights(data);
  
  res.status(200).json({ insights });
}
```

### Content Generation Endpoint

```typescript
// pages/api/content/generate.ts
export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  const config = req.body as ContentConfig;
  
  const content = await generateContent(config);
  
  res.status(200).json({ content });
}
```

## Frontend Integration

### React Component Example

```typescript
// components/ContentPipeline.tsx
'use client';

import { useState } from 'react';

export default function ContentPipeline() {
  const [keyword, setKeyword] = useState('');
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState(null);
  
  const runPipeline = async () => {
    setLoading(true);
    
    try {
      const response = await fetch('/api/pipeline/run', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          keyword,
          options: {
            aiProvider: 'claude',
            format: 'toplist',
            tone: 'professional',
            generateVideo: true,
            platform: 'reels',
          },
        }),
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
    <div>
      <input
        type="text"
        value={keyword}
        onChange={(e) => setKeyword(e.target.value)}
        placeholder="Enter keyword..."
      />
      
      <button onClick={runPipeline} disabled={loading}>
        {loading ? 'Processing...' : 'Generate Content'}
      </button>
      
      {result && (
        <div>
          <h2>English Version</h2>
          <article>{result.content.en.body}</article>
          
          <h2>Vietnamese Version</h2>
          <article>{result.content.vi.body}</article>
          
          {result.videos.en && (
            <video src={result.videos.en} controls />
          )}
        </div>
      )}
    </div>
  );
}
```

## Common Patterns

### Custom Research Sources

```typescript
// config/research-sources.ts
export const customSources: ResearchSource[] = [
  {
    name: 'TechCrunch',
    url: 'https://techcrunch.com/wp-json/wp/v2/posts',
  },
  {
    name: 'ProductHunt',
    url: 'https://api.producthunt.com/v2/api/graphql',
  },
  {
    name: 'HackerNews',
    url: 'https://hacker-news.firebaseio.com/v0/topstories.json',
  },
];
```

### Content Format Templates

```typescript
// lib/templates/formats.ts
export const contentFormats = {
  toplist: {
    structure: ['intro', 'items', 'conclusion'],
    minItems: 5,
    maxItems: 10,
  },
  pov: {
    structure: ['hook', 'perspective', 'arguments', 'conclusion'],
    includePersonalStory: true,
  },
  caseStudy: {
    structure: ['background', 'challenge', 'solution', 'results'],
    requireData: true,
  },
  howTo: {
    structure: ['overview', 'steps', 'tips', 'conclusion'],
    stepByStep: true,
  },
};
```

### Scheduling Content Publication

```typescript
// lib/scheduler/publish.ts
export async function schedulePost(
  content: GeneratedContent,
  publishAt: Date,
  platforms: string[]
): Promise<void> {
  // Schedule to Facebook, LinkedIn, etc.
  for (const platform of platforms) {
    await queuePublish({
      platform,
      content,
      scheduledTime: publishAt,
    });
  }
}
```

## Troubleshooting

### API Rate Limits

If you encounter rate limits:

```typescript
// lib/utils/rate-limiter.ts
import pLimit from 'p-limit';

const limit = pLimit(5); // Max 5 concurrent requests

export async function crawlWithRateLimit(
  sources: ResearchSource[]
): Promise<ResearchData[]> {
  const promises = sources.map(source =>
    limit(() => crawlSingleSource(source))
  );
  
  return Promise.all(promises);
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
  timeoutInMilliseconds: 300000, // 5 minutes
});
```

### Memory Issues with Large Content

Use streaming for large datasets:

```typescript
import { pipeline } from 'stream/promises';

export async function processLargeResearch(
  keyword: string
): Promise<void> {
  const dataStream = createResearchStream(keyword);
  const processStream = createProcessingStream();
  const outputStream = createOutputStream();
  
  await pipeline(dataStream, processStream, outputStream);
}
```

### Claude/OpenAI Error Handling

```typescript
async function generateWithRetry(
  config: ContentConfig,
  maxRetries = 3
): Promise<GeneratedContent> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await generateContent(config);
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      
      // Exponential backoff
      await new Promise(r => setTimeout(r, Math.pow(2, i) * 1000));
    }
  }
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

# Run linting
npm run lint

# Type checking
npm run type-check

# Render video locally (Remotion)
npm run video:render

# Preview Remotion compositions
npm run video:preview
```

## Production Deployment

```bash
# Build Next.js app
npm run build

# Set production environment variables
export NODE_ENV=production
export OPENAI_API_KEY=your_key
export ANTHROPIC_API_KEY=your_key
export RAPIDAPI_KEY=your_key

# Start production server
npm start
```

For Vercel deployment, configure environment variables in the dashboard and deploy:

```bash
vercel --prod
```
