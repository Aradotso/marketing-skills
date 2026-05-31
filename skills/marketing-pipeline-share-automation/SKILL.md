---
name: marketing-pipeline-share-automation
description: Automated content pipeline from research to video generation using AI (Claude/OpenAI) and Remotion
triggers:
  - how do I automate content creation with AI research
  - generate video content from text automatically
  - set up marketing content pipeline with Claude
  - crawl news and create social media posts
  - build automated content workflow with Remotion
  - create multilingual content with OpenAI
  - research to video content automation
  - auto-generate marketing content from keywords
---

# Marketing Pipeline Share Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

Marketing Pipeline Share is an end-to-end AI-powered content automation system that transforms a single keyword into fully-formatted articles and videos. The pipeline includes:

1. **Auto-Research**: Crawls news from TechCrunch, a16z, Twitter, LinkedIn
2. **AI Content Generation**: Creates articles in multiple formats (toplist, POV, case study, how-to) using Claude 3 or OpenAI
3. **Multilingual Output**: Generates English and Vietnamese versions
4. **Video Rendering**: Automatically creates videos and infographics using Remotion

The system is built with TypeScript, Next.js, and integrates with Anthropic (Claude), OpenAI, and RapidAPI services.

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

### Required Environment Variables

```bash
# .env.local
OPENAI_API_KEY=your_openai_key_here
ANTHROPIC_API_KEY=your_claude_key_here
RAPIDAPI_KEY=your_rapidapi_key_here
NEXT_PUBLIC_API_URL=http://localhost:3000
```

### Run Development Server

```bash
npm run dev
# or
yarn dev
```

Access the application at `http://localhost:3000`

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/             # Core utilities
│   │   ├── ai/          # AI service integrations
│   │   ├── crawlers/    # News crawling modules
│   │   └── video/       # Remotion video generation
│   └── types/           # TypeScript definitions
├── public/              # Static assets
└── remotion/            # Remotion video templates
```

## Core API Usage

### 1. Research & Crawling Module

```typescript
// src/lib/crawlers/newsCollector.ts
import { TechCrunchCrawler } from './techcrunch';
import { TwitterCrawler } from './twitter';

interface NewsItem {
  title: string;
  url: string;
  summary: string;
  publishedAt: Date;
  source: string;
}

export async function collectNews(keyword: string): Promise<NewsItem[]> {
  const techCrunchNews = await TechCrunchCrawler.search(keyword);
  const twitterPosts = await TwitterCrawler.searchRecent(keyword);
  
  return [...techCrunchNews, ...twitterPosts].sort(
    (a, b) => b.publishedAt.getTime() - a.publishedAt.getTime()
  );
}
```

### 2. AI Content Generation

```typescript
// src/lib/ai/contentGenerator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

export type ContentFormat = 'toplist' | 'pov' | 'case-study' | 'how-to';
export type ToneStyle = 'expert' | 'friendly' | 'humorous';

interface GenerationConfig {
  format: ContentFormat;
  tone: ToneStyle;
  language: 'en' | 'vi';
  researchData: NewsItem[];
}

export async function generateContent(
  keyword: string,
  config: GenerationConfig
): Promise<string> {
  const anthropic = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY,
  });

  const prompt = buildPrompt(keyword, config);

  const message = await anthropic.messages.create({
    model: 'claude-3-opus-20240229',
    max_tokens: 4096,
    messages: [
      {
        role: 'user',
        content: prompt,
      },
    ],
  });

  return message.content[0].type === 'text' 
    ? message.content[0].text 
    : '';
}

function buildPrompt(keyword: string, config: GenerationConfig): string {
  const researchContext = config.researchData
    .map(item => `- ${item.title} (${item.source}): ${item.summary}`)
    .join('\n');

  return `Create a ${config.format} article about "${keyword}" in ${config.language}.
Tone: ${config.tone}

Research data (last 24h):
${researchContext}

Requirements:
- Use data-backed insights
- Include specific examples from research
- Optimize for social media engagement
- ${config.language === 'vi' ? 'Write in Vietnamese' : 'Write in English'}`;
}
```

### 3. OpenAI Alternative

```typescript
// src/lib/ai/openaiGenerator.ts
import OpenAI from 'openai';

