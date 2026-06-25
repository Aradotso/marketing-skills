---
name: ultimate-ai-content-pipeline
description: Automated content pipeline from research to video generation using AI (Claude/OpenAI) and Remotion
triggers:
  - how do I use the AI content pipeline for automated marketing
  - set up automated content generation with research and video
  - create AI-powered content workflow from research to publishing
  - generate marketing content automatically with Claude and OpenAI
  - build content automation pipeline with video rendering
  - automate content research and video generation workflow
  - use Remotion for automated marketing video creation
  - integrate AI content generation with social media automation
---

# Ultimate AI Content Pipeline Skill

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill provides expertise in using the Ultimate AI Content Pipeline, a TypeScript-based system that automates the entire content creation workflow: from research and scriptwriting to automated posting and video generation using Claude 3, OpenAI, and Remotion.

## What This Project Does

Ultimate AI Content Pipeline is an all-in-one content automation system that:

- **Auto-crawls news sources** (TechCrunch, a16z, Twitter/X, LinkedIn) for fresh data within 24 hours
- **Generates multi-format content** (Toplist, POV, Case Study, How-to) in multiple languages
- **Renders videos automatically** using Remotion for Reels, TikTok, and YouTube Shorts
- **Supports multiple AI providers** (Claude 3 via Anthropic, OpenAI GPT models)
- **Provides Next.js interface** for easy content management and scheduling

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

# Copy environment template
cp .env.example .env.local
```

### Environment Configuration

Create `.env.local` with the following variables:

```bash
# AI Provider APIs
OPENAI_API_KEY=your_openai_key_here
ANTHROPIC_API_KEY=your_anthropic_claude_key_here

# Research & Crawling
RAPIDAPI_KEY=your_rapidapi_key_here
SERPER_API_KEY=your_serper_key_here

# Video Rendering (Remotion)
REMOTION_WEBHOOK_SECRET=your_webhook_secret
REMOTION_OUTPUT_BUCKET=your_s3_bucket_or_storage

# Database (if using)
DATABASE_URL=postgresql://user:password@localhost:5432/content_pipeline

# Application
NEXT_PUBLIC_APP_URL=http://localhost:3000
NODE_ENV=development
```

### Running the Application

```bash
# Development mode
npm run dev

# Production build
npm run build
npm start

# Video rendering (Remotion)
npm run remotion:studio
```

## Core Architecture

### Directory Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js 14+ app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI providers (Claude, OpenAI)
│   │   ├── research/    # Content research & crawling
│   │   ├── content/     # Content generation logic
│   │   └── video/       # Remotion video rendering
│   ├── remotion/        # Remotion video compositions
│   └── utils/           # Helper functions
├── public/              # Static assets
└── prisma/             # Database schema (if applicable)
```

## Key APIs & Usage Patterns

### 1. Content Research Module

```typescript
// src/lib/research/auto-scan.ts
import { fetchLatestNews } from '@/lib/research/crawler';
import { analyzeContent } from '@/lib/ai/analyzer';

interface ResearchOptions {
  keyword: string;
  sources?: string[];
  timeRange?: '24h' | '7d' | '30d';
  language?: 'en' | 'vi';
}

export async function performResearch(options: ResearchOptions) {
  const { keyword, sources = ['techcrunch', 'a16z', 'twitter'], timeRange = '24h' } = options;
  
  // Crawl multiple sources
  const rawData = await fetchLatestNews({
    query: keyword,
    sources,
    since: timeRange
  });
  
  // AI analysis to extract insights
  const insights = await analyzeContent(rawData, {
    extractStats: true,
    identifyTrends: true,
    summarize: true
  });
  
  return {
    rawData,
    insights,
    sources: insights.topSources,
    statistics: insights.dataPoints
  };
}

// Usage
const research = await performResearch({
  keyword: 'AI automation marketing',
  sources: ['techcrunch', 'twitter', 'linkedin'],
  timeRange: '24h',
  language: 'en'
});
```

### 2. AI Content Generation

```typescript
// src/lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY
});

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY
});

type ContentFormat = 'toplist' | 'pov' | 'case-study' | 'how-to';
type ToneStyle = 'expert' | 'friendly' | 'humorous';

interface GenerateContentOptions {
  research: any;
  format: ContentFormat;
  tone: ToneStyle;
  language: 'en' | 'vi';
  provider?: 'claude' | 'openai';
}

export async function generateContent(options: GenerateContentOptions) {
  const { research, format, tone, language, provider = 'claude' } = options;
  
  const prompt = buildPrompt(research, format, tone, language);
  
  if (provider === 'claude') {
    const response = await anthropic.messages.create({
      model: 'claude-3-5-sonnet-20241022',
      max_tokens: 4096,
      messages: [{
        role: 'user',
        content: prompt
      }]
    });
    
    return {
      content: response.content[0].text,
      provider: 'claude',
      tokens: response.usage
    };
  } else {
    const response = await openai.chat.completions.create({
      model: 'gpt-4-turbo-preview',
      messages: [{
        role: 'user',
        content: prompt
      }],
      max_tokens: 4096
    });
    
    return {
      content: response.choices[0].message.content,
      provider: 'openai',
      tokens: response.usage
    };
  }
}

function buildPrompt(research: any, format: ContentFormat, tone: ToneStyle, language: string): string {
  const formatInstructions = {
    'toplist': 'Create a numbered list article with clear rankings and explanations',
    'pov': 'Write an opinion piece with a unique perspective',
    'case-study': 'Develop a detailed case study with data and outcomes',
    'how-to': 'Create a step-by-step tutorial guide'
  };
  
  return `
