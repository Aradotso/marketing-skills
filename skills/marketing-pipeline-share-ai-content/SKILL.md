---
name: marketing-pipeline-share-ai-content
description: Automate content creation from research to video generation using AI (Claude/OpenAI) and Remotion
triggers:
  - "how do I automate content creation with AI"
  - "generate video content from text automatically"
  - "set up AI content pipeline with Claude and OpenAI"
  - "create marketing content using research automation"
  - "build automated content workflow with Remotion"
  - "scrape news and generate content with AI"
  - "automate social media video creation"
  - "set up multi-language content generation pipeline"
---

# Marketing Pipeline Share AI Content

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to help developers use the **Ultimate AI Content Pipeline** - an automated content creation system that handles research, script writing, and video generation. The pipeline scrapes news from sources like TechCrunch, a16z, Twitter, and LinkedIn, then uses Claude/OpenAI to generate multilingual content in various formats, and finally renders videos using Remotion.

## What This Project Does

Marketing Pipeline Share is a Next.js TypeScript application that creates a complete content automation workflow:

1. **Auto-Research**: Crawls news sources for trending topics within 24 hours
2. **AI Content Generation**: Creates articles in multiple formats (Top Lists, POV, Case Studies, How-To) using Claude 3 or OpenAI
3. **Multi-language Support**: Generates content in English and Vietnamese simultaneously
4. **Video Rendering**: Automatically creates infographics and short videos from content using Remotion
5. **Platform Optimization**: Exports videos optimized for Reels, TikTok, and Shorts

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

# Set up environment variables
cp .env.example .env.local
```

### Required Environment Variables

```bash
# AI Services
ANTHROPIC_API_KEY=your_anthropic_api_key_here
OPENAI_API_KEY=your_openai_api_key_here

# Research/Scraping
RAPIDAPI_KEY=your_rapidapi_key_here

# Optional: Database (if using)
DATABASE_URL=your_database_connection_string

# Optional: Video Storage
STORAGE_BUCKET_URL=your_storage_url
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integrations (Claude, OpenAI)
│   │   ├── scraper/     # News scraping modules
│   │   └── video/       # Remotion video rendering
│   ├── types/           # TypeScript type definitions
│   └── utils/           # Utility functions
├── public/              # Static assets
└── remotion/            # Remotion video templates
```

## Key API Usage Patterns

### 1. Research/Scraping Module

```typescript
import { scrapeNews } from '@/lib/scraper/news-scraper';

interface NewsSource {
  source: string;
  title: string;
  url: string;
  publishedAt: Date;
  content: string;
}

async function gatherResearch(keyword: string): Promise<NewsSource[]> {
  const sources = ['techcrunch', 'a16z', 'twitter', 'linkedin'];
  
  const results = await scrapeNews({
    keyword,
    sources,
    timeRange: '24h',
    maxResults: 20
  });
  
  return results;
}

// Usage
const research = await gatherResearch('AI automation');
console.log(`Found ${research.length} relevant articles`);
```

### 2. AI Content Generation with Claude

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

interface ContentRequest {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  researchData: NewsSource[];
}

async function generateContent(request: ContentRequest): Promise<string> {
  const prompt = `
You are a content creator. Generate a ${request.format} article about "${request.keyword}".

Language: ${request.language}
Tone: ${request.tone}

Research data:
${request.researchData.map(item => `- ${item.title} (${item.source}): ${item.content}`).join('\n')}

Requirements:
- Include data-backed insights
- Add relevant statistics
- Make it engaging and actionable
- Format in markdown
`;

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [
      { role: 'user', content: prompt }
    ],
  });

  return message.content[0].type === 'text' ? message.content[0].text : '';
}

// Usage
const content = await generateContent({
  keyword: 'AI Marketing Tools',
  format: 'toplist',
  language: 'en',
  tone: 'expert',
  researchData: research
});
```

