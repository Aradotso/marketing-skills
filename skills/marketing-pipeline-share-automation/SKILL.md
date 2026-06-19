---
name: marketing-pipeline-share-automation
description: Automated AI content pipeline for research, scriptwriting, and video generation using Claude, OpenAI, and Remotion
triggers:
  - how do I automate content creation with AI
  - set up automated marketing content pipeline
  - generate video content from text automatically
  - create content using Claude and OpenAI APIs
  - build AI-powered content research and generation
  - automate blog posts and video generation
  - use Remotion for automated video rendering
  - scrape news and generate content with AI
---

# Marketing Pipeline Share Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the **Ultimate AI Content Pipeline**, a TypeScript-based system that automates the entire content creation workflow: from researching trending topics, generating multi-format articles, to rendering videos automatically using AI (Claude 3, OpenAI) and Remotion.

## What This Project Does

The Marketing Pipeline Share is a complete content automation system that:

- **Auto-researches** trending topics by crawling TechCrunch, a16z, Twitter/X, LinkedIn
- **Generates content** in multiple formats (toplist, POV, case study, how-to) using Claude/OpenAI
- **Supports multi-language** output (English & Vietnamese)
- **Renders videos** automatically using Remotion for social media (Reels, TikTok, Shorts)
- **Manages workflows** through a Next.js interface

## Installation

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
yarn install

# Set up environment variables
cp .env.example .env
```

### Required Environment Variables

```bash
# AI API Keys
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# Research/Scraping APIs
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_url

# Next.js Configuration
NEXT_PUBLIC_API_URL=http://localhost:3000
```

### Start Development Server

```bash
npm run dev
# or
yarn dev
```

The application will be available at `http://localhost:3000`

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app router pages
│   ├── components/       # React components
│   ├── lib/             # Core utilities
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── research/    # Content scraping & research
│   │   └── video/       # Remotion video generation
│   ├── types/           # TypeScript type definitions
│   └── utils/           # Helper functions
├── public/              # Static assets
├── remotion/            # Remotion video templates
└── package.json
```

## Core API Usage

### 1. Research & Content Scraping

```typescript
import { researchTopic } from '@/lib/research/scraper';

// Auto-research a topic from multiple sources
async function gatherResearch(keyword: string) {
  const research = await researchTopic({
    keyword: keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeframe: '24h', // Last 24 hours
    maxResults: 20
  });

  return {
    insights: research.insights,
    statistics: research.statistics,
    trendingTopics: research.trending,
    sourceLinks: research.sources
  };
}

// Example usage
const aiResearch = await gatherResearch('artificial intelligence trends');
console.log(aiResearch.insights);
```

### 2. AI Content Generation with Claude

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

async function generateContent(
  topic: string,
  format: 'toplist' | 'pov' | 'case-study' | 'how-to',
  language: 'en' | 'vi'
) {
  const prompts = {
    'toplist': `Create a comprehensive top list article about ${topic}`,
    'pov': `Write a perspective piece on ${topic}`,
    'case-study': `Develop a detailed case study about ${topic}`,
    'how-to': `Write a step-by-step guide on ${topic}`
  };

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [{
      role: 'user',
      content: `${prompts[format]}. Language: ${language}. Include data-backed insights and real examples.`
    }]
  });

  return message.content[0].text;
}

// Generate Vietnamese toplist
const content = await generateContent(
  'AI marketing tools 2026',
  'toplist',
  'vi'
);
```

### 3. OpenAI Integration

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function enhanceContent(rawContent: string, tone: string) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are a content optimizer. Adjust the tone to be ${tone} while maintaining factual accuracy.`
      },
      {
        role: 'user',
        content: rawContent
      }
    ],
    temperature: 0.7,
  });

  return completion.choices[0].message.content;
}

// Optimize for friendly, engaging tone
const optimized = await enhanceContent(content, 'friendly and engaging');
```

### 4. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

