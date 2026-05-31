---
name: marketing-pipeline-share-automation
description: AI-powered content pipeline for automated research, script writing, video generation, and multi-platform publishing
triggers:
  - automate content creation with AI research
  - set up automated video generation pipeline
  - create content from keyword research to video
  - build AI content automation workflow
  - generate videos from blog posts automatically
  - automate social media content pipeline
  - create multi-format content with AI
  - set up Claude AI content generation
---

# Marketing Pipeline Share - AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to help developers use **Marketing Pipeline Share**, an end-to-end content automation system that transforms keywords into fully-realized content: research → script → blog post → video. Built with TypeScript, Next.js, and integrates Claude AI, OpenAI, and Remotion for video rendering.

## What This Project Does

Marketing Pipeline Share is a complete content production pipeline that:

- **Auto-scans research sources** (TechCrunch, a16z, Twitter, LinkedIn) for latest news and trends
- **Generates multi-format content** using Claude/OpenAI (toplist, POV, case study, how-to)
- **Creates bilingual content** (English & Vietnamese) with customizable tone
- **Renders videos automatically** using Remotion for social media platforms
- **Exports platform-optimized content** for Reels, TikTok, Shorts

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

Create a `.env.local` file in the project root:

```bash
# AI Provider Keys
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research API (RapidAPI)
RAPIDAPI_KEY=your_rapidapi_key

# Next.js Configuration
NEXT_PUBLIC_API_URL=http://localhost:3000

# Optional: Database (if using)
DATABASE_URL=your_database_connection_string

# Remotion Video Rendering
REMOTION_AWS_ACCESS_KEY_ID=your_aws_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret
```

## Running the Application

```bash
# Development mode
npm run dev

# Production build
npm run build
npm run start

# Video rendering with Remotion
npm run render
```

## Core API Structure

### 1. Research Pipeline

```typescript
// app/api/research/route.ts
import Anthropic from '@anthropic-ai/sdk';

export async function POST(request: Request) {
  const { keyword, sources } = await request.json();
  
  const anthropic = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY,
  });

  // Crawl news sources
  const researchData = await fetchLatestNews(keyword, sources);
  
  // Use Claude to analyze and extract insights
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [{
      role: 'user',
      content: `Analyze these research findings and extract key insights for content creation: ${JSON.stringify(researchData)}`
    }]
  });

  return Response.json({
    insights: message.content,
    rawData: researchData
  });
}
```

### 2. Content Generation

```typescript
// lib/content-generator.ts
import OpenAI from 'openai';

interface ContentConfig {
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  tone: 'expert' | 'friendly' | 'humorous';
  language: 'en' | 'vi';
  insights: string;
}

export async function generateContent(config: ContentConfig) {
  const openai = new OpenAI({
    apiKey: process.env.OPENAI_API_KEY,
  });

  const systemPrompt = buildSystemPrompt(config.format, config.tone, config.language);
  
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      { role: 'system', content: systemPrompt },
      { role: 'user', content: `Create content based on: ${config.insights}` }
    ],
    temperature: 0.7,
  });

  return {
    content: completion.choices[0].message.content,
    metadata: {
      format: config.format,
      language: config.language,
      wordCount: completion.choices[0].message.content?.split(' ').length
    }
  };
}

function buildSystemPrompt(format: string, tone: string, language: string): string {
  const prompts = {
    'toplist': 'You are an expert at creating engaging top-list articles...',
    'pov': 'You create thought-provoking point-of-view pieces...',
    'case-study': 'You write detailed, data-driven case studies...',
    'how-to': 'You create clear, actionable how-to guides...'
  };
  
  const toneAdjustments = {
    'expert': 'Use professional, authoritative language.',
    'friendly': 'Use conversational, approachable tone.',
    'humorous': 'Incorporate light humor and wit.'
  };

  return `${prompts[format]} ${toneAdjustments[tone]} Write in ${language === 'vi' ? 'Vietnamese' : 'English'}.`;
}
```

