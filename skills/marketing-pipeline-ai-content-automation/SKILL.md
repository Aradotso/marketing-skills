---
name: marketing-pipeline-ai-content-automation
description: AI-powered content automation pipeline for research, scriptwriting, and video generation using Claude, OpenAI, and Remotion
triggers:
  - automate content creation with AI research and video generation
  - set up marketing pipeline with Claude and OpenAI integration
  - create automated content workflow from research to video
  - build AI content pipeline with auto-posting capabilities
  - implement content automation using Remotion for video rendering
  - configure marketing automation pipeline with multilingual support
  - generate content from keyword research to final video
  - set up automated content research and publishing system
---

# Marketing Pipeline AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the Ultimate AI Content Pipeline, a comprehensive TypeScript-based system that automates the entire content creation workflow from research to video generation and publishing.

## What This Project Does

The Marketing Pipeline is an all-in-one content automation system that:

- **Auto-crawls** news and insights from sources like TechCrunch, a16z, Twitter/X, and LinkedIn
- **Generates content** in multiple formats (toplist, POV, case study, how-to) using Claude/OpenAI
- **Supports multilingual** output (English & Vietnamese) with customizable tone
- **Renders videos** automatically using Remotion for social media platforms
- **Publishes content** to Facebook pages and other platforms automatically

## Project Structure

```
marketing-pipeline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/              # Core libraries
│   │   ├── ai/          # AI integrations (Claude, OpenAI)
│   │   ├── crawler/     # Content crawling logic
│   │   ├── video/       # Remotion video generation
│   │   └── publishers/  # Platform publishing adapters
│   ├── config/          # Configuration files
│   └── types/           # TypeScript type definitions
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Installation

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
pnpm install
# or
yarn install

# Set up environment variables
cp .env.example .env.local
```

## Configuration

### Environment Variables

```bash
# AI Providers
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# Data Sources
RAPIDAPI_KEY=your_rapidapi_key

# Social Media Publishing
FACEBOOK_PAGE_ACCESS_TOKEN=your_fb_token
FACEBOOK_PAGE_ID=your_page_id

# Video Rendering
REMOTION_LICENSE_KEY=your_remotion_key

# Application
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

### Core Configuration File

Create `src/config/pipeline.config.ts`:

```typescript
export const pipelineConfig = {
  research: {
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeRange: '24h',
    maxArticles: 10
  },
  content: {
    formats: ['toplist', 'pov', 'case-study', 'how-to'],
    languages: ['en', 'vi'],
    tones: ['expert', 'friendly', 'humorous']
  },
  video: {
    platforms: ['reels', 'tiktok', 'shorts'],
    aspectRatios: {
      reels: '9:16',
      tiktok: '9:16',
      shorts: '9:16'
    }
  }
};
```

## Key API Usage Patterns

### 1. Research & Data Crawling

```typescript
import { ResearchCrawler } from '@/lib/crawler/research-crawler';

// Initialize crawler
const crawler = new ResearchCrawler({
  apiKey: process.env.RAPIDAPI_KEY!,
  sources: ['techcrunch', 'a16z']
});

// Crawl content by keyword
async function fetchResearch(keyword: string) {
  const results = await crawler.crawl({
    keyword,
    timeRange: '24h',
    limit: 10
  });
  
  return results.map(article => ({
    title: article.title,
    url: article.url,
    content: article.content,
    publishedAt: article.publishedAt,
    insights: article.extractedInsights
  }));
}
```

### 2. AI Content Generation with Claude

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY!
});

async function generateContent(
  research: any[],
  format: string,
  language: string,
  tone: string
) {
  const prompt = `
You are a professional content creator. Using the following research data:
${JSON.stringify(research, null, 2)}

Create a ${format} article in ${language} with a ${tone} tone.
Include data-backed insights and current trends.
`;

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [{
      role: 'user',
      content: prompt
    }]
  });

  return message.content[0].type === 'text' 
    ? message.content[0].text 
    : '';
}
```

### 3. AI Content Generation with OpenAI

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY!
});

async function generateContentOpenAI(
  research: any[],
  options: {
    format: string;
    language: string;
    tone: string;
  }
) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are an expert content creator specializing in ${options.format} format.`
      },
      {
        role: 'user',
        content: `Create content in ${options.language} with ${options.tone} tone based on: ${JSON.stringify(research)}`
      }
    ],
    temperature: 0.7
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
  platform: 'reels' | 'tiktok' | 'shorts'
) {
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: (config) => config
  });

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      content,
      platform
    }
  });

  const outputLocation = path.join(
    process.cwd(), 
    'public', 
    'videos', 
    `${Date.now()}.mp4`
  );

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: {
      content,
      aspectRatio: platform === 'reels' ? '9:16' : '9:16'
    }
  });

  return outputLocation;
}
```

### 5. Remotion Video Template

Create `remotion/ContentVideo.tsx`:

```typescript
import { AbsoluteFill, useCurrentFrame, useVideoConfig } from 'remotion';

interface ContentVideoProps {
  content: string;
  platform: string;
}

export const ContentVideo: React.FC<ContentVideoProps> = ({ 
  content, 
  platform 
}) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  const opacity = Math.min(1, frame / (fps * 0.5));

  return (
    <AbsoluteFill
      style={{
        backgroundColor: '#000',
        justifyContent: 'center',
        alignItems: 'center'
      }}
    >
      <div
        style={{
          fontSize: 48,
          color: 'white',
          opacity,
          padding: 40,
          textAlign: 'center'
        }}
      >
        {content.slice(0, 100)}...
      </div>
    </AbsoluteFill>
  );
};
```

