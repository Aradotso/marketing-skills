---
name: marketing-pipeline-share-automation
description: Automated AI content pipeline for research, scriptwriting, and video generation using Claude/OpenAI and Remotion
triggers:
  - automate content creation with AI pipeline
  - generate marketing videos from text automatically
  - scrape news and create content with Claude
  - build automated content research system
  - create video content pipeline with Remotion
  - set up AI marketing automation workflow
  - generate multilingual content with OpenAI
  - automate social media content generation
---

# Marketing Pipeline Share Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the Ultimate AI Content Pipeline, an end-to-end content automation system that handles research, scriptwriting, and video generation. The system crawls news sources, generates multilingual content using Claude/OpenAI, and automatically renders videos using Remotion.

## What It Does

The Marketing Pipeline Share project is a TypeScript-based Next.js application that automates the entire content creation workflow:

1. **Auto-Research**: Crawls news from TechCrunch, a16z, Twitter/X, LinkedIn within 24h
2. **AI Content Generation**: Creates content in multiple formats (Toplist, POV, Case Study, How-to) using Claude 3 or OpenAI
3. **Multilingual Output**: Generates content in English and Vietnamese simultaneously
4. **Video Rendering**: Automatically creates infographics and short-form videos using Remotion
5. **Multi-Platform**: Optimizes video output for Reels, TikTok, Shorts

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

### Required Environment Variables

```bash
# AI Services
ANTHROPIC_API_KEY=your_claude_api_key_here
OPENAI_API_KEY=your_openai_api_key_here

# Research/Scraping APIs
RAPIDAPI_KEY=your_rapidapi_key_here

# Database (if applicable)
DATABASE_URL=your_database_url_here

# Next.js
NEXT_PUBLIC_API_URL=http://localhost:3000
```

## Key Configuration

### API Configuration

```typescript
// lib/config/ai.config.ts
export const aiConfig = {
  claude: {
    model: 'claude-3-opus-20240229',
    maxTokens: 4096,
    temperature: 0.7
  },
  openai: {
    model: 'gpt-4-turbo-preview',
    maxTokens: 4096,
    temperature: 0.7
  }
};

export const researchConfig = {
  sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
  timeRange: '24h',
  maxArticles: 50
};
```

### Remotion Video Configuration

```typescript
// remotion.config.ts
import { Config } from '@remotion/cli/config';

Config.setVideoImageFormat('jpeg');
Config.setOverwriteOutput(true);
Config.setConcurrency(4);
Config.setCodec('h264');

export const videoConfig = {
  fps: 30,
  durationInFrames: 900, // 30 seconds at 30fps
  width: 1080,
  height: 1920, // Vertical for Reels/TikTok/Shorts
};
```

## Core Usage Patterns

### 1. Research Content from Keywords

```typescript
// lib/research/scraper.ts
import axios from 'axios';

interface ResearchResult {
  title: string;
  summary: string;
  url: string;
  source: string;
  publishedAt: Date;
}

export async function researchKeyword(
  keyword: string
): Promise<ResearchResult[]> {
  const rapidApiKey = process.env.RAPIDAPI_KEY;
  
  const options = {
    method: 'GET',
    url: 'https://news-api14.p.rapidapi.com/search',
    params: {
      q: keyword,
      language: 'en',
      sortBy: 'publishedAt',
      from: new Date(Date.now() - 24 * 60 * 60 * 1000).toISOString()
    },
    headers: {
      'X-RapidAPI-Key': rapidApiKey,
      'X-RapidAPI-Host': 'news-api14.p.rapidapi.com'
    }
  };

  try {
    const response = await axios.request(options);
    return response.data.articles.map((article: any) => ({
      title: article.title,
      summary: article.description,
      url: article.url,
      source: article.source.name,
      publishedAt: new Date(article.publishedAt)
    }));
  } catch (error) {
    console.error('Research error:', error);
    throw new Error('Failed to fetch research data');
  }
}
```

### 2. Generate Content with Claude

