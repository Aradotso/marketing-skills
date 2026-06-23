---
name: ultimate-ai-content-pipeline
description: Automated content pipeline from research to video generation using AI (Claude, OpenAI) and Remotion for Vietnamese/English marketing content
triggers:
  - how do I set up the AI content pipeline
  - generate automated content with research and video
  - create marketing content from keyword to video
  - use the content automation pipeline
  - configure Claude and OpenAI for content generation
  - render videos from blog posts automatically
  - automate content research and scriptwriting
  - build automated marketing content workflow
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

A comprehensive TypeScript-based content automation system that handles the entire content lifecycle: from real-time research (crawling TechCrunch, a16z, Twitter, LinkedIn), to AI-powered content generation in multiple formats and languages, to automatic video rendering with Remotion.

## What It Does

This pipeline automates 90% of content creation work by:
- **Auto-scanning** latest news from major sources (24h data)
- **Generating** multilingual content (Vietnamese/English) in various formats (toplist, POV, case study, how-to)
- **Rendering** videos and infographics automatically from written content
- **Optimizing** for multiple platforms (Reels, TikTok, Shorts)

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

Create a `.env.local` file in the root directory:

```bash
# AI Providers
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Next.js
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
│   │   ├── research/    # Content crawling & research
│   │   ├── video/       # Remotion video generation
│   │   └── utils/       # Helper functions
│   └── types/           # TypeScript types
├── public/              # Static assets
└── remotion/            # Video templates
```

## Core Modules

### 1. Research Module (Content Crawling)

```typescript
// lib/research/crawler.ts
import { AnthropicAI } from '../ai/anthropic';

interface ResearchOptions {
  keyword: string;
  sources: string[];
  timeframe: '24h' | '7d' | '30d';
  language: 'en' | 'vi';
}

export async function researchContent(options: ResearchOptions) {
  const { keyword, sources, timeframe, language } = options;
  
  // Crawl data from sources
  const rawData = await Promise.all(
    sources.map(source => crawlSource(source, keyword, timeframe))
  );
  
  // Use AI to extract insights
  const ai = new AnthropicAI(process.env.ANTHROPIC_API_KEY!);
  const insights = await ai.extractInsights({
    data: rawData,
    keyword,
    language
  });
  
  return {
    rawData,
    insights,
    sources: sources.length,
    timestamp: new Date().toISOString()
  };
}

async function crawlSource(source: string, keyword: string, timeframe: string) {
  // Implementation for different sources
  switch(source) {
    case 'techcrunch':
      return await crawlTechCrunch(keyword, timeframe);
    case 'a16z':
      return await crawlA16z(keyword, timeframe);
    case 'twitter':
      return await crawlTwitter(keyword, timeframe);
    default:
      throw new Error(`Unknown source: ${source}`);
  }
}
```

### 2. AI Content Generation

