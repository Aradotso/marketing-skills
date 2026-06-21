---
name: ultimate-ai-content-pipeline
description: Automated content creation pipeline from research to video generation using Claude/OpenAI and Remotion
triggers:
  - how do I generate automated content with AI research
  - set up content pipeline with video generation
  - automate content creation from research to posting
  - generate social media videos from articles
  - create content pipeline with Claude and Remotion
  - build automated marketing content workflow
  - scrape news and generate videos automatically
  - use AI content pipeline for social media
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

Ultimate AI Content Pipeline is a TypeScript-based automation system that transforms keywords into complete content packages including research, written articles, and rendered videos. It crawls news sources (TechCrunch, a16z, Twitter, LinkedIn), generates multi-format content using Claude/OpenAI, and creates videos using Remotion.

**Key Features:**
- Auto-crawls breaking news from major sources (24h data)
- Generates multiple content formats (listicles, POV, case studies, how-tos)
- Bilingual support (English/Vietnamese)
- Automatic video rendering for social media (Reels, TikTok, Shorts)
- Next.js web interface for content management

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
cp .env.example .env.local
```

### Required Environment Variables

```bash
# AI Services
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key
TWITTER_API_KEY=your_twitter_key

# Database (if applicable)
DATABASE_URL=your_database_url

# Remotion (for video generation)
REMOTION_LICENSE_KEY=your_remotion_key
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/                 # Next.js app router
│   ├── components/          # React components
│   ├── lib/
│   │   ├── ai/             # AI service integrations
│   │   ├── research/       # Web scraping modules
│   │   └── video/          # Remotion video generation
│   ├── types/              # TypeScript type definitions
│   └── utils/              # Helper functions
├── public/                 # Static assets
└── remotion/              # Video templates
```

## Core Modules

### 1. Research Automation

```typescript
// src/lib/research/newsScanner.ts
import { scrapeSource } from './scrapers';

interface ResearchResult {
  title: string;
  source: string;
  url: string;
  content: string;
  publishedAt: Date;
  insights: string[];
}

async function scanLatestNews(keyword: string, hours: number = 24): Promise<ResearchResult[]> {
  const sources = ['techcrunch', 'a16z', 'twitter', 'linkedin'];
  const results: ResearchResult[] = [];
  
  for (const source of sources) {
    try {
      const articles = await scrapeSource(source, keyword, hours);
      results.push(...articles);
    } catch (error) {
      console.error(`Failed to scrape ${source}:`, error);
    }
  }
  
  return results.sort((a, b) => 
    b.publishedAt.getTime() - a.publishedAt.getTime()
  );
}

export { scanLatestNews };
```

### 2. AI Content Generation

```typescript
// src/lib/ai/contentGenerator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

interface ContentRequest {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  research: ResearchResult[];
}

async function generateContent(request: ContentRequest): Promise<string> {
  const researchContext = request.research
    .map(r => `- ${r.title} (${r.source}): ${r.insights.join(', ')}`)
    .join('\n');

  const prompt = `Create a ${request.format} article about "${request.keyword}" in ${request.language} with a ${request.tone} tone.

Research context (last 24h):
${researchContext}

Requirements:
- Use real data and insights from the research
- Include specific examples and statistics
- Make it engaging and actionable
- Length: 800-1200 words
`;

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4000,
    messages: [{
      role: 'user',
      content: prompt
    }]
  });

  return message.content[0].type === 'text' 
    ? message.content[0].text 
    : '';
}

export { generateContent };
```

### 3. Video Generation with Remotion

```typescript
// src/lib/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  title: string;
  keyPoints: string[];
  backgroundColor: string;
  textColor: string;
}

async function renderContentVideo(
  config: VideoConfig,
  outputPath: string
): Promise<string> {
  const compositionId = 'ContentVideo';
  
  // Bundle the Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  // Get composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: config,
  });

  // Render video
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: config,
  });

  return outputPath;
}

export { renderContentVideo };
```

### 4. Complete Pipeline Workflow

```typescript
// src/lib/pipeline/orchestrator.ts
import { scanLatestNews } from '../research/newsScanner';
import { generateContent } from '../ai/contentGenerator';
import { renderContentVideo } from '../video/renderer';
import { extractKeyPoints } from '../utils/textAnalysis';

interface PipelineInput {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  languages: ('en' | 'vi')[];
  generateVideo: boolean;
}

interface PipelineOutput {
  research: ResearchResult[];
  articles: Record<string, string>;
  videos?: Record<string, string>;
}

async function runContentPipeline(
  input: PipelineInput
): Promise<PipelineOutput> {
  // Step 1: Research
  console.log('🔍 Scanning latest news...');
  const research = await scanLatestNews(input.keyword, 24);
  
  if (research.length === 0) {
    throw new Error('No research data found for keyword');
  }

  // Step 2: Generate articles
  console.log('✍️ Generating content...');
  const articles: Record<string, string> = {};
  
  for (const language of input.languages) {
    const content = await generateContent({
      keyword: input.keyword,
      format: input.format,
      language,
      tone: 'expert',
      research,
    });
    articles[language] = content;
  }

  // Step 3: Generate videos (optional)
  let videos: Record<string, string> | undefined;
  
  if (input.generateVideo) {
    console.log('🎬 Rendering videos...');
    videos = {};
    
    for (const [lang, content] of Object.entries(articles)) {
      const keyPoints = extractKeyPoints(content);
      const outputPath = `./output/video-${input.keyword}-${lang}.mp4`;
      
      await renderContentVideo({
        title: input.keyword,
        keyPoints: keyPoints.slice(0, 5),
        backgroundColor: '#1a1a1a',
        textColor: '#ffffff',
      }, outputPath);
      
      videos[lang] = outputPath;
    }
  }

  return { research, articles, videos };
}