```typescript
// lib/ai/claude.ts
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY
});

export interface ContentRequest {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  tone: 'expert' | 'friendly' | 'humorous';
  language: 'en' | 'vi';
  researchData: string;
}

export async function generateContent(
  request: ContentRequest
): Promise<string> {
  const prompt = `
You are an expert content writer. Create a ${request.format} article about "${request.keyword}".

Research Data:
${request.researchData}

Requirements:
- Format: ${request.format}
- Tone: ${request.tone}
- Language: ${request.language}
- Include data-backed insights
- Make it engaging and actionable
- Length: 1000-1500 words

Generate the complete article now:
`;

  const message = await anthropic.messages.create({
    model: 'claude-3-opus-20240229',
    max_tokens: 4096,
    temperature: 0.7,
    messages: [
      {
        role: 'user',
        content: prompt
      }
    ]
  });

  return message.content[0].type === 'text' 
    ? message.content[0].text 
    : '';
}
```

### 3. Generate Content with OpenAI

```typescript
// lib/ai/openai.ts
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY
});

export async function generateContentOpenAI(
  request: ContentRequest
): Promise<string> {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are an expert ${request.tone} content writer specializing in ${request.format} articles.`
      },
      {
        role: 'user',
        content: `Create a ${request.format} article about "${request.keyword}" in ${request.language}. Use this research: ${request.researchData}`
      }
    ],
    temperature: 0.7,
    max_tokens: 4096
  });

  return completion.choices[0].message.content || '';
}
```

### 4. Render Video with Remotion

```typescript
// lib/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

export interface VideoContent {
  title: string;
  keyPoints: string[];
  images?: string[];
}

export async function renderContentVideo(
  content: VideoContent,
  outputPath: string
): Promise<string> {
  const compositionId = 'ContentVideo';
  const bundleLocation = await bundle(
    path.join(process.cwd(), 'remotion/index.ts')
  );

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: content
  });

  const outputLocation = path.join(
    process.cwd(),
    'public',
    'videos',
    outputPath
  );

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: content
  });

  return outputLocation;
}
```

### 5. Remotion Video Component

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, Sequence, useCurrentFrame } from 'remotion';
import { VideoContent } from '../lib/video/renderer';

export const ContentVideo: React.FC<VideoContent> = ({
  title,
  keyPoints,
  images = []
}) => {
  const frame = useCurrentFrame();

  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a1a' }}>
      {/* Title Sequence */}
      <Sequence from={0} durationInFrames={90}>
        <AbsoluteFill
          style={{
            justifyContent: 'center',
            alignItems: 'center',
            padding: 40
          }}
        >
          <h1
            style={{
              color: 'white',
              fontSize: 60,
              fontWeight: 'bold',
              textAlign: 'center',
              opacity: Math.min(1, frame / 30)
            }}
          >
            {title}
          </h1>
        </AbsoluteFill>
      </Sequence>

      {/* Key Points */}
      {keyPoints.map((point, index) => (
        <Sequence
          key={index}
          from={90 + index * 90}
          durationInFrames={90}
        >
          <AbsoluteFill
            style={{
              justifyContent: 'center',
              alignItems: 'center',
              padding: 60
            }}
          >
            <div
              style={{
                backgroundColor: 'rgba(255, 255, 255, 0.1)',
                borderRadius: 20,
                padding: 40,
                maxWidth: '80%'
              }}
            >
              <p
                style={{
                  color: 'white',
                  fontSize: 40,
                  lineHeight: 1.5
                }}
              >
                {point}
              </p>
            </div>
          </AbsoluteFill>
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};
```

## Complete Pipeline Example

