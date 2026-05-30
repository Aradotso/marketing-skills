---
name: ultimate-ai-content-pipeline
description: Automated content creation pipeline from research to script generation and video rendering using Claude, OpenAI, and Remotion
triggers:
  - how do I generate automated content with AI
  - create video content from text automatically
  - set up content pipeline with Claude and OpenAI
  - generate multilingual content from research
  - automate content creation from keyword to video
  - use Remotion to render AI-generated videos
  - scrape news and generate blog posts automatically
  - build automated marketing content workflow
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to help developers work with the Ultimate AI Content Pipeline - a comprehensive TypeScript-based system that automates the entire content creation process from research and scriptwriting to video generation. The pipeline integrates Claude 3, OpenAI, web scraping, and Remotion for video rendering.

## What This Project Does

The Ultimate AI Content Pipeline is an all-in-one content automation system that:

- **Auto-scrapes research** from sources like TechCrunch, a16z, Twitter/X, LinkedIn for fresh insights
- **Generates diverse content formats** (listicles, POV pieces, case studies, how-tos) using Claude/OpenAI
- **Creates multilingual content** (English & Vietnamese) with customizable tone
- **Renders videos and infographics** automatically using Remotion
- **Optimizes for multiple platforms** (Reels, TikTok, Shorts)

## Installation

### Prerequisites

```bash
# Required tools
node >= 18.x
npm or yarn
```

### Setup Steps

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
# AI Model APIs
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# Research/Scraping APIs
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Next.js Configuration
NEXT_PUBLIC_API_URL=http://localhost:3000

# Remotion Configuration
REMOTION_RENDER_CONCURRENCY=4
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── scraper/     # Web scraping modules
│   │   └── video/       # Remotion video rendering
│   ├── utils/           # Utility functions
│   └── types/           # TypeScript type definitions
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Core APIs and Usage

### 1. Research & Scraping Module

```typescript
import { researchTopic } from '@/lib/scraper/research';

// Scrape recent news and insights
async function gatherResearch(keyword: string) {
  const research = await researchTopic({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeframe: '24h',
    limit: 10
  });
  
  return research;
}

// Usage
const insights = await gatherResearch('AI automation');
console.log(insights.articles); // Array of scraped articles
console.log(insights.trends);   // Extracted trends
```

### 2. AI Content Generation

#### Using Claude (Anthropic)

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

async function generateContentClaude(research: any, format: string) {
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [
      {
        role: 'user',
        content: `Based on this research: ${JSON.stringify(research)}
        
Create a ${format} article in both English and Vietnamese.
Format: ${format}
Tone: Professional yet engaging
Include data and insights from the research.`
      }
    ],
  });

  return message.content;
}
```

#### Using OpenAI

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateContentOpenAI(research: any, options: {
  format: string;
  language: string;
  tone: string;
}) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are an expert content creator specializing in ${options.format} articles.`
      },
      {
        role: 'user',
        content: `Create a ${options.format} in ${options.language} with a ${options.tone} tone based on: ${JSON.stringify(research)}`
      }
    ],
    temperature: 0.7,
  });

  return completion.choices[0].message.content;
}
```

### 3. Complete Content Pipeline

```typescript
import { researchTopic } from '@/lib/scraper/research';
import { generateContent } from '@/lib/ai/generator';
import { renderVideo } from '@/lib/video/renderer';

interface ContentPipelineOptions {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  languages: ('en' | 'vi')[];
  tone: 'professional' | 'friendly' | 'humorous';
  includeVideo: boolean;
}

async function runContentPipeline(options: ContentPipelineOptions) {
  // Step 1: Research
  const research = await researchTopic({
    keyword: options.keyword,
    sources: ['techcrunch', 'a16z', 'twitter'],
    timeframe: '24h'
  });

  // Step 2: Generate content for each language
  const contentByLang = await Promise.all(
    options.languages.map(async (lang) => {
      const content = await generateContent({
        research,
        format: options.format,
        language: lang,
        tone: options.tone,
      });
      
      return { language: lang, content };
    })
  );

  // Step 3: Render video (optional)
  let videoUrl = null;
  if (options.includeVideo) {
    videoUrl = await renderVideo({
      content: contentByLang[0].content,
      template: 'short-form',
      format: '9:16', // Vertical for Reels/TikTok
    });
  }

  return {
    research,
    content: contentByLang,
    video: videoUrl,
  };
}

