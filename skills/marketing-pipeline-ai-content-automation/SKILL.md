---
name: marketing-pipeline-ai-content-automation
description: Automated AI content pipeline for research, scriptwriting, Facebook posting, and video generation using Claude/OpenAI and Remotion
triggers:
  - automate content creation pipeline with AI
  - set up automated marketing content generation
  - create AI-powered content research and writing system
  - generate videos automatically from content scripts
  - build end-to-end content automation workflow
  - scrape news and generate marketing content with AI
  - automate Facebook posting with AI-generated content
  - set up Remotion video rendering pipeline
---

# Marketing Pipeline AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

Marketing Pipeline AI Content Automation is a complete end-to-end TypeScript system that automates content creation from research to publication. It crawls news sources (TechCrunch, a16z, Twitter/X, LinkedIn), uses Claude 3 or OpenAI to generate multi-format content (toplist, POV, case studies, how-to), automatically posts to Facebook, and renders videos using Remotion.

**Key Capabilities:**
- Auto-scan research from multiple news sources (last 24h)
- AI-powered content generation in multiple formats and languages (EN/VI)
- Automatic Facebook page posting
- Video/infographic generation with Remotion
- Next.js dashboard for content management

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

## Configuration

Create `.env.local` with the following required variables:

```bash
# AI Provider Keys
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# News/Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Facebook Integration
FACEBOOK_PAGE_ACCESS_TOKEN=your_fb_page_token
FACEBOOK_PAGE_ID=your_page_id

# Remotion (Video Generation)
REMOTION_LICENSE_KEY=your_remotion_license

# Database (if applicable)
DATABASE_URL=your_database_url

# App Configuration
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── app/                    # Next.js app directory
├── components/             # React components
├── lib/
│   ├── ai/                # AI provider integrations
│   ├── research/          # News crawling & research
│   ├── content/           # Content generation logic
│   ├── facebook/          # Facebook API integration
│   └── remotion/          # Video rendering setup
├── remotion/              # Remotion video templates
├── public/                # Static assets
└── types/                 # TypeScript type definitions
```

## Core Workflows

### 1. Research Pipeline - Auto News Crawling

```typescript
import { crawlLatestNews } from '@/lib/research/crawler';
import { analyzeContent } from '@/lib/ai/analyzer';

async function runResearch(keyword: string) {
  // Crawl news from multiple sources
  const newsData = await crawlLatestNews({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeRange: '24h'
  });

  // Analyze and extract insights with AI
  const insights = await analyzeContent({
    data: newsData,
    model: 'claude-3-sonnet-20240229',
    apiKey: process.env.ANTHROPIC_API_KEY!
  });

  return {
    rawNews: newsData,
    insights,
    timestamp: new Date().toISOString()
  };
}
```

### 2. Content Generation with Claude/OpenAI

```typescript
import Anthropic from '@anthropic-ai/sdk';

interface ContentGenerationOptions {
  topic: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  researchData: any;
}

async function generateContent(options: ContentGenerationOptions) {
  const anthropic = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY!
  });

  const systemPrompt = buildSystemPrompt(options);
  const userPrompt = buildUserPrompt(options);

  const message = await anthropic.messages.create({
    model: 'claude-3-sonnet-20240229',
    max_tokens: 4096,
    messages: [
      {
        role: 'user',
        content: `${systemPrompt}\n\n${userPrompt}`
      }
    ]
  });

  return {
    content: message.content[0].text,
    metadata: {
      format: options.format,
      language: options.language,
      wordCount: message.content[0].text.split(' ').length
    }
  };
}

function buildSystemPrompt(options: ContentGenerationOptions): string {
  const toneMap = {
    expert: 'professional and authoritative',
    friendly: 'casual and approachable',
    humorous: 'witty and entertaining'
  };

  return `You are an expert content writer creating ${options.format} content in ${options.language}.
  Use a ${toneMap[options.tone]} tone.
  Base your writing on the provided research data and ensure all claims are data-backed.`;
}

function buildUserPrompt(options: ContentGenerationOptions): string {
  return `Topic: ${options.topic}
  
  Research Data: ${JSON.stringify(options.researchData)}
  
  Create a compelling ${options.format} article that:
  1. Opens with a strong hook
  2. Incorporates latest insights from the research
  3. Provides actionable takeaways
  4. Ends with a clear call-to-action`;
}
```

