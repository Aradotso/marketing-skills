---
name: marketing-pipeline-automation
description: Automated AI content pipeline for research, scriptwriting, and video generation using Claude, OpenAI, and Remotion
triggers:
  - "set up an automated content pipeline"
  - "create AI-powered content generation workflow"
  - "automate content research and video creation"
  - "build a marketing automation system with AI"
  - "generate content from research to video automatically"
  - "create multi-format content with Claude and OpenAI"
  - "set up automated social media content pipeline"
  - "build content workflow with Remotion video rendering"
---

# Marketing Pipeline Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI agents to help developers use **marketing-pipeline-share**, an end-to-end AI content automation system that handles research, scriptwriting, multi-format content generation, and video rendering. The pipeline crawls real-time data from sources like TechCrunch, a16z, Twitter, and LinkedIn, then uses Claude 3/OpenAI to generate content in multiple formats (toplist, POV, case study, how-to) and languages (English/Vietnamese), before rendering videos using Remotion.

## Project Overview

The marketing-pipeline-share system automates 90% of content creation workflows:

1. **Auto-Scan Research**: Crawls fresh news and insights from major tech sources (24h data)
2. **AI Content Generation**: Uses Claude/OpenAI to create diverse content formats with customizable tone
3. **Video Rendering**: Automatically generates infographics and short-form videos via Remotion
4. **Multi-Platform Export**: Outputs optimized content for Reels, TikTok, Shorts

**Tech Stack**: TypeScript, Next.js, OpenAI API, Anthropic Claude API, Remotion, RapidAPI

## Installation

### Prerequisites

```bash
node >= 18.0.0
npm >= 9.0.0
```

### Setup

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install

# Configure environment variables
cp .env.example .env.local
```

### Required Environment Variables

```bash
# AI API Keys
OPENAI_API_KEY=your_openai_key_here
ANTHROPIC_API_KEY=your_anthropic_key_here

# Research & Scraping
RAPIDAPI_KEY=your_rapidapi_key_here

# Content Configuration
DEFAULT_LANGUAGE=en
SECONDARY_LANGUAGE=vi

# Video Rendering (Remotion)
REMOTION_OUTPUT_DIR=./public/videos
REMOTION_COMPOSITION=ContentVideo
```

## Core Architecture

### Directory Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app router
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── research/    # Content scraping & research
│   │   ├── content/     # Content generation logic
│   │   └── video/       # Remotion video rendering
│   ├── types/           # TypeScript definitions
│   └── utils/           # Helper functions
├── remotion/            # Video templates
└── public/              # Static assets
```

## Key APIs and Usage

### 1. Research Module (Content Scraping)

```typescript
// src/lib/research/scanner.ts
import { scrapeNews } from './sources';

interface ResearchResult {
  title: string;
  source: string;
  url: string;
  summary: string;
  publishedAt: Date;
  keywords: string[];
}

async function scanLatestNews(keyword: string): Promise<ResearchResult[]> {
  const sources = ['techcrunch', 'a16z', 'twitter', 'linkedin'];
  
  const results = await Promise.all(
    sources.map(source => scrapeNews({
      source,
      keyword,
      timeRange: '24h',
      rapidApiKey: process.env.RAPIDAPI_KEY
    }))
  );
  
  return results.flat().sort((a, b) => 
    b.publishedAt.getTime() - a.publishedAt.getTime()
  );
}

export { scanLatestNews };
```

### 2. AI Content Generation

#### Using Claude (Anthropic)

```typescript
// src/lib/ai/claude-generator.ts
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

interface ContentRequest {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  tone: 'expert' | 'friendly' | 'humorous';
  language: 'en' | 'vi';
  researchData: ResearchResult[];
}

async function generateContentClaude(req: ContentRequest): Promise<string> {
  const prompt = buildPrompt(req);
  
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4000,
    messages: [{
      role: 'user',
      content: prompt
    }]
  });
  
  return message.content[0].type === 'text' 
    ? message.content[0].text 
    : '';
}

function buildPrompt(req: ContentRequest): string {
  const formatInstructions = {
    'toplist': 'Create a numbered list article with clear rankings',
    'pov': 'Write a personal perspective piece with strong opinions',
    'case-study': 'Analyze with data, examples, and actionable insights',
    'how-to': 'Provide step-by-step instructions with practical tips'
  };
  
  const researchContext = req.researchData
    .map(r => `- ${r.title} (${r.source}): ${r.summary}`)
    .join('\n');
  
  return `
You are a ${req.tone} content writer. Create a ${req.format} article in ${req.language}.

Topic: ${req.keyword}

Recent Research Data:
${researchContext}

Instructions: ${formatInstructions[req.format]}

Provide a complete article with:
- Engaging headline
- Introduction hook
- Main content with subheadings
- Data-backed insights
- Clear conclusion with CTA
`;
}

export { generateContentClaude };
```

#### Using OpenAI

```typescript
// src/lib/ai/openai-generator.ts
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateContentOpenAI(req: ContentRequest): Promise<string> {
  const prompt = buildPrompt(req);
  
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are a professional ${req.tone} content writer specializing in ${req.format} content.`
      },
      {
        role: 'user',
        content: prompt
      }
    ],
    temperature: 0.7,
    max_tokens: 3000
  });
  
  return completion.choices[0]?.message?.content || '';
}

