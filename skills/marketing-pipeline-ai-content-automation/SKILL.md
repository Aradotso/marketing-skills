---
name: marketing-pipeline-ai-content-automation
description: Automated content pipeline from research to video generation using Claude, OpenAI, and Remotion
triggers:
  - automate content creation with AI research and video
  - generate marketing content from keywords automatically
  - create social media videos from AI-written articles
  - build automated content pipeline with Claude
  - set up AI-powered marketing automation system
  - crawl news and generate video content automatically
  - use Remotion to render marketing videos
  - automate content research and script generation
---

# Marketing Pipeline AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This project is a comprehensive AI-powered content automation pipeline that handles the entire content creation workflow: from researching trending topics, generating scripts in multiple formats and languages, to automatically rendering videos using Remotion. Built with TypeScript, Next.js, and integrated with Claude 3, OpenAI, and various data sources (TechCrunch, X/Twitter, LinkedIn, a16z).

## What It Does

The Ultimate AI Content Pipeline automates:
- **Auto-research**: Crawls recent news from major tech sources (TechCrunch, a16z, social media)
- **AI content generation**: Creates articles in multiple formats (toplist, POV, case study, how-to) using Claude/OpenAI
- **Multi-language support**: Generates content in English and Vietnamese with customizable tone
- **Video rendering**: Converts written content into videos/infographics using Remotion
- **Platform optimization**: Exports videos optimized for Reels, TikTok, Shorts

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
# AI Services
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Data Sources
RAPIDAPI_KEY=your_rapidapi_key

# Optional: Social Media APIs
TWITTER_API_KEY=your_twitter_api_key
LINKEDIN_API_KEY=your_linkedin_api_key

# Application
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

## Running the Application

```bash
# Development mode
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Render videos with Remotion
npm run render
```

## Core Architecture

### Directory Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/             
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── crawlers/    # News crawling modules
│   │   ├── content/     # Content generation logic
│   │   └── video/       # Remotion video generation
│   ├── remotion/        # Remotion video templates
│   └── utils/           # Utility functions
├── public/              # Static assets
└── package.json
```

## Key Usage Patterns

### 1. Research & Data Crawling

```typescript
import { crawlTechNews } from '@/lib/crawlers/tech-news';
import { crawlSocialMedia } from '@/lib/crawlers/social-media';

async function gatherResearch(keyword: string) {
  const [techNews, socialPosts] = await Promise.all([
    crawlTechNews(keyword, { 
      sources: ['techcrunch', 'a16z'],
      timeframe: '24h' 
    }),
    crawlSocialMedia(keyword, {
      platforms: ['twitter', 'linkedin'],
      limit: 50
    })
  ]);

  return {
    articles: techNews,
    socialInsights: socialPosts,
    timestamp: new Date()
  };
}
```

### 2. AI Content Generation with Claude

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

async function generateArticle(
  research: any, 
  format: 'toplist' | 'pov' | 'case-study' | 'how-to',
  language: 'en' | 'vi'
) {
  const prompt = buildPrompt(research, format, language);
  
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [{
      role: 'user',
      content: prompt
    }],
    temperature: 0.7
  });

  return {
    content: message.content[0].text,
    format,
    language,
    wordCount: message.content[0].text.split(' ').length
  };
}

function buildPrompt(research: any, format: string, language: string): string {
  const basePrompt = `Based on this research data: ${JSON.stringify(research)}
  
Create a ${format} article in ${language === 'en' ? 'English' : 'Vietnamese'}.

Requirements:
- Data-backed insights with specific numbers
- Engaging tone for social media
- 800-1200 words
- Include actionable takeaways`;

  return basePrompt;
}
```

### 3. OpenAI Integration (Alternative)

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY
});

async function generateWithGPT(
  topic: string, 
  tone: 'expert' | 'friendly' | 'humorous'
) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are a ${tone} content writer specializing in marketing and tech trends.`
      },
      {
        role: 'user',
        content: `Write an engaging article about: ${topic}`
      }
    ],
    temperature: tone === 'humorous' ? 0.9 : 0.7,
    max_tokens: 2000
  });

  return completion.choices[0].message.content;
}
```

### 4. Remotion Video Generation

```typescript
// src/remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig } from 'remotion';

export const ContentVideo: React.FC<{
  title: string;
  points: string[];
  brandColor: string;
}> = ({ title, points, brandColor }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  const opacity = Math.min(1, frame / (fps * 0.5));

  return (
    <AbsoluteFill style={{ backgroundColor: brandColor }}>
      <div style={{ 
        padding: 60, 
        opacity,
        transform: `translateY(${Math.max(0, 30 - frame)}px)`
      }}>
        <h1 style={{ fontSize: 72, color: 'white', marginBottom: 40 }}>
          {title}
        </h1>
        {points.map((point, i) => (
          <p 
            key={i} 
            style={{ 
              fontSize: 36, 
              color: 'white',
              opacity: frame > (i + 1) * fps ? 1 : 0,
              transition: 'opacity 0.5s'
            }}
          >
            {point}
          </p>
        ))}
      </div>
    </AbsoluteFill>
  );
};
```

```typescript
// Render video programmatically
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

async function renderContentVideo(articleData: any) {
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./src/remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title: articleData.title,
      points: articleData.keyPoints,
      brandColor: '#6366f1'
    }
  });

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: `out/${articleData.slug}.mp4`,
    inputProps: composition.props,
  });

  console.log('Video rendered successfully!');
}
```

### 5. Complete Pipeline Workflow

