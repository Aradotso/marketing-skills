---
name: ultimate-ai-content-pipeline
description: Automated content pipeline from research to video generation using Claude, OpenAI, and Remotion for multi-format content creation
triggers:
  - automate content creation with AI research
  - generate video from written content automatically
  - create multilingual content with AI pipeline
  - crawl news and generate formatted articles
  - build automated content workflow with Remotion
  - setup AI content generation from research to video
  - configure Claude and OpenAI for content automation
  - scrape trending topics and create video content
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

An end-to-end automated content creation system that handles research, script writing, article generation, and video rendering. This TypeScript/Next.js project integrates Claude 3, OpenAI, and Remotion to transform keywords into multi-format content (articles, videos, infographics) with automatic news crawling and data-backed insights.

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
# AI Models
OPENAI_API_KEY=your_openai_key_here
ANTHROPIC_API_KEY=your_claude_key_here

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key_here

# Optional: Video Rendering
REMOTION_LICENSE_KEY=your_remotion_license_here

# App Configuration
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

## Core Architecture

The pipeline consists of four main stages:

1. **Research Phase**: Auto-crawl news sources (TechCrunch, a16z, Twitter, LinkedIn)
2. **Content Generation**: AI-powered writing with multiple formats
3. **Translation**: Dual language output (EN/VI)
4. **Media Rendering**: Automatic video/infographic generation via Remotion

## Key Components & Usage

### 1. Research & Data Crawling

```typescript
// lib/research/crawler.ts
import { NewsSource } from '@/types/research';

interface CrawlerConfig {
  sources: NewsSource[];
  timeRange: '24h' | '7d' | '30d';
  keywords: string[];
}

async function crawlNews(config: CrawlerConfig) {
  const results = await Promise.all(
    config.sources.map(source => 
      fetchFromSource(source, config.timeRange, config.keywords)
    )
  );
  
  return {
    articles: results.flat(),
    insights: extractInsights(results),
    trends: analyzeTrends(results)
  };
}

// Usage
const research = await crawlNews({
  sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
  timeRange: '24h',
  keywords: ['AI', 'automation', 'marketing']
});
```

### 2. AI Content Generation

```typescript
// lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

interface ContentConfig {
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  tone: 'expert' | 'friendly' | 'humorous';
  language: 'en' | 'vi' | 'both';
  researchData: any;
}

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY
});

async function generateContent(config: ContentConfig) {
  const prompt = buildPrompt(config);
  
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [{
      role: 'user',
      content: prompt
    }]
  });
  
  return {
    content: message.content[0].text,
    metadata: {
      format: config.format,
      wordCount: countWords(message.content[0].text),
      tone: config.tone
    }
  };
}

// Alternative with OpenAI
async function generateWithOpenAI(config: ContentConfig) {
  const openai = new OpenAI({
    apiKey: process.env.OPENAI_API_KEY
  });
  
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      { role: 'system', content: 'You are a content marketing expert.' },
      { role: 'user', content: buildPrompt(config) }
    ],
    temperature: 0.7
  });
  
  return completion.choices[0].message.content;
}
```

### 3. Multi-Format Content Pipeline

```typescript
// app/api/content/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';

export async function POST(req: NextRequest) {
  const { keyword, format, languages } = await req.json();
  
  try {
    // Step 1: Research
    const research = await crawlNews({
      sources: ['techcrunch', 'a16z'],
      timeRange: '24h',
      keywords: [keyword]
    });
    
    // Step 2: Generate Content
    const contents = await Promise.all(
      languages.map(lang => 
        generateContent({
          format,
          tone: 'expert',
          language: lang,
          researchData: research
        })
      )
    );
    
    // Step 3: Prepare for Video
    const videoScript = extractKeyPoints(contents[0].content);
    
    return NextResponse.json({
      success: true,
      data: {
        articles: contents,
        videoScript,
        insights: research.insights
      }
    });
  } catch (error) {
    return NextResponse.json(
      { error: error.message },
      { status: 500 }
    );
  }
}
```

