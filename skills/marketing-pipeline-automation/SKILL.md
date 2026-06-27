---
name: marketing-pipeline-automation
description: AI-powered content automation pipeline from research to video generation using Claude, OpenAI, and Remotion
triggers:
  - automate content creation with AI pipeline
  - set up marketing automation workflow
  - generate content from research to video
  - create automated content pipeline
  - use AI for content research and scripting
  - build automated marketing content system
  - integrate Claude and OpenAI for content generation
  - automate video generation from written content
---

# Marketing Pipeline Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

Marketing Pipeline Automation is a comprehensive TypeScript-based system that automates the entire content creation workflow: from research and data collection to script writing and video generation. It integrates Claude 3, OpenAI, and Remotion to create a complete content production pipeline.

**Key capabilities:**
- Auto-crawl news and trending topics from TechCrunch, a16z, Twitter/X, LinkedIn
- Generate content in multiple formats (listicles, POV, case studies, how-to)
- Support bilingual content (English & Vietnamese)
- Automatically render infographics and short-form videos
- Export optimized videos for Reels, TikTok, Shorts

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

Create a `.env.local` file in the project root:

```env
# AI Model APIs
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs
RAPID_API_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Optional: Video rendering settings
REMOTION_CONCURRENCY=4
REMOTION_QUALITY=80
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── research/    # Web scraping & data collection
│   │   ├── content/     # Content generation logic
│   │   └── video/       # Remotion video generation
│   ├── config/          # Configuration files
│   └── types/           # TypeScript type definitions
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Core Modules

### 1. Research & Data Collection

```typescript
import { ResearchService } from '@/lib/research/service';

// Initialize research service
const researcher = new ResearchService({
  apiKey: process.env.RAPID_API_KEY,
  sources: ['techcrunch', 'a16z', 'twitter', 'linkedin']
});

// Collect latest news on a topic
async function collectResearch(keyword: string) {
  const results = await researcher.search({
    query: keyword,
    timeframe: '24h',
    limit: 20,
    language: 'en'
  });

  // Extract insights
  const insights = await researcher.extractInsights(results);
  
  return {
    rawData: results,
    insights: insights,
    trends: insights.trends,
    statistics: insights.stats
  };
}

// Usage
const data = await collectResearch('AI marketing tools');
console.log(data.insights);
```

### 2. AI Content Generation

```typescript
import { ContentGenerator } from '@/lib/content/generator';
import { Anthropic } from '@anthropic-ai/sdk';

// Initialize AI client
const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY
});

const generator = new ContentGenerator({ client: anthropic });

// Generate content in different formats
async function generateContent(
  topic: string,
  format: 'toplist' | 'pov' | 'case-study' | 'how-to',
  language: 'en' | 'vi' = 'en'
) {
  const prompt = generator.buildPrompt({
    topic,
    format,
    language,
    tone: 'professional', // or 'friendly', 'humorous'
    includeStats: true
  });

  const response = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [
      {
        role: 'user',
        content: prompt
      }
    ]
  });

  return generator.parseResponse(response);
}

// Generate bilingual content
async function generateBilingualContent(topic: string) {
  const [englishContent, vietnameseContent] = await Promise.all([
    generateContent(topic, 'toplist', 'en'),
    generateContent(topic, 'toplist', 'vi')
  ]);

  return { en: englishContent, vi: vietnameseContent };
}
```

### 3. OpenAI Integration (Alternative)

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY
});

async function generateWithOpenAI(
  researchData: any,
  format: string
) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: 'You are an expert content marketer creating engaging, data-backed content.'
      },
      {
        role: 'user',
        content: `Create a ${format} article based on this research: ${JSON.stringify(researchData)}`
      }
    ],
    temperature: 0.7,
    max_tokens: 3000
  });

  return completion.choices[0].message.content;
}
```

### 4. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import { VideoTemplate } from '@/remotion/VideoTemplate';

interface VideoConfig {
  title: string;
  points: string[];
  style: 'minimal' | 'dynamic' | 'corporate';
  duration: number; // in frames (30fps)
}

async function generateVideo(content: any, config: VideoConfig) {
  // Bundle Remotion project
  const bundleLocation = await bundle({
    entryPoint: './remotion/index.ts',
    webpackOverride: (config) => config
  });

  // Get composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title: config.title,
      points: config.points,
      style: config.style
    }
  });

  // Render video
  const outputPath = `./output/video-${Date.now()}.mp4`;
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: {
      title: config.title,
      points: config.points,
      style: config.style
    },
    concurrency: parseInt(process.env.REMOTION_CONCURRENCY || '4'),
    quality: parseInt(process.env.REMOTION_QUALITY || '80')
  });

  return outputPath;
}

// Create different aspect ratios
async function generateMultiPlatformVideos(content: any) {
  const configs = [
    { name: 'reels', width: 1080, height: 1920 },  // 9:16
    { name: 'youtube', width: 1920, height: 1080 }, // 16:9
    { name: 'square', width: 1080, height: 1080 }   // 1:1
  ];

  const videos = await Promise.all(
    configs.map(config => 
      generateVideo(content, {
        ...config,
        title: content.title,
        points: content.keyPoints,
        style: 'dynamic',
        duration: 900 // 30 seconds at 30fps
      })
    )
  );

  return videos;
}
```

## Complete Pipeline Workflow

```typescript
import { PipelineOrchestrator } from '@/lib/pipeline/orchestrator';