You are a professional content writer specializing in ${format} articles.

Research Data:
${JSON.stringify(research.insights, null, 2)}

Instructions:
- Format: ${formatInstructions[format]}
- Tone: ${tone}
- Language: ${language}
- Include statistics and data from research
- Make it engaging and actionable
- Length: 1500-2000 words

Generate the complete article now:
  `.trim();
}
```

### 3. Video Generation with Remotion

```typescript
// src/lib/video/render.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  composition: string;
  contentData: any;
  outputName: string;
  format?: 'reels' | 'tiktok' | 'shorts';
}

export async function renderContentVideo(config: VideoConfig) {
  const { composition, contentData, outputName, format = 'reels' } = config;
  
  // Dimension presets
  const dimensions = {
    reels: { width: 1080, height: 1920 },
    tiktok: { width: 1080, height: 1920 },
    shorts: { width: 1080, height: 1920 }
  };
  
  const bundleLocation = await bundle({
    entryPoint: path.join(process.cwd(), 'src/remotion/index.ts'),
    webpackOverride: (config) => config
  });
  
  const compositionData = await selectComposition({
    serveUrl: bundleLocation,
    id: composition,
    inputProps: contentData
  });
  
  const outputLocation = path.join(
    process.cwd(),
    'public/videos',
    `${outputName}.mp4`
  );
  
  await renderMedia({
    composition: compositionData,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: {
      ...contentData,
      ...dimensions[format]
    }
  });
  
  return {
    path: outputLocation,
    url: `/videos/${outputName}.mp4`,
    format,
    duration: compositionData.durationInFrames / compositionData.fps
  };
}
```

### 4. Remotion Video Composition Example

```typescript
// src/remotion/compositions/ContentVideo.tsx
import { AbsoluteFill, Sequence, useCurrentFrame, useVideoConfig } from 'remotion';
import React from 'react';

interface ContentVideoProps {
  title: string;
  points: string[];
  brandColor?: string;
}

export const ContentVideo: React.FC<ContentVideoProps> = ({
  title,
  points,
  brandColor = '#6366f1'
}) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  
  const titleOpacity = Math.min(1, frame / 30);
  
  return (
    <AbsoluteFill style={{ backgroundColor: 'white' }}>
      {/* Title Sequence */}
      <Sequence from={0} durationInFrames={fps * 2}>
        <AbsoluteFill
          style={{
            justifyContent: 'center',
            alignItems: 'center',
            opacity: titleOpacity
          }}
        >
          <h1
            style={{
              fontSize: 72,
              fontWeight: 'bold',
              color: brandColor,
              textAlign: 'center',
              padding: '0 60px'
            }}
          >
            {title}
          </h1>
        </AbsoluteFill>
      </Sequence>
      
      {/* Content Points */}
      {points.map((point, index) => (
        <Sequence
          key={index}
          from={fps * (2 + index * 3)}
          durationInFrames={fps * 3}
        >
          <ContentPoint text={point} index={index + 1} color={brandColor} />
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};

const ContentPoint: React.FC<{ text: string; index: number; color: string }> = ({
  text,
  index,
  color
}) => {
  const frame = useCurrentFrame();
  const slideIn = Math.min(1, frame / 20);
  
  return (
    <AbsoluteFill
      style={{
        justifyContent: 'center',
        alignItems: 'flex-start',
        padding: '100px 80px',
        transform: `translateX(${(1 - slideIn) * -100}px)`,
        opacity: slideIn
      }}
    >
      <div style={{ display: 'flex', gap: 30, alignItems: 'flex-start' }}>
        <div
          style={{
            fontSize: 64,
            fontWeight: 'bold',
            color,
            minWidth: 80
          }}
        >
          {index}.
        </div>
        <p
          style={{
            fontSize: 48,
            lineHeight: 1.5,
            color: '#1f2937',
            margin: 0
          }}
        >
          {text}
        </p>
      </div>
    </AbsoluteFill>
  );
};
```

### 5. Complete Workflow Integration

