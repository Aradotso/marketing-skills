---
name: marketing-pipeline-ai-content-automation
description: Automate content creation from research to video generation using AI (Claude/OpenAI) and Remotion
triggers:
  - how do I automate content creation with AI
  - set up AI content pipeline with video generation
  - create automated marketing content from research to video
  - use Claude and OpenAI for content automation
  - generate videos from content automatically with Remotion
  - build AI-powered content workflow
  - automate content research and script generation
  - create multi-format content with AI
---

# Marketing Pipeline AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the Ultimate AI Content Pipeline - a TypeScript-based system that automates the entire content creation workflow from research (crawling news sources), to scriptwriting (using Claude/OpenAI), to video generation (using Remotion).

## What This Project Does

The Marketing Pipeline is an all-in-one content automation system that:

- **Auto-scans research sources**: Crawls TechCrunch, a16z, Twitter/X, LinkedIn for fresh data (24h)
- **Generates multi-format content**: Creates articles in various formats (Toplist, POV, Case Study, How-to) in multiple languages
- **Renders videos automatically**: Converts written content into videos/infographics using Remotion
- **Optimizes for platforms**: Exports content ready for Reels, TikTok, Shorts
- **Integrates AI providers**: Uses Claude 3 (Anthropic) and OpenAI for content generation

## Installation

### Prerequisites

```bash
# Node.js 18+ and npm/yarn required
node --version  # Should be 18+
```

### Clone and Install

```bash
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share
npm install
# or
yarn install
```

### Environment Configuration

Create a `.env.local` file in the root directory:

```bash
# AI Providers
OPENAI_API_KEY=your_openai_key_here
ANTHROPIC_API_KEY=your_claude_key_here

# RapidAPI for research crawling
RAPIDAPI_KEY=your_rapidapi_key_here

# Optional: Content generation settings
DEFAULT_LANGUAGE=vi
FALLBACK_LANGUAGE=en
OUTPUT_FORMAT=json

# Remotion video rendering
REMOTION_LICENSE_KEY=your_remotion_key_here
```

### Start Development Server

```bash
npm run dev
# or
yarn dev
```

The app will be available at `http://localhost:3000`

## Key Architecture

### Directory Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app router pages
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI provider integrations
│   │   ├── research/    # Content research/crawling
│   │   ├── video/       # Remotion video generation
│   │   └── utils/       # Utility functions
│   └── types/           # TypeScript type definitions
├── public/              # Static assets
└── remotion/            # Remotion video templates
```

## Core API Usage

### 1. Research & Crawling

```typescript
// src/lib/research/crawler.ts
import { RapidAPIClient } from '@/lib/research/rapidapi-client';

interface ResearchResult {
  title: string;
  source: string;
  url: string;
  content: string;
  publishedAt: Date;
}

export async function crawlNewsForKeyword(
  keyword: string,
  sources: string[] = ['techcrunch', 'a16z']
): Promise<ResearchResult[]> {
  const client = new RapidAPIClient(process.env.RAPIDAPI_KEY!);
  
  const results = await Promise.all(
    sources.map(source => 
      client.searchNews({
        query: keyword,
        source: source,
        timeRange: '24h'
      })
    )
  );
  
  return results.flat();
}

// Usage
const research = await crawlNewsForKeyword('AI marketing automation');
console.log(`Found ${research.length} articles`);
```

### 2. AI Content Generation

```typescript
// src/lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

interface ContentGenerationOptions {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  researchData: ResearchResult[];
}

export class ContentGenerator {
  private anthropic: Anthropic;
  private openai: OpenAI;
  
  constructor() {
    this.anthropic = new Anthropic({
      apiKey: process.env.ANTHROPIC_API_KEY
    });
    this.openai = new OpenAI({
      apiKey: process.env.OPENAI_API_KEY
    });
  }
  
  async generateWithClaude(options: ContentGenerationOptions): Promise<string> {
    const prompt = this.buildPrompt(options);
    
    const message = await this.anthropic.messages.create({
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
  
  async generateWithOpenAI(options: ContentGenerationOptions): Promise<string> {
    const prompt = this.buildPrompt(options);
    
    const completion = await this.openai.chat.completions.create({
      model: 'gpt-4-turbo-preview',
      messages: [{
        role: 'user',
        content: prompt
      }],
      max_tokens: 4096
    });
    
    return completion.choices[0]?.message?.content || '';
  }
  
  private buildPrompt(options: ContentGenerationOptions): string {
    const researchSummary = options.researchData
      .map(r => `- ${r.title}: ${r.content.substring(0, 200)}...`)
      .join('\n');
    
    return `Create a ${options.format} article in ${options.language} about "${options.keyword}".
    
Tone: ${options.tone}
Recent research data:
${researchSummary}

Requirements:
- Use real data from research
- Include statistics and examples
- Optimize for SEO
- ${options.language === 'vi' ? 'Write in Vietnamese' : 'Write in English'}`;
  }
}

// Usage
const generator = new ContentGenerator();
const content = await generator.generateWithClaude({
  keyword: 'AI marketing automation',
  format: 'how-to',
  language: 'en',
  tone: 'expert',
  researchData: research
});
```

### 3. Video Generation with Remotion

```typescript
// src/lib/video/video-renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoRenderOptions {
  content: string;
  title: string;
  format: 'reels' | 'tiktok' | 'shorts'; // All use 9:16
  outputPath: string;
}

export async function renderContentVideo(
  options: VideoRenderOptions
): Promise<string> {
  // Define composition settings based on format
  const compositionConfig = {
    width: 1080,
    height: 1920, // 9:16 aspect ratio
    fps: 30,
    durationInFrames: 300 // 10 seconds at 30fps
  };
  
  // Bundle the Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.join(process.cwd(), 'remotion/index.ts'),
    webpackOverride: (config) => config
  });
  
