---
name: marketing-pipeline-ai-content-automation
description: AI-powered content automation pipeline for research, scriptwriting, and video generation using Claude, OpenAI, and Remotion
triggers:
  - automate content creation with AI research and video generation
  - set up marketing pipeline for automated content workflow
  - create AI-powered content from research to video
  - build automated content pipeline with Claude and OpenAI
  - generate marketing videos automatically from articles
  - implement AI content automation with Remotion video rendering
  - create multi-format content with automated research crawling
  - set up end-to-end content generation pipeline
---

# Marketing Pipeline AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

A comprehensive AI-powered content automation system that handles the entire content creation pipeline: from automated research and data crawling, to multi-format article generation, and automated video/infographic rendering. Built with Next.js, TypeScript, Claude/OpenAI APIs, and Remotion for video generation.

## What It Does

This system provides an end-to-end content automation workflow:

1. **Auto-Research**: Crawls recent articles from TechCrunch, a16z, Twitter/X, LinkedIn for up-to-date insights
2. **AI Content Generation**: Uses Claude 3 or OpenAI to generate articles in multiple formats (toplist, POV, case study, how-to)
3. **Multi-Language Support**: Generates content in both English and Vietnamese simultaneously
4. **Automated Video Rendering**: Converts written content into videos/infographics using Remotion
5. **Platform Optimization**: Exports videos in formats optimized for Reels, TikTok, and YouTube Shorts

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

## Configuration

Create a `.env.local` file with the following variables:

```bash
# AI Model APIs
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research/Crawling APIs
RAPIDAPI_KEY=your_rapidapi_key

# Video Rendering (Remotion)
REMOTION_LICENSE_KEY=your_remotion_license

# Database (if applicable)
DATABASE_URL=your_database_url

# Next.js Configuration
NEXT_PUBLIC_API_URL=http://localhost:3000
```

## Key API Patterns

### 1. Research & Data Crawling

```typescript
// lib/research/crawler.ts
import axios from 'axios';

interface ResearchSource {
  url: string;
  title: string;
  content: string;
  publishedAt: Date;
}

export async function crawlTechNews(keyword: string): Promise<ResearchSource[]> {
  const sources = [
    `https://techcrunch.com/search/${encodeURIComponent(keyword)}`,
    `https://news.ycombinator.com/search?q=${encodeURIComponent(keyword)}`
  ];

  const results: ResearchSource[] = [];

  for (const url of sources) {
    try {
      const response = await axios.get(url, {
        headers: {
          'X-RapidAPI-Key': process.env.RAPIDAPI_KEY,
          'X-RapidAPI-Host': 'news-api.rapidapi.com'
        }
      });

      // Parse and extract articles
      const articles = parseArticles(response.data);
      results.push(...articles);
    } catch (error) {
      console.error(`Failed to crawl ${url}:`, error);
    }
  }

  return results;
}

function parseArticles(data: any): ResearchSource[] {
  // Implementation depends on API response structure
  return data.articles?.map((article: any) => ({
    url: article.url,
    title: article.title,
    content: article.description || article.content,
    publishedAt: new Date(article.publishedAt)
  })) || [];
}
```

### 2. AI Content Generation with Claude

```typescript
// lib/ai/claude-generator.ts
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

interface ContentRequest {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  tone: 'expert' | 'friendly' | 'humorous';
  language: 'en' | 'vi';
  researchData: string;
}

