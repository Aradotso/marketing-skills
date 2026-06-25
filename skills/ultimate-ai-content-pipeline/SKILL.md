---
name: ultimate-ai-content-pipeline
description: Automate content creation from research to video generation using AI (Claude/OpenAI) and Remotion
triggers:
  - automate content creation with AI pipeline
  - generate marketing content automatically
  - create videos from text with Remotion
  - set up AI content automation system
  - build automated content research pipeline
  - use Claude API for content generation
  - render videos automatically from articles
  - crawl news and generate content
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

Complete AI-powered content automation system that handles research, scriptwriting, and video generation. Automatically crawls news sources (TechCrunch, a16z, Twitter, LinkedIn), generates multi-format content using Claude/OpenAI, and renders videos with Remotion.

## What This Project Does

Ultimate AI Content Pipeline is a TypeScript-based content automation system that:

- **Auto-researches** trending topics from major news sources in real-time
- **Generates content** in multiple formats (listicles, POV, case studies, how-tos)
- **Supports bilingual** output (English & Vietnamese)
- **Renders videos** automatically using Remotion for social media platforms
- **Optimizes for multiple platforms** (Reels, TikTok, Shorts)

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
# AI APIs
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Content Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Remotion (for video rendering)
REMOTION_LICENSE_KEY=your_remotion_license
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── lib/              # Core libraries
│   │   ├── ai/           # AI integration (Claude, OpenAI)
│   │   ├── crawler/      # News crawling logic
│   │   ├── content/      # Content generation
│   │   └── video/        # Remotion video rendering
│   ├── components/       # React components
│   └── types/            # TypeScript definitions
├── remotion/             # Remotion video templates
└── public/               # Static assets
```

## Core Components

### 1. Content Research & Crawling

```typescript
// src/lib/crawler/news-fetcher.ts
import axios from 'axios';

interface NewsSource {
  url: string;
  source: string;
  publishedAt: Date;
  title: string;
  content: string;
}

export async function fetchLatestNews(
  topic: string,
  sources: string[] = ['techcrunch', 'a16z', 'twitter']
): Promise<NewsSource[]> {
  const results: NewsSource[] = [];
  
  for (const source of sources) {
    try {
      const response = await axios.get(
        `https://api.rapidapi.com/news/${source}`,
        {
          params: { q: topic, timeframe: '24h' },
          headers: {
            'X-RapidAPI-Key': process.env.RAPIDAPI_KEY!,
            'X-RapidAPI-Host': 'news-aggregator.p.rapidapi.com'
          }
        }
      );
      
      results.push(...response.data.articles);
    } catch (error) {
      console.error(`Failed to fetch from ${source}:`, error);
    }
  }
  
  return results;
}

// Extract insights from crawled data
export function extractInsights(articles: NewsSource[]): string {
  const insights = articles
    .map(article => `${article.source}: ${article.title}`)
    .join('\n');
    
  return insights;
}
```

### 2. AI Content Generation with Claude

```typescript
// src/lib/ai/claude-generator.ts
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY!,
});

interface ContentRequest {
  topic: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  tone: 'expert' | 'friendly' | 'humorous';
  language: 'en' | 'vi';
  researchData: string;
}

export async function generateContent(
  request: ContentRequest
): Promise<string> {
  const prompt = buildPrompt(request);
  
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [
      {
        role: 'user',
        content: prompt
      }
    ]
  });
  
  const content = message.content[0];
  return content.type === 'text' ? content.text : '';
}

function buildPrompt(request: ContentRequest): string {
  const formatInstructions = {
    'toplist': 'Create a numbered list article with clear points',
    'pov': 'Write from a personal perspective with opinions',
    'case-study': 'Analyze real examples with data and outcomes',
    'how-to': 'Provide step-by-step actionable instructions'
  };
  
  return `
You are an expert content creator. Generate a ${request.format} article about "${request.topic}".

Research Data:
${request.researchData}

Requirements:
- Format: ${formatInstructions[request.format]}
- Tone: ${request.tone}
- Language: ${request.language === 'en' ? 'English' : 'Vietnamese'}
- Include data-backed insights from the research
- Make it engaging and actionable
- Length: 800-1200 words

Generate the content now:
`;
}
```

### 3. Alternative: OpenAI Content Generation

```typescript
// src/lib/ai/openai-generator.ts
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY!,
});

