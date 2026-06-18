---
name: ultimate-ai-content-pipeline
description: Automated content creation pipeline from research to video generation using Claude, OpenAI, and Remotion
triggers:
  - "help me set up the AI content pipeline"
  - "how do I generate content with this marketing automation system"
  - "create automated content with research and video"
  - "use the content pipeline to generate posts"
  - "set up auto content generation with AI"
  - "generate videos from content automatically"
  - "research and create content with Claude and OpenAI"
  - "automate my content workflow with this pipeline"
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

A comprehensive TypeScript-based content automation system that handles the entire content creation workflow: from research and article generation to automatic video rendering. Built with Next.js, integrates Claude/OpenAI for content generation, web scraping for research, and Remotion for video production.

## What It Does

This pipeline automates:
- **Auto-research**: Crawls recent news from TechCrunch, a16z, Twitter/X, LinkedIn
- **Multi-format content**: Generates articles in various formats (toplist, POV, case study, how-to)
- **Bilingual output**: Produces content in both English and Vietnamese
- **Video generation**: Converts text content to videos using Remotion
- **Multi-platform optimization**: Exports for Reels, TikTok, Shorts

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

Create a `.env.local` file in the project root:

```env
# AI API Keys
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key
TWITTER_API_KEY=your_twitter_api_key

# Database (if using)
DATABASE_URL=your_database_url

# Remotion Configuration
REMOTION_AWS_ACCESS_KEY_ID=your_aws_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret
```

## Key Commands

### Development Server

```bash
# Start the Next.js development server
npm run dev

# Build for production
npm run build

# Start production server
npm start
```

### Remotion Video Rendering

```bash
# Preview video compositions
npm run remotion:preview

# Render a video
npm run remotion:render

# Deploy Remotion Lambda (for cloud rendering)
npm run remotion:deploy
```

## Core API Usage

### Content Generation Pipeline

```typescript
import { generateContent } from '@/lib/content-generator';
import { researchTopic } from '@/lib/research';
import { renderVideo } from '@/lib/video-renderer';

async function createContentPipeline(keyword: string) {
  // Step 1: Research
  const researchData = await researchTopic({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeframe: '24h'
  });

  // Step 2: Generate content
  const content = await generateContent({
    keyword,
    research: researchData,
    format: 'toplist', // 'pov' | 'case-study' | 'how-to'
    languages: ['en', 'vi'],
    tone: 'professional', // 'friendly' | 'humorous'
    aiProvider: 'claude' // or 'openai'
  });

  // Step 3: Render video
  const video = await renderVideo({
    content: content.en,
    format: 'shorts', // 'reels' | 'tiktok'
    duration: 60
  });

  return { content, video };
}
```

### Research Module

```typescript
import { crawlNews } from '@/lib/scrapers/news';
import { analyzeInsights } from '@/lib/ai/analyzer';

async function performResearch(topic: string) {
  // Crawl multiple sources
  const articles = await crawlNews({
    query: topic,
    sources: ['techcrunch', 'a16z'],
    limit: 20,
    timeRange: '24h'
  });

  // Extract insights using AI
  const insights = await analyzeInsights(articles, {
    provider: 'claude',
    model: 'claude-3-sonnet-20240229'
  });

  return {
    rawData: articles,
    insights: insights.keyPoints,
    statistics: insights.dataPoints,
    trends: insights.trends
  };
}
```

### AI Content Generation

```typescript
import { Anthropic } from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY
});

async function generateArticle(prompt: string, research: any) {
  const message = await anthropic.messages.create({
    model: 'claude-3-sonnet-20240229',
    max_tokens: 4096,
    messages: [
      {
        role: 'user',
        content: `Based on this research data: ${JSON.stringify(research)}
        
        Create a comprehensive article about: ${prompt}
        
        Format: Professional blog post
        Include: Statistics, quotes, and actionable insights
        Tone: Expert but accessible`
      }
    ]
  });

  return message.content[0].text;
}
```

