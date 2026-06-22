---
name: marketing-pipeline-share-content-automation
description: AI-powered content pipeline that researches, generates scripts, and creates videos automatically using Claude/OpenAI and Remotion
triggers:
  - automate my content creation workflow
  - generate social media content with AI
  - create videos from text automatically
  - build an AI content pipeline
  - research and write articles with AI
  - set up automated video generation
  - use marketing pipeline share for content
  - generate multilingual content with AI
---

# Marketing Pipeline Share — AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI agents to help developers implement the **Ultimate AI Content Pipeline** (marketing-pipeline-share), a complete content automation system that handles research, scriptwriting, article generation, and video rendering using AI (Claude 3, OpenAI) and Remotion.

## What It Does

The Marketing Pipeline Share automates the entire content creation workflow:

1. **Auto-Research**: Crawls recent news from TechCrunch, a16z, Twitter/X, LinkedIn (24h data)
2. **AI Content Generation**: Creates articles in multiple formats (toplist, POV, case study, how-to) using Claude/OpenAI
3. **Multi-language Support**: Generates content in English and Vietnamese simultaneously
4. **Video Rendering**: Automatically creates infographics and short videos using Remotion
5. **Platform Optimization**: Outputs video in formats optimized for Reels, TikTok, Shorts

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
# AI APIs
OPENAI_API_KEY=your_openai_key_here
ANTHROPIC_API_KEY=your_claude_key_here

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key_here
TWITTER_API_KEY=your_twitter_key_here

# Database (if applicable)
DATABASE_URL=your_database_url_here

# Remotion License (for video rendering)
REMOTION_LICENSE_KEY=your_remotion_key_here

# Next.js
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
│   │   ├── research/    # Web scraping and research
│   │   ├── video/       # Remotion video generation
│   │   └── content/     # Content formatting
│   └── types/           # TypeScript types
├── remotion/            # Remotion video templates
├── public/              # Static assets
└── .env.local          # Environment variables
```

## Core API Usage

### 1. Research Module

Automatically scrape and analyze recent content:

```typescript
import { researchTopic } from '@/lib/research/scraper';
import { analyzeInsights } from '@/lib/research/analyzer';

async function gatherResearch(keyword: string) {
  // Scrape latest news and posts
  const rawData = await researchTopic({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeframe: '24h'
  });

  // Extract insights
  const insights = await analyzeInsights(rawData, {
    model: 'claude-3-sonnet-20240229',
    apiKey: process.env.ANTHROPIC_API_KEY
  });

  return {
    rawData,
    insights,
    timestamp: new Date().toISOString()
  };
}
```

### 2. AI Content Generation

Generate articles in multiple formats:

```typescript
import { generateContent } from '@/lib/ai/content-generator';

async function createArticle(topic: string, researchData: any) {
  const content = await generateContent({
    topic,
    format: 'toplist', // or 'pov', 'case-study', 'how-to'
    languages: ['en', 'vi'],
    tone: 'expert', // or 'friendly', 'humorous'
    research: researchData,
    aiProvider: 'claude', // or 'openai'
    apiKey: process.env.ANTHROPIC_API_KEY
  });

  return {
    english: content.en,
    vietnamese: content.vi,
    metadata: {
      wordCount: content.wordCount,
      readingTime: content.readingTime,
      keywords: content.keywords
    }
  };
}
```

### 3. Video Generation with Remotion

Render videos from article content:

```typescript
import { renderVideo } from '@/lib/video/renderer';
import { createVideoScript } from '@/lib/video/script-generator';

async function generateVideo(article: any) {
  // Create video script from article
  const script = await createVideoScript({
    content: article.english,
    style: 'infographic',
    duration: 60, // seconds
    platform: 'reels' // or 'tiktok', 'shorts'
  });

  // Render video using Remotion
  const video = await renderVideo({
    script,
    composition: 'ContentVideo',
    outputFormat: 'mp4',
    fps: 30,
    width: 1080,
    height: 1920, // 9:16 for vertical video
    licenseKey: process.env.REMOTION_LICENSE_KEY
  });

  return {
    videoUrl: video.url,
    thumbnail: video.thumbnail,
    duration: video.duration
  };
}
```

### 4. Complete Pipeline Orchestration

Full end-to-end automation:

```typescript
import { ContentPipeline } from '@/lib/pipeline';

