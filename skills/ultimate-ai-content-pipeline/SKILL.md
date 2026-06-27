---
name: ultimate-ai-content-pipeline
description: Automated content pipeline from research to video generation using Claude/OpenAI and Remotion
triggers:
  - how do I use the AI content pipeline
  - set up automated content generation workflow
  - create content from research to video automatically
  - configure the marketing pipeline system
  - generate social media content with AI
  - automate content research and video creation
  - use the content automation pipeline
  - set up AI-powered content workflow
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

A comprehensive TypeScript-based content automation system that handles the entire content lifecycle: from researching trending topics across news sources, generating articles in multiple formats and languages, to automatically rendering videos for social media platforms.

## What This Project Does

Ultimate AI Content Pipeline is an end-to-end content automation system that:

- **Auto-crawls research data** from sources like TechCrunch, a16z, Twitter/X, LinkedIn
- **Generates content** in multiple formats (Toplist, POV, Case Study, How-to) using Claude 3 or OpenAI
- **Supports bilingual output** (English & Vietnamese) with customizable tone
- **Renders videos automatically** using Remotion for Reels, TikTok, Shorts
- **Provides Next.js frontend** for easy content management and scheduling

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
# AI Model Configuration
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Content Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Video Rendering
REMOTION_LICENSE_KEY=your_remotion_license_key

# Next.js Configuration
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

## Running the Application

```bash
# Development mode
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Render video (Remotion)
npm run render
```

## Core API Structure

### Research Module

```typescript
// lib/research/crawler.ts
import axios from 'axios';

interface ResearchSource {
  name: string;
  url: string;
  selector?: string;
}

export async function fetchTrendingTopics(
  sources: ResearchSource[],
  timeframe: '24h' | '7d' = '24h'
): Promise<ResearchData[]> {
  const results: ResearchData[] = [];
  
  for (const source of sources) {
    try {
      const response = await axios.get(source.url, {
        headers: {
          'X-RapidAPI-Key': process.env.RAPIDAPI_KEY,
        },
      });
      
      results.push({
        source: source.name,
        data: response.data,
        timestamp: new Date(),
      });
    } catch (error) {
      console.error(`Failed to fetch from ${source.name}:`, error);
    }
  }
  
  return results;
}

export async function analyzeResearch(
  data: ResearchData[]
): Promise<InsightSummary> {
  // Extract insights, trending keywords, and data points
  const insights = data.map(item => extractInsights(item));
  
  return {
    topKeywords: aggregateKeywords(insights),
    trendingTopics: identifyTrends(insights),
    dataPoints: collectStats(insights),
  };
}
```

### Content Generation Module

```typescript
// lib/content/generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

type ContentFormat = 'toplist' | 'pov' | 'case-study' | 'how-to';
type Language = 'en' | 'vi';
type Tone = 'expert' | 'friendly' | 'humorous';

interface ContentConfig {
  format: ContentFormat;
  language: Language;
  tone: Tone;
  targetAudience: string;
}

export async function generateContent(
  research: InsightSummary,
  config: ContentConfig,
  provider: 'claude' | 'openai' = 'claude'
): Promise<GeneratedContent> {
  const prompt = buildPrompt(research, config);
  
  if (provider === 'claude') {
    const anthropic = new Anthropic({
      apiKey: process.env.ANTHROPIC_API_KEY,
    });
    
    const message = await anthropic.messages.create({
      model: 'claude-3-5-sonnet-20241022',
      max_tokens: 4096,
      messages: [{
        role: 'user',
        content: prompt,
      }],
    });
    
    return parseClaudeResponse(message);
  } else {
    const openai = new OpenAI({
      apiKey: process.env.OPENAI_API_KEY,
    });
    
    const completion = await openai.chat.completions.create({
      model: 'gpt-4-turbo-preview',
      messages: [{
        role: 'user',
        content: prompt,
      }],
      max_tokens: 4096,
    });
    
    return parseOpenAIResponse(completion);
  }
}

function buildPrompt(
  research: InsightSummary,
  config: ContentConfig
): string {
  const formatInstructions = {
    'toplist': 'Create a ranked list article with clear numbering and explanations',
    'pov': 'Write from a strong personal perspective with opinions and analysis',
    'case-study': 'Structure as a detailed case study with problem-solution-results',
    'how-to': 'Create a step-by-step guide with actionable instructions',
  };
  
  return `
You are an expert content creator. Generate a ${config.format} article in ${config.language} with a ${config.tone} tone.

Target Audience: ${config.targetAudience}

Research Data:
- Trending Topics: ${research.trendingTopics.join(', ')}
- Key Keywords: ${research.topKeywords.join(', ')}
- Data Points: ${JSON.stringify(research.dataPoints)}

Instructions:
${formatInstructions[config.format]}

