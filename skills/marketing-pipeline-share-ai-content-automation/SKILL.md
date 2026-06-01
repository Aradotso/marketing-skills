---
name: marketing-pipeline-share-ai-content-automation
description: Automate content creation from research to video generation using Claude/OpenAI and Remotion for marketing pipelines
triggers:
  - how do I automate content creation with AI
  - generate video content from text automatically
  - create marketing content pipeline with Claude
  - research and write articles with AI automation
  - build automated content workflow with Remotion
  - scrape news and generate content automatically
  - create multilingual marketing content with AI
  - automate social media video generation
---

# Marketing Pipeline Share - AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

Complete content automation pipeline that handles research, scriptwriting, article generation, and video rendering. Built with Next.js, TypeScript, Claude/OpenAI APIs, and Remotion for video generation.

## What It Does

This system automates the entire content creation workflow:

1. **Auto-Research**: Crawls recent news from TechCrunch, a16z, X (Twitter), LinkedIn
2. **AI Content Generation**: Creates articles in multiple formats (toplist, POV, case study, how-to) using Claude/OpenAI
3. **Multilingual Support**: Generates content in both English and Vietnamese
4. **Video Rendering**: Converts articles to videos/infographics via Remotion
5. **Multi-Platform Export**: Outputs video in formats optimized for Reels, TikTok, Shorts

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
# AI Provider API Keys
OPENAI_API_KEY=your_openai_key_here
ANTHROPIC_API_KEY=your_claude_api_key_here

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key_here

# Database (if applicable)
DATABASE_URL=your_database_url_here

# Remotion Config
REMOTION_CONCURRENCY=4
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── research/    # Web scraping & data collection
│   │   ├── content/     # Content generation logic
│   │   └── video/       # Remotion video rendering
│   ├── remotion/        # Remotion compositions
│   └── types/           # TypeScript type definitions
├── public/              # Static assets
└── package.json
```

## Core API Usage

### Research & Data Collection

```typescript
import { researchTopic } from '@/lib/research/scraper';

// Scrape recent news about a topic
async function gatherResearch(keyword: string) {
  const research = await researchTopic({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeframe: '24h',
    maxResults: 20
  });

  return {
    articles: research.articles,
    insights: research.insights,
    trends: research.trends,
    statistics: research.statistics
  };
}
```

### AI Content Generation with Claude

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

async function generateArticle(
  topic: string,
  format: 'toplist' | 'pov' | 'case-study' | 'how-to',
  language: 'en' | 'vi',
  researchData: any
) {
  const prompt = `
You are an expert content creator. Generate a ${format} article about "${topic}".

Research Data:
${JSON.stringify(researchData, null, 2)}

Requirements:
- Language: ${language}
- Format: ${format}
- Include data-backed insights
- Engaging and SEO-optimized
- Target audience: marketers and business owners
`;

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [{
      role: 'user',
      content: prompt
    }]
  });

  return message.content[0].text;
}
```

### AI Content Generation with OpenAI

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateWithGPT(
  topic: string,
  tone: 'expert' | 'friendly' | 'humorous',
  researchData: any
) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are a ${tone} content creator specializing in marketing content.`
      },
      {
        role: 'user',
        content: `Create content about ${topic}. Research: ${JSON.stringify(researchData)}`
      }
    ],
    temperature: 0.7,
    max_tokens: 3000
  });

  return completion.choices[0].message.content;
}
```

### Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

async function generateVideo(
  articleContent: string,
  platform: 'reels' | 'tiktok' | 'shorts'
) {
  const platformConfig = {
    reels: { width: 1080, height: 1920, fps: 30 },
    tiktok: { width: 1080, height: 1920, fps: 30 },
    shorts: { width: 1080, height: 1920, fps: 30 }
  };

  const config = platformConfig[platform];

  // Bundle Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.join(process.cwd(), 'src/remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  // Get composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ArticleVideo',
    inputProps: {
      content: articleContent,
      platform
    },
  });

  // Render video
  const outputLocation = path.join(
    process.cwd(),
    'public/videos',
    `${Date.now()}-${platform}.mp4`
  );

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: {
      content: articleContent,
      platform
    },
  });

  return outputLocation;
}
```

### Complete Content Pipeline

```typescript
import { researchTopic } from '@/lib/research/scraper';
import { generateArticle } from '@/lib/ai/claude';
import { generateVideo } from '@/lib/video/remotion';

interface ContentPipelineOptions {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  languages: Array<'en' | 'vi'>;
  generateVideo: boolean;
  platforms?: Array<'reels' | 'tiktok' | 'shorts'>;
}

async function runContentPipeline(options: ContentPipelineOptions) {
  try {
    // Step 1: Research
    console.log('🔍 Researching topic...');
    const research = await researchTopic({
      keyword: options.keyword,
      sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
      timeframe: '24h',
      maxResults: 20
    });

    // Step 2: Generate articles
    console.log('✍️ Generating articles...');
    const articles = await Promise.all(
      options.languages.map(lang =>
        generateArticle(
          options.keyword,
          options.format,
          lang,
          research
        )
      )
    );

    const result = {
      research,
      articles: articles.map((content, i) => ({
        language: options.languages[i],
        content
      })),
      videos: []
    };

    // Step 3: Generate videos (optional)
    if (options.generateVideo && options.platforms) {
      console.log('🎬 Rendering videos...');
      const videos = await Promise.all(
        options.platforms.map(platform =>
          generateVideo(articles[0], platform)
        )
      );
      result.videos = videos;
    }

    console.log('✅ Pipeline complete!');
    return result;

  } catch (error) {
    console.error('❌ Pipeline failed:', error);
    throw error;
  }
}