  // Select composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title: options.title,
      content: options.content,
      format: options.format
    }
  });
  
  // Render video
  await renderMedia({
    composition: {
      ...composition,
      ...compositionConfig
    },
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: options.outputPath,
    inputProps: {
      title: options.title,
      content: options.content,
      format: options.format
    }
  });
  
  return options.outputPath;
}

// Usage
const videoPath = await renderContentVideo({
  content: content,
  title: 'AI Marketing Automation Guide',
  format: 'reels',
  outputPath: './output/video.mp4'
});
```

### 4. Complete Pipeline Integration

```typescript
// src/lib/pipeline/content-pipeline.ts
import { crawlNewsForKeyword } from '@/lib/research/crawler';
import { ContentGenerator } from '@/lib/ai/content-generator';
import { renderContentVideo } from '@/lib/video/video-renderer';

interface PipelineConfig {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  generateVideo: boolean;
  videoFormat?: 'reels' | 'tiktok' | 'shorts';
}

export class ContentPipeline {
  private generator: ContentGenerator;
  
  constructor() {
    this.generator = new ContentGenerator();
  }
  
  async execute(config: PipelineConfig) {
    console.log(`Starting pipeline for: ${config.keyword}`);
    
    // Step 1: Research
    console.log('Step 1/3: Crawling research data...');
    const research = await crawlNewsForKeyword(config.keyword);
    console.log(`Found ${research.length} articles`);
    
    // Step 2: Generate content
    console.log('Step 2/3: Generating content with AI...');
    const content = await this.generator.generateWithClaude({
      keyword: config.keyword,
      format: config.format,
      language: config.language,
      tone: 'expert',
      researchData: research
    });
    
    // Step 3: Generate video (optional)
    let videoPath: string | null = null;
    if (config.generateVideo && config.videoFormat) {
      console.log('Step 3/3: Rendering video...');
      videoPath = await renderContentVideo({
        content: content,
        title: config.keyword,
        format: config.videoFormat,
        outputPath: `./output/${Date.now()}.mp4`
      });
      console.log(`Video saved to: ${videoPath}`);
    }
    
    return {
      content,
      videoPath,
      research,
      metadata: {
        keyword: config.keyword,
        format: config.format,
        language: config.language,
        generatedAt: new Date()
      }
    };
  }
}

// Usage
const pipeline = new ContentPipeline();
const result = await pipeline.execute({
  keyword: 'AI marketing automation trends 2024',
  format: 'toplist',
  language: 'en',
  generateVideo: true,
  videoFormat: 'reels'
});

console.log('Content:', result.content);
console.log('Video:', result.videoPath);
```

## Common Patterns

### Multi-Language Content Generation

```typescript
// Generate content in both languages simultaneously
async function generateBilingualContent(keyword: string) {
  const generator = new ContentGenerator();
  const research = await crawlNewsForKeyword(keyword);
  
  const [englishContent, vietnameseContent] = await Promise.all([
    generator.generateWithClaude({
      keyword,
      format: 'how-to',
      language: 'en',
      tone: 'expert',
      researchData: research
    }),
    generator.generateWithClaude({
      keyword,
      format: 'how-to',
      language: 'vi',
      tone: 'friendly',
      researchData: research
    })
  ]);
  
  return { english: englishContent, vietnamese: vietnameseContent };
}
```

### Batch Content Generation

```typescript
// Generate multiple pieces of content for different keywords
async function batchGenerateContent(keywords: string[]) {
  const pipeline = new ContentPipeline();
  
  const results = await Promise.all(
    keywords.map(keyword => 
      pipeline.execute({
        keyword,
        format: 'toplist',
        language: 'en',
        generateVideo: false
      })
    )
  );
  
  return results;
}

