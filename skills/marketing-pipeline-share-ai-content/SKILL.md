---
name: marketing-pipeline-share-ai-content
description: Automated AI content pipeline for research, scriptwriting, and video generation using Claude/OpenAI and Remotion
triggers:
  - how do I generate automated content with marketing pipeline
  - set up AI content pipeline with Claude and OpenAI
  - create automated video content from research
  - configure marketing pipeline content automation
  - use remotion for automated video generation
  - build content pipeline with AI research and scripting
  - automate content creation from keyword to video
  - integrate Claude API for content generation pipeline
---

# Marketing Pipeline Share - AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

Marketing Pipeline Share is a comprehensive TypeScript-based content automation system that handles the entire content creation lifecycle: from research and data crawling, through AI-powered scriptwriting, to automated video generation. It leverages Claude 3, OpenAI, and Remotion to transform a single keyword into publication-ready content with accompanying visuals.

**Key capabilities:**
- Auto-crawl research from TechCrunch, a16z, Twitter/X, LinkedIn (last 24h)
- Multi-format content generation (toplist, POV, case studies, how-tos)
- Bilingual support (English & Vietnamese) with tone customization
- Automated video/infographic rendering via Remotion
- Next.js web interface for content management

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
# AI Provider APIs
OPENAI_API_KEY=your_openai_key_here
ANTHROPIC_API_KEY=your_claude_key_here

# Research Data APIs (RapidAPI)
RAPIDAPI_KEY=your_rapidapi_key_here

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Next.js Configuration
NEXT_PUBLIC_API_URL=http://localhost:3000

# Remotion Configuration (for video rendering)
REMOTION_LICENSE_KEY=your_remotion_license_key
```

## Running the Application

```bash
# Development mode
npm run dev

# Production build
npm run build
npm start

# Remotion video preview
npm run remotion:preview

# Render video
npm run remotion:render
```

## Core API Usage Patterns

### 1. Research & Data Crawling

```typescript
import { ResearchService } from '@/services/research';

// Initialize research service
const research = new ResearchService({
  rapidApiKey: process.env.RAPIDAPI_KEY!,
  sources: ['techcrunch', 'a16z', 'twitter', 'linkedin']
});

// Crawl recent content by keyword
async function fetchResearch(keyword: string) {
  const results = await research.crawl({
    keyword,
    timeRange: '24h',
    maxResults: 50,
    language: 'en'
  });

  // Extract insights
  const insights = await research.extractInsights(results);
  
  return {
    rawData: results,
    insights,
    sources: results.map(r => r.source)
  };
}
```

### 2. AI Content Generation with Claude

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY!
});

interface ContentRequest {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  tone: 'expert' | 'friendly' | 'humorous';
  language: 'en' | 'vi';
  researchData: any;
}

async function generateContent(request: ContentRequest) {
  const prompt = `
You are a content creator. Generate a ${request.format} article about "${request.keyword}".

Tone: ${request.tone}
Language: ${request.language}

Research data:
${JSON.stringify(request.researchData, null, 2)}

Create engaging, data-backed content with:
- Compelling headline
- Introduction hook
- Main body with insights
- Conclusion with CTA
- Meta description for SEO
`;

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
```

### 3. OpenAI Alternative Implementation

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY!
});

async function generateContentOpenAI(request: ContentRequest) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are an expert content writer specializing in ${request.format} format with a ${request.tone} tone.`
      },
      {
        role: 'user',
        content: `Create ${request.language} content about: ${request.keyword}\n\nResearch: ${JSON.stringify(request.researchData)}`
      }
    ],
    temperature: 0.7,
    max_tokens: 3000
  });

  return completion.choices[0].message.content;
}
```

### 4. Video Generation with Remotion

```typescript
// remotion/VideoComposition.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig } from 'remotion';

interface VideoProps {
  title: string;
  content: string[];
  bgColor: string;
}

export const ContentVideo: React.FC<VideoProps> = ({ title, content, bgColor }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  const opacity = Math.min(1, frame / 30);

  return (
    <AbsoluteFill style={{ backgroundColor: bgColor }}>
      <div style={{ 
        opacity, 
        padding: 60,
        fontFamily: 'Arial, sans-serif'
      }}>
        <h1 style={{ fontSize: 64, marginBottom: 40 }}>{title}</h1>
        {content.map((text, idx) => (
          <p key={idx} style={{ fontSize: 32, marginBottom: 20 }}>
            {text}
          </p>
        ))}
      </div>
    </AbsoluteFill>
  );
};
```

```typescript
// services/video-renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface RenderOptions {
  title: string;
  content: string[];
  outputPath: string;
}

export async function renderContentVideo(options: RenderOptions) {
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: (config) => config
  });

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title: options.title,
      content: options.content,
      bgColor: '#1a1a2e'
    }
  });

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: options.outputPath,
    inputProps: composition.props
  });

  return options.outputPath;
}
```

### 5. Complete Pipeline Integration