### 3. Alternative: Using OpenAI

```typescript
import OpenAI from 'openai';

async function generateContentWithOpenAI(options: ContentGenerationOptions) {
  const openai = new OpenAI({
    apiKey: process.env.OPENAI_API_KEY!
  });

  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: buildSystemPrompt(options)
      },
      {
        role: 'user',
        content: buildUserPrompt(options)
      }
    ],
    temperature: 0.7,
    max_tokens: 3000
  });

  return completion.choices[0].message.content;
}
```

### 4. Facebook Auto-Posting

```typescript
import axios from 'axios';

interface FacebookPostOptions {
  message: string;
  link?: string;
  imageUrl?: string;
}

async function postToFacebook(options: FacebookPostOptions) {
  const pageId = process.env.FACEBOOK_PAGE_ID!;
  const accessToken = process.env.FACEBOOK_PAGE_ACCESS_TOKEN!;
  
  const url = `https://graph.facebook.com/v18.0/${pageId}/feed`;
  
  const payload: any = {
    message: options.message,
    access_token: accessToken
  };

  if (options.link) {
    payload.link = options.link;
  }

  if (options.imageUrl) {
    // Upload photo first
    const photoUrl = `https://graph.facebook.com/v18.0/${pageId}/photos`;
    const photoResponse = await axios.post(photoUrl, {
      url: options.imageUrl,
      access_token: accessToken,
      published: false
    });

    payload.attached_media = [{ media_fbid: photoResponse.data.id }];
    delete payload.link; // Can't use link with photo
  }

  const response = await axios.post(url, payload);
  
  return {
    postId: response.data.id,
    url: `https://facebook.com/${response.data.id}`
  };
}
```

### 5. Video Generation with Remotion

```typescript
// remotion/VideoComposition.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig } from 'remotion';

interface VideoProps {
  title: string;
  points: string[];
  brandColor: string;
}

export const ContentVideo: React.FC<VideoProps> = ({ 
  title, 
  points, 
  brandColor 
}) => {
  const frame = useCurrentFrame();
  const { fps, durationInFrames } = useVideoConfig();

  const opacity = Math.min(1, frame / 30);

  return (
    <AbsoluteFill style={{ backgroundColor: '#000' }}>
      <div style={{ 
        padding: 60, 
        color: '#fff',
        opacity 
      }}>
        <h1 style={{ 
          fontSize: 64, 
          marginBottom: 40,
          color: brandColor 
        }}>
          {title}
        </h1>
        <ul style={{ fontSize: 32, lineHeight: 1.6 }}>
          {points.map((point, index) => {
            const pointFrame = 60 + (index * 30);
            const pointOpacity = Math.min(1, Math.max(0, (frame - pointFrame) / 20));
            
            return (
              <li key={index} style={{ 
                marginBottom: 20,
                opacity: pointOpacity,
                transform: `translateX(${Math.max(0, 20 - (frame - pointFrame))}px)`
              }}>
                {point}
              </li>
            );
          })}
        </ul>
      </div>
    </AbsoluteFill>
  );
};
```

```typescript
// lib/remotion/render.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface RenderVideoOptions {
  title: string;
  points: string[];
  brandColor?: string;
  outputPath: string;
}

