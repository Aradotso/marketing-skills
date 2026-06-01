---
name: marketing-pipeline-ai-content-automation
description: Ultimate AI Content Pipeline for automated research, content generation, and video creation using Claude, OpenAI, and Remotion
triggers:
  - automate content creation pipeline with AI
  - generate marketing content from research automatically
  - create AI-powered content with video rendering
  - set up automated content research and generation
  - build content automation system with Claude and OpenAI
  - generate videos from content using Remotion
  - automate marketing content workflow
  - create AI content pipeline with multi-language support
---

# Marketing Pipeline AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

The Marketing Pipeline AI Content Automation system is a comprehensive TypeScript-based solution that automates the entire content creation workflow from research to video generation. It crawls real-time data from sources like TechCrunch, a16z, Twitter, and LinkedIn, generates content in multiple formats using Claude/OpenAI, and automatically renders videos using Remotion.

**Key Capabilities:**
- Auto-research: Crawl and analyze recent news (24h data)
- Multi-format content generation (Toplist, POV, Case Study, How-to)
- Bilingual support (English/Vietnamese)
- Automatic video/infographic rendering via Remotion
- Next.js frontend for easy content management

## Installation

### Prerequisites

```bash
# Node.js 18+ and npm/yarn required
node --version  # Should be 18+
```

### Setup

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
yarn install

# Create environment variables file
cp .env.example .env.local
```

### Environment Configuration

Create `.env.local` with the following variables:

```bash
# AI APIs
OPENAI_API_KEY=your_openai_key_here
ANTHROPIC_API_KEY=your_claude_api_key_here

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key_here

# Database (if using)
DATABASE_URL=your_database_connection_string

# Remotion Configuration
REMOTION_REGION=us-east-1

# Application
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

### Run Development Server

```bash
npm run dev
# or
yarn dev
```

The application will be available at `http://localhost:3000`

## Core Architecture

### Project Structure

```
marketing-pineline-share/
├── app/                    # Next.js app directory
├── components/            # React components
├── lib/                   # Core utilities
│   ├── ai/               # AI integrations (Claude, OpenAI)
│   ├── research/         # Web scraping & research
│   └── video/            # Remotion video rendering
├── remotion/             # Remotion video templates
└── public/               # Static assets
```

## Key Features & Usage

### 1. Content Research Automation

**Crawl Recent News:**

```typescript
import { researchService } from '@/lib/research/research-service';

// Fetch recent articles on a topic
async function fetchLatestNews(keyword: string) {
  const articles = await researchService.crawlNews({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter'],
    timeRange: '24h',
    limit: 20
  });
  
  return articles;
}

// Extract insights from articles
async function extractInsights(articles: Article[]) {
  const insights = await researchService.analyzeArticles(articles, {
    extractStats: true,
    findTrends: true,
    identifyExperts: true
  });
  
  return insights;
}
```

**Custom Research Pipeline:**

```typescript
import { ResearchPipeline } from '@/lib/research/pipeline';

const pipeline = new ResearchPipeline({
  sources: ['techcrunch', 'linkedin', 'twitter'],
  keywords: ['AI marketing', 'content automation'],
  filters: {
    minEngagement: 100,
    language: 'en',
    recency: '24h'
  }
});

// Run full research cycle
const researchData = await pipeline.execute();
// Returns: { articles, insights, stats, experts }
```

### 2. AI Content Generation

**Generate Content with Claude:**

```typescript
import { anthropic } from '@anthropic-ai/sdk';
import { generateContent } from '@/lib/ai/claude-generator';

async function createArticle(topic: string, research: ResearchData) {
  const content = await generateContent({
    model: 'claude-3-5-sonnet-20241022',
    format: 'toplist', // or 'pov', 'case-study', 'how-to'
    topic,
    research,
    language: 'en',
    tone: 'expert', // or 'friendly', 'humorous'
    apiKey: process.env.ANTHROPIC_API_KEY
  });
  
  return content;
}

// Example output structure
interface GeneratedContent {
  title: string;
  introduction: string;
  sections: Array<{
    heading: string;
    content: string;
    stats?: string[];
  }>;
  conclusion: string;
  metadata: {
    wordCount: number;
    readingTime: number;
    keywords: string[];
  };
}
```

**Multi-Language Content:**

```typescript
import { translateContent } from '@/lib/ai/translator';

async function generateBilingualContent(topic: string, research: ResearchData) {
  // Generate English version
  const englishContent = await generateContent({
    topic,
    research,
    language: 'en'
  });
  
  // Generate Vietnamese version
  const vietnameseContent = await generateContent({
    topic,
    research,
    language: 'vi'
  });
  
  return {
    en: englishContent,
    vi: vietnameseContent
  };
}
```