### 4. Video Rendering with Remotion

```typescript
// remotion/compositions/ArticleVideo.tsx
import { AbsoluteFill, Sequence, useCurrentFrame } from 'remotion';

interface ArticleVideoProps {
  title: string;
  keyPoints: string[];
  duration: number;
}

export const ArticleVideo: React.FC<ArticleVideoProps> = ({
  title,
  keyPoints,
  duration
}) => {
  const frame = useCurrentFrame();
  
  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a1a' }}>
      <Sequence from={0} durationInFrames={60}>
        <Title text={title} />
      </Sequence>
      
      {keyPoints.map((point, index) => (
        <Sequence
          key={index}
          from={60 + index * 90}
          durationInFrames={90}
        >
          <KeyPoint text={point} index={index + 1} />
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};

// lib/video/render.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';

async function renderArticleVideo(content: any) {
  const bundled = await bundle({
    entryPoint: './remotion/index.ts',
    webpackOverride: (config) => config
  });
  
  const composition = await selectComposition({
    serveUrl: bundled,
    id: 'ArticleVideo',
    inputProps: {
      title: content.title,
      keyPoints: content.keyPoints,
      duration: 300
    }
  });
  
  await renderMedia({
    composition,
    serveUrl: bundled,
    codec: 'h264',
    outputLocation: `out/${content.id}.mp4`
  });
  
  return `out/${content.id}.mp4`;
}
```

### 5. Complete Workflow Implementation

```typescript
// lib/pipeline/orchestrator.ts
export class ContentPipeline {
  async execute(keyword: string, options: PipelineOptions) {
    console.log(`Starting pipeline for keyword: ${keyword}`);
    
    // Phase 1: Research
    const research = await this.researchPhase(keyword);
    console.log(`Found ${research.articles.length} articles`);
    
    // Phase 2: Content Generation
    const content = await this.contentPhase(research, options.format);
    console.log(`Generated ${options.format} content`);
    
    // Phase 3: Translation (if needed)
    const translated = options.languages.includes('both')
      ? await this.translatePhase(content)
      : content;
    
    // Phase 4: Media Generation
    const media = await this.mediaPhase(translated);
    console.log(`Rendered video: ${media.videoPath}`);
    
    return {
      articles: translated,
      media,
      insights: research.insights,
      timestamp: new Date().toISOString()
    };
  }
  
  private async researchPhase(keyword: string) {
    return crawlNews({
      sources: ['techcrunch', 'a16z', 'twitter'],
      timeRange: '24h',
      keywords: [keyword]
    });
  }
  
  private async contentPhase(research: any, format: string) {
    return generateContent({
      format,
      tone: 'expert',
      language: 'en',
      researchData: research
    });
  }
  
  private async mediaPhase(content: any) {
    const videoPath = await renderArticleVideo({
      id: content.metadata.id,
      title: content.title,
      keyPoints: extractKeyPoints(content.content)
    });
    
    return { videoPath };
  }
}

// Usage
const pipeline = new ContentPipeline();
const result = await pipeline.execute('AI automation', {
  format: 'toplist',
  languages: ['en', 'vi']
});
```

## API Routes

### Generate Content Endpoint

```typescript
// POST /api/content/generate
{
  "keyword": "AI marketing automation",
  "format": "toplist",
  "languages": ["en", "vi"],
  "includeVideo": true
}

// Response
{
  "success": true,
  "data": {
    "articles": [
      { "language": "en", "content": "...", "wordCount": 1500 },
      { "language": "vi", "content": "...", "wordCount": 1500 }
    ],
    "videoScript": { "title": "...", "keyPoints": [...] },
    "videoPath": "out/video-123.mp4",
    "insights": { "trends": [...], "sources": [...] }
  }
}
```

### Research Endpoint

