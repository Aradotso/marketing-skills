---
name: marketing-pipeline-ai-content-automation
description: Automated AI content pipeline from research to video generation using Claude, OpenAI, and Remotion
triggers:
  - how do I automate content creation with AI
  - create automated marketing content pipeline
  - generate videos from text content automatically
  - set up AI content research and generation
  - build automated social media content workflow
  - use Claude and OpenAI for content automation
  - create video content from articles with Remotion
  - automate content from research to publishing
---

# Marketing Pipeline AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to help developers build and use an automated content pipeline that handles research, script writing, and video generation using Claude 3, OpenAI, and Remotion.

## What This Project Does

The Marketing Pipeline is a complete content automation system that:

- **Auto-scans research sources** (TechCrunch, a16z, Twitter, LinkedIn) for fresh content
- **Generates multi-format content** (top lists, POV articles, case studies, how-tos) in Vietnamese and English
- **Renders videos and infographics** automatically using Remotion
- **Optimizes for multiple platforms** (Reels, TikTok, Shorts)

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
# AI APIs
ANTHROPIC_API_KEY=your_claude_api_key_here
OPENAI_API_KEY=your_openai_api_key_here

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key_here

# Optional: Custom configuration
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

## Project Structure

```
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/              # Core libraries
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── research/    # Content research & scraping
│   │   ├── video/       # Remotion video generation
│   │   └── utils/       # Helper functions
│   └── types/           # TypeScript types
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Key APIs and Usage

### 1. Research & Content Scraping

```typescript
import { researchTopic } from '@/lib/research/scraper';

