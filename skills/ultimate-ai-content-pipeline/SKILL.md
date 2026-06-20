---
name: ultimate-ai-content-pipeline
description: Full-stack AI content automation pipeline with research, scripting, video generation, and multi-format content creation using Claude/OpenAI and Remotion
triggers:
  - automate content creation with AI research and video generation
  - build content pipeline from research to video with AI
  - create automated marketing content with Claude and OpenAI
  - generate videos and articles from keywords automatically
  - set up AI content automation with Remotion video rendering
  - integrate content research and video generation pipeline
  - automate content workflow from scraping to publishing
  - build AI-powered content production system
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the Ultimate AI Content Pipeline, a comprehensive TypeScript-based system that automates the entire content creation workflow: from research and article generation to video rendering and multi-platform distribution.

## What This Project Does

Ultimate AI Content Pipeline is an all-in-one content automation system that:

- **Auto-Research**: Crawls and analyzes real-time data from sources like TechCrunch, a16z, Twitter/X, and LinkedIn
- **AI Content Generation**: Creates articles in multiple formats (toplist, POV, case studies, how-to) using Claude 3 and OpenAI
- **Multi-Language Support**: Generates content in both English and Vietnamese with customizable tone
- **Video Rendering**: Automatically creates infographics and short videos using Remotion
- **Platform Optimization**: Exports content optimized for Reels, TikTok, and Shorts

## Installation

### Prerequisites

```bash
# Node.js 18+ required
node --version

# Package manager (npm, yarn, or pnpm)
npm --version
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
# or
pnpm install

# Set up environment variables
cp .env.example .env
```

### Environment Configuration

Create a `.env` file with the following required variables:

```env
# AI Model APIs
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_url

# Video Rendering
REMOTION_LICENSE_KEY=your_remotion_key

# Application
NEXT_PUBLIC_APP_URL=http://localhost:3000
NODE_ENV=development
```

### Development Server

```bash
# Run development server
npm run dev

# Access at http://localhost:3000
```

## Core Architecture

### Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app router pages
│   ├── components/       # React components
│   ├── lib/             # Core utilities and services
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── research/    # Web scraping and data collection
│   │   ├── video/       # Remotion video generation
│   │   └── content/     # Content formatting and processing
│   ├── types/           # TypeScript type definitions
│   └── config/          # Configuration files
├── public/              # Static assets
└── remotion/            # Remotion video templates
```

## Key Features & Usage

### 1. Content Research Module

The research module automatically scrapes and analyzes content from multiple sources.

```typescript
// src/lib/research/scraper.ts
import { ResearchService } from '@/lib/research/research-service';

interface ResearchResult {
  sources: Array<{
    url: string;
    title: string;
    content: string;
    publishedAt: Date;
  }>;
  insights: string[];
  trends: string[];
}

async function researchTopic(keyword: string): Promise<ResearchResult> {
  const researchService = new ResearchService({
    rapidApiKey: process.env.RAPIDAPI_KEY!,
  });

  // Scrape content from last 24 hours
  const results = await researchService.search({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeRange: '24h',
    maxResults: 50,
  });

  // Extract insights using AI
  const insights = await researchService.extractInsights(results);

  return {
    sources: results,
    insights: insights.keyPoints,
    trends: insights.emergingTrends,
  };
}

// Usage
const research = await researchTopic('AI automation');
console.log(`Found ${research.sources.length} sources`);
```

### 2. AI Content Generation

Generate content in multiple formats using Claude or OpenAI.

```typescript
// src/lib/ai/content-generator.ts
import { Anthropic } from '@anthropic-ai/sdk';
import { OpenAI } from 'openai';

interface ContentOptions {
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  targetAudience: string;
}

class ContentGenerator {
  private claude: Anthropic;
  private openai: OpenAI;

