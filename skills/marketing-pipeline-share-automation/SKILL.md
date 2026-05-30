---
name: marketing-pipeline-share-automation
description: AI-powered content automation pipeline from research to video generation using Claude, OpenAI, and Remotion
triggers:
  - automate content creation with AI research
  - generate marketing content from keywords
  - create videos from blog posts automatically
  - set up AI content pipeline with Claude
  - crawl news and generate content
  - build automated marketing workflow
  - convert articles to video with Remotion
  - research and write content with AI
---

# Marketing Pipeline Share Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with **marketing-pipeline-share**, an end-to-end content automation system that handles research, script generation, and video production. The pipeline crawls news sources, generates multi-format content using Claude/OpenAI, and renders videos via Remotion.

## What It Does

The Ultimate AI Content Pipeline automates:
- **Research Phase**: Crawls TechCrunch, a16z, Twitter/X, LinkedIn for trending topics
- **Content Generation**: Creates articles in multiple formats (listicles, POV, case studies, how-tos) using Claude 3 or OpenAI
- **Multi-language Support**: Generates Vietnamese and English versions simultaneously
- **Video Production**: Converts text to infographics and short-form videos using Remotion
- **Platform Optimization**: Outputs video formats for Reels, TikTok, Shorts

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

Create a `.env.local` file in the project root:

```bash
# AI Provider Keys
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research API Keys
RAPIDAPI_KEY=your_rapidapi_key

# Remotion Configuration
REMOTION_AWS_ACCESS_KEY_ID=your_aws_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret

# Application Settings
NEXT_PUBLIC_APP_URL=http://localhost:3000
DATABASE_URL=your_database_connection_string
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
│   │   ├── video/       # Remotion video generation
│   │   └── utils/       # Helpers and utilities
│   └── types/           # TypeScript type definitions
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Core API Usage

### 1. Research & Crawling

```typescript
import { crawlNewsSources } from '@/lib/crawler';

async function researchTopic(keyword: string) {
  const sources = await crawlNewsSources({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeframe: '24h'
  });
  
  return {
    articles: sources.articles,
    insights: sources.extractedInsights,
    dataPoints: sources.statistics
  };
}
```

### 2. Content Generation with Claude

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

async function generateContent(topic: string, format: string) {
  const message = await anthropic.messages.create({
    model: 'claude-3-opus-20240229',
    max_tokens: 4096,
    messages: [{
      role: 'user',
      content: `Write a ${format} article about ${topic}. 
      Include data-backed insights and maintain a professional yet engaging tone.
      Output in both English and Vietnamese.`
    }]
  });
  
  return message.content[0].text;
}
```

### 3. OpenAI Alternative

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateWithGPT(prompt: string, language: 'en' | 'vi') {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are a content marketing expert. Write in ${language === 'vi' ? 'Vietnamese' : 'English'}.`
      },
      {
        role: 'user',
        content: prompt
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
import { VideoComposition } from '@/remotion/Composition';

async function generateVideo(content: {
  title: string;
  points: string[];
  images: string[];
}) {
  const bundleLocation = await bundle({
    entryPoint: './remotion/index.ts',
    webpackOverride: (config) => config,
  });
  
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: content,
  });
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: `out/${content.title}.mp4`,
    inputProps: content,
  });
}
```

## Common Workflows

### Full Content Pipeline

```typescript
import { crawlNewsSources } from '@/lib/crawler';
import { generateContent } from '@/lib/ai/claude';
import { renderVideo } from '@/lib/video/remotion';

async function runFullPipeline(keyword: string) {
  // Step 1: Research
  console.log('🔍 Researching topic...');
  const research = await crawlNewsSources({
    keyword,
    sources: ['techcrunch', 'a16z'],
    timeframe: '24h'
  });
  
  // Step 2: Generate content
  console.log('✍️ Generating content...');
  const article = await generateContent({
    topic: keyword,
    format: 'toplist',
    insights: research.insights,
    language: 'both' // en + vi
  });
  
  // Step 3: Create video
  console.log('🎬 Rendering video...');
  const video = await renderVideo({
    script: article.content,
    format: 'reels', // 9:16 aspect ratio
    duration: 60
  });
  
  return {
    article,
    video,
    research
  };
}
```

### Batch Processing Multiple Keywords

```typescript
async function batchGenerateContent(keywords: string[]) {
  const results = await Promise.all(
    keywords.map(async (keyword) => {
      try {
        return await runFullPipeline(keyword);
      } catch (error) {
        console.error(`Failed for ${keyword}:`, error);
        return null;
      }
    })
  );
  
  return results.filter(Boolean);
}
```

## Content Format Templates

```typescript
type ContentFormat = 'toplist' | 'pov' | 'casestudy' | 'howto';

const formatPrompts: Record<ContentFormat, string> = {
  toplist: 'Create a numbered list article with data-backed points',
  pov: 'Write from a unique perspective with personal insights',
  casestudy: 'Analyze a real-world example with metrics and outcomes',
  howto: 'Provide step-by-step instructions with actionable tips'
};

function getPromptForFormat(format: ContentFormat, topic: string) {
  return `${formatPrompts[format]} about ${topic}.`;
}
```

## API Routes (Next.js)

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runFullPipeline } from '@/lib/pipeline';

