---
name: marketing-pipeline-ai-content-automation
description: Automated AI content pipeline for research, scriptwriting, and video generation using Claude, OpenAI, and Remotion
triggers:
  - how do I automate content creation with AI research
  - set up the marketing pipeline for auto video generation
  - create automated content from research to video
  - configure Claude and OpenAI for content automation
  - generate AI-powered marketing content with Remotion
  - build an automated content pipeline with TypeScript
  - use marketing-pipeline-share for content automation
  - automate content research and video rendering
---

# Marketing Pipeline AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the Ultimate AI Content Pipeline - a complete automated content creation system that handles research, scriptwriting, posting, and video generation using Claude 3, OpenAI, and Remotion.

## What This Project Does

The marketing-pipeline-share project is a TypeScript-based automation system that:

- **Auto-crawls** fresh content from major sources (TechCrunch, a16z, Twitter/X, LinkedIn)
- **Generates scripts** in multiple formats (toplist, POV, case study, how-to) using Claude/OpenAI
- **Creates bilingual content** (English & Vietnamese) with customizable tone
- **Renders videos** automatically using Remotion integration
- **Optimizes for platforms** (Reels, TikTok, Shorts) with proper aspect ratios

## Installation

### Prerequisites

```bash
# Node.js 18+ required
node --version

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

### Environment Configuration

Create a `.env.local` file in the project root:

```bash
# AI APIs
ANTHROPIC_API_KEY=your_anthropic_key_here
OPENAI_API_KEY=your_openai_key_here

# RapidAPI for content crawling
RAPIDAPI_KEY=your_rapidapi_key_here

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Remotion License (if using cloud rendering)
REMOTION_LICENSE_KEY=your_remotion_license_key

# Next.js Configuration
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

### Start Development Server

```bash
npm run dev
# or
yarn dev
# or
pnpm dev
```

Access the application at `http://localhost:3000`

## Key API Patterns

### Content Research Module

```typescript
// src/lib/research/crawler.ts
import { RapidAPI } from '@/lib/api/rapidapi';

interface ResearchResult {
  title: string;
  source: string;
  url: string;
  publishedAt: Date;
  summary: string;
  insights: string[];
}

export async function crawlLatestNews(
  keyword: string,
  sources: string[] = ['techcrunch', 'a16z', 'twitter']
): Promise<ResearchResult[]> {
  const rapidApi = new RapidAPI(process.env.RAPIDAPI_KEY!);
  
  const results: ResearchResult[] = [];
  
  for (const source of sources) {
    const data = await rapidApi.search({
      query: keyword,
      source: source,
      timeRange: '24h'
    });
    
    results.push(...data.articles);
  }
  
  return results;
}
```

### AI Content Generation

```typescript
// src/lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

type ContentFormat = 'toplist' | 'pov' | 'case-study' | 'how-to';
type Language = 'en' | 'vi';
type Tone = 'expert' | 'friendly' | 'humorous';

interface GenerateContentOptions {
  keyword: string;
  format: ContentFormat;
  language: Language;
  tone: Tone;
  researchData: ResearchResult[];
}

export async function generateContent(
  options: GenerateContentOptions,
  provider: 'claude' | 'openai' = 'claude'
): Promise<string> {
  const prompt = buildPrompt(options);
  
  if (provider === 'claude') {
    const anthropic = new Anthropic({
      apiKey: process.env.ANTHROPIC_API_KEY
    });
    
    const message = await anthropic.messages.create({
      model: 'claude-3-opus-20240229',
      max_tokens: 4096,
      messages: [{
        role: 'user',
        content: prompt
      }]
    });
    
    return message.content[0].text;
  } else {
    const openai = new OpenAI({
      apiKey: process.env.OPENAI_API_KEY
    });
    
    const completion = await openai.chat.completions.create({
      model: 'gpt-4-turbo-preview',
      messages: [{
        role: 'user',
        content: prompt
      }],
      max_tokens: 4096
    });
    
    return completion.choices[0].message.content || '';
  }
}

function buildPrompt(options: GenerateContentOptions): string {
  const { keyword, format, language, tone, researchData } = options;
  
  const researchContext = researchData.map(r => 
    `- ${r.title} (${r.source}): ${r.summary}`
  ).join('\n');
  
  return `
