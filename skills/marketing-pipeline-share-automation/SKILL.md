---
name: marketing-pipeline-share-automation
description: AI-powered content automation pipeline for research, scripting, and video generation using Claude/OpenAI and Remotion
triggers:
  - automate content creation with AI research and video generation
  - set up marketing content pipeline with auto-research
  - generate videos from content using Remotion
  - crawl news sources and create content automatically
  - build AI content workflow from research to video
  - configure content automation with Claude and OpenAI
  - create multilingual content pipeline with video rendering
  - automate social media content with AI and video generation
---

# Marketing Pipeline Share - AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

A complete TypeScript-based content automation system that handles research (crawling news sources), content generation (using Claude/OpenAI), and video rendering (Remotion). This pipeline transforms a single keyword into fully-formatted articles and videos ready for multi-platform distribution.

## What This Project Does

Marketing Pipeline Share is an all-in-one content factory that:
- **Auto-scans research**: Crawls recent content from TechCrunch, a16z, Twitter/X, LinkedIn
- **Generates diverse formats**: Creates toplist, POV, case study, and how-to content
- **Multilingual output**: Produces parallel English and Vietnamese versions
- **Renders videos**: Automatically creates infographics and short-form videos from content
- **Platform-optimized**: Exports video formats for Reels, TikTok, Shorts

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
# AI Provider Keys
OPENAI_API_KEY=your_openai_key_here
ANTHROPIC_API_KEY=your_claude_key_here

# Research/Crawling APIs
RAPIDAPI_KEY=your_rapidapi_key_here

# Optional: Database (if using persistence)
DATABASE_URL=your_database_url_here

# Remotion (Video Rendering)
REMOTION_LICENSE_KEY=your_remotion_license_key_here
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app router pages
│   ├── components/       # React components for UI
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── crawlers/    # Web scraping modules
│   │   ├── generators/  # Content generation logic
│   │   └── video/       # Remotion video rendering
│   ├── remotion/        # Remotion compositions
│   └── types/           # TypeScript type definitions
├── public/              # Static assets
└── package.json
```

## Core API Usage

### 1. Research & Crawling

```typescript
import { crawlNewsSource } from '@/lib/crawlers/news';
import { analyzeContent } from '@/lib/ai/analyzer';

// Crawl recent articles from a source
async function gatherResearch(keyword: string) {
  const sources = ['techcrunch', 'a16z', 'twitter'];
  
  const articles = await Promise.all(
    sources.map(source => 
      crawlNewsSource({
        source,
        keyword,
        timeRange: '24h',
        limit: 10
      })
    )
  );

  // Flatten and analyze
  const allArticles = articles.flat();
  const insights = await analyzeContent(allArticles, {
    provider: 'claude', // or 'openai'
    model: 'claude-3-opus-20240229'
  });

  return insights;
}
```

### 2. Content Generation

```typescript
import { generateContent } from '@/lib/generators/content';

// Generate multilingual content in specific format
async function createContent(
  keyword: string,
  researchData: any
) {
  const content = await generateContent({
    keyword,
    research: researchData,
    format: 'toplist', // 'pov' | 'case-study' | 'how-to'
    languages: ['en', 'vi'],
    tone: 'professional', // 'friendly' | 'humorous'
    aiProvider: 'claude',
    aiModel: process.env.CLAUDE_MODEL || 'claude-3-sonnet-20240229'
  });

  return content;
  // Returns: { en: { title, body, metadata }, vi: { title, body, metadata } }
}
```

### 3. Video Rendering with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

// Render video from content
async function renderContentVideo(
  content: any,
  platform: 'reels' | 'tiktok' | 'shorts'
) {
  const compositionId = 'ContentVideo';
  
  // Bundle Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.join(process.cwd(), 'src/remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  // Select composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: {
      content: content,
      platform: platform,
    },
  });

  // Render video
  const outputLocation = path.join(
    process.cwd(),
    'public/videos',
    `${content.id}-${platform}.mp4`
  );

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: {
      content,
      platform,
    },
  });

  return outputLocation;
}
```

### 4. Complete Pipeline

```typescript
import { ContentPipeline } from '@/lib/pipeline';

// Full automation from keyword to video
async function runFullPipeline(keyword: string) {
  const pipeline = new ContentPipeline({
    openaiKey: process.env.OPENAI_API_KEY!,
    claudeKey: process.env.ANTHROPIC_API_KEY!,
    rapidApiKey: process.env.RAPIDAPI_KEY!,
  });

  // Execute complete workflow
  const result = await pipeline.execute({
    keyword,
    formats: ['toplist', 'how-to'],
    languages: ['en', 'vi'],
    platforms: ['reels', 'tiktok', 'shorts'],
    renderVideo: true,
  });

  return result;
  /*
  Returns:
  {
    research: { sources: [...], insights: [...] },
    content: {
      toplist: { en: {...}, vi: {...} },
      howTo: { en: {...}, vi: {...} }
    },
    videos: {
      reels: 'path/to/video.mp4',
      tiktok: 'path/to/video.mp4',
      shorts: 'path/to/video.mp4'
    }
  }
  */
}
```

## Next.js API Routes

### Create Content Endpoint

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { ContentPipeline } from '@/lib/pipeline';

