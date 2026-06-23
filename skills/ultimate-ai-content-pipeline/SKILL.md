---
name: ultimate-ai-content-pipeline
description: Automated content generation system from research to video using Claude/OpenAI AI, web scraping, and Remotion video rendering
triggers:
  - how do I generate content with AI pipeline
  - automate content creation from research to video
  - use ultimate ai content pipeline for marketing
  - generate videos from text with remotion
  - scrape news and create content automatically
  - build automated content workflow with claude
  - create social media videos with ai
  - set up ai content generation pipeline
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

Ultimate AI Content Pipeline is an end-to-end automated content generation system that scrapes research from news sources, generates multi-format content using Claude/OpenAI, and renders videos using Remotion. It's designed for content creators, marketers, and businesses to automate 90% of their content workflow.

## What It Does

- **Auto-Research**: Crawls TechCrunch, a16z, Twitter/X, LinkedIn for fresh data
- **AI Content Generation**: Creates articles in multiple formats (Toplist, POV, Case Study, How-to) using Claude 3 or OpenAI
- **Multi-language**: Generates Vietnamese and English content simultaneously
- **Video Generation**: Automatically renders infographics and short-form videos using Remotion
- **Multi-platform**: Exports videos optimized for Reels, TikTok, Shorts

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
ANTHROPIC_API_KEY=your_claude_api_key_here
OPENAI_API_KEY=your_openai_api_key_here

# Research/Scraping APIs
RAPIDAPI_KEY=your_rapidapi_key_here

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Remotion Config
REMOTION_LICENSE_KEY=your_remotion_license_key_here

# Next.js
NEXT_PUBLIC_API_URL=http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── app/                    # Next.js app directory
│   ├── api/               # API routes
│   │   ├── research/      # Research/scraping endpoints
│   │   ├── generate/      # Content generation endpoints
│   │   └── render/        # Video rendering endpoints
│   └── components/        # React components
├── lib/                   # Core utilities
│   ├── ai/               # AI integrations (Claude, OpenAI)
│   ├── scraper/          # Web scraping logic
│   └── remotion/         # Remotion video templates
├── public/               # Static assets
└── remotion/            # Remotion video compositions
```

## Core APIs and Usage

### 1. Research & Scraping

Automatically fetch fresh content from news sources:

```typescript
// lib/scraper/research.ts
import { ScraperService } from './scraper-service';

interface ResearchResult {
  title: string;
  content: string;
  source: string;
  publishedAt: Date;
  insights: string[];
}

export async function performResearch(
  keyword: string,
  sources: string[] = ['techcrunch', 'a16z', 'twitter']
): Promise<ResearchResult[]> {
  const scraper = new ScraperService({
    rapidApiKey: process.env.RAPIDAPI_KEY!
  });

  const results = await scraper.searchMultipleSources({
    keyword,
    sources,
    timeRange: '24h',
    maxResults: 10
  });

  return results.map(item => ({
    title: item.title,
    content: item.content,
    source: item.source,
    publishedAt: new Date(item.published),
    insights: extractInsights(item.content)
  }));
}

function extractInsights(content: string): string[] {
  // Extract key insights, statistics, and quotes
  const insights: string[] = [];
  const statPattern = /\d+%|\d+x|\$\d+[MBK]/g;
  const stats = content.match(statPattern);
  
  if (stats) {
    insights.push(...stats);
  }
  
  return insights;
}
```

### 2. AI Content Generation

Generate content using Claude or OpenAI:

```typescript
// lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

interface ContentRequest {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi' | 'both';
  tone: 'expert' | 'friendly' | 'humorous';
  researchData: ResearchResult[];
}

interface GeneratedContent {
  title: string;
  content: string;
  language: string;
  metadata: {
    wordCount: number;
    readingTime: number;
  };
}

export class ContentGenerator {
  private anthropic: Anthropic;
  private openai: OpenAI;

  constructor() {
    this.anthropic = new Anthropic({
      apiKey: process.env.ANTHROPIC_API_KEY!
    });
    
    this.openai = new OpenAI({
      apiKey: process.env.OPENAI_API_KEY!
    });
  }

