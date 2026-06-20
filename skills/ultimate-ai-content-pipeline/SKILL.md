---
name: ultimate-ai-content-pipeline
description: Vietnamese-focused automated content pipeline from research to video generation using Claude/OpenAI and Remotion
triggers:
  - how do I automate content creation with AI research
  - set up automated marketing content pipeline
  - generate video content from text automatically
  - create multilingual content with Claude and OpenAI
  - build AI-powered content workflow with Remotion
  - scrape news and generate social media posts
  - automate Vietnamese and English content creation
  - create TikTok and Reels videos from articles
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

A comprehensive TypeScript-based automated content creation system that handles research (web scraping), content generation (Claude/OpenAI), and video rendering (Remotion). Designed for Vietnamese and English content creators to automate their entire workflow from keyword to published video.

## What It Does

This pipeline automates:
1. **Research**: Auto-scrapes news from TechCrunch, a16z, Twitter/X, LinkedIn
2. **Content Generation**: Creates articles in multiple formats (toplist, POV, case study, how-to) using Claude 3 or OpenAI
3. **Multilingual Output**: Generates both English and Vietnamese versions
4. **Video Rendering**: Converts text content to videos/infographics using Remotion
5. **Platform Optimization**: Outputs video formats for Reels, TikTok, YouTube Shorts

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

## Environment Configuration

Create a `.env.local` file in the project root:

```bash
# AI Provider Keys
ANTHROPIC_API_KEY=your_claude_api_key_here
OPENAI_API_KEY=your_openai_api_key_here

# Research & Scraping
RAPIDAPI_KEY=your_rapidapi_key_here

# Optional: Database
DATABASE_URL=postgresql://user:password@localhost:5432/content_db

# Optional: Video Rendering
REMOTION_AWS_ACCESS_KEY=your_aws_key_here
REMOTION_AWS_SECRET_KEY=your_aws_secret_here

# Application
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

## Project Structure

```
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integrations (Claude, OpenAI)
│   │   ├── scrapers/    # Web scraping modules
│   │   └── video/       # Remotion video generation
│   ├── remotion/        # Remotion compositions
│   └── utils/           # Helper functions
├── public/              # Static assets
└── package.json
```

## Key API Usage

### 1. Content Research & Scraping

```typescript
// src/lib/scrapers/news-scraper.ts
import axios from 'axios';

interface NewsArticle {
  title: string;
  url: string;
  publishedAt: string;
  source: string;
  summary?: string;
}

export async function scrapeRecentNews(
  keyword: string,
  sources: string[] = ['techcrunch', 'a16z']
): Promise<NewsArticle[]> {
  const articles: NewsArticle[] = [];
  
  for (const source of sources) {
    try {
      const response = await axios.get(
        `https://news-scraper.p.rapidapi.com/v1/search`,
        {
          params: { q: keyword, source, hours: 24 },
          headers: {
            'X-RapidAPI-Key': process.env.RAPIDAPI_KEY!,
            'X-RapidAPI-Host': 'news-scraper.p.rapidapi.com'
          }
        }
      );
      
      articles.push(...response.data.articles);
    } catch (error) {
      console.error(`Failed to scrape ${source}:`, error);
    }
  }
  
  return articles;
}
```

### 2. AI Content Generation with Claude

```typescript
// src/lib/ai/claude-generator.ts
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY!
});

interface ContentRequest {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  research: string; // Research data as context
}

export async function generateContent(
  request: ContentRequest
): Promise<string> {
  const systemPrompt = buildSystemPrompt(request.format, request.tone);
  const userPrompt = buildUserPrompt(request.keyword, request.research, request.language);
  
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    system: systemPrompt,
    messages: [
      {
        role: 'user',
        content: userPrompt
      }
    ]
  });
  
  return message.content[0].type === 'text' 
    ? message.content[0].text 
    : '';
}

function buildSystemPrompt(format: string, tone: string): string {
  const basePrompt = `You are an expert content writer specializing in ${format} articles.`;
  const toneMap = {
    expert: 'Use professional, authoritative language with data-backed insights.',
    friendly: 'Write in a conversational, approachable tone.',
    humorous: 'Inject wit and humor while maintaining value.'
  };
  
  return `${basePrompt} ${toneMap[tone as keyof typeof toneMap]}`;
}

