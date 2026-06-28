---
name: ultimate-ai-content-pipeline
description: Automated content creation pipeline with AI research, script generation, auto-posting, and video rendering using Claude/OpenAI and Remotion
triggers:
  - "set up automated content pipeline with AI"
  - "create content from research to video automatically"
  - "generate marketing content with Claude and OpenAI"
  - "automate content research and video generation"
  - "build AI content pipeline with Remotion"
  - "scrape news and generate content automatically"
  - "create multilingual content with AI automation"
  - "render videos from AI-generated content"
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

A comprehensive TypeScript-based content automation system that handles the entire content lifecycle: from researching trending topics, generating scripts in multiple formats and languages, to rendering videos and auto-posting to social platforms.

## What It Does

This pipeline automates:
- **Auto-Scan Research**: Crawls real-time data from TechCrunch, a16z, Twitter/X, LinkedIn
- **AI Content Generation**: Creates content in multiple formats (Toplist, POV, Case Study, How-to) using Claude 3 and OpenAI
- **Multilingual Support**: Generates parallel English and Vietnamese content
- **Video Rendering**: Automatically creates infographics and short-form videos using Remotion
- **Multi-Platform Export**: Optimizes video output for Reels, TikTok, and Shorts

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

# RapidAPI for web scraping
RAPIDAPI_KEY=your_rapidapi_key

# Database (if using)
DATABASE_URL=your_database_connection_string

# Remotion configuration
REMOTION_AWS_ACCESS_KEY_ID=your_aws_access_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret_key

# Next.js configuration
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app router
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── scraper/     # Web scraping modules
│   │   ├── content/     # Content generation logic
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

# Render Remotion video
npm run remotion:render
```

## Core Modules and Usage

### 1. Research & Scraping Module

```typescript
import { scrapeNewsSource } from '@/lib/scraper';

async function gatherResearch(keyword: string) {
  const sources = [
    'techcrunch',
    'a16z',
    'twitter',
    'linkedin'
  ];
  
  const researchData = await Promise.all(
    sources.map(source => 
      scrapeNewsSource({
        source,
        keyword,
        timeframe: '24h',
        maxResults: 10
      })
    )
  );
  
  return researchData.flat();
}
```

### 2. AI Content Generation

```typescript
import { generateContent } from '@/lib/ai/generator';

interface ContentRequest {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  researchData: any[];
}

async function createContent(request: ContentRequest) {
  const { keyword, format, language, tone, researchData } = request;
  
  // Generate content using Claude or OpenAI
  const content = await generateContent({
    provider: 'claude', // or 'openai'
    model: 'claude-3-opus-20240229',
    prompt: {
      keyword,
      format,
      language,
      tone,
      context: researchData
    },
    maxTokens: 4000
  });
  
  return content;
}
```

### 3. Claude API Integration

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY
});

async function generateWithClaude(prompt: string, context: any) {
  const message = await anthropic.messages.create({
    model: 'claude-3-opus-20240229',
    max_tokens: 4000,
    messages: [
      {
        role: 'user',
        content: `
Research Data: ${JSON.stringify(context)}

Task: ${prompt}

Please generate engaging content based on the research data above.
        `
      }
    ]
  });
  
  return message.content[0].text;
}
```

### 4. OpenAI API Integration

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY
});

async function generateWithOpenAI(prompt: string, context: any) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: 'You are an expert content creator specializing in marketing and trending topics.'
      },
      {
        role: 'user',
        content: `
Research Data: ${JSON.stringify(context)}

${prompt}
        `
      }
    ],
    temperature: 0.7,
    max_tokens: 3000
  });
  
  return completion.choices[0].message.content;
}
```

### 5. Video Rendering with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

async function renderContentVideo(content: any) {
  const compositionId = 'ContentVideo';
  const bundleLocation = await bundle(
    path.join(process.cwd(), 'remotion/index.ts')
  );
  
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: {
      title: content.title,
      highlights: content.highlights,
      style: 'reels' // or 'tiktok', 'shorts'
    }
  });
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: `out/${content.id}.mp4`,
    inputProps: {
      title: content.title,
      highlights: content.highlights
    }
  });
}
```

### 6. Remotion Video Component

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig } from 'remotion';

export const ContentVideo: React.FC<{
  title: string;
  highlights: string[];
  style: string;
}> = ({ title, highlights, style }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  
  const opacity = Math.min(1, frame / 30);
  
  return (
    <AbsoluteFill
      style={{
        backgroundColor: '#000',
        justifyContent: 'center',
        alignItems: 'center',
        opacity
      }}
    >
      <h1 style={{ color: '#fff', fontSize: 60 }}>{title}</h1>
      {highlights.map((highlight, i) => (
        <p
          key={i}
          style={{
            color: '#fff',
            fontSize: 30,
            opacity: frame > (i + 1) * fps ? 1 : 0
          }}
        >
          {highlight}
        </p>
      ))}
    </AbsoluteFill>
  );
};
```

## Common Workflow Patterns

### Full Content Pipeline

```typescript
import { 
  gatherResearch, 
  createContent, 
  renderContentVideo 
} from '@/lib/pipeline';

