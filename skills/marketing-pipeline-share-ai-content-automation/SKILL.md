---
name: marketing-pipeline-share-ai-content-automation
description: Automate content creation from research to video generation using Claude/OpenAI and Remotion
triggers:
  - how do I automate content creation with AI
  - set up automated marketing pipeline with video generation
  - create AI-powered content workflow from research to publishing
  - generate videos from articles automatically
  - build content automation system with Claude and OpenAI
  - automate research and scriptwriting for marketing content
  - use Remotion to render videos from AI-generated content
  - create end-to-end content pipeline with AI
---

# Marketing Pipeline Share AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to help developers use **Ultimate AI Content Pipeline**, a complete content automation system that handles research, scriptwriting, content generation, and video rendering. The pipeline integrates Claude 3, OpenAI, and Remotion to transform keywords into multi-format content including articles and videos.

## What It Does

Marketing Pipeline Share automates the entire content creation workflow:

1. **Auto-Research**: Crawls recent news from TechCrunch, a16z, Twitter, LinkedIn within 24 hours
2. **AI Content Generation**: Creates content in multiple formats (toplist, POV, case study, how-to) using Claude/OpenAI
3. **Multi-language Support**: Generates content in both English and Vietnamese
4. **Video Rendering**: Automatically renders infographics and short videos using Remotion
5. **Platform Optimization**: Exports videos for Reels, TikTok, Shorts

## Installation

### Prerequisites

```bash
node -v  # Requires Node.js 18+
npm -v   # or yarn/pnpm
```

### Setup

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
yarn install

# Copy environment configuration
cp .env.example .env
```

### Environment Configuration

Edit `.env` file with required API keys:

```bash
# AI Services
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Remotion Configuration
REMOTION_LICENSE_KEY=your_remotion_license

# Database (if applicable)
DATABASE_URL=postgresql://user:password@localhost:5432/content_pipeline

# App Configuration
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

### Start Development Server

```bash
npm run dev
# or
yarn dev

# Access at http://localhost:3000
```

## Core Architecture

### Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── services/         # API integrations
│   │   ├── research/     # Research crawlers
│   │   ├── ai/           # Claude/OpenAI handlers
│   │   └── video/        # Remotion rendering
│   ├── lib/              # Utilities
│   └── types/            # TypeScript types
├── public/               # Static assets
├── remotion/             # Video templates
└── .env                  # Environment variables
```

## Key Features & Usage

### 1. Research Automation

```typescript
// src/services/research/crawler.ts
import { ResearchService } from '@/services/research';

interface ResearchOptions {
  keywords: string[];
  sources: ('techcrunch' | 'a16z' | 'twitter' | 'linkedin')[];
  timeRange: '24h' | '7d' | '30d';
}

async function performResearch(options: ResearchOptions) {
  const researchService = new ResearchService({
    rapidApiKey: process.env.RAPIDAPI_KEY!,
  });

  const results = await researchService.crawl({
    keywords: options.keywords,
    sources: options.sources,
    timeRange: options.timeRange,
  });

  return results;
}

// Usage
const insights = await performResearch({
  keywords: ['AI marketing', 'content automation'],
  sources: ['techcrunch', 'twitter'],
  timeRange: '24h',
});
```

### 2. AI Content Generation with Claude

```typescript
// src/services/ai/claude-generator.ts
import Anthropic from '@anthropic-ai/sdk';

interface ContentGenerationParams {
  topic: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  researchData: any[];
}

export class ClaudeContentGenerator {
  private client: Anthropic;

  constructor() {
    this.client = new Anthropic({
      apiKey: process.env.ANTHROPIC_API_KEY!,
    });
  }

  async generateContent(params: ContentGenerationParams) {
    const prompt = this.buildPrompt(params);

    const message = await this.client.messages.create({
      model: 'claude-3-5-sonnet-20241022',
      max_tokens: 4096,
      messages: [
        {
          role: 'user',
          content: prompt,
        },
      ],
    });

    return this.parseResponse(message.content);
  }

