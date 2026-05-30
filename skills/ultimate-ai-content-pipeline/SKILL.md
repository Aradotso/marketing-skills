---
name: ultimate-ai-content-pipeline
description: Automated content pipeline from research to video generation using Claude/OpenAI and Remotion for multi-platform content creation
triggers:
  - "set up AI content automation pipeline"
  - "automate content research and video generation"
  - "create automated marketing content workflow"
  - "generate videos from text with Remotion"
  - "scrape news and create content automatically"
  - "build content pipeline with Claude and OpenAI"
  - "automate social media content creation"
  - "research and generate multi-format content"
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

A complete TypeScript-based content automation system that handles research, script writing, and video generation. The pipeline automatically crawls news sources, generates content in multiple formats using AI (Claude 3, OpenAI), and renders videos with Remotion.

## What This Project Does

Ultimate AI Content Pipeline is an end-to-end content automation system that:
- **Auto-scans research sources**: Crawls TechCrunch, a16z, Twitter/X, LinkedIn for latest news (24h)
- **Generates multi-format content**: Creates Toplists, POVs, Case Studies, How-tos in Vietnamese & English
- **Renders videos automatically**: Converts content to infographics and short-form videos for Reels/TikTok/Shorts
- **Supports multiple AI providers**: Claude 3 (Anthropic) and OpenAI integration
- **Built on Next.js**: Modern web interface for content creation and scheduling

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
# AI Providers
ANTHROPIC_API_KEY=your_anthropic_key
OPENAI_API_KEY=your_openai_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Next.js
NEXT_PUBLIC_API_URL=http://localhost:3000

# Optional: Database (if using)
DATABASE_URL=your_database_url
```

## Running the Application

```bash
# Development mode
npm run dev

# Production build
npm run build
npm run start

# Remotion video rendering (if separate)
npm run remotion:render
```

## Project Structure

```
marketing-pineline-share/
├── app/                    # Next.js 13+ app directory
├── components/             # React components
├── lib/
│   ├── ai/                # AI provider integrations
│   │   ├── claude.ts
│   │   └── openai.ts
│   ├── research/          # Content research modules
│   │   ├── crawler.ts
│   │   └── analyzer.ts
│   └── video/             # Remotion video generation
├── remotion/              # Remotion compositions
├── public/                # Static assets
└── types/                 # TypeScript definitions
```

## Core API Usage

### 1. Research & Data Crawling

```typescript
// lib/research/crawler.ts
import axios from 'axios';

interface NewsSource {
  url: string;
  category: string;
  lastCrawled?: Date;
}

export async function crawlNewsSource(source: NewsSource) {
  const config = {
    method: 'get',
    url: source.url,
    headers: {
      'X-RapidAPI-Key': process.env.RAPIDAPI_KEY,
      'X-RapidAPI-Host': 'news-api.rapidapi.com'
    }
  };
  
  try {
    const response = await axios.request(config);
    return response.data;
  } catch (error) {
    console.error(`Error crawling ${source.url}:`, error);
    throw error;
  }
}

export async function crawlMultipleSources(sources: NewsSource[]) {
  const results = await Promise.allSettled(
    sources.map(source => crawlNewsSource(source))
  );
  
  return results
    .filter(result => result.status === 'fulfilled')
    .map(result => (result as PromiseFulfilledResult<any>).value);
}
```

### 2. AI Content Generation with Claude

```typescript
// lib/ai/claude.ts
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

interface ContentRequest {
  topic: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  researchData?: any[];
}

export async function generateContent(request: ContentRequest) {
  const systemPrompt = buildSystemPrompt(request);
  const userPrompt = buildUserPrompt(request);
  
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    system: systemPrompt,
    messages: [
      {
        role: 'user',
        content: userPrompt
      }
    ]
  });
  
  return message.content[0].type === 'text' 
    ? message.content[0].text 
    : '';
}

function buildSystemPrompt(request: ContentRequest): string {
  const toneMap = {
    expert: 'professional and authoritative',
    friendly: 'conversational and approachable',
    humorous: 'witty and entertaining'
  };
  
  return `You are a ${toneMap[request.tone]} content writer specializing in ${request.format} articles. 
Write in ${request.language === 'vi' ? 'Vietnamese' : 'English'}.
Use data-driven insights and always cite sources when using research data.`;
}

