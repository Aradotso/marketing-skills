---
name: marketing-pipeline-share-automation
description: Automated AI content pipeline for research, scripting, posting, and video generation using Claude, OpenAI, and Remotion
triggers:
  - how do I automate content creation with AI
  - set up marketing pipeline for content generation
  - create automated video content from research
  - build AI-powered content workflow
  - generate multilingual marketing content automatically
  - configure Claude and OpenAI for content automation
  - automate research to video pipeline
  - set up AI content scraping and generation
---

# Marketing Pipeline Share - AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

Ultimate AI Content Pipeline is an automated content production system that handles everything from research and scraping to content generation and video rendering. It uses Claude 3, OpenAI, and Remotion to create a fully automated content factory.

## What It Does

This TypeScript/Next.js system automates:
- **Auto-research**: Crawls TechCrunch, a16z, Twitter/X, LinkedIn for fresh data (last 24h)
- **Multi-format content**: Generates toplists, POV articles, case studies, how-tos
- **Multilingual output**: Simultaneous English & Vietnamese content
- **Video generation**: Automatic video/infographic rendering via Remotion
- **Platform optimization**: Exports for Reels, TikTok, Shorts

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
cp .env.example .env.local
```

## Configuration

Create `.env.local` with required API keys:

```bash
# AI Providers
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Content Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Optional: Custom configurations
CONTENT_LANGUAGE=en,vi
DEFAULT_TONE=professional
VIDEO_ASPECT_RATIO=9:16
```

## Core Architecture

### 1. Research Pipeline

The research module crawls and aggregates content:

```typescript
// lib/research/crawler.ts
import { fetchTechCrunchNews } from './sources/techcrunch';
import { fetchTwitterTrends } from './sources/twitter';
import { fetchLinkedInPosts } from './sources/linkedin';

interface ResearchResult {
  sources: string[];
  insights: string[];
  data: any[];
  timestamp: Date;
}

export async function runResearchPipeline(
  keyword: string,
  timeframe: string = '24h'
): Promise<ResearchResult> {
  const [techNews, twitterData, linkedinPosts] = await Promise.all([
    fetchTechCrunchNews(keyword, timeframe),
    fetchTwitterTrends(keyword),
    fetchLinkedInPosts(keyword)
  ]);

  return {
    sources: ['TechCrunch', 'Twitter', 'LinkedIn'],
    insights: extractInsights([techNews, twitterData, linkedinPosts]),
    data: [...techNews, ...twitterData, ...linkedinPosts],
    timestamp: new Date()
  };
}

function extractInsights(data: any[]): string[] {
  // AI-powered insight extraction logic
  return data
    .flatMap(source => source.highlights || [])
    .filter(unique);
}
```

### 2. Content Generation

Generate content using Claude or OpenAI:

```typescript
// lib/generation/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

interface ContentConfig {
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'professional' | 'friendly' | 'humorous';
  wordCount: number;
}

export async function generateContent(
  research: ResearchResult,
  config: ContentConfig
): Promise<string> {
  const anthropic = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY
  });

  const prompt = buildPrompt(research, config);

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [{
      role: 'user',
      content: prompt
    }]
  });

  return message.content[0].text;
}

function buildPrompt(research: ResearchResult, config: ContentConfig): string {
  return `
You are a ${config.tone} content writer creating a ${config.format} article.

Research Data:
${JSON.stringify(research.insights, null, 2)}

Requirements:
- Language: ${config.language}
- Format: ${config.format}
- Word count: ~${config.wordCount} words
- Include data-backed insights
- Use recent sources (last 24h)

Generate the complete article:
`;
}
```

### 3. Multilingual Generation

Create parallel language versions:

```typescript
// lib/generation/multilingual.ts
export async function generateMultilingual(
  research: ResearchResult,
  baseConfig: ContentConfig
): Promise<Record<string, string>> {
  const languages = ['en', 'vi'];
  
  const contents = await Promise.all(
    languages.map(lang => 
      generateContent(research, { ...baseConfig, language: lang as 'en' | 'vi' })
    )
  );

  return Object.fromEntries(
    languages.map((lang, i) => [lang, contents[i]])
  );
}
```

### 4. Video Rendering with Remotion

Convert content to video:

```typescript
// lib/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  aspectRatio: '16:9' | '9:16' | '1:1';
  duration: number; // in frames (30fps)
  content: string;
}

export async function renderContentVideo(
  config: VideoConfig
): Promise<string> {
  const bundleLocation = await bundle(
    path.join(process.cwd(), 'remotion/index.ts')
  );

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo'
  });

  const outputPath = path.join(
    process.cwd(), 
    'public/videos',
    `video-${Date.now()}.mp4`
  );

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: {
      content: config.content,
      aspectRatio: config.aspectRatio
    }
  });

  return outputPath;
}
```

### 5. Remotion Video Component

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, Sequence, useCurrentFrame, useVideoConfig } from 'remotion';
import React from 'react';

interface ContentVideoProps {
  content: string;
  aspectRatio: string;
}

export const ContentVideo: React.FC<ContentVideoProps> = ({ content, aspectRatio }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  const sections = content.split('\n\n').filter(Boolean);

  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a2e' }}>
      {sections.map((section, i) => (
        <Sequence
          key={i}
          from={i * fps * 3}
          durationInFrames={fps * 3}
        >
          <AbsoluteFill
            style={{
              justifyContent: 'center',
              alignItems: 'center',
              padding: 60
            }}
          >
            <h2
              style={{
                color: 'white',
                fontSize: aspectRatio === '9:16' ? 48 : 72,
                textAlign: 'center',
                lineHeight: 1.4
              }}
            >
              {section}
            </h2>
          </AbsoluteFill>
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};
```