// Usage
const contents = await batchGenerateContent([
  'AI marketing tools',
  'Content automation strategies',
  'Video marketing trends'
]);
```

### Error Handling and Retry Logic

```typescript
async function generateWithRetry(
  options: ContentGenerationOptions,
  maxRetries: number = 3
): Promise<string> {
  const generator = new ContentGenerator();
  
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await generator.generateWithClaude(options);
    } catch (error) {
      console.error(`Attempt ${i + 1} failed:`, error);
      
      if (i === maxRetries - 1) {
        // Fallback to OpenAI if Claude fails
        console.log('Falling back to OpenAI...');
        return await generator.generateWithOpenAI(options);
      }
      
      // Exponential backoff
      await new Promise(resolve => 
        setTimeout(resolve, Math.pow(2, i) * 1000)
      );
    }
  }
  
  throw new Error('All retry attempts failed');
}
```

## API Routes (Next.js)

### Generate Content Endpoint

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { ContentPipeline } from '@/lib/pipeline/content-pipeline';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const { keyword, format, language, generateVideo, videoFormat } = body;
    
    // Validate input
    if (!keyword) {
      return NextResponse.json(
        { error: 'Keyword is required' },
        { status: 400 }
      );
    }
    
    const pipeline = new ContentPipeline();
    const result = await pipeline.execute({
      keyword,
      format: format || 'how-to',
      language: language || 'en',
      generateVideo: generateVideo || false,
      videoFormat: videoFormat
    });
    
    return NextResponse.json(result);
  } catch (error) {
    console.error('Generation error:', error);
    return NextResponse.json(
      { error: 'Failed to generate content' },
      { status: 500 }
    );
  }
}
```

### Usage from Frontend

```typescript
// Example React component usage
'use client';

import { useState } from 'react';

export default function ContentGenerator() {
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState(null);
  
  const generateContent = async () => {
    setLoading(true);
    
    try {
      const response = await fetch('/api/generate', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          keyword: 'AI marketing automation',
          format: 'how-to',
          language: 'en',
          generateVideo: true,
          videoFormat: 'reels'
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
    <div>
      <button onClick={generateContent} disabled={loading}>
        {loading ? 'Generating...' : 'Generate Content'}
      </button>
      
      {result && (
        <div>
          <h2>Generated Content:</h2>
          <pre>{JSON.stringify(result, null, 2)}</pre>
        </div>
      )}
    </div>
  );
}
```

## Troubleshooting

### API Key Issues

```typescript
// Validate API keys on startup
function validateEnvironment() {
  const required = [
    'OPENAI_API_KEY',
    'ANTHROPIC_API_KEY',
    'RAPIDAPI_KEY'
  ];
  
  const missing = required.filter(key => !process.env[key]);
  
  if (missing.length > 0) {
    throw new Error(
      `Missing required environment variables: ${missing.join(', ')}`
    );
  }
}

// Call in your main application file
validateEnvironment();
```

### Rate Limiting

```typescript
// Implement rate limiting for API calls
class RateLimiter {
  private queue: (() => Promise<any>)[] = [];
  private processing = false;
  private requestsPerMinute: number;
  
  constructor(requestsPerMinute: number = 10) {
    this.requestsPerMinute = requestsPerMinute;
  }
  
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
      
      this.processQueue();
    });
  }
  
  private async processQueue() {
    if (this.processing || this.queue.length === 0) return;
    
    this.processing = true;
    const fn = this.queue.shift()!;
    
    await fn();
    
    setTimeout(() => {
      this.processing = false;
      this.processQueue();
    }, 60000 / this.requestsPerMinute);
  }
}

// Usage
const limiter = new RateLimiter(10);
const result = await limiter.add(() => 
  generator.generateWithClaude(options)
);
```

### Video Rendering Memory Issues

```typescript
// For large video renders, use chunked processing
async function renderLargeVideo(options: VideoRenderOptions) {
  // Increase Node.js memory limit
  process.env.NODE_OPTIONS = '--max-old-space-size=4096';
  
  try {
    return await renderContentVideo(options);
  } catch (error) {
    if (error.message.includes('heap out of memory')) {
      console.error('Memory error - try reducing video duration or quality');
    }
    throw error;
  }
}
```

### Research Crawling Timeouts

```typescript
// Add timeout handling for research crawling
async function crawlWithTimeout(
  keyword: string,
  timeoutMs: number = 30000
): Promise<ResearchResult[]> {
  const timeoutPromise = new Promise<never>((_, reject) => 
    setTimeout(() => reject(new Error('Crawl timeout')), timeoutMs)
  );
  
  try {
    return await Promise.race([
      crawlNewsForKeyword(keyword),
      timeoutPromise
    ]);
  } catch (error) {
    console.warn('Crawl timed out, using cached data');
    return []; // Return empty or cached results
  }
}
```

## Best Practices

1. **Always use environment variables** for API keys - never hardcode them
2. **Implement retry logic** for AI generation calls (APIs can be flaky)
3. **Cache research results** to avoid unnecessary API calls
4. **Use streaming responses** for real-time content generation feedback
5. **Monitor token usage** for OpenAI/Claude to control costs
6. **Validate input** thoroughly before passing to AI providers
7. **Store generated content** in a database for reuse and analytics
