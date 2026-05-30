---
name: marketing-pipeline-share-ai-content-automation
description: Automate content creation from research to video generation using AI (Claude/OpenAI) and Remotion for Vietnamese and English marketing content
triggers:
  - how do I automate content creation with AI
  - set up marketing pipeline for auto content generation
  - create automated content workflow from research to video
  - generate blog posts and videos automatically with AI
  - build AI content pipeline for marketing automation
  - automate social media content with Claude and OpenAI
  - use Remotion to generate marketing videos from text
  - set up Vietnamese English bilingual content automation
---

# Marketing Pipeline Share - AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This project is an **end-to-end AI-powered content pipeline** that automates content creation from research and scriptwriting to video generation. It crawls news sources (TechCrunch, a16z, Twitter, LinkedIn), generates bilingual content (Vietnamese/English) using Claude 3 and OpenAI, and renders videos automatically using Remotion.

## What It Does

1. **Auto-Research**: Crawls recent news and data from major sources
2. **AI Content Generation**: Creates multi-format content (listicles, case studies, how-tos) in Vietnamese and English
3. **Video Rendering**: Automatically generates infographics and short videos using Remotion
4. **Multi-Platform Export**: Optimized output for Reels, TikTok, Shorts

## Installation

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
pnpm install
# or
yarn install
```

## Environment Setup

Create a `.env.local` file in the root directory:

```bash
# AI API Keys
OPENAI_API_KEY=your_openai_key_here
ANTHROPIC_API_KEY=your_claude_key_here

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key_here

# Social Media APIs (optional)
TWITTER_BEARER_TOKEN=your_twitter_token_here
LINKEDIN_API_KEY=your_linkedin_key_here

# Database (if using)
DATABASE_URL=your_database_url_here

# Next.js
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── crawler/     # Web scraping modules
│   │   ├── video/       # Remotion video generation
│   │   └── utils/       # Helper functions
│   └── types/           # TypeScript types
├── public/              # Static assets
└── remotion/           # Remotion video templates
```

## Core API Usage

### 1. Content Research Module

```typescript
// src/lib/crawler/newsResearch.ts
import axios from 'axios';

interface NewsSource {
  url: string;
  title: string;
  content: string;
  publishedAt: Date;
}

export async function crawlRecentNews(
  keyword: string,
  sources: string[] = ['techcrunch', 'a16z']
): Promise<NewsSource[]> {
  const results: NewsSource[] = [];
  
  for (const source of sources) {
    try {
      const response = await axios.get(
        `https://api.rapidapi.com/news/${source}`,
        {
          params: { q: keyword, from: 'last24h' },
          headers: {
            'X-RapidAPI-Key': process.env.RAPIDAPI_KEY!,
          },
        }
      );
      
      results.push(...response.data.articles);
    } catch (error) {
      console.error(`Error crawling ${source}:`, error);
    }
  }
  
  return results;
}

export async function extractInsights(articles: NewsSource[]): Promise<string> {
  const combinedContent = articles
    .map(a => `${a.title}\n${a.content}`)
    .join('\n\n');
  
  return combinedContent;
}
```

### 2. AI Content Generation with Claude

```typescript
// src/lib/ai/claudeGenerator.ts
import Anthropic from '@anthropic-ai/sdk';

const client = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY!,
});

interface ContentOptions {
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'vi' | 'en';
  tone: 'expert' | 'friendly' | 'humorous';
  researchData: string;
}

export async function generateContent(
  topic: string,
  options: ContentOptions
): Promise<string> {
  const systemPrompt = `You are an expert content creator specializing in ${options.format} format.
Write in ${options.language === 'vi' ? 'Vietnamese' : 'English'} with a ${options.tone} tone.
Use the provided research data to create data-backed, trend-leading content.`;

  const userPrompt = `Topic: ${topic}

Research Data:
${options.researchData}

Create a comprehensive ${options.format} article about this topic.`;

  const message = await client.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    system: systemPrompt,
    messages: [
      {
        role: 'user',
        content: userPrompt,
      },
    ],
  });

  return message.content[0].type === 'text' 
    ? message.content[0].text 
    : '';
}
```

### 3. OpenAI Alternative Generator

```typescript
// src/lib/ai/openaiGenerator.ts
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY!,
});

export async function generateContentOpenAI(
  topic: string,
  options: ContentOptions
): Promise<string> {
  const systemPrompt = `You are an expert content creator specializing in ${options.format} format.
Write in ${options.language === 'vi' ? 'Vietnamese' : 'English'} with a ${options.tone} tone.`;

  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      { role: 'system', content: systemPrompt },
      {
        role: 'user',
        content: `Topic: ${topic}\n\nResearch:\n${options.researchData}\n\nCreate a ${options.format} article.`,
      },
    ],
    temperature: 0.7,
    max_tokens: 3000,
  });

  return completion.choices[0]?.message?.content || '';
}
```

### 4. Bilingual Content Generation

```typescript
// src/lib/ai/bilingualGenerator.ts
import { generateContent } from './claudeGenerator';
import { ContentOptions } from './types';