export { generateContentOpenAI };
```

### 3. Dual-Language Content Generation

```typescript
// src/lib/content/multilang-pipeline.ts
interface BilingualContent {
  en: string;
  vi: string;
  metadata: {
    keyword: string;
    format: string;
    generatedAt: Date;
  };
}

async function generateBilingualContent(
  keyword: string,
  format: ContentRequest['format']
): Promise<BilingualContent> {
  const research = await scanLatestNews(keyword);
  
  const [enContent, viContent] = await Promise.all([
    generateContentClaude({
      keyword,
      format,
      tone: 'expert',
      language: 'en',
      researchData: research
    }),
    generateContentClaude({
      keyword,
      format,
      tone: 'friendly',
      language: 'vi',
      researchData: research
    })
  ]);
  
  return {
    en: enContent,
    vi: viContent,
    metadata: {
      keyword,
      format,
      generatedAt: new Date()
    }
  };
}

export { generateBilingualContent };
```

### 4. Video Rendering with Remotion

```typescript
// src/lib/video/render.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  content: string;
  title: string;
  platform: 'reels' | 'tiktok' | 'shorts';
}

const platformSpecs = {
  reels: { width: 1080, height: 1920, fps: 30 },
  tiktok: { width: 1080, height: 1920, fps: 30 },
  shorts: { width: 1080, height: 1920, fps: 30 }
};

async function renderContentVideo(config: VideoConfig): Promise<string> {
  const specs = platformSpecs[config.platform];
  const compositionId = process.env.REMOTION_COMPOSITION || 'ContentVideo';
  
  // Bundle Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.join(process.cwd(), 'remotion/index.ts'),
    webpackOverride: (config) => config,
  });
  
  // Get composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: {
      title: config.title,
      content: parseContentToScenes(config.content),
      ...specs
    },
  });
  
  // Render video
  const outputLocation = path.join(
    process.env.REMOTION_OUTPUT_DIR || './public/videos',
    `${Date.now()}-${config.platform}.mp4`
  );
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: composition.props,
  });
  
  return outputLocation;
}

function parseContentToScenes(content: string): Array<{text: string; duration: number}> {
  // Split content into video scenes (every 2-3 sentences)
  const sentences = content.match(/[^.!?]+[.!?]+/g) || [];
  const scenes = [];
  
  for (let i = 0; i < sentences.length; i += 2) {
    scenes.push({
      text: sentences.slice(i, i + 2).join(' ').trim(),
      duration: 90 // 3 seconds per scene
    });
  }
  
  return scenes;
}

export { renderContentVideo };
```

### 5. Complete Pipeline Workflow

```typescript
// src/lib/pipeline/orchestrator.ts
import { scanLatestNews } from '../research/scanner';
import { generateContentClaude } from '../ai/claude-generator';
import { renderContentVideo } from '../video/render';

interface PipelineConfig {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  platforms: Array<'reels' | 'tiktok' | 'shorts'>;
  languages: Array<'en' | 'vi'>;
}

interface PipelineResult {
  content: Record<string, string>;
  videos: Record<string, string>;
  research: ResearchResult[];
}

async function runContentPipeline(config: PipelineConfig): Promise<PipelineResult> {
  // Step 1: Research
  console.log(`🔍 Scanning news for: ${config.keyword}`);
  const research = await scanLatestNews(config.keyword);
  
  // Step 2: Generate content in all languages
  console.log(`✍️ Generating ${config.format} content`);
  const contentPromises = config.languages.map(lang =>
    generateContentClaude({
      keyword: config.keyword,
      format: config.format,
      tone: 'expert',
      language: lang,
      researchData: research
    }).then(content => [lang, content] as const)
  );
  
  const contentResults = await Promise.all(contentPromises);
  const content = Object.fromEntries(contentResults);
  
  // Step 3: Render videos for all platforms (using primary language)
  console.log(`🎬 Rendering videos for platforms: ${config.platforms.join(', ')}`);
  const primaryContent = content[config.languages[0]];
  const videoPromises = config.platforms.map(platform =>
    renderContentVideo({
      content: primaryContent,
      title: config.keyword,
      platform
    }).then(path => [platform, path] as const)
  );
  
  const videoResults = await Promise.all(videoPromises);
  const videos = Object.fromEntries(videoResults);
  
  return { content, videos, research };
}

export { runContentPipeline };
```

## Next.js API Routes

```typescript
// src/app/api/pipeline/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline/orchestrator';