export async function generateContentWithGPT(
  request: ContentRequest
): Promise<string> {
  const prompt = buildPrompt(request);
  
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: 'You are an expert content creator specializing in marketing and technology topics.'
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

### 4. Video Generation with Remotion

```typescript
// src/lib/video/render-video.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  title: string;
  content: string[];
  duration: number; // in seconds
  format: 'reels' | 'tiktok' | 'shorts';
}

export async function renderContentVideo(
  config: VideoConfig
): Promise<string> {
  // Get video dimensions based on format
  const dimensions = {
    'reels': { width: 1080, height: 1920 },
    'tiktok': { width: 1080, height: 1920 },
    'shorts': { width: 1080, height: 1920 }
  };
  
  const { width, height } = dimensions[config.format];
  
  // Bundle Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.join(process.cwd(), 'remotion', 'index.ts'),
    webpackOverride: (config) => config,
  });
  
  // Select composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title: config.title,
      slides: config.content,
    },
  });
  
  // Output path
  const outputPath = path.join(
    process.cwd(),
    'output',
    `video-${Date.now()}.mp4`
  );
  
  // Render video
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: {
      title: config.title,
      slides: config.content,
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
  slides: string[];
}

export const ContentVideo: React.FC<ContentVideoProps> = ({ title, slides }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  
  const slidesDuration = Math.floor((fps * 3)); // 3 seconds per slide
  
  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a1a' }}>
      {/* Title Sequence */}
      <Sequence durationInFrames={fps * 2}>
        <AbsoluteFill
          style={{
            justifyContent: 'center',
            alignItems: 'center',
            padding: 60,
          }}
        >
          <h1
            style={{
              fontSize: 72,
              color: 'white',
              textAlign: 'center',
              fontWeight: 'bold',
              lineHeight: 1.2,
            }}
          >
            {title}
          </h1>
        </AbsoluteFill>
      </Sequence>
      
      {/* Content Slides */}
      {slides.map((slide, index) => (
        <Sequence
          key={index}
          from={(index + 1) * slidesDuration}
          durationInFrames={slidesDuration}
        >
          <AbsoluteFill
            style={{
              justifyContent: 'center',
              alignItems: 'center',
              padding: 80,
            }}
          >
            <p
              style={{
                fontSize: 48,
                color: 'white',
                textAlign: 'center',
                lineHeight: 1.4,
              }}
            >
              {slide}
            </p>
          </AbsoluteFill>
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};
```

## Complete Pipeline Example

```typescript
// src/lib/pipeline/content-pipeline.ts
import { fetchLatestNews, extractInsights } from '../crawler/news-fetcher';
import { generateContent } from '../ai/claude-generator';
import { renderContentVideo } from '../video/render-video';

interface PipelineConfig {
  topic: string;
  contentFormat: 'toplist' | 'pov' | 'case-study' | 'how-to';
  tone: 'expert' | 'friendly' | 'humorous';
  language: 'en' | 'vi';
  generateVideo: boolean;
  videoFormat?: 'reels' | 'tiktok' | 'shorts';
}

export async function runContentPipeline(
  config: PipelineConfig
): Promise<{
  content: string;
  videoPath?: string;
}> {
  // Step 1: Research
  console.log('🔍 Researching latest news...');
  const articles = await fetchLatestNews(config.topic);
  const insights = extractInsights(articles);
  
  // Step 2: Generate Content
  console.log('✍️ Generating content with AI...');
  const content = await generateContent({
    topic: config.topic,
    format: config.contentFormat,
    tone: config.tone,
    language: config.language,
    researchData: insights,
  });
  
  // Step 3: Generate Video (optional)
  let videoPath: string | undefined;
  if (config.generateVideo && config.videoFormat) {
    console.log('🎬 Rendering video...');
    
    // Split content into slides
    const slides = content
      .split('\n\n')
      .filter(p => p.trim().length > 0)
      .slice(0, 5); // First 5 paragraphs
    
    videoPath = await renderContentVideo({
      title: config.topic,
      content: slides,
      duration: slides.length * 3,
      format: config.videoFormat,
    });
  }
  
  console.log('✅ Pipeline complete!');
  
  return { content, videoPath };
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
    
    const result = await runContentPipeline({
      topic: body.topic,
      contentFormat: body.format || 'toplist',
      tone: body.tone || 'friendly',
      language: body.language || 'en',
      generateVideo: body.generateVideo || false,
      videoFormat: body.videoFormat,
    });
    
    return NextResponse.json({
      success: true,
      data: result,
    });
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { success: false, error: 'Failed to generate content' },
      { status: 500 }
    );
  }
}
```

## Usage in Frontend

```typescript
// Example: Using the pipeline from a React component
'use client';

import { useState } from 'react';

export default function ContentGenerator() {
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState<any>(null);
  
  const handleGenerate = async () => {
    setLoading(true);
    
    try {
      const response = await fetch('/api/generate', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          topic: 'AI in Marketing 2026',
          format: 'toplist',
          tone: 'expert',
          language: 'en',
          generateVideo: true,
          videoFormat: 'reels',
        }),
      });
      
      const data = await response.json();
      setResult(data);
    } catch (error) {
      console.error('Generation failed:', error);
    } finally {
      setLoading(false);
    }
  };
  
  return (
    <div>
      <button onClick={handleGenerate} disabled={loading}>
        {loading ? 'Generating...' : 'Generate Content'}
      </button>
      
      {result && (
        <div>
          <h2>Generated Content:</h2>
          <pre>{result.data.content}</pre>
          {result.data.videoPath && (
            <p>Video saved to: {result.data.videoPath}</p>
          )}
        </div>
      )}
    </div>
  );
}
```

## Running the Project

```bash
# Development mode
npm run dev

# Build for production
npm run build

# Start production server
npm run start

# Render video only (if you have a separate script)
npm run render
```

## Common Patterns

### Batch Content Generation

```typescript
// Generate multiple pieces of content
const topics = [
  'AI Marketing Trends',
  'Social Media Automation',
  'Content Strategy 2026'
];

const results = await Promise.all(
  topics.map(topic =>
    runContentPipeline({
      topic,
      contentFormat: 'toplist',
      tone: 'expert',
      language: 'en',
      generateVideo: false,
    })
  )
);
```

### Scheduled Content Generation

```typescript
// Using node-cron for scheduled generation
import cron from 'node-cron';

cron.schedule('0 9 * * *', async () => {
  console.log('Running daily content generation...');
  
  const result = await runContentPipeline({
    topic: 'Daily Tech News',
    contentFormat: 'toplist',
    tone: 'friendly',
    language: 'en',
    generateVideo: true,
    videoFormat: 'reels',
  });
  
  // Save to database or publish to social media
});
```

### Bilingual Content Generation

```typescript
// Generate content in both languages
const [englishContent, vietnameseContent] = await Promise.all([
  generateContent({
    topic: 'AI Marketing',
    format: 'toplist',
    tone: 'expert',
    language: 'en',
    researchData: insights,
  }),
  generateContent({
    topic: 'AI Marketing',
    format: 'toplist',
    tone: 'expert',
    language: 'vi',
    researchData: insights,
  })
]);
```

## Troubleshooting

### API Key Issues

```typescript
// Verify API keys are loaded
if (!process.env.ANTHROPIC_API_KEY) {
  throw new Error('ANTHROPIC_API_KEY not found in environment');
}

if (!process.env.OPENAI_API_KEY) {
  throw new Error('OPENAI_API_KEY not found in environment');
}
```

### Rate Limiting

```typescript
// Add rate limiting for API calls
import pLimit from 'p-limit';

const limit = pLimit(2); // Max 2 concurrent requests

const results = await Promise.all(
  sources.map(source =>
    limit(() => fetchLatestNews(source))
  )
);
```

### Video Rendering Issues

```bash
# Ensure ffmpeg is installed
which ffmpeg

# Install if missing (macOS)
brew install ffmpeg

# Install if missing (Ubuntu)
sudo apt-get install ffmpeg

# Check Remotion installation
npx remotion versions
```

### Memory Issues with Large Content

```typescript
// Process content in chunks
function chunkContent(content: string, maxChunkSize: number = 1000) {
  const words = content.split(' ');
  const chunks: string[] = [];
  
  for (let i = 0; i < words.length; i += maxChunkSize) {
    chunks.push(words.slice(i, i + maxChunkSize).join(' '));
  }
  
  return chunks;
}
```

### Debugging Pipeline

```typescript
// Enable verbose logging
export async function runContentPipeline(
  config: PipelineConfig,
  debug: boolean = false
): Promise<any> {
  if (debug) {
    console.log('Pipeline config:', JSON.stringify(config, null, 2));
  }
  
  // ... rest of pipeline with debug logs
}
```

## Configuration Options

Key configuration locations:

- **Environment variables**: `.env.local`
- **Remotion config**: `remotion.config.ts`
- **Next.js config**: `next.config.js`
- **TypeScript**: `tsconfig.json`

This skill covers the complete workflow for automating content creation with AI research, generation, and video rendering capabilities.