Requirements:
- Include all relevant data points and statistics
- Maintain consistent tone throughout
- Structure with clear headings and sections
- Include actionable insights
- Optimize for readability
`;
}
```

### Bilingual Content Generation

```typescript
// lib/content/bilingual.ts
export async function generateBilingualContent(
  research: InsightSummary,
  config: Omit<ContentConfig, 'language'>
): Promise<{ en: GeneratedContent; vi: GeneratedContent }> {
  const [englishContent, vietnameseContent] = await Promise.all([
    generateContent(research, { ...config, language: 'en' }),
    generateContent(research, { ...config, language: 'vi' }),
  ]);
  
  return {
    en: englishContent,
    vi: vietnameseContent,
  };
}
```

### Video Rendering with Remotion

```typescript
// lib/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  content: GeneratedContent;
  platform: 'reels' | 'tiktok' | 'shorts';
  style: 'minimal' | 'dynamic' | 'professional';
}

export async function renderContentVideo(
  config: VideoConfig
): Promise<string> {
  const { platform, content, style } = config;
  
  // Platform-specific dimensions
  const dimensions = {
    reels: { width: 1080, height: 1920 },
    tiktok: { width: 1080, height: 1920 },
    shorts: { width: 1080, height: 1920 },
  };
  
  // Bundle Remotion composition
  const bundled = await bundle(
    path.join(process.cwd(), 'remotion/index.tsx')
  );
  
  const composition = await selectComposition({
    serveUrl: bundled,
    id: 'ContentVideo',
    inputProps: {
      content: content.body,
      title: content.title,
      style,
      ...dimensions[platform],
    },
  });
  
  const outputPath = path.join(
    process.cwd(),
    'output',
    `${content.id}-${platform}.mp4`
  );
  
  await renderMedia({
    composition,
    serveUrl: bundled,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: composition.inputProps,
  });
  
  return outputPath;
}
```

### Remotion Video Component

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig } from 'remotion';
import React from 'react';

interface ContentVideoProps {
  title: string;
  content: string;
  style: 'minimal' | 'dynamic' | 'professional';
}

export const ContentVideo: React.FC<ContentVideoProps> = ({
  title,
  content,
  style,
}) => {
  const frame = useCurrentFrame();
  const { width, height, fps } = useVideoConfig();
  
  const opacity = Math.min(1, frame / 30);
  const scale = 0.8 + Math.min(0.2, frame / 60);
  
  return (
    <AbsoluteFill
      style={{
        backgroundColor: style === 'minimal' ? '#fff' : '#000',
        justifyContent: 'center',
        alignItems: 'center',
      }}
    >
      <div
        style={{
          opacity,
          transform: `scale(${scale})`,
          padding: 60,
          maxWidth: width * 0.8,
        }}
      >
        <h1
          style={{
            fontSize: 72,
            fontWeight: 'bold',
            color: style === 'minimal' ? '#000' : '#fff',
            marginBottom: 40,
            textAlign: 'center',
          }}
        >
          {title}
        </h1>
        <p
          style={{
            fontSize: 36,
            lineHeight: 1.6,
            color: style === 'minimal' ? '#333' : '#ddd',
            textAlign: 'center',
          }}
        >
          {content.slice(0, 200)}...
        </p>
      </div>
    </AbsoluteFill>
  );
};
```

## Complete Pipeline Workflow

```typescript
// lib/pipeline/workflow.ts
export async function runContentPipeline(
  keyword: string,
  config: PipelineConfig
): Promise<PipelineResult> {
  console.log(`Starting pipeline for keyword: ${keyword}`);
  
  // Step 1: Research
  const sources = [
    { name: 'TechCrunch', url: 'https://api.techcrunch.com/...' },
    { name: 'a16z', url: 'https://api.a16z.com/...' },
  ];
  
  const researchData = await fetchTrendingTopics(sources);
  const insights = await analyzeResearch(researchData);
  
  // Step 2: Content Generation
  const contentConfig: ContentConfig = {
    format: config.format || 'toplist',
    language: config.language || 'en',
    tone: config.tone || 'expert',
    targetAudience: config.targetAudience || 'marketers',
  };
  
  let content: GeneratedContent;
  
  if (config.bilingual) {
    const bilingualContent = await generateBilingualContent(
      insights,
      contentConfig
    );
    content = bilingualContent[config.primaryLanguage || 'en'];
  } else {
    content = await generateContent(insights, contentConfig);
  }
  
  // Step 3: Video Rendering (if enabled)
  let videoPath: string | null = null;
  
  if (config.generateVideo) {
    videoPath = await renderContentVideo({
      content,
      platform: config.platform || 'reels',
      style: config.videoStyle || 'dynamic',
    });
  }
  
  return {
    content,
    videoPath,
    research: insights,
    timestamp: new Date(),
  };
}
```

## Next.js API Route Example

```typescript
// pages/api/generate.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { runContentPipeline } from '@/lib/pipeline/workflow';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }
  
  const { keyword, config } = req.body;
  
  if (!keyword) {
    return res.status(400).json({ error: 'Keyword is required' });
  }
  
  try {
    const result = await runContentPipeline(keyword, config);
    
    return res.status(200).json({
      success: true,
      data: result,
    });
  } catch (error) {
    console.error('Pipeline error:', error);
    return res.status(500).json({
      success: false,
      error: error.message,
    });
  }
}
```

## Frontend Usage Example

```typescript
// components/ContentGenerator.tsx
'use client';

