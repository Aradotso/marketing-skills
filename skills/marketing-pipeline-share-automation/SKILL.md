---
name: marketing-pipeline-share-automation
description: Vietnamese AI content automation pipeline for research, scriptwriting, and video generation using Claude/OpenAI and Remotion
triggers:
  - automate content creation with AI research
  - generate marketing videos from articles
  - create Vietnamese content with AI pipeline
  - auto-research and write blog posts
  - set up automated content workflow
  - build AI content generation system
  - create TikTok videos from research
  - automate social media content pipeline
---

# Marketing Pipeline Share Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to use the **Ultimate AI Content Pipeline** (marketing-pipeline-share), a Vietnamese-focused automated content system that handles research, scriptwriting, article generation, and video rendering. The pipeline crawls news sources, generates multi-format content in Vietnamese and English, and renders videos using Remotion.

## What This Project Does

The marketing-pipeline-share automates the entire content creation workflow:

1. **Auto-Research**: Crawls news from TechCrunch, a16z, Twitter, LinkedIn (last 24h)
2. **AI Content Generation**: Creates articles in multiple formats (Top Lists, POV, Case Studies, How-to) using Claude 3 or OpenAI
3. **Multi-language**: Generates Vietnamese and English versions simultaneously
4. **Video Rendering**: Converts articles to infographics and short videos using Remotion
5. **Platform Optimization**: Exports videos formatted for Reels, TikTok, Shorts

## Installation

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
yarn install

# Copy environment variables
cp .env.example .env
```

## Environment Configuration

Create a `.env` file with the following variables:

```bash
# AI Provider (choose one or both)
ANTHROPIC_API_KEY=your_claude_api_key_here
OPENAI_API_KEY=your_openai_api_key_here

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key_here
TWITTER_BEARER_TOKEN=your_twitter_api_token_here

# Database (if applicable)
DATABASE_URL=postgresql://user:pass@localhost:5432/content_db

# Next.js
NEXT_PUBLIC_API_URL=http://localhost:3000

# Remotion (for video rendering)
REMOTION_AWS_ACCESS_KEY_ID=your_aws_key_here
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret_here
```

## Key Commands

```bash
# Development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Render videos with Remotion
npm run render

# Run research crawler
npm run research

# Generate content from research
npm run generate
```

## Core Architecture

### 1. Research Module (Auto-Crawling)

```typescript
// lib/research/crawler.ts
import { fetchTechCrunchArticles, fetchTwitterTrends } from './sources';

interface ResearchResult {
  topic: string;
  sources: Array<{
    title: string;
    url: string;
    summary: string;
    publishedAt: Date;
  }>;
  insights: string[];
}

export async function conductResearch(
  keyword: string,
  timeframe: '24h' | '7d' = '24h'
): Promise<ResearchResult> {
  const sources = await Promise.all([
    fetchTechCrunchArticles(keyword, timeframe),
    fetchTwitterTrends(keyword),
    // Add more sources as needed
  ]);

  return {
    topic: keyword,
    sources: sources.flat(),
    insights: extractInsights(sources),
  };
}

// Example usage
const research = await conductResearch('AI marketing trends');
```

### 2. Content Generation with AI

```typescript
// lib/ai/generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

interface ContentConfig {
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  tone: 'expert' | 'friendly' | 'humorous';
  language: 'vi' | 'en' | 'both';
  research: ResearchResult;
}

export async function generateContent(config: ContentConfig) {
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

  return parseContentResponse(message.content);
}

function buildPrompt(config: ContentConfig): string {
  const { format, tone, language, research } = config;
  
  let basePrompt = `Based on this research data:\n`;
  research.sources.forEach((source) => {
    basePrompt += `- ${source.title}: ${source.summary}\n`;
  });

  basePrompt += `\n\nCreate a ${format} article with ${tone} tone in ${language}.`;
  
  if (language === 'both') {
    basePrompt += ' Provide both Vietnamese and English versions.';
  }

  return basePrompt;
}
```

### 3. OpenAI Alternative

```typescript
// lib/ai/openai-generator.ts
import OpenAI from 'openai';

