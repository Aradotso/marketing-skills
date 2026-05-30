---
name: ultimate-ai-content-pipeline
description: Automated content pipeline from research to script to video generation using Claude, OpenAI, and Remotion
triggers:
  - how do I automate content creation with AI
  - generate video content automatically from text
  - crawl news and create content with AI
  - set up automated marketing content pipeline
  - use Claude and OpenAI for content generation
  - create videos with Remotion from AI scripts
  - build an automated content research system
  - generate multilingual content automatically
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This project is a complete content automation pipeline that researches topics, generates scripts in multiple formats (toplist, POV, case study, how-to), and renders them into videos. It combines web scraping, AI content generation (Claude 3, OpenAI), and video rendering (Remotion) into a single workflow.

## What It Does

The Ultimate AI Content Pipeline automates the entire content creation process:

1. **Auto-Research**: Crawls recent articles from TechCrunch, a16z, Twitter/X, LinkedIn (last 24h)
2. **AI Script Generation**: Creates content in multiple formats using Claude/OpenAI
3. **Multilingual Output**: Generates both English and Vietnamese versions
4. **Video Rendering**: Converts text content into videos/infographics using Remotion
5. **Multi-Platform Optimization**: Exports videos optimized for Reels, TikTok, Shorts

## Installation

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
pnpm install
# or
yarn install
```

## Environment Configuration

Create a `.env.local` file in the root directory:

```env
# AI APIs
ANTHROPIC_API_KEY=your_claude_api_key_here
OPENAI_API_KEY=your_openai_api_key_here

# Research/Scraping APIs
RAPIDAPI_KEY=your_rapidapi_key_here

# Optional: Database/Storage
DATABASE_URL=your_database_url_here

# Next.js Configuration
NEXT_PUBLIC_API_URL=http://localhost:3000
```

## Running the Application

```bash
# Development mode
npm run dev

# Production build
npm run build
npm start

# Remotion video rendering
npm run remotion
```

Access the application at `http://localhost:3000`

## Project Structure

```
marketing-pineline-share/
├── app/                    # Next.js app directory
│   ├── api/               # API routes
│   │   ├── research/      # Research/crawling endpoints
│   │   ├── generate/      # Content generation endpoints
│   │   └── render/        # Video rendering endpoints
│   └── components/        # React components
├── lib/                   # Core libraries
│   ├── ai/               # AI integration (Claude, OpenAI)
│   ├── scraper/          # Web scraping utilities
│   └── remotion/         # Video rendering logic
├── remotion/             # Remotion video templates
└── public/               # Static assets
```

## Core API Endpoints

### Research Endpoint

```typescript
// app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';

export async function POST(request: NextRequest) {
  const { keyword, sources, timeframe } = await request.json();
  
  // Crawl multiple sources
  const results = await scrapeMultipleSources({
    keyword,
    sources: sources || ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeframe: timeframe || '24h'
  });
  
  return NextResponse.json({ data: results });
}
```

### Content Generation Endpoint

```typescript
// app/api/generate/route.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

export async function POST(request: NextRequest) {
  const { researchData, format, language, tone } = await request.json();
  
  // Generate content using Claude
  const message = await anthropic.messages.create({
    model: 'claude-3-opus-20240229',
    max_tokens: 4000,
    messages: [{
      role: 'user',
      content: buildPrompt(researchData, format, language, tone)
    }]
  });
  
  return NextResponse.json({ 
    content: message.content[0].text 
  });
}

function buildPrompt(data: any, format: string, language: string, tone: string) {
  return `Based on this research data: ${JSON.stringify(data)}
  
Create a ${format} article in ${language} with a ${tone} tone.
Include data-backed insights and recent trends.`;
}
```

## Client-Side Usage

### Basic Content Generation Flow

```typescript
// app/components/ContentGenerator.tsx
'use client';

import { useState } from 'react';

export function ContentGenerator() {
  const [keyword, setKeyword] = useState('');
  const [content, setContent] = useState(null);
  const [loading, setLoading] = useState(false);

  const generateContent = async () => {
    setLoading(true);
    
    // Step 1: Research
    const researchRes = await fetch('/api/research', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ 
        keyword,
        sources: ['techcrunch', 'a16z'],
        timeframe: '24h'
      })
    });
    const researchData = await researchRes.json();
    
    // Step 2: Generate Content
    const generateRes = await fetch('/api/generate', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        researchData: researchData.data,
        format: 'toplist',
        language: 'vietnamese',
        tone: 'professional'
      })
    });
    const generated = await generateRes.json();
    
    // Step 3: Render Video (optional)
    const videoRes = await fetch('/api/render', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        content: generated.content,
        platform: 'reels'
      })
    });
    const video = await videoRes.json();
    
    setContent({ text: generated.content, video: video.url });
    setLoading(false);
  };

  return (
    <div>
      <input 
        value={keyword}
        onChange={(e) => setKeyword(e.target.value)}
        placeholder="Enter keyword..."
      />
      <button onClick={generateContent} disabled={loading}>
        {loading ? 'Generating...' : 'Generate Content'}
      </button>
      {content && (
        <div>
          <div>{content.text}</div>
          <video src={content.video} controls />
        </div>
      )}
    </div>
  );
}
```