```typescript
// POST /api/research/crawl
{
  "keywords": ["AI", "automation"],
  "sources": ["techcrunch", "a16z"],
  "timeRange": "24h"
}
```

## CLI Commands

```bash
# Development
npm run dev              # Start Next.js dev server
npm run build           # Build for production
npm run start           # Start production server

# Remotion
npm run remotion        # Open Remotion Studio
npm run render          # Render all videos

# Custom scripts
npm run crawl -- --keyword "AI"           # Run research crawler
npm run generate -- --format toplist      # Generate content
npm run pipeline -- --keyword "marketing" # Full pipeline
```

## Common Patterns

### Multi-Language Content Generation

```typescript
async function generateMultiLanguage(keyword: string) {
  const research = await crawlNews({
    sources: ['techcrunch'],
    timeRange: '24h',
    keywords: [keyword]
  });
  
  const [enContent, viContent] = await Promise.all([
    generateContent({
      format: 'toplist',
      tone: 'expert',
      language: 'en',
      researchData: research
    }),
    generateContent({
      format: 'toplist',
      tone: 'friendly',
      language: 'vi',
      researchData: research
    })
  ]);
  
  return { en: enContent, vi: viContent };
}
```

### Batch Processing

```typescript
async function batchGenerate(keywords: string[]) {
  const pipeline = new ContentPipeline();
  
  const results = await Promise.allSettled(
    keywords.map(keyword => 
      pipeline.execute(keyword, {
        format: 'toplist',
        languages: ['en']
      })
    )
  );
  
  const successful = results
    .filter(r => r.status === 'fulfilled')
    .map(r => r.value);
    
  return successful;
}
```

## Troubleshooting

### API Rate Limits

```typescript
// lib/utils/rate-limiter.ts
class RateLimiter {
  private queue: Array<() => Promise<any>> = [];
  private processing = false;
  
  async add<T>(fn: () => Promise<T>): Promise<T> {
    return new Promise((resolve, reject) => {
      this.queue.push(async () => {
        try {
          const result = await fn();
          resolve(result);
        } catch (error) {
          reject(error);
        }
      });
      
      this.process();
    });
  }
  
  private async process() {
    if (this.processing || this.queue.length === 0) return;
    
    this.processing = true;
    const fn = this.queue.shift()!;
    await fn();
    await new Promise(resolve => setTimeout(resolve, 1000)); // 1s delay
    this.processing = false;
    
    this.process();
  }
}

// Usage
const limiter = new RateLimiter();
const content = await limiter.add(() => generateContent(config));
```

### Error Handling

```typescript
async function safeGenerate(config: ContentConfig) {
  try {
    return await generateContent(config);
  } catch (error) {
    if (error.status === 429) {
      console.log('Rate limited, retrying in 60s...');
      await new Promise(resolve => setTimeout(resolve, 60000));
      return safeGenerate(config);
    }
    
    if (error.status === 500) {
      console.log('API error, falling back to OpenAI...');
      return generateWithOpenAI(config);
    }
    
    throw error;
  }
}
```

### Video Rendering Issues

```typescript
// Ensure Remotion dependencies are installed
// Check REMOTION_LICENSE_KEY if using premium features
// For large videos, increase Node memory:
// NODE_OPTIONS=--max-old-space-size=8192 npm run render
```

## Performance Optimization

```typescript
// Cache research results
import { Redis } from '@upstash/redis';

const redis = new Redis({
  url: process.env.REDIS_URL,
  token: process.env.REDIS_TOKEN
});

async function getCachedResearch(keyword: string) {
  const cached = await redis.get(`research:${keyword}`);
  
  if (cached) {
    return JSON.parse(cached as string);
  }
  
  const fresh = await crawlNews({
    sources: ['techcrunch'],
    timeRange: '24h',
    keywords: [keyword]
  });
  
  await redis.setex(`research:${keyword}`, 3600, JSON.stringify(fresh));
  return fresh;
}
```
