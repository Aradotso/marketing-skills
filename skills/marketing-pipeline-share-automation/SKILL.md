---
name: marketing-pipeline-share-automation
description: Automate content creation from research to video generation using AI-powered pipeline with Claude, OpenAI, and Remotion
triggers:
  - how do I set up the AI content pipeline
  - create automated content with research and video generation
  - use marketing pipeline share for content automation
  - generate content from keyword to video automatically
  - configure Claude and OpenAI for content generation
  - build automated marketing content workflow
  - render videos from AI generated content
  - setup content research and script generation pipeline
---

# Marketing Pipeline Share Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill helps you use the **Ultimate AI Content Pipeline** - an end-to-end content automation system that handles research, script generation, and video rendering. The pipeline crawls real-time data from sources like TechCrunch, a16z, Twitter/X, and LinkedIn, then uses AI (Claude 3, OpenAI) to generate content in multiple formats and languages, finally rendering videos with Remotion.

## What It Does

The marketing-pipeline-share system automates:

1. **Auto-Scan Research**: Crawls news from major tech sources in the last 24 hours
2. **AI Content Generation**: Creates content in multiple formats (toplist, POV, case study, how-to) using Claude/OpenAI
3. **Multi-language Support**: Generates parallel Vietnamese and English versions
4. **Video Rendering**: Converts content to infographics and short-form videos using Remotion
5. **Platform Optimization**: Outputs video formats for Reels, TikTok, and Shorts

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
# AI APIs
ANTHROPIC_API_KEY=your_claude_api_key_here
OPENAI_API_KEY=your_openai_api_key_here

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key_here

# Remotion (for video rendering)
REMOTION_LICENSE_KEY=your_remotion_license_key_here

# Next.js
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

## Key API Patterns

### 1. Research Data Crawling

```typescript
// src/lib/research/crawler.ts
import { fetchNewsFromSources } from '@/lib/research/sources';

interface ResearchConfig {
  keyword: string;
  sources: string[];
  timeRange: '24h' | '7d' | '30d';
  maxResults: number;
}

async function crawlResearchData(config: ResearchConfig) {
  const { keyword, sources, timeRange, maxResults } = config;
  
  const results = await fetchNewsFromSources({
    query: keyword,
    sources: sources,
    publishedAfter: getTimeRangeDate(timeRange),
    limit: maxResults
  });
  
  return results.map(article => ({
    title: article.title,
    url: article.url,
    source: article.source,
    publishedAt: article.publishedAt,
    summary: article.description,
    insights: extractInsights(article.content)
  }));
}

// Usage
const researchData = await crawlResearchData({
  keyword: 'AI automation',
  sources: ['techcrunch', 'a16z', 'twitter'],
  timeRange: '24h',
  maxResults: 10
});
```

### 2. Content Generation with Claude

```typescript
// src/lib/ai/claude-generator.ts
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

interface ContentGenerationParams {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  tone: 'expert' | 'friendly' | 'humorous';
  language: 'vi' | 'en';
  researchData: any[];
}

async function generateContentWithClaude(params: ContentGenerationParams) {
  const { keyword, format, tone, language, researchData } = params;
  
  const systemPrompt = buildSystemPrompt(format, tone, language);
  const userPrompt = buildUserPrompt(keyword, researchData);
  
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    temperature: 0.7,
    system: systemPrompt,
    messages: [{
      role: 'user',
      content: userPrompt
    }]
  });
  
  return {
    content: message.content[0].text,
    metadata: {
      format,
      tone,
      language,
      generatedAt: new Date().toISOString()
    }
  };
}

// Example system prompt builder
function buildSystemPrompt(format: string, tone: string, language: string) {
  const toneMap = {
    expert: 'chuyên sâu, dựa trên dữ liệu và nghiên cứu',
    friendly: 'thân thiện, dễ hiểu, gần gũi',
    humorous: 'hài hước, sáng tạo nhưng vẫn giữ độ chuyên nghiệp'
  };
  
  return `Bạn là một chuyên gia content marketing. Nhiệm vụ của bạn là tạo nội dung ${format} với giọng văn ${toneMap[tone]} bằng ${language === 'vi' ? 'tiếng Việt' : 'English'}. Sử dụng dữ liệu nghiên cứu được cung cấp để tạo nội dung chính xác và cập nhật.`;
}
```