### 3. Video Generation with Remotion

```typescript
// remotion/compositions/ContentVideo.tsx
import { Composition } from 'remotion';
import { VideoTemplate } from './VideoTemplate';

export const RemotionRoot = () => {
  return (
    <>
      <Composition
        id="ContentVideo"
        component={VideoTemplate}
        durationInFrames={900} // 30 seconds at 30fps
        fps={30}
        width={1080}
        height={1920} // Vertical for Reels/TikTok
        defaultProps={{
          title: 'Sample Title',
          points: [],
          bgColor: '#000000'
        }}
      />
    </>
  );
};
```

```typescript
// remotion/compositions/VideoTemplate.tsx
import { AbsoluteFill, useCurrentFrame, interpolate, Sequence } from 'remotion';

interface VideoProps {
  title: string;
  points: string[];
  bgColor: string;
}

export const VideoTemplate: React.FC<VideoProps> = ({ title, points, bgColor }) => {
  const frame = useCurrentFrame();
  
  const titleOpacity = interpolate(frame, [0, 30], [0, 1], {
    extrapolateRight: 'clamp',
  });

  return (
    <AbsoluteFill style={{ backgroundColor: bgColor }}>
      <Sequence from={0} durationInFrames={90}>
        <div style={{ 
          fontSize: 60, 
          textAlign: 'center',
          opacity: titleOpacity,
          color: 'white',
          padding: 40
        }}>
          {title}
        </div>
      </Sequence>
      
      {points.map((point, index) => (
        <Sequence key={index} from={90 + (index * 90)} durationInFrames={90}>
          <div style={{
            fontSize: 40,
            color: 'white',
            padding: 40,
            opacity: interpolate(frame, [90 + (index * 90), 120 + (index * 90)], [0, 1])
          }}>
            {point}
          </div>
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};
```

### 4. Complete Pipeline Orchestration

```typescript
// app/api/pipeline/route.ts
export async function POST(request: Request) {
  const { keyword, format, language, tone } = await request.json();

  try {
    // Step 1: Research
    const research = await fetch('/api/research', {
      method: 'POST',
      body: JSON.stringify({ keyword, sources: ['techcrunch', 'a16z', 'twitter'] })
    }).then(r => r.json());

    // Step 2: Generate Content
    const content = await generateContent({
      format,
      tone,
      language,
      insights: research.insights
    });

    // Step 3: Extract video points
    const videoPoints = extractKeyPoints(content.content);

    // Step 4: Render video
    const videoId = await renderVideo({
      title: keyword,
      points: videoPoints,
      bgColor: '#1a1a1a'
    });

    return Response.json({
      success: true,
      content: content.content,
      videoId,
      metadata: content.metadata
    });
  } catch (error) {
    return Response.json({ error: error.message }, { status: 500 });
  }
}
```

### 5. News Crawling Implementation

```typescript
// lib/news-crawler.ts
interface NewsSource {
  name: string;
  url: string;
  apiEndpoint?: string;
}

export async function fetchLatestNews(keyword: string, sources: string[]) {
  const rapidApiKey = process.env.RAPIDAPI_KEY;
  
  const results = await Promise.all(
    sources.map(async (source) => {
      if (source === 'techcrunch') {
        return fetchTechCrunch(keyword, rapidApiKey);
      } else if (source === 'twitter') {
        return fetchTwitter(keyword, rapidApiKey);
      }
      // Add more sources as needed
    })
  );

  return results.flat().slice(0, 10); // Top 10 articles
}

async function fetchTechCrunch(keyword: string, apiKey: string) {
  const response = await fetch(
    `https://techcrunch-articles.p.rapidapi.com/search?query=${encodeURIComponent(keyword)}`,
    {
      headers: {
        'X-RapidAPI-Key': apiKey,
        'X-RapidAPI-Host': 'techcrunch-articles.p.rapidapi.com'
      }
    }
  );
  
  return response.json();
}
```

## Common Usage Patterns

### Pattern 1: Single Keyword to Full Content

```typescript
// Client-side component
'use client';

