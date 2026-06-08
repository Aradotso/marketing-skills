---
name: marketing-pipeline-automation
description: AI-powered content automation pipeline from research to video generation using Claude, OpenAI, and Remotion
triggers:
  - how do I automate content creation with AI
  - generate video content from text automatically
  - set up an AI marketing pipeline
  - create automated content workflow
  - research and generate marketing content with AI
  - build automated video generation system
  - use Claude and OpenAI for content automation
  - automate content from research to publication
---

# Marketing Pipeline Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the **Ultimate AI Content Pipeline**, a comprehensive TypeScript-based system that automates the entire content creation workflow: from research and scriptwriting to video generation and publishing. The pipeline leverages Claude 3, OpenAI, and Remotion to transform keywords into production-ready content across multiple formats and languages.

## What This Project Does

The Marketing Pipeline Automation system provides:

- **Automated Research**: Crawls and analyzes real-time data from sources like TechCrunch, a16z, Twitter/X, and LinkedIn
- **AI Content Generation**: Creates diverse content formats (toplists, POV pieces, case studies, how-tos) in multiple languages using Claude/OpenAI
- **Video Rendering**: Automatically generates videos and infographics from text content using Remotion
- **Multi-Platform Optimization**: Exports content optimized for Reels, TikTok, Shorts, and other platforms

## Installation

### Prerequisites

```bash
# Required Node.js version
node --version  # Should be >= 18.x

# Required dependencies
npm install
# or
yarn install
# or
pnpm install
```

### Environment Configuration

Create a `.env.local` file in the project root:

```bash
# AI APIs
OPENAI_API_KEY=your_openai_key_here
ANTHROPIC_API_KEY=your_anthropic_claude_key_here

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key_here

# Optional: Custom research sources
RESEARCH_SOURCES=techcrunch,a16z,twitter,linkedin

# Video rendering
REMOTION_TIMEOUT=300000
VIDEO_OUTPUT_DIR=./output/videos

# Content settings
DEFAULT_LANGUAGE=vi,en
CONTENT_TONE=professional
```

### Running the Application

```bash
# Development mode
npm run dev

# Production build
npm run build
npm run start

# Video rendering server (Remotion)
npm run video-server
```

## Project Structure

```
marketing-pipeline-share/
├── src/
│   ├── app/              # Next.js app router pages
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── research/    # Content research & crawling
│   │   ├── video/       # Remotion video generation
│   │   └── utils/       # Helper functions
│   └── types/           # TypeScript definitions
├── public/              # Static assets
└── remotion/            # Video templates
```

## Core API Usage

### 1. Content Research Module

```typescript
import { researchTopic } from '@/lib/research/crawler';
import { analyzeInsights } from '@/lib/research/analyzer';

async function conductResearch(keyword: string) {
  // Crawl multiple sources for recent content
  const rawData = await researchTopic({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter'],
    timeframe: '24h',
    maxResults: 50
  });

  // Extract insights and data points
  const insights = await analyzeInsights(rawData, {
    extractStats: true,
    findTrends: true,
    includeQuotes: true
  });

  return insights;
}

// Usage
const insights = await conductResearch('AI automation marketing');
console.log(insights.trends, insights.keyStats);
```

### 2. AI Content Generation

```typescript
import { generateContent } from '@/lib/ai/content-generator';
import { ContentFormat, Language } from '@/types';

async function createArticle(
  topic: string,
  research: any,
  format: ContentFormat = 'toplist'
) {
  const content = await generateContent({
    topic,
    format, // 'toplist' | 'pov' | 'case-study' | 'how-to'
    research,
    languages: ['en', 'vi'],
    tone: 'professional',
    aiProvider: 'claude', // or 'openai'
    model: 'claude-3-opus-20240229'
  });

  return content;
}

// Generate bilingual content
const article = await createArticle(
  'Top 10 AI Marketing Tools 2024',
  researchData,
  'toplist'
);

// Access different language versions
console.log(article.en.title);
console.log(article.vi.title);
```

### 3. Using Claude for Content

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY
});

