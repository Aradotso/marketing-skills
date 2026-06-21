---
name: ai-content-pipeline-automation
description: Automated content marketing pipeline from research to video generation using AI (Claude/OpenAI) and Remotion
triggers:
  - "set up automated content pipeline"
  - "generate content from research to video"
  - "automate blog post and video creation"
  - "crawl news and create AI content"
  - "use Remotion to render marketing videos"
  - "build AI content automation system"
  - "create automated marketing pipeline"
  - "generate multi-format content with AI"
---

# AI Content Pipeline Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This project is an end-to-end automated content marketing pipeline built with TypeScript, Next.js, and AI APIs. It crawls news sources, generates multi-format content (blog posts, scripts) using Claude 3 or OpenAI, and renders videos/graphics using Remotion. Perfect for marketers who want to automate content research, creation, and video production.

## What It Does

1. **Auto-Research**: Crawls real-time data from TechCrunch, a16z, Twitter, LinkedIn within the last 24 hours
2. **AI Content Generation**: Creates posts in multiple formats (Top List, POV, Case Study, How-to) using Claude or OpenAI
3. **Multi-Language**: Generates content in both English and Vietnamese with customizable tone
4. **Video Rendering**: Automatically renders infographics and short videos from content using Remotion
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

## Configuration

Create a `.env.local` file in the project root:

```env
# AI API Keys
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Next.js
NEXT_PUBLIC_API_URL=http://localhost:3000

# Remotion (optional)
REMOTION_LICENSE_KEY=your_remotion_license
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI providers (OpenAI, Claude)
│   │   ├── crawler/     # News crawling logic
│   │   └── video/       # Remotion video rendering
│   ├── types/           # TypeScript types
│   └── utils/           # Helper functions
├── remotion/            # Remotion compositions
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

# Render videos with Remotion
npm run render
```

## Key APIs and Usage

### 1. Content Research Module

```typescript
import { researchTopic } from '@/lib/crawler/research';

async function gatherInsights(keyword: string) {
  const research = await researchTopic({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeframe: '24h'
  });
  
  return {
    articles: research.articles,
    insights: research.insights,
    dataPoints: research.statistics
  };
}
```

### 2. AI Content Generation

```typescript
import { generateContent } from '@/lib/ai/content-generator';

async function createPost(topic: string) {
  const content = await generateContent({
    topic,
    format: 'toplist', // 'pov' | 'casestudy' | 'howto'
    language: 'vi', // 'en' | 'vi'
    tone: 'expert', // 'friendly' | 'humorous'
    provider: 'claude', // 'openai' | 'claude'
    researchData: await researchTopic({ keyword: topic })
  });
  
  return {
    title: content.title,
    body: content.body,
    metadata: content.metadata
  };
}
```

### 3. Using OpenAI Provider

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY
});

async function generateWithOpenAI(prompt: string) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: 'You are an expert content marketer.'
      },
      {
        role: 'user',
        content: prompt
      }
    ],
    temperature: 0.7,
    max_tokens: 2000
  });
  
  return completion.choices[0].message.content;
}
```

### 4. Using Claude Provider

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY
});

async function generateWithClaude(prompt: string) {
  const message = await anthropic.messages.create({
    model: 'claude-3-opus-20240229',
    max_tokens: 2048,
    messages: [
      {
        role: 'user',
        content: prompt
      }
    ]
  });
  
  return message.content[0].text;
}
```

### 5. Video Rendering with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

async function renderContentVideo(content: any) {
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: (config) => config
  });

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title: content.title,
      points: content.keyPoints,
      duration: 30
    }
  });

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: `out/${content.id}.mp4`,
    inputProps: {
      title: content.title,
      points: content.keyPoints
    }
  });
}
```

### 6. Remotion Composition Example

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, interpolate } from 'remotion';

export const ContentVideo: React.FC<{
  title: string;
  points: string[];
  duration: number;
}> = ({ title, points }) => {
  const frame = useCurrentFrame();
  
  const opacity = interpolate(
    frame,
    [0, 30],
    [0, 1],
    { extrapolateRight: 'clamp' }
  );

  return (
    <AbsoluteFill
      style={{
        backgroundColor: '#1a1a1a',
        justifyContent: 'center',
        alignItems: 'center',
        opacity
      }}
    >
      <h1 style={{ color: 'white', fontSize: 60 }}>{title}</h1>
      <ul>
        {points.map((point, i) => (
          <li key={i} style={{ color: 'white', fontSize: 30 }}>
            {point}
          </li>
        ))}
      </ul>
    </AbsoluteFill>
  );
};
```