```typescript
// lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

interface ContentOptions {
  keyword: string;
  research: any;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
}

export class ContentGenerator {
  private claude: Anthropic;
  private openai: OpenAI;
  
  constructor() {
    this.claude = new Anthropic({
      apiKey: process.env.ANTHROPIC_API_KEY!
    });
    this.openai = new OpenAI({
      apiKey: process.env.OPENAI_API_KEY!
    });
  }
  
  async generate(options: ContentOptions) {
    const { keyword, research, format, language, tone } = options;
    
    const systemPrompt = this.buildSystemPrompt(format, language, tone);
    const userPrompt = this.buildUserPrompt(keyword, research);
    
    // Use Claude for Vietnamese content
    if (language === 'vi') {
      const response = await this.claude.messages.create({
        model: 'claude-3-sonnet-20240229',
        max_tokens: 4000,
        system: systemPrompt,
        messages: [{
          role: 'user',
          content: userPrompt
        }]
      });
      
      return this.parseContent(response.content[0].text);
    }
    
    // Use OpenAI for English content
    const response = await this.openai.chat.completions.create({
      model: 'gpt-4-turbo-preview',
      messages: [
        { role: 'system', content: systemPrompt },
        { role: 'user', content: userPrompt }
      ],
      temperature: 0.7,
      max_tokens: 4000
    });
    
    return this.parseContent(response.choices[0].message.content!);
  }
  
  private buildSystemPrompt(format: string, language: string, tone: string): string {
    const prompts = {
      'toplist': {
        'vi': `Bạn là chuyên gia viết content dạng Top List chuyên nghiệp. Viết theo phong cách ${tone}, đầy đủ số liệu và insight từ research.`,
        'en': `You are an expert content writer specializing in Top Lists. Write in a ${tone} tone with data-backed insights.`
      },
      'pov': {
        'vi': `Bạn là chuyên gia phân tích xu hướng. Viết góc nhìn (POV) sâu sắc theo phong cách ${tone}.`,
        'en': `You are a trend analyst. Write insightful POV content in a ${tone} style.`
      },
      'case-study': {
        'vi': `Bạn là chuyên gia phân tích case study. Viết chi tiết, có số liệu cụ thể theo phong cách ${tone}.`,
        'en': `You are a case study analyst. Write detailed, data-driven content in a ${tone} tone.`
      },
      'how-to': {
        'vi': `Bạn là chuyên gia viết hướng dẫn thực hành. Viết dễ hiểu, từng bước theo phong cách ${tone}.`,
        'en': `You are a how-to guide expert. Write clear, step-by-step content in a ${tone} style.`
      }
    };
    
    return prompts[format][language];
  }
  
  private buildUserPrompt(keyword: string, research: any): string {
    return `
Keyword: ${keyword}

Research Data:
${JSON.stringify(research.insights, null, 2)}

Yêu cầu:
1. Tạo tiêu đề hấp dẫn (H1)
2. Viết intro cuốn hút (150-200 từ)
3. Phát triển nội dung chính với heading rõ ràng
4. Kết luận và CTA
5. Sử dụng số liệu từ research
6. Tối ưu SEO tự nhiên
`;
  }
  
  private parseContent(content: string) {
    // Parse structured content
    return {
      title: this.extractTitle(content),
      intro: this.extractIntro(content),
      body: this.extractBody(content),
      conclusion: this.extractConclusion(content),
      metadata: {
        wordCount: content.split(' ').length,
        generatedAt: new Date().toISOString()
      }
    };
  }
  
  private extractTitle(content: string): string {
    const match = content.match(/^#\s+(.+)$/m);
    return match ? match[1].trim() : '';
  }
  
  private extractIntro(content: string): string {
    const match = content.match(/^#\s+.+\n\n(.+?)(?=\n##)/s);
    return match ? match[1].trim() : '';
  }
  
  private extractBody(content: string): string {
    const match = content.match(/##\s+.+/s);
    return match ? match[0] : '';
  }
  
  private extractConclusion(content: string): string {
    const match = content.match(/##\s+Kết luận\n\n(.+?)$/s) || 
                  content.match(/##\s+Conclusion\n\n(.+?)$/s);
    return match ? match[1].trim() : '';
  }
}
```

### 3. Video Generation with Remotion

```typescript
// lib/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoOptions {
  content: any;
  template: 'reels' | 'tiktok' | 'shorts';
  style: 'minimal' | 'dynamic' | 'professional';
}

export async function generateVideo(options: VideoOptions) {
  const { content, template, style } = options;
  
  // Bundle Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: (config) => config
  });
  
  // Get composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: getCompositionId(template),
    inputProps: {
      content,
      style
    }
  });
  
  // Render video
  const outputLocation = path.resolve(`./output/${Date.now()}-${template}.mp4`);
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: {
      content,
      style
    }
  });
  
  return {
    path: outputLocation,
    duration: composition.durationInFrames / composition.fps,
    resolution: {
      width: composition.width,
      height: composition.height
    }
  };
}

function getCompositionId(template: string): string {
  const compositions = {
    'reels': 'InstagramReel',
    'tiktok': 'TikTokVideo',
    'shorts': 'YouTubeShort'
  };
  return compositions[template];
}
```

### 4. Complete Pipeline Integration

