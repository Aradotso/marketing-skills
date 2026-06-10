---
name: marketing-pipeline-share-content-automation
description: AI-powered content automation pipeline that researches, generates scripts, and creates videos automatically using Claude, OpenAI, and Remotion
triggers:
  - automate content creation with AI research and video generation
  - build automated marketing content pipeline with Claude
  - create content from research to video automatically
  - set up AI content automation with Remotion rendering
  - generate marketing videos from text with AI
  - automate social media content workflow
  - build end-to-end content generation pipeline
  - research and create videos automatically with AI
---

# Marketing Pipeline Share - Content Automation Skill

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the Ultimate AI Content Pipeline, a comprehensive TypeScript-based system that automates content creation from research through video generation. The pipeline integrates Claude 3, OpenAI, and Remotion to transform keywords into finished content and videos.

## What This Project Does

The Marketing Pipeline Share project is an all-in-one content automation system that:

- **Auto-scans and researches** recent news from TechCrunch, a16z, Twitter/X, and LinkedIn
- **Generates multi-format content** (Toplist, POV, Case Study, How-to) in multiple languages
- **Renders videos and infographics** automatically using Remotion
- **Optimizes for multiple platforms** (Reels, TikTok, Shorts)
- **Provides a Next.js interface** for managing the entire workflow

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

Create a `.env.local` file in the root directory:

```bash
# AI Providers
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Remotion Configuration
REMOTION_TIMEOUT=120000

# Next.js Configuration
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

### Running the Application

```bash
# Development mode
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Render videos (Remotion)
npm run render
```

## Core Components

### 1. Research & Data Collection

The system automatically crawls and analyzes recent news sources:

```typescript
// lib/research/crawler.ts
import { ResearchSource } from '@/types';

export async function fetchLatestNews(
  keyword: string,
  sources: ResearchSource[] = ['techcrunch', 'a16z', 'twitter']
): Promise<NewsItem[]> {
  const results = await Promise.all(
    sources.map(source => crawlSource(source, keyword))
  );
  
  return results.flat().sort((a, b) => 
    new Date(b.publishedAt).getTime() - new Date(a.publishedAt).getTime()
  );
}

async function crawlSource(
  source: ResearchSource, 
  keyword: string
): Promise<NewsItem[]> {
  const endpoint = getSourceEndpoint(source);
  
  const response = await fetch(endpoint, {
    headers: {
      'X-RapidAPI-Key': process.env.RAPIDAPI_KEY!,
      'X-RapidAPI-Host': getSourceHost(source)
    }
  });
  
  const data = await response.json();
  return parseSourceData(source, data, keyword);
}
```

### 2. AI Content Generation

Generate content using Claude or OpenAI:

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

export interface ContentRequest {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  researchData: NewsItem[];
}

export async function generateContent(
  request: ContentRequest,
  provider: 'claude' | 'openai' = 'claude'
): Promise<GeneratedContent> {
  const prompt = buildContentPrompt(request);
  
  if (provider === 'claude') {
    const message = await anthropic.messages.create({
      model: 'claude-3-5-sonnet-20241022',
      max_tokens: 4096,
      messages: [{
        role: 'user',
        content: prompt
      }]
    });
    
    return parseClaudeResponse(message);
  } else {
    const completion = await openai.chat.completions.create({
      model: 'gpt-4-turbo-preview',
      messages: [{
        role: 'user',
        content: prompt
      }],
      temperature: 0.7,
    });
    
    return parseOpenAIResponse(completion);
  }
}

function buildContentPrompt(request: ContentRequest): string {
  const { keyword, format, language, tone, researchData } = request;
  
  const researchContext = researchData
    .map(item => `- ${item.title} (${item.source}): ${item.summary}`)
    .join('\n');
  
  return `
You are a ${tone} content creator. Generate a ${format} article about "${keyword}" in ${language}.

Recent Research Data:
${researchContext}

Requirements:
- Format: ${format}
- Language: ${language}
- Tone: ${tone}
- Include data-backed insights from the research
- Make it engaging and actionable
- Optimize for social media sharing

Generate the content with proper structure including title, introduction, main sections, and conclusion.
`;
}
```

### 3. Video Rendering with Remotion

Create videos from generated content:

```typescript
// remotion/compositions/ContentVideo.tsx
import { Composition } from 'remotion';
import { VideoSequence } from './VideoSequence';

export const RemotionRoot: React.FC = () => {
  return (
    <>
      <Composition
        id="ContentVideo"
        component={VideoSequence}
        durationInFrames={300}
        fps={30}
        width={1080}
        height={1920}
        defaultProps={{
          title: '',
          sections: [],
          style: 'modern'
        }}
      />
    </>
  );
};
```

