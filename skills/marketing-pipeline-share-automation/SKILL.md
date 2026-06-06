---
name: marketing-pipeline-share-automation
description: AI-powered content automation pipeline for research, scriptwriting, and video generation with Claude/OpenAI
triggers:
  - automate content creation with AI research
  - generate videos from content automatically
  - scrape news and create social media content
  - build AI content pipeline with remotion
  - setup automated marketing workflow
  - create content from research to video
  - automate social media content generation
  - build content automation with Claude API
---

# Marketing Pipeline Share Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with **Marketing Pipeline Share**, an all-in-one AI content automation system that transforms keywords into complete content packages including research, scripts, and rendered videos. Built with TypeScript/Next.js, it integrates Claude 3, OpenAI, and Remotion for end-to-end content production.

## What It Does

Marketing Pipeline Share automates the entire content creation workflow:

1. **Auto-Research**: Crawls real-time data from TechCrunch, a16z, X (Twitter), LinkedIn
2. **AI Content Generation**: Creates posts in multiple formats (toplist, POV, case study, how-to) using Claude/OpenAI
3. **Multi-language Support**: Generates content in English and Vietnamese with customizable tone
4. **Video Rendering**: Automatically creates infographics and short videos using Remotion
5. **Platform Optimization**: Exports video in formats optimized for Reels, TikTok, Shorts

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
# AI API Keys
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# RapidAPI for Research/Scraping
RAPIDAPI_KEY=your_rapidapi_key

# Optional: Remotion License (for video rendering)
REMOTION_LICENSE_KEY=your_remotion_license

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Next.js Configuration
NEXT_PUBLIC_API_URL=http://localhost:3000
```

## Running the Application

```bash
# Development mode
npm run dev

# Production build
npm run build
npm start

# Remotion video rendering
npm run remotion
```

## Core Architecture & API

### Content Pipeline Flow

```typescript
// types/content-pipeline.ts
export interface ContentRequest {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi' | 'both';
  tone: 'expert' | 'friendly' | 'humorous';
  includeVideo: boolean;
}

export interface ResearchData {
  sources: Array<{
    title: string;
    url: string;
    publishedAt: string;
    summary: string;
  }>;
  insights: string[];
  statistics: Array<{
    metric: string;
    value: string;
    source: string;
  }>;
}

export interface GeneratedContent {
  title: string;
  content: string;
  language: string;
  metadata: {
    wordCount: number;
    readingTime: number;
    format: string;
  };
}
```

### Research Module

```typescript
// lib/research/auto-scraper.ts
import axios from 'axios';

export async function autoResearch(keyword: string): Promise<ResearchData> {
  const sources = [
    { name: 'TechCrunch', endpoint: 'techcrunch-api' },
    { name: 'a16z', endpoint: 'a16z-blog' },
    { name: 'Twitter', endpoint: 'twitter-search' },
  ];

  const results = await Promise.all(
    sources.map(async (source) => {
      try {
        const response = await axios.get(
          `https://api.rapidapi.com/${source.endpoint}`,
          {
            params: { q: keyword, within: '24h' },
            headers: {
              'X-RapidAPI-Key': process.env.RAPIDAPI_KEY,
              'X-RapidAPI-Host': 'your-rapidapi-host.p.rapidapi.com',
            },
          }
        );
        return response.data.articles || [];
      } catch (error) {
        console.error(`Error fetching from ${source.name}:`, error);
        return [];
      }
    })
  );

  const allArticles = results.flat();
  
  return {
    sources: allArticles.map((article) => ({
      title: article.title,
      url: article.url,
      publishedAt: article.publishedAt,
      summary: article.description || '',
    })),
    insights: extractInsights(allArticles),
    statistics: extractStatistics(allArticles),
  };
}

function extractInsights(articles: any[]): string[] {
  // AI-powered insight extraction logic
  return articles
    .map((a) => a.keyPoints || [])
    .flat()
    .slice(0, 10);
}

function extractStatistics(articles: any[]): Array<any> {
  // Extract numerical data and metrics
  return articles
    .flatMap((a) => a.statistics || [])
    .slice(0, 5);
}
```

### AI Content Generation

```typescript
// lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