## Common Patterns

### Full Pipeline Example

```typescript
import { researchTopic } from '@/lib/crawler/research';
import { generateContent } from '@/lib/ai/content-generator';
import { renderContentVideo } from '@/lib/video/renderer';

async function runFullPipeline(keyword: string) {
  // Step 1: Research
  console.log('📡 Researching topic...');
  const research = await researchTopic({
    keyword,
    sources: ['techcrunch', 'twitter'],
    timeframe: '24h'
  });

  // Step 2: Generate content in multiple formats
  console.log('🧠 Generating content...');
  const formats = ['toplist', 'pov', 'howto'] as const;
  const contents = await Promise.all(
    formats.map(format =>
      generateContent({
        topic: keyword,
        format,
        language: 'vi',
        provider: 'claude',
        researchData: research
      })
    )
  );

  // Step 3: Render videos
  console.log('🎬 Rendering videos...');
  const videos = await Promise.all(
    contents.map(content => renderContentVideo(content))
  );

  return {
    research,
    contents,
    videos
  };
}
```

### Multi-Language Content Generation

```typescript
async function generateMultiLangContent(topic: string) {
  const languages = ['en', 'vi'];
  
  const translations = await Promise.all(
    languages.map(async (lang) => {
      const content = await generateContent({
        topic,
        format: 'toplist',
        language: lang,
        tone: 'expert',
        provider: 'claude'
      });
      
      return { language: lang, content };
    })
  );
  
  return Object.fromEntries(
    translations.map(t => [t.language, t.content])
  );
}
```

### Batch Processing

```typescript
async function batchGenerateContent(keywords: string[]) {
  const results = [];
  
  for (const keyword of keywords) {
    try {
      const result = await runFullPipeline(keyword);
      results.push({ keyword, success: true, data: result });
    } catch (error) {
      console.error(`Failed for ${keyword}:`, error);
      results.push({ keyword, success: false, error });
    }
    
    // Rate limiting
    await new Promise(resolve => setTimeout(resolve, 2000));
  }
  
  return results;
}
```

## API Routes

The Next.js app includes API routes for the pipeline:

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';

export async function POST(request: NextRequest) {
  const { keyword, format, language } = await request.json();
  
  const research = await researchTopic({ keyword });
  const content = await generateContent({
    topic: keyword,
    format,
    language,
    provider: 'claude',
    researchData: research
  });
  
  return NextResponse.json({ content });
}
```

Usage from client:

```typescript
async function generateFromClient(keyword: string) {
  const response = await fetch('/api/generate', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      keyword,
      format: 'toplist',
      language: 'vi'
    })
  });
  
  return response.json();
}
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement retry logic with exponential backoff
async function retryWithBackoff<T>(
  fn: () => Promise<T>,
  maxRetries = 3
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error: any) {
      if (error.status === 429 && i < maxRetries - 1) {
        const delay = Math.pow(2, i) * 1000;
        await new Promise(resolve => setTimeout(resolve, delay));
      } else {
        throw error;
      }
    }
  }
  throw new Error('Max retries exceeded');
}
```

### Video Rendering Issues

```bash
# Install required codecs
npm install @remotion/lambda

# Increase memory for large renders
NODE_OPTIONS=--max-old-space-size=4096 npm run render
```

### Missing Environment Variables

```typescript
function validateEnv() {
  const required = [
    'OPENAI_API_KEY',
    'ANTHROPIC_API_KEY',
    'RAPIDAPI_KEY'
  ];
  
  const missing = required.filter(key => !process.env[key]);
  
  if (missing.length > 0) {
    throw new Error(
      `Missing required env vars: ${missing.join(', ')}`
    );
  }
}
```

## Advanced Configuration

### Custom Remotion Config

```typescript
// remotion.config.ts
import { Config } from '@remotion/cli/config';

Config.setVideoImageFormat('jpeg');
Config.setOverwriteOutput(true);
Config.setConcurrency(8);
Config.setCodec('h264');
```

### Custom AI Prompts

```typescript
const PROMPT_TEMPLATES = {
  toplist: (topic: string, research: any) => `
    Create a top 10 list about ${topic}.
    Use these recent insights: ${JSON.stringify(research.insights)}
    Format: Vietnamese, expert tone, data-backed claims.
  `,
  pov: (topic: string, research: any) => `
    Write a personal perspective on ${topic}.
    Include contrarian views based on: ${JSON.stringify(research.insights)}
  `
};
```

This skill enables AI agents to help developers set up and use the complete automated content marketing pipeline, from research through AI generation to video rendering.
