---
name: marketing-pipeline-automation
description: AI-powered content automation pipeline for research, scriptwriting, and video generation with Claude/OpenAI integration
triggers:
  - automate content creation with AI
  - generate marketing content from keywords
  - create videos from blog posts automatically
  - scrape news for content research
  - build automated content pipeline
  - generate multilingual marketing content
  - render videos with Remotion from text
  - automate social media content workflow
---

# Marketing Pipeline Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the Ultimate AI Content Pipeline, a TypeScript-based system that automates content creation from research through video generation using Claude, OpenAI, and Remotion.

## What This Project Does

The Marketing Pipeline is an end-to-end content automation system that:

- **Auto-scrapes** trending news from sources like TechCrunch, a16z, Twitter/X, LinkedIn
- **Generates** multi-format content (listicles, POV pieces, case studies, how-tos) in multiple languages
- **Renders** videos and infographics automatically using Remotion
- **Optimizes** content for multiple platforms (Reels, TikTok, Shorts)

The pipeline transforms a single keyword into complete content packages including articles, images, and videos.

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
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# Research/Scraping APIs
RAPIDAPI_KEY=your_rapidapi_key

# Optional: Video Rendering
REMOTION_LICENSE_KEY=your_remotion_license

# Database (if applicable)
DATABASE_URL=your_database_url

# Next.js Configuration
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── scraper/     # News scraping logic
│   │   ├── content/     # Content generation
│   │   └── video/       # Remotion video rendering
│   └── utils/           # Utility functions
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Core API Usage

### 1. Content Research & Scraping

```typescript
import { scrapeRecentNews } from '@/lib/scraper/news-scraper';

async function researchTopic(keyword: string) {
  const newsData = await scrapeRecentNews({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter'],
    timeframe: '24h',
    limit: 20
  });

  return {
    articles: newsData.articles,
    insights: newsData.extractedInsights,
    trends: newsData.trendingTopics
  };
}

// Usage
const research = await researchTopic('AI automation');
```

### 2. AI Content Generation with Claude

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

async function generateContentWithClaude(
  topic: string,
  format: 'toplist' | 'pov' | 'case-study' | 'how-to',
  language: 'en' | 'vi'
) {
  const prompt = `Create a ${format} article about ${topic} in ${language}. 
  Include data-backed insights, actionable takeaways, and engaging storytelling.`;

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [
      {
        role: 'user',
        content: prompt
      }
    ],
  });

  return message.content[0].text;
}

// Usage
const article = await generateContentWithClaude(
  'AI in marketing',
  'toplist',
  'en'
);
```

### 3. OpenAI Content Generation

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateWithOpenAI(
  researchData: any,
  tone: 'professional' | 'friendly' | 'humorous'
) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are a ${tone} content writer. Create engaging marketing content.`
      },
      {
        role: 'user',
        content: `Based on this research: ${JSON.stringify(researchData)}, 
        create a comprehensive article.`
      }
    ],
    temperature: 0.7,
  });

  return completion.choices[0].message.content;
}
```

### 4. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

async function renderContentVideo(
  articleData: {
    title: string;
    keyPoints: string[];
    images: string[];
  },
  platform: 'reels' | 'tiktok' | 'shorts'
) {
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title: articleData.title,
      keyPoints: articleData.keyPoints,
      images: articleData.images,
      aspectRatio: platform === 'reels' ? '9:16' : '16:9',
    },
  });

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: `out/${platform}-video.mp4`,
  });

  return `out/${platform}-video.mp4`;
}
```

## Complete Pipeline Example

```typescript
import { scrapeRecentNews } from '@/lib/scraper/news-scraper';
import { generateContentWithClaude } from '@/lib/ai/claude';
import { renderContentVideo } from '@/lib/video/render';
import { publishToSocial } from '@/lib/publishing/social';

async function runFullPipeline(keyword: string) {
  // Step 1: Research
  console.log('🔍 Researching topic...');
  const research = await scrapeRecentNews({
    keyword,
    sources: ['techcrunch', 'twitter'],
    timeframe: '24h',
  });

  // Step 2: Generate Content (English & Vietnamese)
  console.log('✍️ Generating content...');
  const [contentEN, contentVI] = await Promise.all([
    generateContentWithClaude(keyword, 'toplist', 'en'),
    generateContentWithClaude(keyword, 'toplist', 'vi'),
  ]);

  // Step 3: Extract key points for video
  const keyPoints = extractKeyPoints(contentEN);

  // Step 4: Render videos for multiple platforms
  console.log('🎬 Rendering videos...');
  const videos = await Promise.all([
    renderContentVideo(
      { title: keyword, keyPoints, images: research.images },
      'reels'
    ),
    renderContentVideo(
      { title: keyword, keyPoints, images: research.images },
      'tiktok'
    ),
  ]);

  // Step 5: Schedule publishing
  console.log('📤 Scheduling posts...');
  await publishToSocial({
    contentEN,
    contentVI,
    videos,
    platforms: ['facebook', 'instagram', 'tiktok'],
  });

  return {
    articles: { en: contentEN, vi: contentVI },
    videos,
    research,
  };
}

// Execute pipeline
runFullPipeline('AI marketing automation').then((result) => {
  console.log('✅ Pipeline complete!', result);
});
```

## CLI Commands