## Web Scraping Implementation

```typescript
// lib/scraper/index.ts
import axios from 'axios';

export interface ScrapeOptions {
  keyword: string;
  sources: string[];
  timeframe: string;
}

export async function scrapeMultipleSources(options: ScrapeOptions) {
  const { keyword, sources, timeframe } = options;
  const results = await Promise.all(
    sources.map(source => scrapeSingleSource(source, keyword, timeframe))
  );
  
  return results.flat();
}

async function scrapeSingleSource(source: string, keyword: string, timeframe: string) {
  // Use RapidAPI or custom scraper
  const response = await axios.get(`https://api.rapidapi.com/${source}/search`, {
    params: { q: keyword, time: timeframe },
    headers: {
      'X-RapidAPI-Key': process.env.RAPIDAPI_KEY!,
      'X-RapidAPI-Host': `${source}.p.rapidapi.com`
    }
  });
  
  return response.data.articles.map((article: any) => ({
    title: article.title,
    url: article.url,
    summary: article.description,
    publishedAt: article.publishedAt,
    source
  }));
}
```

## AI Content Generation Patterns

### Using Claude for Long-Form Content

```typescript
// lib/ai/claude.ts
import Anthropic from '@anthropic-ai/sdk';

const client = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

export async function generateWithClaude(
  prompt: string,
  systemPrompt?: string
) {
  const message = await client.messages.create({
    model: 'claude-3-opus-20240229',
    max_tokens: 4000,
    system: systemPrompt || 'You are an expert content creator.',
    messages: [{ role: 'user', content: prompt }]
  });
  
  return message.content[0].type === 'text' 
    ? message.content[0].text 
    : '';
}

export async function generateMultiFormat(
  researchData: any[],
  formats: string[]
) {
  const results = await Promise.all(
    formats.map(format => 
      generateWithClaude(
        buildFormatPrompt(researchData, format),
        getSystemPromptForFormat(format)
      )
    )
  );
  
  return formats.reduce((acc, format, idx) => ({
    ...acc,
    [format]: results[idx]
  }), {});
}

function buildFormatPrompt(data: any[], format: string): string {
  const dataContext = data.map(d => 
    `- ${d.title} (${d.source}): ${d.summary}`
  ).join('\n');
  
  switch(format) {
    case 'toplist':
      return `Create a top 10 list based on:\n${dataContext}\n\nMake it engaging with data points.`;
    case 'pov':
      return `Write a POV article analyzing:\n${dataContext}\n\nInclude unique perspective.`;
    case 'case-study':
      return `Create a case study from:\n${dataContext}\n\nInclude metrics and outcomes.`;
    case 'how-to':
      return `Write a how-to guide based on:\n${dataContext}\n\nMake it actionable.`;
    default:
      return `Write about:\n${dataContext}`;
  }
}
```

### Using OpenAI for Quick Generation

```typescript
// lib/ai/openai.ts
import OpenAI from 'openai';

const client = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

export async function generateWithOpenAI(
  prompt: string,
  model: string = 'gpt-4-turbo-preview'
) {
  const completion = await client.chat.completions.create({
    model,
    messages: [{ role: 'user', content: prompt }],
    temperature: 0.7,
    max_tokens: 2000
  });
  
  return completion.choices[0].message.content || '';
}

export async function generateBilingual(researchData: any[]) {
  const [english, vietnamese] = await Promise.all([
    generateWithOpenAI(
      `Write in English:\n${JSON.stringify(researchData)}`
    ),
    generateWithOpenAI(
      `Viết bằng tiếng Việt:\n${JSON.stringify(researchData)}`
    )
  ]);
  
  return { english, vietnamese };
}
```

## Remotion Video Rendering

### Basic Remotion Composition

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig } from 'remotion';

export const ContentVideo: React.FC<{
  title: string;
  points: string[];
}> = ({ title, points }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  
  const opacity = Math.min(1, frame / (fps * 0.5));
  
  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a1a' }}>
      <div style={{ opacity, padding: 60 }}>
        <h1 style={{ color: '#fff', fontSize: 72 }}>{title}</h1>
        <ul>
          {points.map((point, idx) => (
            <li 
              key={idx}
              style={{
                color: '#fff',
                fontSize: 32,
                marginTop: 20,
                opacity: frame > (idx + 1) * fps ? 1 : 0
              }}
            >
              {point}
            </li>
          ))}
        </ul>
      </div>
    </AbsoluteFill>
  );
};
```