function buildUserPrompt(request: ContentRequest): string {
  let prompt = `Create a ${request.format} article about: ${request.topic}\n\n`;
  
  if (request.researchData && request.researchData.length > 0) {
    prompt += 'Use the following research data:\n';
    prompt += JSON.stringify(request.researchData, null, 2);
  }
  
  return prompt;
}
```

### 3. AI Content Generation with OpenAI

```typescript
// lib/ai/openai.ts
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

export async function generateContentOpenAI(request: ContentRequest) {
  const systemPrompt = buildSystemPrompt(request);
  const userPrompt = buildUserPrompt(request);
  
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      { role: 'system', content: systemPrompt },
      { role: 'user', content: userPrompt }
    ],
    temperature: 0.7,
    max_tokens: 4096
  });
  
  return completion.choices[0].message.content || '';
}

export async function generateMultipleVariations(
  request: ContentRequest,
  count: number = 3
) {
  const promises = Array(count).fill(null).map(() => 
    generateContentOpenAI(request)
  );
  
  return await Promise.all(promises);
}
```

### 4. Video Generation with Remotion

```typescript
// lib/video/generator.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  title: string;
  content: string[];
  style: 'minimal' | 'dynamic' | 'corporate';
  duration: number; // in seconds
}

export async function generateVideo(config: VideoConfig) {
  const compositionId = 'ContentVideo';
  const bundleLocation = await bundle(
    path.join(process.cwd(), 'remotion/index.ts')
  );
  
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: {
      title: config.title,
      content: config.content,
      style: config.style
    }
  });
  
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
    inputProps: {
      title: config.title,
      content: config.content,
      style: config.style
    }
  });
  
  return outputLocation;
}
```

### 5. Remotion Composition Example

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig, Sequence } from 'remotion';
import React from 'react';

interface ContentVideoProps {
  title: string;
  content: string[];
  style: 'minimal' | 'dynamic' | 'corporate';
}

export const ContentVideo: React.FC<ContentVideoProps> = ({
  title,
  content,
  style
}) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  
  const opacity = Math.min(1, frame / (fps * 0.5));
  
  return (
    <AbsoluteFill style={{ backgroundColor: '#000' }}>
      <Sequence from={0} durationInFrames={fps * 2}>
        <AbsoluteFill
          style={{
            justifyContent: 'center',
            alignItems: 'center',
            opacity
          }}
        >
          <h1 style={{ color: '#fff', fontSize: 60 }}>{title}</h1>
        </AbsoluteFill>
      </Sequence>
      
      {content.map((text, index) => (
        <Sequence
          key={index}
          from={fps * (2 + index * 3)}
          durationInFrames={fps * 3}
        >
          <AbsoluteFill
            style={{
              justifyContent: 'center',
              alignItems: 'center',
              padding: 40
            }}
          >
            <p style={{ 
              color: '#fff', 
              fontSize: 32,
              textAlign: 'center',
              maxWidth: '80%'
            }}>
              {text}
            </p>
          </AbsoluteFill>
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};
```

## Complete Pipeline Example

```typescript
// app/api/pipeline/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { crawlMultipleSources } from '@/lib/research/crawler';
import { generateContent } from '@/lib/ai/claude';
import { generateVideo } from '@/lib/video/generator';

export async function POST(request: NextRequest) {
  try {
    const { topic, format, language, tone } = await request.json();
    
    // Step 1: Research
    const sources = [
      { url: 'https://techcrunch.com/feed', category: 'tech' },
      { url: 'https://a16z.com/feed', category: 'startup' }
    ];
    const researchData = await crawlMultipleSources(sources);
    
    // Step 2: Generate content
    const content = await generateContent({
      topic,
      format,
      language,
      tone,
      researchData
    });
    
    // Step 3: Parse content for video
    const contentLines = content.split('\n').filter(line => line.trim());
    const videoTitle = contentLines[0];
    const videoContent = contentLines.slice(1, 6); // First 5 points
    
    // Step 4: Generate video
    const videoPath = await generateVideo({
      title: videoTitle,
      content: videoContent,
      style: 'dynamic',
      duration: 30
    });
    
    return NextResponse.json({
      success: true,
      content,
      videoUrl: `/videos/${path.basename(videoPath)}`
    });
    
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { error: 'Pipeline failed' },
      { status: 500 }
    );
  }
}
```