function buildUserPrompt(keyword: string, research: string, language: string): string {
  const langInstruction = language === 'vi' 
    ? 'Write in Vietnamese.' 
    : 'Write in English.';
    
  return `
Topic: ${keyword}

Research Data:
${research}

${langInstruction}

Create a comprehensive article that:
1. Uses the latest research data provided
2. Includes specific examples and statistics
3. Provides actionable insights
4. Is optimized for social media sharing
`;
}
```

### 3. OpenAI Alternative

```typescript
// src/lib/ai/openai-generator.ts
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY!
});

export async function generateContentOpenAI(
  request: ContentRequest
): Promise<string> {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: buildSystemPrompt(request.format, request.tone)
      },
      {
        role: 'user',
        content: buildUserPrompt(request.keyword, request.research, request.language)
      }
    ],
    temperature: 0.7,
    max_tokens: 4000
  });
  
  return completion.choices[0].message.content || '';
}
```

### 4. Video Generation with Remotion

```typescript
// src/lib/video/render-video.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  content: string;
  title: string;
  format: 'reels' | 'tiktok' | 'shorts'; // 9:16 aspect ratio
}

export async function renderContentVideo(
  config: VideoConfig
): Promise<string> {
  // Bundle Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./src/remotion/index.ts'),
    webpackOverride: (config) => config
  });
  
  // Select composition based on format
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: `content-${config.format}`,
    inputProps: {
      title: config.title,
      content: config.content
    }
  });
  
  // Render video
  const outputLocation = path.resolve(`./output/${Date.now()}.mp4`);
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: {
      title: config.title,
      content: config.content
    }
  });
  
  return outputLocation;
}
```

### 5. Remotion Composition Example

```typescript
// src/remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig, Sequence } from 'remotion';
import React from 'react';

interface ContentVideoProps {
  title: string;
  content: string;
}