## API Routes (Next.js)

### Content Generation Endpoint

```typescript
// pages/api/generate.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { runResearchPipeline } from '@/lib/research/crawler';
import { generateMultilingual } from '@/lib/generation/multilingual';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  const { keyword, format, tone } = req.body;

  try {
    // Step 1: Research
    const research = await runResearchPipeline(keyword);

    // Step 2: Generate content
    const content = await generateMultilingual(research, {
      format: format || 'toplist',
      tone: tone || 'professional',
      wordCount: 800,
      language: 'en' // Will be overridden
    });

    res.status(200).json({
      success: true,
      research: research.insights,
      content
    });
  } catch (error) {
    res.status(500).json({ 
      error: 'Generation failed',
      message: error.message 
    });
  }
}
```

### Video Generation Endpoint

```typescript
// pages/api/video.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { renderContentVideo } from '@/lib/video/renderer';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  const { content, aspectRatio = '9:16' } = req.body;

  try {
    const videoPath = await renderContentVideo({
      content,
      aspectRatio,
      duration: 900 // 30 seconds at 30fps
    });

    res.status(200).json({
      success: true,
      videoUrl: videoPath.replace('public', '')
    });
  } catch (error) {
    res.status(500).json({ 
      error: 'Video rendering failed',
      message: error.message 
    });
  }
}
```

## Common Usage Patterns

### Full Pipeline Automation

```typescript
// lib/pipeline/full-automation.ts
import { runResearchPipeline } from '@/lib/research/crawler';
import { generateMultilingual } from '@/lib/generation/multilingual';
import { renderContentVideo } from '@/lib/video/renderer';

export async function runFullPipeline(keyword: string) {
  // 1. Research
  console.log('Starting research...');
  const research = await runResearchPipeline(keyword);

  // 2. Generate content
  console.log('Generating content...');
  const content = await generateMultilingual(research, {
    format: 'toplist',
    language: 'en',
    tone: 'professional',
    wordCount: 800
  });

  // 3. Render videos
  console.log('Rendering videos...');
  const videos = await Promise.all(
    Object.entries(content).map(([lang, text]) =>
      renderContentVideo({
        content: text,
        aspectRatio: '9:16',
        duration: 900
      })
    )
  );

  return {
    research: research.insights,
    content,
    videos
  };
}
```

### Scheduled Content Generation

```typescript
// lib/scheduler/content-scheduler.ts
import cron from 'node-cron';
import { runFullPipeline } from '@/lib/pipeline/full-automation';

export function scheduleContentGeneration(keywords: string[]) {
  // Run every day at 8 AM
  cron.schedule('0 8 * * *', async () => {
    console.log('Running scheduled content generation...');
    
    for (const keyword of keywords) {
      try {
        const result = await runFullPipeline(keyword);
        console.log(`Generated content for: ${keyword}`);
        // Save to database or post to social media
      } catch (error) {
        console.error(`Failed for ${keyword}:`, error);
      }
    }
  });
}
```

## Running the Application

```bash
# Development mode
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Render videos (Remotion)
npm run remotion:render
```

## Troubleshooting

### API Rate Limits

If you hit rate limits:

```typescript
// lib/utils/rate-limiter.ts
import pLimit from 'p-limit';

const limit = pLimit(3); // Max 3 concurrent requests

export async function rateLimitedFetch(urls: string[]) {
  return Promise.all(
    urls.map(url => limit(() => fetch(url)))
  );
}
```

### Video Rendering Timeouts

For long videos, increase timeout:

```typescript
await renderMedia({
  composition,
  serveUrl: bundleLocation,
  codec: 'h264',
  outputLocation: outputPath,
  timeoutInMilliseconds: 300000, // 5 minutes
  inputProps: { content, aspectRatio }
});
```

### Memory Issues with Large Content

Use streaming for large datasets:

```typescript
import { Readable } from 'stream';

export async function* streamResearch(keyword: string) {
  const sources = ['techcrunch', 'twitter', 'linkedin'];
  
  for (const source of sources) {
    const data = await fetchFromSource(source, keyword);
    yield data;
  }
}
```

## Best Practices

1. **Cache research results** to avoid redundant API calls
2. **Use job queues** (Bull, BullMQ) for video rendering
3. **Store generated content** in a database (MongoDB, PostgreSQL)
4. **Implement retry logic** for API failures
5. **Monitor API usage** to stay within quota limits
6. **Version your prompts** for reproducible content generation