export async function POST(req: NextRequest) {
  try {
    const { keyword, format, platforms, languages } = await req.json();
    
    if (!keyword || !format) {
      return NextResponse.json(
        { error: 'Missing required fields: keyword, format' },
        { status: 400 }
      );
    }
    
    const result = await runContentPipeline({
      keyword,
      format,
      platforms: platforms || ['reels', 'tiktok'],
      languages: languages || ['en', 'vi']
    });
    
    return NextResponse.json({
      success: true,
      data: result
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

## Frontend Usage Example

```typescript
// src/components/PipelineRunner.tsx
'use client';

import { useState } from 'react';

export default function PipelineRunner() {
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState(null);
  
  async function handleSubmit(e: React.FormEvent<HTMLFormElement>) {
    e.preventDefault();
    setLoading(true);
    
    const formData = new FormData(e.currentTarget);
    const response = await fetch('/api/pipeline', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        keyword: formData.get('keyword'),
        format: formData.get('format'),
        platforms: ['reels', 'tiktok', 'shorts'],
        languages: ['en', 'vi']
      })
    });
    
    const data = await response.json();
    setResult(data);
    setLoading(false);
  }
  
  return (
    <div className="max-w-2xl mx-auto p-6">
      <h1 className="text-3xl font-bold mb-6">AI Content Pipeline</h1>
      
      <form onSubmit={handleSubmit} className="space-y-4">
        <div>
          <label className="block mb-2">Keyword</label>
          <input 
            name="keyword" 
            type="text" 
            required 
            className="w-full p-2 border rounded"
            placeholder="e.g., AI Marketing Tools 2024"
          />
        </div>
        
        <div>
          <label className="block mb-2">Format</label>
          <select name="format" className="w-full p-2 border rounded" required>
            <option value="toplist">Top List</option>
            <option value="pov">POV / Opinion</option>
            <option value="case-study">Case Study</option>
            <option value="how-to">How-To Guide</option>
          </select>
        </div>
        
        <button 
          type="submit" 
          disabled={loading}
          className="w-full bg-blue-600 text-white p-3 rounded hover:bg-blue-700 disabled:opacity-50"
        >
          {loading ? 'Processing...' : 'Run Pipeline'}
        </button>
      </form>
      
      {result && (
        <div className="mt-8 p-6 bg-gray-50 rounded">
          <h2 className="text-xl font-bold mb-4">Results</h2>
          <pre className="text-sm overflow-auto">
            {JSON.stringify(result, null, 2)}
          </pre>
        </div>
      )}
    </div>
  );
}
```

## Common Patterns

### Pattern 1: Scheduled Content Generation

```typescript
// src/lib/scheduler/cron.ts
import cron from 'node-cron';
import { runContentPipeline } from '../pipeline/orchestrator';

const contentTopics = [
  { keyword: 'AI Marketing Trends', format: 'toplist' as const },
  { keyword: 'Social Media Strategy 2024', format: 'how-to' as const },
  { keyword: 'Content Automation Tools', format: 'case-study' as const }
];

// Run daily at 6 AM
cron.schedule('0 6 * * *', async () => {
  console.log('Starting scheduled content generation...');
  
  for (const topic of contentTopics) {
    try {
      await runContentPipeline({
        keyword: topic.keyword,
        format: topic.format,
        platforms: ['reels', 'tiktok'],
        languages: ['en', 'vi']
      });
      console.log(`✅ Generated content for: ${topic.keyword}`);
    } catch (error) {
      console.error(`❌ Failed for ${topic.keyword}:`, error);
    }
  }
});
```

### Pattern 2: Content Variation Testing

```typescript
// Generate multiple versions with different tones
async function generateVariations(keyword: string) {
  const tones = ['expert', 'friendly', 'humorous'] as const;
  const research = await scanLatestNews(keyword);
  
  const variations = await Promise.all(
    tones.map(tone =>
      generateContentClaude({
        keyword,
        format: 'toplist',
        tone,
        language: 'en',
        researchData: research
      }).then(content => ({ tone, content }))
    )
  );
  
  return variations;
}
```

## Troubleshooting

### API Rate Limits

```typescript
// src/lib/utils/rate-limiter.ts
class RateLimiter {
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
    const task = this.queue.shift();
    
    if (task) {
      await task();
      await new Promise(resolve => setTimeout(resolve, 1000)); // 1 req/sec
    }
    
    this.processing = false;
    this.process();
  }
}

export const aiLimiter = new RateLimiter();
```

### Error Handling

```typescript
// Wrap AI calls with retry logic
async function generateWithRetry(
  req: ContentRequest, 
  maxRetries = 3
): Promise<string> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await generateContentClaude(req);
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      await new Promise(resolve => setTimeout(resolve, 2000 * (i + 1)));
    }
  }
  throw new Error('Max retries exceeded');
}
```

### Environment Validation

```typescript
// src/lib/config/validate.ts
const requiredEnvVars = [
  'OPENAI_API_KEY',
  'ANTHROPIC_API_KEY',
  'RAPIDAPI_KEY'
];

export function validateEnv() {
  const missing = requiredEnvVars.filter(key => !process.env[key]);
  
  if (missing.length > 0) {
    throw new Error(
      `Missing required environment variables: ${missing.join(', ')}`
    );
  }
}
```

## Running the Project

```bash
# Development
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Type checking
npm run type-check

# Run specific pipeline
npm run pipeline -- --keyword "AI Tools" --format toplist
```

This skill enables comprehensive automation of marketing content workflows from research through video production using modern AI and rendering technologies.
