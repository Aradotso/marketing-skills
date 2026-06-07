---
name: marketing-pipeline-share-ai-content
description: Automated AI content pipeline from research to video generation using Claude/OpenAI and Remotion
triggers:
  - generate content with AI marketing pipeline
  - automate content creation from research to video
  - create AI-powered content with Remotion
  - set up automated marketing content pipeline
  - generate videos from content with AI
  - build content automation with Claude and OpenAI
  - create AI content research pipeline
  - automate content workflow with marketing pipeline
---

# Marketing Pipeline Share AI Content

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

Marketing Pipeline Share is a comprehensive AI-powered content automation system built with TypeScript and Next.js. It automates the entire content creation workflow: from research and article generation to video rendering. The system crawls recent news from sources like TechCrunch, a16z, Twitter/X, and LinkedIn, then uses Claude 3 or OpenAI to generate content in multiple formats (toplist, POV, case study, how-to) and languages (English/Vietnamese). Finally, it renders videos using Remotion.

**Key capabilities:**
- Auto-crawl research from major tech news sources (last 24h)
- AI content generation with Claude/OpenAI in multiple formats and tones
- Automatic video/infographic rendering with Remotion
- Multi-language support (English/Vietnamese)
- Next.js frontend for content management

## Installation

### Prerequisites

```bash
# Node.js 18+ required
node --version

# Clone repository
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
# AI Providers
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research API (RapidAPI for news crawling)
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Remotion (Video rendering)
REMOTION_LICENSE_KEY=your_remotion_license

# Next.js
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

Access at `http://localhost:3000`

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI service integrations
│   │   ├── research/    # Content crawling logic
│   │   └── video/       # Remotion video generation
│   ├── types/           # TypeScript types
│   └── utils/           # Utility functions
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Core Features & Usage

### 1. Research Auto-Crawling

Automatically fetch and analyze recent content from multiple sources:

```typescript
// src/lib/research/crawler.ts
import { fetchNewsFromSources } from '@/lib/research/crawler';

interface NewsSource {
  name: string;
  url: string;
  type: 'techcrunch' | 'a16z' | 'twitter' | 'linkedin';
}

async function gatherResearch(keyword: string): Promise<ResearchData[]> {
  const sources: NewsSource[] = [
    { name: 'TechCrunch', url: 'https://techcrunch.com', type: 'techcrunch' },
    { name: 'a16z', url: 'https://a16z.com/blog', type: 'a16z' }
  ];

  const results = await fetchNewsFromSources({
    keyword,
    sources,
    timeframe: '24h',
    maxResults: 20
  });

  return results;
}

// Usage
const research = await gatherResearch('AI automation');
console.log(`Found ${research.length} relevant articles`);
```

### 2. AI Content Generation

Generate content in multiple formats using Claude or OpenAI:

```typescript
// src/lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

type ContentFormat = 'toplist' | 'pov' | 'case-study' | 'how-to';
type Tone = 'expert' | 'friendly' | 'humorous';
type Language = 'en' | 'vi';

interface ContentConfig {
  format: ContentFormat;
  tone: Tone;
  language: Language;
  researchData: ResearchData[];
}

// Using Claude
async function generateWithClaude(config: ContentConfig): Promise<string> {
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

  return message.content[0].type === 'text' 
    ? message.content[0].text 
    : '';
}

// Using OpenAI
async function generateWithOpenAI(config: ContentConfig): Promise<string> {
  const openai = new OpenAI({
    apiKey: process.env.OPENAI_API_KEY,
  });

  const prompt = buildPrompt(config);

  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [{
      role: 'system',
      content: 'You are an expert content creator specializing in marketing and tech content.'
    }, {
      role: 'user',
      content: prompt
    }],
    max_tokens: 4096,
  });

  return completion.choices[0].message.content || '';
}

function buildPrompt(config: ContentConfig): string {
  const { format, tone, language, researchData } = config;
  
  const researchSummary = researchData
    .map(d => `- ${d.title}: ${d.summary}`)
    .join('\n');

  return `
Create a ${format} article in ${language === 'vi' ? 'Vietnamese' : 'English'} 
with a ${tone} tone based on this research:

${researchSummary}

