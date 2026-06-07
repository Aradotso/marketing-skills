---
name: marketing-pipeline-share-automation
description: Automated AI content pipeline for research, script generation, and video creation using Claude/OpenAI and Remotion
triggers:
  - automate my content creation workflow
  - generate marketing content from research
  - create videos from blog posts automatically
  - set up AI content pipeline
  - build automated marketing system
  - generate content with Claude and OpenAI
  - create social media videos automatically
  - scrape news and generate content
---

# Marketing Pipeline Share Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

Marketing Pipeline Share is an all-in-one AI content automation system that transforms keywords into fully-realized content across multiple formats. Built with TypeScript and Next.js, it handles the complete content lifecycle: research scraping from news sources, AI-powered content generation in multiple formats (blog posts, case studies, how-tos), and automatic video rendering via Remotion.

The system supports both English and Vietnamese content generation using Claude 3 and OpenAI APIs, with automated content distribution capabilities.

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

```env
# AI Services
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# News/Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Optional: Facebook/Social Media
FACEBOOK_PAGE_ACCESS_TOKEN=your_token
FACEBOOK_PAGE_ID=your_page_id

# Application
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── app/                    # Next.js app directory
├── components/            # React components
├── lib/
│   ├── ai/               # AI integration (Claude, OpenAI)
│   ├── scraper/          # News scraping logic
│   ├── content/          # Content generation
│   └── video/            # Remotion video rendering
├── remotion/             # Remotion video templates
└── public/               # Static assets
```

## Core Features

### 1. Research Scraping

The system automatically scrapes recent news from multiple sources (TechCrunch, a16z, Twitter/X, LinkedIn) to gather fresh data for content generation.

```typescript
// lib/scraper/news-scraper.ts
import axios from 'axios';

export interface NewsArticle {
  title: string;
  url: string;
  publishedAt: string;
  source: string;
  summary?: string;
}

export async function scrapeRecentNews(
  keyword: string,
  sources: string[] = ['techcrunch', 'twitter', 'linkedin']
): Promise<NewsArticle[]> {
  const articles: NewsArticle[] = [];
  
  for (const source of sources) {
    try {
      const response = await axios.get(
        `https://news-api.rapidapi.com/search`,
        {
          params: {
            q: keyword,
            sources: source,
            from: new Date(Date.now() - 24 * 60 * 60 * 1000).toISOString()
          },
          headers: {
            'X-RapidAPI-Key': process.env.RAPIDAPI_KEY,
            'X-RapidAPI-Host': 'news-api.rapidapi.com'
          }
        }
      );
      
      articles.push(...response.data.articles);
    } catch (error) {
      console.error(`Failed to scrape ${source}:`, error);
    }
  }
  
  return articles;
}
```

### 2. AI Content Generation

Generate content in multiple formats using Claude or OpenAI:

```typescript
// lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

export type ContentFormat = 'toplist' | 'pov' | 'case-study' | 'how-to';
export type Language = 'en' | 'vi';
export type Tone = 'professional' | 'friendly' | 'humorous';

export interface ContentRequest {
  keyword: string;
  format: ContentFormat;
  language: Language;
  tone: Tone;
  researchData: NewsArticle[];
}

export async function generateContentWithClaude(
  request: ContentRequest
): Promise<string> {
  const anthropic = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY
  });

  const researchContext = request.researchData
    .map(article => `- ${article.title} (${article.source}): ${article.summary}`)
    .join('\n');

  const prompt = buildPrompt(request, researchContext);

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [
      {
        role: 'user',
        content: prompt
      }
    ]
  });

  return message.content[0].type === 'text' 
    ? message.content[0].text 
    : '';
}

export async function generateContentWithOpenAI(
  request: ContentRequest
): Promise<string> {
  const openai = new OpenAI({
    apiKey: process.env.OPENAI_API_KEY
  });

  const researchContext = request.researchData
    .map(article => `- ${article.title}: ${article.summary}`)
    .join('\n');

  const prompt = buildPrompt(request, researchContext);

  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: 'You are an expert content marketer and writer.'
      },
      {
        role: 'user',
        content: prompt
      }
    ],
    temperature: 0.7,
    max_tokens: 4000
  });

  return completion.choices[0].message.content || '';
}

