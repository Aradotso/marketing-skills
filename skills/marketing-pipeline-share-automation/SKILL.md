---
name: marketing-pipeline-share-automation
description: Automated AI content pipeline for research, scriptwriting, posting, and video generation using Claude, OpenAI, and Remotion
triggers:
  - how do i set up the marketing content automation pipeline
  - generate automated content with ai research and video
  - create content pipeline with claude and openai
  - automate social media content from research to video
  - build ai content workflow with remotion video rendering
  - set up automated content generation and posting
  - create multi-format content with ai pipeline
  - automate content research and video creation
---

# Marketing Pipeline Share Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

Marketing Pipeline Share is an end-to-end AI-powered content automation system that handles the complete workflow: research crawling from news sources (TechCrunch, a16z, X/Twitter, LinkedIn), AI-generated content creation in multiple formats, automatic posting, and video generation using Remotion. Built with Next.js and TypeScript, it integrates Claude 3, OpenAI, and RapidAPI to create a fully automated content factory.

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

# Set up environment variables
cp .env.example .env.local
```

### Required Environment Variables

```bash
# AI APIs
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Optional: Social Media APIs
FACEBOOK_PAGE_ACCESS_TOKEN=your_token
LINKEDIN_ACCESS_TOKEN=your_token

# Application
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app router
│   ├── components/       # React components
│   ├── lib/              # Core libraries
│   │   ├── ai/          # AI integrations (Claude, OpenAI)
│   │   ├── crawler/     # News crawling logic
│   │   ├── content/     # Content generation
│   │   └── video/       # Remotion video rendering
│   ├── remotion/        # Video templates
│   └── utils/           # Utilities
├── public/              # Static assets
└── package.json
```

## Core Features & Usage

### 1. Research Crawling

Automatically crawl and analyze recent content from major tech news sources:

```typescript
import { crawlNews } from '@/lib/crawler/news-crawler';
import { analyzeInsights } from '@/lib/ai/insight-analyzer';

async function gatherResearch(keyword: string) {
  // Crawl news from multiple sources
  const newsData = await crawlNews({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeRange: '24h'
  });

  // Extract insights using AI
  const insights = await analyzeInsights(newsData, {
    model: 'claude-3-sonnet',
    extractStats: true,
    findTrends: true
  });

  return {
    rawData: newsData,
    insights: insights,
    timestamp: new Date()
  };
}

// Usage
const research = await gatherResearch('AI automation');
console.log(research.insights);
```

### 2. AI Content Generation

Generate content in multiple formats using Claude or OpenAI:

```typescript
import { generateContent } from '@/lib/content/generator';
import { ContentFormat, Language, Tone } from '@/types/content';

async function createMultiFormatContent(research: any) {
  const formats: ContentFormat[] = ['toplist', 'pov', 'case-study', 'how-to'];
  
  const contentPieces = await Promise.all(
    formats.map(async (format) => {
      return await generateContent({
        research: research.insights,
        format,
        languages: [Language.EN, Language.VI],
        tone: Tone.EXPERT,
        aiProvider: 'claude', // or 'openai'
        model: 'claude-3-sonnet-20240229',
        includeStats: true,
        includeReferences: true
      });
    })
  );

  return contentPieces;
}

// Example output structure
interface ContentPiece {
  title: string;
  content: string;
  format: ContentFormat;
  language: Language;
  metadata: {
    wordCount: number;
    readingTime: number;
    keywords: string[];
    stats: Array<{ label: string; value: string }>;
  };
}
```

### 3. Claude Integration

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

async function generateWithClaude(prompt: string, context: any) {
  const message = await anthropic.messages.create({
    model: 'claude-3-sonnet-20240229',
    max_tokens: 4096,
    messages: [
      {
        role: 'user',
        content: `${prompt}\n\nContext: ${JSON.stringify(context)}`
      }
    ],
    system: 'You are an expert content creator specializing in marketing and tech trends.'
  });

  return message.content[0].type === 'text' 
    ? message.content[0].text 
    : '';
}

// Generate bilingual content
async function generateBilingualPost(topic: string, insights: any) {
  const englishPrompt = `Create an engaging social media post about ${topic}. Use professional tone with data-backed insights.`;
  const vietnamesePrompt = `Tạo bài viết mạng xã hội hấp dẫn về ${topic}. Sử dụng giọng văn chuyên nghiệp với dữ liệu rõ ràng.`;

  const [english, vietnamese] = await Promise.all([
    generateWithClaude(englishPrompt, insights),
    generateWithClaude(vietnamesePrompt, insights)
  ]);

  return { english, vietnamese };
}
```

