---
name: marketing-pipeline-ai-content-automation
description: Automate content creation from research to video generation using AI (Claude, OpenAI) and Remotion for Vietnamese and English marketing content
triggers:
  - create automated content pipeline with AI
  - generate marketing content from research to video
  - automate blog post and video creation
  - set up AI content automation system
  - build content pipeline with Claude and OpenAI
  - create multi-format content with AI research
  - automate Vietnamese and English content generation
  - generate videos from blog posts automatically
---

# Marketing Pipeline AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to help developers use the Ultimate AI Content Pipeline - a comprehensive content automation system that handles research, scriptwriting, and video generation. The system crawls news sources, generates multi-format content in Vietnamese and English, and automatically renders videos using Remotion.

## What This Project Does

The Marketing Pipeline is an end-to-end content automation system that:

- **Auto-scans research**: Crawls news from TechCrunch, a16z, Twitter, LinkedIn within the last 24 hours
- **Generates AI content**: Creates blog posts in multiple formats (Toplist, POV, Case Study, How-to) using Claude/OpenAI
- **Multi-language support**: Simultaneously generates Vietnamese and English versions
- **Auto-renders media**: Converts content to infographics and short videos via Remotion
- **Platform optimization**: Exports videos optimized for Reels, TikTok, Shorts

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
# AI API Keys
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# RapidAPI for news crawling
RAPIDAPI_KEY=your_rapidapi_key

# Database (if using)
DATABASE_URL=your_database_url

# Remotion (for video rendering)
REMOTION_LICENSE_KEY=your_remotion_license

# Next.js
NEXT_PUBLIC_API_URL=http://localhost:3000
```

## Development Server

```bash
# Run development server
npm run dev
# or
yarn dev
# or
pnpm dev
```

The application will be available at `http://localhost:3000`

## Key API Endpoints and Usage

### 1. Research Crawling API

```typescript
// pages/api/research/crawl.ts
import type { NextApiRequest, NextApiResponse } from 'next';

interface CrawlRequest {
  keyword: string;
  sources: string[]; // ['techcrunch', 'a16z', 'twitter', 'linkedin']
  timeRange: number; // hours, default 24
  language: 'en' | 'vi' | 'both';
}

interface CrawlResponse {
  articles: Array<{
    title: string;
    url: string;
    source: string;
    publishedAt: string;
    summary: string;
    relevanceScore: number;
  }>;
  insights: string[];
  stats: {
    totalArticles: number;
    avgRelevance: number;
  };
}

// Usage example
const crawlNews = async (keyword: string): Promise<CrawlResponse> => {
  const response = await fetch('/api/research/crawl', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      keyword,
      sources: ['techcrunch', 'twitter'],
      timeRange: 24,
      language: 'both'
    })
  });
  
  return await response.json();
};
```

### 2. AI Content Generation API

```typescript
// pages/api/content/generate.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

interface ContentRequest {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  tone: 'expert' | 'friendly' | 'humorous';
  language: 'en' | 'vi' | 'both';
  researchData: CrawlResponse;
  aiProvider: 'claude' | 'openai';
}

interface ContentResponse {
  en?: {
    title: string;
    content: string;
    metadata: {
      wordCount: number;
      readingTime: number;
    };
  };
  vi?: {
    title: string;
    content: string;
    metadata: {
      wordCount: number;
      readingTime: number;
    };
  };
}

// Claude implementation
const generateWithClaude = async (
  prompt: string,
  researchData: string
): Promise<string> => {
  const anthropic = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY,
  });

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [
      {
        role: 'user',
        content: `${prompt}\n\nResearch Data:\n${researchData}`
      }
    ],
  });

  return message.content[0].type === 'text' 
    ? message.content[0].text 
    : '';
};

// OpenAI implementation
const generateWithOpenAI = async (
  prompt: string,
  researchData: string
): Promise<string> => {
  const openai = new OpenAI({
    apiKey: process.env.OPENAI_API_KEY,
  });

  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: 'You are an expert content writer specializing in marketing and tech content.'
      },
      {
        role: 'user',
        content: `${prompt}\n\nResearch Data:\n${researchData}`
      }
    ],
    max_tokens: 4096,
  });

  return completion.choices[0].message.content || '';
};
```

### 3. Video Rendering API

