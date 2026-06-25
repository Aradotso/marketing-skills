---
name: marketing-pipeline-ai-content-automation
description: Automated AI content pipeline for research, scriptwriting, and video generation using Claude, OpenAI, and Remotion
triggers:
  - how do I automate content creation from research to video
  - set up AI content pipeline with Claude and OpenAI
  - generate automated blog posts and videos from keywords
  - crawl news sources and create content automatically
  - use Remotion to render videos from AI content
  - configure multi-language content generation pipeline
  - integrate Claude and OpenAI for content automation
  - build end-to-end marketing content workflow
---

# Marketing Pipeline AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the Ultimate AI Content Pipeline, a TypeScript-based system that automates the entire content creation workflow from research and scriptwriting to video generation. The pipeline integrates Claude 3, OpenAI, web scraping, and Remotion for video rendering.

## What This Project Does

The Marketing Pipeline Share is an end-to-end content automation system that:

- **Auto-scans and researches** trending topics from TechCrunch, a16z, Twitter/X, LinkedIn
- **Generates multi-format content** using Claude/OpenAI (articles, case studies, how-tos, POV pieces)
- **Supports multi-language output** (English and Vietnamese by default)
- **Renders videos automatically** using Remotion for social media (Reels, TikTok, Shorts)
- **Customizes tone and style** based on target audience

## Installation

### Prerequisites

```bash
# Node.js 18+ required
node --version

# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share
```

### Install Dependencies

```bash
# Install packages
npm install
# or
yarn install
# or
pnpm install
```

### Environment Configuration

Create a `.env.local` file in the root directory:

```bash
# AI APIs
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# Research APIs (RapidAPI)
RAPIDAPI_KEY=your_rapidapi_key

# Database (if using)
DATABASE_URL=your_database_connection

# Remotion Configuration
REMOTION_LAMBDA_REGION=us-east-1
REMOTION_LAMBDA_FUNCTION_NAME=your_function_name

# Application
NEXT_PUBLIC_API_URL=http://localhost:3000
```

### Run Development Server

```bash
npm run dev
# or
yarn dev
# or
pnpm dev
```

Access the application at `http://localhost:3000`

## Core Architecture

### Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app router
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── scraper/     # Web scraping utilities
│   │   ├── video/       # Remotion video generation
│   │   └── utils/       # Helper functions
│   └── types/           # TypeScript definitions
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Key APIs and Usage

### 1. Content Research and Scraping

```typescript
// src/lib/scraper/news-scraper.ts
import axios from 'axios';

interface NewsArticle {
  title: string;
  url: string;
  publishedAt: string;
  source: string;
  summary: string;
}

export async function scrapeNewsForKeyword(
  keyword: string,
  timeframe: '24h' | '7d' = '24h'
): Promise<NewsArticle[]> {
  const rapidApiKey = process.env.RAPIDAPI_KEY;
  
  const options = {
    method: 'GET',
    url: 'https://news-api14.p.rapidapi.com/search',
    params: {
      query: keyword,
      language: 'en',
      time: timeframe
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
      url: article.url,
      publishedAt: article.publishedAt,
      source: article.source.name,
      summary: article.description || ''
    }));
  } catch (error) {
    console.error('Error scraping news:', error);
    return [];
  }
}

// Usage example
const articles = await scrapeNewsForKeyword('AI marketing', '24h');
```

### 2. AI Content Generation with Claude

```typescript
// src/lib/ai/claude-generator.ts
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

interface ContentGenerationOptions {
  keyword: string;
  format: 'article' | 'case-study' | 'how-to' | 'toplist' | 'pov';
  tone: 'professional' | 'friendly' | 'humorous';
  language: 'en' | 'vi';
  researchData: string;
}

export async function generateContentWithClaude(
  options: ContentGenerationOptions
): Promise<string> {
  const { keyword, format, tone, language, researchData } = options;

  const systemPrompt = `You are an expert content writer specializing in ${format} format.
Write in ${language === 'en' ? 'English' : 'Vietnamese'} with a ${tone} tone.`;

  const userPrompt = `Create a comprehensive ${format} about "${keyword}" using this research:

${researchData}

Requirements:
- Engaging headline
- Clear structure with sections
- Data-driven insights
- Actionable takeaways
- SEO-optimized
- ${language === 'en' ? '1500-2000 words' : '1200-1500 từ'}`;

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

### 3. OpenAI Alternative Implementation

```typescript
// src/lib/ai/openai-generator.ts
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

