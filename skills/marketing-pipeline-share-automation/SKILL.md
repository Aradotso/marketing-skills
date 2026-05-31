---
name: marketing-pipeline-share-automation
description: AI-powered content automation pipeline for research, scriptwriting, video generation, and multi-platform publishing
triggers:
  - automate content creation with AI research and video generation
  - set up marketing pipeline with Claude and OpenAI
  - create automated content from research to video
  - build AI content pipeline with Remotion video rendering
  - configure multi-language content automation system
  - generate videos and articles from trending topics automatically
  - use marketing pipeline for automated social media content
  - set up AI-powered content research and publishing workflow
---

# Marketing Pipeline Share Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

Marketing Pipeline Share is a complete AI-powered content automation system that handles the entire content creation workflow: from researching trending topics, generating multi-format articles (Vietnamese & English), to rendering videos automatically. Built with Next.js, TypeScript, Claude 3, OpenAI, and Remotion for video generation.

**Key capabilities:**
- Auto-crawl news from TechCrunch, a16z, Twitter/X, LinkedIn
- Generate content in multiple formats (Toplist, POV, Case Study, How-to)
- Dual-language output (English & Vietnamese)
- Automatic video/infographic rendering via Remotion
- Optimized for Reels, TikTok, YouTube Shorts

## Installation

### Prerequisites

```bash
node >= 18.x
npm or pnpm
```

### Setup

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
pnpm install

# Copy environment template
cp .env.example .env.local
```

### Environment Configuration

Create `.env.local` with required API keys:

```bash
# AI Services
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Data Sources
RAPIDAPI_KEY=your_rapidapi_key

# Optional: Social Media APIs
TWITTER_API_KEY=your_twitter_api_key
LINKEDIN_API_KEY=your_linkedin_api_key

# App Configuration
NEXT_PUBLIC_APP_URL=http://localhost:3000
DATABASE_URL=postgresql://user:password@localhost:5432/marketing_pipeline
```

### Run Development Server

```bash
npm run dev
# or
pnpm dev
```

Access at `http://localhost:3000`

## Core Components

### 1. Research Module (Auto-Scan)

The research module crawls and analyzes trending content from multiple sources.

```typescript
// lib/research/scanner.ts
import { AnthropicClient } from '@/lib/ai/anthropic';
import { fetchTechCrunch, fetchA16z, fetchTwitterTrends } from '@/lib/sources';

export async function autoResearch(keyword: string, timeframe: '24h' | '7d' = '24h') {
  const sources = await Promise.all([
    fetchTechCrunch(keyword, timeframe),
    fetchA16z(keyword),
    fetchTwitterTrends(keyword)
  ]);

  const anthropic = new AnthropicClient(process.env.ANTHROPIC_API_KEY!);
  
  const insights = await anthropic.analyzeResearch({
    keyword,
    sources: sources.flat(),
    extractInsights: true,
    findDataPoints: true
  });

  return {
    rawData: sources,
    insights: insights.keyInsights,
    statistics: insights.dataPoints,
    trends: insights.emergingTrends
  };
}
```

**Usage:**

```typescript
import { autoResearch } from '@/lib/research/scanner';

const research = await autoResearch('AI automation', '24h');
console.log(research.insights); // AI-extracted key insights
console.log(research.statistics); // Data-backed metrics
```

### 2. Content Generation (Multi-Format)

Generate content in various formats using Claude or OpenAI.

```typescript
// lib/content/generator.ts
import Anthropic from '@anthropic-ai/sdk';

export type ContentFormat = 'toplist' | 'pov' | 'case-study' | 'how-to';
export type Tone = 'expert' | 'friendly' | 'humorous';
export type Language = 'en' | 'vi';

interface GenerateContentOptions {
  keyword: string;
  research: any;
  format: ContentFormat;
  tone?: Tone;
  languages?: Language[];
}

export async function generateContent(options: GenerateContentOptions) {
  const {
    keyword,
    research,
    format,
    tone = 'expert',
    languages = ['en', 'vi']
  } = options;

  const anthropic = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY!
  });

  const formatPrompts = {
    'toplist': 'Create a numbered list article with clear rankings',
    'pov': 'Write from a strong personal perspective with opinions',
    'case-study': 'Develop a detailed case study with data and outcomes',
    'how-to': 'Create step-by-step instructional content'
  };

  const toneInstructions = {
    'expert': 'professional, authoritative, data-driven',
    'friendly': 'conversational, approachable, relatable',
    'humorous': 'witty, entertaining, light-hearted'
  };

  const results = await Promise.all(
    languages.map(async (lang) => {
      const message = await anthropic.messages.create({
        model: 'claude-3-5-sonnet-20241022',
        max_tokens: 4096,
        messages: [{
          role: 'user',
          content: `
            Write a ${format} article about "${keyword}" in ${lang === 'en' ? 'English' : 'Vietnamese'}.
            
            Format: ${formatPrompts[format]}
            Tone: ${toneInstructions[tone]}
            
            Use this research data:
            ${JSON.stringify(research.insights, null, 2)}
            
            Include statistics: ${JSON.stringify(research.statistics, null, 2)}
            
            Requirements:
            - 800-1200 words
            - SEO optimized with H2, H3 headings
            - Include data points from research
            - Add meta description and title
            - End with clear CTA
          `
        }]
      });

      return {
        language: lang,
        content: message.content[0].type === 'text' ? message.content[0].text : '',
        model: message.model
      };
    })
  );

  return results;
}
```