async function runContentPipeline(keyword: string) {
  // Step 1: Research
  console.log('Gathering research data...');
  const researchData = await gatherResearch(keyword);
  
  // Step 2: Generate content in multiple languages
  console.log('Generating content...');
  const [contentEN, contentVI] = await Promise.all([
    createContent({
      keyword,
      format: 'toplist',
      language: 'en',
      tone: 'expert',
      researchData
    }),
    createContent({
      keyword,
      format: 'toplist',
      language: 'vi',
      tone: 'friendly',
      researchData
    })
  ]);
  
  // Step 3: Render videos
  console.log('Rendering videos...');
  await Promise.all([
    renderContentVideo(contentEN),
    renderContentVideo(contentVI)
  ]);
  
  return { contentEN, contentVI };
}
```

### Multi-Format Generation

```typescript
async function generateMultipleFormats(keyword: string, researchData: any[]) {
  const formats = ['toplist', 'pov', 'case-study', 'how-to'] as const;
  
  const contents = await Promise.all(
    formats.map(format =>
      createContent({
        keyword,
        format,
        language: 'en',
        tone: 'expert',
        researchData
      })
    )
  );
  
  return contents;
}
```

### API Route Example

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline';

export async function POST(request: NextRequest) {
  try {
    const { keyword } = await request.json();
    
    if (!keyword) {
      return NextResponse.json(
        { error: 'Keyword is required' },
        { status: 400 }
      );
    }
    
    const result = await runContentPipeline(keyword);
    
    return NextResponse.json({
      success: true,
      data: result
    });
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { error: 'Pipeline failed' },
      { status: 500 }
    );
  }
}
```

## Configuration Options

### AI Provider Selection

```typescript
// lib/ai/config.ts
export const AI_CONFIG = {
  defaultProvider: 'claude', // or 'openai'
  models: {
    claude: 'claude-3-opus-20240229',
    openai: 'gpt-4-turbo-preview'
  },
  maxTokens: 4000,
  temperature: 0.7
};
```

### Scraper Configuration

```typescript
// lib/scraper/config.ts
export const SCRAPER_CONFIG = {
  sources: {
    techcrunch: {
      url: 'https://techcrunch.com',
      selector: '.post-block'
    },
    twitter: {
      apiEndpoint: '/api/twitter/search',
      maxResults: 20
    }
  },
  timeframe: '24h',
  rateLimiting: {
    requestsPerMinute: 10
  }
};
```

### Video Export Settings

```typescript
// remotion/config.ts
export const VIDEO_CONFIG = {
  reels: {
    width: 1080,
    height: 1920,
    fps: 30,
    durationInFrames: 300
  },
  tiktok: {
    width: 1080,
    height: 1920,
    fps: 30,
    durationInFrames: 300
  },
  shorts: {
    width: 1080,
    height: 1920,
    fps: 30,
    durationInFrames: 300
  }
};
```

## Troubleshooting

### API Rate Limiting

If you encounter rate limits:

```typescript
import pRetry from 'p-retry';

async function generateWithRetry(prompt: string) {
  return pRetry(
    async () => {
      return await generateWithClaude(prompt, {});
    },
    {
      retries: 3,
      minTimeout: 1000,
      onFailedAttempt: error => {
        console.log(`Attempt ${error.attemptNumber} failed. Retrying...`);
      }
    }
  );
}
```

### Video Rendering Issues

Check Remotion setup:

```bash
# Ensure Remotion CLI is installed
npm install -g @remotion/cli

# Test video rendering
npx remotion preview remotion/index.ts
```

### Memory Issues with Large Content

Use streaming for large content:

```typescript
async function generateLargeContent(keyword: string) {
  const stream = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [{ role: 'user', content: keyword }],
    stream: true
  });
  
  let content = '';
  for await (const chunk of stream) {
    const text = chunk.choices[0]?.delta?.content || '';
    content += text;
    // Process chunks as they arrive
  }
  
  return content;
}
```

### Environment Variables Not Loading

Ensure `.env.local` is in the root and restart the dev server:

```bash
# Kill existing process
pkill -f "next dev"

# Restart
npm run dev
```

## Advanced Usage

### Custom Content Templates

```typescript
// lib/content/templates.ts
export const CONTENT_TEMPLATES = {
  toplist: {
    structure: [
      'compelling_intro',
      'numbered_points',
      'data_backed_insights',
      'actionable_conclusion'
    ],
    minPoints: 5,
    maxPoints: 10
  },
  pov: {
    structure: [
      'hook',
      'personal_perspective',
      'supporting_evidence',
      'controversial_take',
      'call_to_action'
    ]
  }
};
```

### Scheduling and Automation

```typescript
import cron from 'node-cron';

// Run pipeline daily at 9 AM
cron.schedule('0 9 * * *', async () => {
  const trendingTopics = await fetchTrendingTopics();
  for (const topic of trendingTopics) {
    await runContentPipeline(topic);
  }
});
```

This skill provides comprehensive guidance for AI coding agents to effectively use the Ultimate AI Content Pipeline for automated marketing content creation.
