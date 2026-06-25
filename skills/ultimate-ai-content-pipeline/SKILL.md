---
name: ultimate-ai-content-pipeline
description: Automated content creation pipeline from research to video generation using Claude AI, OpenAI, and Remotion
triggers:
  - how do I generate automated content with AI
  - create content pipeline with research and video
  - automate content creation from research to video
  - build AI content generation system
  - set up automated marketing content pipeline
  - generate videos from written content automatically
  - crawl news and create content automatically
  - build content automation with Claude and OpenAI
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection

A complete automated content creation system that handles research, script writing, and video generation. The pipeline crawls fresh news from sources like TechCrunch, a16z, Twitter/X, and LinkedIn, then uses Claude/OpenAI to generate content in multiple formats (toplist, POV, case study, how-to) and finally renders videos using Remotion.

## What It Does

- **Auto-Research**: Crawls and analyzes real-time data from major news sources within 24 hours
- **Multi-Format Content**: Generates content in various formats with customizable tone and language (English/Vietnamese)
- **Video Generation**: Automatically renders infographics and short videos from written content using Remotion
- **Multi-Platform**: Exports videos optimized for Reels, TikTok, Shorts

## Installation

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
yarn install

# Set up environment variables
cp .env.example .env
```

## Environment Configuration

Create a `.env` file with the following variables:

```bash
# AI Provider Keys
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# News Sources (if using specific APIs)
TECHCRUNCH_API_KEY=your_key
TWITTER_BEARER_TOKEN=your_token

# Application Config
NEXT_PUBLIC_API_URL=http://localhost:3000/api
DATABASE_URL=your_database_url

# Remotion Config
REMOTION_AWS_ACCESS_KEY_ID=your_aws_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret
```

## Key API Endpoints

### Research & Crawl

```typescript
// API route: /api/research
import { NextRequest, NextResponse } from 'next/server';

export async function POST(req: NextRequest) {
  const { keyword, sources } = await req.json();
  
  // Crawl news from specified sources
  const results = await crawlNews({
    keyword,
    sources: sources || ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeframe: '24h'
  });
  
  return NextResponse.json({ data: results });
}
```

### Content Generation

```typescript
// API route: /api/generate-content
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

export async function POST(req: NextRequest) {
  const { researchData, format, tone, language } = await req.json();
  
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [{
      role: 'user',
      content: `Generate ${format} content in ${language} with ${tone} tone based on: ${JSON.stringify(researchData)}`
    }]
  });
  
  return NextResponse.json({ content: message.content });
}
```

### Video Rendering

```typescript
// API route: /api/render-video
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';

export async function POST(req: NextRequest) {
  const { content, videoType } = await req.json();
  
  const bundleLocation = await bundle({
    entryPoint: './src/remotion/index.ts',
    webpackOverride: (config) => config,
  });
  
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: videoType, // 'reels', 'tiktok', 'shorts'
    inputProps: { content },
  });
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: `out/${videoType}-${Date.now()}.mp4`,
  });
  
  return NextResponse.json({ success: true });
}
```

## Core Usage Patterns

### 1. Complete Pipeline Execution

```typescript
import { ContentPipeline } from './lib/pipeline';

const pipeline = new ContentPipeline({
  aiProvider: 'claude', // or 'openai'
  language: 'vi', // or 'en'
  format: 'toplist', // 'pov', 'case-study', 'how-to'
});

async function generateContent(keyword: string) {
  // Step 1: Research
  const research = await pipeline.research({
    keyword,
    sources: ['techcrunch', 'a16z'],
    limit: 10
  });
  
  // Step 2: Generate content
  const content = await pipeline.generateContent({
    research,
    tone: 'professional',
    wordCount: 1500
  });
  
  // Step 3: Render video
  const video = await pipeline.renderVideo({
    content,
    platform: 'reels',
    duration: 60
  });
  
  return { content, video };
}
```

### 2. Research Module

```typescript
import { NewsResearcher } from './lib/research';

const researcher = new NewsResearcher({
  rapidApiKey: process.env.RAPIDAPI_KEY,
});

// Crawl TechCrunch
const techCrunchNews = await researcher.crawlTechCrunch({
  keyword: 'artificial intelligence',
  limit: 20,
  timeframe: '24h'
});

// Crawl Twitter/X
const tweets = await researcher.crawlTwitter({
  keyword: 'AI startups',
  count: 50,
  verified: true
});

// Analyze and extract insights
const insights = await researcher.analyzeData({
  sources: [techCrunchNews, tweets],
  extractMetrics: true,
  summarize: true
});
```

### 3. Content Generator

```typescript
import { ContentGenerator } from './lib/generator';

const generator = new ContentGenerator({
  provider: 'claude',
  apiKey: process.env.ANTHROPIC_API_KEY,
});

// Generate toplist
const toplist = await generator.generate({
  type: 'toplist',
  data: insights,
  config: {
    itemCount: 10,
    language: 'vi',
    tone: 'engaging',
    includeStats: true
  }
});

// Generate POV article
const povArticle = await generator.generate({
  type: 'pov',
  data: insights,
  config: {
    perspective: 'industry-expert',
    language: 'en',
    tone: 'authoritative',
    wordCount: 2000
  }
});

// Bilingual generation
const bilingual = await generator.generateBilingual({
  data: insights,
  type: 'case-study',
  tone: 'professional'
});
```

### 4. Video Renderer (Remotion)

```typescript
// src/remotion/ReelsComposition.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig } from 'remotion';