  private buildPrompt(params: ContentGenerationParams): string {
    const formatInstructions = {
      'toplist': 'Create a numbered list of top items with detailed explanations',
      'pov': 'Write from a specific point of view with personal insights',
      'case-study': 'Analyze a specific example with data and outcomes',
      'how-to': 'Provide step-by-step instructions with actionable tips',
    };

    return `
You are an expert content creator. Generate ${params.language === 'vi' ? 'Vietnamese' : 'English'} content about "${params.topic}".

Format: ${formatInstructions[params.format]}
Tone: ${params.tone}

Research data:
${JSON.stringify(params.researchData, null, 2)}

Requirements:
- Use data-backed insights from the research
- Include specific examples and statistics
- Make it engaging and actionable
- Optimize for SEO and social sharing
`;
  }

  private parseResponse(content: any) {
    return {
      title: '',
      body: content[0].text,
      metadata: {},
    };
  }
}
```

### 3. OpenAI Integration Alternative

```typescript
// src/services/ai/openai-generator.ts
import OpenAI from 'openai';

export class OpenAIContentGenerator {
  private client: OpenAI;

  constructor() {
    this.client = new OpenAI({
      apiKey: process.env.OPENAI_API_KEY!,
    });
  }

  async generateContent(params: ContentGenerationParams) {
    const completion = await this.client.chat.completions.create({
      model: 'gpt-4-turbo-preview',
      messages: [
        {
          role: 'system',
          content: 'You are an expert content marketer and writer.',
        },
        {
          role: 'user',
          content: this.buildPrompt(params),
        },
      ],
      temperature: 0.7,
      max_tokens: 3000,
    });

    return {
      content: completion.choices[0].message.content,
      usage: completion.usage,
    };
  }

  private buildPrompt(params: ContentGenerationParams): string {
    // Similar to Claude implementation
    return `Generate ${params.format} content about ${params.topic}...`;
  }
}
```

### 4. Video Generation with Remotion

```typescript
// src/services/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  content: {
    title: string;
    keyPoints: string[];
    images: string[];
  };
  platform: 'reels' | 'tiktok' | 'shorts';
}

export class VideoRenderer {
  async renderVideo(config: VideoConfig) {
    const bundleLocation = await bundle({
      entryPoint: path.join(process.cwd(), 'remotion/index.ts'),
      webpackOverride: (config) => config,
    });

    const compositionId = this.getCompositionId(config.platform);
    const composition = await selectComposition({
      serveUrl: bundleLocation,
      id: compositionId,
      inputProps: {
        title: config.content.title,
        keyPoints: config.content.keyPoints,
        images: config.content.images,
      },
    });

    const outputPath = path.join(
      process.cwd(),
      'public',
      'videos',
      `${Date.now()}.mp4`
    );

    await renderMedia({
      composition,
      serveUrl: bundleLocation,
      codec: 'h264',
      outputLocation: outputPath,
      inputProps: composition.props,
    });

    return outputPath;
  }

  private getCompositionId(platform: string): string {
    const compositions = {
      reels: 'InstagramReel',
      tiktok: 'TikTokVideo',
      shorts: 'YouTubeShort',
    };
    return compositions[platform] || compositions.reels;
  }
}
```

### 5. Remotion Video Template

```typescript
// remotion/InstagramReel.tsx
import { AbsoluteFill, Sequence, useCurrentFrame, useVideoConfig } from 'remotion';
import React from 'react';

interface ReelProps {
  title: string;
  keyPoints: string[];
  images: string[];
}

