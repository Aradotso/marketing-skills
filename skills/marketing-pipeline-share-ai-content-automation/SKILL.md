---
name: marketing-pipeline-share-ai-content-automation
description: Automated AI content pipeline for research, scriptwriting, and video generation using Claude/OpenAI and Remotion
triggers:
  - how do I automate content creation with AI research
  - create video content from articles automatically
  - set up AI content pipeline with Claude and OpenAI
  - generate multi-format content from trending news
  - build automated marketing content workflow
  - scrape and synthesize content with AI
  - render videos from blog posts automatically
  - create multilingual content with AI automation
---

# Marketing Pipeline Share - AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

Marketing Pipeline Share is a comprehensive TypeScript-based content automation system that creates a complete pipeline from research to final video output. It automatically scrapes trending news from sources like TechCrunch, a16z, Twitter, and LinkedIn, generates content in multiple formats using Claude/OpenAI, and renders videos using Remotion.

**Key capabilities:**
- Auto-scan and research from live news sources (24h data)
- Multi-format content generation (Toplist, POV, Case Study, How-to)
- Bilingual support (English & Vietnamese)
- Automated video/infographic rendering via Remotion
- Next.js web interface for content management

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
```

### Environment Configuration

Create a `.env.local` file in the root directory:

```env
# AI Provider Keys
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# RapidAPI for news scraping
RAPIDAPI_KEY=your_rapidapi_key

# Optional: Database
DATABASE_URL=your_database_url

# Remotion settings
REMOTION_ENABLE=true
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app router
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── scraper/     # News scraping logic
│   │   └── video/       # Remotion video rendering
│   └── types/           # TypeScript definitions
├── public/              # Static assets
└── remotion/            # Video templates
```

## Core API Usage

### 1. Content Research & Scraping

```typescript
import { scrapeNews } from '@/lib/scraper/newsAggregator';

interface NewsSource {
  platform: 'techcrunch' | 'a16z' | 'twitter' | 'linkedin';
  timeRange: '24h' | '7d' | '30d';
  keywords?: string[];
}

async function gatherResearch(topic: string): Promise<Article[]> {
  const sources: NewsSource[] = [
    { platform: 'techcrunch', timeRange: '24h' },
    { platform: 'twitter', timeRange: '24h', keywords: [topic] }
  ];

  const articles = await scrapeNews({
    sources,
    limit: 20,
    filters: {
      minEngagement: 100,
      excludePromoted: true
    }
  });

  return articles;
}
```

### 2. AI Content Generation with Claude

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY
});

interface ContentFormat {
  type: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
}

async function generateContent(
  research: Article[],
  format: ContentFormat
): Promise<string> {
  const prompt = `
Based on the following research articles, create a ${format.type} in ${format.language} with a ${format.tone} tone:

${research.map(a => `- ${a.title}: ${a.summary}`).join('\n')}

Format requirements:
- Engaging headline
- Data-backed insights
- Actionable takeaways
- SEO optimized
`;

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [
      { role: 'user', content: prompt }
    ]
  });

  return message.content[0].type === 'text' 
    ? message.content[0].text 
    : '';
}
```

### 3. OpenAI Alternative

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY
});

async function generateWithGPT(
  topic: string,
  format: ContentFormat
): Promise<string> {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are an expert content creator specializing in ${format.type} articles.`
      },
      {
        role: 'user',
        content: `Create a comprehensive ${format.type} about ${topic} in ${format.language}.`
      }
    ],
    temperature: 0.7,
    max_tokens: 3000
  });

  return completion.choices[0].message.content || '';
}
```

### 4. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  title: string;
  content: string[];
  imageUrls?: string[];
  duration: number;
  format: 'reels' | 'tiktok' | 'youtube-short';
}

async function renderContentVideo(config: VideoConfig): Promise<string> {
  const compositionId = 'ContentVideo';
  const bundleLocation = await bundle(
    path.join(process.cwd(), 'remotion/index.ts')
  );

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: {
      title: config.title,
      slides: config.content,
      images: config.imageUrls,
      format: config.format
    }
  });

  const outputPath = path.join(
    process.cwd(),
    'public/videos',
    `${Date.now()}.mp4`
  );

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: composition.defaultProps
  });

  return outputPath;
}
```

## Complete Pipeline Example

```typescript
import { scrapeNews } from '@/lib/scraper/newsAggregator';
import { generateContent } from '@/lib/ai/claude';
import { renderContentVideo } from '@/lib/video/remotion';

interface PipelineConfig {
  topic: string;
  contentFormat: ContentFormat;
  videoEnabled: boolean;
}

