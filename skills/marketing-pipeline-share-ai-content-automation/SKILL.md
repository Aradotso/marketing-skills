---
name: marketing-pipeline-share-ai-content-automation
description: Automated AI content pipeline for research, scripting, publishing and video generation using Claude, OpenAI and Remotion
triggers:
  - how do I automate content creation with AI research and video generation
  - set up an AI marketing content pipeline with auto-publishing
  - create automated content workflow from research to video
  - build AI-powered content automation system with Claude and OpenAI
  - generate marketing content automatically with research crawling
  - automate social media content creation and video rendering
  - implement end-to-end AI content pipeline for marketing
  - set up content automation with research scanning and video export
---

# Marketing Pipeline Share - AI Content Automation Skill

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to help developers implement an end-to-end automated content creation pipeline. The system crawls research data from news sources, generates multi-format content using Claude/OpenAI, and automatically renders videos using Remotion — all from a single keyword input.

## What This Project Does

**Marketing Pipeline Share** is a comprehensive TypeScript-based content automation system that:

- **Auto-scans research**: Crawls fresh data from TechCrunch, a16z, Twitter/X, LinkedIn within the last 24 hours
- **AI content generation**: Creates articles in multiple formats (toplist, POV, case study, how-to) using Claude 3 or OpenAI
- **Multi-language support**: Generates content in both English and Vietnamese with customizable tone
- **Video rendering**: Automatically creates infographics and short-form videos from content using Remotion
- **Platform optimization**: Exports videos optimized for Reels, TikTok, and YouTube Shorts

## Installation

### Prerequisites

```bash
# Node.js 18+ required
node --version

# Package manager (yarn recommended)
yarn --version
```

### Setup Steps

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
yarn install
# or
npm install

# Copy environment variables
cp .env.example .env
```

### Environment Configuration

Create a `.env` file with the following variables:

```bash
# AI Providers
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Next.js
NEXT_PUBLIC_API_URL=http://localhost:3000
```

## Key Architecture

### Project Structure

```
marketing-pineline-share/
├── app/                    # Next.js app directory
│   ├── api/               # API routes
│   ├── components/        # React components
│   └── lib/               # Utility functions
├── services/              # Core business logic
│   ├── research/          # Content crawling
│   ├── ai/                # AI generation
│   └── video/             # Remotion rendering
├── remotion/              # Video templates
└── types/                 # TypeScript definitions
```

## Core API Usage

### 1. Research & Content Crawling

```typescript
// services/research/crawler.ts
import axios from 'axios';

interface ResearchSource {
  url: string;
  title: string;
  publishedAt: string;
  content: string;
}

export async function crawlRecentNews(
  keyword: string,
  sources: string[] = ['techcrunch', 'a16z']
): Promise<ResearchSource[]> {
  const results: ResearchSource[] = [];
  
  for (const source of sources) {
    try {
      const response = await axios.get(`https://api.rapidapi.com/news/${source}`, {
        headers: {
          'X-RapidAPI-Key': process.env.RAPIDAPI_KEY,
        },
        params: {
          q: keyword,
          from: new Date(Date.now() - 24 * 60 * 60 * 1000).toISOString(),
        },
      });
      
      results.push(...response.data.articles);
    } catch (error) {
      console.error(`Failed to crawl ${source}:`, error);
    }
  }
  
  return results;
}
```

### 2. AI Content Generation with Claude

```typescript
// services/ai/claude-generator.ts
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

interface ContentGenerationParams {
  keyword: string;
  research: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'professional' | 'friendly' | 'humorous';
}

