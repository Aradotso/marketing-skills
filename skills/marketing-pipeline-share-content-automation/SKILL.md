---
name: marketing-pipeline-share-content-automation
description: AI-powered content pipeline for research, scriptwriting, and automated video generation using Claude/OpenAI and Remotion
triggers:
  - automate content creation with AI research and video generation
  - build automated marketing content pipeline
  - generate content from keyword to video automatically
  - research and create social media content with AI
  - automate content workflow from research to posting
  - create AI-driven content pipeline with video rendering
  - set up automated content generation system
  - build content automation with Claude and Remotion
---

# Marketing Pipeline Share - Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This project is an end-to-end AI content automation pipeline that researches trending topics, generates multilingual content in various formats, and automatically renders videos. Built with Next.js, TypeScript, Claude/OpenAI, and Remotion.

## What It Does

The **Marketing Pipeline Share** system automates the entire content creation workflow:

1. **Auto-Research**: Crawls real-time data from sources like TechCrunch, a16z, X (Twitter), LinkedIn
2. **AI Content Generation**: Creates posts in multiple formats (Toplist, POV, Case Study, How-to) using Claude 3 or OpenAI
3. **Multilingual Support**: Generates content in both English and Vietnamese with customizable tone
4. **Video Rendering**: Automatically creates short-form videos and infographics using Remotion
5. **Platform Optimization**: Exports content optimized for Reels, TikTok, Shorts

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

# Set up environment variables
cp .env.example .env.local
```

### Required Environment Variables

```bash
# AI Services
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Optional: Database (if using)
DATABASE_URL=your_database_url

# Next.js
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── app/                    # Next.js app directory
│   ├── api/               # API routes
│   ├── components/        # React components
│   └── page.tsx           # Main page
├── lib/                   # Core utilities
│   ├── ai/               # AI service integrations
│   ├── research/         # Content research modules
│   └── video/            # Remotion video generation
├── remotion/             # Remotion video templates
└── types/                # TypeScript definitions
```

## Core API Usage

### 1. Research Module

```typescript
import { ResearchService } from '@/lib/research/research-service';

// Initialize research service
const researchService = new ResearchService({
  rapidApiKey: process.env.RAPIDAPI_KEY!,
  sources: ['techcrunch', 'twitter', 'linkedin']
});

// Fetch trending content
async function getTrendingTopics(keyword: string) {
  const results = await researchService.search({
    keyword,
    timeframe: '24h',
    limit: 10,
    language: 'en'
  });

  return results.map(item => ({
    title: item.title,
    url: item.url,
    summary: item.summary,
    publishedAt: item.publishedAt,
    sentiment: item.sentiment
  }));
}

// Example usage
const topics = await getTrendingTopics('AI marketing automation');
```

### 2. AI Content Generation

```typescript
import { ContentGenerator } from '@/lib/ai/content-generator';
import Anthropic from '@anthropic-ai/sdk';

// Initialize Claude client
const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY
});

// Generate content with Claude
async function generateContent(
  topic: string,
  format: 'toplist' | 'pov' | 'case-study' | 'how-to',
  language: 'en' | 'vi'
) {
  const generator = new ContentGenerator(anthropic);

  const content = await generator.create({
    topic,
    format,
    language,
    tone: 'professional', // or 'friendly', 'humorous'
    includeData: true,
    wordCount: 800
  });

  return {
    title: content.title,
    body: content.body,
    metadata: content.metadata,
    suggestions: content.hashtags
  };
}

// Example: Generate a toplist
const post = await generateContent(
  'Top AI Marketing Tools 2026',
  'toplist',
  'en'
);
```

### 3. OpenAI Alternative

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY
});

async function generateWithGPT(prompt: string, research: any[]) {
  const systemPrompt = `You are an expert content creator. 
Use the following research data to create engaging content:
${JSON.stringify(research, null, 2)}`;

  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      { role: 'system', content: systemPrompt },
      { role: 'user', content: prompt }
    ],
    temperature: 0.7,
    max_tokens: 2000
  });

  return completion.choices[0].message.content;
}
```

### 4. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import { VideoTemplate } from '@/remotion/VideoTemplate';

async function renderContentVideo(
  content: {
    title: string;
    points: string[];
    branding: { logo: string; colors: string[] };
  },
  outputPath: string
) {
  // Bundle the Remotion project
  const bundleLocation = await bundle({
    entryPoint: './remotion/index.ts',
    webpackOverride: (config) => config
  });

  // Get composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: content
  });

  // Render video
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: content
  });

  console.log(`Video rendered to ${outputPath}`);
}

// Example: Render a TikTok-format video
await renderContentVideo(
  {
    title: 'Top 5 AI Marketing Tools',
    points: [
      'Claude for content generation',
      'Remotion for video automation',
      'RapidAPI for data research'
    ],
    branding: {
      logo: '/assets/logo.png',
      colors: ['#FF6B6B', '#4ECDC4']
    }
  },
  './output/video-tiktok.mp4'
);
```

### 5. Complete Pipeline Example

```typescript
import { ContentPipeline } from '@/lib/pipeline';

// Initialize the full pipeline
const pipeline = new ContentPipeline({
  aiProvider: 'claude', // or 'openai'
  researchEnabled: true,
  videoEnabled: true
});

