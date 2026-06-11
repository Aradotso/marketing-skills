---
name: marketing-pipeline-auto-content
description: Automated AI content pipeline for research, script writing, auto-posting, and video generation using Claude/OpenAI and Remotion
triggers:
  - how do I set up the automated content pipeline
  - generate content from keyword research automatically
  - create AI-powered marketing videos with this pipeline
  - configure Claude and OpenAI for content generation
  - automate content research and posting workflow
  - use Remotion to render videos from content
  - set up the marketing automation pipeline
  - crawl news and generate content scripts automatically
---

# Marketing Pipeline Auto Content

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

Marketing Pipeline Auto Content is a comprehensive TypeScript-based system that automates the entire content creation workflow: from researching trending topics across news sources (TechCrunch, a16z, Twitter, LinkedIn), to generating content scripts in multiple formats and languages using Claude/OpenAI, to rendering videos with Remotion. This tool transforms a single keyword into publication-ready content and video assets across multiple platforms.

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

Create a `.env.local` file in the project root:

```bash
# AI Provider APIs
OPENAI_API_KEY=your_openai_key_here
ANTHROPIC_API_KEY=your_claude_key_here

# Research & Scraping APIs
RAPIDAPI_KEY=your_rapidapi_key_here

# Optional: Database & Storage
DATABASE_URL=your_database_url_here
STORAGE_BUCKET=your_storage_bucket_here

# Next.js Configuration
NEXT_PUBLIC_API_URL=http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/              # Core library code
│   │   ├── ai/           # AI integrations (Claude, OpenAI)
│   │   ├── research/     # Content research & crawling
│   │   ├── generation/   # Content generation logic
│   │   └── video/        # Remotion video rendering
│   ├── remotion/         # Remotion video templates
│   └── types/            # TypeScript type definitions
├── public/               # Static assets
└── package.json
```

## Key Commands

```bash
# Development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Render video with Remotion
npm run render

# Type checking
npm run type-check

# Linting
npm run lint
```

## Core Workflow

### 1. Research & Content Crawling

```typescript
// src/lib/research/crawler.ts
import axios from 'axios';

interface ResearchResult {
  title: string;
  url: string;
  summary: string;
  publishedAt: Date;
  source: string;
}

export async function researchTopic(keyword: string): Promise<ResearchResult[]> {
  const rapidApiKey = process.env.RAPIDAPI_KEY;
  
  const sources = [
    {
      name: 'techcrunch',
      endpoint: 'https://techcrunch.com/wp-json/wp/v2/posts'
    },
    {
      name: 'twitter',
      endpoint: 'https://twitter-api.p.rapidapi.com/search'
    }
  ];

  const results: ResearchResult[] = [];

  for (const source of sources) {
    try {
      const response = await axios.get(source.endpoint, {
        params: { q: keyword, count: 10 },
        headers: {
          'X-RapidAPI-Key': rapidApiKey,
          'X-RapidAPI-Host': 'twitter-api.p.rapidapi.com'
        }
      });

      const articles = response.data.map((item: any) => ({
        title: item.title || item.text,
        url: item.link || item.url,
        summary: item.excerpt || item.text?.substring(0, 200),
        publishedAt: new Date(item.date || item.created_at),
        source: source.name
      }));

      results.push(...articles);
    } catch (error) {
      console.error(`Error fetching from ${source.name}:`, error);
    }
  }

  return results.filter(r => r.publishedAt > new Date(Date.now() - 24 * 60 * 60 * 1000));
}
```

### 2. AI Content Generation

```typescript
// src/lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

interface ContentRequest {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  researchData: ResearchResult[];
}

interface GeneratedContent {
  title: string;
  content: string;
  outline: string[];
  metadata: {
    wordCount: number;
    readTime: number;
  };
}

export async function generateContentWithClaude(
  request: ContentRequest
): Promise<GeneratedContent> {
  const anthropic = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY,
  });

  const researchContext = request.researchData
    .map(r => `${r.title}\n${r.summary}\nSource: ${r.source}`)
    .join('\n\n');

  const prompt = `You are a professional content writer. Create a ${request.format} article about "${request.keyword}" in ${request.language} language with a ${request.tone} tone.

Research data from the last 24 hours:
${researchContext}

Format: ${request.format}
Requirements:
- Include specific data points and examples from the research
- Make it engaging and actionable
- Optimize for SEO
- ${request.language === 'vi' ? 'Write in Vietnamese' : 'Write in English'}

Generate:
1. A compelling title
2. An outline (5-7 main points)
3. Full article content (800-1200 words)`;

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [{ role: 'user', content: prompt }],
  });

  const responseText = message.content[0].type === 'text' 
    ? message.content[0].text 
    : '';

  return parseContentResponse(responseText);
}

export async function generateContentWithOpenAI(
  request: ContentRequest
): Promise<GeneratedContent> {
  const openai = new OpenAI({
    apiKey: process.env.OPENAI_API_KEY,
  });

  const researchContext = request.researchData
    .map(r => `${r.title}\n${r.summary}`)
    .join('\n\n');

  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are an expert content creator specializing in ${request.format} articles.`
      },
      {
        role: 'user',
        content: `Create a ${request.format} about "${request.keyword}" using this research:\n\n${researchContext}`
      }
    ],
    temperature: 0.7,
    max_tokens: 3000,
  });

  return parseContentResponse(completion.choices[0].message.content || '');
}