export const ReelsVideo: React.FC<{ content: string }> = ({ content }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  
  const opacity = Math.min(1, frame / (fps * 0.5));
  
  return (
    <AbsoluteFill style={{ backgroundColor: '#000' }}>
      <div style={{ 
        color: '#fff', 
        fontSize: 60, 
        opacity,
        padding: 40 
      }}>
        {content}
      </div>
    </AbsoluteFill>
  );
};
```

```typescript
// Render programmatically
import { VideoRenderer } from './lib/video-renderer';

const renderer = new VideoRenderer({
  awsAccessKey: process.env.REMOTION_AWS_ACCESS_KEY_ID,
  awsSecretKey: process.env.REMOTION_AWS_SECRET_ACCESS_KEY,
});

const videoUrl = await renderer.render({
  composition: 'ReelsVideo',
  inputProps: { content: generatedContent },
  codec: 'h264',
  dimensions: { width: 1080, height: 1920 }, // Reels/Stories format
  fps: 30,
  durationInFrames: 1800 // 60 seconds at 30fps
});
```

### 5. Scheduling & Automation

```typescript
import { ContentScheduler } from './lib/scheduler';

const scheduler = new ContentScheduler();

// Schedule content generation
scheduler.schedule({
  keyword: 'tech trends',
  frequency: 'daily',
  time: '09:00',
  platforms: ['reels', 'tiktok'],
  autoPost: true
});

// Batch processing
const batch = await scheduler.processBatch({
  keywords: ['AI', 'Web3', 'SaaS', 'Startup'],
  format: 'toplist',
  generateVideos: true
});
```

## Configuration Files

### Pipeline Config (`pipeline.config.ts`)

```typescript
export const pipelineConfig = {
  research: {
    sources: {
      techcrunch: {
        enabled: true,
        weight: 0.3
      },
      a16z: {
        enabled: true,
        weight: 0.2
      },
      twitter: {
        enabled: true,
        weight: 0.3
      },
      linkedin: {
        enabled: true,
        weight: 0.2
      }
    },
    timeframe: '24h',
    maxResults: 50
  },
  content: {
    defaultLanguage: 'vi',
    supportedFormats: ['toplist', 'pov', 'case-study', 'how-to'],
    tones: ['professional', 'casual', 'humorous', 'authoritative'],
    wordCount: {
      min: 800,
      max: 2500
    }
  },
  video: {
    platforms: {
      reels: { width: 1080, height: 1920, fps: 30 },
      tiktok: { width: 1080, height: 1920, fps: 30 },
      shorts: { width: 1080, height: 1920, fps: 30 }
    },
    defaultDuration: 60
  }
};
```

## CLI Commands

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Run research only
npm run research -- --keyword "AI trends" --sources techcrunch,a16z

# Generate content from existing research
npm run generate -- --input research.json --format toplist --lang vi

# Render video
npm run render -- --input content.json --platform reels

# Full pipeline
npm run pipeline -- --keyword "startup funding" --all
```

## Development Server

```bash
# Start Next.js app
npm run dev
# Visit http://localhost:3000

# Start Remotion studio (for video editing)
npm run remotion:studio
# Visit http://localhost:3001
```

## Common Workflows

### Workflow 1: Daily News Digest

```typescript
async function dailyDigest() {
  const pipeline = new ContentPipeline();
  
  const keywords = ['AI', 'startup', 'funding', 'tech'];
  
  for (const keyword of keywords) {
    const research = await pipeline.research({ keyword });
    const content = await pipeline.generateContent({
      research,
      format: 'toplist',
      language: 'vi'
    });
    
    await saveToDatabase(content);
  }
}
```

### Workflow 2: Multi-Platform Content

```typescript
async function multiPlatform(keyword: string) {
  const pipeline = new ContentPipeline();
  
  const research = await pipeline.research({ keyword });
  const content = await pipeline.generateContent({ research });
  
  // Generate for all platforms
  const platforms = ['reels', 'tiktok', 'shorts'];
  const videos = await Promise.all(
    platforms.map(platform => 
      pipeline.renderVideo({ content, platform })
    )
  );
  
  return { content, videos };
}
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement retry logic
import { retry } from './lib/utils';

const content = await retry(
  () => anthropic.messages.create({ /* ... */ }),
  { maxAttempts: 3, delayMs: 1000 }
);
```

### Remotion Memory Issues

```bash
# Increase Node memory
NODE_OPTIONS=--max-old-space-size=8192 npm run render
```

### Video Rendering Timeout

```typescript
// Increase timeout in render config
await renderMedia({
  composition,
  serveUrl: bundleLocation,
  timeoutInMilliseconds: 120000, // 2 minutes
});
```

### Research Data Quality

```typescript
// Filter and validate research data
const validatedData = research
  .filter(item => item.publishedDate > Date.now() - 86400000) // 24h
  .filter(item => item.content.length > 100)
  .slice(0, 20); // Limit to top 20
```

## Advanced Features

### Custom Content Templates

```typescript
const customTemplate = {
  introduction: 'Hook with trending data',
  body: 'List format with metrics',
  conclusion: 'Call to action',
  seo: {
    keywords: true,
    metaDescription: true
  }
};

const content = await generator.generate({
  type: 'custom',
  template: customTemplate,
  data: insights
});
```

### A/B Testing Content Variations

```typescript
const variations = await generator.generateVariations({
  data: insights,
  count: 3,
  differentiators: ['tone', 'structure', 'length']
});
```

This skill enables AI coding agents to help developers build and customize automated content generation pipelines using the Ultimate AI Content Pipeline system.