```typescript
// services/content-pipeline.ts
import { ResearchService } from './research';
import { generateContent } from './ai-content';
import { renderContentVideo } from './video-renderer';

export class ContentPipeline {
  private research: ResearchService;

  constructor() {
    this.research = new ResearchService({
      rapidApiKey: process.env.RAPIDAPI_KEY!
    });
  }

  async execute(keyword: string, options: {
    format: 'toplist' | 'pov' | 'case-study' | 'how-to';
    tone: 'expert' | 'friendly' | 'humorous';
    languages: ('en' | 'vi')[];
    generateVideo: boolean;
  }) {
    // Step 1: Research
    console.log(`[Pipeline] Researching: ${keyword}`);
    const researchData = await this.research.crawl({
      keyword,
      timeRange: '24h',
      maxResults: 50
    });

    const insights = await this.research.extractInsights(researchData);

    // Step 2: Generate content for each language
    const contents = await Promise.all(
      options.languages.map(async (lang) => {
        console.log(`[Pipeline] Generating ${lang} content`);
        const content = await generateContent({
          keyword,
          format: options.format,
          tone: options.tone,
          language: lang,
          researchData: insights
        });

        return { language: lang, content };
      })
    );

    // Step 3: Generate video (if requested)
    let videoPath: string | null = null;
    if (options.generateVideo) {
      console.log('[Pipeline] Rendering video');
      const mainContent = contents.find(c => c.language === 'en') || contents[0];
      
      // Parse content into video slides
      const slides = mainContent.content
        .split('\n\n')
        .filter(p => p.trim().length > 0)
        .slice(0, 5); // First 5 paragraphs

      videoPath = await renderContentVideo({
        title: keyword,
        content: slides,
        outputPath: `./output/${keyword.replace(/\s+/g, '-')}.mp4`
      });
    }

    return {
      research: insights,
      contents,
      videoPath,
      metadata: {
        keyword,
        format: options.format,
        createdAt: new Date().toISOString()
      }
    };
  }
}
```

### 6. Next.js API Route Example

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { ContentPipeline } from '@/services/content-pipeline';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const { keyword, format, tone, languages, generateVideo } = body;

    // Validate input
    if (!keyword || !format || !tone) {
      return NextResponse.json(
        { error: 'Missing required fields' },
        { status: 400 }
      );
    }

    // Execute pipeline
    const pipeline = new ContentPipeline();
    const result = await pipeline.execute(keyword, {
      format,
      tone,
      languages: languages || ['en'],
      generateVideo: generateVideo || false
    });

    return NextResponse.json(result);
  } catch (error) {
    console.error('[API Error]', error);
    return NextResponse.json(
      { error: 'Content generation failed' },
      { status: 500 }
    );
  }
}
```

## Common Patterns

### Multi-Provider AI Failover

```typescript
async function generateWithFailover(request: ContentRequest) {
  try {
    // Try Claude first
    return await generateContent(request);
  } catch (claudeError) {
    console.warn('[AI] Claude failed, falling back to OpenAI', claudeError);
    try {
      return await generateContentOpenAI(request);
    } catch (openaiError) {
      console.error('[AI] All providers failed', openaiError);
      throw new Error('Content generation failed');
    }
  }
}
```

### Batch Content Generation

```typescript
async function generateBatch(keywords: string[]) {
  const pipeline = new ContentPipeline();
  
  const results = await Promise.allSettled(
    keywords.map(keyword =>
      pipeline.execute(keyword, {
        format: 'toplist',
        tone: 'expert',
        languages: ['en', 'vi'],
        generateVideo: true
      })
    )
  );

  return results.map((result, idx) => ({
    keyword: keywords[idx],
    status: result.status,
    data: result.status === 'fulfilled' ? result.value : null,
    error: result.status === 'rejected' ? result.reason : null
  }));
}
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement exponential backoff
async function withRetry<T>(
  fn: () => Promise<T>,
  maxRetries = 3
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error: any) {
      if (error.status === 429 && i < maxRetries - 1) {
        const delay = Math.pow(2, i) * 1000;
        console.log(`[Retry] Waiting ${delay}ms before retry ${i + 1}`);
        await new Promise(resolve => setTimeout(resolve, delay));
      } else {
        throw error;
      }
    }
  }
  throw new Error('Max retries exceeded');
}
```

### Video Rendering Memory Issues

```bash
# Increase Node.js memory limit
NODE_OPTIONS="--max-old-space-size=4096" npm run remotion:render
```

### Missing Environment Variables

```typescript
// services/config-validator.ts
export function validateConfig() {
  const required = [
    'OPENAI_API_KEY',
    'ANTHROPIC_API_KEY',
    'RAPIDAPI_KEY'
  ];

  const missing = required.filter(key => !process.env[key]);

  if (missing.length > 0) {
    throw new Error(
      `Missing required environment variables: ${missing.join(', ')}\n` +
      'Please check your .env.local file'
    );
  }
}
```

## Performance Optimization

```typescript
// Cache research results
import NodeCache from 'node-cache';

const cache = new NodeCache({ stdTTL: 3600 }); // 1 hour

async function getCachedResearch(keyword: string) {
  const cacheKey = `research:${keyword}`;
  const cached = cache.get(cacheKey);
  
  if (cached) {
    console.log('[Cache] Hit for:', keyword);
    return cached;
  }

  const research = new ResearchService({
    rapidApiKey: process.env.RAPIDAPI_KEY!
  });
  
  const data = await research.crawl({ keyword, timeRange: '24h' });
  cache.set(cacheKey, data);
  
  return data;
}
```