import { useState } from 'react';

export default function ContentPipeline() {
  const [keyword, setKeyword] = useState('');
  const [result, setResult] = useState(null);

  const runPipeline = async () => {
    const response = await fetch('/api/pipeline', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        keyword,
        format: 'toplist',
        language: 'vi',
        tone: 'friendly'
      })
    });
    
    const data = await response.json();
    setResult(data);
  };

  return (
    <div>
      <input 
        value={keyword} 
        onChange={(e) => setKeyword(e.target.value)}
        placeholder="Enter keyword..."
      />
      <button onClick={runPipeline}>Generate Content</button>
      {result && (
        <div>
          <h2>Generated Content</h2>
          <p>{result.content}</p>
          <video src={`/api/video/${result.videoId}`} controls />
        </div>
      )}
    </div>
  );
}
```

### Pattern 2: Batch Content Generation

```typescript
// lib/batch-generator.ts
export async function generateBatchContent(keywords: string[]) {
  const results = [];
  
  for (const keyword of keywords) {
    const content = await fetch('/api/pipeline', {
      method: 'POST',
      body: JSON.stringify({
        keyword,
        format: 'how-to',
        language: 'en',
        tone: 'expert'
      })
    }).then(r => r.json());
    
    results.push(content);
    
    // Rate limiting
    await new Promise(resolve => setTimeout(resolve, 2000));
  }
  
  return results;
}
```

### Pattern 3: Custom Video Rendering

```typescript
// app/api/render-video/route.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

export async function POST(request: Request) {
  const { title, points, aspectRatio } = await request.json();
  
  const compositionId = 'ContentVideo';
  const bundleLocation = await bundle(path.join(process.cwd(), 'remotion/index.ts'));
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
  });

  const outputLocation = path.join(process.cwd(), `public/videos/${Date.now()}.mp4`);
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: {
      title,
      points,
      bgColor: '#000000'
    },
  });

  return Response.json({ videoPath: outputLocation });
}
```

## Troubleshooting

### API Rate Limits

If you encounter rate limiting:

```typescript
// lib/rate-limiter.ts
class RateLimiter {
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
    const fn = this.queue.shift()!;
    await fn();
    await new Promise(resolve => setTimeout(resolve, 1000)); // 1 second delay
    this.processing = false;
    this.process();
  }
}

export const limiter = new RateLimiter();
```

### Video Rendering Memory Issues

For large videos, use server-side rendering with proper memory allocation:

```bash
# Increase Node.js memory
NODE_OPTIONS=--max-old-space-size=4096 npm run render
```

### Claude API Timeouts

Handle long-running Claude requests:

```typescript
const message = await anthropic.messages.create({
  model: 'claude-3-5-sonnet-20241022',
  max_tokens: 4096,
  timeout: 120000, // 2 minutes
  messages: [/* ... */]
}).catch(error => {
  if (error.message.includes('timeout')) {
    // Retry with smaller chunk
    return retryWithSmallerChunk();
  }
  throw error;
});
```

## Key Files Structure

```
marketing-pineline-share/
├── app/
│   ├── api/
│   │   ├── research/route.ts      # News crawling endpoint
│   │   ├── content/route.ts       # Content generation
│   │   ├── pipeline/route.ts      # Full pipeline orchestration
│   │   └── render-video/route.ts  # Video rendering
│   └── page.tsx                   # Main UI
├── lib/
│   ├── content-generator.ts       # AI content generation logic
│   ├── news-crawler.ts            # Multi-source news fetching
│   └── rate-limiter.ts            # API rate limiting
├── remotion/
│   ├── index.ts                   # Remotion entry
│   └── compositions/
│       └── VideoTemplate.tsx      # Video component
└── .env.local                     # Environment variables
```

This skill provides AI agents with comprehensive knowledge to help developers implement automated content pipelines with research, AI writing, and video generation capabilities.
