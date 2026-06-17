---
name: ultimate-ai-content-pipeline
description: Automated content pipeline from research to script to video generation using AI (Claude/OpenAI) and Remotion
triggers:
  - automate content creation with AI research and video generation
  - build automated marketing content pipeline with Claude
  - generate videos from AI-written content automatically
  - create content from keyword research to video render
  - set up AI-powered content automation system
  - crawl news sources and generate social media content
  - build end-to-end content pipeline with Remotion
  - automate blog posts and video creation workflow
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

Ultimate AI Content Pipeline is a TypeScript-based automation system that transforms a single keyword into complete content assets including research, written content, and rendered videos. It crawls news sources (TechCrunch, a16z, Twitter, LinkedIn), uses AI (Claude 3/OpenAI) to generate multi-format content in multiple languages, and automatically renders videos using Remotion.

**Key capabilities:**
- Auto-crawl fresh news data from major tech sources
- Generate content in multiple formats (toplist, POV, case study, how-to)
- Bilingual content (English/Vietnamese) with customizable tone
- Automatic video/infographic rendering via Remotion
- Built on Next.js with API integration to OpenAI, Anthropic, and RapidAPI

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
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_anthropic_key

# Research/Crawling APIs
RAPIDAPI_KEY=your_rapidapi_key

# Next.js
NEXT_PUBLIC_API_URL=http://localhost:3000

# Remotion (if using cloud rendering)
REMOTION_LICENSE_KEY=your_remotion_license

# Database (if applicable)
DATABASE_URL=your_database_connection_string
```

## Project Structure

```
marketing-pipeline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (OpenAI, Claude)
│   │   ├── crawler/     # News crawling logic
│   │   ├── content/     # Content generation
│   │   └── video/       # Remotion video rendering
│   ├── types/           # TypeScript types
│   └── utils/           # Utility functions
├── remotion/            # Remotion video compositions
└── public/              # Static assets
```

## Core Usage Patterns

### 1. Research & Data Crawling

```typescript
import { NewsResearcher } from '@/lib/crawler/news-researcher';

async function gatherResearch(keyword: string) {
  const researcher = new NewsResearcher({
    apiKey: process.env.RAPIDAPI_KEY!,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin']
  });

  // Crawl last 24 hours of news
  const articles = await researcher.search({
    query: keyword,
    timeRange: '24h',
    limit: 20
  });

  // Extract insights
  const insights = await researcher.extractInsights(articles);
  
  return {
    articles,
    insights,
    stats: researcher.getStatistics()
  };
}
```

### 2. AI Content Generation with Claude

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

async function generateContent(
  research: ResearchData,
  format: 'toplist' | 'pov' | 'case-study' | 'how-to',
  language: 'en' | 'vi',
  tone: 'expert' | 'friendly' | 'humorous'
) {
  const prompt = buildContentPrompt(research, format, language, tone);

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [{
      role: 'user',
      content: prompt
    }]
  });

  return {
    content: message.content[0].text,
    format,
    language,
    metadata: extractMetadata(message.content[0].text)
  };
}

function buildContentPrompt(
  research: ResearchData,
  format: string,
  language: string,
  tone: string
): string {
  return `
You are a content creator. Based on the following research data:

${JSON.stringify(research.insights, null, 2)}

Create a ${format} article in ${language} with a ${tone} tone.

Requirements:
- Include latest statistics and data points
- Reference sources from the research
- Make it engaging and actionable
- Length: 1500-2000 words
${language === 'vi' ? '- Write naturally in Vietnamese, avoid direct translation patterns' : ''}
`;
}
```

### 3. OpenAI Alternative

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateWithGPT(
  research: ResearchData,
  format: string
) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: 'You are an expert content marketer specializing in data-driven articles.'
      },
      {
        role: 'user',
        content: buildContentPrompt(research, format, 'en', 'expert')
      }
    ],
    temperature: 0.7,
    max_tokens: 4000
  });

  return completion.choices[0].message.content;
}
```

### 4. Video Generation with Remotion

```typescript
// remotion/compositions/ContentVideo.tsx
import { AbsoluteFill, Sequence, useCurrentFrame } from 'remotion';

