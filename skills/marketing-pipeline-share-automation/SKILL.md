---
name: marketing-pipeline-share-automation
description: Ultimate AI content pipeline for automated research, script generation, and video creation using Claude, OpenAI, and Remotion
triggers:
  - automate content creation from research to video
  - generate marketing content with AI pipeline
  - create automated video content from text
  - build AI-powered content workflow
  - set up automated marketing pipeline
  - generate multilingual content with Claude and OpenAI
  - automate research and content generation
  - create video content from AI-generated scripts
---

# Marketing Pipeline Share - AI Content Automation Skill

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

Marketing Pipeline Share is a comprehensive AI-powered content automation system that handles the entire content creation workflow: from researching trending topics, generating multilingual scripts, to rendering videos automatically. It leverages Claude 3, OpenAI, and Remotion to create a complete content production pipeline.

**Key capabilities:**
- Auto-scan and research from TechCrunch, a16z, Twitter, LinkedIn
- Generate content in multiple formats (Toplist, POV, Case Study, How-to)
- Multilingual support (English & Vietnamese)
- Automatic video rendering with Remotion
- Next.js-based UI for content management

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
# AI APIs
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Application
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

## Running the Application

```bash
# Development mode
npm run dev

# Production build
npm run build
npm start

# Remotion video rendering (if separate)
npm run remotion
```

## Core Architecture

### 1. Research Module

The research module crawls and aggregates content from multiple sources:

```typescript
// lib/research/crawler.ts
import { Anthropic } from '@anthropic-ai/sdk';

interface ResearchSource {
  platform: 'techcrunch' | 'a16z' | 'twitter' | 'linkedin';
  timeframe: '24h' | '7d' | '30d';
  keywords: string[];
}

export async function crawlResearch(sources: ResearchSource[]) {
  const results = [];
  
  for (const source of sources) {
    const data = await fetchSourceData(source);
    const analyzed = await analyzeWithAI(data);
    results.push(analyzed);
  }
  
  return aggregateInsights(results);
}

async function fetchSourceData(source: ResearchSource) {
  // Use RapidAPI or custom scrapers
  const response = await fetch(`https://api.rapidapi.com/${source.platform}`, {
    headers: {
      'X-RapidAPI-Key': process.env.RAPIDAPI_KEY!,
    }
  });
  
  return response.json();
}
```

### 2. Content Generation

Generate content using Claude or OpenAI with customizable formats:

```typescript
// lib/content/generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

type ContentFormat = 'toplist' | 'pov' | 'case-study' | 'how-to';
type Language = 'en' | 'vi';
type Tone = 'expert' | 'friendly' | 'humorous';

interface ContentConfig {
  topic: string;
  format: ContentFormat;
  language: Language;
  tone: Tone;
  researchData: any[];
}

export async function generateContent(config: ContentConfig) {
  const anthropic = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY,
  });

  const prompt = buildPrompt(config);
  
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [{
      role: 'user',
      content: prompt
    }]
  });

  return parseContentResponse(message.content);
}

function buildPrompt(config: ContentConfig): string {
  const formatInstructions = {
    'toplist': 'Create a numbered list with detailed explanations for each point',
    'pov': 'Write from a specific perspective with personal insights',
    'case-study': 'Present a detailed analysis with data and outcomes',
    'how-to': 'Provide step-by-step instructions with examples'
  };

  return `
    Topic: ${config.topic}
    Format: ${formatInstructions[config.format]}
    Language: ${config.language === 'vi' ? 'Vietnamese' : 'English'}
    Tone: ${config.tone}
    
    Research Data:
    ${JSON.stringify(config.researchData, null, 2)}
    
    Generate comprehensive content that incorporates the research insights and follows the specified format.
  `;
}
```

### 3. Multilingual Content Generation

Generate content simultaneously in multiple languages:

```typescript
// lib/content/multilingual.ts
export async function generateMultilingualContent(
  topic: string,
  format: ContentFormat,
  researchData: any[]
) {
  const [englishContent, vietnameseContent] = await Promise.all([
    generateContent({
      topic,
      format,
      language: 'en',
      tone: 'expert',
      researchData
    }),
    generateContent({
      topic,
      format,
      language: 'vi',
      tone: 'expert',
      researchData
    })
  ]);

  return {
    en: englishContent,
    vi: vietnameseContent
  };
}
```

### 4. Video Generation with Remotion

Automatically create videos from generated content:

```typescript
// remotion/VideoComposition.tsx
import { Composition } from 'remotion';
import { ContentVideo } from './ContentVideo';

export const RemotionRoot: React.FC = () => {
  return (
    <>
      <Composition
        id="ContentVideo"
        component={ContentVideo}
        durationInFrames={300}
        fps={30}
        width={1080}
        height={1920}
        defaultProps={{
          title: 'Your Video Title',
          content: [],
          style: 'modern'
        }}
      />
    </>
  );
};
```

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, interpolate } from 'remotion';

interface ContentVideoProps {
  title: string;
  content: Array<{ text: string; duration: number }>;
  style: 'modern' | 'minimal' | 'bold';
}

export const ContentVideo: React.FC<ContentVideoProps> = ({
  title,
  content,
  style
}) => {
  const frame = useCurrentFrame();
  
  const opacity = interpolate(frame, [0, 30], [0, 1], {
    extrapolateRight: 'clamp',
  });

  return (
    <AbsoluteFill style={{ backgroundColor: '#000' }}>
      <div style={{ opacity, padding: 40 }}>
        <h1 style={{ color: '#fff', fontSize: 60 }}>{title}</h1>
        {content.map((item, index) => (
          <p key={index} style={{ color: '#fff', fontSize: 32 }}>
            {item.text}
          </p>
        ))}
      </div>
    </AbsoluteFill>
  );
};
```