function parseContentResponse(text: string): GeneratedContent {
  const lines = text.split('\n');
  const title = lines.find(l => l.startsWith('#'))?.replace(/^#\s*/, '') || '';
  const wordCount = text.split(/\s+/).length;
  
  return {
    title,
    content: text,
    outline: lines.filter(l => l.match(/^#{2,3}\s/)),
    metadata: {
      wordCount,
      readTime: Math.ceil(wordCount / 200),
    },
  };
}
```

### 3. Video Generation with Remotion

```typescript
// src/lib/video/video-renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  content: GeneratedContent;
  template: 'infographic' | 'slideshow' | 'animated-text';
  platform: 'reels' | 'tiktok' | 'shorts' | 'youtube';
}

const platformDimensions = {
  reels: { width: 1080, height: 1920, fps: 30 },
  tiktok: { width: 1080, height: 1920, fps: 30 },
  shorts: { width: 1080, height: 1920, fps: 30 },
  youtube: { width: 1920, height: 1080, fps: 30 },
};

export async function renderContentVideo(config: VideoConfig): Promise<string> {
  const bundled = await bundle({
    entryPoint: path.join(process.cwd(), 'src/remotion/index.ts'),
    webpackOverride: (currentConfiguration) => currentConfiguration,
  });

  const dimensions = platformDimensions[config.platform];
  const inputProps = {
    title: config.content.title,
    outline: config.content.outline,
    template: config.template,
  };

  const composition = await selectComposition({
    serveUrl: bundled,
    id: 'ContentVideo',
    inputProps,
  });

  const outputPath = path.join(
    process.cwd(),
    'public',
    'videos',
    `${Date.now()}-${config.platform}.mp4`
  );

  await renderMedia({
    composition,
    serveUrl: bundled,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps,
    ...dimensions,
  });

  return outputPath;
}
```

```typescript
// src/remotion/ContentVideo.tsx
import { AbsoluteFill, Sequence, useCurrentFrame, interpolate } from 'remotion';
import React from 'react';

interface ContentVideoProps {
  title: string;
  outline: string[];
  template: 'infographic' | 'slideshow' | 'animated-text';
}

export const ContentVideo: React.FC<ContentVideoProps> = ({ title, outline, template }) => {
  const frame = useCurrentFrame();

  const titleOpacity = interpolate(frame, [0, 30], [0, 1], {
    extrapolateRight: 'clamp',
  });

  return (
    <AbsoluteFill style={{ backgroundColor: '#0a0a0a' }}>
      <Sequence from={0} durationInFrames={60}>
        <AbsoluteFill style={{ justifyContent: 'center', alignItems: 'center' }}>
          <h1
            style={{
              fontSize: 80,
              color: 'white',
              textAlign: 'center',
              opacity: titleOpacity,
              padding: '0 100px',
            }}
          >
            {title}
          </h1>
        </AbsoluteFill>
      </Sequence>

      {outline.map((point, index) => (
        <Sequence key={index} from={60 + index * 90} durationInFrames={90}>
          <AbsoluteFill style={{ justifyContent: 'center', alignItems: 'center' }}>
            <div style={{ fontSize: 60, color: 'white', padding: '0 100px' }}>
              {point}
            </div>
          </AbsoluteFill>
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};
```

### 4. Complete Pipeline API Route

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { researchTopic } from '@/lib/research/crawler';
import { generateContentWithClaude } from '@/lib/ai/content-generator';
import { renderContentVideo } from '@/lib/video/video-renderer';

export async function POST(request: NextRequest) {
  try {
    const { keyword, format, language, tone, generateVideo, platform } = await request.json();

    // Step 1: Research
    const researchData = await researchTopic(keyword);
    
    if (researchData.length === 0) {
      return NextResponse.json(
        { error: 'No recent research data found' },
        { status: 404 }
      );
    }

    // Step 2: Generate Content
    const content = await generateContentWithClaude({
      keyword,
      format,
      language,
      tone,
      researchData,
    });

    // Step 3: Generate Video (optional)
    let videoUrl = null;
    if (generateVideo) {
      const videoPath = await renderContentVideo({
        content,
        template: 'infographic',
        platform: platform || 'reels',
      });
      videoUrl = `/videos/${path.basename(videoPath)}`;
    }

    return NextResponse.json({
      success: true,
      data: {
        content,
        videoUrl,
        researchSources: researchData.length,
      },
    });
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { error: 'Content generation failed' },
      { status: 500 }
    );
  }
}
```

### 5. Frontend Component

```typescript
// src/components/ContentPipelineForm.tsx
'use client';

import { useState } from 'react';

export default function ContentPipelineForm() {
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState<any>(null);

  const handleSubmit = async (e: React.FormEvent<HTMLFormElement>) => {
    e.preventDefault();
    setLoading(true);

    const formData = new FormData(e.currentTarget);
    const payload = {
      keyword: formData.get('keyword'),
      format: formData.get('format'),
      language: formData.get('language'),
      tone: formData.get('tone'),
      generateVideo: formData.get('generateVideo') === 'on',
      platform: formData.get('platform'),
    };

    try {
      const response = await fetch('/api/generate', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(payload),
      });

      const data = await response.json();
      setResult(data);
    } catch (error) {
      console.error('Error:', error);
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="max-w-4xl mx-auto p-6">
      <form onSubmit={handleSubmit} className="space-y-4">
        <input
          name="keyword"
          type="text"
          placeholder="Enter keyword..."
          className="w-full p-3 border rounded"
          required
        />

        <select name="format" className="w-full p-3 border rounded">
          <option value="toplist">Top List</option>
          <option value="pov">POV</option>
          <option value="case-study">Case Study</option>
          <option value="how-to">How-to</option>
        </select>

        <select name="language" className="w-full p-3 border rounded">
          <option value="en">English</option>
          <option value="vi">Tiếng Việt</option>
        </select>

        <select name="tone" className="w-full p-3 border rounded">
          <option value="expert">Expert</option>
          <option value="friendly">Friendly</option>
          <option value="humorous">Humorous</option>
        </select>

        <label className="flex items-center gap-2">
          <input type="checkbox" name="generateVideo" />
          Generate Video
        </label>

        <select name="platform" className="w-full p-3 border rounded">
          <option value="reels">Instagram Reels</option>
          <option value="tiktok">TikTok</option>
          <option value="shorts">YouTube Shorts</option>
          <option value="youtube">YouTube</option>
        </select>

        <button
          type="submit"
          disabled={loading}
          className="w-full bg-blue-600 text-white p-3 rounded hover:bg-blue-700 disabled:bg-gray-400"
        >
          {loading ? 'Generating...' : 'Generate Content'}
        </button>
      </form>

      {result && (
        <div className="mt-8 space-y-4">
          <h2 className="text-2xl font-bold">{result.data?.content?.title}</h2>
          <div className="prose max-w-none">
            <pre className="whitespace-pre-wrap">{result.data?.content?.content}</pre>
          </div>
          {result.data?.videoUrl && (
            <video controls src={result.data.videoUrl} className="w-full" />
          )}
        </div>
      )}
    </div>
  );
}
```

## Common Patterns

### Batch Content Generation

```typescript
async function generateBatchContent(keywords: string[]) {
  const results = await Promise.allSettled(
    keywords.map(async (keyword) => {
      const research = await researchTopic(keyword);
      return generateContentWithClaude({
        keyword,
        format: 'toplist',
        language: 'en',
        tone: 'expert',
        researchData: research,
      });
    })
  );

  return results
    .filter((r) => r.status === 'fulfilled')
    .map((r) => (r as PromiseFulfilledResult<GeneratedContent>).value);
}
```

### Scheduled Content Pipeline

```typescript
// Using node-cron for scheduling
import cron from 'node-cron';

cron.schedule('0 9 * * *', async () => {
  const trendingKeywords = await fetchTrendingTopics();
  
  for (const keyword of trendingKeywords.slice(0, 3)) {
    const research = await researchTopic(keyword);
    const content = await generateContentWithClaude({
      keyword,
      format: 'pov',
      language: 'vi',
      tone: 'friendly',
      researchData: research,
    });
    
    // Auto-post to platforms
    await publishToSocialMedia(content);
  }
});
```

## Troubleshooting

### API Rate Limits

If you encounter rate limiting:

```typescript
// Add exponential backoff
async function withRetry<T>(fn: () => Promise<T>, maxRetries = 3): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error: any) {
      if (error.status === 429 && i < maxRetries - 1) {
        await new Promise(resolve => setTimeout(resolve, 2 ** i * 1000));
        continue;
      }
      throw error;
    }
  }
  throw new Error('Max retries exceeded');
}
```

### Video Rendering Issues

Ensure ffmpeg is installed:

```bash
# macOS
brew install ffmpeg

# Ubuntu/Debian
sudo apt-get install ffmpeg

# Windows
# Download from https://ffmpeg.org/download.html
```

### Memory Issues with Large Batches

Use streaming for large content generation:

```typescript
async function* streamContentGeneration(keywords: string[]) {
  for (const keyword of keywords) {
    const research = await researchTopic(keyword);
    const content = await generateContentWithClaude({
      keyword,
      format: 'toplist',
      language: 'en',
      tone: 'expert',
      researchData: research,
    });
    yield content;
  }
}
```

## Best Practices

1. **Cache Research Results**: Store research data for 24h to avoid redundant API calls
2. **Use Environment-Specific AI Models**: GPT-3.5 for development, GPT-4/Claude for production
3. **Implement Content Moderation**: Filter generated content before auto-posting
4. **Monitor Token Usage**: Track API costs and set usage limits
5. **Version Control Videos**: Keep rendered video templates in Git for consistency