### Render Video API

```typescript
// app/api/render/route.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

export async function POST(request: NextRequest) {
  const { content, platform } = await request.json();
  
  // Parse content into video props
  const props = parseContentForVideo(content);
  
  // Bundle Remotion project
  const bundleLocation = await bundle(
    path.join(process.cwd(), 'remotion', 'index.ts')
  );
  
  // Get composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: props,
  });
  
  // Render video
  const outputLocation = path.join(
    process.cwd(), 
    'public', 
    'videos', 
    `${Date.now()}.mp4`
  );
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
  });
  
  return NextResponse.json({ 
    url: `/videos/${path.basename(outputLocation)}` 
  });
}

function parseContentForVideo(content: string) {
  const lines = content.split('\n').filter(l => l.trim());
  return {
    title: lines[0],
    points: lines.slice(1, 11) // Top 10 points
  };
}
```

## Common Workflows

### Complete End-to-End Pipeline

```typescript
// lib/pipeline/index.ts
export async function runCompletePipeline(keyword: string) {
  // 1. Research
  console.log('Starting research...');
  const researchData = await scrapeMultipleSources({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter'],
    timeframe: '24h'
  });
  
  // 2. Generate Content
  console.log('Generating content...');
  const content = await generateMultiFormat(researchData, [
    'toplist',
    'pov',
    'how-to'
  ]);
  
  // 3. Create Bilingual Versions
  console.log('Creating bilingual versions...');
  const bilingual = await generateBilingual(researchData);
  
  // 4. Render Videos
  console.log('Rendering videos...');
  const videos = await Promise.all(
    Object.entries(content).map(([format, text]) =>
      renderVideo({ content: text, platform: 'reels' })
    )
  );
  
  return {
    research: researchData,
    content,
    bilingual,
    videos
  };
}
```

## Troubleshooting

### API Rate Limits

```typescript
// lib/utils/ratelimit.ts
export class RateLimiter {
  private queue: Array<() => Promise<any>> = [];
  private processing = false;
  private delay: number;
  
  constructor(requestsPerMinute: number) {
    this.delay = 60000 / requestsPerMinute;
  }
  
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
    const fn = this.queue.shift()!;
    await fn();
    await new Promise(resolve => setTimeout(resolve, this.delay));
    this.processing = false;
    this.process();
  }
}

// Usage
const limiter = new RateLimiter(10); // 10 requests per minute
await limiter.add(() => generateWithClaude(prompt));
```

### Error Handling for AI APIs

```typescript
// lib/utils/retry.ts
export async function retryWithBackoff<T>(
  fn: () => Promise<T>,
  maxRetries: number = 3,
  baseDelay: number = 1000
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error: any) {
      if (i === maxRetries - 1) throw error;
      
      const delay = baseDelay * Math.pow(2, i);
      console.log(`Retry ${i + 1}/${maxRetries} after ${delay}ms`);
      await new Promise(resolve => setTimeout(resolve, delay));
    }
  }
  throw new Error('Max retries exceeded');
}

// Usage
const result = await retryWithBackoff(() => 
  generateWithClaude(prompt)
);
```

### Memory Management for Video Rendering

```typescript
// Render videos in batches to avoid memory issues
async function renderVideosInBatches(contents: string[], batchSize: number = 3) {
  const results = [];
  
  for (let i = 0; i < contents.length; i += batchSize) {
    const batch = contents.slice(i, i + batchSize);
    const rendered = await Promise.all(
      batch.map(content => renderVideo({ content, platform: 'reels' }))
    );
    results.push(...rendered);
    
    // Clear memory between batches
    if (global.gc) global.gc();
  }
  
  return results;
}
```

## Best Practices

1. **Always validate API keys** before starting pipeline
2. **Cache research results** to avoid redundant scraping
3. **Use streaming responses** for long-running AI generations
4. **Implement proper error boundaries** in React components
5. **Monitor API costs** - both AI and scraping APIs have usage limits
6. **Optimize video rendering** - use lower quality for previews
7. **Store generated content** in a database for reuse
