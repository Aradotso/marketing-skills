---
name: pennydinh-marketing-pipeline-automation
description: Automated AI content pipeline for research, scriptwriting, video generation, and multi-platform publishing using Claude, OpenAI, and Remotion
triggers:
  - how do I automate content creation with AI research
  - set up marketing pipeline with video generation
  - create automated content from research to video
  - use Claude and OpenAI for content automation
  - build AI content pipeline with Remotion
  - automate social media content with AI
  - generate videos from text content automatically
  - set up multi-language content automation
---

# Ultimate AI Content Pipeline Skill

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the Ultimate AI Content Pipeline, a comprehensive TypeScript-based system that automates the entire content creation workflow: from real-time research and script generation to video rendering and multi-platform publishing.

## What This Project Does

The Ultimate AI Content Pipeline is an end-to-end content automation system that:

- **Auto-crawls research data** from sources like TechCrunch, a16z, Twitter/X, and LinkedIn
- **Generates multi-format content** using Claude 3 and OpenAI (toplist, POV, case studies, how-to guides)
- **Supports multi-language output** (English and Vietnamese simultaneously)
- **Renders videos and infographics** automatically using Remotion
- **Optimizes for multiple platforms** (Reels, TikTok, Shorts)
- **Provides a Next.js interface** for managing the entire pipeline

## Installation

### Prerequisites

```bash
# Node.js 18+ and npm/yarn required
node --version  # Should be v18 or higher
```

### Setup Steps

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
yarn install

# Copy environment variables template
cp .env.example .env.local
```

### Environment Configuration

Create a `.env.local` file with the following variables:

```bash
# AI API Keys
OPENAI_API_KEY=your_openai_key_here
ANTHROPIC_API_KEY=your_claude_key_here
RAPIDAPI_KEY=your_rapidapi_key_here

# Application Settings
NEXT_PUBLIC_APP_URL=http://localhost:3000
NODE_ENV=development

# Remotion Configuration
REMOTION_COMPOSITION_ID=MainVideo
REMOTION_CODEC=h264
REMOTION_FPS=30

# Database (if applicable)
DATABASE_URL=your_database_url_here

# Optional: Social Media API Keys
FACEBOOK_PAGE_ACCESS_TOKEN=your_token_here
LINKEDIN_ACCESS_TOKEN=your_token_here
```

### Running the Development Server

```bash
npm run dev
# or
yarn dev
```

Access the application at `http://localhost:3000`

## Key Components & API

### 1. Research Module (Auto-Scan)

The research module crawls and analyzes recent content from multiple sources.

```typescript
// src/lib/research/crawler.ts
import { ResearchCrawler } from './research/crawler';

interface CrawlerConfig {
  sources: string[];
  timeRange: '24h' | '7d' | '30d';
  keywords: string[];
  maxResults?: number;
}

// Initialize crawler
const crawler = new ResearchCrawler({
  sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
  timeRange: '24h',
  keywords: ['AI', 'marketing', 'automation'],
  maxResults: 50
});

// Execute research
const researchData = await crawler.scan();

// Process and extract insights
const insights = await crawler.extractInsights(researchData);

console.log(insights);
// Output: { trends, keyPoints, statistics, sources }
```

### 2. Content Generation with AI

Generate content in multiple formats using Claude or OpenAI.

```typescript
// src/lib/ai/content-generator.ts
import { ContentGenerator } from './ai/content-generator';

interface ContentConfig {
  provider: 'claude' | 'openai';
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi' | 'both';
  tone: 'expert' | 'friendly' | 'humorous';
  research: ResearchData;
}

const generator = new ContentGenerator({
  provider: 'claude',
  format: 'toplist',
  language: 'both',
  tone: 'expert',
  research: researchData
});

// Generate content
const content = await generator.generate();

console.log(content);
// Output: { 
//   en: { title, body, metadata },
//   vi: { title, body, metadata }
// }
```

### 3. Video Rendering with Remotion

Automatically render videos from generated content.