export { runContentPipeline };
```

## API Routes (Next.js)

```typescript
// src/app/api/pipeline/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline/orchestrator';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    
    const result = await runContentPipeline({
      keyword: body.keyword,
      format: body.format || 'toplist',
      languages: body.languages || ['en'],
      generateVideo: body.generateVideo || false,
    });

    return NextResponse.json({
      success: true,
      data: result,
    });
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

## Client-Side Usage

```typescript
// src/components/ContentPipelineForm.tsx
'use client';

import { useState } from 'react';

export function ContentPipelineForm() {
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState(null);

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    setLoading(true);

    const formData = new FormData(e.target as HTMLFormElement);
    
    const response = await fetch('/api/pipeline', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        keyword: formData.get('keyword'),
        format: formData.get('format'),
        languages: ['en', 'vi'],
        generateVideo: formData.get('generateVideo') === 'on',
      }),
    });

    const data = await response.json();
    setResult(data);
    setLoading(false);
  };

  return (
    <form onSubmit={handleSubmit}>
      <input name="keyword" placeholder="Enter keyword" required />
      
      <select name="format">
        <option value="toplist">Top List</option>
        <option value="pov">POV</option>
        <option value="case-study">Case Study</option>
        <option value="how-to">How-to</option>
      </select>
      
      <label>
        <input type="checkbox" name="generateVideo" />
        Generate video
      </label>
      
      <button type="submit" disabled={loading}>
        {loading ? 'Processing...' : 'Generate Content'}
      </button>

      {result && (
        <div>
          <h3>Results</h3>
          <pre>{JSON.stringify(result, null, 2)}</pre>
        </div>
      )}
    </form>
  );
}
```

## Common Patterns

### Batch Processing Multiple Keywords

```typescript
async function batchGenerateContent(keywords: string[]) {
  const results = [];
  
  for (const keyword of keywords) {
    try {
      const result = await runContentPipeline({
        keyword,
        format: 'toplist',
        languages: ['en'],
        generateVideo: true,
      });
      
      results.push({ keyword, success: true, data: result });
      
      // Rate limiting
      await new Promise(resolve => setTimeout(resolve, 5000));
    } catch (error) {
      results.push({ keyword, success: false, error: error.message });
    }
  }
  
  return results;
}
```

### Custom Research Sources

```typescript
// src/lib/research/customSource.ts
interface CustomSourceConfig {
  url: string;
  selector: string;
  titleSelector: string;
  contentSelector: string;
}

async function addCustomSource(config: CustomSourceConfig) {
  // Implement custom scraping logic
  // Use cheerio or puppeteer based on source complexity
}
```

### Video Template Customization

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, Sequence, useCurrentFrame } from 'remotion';

export const ContentVideo: React.FC<VideoConfig> = ({
  title,
  keyPoints,
  backgroundColor,
  textColor,
}) => {
  const frame = useCurrentFrame();
  
  return (
    <AbsoluteFill style={{ backgroundColor }}>
      <Sequence from={0} durationInFrames={60}>
        <h1 style={{ color: textColor }}>{title}</h1>
      </Sequence>
      
      {keyPoints.map((point, i) => (
        <Sequence key={i} from={60 + i * 90} durationInFrames={90}>
          <div style={{ color: textColor }}>{point}</div>
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};
```

## Development Commands

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Render video preview
npm run remotion:preview

# Render specific video
npm run remotion:render
```

## Troubleshooting

### Research Module Returns Empty Results

```typescript
// Check API rate limits and add retries
async function scrapeWithRetry(source: string, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await scrapeSource(source);
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      await new Promise(r => setTimeout(r, 2000 * (i + 1)));
    }
  }
}
```

### AI Generation Timeout

```typescript
// Increase timeout and add fallback
const controller = new AbortController();
const timeoutId = setTimeout(() => controller.abort(), 60000);

try {
  const content = await generateContent(request);
  clearTimeout(timeoutId);
  return content;
} catch (error) {
  if (error.name === 'AbortError') {
    // Use shorter prompt or switch to GPT-3.5
    return await generateContentFallback(request);
  }
  throw error;
}
```

### Video Rendering Fails

```typescript
// Check Remotion configuration and dependencies
import { getCompositions } from '@remotion/renderer';

async function debugVideoSetup() {
  try {
    const compositions = await getCompositions(bundleLocation);
    console.log('Available compositions:', compositions);
  } catch (error) {
    console.error('Remotion setup error:', error);
  }
}
```

### Memory Issues with Large Batches

```typescript
// Use queue system with concurrency control
import PQueue from 'p-queue';

const queue = new PQueue({ concurrency: 2 });

async function processWithQueue(keywords: string[]) {
  return Promise.all(
    keywords.map(keyword => 
      queue.add(() => runContentPipeline({ keyword, /* ... */ }))
    )
  );
}
```

## Best Practices

1. **Cache research results** to avoid duplicate API calls
2. **Use webhook notifications** for long-running video renders
3. **Implement content versioning** to track iterations
4. **Add content approval workflow** before auto-posting
5. **Monitor API usage** to stay within rate limits
6. **Store generated assets** in cloud storage (S3, Cloudinary)
