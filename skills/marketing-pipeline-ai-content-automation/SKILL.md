---
name: marketing-pipeline-ai-content-automation
description: AI-powered content automation pipeline that researches trends, generates multi-format content in multiple languages, and renders videos automatically
triggers:
  - automate content creation with AI research
  - generate blog posts and videos from keywords
  - set up AI content pipeline with Claude and OpenAI
  - create automated marketing content workflow
  - build AI-powered content generation system
  - research and write content automatically with AI
  - generate multilingual content with video rendering
  - automate social media content production
---

# Marketing Pipeline AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This TypeScript-based system automates the entire content creation pipeline: from researching trending topics across TechCrunch, a16z, Twitter, and LinkedIn, to generating multi-format content (blog posts, case studies, how-tos) in multiple languages, to automatically rendering videos and infographics using Remotion.

## What It Does

- **Auto-Research**: Crawls real-time data from major tech news sources and social platforms
- **AI Content Generation**: Uses Claude 3 and OpenAI to write content in various formats (toplist, POV, case study, how-to)
- **Multi-language Output**: Generates content simultaneously in English and Vietnamese
- **Video Rendering**: Automatically creates videos and infographics from written content using Remotion
- **Platform Optimization**: Exports videos in formats suitable for Reels, TikTok, Shorts

## Installation

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
pnpm install

# Set up environment variables
cp .env.example .env
```

## Configuration

Create a `.env` file with the following variables:

```bash
# AI Provider Keys
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_url

# Next.js Configuration
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app router pages
│   ├── components/       # React components
│   ├── lib/             
│   │   ├── research/    # Auto-scan research logic
│   │   ├── ai/          # AI content generation
│   │   └── video/       # Remotion video rendering
│   ├── types/           # TypeScript type definitions
│   └── utils/           # Utility functions
├── remotion/            # Video templates
└── public/              # Static assets
```

## Core Workflows

### 1. Research Automation

```typescript
// lib/research/crawler.ts
import { fetchTechCrunchArticles, fetchTwitterTrends } from './sources';

interface ResearchResult {
  title: string;
  url: string;
  summary: string;
  publishedAt: Date;
  source: string;
}

export async function performResearch(
  keyword: string,
  timeframe: '24h' | '7d' = '24h'
): Promise<ResearchResult[]> {
  const sources = await Promise.all([
    fetchTechCrunchArticles(keyword, timeframe),
    fetchTwitterTrends(keyword),
    fetchLinkedInPosts(keyword),
    fetchA16zContent(keyword)
  ]);

  return sources.flat().sort((a, b) => 
    b.publishedAt.getTime() - a.publishedAt.getTime()
  );
}
```

### 2. AI Content Generation

```typescript
// lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

export type ContentFormat = 'toplist' | 'pov' | 'case-study' | 'how-to';
export type ContentTone = 'expert' | 'friendly' | 'humorous';

interface GenerateContentParams {
  keyword: string;
  research: ResearchResult[];
  format: ContentFormat;
  tone: ContentTone;
  language: 'en' | 'vi';
  provider?: 'claude' | 'openai';
}

export async function generateContent({
  keyword,
  research,
  format,
  tone,
  language,
  provider = 'claude'
}: GenerateContentParams): Promise<string> {
  const researchContext = research
    .map(r => `- ${r.title}: ${r.summary} (${r.source})`)
    .join('\n');

  const prompt = buildPrompt(keyword, researchContext, format, tone, language);

  if (provider === 'claude') {
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
  } else {
    const completion = await openai.chat.completions.create({
      model: 'gpt-4-turbo-preview',
      messages: [{
        role: 'user',
        content: prompt
      }]
    });

    return completion.choices[0]?.message?.content || '';
  }
}

