---
name: marketing-pipeline-share-automation
description: Automated content pipeline for research, scriptwriting, video generation, and social media posting using AI (Claude/OpenAI) and Remotion
triggers:
  - how do I set up the AI content pipeline
  - automate content creation with marketing pipeline
  - generate videos from blog posts automatically
  - scrape news and create content with AI
  - use marketing pipeline share for content automation
  - create multilingual content with Claude and OpenAI
  - render videos with Remotion in content pipeline
  - auto-post to social media with marketing pipeline
---

# Marketing Pipeline Share Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

Ultimate AI Content Pipeline is a TypeScript-based automated content creation system that handles the complete workflow from research (news scraping), to AI-powered scriptwriting (Claude/OpenAI), to video generation (Remotion), and automatic social media posting. Perfect for content creators, marketers, and agencies looking to scale content production.

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

```env
# AI Services
OPENAI_API_KEY=your_openai_key_here
ANTHROPIC_API_KEY=your_claude_key_here

# News/Research APIs
RAPIDAPI_KEY=your_rapidapi_key_here

# Social Media (if auto-posting)
FACEBOOK_PAGE_TOKEN=your_page_token_here
FACEBOOK_PAGE_ID=your_page_id_here

# Optional: Database
DATABASE_URL=your_database_connection_string

# Next.js Configuration
NEXT_PUBLIC_API_URL=http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── scraper/     # News scraping modules
│   │   ├── video/       # Remotion video generation
│   │   └── social/      # Social media posting
│   └── utils/           # Helper functions
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Core Features & Usage

### 1. Research & News Scraping

The system automatically scrapes news from major sources like TechCrunch, a16z, Twitter/X, and LinkedIn.

```typescript
// src/lib/scraper/news-fetcher.ts
import { fetchLatestNews } from '@/lib/scraper/news-fetcher';

interface NewsSource {
  source: string;
  title: string;
  url: string;
  publishedAt: string;
  content: string;
}

async function gatherResearch(keyword: string): Promise<NewsSource[]> {
  const news = await fetchLatestNews({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeRange: '24h',
    limit: 20
  });
  
  return news;
}

// Usage
const aiNews = await gatherResearch('artificial intelligence');
console.log(`Found ${aiNews.length} articles`);
```

### 2. AI Content Generation

Generate content in multiple formats using Claude or OpenAI:

```typescript
// src/lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

interface ContentRequest {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  tone: 'expert' | 'friendly' | 'humorous';
  language: 'en' | 'vi' | 'both';
  researchData: NewsSource[];
}

async function generateContent(request: ContentRequest): Promise<string> {
  const prompt = `
Based on the following recent news and insights:
${request.researchData.map(n => `- ${n.title}: ${n.content}`).join('\n')}

Create a ${request.format} article about "${request.keyword}" in ${request.tone} tone.
Language: ${request.language}
Include data-backed insights and real examples from the research.
`;

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [
      { role: 'user', content: prompt }
    ],
  });

  return message.content[0].text;
}

// Usage
const content = await generateContent({
  keyword: 'AI automation',
  format: 'toplist',
  tone: 'expert',
  language: 'both',
  researchData: aiNews
});
```

### 3. Multilingual Content Generation

Generate content in both English and Vietnamese simultaneously:

```typescript
// src/lib/ai/multilingual-generator.ts
interface MultilingualContent {
  english: string;
  vietnamese: string;
}

async function generateBilingualContent(
  keyword: string,
  researchData: NewsSource[]
): Promise<MultilingualContent> {
  const englishPrompt = `Write a professional article about "${keyword}" in English...`;
  const vietnamesePrompt = `Viết một bài viết chuyên nghiệp về "${keyword}" bằng tiếng Việt...`;

  const [english, vietnamese] = await Promise.all([
    anthropic.messages.create({
      model: 'claude-3-5-sonnet-20241022',
      max_tokens: 4096,
      messages: [{ role: 'user', content: englishPrompt }],
    }),
    anthropic.messages.create({
      model: 'claude-3-5-sonnet-20241022',
      max_tokens: 4096,
      messages: [{ role: 'user', content: vietnamesePrompt }],
    }),
  ]);

  return {
    english: english.content[0].text,
    vietnamese: vietnamese.content[0].text,
  };
}
```

### 4. Video Generation with Remotion

Transform written content into videos automatically:

```typescript
// src/lib/video/video-renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  content: string;
  title: string;
  format: 'reels' | 'tiktok' | 'shorts'; // 9:16 aspect ratio
  duration: number; // in frames (30fps)
}

