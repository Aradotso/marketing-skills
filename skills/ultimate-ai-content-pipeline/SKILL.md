---
name: ultimate-ai-content-pipeline
description: AI-powered content automation pipeline for research, scripting, and video generation with Claude/OpenAI
triggers:
  - automate content creation with AI research and video generation
  - set up AI content pipeline with Claude and Remotion
  - generate automated blog posts and social videos
  - crawl news sources and create content automatically
  - build end-to-end content automation system
  - create AI-powered marketing content pipeline
  - automate content research and video rendering
  - use Ultimate AI Content Pipeline for marketing
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

Ultimate AI Content Pipeline is a complete content automation system that handles research, content generation, and video production. It automatically crawls news sources (TechCrunch, a16z, Twitter, LinkedIn), generates multi-format content using Claude/OpenAI, and renders videos using Remotion.

## What It Does

- **Auto-Research**: Crawls fresh news from major sources (24h window)
- **AI Content Generation**: Creates articles in multiple formats (toplist, POV, case study, how-to)
- **Multi-language**: Generates Vietnamese and English versions simultaneously
- **Video/Image Rendering**: Converts content to videos using Remotion for Reels/TikTok/Shorts
- **Flexible Architecture**: Built with Next.js, TypeScript, integrates OpenAI, Anthropic Claude, RapidAPI

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

## Environment Configuration

Create `.env.local` with the following variables:

```bash
# AI Provider Keys
OPENAI_API_KEY=your_key_here
ANTHROPIC_API_KEY=your_key_here

# Research APIs
RAPIDAPI_KEY=your_key_here

# Optional: Video rendering
REMOTION_LICENSE_KEY=your_key_here

# Database (if applicable)
DATABASE_URL=your_database_url

# Next.js
NEXT_PUBLIC_API_URL=http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI provider integrations
│   │   ├── crawler/     # News crawling logic
│   │   ├── generator/   # Content generation
│   │   └── video/       # Remotion video rendering
│   └── types/           # TypeScript definitions
├── public/              # Static assets
└── remotion/            # Remotion video templates
```

## Core API Usage

### 1. Research/Crawling Module

```typescript
import { NewsCrawler } from '@/lib/crawler';

// Initialize crawler
const crawler = new NewsCrawler({
  sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
  timeWindow: 24 // hours
});

// Fetch latest news
const news = await crawler.fetchLatestNews('AI startup funding');

// Extract insights
const insights = await crawler.analyzeContent(news);
```

### 2. AI Content Generation

```typescript
import { ContentGenerator } from '@/lib/generator';
import { Anthropic } from '@anthropic-ai/sdk';
import OpenAI from 'openai';

// Initialize AI clients
const claude = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY
});

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY
});

// Generate content
const generator = new ContentGenerator({
  provider: 'claude', // or 'openai'
  client: claude
});

const content = await generator.createArticle({
  keyword: 'AI marketing tools',
  format: 'toplist', // 'pov', 'case-study', 'how-to'
  tone: 'professional', // 'friendly', 'humorous'
  languages: ['en', 'vi'],
  research: insights
});

// Result structure
/*
{
  english: {
    title: string,
    content: string,
    metadata: {...}
  },
  vietnamese: {
    title: string,
    content: string,
    metadata: {...}
  }
}
*/
```

### 3. Video Generation with Remotion

```typescript
import { renderVideo } from '@/lib/video/renderer';
import { bundle } from '@remotion/bundler';
import { renderMedia } from '@remotion/renderer';

// Prepare video data from content
const videoData = {
  title: content.english.title,
  keyPoints: content.english.keyPoints,
  duration: 30, // seconds
  format: 'reels' // 'tiktok', 'shorts'
};

// Render video
const video = await renderVideo({
  composition: 'ContentVideo',
  props: videoData,
  outputPath: './output/video.mp4'
});
```

## Complete Content Pipeline Example

```typescript
import { ContentPipeline } from '@/lib/pipeline';

async function runContentPipeline(keyword: string) {
  const pipeline = new ContentPipeline({
    openaiKey: process.env.OPENAI_API_KEY,
    claudeKey: process.env.ANTHROPIC_API_KEY,
    rapidApiKey: process.env.RAPIDAPI_KEY
  });

  try {
    // Step 1: Research
    console.log('🔍 Starting research...');
    const research = await pipeline.research(keyword);
    
    // Step 2: Generate content
    console.log('✍️ Generating content...');
    const article = await pipeline.generateContent({
      keyword,
      research,
      format: 'toplist',
      languages: ['en', 'vi']
    });
    
    // Step 3: Create video
    console.log('🎬 Rendering video...');
    const video = await pipeline.renderVideo({
      content: article.english,
      platform: 'reels'
    });
    
    return {
      article,
      video,
      research
    };
    
  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}

// Usage
runContentPipeline('AI content automation')
  .then(result => {
    console.log('✅ Pipeline complete!');
    console.log('Articles:', result.article);
    console.log('Video:', result.video.path);
  });
```