export async function generateWithOpenAI(
  keyword: string,
  config: GenerationConfig
): Promise<string> {
  const openai = new OpenAI({
    apiKey: process.env.OPENAI_API_KEY,
  });

  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are an expert content creator specializing in ${config.format} articles with a ${config.tone} tone.`,
      },
      {
        role: 'user',
        content: buildPrompt(keyword, config),
      },
    ],
    temperature: 0.7,
  });

  return completion.choices[0]?.message?.content || '';
}
```

### 4. Video Generation with Remotion

```typescript
// src/lib/video/renderVideo.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  title: string;
  content: string[];
  platform: 'reels' | 'tiktok' | 'shorts';
}

export async function renderContentVideo(
  config: VideoConfig
): Promise<string> {
  const compositionId = 'ContentVideo';
  
  // Bundle the Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.join(process.cwd(), 'remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  // Get composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: {
      title: config.title,
      slides: config.content,
    },
  });

  // Render video
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
  });

  return outputLocation;
}
```

### 5. Remotion Video Component

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, interpolate } from 'remotion';

interface ContentVideoProps {
  title: string;
  slides: string[];
}

export const ContentVideo: React.FC<ContentVideoProps> = ({ title, slides }) => {
  const frame = useCurrentFrame();
  
  const opacity = interpolate(frame, [0, 30], [0, 1], {
    extrapolateRight: 'clamp',
  });

  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a1a' }}>
      <div style={{ opacity, padding: 60 }}>
        <h1 style={{ color: 'white', fontSize: 72, marginBottom: 40 }}>
          {title}
        </h1>
        {slides.map((slide, index) => (
          <div
            key={index}
            style={{
              color: '#e0e0e0',
              fontSize: 36,
              marginBottom: 20,
            }}
          >
            {slide}
          </div>
        ))}
      </div>
    </AbsoluteFill>
  );
};
```

## API Routes

### Content Generation Endpoint

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { collectNews } from '@/lib/crawlers/newsCollector';
import { generateContent } from '@/lib/ai/contentGenerator';

export async function POST(request: NextRequest) {
  try {
    const { keyword, format, tone, language } = await request.json();

    // Step 1: Collect research
    const researchData = await collectNews(keyword);

    // Step 2: Generate content
    const content = await generateContent(keyword, {
      format,
      tone,
      language,
      researchData,
    });

    return NextResponse.json({
      success: true,
      content,
      sources: researchData.length,
    });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

### Video Rendering Endpoint

```typescript
// src/app/api/render-video/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { renderContentVideo } from '@/lib/video/renderVideo';

