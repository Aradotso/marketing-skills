---
name: marketing-pipeline-share-automation
description: AI-powered content automation pipeline for research, scriptwriting, video generation, and auto-publishing
triggers:
  - automate content creation with AI research and video generation
  - set up marketing pipeline for automated content publishing
  - create AI content workflow from research to video
  - build automated content pipeline with Claude and OpenAI
  - generate videos automatically from written content
  - crawl news sources and create content automatically
  - automate social media content with AI pipeline
  - set up remotion video rendering in content workflow
---

# Marketing Pipeline Share Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to help developers use **marketing-pipeline-share**, a comprehensive content automation system that handles the entire content lifecycle: from research/crawling news sources, to AI-powered scriptwriting (Claude/OpenAI), to automatic video generation (Remotion), and platform publishing.

## What It Does

Marketing Pipeline Share automates:

1. **Auto-Research**: Crawls real-time data from TechCrunch, a16z, Twitter/X, LinkedIn
2. **AI Content Generation**: Creates multi-format content (toplist, POV, case studies, how-to) in multiple languages
3. **Video Rendering**: Automatically generates infographics and short-form videos using Remotion
4. **Multi-Platform Publishing**: Optimized for Reels, TikTok, Shorts with proper aspect ratios

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
# AI Services
ANTHROPIC_API_KEY=your_claude_key_here
OPENAI_API_KEY=your_openai_key_here

# Research/Crawling APIs
RAPIDAPI_KEY=your_rapidapi_key_here
TWITTER_BEARER_TOKEN=your_twitter_token_here

# Video Rendering (Remotion)
REMOTION_LICENSE_KEY=your_remotion_license_here

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Optional: Publishing APIs
FACEBOOK_ACCESS_TOKEN=your_fb_token_here
TIKTOK_API_KEY=your_tiktok_key_here
```

## Key Commands

```bash
# Development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Render videos (Remotion)
npm run render

# Run specific content pipeline
npm run pipeline:research
npm run pipeline:generate
npm run pipeline:video
```

## Core API Usage Patterns

### 1. Research & Data Crawling

```typescript
import { ResearchService } from '@/services/research';

async function gatherResearch(keyword: string) {
  const researchService = new ResearchService({
    rapidApiKey: process.env.RAPIDAPI_KEY,
    twitterToken: process.env.TWITTER_BEARER_TOKEN,
  });

  // Crawl multiple sources
  const data = await researchService.aggregateData({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeRange: '24h',
    limit: 50,
  });

  return data;
}

// Usage
const insights = await gatherResearch('AI automation');
```

### 2. AI Content Generation with Claude/OpenAI

```typescript
import { ContentGenerator } from '@/services/content-generator';
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

async function generateContent(research: any, format: string) {
  const anthropic = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY,
  });

  const generator = new ContentGenerator({
    aiProvider: 'claude', // or 'openai'
    client: anthropic,
  });

  const content = await generator.create({
    format, // 'toplist', 'pov', 'case-study', 'how-to'
    research,
    language: 'vi', // 'en' or 'vi'
    tone: 'professional', // 'friendly', 'humorous', 'expert'
    targetAudience: 'marketers',
  });

  return content;
}

// With OpenAI
async function generateWithOpenAI(prompt: string) {
  const openai = new OpenAI({
    apiKey: process.env.OPENAI_API_KEY,
  });

  const completion = await openai.chat.completions.create({
    model: 'gpt-4',
    messages: [
      {
        role: 'system',
        content: 'You are an expert content marketer creating engaging posts.',
      },
      { role: 'user', content: prompt },
    ],
    temperature: 0.7,
  });

  return completion.choices[0].message.content;
}
```

### 3. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

async function generateVideo(content: any) {
  // Bundle Remotion composition
  const bundleLocation = await bundle({
    entryPoint: path.join(process.cwd(), 'src/remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  // Select composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo', // Your composition ID
    inputProps: {
      title: content.title,
      body: content.body,
      images: content.images,
      style: 'reels', // 'tiktok', 'shorts'
    },
  });

  // Render video
  const outputPath = path.join(process.cwd(), 'output', `video-${Date.now()}.mp4`);
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: composition.inputProps,
  });

  return outputPath;
}
```

### 4. Complete Pipeline Integration

```typescript
import { ContentPipeline } from '@/services/pipeline';

async function runFullPipeline(keyword: string) {
  const pipeline = new ContentPipeline({
    anthropicKey: process.env.ANTHROPIC_API_KEY,
    openaiKey: process.env.OPENAI_API_KEY,
    rapidApiKey: process.env.RAPIDAPI_KEY,
  });

  // Step 1: Research
  const research = await pipeline.research({ keyword, timeRange: '24h' });

  // Step 2: Generate content
  const content = await pipeline.generateContent({
    research,
    format: 'toplist',
    languages: ['en', 'vi'],
  });

  // Step 3: Create video
  const video = await pipeline.renderVideo({
    content,
    platform: 'reels', // 9:16 aspect ratio
  });

  // Step 4: Publish (optional)
  const published = await pipeline.publish({
    content,
    video,
    platforms: ['facebook', 'tiktok', 'youtube'],
  });

  return { content, video, published };
}

// Execute pipeline
runFullPipeline('marketing automation')
  .then((result) => console.log('Pipeline completed:', result))
  .catch((error) => console.error('Pipeline error:', error));
```

## Configuration Patterns

### Custom Content Formats