export async function generateContent(
  params: ContentGenerationParams
): Promise<string> {
  const systemPrompt = buildSystemPrompt(params.format, params.tone);
  const userPrompt = buildUserPrompt(params.keyword, params.research, params.language);
  
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    temperature: 0.7,
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

function buildSystemPrompt(format: string, tone: string): string {
  const formats = {
    'toplist': 'You are an expert content writer specializing in list-based articles.',
    'pov': 'You are a thought leader sharing unique perspectives on industry trends.',
    'case-study': 'You are a data-driven analyst creating detailed case studies.',
    'how-to': 'You are an educational content creator writing step-by-step guides.',
  };
  
  const tones = {
    'professional': 'Maintain a professional, authoritative tone.',
    'friendly': 'Use a conversational, approachable tone.',
    'humorous': 'Inject appropriate humor while staying informative.',
  };
  
  return `${formats[format]} ${tones[tone]} Always cite sources and include data-backed insights.`;
}

function buildUserPrompt(keyword: string, research: string, language: string): string {
  const languageInstruction = language === 'vi' 
    ? 'Write the content in Vietnamese.' 
    : 'Write the content in English.';
  
  return `
Create compelling content about: ${keyword}

Based on this recent research:
${research}

${languageInstruction}

Include:
- Engaging headline
- Data-backed insights
- Clear structure with subheadings
- Actionable takeaways
`;
}
```

### 3. OpenAI Alternative

```typescript
// services/ai/openai-generator.ts
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

export async function generateContentWithOpenAI(
  params: ContentGenerationParams
): Promise<string> {
  const systemPrompt = buildSystemPrompt(params.format, params.tone);
  const userPrompt = buildUserPrompt(params.keyword, params.research, params.language);
  
  const completion = await openai.chat.completions.create({
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
```

### 4. Video Generation with Remotion

```typescript
// services/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoRenderParams {
  content: string;
  title: string;
  platform: 'reels' | 'tiktok' | 'shorts';
}

const platformConfigs = {
  reels: { width: 1080, height: 1920, fps: 30 },
  tiktok: { width: 1080, height: 1920, fps: 30 },
  shorts: { width: 1080, height: 1920, fps: 30 },
};

export async function renderContentVideo(
  params: VideoRenderParams
): Promise<string> {
  const config = platformConfigs[params.platform];
  
  // Bundle Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: (config) => config,
  });
  
  // Get composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title: params.title,
      content: params.content,
    },
  });
  
  // Output path
  const outputPath = path.join(
    process.cwd(),
    'output',
    `${Date.now()}-${params.platform}.mp4`
  );
  
  // Render video
  await renderMedia({
    composition: {
      ...composition,
      width: config.width,
      height: config.height,
      fps: config.fps,
    },
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: {
      title: params.title,
      content: params.content,
    },
  });
  
  return outputPath;
}
```

### 5. Remotion Video Component

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig, interpolate } from 'remotion';
import React from 'react';

interface ContentVideoProps {
  title: string;
  content: string;
}

export const ContentVideo: React.FC<ContentVideoProps> = ({ title, content }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  
  const opacity = interpolate(frame, [0, 30], [0, 1], {
    extrapolateRight: 'clamp',
  });
  
  const scale = interpolate(frame, [0, 30], [0.8, 1], {
    extrapolateRight: 'clamp',
  });
  
  // Parse content into key points
  const keyPoints = content.split('\n')
    .filter(line => line.trim().startsWith('-'))
    .map(line => line.replace('-', '').trim())
    .slice(0, 5);
  
  return (
    <AbsoluteFill
      style={{
        backgroundColor: '#1a1a1a',
        justifyContent: 'center',
        alignItems: 'center',
        padding: 60,
      }}
    >
      <div
        style={{
          opacity,
          transform: `scale(${scale})`,
          width: '100%',
        }}
      >
        <h1
          style={{
            color: 'white',
            fontSize: 72,
            fontWeight: 'bold',
            marginBottom: 40,
            textAlign: 'center',
          }}
        >
          {title}
        </h1>
        
        <div style={{ color: 'white', fontSize: 36 }}>
          {keyPoints.map((point, index) => {
            const pointFrame = frame - (index * fps);
            const pointOpacity = interpolate(pointFrame, [0, 15], [0, 1], {
              extrapolateRight: 'clamp',
              extrapolateLeft: 'clamp',
            });
            
            return (
              <div
                key={index}
                style={{
                  opacity: pointOpacity,
                  marginBottom: 30,
                  display: 'flex',
                  alignItems: 'center',
                }}
              >
                <span style={{ color: '#00ff88', marginRight: 20, fontSize: 48 }}>
                  •
                </span>
                <span>{point}</span>
              </div>
            );
          })}
        </div>
      </div>
    </AbsoluteFill>
  );
};
```

## Complete Pipeline Integration

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { crawlRecentNews } from '@/services/research/crawler';
import { generateContent } from '@/services/ai/claude-generator';
import { renderContentVideo } from '@/services/video/renderer';

