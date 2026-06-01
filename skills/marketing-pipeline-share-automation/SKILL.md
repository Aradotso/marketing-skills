---
name: marketing-pipeline-share-automation
description: Automated AI content pipeline that researches, generates scripts, and creates videos from keywords using Claude, OpenAI, and Remotion
triggers:
  - how do I automate content creation from research to video
  - generate marketing content pipeline with AI
  - create automated video content from keywords
  - set up AI content research and generation workflow
  - build content automation with Claude and Remotion
  - automate social media content pipeline with AI
  - create video content from text using marketing pipeline
  - research and generate content automatically with AI
---

# Marketing Pipeline Share - AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

Marketing Pipeline Share is a comprehensive TypeScript-based content automation system that transforms keywords into complete content packages including research, written content, and rendered videos. It orchestrates multiple AI services (Claude 3, OpenAI) and video rendering (Remotion) to create a fully automated content production pipeline.

**Key capabilities:**
- Automated research from news sources (TechCrunch, a16z, Twitter, LinkedIn)
- Multi-format content generation (Toplist, POV, Case Study, How-to)
- Bilingual output (English & Vietnamese)
- Automatic video and infographic rendering
- Multi-platform optimization (Reels, TikTok, Shorts)

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
# AI Services
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# RapidAPI (for news/research crawling)
RAPIDAPI_KEY=your_rapidapi_key

# Optional: Additional service keys
NEXT_PUBLIC_API_URL=http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app router pages
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── crawler/     # Content research & crawling
│   │   ├── content/     # Content generation logic
│   │   └── video/       # Remotion video rendering
│   ├── types/           # TypeScript type definitions
│   └── utils/           # Helper functions
├── public/              # Static assets
└── remotion/            # Remotion video templates
```

## Core Workflow

### 1. Research & Data Collection

```typescript
import { researchTopic } from '@/lib/crawler/research';

// Automated research from multiple sources
async function performResearch(keyword: string) {
  const researchData = await researchTopic({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeRange: '24h',
    depth: 'deep'
  });

  return {
    insights: researchData.insights,
    statistics: researchData.statistics,
    trends: researchData.trends,
    sources: researchData.sourcesUsed
  };
}
```

### 2. AI Content Generation

```typescript
import { generateContent } from '@/lib/ai/content-generator';
import { ContentFormat, Language, Tone } from '@/types';

// Generate content with AI
async function createContent(research: any, options: ContentOptions) {
  const content = await generateContent({
    research,
    format: 'toplist', // 'toplist' | 'pov' | 'case-study' | 'how-to'
    language: 'en', // 'en' | 'vi' | 'both'
    tone: 'expert', // 'expert' | 'friendly' | 'humorous'
    provider: 'claude' // 'claude' | 'openai'
  });

  return content;
}
```

### 3. Using Claude for Content

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

async function generateWithClaude(prompt: string, systemPrompt: string) {
  const message = await anthropic.messages.create({
    model: 'claude-3-opus-20240229',
    max_tokens: 4096,
    system: systemPrompt,
    messages: [
      {
        role: 'user',
        content: prompt
      }
    ]
  });

  return message.content[0].text;
}
```

### 4. Using OpenAI for Content

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateWithOpenAI(prompt: string, systemPrompt: string) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      { role: 'system', content: systemPrompt },
      { role: 'user', content: prompt }
    ],
    temperature: 0.7,
    max_tokens: 4000
  });

  return completion.choices[0].message.content;
}
```

### 5. Video Rendering with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import { webpackOverride } from './remotion/webpack-override';

async function renderContentVideo(content: any, outputPath: string) {
  // Bundle Remotion project
  const bundled = await bundle({
    entryPoint: './remotion/index.tsx',
    webpackOverride,
  });

  // Select composition
  const composition = await selectComposition({
    serveUrl: bundled,
    id: 'ContentVideo',
    inputProps: {
      title: content.title,
      highlights: content.highlights,
      statistics: content.statistics,
      style: 'modern'
    },
  });

  // Render video
  await renderMedia({
    composition,
    serveUrl: bundled,
    codec: 'h264',
    outputLocation: outputPath,
    ffmpegOverride: (options) => {
      // Optimize for social media
      return {
        ...options,
        '-movflags': 'faststart',
        '-pix_fmt': 'yuv420p',
      };
    },
  });

  return outputPath;
}
```