```typescript
// src/config/content-formats.ts
export const contentFormats = {
  toplist: {
    structure: 'numbered-list',
    minItems: 5,
    maxItems: 10,
    includeIntro: true,
    includeConclusion: true,
  },
  pov: {
    structure: 'opinion-piece',
    perspective: 'first-person',
    includeCounterArgument: true,
  },
  caseStudy: {
    structure: 'problem-solution-results',
    includeMetrics: true,
    includeQuotes: true,
  },
  howTo: {
    structure: 'step-by-step',
    includeImages: true,
    includeResources: true,
  },
};
```

### Research Source Configuration

```typescript
// src/config/research-sources.ts
export const researchSources = {
  techcrunch: {
    enabled: true,
    endpoint: 'https://api.techcrunch.com/v1',
    rateLimit: 100,
  },
  a16z: {
    enabled: true,
    rssUrl: 'https://a16z.com/feed/',
  },
  twitter: {
    enabled: true,
    hashtags: ['#marketing', '#AI', '#automation'],
    maxTweets: 50,
  },
  linkedin: {
    enabled: true,
    searchQuery: 'marketing automation',
  },
};
```

### Video Rendering Presets

```typescript
// src/config/video-presets.ts
export const videoPresets = {
  reels: {
    width: 1080,
    height: 1920,
    fps: 30,
    durationFrames: 900, // 30 seconds
  },
  tiktok: {
    width: 1080,
    height: 1920,
    fps: 30,
    durationFrames: 1800, // 60 seconds
  },
  youtube: {
    width: 1920,
    height: 1080,
    fps: 30,
    durationFrames: 3600, // 2 minutes
  },
};
```

## Common Patterns

### Pattern 1: Scheduled Content Pipeline

```typescript
import cron from 'node-cron';

// Run pipeline every day at 9 AM
cron.schedule('0 9 * * *', async () => {
  const keywords = ['AI trends', 'marketing automation', 'social media'];
  
  for (const keyword of keywords) {
    try {
      await runFullPipeline(keyword);
      console.log(`✓ Pipeline completed for: ${keyword}`);
    } catch (error) {
      console.error(`✗ Pipeline failed for ${keyword}:`, error);
    }
  }
});
```

### Pattern 2: Batch Content Generation

```typescript
async function batchGenerate(keywords: string[]) {
  const results = await Promise.allSettled(
    keywords.map(async (keyword) => {
      const research = await gatherResearch(keyword);
      const content = await generateContent(research, 'toplist');
      return { keyword, content };
    })
  );

  const successful = results
    .filter((r) => r.status === 'fulfilled')
    .map((r) => (r as PromiseFulfilledResult<any>).value);

  return successful;
}
```

### Pattern 3: Multi-Language Content Variants

```typescript
async function createMultiLanguageContent(keyword: string) {
  const research = await gatherResearch(keyword);
  
  const variants = await Promise.all([
    generateContent(research, 'toplist').then((content) => ({
      language: 'en',
      content,
    })),
    generateContent(research, 'toplist').then((content) => ({
      language: 'vi',
      content,
    })),
  ]);

  return variants;
}
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement retry with exponential backoff
async function withRetry<T>(
  fn: () => Promise<T>,
  maxRetries = 3
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error: any) {
      if (error.status === 429 && i < maxRetries - 1) {
        const delay = Math.pow(2, i) * 1000;
        console.log(`Rate limited, retrying in ${delay}ms...`);
        await new Promise((resolve) => setTimeout(resolve, delay));
      } else {
        throw error;
      }
    }
  }
  throw new Error('Max retries exceeded');
}

// Usage
const content = await withRetry(() =>
  generateContent(research, 'toplist')
);
```

### Remotion Rendering Issues

```typescript
// Check for missing dependencies
import { getCompositions } from '@remotion/renderer';

async function validateRemotionSetup() {
  try {
    const bundleLocation = await bundle({
      entryPoint: path.join(process.cwd(), 'src/remotion/index.ts'),
    });
    
    const compositions = await getCompositions(bundleLocation);
    console.log('Available compositions:', compositions.map((c) => c.id));
    
    return true;
  } catch (error) {
    console.error('Remotion setup error:', error);
    return false;
  }
}
```

### Memory Management for Large Batches

```typescript
// Process in chunks to avoid memory issues
async function processBatches<T, R>(
  items: T[],
  processor: (item: T) => Promise<R>,
  batchSize = 5
): Promise<R[]> {
  const results: R[] = [];
  
  for (let i = 0; i < items.length; i += batchSize) {
    const batch = items.slice(i, i + batchSize);
    const batchResults = await Promise.all(batch.map(processor));
    results.push(...batchResults);
    
    // Small delay between batches
    await new Promise((resolve) => setTimeout(resolve, 1000));
  }
  
  return results;
}
```

### Error Handling Best Practices

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

async function safePipeline(keyword: string) {
  try {
    const research = await gatherResearch(keyword);
    return research;
  } catch (error) {
    throw new PipelineError(
      'Research stage failed',
      'research',
      error as Error
    );
  }
}
```

## Advanced Usage

### Custom AI Prompt Templates

```typescript
const promptTemplates = {
  toplist: `
Create a compelling toplist article about {keyword}.
Research data: {research}
Format: Numbered list with 7-10 items
Include: Introduction, detailed items with examples, conclusion
Tone: {tone}
Language: {language}
`,
  caseStudy: `
Write a detailed case study about {keyword}.
Research data: {research}
Structure: Problem → Solution → Results
Include: Specific metrics, quotes, visual data points
Tone: {tone}
Language: {language}
`,
};

function fillTemplate(
  template: string,
  vars: Record<string, string>
): string {
  return template.replace(/{(\w+)}/g, (_, key) => vars[key] || '');
}
```

This skill provides comprehensive coverage of the marketing-pipeline-share automation system, enabling AI agents to help developers implement end-to-end content automation workflows.
