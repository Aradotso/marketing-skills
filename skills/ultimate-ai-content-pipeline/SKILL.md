---
name: ultimate-ai-content-pipeline
description: Automate content creation from research to video generation with AI-powered research, multi-format writing, and automatic video rendering
triggers:
  - create automated content pipeline with AI
  - set up AI content generation workflow
  - generate videos from written content automatically
  - research and write content with Claude and OpenAI
  - automate content creation from research to video
  - build AI-powered content automation system
  - create AI content pipeline with video rendering
  - set up automated marketing content generation
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

A complete content automation system that handles research, content generation, and video rendering. Uses Claude/OpenAI for writing, web scraping for fresh data, and Remotion for video generation.

## What It Does

This TypeScript/Next.js application automates the entire content creation workflow:

1. **Auto-Research**: Crawls recent articles from TechCrunch, a16z, Twitter, LinkedIn
2. **AI Writing**: Generates content in multiple formats (toplist, POV, case study, how-to) using Claude/OpenAI
3. **Multi-language**: Creates content in both English and Vietnamese
4. **Video Generation**: Automatically renders videos and infographics using Remotion
5. **Platform Optimization**: Exports videos optimized for Reels, TikTok, Shorts

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

Create a `.env.local` file with the following variables:

```bash
# AI Services
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_url

# Next.js
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

## Key APIs and Usage

### Research Module

The research module crawls and analyzes content from multiple sources:

```typescript
import { researchTopic } from '@/lib/research';

async function gatherInsights(keyword: string) {
  const research = await researchTopic({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeframe: '24h',
    maxArticles: 20
  });
  
  return {
    articles: research.articles,
    insights: research.insights,
    trends: research.trends,
    dataPoints: research.statistics
  };
}
```

### Content Generation with Claude

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY
});

async function generateContent(topic: string, format: string, research: any) {
  const prompt = `
Create a ${format} article about ${topic}.
Use these insights: ${JSON.stringify(research.insights)}
Include data points: ${JSON.stringify(research.dataPoints)}
`;

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
  
  return message.content[0].text;
}
```

### Content Generation with OpenAI

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY
});

async function generateWithGPT(topic: string, tone: string, research: any) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo',
    messages: [
      {
        role: 'system',
        content: `You are a content creator with a ${tone} tone.`
      },
      {
        role: 'user',
        content: `Write about ${topic} using these insights: ${JSON.stringify(research)}`
      }
    ],
    temperature: 0.7
  });
  
  return completion.choices[0].message.content;
}
```

### Multi-Language Content Generation

```typescript
async function generateBilingualContent(topic: string, research: any) {
  const [englishContent, vietnameseContent] = await Promise.all([
    generateContent(topic, 'how-to', research),
    generateContent(topic, 'how-to', {
      ...research,
      language: 'vi',
      culturalContext: 'vietnamese'
    })
  ]);
  
  return {
    en: englishContent,
    vi: vietnameseContent
  };
}
```

### Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

async function generateVideo(content: any, format: 'reels' | 'tiktok' | 'shorts') {
  const compositionId = 'ContentVideo';
  
  const bundleLocation = await bundle({
    entryPoint: path.join(process.cwd(), 'src/remotion/index.ts'),
    webpackOverride: (config) => config
  });
  
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: {
      content: content.text,
      title: content.title,
      format: format
    }
  });
  
  const outputLocation = path.join(
    process.cwd(), 
    'public/videos',
    `${content.id}-${format}.mp4`
  );
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: {
      content: content.text,
      title: content.title,
      format: format
    }
  });
  
  return outputLocation;
}
```

## Common Patterns

### Complete Content Pipeline

```typescript
import { researchTopic } from '@/lib/research';
import { generateContent } from '@/lib/ai/claude';
import { generateVideo } from '@/lib/video/remotion';
import { publishToSocial } from '@/lib/publish';

async function runContentPipeline(keyword: string) {
  // Step 1: Research
  console.log('Starting research...');
  const research = await researchTopic({
    keyword,
    sources: ['techcrunch', 'twitter', 'linkedin'],
    timeframe: '24h'
  });
  
  // Step 2: Generate content in multiple formats
  console.log('Generating content...');
  const formats = ['toplist', 'case-study', 'how-to'];
  const contentPieces = await Promise.all(
    formats.map(format => 
      generateContent(keyword, format, research)
    )
  );
  
  // Step 3: Generate videos for each platform
  console.log('Rendering videos...');
  const videos = await Promise.all(
    contentPieces.flatMap(content =>
      ['reels', 'tiktok', 'shorts'].map(platform =>
        generateVideo(content, platform)
      )
    )
  );
  
  // Step 4: Schedule publishing
  console.log('Scheduling posts...');
  await publishToSocial({
    content: contentPieces,
    videos: videos,
    schedule: 'auto'
  });
  
  return {
    research,
    content: contentPieces,
    videos
  };
}
```

### Custom Tone and Style

```typescript
type ContentTone = 'professional' | 'friendly' | 'humorous' | 'expert';

interface ContentConfig {
  format: string;
  tone: ContentTone;
  language: 'en' | 'vi';
  targetAudience: string;
}

async function generateCustomContent(
  topic: string, 
  research: any, 
  config: ContentConfig
) {
  const systemPrompt = `