async function generateVideo(
  contentData: {
    title: string;
    points: string[];
    statistics: Array<{ label: string; value: string }>;
  },
  format: 'reels' | 'tiktok' | 'shorts'
) {
  const compositions = {
    'reels': { width: 1080, height: 1920, fps: 30 },
    'tiktok': { width: 1080, height: 1920, fps: 30 },
    'shorts': { width: 1080, height: 1920, fps: 30 }
  };

  // Bundle Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  const comp = compositions[format];

  // Render video
  const outputLocation = `./output/${contentData.title}-${format}.mp4`;
  
  await renderMedia({
    composition: {
      id: 'ContentVideo',
      width: comp.width,
      height: comp.height,
      fps: comp.fps,
      durationInFrames: 900, // 30 seconds at 30fps
      defaultProps: contentData,
    },
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
  });

  return outputLocation;
}

// Generate TikTok video
const videoPath = await generateVideo({
  title: 'Top 5 AI Tools 2026',
  points: [
    'ChatGPT-5: Revolutionary language model',
    'Midjourney V7: Photorealistic generation',
    'Claude Opus: Advanced reasoning'
  ],
  statistics: [
    { label: 'Market Size', value: '$200B' },
    { label: 'Growth Rate', value: '45% YoY' }
  ]
}, 'tiktok');
```

## Complete Content Pipeline Workflow

```typescript
import { researchTopic } from '@/lib/research/scraper';
import { generateContent } from '@/lib/ai/claude';
import { generateVideo } from '@/lib/video/remotion';
import { schedulePost } from '@/lib/social/scheduler';

async function runContentPipeline(keyword: string) {
  // Step 1: Research
  console.log('🔍 Researching topic...');
  const research = await researchTopic({
    keyword,
    sources: ['techcrunch', 'twitter'],
    timeframe: '24h'
  });

  // Step 2: Generate content (bilingual)
  console.log('✍️ Generating content...');
  const [englishContent, vietnameseContent] = await Promise.all([
    generateContent(keyword, 'toplist', 'en'),
    generateContent(keyword, 'toplist', 'vi')
  ]);

  // Step 3: Extract key points for video
  const videoData = {
    title: keyword,
    points: research.insights.slice(0, 5),
    statistics: research.statistics
  };

  // Step 4: Generate videos for multiple platforms
  console.log('🎬 Rendering videos...');
  const videos = await Promise.all([
    generateVideo(videoData, 'reels'),
    generateVideo(videoData, 'tiktok'),
    generateVideo(videoData, 'shorts')
  ]);

  // Step 5: Schedule posts (optional)
  console.log('📅 Scheduling posts...');
  await schedulePost({
    content: englishContent,
    videos: videos,
    platforms: ['facebook', 'instagram', 'tiktok'],
    scheduledTime: new Date(Date.now() + 3600000) // 1 hour from now
  });

  return {
    research,
    content: { en: englishContent, vi: vietnameseContent },
    videos
  };
}

// Execute full pipeline
const result = await runContentPipeline('AI marketing automation 2026');
```

## API Routes (Next.js)

### Research Endpoint

```typescript
// app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { researchTopic } from '@/lib/research/scraper';