// Usage example
const result = await runContentPipeline({
  keyword: 'AI marketing automation',
  format: 'toplist',
  languages: ['en', 'vi'],
  tone: 'professional',
  includeVideo: true,
});
```

### 4. Remotion Video Rendering

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoRenderOptions {
  content: string;
  template: 'short-form' | 'infographic' | 'explainer';
  format: '16:9' | '9:16' | '1:1';
}

async function renderVideo(options: VideoRenderOptions) {
  // Bundle Remotion composition
  const bundleLocation = await bundle({
    entryPoint: path.join(process.cwd(), 'remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  // Get composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: options.template,
    inputProps: {
      content: options.content,
      aspectRatio: options.format,
    },
  });

  // Render video
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
      content: options.content,
      aspectRatio: options.format,
    },
  });

  return outputLocation;
}
```

### 5. Next.js API Route Example

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const { keyword, format, languages, tone, includeVideo } = body;

    // Validate input
    if (!keyword || !format) {
      return NextResponse.json(
        { error: 'Keyword and format are required' },
        { status: 400 }
      );
    }

    // Run pipeline
    const result = await runContentPipeline({
      keyword,
      format,
      languages: languages || ['en'],
      tone: tone || 'professional',
      includeVideo: includeVideo || false,
    });

    return NextResponse.json(result);
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { error: 'Failed to generate content' },
      { status: 500 }
    );
  }
}
```

### 6. Frontend Usage

```typescript
// src/components/ContentGenerator.tsx
'use client';

import { useState } from 'react';

export default function ContentGenerator() {
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState(null);

  async function handleGenerate(formData: FormData) {
    setLoading(true);
    
    try {
      const response = await fetch('/api/generate', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          keyword: formData.get('keyword'),
          format: formData.get('format'),
          languages: ['en', 'vi'],
          tone: formData.get('tone'),
          includeVideo: formData.get('includeVideo') === 'on',
        }),
      });

      const data = await response.json();
      setResult(data);
    } catch (error) {
      console.error('Generation failed:', error);
    } finally {
      setLoading(false);
    }
  }

  return (
    <div>
      <form action={handleGenerate}>
        <input name="keyword" placeholder="Enter keyword" required />
        <select name="format">
          <option value="toplist">Top List</option>
          <option value="pov">POV</option>
          <option value="case-study">Case Study</option>
          <option value="how-to">How To</option>
        </select>
        <select name="tone">
          <option value="professional">Professional</option>
          <option value="friendly">Friendly</option>
          <option value="humorous">Humorous</option>
        </select>
        <label>
          <input type="checkbox" name="includeVideo" />
          Generate Video
        </label>
        <button type="submit" disabled={loading}>
          {loading ? 'Generating...' : 'Generate Content'}
        </button>
      </form>

      {result && (
        <div>
          <h2>Generated Content</h2>
          {/* Display content */}
        </div>
      )}
    </div>
  );
}
```

## Development Workflow

### Running the Development Server

```bash
# Start Next.js development server
npm run dev

# Access at http://localhost:3000
```

### Testing Remotion Compositions

```bash
# Preview Remotion compositions
npm run remotion:preview

# Render a specific composition
npm run remotion:render
```

### Building for Production

```bash
# Build Next.js application
npm run build