async function runContentPipeline(config: PipelineConfig) {
  try {
    // Step 1: Research
    console.log('🔍 Gathering research...');
    const articles = await scrapeNews({
      sources: [
        { platform: 'techcrunch', timeRange: '24h' },
        { platform: 'twitter', timeRange: '24h', keywords: [config.topic] }
      ],
      limit: 15
    });

    // Step 2: Generate Content
    console.log('✍️ Generating content...');
    const content = await generateContent(articles, config.contentFormat);

    // Step 3: Extract key points for video
    const keyPoints = extractKeyPoints(content);

    // Step 4: Render Video (optional)
    let videoPath = null;
    if (config.videoEnabled) {
      console.log('🎬 Rendering video...');
      videoPath = await renderContentVideo({
        title: config.topic,
        content: keyPoints,
        duration: 30,
        format: 'reels'
      });
    }

    return {
      success: true,
      content,
      videoPath,
      metadata: {
        sourcesCount: articles.length,
        wordCount: content.split(' ').length,
        generatedAt: new Date().toISOString()
      }
    };

  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}

// Usage
const result = await runContentPipeline({
  topic: 'AI in Marketing 2024',
  contentFormat: {
    type: 'toplist',
    language: 'en',
    tone: 'expert'
  },
  videoEnabled: true
});
```

## Next.js API Routes

```typescript
// app/api/content/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline';

export async function POST(req: NextRequest) {
  try {
    const body = await req.json();
    const { topic, format, language, generateVideo } = body;

    const result = await runContentPipeline({
      topic,
      contentFormat: {
        type: format,
        language,
        tone: 'expert'
      },
      videoEnabled: generateVideo
    });

    return NextResponse.json(result);
  } catch (error) {
    return NextResponse.json(
      { error: 'Content generation failed', details: error.message },
      { status: 500 }
    );
  }
}
```

## Common Patterns

### Batch Content Generation

```typescript
async function generateBatchContent(topics: string[]) {
  const results = await Promise.all(
    topics.map(topic => 
      runContentPipeline({
        topic,
        contentFormat: { type: 'how-to', language: 'en', tone: 'friendly' },
        videoEnabled: false
      })
    )
  );

  return results;
}
```

### Scheduled Content Creation

```typescript
import cron from 'node-cron';

// Run daily at 9 AM
cron.schedule('0 9 * * *', async () => {
  console.log('Running scheduled content generation...');
  
  const trendingTopics = await fetchTrendingTopics();
  
  for (const topic of trendingTopics.slice(0, 3)) {
    await runContentPipeline({
      topic: topic.name,
      contentFormat: { type: 'pov', language: 'vi', tone: 'expert' },
      videoEnabled: true
    });
  }
});
```

### Multi-language Content

```typescript
async function createMultilingualContent(topic: string) {
  const languages: Array<'en' | 'vi'> = ['en', 'vi'];
  
  const contentPieces = await Promise.all(
    languages.map(lang =>
      generateContent(articles, {
        type: 'toplist',
        language: lang,
        tone: 'friendly'
      })
    )
  );

  return {
    en: contentPieces[0],
    vi: contentPieces[1]
  };
}
```

## Running the Application

```bash
# Development mode
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Render videos locally
npm run remotion:preview
```

## Troubleshooting

### API Rate Limits

```typescript
import pRetry from 'p-retry';

async function generateWithRetry(prompt: string) {
  return pRetry(
    async () => {
      return await anthropic.messages.create({
        model: 'claude-3-5-sonnet-20241022',
        max_tokens: 4096,
        messages: [{ role: 'user', content: prompt }]
      });
    },
    {
      retries: 3,
      onFailedAttempt: error => {
        console.log(`Attempt ${error.attemptNumber} failed. Retrying...`);
      }
    }
  );
}
```

### Video Rendering Memory Issues

Adjust Remotion settings:

```typescript
await renderMedia({
  composition,
  serveUrl: bundleLocation,
  codec: 'h264',
  outputLocation: outputPath,
  concurrency: 2, // Reduce from default
  chromiumOptions: {
    gl: 'swiftshader',
    headless: true
  }
});
```

### News Scraping Failures

```typescript
async function scrapeWithFallback(sources: NewsSource[]) {
  for (const source of sources) {
    try {
      const articles = await scrapeNews({ sources: [source], limit: 10 });
      if (articles.length > 0) return articles;
    } catch (error) {
      console.warn(`Failed to scrape ${source.platform}, trying next...`);
      continue;
    }
  }
  throw new Error('All news sources failed');
}
```

## Best Practices

1. **Cache Research Data**: Store scraped articles for 24h to avoid redundant API calls
2. **Queue Video Rendering**: Use a job queue (Bull, BullMQ) for video processing
3. **Content Validation**: Always validate AI-generated content before publishing
4. **Error Logging**: Implement comprehensive logging for pipeline failures
5. **Cost Monitoring**: Track AI API usage to manage costs effectively