export const InstagramReel: React.FC<ReelProps> = ({ title, keyPoints, images }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  const opacity = Math.min(1, frame / 30);

  return (
    <AbsoluteFill style={{ backgroundColor: '#000' }}>
      {/* Title sequence */}
      <Sequence from={0} durationInFrames={fps * 3}>
        <AbsoluteFill
          style={{
            justifyContent: 'center',
            alignItems: 'center',
            opacity,
          }}
        >
          <h1 style={{ color: 'white', fontSize: 60, textAlign: 'center', padding: 40 }}>
            {title}
          </h1>
        </AbsoluteFill>
      </Sequence>

      {/* Key points sequence */}
      {keyPoints.map((point, index) => (
        <Sequence
          key={index}
          from={fps * (3 + index * 4)}
          durationInFrames={fps * 4}
        >
          <AbsoluteFill
            style={{
              justifyContent: 'center',
              alignItems: 'center',
              padding: 40,
            }}
          >
            {images[index] && (
              <img
                src={images[index]}
                style={{
                  width: '100%',
                  height: '60%',
                  objectFit: 'cover',
                  borderRadius: 20,
                }}
                alt=""
              />
            )}
            <p style={{ color: 'white', fontSize: 36, marginTop: 20, textAlign: 'center' }}>
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
// src/services/pipeline/orchestrator.ts
import { ResearchService } from '@/services/research';
import { ClaudeContentGenerator } from '@/services/ai/claude-generator';
import { VideoRenderer } from '@/services/video/renderer';

interface PipelineConfig {
  keyword: string;
  contentFormat: 'toplist' | 'pov' | 'case-study' | 'how-to';
  languages: ('en' | 'vi')[];
  generateVideo: boolean;
  platform?: 'reels' | 'tiktok' | 'shorts';
}

export class ContentPipeline {
  private research: ResearchService;
  private contentGenerator: ClaudeContentGenerator;
  private videoRenderer: VideoRenderer;

  constructor() {
    this.research = new ResearchService({
      rapidApiKey: process.env.RAPIDAPI_KEY!,
    });
    this.contentGenerator = new ClaudeContentGenerator();
    this.videoRenderer = new VideoRenderer();
  }

  async execute(config: PipelineConfig) {
    // Step 1: Research
    console.log('🔍 Starting research phase...');
    const researchData = await this.research.crawl({
      keywords: [config.keyword],
      sources: ['techcrunch', 'twitter', 'linkedin'],
      timeRange: '24h',
    });

    // Step 2: Generate content for each language
    console.log('✍️ Generating content...');
    const contentResults = await Promise.all(
      config.languages.map(async (language) => {
        const content = await this.contentGenerator.generateContent({
          topic: config.keyword,
          format: config.contentFormat,
          language,
          tone: 'expert',
          researchData,
        });

        return { language, content };
      })
    );

    // Step 3: Generate video if requested
    let videoPath = null;
    if (config.generateVideo && config.platform) {
      console.log('🎬 Rendering video...');
      const primaryContent = contentResults[0].content;

      videoPath = await this.videoRenderer.renderVideo({
        content: {
          title: primaryContent.title,
          keyPoints: this.extractKeyPoints(primaryContent.body),
          images: [], // Add image extraction logic
        },
        platform: config.platform,
      });
    }

    return {
      research: researchData,
      content: contentResults,
      video: videoPath,
    };
  }

  private extractKeyPoints(body: string): string[] {
    // Extract key bullet points or main ideas from content
    const lines = body.split('\n').filter((line) => line.trim().startsWith('-'));
    return lines.slice(0, 5).map((line) => line.replace(/^-\s*/, '').trim());
  }
}
```

### 7. API Route Example (Next.js)

```typescript
// src/app/api/pipeline/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { ContentPipeline } from '@/services/pipeline/orchestrator';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const { keyword, format, languages, generateVideo, platform } = body;

    if (!keyword) {
      return NextResponse.json(
        { error: 'Keyword is required' },
        { status: 400 }
      );
    }

    const pipeline = new ContentPipeline();
    const results = await pipeline.execute({
      keyword,
      contentFormat: format || 'toplist',
      languages: languages || ['en'],
      generateVideo: generateVideo || false,
      platform: platform || 'reels',
    });

    return NextResponse.json({
      success: true,
      data: results,
    });
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { error: 'Pipeline execution failed', details: error.message },
      { status: 500 }
    );
  }
}
```

### 8. React Component for UI

```typescript
// src/components/PipelineForm.tsx
'use client';

import { useState } from 'react';

export function PipelineForm() {
  const [keyword, setKeyword] = useState('');
  const [format, setFormat] = useState('toplist');
  const [loading, setLoading] = useState(false);
  const [results, setResults] = useState(null);

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    setLoading(true);

    try {
      const response = await fetch('/api/pipeline', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          keyword,
          format,
          languages: ['en', 'vi'],
          generateVideo: true,
          platform: 'reels',
        }),
      });

      const data = await response.json();
      setResults(data);
    } catch (error) {
      console.error('Error:', error);
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="max-w-2xl mx-auto p-6">
      <form onSubmit={handleSubmit} className="space-y-4">
        <div>
          <label className="block text-sm font-medium mb-2">
            Topic/Keyword
          </label>
          <input
            type="text"
            value={keyword}
            onChange={(e) => setKeyword(e.target.value)}
            className="w-full px-4 py-2 border rounded-lg"
            placeholder="e.g., AI Marketing Trends 2024"
            required
          />
        </div>

        <div>
          <label className="block text-sm font-medium mb-2">
            Content Format
          </label>
          <select
            value={format}
            onChange={(e) => setFormat(e.target.value)}
            className="w-full px-4 py-2 border rounded-lg"
          >
            <option value="toplist">Top List</option>
            <option value="pov">Point of View</option>
            <option value="case-study">Case Study</option>
            <option value="how-to">How-To Guide</option>
          </select>
        </div>

        <button
          type="submit"
          disabled={loading}
          className="w-full bg-blue-600 text-white py-2 px-4 rounded-lg hover:bg-blue-700 disabled:opacity-50"
        >
          {loading ? 'Generating...' : 'Generate Content'}
        </button>
      </form>

      {results && (
        <div className="mt-8">
          <h3 className="text-lg font-bold mb-4">Results</h3>
          <pre className="bg-gray-100 p-4 rounded-lg overflow-auto">
            {JSON.stringify(results, null, 2)}
          </pre>
        </div>
      )}
    </div>
  );
}
```

## CLI Commands

```bash
# Development
npm run dev              # Start development server
npm run build           # Build for production
npm run start           # Start production server