**Using OpenAI Alternative:**

```typescript
import OpenAI from 'openai';
import { generateWithOpenAI } from '@/lib/ai/openai-generator';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY
});

async function createContentOpenAI(topic: string, research: ResearchData) {
  const content = await generateWithOpenAI({
    model: 'gpt-4-turbo-preview',
    topic,
    research,
    format: 'case-study',
    systemPrompt: 'You are an expert marketing content writer...'
  });
  
  return content;
}
```

### 3. Video Generation with Remotion

**Basic Video Rendering:**

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

async function renderContentVideo(content: GeneratedContent) {
  // Bundle Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: (config) => config
  });
  
  // Select composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title: content.title,
      sections: content.sections,
      duration: 60 // seconds
    }
  });
  
  // Render video
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: `out/${content.title.replace(/\s+/g, '-')}.mp4`,
    inputProps: {
      title: content.title,
      sections: content.sections
    }
  });
}
```

**Remotion Composition Example:**

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, Sequence, useCurrentFrame } from 'remotion';

interface ContentVideoProps {
  title: string;
  sections: Array<{ heading: string; content: string }>;
}

export const ContentVideo: React.FC<ContentVideoProps> = ({ title, sections }) => {
  const frame = useCurrentFrame();
  
  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a1a' }}>
      {/* Title sequence */}
      <Sequence from={0} durationInFrames={90}>
        <TitleCard title={title} frame={frame} />
      </Sequence>
      
      {/* Content sections */}
      {sections.map((section, index) => (
        <Sequence
          key={index}
          from={90 + index * 120}
          durationInFrames={120}
        >
          <ContentCard section={section} />
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};
```

**Platform-Specific Rendering:**

```typescript
import { renderForPlatform } from '@/lib/video/platform-renderer';

async function renderMultiPlatform(content: GeneratedContent) {
  // Render for different platforms
  await Promise.all([
    // Instagram Reels (9:16)
    renderForPlatform(content, {
      platform: 'reels',
      width: 1080,
      height: 1920,
      fps: 30
    }),
    
    // YouTube Shorts (9:16)
    renderForPlatform(content, {
      platform: 'shorts',
      width: 1080,
      height: 1920,
      fps: 30
    }),
    
    // TikTok (9:16)
    renderForPlatform(content, {
      platform: 'tiktok',
      width: 1080,
      height: 1920,
      fps: 30
    })
  ]);
}
```

### 4. Complete Pipeline Integration

**Full Automation Workflow:**

```typescript
import { ContentPipeline } from '@/lib/pipeline/content-pipeline';

async function runFullPipeline(keyword: string) {
  const pipeline = new ContentPipeline({
    aiProvider: 'claude', // or 'openai'
    languages: ['en', 'vi'],
    formats: ['toplist', 'how-to'],
    generateVideo: true
  });
  
  // Execute complete pipeline
  const result = await pipeline.execute({
    keyword,
    researchConfig: {
      sources: ['techcrunch', 'a16z'],
      timeRange: '24h'
    },
    contentConfig: {
      tone: 'expert',
      minWordCount: 1500
    },
    videoConfig: {
      platforms: ['reels', 'tiktok'],
      duration: 60
    }
  });
  
  return result;
  // Returns: { content, videos, research, metadata }
}
```

**Scheduled Content Generation:**

```typescript
import { scheduleContentGeneration } from '@/lib/scheduler';

// Schedule daily content generation
scheduleContentGeneration({
  schedule: '0 9 * * *', // Every day at 9 AM
  keywords: ['AI marketing', 'content automation'],
  pipeline: {
    research: true,
    generate: true,
    render: true,
    publish: false // Set to true for auto-publishing
  },
  notification: {
    email: process.env.NOTIFICATION_EMAIL,
    webhook: process.env.WEBHOOK_URL
  }
});
```

## API Routes

### Research Endpoint

```typescript
// app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { researchService } from '@/lib/research/research-service';

export async function POST(request: NextRequest) {
  const { keyword, sources, timeRange } = await request.json();
  
  const articles = await researchService.crawlNews({
    keyword,
    sources,
    timeRange
  });
  
  return NextResponse.json({ articles });
}
```

### Content Generation Endpoint

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { generateContent } from '@/lib/ai/claude-generator';

export async function POST(request: NextRequest) {
  const { topic, research, format, language } = await request.json();
  
  const content = await generateContent({
    topic,
    research,
    format,
    language,
    apiKey: process.env.ANTHROPIC_API_KEY
  });
  
  return NextResponse.json({ content });
}
```

### Video Rendering Endpoint

```typescript
// app/api/render/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { renderContentVideo } from '@/lib/video/renderer';

