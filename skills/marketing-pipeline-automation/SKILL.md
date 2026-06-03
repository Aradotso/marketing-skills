---
name: marketing-pipeline-automation
description: AI-powered content automation pipeline for research, scriptwriting, and video generation using Claude/OpenAI and Remotion
triggers:
  - automate content creation with AI
  - generate marketing content automatically
  - create video content from text
  - research and write blog posts with AI
  - build content pipeline with Claude
  - automate social media content generation
  - create videos with Remotion from scripts
  - set up AI marketing automation
---

# Marketing Pipeline Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

Ultimate AI Content Pipeline is a comprehensive TypeScript-based automation system that handles the entire content creation workflow: from researching trending topics, generating multi-format scripts (blog posts, case studies, how-tos), to automatically rendering videos using Remotion. It leverages Claude 3, OpenAI, and web scraping to create data-backed, multilingual content.

## What It Does

This system automates:
- **Auto-Research**: Crawls recent news from TechCrunch, a16z, Twitter/X, LinkedIn (last 24h)
- **AI Content Generation**: Creates blog posts in multiple formats (toplist, POV, case study, how-to) using Claude/OpenAI
- **Multilingual Output**: Generates content in both English and Vietnamese with customizable tone
- **Video Generation**: Renders infographics and short-form videos using Remotion for TikTok, Reels, Shorts
- **Next.js Interface**: Web UI for managing the entire pipeline

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
# AI API Keys
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# RapidAPI for web scraping
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_url

# Next.js
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app router
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── scraper/     # Web scraping modules
│   │   └── video/       # Remotion video generation
│   └── types/           # TypeScript type definitions
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Core Usage Patterns

### 1. Research & Scraping

```typescript
import { searchRecentNews } from '@/lib/scraper/news-aggregator';
import { analyzeContent } from '@/lib/ai/content-analyzer';

// Scrape recent news on a topic
async function researchTopic(keyword: string) {
  const newsArticles = await searchRecentNews({
    query: keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeframe: '24h'
  });

  // Extract insights using AI
  const insights = await analyzeContent(newsArticles, {
    provider: 'anthropic', // or 'openai'
    model: 'claude-3-opus-20240229',
    extractInsights: true,
    generateStats: true
  });

  return insights;
}
```

### 2. Content Generation

```typescript
import { generateContent } from '@/lib/ai/content-generator';

// Generate blog post in multiple formats
async function createBlogPost(topic: string, format: 'toplist' | 'pov' | 'case-study' | 'how-to') {
  const content = await generateContent({
    topic,
    format,
    languages: ['en', 'vi'],
    tone: 'expert', // or 'friendly', 'humorous'
    includeData: true,
    research: await researchTopic(topic),
    provider: 'anthropic',
    apiKey: process.env.ANTHROPIC_API_KEY
  });

  return {
    english: content.en,
    vietnamese: content.vi,
    metadata: content.metadata
  };
}
```

### 3. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import { webpackOverride } from './remotion/webpack-override';

// Render video from generated content
async function generateVideo(content: any, format: 'reels' | 'tiktok' | 'shorts') {
  const bundleLocation = await bundle({
    entryPoint: './remotion/index.ts',
    webpackOverride
  });

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: format === 'reels' ? 'InstagramReel' : 'ShortVideo',
    inputProps: {
      title: content.title,
      points: content.keyPoints,
      branding: {
        logo: '/logo.png',
        colors: ['#FF6B6B', '#4ECDC4']
      }
    }
  });

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: `out/${format}-${Date.now()}.mp4`,
    inputProps: composition.inputProps
  });
}
```

### 4. Complete Pipeline Workflow

```typescript
import { researchTopic } from '@/lib/research';
import { createBlogPost } from '@/lib/content';
import { generateVideo } from '@/lib/video';
import { schedulePost } from '@/lib/social-media';

