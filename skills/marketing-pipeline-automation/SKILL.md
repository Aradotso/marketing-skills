---
name: marketing-pipeline-automation
description: Automated AI content pipeline from research to video generation using Claude/OpenAI and Remotion
triggers:
  - set up automated content pipeline
  - create AI content workflow
  - automate content from research to video
  - generate content with AI pipeline
  - build marketing automation system
  - create automated video content
  - set up content generation pipeline
  - automate social media content creation
---

# Marketing Pipeline Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI agents to work with **Ultimate AI Content Pipeline**, a complete automated content generation system that handles research, script writing, and video generation. The pipeline crawls news sources, generates multi-format content using Claude/OpenAI, and produces videos with Remotion.

## What It Does

The marketing-pipeline-share project automates the entire content creation workflow:

1. **Auto-Research**: Crawls TechCrunch, a16z, Twitter, LinkedIn for recent news (24h)
2. **AI Content Generation**: Creates articles in multiple formats (toplist, POV, case study, how-to) using Claude 3 or OpenAI
3. **Multi-language Support**: Generates content in English and Vietnamese
4. **Video Generation**: Automatically renders infographics and videos using Remotion
5. **Platform Optimization**: Exports content optimized for Reels, TikTok, Shorts

## Installation

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
yarn install

# Set up environment variables
cp .env.example .env
```

## Configuration

Create a `.env` file with the following variables:

```bash
# AI Services
ANTHROPIC_API_KEY=your_claude_api_key_here
OPENAI_API_KEY=your_openai_api_key_here

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key_here

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Next.js
NEXT_PUBLIC_APP_URL=http://localhost:3000

# Remotion (for video rendering)
REMOTION_LICENSE_KEY=your_remotion_license_key
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI service integrations
│   │   ├── crawler/     # News crawling logic
│   │   ├── video/       # Remotion video generation
│   │   └── utils/       # Utility functions
│   └── types/           # TypeScript type definitions
├── public/              # Static assets
└── remotion/            # Remotion video templates
```

## Core Usage Patterns

### 1. Running the Research Pipeline

```typescript
import { researchTopic } from '@/lib/crawler/research';

async function gatherContent(keyword: string) {
  const results = await researchTopic({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeframe: '24h'
  });
  
  return results;
}

// Usage
const insights = await gatherContent('AI marketing trends');
console.log(`Found ${insights.length} articles`);
```

### 2. Generating Content with Claude

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY
});

async function generateContent(
  research: any[], 
  format: 'toplist' | 'pov' | 'case-study' | 'how-to',
  language: 'en' | 'vi'
) {
  const prompt = buildPrompt(research, format, language);
  
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [{
      role: 'user',
      content: prompt
    }]
  });
  
  return message.content[0].text;
}

function buildPrompt(research: any[], format: string, language: string): string {
  const researchSummary = research.map(r => 
    `Title: ${r.title}\nInsight: ${r.summary}\nSource: ${r.url}`
  ).join('\n\n');
  
  return `Based on this research:\n${researchSummary}\n\n` +
         `Create a ${format} article in ${language} language. ` +
         `Include data-backed insights and maintain an expert tone.`;
}
```

### 3. Generating Content with OpenAI

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY
});

async function generateWithOpenAI(
  research: any[],
  format: string,
  tone: 'expert' | 'friendly' | 'humorous'
) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are a content creator specializing in ${format} format with a ${tone} tone.`
      },
      {
        role: 'user',
        content: buildContentPrompt(research, format)
      }
    ],
    temperature: 0.7
  });
  
  return completion.choices[0].message.content;
}
```

### 4. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

async function generateVideo(content: string, format: 'reel' | 'tiktok' | 'shorts') {
  const bundleLocation = await bundle({
    entryPoint: path.join(process.cwd(), 'remotion/index.ts'),
    webpackOverride: (config) => config
  });
  
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: getCompositionId(format),
    inputProps: {
      content,
      format
    }
  });
  
  const outputPath = path.join(process.cwd(), `output/video-${Date.now()}.mp4`);
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath
  });
  
  return outputPath;
}

function getCompositionId(format: string): string {
  const compositions = {
    'reel': 'InstagramReel',
    'tiktok': 'TikTokVideo',
    'shorts': 'YouTubeShorts'
  };
  return compositions[format] || 'DefaultVideo';
}
```

### 5. Complete Pipeline Example

```typescript
import { researchTopic } from '@/lib/crawler/research';
import { generateContent } from '@/lib/ai/claude';
import { generateVideo } from '@/lib/video/remotion';