You are writing ${config.format} content with a ${config.tone} tone.
Target audience: ${config.targetAudience}
Language: ${config.language}
`;

  const content = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    system: systemPrompt,
    messages: [
      {
        role: 'user',
        content: `Topic: ${topic}\n\nResearch: ${JSON.stringify(research)}`
      }
    ]
  });
  
  return content.content[0].text;
}
```

### Batch Processing

```typescript
async function batchContentGeneration(keywords: string[]) {
  const results = [];
  
  for (const keyword of keywords) {
    try {
      const result = await runContentPipeline(keyword);
      results.push({
        keyword,
        success: true,
        data: result
      });
      
      // Rate limiting
      await new Promise(resolve => setTimeout(resolve, 2000));
    } catch (error) {
      results.push({
        keyword,
        success: false,
        error: error.message
      });
    }
  }
  
  return results;
}
```

### Remotion Video Component

```typescript
// src/remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, interpolate } from 'remotion';

interface VideoProps {
  title: string;
  content: string;
  format: string;
}

export const ContentVideo: React.FC<VideoProps> = ({ title, content, format }) => {
  const frame = useCurrentFrame();
  
  const opacity = interpolate(
    frame,
    [0, 30],
    [0, 1],
    { extrapolateRight: 'clamp' }
  );
  
  const dimensions = {
    reels: { width: 1080, height: 1920 },
    tiktok: { width: 1080, height: 1920 },
    shorts: { width: 1080, height: 1920 }
  };
  
  return (
    <AbsoluteFill
      style={{
        backgroundColor: '#1a1a1a',
        justifyContent: 'center',
        alignItems: 'center',
        opacity
      }}
    >
      <div style={{ padding: 60, maxWidth: 900 }}>
        <h1 style={{ 
          color: 'white', 
          fontSize: 72,
          marginBottom: 40 
        }}>
          {title}
        </h1>
        <p style={{ 
          color: '#e0e0e0', 
          fontSize: 36,
          lineHeight: 1.6 
        }}>
          {content}
        </p>
      </div>
    </AbsoluteFill>
  );
};
```

## Development Commands

```bash
# Run development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Render video compositions (Remotion)
npx remotion render src/remotion/index.ts ContentVideo out/video.mp4

# Preview Remotion compositions
npx remotion preview src/remotion/index.ts
```

## API Endpoints

### Research Endpoint

```typescript
// pages/api/research.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { researchTopic } from '@/lib/research';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }
  
  const { keyword, sources, timeframe } = req.body;
  
  try {
    const research = await researchTopic({
      keyword,
      sources: sources || ['techcrunch', 'twitter'],
      timeframe: timeframe || '24h'
    });
    
    res.status(200).json(research);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
}
```

### Content Generation Endpoint

```typescript
// pages/api/generate.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { generateContent } from '@/lib/ai/claude';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }
  
  const { topic, format, research } = req.body;
  
  try {
    const content = await generateContent(topic, format, research);
    res.status(200).json({ content });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
}
```

### Video Generation Endpoint

```typescript
// pages/api/video/render.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { generateVideo } from '@/lib/video/remotion';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }
  
  const { content, format } = req.body;
  
  try {
    const videoPath = await generateVideo(content, format);
    res.status(200).json({ videoPath });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
}
```

## Troubleshooting

### API Rate Limits

If you hit rate limits, implement exponential backoff:

```typescript
async function retryWithBackoff<T>(
  fn: () => Promise<T>,
  maxRetries = 3
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      
      const delay = Math.pow(2, i) * 1000;
      console.log(`Retry ${i + 1} after ${delay}ms`);
      await new Promise(resolve => setTimeout(resolve, delay));
    }
  }
  throw new Error('Max retries reached');
}
```

### Video Rendering Issues

Ensure ffmpeg is installed:

```bash
# macOS
brew install ffmpeg

# Ubuntu/Debian
sudo apt-get install ffmpeg

# Verify installation
ffmpeg -version
```

### Memory Issues with Large Batches

Process content in smaller chunks:

```typescript
async function processInChunks<T, R>(
  items: T[],
  processFn: (item: T) => Promise<R>,
  chunkSize = 5
): Promise<R[]> {
  const results: R[] = [];
  
  for (let i = 0; i < items.length; i += chunkSize) {
    const chunk = items.slice(i, i + chunkSize);
    const chunkResults = await Promise.all(
      chunk.map(processFn)
    );
    results.push(...chunkResults);
  }
  
  return results;
}
```

### Claude API Timeout

Increase timeout and handle streaming:

```typescript
async function generateWithTimeout(topic: string, timeoutMs = 60000) {
  const controller = new AbortController();
  const timeout = setTimeout(() => controller.abort(), timeoutMs);
  
  try {
    const response = await anthropic.messages.create({
      model: 'claude-3-5-sonnet-20241022',
      max_tokens: 4096,
      messages: [{ role: 'user', content: topic }]
    }, {
      signal: controller.signal
    });
    
    return response;
  } finally {
    clearTimeout(timeout);
  }
}
```
