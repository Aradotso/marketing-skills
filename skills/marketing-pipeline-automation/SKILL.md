---
name: marketing-pipeline-automation
description: Automate content creation from research to video generation using AI for marketing pipelines
triggers:
  - "set up automated content pipeline with AI"
  - "create marketing content from research to video"
  - "automate content creation with Claude and OpenAI"
  - "generate videos from blog posts automatically"
  - "scrape news and create content with AI"
  - "build content automation workflow"
  - "use Remotion to render marketing videos"
  - "create multilingual content with AI pipeline"
---

# Marketing Pipeline Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the Ultimate AI Content Pipeline, a comprehensive TypeScript-based system that automates content creation from research through video generation. The pipeline integrates web scraping, AI content generation (Claude 3, OpenAI), and video rendering (Remotion) into a single workflow.

## What This Project Does

The Marketing Pipeline automates the entire content creation process:

1. **Research Phase**: Automatically scrapes and analyzes fresh content from TechCrunch, a16z, Twitter/X, and LinkedIn
2. **Content Generation**: Uses Claude/OpenAI to create articles in multiple formats (toplist, POV, case study, how-to) and languages (English/Vietnamese)
3. **Video Generation**: Renders videos and infographics from written content using Remotion
4. **Multi-Platform Export**: Outputs optimized content for Reels, TikTok, Shorts, and blog posts

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

```bash
# AI API Keys
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Optional: Social Media APIs
TWITTER_API_KEY=your_twitter_api_key
LINKEDIN_API_KEY=your_linkedin_api_key

# Next.js Configuration
NEXT_PUBLIC_APP_URL=http://localhost:3000

# Remotion Configuration
REMOTION_AWS_ACCESS_KEY_ID=your_aws_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret
```

## Key Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration modules
│   │   ├── scraper/     # Web scraping utilities
│   │   └── video/       # Remotion video rendering
│   ├── config/          # Configuration files
│   └── types/           # TypeScript type definitions
├── remotion/            # Video templates
└── public/              # Static assets
```

## Core API Usage Patterns

### 1. Research & Content Scraping

```typescript
// lib/scraper/news-scraper.ts
import axios from 'axios';

interface NewsArticle {
  title: string;
  url: string;
  source: string;
  publishedAt: Date;
  summary: string;
}

export async function scrapeLatestNews(
  topic: string,
  sources: string[] = ['techcrunch', 'a16z']
): Promise<NewsArticle[]> {
  const articles: NewsArticle[] = [];
  
  for (const source of sources) {
    const response = await axios.get(
      `https://api.rapidapi.com/news/${source}`,
      {
        params: { q: topic, language: 'en' },
        headers: {
          'X-RapidAPI-Key': process.env.RAPIDAPI_KEY!,
          'X-RapidAPI-Host': 'news-api.p.rapidapi.com'
        }
      }
    );
    
    articles.push(...response.data.articles);
  }
  
  return articles;
}

// Extract insights from scraped content
export async function extractInsights(
  articles: NewsArticle[]
): Promise<string> {
  const combinedContent = articles
    .map(a => `${a.title}\n${a.summary}`)
    .join('\n\n');
  
  return combinedContent;
}
```

### 2. AI Content Generation with Claude

```typescript
// lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY!,
});

interface ContentRequest {
  topic: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'professional' | 'friendly' | 'humorous';
  researchData: string;
}