Requirements:
- Format: ${format}
- Length: 1500-2000 words
- Include data-backed insights
- Add actionable takeaways
- Optimize for SEO
`;
}
```

### 3. Video Generation with Remotion

Render videos from generated content:

```typescript
// src/lib/video/render.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  content: string;
  title: string;
  duration: number;
  format: 'reel' | 'tiktok' | 'short';
}

async function renderContentVideo(config: VideoConfig): Promise<string> {
  // Bundle Remotion composition
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  // Select composition
  const compositions = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title: config.title,
      content: config.content,
      format: config.format,
    },
  });

  const composition = compositions[0];

  // Render video
  const outputLocation = path.join(
    process.cwd(),
    'public',
    'videos',
    `${Date.now()}.mp4`
  );

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: {
      title: config.title,
      content: config.content,
    },
  });

  return outputLocation;
}

// Usage
const videoPath = await renderContentVideo({
  content: generatedArticle,
  title: 'Top 5 AI Tools for 2024',
  duration: 60,
  format: 'reel'
});
```

### 4. Remotion Video Component

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig } from 'remotion';
import React from 'react';

interface ContentVideoProps {
  title: string;
  content: string;
  format: 'reel' | 'tiktok' | 'short';
}

export const ContentVideo: React.FC<ContentVideoProps> = ({
  title,
  content,
  format,
}) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  const opacity = Math.min(1, frame / 30);
  
  // Extract key points from content
  const points = content.split('\n')
    .filter(line => line.trim().startsWith('-'))
    .slice(0, 5);

  return (
    <AbsoluteFill
      style={{
        backgroundColor: '#0f172a',
        justifyContent: 'center',
        alignItems: 'center',
        fontFamily: 'Arial, sans-serif',
      }}
    >
      <div style={{ opacity, padding: '40px', maxWidth: '90%' }}>
        <h1 style={{ color: '#fff', fontSize: '48px', marginBottom: '30px' }}>
          {title}
        </h1>
        <div style={{ color: '#e2e8f0', fontSize: '24px', lineHeight: 1.6 }}>
          {points.map((point, i) => (
            <div
              key={i}
              style={{
                opacity: Math.min(1, Math.max(0, (frame - i * 30) / 30)),
                marginBottom: '20px',
              }}
            >
              {point}
            </div>
          ))}
        </div>
      </div>
    </AbsoluteFill>
  );
};
```

### 5. Complete Pipeline Workflow

```typescript
// src/lib/pipeline/workflow.ts
import { gatherResearch } from '@/lib/research/crawler';
import { generateWithClaude } from '@/lib/ai/content-generator';
import { renderContentVideo } from '@/lib/video/render';

interface PipelineConfig {
  keyword: string;
  format: ContentFormat;
  tone: Tone;
  language: Language;
  generateVideo: boolean;
}