```typescript
// src/lib/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  content: GeneratedContent;
  format: 'reels' | 'tiktok' | 'shorts';
  duration?: number;
}

async function renderVideo(config: VideoConfig) {
  const compositionId = 'ContentVideo';
  
  // Bundle the Remotion project
  const bundled = await bundle({
    entryPoint: path.resolve('./src/remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  // Get composition
  const composition = await selectComposition({
    serveUrl: bundled,
    id: compositionId,
    inputProps: {
      title: config.content.title,
      body: config.content.body,
      format: config.format,
    },
  });

  // Render video
  const outputPath = path.join(
    process.cwd(),
    'out',
    `video-${Date.now()}.mp4`
  );

  await renderMedia({
    composition,
    serveUrl: bundled,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: composition.defaultProps,
  });

  return outputPath;
}

// Usage
const videoPath = await renderVideo({
  content: generatedContent,
  format: 'reels',
  duration: 30
});
```

### 4. Complete Pipeline Workflow

Orchestrate the entire content creation pipeline.

```typescript
// src/lib/pipeline/orchestrator.ts
import { Pipeline } from './pipeline/orchestrator';

interface PipelineConfig {
  keyword: string;
  contentFormat: 'toplist' | 'pov' | 'case-study' | 'how-to';
  languages: ('en' | 'vi')[];
  generateVideo: boolean;
  videoFormat?: 'reels' | 'tiktok' | 'shorts';
  autoPublish?: boolean;
}

async function runPipeline(config: PipelineConfig) {
  const pipeline = new Pipeline();

  // Step 1: Research
  console.log('🔍 Starting research phase...');
  const research = await pipeline.research({
    keyword: config.keyword,
    timeRange: '24h',
    sources: ['techcrunch', 'a16z', 'twitter']
  });

  // Step 2: Generate Content
  console.log('✍️ Generating content...');
  const content = await pipeline.generateContent({
    research,
    format: config.contentFormat,
    languages: config.languages,
    tone: 'expert'
  });

  // Step 3: Render Video (if enabled)
  let videoPath = null;
  if (config.generateVideo) {
    console.log('🎬 Rendering video...');
    videoPath = await pipeline.renderVideo({
      content: content.en,
      format: config.videoFormat || 'reels'
    });
  }

  // Step 4: Publish (if enabled)
  if (config.autoPublish) {
    console.log('📤 Publishing content...');
    await pipeline.publish({
      content,
      videoPath,
      platforms: ['facebook', 'linkedin']
    });
  }

  return {
    research,
    content,
    videoPath,
    status: 'completed'
  };
}

// Execute pipeline
const result = await runPipeline({
  keyword: 'AI marketing automation',
  contentFormat: 'toplist',
  languages: ['en', 'vi'],
  generateVideo: true,
  videoFormat: 'reels',
  autoPublish: false
});
```

## API Routes (Next.js)

### Trigger Pipeline via API

```typescript
// pages/api/pipeline/run.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { runPipeline } from '@/lib/pipeline/orchestrator';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  try {
    const { keyword, format, languages, generateVideo } = req.body;

    const result = await runPipeline({
      keyword,
      contentFormat: format,
      languages: languages || ['en'],
      generateVideo: generateVideo ?? true,
      autoPublish: false
    });

    res.status(200).json(result);
  } catch (error) {
    console.error('Pipeline error:', error);
    res.status(500).json({ 
      error: 'Pipeline execution failed',
      message: error.message 
    });
  }
}
```

### Usage with curl

```bash
curl -X POST http://localhost:3000/api/pipeline/run \
  -H "Content-Type: application/json" \
  -d '{
    "keyword": "AI marketing tools",
    "format": "toplist",
    "languages": ["en", "vi"],
    "generateVideo": true
  }'
```

## Common Patterns

### Pattern 1: Batch Content Generation

```typescript
// Generate multiple content pieces in parallel
async function batchGenerate(keywords: string[]) {
  const results = await Promise.all(
    keywords.map(keyword => 
      runPipeline({
        keyword,
        contentFormat: 'toplist',
        languages: ['en'],
        generateVideo: false
      })
    )
  );

  return results;
}

const batch = await batchGenerate([
  'AI tools 2026',
  'Marketing automation',
  'Content creation AI'
]);
```

