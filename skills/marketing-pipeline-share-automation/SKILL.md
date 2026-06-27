```markdown
---
name: marketing-pipeline-share-automation
description: AI-powered content automation pipeline for research, scriptwriting, and video generation with Claude/OpenAI
triggers:
  - how do I automate content creation with AI
  - set up marketing pipeline automation
  - generate videos from content automatically
  - use Claude and OpenAI for content research
  - automate social media content creation
  - create AI content pipeline with Remotion
  - build automated marketing content system
  - research and generate content with AI agents
---

# Marketing Pipeline Share Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the Ultimate AI Content Pipeline - an end-to-end content automation system that handles research, scriptwriting, content generation, and video rendering using Claude 3, OpenAI, and Remotion.

## What This Project Does

Marketing Pipeline Share is a TypeScript-based automation system that:

- **Auto-researches** trending topics from TechCrunch, a16z, Twitter/X, LinkedIn within the last 24 hours
- **Generates content** in multiple formats (Toplist, POV, Case Study, How-to) using Claude/OpenAI
- **Multi-language support** with automatic English/Vietnamese content generation
- **Renders videos** automatically using Remotion for Reels, TikTok, and Shorts
- **Provides a Next.js interface** for content management and scheduling

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

Create a `.env.local` file in the project root:

```bash
# AI Provider Keys
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research API (RapidAPI)
RAPIDAPI_KEY=your_rapidapi_key

# Remotion Configuration
REMOTION_LICENSE_KEY=your_remotion_license_key

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Next.js Configuration
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

## Core Architecture

### Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI provider integrations
│   │   ├── research/    # Content research modules
│   │   ├── video/       # Remotion video generation
│   │   └── utils/       # Utility functions
│   └── types/           # TypeScript type definitions
├── remotion/            # Remotion video templates
├── public/              # Static assets
└── .env.local          # Environment variables
```

## Key APIs and Usage Patterns

### 1. Content Research Module

```typescript
// src/lib/research/scraper.ts
import { RapidAPIClient } from '@/lib/api/rapidapi';

interface ResearchSource {
  platform: 'techcrunch' | 'a16z' | 'twitter' | 'linkedin';
  timeRange: '24h' | '7d' | '30d';
  keywords: string[];
}

export async function researchContent(source: ResearchSource) {
  const client = new RapidAPIClient(process.env.RAPIDAPI_KEY!);
  
  const articles = await client.search({
    platform: source.platform,
    query: source.keywords.join(' OR '),
    timeRange: source.timeRange,
  });
  
  return articles.map(article => ({
    title: article.title,
    url: article.url,
    publishedAt: article.publishedAt,
    summary: article.excerpt,
    insights: extractInsights(article.content),
  }));
}

function extractInsights(content: string): string[] {
  // Extract data-backed insights, statistics, quotes
  const insights: string[] = [];
  
  // Pattern matching for statistics
  const statPattern = /(\d+%|\$\d+[MBK]?|\d+[MBK]?\+?)\s+([^.]+)/g;
  const matches = content.matchAll(statPattern);
  
  for (const match of matches) {
    insights.push(match[0]);
  }
  
  return insights;
}
```

### 2. AI Content Generation with Claude/OpenAI

```typescript
// src/lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

type ContentFormat = 'toplist' | 'pov' | 'case-study' | 'how-to';
type Tone = 'expert' | 'friendly' | 'humorous';
type Language = 'en' | 'vi';

interface ContentRequest {
  topic: string;
  format: ContentFormat;
  tone: Tone;
  language: Language;
  researchData: any[];
}

export class ContentGenerator {
  private anthropic: Anthropic;
  private openai: OpenAI;

  constructor() {
    this.anthropic = new Anthropic({
      apiKey: process.env.ANTHROPIC_API_KEY,
    });
    this.openai = new OpenAI({
      apiKey: process.env.OPENAI_API_KEY,
    });
  }