### OpenAI Alternative

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY
});

async function generateWithGPT(prompt: string, context: any) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: 'You are an expert content creator specializing in marketing and technology.'
      },
      {
        role: 'user',
        content: `Context: ${JSON.stringify(context)}\n\nTask: ${prompt}`
      }
    ],
    temperature: 0.7,
    max_tokens: 3000
  });

  return completion.choices[0].message.content;
}
```

### Video Rendering with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

async function renderContentVideo(content: {
  title: string;
  points: string[];
  images?: string[];
}) {
  // Bundle the Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./src/remotion/index.ts'),
    webpackOverride: (config) => config
  });

  // Get composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title: content.title,
      points: content.points,
      images: content.images || []
    }
  });

  // Render video
  const outputLocation = path.resolve('./output/video.mp4');
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: {
      title: content.title,
      points: content.points,
      images: content.images || []
    }
  });

  return outputLocation;
}
```

### Remotion Composition Example

```tsx
// src/remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig } from 'remotion';

interface ContentVideoProps {
  title: string;
  points: string[];
  images?: string[];
}

export const ContentVideo: React.FC<ContentVideoProps> = ({
  title,
  points,
  images = []
}) => {
  const frame = useCurrentFrame();
  const { fps, durationInFrames } = useVideoConfig();

  const opacity = Math.min(1, frame / 30);
  const currentPointIndex = Math.floor((frame / durationInFrames) * points.length);

  return (
    <AbsoluteFill
      style={{
        backgroundColor: '#1a1a2e',
        justifyContent: 'center',
        alignItems: 'center',
        padding: 60
      }}
    >
      <div style={{ opacity }}>
        <h1 style={{ color: 'white', fontSize: 60, marginBottom: 40 }}>
          {title}
        </h1>
        {points[currentPointIndex] && (
          <p style={{ color: '#eee', fontSize: 32, textAlign: 'center' }}>
            {points[currentPointIndex]}
          </p>
        )}
      </div>
    </AbsoluteFill>
  );
};
```

## Common Patterns

### Full Content Pipeline Workflow

```typescript
// pages/api/generate-content.ts
import type { NextApiRequest, NextApiResponse } from 'next';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  const { keyword, format, languages } = req.body;

  try {
    // 1. Research phase
    const research = await researchTopic({
      keyword,
      sources: ['techcrunch', 'a16z', 'twitter'],
      timeframe: '24h'
    });

    // 2. Content generation
    const articles = await Promise.all(
      languages.map(async (lang: string) => {
        const content = await generateContent({
          keyword,
          research,
          format,
          language: lang,
          provider: 'claude'
        });
        return { language: lang, content };
      })
    );

    // 3. Video rendering (async, don't wait)
    renderVideo({
      content: articles[0].content,
      format: 'shorts'
    }).then((videoUrl) => {
      console.log('Video rendered:', videoUrl);
    });

    res.status(200).json({
      success: true,
      articles,
      research: research.insights
    });
  } catch (error) {
    console.error('Pipeline error:', error);
    res.status(500).json({ error: 'Content generation failed' });
  }
}
```

### Scheduled Content Generation

```typescript
// lib/scheduler.ts
import cron from 'node-cron';

export function scheduleContentGeneration(topics: string[]) {
  // Run every day at 9 AM
  cron.schedule('0 9 * * *', async () => {
    console.log('Starting scheduled content generation...');

    for (const topic of topics) {
      try {
        await createContentPipeline(topic);
        console.log(`Generated content for: ${topic}`);
      } catch (error) {
        console.error(`Failed to generate for ${topic}:`, error);
      }
    }
  });
}
```

### Multi-Provider AI Fallback

```typescript
async function generateWithFallback(prompt: string, context: any) {
  try {
    // Try Claude first
    return await generateWithClaude(prompt, context);
  } catch (claudeError) {
    console.warn('Claude failed, falling back to OpenAI', claudeError);
    
    try {
      return await generateWithGPT(prompt, context);
    } catch (openaiError) {
      console.error('All AI providers failed', openaiError);
      throw new Error('Content generation failed');
    }
  }
}
```