export async function generateBilingualContent(
  topic: string,
  researchData: string,
  format: ContentOptions['format'],
  tone: ContentOptions['tone'] = 'expert'
) {
  const [vietnamese, english] = await Promise.all([
    generateContent(topic, {
      format,
      language: 'vi',
      tone,
      researchData,
    }),
    generateContent(topic, {
      format,
      language: 'en',
      tone,
      researchData,
    }),
  ]);

  return {
    vi: vietnamese,
    en: english,
  };
}
```

### 5. Video Generation with Remotion

```typescript
// src/lib/video/renderVideo.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoData {
  title: string;
  keyPoints: string[];
  bgColor: string;
}

export async function renderContentVideo(
  content: string,
  outputPath: string,
  platform: 'reels' | 'tiktok' | 'shorts' = 'reels'
): Promise<string> {
  // Extract key points from content
  const keyPoints = extractKeyPoints(content);
  
  const compositionData: VideoData = {
    title: keyPoints[0] || 'Content Video',
    keyPoints: keyPoints.slice(1, 6),
    bgColor: '#1a1a1a',
  };

  // Bundle Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.join(process.cwd(), 'remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  // Select composition based on platform
  const aspectRatios = {
    reels: { width: 1080, height: 1920 },
    tiktok: { width: 1080, height: 1920 },
    shorts: { width: 1080, height: 1920 },
  };

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: compositionData,
  });

  // Render video
  await renderMedia({
    composition: {
      ...composition,
      ...aspectRatios[platform],
    },
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: compositionData,
  });

  return outputPath;
}

function extractKeyPoints(content: string): string[] {
  const lines = content.split('\n').filter(line => 
    line.trim().startsWith('-') || 
    line.trim().startsWith('*') ||
    line.trim().match(/^\d+\./)
  );
  
  return lines.map(line => 
    line.replace(/^[-*\d.]\s*/, '').trim()
  ).slice(0, 6);
}
```

### 6. Remotion Video Component

```typescript
// remotion/ContentVideo.tsx
import React from 'react';
import {
  AbsoluteFill,
  interpolate,
  useCurrentFrame,
  useVideoConfig,
  Sequence,
} from 'remotion';

interface ContentVideoProps {
  title: string;
  keyPoints: string[];
  bgColor: string;
}