```typescript
// src/lib/workflow/content-pipeline.ts
import { performResearch } from '@/lib/research/auto-scan';
import { generateContent } from '@/lib/ai/content-generator';
import { renderContentVideo } from '@/lib/video/render';

export async function runCompletePipeline(keyword: string) {
  console.log('🔍 Step 1: Research...');
  const research = await performResearch({
    keyword,
    sources: ['techcrunch', 'twitter', 'linkedin'],
    timeRange: '24h',
    language: 'en'
  });
  
  console.log('✍️ Step 2: Generate Content...');
  const contentEN = await generateContent({
    research,
    format: 'toplist',
    tone: 'expert',
    language: 'en',
    provider: 'claude'
  });
  
  const contentVI = await generateContent({
    research,
    format: 'toplist',
    tone: 'friendly',
    language: 'vi',
    provider: 'claude'
  });
  
  console.log('🎬 Step 3: Render Video...');
  const video = await renderContentVideo({
    composition: 'ContentVideo',
    contentData: {
      title: research.insights.mainTopic,
      points: research.insights.keyPoints.slice(0, 5)
    },
    outputName: `video-${Date.now()}`,
    format: 'reels'
  });
  
  return {
    research,
    content: {
      en: contentEN.content,
      vi: contentVI.content
    },
    video,
    metadata: {
      keyword,
      generatedAt: new Date().toISOString(),
      tokensUsed: contentEN.tokens.total_tokens + contentVI.tokens.total_tokens
    }
  };
}
```

### 6. Next.js API Route Example

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runCompletePipeline } from '@/lib/workflow/content-pipeline';

export async function POST(request: NextRequest) {
  try {
    const { keyword, format, language } = await request.json();
    
    if (!keyword) {
      return NextResponse.json(
        { error: 'Keyword is required' },
        { status: 400 }
      );
    }
    
    const result = await runCompletePipeline(keyword);
    
    return NextResponse.json({
      success: true,
      data: result
    });
    
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { error: 'Content generation failed', details: error.message },
      { status: 500 }
    );
  }
}
```

## Configuration Best Practices

### API Rate Limiting

```typescript
// src/lib/utils/rate-limiter.ts
import { Ratelimit } from '@upstash/ratelimit';
import { Redis } from '@upstash/redis';

const redis = new Redis({
  url: process.env.UPSTASH_REDIS_URL,
  token: process.env.UPSTASH_REDIS_TOKEN
});

export const rateLimiter = new Ratelimit({
  redis,
  limiter: Ratelimit.slidingWindow(10, '1 h'), // 10 requests per hour
  analytics: true
});

// Usage in API routes
export async function checkRateLimit(identifier: string) {
  const { success, limit, remaining } = await rateLimiter.limit(identifier);
  
  if (!success) {
    throw new Error(`Rate limit exceeded. Try again later. Remaining: ${remaining}`);
  }
  
  return { limit, remaining };
}
```

### Content Quality Validation

```typescript
// src/lib/validation/content-quality.ts
export function validateContentQuality(content: string): {
  valid: boolean;
  issues: string[];
} {
  const issues: string[] = [];
  
  // Check minimum length
  if (content.length < 1000) {
    issues.push('Content too short (< 1000 characters)');
  }
  
  // Check for proper structure
  if (!content.includes('\n\n')) {
    issues.push('Missing paragraph breaks');
  }
  
  // Check for data/statistics
  const hasNumbers = /\d+%|\d+\s+(percent|users|companies)/.test(content);
  if (!hasNumbers) {
    issues.push('No statistics or data points found');
  }
  
  return {
    valid: issues.length === 0,
    issues
  };
}
```

## Troubleshooting

### Common Issues

**Issue: API Rate Limits Exceeded**
```typescript
// Implement exponential backoff
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
  throw new Error('Max retries exceeded');
}
```

**Issue: Remotion Rendering Fails**
```bash
# Check Chrome/Chromium installation
npx remotion browser ensure

# Increase memory for large videos
NODE_OPTIONS="--max-old-space-size=4096" npm run remotion:render
```

**Issue: Research Returns No Data**
```typescript
// Fallback to multiple sources
const sources = ['techcrunch', 'a16z', 'twitter', 'linkedin', 'producthunt'];
const results = await Promise.allSettled(
  sources.map(source => fetchFromSource(source, keyword))
);

const validResults = results
  .filter(r => r.status === 'fulfilled')
  .map(r => r.value);
```

**Issue: AI Hallucination in Content**
```typescript
// Add fact-checking step
import { verifyFacts } from '@/lib/ai/fact-checker';

const generatedContent = await generateContent(options);
const factCheck = await verifyFacts(generatedContent, research.rawData);

if (factCheck.confidence < 0.8) {
  // Regenerate with stricter prompts
  console.warn('Low fact confidence, regenerating...');
}
```

## Performance Optimization

### Parallel Processing

```typescript
// Process multiple languages simultaneously
const [contentEN, contentVI, video] = await Promise.all([
  generateContent({ ...options, language: 'en' }),
  generateContent({ ...options, language: 'vi' }),
  renderContentVideo(videoConfig)
]);
```

### Caching Research Results

```typescript
// src/lib/cache/research-cache.ts
import { Redis } from 'ioredis';

const redis = new Redis(process.env.REDIS_URL);

export async function getCachedResearch(keyword: string) {
  const cached = await redis.get(`research:${keyword}`);
  return cached ? JSON.parse(cached) : null;
}

export async function cacheResearch(keyword: string, data: any, ttl = 3600) {
  await redis.setex(`research:${keyword}`, ttl, JSON.stringify(data));
}
```

This skill enables AI agents to effectively use the Ultimate AI Content Pipeline for automated marketing content generation, from research through video production.