```typescript
// remotion/compositions/VideoSequence.tsx
import { AbsoluteFill, useCurrentFrame, interpolate } from 'remotion';

interface VideoSequenceProps {
  title: string;
  sections: ContentSection[];
  style: 'modern' | 'minimal' | 'vibrant';
}

export const VideoSequence: React.FC<VideoSequenceProps> = ({
  title,
  sections,
  style
}) => {
  const frame = useCurrentFrame();
  
  const opacity = interpolate(
    frame,
    [0, 30],
    [0, 1],
    { extrapolateRight: 'clamp' }
  );
  
  return (
    <AbsoluteFill style={{ backgroundColor: '#000' }}>
      <div style={{ opacity, padding: 60 }}>
        <h1 style={{ 
          fontSize: 72, 
          color: '#fff',
          fontWeight: 'bold',
          marginBottom: 40
        }}>
          {title}
        </h1>
        
        {sections.map((section, index) => (
          <SectionFrame
            key={index}
            section={section}
            startFrame={30 + (index * 90)}
            style={style}
          />
        ))}
      </div>
    </AbsoluteFill>
  );
};
```

```typescript
// lib/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

export async function renderContentVideo(
  content: GeneratedContent,
  outputPath: string
): Promise<string> {
  const bundleLocation = await bundle(
    path.join(process.cwd(), 'remotion/index.ts')
  );
  
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title: content.title,
      sections: content.sections,
      style: 'modern'
    },
  });
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: {
      title: content.title,
      sections: content.sections,
      style: 'modern'
    },
  });
  
  return outputPath;
}
```

## API Routes (Next.js)

### Generate Content API

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { fetchLatestNews } from '@/lib/research/crawler';
import { generateContent } from '@/lib/ai/content-generator';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const { keyword, format, language, tone, provider } = body;
    
    // Validate input
    if (!keyword || !format || !language) {
      return NextResponse.json(
        { error: 'Missing required fields' },
        { status: 400 }
      );
    }
    
    // Step 1: Research
    const researchData = await fetchLatestNews(keyword);
    
    // Step 2: Generate content
    const content = await generateContent(
      {
        keyword,
        format,
        language,
        tone: tone || 'expert',
        researchData
      },
      provider || 'claude'
    );
    
    return NextResponse.json({
      success: true,
      content,
      researchCount: researchData.length
    });
    
  } catch (error) {
    console.error('Content generation error:', error);
    return NextResponse.json(
      { error: 'Failed to generate content' },
      { status: 500 }
    );
  }
}
```

### Render Video API

```typescript
// app/api/render-video/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { renderContentVideo } from '@/lib/video/renderer';
import path from 'path';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const { content, contentId } = body;
    
    if (!content || !contentId) {
      return NextResponse.json(
        { error: 'Missing content or contentId' },
        { status: 400 }
      );
    }
    
    const outputPath = path.join(
      process.cwd(),
      'public',
      'videos',
      `${contentId}.mp4`
    );
    
    await renderContentVideo(content, outputPath);
    
    return NextResponse.json({
      success: true,
      videoUrl: `/videos/${contentId}.mp4`
    });
    
  } catch (error) {
    console.error('Video rendering error:', error);
    return NextResponse.json(
      { error: 'Failed to render video' },
      { status: 500 }
    );
  }
}
```

## Common Usage Patterns

### Full Pipeline Workflow

```typescript
// lib/pipeline/orchestrator.ts
export class ContentPipeline {
  async execute(keyword: string, options: PipelineOptions) {
    console.log(`Starting pipeline for: ${keyword}`);
    
    // Phase 1: Research
    const research = await this.research(keyword);
    console.log(`Found ${research.length} sources`);
    
    // Phase 2: Content Generation
    const content = await this.generateContent(keyword, research, options);
    console.log(`Generated ${options.language} content`);
    
    // Phase 3: Video Rendering (optional)
    let videoUrl = null;
    if (options.generateVideo) {
      videoUrl = await this.renderVideo(content);
      console.log(`Video rendered: ${videoUrl}`);
    }
    
    return {
      content,
      research,
      videoUrl,
      metadata: {
        keyword,
        createdAt: new Date().toISOString(),
        format: options.format,
        language: options.language
      }
    };
  }
  
  private async research(keyword: string) {
    return fetchLatestNews(keyword, ['techcrunch', 'a16z', 'twitter']);
  }
  
  private async generateContent(
    keyword: string,
    research: NewsItem[],
    options: PipelineOptions
  ) {
    return generateContent({
      keyword,
      format: options.format,
      language: options.language,
      tone: options.tone,
      researchData: research
    }, options.aiProvider);
  }
  
  private async renderVideo(content: GeneratedContent) {
    const contentId = generateId();
    const outputPath = `./public/videos/${contentId}.mp4`;
    await renderContentVideo(content, outputPath);
    return `/videos/${contentId}.mp4`;
  }
}
```

### Using the Pipeline

```typescript
// Example: Create content from keyword
import { ContentPipeline } from '@/lib/pipeline/orchestrator';