### 3. OpenAI Alternative Implementation

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateContentOpenAI(request: ContentRequest): Promise<string> {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are an expert content creator specializing in ${request.format} articles with a ${request.tone} tone.`
      },
      {
        role: 'user',
        content: `Create a ${request.language} ${request.format} article about "${request.keyword}" using this research:\n\n${JSON.stringify(request.researchData, null, 2)}`
      }
    ],
    temperature: 0.7,
    max_tokens: 3000,
  });

  return completion.choices[0].message.content || '';
}
```

### 4. Video Rendering with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import { webpackOverride } from './remotion/webpack-override';
import path from 'path';

interface VideoConfig {
  content: string;
  title: string;
  format: 'reels' | 'tiktok' | 'shorts';
  duration: number;
}

async function renderContentVideo(config: VideoConfig): Promise<string> {
  // Resolution mapping for different platforms
  const resolutions = {
    reels: { width: 1080, height: 1920 },
    tiktok: { width: 1080, height: 1920 },
    shorts: { width: 1080, height: 1920 },
  };

  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride,
  });

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title: config.title,
      content: config.content,
      duration: config.duration,
    },
  });

  const outputPath = `./public/videos/${Date.now()}-${config.format}.mp4`;

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    ...resolutions[config.format],
  });

  return outputPath;
}

// Usage
const videoPath = await renderContentVideo({
  content: content,
  title: 'Top 10 AI Marketing Tools',
  format: 'reels',
  duration: 60
});
```

### 5. Complete Pipeline Workflow

```typescript
import { scrapeNews } from '@/lib/scraper/news-scraper';
import { generateContent } from '@/lib/ai/claude-generator';
import { renderContentVideo } from '@/lib/video/renderer';

interface PipelineConfig {
  keyword: string;
  contentFormat: 'toplist' | 'pov' | 'case-study' | 'how-to';
  languages: ('en' | 'vi')[];
  videoFormats: ('reels' | 'tiktok' | 'shorts')[];
  tone: 'expert' | 'friendly' | 'humorous';
}

async function runContentPipeline(config: PipelineConfig) {
  console.log(`Starting pipeline for keyword: ${config.keyword}`);
  
  // Step 1: Research
  console.log('📡 Gathering research...');
  const research = await scrapeNews({
    keyword: config.keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeRange: '24h',
    maxResults: 20
  });
  
  // Step 2: Generate content in multiple languages
  console.log('🧠 Generating content...');
  const contents = await Promise.all(
    config.languages.map(language =>
      generateContent({
        keyword: config.keyword,
        format: config.contentFormat,
        language,
        tone: config.tone,
        researchData: research
      })
    )
  );
  
  // Step 3: Render videos for each content and platform
  console.log('🎬 Rendering videos...');
  const videos = [];
  
  for (const [index, content] of contents.entries()) {
    const language = config.languages[index];
    
    for (const format of config.videoFormats) {
      const videoPath = await renderContentVideo({
        content,
        title: `${config.keyword} - ${language.toUpperCase()}`,
        format,
        duration: 60
      });
      
      videos.push({ language, format, path: videoPath });
    }
  }
  
  console.log('✅ Pipeline complete!');
  
  return {
    research,
    contents,
    videos
  };
}

// Usage example
const result = await runContentPipeline({
  keyword: 'AI Marketing Automation 2026',
  contentFormat: 'toplist',
  languages: ['en', 'vi'],
  videoFormats: ['reels', 'tiktok'],
  tone: 'expert'
});

console.log(`Generated ${result.contents.length} articles`);
console.log(`Rendered ${result.videos.length} videos`);
```

## Running the Development Server

```bash
# Start the Next.js development server
npm run dev

# Access at http://localhost:3000
```

## API Routes

### Generate Content Endpoint

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    
    const result = await runContentPipeline({
      keyword: body.keyword,
      contentFormat: body.format || 'toplist',
      languages: body.languages || ['en'],
      videoFormats: body.videoFormats || ['reels'],
      tone: body.tone || 'expert'
    });
    
    return NextResponse.json({
      success: true,
      data: result
    });
  } catch (error) {
    return NextResponse.json({
      success: false,
      error: error.message
    }, { status: 500 });
  }
}
```

### Client-side Usage