### 3. Multi-Language Content Generation

```typescript
// src/lib/ai/multi-language.ts
async function generateBilingualContent(
  keyword: string,
  researchData: any[],
  format: string,
  tone: string
) {
  const [vietnameseContent, englishContent] = await Promise.all([
    generateContentWithClaude({
      keyword,
      format,
      tone,
      language: 'vi',
      researchData
    }),
    generateContentWithClaude({
      keyword,
      format,
      tone,
      language: 'en',
      researchData
    })
  ]);
  
  return {
    vi: vietnameseContent,
    en: englishContent
  };
}
```

### 4. OpenAI Integration (Alternative)

```typescript
// src/lib/ai/openai-generator.ts
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateContentWithOpenAI(params: ContentGenerationParams) {
  const systemPrompt = buildSystemPrompt(params.format, params.tone, params.language);
  const userPrompt = buildUserPrompt(params.keyword, params.researchData);
  
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      { role: 'system', content: systemPrompt },
      { role: 'user', content: userPrompt }
    ],
    temperature: 0.7,
    max_tokens: 4096
  });
  
  return {
    content: completion.choices[0].message.content,
    metadata: {
      format: params.format,
      tone: params.tone,
      language: params.language,
      model: completion.model,
      generatedAt: new Date().toISOString()
    }
  };
}
```

### 5. Video Rendering with Remotion

```typescript
// src/remotion/compositions.tsx
import { Composition } from 'remotion';
import { ContentVideo } from './ContentVideo';

export const RemotionRoot = () => {
  return (
    <>
      <Composition
        id="ContentVideo"
        component={ContentVideo}
        durationInFrames={300}
        fps={30}
        width={1080}
        height={1920}
        defaultProps={{
          title: 'Your Title Here',
          content: 'Your content here',
          format: 'toplist'
        }}
      />
    </>
  );
};
```

```typescript
// src/remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig } from 'remotion';

interface ContentVideoProps {
  title: string;
  content: string;
  format: string;
}

export const ContentVideo: React.FC<ContentVideoProps> = ({
  title,
  content,
  format
}) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  
  const opacity = Math.min(1, frame / 30);
  
  return (
    <AbsoluteFill
      style={{
        backgroundColor: '#1a1a1a',
        justifyContent: 'center',
        alignItems: 'center',
        opacity
      }}
    >
      <div style={{ padding: 60, maxWidth: 900 }}>
        <h1 style={{ color: 'white', fontSize: 72, marginBottom: 40 }}>
          {title}
        </h1>
        <p style={{ color: '#e0e0e0', fontSize: 36, lineHeight: 1.6 }}>
          {content}
        </p>
      </div>
    </AbsoluteFill>
  );
};
```

### 6. Render Video Programmatically

```typescript
// src/lib/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface RenderVideoParams {
  title: string;
  content: string;
  format: string;
  outputPath: string;
}

async function renderContentVideo(params: RenderVideoParams) {
  const { title, content, format, outputPath } = params;
  
  // Bundle Remotion project
  const bundleLocation = await bundle(
    path.join(process.cwd(), 'src/remotion/index.ts')
  );
  
  // Select composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: { title, content, format }
  });
  
  // Render video
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: { title, content, format }
  });
  
  return outputPath;
}

// Usage
const videoPath = await renderContentVideo({
  title: 'Top 5 AI Trends in 2024',
  content: 'Generated content here...',
  format: 'toplist',
  outputPath: './output/video.mp4'
});
```

## Complete Pipeline Example

