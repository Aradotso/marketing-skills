```markdown
---
name: marketing-pipeline-share-automation
description: Automated AI content pipeline for research, scripting, and video generation using Claude, OpenAI, and Remotion
triggers:
  - automate content creation with AI research
  - generate videos from blog posts automatically
  - crawl news and create content with AI
  - build automated marketing content pipeline
  - create multilingual content with Claude and OpenAI
  - set up AI-powered content workflow
  - generate social media videos from articles
  - automate research and scriptwriting pipeline
---

# Marketing Pipeline Share - AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

Marketing Pipeline Share is a complete AI-powered content automation system that transforms keywords into finished content and videos. It automatically:

1. **Researches** - Crawls news from TechCrunch, a16z, Twitter, LinkedIn
2. **Generates** - Creates multi-format content (toplist, POV, case study, how-to) in multiple languages
3. **Renders** - Produces videos and infographics using Remotion
4. **Publishes** - Prepares content for automatic social media posting

Built with Next.js, TypeScript, Claude 3, OpenAI, and Remotion.

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

### Required Environment Variables

```bash
# AI APIs
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research/Crawling
RAPIDAPI_KEY=your_rapidapi_key

# Optional: Social media publishing
FACEBOOK_ACCESS_TOKEN=your_token
TWITTER_API_KEY=your_key
```

### Run Development Server

```bash
npm run dev
# Server starts at http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app router pages
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── crawler/     # News crawling logic
│   │   ├── video/       # Remotion video generation
│   │   └── utils/       # Utilities
│   └── types/           # TypeScript definitions
├── public/              # Static assets
└── remotion/            # Video templates
```

## Core Features & Usage

### 1. AI-Powered Research & Crawling

Automatically fetch and analyze recent news:

```typescript
import { crawlNews } from '@/lib/crawler/news-crawler';
import { analyzeContent } from '@/lib/ai/analyzer';

async function researchTopic(keyword: string) {
  // Crawl news from multiple sources
  const articles = await crawlNews({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeframe: '24h'
  });

  // AI analysis with Claude
  const insights = await analyzeContent(articles, {
    model: 'claude-3-opus',
    focusAreas: ['trends', 'data-points', 'expert-opinions']
  });

  return insights;
}
```

### 2. Multi-Format Content Generation

Generate content in various formats:

```typescript
import { generateContent } from '@/lib/ai/content-generator';

async function createContent(topic: string, format: string) {
  const content = await generateContent({
    topic,
    format, // 'toplist' | 'pov' | 'case-study' | 'how-to'
    languages: ['en', 'vi'],
    tone: 'expert', // 'expert' | 'friendly' | 'humorous'
    aiProvider: 'claude', // 'claude' | 'openai'
    includeData: true
  });

  return content;
}

// Example: Generate a toplist
const toplist = await createContent(
  'AI tools for marketers',
  'toplist'
);

console.log(toplist.en); // English version
console.log(toplist.vi); // Vietnamese version
```

### 3. Content Generation with Research

Combine research and generation:

```typescript
import { createContentPipeline } from '@/lib/pipeline';

async function fullPipeline(keyword: string) {
  const result = await createContentPipeline({
    keyword,
    steps: {
      research: {
        enabled: true,
        sources: ['techcrunch', 'a16z', 'twitter'],
        depth: 'deep'
      },
      generate: {
        formats: ['toplist', 'how-to'],
        languages: ['en', 'vi'],
        aiModel: 'claude-3-opus'
      },
      video: {
        enabled: true,
        platforms: ['reels', 'tiktok', 'shorts']
      }
    }
  });

  return result;
}
```

### 4. Video Generation with Remotion

Transform content into videos:

```typescript
import { renderVideo } from '@/lib/video/renderer';
import { bundle } from '@remotion/bundler';
import { renderMedia } from '@remotion/renderer';

async function generateVideo(content: any) {
  // Prepare video data
  const videoData = {
    title: content.title,
    points: content.keyPoints,
    images: content.images,
    duration: 60 // seconds
  };

  // Render with Remotion
  const video = await renderVideo({
    composition: 'ContentVideo',
    data: videoData,
    format: 'vertical', // 'vertical' | 'square' | 'horizontal'
    platform: 'reels', // 'reels' | 'tiktok' | 'shorts'
    outputPath: './output/video.mp4'
  });

  return video;
}
```

### 5. Using Claude for Content Analysis

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

async function analyzeWithClaude(articles: string[]) {
  const message = await anthropic.messages.create({
    model: 'claude-3-opus-20240229',
    max_tokens: 4096,
    messages: [{
      role: 'user',
      content: `Analyze these articles and extract key insights, trends, and data points:
      
${articles.join('\n\n---\n\n')}

Provide:
1. Top 5 trends
2. Important statistics
3. Expert quotes
4. Actionable insights`
    }]
  });

  return message.content;
}
```

