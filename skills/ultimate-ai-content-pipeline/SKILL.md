---
name: ultimate-ai-content-pipeline
description: Automated content creation pipeline from research to script to video generation using Claude, OpenAI, and Remotion
triggers:
  - automate content creation with AI
  - generate video content from text automatically
  - research and create marketing content pipeline
  - build AI-powered content automation system
  - create videos from blog posts with Remotion
  - set up automated content research and writing
  - implement end-to-end content generation workflow
  - scrape news and generate content automatically
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

Ultimate AI Content Pipeline is a comprehensive TypeScript-based content automation system that handles the entire content creation workflow: from researching trending topics across platforms like TechCrunch, Twitter, and LinkedIn, to generating multilingual written content, to automatically rendering videos using Remotion. It leverages Claude 3, OpenAI, and various APIs to create a fully automated content production pipeline.

## Installation

### Prerequisites

- Node.js 18+ and npm/yarn
- API keys for: OpenAI, Anthropic (Claude), RapidAPI
- (Optional) Remotion Lambda setup for video rendering

### Setup

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
yarn install

# Create environment file
cp .env.example .env
```

### Environment Configuration

Create a `.env` file with the following variables:

```bash
# AI Provider Keys
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# Research & Scraping
RAPIDAPI_KEY=your_rapidapi_key

# Remotion Video Rendering
REMOTION_AWS_ACCESS_KEY_ID=your_aws_access_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret_key
REMOTION_LAMBDA_REGION=us-east-1

# Database (if applicable)
DATABASE_URL=postgresql://user:pass@localhost:5432/content_pipeline

# Next.js
NEXT_PUBLIC_API_URL=http://localhost:3000
```

## Core Components

### 1. Research Engine (Auto-Scan)

The research module automatically crawls and analyzes recent content from multiple sources:

```typescript
// lib/research/scanner.ts
import { searchTechCrunch, searchTwitter, searchLinkedIn } from './sources';

interface ResearchResult {
  title: string;
  url: string;
  summary: string;
  publishedAt: Date;
  source: string;
  insights: string[];
}

export async function conductResearch(
  keyword: string,
  timeframe: '24h' | '7d' = '24h'
): Promise<ResearchResult[]> {
  const sources = await Promise.all([
    searchTechCrunch(keyword, timeframe),
    searchTwitter(keyword, timeframe),
    searchLinkedIn(keyword, timeframe)
  ]);

  const results = sources.flat();
  
  // Filter and deduplicate
  return deduplicateAndRank(results);
}

// Usage example
const research = await conductResearch('AI marketing automation', '24h');
console.log(`Found ${research.length} relevant articles`);
```

### 2. Content Generation with AI

Generate content in multiple formats using Claude or OpenAI:

```typescript
// lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

type ContentFormat = 'toplist' | 'pov' | 'case-study' | 'how-to';
type Language = 'en' | 'vi';
type Tone = 'expert' | 'friendly' | 'humorous';

interface ContentRequest {
  keyword: string;
  format: ContentFormat;
  language: Language;
  tone: Tone;
  researchData: ResearchResult[];
}

export async function generateContent(
  request: ContentRequest,
  provider: 'claude' | 'openai' = 'claude'
): Promise<string> {
  const prompt = buildPrompt(request);

  if (provider === 'claude') {
    const anthropic = new Anthropic({
      apiKey: process.env.ANTHROPIC_API_KEY,
    });

    const message = await anthropic.messages.create({
      model: 'claude-3-5-sonnet-20241022',
      max_tokens: 4000,
      messages: [{
        role: 'user',
        content: prompt
      }],
    });

    return message.content[0].type === 'text' 
      ? message.content[0].text 
      : '';
  } else {
    const openai = new OpenAI({
      apiKey: process.env.OPENAI_API_KEY,
    });

    const completion = await openai.chat.completions.create({
      model: 'gpt-4-turbo-preview',
      messages: [{ role: 'user', content: prompt }],
      max_tokens: 4000,
    });

    return completion.choices[0].message.content || '';
  }
}