### 4. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

async function generateContentVideo(content: ContentPiece) {
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
      stats: content.metadata.stats,
      format: content.format,
      theme: 'modern'
    },
  });

  // Render video
  const outputLocation = path.join(
    process.cwd(), 
    'public/videos',
    `${content.format}-${Date.now()}.mp4`
  );

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: composition.inputProps,
  });

  return {
    videoPath: outputLocation,
    duration: composition.durationInFrames / composition.fps,
    dimensions: {
      width: composition.width,
      height: composition.height
    }
  };
}
```

### 5. Remotion Video Template

```tsx
// src/remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig, spring } from 'remotion';
import React from 'react';

interface ContentVideoProps {
  title: string;
  stats: Array<{ label: string; value: string }>;
  format: string;
  theme: 'modern' | 'minimal';
}

export const ContentVideo: React.FC<ContentVideoProps> = ({
  title,
  stats,
  format,
  theme
}) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  const titleOpacity = spring({
    frame,
    fps,
    from: 0,
    to: 1,
    durationInFrames: 30,
  });

  return (
    <AbsoluteFill style={{ backgroundColor: '#000' }}>
      <div style={{
        display: 'flex',
        flexDirection: 'column',
        justifyContent: 'center',
        alignItems: 'center',
        padding: 60,
        opacity: titleOpacity
      }}>
        <h1 style={{
          fontSize: 72,
          color: '#fff',
          textAlign: 'center',
          marginBottom: 40
        }}>
          {title}
        </h1>
        
        <div style={{ display: 'flex', gap: 40 }}>
          {stats.map((stat, index) => (
            <div key={index} style={{ textAlign: 'center' }}>
              <div style={{ fontSize: 48, color: '#00ff88', fontWeight: 'bold' }}>
                {stat.value}
              </div>
              <div style={{ fontSize: 24, color: '#ccc' }}>
                {stat.label}
              </div>
            </div>
          ))}
        </div>
      </div>
    </AbsoluteFill>
  );
};
```

### 6. Complete Pipeline Workflow

```typescript
import { crawlNews } from '@/lib/crawler/news-crawler';
import { generateContent } from '@/lib/content/generator';
import { generateContentVideo } from '@/lib/video/renderer';
import { postToSocialMedia } from '@/lib/social/publisher';