export async function renderContentVideo(options: RenderVideoOptions) {
  const compositionId = 'ContentVideo';
  
  // Bundle the Remotion project
  const bundleLocation = await bundle(
    path.join(process.cwd(), 'remotion', 'index.ts')
  );

  // Get composition metadata
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: {
      title: options.title,
      points: options.points,
      brandColor: options.brandColor || '#3b82f6'
    }
  });

  // Render the video
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: options.outputPath,
    inputProps: {
      title: options.title,
      points: options.points,
      brandColor: options.brandColor || '#3b82f6'
    }
  });

  return options.outputPath;
}
```

### 6. Complete Pipeline Orchestration

```typescript
// lib/pipeline/orchestrator.ts
import { runResearch } from '@/lib/research/crawler';
import { generateContent } from '@/lib/ai/generator';
import { postToFacebook } from '@/lib/facebook/poster';
import { renderContentVideo } from '@/lib/remotion/render';
import path from 'path';

interface PipelineOptions {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  autoPost?: boolean;
  generateVideo?: boolean;
}

export async function runContentPipeline(options: PipelineOptions) {
  console.log(`Starting pipeline for: ${options.keyword}`);

  // Step 1: Research
  console.log('Step 1: Running research...');
  const research = await runResearch(options.keyword);

  // Step 2: Generate Content
  console.log('Step 2: Generating content...');
  const content = await generateContent({
    topic: options.keyword,
    format: options.format,
    language: options.language,
    tone: 'expert',
    researchData: research.insights
  });

  let videoPath: string | null = null;
  let postResult: any = null;

  // Step 3: Generate Video (if requested)
  if (options.generateVideo) {
    console.log('Step 3: Rendering video...');
    const videoFileName = `${Date.now()}-${options.keyword.replace(/\s+/g, '-')}.mp4`;
    videoPath = path.join(process.cwd(), 'public', 'videos', videoFileName);

    await renderContentVideo({
      title: options.keyword,
      points: extractKeyPoints(content.content),
      outputPath: videoPath
    });
  }

  // Step 4: Post to Facebook (if requested)
  if (options.autoPost) {
    console.log('Step 4: Posting to Facebook...');
    postResult = await postToFacebook({
      message: content.content.substring(0, 500) + '...',
      link: videoPath ? `${process.env.NEXT_PUBLIC_APP_URL}/videos/${path.basename(videoPath)}` : undefined
    });
  }

  return {
    research,
    content,
    videoPath,
    postResult,
    completedAt: new Date().toISOString()
  };
}

function extractKeyPoints(content: string): string[] {
  // Simple extraction - split by numbered points or bullet points
  const lines = content.split('\n');
  const points = lines
    .filter(line => /^[\d\-\*]/.test(line.trim()))
    .map(line => line.replace(/^[\d\-\*\.]+\s*/, '').trim())
    .filter(line => line.length > 0)
    .slice(0, 5);

  return points.length > 0 ? points : ['AI-Generated Content', 'Data-Backed Insights', 'Trending Topics'];
}
```

### 7. Next.js API Route Example

```typescript
// app/api/pipeline/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline/orchestrator';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const { keyword, format, language, autoPost, generateVideo } = body;

    if (!keyword) {
      return NextResponse.json(
        { error: 'Keyword is required' },
        { status: 400 }
      );
    }

    const result = await runContentPipeline({
      keyword,
      format: format || 'toplist',
      language: language || 'en',
      autoPost: autoPost || false,
      generateVideo: generateVideo || false
    });

    return NextResponse.json({
      success: true,
      data: result
    });
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { error: 'Pipeline execution failed', details: error.message },
      { status: 500 }
    );
  }
}
```

## Usage Patterns

### Basic Content Generation

```typescript
// Simple one-off content generation
import { runContentPipeline } from '@/lib/pipeline/orchestrator';

const result = await runContentPipeline({
  keyword: 'AI Marketing Trends 2024',
  format: 'toplist',
  language: 'en',
  autoPost: false,
  generateVideo: false
});

console.log(result.content.content);
```

### Full Automation with Video and Posting

```typescript
// Complete automation - research, write, render, post
const result = await runContentPipeline({
  keyword: 'Latest AI Tools for Marketers',
  format: 'how-to',
  language: 'vi',
  autoPost: true,
  generateVideo: true
});

