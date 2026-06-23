---
name: marketing-content-pipeline-automation
description: AI-powered content automation pipeline for research, script generation, video rendering, and multi-platform publishing using Claude, OpenAI, and Remotion
triggers:
  - automate content creation pipeline
  - generate video content from text
  - crawl and research trending topics
  - create multilingual marketing content
  - render videos with Remotion integration
  - build AI content automation system
  - setup content research and generation workflow
  - create automated social media content pipeline
---

# Marketing Content Pipeline Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with an automated content pipeline that handles research, scriptwriting, video generation, and publishing. The system crawls fresh data from sources like TechCrunch, a16z, Twitter/X, and LinkedIn, then uses Claude/OpenAI to generate content in multiple formats (Toplist, POV, Case Study, How-to) and languages (English/Vietnamese), finally rendering videos via Remotion.

## What This Project Does

The Ultimate AI Content Pipeline automates the entire content creation workflow:

1. **Auto-Research**: Crawls and analyzes real-time data from news sources within 24 hours
2. **AI Content Generation**: Creates multi-format content using Claude 3 or OpenAI with customizable tone/voice
3. **Video Rendering**: Automatically generates infographics and short videos using Remotion
4. **Multi-Platform Export**: Outputs video in formats optimized for Reels, TikTok, Shorts

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
# AI Provider Keys
ANTHROPIC_API_KEY=your_claude_api_key_here
OPENAI_API_KEY=your_openai_api_key_here

# Research/Crawling APIs
RAPIDAPI_KEY=your_rapidapi_key_here

# Optional: Twitter/X API
TWITTER_BEARER_TOKEN=your_twitter_bearer_token

# Optional: LinkedIn API
LINKEDIN_ACCESS_TOKEN=your_linkedin_token

# Database (if used)
DATABASE_URL=your_database_url

# Remotion Configuration
REMOTION_AWS_ACCESS_KEY_ID=your_aws_access_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret_key
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── crawler/     # Research crawling modules
│   │   ├── content/     # Content generation logic
│   │   └── video/       # Remotion video rendering
│   ├── remotion/        # Remotion video templates
│   └── utils/           # Utility functions
├── public/              # Static assets
└── .env.local          # Environment variables
```

## Key API Patterns

### 1. Content Research & Crawling

```typescript
// src/lib/crawler/research.ts
import { fetchNewsData } from './sources';

interface ResearchResult {
  title: string;
  summary: string;
  source: string;
  url: string;
  publishedAt: Date;
  insights: string[];
}

export async function researchTopic(
  keyword: string,
  sources: string[] = ['techcrunch', 'a16z', 'twitter']
): Promise<ResearchResult[]> {
  const results: ResearchResult[] = [];
  
  for (const source of sources) {
    const data = await fetchNewsData(source, keyword, {
      timeRange: '24h',
      limit: 10
    });
    
    results.push(...data.map(item => ({
      title: item.title,
      summary: item.description,
      source: source,
      url: item.url,
      publishedAt: new Date(item.published),
      insights: extractInsights(item.content)
    })));
  }
  
  return results;
}

function extractInsights(content: string): string[] {
  // Logic to extract key insights from content
  const sentences = content.split('.');
  return sentences
    .filter(s => s.includes('data') || s.includes('%') || s.includes('study'))
    .slice(0, 3);
}
```

### 2. AI Content Generation with Claude

```typescript
// src/lib/ai/claude.ts
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

interface ContentOptions {
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  researchData: string[];
}

export async function generateContent(
  topic: string,
  options: ContentOptions
): Promise<string> {
  const prompt = buildPrompt(topic, options);
  
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [{
      role: 'user',
      content: prompt
    }]
  });
  
  return message.content[0].type === 'text' 
    ? message.content[0].text 
    : '';
}

