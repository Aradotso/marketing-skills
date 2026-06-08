---
name: ultimate-ai-content-pipeline
description: Automated content pipeline from research to scripting to video generation using Claude/OpenAI and Remotion
triggers:
  - how do I generate automated content with AI research
  - set up content pipeline with video generation
  - automate content creation from keyword to video
  - use Claude and OpenAI for content automation
  - create AI-powered marketing content pipeline
  - generate videos from content automatically
  - build automated content research system
  - set up remotion video rendering pipeline
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

Complete automation system for content creation: from research and scriptwriting to automatic video generation. Leverages Claude 3, OpenAI, and Remotion to transform keywords into published content and videos.

## What It Does

- **Auto-Research**: Crawls latest news from TechCrunch, a16z, X/Twitter, LinkedIn (last 24h)
- **AI Content Generation**: Creates articles in multiple formats (toplist, POV, case study, how-to)
- **Multi-language**: Generates English and Vietnamese content simultaneously
- **Video Rendering**: Auto-converts content to infographics and short videos via Remotion
- **Platform Optimization**: Exports videos for Reels, TikTok, Shorts

## Installation

```bash
# Clone repository
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

## Configuration

Create `.env.local` with required API keys:

```bash
# AI Services
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_url

# Remotion (Video Rendering)
REMOTION_LICENSE_KEY=your_remotion_key

# Optional
NODE_ENV=development
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
│   │   └── video/       # Remotion video generation
│   ├── types/           # TypeScript types
│   └── utils/           # Utility functions
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Core Usage Patterns

### 1. Research Pipeline

```typescript
import { researchContent } from '@/lib/research/crawler';

// Auto-scan for latest content
const researchData = await researchContent({
  keyword: 'AI automation',
  sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
  timeframe: '24h',
  language: 'en'
});

// Returns structured data
interface ResearchResult {
  articles: Article[];
  insights: string[];
  trends: Trend[];
  statistics: Statistic[];
}
```

### 2. AI Content Generation

```typescript
import { generateContent } from '@/lib/ai/content-generator';

// Generate multi-format content
const content = await generateContent({
  keyword: 'AI automation',
  format: 'toplist', // 'pov' | 'case-study' | 'how-to'
  tone: 'professional', // 'friendly' | 'humorous'
  languages: ['en', 'vi'],
  aiProvider: 'claude', // 'openai'
  researchData: researchData
});

// Returns generated content
interface ContentResult {
  title: string;
  body: string;
  metadata: {
    language: string;
    wordCount: number;
    readingTime: number;
  };
  seo: {
    description: string;
    keywords: string[];
  };
}
```

### 3. Using Claude API

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

async function generateWithClaude(prompt: string, context: string) {
  const message = await anthropic.messages.create({
    model: 'claude-3-sonnet-20240229',
    max_tokens: 4096,
    messages: [
      {
        role: 'user',
        content: `${context}\n\n${prompt}`
      }
    ],
  });

  return message.content[0].text;
}

// Example: Generate article outline
const outline = await generateWithClaude(
  'Create a detailed outline for an article about AI automation in marketing',
  `Research data: ${JSON.stringify(researchData)}`
);
```

### 4. Using OpenAI API

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateWithOpenAI(prompt: string, systemPrompt: string) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      { role: 'system', content: systemPrompt },
      { role: 'user', content: prompt }
    ],
    temperature: 0.7,
    max_tokens: 2000,
  });

  return completion.choices[0].message.content;
}

// Example: Translate content
const translated = await generateWithOpenAI(
  content.body,
  'You are a professional translator. Translate the following content to Vietnamese while maintaining tone and context.'
);
```

### 5. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

async function renderContentVideo(content: ContentResult) {
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo', // Your composition ID
    inputProps: {
      title: content.title,
      points: content.body.split('\n').filter(line => line.trim()),
      duration: 30, // seconds
    },
  });

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: `out/${content.metadata.language}-video.mp4`,
    inputProps: composition.defaultProps,
  });

  return {
    videoPath: `out/${content.metadata.language}-video.mp4`,
    duration: composition.durationInFrames / composition.fps,
  };
}
```

### 6. Complete Pipeline Example

```typescript
import { ContentPipeline } from '@/lib/pipeline';

async function runFullPipeline(keyword: string) {
  const pipeline = new ContentPipeline({
    aiProvider: 'claude',
    outputFormats: ['article', 'video'],
    languages: ['en', 'vi'],
  });

  // Step 1: Research
  const research = await pipeline.research(keyword);
  
  // Step 2: Generate content
  const contents = await pipeline.generateContent({
    keyword,
    research,
    formats: ['toplist', 'how-to'],
  });

  // Step 3: Render videos
  const videos = await pipeline.renderVideos(contents);

  // Step 4: Export results
  return {
    articles: contents,
    videos: videos,
    analytics: {
      timeElapsed: pipeline.getExecutionTime(),
      tokensUsed: pipeline.getTokenCount(),
    },
  };
}

// Execute pipeline
const result = await runFullPipeline('AI marketing automation');
console.log(`Generated ${result.articles.length} articles and ${result.videos.length} videos`);
```

## API Routes (Next.js)

### Generate Content Endpoint

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { generateContent } from '@/lib/ai/content-generator';

export async function POST(request: NextRequest) {
  try {
    const { keyword, format, languages } = await request.json();

    const content = await generateContent({
      keyword,
      format,
      languages,
      aiProvider: 'claude',
    });

    return NextResponse.json({ success: true, data: content });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

### Research Endpoint

```typescript
// app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { researchContent } from '@/lib/research/crawler';