You are an expert content writer. Create a ${format} article about "${keyword}" in ${language}.

Tone: ${tone}
Format: ${format}

Recent Research Data (last 24h):
${researchContext}

Requirements:
- Use data-backed insights from the research
- Write in ${language === 'vi' ? 'Vietnamese' : 'English'}
- Follow ${format} structure
- Maintain ${tone} tone throughout
- Include specific examples and statistics
- Make it engaging and actionable

Generate the complete article now:
  `.trim();
}
```

### Bilingual Content Generation

```typescript
// src/lib/ai/bilingual.ts
export async function generateBilingualContent(
  keyword: string,
  format: ContentFormat,
  researchData: ResearchResult[]
): Promise<{ en: string; vi: string }> {
  const [enContent, viContent] = await Promise.all([
    generateContent({
      keyword,
      format,
      language: 'en',
      tone: 'expert',
      researchData
    }),
    generateContent({
      keyword,
      format,
      language: 'vi',
      tone: 'expert',
      researchData
    })
  ]);
  
  return { en: enContent, vi: viContent };
}
```

## Video Generation with Remotion

### Basic Video Composition

```typescript
// src/remotion/compositions/ContentVideo.tsx
import { AbsoluteFill, Sequence, useCurrentFrame } from 'remotion';

interface ContentVideoProps {
  title: string;
  points: string[];
  duration: number;
}

export const ContentVideo: React.FC<ContentVideoProps> = ({
  title,
  points,
  duration
}) => {
  const frame = useCurrentFrame();
  
  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a1a' }}>
      <Sequence from={0} durationInFrames={60}>
        <div style={{
          fontSize: 60,
          color: 'white',
          textAlign: 'center',
          padding: 40
        }}>
          {title}
        </div>
      </Sequence>
      
      {points.map((point, index) => (
        <Sequence
          key={index}
          from={60 + (index * 90)}
          durationInFrames={90}
        >
          <div style={{
            fontSize: 40,
            color: 'white',
            padding: 40,
            opacity: frame > 60 + (index * 90) + 15 ? 1 : 0,
            transition: 'opacity 0.3s'
          }}>
            {point}
          </div>
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};
```

### Remotion Configuration

```typescript
// remotion.config.ts
import { Config } from '@remotion/cli/config';

Config.setVideoImageFormat('jpeg');
Config.setOverwriteOutput(true);
Config.setPixelFormat('yuv420p');
Config.setConcurrency(2);

// For different platform formats
export const platformConfigs = {
  reels: {
    width: 1080,
    height: 1920,
    fps: 30
  },
  tiktok: {
    width: 1080,
    height: 1920,
    fps: 30
  },
  youtube: {
    width: 1920,
    height: 1080,
    fps: 30
  }
};
```

### Rendering Videos

```typescript
// src/lib/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface RenderOptions {
  content: string;
  platform: 'reels' | 'tiktok' | 'youtube';
  outputPath: string;
}

export async function renderContentVideo(
  options: RenderOptions
): Promise<string> {
  const { content, platform, outputPath } = options;
  
  // Parse content into video data
  const videoData = parseContentForVideo(content);
  
  // Bundle the Remotion project
  const bundleLocation = await bundle(
    path.join(process.cwd(), 'src/remotion/index.ts')
  );
  
  // Get composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: videoData
  });
  
  // Render video
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: videoData,
    ...platformConfigs[platform]
  });
  
  return outputPath;
}

function parseContentForVideo(content: string) {
  // Extract title and key points from content
  const lines = content.split('\n');
  const title = lines[0].replace(/^#\s+/, '');
  const points = lines
    .filter(line => line.startsWith('- '))
    .map(line => line.replace(/^-\s+/, ''));
  
  return {
    title,
    points: points.slice(0, 5), // Top 5 points
    duration: 60 + (points.slice(0, 5).length * 90)
  };
}
```

