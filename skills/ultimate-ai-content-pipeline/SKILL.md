---
name: ultimate-ai-content-pipeline
description: Automated content creation pipeline from research to video generation using Claude/OpenAI and Remotion
triggers:
  - how do I automate content creation with AI
  - generate blog posts and videos from keywords
  - set up AI content pipeline with research and video
  - use Claude and OpenAI for automated content generation
  - create automated marketing content workflow
  - build content pipeline with Remotion video rendering
  - automate research and content writing with AI
  - generate multilingual content with AI automation
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

Ultimate AI Content Pipeline is a comprehensive TypeScript-based content automation system that transforms keywords into fully-researched articles and videos. It automatically scrapes recent news from sources like TechCrunch, a16z, Twitter/X, and LinkedIn, generates content in multiple formats (toplist, POV, case study, how-to) using Claude 3 or OpenAI, and renders videos using Remotion.

## What It Does

- **Auto-Research**: Crawls and analyzes real-time data from major tech news sources
- **AI Content Generation**: Creates articles in multiple formats with Claude/OpenAI
- **Multilingual Support**: Generates content in English and Vietnamese simultaneously
- **Video Rendering**: Automatically creates infographics and short videos from content
- **Multi-Platform Optimization**: Exports videos optimized for Reels, TikTok, Shorts

## Installation

### Prerequisites

```bash
node >= 18.0.0
npm or yarn
```

### Clone and Setup

```bash
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share
npm install
# or
yarn install
```

### Environment Configuration

Create a `.env.local` file in the root directory:

```env
# AI Providers
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Content Sources
TWITTER_API_KEY=your_twitter_key
LINKEDIN_API_KEY=your_linkedin_key

# Database (if used)
DATABASE_URL=your_database_url

# Remotion Config
REMOTION_AWS_ACCESS_KEY_ID=your_aws_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret
```

### Run Development Server

```bash
npm run dev
# or
yarn dev
```