  async generateWithClaude(request: ContentRequest): Promise<GeneratedContent> {
    const systemPrompt = this.buildSystemPrompt(request);
    const userPrompt = this.buildUserPrompt(request);

    const message = await this.anthropic.messages.create({
      model: 'claude-3-5-sonnet-20241022',
      max_tokens: 4096,
      system: systemPrompt,
      messages: [
        {
          role: 'user',
          content: userPrompt
        }
      ]
    });

    const content = message.content[0].type === 'text' 
      ? message.content[0].text 
      : '';

    return this.parseContent(content, request.language);
  }

  async generateWithOpenAI(request: ContentRequest): Promise<GeneratedContent> {
    const systemPrompt = this.buildSystemPrompt(request);
    const userPrompt = this.buildUserPrompt(request);

    const completion = await this.openai.chat.completions.create({
      model: 'gpt-4-turbo-preview',
      messages: [
        { role: 'system', content: systemPrompt },
        { role: 'user', content: userPrompt }
      ],
      temperature: 0.7,
      max_tokens: 4096
    });

    const content = completion.choices[0].message.content || '';
    return this.parseContent(content, request.language);
  }

  private buildSystemPrompt(request: ContentRequest): string {
    const toneMap = {
      expert: 'professional and authoritative',
      friendly: 'warm and conversational',
      humorous: 'entertaining and witty'
    };

    return `You are an expert content writer specializing in ${request.format} format.
Write in a ${toneMap[request.tone]} tone.
Use the provided research data to create data-backed, insightful content.
Include specific statistics, quotes, and examples from the research.`;
  }

  private buildUserPrompt(request: ContentRequest): string {
    const researchSummary = request.researchData
      .map(r => `Source: ${r.source}\nTitle: ${r.title}\nContent: ${r.content.slice(0, 500)}...\nInsights: ${r.insights.join(', ')}`)
      .join('\n\n---\n\n');

    return `Create a ${request.format} article about "${request.keyword}" in ${request.language === 'both' ? 'English and Vietnamese' : request.language}.

Research Data:
${researchSummary}

Generate comprehensive content with:
1. Engaging title
2. Introduction with hook
3. Main content (use ${request.format} structure)
4. Data-backed insights from research
5. Conclusion with call-to-action

Format as JSON with title and content fields.`;
  }

  private parseContent(rawContent: string, language: string): GeneratedContent {
    try {
      const parsed = JSON.parse(rawContent);
      const wordCount = parsed.content.split(/\s+/).length;
      
      return {
        title: parsed.title,
        content: parsed.content,
        language,
        metadata: {
          wordCount,
          readingTime: Math.ceil(wordCount / 200)
        }
      };
    } catch {
      // Fallback if not JSON
      return {
        title: 'Generated Content',
        content: rawContent,
        language,
        metadata: {
          wordCount: rawContent.split(/\s+/).length,
          readingTime: Math.ceil(rawContent.split(/\s+/).length / 200)
        }
      };
    }
  }
}
```

### 3. Video Rendering with Remotion

Create videos from generated content:

```typescript
// lib/remotion/video-renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import { webpackOverride } from './webpack-override';

interface VideoConfig {
  content: GeneratedContent;
  format: 'reels' | 'tiktok' | 'shorts';
  style: 'minimal' | 'dynamic' | 'professional';
}

interface VideoOutput {
  path: string;
  duration: number;
  size: number;
}

export class VideoRenderer {
  async renderContentVideo(config: VideoConfig): Promise<VideoOutput> {
    const { content, format, style } = config;
    
    // Bundle Remotion project
    const bundleLocation = await bundle({
      entryPoint: './remotion/index.ts',
      webpackOverride
    });

    // Get composition dimensions based on format
    const dimensions = this.getDimensions(format);
    
    // Select composition
    const composition = await selectComposition({
      serveUrl: bundleLocation,
      id: `ContentVideo-${style}`,
      inputProps: {
        title: content.title,
        content: content.content,
        language: content.language
      }
    });

    // Render video
    const outputPath = `./public/videos/output-${Date.now()}.mp4`;
    
    await renderMedia({
      composition,
      serveUrl: bundleLocation,
      codec: 'h264',
      outputLocation: outputPath,
      inputProps: {
        title: content.title,
        content: content.content,
        style
      }
    });

    return {
      path: outputPath,
      duration: composition.durationInFrames / composition.fps,
      size: 0 // Get actual file size
    };
  }

