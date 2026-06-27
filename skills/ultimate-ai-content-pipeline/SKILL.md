---
name: ultimate-ai-content-pipeline
description: Automated content creation pipeline that researches, generates scripts, and produces videos using Claude/OpenAI and Remotion
triggers:
  - how do I generate automated content with AI
  - set up content pipeline with video generation
  - automate content research and video creation
  - use remotion for automated video rendering
  - create AI-powered content workflow
  - generate multilingual content with Claude
  - build automated marketing content system
  - crawl news and generate content automatically
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This TypeScript-based system automates the entire content creation workflow: from researching trending news, generating scripts in multiple formats and languages, to rendering videos automatically. It combines web scraping, AI content generation (Claude 3/OpenAI), and video production (Remotion) into a single pipeline.

## What It Does

- **Auto-Research**: Crawls recent content from TechCrunch, a16z, Twitter, LinkedIn
- **AI Content Generation**: Creates articles in multiple formats (listicles, POV, case studies, how-tos)
- **Multilingual**: Generates content in both English and Vietnamese
- **Video Rendering**: Automatically creates infographics and short videos from content using Remotion
- **Platform-Ready**: Exports videos optimized for Reels, TikTok, Shorts

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
# AI Service APIs
ANTHROPIC_API_KEY=your_claude_api_key_here
OPENAI_API_KEY=your_openai_api_key_here

# Web Scraping (RapidAPI)
RAPIDAPI_KEY=your_rapidapi_key_here

# Next.js Configuration
NEXT_PUBLIC_APP_URL=http://localhost:3000

# Remotion Configuration (optional)
REMOTION_LICENSE_KEY=your_remotion_license_key_here
```

## Key Commands

### Development

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Start production server
npm start
```

### Video Rendering (Remotion)

```bash
# Render a video composition
npx remotion render src/remotion/index.ts <composition-id> out/video.mp4

# Preview Remotion compositions
npx remotion preview
```

## Core API Usage

### 1. Content Research Module

```typescript
import { scrapeNews } from '@/lib/scraper';

// Crawl recent news from multiple sources
async function researchTopic(keyword: string) {
  const sources = ['techcrunch', 'a16z', 'twitter'];
  
  const newsData = await scrapeNews({
    keyword,
    sources,
    timeframe: '24h',
    limit: 10
  });
  
  return newsData;
}
```

### 2. AI Content Generation with Claude

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

async function generateContent(
  topic: string,
  format: 'toplist' | 'pov' | 'case-study' | 'how-to',
  language: 'en' | 'vi'
) {
  const prompt = `Create a ${format} article about "${topic}" in ${language}. 
  Use recent data and insights. Include data-backed claims.`;
  
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

### 3. OpenAI Alternative

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateWithGPT(topic: string, tone: string) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are a professional content writer with a ${tone} tone.`
      },
      {
        role: 'user',
        content: `Write an engaging article about: ${topic}`
      }
    ],
    temperature: 0.7,
  });
  
  return completion.choices[0].message.content;
}
```

### 4. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

async function generateVideo(
  content: string,
  title: string,
  outputPath: string
) {
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./src/remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title,
      content,
      duration: 30, // seconds
    },
  });

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: {
      title,
      content,
    },
  });
}
```

## Complete Pipeline Example

```typescript
import { scrapeNews } from '@/lib/scraper';
import { generateContent } from '@/lib/ai-generator';
import { generateVideo } from '@/lib/video-renderer';

async function runContentPipeline(keyword: string) {
  try {
    // Step 1: Research
    console.log('🔍 Researching topic...');
    const research = await scrapeNews({
      keyword,
      sources: ['techcrunch', 'twitter'],
      timeframe: '24h',
    });
    
    // Step 2: Generate content in both languages
    console.log('✍️ Generating content...');
    const contentEN = await generateContent(
      keyword,
      'toplist',
      'en'
    );
    
    const contentVI = await generateContent(
      keyword,
      'toplist',
      'vi'
    );
    
    // Step 3: Create video
    console.log('🎬 Rendering video...');
    await generateVideo(
      contentEN,
      `Top Trends: ${keyword}`,
      './output/video-en.mp4'
    );
    
    await generateVideo(
      contentVI,
      `Top Trends: ${keyword}`,
      './output/video-vi.mp4'
    );
    
    console.log('✅ Pipeline completed!');
    
    return {
      contentEN,
      contentVI,
      videos: ['./output/video-en.mp4', './output/video-vi.mp4']
    };
  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}

// Usage
runContentPipeline('AI Marketing Tools 2026');
```

## API Route Pattern (Next.js)

```typescript
// pages/api/generate-content.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { generateContent } from '@/lib/ai-generator';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }
  
  const { topic, format, language } = req.body;
  
  if (!topic || !format || !language) {
    return res.status(400).json({ error: 'Missing required fields' });
  }
  
  try {
    const content = await generateContent(topic, format, language);
    
    res.status(200).json({
      success: true,
      content,
      metadata: {
        topic,
        format,
        language,
        generatedAt: new Date().toISOString()
      }
    });
  } catch (error) {
    console.error('Generation error:', error);
    res.status(500).json({ error: 'Content generation failed' });
  }
}
```

## Remotion Composition Structure

```typescript
// src/remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig } from 'remotion';

