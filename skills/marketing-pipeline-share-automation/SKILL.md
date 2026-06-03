---
name: marketing-pipeline-share-automation
description: AI-powered content automation pipeline from research to video generation using Claude, OpenAI, and Remotion
triggers:
  - automate content creation with AI research and video generation
  - set up marketing pipeline for automated content
  - use Claude and OpenAI for content automation
  - generate videos from written content automatically
  - crawl news sources and create AI content
  - build automated content workflow with Remotion
  - create multi-format content from research to video
  - set up AI content pipeline with auto-posting
---

# Marketing Pipeline Share Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This project is an end-to-end AI content automation system that handles research, scriptwriting, and video generation. It crawls real-time data from sources like TechCrunch, a16z, Twitter/X, and LinkedIn, then uses Claude 3 or OpenAI to generate multi-format content (articles, videos, infographics) in multiple languages.

## What It Does

- **Auto Research**: Crawls news sources and extracts insights from the last 24 hours
- **AI Content Generation**: Creates content in multiple formats (toplist, POV, case study, how-to)
- **Multi-language Support**: Generates content in English and Vietnamese
- **Video Rendering**: Automatically creates videos and infographics using Remotion
- **Platform Optimization**: Exports videos optimized for Reels, TikTok, Shorts

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
# AI APIs
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_url

# Optional: Social Media APIs for auto-posting
FACEBOOK_ACCESS_TOKEN=your_fb_token
TWITTER_API_KEY=your_twitter_key
```

## Key Commands

```bash
# Development mode
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Remotion video rendering
npm run render

# Type checking
npm run type-check
```

## Core API Usage

### 1. Research & Data Crawling

```typescript
import { crawlNewsData } from '@/lib/research/crawler';
import { analyzeInsights } from '@/lib/research/analyzer';

async function performResearch(keyword: string) {
  // Crawl data from multiple sources
  const rawData = await crawlNewsData({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeRange: '24h'
  });

  // Extract insights using AI
  const insights = await analyzeInsights(rawData, {
    model: 'claude-3-opus-20240229',
    extractStats: true,
    extractTrends: true
  });

  return insights;
}
```

### 2. Content Generation with Claude/OpenAI

```typescript
import { generateContent } from '@/lib/ai/content-generator';
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

async function createArticle(insights: any, format: string) {
  const content = await generateContent({
    client: anthropic,
    insights,
    format, // 'toplist' | 'pov' | 'case-study' | 'how-to'
    languages: ['en', 'vi'],
    tone: 'professional', // 'professional' | 'friendly' | 'humorous'
    model: 'claude-3-opus-20240229'
  });

  return content;
}

// Using OpenAI alternative
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function createWithOpenAI(prompt: string) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      { role: 'system', content: 'You are a content creation expert.' },
      { role: 'user', content: prompt }
    ],
    temperature: 0.7,
  });

  return completion.choices[0].message.content;
}
```

### 3. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

async function generateVideo(content: any) {
  // Bundle Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.join(process.cwd(), 'src/remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  // Select composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title: content.title,
      body: content.body,
      stats: content.stats,
      theme: 'modern'
    },
  });

  // Render video
  const outputPath = path.join(process.cwd(), 'output', 'video.mp4');
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: composition.defaultProps,
  });

  return outputPath;
}
```

### 4. Complete Pipeline Example

```typescript
import { performResearch } from '@/lib/research';
import { generateContent } from '@/lib/ai/content-generator';
import { generateVideo } from '@/lib/video/generator';
import { postToSocial } from '@/lib/social/publisher';

async function runContentPipeline(keyword: string) {
  try {
    // Step 1: Research
    console.log('🔍 Researching topic...');
    const insights = await performResearch(keyword);

    // Step 2: Generate content
    console.log('✍️ Generating content...');
    const content = await generateContent({
      insights,
      format: 'toplist',
      languages: ['en', 'vi'],
      tone: 'professional'
    });

    // Step 3: Create video
    console.log('🎬 Rendering video...');
    const videoPath = await generateVideo(content);

    // Step 4: Auto-post (optional)
    console.log('📤 Publishing...');
    await postToSocial({
      content: content.text,
      video: videoPath,
      platforms: ['facebook', 'twitter', 'tiktok']
    });

    console.log('✅ Pipeline completed!');
    return { content, videoPath };
  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}
```

## API Endpoints (Next.js)

### Research Endpoint

```typescript
// pages/api/research.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { performResearch } from '@/lib/research';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  const { keyword } = req.body;

  try {
    const insights = await performResearch(keyword);
    res.status(200).json({ success: true, data: insights });
  } catch (error) {
    res.status(500).json({ success: false, error: error.message });
  }
}
```

### Content Generation Endpoint