  async generateWithClaude(request: ContentRequest): Promise<string> {
    const systemPrompt = this.buildSystemPrompt(request);
    const userPrompt = this.buildUserPrompt(request);

    const message = await this.anthropic.messages.create({
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

  async generateWithOpenAI(request: ContentRequest): Promise<string> {
    const systemPrompt = this.buildSystemPrompt(request);
    const userPrompt = this.buildUserPrompt(request);

    const completion = await this.openai.chat.completions.create({
      model: 'gpt-4-turbo-preview',
      messages: [
        { role: 'system', content: systemPrompt },
        { role: 'user', content: userPrompt },
      ],
      temperature: 0.7,
      max_tokens: 4096,
    });

    return completion.choices[0]?.message?.content || '';
  }

  private buildSystemPrompt(request: ContentRequest): string {
    const toneDescriptions = {
      expert: 'professional, authoritative, data-driven',
      friendly: 'conversational, approachable, engaging',
      humorous: 'witty, entertaining, light-hearted',
    };

    const formatInstructions = {
      toplist: 'Create a numbered list format with clear headers and bullet points',
      pov: 'Write from a personal perspective with strong opinions and insights',
      'case-study': 'Structure as: Problem → Solution → Results with specific data',
      'how-to': 'Provide step-by-step instructions with actionable tips',
    };

    return `You are an expert content writer specializing in ${request.format} articles.
Write in a ${toneDescriptions[request.tone]} tone in ${request.language === 'en' ? 'English' : 'Vietnamese'}.
${formatInstructions[request.format]}
Use the research data provided to create data-backed, insightful content.`;
  }

  private buildUserPrompt(request: ContentRequest): string {
    const researchSummary = request.researchData
      .map(item => `- ${item.title}: ${item.summary}`)
      .join('\n');

    return `Topic: ${request.topic}

Research Data:
${researchSummary}

Create a comprehensive ${request.format} article about this topic using the research data above.`;
  }
}
```

### 3. Remotion Video Generation

```typescript
// src/lib/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import { webpack } from '@remotion/cli';
import path from 'path';

interface VideoConfig {
  content: string;
  format: 'reels' | 'tiktok' | 'shorts'; // 9:16 vertical
  duration: number; // in seconds
  outputPath: string;
}

export class VideoRenderer {
  async renderContentVideo(config: VideoConfig): Promise<string> {
    const compositionId = 'ContentVideo';
    const bundleLocation = await bundle({
      entryPoint: path.resolve('./remotion/index.ts'),
      webpackOverride: webpack,
    });

    const composition = await selectComposition({
      serveUrl: bundleLocation,
      id: compositionId,
      inputProps: {
        content: config.content,
        format: config.format,
      },
    });

    const dimensions = this.getFormatDimensions(config.format);

    await renderMedia({
      composition: {
        ...composition,
        width: dimensions.width,
        height: dimensions.height,
        fps: 30,
        durationInFrames: config.duration * 30,
      },
      serveUrl: bundleLocation,
      codec: 'h264',
      outputLocation: config.outputPath,
      inputProps: {
        content: config.content,
        format: config.format,
      },
    });

    return config.outputPath;
  }

  private getFormatDimensions(format: string): { width: number; height: number } {
    const formats = {
      reels: { width: 1080, height: 1920 },
      tiktok: { width: 1080, height: 1920 },
      shorts: { width: 1080, height: 1920 },
    };
    return formats[format as keyof typeof formats];
  }
}
```

### 4. Remotion Video Component

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, Sequence, useCurrentFrame, useVideoConfig } from 'remotion';
import React from 'react';

interface ContentVideoProps {
  content: string;
  format: 'reels' | 'tiktok' | 'shorts';
}

export const ContentVideo: React.FC<ContentVideoProps> = ({ content, format }) => {
  const frame = useCurrentFrame();
  const { fps, durationInFrames } = useVideoConfig();

  const sections = content.split('\n\n').filter(Boolean);

  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a1a' }}>
      {sections.map((section, index) => {
        const sectionDuration = Math.floor(durationInFrames / sections.length);
        const startFrame = index * sectionDuration;

        return (
          <Sequence
            key={index}
            from={startFrame}
            durationInFrames={sectionDuration}
          >
            <AbsoluteFill
              style={{
                justifyContent: 'center',
                alignItems: 'center',
                padding: 60,
              }}
            >
              <div
                style={{
                  fontSize: 48,
                  fontWeight: 'bold',
                  color: 'white',
                  textAlign: 'center',
                  lineHeight: 1.4,
                  opacity: Math.min(1, (frame - startFrame) / 10),
                }}
              >
                {section}
              </div>
            </AbsoluteFill>
          </Sequence>
        );
      })}
    </AbsoluteFill>
  );
};
```

### 5. Complete Pipeline Integration

```typescript
// src/lib/pipeline/content-pipeline.ts
import { researchContent } from '@/lib/research/scraper';
import { ContentGenerator } from '@/lib/ai/content-generator';
import { VideoRenderer } from '@/lib/video/renderer';

interface PipelineConfig {
  keyword: string;
  contentFormat: 'toplist' | 'pov' | 'case-study' | 'how-to';
  tone: 'expert' | 'friendly' | 'humorous';
  languages: ('en' | 'vi')[];
  generateVideo: boolean;
  videoFormat?: 'reels' | 'tiktok' | 'shorts';
}

export class ContentPipeline {
  private generator: ContentGenerator;
  private renderer: VideoRenderer;

  constructor() {
    this.generator = new ContentGenerator();
    this.renderer = new VideoRenderer();
  }

  async execute(config: PipelineConfig) {
    // Step 1: Research
    console.log('🔍 Researching content...');
    const researchData = await researchContent({
      platform: 'techcrunch',
      timeRange: '24h',
      keywords: [config.keyword],
    });

    const results = [];

    // Step 2: Generate Content for each language
    for (const language of config.languages) {
      console.log(`✍️ Generating ${language} content...`);
      
      const content = await this.generator.generateWithClaude({
        topic: config.keyword,
        format: config.contentFormat,
        tone: config.tone,
        language,
        researchData,
      });

      results.push({
        language,
        content,
        videoPath: null,
      });

      // Step 3: Generate Video (if requested)
      if (config.generateVideo && config.videoFormat) {
        console.log(`🎬 Rendering ${language} video...`);
        
        const outputPath = `./output/video-${language}-${Date.now()}.mp4`;
        const videoPath = await this.renderer.renderContentVideo({
          content,
          format: config.videoFormat,
          duration: 60, // 60 seconds
          outputPath,
        });

        results[results.length - 1].videoPath = videoPath;
      }
    }

    return results;
  }
}
```

### 6. Next.js API Route Example

```typescript
// src/app/api/pipeline/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { ContentPipeline } from '@/lib/pipeline/content-pipeline';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    
    const pipeline = new ContentPipeline();
    const results = await pipeline.execute({
      keyword: body.keyword,
      contentFormat: body.format || 'toplist',
      tone: body.tone || 'friendly',
      languages: body.languages || ['en', 'vi'],
      generateVideo: body.generateVideo || false,
      videoFormat: body.videoFormat || 'reels',
    });

    return NextResponse.json({ 
      success: true, 
      results 
    });
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { success: false, error: (error as Error).message },
      { status: 500 }
    );
  }
}
```

### 7. Frontend Integration

```typescript
// src/app/page.tsx
'use client';

import { useState } from 'react';

export default function Home() {
  const [keyword, setKeyword] = useState('');
  const [loading, setLoading] = useState(false);
  const [results, setResults] = useState<any>(null);

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    setLoading(true);

    try {
      const response = await fetch('/api/pipeline', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          keyword,
          format: 'toplist',
          tone: 'friendly',
          languages: ['en', 'vi'],
          generateVideo: true,
          videoFormat: 'reels',
        }),
      });

      const data = await response.json();
      setResults(data.results);
    } catch (error) {
      console.error('Error:', error);
    } finally {
      setLoading(false);
    }
  };

  return (
    <main className="container mx-auto p-8">
      <h1 className="text-4xl font-bold mb-8">AI Content Pipeline</h1>
      
      <form onSubmit={handleSubmit} className="mb-8">
        <input
          type="text"
          value={keyword}
          onChange={(e) => setKeyword(e.target.value)}
          placeholder="Enter keyword..."
          className="border p-2 mr-2 w-64"
        />
        <button 
          type="submit" 
          disabled={loading}
          className="bg-blue-500 text-white px-4 py-2 rounded"
        >
          {loading ? 'Processing...' : 'Generate Content'}
        </button>
      </form>

      {results && (
        <div className="space-y-4">
          {results.map((result: any, index: number) => (
            <div key={index} className="border p-4 rounded">
              <h3 className="font-bold mb-2">Language: {result.language}</h3>
              <div className="whitespace-pre-wrap mb-4">{result.content}</div>
              {result.videoPath && (
                <p className="text-sm text-gray-600">Video: {result.videoPath}</p>
              )}
            </div>
          ))}
        </div>
      )}
    </main>
  );
}
```

## Common Workflows

### Basic Content Generation

```bash
# Start development server
npm run dev

# Navigate to http://localhost:3000
# Enter keyword and click "Generate Content"
```

### Programmatic Usage

```typescript
import { ContentPipeline } from '@/lib/pipeline/content-pipeline';

const pipeline = new ContentPipeline();

const results = await pipeline.execute({
  keyword: 'AI automation trends 2026',
  contentFormat: 'toplist',
  tone: 'expert',
  languages: ['en'],
  generateVideo: true,
  videoFormat: 'reels',
});

console.log(results);
```

## Troubleshooting

### API Key Issues

**Problem:** `Error: Missing API key`

**Solution:** Ensure all required API keys are set in `.env.local`:
```bash
ANTHROPIC_API_KEY=sk-ant-...
OPENAI_API_KEY=sk-...
RAPIDAPI_KEY=...
```

### Remotion Rendering Errors

**Problem:** Video fails to render

**Solution:** 
- Verify Remotion license key is valid
- Check that ffmpeg is installed: `ffmpeg -version`
- Ensure sufficient disk space in output directory

```bash
# Install ffmpeg (macOS)
brew install ffmpeg

# Install ffmpeg (Ubuntu)
sudo apt-get install ffmpeg
```

### Content Generation Timeout

**Problem:** API requests timeout

**Solution:** Increase timeout settings:

```typescript
const message = await this.anthropic.messages.create({
  model: 'claude-3-5-sonnet-20241022',
  max_tokens: 4096,
  timeout: 120000, // 2 minutes
  // ...
});
```

### Research Data Not Found

**Problem:** No articles returned from research

**Solution:** 
- Verify RapidAPI subscription is active
- Check keyword relevance and timeRange
- Try different platforms: 'techcrunch', 'a16z', 'twitter'

## Performance Optimization

### Parallel Content Generation

```typescript
async executeParallel(config: PipelineConfig) {
  const researchData = await researchContent({
    platform: 'techcrunch',
    timeRange: '24h',
    keywords: [config.keyword],
  });

  // Generate all languages in parallel
  const contentPromises = config.languages.map(language =>
    this.generator.generateWithClaude({
      topic: config.keyword,
      format: config.contentFormat,
      tone: config.tone,
      language,
      researchData,
    })
  );

  const contents = await Promise.all(contentPromises);
  
  return contents.map((content, index) => ({
    language: config.languages[index],
    content,
  }));
}
```

### Caching Research Results

```typescript
import NodeCache from 'node-cache';

const cache = new NodeCache({ stdTTL: 3600 }); // 1 hour

async function cachedResearch(keywords: string[]) {
  const cacheKey = keywords.join('-');
  const cached = cache.get(cacheKey);
  
  if (cached) {
    return cached;
  }
  
  const results = await researchContent({
    platform: 'techcrunch',
    timeRange: '24h',
    keywords,
  });
  
  cache.set(cacheKey, results);
  return results;
}
```

This skill equips AI coding agents with comprehensive knowledge to implement, customize, and troubleshoot the Marketing Pipeline Share automation system for end-to-end content creation workflows.
```