function buildPrompt(request: ContentRequest, researchContext: string): string {
  const formatInstructions = {
    'toplist': 'Create a numbered list article with clear rankings and explanations',
    'pov': 'Write a perspective piece that presents a unique viewpoint',
    'case-study': 'Develop a detailed case study with problem, solution, and results',
    'how-to': 'Create a step-by-step tutorial with actionable instructions'
  };

  const toneInstructions = {
    'professional': 'Use formal, authoritative language',
    'friendly': 'Use conversational, approachable language',
    'humorous': 'Include wit and light humor where appropriate'
  };

  return `
Create a ${request.format} article about "${request.keyword}" in ${request.language === 'vi' ? 'Vietnamese' : 'English'}.

${formatInstructions[request.format]}
${toneInstructions[request.tone]}

Use this recent research data:
${researchContext}

Requirements:
- Include data-backed insights from the research
- Make it engaging and valuable for readers
- Structure with clear headings and sections
- Add a compelling introduction and conclusion
- Keep tone consistent: ${request.tone}
`;
}
```

### 3. Video Generation with Remotion

Automatically render videos from generated content:

```typescript
// lib/video/video-generator.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

export interface VideoConfig {
  title: string;
  keyPoints: string[];
  duration: number; // in frames (30fps)
  format: 'square' | 'vertical' | 'horizontal';
}

export async function generateVideo(
  content: string,
  config: VideoConfig
): Promise<string> {
  const compositionId = 'ContentVideo';
  
  // Bundle Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.join(process.cwd(), 'remotion/index.ts'),
    webpackOverride: (config) => config
  });

  // Select composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: {
      title: config.title,
      keyPoints: config.keyPoints,
      format: config.format
    }
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
    inputProps: {
      title: config.title,
      keyPoints: config.keyPoints
    }
  });

  return outputLocation;
}
```

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig, interpolate } from 'remotion';

export const ContentVideo: React.FC<{
  title: string;
  keyPoints: string[];
}> = ({ title, keyPoints }) => {
  const frame = useCurrentFrame();
  const { fps, durationInFrames } = useVideoConfig();

  const titleOpacity = interpolate(frame, [0, 30], [0, 1], {
    extrapolateRight: 'clamp'
  });

  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a2e' }}>
      <div style={{ 
        padding: '60px',
        color: 'white',
        fontFamily: 'Arial, sans-serif'
      }}>
        <h1 style={{ 
          fontSize: '48px',
          opacity: titleOpacity,
          marginBottom: '40px'
        }}>
          {title}
        </h1>
        
        <div>
          {keyPoints.map((point, index) => {
            const pointFrame = 60 + (index * 90);
            const opacity = interpolate(
              frame,
              [pointFrame, pointFrame + 20],
              [0, 1],
              { extrapolateRight: 'clamp' }
            );
            
            return (
              <div
                key={index}
                style={{
                  fontSize: '32px',
                  marginBottom: '30px',
                  opacity,
                  transform: `translateX(${interpolate(
                    frame,
                    [pointFrame, pointFrame + 20],
                    [-50, 0],
                    { extrapolateRight: 'clamp' }
                  )}px)`
                }}
              >
                • {point}
              </div>
            );
          })}
        </div>
      </div>
    </AbsoluteFill>
  );
};
```

### 4. Complete Pipeline API Route

```typescript
// app/api/pipeline/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { scrapeRecentNews } from '@/lib/scraper/news-scraper';
import { generateContentWithClaude } from '@/lib/ai/content-generator';
import { generateVideo } from '@/lib/video/video-generator';

export async function POST(request: NextRequest) {
  try {
    const { keyword, format, language, tone } = await request.json();

    // Step 1: Research
    console.log('Starting research phase...');
    const researchData = await scrapeRecentNews(keyword);

    // Step 2: Generate Content
    console.log('Generating content...');
    const content = await generateContentWithClaude({
      keyword,
      format,
      language,
      tone,
      researchData
    });

    // Step 3: Extract key points for video
    const keyPoints = extractKeyPoints(content, 5);

    // Step 4: Generate Video
    console.log('Rendering video...');
    const videoPath = await generateVideo(content, {
      title: keyword,
      keyPoints,
      duration: 900, // 30 seconds at 30fps
      format: 'vertical'
    });

    return NextResponse.json({
      success: true,
      content,
      videoUrl: videoPath.replace(process.cwd() + '/public', ''),
      researchCount: researchData.length
    });
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}

function extractKeyPoints(content: string, count: number): string[] {
  // Simple extraction - split by newlines and filter headings/bullets
  const lines = content.split('\n')
    .filter(line => line.match(/^[-•*]|^\d+\./))
    .map(line => line.replace(/^[-•*\d.]+\s*/, '').trim())
    .filter(line => line.length > 10 && line.length < 100);
  
  return lines.slice(0, count);
}
```

### 5. Frontend Usage