export async function POST(request: NextRequest) {
  try {
    const { keyword, format, language, tone, platforms } = await request.json();
    
    // Step 1: Research
    console.log('Crawling research data...');
    const research = await crawlRecentNews(keyword);
    const researchText = research
      .map(r => `${r.title}\n${r.content}`)
      .join('\n\n');
    
    // Step 2: Generate Content
    console.log('Generating content with AI...');
    const content = await generateContent({
      keyword,
      research: researchText,
      format,
      language,
      tone,
    });
    
    // Step 3: Render Videos
    console.log('Rendering videos...');
    const videos = await Promise.all(
      platforms.map((platform: string) =>
        renderContentVideo({
          content,
          title: keyword,
          platform: platform as 'reels' | 'tiktok' | 'shorts',
        })
      )
    );
    
    return NextResponse.json({
      success: true,
      content,
      videos,
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
# Start Next.js development server
yarn dev
# or
npm run dev

# Access at http://localhost:3000
```

### Build for Production

```bash
# Build the application
yarn build

# Start production server
yarn start
```

### Remotion Studio (for video development)

```bash
# Open Remotion studio to preview/edit videos
npx remotion studio remotion/index.ts
```

## Common Patterns

### Pattern 1: Batch Content Generation

```typescript
// scripts/batch-generate.ts
import { crawlRecentNews } from '@/services/research/crawler';
import { generateContent } from '@/services/ai/claude-generator';

const keywords = [
  'AI marketing automation',
  'Content creation trends 2024',
  'Social media video strategy',
];

async function batchGenerate() {
  for (const keyword of keywords) {
    const research = await crawlRecentNews(keyword);
    const researchText = research.map(r => r.content).join('\n\n');
    
    const content = await generateContent({
      keyword,
      research: researchText,
      format: 'toplist',
      language: 'en',
      tone: 'professional',
    });
    
    console.log(`Generated content for: ${keyword}`);
    console.log(content);
    console.log('\n---\n');
  }
}

batchGenerate();
```

### Pattern 2: Multi-language Content Generation

```typescript
async function generateMultiLanguageContent(keyword: string) {
  const research = await crawlRecentNews(keyword);
  const researchText = research.map(r => r.content).join('\n\n');
  
  const [englishContent, vietnameseContent] = await Promise.all([
    generateContent({
      keyword,
      research: researchText,
      format: 'pov',
      language: 'en',
      tone: 'professional',
    }),
    generateContent({
      keyword,
      research: researchText,
      format: 'pov',
      language: 'vi',
      tone: 'friendly',
    }),
  ]);
  
  return { englishContent, vietnameseContent };
}
```

### Pattern 3: Scheduled Content Pipeline

```typescript
// lib/scheduler.ts
import cron from 'node-cron';

// Run every day at 9 AM
cron.schedule('0 9 * * *', async () => {
  const dailyKeyword = 'trending AI tools';
  
  const research = await crawlRecentNews(dailyKeyword);
  const content = await generateContent({
    keyword: dailyKeyword,
    research: research.map(r => r.content).join('\n\n'),
    format: 'toplist',
    language: 'en',
    tone: 'professional',
  });
  
  // Auto-publish logic here
  console.log('Daily content generated:', content);
});
```

## Troubleshooting

### Issue: API Rate Limits

```typescript
// lib/rate-limiter.ts
import pRetry from 'p-retry';

export async function callWithRetry<T>(
  fn: () => Promise<T>,
  retries = 3
): Promise<T> {
  return pRetry(fn, {
    retries,
    onFailedAttempt: (error) => {
      console.log(
        `Attempt ${error.attemptNumber} failed. ${error.retriesLeft} retries left.`
      );
    },
  });
}

// Usage
const content = await callWithRetry(() =>
  generateContent(params)
);
```

### Issue: Large Research Data

```typescript
// Chunk large research data
function chunkResearch(research: string[], maxChunkSize = 5000): string[] {
  const chunks: string[] = [];
  let currentChunk = '';
  
  for (const item of research) {
    if ((currentChunk + item).length > maxChunkSize) {
      chunks.push(currentChunk);
      currentChunk = item;
    } else {
      currentChunk += '\n\n' + item;
    }
  }
  
  if (currentChunk) chunks.push(currentChunk);
  return chunks;
}
```

### Issue: Remotion Rendering Timeout

```typescript
// Increase timeout for complex videos
await renderMedia({
  composition,
  serveUrl: bundleLocation,
  codec: 'h264',
  outputLocation: outputPath,
  timeoutInMilliseconds: 120000, // 2 minutes
  chromiumOptions: {
    headless: true,
    gl: 'angle',
  },
});
```

### Issue: Memory Issues with Large Videos

```bash
# Increase Node.js memory limit
NODE_OPTIONS="--max-old-space-size=4096" yarn dev
```

## Best Practices

1. **API Key Management**: Always use environment variables, never hardcode
2. **Error Handling**: Wrap AI calls in try-catch with proper logging
3. **Rate Limiting**: Implement delays between API calls to avoid rate limits
4. **Content Validation**: Validate generated content before rendering videos
5. **Caching**: Cache research results to reduce API calls
6. **Queue System**: Use a job queue (Bull, BullMQ) for video rendering tasks

This skill provides comprehensive guidance for implementing automated content pipelines with AI research, multi-format content generation, and video rendering capabilities.