import { useState } from 'react';

export function ContentGenerator() {
  const [keyword, setKeyword] = useState('');
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState(null);
  
  const handleGenerate = async () => {
    setLoading(true);
    
    try {
      const response = await fetch('/api/generate', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          keyword,
          config: {
            format: 'toplist',
            language: 'en',
            tone: 'expert',
            bilingual: true,
            generateVideo: true,
            platform: 'reels',
          },
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
    <div className="max-w-4xl mx-auto p-8">
      <input
        type="text"
        value={keyword}
        onChange={(e) => setKeyword(e.target.value)}
        placeholder="Enter keyword..."
        className="w-full p-4 border rounded"
      />
      <button
        onClick={handleGenerate}
        disabled={loading}
        className="mt-4 px-6 py-3 bg-blue-600 text-white rounded"
      >
        {loading ? 'Generating...' : 'Generate Content'}
      </button>
      
      {result && (
        <div className="mt-8">
          <h2 className="text-2xl font-bold">{result.data.content.title}</h2>
          <div className="mt-4 prose">
            {result.data.content.body}
          </div>
          {result.data.videoPath && (
            <video controls className="mt-8 w-full">
              <source src={result.data.videoPath} type="video/mp4" />
            </video>
          )}
        </div>
      )}
    </div>
  );
}
```

## Common Patterns

### Pattern 1: Scheduled Content Generation

```typescript
// lib/scheduler/cron.ts
import cron from 'node-cron';

export function setupContentScheduler(keywords: string[]) {
  // Run every day at 6 AM
  cron.schedule('0 6 * * *', async () => {
    for (const keyword of keywords) {
      await runContentPipeline(keyword, {
        format: 'toplist',
        language: 'en',
        bilingual: true,
        generateVideo: true,
      });
    }
  });
}
```

### Pattern 2: Batch Processing

```typescript
// lib/pipeline/batch.ts
export async function batchGenerateContent(
  keywords: string[],
  config: PipelineConfig
): Promise<PipelineResult[]> {
  const results = await Promise.allSettled(
    keywords.map(keyword => runContentPipeline(keyword, config))
  );
  
  return results
    .filter(r => r.status === 'fulfilled')
    .map(r => (r as PromiseFulfilledResult<PipelineResult>).value);
}
```

### Pattern 3: Content Variation

```typescript
// Generate multiple versions of the same content
export async function generateContentVariations(
  keyword: string,
  variations: Partial<ContentConfig>[]
): Promise<GeneratedContent[]> {
  const research = await fetchAndAnalyzeResearch(keyword);
  
  return Promise.all(
    variations.map(variation =>
      generateContent(research, {
        format: 'toplist',
        language: 'en',
        tone: 'expert',
        targetAudience: 'marketers',
        ...variation,
      })
    )
  );
}
```

## Troubleshooting

### API Rate Limits

```typescript
// lib/utils/rate-limiter.ts
import { RateLimiter } from 'limiter';

const limiter = new RateLimiter({
  tokensPerInterval: 10,
  interval: 'minute',
});

export async function rateLimitedRequest<T>(
  fn: () => Promise<T>
): Promise<T> {
  await limiter.removeTokens(1);
  return fn();
}
```

### Error Handling

```typescript
// Always wrap API calls in try-catch
try {
  const content = await generateContent(research, config);
} catch (error) {
  if (error.status === 429) {
    console.log('Rate limited, retrying in 60s...');
    await new Promise(resolve => setTimeout(resolve, 60000));
    return generateContent(research, config);
  }
  throw error;
}
```

### Video Rendering Timeouts

```typescript
// Increase timeout for large videos
await renderMedia({
  composition,
  serveUrl: bundled,
  codec: 'h264',
  outputLocation: outputPath,
  timeoutInMilliseconds: 300000, // 5 minutes
});
```

### Memory Management

```typescript
// For large batch operations, process in chunks
async function processBatch(items: string[], chunkSize = 5) {
  for (let i = 0; i < items.length; i += chunkSize) {
    const chunk = items.slice(i, i + chunkSize);
    await Promise.all(chunk.map(item => processItem(item)));
    // Allow garbage collection
    await new Promise(resolve => setTimeout(resolve, 1000));
  }
}
```