export async function generateWithOpenAI(config: ContentConfig) {
  const openai = new OpenAI({
    apiKey: process.env.OPENAI_API_KEY,
  });

  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: 'You are an expert content writer specializing in Vietnamese marketing content.',
      },
      {
        role: 'user',
        content: buildPrompt(config),
      },
    ],
    temperature: 0.7,
  });

  return completion.choices[0].message.content;
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
  format: 'reel' | 'tiktok' | 'shorts';
}

export async function renderContentVideo(config: VideoConfig) {
  const bundleLocation = await bundle(
    path.resolve('./src/video/index.ts')
  );

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title: config.title,
      slides: config.content,
    },
  });

  const dimensions = getFormatDimensions(config.format);

  await renderMedia({
    composition: {
      ...composition,
      width: dimensions.width,
      height: dimensions.height,
    },
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: `out/${config.title}-${config.format}.mp4`,
    inputProps: composition.defaultProps,
  });
}

function getFormatDimensions(format: string) {
  const formats = {
    reel: { width: 1080, height: 1920 },
    tiktok: { width: 1080, height: 1920 },
    shorts: { width: 1080, height: 1920 },
  };
  return formats[format] || formats.reel;
}
```

### 5. Remotion Video Component

```typescript
// src/video/ContentVideo.tsx
import { AbsoluteFill, Sequence, useCurrentFrame, interpolate } from 'remotion';

interface ContentVideoProps {
  title: string;
  slides: string[];
}