  constructor() {
    this.claude = new Anthropic({
      apiKey: process.env.ANTHROPIC_API_KEY!,
    });
    this.openai = new OpenAI({
      apiKey: process.env.OPENAI_API_KEY!,
    });
  }

  async generateArticle(
    research: ResearchResult,
    options: ContentOptions
  ): Promise<string> {
    const prompt = this.buildPrompt(research, options);

    // Use Claude for long-form content
    const message = await this.claude.messages.create({
      model: 'claude-3-5-sonnet-20241022',
      max_tokens: 4000,
      messages: [{
        role: 'user',
        content: prompt,
      }],
    });

    return message.content[0].type === 'text' 
      ? message.content[0].text 
      : '';
  }

  private buildPrompt(research: ResearchResult, options: ContentOptions): string {
    const formatInstructions = {
      'toplist': 'Create a numbered list article with actionable items',
      'pov': 'Write from a unique perspective with strong opinions',
      'case-study': 'Analyze real examples with data and outcomes',
      'how-to': 'Provide step-by-step instructions with examples',
    };

    return `
You are a ${options.tone} content writer creating a ${options.format} article.

Research Data:
${research.insights.join('\n')}

Key Trends:
${research.trends.join('\n')}

Sources:
${research.sources.map(s => `- ${s.title} (${s.url})`).join('\n')}

Requirements:
- Format: ${formatInstructions[options.format]}
- Language: ${options.language === 'en' ? 'English' : 'Vietnamese'}
- Tone: ${options.tone}
- Target Audience: ${options.targetAudience}
- Include data points and specific examples from sources
- Add section headings and bullet points for readability

Create a comprehensive article (1500-2000 words).
    `.trim();
  }
}

// Usage
const generator = new ContentGenerator();
const article = await generator.generateArticle(research, {
  format: 'toplist',
  language: 'en',
  tone: 'expert',
  targetAudience: 'marketing professionals',
});
```

### 3. Video Generation with Remotion

Automatically create videos from article content.

```typescript
// src/lib/video/video-generator.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import { getCompositions } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  title: string;
  keyPoints: string[];
  style: 'minimal' | 'dynamic' | 'corporate';
  platform: 'reels' | 'tiktok' | 'youtube-shorts';
}

class VideoGenerator {
  private getVideoDimensions(platform: string) {
    const dimensions = {
      'reels': { width: 1080, height: 1920 },
      'tiktok': { width: 1080, height: 1920 },
      'youtube-shorts': { width: 1080, height: 1920 },
    };
    return dimensions[platform as keyof typeof dimensions];
  }

  async generateVideo(config: VideoConfig): Promise<string> {
    // Bundle Remotion project
    const bundleLocation = await bundle({
      entryPoint: path.join(process.cwd(), 'remotion/index.ts'),
      webpackOverride: (config) => config,
    });

    // Get composition
    const compositions = await getCompositions(bundleLocation);
    const composition = await selectComposition({
      serveUrl: bundleLocation,
      id: 'ContentVideo',
      inputProps: {
        title: config.title,
        points: config.keyPoints,
        style: config.style,
      },
    });

    // Render video
    const dimensions = this.getVideoDimensions(config.platform);
    const outputPath = path.join(
      process.cwd(),
      'public/videos',
      `${Date.now()}.mp4`
    );

    await renderMedia({
      composition: {
        ...composition,
        width: dimensions.width,
        height: dimensions.height,
      },
      serveUrl: bundleLocation,
      codec: 'h264',
      outputLocation: outputPath,
      inputProps: {
        title: config.title,
        points: config.keyPoints,
        style: config.style,
      },
    });

    return outputPath;
  }
}