```typescript
// src/lib/pipeline/full-automation.ts
import { crawlResearchData } from '@/lib/research/crawler';
import { generateBilingualContent } from '@/lib/ai/multi-language';
import { renderContentVideo } from '@/lib/video/renderer';

interface PipelineConfig {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  tone: 'expert' | 'friendly' | 'humorous';
  sources: string[];
  generateVideo: boolean;
}

async function runFullPipeline(config: PipelineConfig) {
  console.log(`🚀 Starting pipeline for keyword: ${config.keyword}`);
  
  // Step 1: Research
  console.log('📡 Crawling research data...');
  const researchData = await crawlResearchData({
    keyword: config.keyword,
    sources: config.sources,
    timeRange: '24h',
    maxResults: 10
  });
  
  // Step 2: Generate content
  console.log('🧠 Generating bilingual content...');
  const content = await generateBilingualContent(
    config.keyword,
    researchData,
    config.format,
    config.tone
  );
  
  // Step 3: Render video (optional)
  let videoPath = null;
  if (config.generateVideo) {
    console.log('🎬 Rendering video...');
    videoPath = await renderContentVideo({
      title: content.vi.metadata.title || config.keyword,
      content: content.vi.content.substring(0, 500),
      format: config.format,
      outputPath: `./output/${config.keyword.replace(/\s+/g, '-')}.mp4`
    });
  }
  
  return {
    research: researchData,
    content: {
      vietnamese: content.vi,
      english: content.en
    },
    video: videoPath,
    completedAt: new Date().toISOString()
  };
}

// Usage
const result = await runFullPipeline({
  keyword: 'AI Content Automation',
  format: 'toplist',
  tone: 'expert',
  sources: ['techcrunch', 'a16z', 'twitter'],
  generateVideo: true
});

console.log('✅ Pipeline completed:', result);
```

