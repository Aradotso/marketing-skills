---
name: marketing-pipeline-share-ai-content
description: Automated AI content pipeline for research, scriptwriting, video generation, and multi-platform publishing
triggers:
  - "set up automated content pipeline with AI"
  - "generate video content from articles automatically"
  - "crawl news sources and create content with Claude"
  - "automate research to video workflow"
  - "create multilingual content with AI pipeline"
  - "build AI-powered content automation system"
  - "generate social media videos from research"
  - "set up Remotion video rendering pipeline"
---

# Marketing Pipeline Share AI Content

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

An end-to-end AI content automation pipeline that researches trending topics, generates multilingual articles, and renders videos automatically. Built with Next.js, TypeScript, Claude/OpenAI, and Remotion.

## What It Does

This system automates the entire content creation workflow:

1. **Auto-Research**: Crawls news sources (TechCrunch, a16z, Twitter, LinkedIn) for trending topics
2. **AI Content Generation**: Creates articles in multiple formats (toplist, POV, case study, how-to) using Claude/OpenAI
3. **Multilingual Output**: Generates content in English and Vietnamese simultaneously
4. **Video Rendering**: Automatically creates infographics and short videos using Remotion
5. **Multi-Platform Export**: Optimized for Reels, TikTok, Shorts

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

## Environment Configuration

Create a `.env.local` file in the root directory:

```bash
# AI Provider APIs
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research & Crawling
RAPIDAPI_KEY=your_rapidapi_key

# Optional: Database (if using)
DATABASE_URL=your_database_url

# Remotion License (if applicable)
REMOTION_LICENSE_KEY=your_remotion_license
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/             
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── crawler/     # News crawling logic
│   │   ├── content/     # Content generation
│   │   └── video/       # Remotion video rendering
│   ├── types/           # TypeScript types
│   └── utils/           # Utility functions
├── public/              # Static assets
└── remotion/            # Video templates
```

## Core API Usage

### 1. Research & Crawling

```typescript
import { crawlNewsSources } from '@/lib/crawler/news-crawler';
import { extractInsights } from '@/lib/crawler/insights';

// Crawl news sources for a topic
async function researchTopic(keyword: string) {
  const sources = await crawlNewsSources({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeRange: '24h'
  });

  // Extract key insights
  const insights = await extractInsights(sources);
  
  return {
    rawData: sources,
    insights,
    trends: insights.filter(i => i.trendScore > 0.7)
  };
}
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
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [
      {
        role: 'user',
        content: `Create a ${format} article about "${topic}" in ${language}. 
        Include data-backed insights, trending perspectives, and actionable takeaways.
        Format: Use markdown with clear sections.`
      }
    ],
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
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateContentWithOpenAI(
  topic: string,
  tone: 'expert' | 'friendly' | 'humorous'
) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are a content creator with a ${tone} tone. Create engaging, data-driven content.`
      },
      {
        role: 'user',
        content: `Write an article about: ${topic}`
      }
    ],
    temperature: 0.7,
  });

  return completion.choices[0].message.content;
}
```

### 4. Video Rendering with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

async function renderContentVideo(
  contentData: {
    title: string;
    points: string[];
    stats: { label: string; value: string }[];
  },
  format: 'reels' | 'tiktok' | 'shorts'
) {
  const bundled = await bundle(
    path.join(process.cwd(), 'remotion/index.ts')
  );

  const compositionId = `${format}-template`;
  const composition = await selectComposition({
    serveUrl: bundled,
    id: compositionId,
    inputProps: contentData,
  });

  const outputPath = path.join(
    process.cwd(), 
    'public/videos', 
    `${Date.now()}.mp4`
  );

  await renderMedia({
    composition,
    serveUrl: bundled,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: contentData,
  });

  return outputPath;
}
```

## Complete Workflow Example

```typescript
import { researchTopic } from '@/lib/crawler/news-crawler';
import { generateContentWithClaude } from '@/lib/ai/claude';
import { renderContentVideo } from '@/lib/video/renderer';

async function fullContentPipeline(keyword: string) {
  // Step 1: Research
  console.log('🔍 Researching topic...');
  const research = await researchTopic(keyword);

  // Step 2: Generate content (both languages)
  console.log('✍️ Generating content...');
  const [contentEN, contentVI] = await Promise.all([
    generateContentWithClaude(
      keyword, 
      'toplist', 
      'en'
    ),
    generateContentWithClaude(
      keyword, 
      'toplist', 
      'vi'
    )
  ]);

  // Step 3: Extract video data from content
  const videoData = {
    title: keyword,
    points: research.insights.slice(0, 5).map(i => i.text),
    stats: research.insights
      .filter(i => i.hasStats)
      .map(i => ({ label: i.label, value: i.value }))
  };

  // Step 4: Render videos for multiple platforms
  console.log('🎬 Rendering videos...');
  const videos = await Promise.all([
    renderContentVideo(videoData, 'reels'),
    renderContentVideo(videoData, 'tiktok'),
    renderContentVideo(videoData, 'shorts')
  ]);

  return {
    research,
    content: { en: contentEN, vi: contentVI },
    videos,
    createdAt: new Date().toISOString()
  };
}

// Usage
const result = await fullContentPipeline('AI trends 2024');
console.log('Pipeline completed:', result);
```