export async function POST(request: NextRequest) {
  const { content, platform } = await request.json();
  
  const videoUrl = await renderContentVideo(content, { platform });
  
  return NextResponse.json({ videoUrl });
}
```

## Common Patterns

### Pattern 1: Research + Generate + Render

```typescript
async function completeContentWorkflow(topic: string) {
  // Step 1: Research
  const research = await researchService.crawlNews({
    keyword: topic,
    sources: ['techcrunch', 'twitter'],
    timeRange: '24h'
  });
  
  // Step 2: Generate content
  const content = await generateContent({
    topic,
    research,
    format: 'toplist',
    language: 'en'
  });
  
  // Step 3: Render video
  const video = await renderContentVideo(content);
  
  return { content, video };
}
```

### Pattern 2: Batch Content Generation

```typescript
async function batchGenerateContent(topics: string[]) {
  const results = await Promise.allSettled(
    topics.map(topic => completeContentWorkflow(topic))
  );
  
  const successful = results
    .filter(r => r.status === 'fulfilled')
    .map(r => (r as PromiseFulfilledResult<any>).value);
  
  return successful;
}
```

### Pattern 3: Custom Content Format

```typescript
interface CustomFormat {
  structure: string[];
  tone: string;
  includeStats: boolean;
  callToAction: string;
}

async function generateCustomFormat(
  topic: string,
  research: ResearchData,
  format: CustomFormat
) {
  const prompt = buildCustomPrompt(format);
  
  const content = await generateContent({
    topic,
    research,
    systemPrompt: prompt,
    language: 'en'
  });
  
  return content;
}
```

## Configuration

### AI Model Configuration

```typescript
// lib/config/ai-config.ts
export const AI_CONFIG = {
  claude: {
    model: 'claude-3-5-sonnet-20241022',
    maxTokens: 4000,
    temperature: 0.7
  },
  openai: {
    model: 'gpt-4-turbo-preview',
    maxTokens: 4000,
    temperature: 0.7
  }
};
```

### Research Configuration

```typescript
// lib/config/research-config.ts
export const RESEARCH_CONFIG = {
  sources: {
    techcrunch: {
      url: 'https://techcrunch.com',
      selector: '.post-block'
    },
    a16z: {
      url: 'https://a16z.com/blog',
      selector: 'article'
    }
  },
  timeRanges: {
    '24h': 86400000,
    '7d': 604800000,
    '30d': 2592000000
  }
};
```

### Video Configuration

```typescript
// remotion.config.ts
import { Config } from '@remotion/cli/config';

Config.setVideoImageFormat('jpeg');
Config.setPixelFormat('yuv420p');
Config.setCodec('h264');
Config.setConcurrency(4);
Config.setOverwriteOutput(true);
```

## Troubleshooting

### Issue: AI API Rate Limits

```typescript
import { retryWithBackoff } from '@/lib/utils/retry';

async function generateWithRetry(params: any) {
  return retryWithBackoff(
    () => generateContent(params),
    {
      maxRetries: 3,
      initialDelay: 1000,
      maxDelay: 10000
    }
  );
}
```

### Issue: Research Data Quality

```typescript
import { validateResearchData } from '@/lib/research/validator';

async function getQualityResearch(keyword: string) {
  const articles = await researchService.crawlNews({ keyword });
  
  // Filter low-quality articles
  const validated = articles.filter(article => 
    validateResearchData(article, {
      minWordCount: 300,
      requireStats: true,
      requireAuthor: true
    })
  );
  
  return validated;
}
```

### Issue: Video Rendering Performance

```typescript
// Use faster codec for development
const devRenderConfig = {
  codec: 'h264',
  crf: 23, // Lower = better quality, slower
  pixelFormat: 'yuv420p',
  concurrency: 8 // Adjust based on CPU cores
};

// Production optimized
const prodRenderConfig = {
  codec: 'h264',
  crf: 18,
  pixelFormat: 'yuv420p',
  concurrency: 4
};
```

### Issue: Memory Management for Large Projects

```typescript
import { processInBatches } from '@/lib/utils/batch-processor';

async function processLargeDataset(items: any[]) {
  return processInBatches(items, {
    batchSize: 10,
    concurrency: 3,
    onBatchComplete: (results) => {
      console.log(`Processed ${results.length} items`);
    }
  });
}
```

## Best Practices

1. **Always validate research data** before generating content
2. **Use environment variables** for all API keys and secrets
3. **Implement rate limiting** for AI API calls
4. **Cache research results** to avoid redundant API calls
5. **Test video rendering** on small compositions first
6. **Monitor token usage** to control AI costs
7. **Implement error handling** at each pipeline stage
8. **Use TypeScript types** for all data structures
9. **Version control your prompts** for consistent output
10. **Test multi-language output** with native speakers