### Pattern 2: Custom Research Sources

```typescript
// Add custom research sources
import { ResearchCrawler } from './research/crawler';

const customCrawler = new ResearchCrawler({
  sources: ['techcrunch', 'a16z'],
  timeRange: '24h',
  keywords: ['AI'],
  customSources: [
    {
      name: 'CustomBlog',
      url: 'https://example.com/feed',
      type: 'rss',
      parser: (data) => {
        // Custom parsing logic
        return data.items.map(item => ({
          title: item.title,
          content: item.content,
          date: item.pubDate
        }));
      }
    }
  ]
});
```

### Pattern 3: Video Template Customization

```typescript
// src/remotion/compositions/CustomTemplate.tsx
import { AbsoluteFill, useCurrentFrame, interpolate } from 'remotion';

export const CustomTemplate: React.FC<{
  title: string;
  body: string;
}> = ({ title, body }) => {
  const frame = useCurrentFrame();
  
  const opacity = interpolate(frame, [0, 30], [0, 1], {
    extrapolateRight: 'clamp',
  });

  return (
    <AbsoluteFill
      style={{
        backgroundColor: '#000',
        justifyContent: 'center',
        alignItems: 'center',
        opacity,
      }}
    >
      <h1 style={{ color: 'white', fontSize: 60 }}>{title}</h1>
      <p style={{ color: 'white', fontSize: 30 }}>{body}</p>
    </AbsoluteFill>
  );
};
```

## Troubleshooting

### Issue: API Rate Limiting

```typescript
// Implement retry logic with exponential backoff
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

// Usage
const content = await retryWithBackoff(() => 
  generator.generate()
);
```

### Issue: Video Rendering Timeout

```typescript
// Adjust timeout settings
await renderMedia({
  composition,
  serveUrl: bundled,
  codec: 'h264',
  outputLocation: outputPath,
  timeoutInMilliseconds: 120000, // 2 minutes
  chromiumOptions: {
    headless: true,
    gl: 'swiftshader', // Use software rendering if GPU issues
  },
});
```

### Issue: Memory Issues with Large Content

```typescript
// Process content in chunks
async function processLargeContent(content: string[]) {
  const chunkSize = 10;
  const results = [];

  for (let i = 0; i < content.length; i += chunkSize) {
    const chunk = content.slice(i, i + chunkSize);
    const processed = await Promise.all(
      chunk.map(item => generator.generate(item))
    );
    results.push(...processed);
    
    // Clear memory between chunks
    if (global.gc) global.gc();
  }

  return results;
}
```

### Issue: Missing Environment Variables

```typescript
// Validate environment variables at startup
function validateEnv() {
  const required = [
    'OPENAI_API_KEY',
    'ANTHROPIC_API_KEY',
    'RAPIDAPI_KEY'
  ];

  const missing = required.filter(key => !process.env[key]);

  if (missing.length > 0) {
    throw new Error(
      `Missing required environment variables: ${missing.join(', ')}\n` +
      'Please check your .env.local file'
    );
  }
}

validateEnv();
```

## Building for Production

```bash
# Build the Next.js application
npm run build

# Start production server
npm run start

# Build Remotion video renderer
npm run remotion:build

# Render video in production
npm run remotion:render
```

## Testing the Pipeline

```typescript
// tests/pipeline.test.ts
import { runPipeline } from '@/lib/pipeline/orchestrator';

describe('Content Pipeline', () => {
  it('should generate content from keyword', async () => {
    const result = await runPipeline({
      keyword: 'test keyword',
      contentFormat: 'toplist',
      languages: ['en'],
      generateVideo: false,
      autoPublish: false
    });

    expect(result.content).toBeDefined();
    expect(result.content.en.title).toBeTruthy();
  }, 30000); // 30s timeout for AI generation
});
```

This skill provides comprehensive guidance for working with the Ultimate AI Content Pipeline, covering installation, configuration, key APIs, common patterns, and troubleshooting for automated content creation workflows.
