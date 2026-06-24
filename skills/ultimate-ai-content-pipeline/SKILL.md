---
name: ultimate-ai-content-pipeline
description: Automated content pipeline from research to video generation using AI (Claude/OpenAI) and Remotion
triggers:
  - "help me set up the AI content pipeline"
  - "how do I generate content with this marketing automation tool"
  - "create automated content from research to video"
  - "configure the content generation pipeline"
  - "use Claude and OpenAI for content automation"
  - "render videos automatically with Remotion"
  - "automate content research and scriptwriting"
  - "set up the marketing pipeline system"
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the Ultimate AI Content Pipeline, a TypeScript-based system that automates the entire content creation workflow from research scanning to video generation using Claude 3, OpenAI, and Remotion.

## What This Project Does

The Ultimate AI Content Pipeline is an end-to-end content automation system that:

1. **Auto-scans research** from sources like TechCrunch, a16z, Twitter/X, LinkedIn
2. **Generates scripts** in multiple formats (Toplist, POV, Case Study, How-to)
3. **Creates bilingual content** (English & Vietnamese) with customizable tone
4. **Renders videos and infographics** automatically using Remotion
5. **Optimizes for multi-platform** (Reels, TikTok, Shorts)

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
# AI API Keys
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_url

# Remotion Configuration
REMOTION_LICENSE_KEY=your_remotion_license

# Application Settings
NEXT_PUBLIC_APP_URL=http://localhost:3000
NODE_ENV=development
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app router pages
│   ├── components/       # React components
│   ├── lib/             
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── scraper/     # Content research scrapers
│   │   ├── video/       # Remotion video rendering
│   │   └── content/     # Content generation logic
│   ├── types/           # TypeScript type definitions
│   └── utils/           # Utility functions
├── public/              # Static assets
└── remotion/            # Remotion video templates
```

## Key API Usage Patterns

### 1. Content Research & Scraping

```typescript
import { researchTopic } from '@/lib/scraper/research';

interface ResearchResult {
  sources: Array<{
    title: string;
    url: string;
    content: string;
    publishedAt: Date;
  }>;
  insights: string[];
  keywords: string[];
}

async function gatherResearch(topic: string): Promise<ResearchResult> {
  const result = await researchTopic({
    query: topic,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeframe: '24h',
    maxResults: 10
  });
  
  return result;
}

// Usage
const research = await gatherResearch('AI automation tools');
console.log(research.insights);
```

### 2. AI Content Generation with Claude

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

interface ContentGenerationOptions {
  topic: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  research: ResearchResult;
}

async function generateContent(options: ContentGenerationOptions): Promise<string> {
  const prompt = `
You are a professional content writer. Based on the following research:

${JSON.stringify(options.research, null, 2)}

Create a ${options.format} article in ${options.language} with a ${options.tone} tone about: ${options.topic}

Include:
- Engaging headline
- Data-backed insights
- Recent statistics from the research
- Actionable takeaways
`;

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [
      { role: 'user', content: prompt }
    ],
  });

  return message.content[0].type === 'text' ? message.content[0].text : '';
}

// Usage
const content = await generateContent({
  topic: 'Top 5 AI Marketing Tools in 2024',
  format: 'toplist',
  language: 'en',
  tone: 'expert',
  research: researchData
});
```

### 3. OpenAI Integration (Alternative)

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateWithOpenAI(prompt: string): Promise<string> {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: 'You are an expert content marketer specializing in data-driven insights.'
      },
      {
        role: 'user',
        content: prompt
      }
    ],
    temperature: 0.7,
    max_tokens: 3000,
  });

  return completion.choices[0].message.content || '';
}
```

### 4. Bilingual Content Generation

```typescript
interface BilingualContent {
  en: string;
  vi: string;
}

async function generateBilingualContent(
  topic: string,
  research: ResearchResult
): Promise<BilingualContent> {
  const [englishContent, vietnameseContent] = await Promise.all([
    generateContent({
      topic,
      format: 'toplist',
      language: 'en',
      tone: 'expert',
      research
    }),
    generateContent({
      topic,
      format: 'toplist',
      language: 'vi',
      tone: 'friendly',
      research
    })
  ]);

  return {
    en: englishContent,
    vi: vietnameseContent
  };
}
```

### 5. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  title: string;
  content: string;
  highlights: string[];
  platform: 'reels' | 'tiktok' | 'shorts';
}

async function renderContentVideo(config: VideoConfig): Promise<string> {
  const composition = {
    reels: { width: 1080, height: 1920, fps: 30 },
    tiktok: { width: 1080, height: 1920, fps: 30 },
    shorts: { width: 1080, height: 1920, fps: 30 },
  }[config.platform];

  // Bundle Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  const inputProps = {
    title: config.title,
    content: config.content,
    highlights: config.highlights,
  };

  // Render video
  const outputPath = path.join('./output', `video-${Date.now()}.mp4`);
  
  await renderMedia({
    composition: {
      id: 'ContentVideo',
      ...composition,
      durationInFrames: 300, // 10 seconds at 30fps
    },
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps,
  });

  return outputPath;
}

// Usage
const videoPath = await renderContentVideo({
  title: 'Top 5 AI Tools',
  content: generatedContent,
  highlights: ['Tool 1', 'Tool 2', 'Tool 3'],
  platform: 'reels'
});
```

## Complete Pipeline Workflow

