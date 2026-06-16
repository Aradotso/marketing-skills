---
name: marketing-pipeline-share-automation
description: TypeScript automation pipeline for AI-powered content creation from research to video generation using Claude, OpenAI, and Remotion
triggers:
  - automate content creation with AI pipeline
  - generate videos from content automatically
  - crawl news and create marketing content
  - build automated content workflow
  - research to video content pipeline
  - use marketing pipeline share automation
  - create AI content from research to post
  - automate video generation from articles
---

# Marketing Pipeline Share - AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the Marketing Pipeline Share project - an end-to-end automated content creation system that handles research, scriptwriting, posting, and video generation using AI (Claude 3, OpenAI) and Remotion.

## What This Project Does

Marketing Pipeline Share is a complete content automation pipeline that:

- **Auto-crawls news** from TechCrunch, a16z, X (Twitter), LinkedIn for real-time insights
- **Generates multi-format content** (toplist, POV, case studies, how-to) in multiple languages
- **Renders videos automatically** using Remotion for social media (Reels, TikTok, Shorts)
- **Optimizes workflow** to reduce content creation time by 90%

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
# AI Services
ANTHROPIC_API_KEY=your_claude_api_key_here
OPENAI_API_KEY=your_openai_api_key_here

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key_here

# Optional: Custom endpoints
NEXT_PUBLIC_API_URL=http://localhost:3000/api

# Remotion (for video rendering)
REMOTION_LICENSE_KEY=your_remotion_license_key_here
```

## Running the Application

```bash
# Development mode
npm run dev

# Production build
npm run build
npm start

# Run Remotion video studio
npm run remotion:studio
```

## Key API Endpoints

### Content Research

```typescript
// pages/api/research/scan.ts
import { NextApiRequest, NextApiResponse } from 'next';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  const { keyword, sources = ['techcrunch', 'twitter', 'linkedin'] } = req.body;

  try {
    const researchData = await scanSources({
      keyword,
      sources,
      timeframe: '24h'
    });

    res.status(200).json({ success: true, data: researchData });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
}
```

### Content Generation

```typescript
// lib/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

export async function generateContent({
  researchData,
  format = 'toplist',
  language = 'vi',
  tone = 'professional'
}: {
  researchData: any;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'professional' | 'friendly' | 'humorous';
}) {
  const prompt = buildPrompt(researchData, format, language, tone);

  // Using Claude for content generation
  const message = await anthropic.messages.create({
    model: 'claude-3-opus-20240229',
    max_tokens: 4096,
    messages: [
      {
        role: 'user',
        content: prompt
      }
    ]
  });

  return message.content;
}

function buildPrompt(data: any, format: string, language: string, tone: string): string {
  const formatInstructions = {
    'toplist': 'Create a numbered list article with at least 5 items',
    'pov': 'Write from a personal perspective with insights and opinions',
    'case-study': 'Analyze with data, examples, and conclusions',
    'how-to': 'Step-by-step guide with actionable instructions'
  };

  return `
You are a professional content writer. Based on this research data:
${JSON.stringify(data, null, 2)}

Create a ${format} article in ${language} with a ${tone} tone.
${formatInstructions[format]}

Include:
- Engaging headline
- Well-structured content
- Data-backed insights
- Call-to-action
`;
}
```

### Video Generation with Remotion

```typescript
// lib/video-renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

export async function renderContentVideo({
  content,
  outputPath,
  format = 'reels'
}: {
  content: any;
  outputPath: string;
  format: 'reels' | 'tiktok' | 'shorts';
}) {
  const compositionId = 'ContentVideo';
  const bundleLocation = await bundle(
    path.join(process.cwd(), 'remotion/index.ts')
  );

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: {
      content,
      format
    }
  });

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: {
      content,
      format
    }
  });

  return outputPath;
}
```

## Remotion Video Composition

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig, interpolate } from 'remotion';
import React from 'react';

export const ContentVideo: React.FC<{
  content: {
    title: string;
    points: string[];
    background?: string;
  };
  format: 'reels' | 'tiktok' | 'shorts';
}> = ({ content, format }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  const opacity = interpolate(frame, [0, 30], [0, 1], {
    extrapolateRight: 'clamp',
  });

  return (
    <AbsoluteFill style={{ backgroundColor: content.background || '#1a1a1a' }}>
      <div style={{ opacity, padding: '40px', color: 'white' }}>
        <h1 style={{ fontSize: '48px', marginBottom: '30px' }}>
          {content.title}
        </h1>
        <ul style={{ fontSize: '32px', lineHeight: '1.6' }}>
          {content.points.map((point, index) => {
            const pointFrame = 60 + index * 90;
            const pointOpacity = interpolate(
              frame,
              [pointFrame, pointFrame + 30],
              [0, 1],
              { extrapolateRight: 'clamp' }
            );
            return (
              <li key={index} style={{ opacity: pointOpacity, marginBottom: '20px' }}>
                {point}
              </li>
            );
          })}
        </ul>
      </div>
    </AbsoluteFill>
  );
};
```

## Complete Pipeline Workflow

