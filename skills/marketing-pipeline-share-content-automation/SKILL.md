---
name: marketing-pipeline-share-content-automation
description: AI-powered content automation pipeline for research, scriptwriting, and video generation using Claude/OpenAI and Remotion
triggers:
  - automate content creation with AI research
  - generate marketing content from keyword research
  - create video content from text automatically
  - build content pipeline with Claude and OpenAI
  - automate social media content generation
  - crawl news and generate blog posts
  - research and write articles with AI
  - remotion video generation from content
---

# Marketing Pipeline Share Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

Marketing Pipeline Share is a comprehensive content automation system that transforms keywords into ready-to-publish content. It automates the entire content creation workflow: researching trending topics from sources like TechCrunch and Twitter, generating multilingual articles using Claude/OpenAI, and rendering videos with Remotion. Built with TypeScript and Next.js.

## Key Capabilities

- **Automated Research**: Crawls news sources (TechCrunch, a16z, X, LinkedIn) for recent data
- **AI Content Generation**: Creates articles in multiple formats (toplist, POV, case study, how-to)
- **Multilingual Support**: Generates content in English and Vietnamese simultaneously
- **Video Generation**: Auto-renders videos and infographics using Remotion
- **Flexible Format**: Customizable tone (expert, friendly, humorous) and style

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
cp .env.example .env.local
```

## Configuration

Create a `.env.local` file with the following variables:

```bash
# AI Services
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Application
NEXT_PUBLIC_APP_URL=http://localhost:3000
DATABASE_URL=your_database_url

# Video Rendering (Remotion)
REMOTION_BUNDLE_DIR=./out/bundle
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI service integrations
│   │   ├── research/    # Content research crawlers
│   │   ├── video/       # Remotion video generation
│   │   └── utils/       # Utility functions
│   └── types/           # TypeScript type definitions
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Core Usage Patterns

### 1. Research Content from Sources

```typescript
import { researchTopic } from '@/lib/research/crawler';

async function gatherInsights(keyword: string) {
  const research = await researchTopic({
    keyword,
    sources: ['techcrunch', 'twitter', 'linkedin'],
    timeRange: '24h',
    maxResults: 20
  });

  return {
    articles: research.articles,
    insights: research.insights,
    statistics: research.statistics
  };
}
```

### 2. Generate Content with AI

```typescript
import { generateContent } from '@/lib/ai/content-generator';

async function createArticle(topic: string, researchData: any) {
  const content = await generateContent({
    topic,
    format: 'toplist', // 'pov' | 'case-study' | 'how-to'
    tone: 'expert', // 'friendly' | 'humorous'
    languages: ['en', 'vi'],
    researchData,
    provider: 'claude', // or 'openai'
    model: 'claude-3-opus-20240229'
  });

  return content;
}
```

### 3. Using Claude API Integration

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

async function generateWithClaude(prompt: string, researchContext: string) {
  const message = await anthropic.messages.create({
    model: 'claude-3-opus-20240229',
    max_tokens: 4096,
    messages: [
      {
        role: 'user',
        content: `${researchContext}\n\n${prompt}`
      }
    ],
    system: 'You are an expert content creator specializing in data-driven marketing articles.'
  });

  return message.content[0].text;
}
```

### 4. Using OpenAI Integration

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateWithOpenAI(prompt: string, context: string) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: 'You are a marketing content expert.'
      },
      {
        role: 'user',
        content: `Context: ${context}\n\nTask: ${prompt}`
      }
    ],
    temperature: 0.7,
  });

  return completion.choices[0].message.content;
}
```

### 5. Generate Video with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

async function generateContentVideo(
  contentData: {
    title: string;
    points: string[];
    statistics: any[];
  }
) {
  // Bundle Remotion composition
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  // Select composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: contentData,
  });

  // Render video
  const outputLocation = path.resolve(`./public/videos/${Date.now()}.mp4`);
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: contentData,
  });

  return outputLocation;
}
```

### 6. Complete Pipeline Example

```typescript
import { Pipeline } from '@/lib/pipeline';