function buildPrompt(
  keyword: string,
  research: string,
  format: ContentFormat,
  tone: ContentTone,
  language: 'en' | 'vi'
): string {
  const formatInstructions = {
    'toplist': 'Create a numbered list article highlighting the top items/trends',
    'pov': 'Write from a specific perspective or opinion angle',
    'case-study': 'Analyze a specific example with problem-solution-results structure',
    'how-to': 'Provide step-by-step instructions'
  };

  const toneInstructions = {
    'expert': 'Use professional, authoritative language with industry terminology',
    'friendly': 'Use conversational, approachable language',
    'humorous': 'Include wit and light humor while staying informative'
  };

  return `
You are a ${tone} content writer creating a ${format} article about "${keyword}".

${formatInstructions[format]}
Tone: ${toneInstructions[tone]}
Language: ${language === 'en' ? 'English' : 'Vietnamese'}

Recent research context:
${research}

Requirements:
- Use data and insights from the research above
- Include specific examples and statistics
- Make it engaging and actionable
- ${language === 'vi' ? 'Write naturally in Vietnamese' : 'Write in clear English'}
- Length: 1000-1500 words

Generate the complete article now:
`;
}
```

### 3. Dual-Language Generation

```typescript
// lib/ai/multilingual.ts
export async function generateMultilingualContent(
  params: Omit<GenerateContentParams, 'language'>
): Promise<{ en: string; vi: string }> {
  const [englishContent, vietnameseContent] = await Promise.all([
    generateContent({ ...params, language: 'en' }),
    generateContent({ ...params, language: 'vi' })
  ]);

  return {
    en: englishContent,
    vi: vietnameseContent
  };
}
```

### 4. Video Rendering with Remotion

```typescript
// lib/video/render.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  title: string;
  keyPoints: string[];
  backgroundImage?: string;
  duration: number; // in seconds
}

export async function renderContentVideo(
  content: string,
  config: VideoConfig
): Promise<string> {
  const compositionId = 'ContentVideo';
  
  // Extract key points from content
  const keyPoints = extractKeyPoints(content, 5);

  const bundled = await bundle({
    entryPoint: path.join(process.cwd(), 'remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  const composition = await selectComposition({
    serveUrl: bundled,
    id: compositionId,
    inputProps: {
      title: config.title,
      keyPoints: keyPoints,
      backgroundImage: config.backgroundImage,
    },
  });

  const outputPath = path.join(
    process.cwd(), 
    'output', 
    `${Date.now()}-video.mp4`
  );

  await renderMedia({
    composition,
    serveUrl: bundled,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: {
      title: config.title,
      keyPoints: keyPoints,
    },
  });

  return outputPath;
}

function extractKeyPoints(content: string, count: number): string[] {
  // Simple extraction - enhance with AI summarization
  const sentences = content.split(/[.!?]+/).filter(s => s.trim().length > 20);
  return sentences.slice(0, count).map(s => s.trim());
}
```

### 5. Complete Pipeline Execution

```typescript
// lib/pipeline/executor.ts
export interface PipelineConfig {
  keyword: string;
  formats: ContentFormat[];
  tone: ContentTone;
  generateVideo: boolean;
  aiProvider: 'claude' | 'openai';
}

export interface PipelineResult {
  research: ResearchResult[];
  content: {
    en: string;
    vi: string;
    format: ContentFormat;
  }[];
  videos?: string[];
}

export async function executePipeline(
  config: PipelineConfig
): Promise<PipelineResult> {
  // Step 1: Research
  console.log('🔍 Starting research phase...');
  const research = await performResearch(config.keyword);
  
  // Step 2: Generate content for each format
  console.log('✍️ Generating content...');
  const contentPromises = config.formats.map(async (format) => {
    const content = await generateMultilingualContent({
      keyword: config.keyword,
      research,
      format,
      tone: config.tone,
      provider: config.aiProvider,
    });

    return {
      ...content,
      format,
    };
  });

  const content = await Promise.all(contentPromises);

  // Step 3: Render videos if requested
  let videos: string[] | undefined;
  if (config.generateVideo) {
    console.log('🎬 Rendering videos...');
    videos = await Promise.all(
      content.map((c) =>
        renderContentVideo(c.en, {
          title: config.keyword,
          keyPoints: extractKeyPoints(c.en, 5),
          duration: 30,
        })
      )
    );
  }

  console.log('✅ Pipeline complete!');
  return { research, content, videos };
}
```

## Usage Examples

### Basic CLI Usage

```typescript
// scripts/generate.ts
import { executePipeline } from '../lib/pipeline/executor';

async function main() {
  const result = await executePipeline({
    keyword: 'AI in Marketing 2024',
    formats: ['toplist', 'how-to'],
    tone: 'expert',
    generateVideo: true,
    aiProvider: 'claude'
  });

  console.log(`Generated ${result.content.length} articles`);
  console.log(`Researched ${result.research.length} sources`);
  if (result.videos) {
    console.log(`Rendered ${result.videos.length} videos`);
  }
}

main();
```

### Next.js API Route

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { executePipeline } from '@/lib/pipeline/executor';

export async function POST(req: NextRequest) {
  try {
    const body = await req.json();
    const { keyword, formats, tone, generateVideo, aiProvider } = body;

    const result = await executePipeline({
      keyword,
      formats: formats || ['toplist'],
      tone: tone || 'friendly',
      generateVideo: generateVideo || false,
      aiProvider: aiProvider || 'claude'
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

### React Component Integration

```typescript
// components/ContentGenerator.tsx
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
          formats: ['toplist', 'how-to'],
          tone: 'friendly',
          generateVideo: true,
          aiProvider: 'claude'
        })
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
    <div className="p-6">
      <input
        type="text"
        value={keyword}
        onChange={(e) => setKeyword(e.target.value)}
        placeholder="Enter keyword..."
        className="border p-2 rounded w-full"
      />
      <button
        onClick={handleGenerate}
        disabled={loading}
        className="mt-4 bg-blue-500 text-white px-4 py-2 rounded"
      >
        {loading ? 'Generating...' : 'Generate Content'}
      </button>

      {result && (
        <div className="mt-6">
          <h3>Research Sources: {result.research.length}</h3>
          <h3>Content Generated: {result.content.length}</h3>
          {result.videos && <h3>Videos: {result.videos.length}</h3>}
        </div>
      )}
    </div>
  );
}
```

## Common Patterns

### Error Handling

```typescript
// lib/utils/error-handler.ts
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