```typescript
// lib/pipeline.ts
import { scanSources } from './research-scanner';
import { generateContent } from './content-generator';
import { renderContentVideo } from './video-renderer';

export async function runContentPipeline({
  keyword,
  format = 'toplist',
  language = 'vi',
  generateVideo = true
}: {
  keyword: string;
  format?: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language?: 'en' | 'vi';
  generateVideo?: boolean;
}) {
  // Step 1: Research
  console.log('🔍 Scanning sources for:', keyword);
  const researchData = await scanSources({
    keyword,
    sources: ['techcrunch', 'twitter', 'linkedin'],
    timeframe: '24h'
  });

  // Step 2: Generate Content
  console.log('✍️ Generating content...');
  const content = await generateContent({
    researchData,
    format,
    language,
    tone: 'professional'
  });

  // Step 3: Render Video (optional)
  let videoPath = null;
  if (generateVideo) {
    console.log('🎬 Rendering video...');
    videoPath = await renderContentVideo({
      content,
      outputPath: `./output/${keyword}-${Date.now()}.mp4`,
      format: 'reels'
    });
  }

  return {
    content,
    videoPath,
    metadata: {
      keyword,
      format,
      language,
      generatedAt: new Date().toISOString()
    }
  };
}
```

## Usage Example

```typescript
// Example: Run complete pipeline
import { runContentPipeline } from '@/lib/pipeline';

async function createContent() {
  try {
    const result = await runContentPipeline({
      keyword: 'AI marketing trends 2024',
      format: 'toplist',
      language: 'vi',
      generateVideo: true
    });

    console.log('Content generated:', result.content);
    console.log('Video saved to:', result.videoPath);
  } catch (error) {
    console.error('Pipeline error:', error);
  }
}

createContent();
```

## Research Scanner Implementation

```typescript
// lib/research-scanner.ts
import axios from 'axios';

export async function scanSources({
  keyword,
  sources,
  timeframe = '24h'
}: {
  keyword: string;
  sources: string[];
  timeframe: string;
}) {
  const results = await Promise.all(
    sources.map(source => fetchFromSource(source, keyword, timeframe))
  );

  return aggregateResults(results);
}

async function fetchFromSource(source: string, keyword: string, timeframe: string) {
  const rapidApiKey = process.env.RAPIDAPI_KEY;

  switch (source) {
    case 'techcrunch':
      return await axios.get('https://techcrunch-api.p.rapidapi.com/search', {
        params: { query: keyword },
        headers: {
          'X-RapidAPI-Key': rapidApiKey,
          'X-RapidAPI-Host': 'techcrunch-api.p.rapidapi.com'
        }
      });

    case 'twitter':
      return await axios.get('https://twitter-api.p.rapidapi.com/search', {
        params: { query: keyword, count: 20 },
        headers: {
          'X-RapidAPI-Key': rapidApiKey,
          'X-RapidAPI-Host': 'twitter-api.p.rapidapi.com'
        }
      });

    default:
      throw new Error(`Unknown source: ${source}`);
  }
}

function aggregateResults(results: any[]) {
  return results.flat().map(item => ({
    title: item.title || item.text,
    url: item.url || item.link,
    source: item.source,
    publishedAt: item.publishedAt || item.created_at,
    summary: item.summary || item.description
  }));
}
```

## Common Patterns

### Multi-language Content Generation

```typescript
async function generateBilingualContent(researchData: any) {
  const [englishContent, vietnameseContent] = await Promise.all([
    generateContent({ researchData, format: 'toplist', language: 'en', tone: 'professional' }),
    generateContent({ researchData, format: 'toplist', language: 'vi', tone: 'professional' })
  ]);

  return { en: englishContent, vi: vietnameseContent };
}
```

### Batch Video Rendering

```typescript
async function batchRenderVideos(contents: any[]) {
  return Promise.all(
    contents.map((content, index) =>
      renderContentVideo({
        content,
        outputPath: `./output/video-${index}.mp4`,
        format: 'reels'
      })
    )
  );
}
```

## Troubleshooting

### API Rate Limits

If you encounter rate limit errors:

```typescript
// Add retry logic with exponential backoff
async function generateContentWithRetry(params: any, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await generateContent(params);
    } catch (error) {
      if (error.status === 429 && i < maxRetries - 1) {
        await new Promise(resolve => setTimeout(resolve, Math.pow(2, i) * 1000));
        continue;
      }
      throw error;
    }
  }
}
```

### Video Rendering Memory Issues

For large video renders, use progressive rendering:

```typescript
// Increase Node memory limit
// In package.json:
{
  "scripts": {
    "remotion:render": "node --max-old-space-size=4096 ./node_modules/@remotion/cli/dist/index.js render"
  }
}
```

### Missing Environment Variables

Always validate required environment variables:

```typescript
function validateEnv() {
  const required = ['ANTHROPIC_API_KEY', 'OPENAI_API_KEY', 'RAPIDAPI_KEY'];
  const missing = required.filter(key => !process.env[key]);
  
  if (missing.length > 0) {
    throw new Error(`Missing required environment variables: ${missing.join(', ')}`);
  }
}
```

This skill equips AI coding agents to effectively use Marketing Pipeline Share for automating content creation workflows from research to video generation.