async function generateWithClaude(prompt: string, systemPrompt: string) {
  const message = await anthropic.messages.create({
    model: 'claude-3-opus-20240229',
    max_tokens: 4096,
    temperature: 0.7,
    system: systemPrompt,
    messages: [
      {
        role: 'user',
        content: prompt
      }
    ]
  });

  return message.content[0].text;
}

// Example: Generate marketing copy
const marketingCopy = await generateWithClaude(
  'Write a compelling headline for an AI automation tool',
  'You are an expert marketing copywriter specializing in B2B SaaS products.'
);
```

### 4. Using OpenAI Alternative

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY
});

async function generateWithOpenAI(prompt: string, systemMessage: string) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      { role: 'system', content: systemMessage },
      { role: 'user', content: prompt }
    ],
    temperature: 0.7,
    max_tokens: 3000
  });

  return completion.choices[0].message.content;
}
```

### 5. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

async function generateVideo(content: any, template: string = 'default') {
  // Bundle the Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.join(process.cwd(), 'remotion/index.ts'),
    webpackOverride: (config) => config
  });

  // Select composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: template,
    inputProps: {
      title: content.title,
      body: content.body,
      stats: content.stats,
      branding: {
        logo: '/logo.png',
        colors: ['#FF6B6B', '#4ECDC4']
      }
    }
  });

  // Render video
  const outputPath = path.join(
    process.env.VIDEO_OUTPUT_DIR || './output/videos',
    `${Date.now()}.mp4`
  );

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: composition.inputProps
  });

  return outputPath;
}

// Usage
const videoPath = await generateVideo(articleContent, 'infographic');
```

## Common Workflow Patterns

### Full Pipeline Execution

```typescript
import { runFullPipeline } from '@/lib/pipeline';

async function createCompleteContent(keyword: string) {
  const result = await runFullPipeline({
    keyword,
    steps: [
      'research',
      'generate-content',
      'create-visuals',
      'render-video'
    ],
    config: {
      contentFormat: 'toplist',
      languages: ['en', 'vi'],
      videoFormats: ['1:1', '9:16', '16:9'],
      aiProvider: 'claude'
    }
  });

  return {
    articles: result.content,
    videos: result.videos,
    metadata: result.metadata
  };
}

// Execute full automation
const output = await createCompleteContent('AI marketing automation trends');
```

### Batch Content Generation

```typescript
async function batchGenerateContent(keywords: string[]) {
  const results = await Promise.all(
    keywords.map(async (keyword) => {
      try {
        const research = await conductResearch(keyword);
        const content = await createArticle(keyword, research);
        const video = await generateVideo(content);
        
        return {
          keyword,
          success: true,
          content,
          video
        };
      } catch (error) {
        console.error(`Failed for ${keyword}:`, error);
        return {
          keyword,
          success: false,
          error: error.message
        };
      }
    })
  );

  return results;
}

// Generate content for multiple topics
const batch = await batchGenerateContent([
  'AI marketing tools',
  'Content automation trends',
  'Video marketing strategies'
]);
```

### Custom Content Templates

```typescript
import { createCustomTemplate } from '@/lib/ai/templates';

const customTemplate = createCustomTemplate({
  name: 'product-launch',
  structure: {
    sections: [
      { type: 'hook', maxWords: 50 },
      { type: 'problem', maxWords: 150 },
      { type: 'solution', maxWords: 200 },
      { type: 'features', listItems: 5 },
      { type: 'cta', maxWords: 30 }
    ]
  },
  tone: 'exciting',
  language: 'vi'
});

const launchContent = await generateContent({
  topic: 'New AI Tool Launch',
  template: customTemplate,
  research: researchData
});
```

## API Routes (Next.js)

### Research API

```typescript
// app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { researchTopic } from '@/lib/research/crawler';