export const ContentVideo: React.FC<ContentVideoProps> = ({ title, slides }) => {
  const frame = useCurrentFrame();

  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a2e' }}>
      <Sequence from={0} durationInFrames={60}>
        <TitleSlide title={title} frame={frame} />
      </Sequence>
      {slides.map((slide, index) => (
        <Sequence
          key={index}
          from={60 + index * 90}
          durationInFrames={90}
        >
          <ContentSlide content={slide} frame={frame} />
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};

const TitleSlide: React.FC<{ title: string; frame: number }> = ({
  title,
  frame,
}) => {
  const opacity = interpolate(frame, [0, 30], [0, 1]);
  
  return (
    <div style={{ opacity, padding: 60 }}>
      <h1 style={{ fontSize: 72, color: '#fff' }}>{title}</h1>
    </div>
  );
};
```

## Complete Pipeline Example

```typescript
// app/api/pipeline/route.ts
import { conductResearch } from '@/lib/research/crawler';
import { generateContent } from '@/lib/ai/generator';
import { renderContentVideo } from '@/lib/video/render';

export async function POST(request: Request) {
  const { keyword, format, tone, language, includeVideo } = await request.json();

  try {
    // Step 1: Research
    const research = await conductResearch(keyword);

    // Step 2: Generate content
    const content = await generateContent({
      format,
      tone,
      language,
      research,
    });

    // Step 3: Render video (optional)
    let videoUrl = null;
    if (includeVideo && content.keyPoints) {
      await renderContentVideo({
        title: content.title,
        content: content.keyPoints,
        format: 'reel',
      });
      videoUrl = `/videos/${content.title}-reel.mp4`;
    }

    return Response.json({
      success: true,
      data: {
        article: content,
        research: research.sources,
        videoUrl,
      },
    });
  } catch (error) {
    return Response.json(
      { success: false, error: error.message },
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

export default function ContentPipeline() {
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState(null);

  const runPipeline = async (e: React.FormEvent<HTMLFormElement>) => {
    e.preventDefault();
    setLoading(true);

    const formData = new FormData(e.currentTarget);
    const response = await fetch('/api/pipeline', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        keyword: formData.get('keyword'),
        format: formData.get('format'),
        tone: formData.get('tone'),
        language: formData.get('language'),
        includeVideo: formData.get('includeVideo') === 'on',
      }),
    });

    const data = await response.json();
    setResult(data);
    setLoading(false);
  };

  return (
    <div className="container mx-auto p-8">
      <h1 className="text-3xl font-bold mb-8">AI Content Pipeline</h1>
      
      <form onSubmit={runPipeline} className="space-y-4">
        <input
          name="keyword"
          placeholder="Enter keyword (e.g., AI marketing)"
          className="w-full p-2 border rounded"
          required
        />
        
        <select name="format" className="w-full p-2 border rounded">
          <option value="toplist">Top List</option>
          <option value="pov">POV</option>
          <option value="case-study">Case Study</option>
          <option value="how-to">How-to</option>
        </select>

        <select name="tone" className="w-full p-2 border rounded">
          <option value="expert">Expert</option>
          <option value="friendly">Friendly</option>
          <option value="humorous">Humorous</option>
        </select>

        <select name="language" className="w-full p-2 border rounded">
          <option value="vi">Vietnamese</option>
          <option value="en">English</option>
          <option value="both">Both</option>
        </select>

        <label className="flex items-center gap-2">
          <input type="checkbox" name="includeVideo" />
          Generate video
        </label>

        <button
          type="submit"
          disabled={loading}
          className="bg-blue-600 text-white px-6 py-2 rounded"
        >
          {loading ? 'Processing...' : 'Generate Content'}
        </button>
      </form>

      {result && (
        <div className="mt-8">
          <h2 className="text-2xl font-bold mb-4">{result.data.article.title}</h2>
          <div dangerouslySetInnerHTML={{ __html: result.data.article.html }} />
          {result.data.videoUrl && (
            <video src={result.data.videoUrl} controls className="mt-4 w-full" />
          )}
        </div>
      )}
    </div>
  );
}
```

## Common Patterns

### Research Data Caching

```typescript
// lib/cache/research.ts
import { Redis } from '@upstash/redis';

const redis = new Redis({
  url: process.env.UPSTASH_REDIS_URL,
  token: process.env.UPSTASH_REDIS_TOKEN,
});

export async function getCachedResearch(keyword: string) {
  const cached = await redis.get(`research:${keyword}`);
  if (cached) return cached;

  const research = await conductResearch(keyword);
  await redis.setex(`research:${keyword}`, 3600, JSON.stringify(research));
  
  return research;
}
```

### Batch Content Generation

```typescript
// lib/batch/generator.ts
export async function generateBatchContent(keywords: string[]) {
  const results = await Promise.allSettled(
    keywords.map(async (keyword) => {
      const research = await conductResearch(keyword);
      return generateContent({
        format: 'toplist',
        tone: 'expert',
        language: 'both',
        research,
      });
    })
  );

  return results
    .filter((r) => r.status === 'fulfilled')
    .map((r) => r.value);
}
```

## Troubleshooting

### API Rate Limits

```typescript
// lib/utils/rate-limiter.ts
import pLimit from 'p-limit';

const limit = pLimit(3); // Max 3 concurrent requests

export async function rateLimitedResearch(keywords: string[]) {
  return Promise.all(
    keywords.map((keyword) =>
      limit(() => conductResearch(keyword))
    )
  );
}
```

### Video Rendering Memory Issues

```typescript
// remotion.config.ts
import { Config } from '@remotion/cli/config';

Config.setConcurrency(1); // Reduce concurrency
Config.setChromiumDisableWebSecurity(true);
Config.setMaxTimelineTracks(15);
```

### Vietnamese Character Encoding

```typescript
// Ensure proper encoding in content generation
const content = Buffer.from(aiResponse, 'utf-8').toString();
```

### Claude API Timeout Handling

```typescript
import { withRetry } from '@/lib/utils/retry';

const content = await withRetry(
  () => generateContent(config),
  { maxAttempts: 3, delay: 1000 }
);
```

This skill enables AI agents to build and customize end-to-end content automation pipelines with research, AI writing, and video generation capabilities.