// Full automation pipeline
async function runContentPipeline(keyword: string) {
  const pipeline = new PipelineOrchestrator({
    anthropicKey: process.env.ANTHROPIC_API_KEY,
    openaiKey: process.env.OPENAI_API_KEY,
    rapidApiKey: process.env.RAPID_API_KEY
  });

  try {
    // Step 1: Research
    console.log('🔍 Starting research...');
    const research = await pipeline.research(keyword);

    // Step 2: Generate content
    console.log('✍️ Generating content...');
    const content = await pipeline.generateContent({
      research,
      format: 'toplist',
      languages: ['en', 'vi'],
      tone: 'professional'
    });

    // Step 3: Create visuals
    console.log('🎨 Creating visuals...');
    const infographic = await pipeline.generateInfographic(content);

    // Step 4: Render videos
    console.log('🎬 Rendering videos...');
    const videos = await pipeline.renderVideos({
      content,
      platforms: ['reels', 'tiktok', 'youtube-shorts']
    });

    // Step 5: Save results
    const result = await pipeline.saveOutput({
      content,
      infographic,
      videos,
      metadata: {
        keyword,
        generatedAt: new Date(),
        sources: research.sources
      }
    });

    console.log('✅ Pipeline complete!', result);
    return result;

  } catch (error) {
    console.error('❌ Pipeline failed:', error);
    throw error;
  }
}

// Execute
runContentPipeline('AI automation trends 2024');
```

## API Routes (Next.js)

```typescript
// app/api/generate/route.ts
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
    return NextResponse.json(
      { error: error.message },
      { status: 500 }
    );
  }
}
```

## CLI Commands

```bash
# Development server
npm run dev

# Build for production
npm run build

# Start production server
npm run start

# Generate content via CLI
npm run generate -- --keyword "AI marketing" --format toplist

# Render video only
npm run render-video -- --input ./content/article.json

# Run full pipeline
npm run pipeline -- --keyword "marketing automation" --all
```

## Common Patterns

### Custom Content Templates

```typescript
import { ContentTemplate } from '@/lib/content/templates';

// Define custom template
const customTemplate: ContentTemplate = {
  name: 'product-launch',
  structure: {
    introduction: {
      hook: true,
      context: true,
      preview: true
    },
    body: {
      sections: [
        'problem-statement',
        'solution-overview',
        'key-features',
        'use-cases',
        'pricing'
      ]
    },
    conclusion: {
      summary: true,
      cta: true
    }
  },
  tone: 'professional',
  length: 'medium' // 800-1200 words
};

// Use template
const content = await generator.generate({
  template: customTemplate,
  data: researchData
});
```

### Batch Processing

```typescript
async function batchGenerateContent(keywords: string[]) {
  const results = [];
  
  for (const keyword of keywords) {
    try {
      const result = await runContentPipeline(keyword);
      results.push({ keyword, success: true, data: result });
      
      // Rate limiting
      await new Promise(resolve => setTimeout(resolve, 2000));
      
    } catch (error) {
      results.push({ keyword, success: false, error: error.message });
    }
  }

  return results;
}

// Process multiple topics
const keywords = [
  'AI content creation',
  'Marketing automation',
  'Video marketing trends'
];

const batchResults = await batchGenerateContent(keywords);
```

## Troubleshooting

### API Rate Limits

```typescript
import { RateLimiter } from '@/lib/utils/rate-limiter';

const limiter = new RateLimiter({
  maxRequests: 50,
  windowMs: 60000 // 1 minute
});

async function apiCallWithRateLimit(fn: Function) {
  await limiter.checkLimit();
  return await fn();
}
```

### Error Handling

```typescript
class PipelineError extends Error {
  constructor(
    message: string,
    public stage: string,
    public originalError?: Error
  ) {
    super(message);
    this.name = 'PipelineError';
  }
}

try {
  await runContentPipeline(keyword);
} catch (error) {
  if (error instanceof PipelineError) {
    console.error(`Error at stage: ${error.stage}`);
    console.error(`Message: ${error.message}`);
    // Retry logic or fallback
  }
}
```

### Video Rendering Issues

```typescript
// Check Remotion configuration
import { getCompositions } from '@remotion/renderer';

async function validateVideoSetup() {
  try {
    const compositions = await getCompositions('./remotion/index.ts');
    console.log('Available compositions:', compositions);
    return true;
  } catch (error) {
    console.error('Remotion setup error:', error);
    return false;
  }
}
```

### Memory Management for Large Batches

```typescript
async function processLargeDataset(keywords: string[]) {
  const chunkSize = 5;
  const chunks = [];
  
  for (let i = 0; i < keywords.length; i += chunkSize) {
    chunks.push(keywords.slice(i, i + chunkSize));
  }

  for (const chunk of chunks) {
    await batchGenerateContent(chunk);
    
    // Force garbage collection (if --expose-gc flag is set)
    if (global.gc) {
      global.gc();
    }
  }
}
```

## Best Practices

1. **Always validate research data** before content generation
2. **Use environment variables** for all API keys and secrets
3. **Implement retry logic** for external API calls
4. **Cache research results** to avoid redundant API calls
5. **Monitor token usage** to control AI API costs
6. **Test video templates** before batch rendering
7. **Store generated content** with proper metadata for tracking