async function runContentPipeline(keyword: string) {
  const pipeline = new Pipeline({
    aiProvider: 'claude',
    videoEnabled: true,
    languages: ['en', 'vi']
  });

  // Step 1: Research
  const research = await pipeline.research({
    keyword,
    sources: ['techcrunch', 'twitter'],
    depth: 'comprehensive'
  });

  // Step 2: Generate Content
  const content = await pipeline.generate({
    research,
    format: 'toplist',
    tone: 'expert',
    minWords: 1500
  });

  // Step 3: Create Video
  const video = await pipeline.createVideo({
    content,
    template: 'infographic',
    aspectRatio: '9:16', // For Reels/TikTok
    duration: 60
  });

  // Step 4: Schedule Publishing (if integrated)
  await pipeline.schedule({
    content,
    video,
    platforms: ['facebook', 'linkedin'],
    publishAt: new Date('2024-01-01T10:00:00Z')
  });

  return {
    content,
    videoUrl: video.url,
    status: 'scheduled'
  };
}
```

## API Routes (Next.js)

### Create Content API

```typescript
// app/api/content/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { generateContent } from '@/lib/ai/content-generator';

export async function POST(request: NextRequest) {
  try {
    const { keyword, format, languages } = await request.json();

    const content = await generateContent({
      topic: keyword,
      format,
      languages,
      provider: 'claude'
    });

    return NextResponse.json({ success: true, content });
  } catch (error) {
    return NextResponse.json(
      { error: 'Content generation failed' },
      { status: 500 }
    );
  }
}
```

### Research API

```typescript
// app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { researchTopic } from '@/lib/research/crawler';

export async function POST(request: NextRequest) {
  try {
    const { keyword, sources, timeRange } = await request.json();

    const research = await researchTopic({
      keyword,
      sources,
      timeRange
    });

    return NextResponse.json({ success: true, research });
  } catch (error) {
    return NextResponse.json(
      { error: 'Research failed' },
      { status: 500 }
    );
  }
}
```

## Running the Application

```bash
# Development mode
npm run dev

# Build for production
npm run build

# Start production server
npm run start

# Render Remotion video preview
npm run remotion:preview

# Bundle Remotion for production
npm run remotion:bundle
```

## Common Patterns

### Multi-Language Content Generation

```typescript
async function generateMultilingualContent(topic: string) {
  const languages = ['en', 'vi'];
  const contents = {};

  for (const lang of languages) {
    contents[lang] = await generateContent({
      topic,
      format: 'toplist',
      languages: [lang],
      provider: 'claude'
    });
  }

  return contents;
}
```

### Content Variation Testing

```typescript
async function generateVariations(topic: string) {
  const formats = ['toplist', 'pov', 'case-study'];
  const tones = ['expert', 'friendly', 'humorous'];

  const variations = [];

  for (const format of formats) {
    for (const tone of tones) {
      const content = await generateContent({
        topic,
        format,
        tone,
        provider: 'openai'
      });
      variations.push({ format, tone, content });
    }
  }

  return variations;
}
```

### Scheduled Content Pipeline

```typescript
import cron from 'node-cron';

function setupAutomatedPipeline(keywords: string[]) {
  // Run daily at 6 AM
  cron.schedule('0 6 * * *', async () => {
    for (const keyword of keywords) {
      try {
        await runContentPipeline(keyword);
        console.log(`Pipeline completed for: ${keyword}`);
      } catch (error) {
        console.error(`Pipeline failed for ${keyword}:`, error);
      }
    }
  });
}
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement retry with exponential backoff
async function generateWithRetry(prompt: string, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await generateContent({ topic: prompt });
    } catch (error) {
      if (error.status === 429) {
        const delay = Math.pow(2, i) * 1000;
        await new Promise(resolve => setTimeout(resolve, delay));
        continue;
      }
      throw error;
    }
  }
  throw new Error('Max retries exceeded');
}
```

### Video Rendering Errors

Check Remotion bundle configuration and ensure all dependencies are installed:

```bash
npm install @remotion/bundler @remotion/renderer @remotion/cli
```

### Missing Research Data

Verify RapidAPI key and source availability:

```typescript
async function validateSources(sources: string[]) {
  const available = [];
  
  for (const source of sources) {
    try {
      await checkSourceAvailability(source);
      available.push(source);
    } catch (error) {
      console.warn(`Source ${source} unavailable:`, error);
    }
  }
  
  return available;
}
```

### Memory Issues with Large Content

Use streaming for large content generation:

```typescript
async function generateLargeContent(topic: string) {
  const stream = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [{ role: 'user', content: topic }],
    stream: true,
  });

  let content = '';
  for await (const chunk of stream) {
    content += chunk.choices[0]?.delta?.content || '';
  }

  return content;
}
```

## Best Practices

1. **Always validate research data** before generating content
2. **Cache AI responses** to reduce API costs
3. **Use environment variables** for all API keys
4. **Implement error handling** for each pipeline step
5. **Test video rendering** locally before production
6. **Monitor API usage** to avoid unexpected costs
7. **Version control** your content templates and prompts