// Usage
const videoGen = new VideoGenerator();
const videoPath = await videoGen.generateVideo({
  title: 'Top 5 AI Tools for Content Creators',
  keyPoints: [
    'Claude for long-form writing',
    'Midjourney for visuals',
    'Remotion for video automation',
    'ChatGPT for ideation',
    'Descript for editing',
  ],
  style: 'dynamic',
  platform: 'reels',
});
```

### 4. Remotion Video Template

Example Remotion composition for content videos:

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig, interpolate } from 'remotion';

interface ContentVideoProps {
  title: string;
  points: string[];
  style: 'minimal' | 'dynamic' | 'corporate';
}

export const ContentVideo: React.FC<ContentVideoProps> = ({ title, points, style }) => {
  const frame = useCurrentFrame();
  const { fps, durationInFrames } = useVideoConfig();

  const titleOpacity = interpolate(frame, [0, 30], [0, 1], {
    extrapolateRight: 'clamp',
  });

  const titleScale = interpolate(frame, [0, 30], [0.8, 1], {
    extrapolateRight: 'clamp',
  });

  return (
    <AbsoluteFill
      style={{
        backgroundColor: style === 'corporate' ? '#1a1a1a' : '#ffffff',
        padding: 60,
        fontFamily: 'Inter, sans-serif',
      }}
    >
      {/* Title Section */}
      <div
        style={{
          opacity: titleOpacity,
          transform: `scale(${titleScale})`,
          marginBottom: 80,
        }}
      >
        <h1
          style={{
            fontSize: 72,
            fontWeight: 'bold',
            color: style === 'corporate' ? '#ffffff' : '#000000',
            lineHeight: 1.2,
            margin: 0,
          }}
        >
          {title}
        </h1>
      </div>

      {/* Points Section */}
      <div style={{ display: 'flex', flexDirection: 'column', gap: 40 }}>
        {points.map((point, index) => {
          const startFrame = 60 + index * 90;
          const opacity = interpolate(
            frame,
            [startFrame, startFrame + 20],
            [0, 1],
            { extrapolateRight: 'clamp' }
          );

          const translateY = interpolate(
            frame,
            [startFrame, startFrame + 20],
            [30, 0],
            { extrapolateRight: 'clamp' }
          );

          return (
            <div
              key={index}
              style={{
                opacity,
                transform: `translateY(${translateY}px)`,
                display: 'flex',
                alignItems: 'center',
                gap: 20,
              }}
            >
              <div
                style={{
                  width: 60,
                  height: 60,
                  borderRadius: '50%',
                  backgroundColor: '#3b82f6',
                  display: 'flex',
                  alignItems: 'center',
                  justifyContent: 'center',
                  fontSize: 32,
                  fontWeight: 'bold',
                  color: '#ffffff',
                }}
              >
                {index + 1}
              </div>
              <p
                style={{
                  fontSize: 42,
                  margin: 0,
                  color: style === 'corporate' ? '#ffffff' : '#000000',
                }}
              >
                {point}
              </p>
            </div>
          );
        })}
      </div>
    </AbsoluteFill>
  );
};
```

## Complete Workflow Example

Full pipeline from keyword to published content:

```typescript
// src/lib/pipeline/content-pipeline.ts
import { ResearchService } from '@/lib/research/research-service';
import { ContentGenerator } from '@/lib/ai/content-generator';
import { VideoGenerator } from '@/lib/video/video-generator';

interface PipelineResult {
  article: {
    title: string;
    content: string;
    language: 'en' | 'vi';
  };
  video: {
    path: string;
    platform: string;
  };
  metadata: {
    keywords: string[];
    sources: string[];
    generatedAt: Date;
  };
}

class ContentPipeline {
  private research: ResearchService;
  private generator: ContentGenerator;
  private videoGen: VideoGenerator;

  constructor() {
    this.research = new ResearchService({
      rapidApiKey: process.env.RAPIDAPI_KEY!,
    });
    this.generator = new ContentGenerator();
    this.videoGen = new VideoGenerator();
  }

  async execute(keyword: string): Promise<PipelineResult> {
    // Step 1: Research
    console.log('📡 Researching topic...');
    const researchData = await this.research.search({
      keyword,
      sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
      timeRange: '24h',
      maxResults: 50,
    });

    const insights = await this.research.extractInsights(researchData);

    // Step 2: Generate Article (English)
    console.log('✍️ Generating English article...');
    const articleEN = await this.generator.generateArticle(
      { sources: researchData, insights: insights.keyPoints, trends: insights.emergingTrends },
      {
        format: 'toplist',
        language: 'en',
        tone: 'expert',
        targetAudience: 'marketing professionals',
      }
    );

    // Step 3: Generate Article (Vietnamese)
    console.log('✍️ Generating Vietnamese article...');
    const articleVI = await this.generator.generateArticle(
      { sources: researchData, insights: insights.keyPoints, trends: insights.emergingTrends },
      {
        format: 'toplist',
        language: 'vi',
        tone: 'friendly',
        targetAudience: 'marketers Việt Nam',
      }
    );

    // Step 4: Extract Key Points for Video
    const keyPoints = insights.keyPoints.slice(0, 5);

    // Step 5: Generate Video
    console.log('🎬 Rendering video...');
    const videoPath = await this.videoGen.generateVideo({
      title: `Top ${keyPoints.length} Insights: ${keyword}`,
      keyPoints,
      style: 'dynamic',
      platform: 'reels',
    });

    return {
      article: {
        title: `${keyword}: Latest Insights and Trends`,
        content: articleEN,
        language: 'en',
      },
      video: {
        path: videoPath,
        platform: 'reels',
      },
      metadata: {
        keywords: [keyword, ...insights.emergingTrends],
        sources: researchData.map(s => s.url),
        generatedAt: new Date(),
      },
    };
  }
}

// Usage
const pipeline = new ContentPipeline();
const result = await pipeline.execute('AI content automation');

console.log('✅ Pipeline complete!');
console.log(`Article: ${result.article.title}`);
console.log(`Video: ${result.video.path}`);
console.log(`Sources: ${result.metadata.sources.length}`);
```

## API Routes (Next.js)

### Generate Content Endpoint

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { ContentPipeline } from '@/lib/pipeline/content-pipeline';

