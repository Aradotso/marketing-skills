---
name: marketing-pipeline-automation
description: AI-powered content automation pipeline from research to video generation with Claude/OpenAI
triggers:
  - automate content creation pipeline
  - generate content from research to video
  - set up AI marketing automation
  - create automated content workflow
  - build content pipeline with Claude
  - generate videos from articles automatically
  - automate social media content creation
  - research and write content with AI
---

# Marketing Pipeline Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the Ultimate AI Content Pipeline, a comprehensive TypeScript-based system that automates content creation from research to video generation. The pipeline integrates Claude 3, OpenAI, and Remotion to crawl news sources, generate multi-format content, and render videos automatically.

## What This Project Does

The Marketing Pipeline automates the entire content creation workflow:

1. **Auto-Research**: Crawls real-time data from TechCrunch, a16z, Twitter/X, LinkedIn (last 24h)
2. **Content Generation**: Creates articles in multiple formats (Toplist, POV, Case Study, How-to) using Claude/OpenAI
3. **Multi-Language Support**: Generates content in English and Vietnamese simultaneously
4. **Video Rendering**: Automatically converts content to infographics and short-form videos using Remotion
5. **Platform Optimization**: Exports videos in aspect ratios for Reels, TikTok, Shorts

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
# AI Provider Keys
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# RapidAPI for web scraping/research
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Remotion (for video rendering)
REMOTION_AWS_ACCESS_KEY_ID=your_aws_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/             # Core utilities
│   │   ├── ai/          # AI provider integrations
│   │   ├── research/    # Web scraping & data collection
│   │   ├── content/     # Content generation logic
│   │   └── video/       # Remotion video generation
│   ├── remotion/        # Remotion compositions
│   └── types/           # TypeScript definitions
├── public/              # Static assets
└── package.json
```

## Core API Usage

### 1. Research Module

Automatically scrape and analyze content from multiple sources:

```typescript
import { researchTopic } from '@/lib/research/scraper';

async function gatherResearch(keyword: string) {
  const research = await researchTopic({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeRange: '24h',
    language: 'en'
  });

  return {
    articles: research.articles,
    insights: research.insights,
    trends: research.trends,
    stats: research.statistics
  };
}
```

### 2. Content Generation with Claude/OpenAI

Generate content in multiple formats:

```typescript
import { generateContent } from '@/lib/ai/content-generator';

async function createArticle(topic: string, format: string) {
  const content = await generateContent({
    topic,
    format: format as 'toplist' | 'pov' | 'case-study' | 'how-to',
    provider: 'claude', // or 'openai'
    language: 'en', // or 'vi' for Vietnamese
    tone: 'expert', // 'friendly', 'humorous'
    research: await gatherResearch(topic)
  });

  return {
    title: content.title,
    body: content.body,
    metadata: content.metadata,
    images: content.suggestedImages
  };
}
```

### 3. Multi-Language Content Generation

Generate content in both English and Vietnamese:

```typescript
import { generateMultiLanguageContent } from '@/lib/content/multi-lang';

async function createBilingualContent(topic: string) {
  const content = await generateMultiLanguageContent({
    topic,
    languages: ['en', 'vi'],
    format: 'toplist'
  });

  return {
    en: content.en,
    vi: content.vi,
    sharedMetadata: content.metadata
  };
}
```

### 4. Video Generation with Remotion

Convert content to video automatically:

```typescript
import { renderVideo } from '@/lib/video/remotion-renderer';
import { bundle } from '@remotion/bundler';

async function generateVideoFromContent(content: any) {
  // Bundle the Remotion composition
  const bundleLocation = await bundle({
    entryPoint: './src/remotion/index.ts',
    webpackOverride: (config) => config,
  });

  // Render video
  const video = await renderVideo({
    composition: 'ContentVideo',
    props: {
      title: content.title,
      points: content.body.split('\n'),
      style: 'infographic'
    },
    codec: 'h264',
    outputLocation: `public/videos/${content.slug}.mp4`,
    aspectRatio: '9:16' // For Reels/TikTok/Shorts
  });

  return video.outputPath;
}
```

## API Routes (Next.js)

### Research Endpoint

```typescript
// src/app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { researchTopic } from '@/lib/research/scraper';

