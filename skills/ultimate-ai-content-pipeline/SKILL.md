---
name: ultimate-ai-content-pipeline
description: Automated content creation pipeline from research to video generation using Claude/OpenAI and Remotion
triggers:
  - how do I generate automated content with AI research
  - set up content pipeline with video generation
  - create automated marketing content from research to video
  - use ultimate AI content pipeline for content automation
  - integrate Claude and OpenAI for content generation
  - automate content research and script writing
  - generate videos from AI written content automatically
  - build content automation workflow with Remotion
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the Ultimate AI Content Pipeline, a comprehensive TypeScript-based system that automates content creation from research to video generation. The pipeline crawls news sources, generates multi-format content using Claude/OpenAI, and renders videos using Remotion.

## What It Does

The Ultimate AI Content Pipeline automates the entire content creation workflow:

1. **Auto-Research**: Crawls real-time data from sources like TechCrunch, a16z, Twitter/X, LinkedIn
2. **AI Content Generation**: Uses Claude 3/OpenAI to create content in multiple formats (Toplist, POV, Case Study, How-to)
3. **Multi-language Support**: Generates content in both English and Vietnamese
4. **Video Rendering**: Automatically converts written content to videos using Remotion
5. **Platform Optimization**: Exports videos optimized for Reels, TikTok, Shorts

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

Create a `.env.local` file in the root directory with the following variables:

```bash
# AI APIs
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# RapidAPI (for news crawling)
RAPIDAPI_KEY=your_rapidapi_key

# Remotion (for video rendering)
REMOTION_LICENSE_KEY=your_remotion_license_key

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Application
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── crawler/     # News crawling logic
│   │   ├── content/     # Content generation
│   │   └── video/       # Remotion video rendering
│   ├── types/           # TypeScript types
│   └── utils/           # Utility functions
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Running the Application

```bash
# Development mode
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Run Remotion studio (for video editing)
npm run remotion
```

## Core API Usage

### 1. Content Research

```typescript
import { ResearchService } from '@/lib/crawler/research-service';

const researchService = new ResearchService({
  rapidApiKey: process.env.RAPIDAPI_KEY!
});

// Crawl news from multiple sources
const researchData = await researchService.crawlNews({
  keyword: 'AI automation',
  sources: ['techcrunch', 'a16z', 'twitter'],
  timeframe: '24h',
  maxResults: 50
});

// Extract insights
const insights = await researchService.extractInsights(researchData);
```

### 2. AI Content Generation with Claude

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY
});

async function generateContent(research: ResearchData, format: ContentFormat) {
  const prompt = `
    Based on this research data: ${JSON.stringify(research)}
    
    Create a ${format} article in both English and Vietnamese.
    Tone: Professional yet engaging
    Length: 1500-2000 words
    Include: Data-backed insights, real examples, actionable takeaways
  `;

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [{
      role: 'user',
      content: prompt
    }]
  });

  return message.content;
}
```

### 3. AI Content Generation with OpenAI

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY
});

async function generateContentOpenAI(research: ResearchData, format: string) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: 'You are an expert content creator specializing in marketing and tech content.'
      },
      {
        role: 'user',
        content: `Create a ${format} article based on: ${JSON.stringify(research)}`
      }
    ],
    temperature: 0.7,
    max_tokens: 3000
  });

  return completion.choices[0].message.content;
}
```

### 4. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

async function generateVideo(content: GeneratedContent) {
  // Bundle Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.join(process.cwd(), 'remotion/index.ts'),
    webpackOverride: (config) => config
  });

  // Select composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title: content.title,
      body: content.body,
      insights: content.insights,
      style: 'modern'
    }
  });

  // Render video
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: `out/${content.id}.mp4`,
    inputProps: composition.inputProps
  });

  return `out/${content.id}.mp4`;
}
```

## Content Format Templates

### Toplist Format

```typescript
interface ToplistContent {
  title: string;
  introduction: string;
  items: Array<{
    rank: number;
    title: string;
    description: string;
    stats?: Record<string, string>;
    source?: string;
  }>;
  conclusion: string;
}

async function generateToplist(keyword: string, research: ResearchData) {
  const prompt = `
    Create a toplist article about "${keyword}" using this research data.
    Format: Top 10 list with detailed explanations
    Include: Statistics, real examples, sources
  `;
  
  // Use AI to generate
  const content = await generateContent(research, 'toplist');
  return parseToplistContent(content);
}
```