```typescript
// pages/api/video/render.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoRequest {
  content: string;
  title: string;
  format: 'reels' | 'tiktok' | 'shorts'; // 9:16, 9:16, 9:16
  style: 'minimal' | 'dynamic' | 'professional';
}

interface VideoResponse {
  videoUrl: string;
  thumbnailUrl: string;
  duration: number;
  size: number;
}

const renderVideo = async (req: VideoRequest): Promise<VideoResponse> => {
  const compositionId = 'ContentVideo';
  const bundleLocation = await bundle(
    path.join(process.cwd(), 'remotion/index.ts'),
    () => undefined,
    {
      webpackOverride: (config) => config,
    }
  );

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: {
      title: req.title,
      content: req.content,
      style: req.style,
    },
  });

  const outputLocation = path.join(
    process.cwd(),
    'public/videos',
    `${Date.now()}.mp4`
  );

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: {
      title: req.title,
      content: req.content,
      style: req.style,
    },
  });

  return {
    videoUrl: `/videos/${path.basename(outputLocation)}`,
    thumbnailUrl: `/videos/thumbnails/${path.basename(outputLocation, '.mp4')}.jpg`,
    duration: composition.durationInFrames / composition.fps,
    size: 0, // Get file size
  };
};
```

## Complete Content Pipeline Workflow

```typescript
// lib/pipeline.ts
export class ContentPipeline {
  async execute(keyword: string, options: {
    format: 'toplist' | 'pov' | 'case-study' | 'how-to';
    tone: 'expert' | 'friendly' | 'humorous';
    generateVideo: boolean;
    language: 'en' | 'vi' | 'both';
  }) {
    // Step 1: Research
    console.log('🔍 Starting research crawl...');
    const researchData = await fetch('/api/research/crawl', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        keyword,
        sources: ['techcrunch', 'twitter', 'linkedin'],
        timeRange: 24,
        language: options.language,
      }),
    }).then(r => r.json());

    // Step 2: Generate content
    console.log('✍️ Generating AI content...');
    const content = await fetch('/api/content/generate', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        keyword,
        format: options.format,
        tone: options.tone,
        language: options.language,
        researchData,
        aiProvider: 'claude',
      }),
    }).then(r => r.json());

    // Step 3: Generate video (optional)
    let video = null;
    if (options.generateVideo) {
      console.log('🎬 Rendering video...');
      video = await fetch('/api/video/render', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          content: content.en?.content || content.vi?.content,
          title: content.en?.title || content.vi?.title,
          format: 'reels',
          style: 'dynamic',
        }),
      }).then(r => r.json());
    }

    return {
      research: researchData,
      content,
      video,
    };
  }
}
```

## React Component Example

```typescript
// components/ContentGenerator.tsx
import { useState } from 'react';
import { ContentPipeline } from '@/lib/pipeline';

export default function ContentGenerator() {
  const [keyword, setKeyword] = useState('');
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState<any>(null);

  const handleGenerate = async () => {
    setLoading(true);
    try {
      const pipeline = new ContentPipeline();
      const result = await pipeline.execute(keyword, {
        format: 'toplist',
        tone: 'expert',
        generateVideo: true,
        language: 'both',
      });
      setResult(result);
    } catch (error) {
      console.error('Pipeline error:', error);
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="max-w-4xl mx-auto p-6">
      <h1 className="text-3xl font-bold mb-6">
        AI Content Pipeline
      </h1>
      
      <div className="space-y-4">
        <input
          type="text"
          value={keyword}
          onChange={(e) => setKeyword(e.target.value)}
          placeholder="Enter keyword (e.g., AI startup funding)"
          className="w-full px-4 py-2 border rounded-lg"
        />
        
        <button
          onClick={handleGenerate}
          disabled={loading || !keyword}
          className="px-6 py-2 bg-blue-600 text-white rounded-lg disabled:bg-gray-400"
        >
          {loading ? 'Generating...' : 'Generate Content'}
        </button>
      </div>

      {result && (
        <div className="mt-8 space-y-6">
          <div className="bg-white p-6 rounded-lg shadow">
            <h2 className="text-xl font-bold mb-4">Research Insights</h2>
            <p>Found {result.research.stats.totalArticles} articles</p>
            <ul className="list-disc pl-5 mt-2">
              {result.research.insights.map((insight: string, i: number) => (
                <li key={i}>{insight}</li>
              ))}
            </ul>
          </div>

          {result.content.en && (
            <div className="bg-white p-6 rounded-lg shadow">
              <h2 className="text-xl font-bold mb-4">
                {result.content.en.title}
              </h2>
              <div 
                className="prose"
                dangerouslySetInnerHTML={{ __html: result.content.en.content }}
              />
            </div>
          )}

          {result.video && (
            <div className="bg-white p-6 rounded-lg shadow">
              <h2 className="text-xl font-bold mb-4">Generated Video</h2>
              <video 
                src={result.video.videoUrl}
                controls
                className="w-full max-w-md mx-auto"
              />
            </div>
          )}
        </div>
      )}
    </div>
  );
}
```

## Remotion Video Component

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig } from 'remotion';

interface ContentVideoProps {
  title: string;
  content: string;
  style: 'minimal' | 'dynamic' | 'professional';
}