console.log(`Content posted: ${result.postResult.url}`);
console.log(`Video rendered: ${result.videoPath}`);
```

### Batch Processing

```typescript
// Process multiple topics
const topics = [
  'Content Marketing AI Tools',
  'Social Media Automation',
  'Video Marketing Trends'
];

const results = await Promise.all(
  topics.map(keyword => runContentPipeline({
    keyword,
    format: 'toplist',
    language: 'en',
    autoPost: false,
    generateVideo: false
  }))
);

console.log(`Generated ${results.length} pieces of content`);
```

## Common Commands

```bash
# Development
npm run dev              # Start Next.js dev server
npm run build           # Build for production
npm run start           # Start production server

# Remotion
npm run remotion        # Open Remotion Studio
npm run render          # Render all compositions

# Type checking
npm run type-check      # Run TypeScript compiler check

# Linting
npm run lint            # Run ESLint
```

## Troubleshooting

### API Rate Limits

If hitting rate limits on Claude or OpenAI:

```typescript
// Add retry logic with exponential backoff
async function generateContentWithRetry(options: ContentGenerationOptions, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await generateContent(options);
    } catch (error) {
      if (error.status === 429 && i < maxRetries - 1) {
        const delay = Math.pow(2, i) * 1000;
        console.log(`Rate limited, retrying in ${delay}ms...`);
        await new Promise(resolve => setTimeout(resolve, delay));
      } else {
        throw error;
      }
    }
  }
}
```

### Facebook Token Expiration

Facebook tokens expire. Refresh them programmatically:

```typescript
// Check token validity
async function validateFacebookToken() {
  const response = await axios.get(
    `https://graph.facebook.com/debug_token?input_token=${process.env.FACEBOOK_PAGE_ACCESS_TOKEN}&access_token=${process.env.FACEBOOK_PAGE_ACCESS_TOKEN}`
  );
  
  const expiresAt = response.data.data.expires_at;
  if (expiresAt && expiresAt * 1000 < Date.now()) {
    throw new Error('Facebook token expired. Please refresh.');
  }
}
```

### Remotion Rendering Memory Issues

For large videos, increase Node memory:

```bash
# In package.json scripts
"render": "NODE_OPTIONS='--max-old-space-size=4096' remotion render"
```

### News Crawling Failures

Handle source-specific errors gracefully:

```typescript
async function crawlLatestNews(options: any) {
  const results = await Promise.allSettled(
    options.sources.map(async (source: string) => {
      try {
        return await crawlSource(source, options.keyword);
      } catch (error) {
        console.warn(`Failed to crawl ${source}:`, error.message);
        return null;
      }
    })
  );

  return results
    .filter(r => r.status === 'fulfilled' && r.value !== null)
    .map(r => (r as PromiseFulfilledResult<any>).value);
}
```

## Best Practices

1. **Environment Variables**: Never commit `.env.local`. Use `.env.example` as template.

2. **Error Handling**: Wrap all AI API calls in try-catch blocks with specific error types.

3. **Content Caching**: Cache research results to avoid redundant API calls:

```typescript
import { Redis } from '@upstash/redis';

const redis = new Redis({
  url: process.env.UPSTASH_REDIS_URL!,
  token: process.env.UPSTASH_REDIS_TOKEN!
});

async function getCachedResearch(keyword: string) {
  const cached = await redis.get(`research:${keyword}`);
  if (cached) return cached;

  const fresh = await runResearch(keyword);
  await redis.set(`research:${keyword}`, fresh, { ex: 3600 }); // 1 hour
  return fresh;
}
```

4. **Video Rendering**: Render videos in background jobs for better performance:

```typescript
// Use a job queue like BullMQ or Inngest
import { Inngest } from 'inngest';

const inngest = new Inngest({ name: 'Marketing Pipeline' });

export const renderVideoJob = inngest.createFunction(
  { name: 'Render Content Video' },
  { event: 'video/render.requested' },
  async ({ event, step }) => {
    const videoPath = await step.run('render', async () => {
      return await renderContentVideo(event.data);
    });

    return { videoPath };
  }
);
```