export async function POST(request: NextRequest) {
  const { keyword, sources, timeframe } = await request.json();

  try {
    const results = await researchTopic({
      keyword,
      sources: sources || ['techcrunch', 'a16z'],
      timeframe: timeframe || '24h'
    });

    return NextResponse.json({ success: true, data: results });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

### Content Generation API

```typescript
// app/api/generate/route.ts
export async function POST(request: NextRequest) {
  const { topic, format, language, aiProvider } = await request.json();

  const content = await generateContent({
    topic,
    format,
    languages: [language],
    aiProvider: aiProvider || 'claude'
  });

  return NextResponse.json({ success: true, content });
}
```

### Video Rendering API

```typescript
// app/api/video/route.ts
export async function POST(request: NextRequest) {
  const { content, template, aspectRatio } = await request.json();

  const videoPath = await generateVideo(content, template);

  return NextResponse.json({
    success: true,
    videoUrl: `/videos/${path.basename(videoPath)}`
  });
}
```

## Configuration Options

### Research Configuration

```typescript
interface ResearchConfig {
  sources: ('techcrunch' | 'a16z' | 'twitter' | 'linkedin')[];
  timeframe: '24h' | '7d' | '30d';
  maxResults: number;
  includeImages: boolean;
  extractQuotes: boolean;
  language?: 'en' | 'vi';
}
```

### Content Generation Configuration

```typescript
interface ContentConfig {
  format: 'toplist' | 'pov' | 'case-study' | 'how-to' | 'listicle';
  tone: 'professional' | 'casual' | 'humorous' | 'authoritative';
  languages: ('en' | 'vi')[];
  wordCount?: number;
  includeSEO: boolean;
  aiProvider: 'claude' | 'openai';
  model?: string;
}
```

### Video Configuration

```typescript
interface VideoConfig {
  template: 'default' | 'infographic' | 'story' | 'presentation';
  aspectRatio: '1:1' | '9:16' | '16:9' | '4:5';
  duration: number; // seconds
  fps: 30 | 60;
  quality: 'low' | 'medium' | 'high';
  watermark?: string;
}
```

## Troubleshooting

### API Rate Limits

```typescript
import { RateLimiter } from '@/lib/utils/rate-limiter';

const limiter = new RateLimiter({
  maxRequests: 10,
  perMinutes: 1
});

async function safeAPICall(fn: () => Promise<any>) {
  await limiter.wait();
  return fn();
}

// Usage
const content = await safeAPICall(() =>
  generateContent({ topic: 'AI trends' })
);
```

### Error Handling

```typescript
import { PipelineError } from '@/lib/errors';

try {
  await runFullPipeline({ keyword: 'marketing' });
} catch (error) {
  if (error instanceof PipelineError) {
    console.log('Step failed:', error.step);
    console.log('Retry attempt:', error.retryCount);
    
    // Retry logic
    if (error.retryable) {
      await error.retry();
    }
  }
}
```

### Video Rendering Issues

```typescript
// Increase timeout for large videos
process.env.REMOTION_TIMEOUT = '600000'; // 10 minutes

// Memory issues - reduce quality
const config = {
  quality: 'medium',
  scale: 0.5
};

// Check Remotion logs
import { Log } from '@remotion/renderer';
Log.setLevel('verbose');
```

### Research Data Quality

```typescript
import { validateResearch } from '@/lib/research/validator';

const research = await conductResearch('keyword');

const validation = validateResearch(research, {
  minSources: 3,
  minDataPoints: 10,
  requireStats: true
});

if (!validation.valid) {
  console.log('Research quality issues:', validation.issues);
  // Fetch more data or adjust sources
}
```

## Performance Optimization

```typescript
// Cache research results
import { CacheManager } from '@/lib/cache';

const cache = new CacheManager({ ttl: 3600 }); // 1 hour

async function getCachedResearch(keyword: string) {
  const cached = await cache.get(`research:${keyword}`);
  if (cached) return cached;

  const fresh = await conductResearch(keyword);
  await cache.set(`research:${keyword}`, fresh);
  return fresh;
}

// Parallel processing
async function generateMultipleFormats(topic: string, research: any) {
  const [toplist, pov, howTo] = await Promise.all([
    createArticle(topic, research, 'toplist'),
    createArticle(topic, research, 'pov'),
    createArticle(topic, research, 'how-to')
  ]);

  return { toplist, pov, howTo };
}
```

This skill provides comprehensive knowledge for automating content marketing workflows from research through video generation using modern AI technologies.