## Complete Pipeline Integration

### End-to-End Content Pipeline

```typescript
// src/lib/pipeline/content-pipeline.ts
import { crawlLatestNews } from '@/lib/research/crawler';
import { generateBilingualContent } from '@/lib/ai/bilingual';
import { renderContentVideo } from '@/lib/video/renderer';

interface PipelineOptions {
  keyword: string;
  format: ContentFormat;
  generateVideo: boolean;
  platform?: 'reels' | 'tiktok' | 'youtube';
}

interface PipelineResult {
  content: {
    en: string;
    vi: string;
  };
  research: ResearchResult[];
  videoPath?: string;
}

export async function runContentPipeline(
  options: PipelineOptions
): Promise<PipelineResult> {
  const { keyword, format, generateVideo, platform = 'reels' } = options;
  
  // Step 1: Research
  console.log('🔍 Starting research...');
  const research = await crawlLatestNews(keyword);
  
  // Step 2: Generate Content
  console.log('✍️ Generating content...');
  const content = await generateBilingualContent(
    keyword,
    format,
    research
  );
  
  // Step 3: Render Video (optional)
  let videoPath: string | undefined;
  if (generateVideo) {
    console.log('🎬 Rendering video...');
    videoPath = await renderContentVideo({
      content: content.en,
      platform,
      outputPath: `./output/${keyword}-${Date.now()}.mp4`
    });
  }
  
  console.log('✅ Pipeline complete!');
  
  return {
    content,
    research,
    videoPath
  };
}
```

### API Route Example (Next.js)

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline/content-pipeline';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const { keyword, format, generateVideo, platform } = body;
    
    // Validate input
    if (!keyword || !format) {
      return NextResponse.json(
        { error: 'Keyword and format are required' },
        { status: 400 }
      );
    }
    
    // Run pipeline
    const result = await runContentPipeline({
      keyword,
      format,
      generateVideo: generateVideo || false,
      platform: platform || 'reels'
    });
    
    return NextResponse.json({
      success: true,
      data: result
    });
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { error: 'Failed to generate content' },
      { status: 500 }
    );
  }
}
```

### Frontend Usage Example

```typescript
// src/app/components/ContentGenerator.tsx
'use client';

import { useState } from 'react';

export function ContentGenerator() {
  const [keyword, setKeyword] = useState('');
  const [format, setFormat] = useState<ContentFormat>('toplist');
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
          format,
          generateVideo: true,
          platform: 'reels'
        })
      });
      
      const data = await response.json();
      setResult(data.data);
    } catch (error) {
      console.error('Error:', error);
    } finally {
      setLoading(false);
    }
  };
  
  return (
    <div className="p-6">
      <input
        type="text"
        value={keyword}
        onChange={(e) => setKeyword(e.target.value)}
        placeholder="Enter keyword..."
        className="border p-2 rounded w-full mb-4"
      />
      
      <select
        value={format}
        onChange={(e) => setFormat(e.target.value as ContentFormat)}
        className="border p-2 rounded w-full mb-4"
      >
        <option value="toplist">Top List</option>
        <option value="pov">POV</option>
        <option value="case-study">Case Study</option>
        <option value="how-to">How-to</option>
      </select>
      
      <button
        onClick={handleGenerate}
        disabled={loading}
        className="bg-blue-500 text-white px-6 py-2 rounded"
      >
        {loading ? 'Generating...' : 'Generate Content'}
      </button>
      
      {result && (
        <div className="mt-6">
          <h3 className="font-bold mb-2">English Content:</h3>
          <pre className="bg-gray-100 p-4 rounded">
            {result.content.en}
          </pre>
          
          <h3 className="font-bold mb-2 mt-4">Vietnamese Content:</h3>
          <pre className="bg-gray-100 p-4 rounded">
            {result.content.vi}
          </pre>
          
          {result.videoPath && (
            <div className="mt-4">
              <h3 className="font-bold mb-2">Video:</h3>
              <p>Generated at: {result.videoPath}</p>
            </div>
          )}
        </div>
      )}
    </div>
  );
}
```

## CLI Commands

### Development

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Start production server
npm run start

# Run type checking
npm run type-check

# Lint code
npm run lint
```