export async function POST(request: NextRequest) {
  try {
    const { keyword, format, language } = await request.json();
    
    const result = await runFullPipeline(keyword);
    
    return NextResponse.json({
      success: true,
      data: result
    });
  } catch (error) {
    return NextResponse.json({
      success: false,
      error: error.message
    }, { status: 500 });
  }
}
```

## Development Commands

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Render Remotion video preview
npm run remotion:preview

# Render video to file
npm run remotion:render
```

## Configuration Options

### AI Model Selection

```typescript
// lib/config.ts
export const AI_CONFIG = {
  provider: process.env.AI_PROVIDER || 'claude', // 'claude' | 'openai'
  model: {
    claude: 'claude-3-opus-20240229',
    openai: 'gpt-4-turbo-preview'
  },
  temperature: 0.7,
  maxTokens: 4096
};
```

### Crawler Settings

```typescript
export const CRAWLER_CONFIG = {
  sources: {
    techcrunch: 'https://techcrunch.com/feed/',
    a16z: 'https://a16z.com/feed/',
  },
  rateLimit: 1000, // ms between requests
  timeout: 30000,
  maxArticles: 50
};
```

### Video Settings

```typescript
export const VIDEO_CONFIG = {
  fps: 30,
  formats: {
    reels: { width: 1080, height: 1920 },
    youtube: { width: 1920, height: 1080 },
    tiktok: { width: 1080, height: 1920 }
  },
  codec: 'h264',
  quality: 'high'
};
```

## Troubleshooting

### API Rate Limits

```typescript
// lib/utils/rateLimit.ts
import { Ratelimit } from '@upstash/ratelimit';
import { Redis } from '@upstash/redis';

const ratelimit = new Ratelimit({
  redis: Redis.fromEnv(),
  limiter: Ratelimit.slidingWindow(10, '1 m'),
});

export async function checkRateLimit(identifier: string) {
  const { success, limit, reset, remaining } = await ratelimit.limit(identifier);
  
  if (!success) {
    throw new Error(`Rate limit exceeded. Reset at ${new Date(reset)}`);
  }
  
  return { remaining, reset };
}
```

### Error Handling

```typescript
class PipelineError extends Error {
  constructor(
    message: string,
    public stage: 'research' | 'generation' | 'video',
    public originalError?: Error
  ) {
    super(message);
    this.name = 'PipelineError';
  }
}

async function safeRunPipeline(keyword: string) {
  try {
    return await runFullPipeline(keyword);
  } catch (error) {
    if (error instanceof PipelineError) {
      console.error(`Pipeline failed at ${error.stage}:`, error.message);
      // Implement retry logic or fallback
    }
    throw error;
  }
}
```

### Memory Management for Large Videos

```typescript
import { renderFrames } from '@remotion/renderer';

async function renderLargeVideo(config) {
  // Use frame-by-frame rendering for memory efficiency
  const frames = await renderFrames({
    ...config,
    onFrameUpdate: (frame) => {
      console.log(`Rendered frame ${frame}/${config.durationInFrames}`);
    },
    onDownload: (src) => {
      console.log(`Downloaded asset: ${src}`);
    }
  });
  
  return frames;
}
```

## Performance Optimization

```typescript
// Parallel processing with controlled concurrency
import pLimit from 'p-limit';

const limit = pLimit(3); // Max 3 concurrent operations

async function processMultipleTopics(topics: string[]) {
  const tasks = topics.map(topic => 
    limit(() => runFullPipeline(topic))
  );
  
  return await Promise.allSettled(tasks);
}
```

## Integration Examples

### Webhook for Scheduled Content

```typescript
// app/api/webhook/schedule/route.ts
export async function POST(req: NextRequest) {
  const { keywords, schedule } = await req.json();
  
  // Queue jobs for later execution
  for (const keyword of keywords) {
    await queue.add('generate-content', {
      keyword,
      scheduledFor: schedule
    });
  }
  
  return NextResponse.json({ queued: keywords.length });
}
```

This skill provides comprehensive coverage of the marketing-pipeline-share project for AI coding agents to effectively assist developers in building automated content marketing workflows.