async function fetchLatestContent(keyword: string) {
  const researchData = await researchTopic({
    keyword,
    sources: ['techcrunch', 'twitter', 'linkedin'],
    timeframe: '24h',
    language: 'en'
  });
  
  return {
    articles: researchData.articles,
    insights: researchData.insights,
    trending: researchData.trendingTopics
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
  topic: string, 
  format: 'toplist' | 'pov' | 'casestudy' | 'howto',
  language: 'en' | 'vi'
) {
  const prompt = `Write a ${format} article about "${topic}" in ${language}.
  Use the following research data: ${researchData}
  Format: Professional, data-backed, with real examples.`;
  
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

### 3. AI Content Generation with OpenAI

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateContentVariations(baseContent: string) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: 'You are a content marketing expert.'
      },
      {
        role: 'user',
        content: `Create 3 variations of this content for different platforms: ${baseContent}`
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
import { webpackOverride } from './remotion/webpack-override';

async function generateVideo(
  contentData: {
    title: string;
    points: string[];
    images: string[];
  },
  outputPath: string
) {
  // Bundle Remotion composition
  const bundled = await bundle({
    entryPoint: './remotion/index.ts',
    webpackOverride,
  });
  
  // Select composition
  const composition = await selectComposition({
    serveUrl: bundled,
    id: 'ContentVideo',
    inputProps: contentData,
  });
  
  // Render video
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

## Complete Pipeline Example

```typescript
import { researchTopic } from '@/lib/research/scraper';
import { generateWithClaude } from '@/lib/ai/claude';
import { renderContentVideo } from '@/lib/video/remotion';

async function runContentPipeline(keyword: string) {
  // Step 1: Research
  console.log('🔍 Researching topic...');
  const research = await researchTopic({
    keyword,
    sources: ['techcrunch', 'twitter'],
    timeframe: '24h'
  });
  
  // Step 2: Generate content
  console.log('✍️ Generating content...');
  const article = await generateWithClaude({
    topic: keyword,
    research: research.insights,
    format: 'toplist',
    language: 'en',
    tone: 'professional'
  });
  
  // Step 3: Generate Vietnamese version
  const articleVi = await generateWithClaude({
    topic: keyword,
    research: research.insights,
    format: 'toplist',
    language: 'vi',
    tone: 'professional'
  });
  
  // Step 4: Generate video
  console.log('🎬 Rendering video...');
  const videoPath = await renderContentVideo({
    title: article.title,
    points: article.keyPoints,
    images: research.images,
    format: 'vertical', // for TikTok/Reels
    duration: 60
  });
  
  return {
    article: article,
    articleVi: articleVi,
    video: videoPath,
    research: research
  };
}

// Usage
const result = await runContentPipeline('AI automation trends 2024');
console.log('✅ Content pipeline complete!', result);
```

## API Routes (Next.js)

### Generate Content Endpoint

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline';

export async function POST(request: NextRequest) {
  try {
    const { keyword, format, language } = await request.json();
    
    const result = await runContentPipeline(keyword);
    
    return NextResponse.json({
      success: true,
      data: result
    });
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
import { researchTopic } from '@/lib/research/scraper';

export async function POST(request: NextRequest) {
  const { keyword, sources, timeframe } = await request.json();
  
  const data = await researchTopic({
    keyword,
    sources: sources || ['techcrunch', 'twitter'],
    timeframe: timeframe || '24h'
  });
  
  return NextResponse.json({ data });
}
```

## Common Patterns

### Content Format Templates

```typescript
type ContentFormat = 'toplist' | 'pov' | 'casestudy' | 'howto';

const formatPrompts: Record<ContentFormat, string> = {
  toplist: 'Create a numbered list article with at least 5 items, each with detailed explanations',
  pov: 'Write an opinion piece from a specific perspective, backed by data and examples',
  casestudy: 'Analyze a real-world example with problem, solution, and results',
  howto: 'Create a step-by-step tutorial with actionable instructions'
};

function buildPrompt(format: ContentFormat, topic: string, research: any) {
  return `${formatPrompts[format]}
  
  Topic: ${topic}
  Research data: ${JSON.stringify(research)}
  
  Requirements:
  - Use data-backed insights
  - Include real examples
  - Optimize for engagement
  - Add relevant statistics`;
}
```

### Multi-Platform Video Export

```typescript
const platformConfigs = {
  tiktok: { width: 1080, height: 1920, fps: 30, duration: 60 },
  reels: { width: 1080, height: 1920, fps: 30, duration: 60 },
  shorts: { width: 1080, height: 1920, fps: 30, duration: 60 },
  youtube: { width: 1920, height: 1080, fps: 30, duration: 120 },
};

async function exportForPlatforms(content: any, platforms: string[]) {
  const videos = await Promise.all(
    platforms.map(platform => 
      renderContentVideo({
        ...content,
        ...platformConfigs[platform]
      })
    )
  );
  
  return videos;
}
```

### Error Handling with Retries

```typescript
async function withRetry<T>(
  fn: () => Promise<T>,
  maxRetries = 3,
  delay = 1000
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      await new Promise(resolve => setTimeout(resolve, delay * (i + 1)));
    }
  }
  throw new Error('Max retries exceeded');
}

// Usage
const content = await withRetry(() => 
  generateWithClaude({ topic: 'AI trends', format: 'toplist' })
);
```

## Running the Application

```bash
# Development mode
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Render videos (Remotion)
npm run remotion:render
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement rate limiting for AI APIs
import pLimit from 'p-limit';

const limit = pLimit(3); // Max 3 concurrent requests

async function batchGenerate(topics: string[]) {
  return Promise.all(
    topics.map(topic => 
      limit(() => generateWithClaude({ topic, format: 'toplist' }))
    )
  );
}
```

### Video Rendering Errors

```typescript
// Check Remotion dependencies
try {
  await renderContentVideo(content);
} catch (error) {
  if (error.message.includes('ENOENT')) {
    console.error('Missing video assets. Check public/assets folder');
  } else if (error.message.includes('memory')) {
    console.error('Increase Node.js memory: NODE_OPTIONS=--max-old-space-size=4096');
  }
  throw error;
}
```

### Claude API Errors

```typescript
// Handle Claude-specific errors
try {
  const result = await anthropic.messages.create({...});
} catch (error) {
  if (error.status === 429) {
    console.error('Rate limit exceeded. Wait before retrying.');
  } else if (error.status === 401) {
    console.error('Invalid API key. Check ANTHROPIC_API_KEY');
  }
  throw error;
}
```

### Environment Variable Issues

```typescript
// Validate environment variables on startup
const requiredEnvVars = [
  'ANTHROPIC_API_KEY',
  'OPENAI_API_KEY',
  'RAPIDAPI_KEY'
];

requiredEnvVars.forEach(varName => {
  if (!process.env[varName]) {
    throw new Error(`Missing required environment variable: ${varName}`);
  }
});
```

## TypeScript Types

```typescript
// types/content.ts
export interface ResearchData {
  articles: Article[];
  insights: string[];
  trendingTopics: string[];
  images: string[];
}

export interface Article {
  title: string;
  content: string;
  keyPoints: string[];
  language: 'en' | 'vi';
  format: ContentFormat;
  metadata: {
    wordCount: number;
    readingTime: number;
    sources: string[];
  };
}

export interface VideoConfig {
  width: number;
  height: number;
  fps: number;
  duration: number;
  codec: 'h264' | 'h265';
}
```

This skill provides comprehensive guidance for using the marketing pipeline to automate content creation from research through video generation using AI and modern web technologies.