async function runContentPipeline(keyword: string) {
  const pipeline = new ContentPipeline({
    openaiKey: process.env.OPENAI_API_KEY,
    claudeKey: process.env.ANTHROPIC_API_KEY,
    rapidApiKey: process.env.RAPIDAPI_KEY
  });

  // Execute full pipeline
  const result = await pipeline.execute({
    keyword,
    steps: [
      'research',
      'generate-article',
      'translate',
      'create-video',
      'optimize-seo'
    ],
    options: {
      articleFormat: 'how-to',
      videoStyle: 'modern',
      platforms: ['instagram', 'tiktok', 'youtube']
    }
  });

  return {
    articles: result.articles, // English + Vietnamese
    videos: result.videos,     // Platform-optimized videos
    metadata: result.metadata,
    publishReady: true
  };
}
```

## Next.js API Routes

### Research Endpoint

```typescript
// src/app/api/research/route.ts
import { NextResponse } from 'next/server';
import { researchTopic } from '@/lib/research/scraper';

export async function POST(request: Request) {
  try {
    const { keyword, sources } = await request.json();

    const research = await researchTopic({
      keyword,
      sources: sources || ['techcrunch', 'a16z'],
      timeframe: '24h'
    });

    return NextResponse.json({
      success: true,
      data: research
    });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

### Content Generation Endpoint

```typescript
// src/app/api/generate/route.ts
import { NextResponse } from 'next/server';
import { generateContent } from '@/lib/ai/content-generator';

export async function POST(request: Request) {
  try {
    const { topic, format, research } = await request.json();

    const content = await generateContent({
      topic,
      format,
      languages: ['en', 'vi'],
      research,
      aiProvider: 'claude',
      apiKey: process.env.ANTHROPIC_API_KEY
    });

    return NextResponse.json({
      success: true,
      content
    });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

### Video Rendering Endpoint

```typescript
// src/app/api/video/render/route.ts
import { NextResponse } from 'next/server';
import { renderVideo } from '@/lib/video/renderer';

export async function POST(request: Request) {
  try {
    const { script, platform } = await request.json();

    const dimensions = {
      reels: { width: 1080, height: 1920 },
      tiktok: { width: 1080, height: 1920 },
      shorts: { width: 1080, height: 1920 },
      youtube: { width: 1920, height: 1080 }
    };

    const { width, height } = dimensions[platform] || dimensions.reels;

    const video = await renderVideo({
      script,
      composition: 'ContentVideo',
      outputFormat: 'mp4',
      fps: 30,
      width,
      height,
      licenseKey: process.env.REMOTION_LICENSE_KEY
    });

    return NextResponse.json({
      success: true,
      video
    });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

## TypeScript Types

```typescript
// src/types/content.ts
export interface ResearchData {
  keyword: string;
  sources: Array<'techcrunch' | 'a16z' | 'twitter' | 'linkedin'>;
  timeframe: string;
  articles: Article[];
  insights: Insight[];
}

export interface Article {
  title: string;
  content: string;
  language: 'en' | 'vi';
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  wordCount: number;
  readingTime: number;
  keywords: string[];
  seoScore: number;
}

export interface VideoScript {
  scenes: Scene[];
  duration: number;
  platform: 'reels' | 'tiktok' | 'shorts' | 'youtube';
  style: 'infographic' | 'modern' | 'minimal';
}

export interface Scene {
  id: string;
  type: 'title' | 'content' | 'transition' | 'cta';
  text: string;
  duration: number;
  animation?: string;
  background?: string;
}

export interface PipelineResult {
  articles: Article[];
  videos: Video[];
  metadata: {
    createdAt: string;
    keyword: string;
    totalTime: number;
  };
}
```

## Common Patterns

### Pattern 1: Batch Content Generation

Generate multiple articles from a list of keywords:

```typescript
async function batchGenerate(keywords: string[]) {
  const results = await Promise.all(
    keywords.map(async (keyword) => {
      const research = await researchTopic({ keyword });
      const article = await generateContent({
        topic: keyword,
        format: 'toplist',
        research
      });
      return { keyword, article };
    })
  );

  return results;
}
```

### Pattern 2: Scheduled Content Pipeline

Set up automated content generation:

```typescript
import { CronJob } from 'cron';

const dailyContentJob = new CronJob('0 9 * * *', async () => {
  const trendingTopics = await fetchTrendingTopics();
  
  for (const topic of trendingTopics) {
    const content = await runContentPipeline(topic);
    await schedulePost(content, {
      platforms: ['instagram', 'tiktok'],
      publishTime: '12:00'
    });
  }
});

dailyContentJob.start();
```

### Pattern 3: Multi-Platform Optimization

Create platform-specific content variants:

```typescript
async function createMultiPlatformContent(article: Article) {
  const platforms = ['instagram', 'tiktok', 'youtube'];
  
  const content = await Promise.all(
    platforms.map(async (platform) => {
      const optimized = await optimizeForPlatform(article, platform);
      const video = await generateVideo({
        content: optimized,
        platform
      });
      
      return { platform, article: optimized, video };
    })
  );

  return content;
}
```

## Running the Development Server

```bash
# Start Next.js development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Render Remotion video preview
npm run remotion
```

## CLI Commands (if applicable)

If the project includes CLI tools:

```bash
# Generate content from command line
npx content-pipeline generate --keyword "AI trends" --format toplist

# Run research only
npx content-pipeline research --keyword "Web3" --sources techcrunch,a16z

# Render video from existing article
npx content-pipeline video --input article.json --platform reels
```

## Troubleshooting

### API Rate Limits

```typescript
import { RateLimiter } from '@/lib/utils/rate-limiter';

const limiter = new RateLimiter({
  maxRequests: 10,
  windowMs: 60000 // 1 minute
});

async function safeApiCall(fn: () => Promise<any>) {
  await limiter.wait();
  return await fn();
}
```

### Video Rendering Timeout

```typescript
const video = await renderVideo({
  script,
  timeout: 300000, // 5 minutes
  onProgress: (progress) => {
    console.log(`Rendering: ${progress}%`);
  }
});
```

### Memory Management for Large Batches

```typescript
import pLimit from 'p-limit';

const limit = pLimit(3); // Process 3 at a time

const results = await Promise.all(
  keywords.map(keyword => 
    limit(() => runContentPipeline(keyword))
  )
);
```

### Error Handling

```typescript
try {
  const content = await generateContent(config);
} catch (error) {
  if (error.code === 'RATE_LIMIT_EXCEEDED') {
    await sleep(60000); // Wait 1 minute
    return generateContent(config);
  }
  
  if (error.code === 'INVALID_API_KEY') {
    throw new Error('Check your API keys in .env.local');
  }
  
  throw error;
}
```

## Best Practices

1. **Always validate research data** before generating content
2. **Cache API responses** to avoid redundant calls
3. **Use environment variables** for all API keys
4. **Implement retry logic** for external API calls
5. **Monitor AI token usage** to control costs
6. **Test video renders** on small compositions first
7. **Version control your prompts** for reproducibility

## Integration Examples

### With WordPress

```typescript
import { WordPressClient } from '@/lib/integrations/wordpress';

const wp = new WordPressClient({
  url: process.env.WP_URL,
  username: process.env.WP_USERNAME,
  password: process.env.WP_PASSWORD
});

const { article } = await runContentPipeline(keyword);
await wp.createPost({
  title: article.title,
  content: article.content,
  status: 'publish'
});
```

### With Social Media APIs

```typescript
import { SocialMediaManager } from '@/lib/integrations/social';

const social = new SocialMediaManager({
  instagram: process.env.INSTAGRAM_TOKEN,
  tiktok: process.env.TIKTOK_TOKEN
});

const { videos } = await runContentPipeline(keyword);
await social.post({
  platforms: ['instagram', 'tiktok'],
  video: videos[0],
  caption: article.title
});
```