export async function generateContent(
  research: ResearchData,
  request: ContentRequest
): Promise<GeneratedContent> {
  const prompt = buildPrompt(research, request);

  // Use Claude for primary content generation
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

  const content = message.content[0].type === 'text' 
    ? message.content[0].text 
    : '';

  return {
    title: extractTitle(content),
    content: content,
    language: request.language === 'both' ? 'en' : request.language,
    metadata: {
      wordCount: content.split(/\s+/).length,
      readingTime: Math.ceil(content.split(/\s+/).length / 200),
      format: request.format,
    },
  };
}

function buildPrompt(research: ResearchData, request: ContentRequest): string {
  const formatInstructions = {
    'toplist': 'Create a numbered list article with clear rankings',
    'pov': 'Write from a unique perspective with strong opinion',
    'case-study': 'Analyze a real case with data and outcomes',
    'how-to': 'Provide step-by-step actionable instructions',
  };

  const toneInstructions = {
    'expert': 'authoritative, data-driven, professional',
    'friendly': 'conversational, approachable, warm',
    'humorous': 'witty, engaging, entertaining',
  };

  return `
You are an expert content creator. Based on the following research data, create a ${request.format} article.

RESEARCH DATA:
${JSON.stringify(research, null, 2)}

REQUIREMENTS:
- Format: ${formatInstructions[request.format]}
- Tone: ${toneInstructions[request.tone]}
- Language: ${request.language}
- Include specific statistics and cite sources
- Make it actionable and engaging
- Use real examples from the research

Generate the complete article now.
  `.trim();
}

function extractTitle(content: string): string {
  const titleMatch = content.match(/^#\s+(.+)$/m);
  return titleMatch ? titleMatch[1] : content.split('\n')[0].slice(0, 100);
}
```

### Bilingual Content Generation

```typescript
// lib/ai/multilingual-generator.ts
export async function generateBilingualContent(
  research: ResearchData,
  request: ContentRequest
): Promise<{ en: GeneratedContent; vi: GeneratedContent }> {
  const [enContent, viContent] = await Promise.all([
    generateContent(research, { ...request, language: 'en' }),
    generateContent(research, { ...request, language: 'vi' }),
  ]);

  return { en: enContent, vi: viContent };
}
```

### Video Generation with Remotion

```typescript
// lib/video/remotion-renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

export interface VideoConfig {
  content: GeneratedContent;
  platform: 'reels' | 'tiktok' | 'shorts';
  duration: number;
}

const platformConfigs = {
  reels: { width: 1080, height: 1920, fps: 30 },
  tiktok: { width: 1080, height: 1920, fps: 30 },
  shorts: { width: 1080, height: 1920, fps: 30 },
};

export async function renderContentVideo(
  config: VideoConfig
): Promise<string> {
  const compositionId = 'ContentVideo';
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: {
      title: config.content.title,
      content: config.content.content,
      stats: config.content.metadata,
    },
  });

  const platformConfig = platformConfigs[config.platform];
  const outputLocation = `./output/video-${Date.now()}.mp4`;

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: composition.defaultProps,
    ...platformConfig,
  });

  return outputLocation;
}
```

### Remotion Video Component

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, Sequence, useCurrentFrame, useVideoConfig } from 'remotion';
import React from 'react';

export const ContentVideo: React.FC<{
  title: string;
  content: string;
  stats: any;
}> = ({ title, content, stats }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  const opacity = Math.min(1, frame / (fps * 0.5));
  const contentParts = content.split('\n\n').slice(0, 5);

  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a2e' }}>
      <Sequence from={0} durationInFrames={fps * 2}>
        <div
          style={{
            opacity,
            color: 'white',
            fontSize: 60,
            fontWeight: 'bold',
            padding: 60,
            textAlign: 'center',
          }}
        >
          {title}
        </div>
      </Sequence>

      {contentParts.map((part, index) => (
        <Sequence
          key={index}
          from={fps * (2 + index * 3)}
          durationInFrames={fps * 3}
        >
          <div
            style={{
              color: 'white',
              fontSize: 36,
              padding: 60,
              lineHeight: 1.6,
            }}
          >
            {part}
          </div>
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};
```

## API Routes (Next.js)