export const ContentVideo: React.FC<{
  title: string;
  keyPoints: string[];
  duration: number;
}> = ({ title, keyPoints, duration }) => {
  const frame = useCurrentFrame();

  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a1a' }}>
      <Sequence from={0} durationInFrames={60}>
        <TitleSlide title={title} frame={frame} />
      </Sequence>
      
      {keyPoints.map((point, index) => (
        <Sequence
          key={index}
          from={60 + index * 90}
          durationInFrames={90}
        >
          <KeyPointSlide point={point} index={index} />
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};
```

```typescript
// src/lib/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

export async function renderContentVideo(
  content: GeneratedContent,
  outputPath: string
) {
  const compositionId = 'ContentVideo';
  
  // Bundle the Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  // Get composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: {
      title: content.metadata.title,
      keyPoints: content.metadata.keyPoints,
      duration: 30 // seconds
    },
  });

  // Render video
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: composition.props,
  });

  return outputPath;
}
```

### 5. Complete Pipeline Orchestration

```typescript
// src/lib/pipeline/orchestrator.ts
import { NewsResearcher } from '@/lib/crawler/news-researcher';
import { ContentGenerator } from '@/lib/content/generator';
import { VideoRenderer } from '@/lib/video/renderer';

export class ContentPipeline {
  private researcher: NewsResearcher;
  private generator: ContentGenerator;
  private renderer: VideoRenderer;

  constructor() {
    this.researcher = new NewsResearcher({
      apiKey: process.env.RAPIDAPI_KEY!
    });
    this.generator = new ContentGenerator({
      anthropicKey: process.env.ANTHROPIC_API_KEY!,
      openaiKey: process.env.OPENAI_API_KEY!
    });
    this.renderer = new VideoRenderer();
  }

  async run(config: PipelineConfig) {
    try {
      // Step 1: Research
      console.log('🔍 Starting research phase...');
      const research = await this.researcher.search({
        query: config.keyword,
        timeRange: '24h'
      });

      // Step 2: Generate content (bilingual)
      console.log('✍️ Generating content...');
      const [contentEN, contentVI] = await Promise.all([
        this.generator.generate({
          research,
          format: config.format,
          language: 'en',
          tone: config.tone
        }),
        this.generator.generate({
          research,
          format: config.format,
          language: 'vi',
          tone: config.tone
        })
      ]);

      // Step 3: Render videos
      console.log('🎬 Rendering videos...');
      const [videoEN, videoVI] = await Promise.all([
        this.renderer.render(contentEN, `output/video-en-${Date.now()}.mp4`),
        this.renderer.render(contentVI, `output/video-vi-${Date.now()}.mp4`)
      ]);

      return {
        research,
        content: { en: contentEN, vi: contentVI },
        videos: { en: videoEN, vi: videoVI }
      };
    } catch (error) {
      console.error('Pipeline error:', error);
      throw error;
    }
  }
}

// Usage
interface PipelineConfig {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  tone: 'expert' | 'friendly' | 'humorous';
}

const pipeline = new ContentPipeline();
const result = await pipeline.run({
  keyword: 'AI marketing automation',
  format: 'toplist',
  tone: 'expert'
});
```

## API Routes (Next.js)

### Content Generation Endpoint

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { ContentPipeline } from '@/lib/pipeline/orchestrator';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const { keyword, format, tone } = body;

    if (!keyword) {
      return NextResponse.json(
        { error: 'Keyword is required' },
        { status: 400 }
      );
    }

    const pipeline = new ContentPipeline();
    const result = await pipeline.run({
      keyword,
      format: format || 'toplist',
      tone: tone || 'expert'
    });

    return NextResponse.json({
      success: true,
      data: result
    });
  } catch (error) {
    console.error('API error:', error);
    return NextResponse.json(
      { error: 'Internal server error' },
      { status: 500 }
    );
  }
}
```

### Research Only Endpoint

```typescript
// src/app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { NewsResearcher } from '@/lib/crawler/news-researcher';

export async function GET(request: NextRequest) {
  const searchParams = request.nextUrl.searchParams;
  const keyword = searchParams.get('keyword');
  const timeRange = searchParams.get('timeRange') || '24h';

  if (!keyword) {
    return NextResponse.json(
      { error: 'Keyword query parameter is required' },
      { status: 400 }
    );
  }

  const researcher = new NewsResearcher({
    apiKey: process.env.RAPIDAPI_KEY!
  });

  const results = await researcher.search({
    query: keyword,
    timeRange
  });

  return NextResponse.json({
    success: true,
    data: results
  });
}
```

## Running the Application

### Development Mode

```bash
# Start Next.js dev server
npm run dev

# The app will be available at http://localhost:3000
```

### Build for Production

```bash
npm run build
npm start
```

### Render Videos Locally