### 6. Publishing to Facebook

```typescript
import axios from 'axios';

async function publishToFacebook(
  content: string,
  videoPath?: string
) {
  const pageAccessToken = process.env.FACEBOOK_PAGE_ACCESS_TOKEN!;
  const pageId = process.env.FACEBOOK_PAGE_ID!;

  const url = `https://graph.facebook.com/v18.0/${pageId}/feed`;

  const response = await axios.post(url, {
    message: content,
    access_token: pageAccessToken,
    ...(videoPath && { 
      link: `${process.env.NEXT_PUBLIC_APP_URL}/videos/${path.basename(videoPath)}` 
    })
  });

  return response.data;
}
```

### 7. Complete Pipeline Orchestration

```typescript
// src/lib/pipeline/orchestrator.ts

export class ContentPipeline {
  async execute(keyword: string, options: {
    format: string;
    language: string;
    tone: string;
    generateVideo: boolean;
    autoPublish: boolean;
  }) {
    // Step 1: Research
    const research = await this.research(keyword);
    
    // Step 2: Generate Content
    const content = await this.generateContent(
      research, 
      options.format,
      options.language,
      options.tone
    );
    
    // Step 3: Generate Video (optional)
    let videoPath;
    if (options.generateVideo) {
      videoPath = await this.generateVideo(content, 'reels');
    }
    
    // Step 4: Publish (optional)
    if (options.autoPublish) {
      await this.publish(content, videoPath);
    }
    
    return {
      content,
      videoPath,
      published: options.autoPublish
    };
  }

  private async research(keyword: string) {
    const crawler = new ResearchCrawler({
      apiKey: process.env.RAPIDAPI_KEY!
    });
    return crawler.crawl({ keyword, timeRange: '24h' });
  }

  private async generateContent(
    research: any[],
    format: string,
    language: string,
    tone: string
  ) {
    return generateContent(research, format, language, tone);
  }

  private async generateVideo(content: string, platform: string) {
    return generateVideo(content, platform as any);
  }

  private async publish(content: string, videoPath?: string) {
    return publishToFacebook(content, videoPath);
  }
}
```

## Usage in Next.js API Routes

```typescript
// src/app/api/pipeline/route.ts

import { NextRequest, NextResponse } from 'next/server';
import { ContentPipeline } from '@/lib/pipeline/orchestrator';

export async function POST(req: NextRequest) {
  const { keyword, format, language, tone, generateVideo, autoPublish } = 
    await req.json();

  const pipeline = new ContentPipeline();

  try {
    const result = await pipeline.execute(keyword, {
      format,
      language,
      tone,
      generateVideo,
      autoPublish
    });

    return NextResponse.json(result);
  } catch (error) {
    return NextResponse.json(
      { error: 'Pipeline execution failed' },
      { status: 500 }
    );
  }
}
```

## CLI Commands

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Start production server
npm run start

# Run Remotion preview
npm run remotion:preview

# Render video directly
npm run remotion:render
```

## Common Patterns

### Multi-Language Content Generation

```typescript
async function generateMultilingualContent(
  keyword: string,
  languages: string[]
) {
  const research = await fetchResearch(keyword);
  
  const contentPromises = languages.map(lang =>
    generateContent(research, 'toplist', lang, 'expert')
  );
  
  const results = await Promise.all(contentPromises);
  
  return languages.reduce((acc, lang, idx) => {
    acc[lang] = results[idx];
    return acc;
  }, {} as Record<string, string>);
}
```

### Batch Video Generation

```typescript
async function batchGenerateVideos(
  contents: string[],
  platforms: Array<'reels' | 'tiktok' | 'shorts'>
) {
  const videos = [];
  
  for (const content of contents) {
    for (const platform of platforms) {
      const videoPath = await generateVideo(content, platform);
      videos.push({ content, platform, videoPath });
    }
  }
  
  return videos;
}
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement rate limiting for API calls
import pLimit from 'p-limit';

const limit = pLimit(3); // 3 concurrent requests max

async function batchWithRateLimit(keywords: string[]) {
  return Promise.all(
    keywords.map(keyword => 
      limit(() => fetchResearch(keyword))
    )
  );
}
```

### Video Rendering Errors

```typescript
// Add error handling and retries
async function generateVideoWithRetry(
  content: string,
  platform: string,
  maxRetries = 3
) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await generateVideo(content, platform as any);
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      await new Promise(resolve => setTimeout(resolve, 1000 * (i + 1)));
    }
  }
}
```

### Memory Management for Large Crawls

```typescript
// Process in chunks to avoid memory issues
async function crawlInChunks(
  keywords: string[],
  chunkSize = 5
) {
  const results = [];
  
  for (let i = 0; i < keywords.length; i += chunkSize) {
    const chunk = keywords.slice(i, i + chunkSize);
    const chunkResults = await Promise.all(
      chunk.map(k => fetchResearch(k))
    );
    results.push(...chunkResults.flat());
  }
  
  return results;
}
```