export async function POST(request: NextRequest) {
  try {
    const { keyword, sources, timeframe } = await request.json();

    const research = await researchTopic({
      keyword,
      sources: sources || ['techcrunch', 'twitter'],
      timeframe: timeframe || '24h'
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

### Content Generation Endpoint

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { generateContent } from '@/lib/ai/claude';

export async function POST(request: NextRequest) {
  try {
    const { topic, format, language } = await request.json();

    const content = await generateContent(topic, format, language);

    return NextResponse.json({ success: true, content });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

## Common Patterns

### Custom Content Templates

```typescript
interface ContentTemplate {
  name: string;
  structure: string[];
  toneGuidelines: string;
  requiredSections: string[];
}

const templates: Record<string, ContentTemplate> = {
  viralPost: {
    name: 'Viral Social Post',
    structure: ['hook', 'problem', 'solution', 'cta'],
    toneGuidelines: 'casual, engaging, uses emojis',
    requiredSections: ['hook', 'cta']
  },
  longForm: {
    name: 'Long-form Article',
    structure: ['introduction', 'context', 'main-points', 'examples', 'conclusion'],
    toneGuidelines: 'professional, data-driven',
    requiredSections: ['introduction', 'main-points', 'conclusion']
  }
};

async function generateFromTemplate(
  template: ContentTemplate,
  data: Record<string, any>
) {
  const prompt = `
    Create content following this structure: ${template.structure.join(' → ')}
    Tone: ${template.toneGuidelines}
    Required sections: ${template.requiredSections.join(', ')}
    Data: ${JSON.stringify(data)}
  `;

  return await generateContent(prompt, 'custom', 'en');
}
```

### Batch Processing

```typescript
async function batchGenerateContent(keywords: string[]) {
  const results = await Promise.allSettled(
    keywords.map(keyword => runContentPipeline(keyword))
  );

  return results.map((result, index) => ({
    keyword: keywords[index],
    status: result.status,
    data: result.status === 'fulfilled' ? result.value : null,
    error: result.status === 'rejected' ? result.reason : null
  }));
}

// Process multiple keywords
const batch = await batchGenerateContent([
  'AI marketing trends',
  'social media automation',
  'video content strategy'
]);
```

## Configuration Best Practices

### Rate Limiting

```typescript
import { Ratelimit } from '@upstash/ratelimit';
import { Redis } from '@upstash/redis';

const ratelimit = new Ratelimit({
  redis: Redis.fromEnv(),
  limiter: Ratelimit.slidingWindow(10, '1 m'), // 10 requests per minute
});

async function rateLimitedGeneration(userId: string, topic: string) {
  const { success } = await ratelimit.limit(userId);

  if (!success) {
    throw new Error('Rate limit exceeded. Please try again later.');
  }

  return await generateContent(topic, 'toplist', 'en');
}
```

### Caching Research Results

```typescript
import { cacheExchange } from '@urql/exchange-graphcache';

const cache = new Map<string, { data: any; timestamp: number }>();
const CACHE_TTL = 3600000; // 1 hour

async function getCachedResearch(keyword: string) {
  const cached = cache.get(keyword);
  const now = Date.now();

  if (cached && (now - cached.timestamp) < CACHE_TTL) {
    return cached.data;
  }

  const fresh = await researchTopic({ keyword, sources: ['techcrunch'], timeframe: '24h' });
  cache.set(keyword, { data: fresh, timestamp: now });

  return fresh;
}
```

## Troubleshooting

### API Key Issues

```typescript
// Validate API keys on startup
function validateEnvironment() {
  const required = [
    'OPENAI_API_KEY',
    'ANTHROPIC_API_KEY',
    'RAPIDAPI_KEY'
  ];

  const missing = required.filter(key => !process.env[key]);

  if (missing.length > 0) {
    throw new Error(`Missing environment variables: ${missing.join(', ')}`);
  }
}

validateEnvironment();
```

### Handling API Errors

```typescript
async function safeGenerate(topic: string, retries = 3) {
  for (let i = 0; i < retries; i++) {
    try {
      return await generateContent(topic, 'toplist', 'en');
    } catch (error) {
      console.error(`Attempt ${i + 1} failed:`, error.message);
      
      if (i === retries - 1) throw error;
      
      // Exponential backoff
      await new Promise(resolve => setTimeout(resolve, Math.pow(2, i) * 1000));
    }
  }
}
```

### Video Rendering Issues

```typescript
// Check Remotion dependencies
async function validateRemotionSetup() {
  try {
    const { getCompositions } = await import('@remotion/renderer');
    console.log('✅ Remotion properly configured');
  } catch (error) {
    console.error('❌ Remotion setup issue:', error);
    console.log('Run: npm install @remotion/cli @remotion/renderer');
  }
}
```

## Testing

```typescript
import { describe, it, expect } from 'vitest';

describe('Content Pipeline', () => {
  it('should research a topic', async () => {
    const result = await researchTopic({
      keyword: 'test topic',
      sources: ['techcrunch'],
      timeframe: '24h'
    });

    expect(result).toHaveProperty('insights');
    expect(result.insights.length).toBeGreaterThan(0);
  });

  it('should generate content', async () => {
    const content = await generateContent('test', 'toplist', 'en');
    expect(content).toBeTruthy();
    expect(typeof content).toBe('string');
  });
});
```

This skill enables AI agents to effectively use the Marketing Pipeline Share for automated content creation, research, and video generation workflows.