```typescript
// lib/pipeline/orchestrator.ts
import { researchContent } from '../research/crawler';
import { ContentGenerator } from '../ai/content-generator';
import { generateVideo } from '../video/renderer';

interface PipelineOptions {
  keyword: string;
  sources?: string[];
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  languages: ('en' | 'vi')[];
  tone?: 'expert' | 'friendly' | 'humorous';
  generateVideo?: boolean;
  videoTemplate?: 'reels' | 'tiktok' | 'shorts';
}

export async function runContentPipeline(options: PipelineOptions) {
  const {
    keyword,
    sources = ['techcrunch', 'a16z', 'twitter'],
    format,
    languages,
    tone = 'expert',
    generateVideo = false,
    videoTemplate = 'reels'
  } = options;
  
  console.log(`🚀 Starting pipeline for keyword: ${keyword}`);
  
  // Step 1: Research
  console.log('📡 Step 1: Researching content...');
  const research = await researchContent({
    keyword,
    sources,
    timeframe: '24h',
    language: languages[0]
  });
  
  // Step 2: Generate content for each language
  console.log('🧠 Step 2: Generating content...');
  const generator = new ContentGenerator();
  const contents = await Promise.all(
    languages.map(language => 
      generator.generate({
        keyword,
        research,
        format,
        language,
        tone
      })
    )
  );
  
  const result: any = {
    research,
    contents: {}
  };
  
  languages.forEach((lang, index) => {
    result.contents[lang] = contents[index];
  });
  
  // Step 3: Generate video (optional)
  if (generateVideo) {
    console.log('🎬 Step 3: Rendering video...');
    result.video = await generateVideo({
      content: contents[0],
      template: videoTemplate,
      style: 'professional'
    });
  }
  
  console.log('✅ Pipeline completed!');
  return result;
}
```

## API Routes (Next.js)

```typescript
// app/api/pipeline/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline/orchestrator';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    
    const {
      keyword,
      format,
      languages = ['en', 'vi'],
      generateVideo = false
    } = body;
    
    if (!keyword || !format) {
      return NextResponse.json(
        { error: 'Missing required fields' },
        { status: 400 }
      );
    }
    
    const result = await runContentPipeline({
      keyword,
      format,
      languages,
      generateVideo
    });
    
    return NextResponse.json(result);
    
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { error: 'Pipeline execution failed' },
      { status: 500 }
    );
  }
}
```

## Usage Examples

### Example 1: Basic Content Generation

```typescript
import { runContentPipeline } from './lib/pipeline/orchestrator';

async function generateBlogPost() {
  const result = await runContentPipeline({
    keyword: 'AI Marketing Trends 2024',
    format: 'toplist',
    languages: ['en', 'vi'],
    tone: 'expert'
  });
  
  console.log('English version:', result.contents.en.title);
  console.log('Vietnamese version:', result.contents.vi.title);
}
```

### Example 2: Full Pipeline with Video

```typescript
async function generateFullContent() {
  const result = await runContentPipeline({
    keyword: 'Content Automation Tools',
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    format: 'case-study',
    languages: ['en'],
    tone: 'professional',
    generateVideo: true,
    videoTemplate: 'reels'
  });
  
  console.log('Research sources:', result.research.sources);
  console.log('Content generated:', result.contents.en.metadata.wordCount, 'words');
  console.log('Video location:', result.video.path);
}
```

### Example 3: React Component Usage