export const ContentVideo: React.FC<{
  title: string;
  content: string;
}> = ({ title, content }) => {
  const frame = useCurrentFrame();
  const { fps, durationInFrames } = useVideoConfig();
  
  const opacity = Math.min(1, frame / 30);
  
  return (
    <AbsoluteFill
      style={{
        backgroundColor: '#1a1a1a',
        justifyContent: 'center',
        alignItems: 'center',
        padding: 60,
      }}
    >
      <div style={{ opacity }}>
        <h1 style={{ color: 'white', fontSize: 64, marginBottom: 40 }}>
          {title}
        </h1>
        <p style={{ color: '#ccc', fontSize: 28, lineHeight: 1.6 }}>
          {content.slice(0, 200)}...
        </p>
      </div>
    </AbsoluteFill>
  );
};
```

## Configuration Patterns

### Content Format Templates

```typescript
// lib/templates.ts
export const contentTemplates = {
  toplist: {
    structure: [
      'Introduction',
      'Item 1 with details',
      'Item 2 with details',
      'Item 3 with details',
      'Conclusion'
    ],
    tone: 'informative'
  },
  pov: {
    structure: [
      'Hook with opinion',
      'Supporting argument 1',
      'Supporting argument 2',
      'Counterargument',
      'Conclusion'
    ],
    tone: 'opinionated'
  },
  'case-study': {
    structure: [
      'Background',
      'Challenge',
      'Solution',
      'Results',
      'Key Takeaways'
    ],
    tone: 'analytical'
  },
  'how-to': {
    structure: [
      'Introduction',
      'Step 1',
      'Step 2',
      'Step 3',
      'Tips & Best Practices'
    ],
    tone: 'instructional'
  }
};
```

## Troubleshooting

### API Rate Limits

```typescript
// lib/rate-limiter.ts
class RateLimiter {
  private requests: number[] = [];
  
  async waitIfNeeded(maxRequests: number, windowMs: number) {
    const now = Date.now();
    this.requests = this.requests.filter(time => now - time < windowMs);
    
    if (this.requests.length >= maxRequests) {
      const oldestRequest = this.requests[0];
      const waitTime = windowMs - (now - oldestRequest);
      await new Promise(resolve => setTimeout(resolve, waitTime));
    }
    
    this.requests.push(now);
  }
}

const limiter = new RateLimiter();

// Usage before API calls
await limiter.waitIfNeeded(5, 60000); // 5 requests per minute
```

### Video Rendering Memory Issues

```typescript
// Render in chunks for long videos
async function renderLongVideo(content: string[], outputPath: string) {
  const chunkSize = 30; // seconds per chunk
  const chunks = [];
  
  for (let i = 0; i < content.length; i++) {
    const chunkPath = `./temp/chunk-${i}.mp4`;
    await generateVideo(content[i], `Part ${i + 1}`, chunkPath);
    chunks.push(chunkPath);
  }
  
  // Merge chunks using ffmpeg or similar
  await mergeVideoChunks(chunks, outputPath);
}
```

### Error Handling for AI APIs

```typescript
async function robustGenerate(prompt: string, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await generateContent(prompt, 'toplist', 'en');
    } catch (error: any) {
      if (error.status === 429) {
        // Rate limit - exponential backoff
        await new Promise(r => setTimeout(r, Math.pow(2, i) * 1000));
        continue;
      }
      throw error;
    }
  }
  throw new Error('Max retries exceeded');
}
```

## Best Practices

1. **Cache Research Data**: Store scraped news to avoid repeated API calls
2. **Queue Video Rendering**: Use a job queue (Bull, BullMQ) for heavy rendering tasks
3. **Monitor API Usage**: Track token consumption for Claude/OpenAI to manage costs
4. **Version Content**: Save generated content with metadata for future reference
5. **Optimize Remotion**: Use `<Sequence>` components to break complex videos into manageable parts