```bash
# Install Remotion CLI globally
npm i -g @remotion/cli

# Preview composition
npx remotion preview remotion/index.ts

# Render specific composition
npx remotion render remotion/index.ts ContentVideo output/video.mp4 \
  --props='{"title":"My Title","keyPoints":["Point 1","Point 2"]}'
```

## Common Patterns

### Batch Content Generation

```typescript
async function generateBatchContent(keywords: string[]) {
  const pipeline = new ContentPipeline();
  
  const results = await Promise.allSettled(
    keywords.map(keyword => 
      pipeline.run({
        keyword,
        format: 'toplist',
        tone: 'expert'
      })
    )
  );

  return results.map((result, index) => ({
    keyword: keywords[index],
    success: result.status === 'fulfilled',
    data: result.status === 'fulfilled' ? result.value : null,
    error: result.status === 'rejected' ? result.reason : null
  }));
}
```

### Scheduled Content Pipeline

```typescript
import cron from 'node-cron';

// Run pipeline every day at 9 AM
cron.schedule('0 9 * * *', async () => {
  const pipeline = new ContentPipeline();
  
  const trendingTopics = await fetchTrendingTopics();
  
  for (const topic of trendingTopics) {
    await pipeline.run({
      keyword: topic,
      format: 'pov',
      tone: 'friendly'
    });
  }
});
```

### Custom Video Templates

```typescript
// remotion/compositions/SocialMediaPost.tsx
export const SocialMediaPost: React.FC<{
  platform: 'reels' | 'tiktok' | 'shorts';
  content: string;
  branding: BrandConfig;
}> = ({ platform, content, branding }) => {
  const aspectRatio = platform === 'reels' ? 9/16 : 9/16;
  
  return (
    <AbsoluteFill style={{ 
      backgroundColor: branding.backgroundColor,
      aspectRatio 
    }}>
      {/* Custom composition based on platform */}
    </AbsoluteFill>
  );
};
```

## Troubleshooting

### API Rate Limits

```typescript
import pRetry from 'p-retry';

async function generateWithRetry(params: GenerateParams) {
  return pRetry(
    async () => {
      return await generateContent(params);
    },
    {
      retries: 3,
      onFailedAttempt: error => {
        console.log(
          `Attempt ${error.attemptNumber} failed. ${error.retriesLeft} retries left.`
        );
      }
    }
  );
}
```

### Memory Issues with Video Rendering

```typescript
// Render videos sequentially instead of parallel
async function renderVideosSequentially(contents: GeneratedContent[]) {
  const results = [];
  
  for (const content of contents) {
    const video = await renderContentVideo(content, `output/${content.id}.mp4`);
    results.push(video);
    
    // Give system time to free memory
    await new Promise(resolve => setTimeout(resolve, 2000));
  }
  
  return results;
}
```

### Claude API Errors

```typescript
import Anthropic from '@anthropic-ai/sdk';

async function safeClaudeCall(prompt: string) {
  const anthropic = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY,
  });

  try {
    const message = await anthropic.messages.create({
      model: 'claude-3-5-sonnet-20241022',
      max_tokens: 4096,
      messages: [{ role: 'user', content: prompt }]
    });
    
    return message.content[0].text;
  } catch (error) {
    if (error instanceof Anthropic.APIError) {
      console.error('Claude API Error:', error.status, error.message);
      // Fallback to OpenAI
      return await generateWithGPT(prompt);
    }
    throw error;
  }
}
```

### News Crawler Timeouts

```typescript
async function crawlWithTimeout(keyword: string, timeoutMs = 30000) {
  const researcher = new NewsResearcher({
    apiKey: process.env.RAPIDAPI_KEY!,
    timeout: timeoutMs
  });

  const timeoutPromise = new Promise((_, reject) =>
    setTimeout(() => reject(new Error('Research timeout')), timeoutMs)
  );

  const researchPromise = researcher.search({ query: keyword });

  return Promise.race([researchPromise, timeoutPromise]);
}
```

## TypeScript Types Reference

```typescript
interface ResearchData {
  articles: Article[];
  insights: Insight[];
  stats: Statistics;
}

interface Article {
  title: string;
  url: string;
  source: string;
  publishedAt: string;
  content: string;
  relevanceScore: number;
}

interface GeneratedContent {
  id: string;
  content: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  metadata: {
    title: string;
    keyPoints: string[];
    wordCount: number;
    estimatedReadTime: number;
  };
}

interface PipelineResult {
  research: ResearchData;
  content: {
    en: GeneratedContent;
    vi: GeneratedContent;
  };
  videos: {
    en: string;
    vi: string;
  };
}
```