export async function POST(request: NextRequest) {
  try {
    const { keyword, format, language } = await request.json();

    if (!keyword) {
      return NextResponse.json(
        { error: 'Keyword is required' },
        { status: 400 }
      );
    }

    const pipeline = new ContentPipeline();
    const result = await pipeline.execute(keyword);

    return NextResponse.json({
      success: true,
      data: result,
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

### Client Usage

```typescript
// src/components/ContentGenerator.tsx
'use client';

import { useState } from 'react';

export default function ContentGeneratorForm() {
  const [keyword, setKeyword] = useState('');
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState(null);

  const handleGenerate = async () => {
    setLoading(true);
    try {
      const response = await fetch('/api/generate', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ keyword }),
      });

      const data = await response.json();
      setResult(data.data);
    } catch (error) {
      console.error('Error:', error);
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="p-6 max-w-2xl mx-auto">
      <input
        type="text"
        value={keyword}
        onChange={(e) => setKeyword(e.target.value)}
        placeholder="Enter keyword..."
        className="w-full px-4 py-2 border rounded"
      />
      <button
        onClick={handleGenerate}
        disabled={loading}
        className="mt-4 px-6 py-2 bg-blue-600 text-white rounded"
      >
        {loading ? 'Generating...' : 'Generate Content'}
      </button>

      {result && (
        <div className="mt-6">
          <h3 className="text-xl font-bold">{result.article.title}</h3>
          <p className="mt-2">{result.article.content.substring(0, 200)}...</p>
          <video src={result.video.path} controls className="mt-4 w-full" />
        </div>
      )}
    </div>
  );
}
```

## Common Patterns

### Rate Limiting for API Calls

```typescript
// src/lib/utils/rate-limiter.ts
class RateLimiter {
  private queue: Array<() => Promise<any>> = [];
  private processing = false;
  private delayMs: number;

  constructor(requestsPerMinute: number) {
    this.delayMs = 60000 / requestsPerMinute;
  }

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

      if (!this.processing) {
        this.process();
      }
    });
  }

  private async process() {
    this.processing = true;

    while (this.queue.length > 0) {
      const fn = this.queue.shift();
      if (fn) {
        await fn();
        await new Promise(resolve => setTimeout(resolve, this.delayMs));
      }
    }

    this.processing = false;
  }
}

// Usage with Claude API
const limiter = new RateLimiter(50); // 50 requests per minute

const result = await limiter.add(() =>
  claude.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4000,
    messages: [{ role: 'user', content: prompt }],
  })
);
```

### Caching Research Results

```typescript
// src/lib/cache/research-cache.ts
import { Redis } from 'ioredis';

class ResearchCache {
  private redis: Redis;

  constructor() {
    this.redis = new Redis(process.env.REDIS_URL);
  }

  async get(keyword: string): Promise<ResearchResult | null> {
    const cached = await this.redis.get(`research:${keyword}`);
    return cached ? JSON.parse(cached) : null;
  }

  async set(keyword: string, data: ResearchResult, ttlSeconds = 3600) {
    await this.redis.setex(
      `research:${keyword}`,
      ttlSeconds,
      JSON.stringify(data)
    );
  }
}

// Usage
const cache = new ResearchCache();
const cached = await cache.get(keyword);

if (cached) {
  return cached;
}

const fresh = await researchService.search({ keyword });
await cache.set(keyword, fresh, 3600); // Cache for 1 hour
```

## Troubleshooting

### API Key Issues

```typescript
// Validate environment variables on startup
function validateEnv() {
  const required = [
    'OPENAI_API_KEY',
    'ANTHROPIC_API_KEY',
    'RAPIDAPI_KEY',
  ];

  const missing = required.filter(key => !process.env[key]);

  if (missing.length > 0) {
    throw new Error(
      `Missing required environment variables: ${missing.join(', ')}`
    );
  }
}

validateEnv();
```

### Remotion Rendering Errors

```typescript
// Add error handling for video generation
try {
  const videoPath = await videoGen.generateVideo(config);
  return videoPath;
} catch (error) {
  if (error.message.includes('ENOMEM')) {
    console.error('Out of memory. Try reducing video resolution or duration.');
  } else if (error.message.includes('FFmpeg')) {
    console.error('FFmpeg not installed. Run: brew install ffmpeg');
  }
  throw error;
}
```

### Content Generation Timeout

```typescript
// Add timeout wrapper
async function withTimeout<T>(
  promise: Promise<T>,
  timeoutMs: number
): Promise<T> {
  const timeout = new Promise<never>((_, reject) =>
    setTimeout(() => reject(new Error('Operation timed out')), timeoutMs)
  );

  return Promise.race([promise, timeout]);
}

// Usage
const article = await withTimeout(
  generator.generateArticle(research, options),
  120000 // 2 minute timeout
);
```

## Performance Optimization

### Parallel Content Generation

```typescript
// Generate multiple language versions in parallel
const [articleEN, articleVI] = await Promise.all([
  generator.generateArticle(research, { ...options, language: 'en' }),
  generator.generateArticle(research, { ...options, language: 'vi' }),
]);
```

### Batch Video Rendering

```typescript
// Queue multiple videos for batch processing
const videos = await Promise.allSettled([
  videoGen.generateVideo({ ...config, platform: 'reels' }),
  videoGen.generateVideo({ ...config, platform: 'tiktok' }),
  videoGen.generateVideo({ ...config, platform: 'youtube-shorts' }),
]);

const successful = videos
  .filter(result => result.status === 'fulfilled')
  .map(result => result.value);
```

This skill provides comprehensive guidance for working with the Ultimate AI Content Pipeline, covering setup, core features, API integration, video generation, and production-ready patterns.