async function runContentPipeline(config: PipelineConfig) {
  try {
    // Step 1: Research
    console.log('🔍 Gathering research...');
    const research = await gatherResearch(config.keyword);
    
    // Step 2: Generate Content
    console.log('✍️ Generating content...');
    const content = await generateWithClaude({
      format: config.format,
      tone: config.tone,
      language: config.language,
      researchData: research,
    });
    
    // Step 3: Render Video (optional)
    let videoPath = null;
    if (config.generateVideo) {
      console.log('🎬 Rendering video...');
      videoPath = await renderContentVideo({
        content,
        title: config.keyword,
        duration: 60,
        format: 'reel',
      });
    }
    
    return {
      success: true,
      content,
      videoPath,
      research: research.length,
    };
  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}

// Usage
const result = await runContentPipeline({
  keyword: 'AI automation trends 2024',
  format: 'toplist',
  tone: 'expert',
  language: 'en',
  generateVideo: true,
});
```

### 6. API Route Example

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline/workflow';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    
    const { keyword, format, tone, language, generateVideo } = body;
    
    // Validate inputs
    if (!keyword || !format) {
      return NextResponse.json(
        { error: 'Missing required fields' },
        { status: 400 }
      );
    }
    
    // Run pipeline
    const result = await runContentPipeline({
      keyword,
      format,
      tone: tone || 'expert',
      language: language || 'en',
      generateVideo: generateVideo || false,
    });
    
    return NextResponse.json(result);
  } catch (error) {
    console.error('API error:', error);
    return NextResponse.json(
      { error: 'Internal server error' },
      { status: 500 }
    );
  }
}
```

## Configuration Patterns

### Multi-Provider AI Strategy

```typescript
// src/lib/ai/provider.ts
type AIProvider = 'claude' | 'openai';

interface AIProviderConfig {
  primary: AIProvider;
  fallback: AIProvider;
}

async function generateContent(
  config: ContentConfig,
  providerConfig: AIProviderConfig
) {
  try {
    if (providerConfig.primary === 'claude') {
      return await generateWithClaude(config);
    } else {
      return await generateWithOpenAI(config);
    }
  } catch (error) {
    console.warn(`Primary provider failed, using fallback...`);
    
    if (providerConfig.fallback === 'claude') {
      return await generateWithClaude(config);
    } else {
      return await generateWithOpenAI(config);
    }
  }
}
```

### Batch Content Generation

```typescript
// src/lib/pipeline/batch.ts
async function generateMultipleContent(
  keywords: string[],
  baseConfig: Partial<ContentConfig>
) {
  const results = await Promise.allSettled(
    keywords.map(keyword =>
      runContentPipeline({
        keyword,
        format: baseConfig.format || 'toplist',
        tone: baseConfig.tone || 'expert',
        language: baseConfig.language || 'en',
        generateVideo: false,
      })
    )
  );
  
  return results.map((result, i) => ({
    keyword: keywords[i],
    success: result.status === 'fulfilled',
    data: result.status === 'fulfilled' ? result.value : null,
    error: result.status === 'rejected' ? result.reason : null,
  }));
}
```

## Common Troubleshooting

### API Key Issues

```typescript
// Validate API keys on startup
function validateEnvironment() {
  const required = [
    'ANTHROPIC_API_KEY',
    'OPENAI_API_KEY',
    'RAPIDAPI_KEY',
  ];
  
  const missing = required.filter(key => !process.env[key]);
  
  if (missing.length > 0) {
    throw new Error(
      `Missing required environment variables: ${missing.join(', ')}`
    );
  }
}
```

### Rate Limiting

```typescript
// src/lib/utils/rate-limiter.ts
class RateLimiter {
  private queue: Array<() => Promise<any>> = [];
  private running = 0;
  
  constructor(private maxConcurrent: number, private delayMs: number) {}
  
  async execute<T>(fn: () => Promise<T>): Promise<T> {
    while (this.running >= this.maxConcurrent) {
      await new Promise(resolve => setTimeout(resolve, this.delayMs));
    }
    
    this.running++;
    try {
      return await fn();
    } finally {
      this.running--;
    }
  }
}

const limiter = new RateLimiter(3, 1000);

// Usage with AI calls
const content = await limiter.execute(() =>
  generateWithClaude(config)
);
```

### Video Rendering Memory Issues

```typescript
// Optimize Remotion rendering for large batches
const renderConfig = {
  concurrency: 2, // Limit concurrent renders
  chromiumOptions: {
    args: [
      '--no-sandbox',
      '--disable-setuid-sandbox',
      '--disable-dev-shm-usage',
    ],
  },
};
```

### Content Quality Validation

```typescript
// src/lib/utils/validator.ts
function validateGeneratedContent(content: string): boolean {
  const minLength = 1500;
  const hasHeadings = /#{1,3}\s/.test(content);
  const hasBulletPoints = /[-*]\s/.test(content);
  
  return (
    content.length >= minLength &&
    hasHeadings &&
    hasBulletPoints
  );
}

// Use in pipeline
if (!validateGeneratedContent(content)) {
  console.warn('Content quality check failed, regenerating...');
  content = await generateWithClaude(config);
}
```

## Best Practices

1. **Always use environment variables** for API keys
2. **Implement rate limiting** for AI API calls
3. **Cache research results** to avoid redundant crawling
4. **Validate generated content** before video rendering
5. **Use fallback providers** for reliability
6. **Monitor API costs** with usage tracking
7. **Test video rendering** on small batches first
8. **Version control prompts** for consistent output quality