// Full automation pipeline
async function runContentPipeline(keyword: string) {
  try {
    // Step 1: Research
    console.log('🔍 Researching topic...');
    const research = await researchTopic(keyword);

    // Step 2: Generate content in multiple formats
    console.log('✍️ Generating content...');
    const blogPost = await createBlogPost(keyword, 'toplist');
    const caseStudy = await createBlogPost(keyword, 'case-study');

    // Step 3: Create videos
    console.log('🎬 Rendering videos...');
    await generateVideo(blogPost.english, 'reels');
    await generateVideo(blogPost.english, 'tiktok');

    // Step 4: Schedule posts (optional)
    console.log('📅 Scheduling posts...');
    await schedulePost({
      content: blogPost.english,
      platforms: ['facebook', 'linkedin'],
      scheduledTime: new Date(Date.now() + 3600000) // 1 hour from now
    });

    return {
      success: true,
      outputs: {
        research,
        blogPost,
        caseStudy,
        videos: ['reels', 'tiktok']
      }
    };
  } catch (error) {
    console.error('Pipeline failed:', error);
    throw error;
  }
}
```

## API Routes (Next.js)

### Generate Content Endpoint

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { generateContent } from '@/lib/ai/content-generator';

export async function POST(request: NextRequest) {
  const { topic, format, language } = await request.json();

  try {
    const content = await generateContent({
      topic,
      format,
      languages: [language],
      tone: 'expert',
      provider: 'anthropic',
      apiKey: process.env.ANTHROPIC_API_KEY
    });

    return NextResponse.json({ success: true, content });
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
// app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { searchRecentNews } from '@/lib/scraper/news-aggregator';

export async function POST(request: NextRequest) {
  const { keyword, sources } = await request.json();

  try {
    const articles = await searchRecentNews({
      query: keyword,
      sources: sources || ['techcrunch', 'a16z'],
      timeframe: '24h'
    });

    return NextResponse.json({ success: true, articles });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

## Development Commands

```bash
# Start Next.js development server
npm run dev

# Build for production
npm run build

# Start production server
npm run start

# Render Remotion video locally
npm run remotion:render

# Preview Remotion composition
npm run remotion:preview

# Type checking
npm run type-check

# Linting
npm run lint
```

## Remotion Video Templates

### Basic Infographic Template

```typescript
// remotion/compositions/Infographic.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig } from 'remotion';

interface InfographicProps {
  title: string;
  points: string[];
  branding: {
    logo: string;
    colors: string[];
  };
}

export const Infographic: React.FC<InfographicProps> = ({ title, points, branding }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  const opacity = Math.min(1, frame / (fps * 0.5));

  return (
    <AbsoluteFill style={{ backgroundColor: branding.colors[0] }}>
      <div style={{ 
        padding: 60, 
        opacity,
        display: 'flex',
        flexDirection: 'column',
        justifyContent: 'center'
      }}>
        <h1 style={{ fontSize: 72, color: 'white', marginBottom: 40 }}>
          {title}
        </h1>
        {points.map((point, index) => (
          <div
            key={index}
            style={{
              fontSize: 36,
              color: 'white',
              marginBottom: 20,
              opacity: frame > (index + 1) * fps ? 1 : 0,
              transform: `translateY(${frame > (index + 1) * fps ? 0 : 20}px)`,
              transition: 'all 0.3s ease'
            }}
          >
            ✓ {point}
          </div>
        ))}
      </div>
    </AbsoluteFill>
  );
};
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement retry logic with exponential backoff
async function callAIWithRetry(prompt: string, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await generateContent({ topic: prompt });
    } catch (error) {
      if (error.status === 429) {
        const delay = Math.pow(2, i) * 1000;
        console.log(`Rate limited. Retrying in ${delay}ms...`);
        await new Promise(resolve => setTimeout(resolve, delay));
      } else {
        throw error;
      }
    }
  }
  throw new Error('Max retries exceeded');
}
```

### Memory Issues with Remotion

```typescript
// Use smaller compositions or chunk rendering
const composition = await selectComposition({
  serveUrl: bundleLocation,
  id: 'ShortVideo',
  inputProps: {
    // Limit data size
    points: content.keyPoints.slice(0, 5)
  }
});

// Render with concurrency limit
await renderMedia({
  composition,
  serveUrl: bundleLocation,
  codec: 'h264',
  outputLocation: `out/video.mp4`,
  concurrency: 2 // Reduce from default
});
```

### Scraping Failures

```typescript
// Implement fallback sources
async function robustResearch(keyword: string) {
  const primarySources = ['techcrunch', 'a16z'];
  const fallbackSources = ['twitter', 'linkedin'];

  try {
    return await searchRecentNews({ query: keyword, sources: primarySources });
  } catch (error) {
    console.warn('Primary sources failed, trying fallback...');
    return await searchRecentNews({ query: keyword, sources: fallbackSources });
  }
}
```

## Best Practices

1. **Cache Research Results**: Store scraped data to avoid redundant API calls
2. **Queue Video Rendering**: Use a job queue (Bull, BullMQ) for heavy Remotion tasks
3. **Content Validation**: Always validate AI-generated content before publishing
4. **Monitor Costs**: Track AI API usage to control expenses
5. **Version Control Prompts**: Store AI prompts in separate files for easy iteration

```typescript
// Example caching pattern
import { Redis } from 'ioredis';

const redis = new Redis(process.env.REDIS_URL);

async function cachedResearch(keyword: string) {
  const cacheKey = `research:${keyword}:24h`;
  const cached = await redis.get(cacheKey);

  if (cached) return JSON.parse(cached);

  const fresh = await researchTopic(keyword);
  await redis.setex(cacheKey, 86400, JSON.stringify(fresh)); // 24h TTL

  return fresh;
}
```