**Usage:**

```typescript
import { generateContent } from '@/lib/content/generator';

const articles = await generateContent({
  keyword: 'AI Marketing Tools 2024',
  research: researchData,
  format: 'toplist',
  tone: 'expert',
  languages: ['en', 'vi']
});

console.log(articles[0].content); // English version
console.log(articles[1].content); // Vietnamese version
```

### 3. Video Rendering (Remotion)

Automatically render videos from content using Remotion.

```typescript
// lib/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoRenderOptions {
  content: {
    title: string;
    keyPoints: string[];
    statistics: Array<{ label: string; value: string }>;
  };
  format: 'reels' | 'tiktok' | 'shorts'; // 9:16
  duration?: number;
}

export async function renderContentVideo(options: VideoRenderOptions) {
  const { content, format, duration = 30 } = options;

  // Bundle Remotion composition
  const bundleLocation = await bundle({
    entryPoint: path.join(process.cwd(), 'remotion/index.ts'),
    webpackOverride: (config) => config
  });

  // Get composition
  const compositions = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title: content.title,
      keyPoints: content.keyPoints,
      statistics: content.statistics,
      theme: format
    }
  });

  // Render video
  const outputPath = path.join(
    process.cwd(),
    'public/videos',
    `${Date.now()}-${format}.mp4`
  );

  await renderMedia({
    composition: compositions,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: {
      title: content.title,
      keyPoints: content.keyPoints,
      statistics: content.statistics
    }
  });

  return {
    path: outputPath,
    format,
    duration
  };
}
```

**Remotion Composition Example:**

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig, interpolate } from 'remotion';

interface ContentVideoProps {
  title: string;
  keyPoints: string[];
  statistics: Array<{ label: string; value: string }>;
}

export const ContentVideo: React.FC<ContentVideoProps> = ({
  title,
  keyPoints,
  statistics
}) => {
  const frame = useCurrentFrame();
  const { fps, durationInFrames } = useVideoConfig();

  const titleOpacity = interpolate(frame, [0, 30], [0, 1], {
    extrapolateRight: 'clamp'
  });

  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a1a' }}>
      {/* Title Scene */}
      <AbsoluteFill style={{ opacity: titleOpacity }}>
        <div style={{
          color: 'white',
          fontSize: 60,
          fontWeight: 'bold',
          padding: 40,
          textAlign: 'center'
        }}>
          {title}
        </div>
      </AbsoluteFill>

      {/* Key Points */}
      {keyPoints.map((point, index) => {
        const startFrame = 60 + (index * 90);
        const opacity = interpolate(
          frame,
          [startFrame, startFrame + 20],
          [0, 1],
          { extrapolateRight: 'clamp' }
        );

        return (
          <AbsoluteFill key={index} style={{ opacity }}>
            <div style={{ padding: 40, color: 'white', fontSize: 40 }}>
              {point}
            </div>
          </AbsoluteFill>
        );
      })}

      {/* Statistics */}
      <AbsoluteFill>
        <div style={{
          position: 'absolute',
          bottom: 100,
          left: 0,
          right: 0,
          display: 'flex',
          justifyContent: 'space-around',
          padding: 40
        }}>
          {statistics.map((stat, i) => (
            <div key={i} style={{ color: 'white', textAlign: 'center' }}>
              <div style={{ fontSize: 48, fontWeight: 'bold' }}>{stat.value}</div>
              <div style={{ fontSize: 24 }}>{stat.label}</div>
            </div>
          ))}
        </div>
      </AbsoluteFill>
    </AbsoluteFill>
  );
};
```

## Complete Pipeline Workflow

```typescript
// app/api/pipeline/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { autoResearch } from '@/lib/research/scanner';
import { generateContent } from '@/lib/content/generator';
import { renderContentVideo } from '@/lib/video/renderer';