### POV (Point of View) Format

```typescript
interface POVContent {
  title: string;
  perspective: string;
  mainArgument: string;
  supportingPoints: Array<{
    point: string;
    evidence: string;
    example?: string;
  }>;
  counterarguments?: string[];
  conclusion: string;
}

async function generatePOV(topic: string, research: ResearchData) {
  const prompt = `
    Write a POV article on "${topic}" based on this research.
    Take a clear stance backed by data and real-world examples.
    Address potential counterarguments.
  `;
  
  return await generateContent(research, 'pov');
}
```

### Case Study Format

```typescript
interface CaseStudyContent {
  title: string;
  background: string;
  challenge: string;
  solution: string;
  implementation: string[];
  results: {
    metric: string;
    before: string;
    after: string;
    improvement: string;
  }[];
  keyTakeaways: string[];
}
```

## Complete Workflow Example

```typescript
import { ContentPipeline } from '@/lib/content/pipeline';

async function runContentPipeline(keyword: string) {
  const pipeline = new ContentPipeline({
    aiProvider: 'claude', // or 'openai'
    language: ['en', 'vi'],
    formats: ['toplist', 'pov', 'case-study'],
    generateVideo: true
  });

  try {
    // Step 1: Research
    console.log('🔍 Starting research...');
    const research = await pipeline.research(keyword);

    // Step 2: Generate content
    console.log('✍️ Generating content...');
    const content = await pipeline.generateContent(research);

    // Step 3: Render video
    if (pipeline.config.generateVideo) {
      console.log('🎬 Rendering video...');
      const videoPath = await pipeline.renderVideo(content);
      console.log(`Video saved to: ${videoPath}`);
    }

    // Step 4: Save and schedule
    console.log('💾 Saving content...');
    await pipeline.saveContent(content);

    return {
      success: true,
      content,
      videoPath: content.videoPath
    };
  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}

// Usage
runContentPipeline('AI automation tools 2026')
  .then(result => console.log('✅ Pipeline complete:', result))
  .catch(error => console.error('❌ Pipeline failed:', error));
```

## Next.js API Routes

### Research Endpoint

```typescript
// app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { ResearchService } from '@/lib/crawler/research-service';

export async function POST(request: NextRequest) {
  const { keyword, sources, timeframe } = await request.json();

  const researchService = new ResearchService({
    rapidApiKey: process.env.RAPIDAPI_KEY!
  });

  const data = await researchService.crawlNews({
    keyword,
    sources,
    timeframe
  });

  return NextResponse.json({ data });
}
```

### Content Generation Endpoint

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { generateContent } from '@/lib/ai/content-generator';

export async function POST(request: NextRequest) {
  const { research, format, language } = await request.json();

  const content = await generateContent(research, format, {
    language,
    tone: 'professional',
    length: 'medium'
  });

  return NextResponse.json({ content });
}
```

### Video Rendering Endpoint

```typescript
// app/api/render-video/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { renderVideo } from '@/lib/video/renderer';

export async function POST(request: NextRequest) {
  const { contentId, content } = await request.json();

  const videoPath = await renderVideo({
    id: contentId,
    ...content
  });

  return NextResponse.json({ videoPath });
}
```

## Remotion Video Templates

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig } from 'remotion';
import { interpolate } from 'remotion';

interface ContentVideoProps {
  title: string;
  body: string;
  insights: string[];
  style: 'modern' | 'minimal' | 'bold';
}

export const ContentVideo: React.FC<ContentVideoProps> = ({
  title,
  body,
  insights,
  style
}) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  const opacity = interpolate(frame, [0, 30], [0, 1], {
    extrapolateRight: 'clamp'
  });

  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a1a' }}>
      <div style={{ 
        opacity,
        padding: '60px',
        color: 'white',
        fontFamily: 'Inter, sans-serif'
      }}>
        <h1 style={{ fontSize: '48px', marginBottom: '30px' }}>
          {title}
        </h1>
        <div style={{ fontSize: '24px', lineHeight: '1.6' }}>
          {body.substring(0, 200)}...
        </div>
        <ul style={{ marginTop: '40px' }}>
          {insights.map((insight, i) => (
            <li key={i} style={{ marginBottom: '20px', fontSize: '20px' }}>
              {insight}
            </li>
          ))}
        </ul>
      </div>
    </AbsoluteFill>
  );
};
```