export async function POST(request: NextRequest) {
  try {
    const { keyword, format, languages, platform } = await request.json();

    const pipeline = new ContentPipeline({
      openaiKey: process.env.OPENAI_API_KEY!,
      claudeKey: process.env.ANTHROPIC_API_KEY!,
      rapidApiKey: process.env.RAPIDAPI_KEY!,
    });

    const result = await pipeline.execute({
      keyword,
      formats: [format],
      languages,
      platforms: [platform],
      renderVideo: true,
    });

    return NextResponse.json(result);
  } catch (error) {
    return NextResponse.json(
      { error: 'Content generation failed' },
      { status: 500 }
    );
  }
}
```

### Frontend Usage

```typescript
// src/components/ContentGenerator.tsx
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
          format: 'toplist',
          languages: ['en', 'vi'],
          platform: 'reels',
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
        value={keyword}
        onChange={(e) => setKeyword(e.target.value)}
        placeholder="Enter keyword..."
      />
      <button onClick={handleGenerate} disabled={loading}>
        {loading ? 'Generating...' : 'Generate Content'}
      </button>
      {result && <pre>{JSON.stringify(result, null, 2)}</pre>}
    </div>
  );
}
```

## Remotion Composition Example

```typescript
// src/remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, interpolate } from 'remotion';

export const ContentVideo: React.FC<{
  content: any;
  platform: string;
}> = ({ content, platform }) => {
  const frame = useCurrentFrame();
  
  const opacity = interpolate(frame, [0, 30], [0, 1], {
    extrapolateRight: 'clamp',
  });

  // Platform-specific dimensions
  const dimensions = {
    reels: { width: 1080, height: 1920 },
    tiktok: { width: 1080, height: 1920 },
    shorts: { width: 1080, height: 1920 },
  };

  return (
    <AbsoluteFill style={{ backgroundColor: '#000' }}>
      <div style={{ opacity, padding: 40, color: '#fff' }}>
        <h1 style={{ fontSize: 48, marginBottom: 20 }}>
          {content.title}
        </h1>
        <p style={{ fontSize: 24, lineHeight: 1.6 }}>
          {content.body.substring(0, 200)}...
        </p>
      </div>
    </AbsoluteFill>
  );
};
```

```typescript
// src/remotion/index.ts
import { registerRoot } from 'remotion';
import { ContentVideo } from './ContentVideo';

registerRoot(() => {
  return (
    <>
      <Composition
        id="ContentVideo"
        component={ContentVideo}
        durationInFrames={150}
        fps={30}
        width={1080}
        height={1920}
        defaultProps={{
          content: {},
          platform: 'reels',
        }}
      />
    </>
  );
});
```

## Running the Application

```bash
# Development mode
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Render Remotion videos (if standalone)
npm run remotion:render
```

## Common Patterns

### Custom AI Provider Integration

```typescript
// src/lib/ai/custom-provider.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

export class AIProvider {
  private claude: Anthropic;
  private openai: OpenAI;

  constructor(claudeKey: string, openaiKey: string) {
    this.claude = new Anthropic({ apiKey: claudeKey });
    this.openai = new OpenAI({ apiKey: openaiKey });
  }

  async generate(
    prompt: string,
    provider: 'claude' | 'openai' = 'claude'
  ): Promise<string> {
    if (provider === 'claude') {
      const response = await this.claude.messages.create({
        model: 'claude-3-sonnet-20240229',
        max_tokens: 4096,
        messages: [{ role: 'user', content: prompt }],
      });
      return response.content[0].text;
    } else {
      const response = await this.openai.chat.completions.create({
        model: 'gpt-4-turbo-preview',
        messages: [{ role: 'user', content: prompt }],
      });
      return response.choices[0].message.content || '';
    }
  }
}
```

### Batch Processing

```typescript
// Process multiple keywords in parallel
async function batchGenerate(keywords: string[]) {
  const pipeline = new ContentPipeline({
    openaiKey: process.env.OPENAI_API_KEY!,
    claudeKey: process.env.ANTHROPIC_API_KEY!,
    rapidApiKey: process.env.RAPIDAPI_KEY!,
  });

  const results = await Promise.allSettled(
    keywords.map(keyword =>
      pipeline.execute({
        keyword,
        formats: ['toplist'],
        languages: ['en'],
        platforms: ['reels'],
        renderVideo: true,
      })
    )
  );

  return results.map((result, index) => ({
    keyword: keywords[index],
    status: result.status,
    data: result.status === 'fulfilled' ? result.value : null,
    error: result.status === 'rejected' ? result.reason : null,
  }));
}
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement retry logic with exponential backoff
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
        await new Promise(resolve => setTimeout(resolve, delay));
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
// Reduce video quality for memory-constrained environments
await renderMedia({
  composition,
  serveUrl: bundleLocation,
  codec: 'h264',
  outputLocation,
  inputProps: { content, platform },
  // Memory optimization
  concurrency: 1,
  chromiumOptions: {
    args: ['--no-sandbox', '--disable-setuid-sandbox'],
  },
  videoBitrate: '1M', // Lower bitrate
});
```

### Missing Environment Variables

```typescript
// Validate environment at startup
function validateEnv() {
  const required = [
    'OPENAI_API_KEY',
    'ANTHROPIC_API_KEY',
    'RAPIDAPI_KEY',
  ];

  const missing = required.filter(key => !process.env[key]);

  if (missing.length > 0) {
    throw new Error(
      `Missing required environment variables: ${missing.join(', ')}`
    );
  }
}

// Call at app initialization
validateEnv();
```