function buildPrompt(request: ContentRequest): string {
  const researchContext = request.researchData
    .map(r => `- ${r.title}: ${r.summary}`)
    .join('\n');

  return `Create a ${request.format} article about "${request.keyword}" in ${request.language} with a ${request.tone} tone.

Recent research data:
${researchContext}

Requirements:
- Include data-backed insights
- Use the latest trends from the past 24 hours
- Format: ${request.format}
- Length: 1500-2000 words
- Include actionable takeaways`;
}
```

### 3. Video Generation with Remotion

Convert written content into video format:

```typescript
// lib/video/render.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import { getCompositions } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  content: string;
  title: string;
  format: 'reels' | 'tiktok' | 'youtube-shorts';
}

export async function renderContentVideo(
  config: VideoConfig
): Promise<string> {
  // Define composition dimensions based on format
  const dimensions = {
    'reels': { width: 1080, height: 1920 },
    'tiktok': { width: 1080, height: 1920 },
    'youtube-shorts': { width: 1080, height: 1920 },
  };

  const { width, height } = dimensions[config.format];

  // Bundle Remotion composition
  const bundleLocation = await bundle({
    entryPoint: path.join(process.cwd(), 'remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  const comps = await getCompositions(bundleLocation);
  const composition = comps.find((c) => c.id === 'ContentVideo');

  if (!composition) {
    throw new Error('Composition not found');
  }

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
      content: config.content,
      title: config.title,
    },
  });

  return outputLocation;
}

// Usage
const videoPath = await renderContentVideo({
  content: generatedArticle,
  title: 'Top 5 AI Marketing Tools',
  format: 'reels'
});
```

### 4. Remotion Composition Example

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig, Sequence } from 'remotion';
import React from 'react';

interface ContentVideoProps {
  title: string;
  content: string;
}

export const ContentVideo: React.FC<ContentVideoProps> = ({ title, content }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  // Parse content into sections
  const sections = content.split('\n\n').slice(0, 5);

  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a1a' }}>
      <Sequence from={0} durationInFrames={fps * 2}>
        <TitleSlide title={title} frame={frame} />
      </Sequence>
      
      {sections.map((section, i) => (
        <Sequence
          key={i}
          from={(i + 1) * fps * 2}
          durationInFrames={fps * 3}
        >
          <ContentSlide text={section} frame={frame} />
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};

const TitleSlide: React.FC<{ title: string; frame: number }> = ({ title, frame }) => {
  const opacity = Math.min(1, frame / 30);
  
  return (
    <AbsoluteFill style={{
      justifyContent: 'center',
      alignItems: 'center',
      opacity,
    }}>
      <h1 style={{ fontSize: 80, color: 'white', textAlign: 'center' }}>
        {title}
      </h1>
    </AbsoluteFill>
  );
};

const ContentSlide: React.FC<{ text: string; frame: number }> = ({ text }) => {
  return (
    <AbsoluteFill style={{
      justifyContent: 'center',
      alignItems: 'center',
      padding: 80,
    }}>
      <p style={{ fontSize: 48, color: 'white', lineHeight: 1.5 }}>
        {text}
      </p>
    </AbsoluteFill>
  );
};
```

## Complete Pipeline Example