```typescript
// app/page.tsx
'use client';

import { useState } from 'react';

export default function Home() {
  const [keyword, setKeyword] = useState('');
  const [format, setFormat] = useState<'toplist' | 'pov' | 'case-study' | 'how-to'>('toplist');
  const [language, setLanguage] = useState<'en' | 'vi'>('en');
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState<any>(null);

  const handleGenerate = async () => {
    setLoading(true);
    try {
      const response = await fetch('/api/pipeline', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          keyword,
          format,
          language,
          tone: 'professional'
        })
      });
      
      const data = await response.json();
      setResult(data);
    } catch (error) {
      console.error('Generation failed:', error);
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="container mx-auto p-8">
      <h1 className="text-4xl font-bold mb-8">AI Content Pipeline</h1>
      
      <div className="space-y-4 max-w-md">
        <input
          type="text"
          value={keyword}
          onChange={(e) => setKeyword(e.target.value)}
          placeholder="Enter keyword..."
          className="w-full p-2 border rounded"
        />
        
        <select
          value={format}
          onChange={(e) => setFormat(e.target.value as any)}
          className="w-full p-2 border rounded"
        >
          <option value="toplist">Top List</option>
          <option value="pov">POV Article</option>
          <option value="case-study">Case Study</option>
          <option value="how-to">How-to Guide</option>
        </select>
        
        <select
          value={language}
          onChange={(e) => setLanguage(e.target.value as any)}
          className="w-full p-2 border rounded"
        >
          <option value="en">English</option>
          <option value="vi">Vietnamese</option>
        </select>
        
        <button
          onClick={handleGenerate}
          disabled={loading || !keyword}
          className="w-full bg-blue-600 text-white p-2 rounded disabled:bg-gray-400"
        >
          {loading ? 'Generating...' : 'Generate Content'}
        </button>
      </div>

      {result && (
        <div className="mt-8">
          <h2 className="text-2xl font-bold mb-4">Results</h2>
          <div className="bg-gray-50 p-4 rounded">
            <p className="mb-2">Research articles: {result.researchCount}</p>
            <div className="prose max-w-none mb-4">
              {result.content}
            </div>
            {result.videoUrl && (
              <video controls className="w-full max-w-md">
                <source src={result.videoUrl} type="video/mp4" />
              </video>
            )}
          </div>
        </div>
      )}
    </div>
  );
}
```

## Configuration

### AI Model Selection

Switch between Claude and OpenAI in your pipeline:

```typescript
// lib/config/ai-config.ts
export const AI_CONFIG = {
  defaultProvider: 'claude', // or 'openai'
  models: {
    claude: 'claude-3-5-sonnet-20241022',
    openai: 'gpt-4-turbo-preview'
  },
  maxTokens: {
    claude: 4096,
    openai: 4000
  }
};
```

### Video Format Presets

```typescript
// lib/config/video-config.ts
export const VIDEO_PRESETS = {
  reels: { width: 1080, height: 1920, fps: 30 },
  square: { width: 1080, height: 1080, fps: 30 },
  youtube: { width: 1920, height: 1080, fps: 30 },
  tiktok: { width: 1080, height: 1920, fps: 30 }
};
```

## Common Patterns

### Batch Content Generation

```typescript
async function batchGenerate(keywords: string[]) {
  const results = await Promise.all(
    keywords.map(async (keyword) => {
      const researchData = await scrapeRecentNews(keyword);
      const content = await generateContentWithClaude({
        keyword,
        format: 'toplist',
        language: 'en',
        tone: 'professional',
        researchData
      });
      return { keyword, content };
    })
  );
  
  return results;
}
```

### Content Scheduling

```typescript
// lib/scheduler/content-scheduler.ts
export async function scheduleContent(
  content: string,
  publishAt: Date,
  platforms: string[]
) {
  // Store in database with scheduled time
  await db.scheduledPosts.create({
    data: {
      content,
      publishAt,
      platforms,
      status: 'pending'
    }
  });
}
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement retry logic with exponential backoff
async function retryWithBackoff<T>(
  fn: () => Promise<T>,
  maxRetries = 3
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      await new Promise(resolve => setTimeout(resolve, 2 ** i * 1000));
    }
  }
  throw new Error('Max retries exceeded');
}
```

### Video Rendering Errors

Check Remotion dependencies and ensure ffmpeg is installed:

```bash
# Install ffmpeg (macOS)
brew install ffmpeg

# Install ffmpeg (Ubuntu)
sudo apt-get install ffmpeg

# Verify installation
ffmpeg -version
```

### Memory Issues with Large Content

```typescript
// Process content in chunks
async function processLargeContent(content: string) {
  const chunkSize = 4000;
  const chunks = [];
  
  for (let i = 0; i < content.length; i += chunkSize) {
    chunks.push(content.slice(i, i + chunkSize));
  }
  
  return chunks;
}
```

## Running the Application

```bash
# Development mode
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Render Remotion video (standalone)
npm run remotion:render
```

## Best Practices

1. **Cache research data** to avoid redundant API calls
2. **Queue video rendering** for large batches using Bull or similar
3. **Validate content** before publishing to social platforms
4. **Monitor API usage** to stay within rate limits
5. **Store generated content** in a database for reuse and analytics