Render videos programmatically:

```typescript
// lib/video/render.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

export async function renderContentVideo(
  contentData: any,
  outputPath: string
) {
  const bundleLocation = await bundle(
    path.join(process.cwd(), 'remotion/index.ts')
  );

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: contentData,
  });

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: contentData,
  });

  return outputPath;
}
```

## API Routes

### Research Endpoint

```typescript
// app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { crawlResearch } from '@/lib/research/crawler';

export async function POST(req: NextRequest) {
  try {
    const { keywords, platforms, timeframe } = await req.json();
    
    const sources = platforms.map((platform: string) => ({
      platform,
      timeframe,
      keywords
    }));
    
    const insights = await crawlResearch(sources);
    
    return NextResponse.json({ success: true, data: insights });
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
import { generateMultilingualContent } from '@/lib/content/multilingual';
import { renderContentVideo } from '@/lib/video/render';

export async function POST(req: NextRequest) {
  try {
    const { topic, format, researchData, includeVideo } = await req.json();
    
    const content = await generateMultilingualContent(
      topic,
      format,
      researchData
    );
    
    let videoUrl = null;
    if (includeVideo) {
      const outputPath = `/tmp/${Date.now()}.mp4`;
      await renderContentVideo(content.en, outputPath);
      videoUrl = outputPath;
    }
    
    return NextResponse.json({
      success: true,
      content,
      videoUrl
    });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

## Complete Workflow Example

```typescript
// lib/pipeline/complete-workflow.ts
import { crawlResearch } from '@/lib/research/crawler';
import { generateMultilingualContent } from '@/lib/content/multilingual';
import { renderContentVideo } from '@/lib/video/render';

export async function runCompletePipeline(
  keyword: string,
  format: ContentFormat = 'toplist'
) {
  // Step 1: Research
  console.log('🔍 Researching...');
  const research = await crawlResearch([
    { platform: 'techcrunch', timeframe: '24h', keywords: [keyword] },
    { platform: 'twitter', timeframe: '24h', keywords: [keyword] }
  ]);

  // Step 2: Generate Content
  console.log('✍️ Generating content...');
  const content = await generateMultilingualContent(
    keyword,
    format,
    research
  );

  // Step 3: Render Video
  console.log('🎬 Rendering video...');
  const videoPath = await renderContentVideo(
    content.en,
    `/output/${keyword}-${Date.now()}.mp4`
  );

  return {
    research,
    content,
    videoPath
  };
}
```

## Common Patterns

### Using with React Components

```typescript
// components/ContentGenerator.tsx
'use client';

import { useState } from 'react';

export function ContentGenerator() {
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState(null);

  const handleGenerate = async (topic: string) => {
    setLoading(true);
    try {
      const response = await fetch('/api/generate', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          topic,
          format: 'toplist',
          researchData: [],
          includeVideo: true
        })
      });
      
      const data = await response.json();
      setResult(data);
    } finally {
      setLoading(false);
    }
  };

  return (
    <div>
      {/* UI implementation */}
    </div>
  );
}
```

### Batch Processing

```typescript
// lib/pipeline/batch.ts
export async function batchGenerateContent(topics: string[]) {
  const results = [];
  
  for (const topic of topics) {
    const result = await runCompletePipeline(topic, 'how-to');
    results.push(result);
    
    // Rate limiting
    await new Promise(resolve => setTimeout(resolve, 2000));
  }
  
  return results;
}
```

## Troubleshooting

### API Key Issues

```typescript
// lib/utils/validate-env.ts
export function validateEnvironment() {
  const required = [
    'ANTHROPIC_API_KEY',
    'OPENAI_API_KEY',
    'RAPIDAPI_KEY'
  ];
  
  const missing = required.filter(key => !process.env[key]);
  
  if (missing.length > 0) {
    throw new Error(`Missing environment variables: ${missing.join(', ')}`);
  }
}
```

### Video Rendering Errors

- Ensure Remotion dependencies are installed: `npm install @remotion/bundler @remotion/renderer`
- Check ffmpeg is installed: `ffmpeg -version`
- Verify output directory has write permissions

### Rate Limiting

```typescript
// lib/utils/rate-limiter.ts
export class RateLimiter {
  private queue: Array<() => Promise<any>> = [];
  private processing = false;
  
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
    await new Promise(resolve => setTimeout(resolve, 1000));
    this.processing = false;
    this.process();
  }
}
```

## Best Practices

1. **Always validate environment variables** before running the pipeline
2. **Implement rate limiting** when using external APIs
3. **Cache research results** to avoid redundant API calls
4. **Use TypeScript interfaces** for all content structures
5. **Implement error handling** at each pipeline stage
6. **Monitor API usage** to stay within quota limits
7. **Test video rendering locally** before deploying