  private getDimensions(format: 'reels' | 'tiktok' | 'shorts') {
    const dimensionMap = {
      reels: { width: 1080, height: 1920 },
      tiktok: { width: 1080, height: 1920 },
      shorts: { width: 1080, height: 1920 }
    };
    return dimensionMap[format];
  }
}
```

### 4. Remotion Video Composition

```typescript
// remotion/compositions/ContentVideo.tsx
import { AbsoluteFill, interpolate, useCurrentFrame, useVideoConfig } from 'remotion';
import React from 'react';

interface ContentVideoProps {
  title: string;
  content: string;
  style: 'minimal' | 'dynamic' | 'professional';
}

export const ContentVideo: React.FC<ContentVideoProps> = ({ title, content, style }) => {
  const frame = useCurrentFrame();
  const { fps, durationInFrames } = useVideoConfig();

  // Title animation
  const titleOpacity = interpolate(
    frame,
    [0, 30],
    [0, 1],
    { extrapolateRight: 'clamp' }
  );

  const titleScale = interpolate(
    frame,
    [0, 30],
    [0.8, 1],
    { extrapolateRight: 'clamp' }
  );

  // Content sections
  const contentParts = content.split('\n\n').filter(p => p.trim());
  const framesPerSection = Math.floor((durationInFrames - 60) / contentParts.length);

  return (
    <AbsoluteFill style={{ backgroundColor: '#0a0a0a' }}>
      {/* Title Section */}
      <div
        style={{
          position: 'absolute',
          top: '15%',
          left: '10%',
          right: '10%',
          opacity: titleOpacity,
          transform: `scale(${titleScale})`
        }}
      >
        <h1 style={{
          fontSize: 72,
          fontWeight: 'bold',
          color: '#ffffff',
          textAlign: 'center',
          margin: 0
        }}>
          {title}
        </h1>
      </div>

      {/* Content Sections */}
      {contentParts.map((part, index) => {
        const startFrame = 60 + (index * framesPerSection);
        const endFrame = startFrame + framesPerSection;
        
        const opacity = interpolate(
          frame,
          [startFrame, startFrame + 20, endFrame - 20, endFrame],
          [0, 1, 1, 0],
          { extrapolateLeft: 'clamp', extrapolateRight: 'clamp' }
        );

        return (
          <div
            key={index}
            style={{
              position: 'absolute',
              top: '40%',
              left: '10%',
              right: '10%',
              opacity,
              transform: 'translateY(-50%)'
            }}
          >
            <p style={{
              fontSize: 42,
              color: '#e0e0e0',
              lineHeight: 1.6,
              textAlign: 'center'
            }}>
              {part}
            </p>
          </div>
        );
      })}
    </AbsoluteFill>
  );
};
```

## API Routes

### Research Endpoint

```typescript
// app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { performResearch } from '@/lib/scraper/research';

export async function POST(request: NextRequest) {
  try {
    const { keyword, sources } = await request.json();

    if (!keyword) {
      return NextResponse.json(
        { error: 'Keyword is required' },
        { status: 400 }
      );
    }

    const results = await performResearch(keyword, sources);

    return NextResponse.json({
      success: true,
      data: results
    });
  } catch (error) {
    console.error('Research error:', error);
    return NextResponse.json(
      { error: 'Failed to perform research' },
      { status: 500 }
    );
  }
}
```

### Content Generation Endpoint

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { ContentGenerator } from '@/lib/ai/content-generator';

export async function POST(request: NextRequest) {
  try {
    const contentRequest = await request.json();
    
    const generator = new ContentGenerator();
    
    // Use Claude by default, fallback to OpenAI
    const content = process.env.ANTHROPIC_API_KEY
      ? await generator.generateWithClaude(contentRequest)
      : await generator.generateWithOpenAI(contentRequest);

    return NextResponse.json({
      success: true,
      data: content
    });
  } catch (error) {
    console.error('Generation error:', error);
    return NextResponse.json(
      { error: 'Failed to generate content' },
      { status: 500 }
    );
  }
}
```

### Video Rendering Endpoint

```typescript
// app/api/render/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { VideoRenderer } from '@/lib/remotion/video-renderer';

export async function POST(request: NextRequest) {
  try {
    const videoConfig = await request.json();
    
    const renderer = new VideoRenderer();
    const output = await renderer.renderContentVideo(videoConfig);

    return NextResponse.json({
      success: true,
      data: output
    });
  } catch (error) {
    console.error('Render error:', error);
    return NextResponse.json(
      { error: 'Failed to render video' },
      { status: 500 }
    );
  }
}
```

## Complete Workflow Example