```typescript
// app/api/generate-content/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { conductResearch } from '@/lib/research/scanner';
import { generateContent } from '@/lib/ai/content-generator';
import { renderContentVideo } from '@/lib/video/render';

export async function POST(req: NextRequest) {
  try {
    const { keyword, format, language, tone, includeVideo } = await req.json();

    // Step 1: Research
    console.log('🔍 Conducting research...');
    const research = await conductResearch(keyword, '24h');

    // Step 2: Generate Content
    console.log('✍️ Generating content...');
    const content = await generateContent({
      keyword,
      format,
      language,
      tone,
      researchData: research,
    }, 'claude');

    // Step 3: Render Video (optional)
    let videoUrl = null;
    if (includeVideo) {
      console.log('🎬 Rendering video...');
      const videoPath = await renderContentVideo({
        content,
        title: keyword,
        format: 'reels',
      });
      videoUrl = `/videos/${path.basename(videoPath)}`;
    }

    return NextResponse.json({
      success: true,
      data: {
        content,
        videoUrl,
        researchSources: research.length,
      },
    });

  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { error: 'Content generation failed' },
      { status: 500 }
    );
  }
}
```

## Running the Application

### Development Mode

```bash
# Start Next.js dev server
npm run dev

# Start Remotion studio (for video editing)
npm run remotion
```

### Production Build

```bash
# Build Next.js application
npm run build

# Start production server
npm start
```

## Common Patterns

### Batch Content Generation

```typescript
// scripts/batch-generate.ts
import { conductResearch } from './lib/research/scanner';
import { generateContent } from './lib/ai/content-generator';

const keywords = [
  'AI marketing automation',
  'Content creation tools',
  'Video marketing trends'
];

async function batchGenerate() {
  for (const keyword of keywords) {
    const research = await conductResearch(keyword, '24h');
    
    const content = await generateContent({
      keyword,
      format: 'toplist',
      language: 'en',
      tone: 'expert',
      researchData: research,
    });

    // Save to database or file
    await saveContent(keyword, content);
    
    // Rate limiting
    await new Promise(resolve => setTimeout(resolve, 5000));
  }
}

batchGenerate().catch(console.error);
```

### Multilingual Content

```typescript
async function generateMultilingual(keyword: string, research: ResearchResult[]) {
  const [englishContent, vietnameseContent] = await Promise.all([
    generateContent({
      keyword,
      format: 'how-to',
      language: 'en',
      tone: 'friendly',
      researchData: research,
    }),
    generateContent({
      keyword,
      format: 'how-to',
      language: 'vi',
      tone: 'friendly',
      researchData: research,
    }),
  ]);

  return { en: englishContent, vi: vietnameseContent };
}
```

## Troubleshooting

### API Rate Limits

```typescript
// lib/utils/rate-limiter.ts
export class RateLimiter {
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
    const fn = this.queue.shift();
    
    if (fn) {
      await fn();
      await new Promise(resolve => setTimeout(resolve, 1000)); // 1s delay
    }
    
    this.processing = false;
    this.process();
  }
}

// Usage
const limiter = new RateLimiter();
const result = await limiter.add(() => generateContent(request));
```

### Video Rendering Timeouts

```typescript
// Increase timeout for longer videos
await renderMedia({
  composition,
  serveUrl: bundleLocation,
  codec: 'h264',
  outputLocation,
  inputProps,
  timeoutInMilliseconds: 300000, // 5 minutes
});
```

### Memory Issues with Large Content

```typescript
// Process content in chunks
function chunkContent(content: string, maxLength: number = 2000): string[] {
  const paragraphs = content.split('\n\n');
  const chunks: string[] = [];
  let currentChunk = '';

  for (const paragraph of paragraphs) {
    if ((currentChunk + paragraph).length > maxLength) {
      chunks.push(currentChunk);
      currentChunk = paragraph;
    } else {
      currentChunk += '\n\n' + paragraph;
    }
  }
  
  if (currentChunk) chunks.push(currentChunk);
  return chunks;
}
```

## Best Practices

1. **Cache Research Results**: Store scraped data to avoid redundant API calls
2. **Queue Video Rendering**: Use a job queue (Bull, BullMQ) for video processing
3. **Validate AI Output**: Check generated content for quality before rendering
4. **Monitor Costs**: Track API usage across OpenAI, Claude, and Remotion
5. **Error Handling**: Implement retry logic for external API failures