function buildPrompt(topic: string, options: ContentOptions): string {
  const formatInstructions = {
    'toplist': 'Create a numbered list article with clear rankings',
    'pov': 'Write from a unique perspective with strong opinions',
    'case-study': 'Present a detailed case study with data and analysis',
    'how-to': 'Write a step-by-step tutorial with actionable advice'
  };
  
  const toneModifiers = {
    'expert': 'Use professional terminology and authoritative voice',
    'friendly': 'Use conversational and approachable language',
    'humorous': 'Include wit and light humor where appropriate'
  };
  
  return `
Write a ${options.format} article about "${topic}" in ${options.language === 'en' ? 'English' : 'Vietnamese'}.

Format: ${formatInstructions[options.format]}
Tone: ${toneModifiers[options.tone]}

Research Data:
${options.researchData.join('\n\n')}

Requirements:
- Use the research data to support your points
- Include specific statistics and examples
- Make it engaging and actionable
- Aim for 800-1200 words
`;
}
```

### 3. OpenAI Alternative Implementation

```typescript
// src/lib/ai/openai.ts
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

export async function generateContentWithGPT(
  topic: string,
  options: ContentOptions
): Promise<string> {
  const prompt = buildPrompt(topic, options);
  
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: 'You are an expert content creator specializing in marketing and trend analysis.'
      },
      {
        role: 'user',
        content: prompt
      }
    ],
    temperature: 0.7,
    max_tokens: 3000
  });
  
  return completion.choices[0].message.content || '';
}
```

### 4. Remotion Video Rendering

```typescript
// src/lib/video/render.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import { webpackOverride } from './webpack-override';
import path from 'path';

interface VideoConfig {
  title: string;
  content: string[];
  style: 'minimal' | 'dynamic' | 'professional';
  duration: number; // in seconds
}

export async function renderContentVideo(
  config: VideoConfig,
  outputPath: string
): Promise<string> {
  const compositionId = 'ContentVideo';
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./src/remotion/index.ts'),
    webpackOverride,
  });
  
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: {
      title: config.title,
      slides: config.content,
      style: config.style,
    },
  });
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: {
      title: config.title,
      slides: config.content,
      style: config.style,
    },
  });
  
  return outputPath;
}
```

### 5. Remotion Video Component

```typescript
// src/remotion/ContentVideo.tsx
import { AbsoluteFill, Sequence, useCurrentFrame, useVideoConfig } from 'remotion';
import { z } from 'zod';

export const contentVideoSchema = z.object({
  title: z.string(),
  slides: z.array(z.string()),
  style: z.enum(['minimal', 'dynamic', 'professional']),
});