## Next.js API Route Example

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runFullPipeline } from '@/lib/pipeline/full-automation';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    
    const { keyword, format, tone, sources, generateVideo } = body;
    
    // Validate inputs
    if (!keyword || !format || !tone) {
      return NextResponse.json(
        { error: 'Missing required fields' },
        { status: 400 }
      );
    }
    
    // Run pipeline
    const result = await runFullPipeline({
      keyword,
      format,
      tone,
      sources: sources || ['techcrunch', 'a16z'],
      generateVideo: generateVideo ?? false
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

## Frontend Integration

```typescript
// src/components/ContentGenerator.tsx
'use client';

import { useState } from 'react';

export default function ContentGenerator() {
  const [keyword, setKeyword] = useState('');
  const [format, setFormat] = useState('toplist');
  const [tone, setTone] = useState('expert');
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState(null);
  
  const handleGenerate = async () => {
    setLoading(true);
    
    try {
      const response = await fetch('/api/generate', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          keyword,
          format,
          tone,
          sources: ['techcrunch', 'a16z', 'twitter'],
          generateVideo: true
        })
      });
      
      const data = await response.json();
      setResult(data);
    } catch (error) {
      console.error('Generation error:', error);
    } finally {
      setLoading(false);
    }
  };
  
  return (
    <div className="p-8">
      <h1 className="text-3xl font-bold mb-6">AI Content Generator</h1>
      
      <div className="space-y-4 mb-6">
        <input
          type="text"
          placeholder="Enter keyword..."
          value={keyword}
          onChange={(e) => setKeyword(e.target.value)}
          className="w-full p-3 border rounded"
        />
        
        <select
          value={format}
          onChange={(e) => setFormat(e.target.value)}
          className="w-full p-3 border rounded"
        >
          <option value="toplist">Top List</option>
          <option value="pov">POV</option>
          <option value="case-study">Case Study</option>
          <option value="how-to">How-to</option>
        </select>
        
        <select
          value={tone}
          onChange={(e) => setTone(e.target.value)}
          className="w-full p-3 border rounded"
        >
          <option value="expert">Expert</option>
          <option value="friendly">Friendly</option>
          <option value="humorous">Humorous</option>
        </select>
        
        <button
          onClick={handleGenerate}
          disabled={loading || !keyword}
          className="w-full bg-blue-600 text-white p-3 rounded disabled:opacity-50"
        >
          {loading ? 'Generating...' : 'Generate Content'}
        </button>
      </div>
      
      {result && (
        <div className="mt-8 space-y-4">
          <h2 className="text-2xl font-bold">Results</h2>
          <div className="p-4 bg-gray-100 rounded">
            <h3 className="font-bold mb-2">Vietnamese Content:</h3>
            <p className="whitespace-pre-wrap">{result.content.vietnamese.content}</p>
          </div>
          <div className="p-4 bg-gray-100 rounded">
            <h3 className="font-bold mb-2">English Content:</h3>
            <p className="whitespace-pre-wrap">{result.content.english.content}</p>
          </div>
          {result.video && (
            <div className="p-4 bg-gray-100 rounded">
              <h3 className="font-bold mb-2">Video:</h3>
              <p>Generated at: {result.video}</p>
            </div>
          )}
        </div>
      )}
    </div>
  );
}
```

## Common Patterns

### Rate Limiting for API Calls

```typescript
// src/lib/utils/rate-limiter.ts
class RateLimiter {
  private queue: Array<() => Promise<any>> = [];
  private processing = false;
  private delayMs: number;
  
  constructor(delayMs: number = 1000) {
    this.delayMs = delayMs;
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
    await new Promise(resolve => setTimeout(resolve, this.delayMs));
    
    this.processing = false;
    this.process();
  }
}

export const aiRateLimiter = new RateLimiter(2000); // 2s between calls
```

### Caching Research Data

```typescript
// src/lib/cache/research-cache.ts
import fs from 'fs/promises';
import path from 'path';

const CACHE_DIR = path.join(process.cwd(), '.cache');

async function getCachedResearch(keyword: string, maxAge: number = 3600000) {
  const cacheFile = path.join(CACHE_DIR, `${keyword}.json`);
  
  try {
    const stat = await fs.stat(cacheFile);
    const age = Date.now() - stat.mtimeMs;
    
    if (age < maxAge) {
      const data = await fs.readFile(cacheFile, 'utf-8');
      return JSON.parse(data);
    }
  } catch (error) {
    // Cache miss
  }
  
  return null;
}

async function setCachedResearch(keyword: string, data: any) {
  await fs.mkdir(CACHE_DIR, { recursive: true });
  const cacheFile = path.join(CACHE_DIR, `${keyword}.json`);
  await fs.writeFile(cacheFile, JSON.stringify(data, null, 2));
}
```

## Troubleshooting

### API Key Issues

```typescript
// Validate API keys on startup
function validateEnvVars() {
  const required = [
    'ANTHROPIC_API_KEY',
    'OPENAI_API_KEY',
    'RAPIDAPI_KEY'
  ];
  
  const missing = required.filter(key => !process.env[key]);
  
  if (missing.length > 0) {
    throw new Error(`Missing required environment variables: ${missing.join(', ')}`);
  }
}
```

### Video Rendering Timeouts

```typescript
// Increase timeout for long videos
await renderMedia({
  composition,
  serveUrl: bundleLocation,
  codec: 'h264',
  outputLocation: outputPath,
  inputProps: { title, content, format },
  timeoutInMilliseconds: 120000 // 2 minutes
});
```

### Memory Issues with Large Content

```typescript
// Process content in chunks
function chunkContent(content: string, maxLength: number = 2000) {
  const chunks = [];
  for (let i = 0; i < content.length; i += maxLength) {
    chunks.push(content.slice(i, i + maxLength));
  }
  return chunks;
}
```

### Error Handling Best Practices

```typescript
async function safeGenerateContent(params: ContentGenerationParams) {
  try {
    return await generateContentWithClaude(params);
  } catch (error) {
    if (error.status === 429) {
      // Rate limit - wait and retry
      await new Promise(resolve => setTimeout(resolve, 60000));
      return await generateContentWithClaude(params);
    }
    
    if (error.status === 401) {
      throw new Error('Invalid API key. Check ANTHROPIC_API_KEY');
    }
    
    // Fallback to OpenAI
    console.warn('Claude failed, falling back to OpenAI:', error);
    return await generateContentWithOpenAI(params);
  }
}
```

This skill provides comprehensive coverage of the marketing-pipeline-share automation system, enabling AI agents to help developers implement end-to-end content automation workflows with research, AI generation, and video rendering capabilities.