```typescript
// app/api/generate-pipeline/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { researchKeyword } from '@/lib/research/scraper';
import { generateContent } from '@/lib/ai/claude';
import { renderContentVideo } from '@/lib/video/renderer';

export async function POST(request: NextRequest) {
  try {
    const { keyword, format, tone, language } = await request.json();

    // Step 1: Research
    console.log('Starting research...');
    const researchResults = await researchKeyword(keyword);
    const researchData = researchResults
      .map(r => `${r.title}\n${r.summary}`)
      .join('\n\n');

    // Step 2: Generate Content
    console.log('Generating content...');
    const content = await generateContent({
      keyword,
      format,
      tone,
      language,
      researchData
    });

    // Step 3: Extract key points for video
    const lines = content.split('\n').filter(line => line.trim());
    const keyPoints = lines
      .filter(line => line.match(/^[-•*\d]/))
      .slice(0, 5)
      .map(line => line.replace(/^[-•*\d.)\s]+/, ''));

    // Step 4: Render Video
    console.log('Rendering video...');
    const videoPath = await renderContentVideo(
      {
        title: keyword,
        keyPoints
      },
      `${Date.now()}-${keyword.replace(/\s+/g, '-')}.mp4`
    );

    return NextResponse.json({
      success: true,
      content,
      videoPath: videoPath.replace(process.cwd() + '/public', '')
    });
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { success: false, error: 'Pipeline failed' },
      { status: 500 }
    );
  }
}
```

## CLI Commands

### Development

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Start production server
npm run start
```

### Remotion Commands

```bash
# Preview Remotion compositions
npm run remotion:preview

# Render specific video
npx remotion render remotion/index.ts ContentVideo output.mp4

# Render with custom props
npx remotion render remotion/index.ts ContentVideo output.mp4 \
  --props='{"title":"AI Marketing","keyPoints":["Point 1","Point 2"]}'
```

## Common Patterns

### Batch Content Generation

```typescript
// scripts/batch-generate.ts
async function batchGenerate(keywords: string[]) {
  const results = [];
  
  for (const keyword of keywords) {
    console.log(`Processing: ${keyword}`);
    
    const research = await researchKeyword(keyword);
    const content = await generateContent({
      keyword,
      format: 'toplist',
      tone: 'expert',
      language: 'en',
      researchData: JSON.stringify(research)
    });
    
    results.push({ keyword, content });
    
    // Rate limiting
    await new Promise(resolve => setTimeout(resolve, 2000));
  }
  
  return results;
}
```

### Multilingual Content Generation

```typescript
async function generateMultilingual(keyword: string) {
  const research = await researchKeyword(keyword);
  const researchData = JSON.stringify(research);
  
  const [englishContent, vietnameseContent] = await Promise.all([
    generateContent({
      keyword,
      format: 'how-to',
      tone: 'friendly',
      language: 'en',
      researchData
    }),
    generateContent({
      keyword,
      format: 'how-to',
      tone: 'friendly',
      language: 'vi',
      researchData
    })
  ]);
  
  return { en: englishContent, vi: vietnameseContent };
}
```

## Troubleshooting

### API Rate Limiting

```typescript
// lib/utils/rate-limiter.ts
export class RateLimiter {
  private queue: Array<() => Promise<any>> = [];
  private processing = false;
  private delay: number;

  constructor(delayMs: number = 2000) {
    this.delay = delayMs;
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
      this.process();
    });
  }

  private async process() {
    if (this.processing || this.queue.length === 0) return;
    
    this.processing = true;
    const fn = this.queue.shift()!;
    await fn();
    await new Promise(resolve => setTimeout(resolve, this.delay));
    this.processing = false;
    this.process();
  }
}

// Usage
const limiter = new RateLimiter(2000);
const result = await limiter.add(() => generateContent(request));
```

### Video Rendering Memory Issues

```typescript
// Increase Node.js memory limit in package.json
{
  "scripts": {
    "remotion:render": "NODE_OPTIONS='--max-old-space-size=4096' remotion render"
  }
}
```

### Error Handling

```typescript
// lib/utils/error-handler.ts
export async function withRetry<T>(
  fn: () => Promise<T>,
  maxRetries: number = 3
): Promise<T> {
  let lastError: Error;
  
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      lastError = error as Error;
      console.log(`Attempt ${i + 1} failed, retrying...`);
      await new Promise(resolve => 
        setTimeout(resolve, 1000 * Math.pow(2, i))
      );
    }
  }
  
  throw lastError!;
}
```

This skill provides comprehensive guidance for AI agents to implement automated content pipelines with research scraping, AI content generation, and video rendering capabilities.