export const ContentVideo: React.FC<ContentVideoProps> = ({ title, content }) => {
  const frame = useCurrentFrame();
  const { fps, durationInFrames } = useVideoConfig();
  
  const opacity = Math.min(1, frame / 30);
  const contentPoints = content.split('\n').filter(p => p.trim());
  
  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a1a' }}>
      <Sequence from={0} durationInFrames={90}>
        <AbsoluteFill
          style={{
            justifyContent: 'center',
            alignItems: 'center',
            opacity
          }}
        >
          <h1
            style={{
              color: 'white',
              fontSize: 80,
              textAlign: 'center',
              padding: '0 100px'
            }}
          >
            {title}
          </h1>
        </AbsoluteFill>
      </Sequence>
      
      {contentPoints.map((point, index) => (
        <Sequence
          key={index}
          from={90 + index * 60}
          durationInFrames={60}
        >
          <AbsoluteFill
            style={{
              justifyContent: 'center',
              alignItems: 'center',
              padding: '0 100px'
            }}
          >
            <p
              style={{
                color: 'white',
                fontSize: 50,
                textAlign: 'center',
                lineHeight: 1.5
              }}
            >
              {point}
            </p>
          </AbsoluteFill>
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};
```

### 6. Complete Pipeline Orchestration

```typescript
// src/lib/pipeline/content-pipeline.ts
import { scrapeRecentNews } from '../scrapers/news-scraper';
import { generateContent } from '../ai/claude-generator';
import { renderContentVideo } from '../video/render-video';

interface PipelineConfig {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  languages: ('en' | 'vi')[];
  tone: 'expert' | 'friendly' | 'humorous';
  generateVideo: boolean;
  videoFormat?: 'reels' | 'tiktok' | 'shorts';
}

export async function runContentPipeline(config: PipelineConfig) {
  console.log(`🚀 Starting pipeline for: ${config.keyword}`);
  
  // Step 1: Research
  console.log('📡 Scraping recent news...');
  const articles = await scrapeRecentNews(config.keyword);
  const research = articles
    .map(a => `${a.title}\n${a.summary || ''}`)
    .join('\n\n');
  
  // Step 2: Generate content for each language
  console.log('🧠 Generating content...');
  const contents = await Promise.all(
    config.languages.map(async (language) => {
      const content = await generateContent({
        keyword: config.keyword,
        format: config.format,
        language,
        tone: config.tone,
        research
      });
      
      return { language, content };
    })
  );
  
  // Step 3: Generate videos if requested
  let videos: string[] = [];
  if (config.generateVideo && config.videoFormat) {
    console.log('🎬 Rendering videos...');
    videos = await Promise.all(
      contents.map(async ({ language, content }) => {
        return renderContentVideo({
          content,
          title: `${config.keyword} (${language.toUpperCase()})`,
          format: config.videoFormat!
        });
      })
    );
  }
  
  console.log('✅ Pipeline complete!');
  return { contents, videos, research };
}
```

## Usage in Next.js API Route

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline/content-pipeline';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    
    const result = await runContentPipeline({
      keyword: body.keyword,
      format: body.format || 'how-to',
      languages: body.languages || ['en', 'vi'],
      tone: body.tone || 'friendly',
      generateVideo: body.generateVideo || false,
      videoFormat: body.videoFormat || 'reels'
    });
    
    return NextResponse.json({
      success: true,
      data: result
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

## CLI Usage (if applicable)

```bash
# Run development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Render video only
npm run remotion:render

# Preview Remotion composition
npm run remotion:preview
```

## Common Patterns

### Pattern 1: Batch Content Generation

```typescript
async function generateBatchContent(keywords: string[]) {
  const results = [];
  
  for (const keyword of keywords) {
    const result = await runContentPipeline({
      keyword,
      format: 'toplist',
      languages: ['en', 'vi'],
      tone: 'expert',
      generateVideo: true,
      videoFormat: 'reels'
    });
    
    results.push({ keyword, ...result });
    
    // Rate limiting
    await new Promise(resolve => setTimeout(resolve, 2000));
  }
  
  return results;
}
```

### Pattern 2: Custom Tone Configuration

```typescript
const customTones = {
  professional: 'Use formal business language with industry terminology.',
  storytelling: 'Write as a narrative with compelling storytelling elements.',
  educational: 'Break down complex topics into easy-to-understand lessons.'
};

// Extend the system prompt
function buildCustomSystemPrompt(format: string, customTone: string): string {
  return `You are an expert content writer. ${customTone} Format: ${format}`;
}
```

### Pattern 3: Save Content to Database

```typescript
import { PrismaClient } from '@prisma/client';

const prisma = new PrismaClient();

async function saveGeneratedContent(
  keyword: string,
  content: string,
  language: string,
  videoPath?: string
) {
  return await prisma.content.create({
    data: {
      keyword,
      content,
      language,
      videoPath,
      createdAt: new Date()
    }
  });
}
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement exponential backoff
async function retryWithBackoff<T>(
  fn: () => Promise<T>,
  maxRetries = 3
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error: any) {
      if (error?.status === 429 && i < maxRetries - 1) {
        const delay = Math.pow(2, i) * 1000;
        await new Promise(resolve => setTimeout(resolve, delay));
        continue;
      }
      throw error;
    }
  }
  throw new Error('Max retries reached');
}
```

### Video Rendering Memory Issues

```typescript
// Use chunked rendering for long videos
import { renderFrames } from '@remotion/renderer';

async function renderLargeVideo(config: VideoConfig) {
  // Split content into chunks
  const chunks = splitContentIntoChunks(config.content, 5);
  
  const videoChunks = await Promise.all(
    chunks.map((chunk, index) =>
      renderContentVideo({
        ...config,
        content: chunk
      })
    )
  );
  
  // Merge video chunks (using ffmpeg or similar)
  return mergeVideoFiles(videoChunks);
}
```

### Missing Environment Variables

```typescript
// src/lib/config/validate-env.ts
export function validateEnv() {
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
```

### Scraping Failures

```typescript
// Implement fallback sources
async function scrapeWithFallback(keyword: string) {
  const primarySources = ['techcrunch', 'a16z'];
  const fallbackSources = ['hackernews', 'reddit'];
  
  let articles = await scrapeRecentNews(keyword, primarySources);
  
  if (articles.length === 0) {
    console.warn('Primary sources failed, trying fallback...');
    articles = await scrapeRecentNews(keyword, fallbackSources);
  }
  
  return articles;
}
```

## Best Practices

1. **Always validate environment variables** at startup
2. **Implement rate limiting** between API calls (2-3 seconds minimum)
3. **Cache research data** for 24 hours to reduce scraping
4. **Use streaming responses** for long content generation
5. **Monitor API costs** by logging token usage
6. **Version control video compositions** separately from content
7. **Test video rendering locally** before deploying to production