## Common Patterns

### Multi-Language Content Generation

```typescript
// lib/ai/multi-language.ts
export async function generateBilingualContent(
  topic: string,
  format: string
) {
  const [englishContent, vietnameseContent] = await Promise.all([
    generateContent({ topic, format, language: 'en', tone: 'expert' }),
    generateContent({ topic, format, language: 'vi', tone: 'expert' })
  ]);
  
  return {
    en: englishContent,
    vi: vietnameseContent
  };
}
```

### Batch Video Generation

```typescript
// lib/video/batch.ts
export async function generateMultipleVideos(
  contents: Array<{ title: string; content: string[] }>
) {
  const videoPromises = contents.map(content =>
    generateVideo({
      ...content,
      style: 'dynamic',
      duration: 30
    })
  );
  
  return await Promise.allSettled(videoPromises);
}
```

### Content Scheduling

```typescript
// lib/scheduler/schedule.ts
interface ScheduledPost {
  content: string;
  videoUrl: string;
  publishAt: Date;
  platform: 'facebook' | 'instagram' | 'tiktok';
}

export async function schedulePost(post: ScheduledPost) {
  // Save to database
  const scheduled = await db.scheduledPosts.create({
    data: {
      content: post.content,
      videoUrl: post.videoUrl,
      publishAt: post.publishAt,
      platform: post.platform,
      status: 'pending'
    }
  });
  
  return scheduled;
}
```

## Troubleshooting

### API Rate Limits

If you encounter rate limits with AI providers:

```typescript
// lib/ai/rate-limiter.ts
import pLimit from 'p-limit';

const limit = pLimit(3); // Max 3 concurrent requests

export async function generateContentWithRateLimit(requests: ContentRequest[]) {
  const results = await Promise.all(
    requests.map(request =>
      limit(() => generateContent(request))
    )
  );
  
  return results;
}
```

### Video Rendering Memory Issues

For large video renders:

```bash
# Increase Node.js memory limit
NODE_OPTIONS="--max-old-space-size=4096" npm run remotion:render
```

Or in code:

```typescript
// lib/video/generator.ts
await renderMedia({
  composition,
  serveUrl: bundleLocation,
  codec: 'h264',
  outputLocation,
  chromiumOptions: {
    headless: true,
    args: ['--no-sandbox', '--disable-setuid-sandbox']
  },
  // Reduce quality for memory efficiency
  videoBitrate: '2M'
});
```

### Research Data Quality

Filter and validate crawled data:

```typescript
// lib/research/validator.ts
export function validateResearchData(data: any[]) {
  return data.filter(item => {
    return (
      item.title &&
      item.content &&
      item.publishedAt &&
      new Date(item.publishedAt) > new Date(Date.now() - 24 * 60 * 60 * 1000)
    );
  });
}
```

### Remotion Composition Not Found

Ensure your Remotion index exports compositions:

```typescript
// remotion/index.ts
import { registerRoot } from 'remotion';
import { ContentVideo } from './ContentVideo';

registerRoot(() => {
  return (
    <>
      <Composition
        id="ContentVideo"
        component={ContentVideo}
        durationInFrames={900}
        fps={30}
        width={1080}
        height={1920}
        defaultProps={{
          title: 'Default Title',
          content: [],
          style: 'minimal'
        }}
      />
    </>
  );
});
```

## Key Commands

```bash
# Development
npm run dev                    # Start Next.js dev server

# Build
npm run build                  # Build for production
npm run start                  # Start production server

# Remotion
npm run remotion:preview       # Preview Remotion compositions
npm run remotion:render        # Render videos

# Testing
npm run test                   # Run tests
npm run test:watch            # Watch mode
```