export const ContentVideo: React.FC<z.infer<typeof contentVideoSchema>> = ({
  title,
  slides,
  style,
}) => {
  const frame = useCurrentFrame();
  const { fps, durationInFrames } = useVideoConfig();
  
  const framesPerSlide = Math.floor(durationInFrames / slides.length);
  const currentSlideIndex = Math.min(
    Math.floor(frame / framesPerSlide),
    slides.length - 1
  );
  
  const opacity = Math.min(1, (frame % framesPerSlide) / 30);
  
  return (
    <AbsoluteFill style={{ backgroundColor: '#000' }}>
      {/* Title Sequence */}
      <Sequence from={0} durationInFrames={framesPerSlide}>
        <AbsoluteFill
          style={{
            justifyContent: 'center',
            alignItems: 'center',
            fontSize: 80,
            color: '#fff',
            fontWeight: 'bold',
            textAlign: 'center',
            padding: 60,
          }}
        >
          {title}
        </AbsoluteFill>
      </Sequence>
      
      {/* Content Slides */}
      {slides.map((slide, index) => (
        <Sequence
          key={index}
          from={(index + 1) * framesPerSlide}
          durationInFrames={framesPerSlide}
        >
          <AbsoluteFill
            style={{
              justifyContent: 'center',
              alignItems: 'center',
              fontSize: 50,
              color: '#fff',
              padding: 80,
              textAlign: 'center',
              opacity: index === currentSlideIndex ? opacity : 0,
            }}
          >
            {slide}
          </AbsoluteFill>
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};
```

### 6. Complete Pipeline Orchestration

```typescript
// src/lib/pipeline/orchestrator.ts
import { researchTopic } from '../crawler/research';
import { generateContent } from '../ai/claude';
import { renderContentVideo } from '../video/render';

interface PipelineConfig {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  generateVideo: boolean;
}

interface PipelineResult {
  content: string;
  research: any[];
  videoPath?: string;
}

export async function runContentPipeline(
  config: PipelineConfig
): Promise<PipelineResult> {
  // Step 1: Research
  console.log('🔍 Starting research phase...');
  const researchResults = await researchTopic(config.keyword);
  
  const researchSummaries = researchResults.map(
    r => `${r.title}\n${r.summary}\nSource: ${r.source}\nInsights: ${r.insights.join(', ')}`
  );
  
  // Step 2: Generate Content
  console.log('✍️ Generating content...');
  const content = await generateContent(config.keyword, {
    format: config.format,
    language: config.language,
    tone: config.tone,
    researchData: researchSummaries,
  });
  
  let videoPath: string | undefined;
  
  // Step 3: Render Video (optional)
  if (config.generateVideo) {
    console.log('🎬 Rendering video...');
    const slides = extractKeyPoints(content, 5);
    videoPath = await renderContentVideo(
      {
        title: config.keyword,
        content: slides,
        style: 'professional',
        duration: 30,
      },
      `./output/video-${Date.now()}.mp4`
    );
  }
  
  return {
    content,
    research: researchResults,
    videoPath,
  };
}

function extractKeyPoints(content: string, count: number): string[] {
  // Simple extraction - in production, use NLP
  const paragraphs = content.split('\n\n').filter(p => p.trim().length > 50);
  return paragraphs.slice(0, count);
}
```

## Next.js API Routes

### Content Generation Endpoint

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline/orchestrator';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const { keyword, format, language, tone, generateVideo } = body;
    
    // Validate input
    if (!keyword || !format || !language || !tone) {
      return NextResponse.json(
        { error: 'Missing required fields' },
        { status: 400 }
      );
    }
    
    // Run pipeline
    const result = await runContentPipeline({
      keyword,
      format,
      language,
      tone,
      generateVideo: generateVideo ?? false,
    });
    
    return NextResponse.json(result);
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
import { researchTopic } from '@/lib/crawler/research';

export async function GET(request: NextRequest) {
  const searchParams = request.nextUrl.searchParams;
  const keyword = searchParams.get('keyword');
  const sources = searchParams.get('sources')?.split(',') || undefined;
  
  if (!keyword) {
    return NextResponse.json(
      { error: 'Keyword is required' },
      { status: 400 }
    );
  }
  
  try {
    const results = await researchTopic(keyword, sources);
    return NextResponse.json({ results });
  } catch (error) {
    console.error('Research error:', error);
    return NextResponse.json(
      { error: 'Failed to research topic' },
      { status: 500 }
    );
  }
}
```

## CLI Commands

```bash
# Development
npm run dev              # Start Next.js dev server
npm run build           # Build for production
npm run start           # Start production server

# Remotion
npm run remotion:preview   # Preview Remotion compositions
npm run remotion:render    # Render video compositions

# Type checking
npm run type-check      # Run TypeScript compiler check

# Linting
npm run lint           # Run ESLint
```

## Common Usage Patterns

### Pattern 1: Quick Content Generation

```typescript
// Quick script to generate content
import { runContentPipeline } from './lib/pipeline/orchestrator';

async function quickGenerate() {
  const result = await runContentPipeline({
    keyword: 'AI trends 2024',
    format: 'toplist',
    language: 'en',
    tone: 'expert',
    generateVideo: false,
  });
  
  console.log(result.content);
}

quickGenerate();
```

### Pattern 2: Batch Content Generation

```typescript
// Generate multiple pieces of content
const topics = [
  'AI in Marketing',
  'Social Media Trends',
  'Content Automation',
];

async function batchGenerate() {
  const results = await Promise.all(
    topics.map(topic =>
      runContentPipeline({
        keyword: topic,
        format: 'pov',
        language: 'en',
        tone: 'friendly',
        generateVideo: true,
      })
    )
  );
  
  return results;
}
```

### Pattern 3: Scheduled Content Pipeline

```typescript
// src/lib/scheduler/cron.ts
import cron from 'node-cron';
import { runContentPipeline } from '../pipeline/orchestrator';

// Run every day at 9 AM
cron.schedule('0 9 * * *', async () => {
  console.log('Running daily content generation...');
  
  const result = await runContentPipeline({
    keyword: 'trending tech news',
    format: 'toplist',
    language: 'en',
    tone: 'expert',
    generateVideo: true,
  });
  
  // Save or publish result
  console.log('Content generated:', result.content.substring(0, 100));
});
```

## Troubleshooting

### Issue: API Rate Limits

```typescript
// Implement rate limiting and retry logic
import pRetry from 'p-retry';

async function generateWithRetry(topic: string, options: ContentOptions) {
  return pRetry(
    () => generateContent(topic, options),
    {
      retries: 3,
      onFailedAttempt: error => {
        console.log(`Attempt ${error.attemptNumber} failed. Retrying...`);
      },
    }
  );
}
```

### Issue: Remotion Rendering Fails

Check your environment variables and ensure AWS credentials are set if using cloud rendering:

```typescript
// src/lib/video/config.ts
export function validateRemotionConfig() {
  const required = [
    'REMOTION_AWS_ACCESS_KEY_ID',
    'REMOTION_AWS_SECRET_ACCESS_KEY',
  ];
  
  const missing = required.filter(key => !process.env[key]);
  
  if (missing.length > 0) {
    throw new Error(`Missing Remotion config: ${missing.join(', ')}`);
  }
}
```

### Issue: Out of Memory During Video Rendering

Increase Node.js memory limit:

```bash
# In package.json scripts
"remotion:render": "NODE_OPTIONS='--max-old-space-size=4096' remotion render"
```

### Issue: Crawler Being Blocked

Implement proper delays and user agents:

```typescript
// src/lib/crawler/fetcher.ts
import axios from 'axios';

export async function fetchWithDelay(url: string, delayMs: number = 1000) {
  await new Promise(resolve => setTimeout(resolve, delayMs));
  
  return axios.get(url, {
    headers: {
      'User-Agent': 'Mozilla/5.0 (compatible; ContentBot/1.0)',
    },
    timeout: 10000,
  });
}
```

## Performance Optimization

### Caching Research Results

```typescript
// src/lib/cache/redis.ts
import { Redis } from 'ioredis';

const redis = new Redis(process.env.REDIS_URL);

export async function getCachedResearch(keyword: string) {
  const cached = await redis.get(`research:${keyword}`);
  return cached ? JSON.parse(cached) : null;
}

export async function cacheResearch(keyword: string, data: any, ttl: number = 3600) {
  await redis.setex(`research:${keyword}`, ttl, JSON.stringify(data));
}
```

### Parallel Processing

```typescript
// Generate content in multiple languages simultaneously
async function generateMultiLanguage(keyword: string) {
  const [englishContent, vietnameseContent] = await Promise.all([
    generateContent(keyword, {
      format: 'toplist',
      language: 'en',
      tone: 'expert',
      researchData: [],
    }),
    generateContent(keyword, {
      format: 'toplist',
      language: 'vi',
      tone: 'expert',
      researchData: [],
    }),
  ]);
  
  return { en: englishContent, vi: vietnameseContent };
}
```

This skill provides comprehensive guidance for AI coding agents to effectively utilize the Marketing Content Pipeline automation system for research, content generation, and video rendering workflows.