```typescript
// app/actions/content-pipeline.ts
'use server';

import { performResearch } from '@/lib/scraper/research';
import { ContentGenerator } from '@/lib/ai/content-generator';
import { VideoRenderer } from '@/lib/remotion/video-renderer';

interface PipelineResult {
  research: ResearchResult[];
  content: GeneratedContent;
  video?: VideoOutput;
}

export async function runContentPipeline(
  keyword: string,
  options: {
    format: 'toplist' | 'pov' | 'case-study' | 'how-to';
    language: 'en' | 'vi' | 'both';
    tone: 'expert' | 'friendly' | 'humorous';
    generateVideo?: boolean;
    videoFormat?: 'reels' | 'tiktok' | 'shorts';
  }
): Promise<PipelineResult> {
  // Step 1: Research
  console.log('🔍 Starting research...');
  const research = await performResearch(keyword);
  
  // Step 2: Generate Content
  console.log('✍️ Generating content...');
  const generator = new ContentGenerator();
  const content = await generator.generateWithClaude({
    keyword,
    format: options.format,
    language: options.language,
    tone: options.tone,
    researchData: research
  });

  const result: PipelineResult = {
    research,
    content
  };

  // Step 3: Render Video (optional)
  if (options.generateVideo) {
    console.log('🎬 Rendering video...');
    const renderer = new VideoRenderer();
    const video = await renderer.renderContentVideo({
      content,
      format: options.videoFormat || 'reels',
      style: 'dynamic'
    });
    result.video = video;
  }

  console.log('✅ Pipeline complete!');
  return result;
}
```

## Running the Development Server

```bash
# Start Next.js dev server
npm run dev

# Start Remotion studio for video editing
npm run remotion

# Build for production
npm run build

# Start production server
npm start
```

## Common Patterns

### Batch Content Generation

```typescript
async function batchGenerate(keywords: string[]) {
  const results = await Promise.all(
    keywords.map(keyword => 
      runContentPipeline(keyword, {
        format: 'toplist',
        language: 'both',
        tone: 'expert',
        generateVideo: true
      })
    )
  );
  
  return results;
}
```

### Scheduled Content Generation

```typescript
// Use with cron job or scheduling service
import cron from 'node-cron';

// Run every day at 9 AM
cron.schedule('0 9 * * *', async () => {
  const trendingTopics = await getTrendingTopics();
  
  for (const topic of trendingTopics) {
    await runContentPipeline(topic, {
      format: 'pov',
      language: 'both',
      tone: 'friendly',
      generateVideo: true
    });
  }
});
```

## Troubleshooting

### API Key Issues

```typescript
// Verify API keys are loaded
if (!process.env.ANTHROPIC_API_KEY) {
  throw new Error('ANTHROPIC_API_KEY is not set');
}

if (!process.env.OPENAI_API_KEY) {
  console.warn('OPENAI_API_KEY is not set - Claude will be used exclusively');
}
```

### Rate Limiting

```typescript
// Add rate limiting for API calls
import { rateLimit } from '@/lib/utils/rate-limit';

const limiter = rateLimit({
  interval: 60 * 1000, // 1 minute
  uniqueTokenPerInterval: 500
});

export async function POST(request: NextRequest) {
  try {
    await limiter.check(request, 10); // 10 requests per minute
    // ... rest of handler
  } catch {
    return NextResponse.json(
      { error: 'Rate limit exceeded' },
      { status: 429 }
    );
  }
}
```

### Remotion Memory Issues

```bash
# Increase Node memory for video rendering
NODE_OPTIONS="--max-old-space-size=8192" npm run render
```

### Scraping Failures

```typescript
// Implement retry logic
async function scrapeWithRetry(url: string, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await scrape(url);
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      await new Promise(resolve => setTimeout(resolve, 1000 * (i + 1)));
    }
  }
}
```

## Performance Optimization

```typescript
// Cache research results
import { cache } from 'react';

export const getCachedResearch = cache(async (keyword: string) => {
  return await performResearch(keyword);
});

// Parallel processing
async function optimizedPipeline(keyword: string) {
  // Start research and video template preparation in parallel
  const [research, template] = await Promise.all([
    performResearch(keyword),
    prepareVideoTemplate()
  ]);
  
  // Then generate content
  const content = await generateContent({ keyword, researchData: research });
  
  // Finally render video
  const video = await renderVideo({ content, template });
  
  return { research, content, video };
}
```