export async function generateContentWithOpenAI(
  options: ContentGenerationOptions
): Promise<string> {
  const { keyword, format, tone, language, researchData } = options;

  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are an expert content writer specializing in ${format} format.
Write in ${language === 'en' ? 'English' : 'Vietnamese'} with a ${tone} tone.`,
      },
      {
        role: 'user',
        content: `Create a comprehensive ${format} about "${keyword}" using this research:\n\n${researchData}`,
      },
    ],
    temperature: 0.7,
    max_tokens: 3000,
  });

  return completion.choices[0]?.message?.content || '';
}
```

### 4. Video Generation with Remotion

```typescript
// src/lib/video/video-generator.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  title: string;
  keyPoints: string[];
  duration: number; // in frames
  platform: 'reels' | 'tiktok' | 'shorts';
}

export async function generateVideo(config: VideoConfig): Promise<string> {
  const { title, keyPoints, duration, platform } = config;

  // Platform dimensions
  const dimensions = {
    reels: { width: 1080, height: 1920 },
    tiktok: { width: 1080, height: 1920 },
    shorts: { width: 1080, height: 1920 },
  };

  const { width, height } = dimensions[platform];

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
      title,
      keyPoints,
    },
  });

  // Render video
  const outputLocation = path.resolve(`./public/videos/${Date.now()}.mp4`);

  await renderMedia({
    composition: {
      ...composition,
      width,
      height,
      durationInFrames: duration,
      fps: 30,
    },
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: {
      title,
      keyPoints,
    },
  });

  return outputLocation;
}
```

### 5. Remotion Video Template

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, Sequence, useCurrentFrame, useVideoConfig } from 'remotion';
import React from 'react';

interface ContentVideoProps {
  title: string;
  keyPoints: string[];
}

