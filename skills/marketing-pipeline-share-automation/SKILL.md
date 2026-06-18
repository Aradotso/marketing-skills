---
name: marketing-pipeline-share-automation
description: AI-powered content automation pipeline from research to video generation with Claude/OpenAI integration
triggers:
  - automate content creation with AI research and video generation
  - set up automated content pipeline with Claude and OpenAI
  - generate marketing content from research to video automatically
  - create AI-driven content workflow with Remotion video rendering
  - build automated content pipeline with web scraping and AI writing
  - integrate Claude AI for automated content generation
  - set up end-to-end content automation with video rendering
  - automate content research, writing, and video creation pipeline
---

# Marketing Pipeline Share - AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

Marketing Pipeline Share is an automated content production pipeline that transforms keywords into complete marketing content including research, written articles, and rendered videos. The system integrates web scraping for real-time data, AI models (Claude 3, OpenAI) for content generation, and Remotion for video rendering.

**Key capabilities:**
- Automated news/trend research from TechCrunch, Twitter, LinkedIn
- Multi-format content generation (listicles, POV, case studies, how-to)
- Bilingual content (English/Vietnamese) with customizable tone
- Automatic video and infographic rendering
- Next.js-based web interface

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

```env
# AI Model APIs
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Web Scraping
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_url

# Application
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app router pages
│   ├── components/       # React components
│   ├── lib/              # Core utilities
│   │   ├── ai/          # AI integration modules
│   │   ├── scraper/     # Web scraping utilities
│   │   └── video/       # Remotion video rendering
│   ├── types/           # TypeScript type definitions
│   └── utils/           # Helper functions
├── public/              # Static assets
└── remotion/            # Remotion video templates
```

## Core Usage Patterns

### 1. Content Research & Scraping

```typescript
import { scrapeNewsArticles } from '@/lib/scraper/news-scraper';
import { analyzeContent } from '@/lib/ai/content-analyzer';

async function performResearch(keyword: string) {
  // Scrape recent articles from multiple sources
  const articles = await scrapeNewsArticles({
    keyword,
    sources: ['techcrunch', 'twitter', 'linkedin'],
    timeframe: '24h'
  });

  // Analyze and extract insights using AI
  const insights = await analyzeContent(articles, {
    model: 'claude-3-opus',
    extractMetrics: true,
    identifyTrends: true
  });

  return {
    rawArticles: articles,
    insights,
    dataPoints: insights.metrics
  };
}
```

### 2. AI Content Generation

```typescript
import { generateContent } from '@/lib/ai/content-generator';
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

async function createArticle(
  topic: string,
  format: 'toplist' | 'pov' | 'case-study' | 'how-to',
  language: 'en' | 'vi'
) {
  const prompt = `Generate a ${format} article about: ${topic}
Language: ${language}
Tone: Professional and engaging
Include: Data-backed insights, real examples, actionable takeaways`;

  const message = await anthropic.messages.create({
    model: 'claude-3-opus-20240229',
    max_tokens: 4000,
    messages: [{
      role: 'user',
      content: prompt
    }]
  });

  return {
    content: message.content[0].text,
    metadata: {
      format,
      language,
      wordCount: message.content[0].text.split(' ').length
    }
  };
}
```

### 3. Multi-Language Content Generation

```typescript
import { generateBilingualContent } from '@/lib/ai/bilingual-generator';

interface ContentRequest {
  keyword: string;
  format: string;
  tone: 'expert' | 'friendly' | 'humorous';
}

async function generateBilingual(request: ContentRequest) {
  const englishContent = await generateContent({
    ...request,
    language: 'en',
    model: 'gpt-4'
  });

  const vietnameseContent = await generateContent({
    ...request,
    language: 'vi',
    model: 'gpt-4'
  });

  return {
    en: englishContent,
    vi: vietnameseContent,
    metadata: {
      format: request.format,
      tone: request.tone,
      generatedAt: new Date().toISOString()
    }
  };
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
  style: 'reels' | 'tiktok' | 'shorts';
}

async function generateVideo(config: VideoConfig) {
  // Define video dimensions based on platform
  const dimensions = {
    reels: { width: 1080, height: 1920 },
    tiktok: { width: 1080, height: 1920 },
    shorts: { width: 1080, height: 1920 }
  };

  const bundleLocation = await bundle({
    entryPoint: path.join(process.cwd(), 'remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title: config.title,
      slides: config.content,
      style: config.style
    },
  });

  const outputPath = path.join(
    process.cwd(),
    'public/videos',
    `${config.style}-${Date.now()}.mp4`
  );

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    ...dimensions[config.style]
  });

  return {
    videoPath: outputPath,
    duration: composition.durationInFrames / composition.fps,
    platform: config.style
  };
}
```

### 5. Complete Pipeline Orchestration

```typescript
import { Pipeline } from '@/lib/pipeline/orchestrator';

async function runContentPipeline(keyword: string) {
  const pipeline = new Pipeline({
    aiProvider: 'claude',
    outputFormats: ['article', 'video'],
    languages: ['en', 'vi']
  });

  // Step 1: Research
  const research = await pipeline.research(keyword);
  
  // Step 2: Generate content
  const content = await pipeline.generate({
    keyword,
    insights: research.insights,
    format: 'toplist',
    tone: 'expert'
  });

  // Step 3: Create video
  const video = await pipeline.renderVideo({
    title: content.title,
    content: content.keyPoints,
    style: 'reels'
  });

  // Step 4: Return complete package
  return {
    research: research.summary,
    article: {
      en: content.en,
      vi: content.vi
    },
    video: video.path,
    metadata: {
      keyword,
      createdAt: new Date(),
      pipeline: 'complete'
    }
  };
}
```