```bash
# Development
npm run dev          # Start Next.js dev server
npm run build        # Build for production
npm run start        # Start production server

# Content Generation
npm run generate -- --keyword "AI trends" --format toplist
npm run research -- --topic "marketing automation" --sources techcrunch,twitter

# Video Rendering
npm run render:video -- --input article.json --platform reels
npm run render:batch -- --folder content/articles
```

## API Routes (Next.js)

### Generate Content Endpoint

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { generateContentWithClaude } from '@/lib/ai/claude';

export async function POST(req: NextRequest) {
  try {
    const { keyword, format, language } = await req.json();

    if (!keyword) {
      return NextResponse.json(
        { error: 'Keyword is required' },
        { status: 400 }
      );
    }

    const content = await generateContentWithClaude(
      keyword,
      format || 'toplist',
      language || 'en'
    );

    return NextResponse.json({
      success: true,
      content,
      metadata: {
        keyword,
        format,
        language,
        timestamp: new Date().toISOString(),
      },
    });
  } catch (error) {
    return NextResponse.json(
      { error: 'Content generation failed' },
      { status: 500 }
    );
  }
}
```

### Research Endpoint

```typescript
// src/app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { scrapeRecentNews } from '@/lib/scraper/news-scraper';

export async function POST(req: NextRequest) {
  const { keyword, sources, timeframe } = await req.json();

  const results = await scrapeRecentNews({
    keyword,
    sources: sources || ['techcrunch', 'twitter'],
    timeframe: timeframe || '24h',
    limit: 20,
  });

  return NextResponse.json({
    success: true,
    data: results,
  });
}
```

## Common Patterns

### Multi-Language Content Generation

```typescript
async function generateMultiLingualContent(
  keyword: string,
  languages: string[] = ['en', 'vi']
) {
  const contentPromises = languages.map((lang) =>
    generateContentWithClaude(keyword, 'toplist', lang as 'en' | 'vi')
  );

  const contents = await Promise.all(contentPromises);

  return languages.reduce((acc, lang, idx) => {
    acc[lang] = contents[idx];
    return acc;
  }, {} as Record<string, string>);
}
```

### Batch Processing

```typescript
async function batchGenerateContent(keywords: string[]) {
  const results = [];

  for (const keyword of keywords) {
    const result = await runFullPipeline(keyword);
    results.push({ keyword, ...result });

    // Rate limiting
    await new Promise((resolve) => setTimeout(resolve, 2000));
  }

  return results;
}
```

### Content Scheduling

```typescript
import { schedulePost } from '@/lib/publishing/scheduler';

async function scheduleContentCalendar(
  content: any,
  scheduleConfig: {
    platform: string;
    publishAt: Date;
  }[]
) {
  return Promise.all(
    scheduleConfig.map((config) =>
      schedulePost({
        content,
        platform: config.platform,
        scheduledTime: config.publishAt,
      })
    )
  );
}
```

## Configuration

### Content Generation Config

```typescript
// config/content.config.ts
export const contentConfig = {
  formats: ['toplist', 'pov', 'case-study', 'how-to'],
  languages: ['en', 'vi'],
  tones: ['professional', 'friendly', 'humorous'],
  defaultWordCount: {
    toplist: 1500,
    pov: 1200,
    'case-study': 2000,
    'how-to': 1800,
  },
};
```

### Scraper Config

```typescript
// config/scraper.config.ts
export const scraperConfig = {
  sources: {
    techcrunch: {
      url: 'https://techcrunch.com',
      selector: '.post-block',
      rateLimit: 1000, // ms between requests
    },
    twitter: {
      apiEndpoint: 'https://api.twitter.com/2',
      maxResults: 50,
    },
  },
  timeframes: {
    '24h': 86400000,
    '7d': 604800000,
    '30d': 2592000000,
  },
};
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement exponential backoff
async function retryWithBackoff(
  fn: () => Promise<any>,
  maxRetries: number = 3
) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error: any) {
      if (error.status === 429 && i < maxRetries - 1) {
        const delay = Math.pow(2, i) * 1000;
        await new Promise((resolve) => setTimeout(resolve, delay));
        continue;
      }
      throw error;
    }
  }
}
```

### Video Rendering Memory Issues

```typescript
// Render in chunks for large batches
async function renderInBatches(items: any[], batchSize: number = 3) {
  const results = [];

  for (let i = 0; i < items.length; i += batchSize) {
    const batch = items.slice(i, i + batchSize);
    const batchResults = await Promise.all(
      batch.map((item) => renderContentVideo(item, 'reels'))
    );
    results.push(...batchResults);

    // Garbage collection hint
    if (global.gc) global.gc();
  }

  return results;
}
```

### Missing Environment Variables

```typescript
// Validate env vars on startup
function validateEnv() {
  const required = [
    'OPENAI_API_KEY',
    'ANTHROPIC_API_KEY',
    'RAPIDAPI_KEY',
  ];

  const missing = required.filter((key) => !process.env[key]);

  if (missing.length > 0) {
    throw new Error(
      `Missing required environment variables: ${missing.join(', ')}`
    );
  }
}

// Call in app initialization
validateEnv();
```

## Best Practices

1. **Always cache research data** to avoid redundant scraping
2. **Use streaming responses** for long-form content generation
3. **Implement proper error handling** for AI API failures
4. **Rate limit all external API calls** to avoid bans
5. **Store generated content** before rendering videos
6. **Use webhooks** for asynchronous video rendering notifications
7. **Monitor API costs** with usage tracking middleware