async function generateVideo(config: VideoConfig): Promise<string> {
  // Bundle Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  // Get composition
  const compositionId = 'ContentVideo';
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: {
      title: config.title,
      content: config.content,
      format: config.format,
    },
  });

  // Render video
  const outputLocation = path.resolve(`./public/videos/${Date.now()}.mp4`);
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
  });

  return outputLocation;
}

// Usage
const videoPath = await generateVideo({
  content: content.english,
  title: 'Top 5 AI Automation Tools',
  format: 'reels',
  duration: 900, // 30 seconds at 30fps
});
```

### 5. Remotion Video Template

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, Sequence, useCurrentFrame, useVideoConfig } from 'remotion';
import React from 'react';

interface ContentVideoProps {
  title: string;
  content: string;
  format: 'reels' | 'tiktok' | 'shorts';
}

export const ContentVideo: React.FC<ContentVideoProps> = ({ title, content, format }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  const opacity = Math.min(1, frame / 30);

  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a1a' }}>
      <Sequence from={0} durationInFrames={90}>
        <AbsoluteFill style={{ 
          justifyContent: 'center', 
          alignItems: 'center',
          opacity 
        }}>
          <h1 style={{ 
            color: 'white', 
            fontSize: 60, 
            textAlign: 'center',
            padding: '0 40px',
            fontFamily: 'Arial, sans-serif'
          }}>
            {title}
          </h1>
        </AbsoluteFill>
      </Sequence>
      
      <Sequence from={90}>
        <AbsoluteFill style={{ 
          padding: 40,
          color: 'white',
          fontSize: 32,
          fontFamily: 'Arial, sans-serif'
        }}>
          {content.split('\n').map((line, i) => (
            <p key={i} style={{ marginBottom: 20 }}>{line}</p>
          ))}
        </AbsoluteFill>
      </Sequence>
    </AbsoluteFill>
  );
};
```

### 6. Automatic Social Media Posting

Post content directly to Facebook Pages:

```typescript
// src/lib/social/facebook-poster.ts
interface PostConfig {
  pageId: string;
  accessToken: string;
  message: string;
  videoPath?: string;
  imageUrl?: string;
}

async function postToFacebook(config: PostConfig): Promise<string> {
  const formData = new FormData();
  formData.append('message', config.message);
  formData.append('access_token', config.accessToken);

  if (config.videoPath) {
    const videoFile = await fetch(config.videoPath);
    const blob = await videoFile.blob();
    formData.append('source', blob);
    
    const response = await fetch(
      `https://graph.facebook.com/v18.0/${config.pageId}/videos`,
      {
        method: 'POST',
        body: formData,
      }
    );
    
    const data = await response.json();
    return data.id;
  }

  // For text/image posts
  const response = await fetch(
    `https://graph.facebook.com/v18.0/${config.pageId}/feed`,
    {
      method: 'POST',
      body: formData,
    }
  );

  const data = await response.json();
  return data.id;
}
```

### 7. Complete Pipeline Workflow

```typescript
// src/lib/pipeline/content-pipeline.ts
async function runContentPipeline(keyword: string) {
  console.log(`Starting pipeline for keyword: ${keyword}`);

  // Step 1: Research
  console.log('Step 1: Gathering research...');
  const research = await gatherResearch(keyword);

  // Step 2: Generate Content
  console.log('Step 2: Generating content...');
  const content = await generateBilingualContent(keyword, research);

  // Step 3: Generate Video
  console.log('Step 3: Rendering video...');
  const videoPath = await generateVideo({
    content: content.english,
    title: keyword,
    format: 'reels',
    duration: 900,
  });

  // Step 4: Post to Social Media
  console.log('Step 4: Posting to Facebook...');
  const postId = await postToFacebook({
    pageId: process.env.FACEBOOK_PAGE_ID!,
    accessToken: process.env.FACEBOOK_PAGE_TOKEN!,
    message: content.vietnamese,
    videoPath,
  });

  console.log(`Pipeline complete! Post ID: ${postId}`);
  
  return {
    research,
    content,
    videoPath,
    postId,
  };
}