Open [http://localhost:3000](http://localhost:3000) to access the UI.

## Key Components

### 1. Research Module (Auto-Scan)

```typescript
import { ResearchAgent } from '@/lib/research/agent';

// Initialize research agent
const researcher = new ResearchAgent({
  sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
  timeframe: '24h'
});

// Scan for recent content
const insights = await researcher.scan({
  keyword: 'AI automation',
  depth: 'deep', // or 'shallow'
  language: 'en'
});

// Returns structured data
interface ResearchResult {
  articles: Array<{
    title: string;
    source: string;
    url: string;
    summary: string;
    publishedAt: Date;
  }>;
  insights: string[];
  trends: string[];
  dataPoints: Array<{
    metric: string;
    value: number;
    source: string;
  }>;
}
```

### 2. Content Generation

```typescript
import { ContentGenerator } from '@/lib/content/generator';
import { ClaudeProvider } from '@/lib/ai/claude';
import { OpenAIProvider } from '@/lib/ai/openai';

// Initialize with AI provider
const generator = new ContentGenerator({
  provider: new ClaudeProvider({
    apiKey: process.env.ANTHROPIC_API_KEY,
    model: 'claude-3-opus-20240229'
  })
  // or
  // provider: new OpenAIProvider({
  //   apiKey: process.env.OPENAI_API_KEY,
  //   model: 'gpt-4-turbo'
  // })
});

// Generate content
const article = await generator.create({
  keyword: 'AI automation trends 2026',
  format: 'toplist', // 'pov', 'case-study', 'how-to'
  tone: 'professional', // 'friendly', 'humorous'
  languages: ['en', 'vi'],
  researchData: insights,
  wordCount: 1500
});

interface GeneratedContent {
  title: string;
  content: string;
  language: string;
  metadata: {
    format: string;
    tone: string;
    wordCount: number;
  };
  sections: Array<{
    heading: string;
    body: string;
  }>;
}
```

### 3. Video Rendering with Remotion

```typescript
import { VideoRenderer } from '@/lib/video/renderer';
import { bundle } from '@remotion/bundler';
import { renderMedia } from '@remotion/renderer';

const renderer = new VideoRenderer({
  compositionId: 'ContentVideo',
  outputFormat: 'mp4',
  fps: 30
});

// Render video from content
const video = await renderer.render({
  content: article,
  style: 'infographic', // 'talking-head', 'slides'
  aspectRatio: '9:16', // '16:9', '1:1'
  duration: 60, // seconds
  platform: 'reels' // 'tiktok', 'shorts', 'general'
});

interface VideoOutput {
  url: string;
  duration: number;
  resolution: {
    width: number;
    height: number;
  };
  fileSize: number;
  format: string;
}
```

### 4. Complete Pipeline Example

```typescript
import { ContentPipeline } from '@/lib/pipeline';

// Initialize full pipeline
const pipeline = new ContentPipeline({
  research: {
    sources: ['techcrunch', 'a16z', 'twitter'],
    timeframe: '24h'
  },
  ai: {
    provider: 'claude',
    model: 'claude-3-opus-20240229'
  },
  video: {
    enabled: true,
    aspectRatio: '9:16'
  }
});

// Run complete automation
const result = await pipeline.execute({
  keyword: 'generative AI business trends',
  contentFormat: 'toplist',
  languages: ['en', 'vi'],
  includeVideo: true,
  autoPublish: false
});

interface PipelineResult {
  research: ResearchResult;
  articles: GeneratedContent[];
  videos: VideoOutput[];
  metadata: {
    executionTime: number;
    status: 'success' | 'partial' | 'failed';
    errors?: string[];
  };
}
```

## API Routes (Next.js)

### Generate Content

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { ContentPipeline } from '@/lib/pipeline';

export async function POST(request: NextRequest) {
  const { keyword, format, languages } = await request.json();
  
  const pipeline = new ContentPipeline({
    ai: { provider: 'claude' }
  });
  
  const result = await pipeline.execute({
    keyword,
    contentFormat: format,
    languages,
    includeVideo: false
  });
  
  return NextResponse.json(result);
}
```

### Research Endpoint

```typescript
// app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { ResearchAgent } from '@/lib/research/agent';

export async function POST(request: NextRequest) {
  const { keyword, sources, timeframe } = await request.json();
  
  const researcher = new ResearchAgent({ sources, timeframe });
  const insights = await researcher.scan({ keyword });
  
  return NextResponse.json(insights);
}
```

### Video Rendering Endpoint

```typescript
// app/api/video/render/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { VideoRenderer } from '@/lib/video/renderer';

export async function POST(request: NextRequest) {
  const { content, style, aspectRatio } = await request.json();
  
  const renderer = new VideoRenderer({
    compositionId: 'ContentVideo',
    outputFormat: 'mp4'
  });
  
  const video = await renderer.render({
    content,
    style,
    aspectRatio
  });
  
  return NextResponse.json({ videoUrl: video.url });
}
```

## Common Patterns

### Batch Content Generation

```typescript
import { ContentPipeline } from '@/lib/pipeline';

async function batchGenerate(keywords: string[]) {
  const pipeline = new ContentPipeline({
    ai: { provider: 'openai' }
  });
  
  const results = await Promise.all(
    keywords.map(keyword => 
      pipeline.execute({
        keyword,
        contentFormat: 'how-to',
        languages: ['en'],
        includeVideo: true
      })
    )
  );
  
  return results;
}

// Usage
const keywords = ['AI content creation', 'Marketing automation', 'Video SEO'];
const content = await batchGenerate(keywords);
```

### Custom AI Prompts

```typescript
import { ContentGenerator } from '@/lib/content/generator';

const generator = new ContentGenerator({
  provider: new ClaudeProvider({
    apiKey: process.env.ANTHROPIC_API_KEY
  })
});

// Override default prompt template
const customArticle = await generator.create({
  keyword: 'AI trends',
  format: 'custom',
  customPrompt: `
    Create a comprehensive article about {keyword}.
    Include:
    - 3 recent case studies
    - Data-backed insights from the past week
    - Actionable recommendations
    - Future predictions
    
    Tone: Professional but engaging
    Length: 2000 words
  `
});
```

### Multi-Language Content Strategy

```typescript
async function createMultilingualCampaign(topic: string) {
  const pipeline = new ContentPipeline({
    ai: { provider: 'claude' }
  });
  
  const formats = ['toplist', 'how-to', 'case-study'];
  const languages = ['en', 'vi'];
  
  const content = [];
  
  for (const format of formats) {
    const result = await pipeline.execute({
      keyword: topic,
      contentFormat: format,
      languages,
      includeVideo: true
    });
    content.push(result);
  }
  
  return content;
}
```

### Video Customization

```typescript
import { VideoRenderer } from '@/lib/video/renderer';

// Custom video composition
const customVideo = await renderer.render({
  content: article,
  style: 'custom',
  config: {
    backgroundColor: '#1a1a1a',
    textColor: '#ffffff',
    fontFamily: 'Inter',
    animations: {
      text: 'fade-in',
      transitions: 'slide'
    },
    branding: {
      logo: '/path/to/logo.png',
      watermark: true
    },
    music: {
      enabled: true,
      track: 'background-1.mp3',
      volume: 0.3
    }
  }
});
```

## Configuration

### AI Provider Settings

```typescript
// lib/config/ai.ts
export const aiConfig = {
  claude: {
    model: 'claude-3-opus-20240229',
    maxTokens: 4096,
    temperature: 0.7,
    topP: 0.9
  },
  openai: {
    model: 'gpt-4-turbo',
    maxTokens: 4096,
    temperature: 0.7,
    frequencyPenalty: 0.2,
    presencePenalty: 0.1
  }
};
```

### Research Source Configuration

```typescript
// lib/config/research.ts
export const researchConfig = {
  sources: {
    techcrunch: {
      enabled: true,
      rss: 'https://techcrunch.com/feed/',
      priority: 1
    },
    a16z: {
      enabled: true,
      rss: 'https://a16z.com/feed/',
      priority: 2
    },
    twitter: {
      enabled: true,
      accounts: ['@elonmusk', '@sama', '@karpathy'],
      priority: 3
    }
  },
  refreshInterval: 3600000 // 1 hour
};
```

### Remotion Video Settings

```typescript
// lib/config/video.ts
export const videoConfig = {
  compositions: {
    ContentVideo: {
      width: 1080,
      height: 1920,
      fps: 30,
      durationInFrames: 1800 // 60 seconds at 30fps
    }
  },
  rendering: {
    concurrency: 4,
    imageFormat: 'png',
    codec: 'h264',
    quality: 80
  }
};
```

## Troubleshooting

### API Rate Limits

```typescript
import { RateLimiter } from '@/lib/utils/rate-limiter';

const limiter = new RateLimiter({
  maxRequests: 50,
  windowMs: 60000 // 1 minute
});

await limiter.check('openai');
const content = await generator.create({ /* ... */ });
```

### Error Handling

```typescript
import { PipelineError } from '@/lib/errors';

try {
  const result = await pipeline.execute({ /* ... */ });
} catch (error) {
  if (error instanceof PipelineError) {
    console.error('Pipeline failed at stage:', error.stage);
    console.error('Error details:', error.details);
    
    // Retry specific stage
    if (error.stage === 'research') {
      // Fallback to cached research data
    } else if (error.stage === 'generation') {
      // Switch to alternative AI provider
    }
  }
}
```

### Memory Management for Large Videos

```typescript
// For large video rendering
const video = await renderer.render({
  content: article,
  chunks: true, // Render in chunks
  chunkSize: 10, // 10 seconds per chunk
  cleanup: true  // Clean temp files after render
});
```

### Debug Mode

```typescript
// Enable verbose logging
process.env.DEBUG = 'pipeline:*';

const pipeline = new ContentPipeline({
  debug: true,
  logLevel: 'verbose'
});

const result = await pipeline.execute({ /* ... */ });
// Outputs detailed logs for each stage
```

## CLI Commands (if applicable)

```bash
# Generate content from CLI
npm run generate -- --keyword "AI trends" --format toplist --lang en,vi

# Run research only
npm run research -- --sources techcrunch,a16z --timeframe 48h

# Render video from existing content
npm run render -- --input content.json --style infographic --ratio 9:16

# Build Remotion compositions
npm run remotion:build

# Preview video composition
npm run remotion:preview
```

This skill enables AI coding agents to effectively implement and customize the Ultimate AI Content Pipeline for automated content creation workflows.