export async function POST(request: NextRequest) {
  try {
    const { keyword, sources } = await request.json();

    const research = await researchContent({
      keyword,
      sources,
      timeframe: '24h',
    });

    return NextResponse.json({ success: true, data: research });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

## TypeScript Types

```typescript
// types/content.ts
export interface ContentConfig {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  tone: 'professional' | 'friendly' | 'humorous';
  languages: string[];
  aiProvider: 'claude' | 'openai';
}

export interface Article {
  id: string;
  title: string;
  body: string;
  language: string;
  format: string;
  metadata: ArticleMetadata;
  createdAt: Date;
}

export interface ArticleMetadata {
  wordCount: number;
  readingTime: number;
  seoScore: number;
  keywords: string[];
}

export interface VideoConfig {
  aspectRatio: '9:16' | '16:9' | '1:1';
  duration: number;
  fps: 30 | 60;
  resolution: '1080p' | '720p' | '4k';
}
```

## Common Workflows

### Workflow 1: Quick Article Generation

```typescript
import { quickGenerate } from '@/lib/shortcuts';

// One-line content generation
const article = await quickGenerate('AI automation trends', {
  language: 'en',
  format: 'toplist',
});

console.log(article.title);
console.log(article.body);
```

### Workflow 2: Batch Content Creation

```typescript
const keywords = [
  'AI automation',
  'marketing automation',
  'content creation AI',
];

const batchResults = await Promise.all(
  keywords.map(keyword =>
    generateContent({
      keyword,
      format: 'toplist',
      languages: ['en', 'vi'],
      aiProvider: 'claude',
    })
  )
);

console.log(`Generated ${batchResults.length * 2} articles`);
```

### Workflow 3: Research + Generate + Render

```typescript
async function completeContentFlow(keyword: string) {
  // 1. Research
  const research = await researchContent({
    keyword,
    sources: ['techcrunch', 'twitter'],
    timeframe: '24h',
  });

  // 2. Generate content with research context
  const content = await generateContent({
    keyword,
    format: 'how-to',
    languages: ['en'],
    aiProvider: 'claude',
    researchData: research,
  });

  // 3. Render video
  const video = await renderContentVideo(content);

  // 4. Return complete package
  return {
    article: content,
    video: video,
    research: research,
  };
}
```

## Development Commands

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Run production server
npm run start

# Type checking
npm run type-check

# Lint code
npm run lint

# Render Remotion video (example)
npx remotion render ContentVideo out/video.mp4
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
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      const delay = Math.pow(2, i) * 1000;
      await new Promise(resolve => setTimeout(resolve, delay));
    }
  }
  throw new Error('Max retries reached');
}

// Usage
const content = await retryWithBackoff(() =>
  generateContent({ keyword: 'AI', format: 'toplist', languages: ['en'] })
);
```

### Memory Issues with Video Rendering

```typescript
// Render videos sequentially instead of parallel
async function renderVideosSequentially(contents: ContentResult[]) {
  const videos = [];
  for (const content of contents) {
    const video = await renderContentVideo(content);
    videos.push(video);
    // Clear memory between renders
    if (global.gc) global.gc();
  }
  return videos;
}
```

### Claude/OpenAI Token Limits

```typescript
// Split long content into chunks
function splitIntoChunks(text: string, maxTokens = 2000): string[] {
  const words = text.split(' ');
  const chunks: string[] = [];
  let currentChunk: string[] = [];
  
  for (const word of words) {
    currentChunk.push(word);
    // Rough estimate: 1 token ≈ 0.75 words
    if (currentChunk.length * 0.75 >= maxTokens) {
      chunks.push(currentChunk.join(' '));
      currentChunk = [];
    }
  }
  
  if (currentChunk.length > 0) {
    chunks.push(currentChunk.join(' '));
  }
  
  return chunks;
}
```

### Environment Variable Not Found

```typescript
// Safe environment variable access
function getRequiredEnv(key: string): string {
  const value = process.env[key];
  if (!value) {
    throw new Error(`Missing required environment variable: ${key}`);
  }
  return value;
}

// Usage
const apiKey = getRequiredEnv('ANTHROPIC_API_KEY');
```

## Best Practices

1. **Always validate API keys before operations**:
```typescript
const validateConfig = () => {
  const required = ['ANTHROPIC_API_KEY', 'OPENAI_API_KEY', 'RAPIDAPI_KEY'];
  const missing = required.filter(key => !process.env[key]);
  if (missing.length > 0) {
    throw new Error(`Missing API keys: ${missing.join(', ')}`);
  }
};
```

2. **Cache research results to avoid redundant API calls**:
```typescript
const cache = new Map<string, ResearchResult>();

async function getCachedResearch(keyword: string) {
  if (cache.has(keyword)) {
    return cache.get(keyword);
  }
  const result = await researchContent({ keyword });
  cache.set(keyword, result);
  return result;
}
```

3. **Use streaming for long-form content generation**:
```typescript
async function* streamContent(prompt: string) {
  const stream = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [{ role: 'user', content: prompt }],
    stream: true,
  });

  for await (const chunk of stream) {
    yield chunk.choices[0]?.delta?.content || '';
  }
}
```