export async function generateContent(request: ContentRequest): Promise<string> {
  const prompt = buildPrompt(request);

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

function buildPrompt(request: ContentRequest): string {
  const formatInstructions = {
    'toplist': 'Create a numbered list article with clear rankings and explanations',
    'pov': 'Write from a unique perspective or opinion-based angle',
    'case-study': 'Analyze a specific example with data and insights',
    'how-to': 'Provide step-by-step actionable instructions'
  };

  return `
You are a ${request.tone} content writer specializing in ${request.format} articles.

Topic: ${request.keyword}
Language: ${request.language}
Format: ${formatInstructions[request.format]}

Research Data:
${request.researchData}

Create a comprehensive, engaging article that:
1. Uses the latest research data provided
2. Includes specific numbers, dates, and examples
3. Follows the ${request.format} format strictly
4. Maintains a ${request.tone} tone throughout
5. Is optimized for social media sharing

Article:
  `.trim();
}
```

### 3. OpenAI Content Generation (Alternative)

```typescript
// lib/ai/openai-generator.ts
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

export async function generateContentOpenAI(request: ContentRequest): Promise<string> {
  const prompt = buildPrompt(request);

  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are an expert content creator specializing in ${request.format} articles with a ${request.tone} tone.`
      },
      {
        role: 'user',
        content: prompt
      }
    ],
    temperature: 0.7,
    max_tokens: 4000
  });

  return completion.choices[0]?.message?.content || '';
}
```

### 4. Remotion Video Generation

```typescript
// lib/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoContent {
  title: string;
  subtitle: string;
  keyPoints: string[];
  imageUrls?: string[];
}

export async function renderContentVideo(
  content: VideoContent,
  outputPath: string
): Promise<string> {
  // Bundle the Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.tsx'),
    webpackOverride: (config) => config,
  });

  // Get composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: content,
  });

  // Render video
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: content,
  });

  return outputPath;
}
```

```tsx
// remotion/ContentVideo.tsx
import { AbsoluteFill, Sequence, useCurrentFrame, useVideoConfig } from 'remotion';

interface ContentVideoProps {
  title: string;
  subtitle: string;
  keyPoints: string[];
}

export const ContentVideo: React.FC<ContentVideoProps> = ({
  title,
  subtitle,
  keyPoints
}) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  const opacity = Math.min(1, frame / 30);

  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a2e' }}>
      <Sequence from={0} durationInFrames={90}>
        <AbsoluteFill style={{ 
          justifyContent: 'center', 
          alignItems: 'center',
          opacity 
        }}>
          <h1 style={{ 
            color: 'white', 
            fontSize: 60,
            textAlign: 'center',
            padding: '0 50px'
          }}>
            {title}
          </h1>
          <p style={{ 
            color: '#aaa', 
            fontSize: 30,
            marginTop: 20 
          }}>
            {subtitle}
          </p>
        </AbsoluteFill>
      </Sequence>

      {keyPoints.map((point, index) => (
        <Sequence 
          key={index}
          from={90 + index * 60} 
          durationInFrames={60}
        >
          <AbsoluteFill style={{ 
            justifyContent: 'center', 
            alignItems: 'center' 
          }}>
            <div style={{
              color: 'white',
              fontSize: 40,
              padding: '0 80px',
              textAlign: 'center'
            }}>
              {point}
            </div>
          </AbsoluteFill>
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};
```

## End-to-End Pipeline Usage

```typescript
// app/api/pipeline/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { crawlTechNews } from '@/lib/research/crawler';
import { generateContent } from '@/lib/ai/claude-generator';
import { renderContentVideo } from '@/lib/video/renderer';

export async function POST(req: NextRequest) {
  try {
    const { keyword, format, tone, language } = await req.json();

    // Step 1: Research
    console.log('🔍 Starting research...');
    const researchData = await crawlTechNews(keyword);
    const researchText = researchData
      .map(r => `${r.title}\n${r.content}`)
      .join('\n\n');

    // Step 2: Generate Content
    console.log('✍️ Generating content...');
    const article = await generateContent({
      keyword,
      format,
      tone,
      language,
      researchData: researchText
    });

    // Step 3: Extract key points for video
    const keyPoints = extractKeyPoints(article);

    // Step 4: Render Video
    console.log('🎬 Rendering video...');
    const videoPath = await renderContentVideo({
      title: keyword,
      subtitle: `${format} article`,
      keyPoints
    }, `./public/videos/${Date.now()}.mp4`);

    return NextResponse.json({
      success: true,
      article,
      videoPath,
      researchSources: researchData.length
    });

  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { error: 'Pipeline failed' },
      { status: 500 }
    );
  }
}

function extractKeyPoints(article: string): string[] {
  // Simple extraction - split by numbered lists or headings
  const points = article
    .split(/\n(?=\d+\.|\#\#)/)
    .filter(p => p.trim().length > 20)
    .slice(0, 5)
    .map(p => p.replace(/^\d+\.\s*|\#+\s*/g, '').trim());
  
  return points;
}
```

## Frontend Component Example

```typescript
// app/components/ContentPipeline.tsx
'use client';

import { useState } from 'react';

export default function ContentPipeline() {
  const [keyword, setKeyword] = useState('');
  const [format, setFormat] = useState<'toplist' | 'pov' | 'case-study' | 'how-to'>('toplist');
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
          tone: 'expert',
          language: 'en'
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
    <div className="max-w-4xl mx-auto p-6">
      <h1 className="text-3xl font-bold mb-6">AI Content Pipeline</h1>
      
      <div className="space-y-4">
        <input
          type="text"
          value={keyword}
          onChange={(e) => setKeyword(e.target.value)}
          placeholder="Enter keyword or topic..."
          className="w-full p-3 border rounded"
        />

        <select
          value={format}
          onChange={(e) => setFormat(e.target.value as any)}
          className="w-full p-3 border rounded"
        >
          <option value="toplist">Top List</option>
          <option value="pov">Point of View</option>
          <option value="case-study">Case Study</option>
          <option value="how-to">How To Guide</option>
        </select>

        <button
          onClick={handleGenerate}
          disabled={loading || !keyword}
          className="w-full p-3 bg-blue-600 text-white rounded hover:bg-blue-700 disabled:bg-gray-400"
        >
          {loading ? '⏳ Generating...' : '🚀 Generate Content'}
        </button>

        {result && (
          <div className="mt-6 space-y-4">
            <div className="p-4 bg-green-50 rounded">
              <h3 className="font-bold">✅ Success!</h3>
              <p>Research sources: {result.researchSources}</p>
            </div>

            <div className="p-4 bg-white border rounded">
              <h3 className="font-bold mb-2">Article:</h3>
              <pre className="whitespace-pre-wrap">{result.article}</pre>
            </div>

            {result.videoPath && (
              <div className="p-4 bg-white border rounded">
                <h3 className="font-bold mb-2">Video:</h3>
                <video controls src={result.videoPath} className="w-full" />
              </div>
            )}
          </div>
        )}
      </div>
    </div>
  );
}
```

## Development Workflow

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Render Remotion video locally
npm run remotion:render

# Preview Remotion compositions
npm run remotion:preview
```

## Common Patterns

### Batch Content Generation

```typescript
// lib/batch/processor.ts
import pLimit from 'p-limit';

const limit = pLimit(3); // Max 3 concurrent requests

export async function batchGenerateContent(keywords: string[]) {
  const results = await Promise.all(
    keywords.map(keyword =>
      limit(async () => {
        const research = await crawlTechNews(keyword);
        const content = await generateContent({
          keyword,
          format: 'toplist',
          tone: 'expert',
          language: 'en',
          researchData: research.map(r => r.content).join('\n')
        });
        return { keyword, content };
      })
    )
  );

  return results;
}
```

### Caching Research Data

```typescript
// lib/cache/research-cache.ts
import { Redis } from '@upstash/redis';

const redis = new Redis({
  url: process.env.UPSTASH_REDIS_URL!,
  token: process.env.UPSTASH_REDIS_TOKEN!,
});

export async function getCachedResearch(keyword: string) {
  const cacheKey = `research:${keyword}`;
  const cached = await redis.get(cacheKey);
  
  if (cached) return cached as any;

  const research = await crawlTechNews(keyword);
  await redis.set(cacheKey, research, { ex: 3600 }); // Cache 1 hour
  
  return research;
}
```

## Troubleshooting

### API Rate Limits

```typescript
// lib/utils/rate-limiter.ts
import { Ratelimit } from '@upstash/ratelimit';
import { Redis } from '@upstash/redis';

const ratelimit = new Ratelimit({
  redis: Redis.fromEnv(),
  limiter: Ratelimit.slidingWindow(10, '1 m'), // 10 requests per minute
});

export async function checkRateLimit(identifier: string) {
  const { success, remaining } = await ratelimit.limit(identifier);
  
  if (!success) {
    throw new Error(`Rate limit exceeded. Try again later.`);
  }
  
  return remaining;
}
```

### Video Rendering Errors

```typescript
// Ensure FFmpeg is installed for Remotion
// macOS: brew install ffmpeg
// Ubuntu: sudo apt install ffmpeg
// Windows: Download from ffmpeg.org

// Check FFmpeg availability
import { ensureFfmpeg } from '@remotion/renderer';

await ensureFfmpeg(); // Will throw if FFmpeg not found
```

### Content Quality Issues

```typescript
// Add content validation
function validateContent(content: string): boolean {
  const minLength = 500;
  const hasHeadings = /#{1,3}\s/.test(content);
  const hasData = /\d+%|\$\d+|€\d+/.test(content);
  
  return content.length >= minLength && hasHeadings && hasData;
}

// Retry with different prompt if validation fails
let content = await generateContent(request);
let attempts = 0;

while (!validateContent(content) && attempts < 3) {
  console.log(`Content validation failed, retrying... (${attempts + 1}/3)`);
  content = await generateContent({
    ...request,
    researchData: request.researchData + '\n\nPlease include more specific data and examples.'
  });
  attempts++;
}
```

### Environment Variable Checklist

```typescript
// lib/config/validate-env.ts
export function validateEnvironment() {
  const required = [
    'ANTHROPIC_API_KEY',
    'OPENAI_API_KEY',
    'RAPIDAPI_KEY',
  ];

  const missing = required.filter(key => !process.env[key]);

  if (missing.length > 0) {
    throw new Error(
      `Missing required environment variables: ${missing.join(', ')}\n` +
      'Please check your .env.local file.'
    );
  }
}
```

This skill provides comprehensive guidance for AI agents to help developers implement and use the marketing pipeline automation system effectively, from research crawling to AI content generation and video rendering.