# Start production server
npm start
```

## Common Patterns

### Pattern 1: Multi-Format Content Generation

```typescript
async function generateMultiFormatContent(keyword: string) {
  const research = await researchTopic({ keyword });
  
  const formats = ['toplist', 'pov', 'case-study', 'how-to'];
  
  const allContent = await Promise.all(
    formats.map(async (format) => {
      const content = await generateContent({
        research,
        format,
        language: 'en',
        tone: 'professional',
      });
      
      return { format, content };
    })
  );
  
  return allContent;
}
```

### Pattern 2: Batch Video Rendering

```typescript
async function batchRenderVideos(contents: string[]) {
  const concurrency = parseInt(process.env.REMOTION_RENDER_CONCURRENCY || '4');
  const results = [];
  
  for (let i = 0; i < contents.length; i += concurrency) {
    const batch = contents.slice(i, i + concurrency);
    const rendered = await Promise.all(
      batch.map((content) =>
        renderVideo({
          content,
          template: 'short-form',
          format: '9:16',
        })
      )
    );
    results.push(...rendered);
  }
  
  return results;
}
```

### Pattern 3: Content Scheduling

```typescript
interface ScheduledContent {
  content: string;
  publishDate: Date;
  platforms: ('facebook' | 'instagram' | 'tiktok')[];
}

async function scheduleContent(items: ScheduledContent[]) {
  // Store in database for later publishing
  for (const item of items) {
    await db.scheduledPosts.create({
      data: {
        content: item.content,
        publishAt: item.publishDate,
        platforms: item.platforms,
        status: 'scheduled',
      },
    });
  }
}
```

## Troubleshooting

### Issue: API Rate Limits

```typescript
// Implement retry logic with exponential backoff
async function callAIWithRetry(fn: () => Promise<any>, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error: any) {
      if (error.status === 429 && i < maxRetries - 1) {
        const delay = Math.pow(2, i) * 1000;
        await new Promise(resolve => setTimeout(resolve, delay));
        continue;
      }
      throw error;
    }
  }
}
```

### Issue: Remotion Rendering Failures

```typescript
// Add error handling and cleanup
async function safeRenderVideo(options: VideoRenderOptions) {
  let bundleLocation: string | null = null;
  
  try {
    bundleLocation = await bundle({
      entryPoint: path.join(process.cwd(), 'remotion/index.ts'),
    });
    
    // Render video
    const output = await renderVideo(options);
    return output;
  } catch (error) {
    console.error('Video rendering failed:', error);
    throw error;
  } finally {
    // Cleanup bundle if needed
    if (bundleLocation) {
      // Cleanup temporary files
    }
  }
}
```

### Issue: Missing Environment Variables

```typescript
// Validate environment on startup
function validateEnv() {
  const required = [
    'OPENAI_API_KEY',
    'ANTHROPIC_API_KEY',
    'RAPIDAPI_KEY',
  ];
  
  const missing = required.filter((key) => !process.env[key]);
  
  if (missing.length > 0) {
    throw new Error(
      `Missing required environment variables: ${missing.join(', ')}`
    );
  }
}

// Call at application start
validateEnv();
```

### Issue: Large Research Data

```typescript
// Chunk and summarize research data
async function summarizeResearch(articles: any[]) {
  const chunks = chunkArray(articles, 10);
  
  const summaries = await Promise.all(
    chunks.map(async (chunk) => {
      const summary = await openai.chat.completions.create({
        model: 'gpt-4-turbo-preview',
        messages: [
          {
            role: 'user',
            content: `Summarize these articles: ${JSON.stringify(chunk)}`
          }
        ],
      });
      
      return summary.choices[0].message.content;
    })
  );
  
  return summaries.join('\n\n');
}

function chunkArray<T>(array: T[], size: number): T[][] {
  const chunks: T[][] = [];
  for (let i = 0; i < array.length; i += size) {
    chunks.push(array.slice(i, i + size));
  }
  return chunks;
}
```

## Best Practices

1. **API Key Management**: Always use environment variables, never hardcode keys
2. **Error Handling**: Implement comprehensive try-catch blocks with logging
3. **Rate Limiting**: Respect API rate limits with exponential backoff
4. **Content Caching**: Cache research results to avoid redundant API calls
5. **Video Optimization**: Use appropriate concurrency settings for Remotion rendering
6. **Type Safety**: Leverage TypeScript types for better code reliability