export const ContentVideo: React.FC<ContentVideoProps> = ({
  title,
  content,
  style,
}) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  const opacity = Math.min(1, frame / 30);
  const scale = 0.8 + Math.min(0.2, frame / 60);

  return (
    <AbsoluteFill
      style={{
        backgroundColor: style === 'professional' ? '#1a1a2e' : '#fff',
        fontFamily: 'Inter, sans-serif',
      }}
    >
      <div
        style={{
          display: 'flex',
          flexDirection: 'column',
          justifyContent: 'center',
          alignItems: 'center',
          padding: 60,
          opacity,
          transform: `scale(${scale})`,
        }}
      >
        <h1
          style={{
            fontSize: 80,
            fontWeight: 'bold',
            color: style === 'professional' ? '#fff' : '#000',
            textAlign: 'center',
            marginBottom: 40,
          }}
        >
          {title}
        </h1>
        
        <p
          style={{
            fontSize: 32,
            color: style === 'professional' ? '#e0e0e0' : '#333',
            textAlign: 'center',
            lineHeight: 1.6,
            maxWidth: 800,
          }}
        >
          {content.substring(0, 200)}...
        </p>
      </div>
    </AbsoluteFill>
  );
};
```

## Common Patterns

### Custom Prompt Templates

```typescript
// lib/prompts.ts
export const contentPrompts = {
  toplist: (keyword: string, language: 'en' | 'vi') => `
    Create a compelling top 10 list article about "${keyword}".
    Language: ${language}
    Format: Numbered list with detailed explanations
    Include: Statistics, examples, and actionable insights
    Tone: Professional yet engaging
  `,
  
  pov: (keyword: string, language: 'en' | 'vi') => `
    Write a thought-provoking opinion piece about "${keyword}".
    Language: ${language}
    Format: Personal perspective with supporting evidence
    Include: Unique angle, counterarguments, and future predictions
    Tone: Authoritative and insightful
  `,
  
  caseStudy: (keyword: string, language: 'en' | 'vi') => `
    Develop a detailed case study analyzing "${keyword}".
    Language: ${language}
    Format: Problem → Solution → Results
    Include: Real data, metrics, and lessons learned
    Tone: Analytical and data-driven
  `,
};
```

### Batch Processing

```typescript
// lib/batch-processor.ts
export async function batchGenerateContent(keywords: string[]) {
  const pipeline = new ContentPipeline();
  const results = [];

  for (const keyword of keywords) {
    console.log(`Processing: ${keyword}`);
    
    try {
      const result = await pipeline.execute(keyword, {
        format: 'toplist',
        tone: 'expert',
        generateVideo: false,
        language: 'both',
      });
      
      results.push({ keyword, success: true, data: result });
    } catch (error) {
      results.push({ keyword, success: false, error: error.message });
    }
    
    // Rate limiting
    await new Promise(resolve => setTimeout(resolve, 2000));
  }

  return results;
}
```

## Troubleshooting

### API Rate Limits

```typescript
// lib/rate-limiter.ts
class RateLimiter {
  private queue: Array<() => Promise<any>> = [];
  private processing = false;
  private requestsPerMinute = 10;
  private interval = 60000 / this.requestsPerMinute;

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
      
      if (!this.processing) {
        this.process();
      }
    });
  }

  private async process() {
    this.processing = true;
    
    while (this.queue.length > 0) {
      const fn = this.queue.shift();
      if (fn) {
        await fn();
        await new Promise(resolve => setTimeout(resolve, this.interval));
      }
    }
    
    this.processing = false;
  }
}

export const rateLimiter = new RateLimiter();
```

### Error Handling

```typescript
// lib/error-handler.ts
export class PipelineError extends Error {
  constructor(
    message: string,
    public stage: 'research' | 'content' | 'video',
    public originalError?: Error
  ) {
    super(message);
    this.name = 'PipelineError';
  }
}

export async function safeExecute<T>(
  fn: () => Promise<T>,
  fallback: T,
  stage: 'research' | 'content' | 'video'
): Promise<T> {
  try {
    return await fn();
  } catch (error) {
    console.error(`Error in ${stage} stage:`, error);
    throw new PipelineError(
      `Failed at ${stage} stage`,
      stage,
      error as Error
    );
  }
}
```

### Memory Management for Large Content

```typescript
// lib/memory-optimizer.ts
export function chunkContent(content: string, maxChunkSize = 2000): string[] {
  const chunks: string[] = [];
  let currentChunk = '';
  
  const sentences = content.split('. ');
  
  for (const sentence of sentences) {
    if ((currentChunk + sentence).length > maxChunkSize) {
      chunks.push(currentChunk);
      currentChunk = sentence + '. ';
    } else {
      currentChunk += sentence + '. ';
    }
  }
  
  if (currentChunk) {
    chunks.push(currentChunk);
  }
  
  return chunks;
}
```

This skill provides comprehensive guidance for using the Marketing Pipeline AI Content Automation system, including research crawling, AI content generation with Claude/OpenAI, video rendering with Remotion, and complete workflow automation for Vietnamese and English marketing content.