export async function POST(request: NextRequest) {
  try {
    const { title, content, platform } = await request.json();

    const videoPath = await renderContentVideo({
      title,
      content: content.split('\n').filter(Boolean),
      platform,
    });

    return NextResponse.json({
      success: true,
      videoUrl: `/videos/${path.basename(videoPath)}`,
    });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

## Common Patterns

### Full Pipeline Execution

```typescript
// src/lib/pipeline/fullPipeline.ts
export async function runFullPipeline(keyword: string) {
  // 1. Research phase
  console.log('📡 Collecting research data...');
  const research = await collectNews(keyword);

  // 2. Generate bilingual content
  console.log('🧠 Generating content...');
  const [englishContent, vietnameseContent] = await Promise.all([
    generateContent(keyword, {
      format: 'toplist',
      tone: 'expert',
      language: 'en',
      researchData: research,
    }),
    generateContent(keyword, {
      format: 'toplist',
      tone: 'expert',
      language: 'vi',
      researchData: research,
    }),
  ]);

  // 3. Render video
  console.log('🎬 Rendering video...');
  const videoPath = await renderContentVideo({
    title: keyword,
    content: englishContent.split('\n').slice(0, 5),
    platform: 'reels',
  });

  return {
    englishContent,
    vietnameseContent,
    videoPath,
    sourcesUsed: research.length,
  };
}
```

### Client-Side Usage

```typescript
// src/components/ContentGenerator.tsx
'use client';

import { useState } from 'react';

export default function ContentGenerator() {
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
          format: 'toplist',
          tone: 'expert',
          language: 'en',
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
    <div>
      <input
        type="text"
        value={keyword}
        onChange={(e) => setKeyword(e.target.value)}
        placeholder="Enter keyword..."
      />
      <button onClick={handleGenerate} disabled={loading}>
        {loading ? 'Generating...' : 'Generate Content'}
      </button>
      {result && (
        <div>
          <h3>Generated Content</h3>
          <pre>{result.content}</pre>
          <p>Sources used: {result.sources}</p>
        </div>
      )}
    </div>
  );
}
```

## Configuration

### Remotion Config

```typescript
// remotion.config.ts
import { Config } from '@remotion/cli/config';

Config.setVideoImageFormat('jpeg');
Config.setOverwriteOutput(true);
Config.setConcurrency(2);
Config.setCodec('h264');
```

### Content Formats Configuration

```typescript
// src/config/formats.ts
export const CONTENT_FORMATS = {
  toplist: {
    structure: ['intro', 'items', 'conclusion'],
    minItems: 5,
    maxItems: 10,
  },
  pov: {
    structure: ['hook', 'argument', 'evidence', 'conclusion'],
    personalPerspective: true,
  },
  'case-study': {
    structure: ['background', 'challenge', 'solution', 'results'],
    dataRequired: true,
  },
  'how-to': {
    structure: ['overview', 'steps', 'tips', 'summary'],
    stepByStep: true,
  },
} as const;
```

## Troubleshooting

### API Rate Limits

```typescript
// src/lib/utils/rateLimiter.ts
export class RateLimiter {
  private queue: Array<() => Promise<any>> = [];
  private processing = false;

  async add<T>(fn: () => Promise<T>): Promise<T> {
    return new Promise((resolve, reject) => {
      this.queue.push(async () => {
        try {
          const result = await fn();
          resolve(result);
        } catch (error) {
          reject(error);
        }
      });
      this.process();
    });
  }

  private async process() {
    if (this.processing || this.queue.length === 0) return;
    
    this.processing = true;
    const fn = this.queue.shift();
    
    if (fn) {
      await fn();
      await new Promise(resolve => setTimeout(resolve, 1000)); // 1s delay
    }
    
    this.processing = false;
    this.process();
  }
}
```

### Error Handling for AI APIs

```typescript
// src/lib/ai/errorHandler.ts
export async function withRetry<T>(
  fn: () => Promise<T>,
  maxRetries = 3
): Promise<T> {
  let lastError: Error;
  
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      lastError = error as Error;
      
      if (error.status === 429) {
        // Rate limit - wait longer
        await new Promise(resolve => setTimeout(resolve, 5000 * (i + 1)));
      } else if (error.status >= 500) {
        // Server error - retry
        await new Promise(resolve => setTimeout(resolve, 2000));
      } else {
        // Client error - don't retry
        throw error;
      }
    }
  }
  
  throw lastError;
}
```

### Video Rendering Issues

If video rendering fails, check:

1. **FFmpeg Installation**: Remotion requires FFmpeg
   ```bash
   # Install FFmpeg
   brew install ffmpeg  # macOS
   sudo apt install ffmpeg  # Ubuntu
   ```

2. **Memory Limits**: Increase Node.js memory
   ```bash
   NODE_OPTIONS=--max-old-space-size=4096 npm run dev
   ```

3. **Composition Not Found**: Ensure Remotion entry point is correct
   ```typescript
   // remotion/index.ts
   import { registerRoot } from 'remotion';
   import { ContentVideo } from './ContentVideo';

   registerRoot(() => <ContentVideo />);
   ```