```tsx
// components/ContentGenerator.tsx
'use client';

import { useState } from 'react';

export function ContentGeneratorForm() {
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState<any>(null);
  
  async function handleSubmit(e: React.FormEvent<HTMLFormElement>) {
    e.preventDefault();
    setLoading(true);
    
    const formData = new FormData(e.currentTarget);
    
    const response = await fetch('/api/pipeline', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        keyword: formData.get('keyword'),
        format: formData.get('format'),
        languages: ['en', 'vi'],
        generateVideo: formData.get('video') === 'on'
      })
    });
    
    const data = await response.json();
    setResult(data);
    setLoading(false);
  }
  
  return (
    <div className="max-w-2xl mx-auto p-6">
      <form onSubmit={handleSubmit} className="space-y-4">
        <div>
          <label className="block mb-2">Keyword</label>
          <input
            type="text"
            name="keyword"
            className="w-full px-4 py-2 border rounded"
            required
          />
        </div>
        
        <div>
          <label className="block mb-2">Format</label>
          <select name="format" className="w-full px-4 py-2 border rounded">
            <option value="toplist">Top List</option>
            <option value="pov">POV</option>
            <option value="case-study">Case Study</option>
            <option value="how-to">How-to Guide</option>
          </select>
        </div>
        
        <div>
          <label className="flex items-center">
            <input type="checkbox" name="video" className="mr-2" />
            Generate Video
          </label>
        </div>
        
        <button
          type="submit"
          disabled={loading}
          className="w-full bg-blue-600 text-white py-2 rounded"
        >
          {loading ? 'Generating...' : 'Generate Content'}
        </button>
      </form>
      
      {result && (
        <div className="mt-8">
          <h2 className="text-2xl font-bold mb-4">Results</h2>
          <pre className="bg-gray-100 p-4 rounded overflow-auto">
            {JSON.stringify(result, null, 2)}
          </pre>
        </div>
      )}
    </div>
  );
}
```

## Development Commands

```bash
# Run development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Type checking
npm run type-check

# Lint code
npm run lint

# Build Remotion video
npm run remotion:build

# Preview Remotion compositions
npm run remotion:preview
```

## Common Patterns

### Pattern 1: Scheduled Content Generation

```typescript
// lib/scheduler/cron.ts
import cron from 'node-cron';
import { runContentPipeline } from '../pipeline/orchestrator';

export function scheduleContentGeneration() {
  // Run every day at 9 AM
  cron.schedule('0 9 * * *', async () => {
    const keywords = await getKeywordsFromDatabase();
    
    for (const keyword of keywords) {
      await runContentPipeline({
        keyword,
        format: 'toplist',
        languages: ['en', 'vi']
      });
    }
  });
}
```

### Pattern 2: Batch Processing

```typescript
async function batchGenerateContent(keywords: string[]) {
  const results = [];
  
  for (const keyword of keywords) {
    try {
      const result = await runContentPipeline({
        keyword,
        format: 'toplist',
        languages: ['en']
      });
      results.push({ keyword, success: true, data: result });
    } catch (error) {
      results.push({ keyword, success: false, error });
    }
  }
  
  return results;
}
```

## Troubleshooting

### Issue: API Rate Limits

```typescript
// lib/utils/rate-limiter.ts
import pLimit from 'p-limit';

const limit = pLimit(3); // Max 3 concurrent requests

export async function batchWithRateLimit<T>(
  items: T[],
  fn: (item: T) => Promise<any>
) {
  return Promise.all(
    items.map(item => limit(() => fn(item)))
  );
}
```

### Issue: Memory Errors During Video Rendering

```typescript
// Increase Node memory limit in package.json
{
  "scripts": {
    "dev": "NODE_OPTIONS='--max-old-space-size=4096' next dev",
    "build": "NODE_OPTIONS='--max-old-space-size=4096' next build"
  }
}
```

### Issue: Claude API Timeout

```typescript
const response = await this.claude.messages.create({
  model: 'claude-3-sonnet-20240229',
  max_tokens: 4000,
  timeout: 120000, // 2 minutes
  messages: [...]
});
```

### Issue: Missing Environment Variables

```typescript
// lib/utils/env.ts
export function validateEnv() {
  const required = [
    'ANTHROPIC_API_KEY',
    'OPENAI_API_KEY',
    'RAPIDAPI_KEY'
  ];
  
  const missing = required.filter(key => !process.env[key]);
  
  if (missing.length > 0) {
    throw new Error(`Missing environment variables: ${missing.join(', ')}`);
  }
}
```

## Performance Optimization

```typescript
// lib/cache/content-cache.ts
import NodeCache from 'node-cache';

const cache = new NodeCache({ stdTTL: 3600 }); // 1 hour

export async function getCachedResearch(keyword: string) {
  const cached = cache.get<any>(keyword);
  if (cached) return cached;
  
  const research = await researchContent({
    keyword,
    sources: ['techcrunch', 'a16z'],
    timeframe: '24h',
    language: 'en'
  });
  
  cache.set(keyword, research);
  return research;
}
```