export async function generateContent(
  request: ContentRequest
): Promise<string> {
  const systemPrompt = `You are an expert content writer specializing in ${request.format} articles. 
Write in a ${request.tone} tone for ${request.language === 'en' ? 'English' : 'Vietnamese'} audience.
Use the provided research data to create data-backed, insightful content.`;

  const userPrompt = `Create a ${request.format} article about "${request.topic}".

Research Data:
${request.researchData}

Requirements:
- Use specific data points and statistics
- Include 3-5 main sections
- Add actionable insights
- Optimize for engagement`;

  const message = await anthropic.messages.create({
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

### 3. AI Content Generation with OpenAI

```typescript
// lib/ai/openai-generator.ts
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY!,
});

export async function generateContentWithGPT(
  request: ContentRequest
): Promise<string> {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `Expert content writer for ${request.format} format`,
      },
      {
        role: 'user',
        content: `Write about ${request.topic} using this research:\n${request.researchData}`,
      },
    ],
    temperature: 0.7,
    max_tokens: 3000,
  });

  return completion.choices[0].message.content || '';
}
```

### 4. Video Generation with Remotion

```typescript
// lib/video/video-renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  title: string;
  content: string[];
  duration: number;
  format: 'reel' | 'tiktok' | 'youtube-short';
}

const formatSpecs = {
  reel: { width: 1080, height: 1920, fps: 30 },
  tiktok: { width: 1080, height: 1920, fps: 30 },
  'youtube-short': { width: 1080, height: 1920, fps: 30 },
};

export async function renderContentVideo(
  config: VideoConfig,
  outputPath: string
): Promise<string> {
  // Bundle Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  // Get composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title: config.title,
      content: config.content,
    },
  });

  const specs = formatSpecs[config.format];

  // Render video
  await renderMedia({
    composition: {
      ...composition,
      width: specs.width,
      height: specs.height,
      fps: specs.fps,
      durationInFrames: config.duration * specs.fps,
    },
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: {
      title: config.title,
      content: config.content,
    },
  });

  return outputPath;
}
```

### 5. Remotion Video Component

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, Sequence, useCurrentFrame, useVideoConfig } from 'remotion';
import React from 'react';

interface ContentVideoProps {
  title: string;
  content: string[];
}

export const ContentVideo: React.FC<ContentVideoProps> = ({ title, content }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  const opacity = Math.min(1, frame / 30);

  return (
    <AbsoluteFill style={{ backgroundColor: '#000' }}>
      <Sequence from={0} durationInFrames={fps * 2}>
        <AbsoluteFill
          style={{
            justifyContent: 'center',
            alignItems: 'center',
            opacity,
          }}
        >
          <h1 style={{ color: 'white', fontSize: 60, textAlign: 'center' }}>
            {title}
          </h1>
        </AbsoluteFill>
      </Sequence>

      {content.map((text, index) => (
        <Sequence
          key={index}
          from={fps * (2 + index * 3)}
          durationInFrames={fps * 3}
        >
          <AbsoluteFill
            style={{
              justifyContent: 'center',
              alignItems: 'center',
              padding: 40,
            }}
          >
            <p style={{ color: 'white', fontSize: 40, textAlign: 'center' }}>
              {text}
            </p>
          </AbsoluteFill>
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};
```

## Complete Workflow Integration

```typescript
// lib/pipeline/content-pipeline.ts
import { scrapeLatestNews, extractInsights } from '../scraper/news-scraper';
import { generateContent } from '../ai/content-generator';
import { renderContentVideo } from '../video/video-renderer';

export async function runContentPipeline(
  topic: string,
  options: {
    format: 'toplist' | 'pov' | 'case-study' | 'how-to';
    languages: ('en' | 'vi')[];
    generateVideo: boolean;
  }
) {
  console.log(`🔍 Step 1: Researching "${topic}"...`);
  const articles = await scrapeLatestNews(topic);
  const researchData = await extractInsights(articles);

  console.log(`✍️ Step 2: Generating content...`);
  const contents: Record<string, string> = {};

  for (const language of options.languages) {
    const content = await generateContent({
      topic,
      format: options.format,
      language,
      tone: 'professional',
      researchData,
    });
    contents[language] = content;
  }

  if (options.generateVideo) {
    console.log(`🎬 Step 3: Rendering video...`);
    
    // Extract key points for video
    const keyPoints = contents['en']
      .split('\n')
      .filter(line => line.trim().length > 0)
      .slice(0, 5);

    const videoPath = await renderContentVideo(
      {
        title: topic,
        content: keyPoints,
        duration: 30,
        format: 'reel',
      },
      `./output/video-${Date.now()}.mp4`
    );

    console.log(`✅ Video rendered: ${videoPath}`);
  }

  console.log(`✅ Pipeline complete!`);
  return contents;
}
```

## Next.js API Route Example

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline/content-pipeline';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const { topic, format, languages, generateVideo } = body;

    if (!topic || !format) {
      return NextResponse.json(
        { error: 'Missing required fields' },
        { status: 400 }
      );
    }

    const contents = await runContentPipeline(topic, {
      format,
      languages: languages || ['en'],
      generateVideo: generateVideo || false,
    });

    return NextResponse.json({
      success: true,
      contents,
    });
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { error: 'Pipeline execution failed' },
      { status: 500 }
    );
  }
}
```

## Running the Application

```bash
# Development mode
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Render Remotion video (standalone)
npx remotion render ContentVideo output.mp4
```

## Common Patterns

### Multilingual Content Generation

```typescript
async function generateMultilingualContent(topic: string) {
  const languages = ['en', 'vi'];
  const results = await Promise.all(
    languages.map(lang =>
      generateContent({
        topic,
        format: 'toplist',
        language: lang as 'en' | 'vi',
        tone: 'professional',
        researchData: '',
      })
    )
  );

  return {
    en: results[0],
    vi: results[1],
  };
}
```

### Batch Video Generation

```typescript
async function batchGenerateVideos(
  contents: Array<{ title: string; text: string }>
) {
  const videos = [];

  for (const content of contents) {
    const videoPath = await renderContentVideo(
      {
        title: content.title,
        content: content.text.split('\n').slice(0, 5),
        duration: 30,
        format: 'reel',
      },
      `./output/${content.title.replace(/\s+/g, '-')}.mp4`
    );

    videos.push(videoPath);
  }

  return videos;
}
```

## Troubleshooting

### API Rate Limits

If you encounter rate limits:

```typescript
// Add exponential backoff
async function retryWithBackoff<T>(
  fn: () => Promise<T>,
  maxRetries = 3
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      await new Promise(resolve => 
        setTimeout(resolve, Math.pow(2, i) * 1000)
      );
    }
  }
  throw new Error('Max retries reached');
}
```

### Video Rendering Memory Issues

For large video renders, increase Node.js memory:

```bash
NODE_OPTIONS="--max-old-space-size=4096" npm run dev
```

### Missing Environment Variables

Validate environment setup:

```typescript
function validateEnv() {
  const required = [
    'ANTHROPIC_API_KEY',
    'OPENAI_API_KEY',
    'RAPIDAPI_KEY',
  ];

  const missing = required.filter(key => !process.env[key]);

  if (missing.length > 0) {
    throw new Error(`Missing environment variables: ${missing.join(', ')}`);
  }
}
```

### Remotion Bundle Errors

Clear Remotion cache if you encounter bundling issues:

```bash
rm -rf node_modules/.cache/remotion
```

## Performance Optimization

```typescript
// Cache research data to avoid repeated scraping
import { Redis } from '@upstash/redis';

const redis = new Redis({
  url: process.env.UPSTASH_REDIS_URL!,
  token: process.env.UPSTASH_REDIS_TOKEN!,
});

async function getCachedResearch(topic: string): Promise<string | null> {
  const cached = await redis.get(`research:${topic}`);
  return cached as string | null;
}

async function cacheResearch(topic: string, data: string) {
  await redis.set(`research:${topic}`, data, { ex: 86400 }); // 24h
}
```