## Configuration

### Content Format Templates

```typescript
// config/content-formats.ts
export const CONTENT_FORMATS = {
  toplist: {
    structure: 'numbered-list',
    sections: ['intro', 'items', 'conclusion'],
    minItems: 5,
    maxItems: 10
  },
  pov: {
    structure: 'opinion-piece',
    sections: ['hook', 'argument', 'evidence', 'counterpoint', 'conclusion'],
    tone: 'assertive'
  },
  'case-study': {
    structure: 'narrative',
    sections: ['background', 'challenge', 'solution', 'results', 'takeaways'],
    includeData: true
  },
  'how-to': {
    structure: 'tutorial',
    sections: ['overview', 'steps', 'tips', 'troubleshooting'],
    stepFormat: 'actionable'
  }
};
```

### Remotion Video Presets

```typescript
// config/video-presets.ts
export const VIDEO_PRESETS = {
  shorts: {
    width: 1080,
    height: 1920,
    fps: 30,
    duration: 60
  },
  reels: {
    width: 1080,
    height: 1920,
    fps: 30,
    duration: 90
  },
  tiktok: {
    width: 1080,
    height: 1920,
    fps: 30,
    duration: 60
  },
  youtube: {
    width: 1920,
    height: 1080,
    fps: 30,
    duration: 300
  }
};
```

## Troubleshooting

### API Rate Limits

```typescript
// lib/rate-limiter.ts
import Bottleneck from 'bottleneck';

const claudeLimiter = new Bottleneck({
  maxConcurrent: 1,
  minTime: 1000 // 1 request per second
});

export const rateLimitedClaudeCall = claudeLimiter.wrap(
  async (prompt: string) => {
    return await anthropic.messages.create({
      model: 'claude-3-sonnet-20240229',
      max_tokens: 4096,
      messages: [{ role: 'user', content: prompt }]
    });
  }
);
```

### Research Scraper Failures

```typescript
async function robustCrawl(url: string, retries = 3) {
  for (let i = 0; i < retries; i++) {
    try {
      const response = await fetch(url, {
        headers: {
          'User-Agent': 'Mozilla/5.0 (compatible; ContentBot/1.0)'
        }
      });
      return await response.text();
    } catch (error) {
      if (i === retries - 1) throw error;
      await new Promise(resolve => setTimeout(resolve, 2000 * (i + 1)));
    }
  }
}
```

### Video Rendering Memory Issues

```typescript
// For large videos, use Remotion Lambda
import { renderMediaOnLambda } from '@remotion/lambda';

async function renderLargeVideo(composition: any) {
  const result = await renderMediaOnLambda({
    region: 'us-east-1',
    functionName: process.env.REMOTION_LAMBDA_FUNCTION,
    composition: composition.id,
    serveUrl: composition.serveUrl,
    codec: 'h264',
    inputProps: composition.inputProps
  });

  return result.publicUrl;
}
```

### Missing Environment Variables

```typescript
// lib/config-validator.ts
export function validateConfig() {
  const required = [
    'ANTHROPIC_API_KEY',
    'OPENAI_API_KEY',
    'RAPIDAPI_KEY'
  ];

  const missing = required.filter(key => !process.env[key]);

  if (missing.length > 0) {
    throw new Error(
      `Missing required environment variables: ${missing.join(', ')}`
    );
  }
}

// Call at startup
validateConfig();
```

## Best Practices

1. **Always use environment variables** for API keys
2. **Implement rate limiting** for all external API calls
3. **Cache research results** to avoid redundant scraping
4. **Use async/await** properly to handle the pipeline stages
5. **Log extensively** for debugging the multi-step pipeline
6. **Handle AI provider failures** with fallbacks
7. **Test video compositions** in preview mode before rendering
8. **Store generated content** in a database for tracking and reuse