// Usage
const result = await runContentPipeline('AI Content Marketing 2024');
```

## Next.js API Routes

```typescript
// src/app/api/pipeline/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline/content-pipeline';

export async function POST(request: NextRequest) {
  try {
    const { keyword, format, language } = await request.json();

    if (!keyword) {
      return NextResponse.json(
        { error: 'Keyword is required' },
        { status: 400 }
      );
    }

    const result = await runContentPipeline(keyword);

    return NextResponse.json({
      success: true,
      data: result,
    });
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { error: 'Pipeline failed', details: error.message },
      { status: 500 }
    );
  }
}
```

## Running the Application

```bash
# Development mode
npm run dev

# Build for production
npm run build

# Start production server
npm run start

# Render Remotion video (standalone)
npm run remotion:render
```

## Common Patterns

### Scheduling Content Generation

```typescript
// src/lib/scheduler/cron-pipeline.ts
import cron from 'node-cron';

// Run pipeline every day at 9 AM
cron.schedule('0 9 * * *', async () => {
  const keywords = ['AI trends', 'Marketing automation', 'Content creation'];
  
  for (const keyword of keywords) {
    try {
      await runContentPipeline(keyword);
      console.log(`✓ Completed pipeline for: ${keyword}`);
    } catch (error) {
      console.error(`✗ Failed pipeline for: ${keyword}`, error);
    }
  }
});
```

### Content Queue Management

```typescript
// src/lib/queue/content-queue.ts
interface QueueItem {
  keyword: string;
  format: string;
  scheduledFor: Date;
  status: 'pending' | 'processing' | 'completed' | 'failed';
}

class ContentQueue {
  private queue: QueueItem[] = [];

  add(item: Omit<QueueItem, 'status'>) {
    this.queue.push({ ...item, status: 'pending' });
  }

  async process() {
    const pending = this.queue.filter(i => i.status === 'pending');
    
    for (const item of pending) {
      if (new Date() >= item.scheduledFor) {
        item.status = 'processing';
        try {
          await runContentPipeline(item.keyword);
          item.status = 'completed';
        } catch (error) {
          item.status = 'failed';
        }
      }
    }
  }
}
```

## Troubleshooting

### API Rate Limits

```typescript
// src/lib/utils/rate-limiter.ts
class RateLimiter {
  private requests: number[] = [];
  private limit: number;
  private window: number;

  constructor(limit: number, windowMs: number) {
    this.limit = limit;
    this.window = windowMs;
  }

  async throttle() {
    const now = Date.now();
    this.requests = this.requests.filter(t => now - t < this.window);

    if (this.requests.length >= this.limit) {
      const oldestRequest = this.requests[0];
      const waitTime = this.window - (now - oldestRequest);
      await new Promise(resolve => setTimeout(resolve, waitTime));
    }

    this.requests.push(Date.now());
  }
}

// Usage
const limiter = new RateLimiter(5, 60000); // 5 requests per minute
await limiter.throttle();
```

### Error Handling

```typescript
// src/lib/utils/error-handler.ts
async function safeExecute<T>(
  fn: () => Promise<T>,
  retries = 3
): Promise<T | null> {
  for (let i = 0; i < retries; i++) {
    try {
      return await fn();
    } catch (error) {
      console.error(`Attempt ${i + 1} failed:`, error);
      if (i === retries - 1) return null;
      await new Promise(resolve => setTimeout(resolve, 1000 * (i + 1)));
    }
  }
  return null;
}
```

### Video Rendering Issues

If Remotion rendering fails, ensure you have the required system dependencies:

```bash
# Linux
sudo apt-get install ffmpeg chromium

# macOS
brew install ffmpeg
brew install --cask chromium
```

Set Remotion configuration:

```typescript
// remotion.config.ts
import { Config } from '@remotion/cli/config';

Config.setVideoImageFormat('jpeg');
Config.setOverwriteOutput(true);
Config.setConcurrency(2); // Reduce if memory issues
```

This skill provides comprehensive coverage of the marketing-pipeline-share automation system for AI content creation, video generation, and social media posting.