## API Routes Pattern

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { researchTopic } from '@/lib/crawler/research';
import { generateContent } from '@/lib/ai/content-generator';

export async function POST(request: NextRequest) {
  try {
    const { keyword, format, language } = await request.json();

    // Step 1: Research
    const research = await researchTopic({
      keyword,
      sources: ['techcrunch', 'twitter'],
      timeRange: '24h'
    });

    // Step 2: Generate content
    const content = await generateContent({
      research,
      format,
      language,
      provider: 'claude'
    });

    // Step 3: Return or queue for video rendering
    return NextResponse.json({
      success: true,
      content,
      research: research.insights
    });

  } catch (error) {
    return NextResponse.json(
      { error: error.message },
      { status: 500 }
    );
  }
}
```

## Complete Pipeline Example

```typescript
import { ContentPipeline } from '@/lib/pipeline';

async function runFullPipeline(keyword: string) {
  const pipeline = new ContentPipeline({
    aiProvider: 'claude',
    videoEnabled: true,
    autoPublish: false
  });

  // Execute full pipeline
  const result = await pipeline.execute({
    keyword,
    formats: ['toplist', 'how-to'],
    languages: ['en', 'vi'],
    platforms: ['tiktok', 'reels', 'shorts'],
    videoAspectRatio: '9:16' // Vertical for social
  });

  return {
    contentId: result.id,
    articles: result.articles,
    videos: result.videos,
    metadata: result.metadata
  };
}

// Usage
const output = await runFullPipeline('AI marketing automation');
console.log(`Generated ${output.articles.length} articles`);
console.log(`Rendered ${output.videos.length} videos`);
```

## Content Format Templates

```typescript
// src/lib/content/formats.ts
export const contentFormats = {
  toplist: {
    structure: ['intro', 'items', 'conclusion'],
    prompt: 'Create a numbered list of top items related to {keyword}...',
    minItems: 5,
    maxItems: 10
  },
  
  pov: {
    structure: ['hook', 'perspective', 'arguments', 'conclusion'],
    prompt: 'Write from a unique perspective about {keyword}...',
    tone: 'opinionated'
  },
  
  caseStudy: {
    structure: ['background', 'challenge', 'solution', 'results'],
    prompt: 'Analyze a real-world case study about {keyword}...',
    includeData: true
  },
  
  howTo: {
    structure: ['intro', 'steps', 'tips', 'conclusion'],
    prompt: 'Write a step-by-step guide for {keyword}...',
    actionOriented: true
  }
};
```

## Crawler Integration

```typescript
// src/lib/crawler/sources.ts
import axios from 'axios';

export async function crawlTechCrunch(keyword: string) {
  const response = await axios.get('https://api.rapidapi.com/techcrunch/search', {
    params: { q: keyword, limit: 10 },
    headers: {
      'X-RapidAPI-Key': process.env.RAPIDAPI_KEY,
      'X-RapidAPI-Host': 'techcrunch.p.rapidapi.com'
    }
  });

  return response.data.articles.map(article => ({
    title: article.title,
    summary: article.excerpt,
    url: article.url,
    publishedAt: article.date,
    source: 'TechCrunch'
  }));
}