```typescript
// pages/api/generate-content.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { generateContent } from '@/lib/ai/content-generator';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  const { insights, format, languages, tone } = req.body;

  try {
    const content = await generateContent({
      insights,
      format,
      languages,
      tone
    });
    
    res.status(200).json({ success: true, content });
  } catch (error) {
    res.status(500).json({ success: false, error: error.message });
  }
}
```

## Common Patterns

### Custom Content Format

```typescript
interface ContentFormat {
  id: string;
  name: string;
  structure: string[];
  aiPromptTemplate: string;
}

const customFormat: ContentFormat = {
  id: 'expert-interview',
  name: 'Expert Interview',
  structure: ['introduction', 'q&a', 'key-takeaways', 'conclusion'],
  aiPromptTemplate: `
    Create an expert interview format article about {topic}.
    Include 5 insightful questions and detailed answers.
    Use data: {insights}
    Tone: {tone}
    Language: {language}
  `
};

async function generateCustomFormat(
  insights: any,
  format: ContentFormat,
  options: any
) {
  const prompt = format.aiPromptTemplate
    .replace('{topic}', options.topic)
    .replace('{insights}', JSON.stringify(insights))
    .replace('{tone}', options.tone)
    .replace('{language}', options.language);

  return await generateContent({ prompt, ...options });
}
```

### Video Template Customization

```typescript
// src/remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, interpolate } from 'remotion';

export const ContentVideo: React.FC<{
  title: string;
  stats: any[];
  theme: 'modern' | 'minimal' | 'bold';
}> = ({ title, stats, theme }) => {
  const frame = useCurrentFrame();
  
  const opacity = interpolate(frame, [0, 30], [0, 1], {
    extrapolateRight: 'clamp',
  });

  return (
    <AbsoluteFill style={{ backgroundColor: theme === 'modern' ? '#1a1a2e' : '#fff' }}>
      <div style={{ opacity, padding: 40 }}>
        <h1 style={{ fontSize: 60, marginBottom: 30 }}>{title}</h1>
        {stats.map((stat, i) => (
          <div key={i} style={{ fontSize: 30, marginBottom: 20 }}>
            {stat.label}: {stat.value}
          </div>
        ))}
      </div>
    </AbsoluteFill>
  );
};
```

### Batch Processing

```typescript
async function processBatch(keywords: string[]) {
  const results = await Promise.allSettled(
    keywords.map(async (keyword) => {
      const insights = await performResearch(keyword);
      const content = await generateContent({
        insights,
        format: 'toplist',
        languages: ['en']
      });
      return { keyword, content };
    })
  );

  const successful = results
    .filter((r) => r.status === 'fulfilled')
    .map((r) => (r as PromiseFulfilledResult<any>).value);

  const failed = results
    .filter((r) => r.status === 'rejected')
    .map((r) => (r as PromiseRejectedResult).reason);

  return { successful, failed };
}
```

## Troubleshooting

### API Rate Limits

```typescript
import pLimit from 'p-limit';

const limit = pLimit(3); // Max 3 concurrent requests

async function batchWithRateLimit(items: string[]) {
  return Promise.all(
    items.map(item => 
      limit(() => performResearch(item))
    )
  );
}
```

### Claude API Errors

```typescript
import Anthropic from '@anthropic-ai/sdk';

async function robustClaudeCall(prompt: string, maxRetries = 3) {
  const anthropic = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY,
  });

  for (let i = 0; i < maxRetries; i++) {
    try {
      const message = await anthropic.messages.create({
        model: 'claude-3-opus-20240229',
        max_tokens: 4096,
        messages: [{ role: 'user', content: prompt }],
      });
      
      return message.content[0].text;
    } catch (error) {
      if (error.status === 429) {
        // Rate limit - wait and retry
        await new Promise(resolve => setTimeout(resolve, 2000 * (i + 1)));
        continue;
      }
      throw error;
    }
  }
  throw new Error('Max retries exceeded');
}
```

### Video Rendering Memory Issues

```typescript
import { renderMedia } from '@remotion/renderer';

async function renderWithMemoryOptimization(composition: any) {
  return renderMedia({
    ...composition,
    codec: 'h264',
    concurrency: 1, // Reduce concurrency
    enforceAudioTrack: false,
    muted: true, // Disable audio if not needed
    scale: 0.5, // Reduce resolution if needed
  });
}
```

### Missing Dependencies

If you encounter module errors:

```bash
# Ensure all peer dependencies are installed
npm install @anthropic-ai/sdk openai @remotion/bundler @remotion/renderer remotion

# For TypeScript types
npm install -D @types/node @types/react
```

## Performance Optimization

```typescript
// Cache research results
import NodeCache from 'node-cache';

const cache = new NodeCache({ stdTTL: 3600 }); // 1 hour

async function cachedResearch(keyword: string) {
  const cached = cache.get(keyword);
  if (cached) return cached;

  const result = await performResearch(keyword);
  cache.set(keyword, result);
  return result;
}
```