```typescript
// pages/api/generate-content.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { autoResearch } from '@/lib/research/auto-scraper';
import { generateContent } from '@/lib/ai/content-generator';
import { renderContentVideo } from '@/lib/video/remotion-renderer';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  try {
    const { keyword, format, language, tone, includeVideo } = req.body;

    // Step 1: Research
    const research = await autoResearch(keyword);

    // Step 2: Generate Content
    const content = await generateContent(research, {
      keyword,
      format,
      language,
      tone,
      includeVideo,
    });

    // Step 3: Render Video (if requested)
    let videoUrl = null;
    if (includeVideo) {
      videoUrl = await renderContentVideo({
        content,
        platform: 'reels',
        duration: 30,
      });
    }

    res.status(200).json({
      success: true,
      research,
      content,
      videoUrl,
    });
  } catch (error) {
    console.error('Content generation error:', error);
    res.status(500).json({ error: 'Content generation failed' });
  }
}
```

## Common Usage Patterns

### Full Pipeline Execution

```typescript
// Example: Complete content generation flow
import { autoResearch } from './lib/research/auto-scraper';
import { generateBilingualContent } from './lib/ai/multilingual-generator';
import { renderContentVideo } from './lib/video/remotion-renderer';

async function createCompleteContentPackage(keyword: string) {
  // 1. Research phase
  console.log('🔍 Researching:', keyword);
  const research = await autoResearch(keyword);

  // 2. Content generation (bilingual)
  console.log('✍️ Generating content...');
  const content = await generateBilingualContent(research, {
    keyword,
    format: 'toplist',
    language: 'both',
    tone: 'expert',
    includeVideo: true,
  });

  // 3. Video rendering
  console.log('🎬 Rendering videos...');
  const [enVideo, viVideo] = await Promise.all([
    renderContentVideo({
      content: content.en,
      platform: 'reels',
      duration: 30,
    }),
    renderContentVideo({
      content: content.vi,
      platform: 'tiktok',
      duration: 30,
    }),
  ]);

  return {
    research,
    content,
    videos: { en: enVideo, vi: viVideo },
  };
}
```

### Custom Research Sources

```typescript
// Add custom data sources
import { ResearchData } from '@/types/content-pipeline';

async function addCustomSource(
  existing: ResearchData,
  customUrl: string
): Promise<ResearchData> {
  const response = await fetch(customUrl);
  const data = await response.json();

  return {
    ...existing,
    sources: [...existing.sources, ...data.articles],
  };
}
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement retry logic for API calls
async function retryWithBackoff<T>(
  fn: () => Promise<T>,
  maxRetries = 3
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error: any) {
      if (error.status === 429 && i < maxRetries - 1) {
        const delay = Math.pow(2, i) * 1000;
        console.log(`Rate limited, retrying in ${delay}ms...`);
        await new Promise((resolve) => setTimeout(resolve, delay));
      } else {
        throw error;
      }
    }
  }
  throw new Error('Max retries exceeded');
}
```

### Video Rendering Memory Issues

```typescript
// For large videos, use chunked rendering
import { renderFrames } from '@remotion/renderer';

async function renderLargeVideo(config: VideoConfig) {
  // Render in chunks to avoid memory issues
  const chunkSize = 300; // frames
  // Implementation depends on your specific needs
  console.log('Rendering in chunks for memory optimization');
}
```

### Missing Environment Variables

```typescript
// Validate environment on startup
function validateEnv() {
  const required = [
    'ANTHROPIC_API_KEY',
    'OPENAI_API_KEY',
    'RAPIDAPI_KEY',
  ];

  const missing = required.filter((key) => !process.env[key]);

  if (missing.length > 0) {
    throw new Error(
      `Missing required environment variables: ${missing.join(', ')}`
    );
  }
}

// Call at app initialization
validateEnv();
```

## Best Practices

1. **Cache Research Data**: Store research results to avoid redundant API calls
2. **Queue Video Rendering**: Use a job queue (Bull, BullMQ) for video rendering to prevent timeout
3. **Implement Rate Limiting**: Respect API rate limits with proper throttling
4. **Error Handling**: Wrap all external API calls with comprehensive error handling
5. **Content Validation**: Validate generated content before rendering to video
6. **Optimize Remotion**: Pre-bundle Remotion compositions for faster rendering