// Usage example
const content = await runContentPipeline({
  keyword: 'AI marketing automation 2024',
  format: 'toplist',
  languages: ['en', 'vi'],
  generateVideo: true,
  platforms: ['reels', 'tiktok']
});
```

## Next.js API Routes

### Research API Endpoint

```typescript
// src/app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { researchTopic } from '@/lib/research/scraper';

export async function POST(request: NextRequest) {
  try {
    const { keyword, sources, timeframe } = await request.json();

    const research = await researchTopic({
      keyword,
      sources: sources || ['techcrunch', 'a16z'],
      timeframe: timeframe || '24h',
      maxResults: 20
    });

    return NextResponse.json({ success: true, data: research });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

### Content Generation API Endpoint

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { generateArticle } from '@/lib/ai/claude';

export async function POST(request: NextRequest) {
  try {
    const { topic, format, language, researchData } = await request.json();

    const article = await generateArticle(
      topic,
      format,
      language,
      researchData
    );

    return NextResponse.json({ success: true, content: article });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

### Video Rendering API Endpoint

```typescript
// src/app/api/render-video/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { generateVideo } from '@/lib/video/remotion';

export async function POST(request: NextRequest) {
  try {
    const { content, platform } = await request.json();

    const videoPath = await generateVideo(content, platform);

    return NextResponse.json({
      success: true,
      videoUrl: `/videos/${path.basename(videoPath)}`
    });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

## Running the Development Server

```bash
# Start Next.js development server
npm run dev

# Server runs at http://localhost:3000
```

## Remotion Commands

```bash
# Preview Remotion compositions
npm run remotion:preview

# Render a specific composition
npx remotion render src/remotion/index.ts ArticleVideo output.mp4

# Render with custom props
npx remotion render src/remotion/index.ts ArticleVideo output.mp4 \
  --props='{"content":"Your article text","platform":"reels"}'
```

## Common Patterns

### Batch Content Generation

```typescript
async function batchGenerateContent(keywords: string[]) {
  const results = [];

  for (const keyword of keywords) {
    const content = await runContentPipeline({
      keyword,
      format: 'toplist',
      languages: ['en', 'vi'],
      generateVideo: false
    });

    results.push({
      keyword,
      timestamp: new Date().toISOString(),
      ...content
    });

    // Rate limiting delay
    await new Promise(resolve => setTimeout(resolve, 2000));
  }

  return results;
}
```

### Scheduled Content Generation

```typescript
// Using node-cron for scheduling
import cron from 'node-cron';

// Run content generation daily at 9 AM
cron.schedule('0 9 * * *', async () => {
  console.log('🤖 Running scheduled content generation...');

  const trendingTopics = await getTrendingTopics();

  for (const topic of trendingTopics) {
    await runContentPipeline({
      keyword: topic,
      format: 'pov',
      languages: ['en'],
      generateVideo: true,
      platforms: ['reels']
    });
  }
});
```

### Custom Remotion Composition

```typescript
// src/remotion/ArticleVideo.tsx
import { AbsoluteFill, useCurrentFrame, interpolate } from 'remotion';

interface ArticleVideoProps {
  content: string;
  platform: 'reels' | 'tiktok' | 'shorts';
}

export const ArticleVideo: React.FC<ArticleVideoProps> = ({
  content,
  platform
}) => {
  const frame = useCurrentFrame();

  const opacity = interpolate(frame, [0, 30], [0, 1], {
    extrapolateRight: 'clamp'
  });

  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a1a' }}>
      <div style={{ opacity, padding: 40, color: 'white' }}>
        <h1 style={{ fontSize: 48, marginBottom: 20 }}>
          {content.split('\n')[0]}
        </h1>
        <p style={{ fontSize: 24, lineHeight: 1.6 }}>
          {content.split('\n').slice(1).join('\n')}
        </p>
      </div>
    </AbsoluteFill>
  );
};
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement exponential backoff
async function withRetry<T>(
  fn: () => Promise<T>,
  maxRetries = 3
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (i === maxRetries - 1) throw error;

      const delay = Math.pow(2, i) * 1000;
      console.log(`Retry ${i + 1}/${maxRetries} after ${delay}ms`);
      await new Promise(resolve => setTimeout(resolve, delay));
    }
  }
  throw new Error('Max retries exceeded');
}

// Usage
const article = await withRetry(() =>
  generateArticle(topic, format, language, research)
);
```

### Memory Issues with Video Rendering

```typescript
// Limit concurrent renders
import pLimit from 'p-limit';

const limit = pLimit(2); // Max 2 concurrent renders

const videos = await Promise.all(
  platforms.map(platform =>
    limit(() => generateVideo(content, platform))
  )
);
```

### Environment Variable Validation

```typescript
function validateEnv() {
  const required = [
    'OPENAI_API_KEY',
    'ANTHROPIC_API_KEY',
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

## Type Definitions

```typescript
// src/types/content.ts
export type ContentFormat = 'toplist' | 'pov' | 'case-study' | 'how-to';
export type Language = 'en' | 'vi';
export type Platform = 'reels' | 'tiktok' | 'shorts';
export type Tone = 'expert' | 'friendly' | 'humorous';

export interface ResearchData {
  articles: Article[];
  insights: string[];
  trends: Trend[];
  statistics: Statistic[];
}

export interface Article {
  title: string;
  content: string;
  language: Language;
  format: ContentFormat;
  createdAt: string;
}

export interface VideoOutput {
  platform: Platform;
  path: string;
  duration: number;
  dimensions: { width: number; height: number };
}
```