async function runContentPipeline(keyword: string) {
  try {
    // Step 1: Research
    console.log('🔍 Researching topic...');
    const research = await researchTopic({
      keyword,
      sources: ['techcrunch', 'twitter'],
      timeframe: '24h'
    });
    
    // Step 2: Generate content in multiple formats
    console.log('✍️ Generating content...');
    const formats = ['toplist', 'pov', 'how-to'] as const;
    const contentPieces = await Promise.all(
      formats.map(format => 
        generateContent(research, format, 'en')
      )
    );
    
    // Step 3: Generate videos
    console.log('🎬 Rendering videos...');
    const videos = await Promise.all(
      contentPieces.map((content, idx) => 
        generateVideo(content, 'reel')
      )
    );
    
    return {
      research,
      content: contentPieces,
      videos
    };
  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}

// Execute pipeline
runContentPipeline('AI marketing automation').then(result => {
  console.log('✅ Pipeline complete!');
  console.log(`Generated ${result.content.length} articles`);
  console.log(`Rendered ${result.videos.length} videos`);
});
```

### 6. API Route Example (Next.js)

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { researchTopic } from '@/lib/crawler/research';
import { generateContent } from '@/lib/ai/claude';

export async function POST(request: NextRequest) {
  try {
    const { keyword, format, language } = await request.json();
    
    // Validate inputs
    if (!keyword || !format || !language) {
      return NextResponse.json(
        { error: 'Missing required fields' },
        { status: 400 }
      );
    }
    
    // Run research
    const research = await researchTopic({
      keyword,
      sources: ['techcrunch', 'twitter'],
      timeframe: '24h'
    });
    
    // Generate content
    const content = await generateContent(research, format, language);
    
    return NextResponse.json({
      success: true,
      content,
      research: research.length
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

## Development Workflow

### Start Development Server

```bash
npm run dev
# or
yarn dev

# Server runs on http://localhost:3000
```

### Build for Production

```bash
npm run build
npm start
```

### Type Checking

```typescript
// src/types/content.ts
export interface ResearchResult {
  title: string;
  summary: string;
  url: string;
  source: string;
  publishedAt: Date;
  insights: string[];
}

export interface ContentOptions {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
}

export interface VideoOptions {
  format: 'reel' | 'tiktok' | 'shorts';
  duration: number;
  aspectRatio: '9:16' | '1:1' | '16:9';
}
```

## Common Patterns

### Error Handling and Retries

```typescript
async function withRetry<T>(
  fn: () => Promise<T>,
  maxRetries: number = 3,
  delay: number = 1000
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      await new Promise(resolve => setTimeout(resolve, delay * (i + 1)));
    }
  }
  throw new Error('Max retries exceeded');
}

// Usage
const content = await withRetry(() => 
  generateContent(research, 'toplist', 'en')
);
```

### Batch Processing

```typescript
async function batchGenerateContent(
  keywords: string[],
  format: string,
  language: string
) {
  const batchSize = 5;
  const results = [];
  
  for (let i = 0; i < keywords.length; i += batchSize) {
    const batch = keywords.slice(i, i + batchSize);
    const batchResults = await Promise.all(
      batch.map(async (keyword) => {
        const research = await researchTopic({ keyword, sources: ['techcrunch'], timeframe: '24h' });
        return generateContent(research, format, language);
      })
    );
    results.push(...batchResults);
  }
  
  return results;
}
```

### Caching Research Results

```typescript
import { LRUCache } from 'lru-cache';

const researchCache = new LRUCache<string, any[]>({
  max: 100,
  ttl: 1000 * 60 * 60 // 1 hour
});

async function cachedResearch(keyword: string) {
  const cached = researchCache.get(keyword);
  if (cached) {
    console.log('📦 Using cached research');
    return cached;
  }
  
  const results = await researchTopic({
    keyword,
    sources: ['techcrunch', 'twitter'],
    timeframe: '24h'
  });
  
  researchCache.set(keyword, results);
  return results;
}
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement rate limiting for API calls
import pLimit from 'p-limit';

const limit = pLimit(5); // Max 5 concurrent requests

async function generateMultipleWithRateLimit(
  items: string[]
) {
  return Promise.all(
    items.map(item => 
      limit(() => generateContent(item, 'toplist', 'en'))
    )
  );
}
```

### Claude API Errors

```typescript
try {
  const content = await generateContent(research, 'toplist', 'en');
} catch (error: any) {
  if (error.status === 429) {
    console.error('Rate limit exceeded. Wait before retrying.');
  } else if (error.status === 401) {
    console.error('Invalid API key. Check ANTHROPIC_API_KEY');
  } else {
    console.error('Claude API error:', error.message);
  }
}
```

### Video Rendering Issues

```typescript
// Ensure ffmpeg is installed
// For remotion video rendering errors:
import { getCompositions } from '@remotion/renderer';

async function debugVideoCompositions() {
  try {
    const compositions = await getCompositions(bundleLocation);
    console.log('Available compositions:', compositions);
  } catch (error) {
    console.error('Check if remotion dependencies are installed correctly');
    console.error('Run: npm install @remotion/cli @remotion/renderer');
  }
}
```

### Environment Variable Validation

```typescript
function validateEnv() {
  const required = [
    'ANTHROPIC_API_KEY',
    'OPENAI_API_KEY',
    'RAPIDAPI_KEY'
  ];
  
  const missing = required.filter(key => !process.env[key]);
  
  if (missing.length > 0) {
    throw new Error(
      `Missing required environment variables: ${missing.join(', ')}`
    );
  }
}

// Call at app startup
validateEnv();
```

## Best Practices

1. **Always validate environment variables** before running the pipeline
2. **Implement caching** for research results to avoid redundant API calls
3. **Use rate limiting** when working with multiple API providers
4. **Handle errors gracefully** with retry logic and meaningful error messages
5. **Monitor token usage** for Claude/OpenAI to manage costs
6. **Test video compositions** locally before deploying
7. **Store generated content** in a database for future reference

## Additional Resources

- Check `HUONG_DAN_CAI_DAT.md` for detailed setup instructions (Vietnamese)
- API documentation for integrations: Anthropic, OpenAI, RapidAPI
- Remotion documentation: https://www.remotion.dev/docs