## API Routes (Next.js)

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { ContentPipeline } from '@/lib/pipeline';

export async function POST(req: NextRequest) {
  const { keyword, format, languages } = await req.json();
  
  const pipeline = new ContentPipeline({
    openaiKey: process.env.OPENAI_API_KEY,
    claudeKey: process.env.ANTHROPIC_API_KEY
  });
  
  const result = await pipeline.run({
    keyword,
    format,
    languages
  });
  
  return NextResponse.json(result);
}
```

## Common Patterns

### Pattern 1: Batch Content Generation

```typescript
async function batchGenerate(keywords: string[]) {
  const pipeline = new ContentPipeline({...});
  
  const results = await Promise.allSettled(
    keywords.map(keyword => 
      pipeline.run({ keyword, format: 'toplist' })
    )
  );
  
  return results.map((result, index) => ({
    keyword: keywords[index],
    status: result.status,
    data: result.status === 'fulfilled' ? result.value : null,
    error: result.status === 'rejected' ? result.reason : null
  }));
}
```

### Pattern 2: Custom AI Prompts

```typescript
import { generatePrompt } from '@/lib/ai/prompts';

const customPrompt = generatePrompt({
  type: 'article',
  format: 'case-study',
  tone: 'professional',
  additionalContext: 'Focus on B2B SaaS companies',
  research: researchData
});

const response = await claude.messages.create({
  model: 'claude-3-5-sonnet-20241022',
  max_tokens: 4096,
  messages: [{
    role: 'user',
    content: customPrompt
  }]
});
```

### Pattern 3: Multi-Platform Video Export

```typescript
const platforms = ['reels', 'tiktok', 'shorts'];

const videos = await Promise.all(
  platforms.map(platform =>
    renderVideo({
      composition: 'ContentVideo',
      props: {
        ...videoData,
        aspectRatio: platform === 'shorts' ? '9:16' : '9:16'
      },
      outputPath: `./output/${platform}-video.mp4`
    })
  )
);
```

## Running the Application

```bash
# Development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Render videos (Remotion)
npm run remotion:render

# Type checking
npm run type-check
```

## Troubleshooting

### API Rate Limits

```typescript
import { RateLimiter } from '@/lib/utils/rate-limiter';

const limiter = new RateLimiter({
  maxRequests: 50,
  perMinutes: 1
});

await limiter.throttle(async () => {
  return await claude.messages.create({...});
});
```

### Handle Long Content Generation

```typescript
// Use streaming for long responses
const stream = await openai.chat.completions.create({
  model: 'gpt-4-turbo-preview',
  messages: [{...}],
  stream: true
});

for await (const chunk of stream) {
  const content = chunk.choices[0]?.delta?.content || '';
  process.stdout.write(content);
}
```

### Video Rendering Errors

```typescript
import { getCompositions } from '@remotion/renderer';

// Validate composition before rendering
const compositions = await getCompositions(bundled);
const target = compositions.find(c => c.id === 'ContentVideo');

if (!target) {
  throw new Error('Composition not found');
}

// Check props validity
if (target.width !== 1080 || target.height !== 1920) {
  console.warn('Unexpected dimensions');
}
```

### Memory Management for Large Batches

```typescript
import pLimit from 'p-limit';

const limit = pLimit(3); // Max 3 concurrent operations

const results = await Promise.all(
  keywords.map(keyword =>
    limit(() => pipeline.run({ keyword }))
  )
);
```

## Configuration Best Practices

```typescript
// config/pipeline.config.ts
export const pipelineConfig = {
  research: {
    sources: ['techcrunch', 'a16z', 'twitter'],
    maxArticles: 20,
    timeWindow: 24
  },
  ai: {
    provider: 'claude',
    model: 'claude-3-5-sonnet-20241022',
    maxTokens: 4096,
    temperature: 0.7
  },
  video: {
    fps: 30,
    codec: 'h264',
    quality: 'high'
  }
};
```