export async function POST(request: NextRequest) {
  const { keyword, sources } = await request.json();

  try {
    const research = await researchTopic({
      keyword,
      sources: sources || ['techcrunch', 'a16z'],
      timeRange: '24h'
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

### Content Generation Endpoint

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { generateContent } from '@/lib/ai/content-generator';

export async function POST(request: NextRequest) {
  const { topic, format, language, provider } = await request.json();

  try {
    const content = await generateContent({
      topic,
      format,
      language: language || 'en',
      provider: provider || 'claude'
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

### Video Rendering Endpoint

```typescript
// src/app/api/render-video/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { renderVideo } from '@/lib/video/remotion-renderer';

export async function POST(request: NextRequest) {
  const { contentId, aspectRatio } = await request.json();

  try {
    const videoPath = await renderVideo({
      contentId,
      aspectRatio: aspectRatio || '9:16'
    });

    return NextResponse.json({ 
      success: true, 
      videoUrl: videoPath 
    });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

## Common Workflow Pattern

Complete end-to-end content pipeline:

```typescript
import { researchTopic } from '@/lib/research/scraper';
import { generateContent } from '@/lib/ai/content-generator';
import { renderVideo } from '@/lib/video/remotion-renderer';

async function completeContentPipeline(keyword: string) {
  // Step 1: Research
  console.log('🔍 Researching topic...');
  const research = await researchTopic({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter'],
    timeRange: '24h'
  });

  // Step 2: Generate Content (Bilingual)
  console.log('✍️ Generating content...');
  const contentEN = await generateContent({
    topic: keyword,
    format: 'toplist',
    provider: 'claude',
    language: 'en',
    research
  });

  const contentVI = await generateContent({
    topic: keyword,
    format: 'toplist',
    provider: 'claude',
    language: 'vi',
    research
  });

  // Step 3: Render Video
  console.log('🎬 Rendering video...');
  const video = await renderVideo({
    composition: 'ToplistVideo',
    props: {
      title: contentEN.title,
      items: contentEN.body.split('\n').filter(Boolean)
    },
    aspectRatio: '9:16'
  });

  return {
    research,
    content: {
      en: contentEN,
      vi: contentVI
    },
    video: video.outputPath
  };
}

// Usage
completeContentPipeline('AI automation tools 2026')
  .then(result => console.log('✅ Pipeline complete:', result))
  .catch(error => console.error('❌ Error:', error));
```

## Configuration Options

### Content Format Types

```typescript
type ContentFormat = 
  | 'toplist'      // Top 10 lists, rankings
  | 'pov'          // Opinion/perspective pieces
  | 'case-study'   // In-depth analysis
  | 'how-to';      // Tutorial/guide format

type ToneStyle = 
  | 'expert'       // Professional, authoritative
  | 'friendly'     // Conversational, approachable
  | 'humorous';    // Engaging, entertaining
```

### Video Aspect Ratios

```typescript
type AspectRatio = 
  | '16:9'   // YouTube, landscape
  | '9:16'   // Reels, TikTok, Shorts
  | '1:1'    // Instagram square
  | '4:5';   // Instagram portrait
```

## Development Commands

```bash
# Start Next.js development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Run Remotion preview (for video development)
npm run remotion:preview

# Render a specific video composition
npm run remotion:render
```

## Troubleshooting

### API Rate Limiting

If you encounter rate limits with AI providers:

```typescript
import { retry } from '@/lib/utils/retry';

const content = await retry(
  () => generateContent({ topic, format }),
  {
    maxAttempts: 3,
    delay: 2000,
    backoff: 'exponential'
  }
);
```

### Video Rendering Issues

For Remotion rendering errors:

```bash
# Ensure ffmpeg is installed
brew install ffmpeg  # macOS
sudo apt install ffmpeg  # Linux

# Check Remotion configuration
npx remotion versions
```

### Memory Issues with Large Pipelines

Handle memory-intensive operations:

```typescript
async function processInBatches(keywords: string[]) {
  const BATCH_SIZE = 5;
  
  for (let i = 0; i < keywords.length; i += BATCH_SIZE) {
    const batch = keywords.slice(i, i + BATCH_SIZE);
    await Promise.all(
      batch.map(keyword => completeContentPipeline(keyword))
    );
    
    // Pause between batches
    await new Promise(resolve => setTimeout(resolve, 5000));
  }
}
```

### Research Scraping Failures

Handle scraping errors gracefully:

```typescript
import { researchTopic } from '@/lib/research/scraper';

async function safeResearch(keyword: string) {
  try {
    return await researchTopic({
      keyword,
      sources: ['techcrunch', 'a16z'],
      timeout: 10000
    });
  } catch (error) {
    console.warn('Research failed, using fallback:', error);
    return {
      articles: [],
      insights: ['Fallback insight based on keyword'],
      trends: [],
      statistics: {}
    };
  }
}
```

## Integration Examples

### Scheduling Content Pipeline

```typescript
import cron from 'node-cron';

// Run pipeline daily at 9 AM
cron.schedule('0 9 * * *', async () => {
  const topics = ['AI trends', 'Marketing automation', 'Content creation'];
  
  for (const topic of topics) {
    await completeContentPipeline(topic);
  }
});
```

### Webhook Integration

```typescript
// src/app/api/webhook/route.ts
export async function POST(request: NextRequest) {
  const { keyword, callback_url } = await request.json();
  
  // Process in background
  completeContentPipeline(keyword).then(async (result) => {
    // Notify completion
    await fetch(callback_url, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(result)
    });
  });
  
  return NextResponse.json({ status: 'processing' });
}
```

This skill enables AI agents to effectively utilize the marketing pipeline for automated content creation, from research through video generation.