export const ContentVideo: React.FC<ContentVideoProps> = ({
  title,
  keyPoints,
  bgColor,
}) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  const titleOpacity = interpolate(frame, [0, 30], [0, 1], {
    extrapolateRight: 'clamp',
  });

  return (
    <AbsoluteFill style={{ backgroundColor: bgColor }}>
      <Sequence from={0} durationInFrames={60}>
        <AbsoluteFill
          style={{
            justifyContent: 'center',
            alignItems: 'center',
            opacity: titleOpacity,
          }}
        >
          <h1 style={{ fontSize: 80, color: 'white', textAlign: 'center', padding: 40 }}>
            {title}
          </h1>
        </AbsoluteFill>
      </Sequence>

      {keyPoints.map((point, index) => (
        <Sequence key={index} from={60 + index * 90} durationInFrames={90}>
          <KeyPointSlide text={point} index={index + 1} />
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};

const KeyPointSlide: React.FC<{ text: string; index: number }> = ({ text, index }) => {
  const frame = useCurrentFrame();
  
  const scale = interpolate(frame, [0, 15], [0.8, 1], {
    extrapolateRight: 'clamp',
  });

  return (
    <AbsoluteFill
      style={{
        justifyContent: 'center',
        alignItems: 'center',
        transform: `scale(${scale})`,
      }}
    >
      <div style={{ maxWidth: '80%', textAlign: 'center' }}>
        <h2 style={{ fontSize: 50, color: '#00ff88', marginBottom: 20 }}>
          #{index}
        </h2>
        <p style={{ fontSize: 40, color: 'white', lineHeight: 1.4 }}>
          {text}
        </p>
      </div>
    </AbsoluteFill>
  );
};
```

### 7. Complete Pipeline Integration

```typescript
// src/lib/pipeline/contentPipeline.ts
import { crawlRecentNews, extractInsights } from '../crawler/newsResearch';
import { generateBilingualContent } from '../ai/bilingualGenerator';
import { renderContentVideo } from '../video/renderVideo';

export async function runContentPipeline(
  keyword: string,
  format: 'toplist' | 'pov' | 'case-study' | 'how-to',
  includeVideo: boolean = true
) {
  console.log(`Starting pipeline for keyword: ${keyword}`);

  // Step 1: Research
  console.log('Step 1: Crawling news sources...');
  const articles = await crawlRecentNews(keyword);
  const researchData = await extractInsights(articles);

  // Step 2: Generate bilingual content
  console.log('Step 2: Generating content...');
  const content = await generateBilingualContent(
    keyword,
    researchData,
    format,
    'expert'
  );

  // Step 3: Generate video (optional)
  let videoPath: string | null = null;
  if (includeVideo) {
    console.log('Step 3: Rendering video...');
    videoPath = await renderContentVideo(
      content.en,
      `./output/${keyword}-${Date.now()}.mp4`,
      'reels'
    );
  }

  return {
    content,
    videoPath,
    metadata: {
      keyword,
      format,
      sourcesCount: articles.length,
      generatedAt: new Date(),
    },
  };
}
```

## Next.js API Routes

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline/contentPipeline';

export async function POST(request: NextRequest) {
  try {
    const { keyword, format, includeVideo } = await request.json();

    if (!keyword) {
      return NextResponse.json(
        { error: 'Keyword is required' },
        { status: 400 }
      );
    }

    const result = await runContentPipeline(keyword, format, includeVideo);

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

## Running the Project

```bash
# Development mode
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Render video only
npx remotion render remotion/index.ts ContentVideo output.mp4
```

## Common Patterns

### Pattern 1: Schedule Automated Content Generation

```typescript
// src/lib/scheduler/autoSchedule.ts
import cron from 'node-cron';
import { runContentPipeline } from '../pipeline/contentPipeline';

export function scheduleContentGeneration(keywords: string[]) {
  // Run every day at 8 AM
  cron.schedule('0 8 * * *', async () => {
    for (const keyword of keywords) {
      try {
        await runContentPipeline(keyword, 'toplist', true);
        console.log(`Generated content for: ${keyword}`);
      } catch (error) {
        console.error(`Failed for ${keyword}:`, error);
      }
    }
  });
}
```

### Pattern 2: Batch Content Generation

```typescript
// src/lib/batch/batchGenerator.ts
export async function generateBatch(
  keywords: string[],
  format: 'toplist' | 'pov' | 'case-study' | 'how-to'
) {
  const results = await Promise.allSettled(
    keywords.map(keyword => 
      runContentPipeline(keyword, format, false)
    )
  );

  return results.map((result, index) => ({
    keyword: keywords[index],
    status: result.status,
    data: result.status === 'fulfilled' ? result.value : null,
    error: result.status === 'rejected' ? result.reason : null,
  }));
}
```

### Pattern 3: Content Customization with Templates

```typescript
// src/lib/templates/contentTemplates.ts
export const contentTemplates = {
  startup: {
    format: 'case-study' as const,
    tone: 'expert' as const,
    keywords: ['startup funding', 'venture capital', 'tech trends'],
  },
  howto: {
    format: 'how-to' as const,
    tone: 'friendly' as const,
    keywords: ['tutorial', 'guide', 'learn'],
  },
  news: {
    format: 'toplist' as const,
    tone: 'expert' as const,
    keywords: ['breaking news', 'latest', 'trending'],
  },
};

export async function generateFromTemplate(
  templateName: keyof typeof contentTemplates,
  customKeyword?: string
) {
  const template = contentTemplates[templateName];
  const keyword = customKeyword || template.keywords[0];
  
  const articles = await crawlRecentNews(keyword);
  const researchData = await extractInsights(articles);
  
  return generateContent(keyword, {
    format: template.format,
    language: 'vi',
    tone: template.tone,
    researchData,
  });
}
```

## Troubleshooting

### API Rate Limits

```typescript
// src/lib/utils/rateLimiter.ts
export class RateLimiter {
  private queue: Array<() => Promise<any>> = [];
  private processing = false;
  
  constructor(private delayMs: number = 1000) {}
  
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
    await new Promise(resolve => setTimeout(resolve, this.delayMs));
    
    this.processing = false;
    this.process();
  }
}

// Usage
const limiter = new RateLimiter(2000);
const result = await limiter.add(() => generateContent(topic, options));
```

### Handling Long Content Generation

```typescript
// src/lib/utils/timeout.ts
export async function withTimeout<T>(
  promise: Promise<T>,
  timeoutMs: number = 60000
): Promise<T> {
  return Promise.race([
    promise,
    new Promise<T>((_, reject) =>
      setTimeout(() => reject(new Error('Timeout')), timeoutMs)
    ),
  ]);
}

// Usage
const content = await withTimeout(
  generateContent(topic, options),
  120000 // 2 minutes
);
```

### Video Rendering Memory Issues

```bash
# Increase Node.js memory limit
NODE_OPTIONS=--max-old-space-size=4096 npm run dev
```

### Missing Environment Variables

```typescript
// src/lib/utils/validateEnv.ts
export function validateEnv() {
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

// Call at app startup
validateEnv();
```

This skill enables AI agents to help developers build and customize automated content marketing pipelines using this TypeScript/Next.js project with Claude, OpenAI, and Remotion.