async function createContentFromKeyword(keyword: string) {
  try {
    // Step 1: Research
    console.log('🔍 Researching...');
    const research = await pipeline.research(keyword);

    // Step 2: Generate content
    console.log('✍️ Generating content...');
    const content = await pipeline.generate({
      keyword,
      research,
      formats: ['toplist', 'pov'],
      languages: ['en', 'vi']
    });

    // Step 3: Render videos
    console.log('🎬 Rendering videos...');
    const videos = await pipeline.renderVideos(content, {
      platforms: ['tiktok', 'reels', 'shorts'],
      aspectRatios: ['9:16', '1:1']
    });

    // Step 4: Export results
    return {
      posts: content.posts,
      videos: videos.map(v => v.path),
      metadata: {
        keyword,
        generatedAt: new Date(),
        sources: research.sources
      }
    };
  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}

// Run the pipeline
const result = await createContentFromKeyword('AI content automation');
console.log('✅ Content created:', result);
```

## API Routes

### POST /api/research

```typescript
// app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { ResearchService } from '@/lib/research/research-service';

export async function POST(request: NextRequest) {
  const { keyword, timeframe, sources } = await request.json();

  const service = new ResearchService({
    rapidApiKey: process.env.RAPIDAPI_KEY!,
    sources
  });

  const results = await service.search({ keyword, timeframe });

  return NextResponse.json({ results });
}
```

### POST /api/generate

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import Anthropic from '@anthropic-ai/sdk';

export async function POST(request: NextRequest) {
  const { topic, format, language, research } = await request.json();

  const anthropic = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY!
  });

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 2048,
    messages: [{
      role: 'user',
      content: `Create a ${format} post about "${topic}" in ${language}.
Research data: ${JSON.stringify(research)}`
    }]
  });

  return NextResponse.json({
    content: message.content[0].text
  });
}
```

### POST /api/render-video

```typescript
// app/api/render-video/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { renderContentVideo } from '@/lib/video/renderer';

export async function POST(request: NextRequest) {
  const { content, platform } = await request.json();

  const videoPath = await renderContentVideo(content, {
    platform,
    quality: 'high'
  });

  return NextResponse.json({ videoPath });
}
```

## Configuration

### Customizing Content Formats

```typescript
// lib/config/content-formats.ts
export const contentFormats = {
  toplist: {
    structure: 'numbered-list',
    minItems: 5,
    maxItems: 10,
    includeIntro: true,
    includeConclusion: true
  },
  pov: {
    structure: 'opinion-piece',
    personalTone: true,
    includeEvidence: true
  },
  caseStudy: {
    structure: 'problem-solution',
    includeSections: ['background', 'challenge', 'solution', 'results'],
    requireData: true
  },
  howTo: {
    structure: 'step-by-step',
    includeVisuals: true,
    difficultyLevel: 'beginner'
  }
};
```

### Video Templates Configuration

```typescript
// remotion/config.ts
export const videoConfig = {
  tiktok: {
    width: 1080,
    height: 1920,
    fps: 30,
    durationInFrames: 900 // 30 seconds
  },
  reels: {
    width: 1080,
    height: 1920,
    fps: 30,
    durationInFrames: 900
  },
  shorts: {
    width: 1080,
    height: 1920,
    fps: 30,
    durationInFrames: 1800 // 60 seconds
  }
};
```

## Common Patterns

### Batch Content Generation

```typescript
async function batchGenerateContent(keywords: string[]) {
  const results = await Promise.all(
    keywords.map(async (keyword) => {
      const research = await pipeline.research(keyword);
      const content = await pipeline.generate({
        keyword,
        research,
        formats: ['toplist'],
        languages: ['en', 'vi']
      });
      return { keyword, content };
    })
  );

  return results;
}
```

### Scheduled Content Creation

```typescript
import cron from 'node-cron';

// Run daily at 9 AM
cron.schedule('0 9 * * *', async () => {
  const trendingKeywords = await fetchTrendingKeywords();
  const content = await batchGenerateContent(trendingKeywords);
  await saveToDatabase(content);
  console.log('Daily content generated');
});
```

### Content Validation

```typescript
function validateContent(content: string): boolean {
  const checks = {
    minLength: content.length >= 500,
    hasTitle: /^#/.test(content),
    hasHashtags: /#\w+/.test(content),
    noPlaceholders: !content.includes('[INSERT]')
  };

  return Object.values(checks).every(Boolean);
}
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement retry logic with exponential backoff
async function retryWithBackoff<T>(
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
        await new Promise(resolve => setTimeout(resolve, delay));
      } else {
        throw error;
      }
    }
  }
  throw new Error('Max retries reached');
}
```

### Video Rendering Issues

```typescript
// Check Remotion dependencies
// Ensure ffmpeg is installed: brew install ffmpeg (macOS)
// For Linux: sudo apt-get install ffmpeg

// Handle rendering errors
try {
  await renderMedia(config);
} catch (error) {
  if (error.message.includes('ffmpeg')) {
    console.error('ffmpeg not found. Install with: brew install ffmpeg');
  }
  throw error;
}
```

### Memory Issues with Large Batches

```typescript
// Process in chunks
async function processBatch<T>(
  items: T[],
  processor: (item: T) => Promise<any>,
  chunkSize = 5
) {
  const chunks = [];
  for (let i = 0; i < items.length; i += chunkSize) {
    chunks.push(items.slice(i, i + chunkSize));
  }

  const results = [];
  for (const chunk of chunks) {
    const chunkResults = await Promise.all(chunk.map(processor));
    results.push(...chunkResults);
    // Give GC time to clean up
    await new Promise(resolve => setTimeout(resolve, 1000));
  }

  return results;
}
```

## Development

```bash
# Run development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Type checking
npm run type-check

# Linting
npm run lint
```

## Key Dependencies

- `next` - React framework
- `@anthropic-ai/sdk` - Claude AI integration
- `openai` - OpenAI GPT integration
- `@remotion/bundler`, `@remotion/renderer` - Video generation
- `typescript` - Type safety