## API Routes

### Create Content API

```typescript
// app/api/content/create/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline/orchestrator';

export async function POST(request: NextRequest) {
  try {
    const { keyword, format, languages } = await request.json();

    if (!keyword) {
      return NextResponse.json(
        { error: 'Keyword is required' },
        { status: 400 }
      );
    }

    const result = await runContentPipeline(keyword);

    return NextResponse.json({
      success: true,
      data: result
    });
  } catch (error) {
    console.error('Content creation error:', error);
    return NextResponse.json(
      { error: 'Failed to create content' },
      { status: 500 }
    );
  }
}
```

### Research API

```typescript
// app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { performResearch } from '@/lib/scraper/research';

export async function GET(request: NextRequest) {
  const { searchParams } = new URL(request.url);
  const keyword = searchParams.get('keyword');
  const sources = searchParams.get('sources')?.split(',') || [];

  if (!keyword) {
    return NextResponse.json(
      { error: 'Keyword parameter required' },
      { status: 400 }
    );
  }

  const research = await performResearch(keyword);

  return NextResponse.json({
    keyword,
    articles: research.rawArticles,
    insights: research.insights,
    timestamp: new Date().toISOString()
  });
}
```

## Running the Application

```bash
# Development mode
npm run dev

# Production build
npm run build
npm run start

# Run Remotion preview (for video development)
npm run remotion:preview
```

Access the application at `http://localhost:3000`

## Common Workflows

### Workflow 1: Quick Content Generation

```typescript
import { quickGenerate } from '@/lib/quick-generate';

// Generate content in one function call
const result = await quickGenerate({
  keyword: 'AI Marketing Trends 2026',
  format: 'toplist',
  includeVideo: true,
  languages: ['en', 'vi']
});

console.log('Article:', result.article);
console.log('Video:', result.videoUrl);
```

### Workflow 2: Custom Research Pipeline

```typescript
import { CustomPipeline } from '@/lib/pipeline/custom';

const pipeline = new CustomPipeline({
  researchSources: ['techcrunch', 'x', 'linkedin'],
  aiModel: 'claude-3-opus',
  contentFormats: ['article', 'infographic', 'video']
});

await pipeline
  .research('SaaS Growth Strategies')
  .then(data => pipeline.generateContent(data))
  .then(content => pipeline.createVisuals(content))
  .then(result => pipeline.export(result));
```

### Workflow 3: Scheduled Content Generation

```typescript
import { scheduleContentGeneration } from '@/lib/scheduler';

// Schedule daily content generation
scheduleContentGeneration({
  keywords: ['AI trends', 'Marketing automation', 'Content strategy'],
  frequency: 'daily',
  time: '09:00',
  formats: ['article', 'video'],
  autoPublish: false
});
```

## Configuration Options

### AI Provider Configuration

```typescript
// lib/config/ai-config.ts
export const aiConfig = {
  claude: {
    model: 'claude-3-opus-20240229',
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

### Content Format Templates

```typescript
// lib/config/content-templates.ts
export const contentTemplates = {
  toplist: {
    structure: ['intro', 'items', 'conclusion'],
    minItems: 5,
    maxItems: 10,
    includeData: true
  },
  pov: {
    structure: ['hook', 'perspective', 'evidence', 'conclusion'],
    tone: 'opinionated',
    includePersonalExperience: true
  },
  caseStudy: {
    structure: ['challenge', 'solution', 'results', 'takeaways'],
    includeMetrics: true,
    visualize: true
  },
  howTo: {
    structure: ['overview', 'steps', 'tips', 'summary'],
    stepByStep: true,
    includeScreenshots: false
  }
};
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
```

### Error Handling

```typescript
import { PipelineError } from '@/lib/errors';

try {
  await runContentPipeline(keyword);
} catch (error) {
  if (error instanceof PipelineError) {
    console.error(`Pipeline failed at stage: ${error.stage}`);
    console.error(`Reason: ${error.message}`);
    // Retry or fallback logic
  }
}
```

### Video Rendering Issues

```typescript
// Check Remotion configuration
import { validateRemotionSetup } from '@/lib/video/validate';

const validation = await validateRemotionSetup();
if (!validation.isValid) {
  console.error('Remotion issues:', validation.errors);
  // Fix: Ensure ffmpeg is installed
  // Fix: Check bundle location
}
```

## Best Practices

1. **Always use environment variables** for API keys
2. **Implement rate limiting** for external API calls
3. **Cache research results** to avoid redundant scraping
4. **Use TypeScript types** for all pipeline data
5. **Monitor AI token usage** to control costs
6. **Validate content quality** before video rendering
7. **Implement retry logic** for network operations

## Advanced Features

### Custom AI Prompts

```typescript
import { PromptBuilder } from '@/lib/ai/prompt-builder';

const prompt = new PromptBuilder()
  .setRole('content marketing expert')
  .setTask('create engaging toplist')
  .setContext('SaaS industry, B2B audience')
  .setConstraints({ wordCount: 1500, tone: 'professional' })
  .build();
```

### Multi-Platform Video Export

```typescript
const platforms = ['reels', 'tiktok', 'shorts', 'linkedin'];

for (const platform of platforms) {
  await generateVideo({
    title: content.title,
    content: content.keyPoints,
    style: platform
  });
}
```