### Remotion Commands

```bash
# Preview Remotion compositions
npm run remotion:preview

# Render a specific composition
npm run remotion:render ContentVideo output.mp4

# Upgrade Remotion
npm run remotion:upgrade
```

## Common Patterns

### Scheduled Content Generation

```typescript
// src/lib/scheduler/cron.ts
import cron from 'node-cron';
import { runContentPipeline } from '@/lib/pipeline/content-pipeline';

// Run every day at 9 AM
export function scheduleContentGeneration(keywords: string[]) {
  cron.schedule('0 9 * * *', async () => {
    console.log('🕐 Running scheduled content generation...');
    
    for (const keyword of keywords) {
      try {
        await runContentPipeline({
          keyword,
          format: 'toplist',
          generateVideo: true
        });
      } catch (error) {
        console.error(`Failed for keyword: ${keyword}`, error);
      }
    }
  });
}
```

### Batch Processing

```typescript
// src/lib/batch/processor.ts
export async function batchGenerateContent(
  keywords: string[],
  concurrency: number = 3
) {
  const chunks = chunkArray(keywords, concurrency);
  
  for (const chunk of chunks) {
    await Promise.all(
      chunk.map(keyword => 
        runContentPipeline({
          keyword,
          format: 'toplist',
          generateVideo: false
        })
      )
    );
  }
}

function chunkArray<T>(array: T[], size: number): T[][] {
  const chunks: T[][] = [];
  for (let i = 0; i < array.length; i += size) {
    chunks.push(array.slice(i, i + size));
  }
  return chunks;
}
```

## Troubleshooting

### API Rate Limits

```typescript
// src/lib/utils/rate-limiter.ts
export class RateLimiter {
  private queue: Array<() => Promise<any>> = [];
  private processing = false;
  
  constructor(private delayMs: number = 1000) {}
  
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
    const task = this.queue.shift()!;
    
    await task();
    await new Promise(resolve => setTimeout(resolve, this.delayMs));
    
    this.processing = false;
    this.process();
  }
}

// Usage
const limiter = new RateLimiter(2000); // 2 second delay
const result = await limiter.add(() => generateContent(options));
```

### Error Handling

```typescript
// src/lib/utils/error-handler.ts
export class PipelineError extends Error {
  constructor(
    message: string,
    public stage: 'research' | 'generation' | 'rendering',
    public originalError?: Error
  ) {
    super(message);
    this.name = 'PipelineError';
  }
}

export async function safeRunPipeline(options: PipelineOptions) {
  try {
    return await runContentPipeline(options);
  } catch (error) {
    if (error instanceof PipelineError) {
      console.error(`Error in ${error.stage}:`, error.message);
      // Handle specific stage errors
    }
    throw error;
  }
}
```

### Memory Management for Large Videos

```typescript
// Render in chunks for large videos
import { renderFrames } from '@remotion/renderer';

export async function renderLargeVideo(options: RenderOptions) {
  const frameRange = [0, 300]; // First 300 frames
  
  await renderFrames({
    ...options,
    frameRange,
    onFrameUpdate: (frame) => {
      console.log(`Rendered frame ${frame}`);
    }
  });
}
```

This skill provides comprehensive coverage of the marketing-pipeline-share project, enabling AI coding agents to effectively assist developers in implementing automated content generation workflows.