# Remotion specific
npm run remotion        # Open Remotion Studio
npm run render          # Render videos from CLI

# Type checking
npm run type-check      # Run TypeScript compiler

# Linting
npm run lint            # Run ESLint
```

## Common Patterns

### Error Handling

```typescript
import { retry } from '@/lib/utils';

async function safeAPICall<T>(fn: () => Promise<T>): Promise<T> {
  try {
    return await retry(fn, { maxAttempts: 3, delay: 1000 });
  } catch (error) {
    console.error('API call failed after retries:', error);
    throw new Error('Service temporarily unavailable');
  }
}
```

### Rate Limiting

```typescript
import { Ratelimit } from '@upstash/ratelimit';
import { Redis } from '@upstash/redis';

const ratelimit = new Ratelimit({
  redis: Redis.fromEnv(),
  limiter: Ratelimit.slidingWindow(10, '1 h'),
});

export async function checkRateLimit(identifier: string) {
  const { success } = await ratelimit.limit(identifier);
  return success;
}
```

## Troubleshooting

### API Key Issues

```typescript
// Verify API keys are loaded
if (!process.env.ANTHROPIC_API_KEY) {
  throw new Error('ANTHROPIC_API_KEY is not set in environment variables');
}
```

### Remotion Rendering Errors

```bash
# Clear Remotion cache
rm -rf node_modules/.cache/remotion

# Rebuild bundle
npm run build
```

### Memory Issues with Large Videos

```typescript
// Increase Node.js memory limit
// package.json
{
  "scripts": {
    "render": "NODE_OPTIONS='--max-old-space-size=4096' remotion render"
  }
}
```

### TypeScript Errors

```bash
# Regenerate types
npm run type-check

# Clear Next.js cache
rm -rf .next
npm run dev
```

This skill enables comprehensive automation of content marketing workflows from research through video generation using modern AI services and rendering technologies.