export async function withErrorHandling<T>(
  fn: () => Promise<T>,
  stage: PipelineError['stage']
): Promise<T> {
  try {
    return await fn();
  } catch (error) {
    throw new PipelineError(
      `Failed at ${stage} stage`,
      stage,
      error as Error
    );
  }
}
```

### Rate Limiting

```typescript
// lib/utils/rate-limiter.ts
export class RateLimiter {
  private queue: (() => Promise<any>)[] = [];
  private running = 0;

  constructor(private maxConcurrent: number = 3) {}

  async add<T>(fn: () => Promise<T>): Promise<T> {
    while (this.running >= this.maxConcurrent) {
      await new Promise(resolve => setTimeout(resolve, 100));
    }

    this.running++;
    try {
      return await fn();
    } finally {
      this.running--;
    }
  }
}

// Usage with AI providers
const limiter = new RateLimiter(3);
const results = await Promise.all(
  tasks.map(task => limiter.add(() => generateContent(task)))
);
```

## Troubleshooting

### API Key Issues

```typescript
// Verify API keys on startup
function validateEnvironment() {
  const required = [
    'ANTHROPIC_API_KEY',
    'OPENAI_API_KEY',
    'RAPIDAPI_KEY'
  ];

  const missing = required.filter(key => !process.env[key]);
  
  if (missing.length > 0) {
    throw new Error(
      `Missing required environment variables: ${missing.join(', ')}`
    );
  }
}
```

### Research Data Quality

```typescript
// Filter low-quality results
function filterQualityResearch(
  results: ResearchResult[]
): ResearchResult[] {
  return results.filter(r => 
    r.summary.length > 50 &&
    r.title.length > 10 &&
    !r.title.toLowerCase().includes('404')
  );
}
```

### Video Rendering Memory Issues

```typescript
// Render videos sequentially for memory management
async function renderVideosSequential(
  contents: string[]
): Promise<string[]> {
  const videos: string[] = [];
  
  for (const content of contents) {
    const videoPath = await renderContentVideo(content, {
      title: extractTitle(content),
      keyPoints: extractKeyPoints(content, 5),
      duration: 30
    });
    videos.push(videoPath);
    
    // Allow garbage collection
    await new Promise(resolve => setTimeout(resolve, 1000));
  }
  
  return videos;
}
```

### Content Generation Timeouts

```typescript
// Add timeout wrapper
async function generateWithTimeout<T>(
  fn: () => Promise<T>,
  timeoutMs: number = 60000
): Promise<T> {
  return Promise.race([
    fn(),
    new Promise<T>((_, reject) =>
      setTimeout(() => reject(new Error('Operation timeout')), timeoutMs)
    )
  ]);
}
```

## Development Commands

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Run content generation script
npm run generate

# Render videos only
npm run render-videos

# Type check
npm run type-check

# Lint
npm run lint
```