## API Routes (Next.js)

### POST /api/pipeline/research

```typescript
// app/api/pipeline/research/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { researchTopic } from '@/lib/crawler/news-crawler';

export async function POST(request: NextRequest) {
  const { keyword } = await request.json();

  if (!keyword) {
    return NextResponse.json(
      { error: 'Keyword is required' },
      { status: 400 }
    );
  }

  try {
    const research = await researchTopic(keyword);
    return NextResponse.json({ success: true, data: research });
  } catch (error) {
    return NextResponse.json(
      { error: 'Research failed', details: error.message },
      { status: 500 }
    );
  }
}
```

### POST /api/pipeline/generate

```typescript
// app/api/pipeline/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { generateContentWithClaude } from '@/lib/ai/claude';

export async function POST(request: NextRequest) {
  const { topic, format, language } = await request.json();

  try {
    const content = await generateContentWithClaude(
      topic,
      format,
      language
    );
    
    return NextResponse.json({ 
      success: true, 
      content,
      metadata: {
        wordCount: content.split(/\s+/).length,
        generatedAt: new Date().toISOString()
      }
    });
  } catch (error) {
    return NextResponse.json(
      { error: 'Content generation failed', details: error.message },
      { status: 500 }
    );
  }
}
```

### POST /api/pipeline/render

```typescript
// app/api/pipeline/render/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { renderContentVideo } from '@/lib/video/renderer';

export async function POST(request: NextRequest) {
  const { contentData, format } = await request.json();

  try {
    const videoPath = await renderContentVideo(contentData, format);
    
    return NextResponse.json({ 
      success: true, 
      videoUrl: videoPath.replace(process.cwd() + '/public', '')
    });
  } catch (error) {
    return NextResponse.json(
      { error: 'Video rendering failed', details: error.message },
      { status: 500 }
    );
  }
}
```

## Common Patterns

### Batch Content Generation

```typescript
async function batchGenerateContent(
  keywords: string[],
  format: 'toplist' | 'pov' | 'case-study' | 'how-to'
) {
  const results = [];

  for (const keyword of keywords) {
    const pipeline = await fullContentPipeline(keyword);
    results.push(pipeline);
    
    // Rate limiting
    await new Promise(resolve => setTimeout(resolve, 2000));
  }

  return results;
}
```

### Custom Tone & Voice

```typescript
interface ContentConfig {
  tone: 'expert' | 'friendly' | 'humorous';
  targetAudience: string;
  includeStats: boolean;
  length: 'short' | 'medium' | 'long';
}

async function generateCustomContent(
  topic: string,
  config: ContentConfig
) {
  const prompt = `
    Topic: ${topic}
    Tone: ${config.tone}
    Target Audience: ${config.targetAudience}
    Include Statistics: ${config.includeStats}
    Length: ${config.length}
    
    Create engaging content following these parameters.
  `;

  return await generateContentWithClaude(topic, 'pov', 'en');
}
```

### Scheduled Pipeline Execution

```typescript
import cron from 'node-cron';

// Run pipeline daily at 9 AM
cron.schedule('0 9 * * *', async () => {
  const trendingTopics = await fetchTrendingTopics();
  
  for (const topic of trendingTopics) {
    await fullContentPipeline(topic);
  }
});
```

## Development Server

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Start production server
npm start
```

Access at `http://localhost:3000`

## Troubleshooting

### API Rate Limits

```typescript
// Implement exponential backoff
async function retryWithBackoff<T>(
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
  throw new Error('Max retries reached');
}
```

### Video Rendering Memory Issues

```typescript
// Process videos sequentially instead of parallel
async function renderVideosSequentially(
  contentData: any,
  formats: string[]
) {
  const results = [];
  
  for (const format of formats) {
    const video = await renderContentVideo(contentData, format);
    results.push(video);
    
    // Clear memory between renders
    if (global.gc) global.gc();
  }
  
  return results;
}
```

### Missing Environment Variables

```typescript
function validateEnv() {
  const required = [
    'ANTHROPIC_API_KEY',
    'OPENAI_API_KEY',
    'RAPIDAPI_KEY'
  ];

  const missing = required.filter(key => !process.env[key]);

  if (missing.length > 0) {
    throw new Error(
      `Missing required environment variables: ${missing.join(', ')}`
    );
  }
}

// Call on app initialization
validateEnv();
```

## Best Practices

1. **Cache Research Results**: Store crawled data to avoid redundant API calls
2. **Queue Video Rendering**: Use a job queue (Bull, BullMQ) for video processing
3. **Monitor API Usage**: Track token consumption for Claude/OpenAI
4. **Optimize Remotion Templates**: Minimize bundle size for faster renders
5. **Implement Error Handling**: Graceful degradation when AI services are unavailable
6. **Use TypeScript Strictly**: Leverage type safety for configuration objects