export const ContentVideo: React.FC<ContentVideoProps> = ({ title, keyPoints }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  const titleOpacity = Math.min(1, frame / (fps * 0.5));

  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a2e' }}>
      <Sequence from={0} durationInFrames={fps * 2}>
        <AbsoluteFill
          style={{
            justifyContent: 'center',
            alignItems: 'center',
            padding: 60,
          }}
        >
          <h1
            style={{
              fontSize: 80,
              fontWeight: 'bold',
              color: 'white',
              textAlign: 'center',
              opacity: titleOpacity,
            }}
          >
            {title}
          </h1>
        </AbsoluteFill>
      </Sequence>

      {keyPoints.map((point, index) => (
        <Sequence
          key={index}
          from={fps * (2 + index * 3)}
          durationInFrames={fps * 3}
        >
          <AbsoluteFill
            style={{
              justifyContent: 'center',
              alignItems: 'center',
              padding: 80,
            }}
          >
            <div
              style={{
                fontSize: 60,
                color: '#16c79a',
                fontWeight: '600',
                textAlign: 'center',
                lineHeight: 1.4,
              }}
            >
              {point}
            </div>
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
import { scrapeNewsForKeyword } from '../scraper/news-scraper';
import { generateContentWithClaude } from '../ai/claude-generator';
import { generateVideo } from '../video/video-generator';

interface PipelineOptions {
  keyword: string;
  format: 'article' | 'case-study' | 'how-to' | 'toplist' | 'pov';
  tone: 'professional' | 'friendly' | 'humorous';
  language: 'en' | 'vi';
  generateVideo: boolean;
  platform?: 'reels' | 'tiktok' | 'shorts';
}

export async function runContentPipeline(options: PipelineOptions) {
  const { keyword, format, tone, language, generateVideo: shouldGenerateVideo, platform } = options;

  console.log(`Starting pipeline for keyword: ${keyword}`);

  // Step 1: Research
  console.log('Step 1: Gathering research data...');
  const articles = await scrapeNewsForKeyword(keyword, '24h');
  const researchData = articles
    .map((article) => `${article.title}\n${article.summary}`)
    .join('\n\n');

  // Step 2: Generate Content
  console.log('Step 2: Generating content with AI...');
  const content = await generateContentWithClaude({
    keyword,
    format,
    tone,
    language,
    researchData,
  });

  // Step 3: Extract key points for video
  let videoPath: string | null = null;
  if (shouldGenerateVideo && platform) {
    console.log('Step 3: Generating video...');
    
    // Simple extraction of first 3-5 sentences as key points
    const sentences = content.split(/[.!?]\s+/).slice(0, 5);
    
    videoPath = await generateVideo({
      title: keyword,
      keyPoints: sentences,
      duration: 30 * 30, // 30 seconds at 30fps
      platform,
    });
  }

  return {
    content,
    videoPath,
    metadata: {
      keyword,
      format,
      language,
      sourcesCount: articles.length,
      generatedAt: new Date().toISOString(),
    },
  };
}
```

## API Routes (Next.js)

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline/content-pipeline';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const { keyword, format, tone, language, generateVideo, platform } = body;

    // Validation
    if (!keyword) {
      return NextResponse.json(
        { error: 'Keyword is required' },
        { status: 400 }
      );
    }

    const result = await runContentPipeline({
      keyword,
      format: format || 'article',
      tone: tone || 'professional',
      language: language || 'en',
      generateVideo: generateVideo || false,
      platform: platform || 'reels',
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

## Frontend Integration

```typescript
// src/components/ContentGenerator.tsx
'use client';

import { useState } from 'react';

export default function ContentGenerator() {
  const [keyword, setKeyword] = useState('');
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState<any>(null);

  const handleGenerate = async () => {
    setLoading(true);
    try {
      const response = await fetch('/api/generate', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          keyword,
          format: 'article',
          tone: 'professional',
          language: 'en',
          generateVideo: true,
          platform: 'reels',
        }),
      });

      const data = await response.json();
      setResult(data);
    } catch (error) {
      console.error('Error:', error);
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="max-w-4xl mx-auto p-6">
      <h1 className="text-3xl font-bold mb-6">AI Content Generator</h1>
      
      <div className="mb-4">
        <input
          type="text"
          value={keyword}
          onChange={(e) => setKeyword(e.target.value)}
          placeholder="Enter keyword..."
          className="w-full p-3 border rounded-lg"
        />
      </div>

      <button
        onClick={handleGenerate}
        disabled={loading || !keyword}
        className="bg-blue-600 text-white px-6 py-3 rounded-lg disabled:opacity-50"
      >
        {loading ? 'Generating...' : 'Generate Content'}
      </button>

      {result && (
        <div className="mt-8">
          <h2 className="text-xl font-bold mb-4">Generated Content</h2>
          <div className="prose max-w-none">
            {result.content}
          </div>
          {result.videoPath && (
            <div className="mt-6">
              <h3 className="text-lg font-bold mb-2">Video</h3>
              <video controls className="w-full max-w-md">
                <source src={result.videoPath} type="video/mp4" />
              </video>
            </div>
          )}
        </div>
      )}
    </div>
  );
}
```

## Configuration Patterns

### Multi-Language Support

```typescript
// src/lib/config/languages.ts
export const languageConfig = {
  en: {
    systemPrompts: {
      article: 'You are a professional content writer creating engaging articles.',
      caseStudy: 'You are a business analyst writing detailed case studies.',
    },
    wordCount: { min: 1500, max: 2000 },
  },
  vi: {
    systemPrompts: {
      article: 'Bạn là nhà viết nội dung chuyên nghiệp tạo bài viết hấp dẫn.',
      caseStudy: 'Bạn là nhà phân tích kinh doanh viết case study chi tiết.',
    },
    wordCount: { min: 1200, max: 1500 },
  },
};
```

### Tone Customization

```typescript
// src/lib/config/tones.ts
export const toneConfig = {
  professional: {
    description: 'Formal, authoritative, data-driven',
    temperature: 0.5,
    keywords: ['research shows', 'according to', 'data indicates'],
  },
  friendly: {
    description: 'Conversational, approachable, relatable',
    temperature: 0.7,
    keywords: ['you might', 'let\'s explore', 'imagine'],
  },
  humorous: {
    description: 'Light-hearted, entertaining, witty',
    temperature: 0.8,
    keywords: ['funny thing', 'believe it or not', 'surprisingly'],
  },
};
```

## Troubleshooting

### API Rate Limits

```typescript
// src/lib/utils/rate-limiter.ts
import pLimit from 'p-limit';

const limit = pLimit(5); // Max 5 concurrent requests

export async function batchRequests<T>(
  items: any[],
  handler: (item: any) => Promise<T>
): Promise<T[]> {
  return Promise.all(
    items.map((item) => limit(() => handler(item)))
  );
}
```

### Error Handling

```typescript
// src/lib/utils/error-handler.ts
export class PipelineError extends Error {
  constructor(
    message: string,
    public stage: 'research' | 'generation' | 'video',
    public originalError?: Error
  ) {
    super(message);
    this.name = 'PipelineError';
  }
}

export function handlePipelineError(error: unknown) {
  if (error instanceof PipelineError) {
    console.error(`Error in ${error.stage} stage:`, error.message);
    // Implement retry logic or fallback
  } else {
    console.error('Unknown error:', error);
  }
}
```

### Video Rendering Issues

If Remotion rendering fails:

```bash
# Install required dependencies
npm install @remotion/bundler @remotion/renderer @remotion/cli

# Test rendering locally
npx remotion render remotion/index.ts ContentVideo output.mp4

# Check FFmpeg installation
ffmpeg -version
```

## Build and Deploy

```bash
# Build for production
npm run build

# Start production server
npm start

# Build Remotion bundle
npm run remotion:bundle
```

This skill enables AI agents to implement complete content automation pipelines using Claude, OpenAI, web scraping, and video generation with Remotion.