async function runCompletePipeline(keyword: string) {
  try {
    // Step 1: Research
    console.log('🔍 Starting research crawl...');
    const research = await crawlNews({
      keyword,
      sources: ['techcrunch', 'a16z', 'twitter'],
      timeRange: '24h'
    });

    // Step 2: Generate content
    console.log('✍️ Generating content...');
    const content = await generateContent({
      research: research.insights,
      format: 'toplist',
      languages: ['en', 'vi'],
      tone: 'expert',
      aiProvider: 'claude'
    });

    // Step 3: Create video
    console.log('🎬 Rendering video...');
    const video = await generateContentVideo(content);

    // Step 4: Publish
    console.log('📤 Publishing to social media...');
    const published = await postToSocialMedia({
      platforms: ['facebook', 'linkedin'],
      content: content.content,
      videoPath: video.videoPath,
      scheduledTime: new Date(Date.now() + 3600000) // 1 hour from now
    });

    return {
      success: true,
      research,
      content,
      video,
      published
    };
  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}

// Execute pipeline
runCompletePipeline('AI automation trends')
  .then(result => console.log('✅ Pipeline complete:', result))
  .catch(err => console.error('❌ Pipeline failed:', err));
```

## API Routes

### Generate Content Endpoint

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { generateContent } from '@/lib/content/generator';

export async function POST(request: NextRequest) {
  try {
    const { keyword, format, language, aiProvider } = await request.json();

    const content = await generateContent({
      keyword,
      format,
      language,
      aiProvider: aiProvider || 'claude',
      model: 'claude-3-sonnet-20240229'
    });

    return NextResponse.json({
      success: true,
      data: content
    });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

### Research Endpoint

```typescript
// src/app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { crawlNews } from '@/lib/crawler/news-crawler';

export async function POST(request: NextRequest) {
  const { keyword, sources, timeRange } = await request.json();

  const results = await crawlNews({
    keyword,
    sources: sources || ['techcrunch', 'a16z'],
    timeRange: timeRange || '24h'
  });

  return NextResponse.json({
    success: true,
    data: results
  });
}
```

## Configuration

### Content Formats

```typescript
// src/types/content.ts
export enum ContentFormat {
  TOPLIST = 'toplist',
  POV = 'pov',
  CASE_STUDY = 'case-study',
  HOW_TO = 'how-to',
  NEWS_SUMMARY = 'news-summary',
  THREAD = 'thread'
}

export enum Language {
  EN = 'en',
  VI = 'vi'
}

export enum Tone {
  EXPERT = 'expert',
  FRIENDLY = 'friendly',
  HUMOROUS = 'humorous',
  FORMAL = 'formal'
}
```

### Remotion Configuration

```typescript
// remotion.config.ts
import { Config } from '@remotion/cli/config';

Config.setVideoImageFormat('jpeg');
Config.setOverwriteOutput(true);
Config.setConcurrency(4);
Config.setCodec('h264');

// Export formats for different platforms
export const platformConfigs = {
  reels: { width: 1080, height: 1920, fps: 30 }, // 9:16
  tiktok: { width: 1080, height: 1920, fps: 30 }, // 9:16
  youtube: { width: 1920, height: 1080, fps: 30 }, // 16:9
  linkedin: { width: 1200, height: 1200, fps: 30 } // 1:1
};
```

## Common Patterns

### Multi-Platform Video Export

```typescript
async function exportMultiPlatform(content: ContentPiece) {
  const platforms = ['reels', 'tiktok', 'youtube', 'linkedin'];
  
  const videos = await Promise.all(
    platforms.map(async (platform) => {
      const config = platformConfigs[platform];
      
      return await renderMedia({
        composition: {
          ...baseComposition,
          width: config.width,
          height: config.height,
          fps: config.fps
        },
        serveUrl: bundleLocation,
        outputLocation: `public/videos/${platform}-${Date.now()}.mp4`,
      });
    })
  );

  return videos;
}
```

### Scheduled Content Queue

```typescript
interface ScheduledContent {
  id: string;
  content: ContentPiece;
  platforms: string[];
  scheduledTime: Date;
  status: 'pending' | 'published' | 'failed';
}

class ContentQueue {
  private queue: ScheduledContent[] = [];

  async schedule(content: ContentPiece, scheduledTime: Date, platforms: string[]) {
    const item: ScheduledContent = {
      id: crypto.randomUUID(),
      content,
      platforms,
      scheduledTime,
      status: 'pending'
    };

    this.queue.push(item);
    this.processQueue();
    
    return item.id;
  }

  private async processQueue() {
    const now = new Date();
    const ready = this.queue.filter(
      item => item.scheduledTime <= now && item.status === 'pending'
    );

    for (const item of ready) {
      try {
        await postToSocialMedia({
          platforms: item.platforms,
          content: item.content.content,
        });
        item.status = 'published';
      } catch (error) {
        item.status = 'failed';
        console.error(`Failed to publish ${item.id}:`, error);
      }
    }
  }
}
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement retry logic with exponential backoff
async function withRetry<T>(
  fn: () => Promise<T>,
  maxRetries = 3,
  baseDelay = 1000
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      
      const delay = baseDelay * Math.pow(2, i);
      console.log(`Retry ${i + 1}/${maxRetries} after ${delay}ms`);
      await new Promise(resolve => setTimeout(resolve, delay));
    }
  }
  throw new Error('Max retries reached');
}

// Usage
const content = await withRetry(() => 
  generateContent({ keyword: 'AI trends', format: 'toplist' })
);
```

### Video Rendering Memory Issues

```typescript
// Reduce concurrency for large videos
Config.setConcurrency(2);

// Use lower quality settings if needed
Config.setQuality(80); // 0-100

// Clear cache between renders
import { cleanupFfmpeg } from '@remotion/renderer';
await cleanupFfmpeg();
```

### Missing Environment Variables

```typescript
// src/lib/config/validate.ts
function validateEnv() {
  const required = [
    'ANTHROPIC_API_KEY',
    'OPENAI_API_KEY',
    'RAPIDAPI_KEY'
  ];

  const missing = required.filter(key => !process.env[key]);

  if (missing.length > 0) {
    throw new Error(
      `Missing required environment variables: ${missing.join(', ')}`
    );
  }
}

// Call on app startup
validateEnv();
```

### Claude API Errors

```typescript
// Handle Claude-specific errors
async function safeClaudeCall(prompt: string) {
  try {
    return await generateWithClaude(prompt, {});
  } catch (error) {
    if (error.status === 529) {
      console.log('Claude overloaded, switching to OpenAI...');
      return await generateWithOpenAI(prompt);
    }
    throw error;
  }
}
```

## Development Commands

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Render a specific video composition
npm run remotion:render -- --composition=ContentVideo --props='{"title":"Test"}'

# Preview Remotion compositions
npm run remotion:preview
```