```typescript
// src/lib/pipeline/content-pipeline.ts
export async function runContentPipeline(keyword: string) {
  try {
    // Step 1: Research
    console.log('🔍 Gathering research...');
    const research = await gatherResearch(keyword);

    // Step 2: Generate content
    console.log('✍️ Generating content...');
    const [englishArticle, vietnameseArticle] = await Promise.all([
      generateArticle(research, 'toplist', 'en'),
      generateArticle(research, 'toplist', 'vi')
    ]);

    // Step 3: Extract key points for video
    const keyPoints = extractKeyPoints(englishArticle.content);

    // Step 4: Render video
    console.log('🎬 Rendering video...');
    await renderContentVideo({
      title: keyword,
      keyPoints,
      slug: keyword.toLowerCase().replace(/\s+/g, '-')
    });

    return {
      success: true,
      articles: { english: englishArticle, vietnamese: vietnameseArticle },
      videoPath: `out/${keyword.toLowerCase().replace(/\s+/g, '-')}.mp4`
    };
  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}

function extractKeyPoints(content: string): string[] {
  // Extract bullet points or numbered lists from content
  const lines = content.split('\n');
  return lines
    .filter(line => line.match(/^[\d\-\*\.]/))
    .map(line => line.replace(/^[\d\-\*\.\s]+/, ''))
    .slice(0, 5);
}
```

### 6. API Route Example (Next.js)

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline/content-pipeline';

export async function POST(request: NextRequest) {
  try {
    const { keyword, format, languages } = await request.json();

    if (!keyword) {
      return NextResponse.json(
        { error: 'Keyword is required' },
        { status: 400 }
      );
    }

    const result = await runContentPipeline(keyword);

    return NextResponse.json({
      success: true,
      data: result
    });
  } catch (error) {
    console.error('API Error:', error);
    return NextResponse.json(
      { error: 'Content generation failed' },
      { status: 500 }
    );
  }
}
```

## Common Patterns

### Caching Research Data

```typescript
import { Redis } from '@upstash/redis';

const redis = new Redis({
  url: process.env.UPSTASH_REDIS_URL!,
  token: process.env.UPSTASH_REDIS_TOKEN!,
});

async function getCachedResearch(keyword: string) {
  const cacheKey = `research:${keyword}`;
  const cached = await redis.get(cacheKey);
  
  if (cached) {
    return JSON.parse(cached as string);
  }

  const fresh = await gatherResearch(keyword);
  await redis.set(cacheKey, JSON.stringify(fresh), { ex: 3600 }); // 1 hour
  
  return fresh;
}
```

### Batch Processing

```typescript
async function generateBatchContent(keywords: string[]) {
  const results = [];
  
  for (const keyword of keywords) {
    const result = await runContentPipeline(keyword);
    results.push(result);
    
    // Rate limiting
    await new Promise(resolve => setTimeout(resolve, 2000));
  }
  
  return results;
}
```

### Custom Video Templates

```typescript
// Multiple aspect ratios
const VIDEO_CONFIGS = {
  reels: { width: 1080, height: 1920, fps: 30 }, // 9:16
  youtube: { width: 1920, height: 1080, fps: 30 }, // 16:9
  square: { width: 1080, height: 1080, fps: 30 }  // 1:1
};

async function renderMultiPlatform(content: any) {
  return Promise.all(
    Object.entries(VIDEO_CONFIGS).map(([platform, config]) =>
      renderMedia({
        composition: { ...composition, ...config },
        outputLocation: `out/${content.slug}-${platform}.mp4`,
      })
    )
  );
}
```

## Troubleshooting

### AI API Rate Limits

```typescript
import pRetry from 'p-retry';

async function generateWithRetry(prompt: string) {
  return pRetry(
    async () => {
      try {
        return await anthropic.messages.create({
          model: 'claude-3-5-sonnet-20241022',
          max_tokens: 4096,
          messages: [{ role: 'user', content: prompt }]
        });
      } catch (error: any) {
        if (error.status === 429) {
          throw error; // Retry on rate limit
        }
        throw new pRetry.AbortError(error); // Don't retry other errors
      }
    },
    {
      retries: 3,
      minTimeout: 1000,
      factor: 2
    }
  );
}
```

### Memory Issues with Video Rendering

```typescript
// Render videos in smaller chunks
async function renderLargeVideo(segments: any[]) {
  const tempFiles = [];
  
  for (let i = 0; i < segments.length; i++) {
    const tempFile = `temp-${i}.mp4`;
    await renderMedia({
      /* config */
      outputLocation: tempFile
    });
    tempFiles.push(tempFile);
  }
  
  // Concatenate using ffmpeg
  await concatenateVideos(tempFiles, 'final-output.mp4');
}
```

### Crawler Blocked

```typescript
// Use rotating proxies and user agents
const USER_AGENTS = [
  'Mozilla/5.0 (Windows NT 10.0; Win64; x64)...',
  'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7)...',
];

async function crawlWithRetries(url: string) {
  const headers = {
    'User-Agent': USER_AGENTS[Math.floor(Math.random() * USER_AGENTS.length)],
  };
  
  // Add delay between requests
  await new Promise(resolve => setTimeout(resolve, 1000));
  
  return fetch(url, { headers });
}
```

## Performance Tips

- Cache research data for 1-6 hours to reduce API calls
- Use streaming responses for real-time content generation
- Batch video renders during off-peak hours
- Implement queue system (Bull, BullMQ) for high-volume processing
- Store rendered videos in CDN (Cloudflare R2, AWS S3)