export async function POST(req: NextRequest) {
  const { keyword, format, videoFormat } = await req.json();

  try {
    // Step 1: Research
    const research = await autoResearch(keyword, '24h');

    // Step 2: Generate Content
    const articles = await generateContent({
      keyword,
      research,
      format,
      tone: 'expert',
      languages: ['en', 'vi']
    });

    // Step 3: Render Video
    const video = await renderContentVideo({
      content: {
        title: keyword,
        keyPoints: research.insights.slice(0, 3),
        statistics: research.statistics.slice(0, 3)
      },
      format: videoFormat || 'reels'
    });

    return NextResponse.json({
      success: true,
      data: {
        research,
        articles,
        video
      }
    });
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { error: 'Pipeline failed' },
      { status: 500 }
    );
  }
}
```

**Frontend Usage:**

```typescript
// app/dashboard/page.tsx
'use client';

import { useState } from 'react';

export default function Dashboard() {
  const [keyword, setKeyword] = useState('');
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState<any>(null);

  const handleGenerate = async () => {
    setLoading(true);
    
    const response = await fetch('/api/pipeline', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        keyword,
        format: 'toplist',
        videoFormat: 'reels'
      })
    });

    const data = await response.json();
    setResult(data);
    setLoading(false);
  };

  return (
    <div className="p-8">
      <h1>Content Pipeline</h1>
      <input
        value={keyword}
        onChange={(e) => setKeyword(e.target.value)}
        placeholder="Enter keyword..."
        className="border p-2"
      />
      <button onClick={handleGenerate} disabled={loading}>
        {loading ? 'Generating...' : 'Generate Content'}
      </button>

      {result && (
        <div className="mt-8">
          <h2>Results</h2>
          <div>
            <h3>Articles</h3>
            {result.data.articles.map((article: any, i: number) => (
              <div key={i}>
                <h4>{article.language}</h4>
                <p>{article.content.substring(0, 200)}...</p>
              </div>
            ))}
          </div>
          <div>
            <h3>Video</h3>
            <video src={result.data.video.path} controls />
          </div>
        </div>
      )}
    </div>
  );
}
```

## Common Patterns

### Scheduled Content Generation

```typescript
// lib/scheduler/cron.ts
import cron from 'node-cron';
import { autoResearch } from '@/lib/research/scanner';
import { generateContent } from '@/lib/content/generator';

// Run daily at 9 AM
cron.schedule('0 9 * * *', async () => {
  const trendingKeywords = ['AI', 'Marketing Automation', 'SEO'];
  
  for (const keyword of trendingKeywords) {
    const research = await autoResearch(keyword, '24h');
    const articles = await generateContent({
      keyword,
      research,
      format: 'toplist',
      languages: ['en', 'vi']
    });
    
    // Save to database or publish
    await saveToDatabase(articles);
  }
});
```

### Batch Video Rendering

```typescript
async function batchRenderVideos(contents: any[]) {
  const videos = await Promise.all(
    contents.map(content =>
      renderContentVideo({
        content,
        format: 'reels'
      })
    )
  );
  
  return videos;
}
```

## Troubleshooting

### API Rate Limits

```typescript
// lib/utils/rate-limiter.ts
import pLimit from 'p-limit';

const limit = pLimit(3); // Max 3 concurrent API calls

export async function rateLimitedRequests<T>(
  requests: Array<() => Promise<T>>
): Promise<T[]> {
  return Promise.all(requests.map(req => limit(req)));
}
```

### Video Rendering Failures

If Remotion rendering fails:

1. Check FFmpeg installation: `ffmpeg -version`
2. Increase memory limit in `remotion.config.ts`:

```typescript
// remotion.config.ts
export default {
  maxMemory: '4gb',
  concurrency: 2
};
```

### Content Generation Quality

Improve output quality by tuning prompts:

```typescript
const enhancedPrompt = `
  ${basePrompt}
  
  Additional requirements:
  - Use active voice
  - Include specific examples
  - Add transition phrases
  - Cite data sources
  - Keep paragraphs under 4 sentences
`;
```

### Database Connection Issues

```typescript
// lib/db/client.ts
import { PrismaClient } from '@prisma/client';

const globalForPrisma = global as unknown as { prisma: PrismaClient };

export const prisma =
  globalForPrisma.prisma ||
  new PrismaClient({
    log: ['query', 'error', 'warn'],
  });

if (process.env.NODE_ENV !== 'production') globalForPrisma.prisma = prisma;
```

## Advanced Configuration

### Custom Content Templates

```typescript
// config/templates.ts
export const contentTemplates = {
  'startup-news': {
    format: 'case-study',
    tone: 'expert',
    sections: ['overview', 'funding', 'impact', 'analysis'],
    minWords: 1000
  },
  'quick-tips': {
    format: 'how-to',
    tone: 'friendly',
    sections: ['intro', 'steps', 'conclusion'],
    minWords: 500
  }
};
```

### Multi-Platform Publishing

```typescript
// lib/publishing/platforms.ts
export async function publishToPlatforms(content: any, video: any) {
  await Promise.all([
    publishToWordPress(content.en),
    publishToMedium(content.en),
    uploadToYouTube(video),
    uploadToTikTok(video)
  ]);
}
```