## Configuration Options

```typescript
// lib/config/pipeline-config.ts
export interface PipelineConfig {
  // AI Settings
  aiProvider: 'claude' | 'openai' | 'both';
  model?: string;
  temperature?: number;
  maxTokens?: number;

  // Content Settings
  language: string[];
  formats: ContentFormat[];
  tone: 'professional' | 'casual' | 'expert' | 'friendly';
  length: 'short' | 'medium' | 'long';

  // Research Settings
  sources: string[];
  timeframe: '24h' | '7d' | '30d';
  maxResults: number;

  // Video Settings
  generateVideo: boolean;
  videoFormat: 'mp4' | 'webm';
  videoResolution: '1080p' | '720p' | '4k';
  videoDuration: number;

  // Output Settings
  outputDir: string;
  autoSchedule: boolean;
}

export const defaultConfig: PipelineConfig = {
  aiProvider: 'claude',
  temperature: 0.7,
  maxTokens: 4096,
  language: ['en', 'vi'],
  formats: ['toplist', 'pov'],
  tone: 'professional',
  length: 'medium',
  sources: ['techcrunch', 'a16z', 'twitter'],
  timeframe: '24h',
  maxResults: 50,
  generateVideo: true,
  videoFormat: 'mp4',
  videoResolution: '1080p',
  videoDuration: 60,
  outputDir: './output',
  autoSchedule: false
};
```

## Common Patterns

### Batch Content Generation

```typescript
async function batchGenerateContent(keywords: string[]) {
  const results = await Promise.allSettled(
    keywords.map(keyword => runContentPipeline(keyword))
  );

  const successful = results.filter(r => r.status === 'fulfilled');
  const failed = results.filter(r => r.status === 'rejected');

  console.log(`✅ Success: ${successful.length}`);
  console.log(`❌ Failed: ${failed.length}`);

  return { successful, failed };
}
```

### Content Scheduling

```typescript
import { ContentScheduler } from '@/lib/content/scheduler';

const scheduler = new ContentScheduler();

// Schedule content for posting
await scheduler.schedule({
  content: generatedContent,
  platforms: ['facebook', 'linkedin', 'twitter'],
  publishAt: new Date('2026-06-10T10:00:00Z'),
  timezone: 'Asia/Ho_Chi_Minh'
});
```

### Error Handling

```typescript
async function safeContentGeneration(keyword: string) {
  try {
    return await runContentPipeline(keyword);
  } catch (error) {
    if (error.message.includes('rate_limit')) {
      console.log('Rate limit hit, retrying in 60s...');
      await new Promise(resolve => setTimeout(resolve, 60000));
      return await runContentPipeline(keyword);
    }
    
    if (error.message.includes('insufficient_credits')) {
      console.error('❌ Insufficient API credits');
      // Notify admin or switch to backup provider
    }
    
    throw error;
  }
}
```

## Troubleshooting

### API Rate Limits

If you encounter rate limits:

```typescript
import pLimit from 'p-limit';

const limit = pLimit(3); // Max 3 concurrent requests

const results = await Promise.all(
  keywords.map(keyword => 
    limit(() => runContentPipeline(keyword))
  )
);
```

### Video Rendering Timeout

For long videos, increase timeout:

```typescript
await renderMedia({
  composition,
  serveUrl: bundleLocation,
  codec: 'h264',
  outputLocation: outputPath,
  timeoutInMilliseconds: 300000 // 5 minutes
});
```

### Memory Issues with Large Research Data

```typescript
// Process research data in chunks
function chunkArray<T>(array: T[], size: number): T[][] {
  return Array.from(
    { length: Math.ceil(array.length / size) },
    (_, i) => array.slice(i * size, i * size + size)
  );
}

const chunks = chunkArray(researchData, 10);
for (const chunk of chunks) {
  await processChunk(chunk);
}
```

### Claude/OpenAI Connection Issues

```typescript
import axios from 'axios';

async function retryWithBackoff<T>(
  fn: () => Promise<T>,
  maxRetries = 3
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      const delay = Math.pow(2, i) * 1000;
      console.log(`Retry ${i + 1} after ${delay}ms`);
      await new Promise(resolve => setTimeout(resolve, delay));
    }
  }
  throw new Error('Max retries exceeded');
}
```

This skill provides comprehensive coverage of the Ultimate AI Content Pipeline system for automated content creation and video generation workflows.