```typescript
import { researchTopic } from '@/lib/scraper/research';
import { generateContent } from '@/lib/ai/content-generator';
import { renderContentVideo } from '@/lib/video/renderer';

interface PipelineResult {
  research: ResearchResult;
  content: BilingualContent;
  videos: {
    reels: string;
    tiktok: string;
    shorts: string;
  };
}

async function runContentPipeline(topic: string): Promise<PipelineResult> {
  // Step 1: Research
  console.log('🔍 Researching topic...');
  const research = await researchTopic({
    query: topic,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeframe: '24h',
    maxResults: 10
  });

  // Step 2: Generate Content
  console.log('✍️ Generating content...');
  const content = await generateBilingualContent(topic, research);

  // Step 3: Extract highlights for video
  const highlights = research.insights.slice(0, 5);

  // Step 4: Render videos for all platforms
  console.log('🎬 Rendering videos...');
  const [reelsVideo, tiktokVideo, shortsVideo] = await Promise.all([
    renderContentVideo({
      title: topic,
      content: content.en,
      highlights,
      platform: 'reels'
    }),
    renderContentVideo({
      title: topic,
      content: content.en,
      highlights,
      platform: 'tiktok'
    }),
    renderContentVideo({
      title: topic,
      content: content.en,
      highlights,
      platform: 'shorts'
    })
  ]);

  return {
    research,
    content,
    videos: {
      reels: reelsVideo,
      tiktok: tiktokVideo,
      shorts: shortsVideo
    }
  };
}

// Execute the pipeline
const result = await runContentPipeline('AI Marketing Automation 2024');
console.log('✅ Pipeline complete!', result);
```

## Next.js API Routes

### Content Generation Endpoint

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline';

export async function POST(request: NextRequest) {
  try {
    const { topic, format, languages } = await request.json();

    if (!topic) {
      return NextResponse.json(
        { error: 'Topic is required' },
        { status: 400 }
      );
    }

    const result = await runContentPipeline(topic);

    return NextResponse.json({
      success: true,
      data: result
    });
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { error: 'Failed to generate content' },
      { status: 500 }
    );
  }
}
```

### Research Endpoint

```typescript
// src/app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { researchTopic } from '@/lib/scraper/research';

export async function POST(request: NextRequest) {
  try {
    const { query, sources, timeframe } = await request.json();

    const research = await researchTopic({
      query,
      sources: sources || ['techcrunch', 'a16z', 'twitter', 'linkedin'],
      timeframe: timeframe || '24h',
      maxResults: 10
    });

    return NextResponse.json({
      success: true,
      data: research
    });
  } catch (error) {
    console.error('Research error:', error);
    return NextResponse.json(
      { error: 'Failed to research topic' },
      { status: 500 }
    );
  }
}
```

## Common Patterns

### Rate Limiting for API Calls

```typescript
import pLimit from 'p-limit';

const limit = pLimit(3); // Max 3 concurrent requests

async function batchGenerateContent(topics: string[]) {
  const promises = topics.map(topic =>
    limit(() => runContentPipeline(topic))
  );
  
  return await Promise.all(promises);
}
```

### Caching Research Results

```typescript
import { Redis } from '@upstash/redis';

const redis = new Redis({
  url: process.env.UPSTASH_REDIS_URL,
  token: process.env.UPSTASH_REDIS_TOKEN,
});

async function getCachedResearch(topic: string): Promise<ResearchResult | null> {
  const cached = await redis.get(`research:${topic}`);
  return cached as ResearchResult | null;
}

async function cacheResearch(topic: string, data: ResearchResult) {
  await redis.set(`research:${topic}`, data, { ex: 3600 }); // 1 hour cache
}
```

### Error Handling

```typescript
async function safeContentGeneration(topic: string) {
  try {
    return await runContentPipeline(topic);
  } catch (error) {
    if (error instanceof Anthropic.APIError) {
      console.error('Claude API error:', error.status, error.message);
      // Fallback to OpenAI
      return await generateWithOpenAI(topic);
    }
    
    throw error;
  }
}
```

## Running the Development Server

```bash
# Start Next.js development server
npm run dev

# Build for production
npm run build

# Start production server
npm run start

# Render a specific video composition
npx remotion render src/index.ts ContentVideo output/video.mp4
```

## Troubleshooting

### API Key Issues

```typescript
// Validate API keys on startup
function validateEnvVars() {
  const required = [
    'OPENAI_API_KEY',
    'ANTHROPIC_API_KEY',
    'RAPIDAPI_KEY'
  ];
  
  const missing = required.filter(key => !process.env[key]);
  
  if (missing.length > 0) {
    throw new Error(`Missing required env vars: ${missing.join(', ')}`);
  }
}

validateEnvVars();
```

### Remotion Rendering Errors

```typescript
// Check if ffmpeg is installed
import { ensureFfmpeg } from '@remotion/renderer';

async function setupRemotionDependencies() {
  try {
    await ensureFfmpeg();
    console.log('✅ FFmpeg is ready');
  } catch (error) {
    console.error('❌ FFmpeg not found. Install with: brew install ffmpeg');
    throw error;
  }
}
```

### Memory Issues with Large Content

```typescript
// Stream large responses
async function streamContent(topic: string) {
  const stream = await anthropic.messages.stream({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [{ role: 'user', content: topic }],
  });

  let fullContent = '';
  
  for await (const chunk of stream) {
    if (chunk.type === 'content_block_delta' && 
        chunk.delta.type === 'text_delta') {
      fullContent += chunk.delta.text;
      // Process incrementally
    }
  }
  
  return fullContent;
}
```

This skill provides comprehensive guidance for AI coding agents to effectively use the Ultimate AI Content Pipeline for automated marketing content generation, from research to video production.