### 6. Using OpenAI for Content Generation

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateWithOpenAI(prompt: string, format: string) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are an expert content creator specializing in ${format} format.`
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

## API Routes

### Generate Content Endpoint

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { createContentPipeline } from '@/lib/pipeline';

export async function POST(req: NextRequest) {
  const { keyword, format, languages } = await req.json();

  try {
    const result = await createContentPipeline({
      keyword,
      steps: {
        research: { enabled: true },
        generate: { formats: [format], languages },
        video: { enabled: false }
      }
    });

    return NextResponse.json(result);
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
// app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { crawlNews } from '@/lib/crawler/news-crawler';

export async function POST(req: NextRequest) {
  const { keyword, sources } = await req.json();

  const articles = await crawlNews({
    keyword,
    sources: sources || ['techcrunch', 'a16z'],
    timeframe: '24h'
  });

  return NextResponse.json({ articles });
}
```

## Configuration

### Content Formats

```typescript
// config/formats.ts
export const contentFormats = {
  toplist: {
    structure: 'numbered-list',
    minItems: 5,
    maxItems: 10,
    includeIntro: true,
    includeConclusion: true
  },
  pov: {
    structure: 'opinion-piece',
    includeEvidence: true,
    tone: 'authoritative'
  },
  caseStudy: {
    structure: 'problem-solution-results',
    includeMetrics: true,
    includeTakeaways: true
  },
  howTo: {
    structure: 'step-by-step',
    includeVisuals: true,
    difficultyLevel: 'beginner'
  }
};
```

### Video Templates

```typescript
// remotion/config.ts
export const videoConfigs = {
  reels: {
    width: 1080,
    height: 1920,
    fps: 30,
    duration: 60
  },
  tiktok: {
    width: 1080,
    height: 1920,
    fps: 30,
    duration: 60
  },
  shorts: {
    width: 1080,
    height: 1920,
    fps: 30,
    duration: 60
  }
};
```

## Common Patterns

### Complete Content Workflow

```typescript
import { createContentPipeline } from '@/lib/pipeline';
import { publishToSocial } from '@/lib/publisher';

async function completeWorkflow(keyword: string) {
  // 1. Research + Generate + Video
  const content = await createContentPipeline({
    keyword,
    steps: {
      research: {
        enabled: true,
        sources: ['techcrunch', 'a16z', 'twitter']
      },
      generate: {
        formats: ['toplist', 'how-to'],
        languages: ['en', 'vi'],
        aiModel: 'claude-3-opus'
      },
      video: {
        enabled: true,
        platforms: ['reels', 'tiktok']
      }
    }
  });

  // 2. Schedule publishing
  await publishToSocial({
    content: content.articles,
    videos: content.videos,
    platforms: ['facebook', 'twitter', 'tiktok'],
    schedule: new Date(Date.now() + 24 * 60 * 60 * 1000) // Tomorrow
  });

  return content;
}
```

### Batch Content Generation

```typescript
async function batchGenerate(keywords: string[]) {
  const results = await Promise.all(
    keywords.map(keyword =>
      createContentPipeline({
        keyword,
        steps: {
          research: { enabled: true },
          generate: {
            formats: ['toplist'],
            languages: ['en']
          },
          video: { enabled: false }
        }
      })
    )
  );

  return results;
}
```

## Troubleshooting

### API Rate Limits

```typescript
// Add retry logic with exponential backoff
import { retry } from '@/lib/utils/retry';

async function generateWithRetry(prompt: string) {
  return retry(
    () => generateWithOpenAI(prompt, 'toplist'),
    {
      maxAttempts: 3,
      delayMs: 1000,
      backoff: 'exponential'
    }
  );
}
```

### Video Rendering Issues

```bash
# Ensure FFmpeg is installed
brew install ffmpeg  # macOS
apt-get install ffmpeg  # Ubuntu

# Check Remotion installation
npx remotion versions
```

### Claude API Errors

```typescript
// Handle token limits
async function handleLongContent(content: string) {
  const maxTokens = 100000; // Claude 3 Opus limit
  
  if (estimateTokens(content) > maxTokens) {
    // Split into chunks
    const chunks = splitIntoChunks(content, maxTokens);
    const results = await Promise.all(
      chunks.map(chunk => analyzeWithClaude([chunk]))
    );
    return mergeResults(results);
  }
  
  return analyzeWithClaude([content]);
}
```

### Memory Issues During Video Rendering

```typescript
// Render videos sequentially instead of parallel
async function renderVideosSequentially(contents: any[]) {
  const videos = [];
  
  for (const content of contents) {
    const video = await generateVideo(content);
    videos.push(video);
    
    // Clear memory between renders
    if (global.gc) global.gc();
  }
  
  return videos;
}
```

### Missing Environment Variables

```typescript
// Add validation at startup
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

validateEnv();
```

## Best Practices

1. **Rate Limiting**: Implement delays between API calls
2. **Caching**: Cache research results to avoid redundant crawls
3. **Error Handling**: Always wrap AI calls in try-catch blocks
4. **Cost Management**: Monitor API usage and set budget limits
5. **Quality Control**: Review AI-generated content before publishing
6. **Video Optimization**: Pre-process images before video rendering

```typescript
// Example: Implement caching
import { cache } from '@/lib/utils/cache';

async function getCachedResearch(keyword: string) {
  const cacheKey = `research:${keyword}`;
  const cached = await cache.get(cacheKey);
  
  if (cached) return cached;
  
  const research = await crawlNews({ keyword });
  await cache.set(cacheKey, research, 3600); // 1 hour TTL
  
  return research;
}
```
```