async function createMarketingContent() {
  const pipeline = new ContentPipeline();
  
  const result = await pipeline.execute('AI Content Marketing 2024', {
    format: 'toplist',
    language: 'en',
    tone: 'expert',
    aiProvider: 'claude',
    generateVideo: true
  });
  
  console.log('Content:', result.content.title);
  console.log('Video URL:', result.videoUrl);
  console.log('Sources:', result.research.length);
  
  return result;
}
```

### Batch Processing

```typescript
// lib/pipeline/batch-processor.ts
export async function processBatch(keywords: string[]) {
  const pipeline = new ContentPipeline();
  const results = [];
  
  for (const keyword of keywords) {
    try {
      const result = await pipeline.execute(keyword, {
        format: 'how-to',
        language: 'vi',
        tone: 'friendly',
        aiProvider: 'claude',
        generateVideo: false
      });
      
      results.push({ keyword, success: true, result });
      
      // Rate limiting
      await new Promise(resolve => setTimeout(resolve, 2000));
      
    } catch (error) {
      results.push({ 
        keyword, 
        success: false, 
        error: (error as Error).message 
      });
    }
  }
  
  return results;
}
```

## Configuration

### Content Formats Configuration

```typescript
// config/content-formats.ts
export const CONTENT_FORMATS = {
  toplist: {
    structure: ['intro', 'items', 'conclusion'],
    itemCount: { min: 5, max: 10 },
    sections: ['title', 'description', 'why']
  },
  pov: {
    structure: ['hook', 'opinion', 'evidence', 'conclusion'],
    tone: ['controversial', 'thoughtful', 'provocative']
  },
  'case-study': {
    structure: ['background', 'challenge', 'solution', 'results'],
    includeMetrics: true
  },
  'how-to': {
    structure: ['overview', 'steps', 'tips', 'conclusion'],
    stepFormat: 'numbered'
  }
};
```

### Video Style Configuration

```typescript
// config/video-styles.ts
export const VIDEO_STYLES = {
  modern: {
    backgroundColor: '#000000',
    primaryColor: '#00ff88',
    fontFamily: 'Inter',
    animation: 'smooth'
  },
  minimal: {
    backgroundColor: '#ffffff',
    primaryColor: '#000000',
    fontFamily: 'Helvetica',
    animation: 'subtle'
  },
  vibrant: {
    backgroundColor: '#ff0080',
    primaryColor: '#00ffff',
    fontFamily: 'Poppins',
    animation: 'energetic'
  }
};
```

## Troubleshooting

### API Rate Limits

```typescript
// lib/utils/rate-limiter.ts
export class RateLimiter {
  private queue: Array<() => Promise<any>> = [];
  private processing = false;
  private delay: number;
  
  constructor(requestsPerMinute: number) {
    this.delay = 60000 / requestsPerMinute;
  }
  
  async execute<T>(fn: () => Promise<T>): Promise<T> {
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
    const fn = this.queue.shift();
    
    if (fn) {
      await fn();
      await new Promise(resolve => setTimeout(resolve, this.delay));
    }
    
    this.processing = false;
    this.processQueue();
  }
}

// Usage
const limiter = new RateLimiter(10); // 10 requests per minute

await limiter.execute(() => 
  generateContent(contentRequest, 'claude')
);
```

### Video Rendering Timeouts

```typescript
// Increase timeout in remotion.config.ts
import { Config } from '@remotion/cli/config';

Config.setVideoImageFormat('jpeg');
Config.setOverwriteOutput(true);
Config.setDelayRenderTimeoutInMilliseconds(120000);
Config.setConcurrency(4);
```

### Memory Issues with Large Content

```typescript
// lib/utils/chunk-processor.ts
export async function processInChunks<T, R>(
  items: T[],
  processor: (chunk: T[]) => Promise<R[]>,
  chunkSize: number = 5
): Promise<R[]> {
  const results: R[] = [];
  
  for (let i = 0; i < items.length; i += chunkSize) {
    const chunk = items.slice(i, i + chunkSize);
    const chunkResults = await processor(chunk);
    results.push(...chunkResults);
    
    // Clear memory
    if (global.gc) {
      global.gc();
    }
  }
  
  return results;
}
```

### Error Handling Best Practices

```typescript
// lib/utils/error-handler.ts
export class PipelineError extends Error {
  constructor(
    message: string,
    public phase: 'research' | 'generation' | 'rendering',
    public originalError?: Error
  ) {
    super(message);
    this.name = 'PipelineError';
  }
}

export function handlePipelineError(error: unknown): never {
  if (error instanceof PipelineError) {
    console.error(`[${error.phase}] ${error.message}`);
    if (error.originalError) {
      console.error('Original error:', error.originalError);
    }
  } else if (error instanceof Error) {
    console.error('Unexpected error:', error.message);
  }
  
  throw error;
}
```

This skill provides comprehensive guidance for working with the Marketing Pipeline Share project, covering research automation, AI content generation, video rendering, and production deployment patterns.