```typescript
// Client component
'use client';

import { useState } from 'react';

export default function ContentGenerator() {
  const [loading, setLoading] = useState(false);
  
  async function handleGenerate(keyword: string) {
    setLoading(true);
    
    try {
      const response = await fetch('/api/generate', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          keyword,
          format: 'toplist',
          languages: ['en', 'vi'],
          videoFormats: ['reels'],
          tone: 'expert'
        })
      });
      
      const result = await response.json();
      
      if (result.success) {
        console.log('Content generated:', result.data);
        // Handle success
      }
    } catch (error) {
      console.error('Generation failed:', error);
    } finally {
      setLoading(false);
    }
  }
  
  return (
    <div>
      <button onClick={() => handleGenerate('AI Marketing')} disabled={loading}>
        {loading ? 'Generating...' : 'Generate Content'}
      </button>
    </div>
  );
}
```

## Common Patterns

### Custom Tone Presets

```typescript
const tonePresets = {
  expert: {
    systemPrompt: 'You are an industry expert providing authoritative insights.',
    temperature: 0.6,
    style: 'professional, data-driven, analytical'
  },
  friendly: {
    systemPrompt: 'You are a helpful friend sharing useful knowledge.',
    temperature: 0.8,
    style: 'conversational, approachable, warm'
  },
  humorous: {
    systemPrompt: 'You are a witty content creator who makes learning fun.',
    temperature: 0.9,
    style: 'entertaining, light-hearted, engaging'
  }
};

function applyTone(tone: keyof typeof tonePresets, basePrompt: string) {
  const preset = tonePresets[tone];
  return `${preset.systemPrompt}\n\nStyle: ${preset.style}\n\n${basePrompt}`;
}
```

### Batch Processing

```typescript
async function batchGenerateContent(keywords: string[]) {
  const results = [];
  
  for (const keyword of keywords) {
    console.log(`Processing: ${keyword}`);
    
    const result = await runContentPipeline({
      keyword,
      contentFormat: 'toplist',
      languages: ['en'],
      videoFormats: ['reels'],
      tone: 'expert'
    });
    
    results.push({ keyword, ...result });
    
    // Rate limiting
    await new Promise(resolve => setTimeout(resolve, 2000));
  }
  
  return results;
}
```

## Troubleshooting

### API Key Issues

```typescript
// Validate API keys on startup
function validateEnvironment() {
  const required = ['ANTHROPIC_API_KEY', 'OPENAI_API_KEY', 'RAPIDAPI_KEY'];
  const missing = required.filter(key => !process.env[key]);
  
  if (missing.length > 0) {
    throw new Error(`Missing required environment variables: ${missing.join(', ')}`);
  }
}
```

### Rate Limiting

```typescript
import pLimit from 'p-limit';

const limit = pLimit(3); // Max 3 concurrent requests

async function generateMultipleContents(requests: ContentRequest[]) {
  return Promise.all(
    requests.map(request => 
      limit(() => generateContent(request))
    )
  );
}
```

### Video Rendering Errors

```typescript
async function safeRenderVideo(config: VideoConfig): Promise<string | null> {
  try {
    return await renderContentVideo(config);
  } catch (error) {
    console.error('Video rendering failed:', error);
    
    // Fallback: save content as text
    const fallbackPath = `./public/content/${Date.now()}.txt`;
    await fs.writeFile(fallbackPath, config.content);
    
    return null;
  }
}
```

### Scraping Timeouts

```typescript
async function scrapeWithRetry(
  keyword: string, 
  maxRetries = 3
): Promise<NewsSource[]> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await scrapeNews({
        keyword,
        sources: ['techcrunch'],
        timeRange: '24h',
        timeout: 10000
      });
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      
      console.log(`Retry ${i + 1}/${maxRetries}...`);
      await new Promise(resolve => setTimeout(resolve, 2000));
    }
  }
  
  return [];
}
```

## Performance Optimization

### Caching Research Results

```typescript
import NodeCache from 'node-cache';

const cache = new NodeCache({ stdTTL: 3600 }); // 1 hour cache

async function getCachedResearch(keyword: string): Promise<NewsSource[]> {
  const cacheKey = `research:${keyword}`;
  const cached = cache.get<NewsSource[]>(cacheKey);
  
  if (cached) {
    console.log('Using cached research');
    return cached;
  }
  
  const research = await scrapeNews({ keyword, sources: ['techcrunch'], timeRange: '24h' });
  cache.set(cacheKey, research);
  
  return research;
}
```

This skill provides comprehensive guidance for AI coding agents to help developers implement and customize the Marketing Pipeline Share AI Content automation system.