export async function aggregateResearch(keyword: string) {
  const [techcrunch, twitter, linkedin] = await Promise.all([
    crawlTechCrunch(keyword),
    crawlTwitter(keyword),
    crawlLinkedIn(keyword)
  ]);

  return {
    allSources: [...techcrunch, ...twitter, ...linkedin],
    summary: generateSummary([...techcrunch, ...twitter, ...linkedin]),
    topInsights: extractInsights([...techcrunch, ...twitter, ...linkedin])
  };
}
```

## Video Template Example (Remotion)

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, interpolate } from 'remotion';
import React from 'react';

export const ContentVideo: React.FC<{
  title: string;
  highlights: string[];
  statistics: any[];
}> = ({ title, highlights, statistics }) => {
  const frame = useCurrentFrame();
  
  const titleOpacity = interpolate(frame, [0, 30], [0, 1], {
    extrapolateRight: 'clamp',
  });

  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a2e' }}>
      <div style={{ padding: 60, color: 'white' }}>
        <h1 style={{ 
          opacity: titleOpacity,
          fontSize: 48,
          marginBottom: 40 
        }}>
          {title}
        </h1>
        
        {highlights.map((highlight, i) => (
          <div key={i} style={{
            opacity: interpolate(
              frame,
              [30 + i * 30, 60 + i * 30],
              [0, 1],
              { extrapolateRight: 'clamp' }
            ),
            marginBottom: 20,
            fontSize: 24
          }}>
            • {highlight}
          </div>
        ))}
      </div>
    </AbsoluteFill>
  );
};
```

## Development Server

```bash
# Run Next.js development server
npm run dev

# Access at http://localhost:3000
```

## Common Patterns

### Batch Content Generation

```typescript
async function batchGenerate(keywords: string[]) {
  const results = [];
  
  for (const keyword of keywords) {
    const content = await runFullPipeline(keyword);
    results.push(content);
    
    // Rate limiting
    await new Promise(resolve => setTimeout(resolve, 2000));
  }
  
  return results;
}
```

### Custom AI Provider

```typescript
// src/lib/ai/custom-provider.ts
export class CustomAIProvider {
  async generate(prompt: string, options: any) {
    // Implement your custom AI logic
    const provider = options.provider === 'claude' 
      ? this.useClaude 
      : this.useOpenAI;
    
    return await provider(prompt, options);
  }
  
  private async useClaude(prompt: string, options: any) {
    // Claude implementation
  }
  
  private async useOpenAI(prompt: string, options: any) {
    // OpenAI implementation
  }
}
```

## Troubleshooting

### API Rate Limits
```typescript
// Implement rate limiting
import pLimit from 'p-limit';

const limit = pLimit(3); // 3 concurrent requests max

const promises = keywords.map(keyword =>
  limit(() => generateContent(keyword))
);

await Promise.all(promises);
```

### Video Rendering Fails
```typescript
// Add error handling and retry logic
async function renderWithRetry(content: any, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await renderContentVideo(content, `output-${Date.now()}.mp4`);
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      console.log(`Retry ${i + 1}/${maxRetries}`);
      await new Promise(resolve => setTimeout(resolve, 5000));
    }
  }
}
```

### Memory Issues with Large Batches
```typescript
// Process in chunks
async function processInChunks<T>(items: T[], chunkSize: number, processor: (item: T) => Promise<any>) {
  const results = [];
  
  for (let i = 0; i < items.length; i += chunkSize) {
    const chunk = items.slice(i, i + chunkSize);
    const chunkResults = await Promise.all(chunk.map(processor));
    results.push(...chunkResults);
    
    // Force garbage collection between chunks
    if (global.gc) global.gc();
  }
  
  return results;
}
```

### API Key Validation
```typescript
// Validate environment variables on startup
function validateConfig() {
  const required = [
    'OPENAI_API_KEY',
    'ANTHROPIC_API_KEY',
    'RAPIDAPI_KEY'
  ];
  
  const missing = required.filter(key => !process.env[key]);
  
  if (missing.length > 0) {
    throw new Error(`Missing required environment variables: ${missing.join(', ')}`);
  }
}

validateConfig();
```
