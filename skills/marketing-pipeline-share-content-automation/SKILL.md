---
name: marketing-pipeline-share-content-automation
description: AI-powered content automation pipeline for research, scripting, and video generation with Claude/OpenAI
triggers:
  - automate content creation with AI pipeline
  - generate research-based articles automatically
  - create videos from written content
  - set up automated content workflow
  - build AI marketing content system
  - integrate Claude and OpenAI for content generation
  - render videos with Remotion from articles
  - crawl news sources for content research
---

# Marketing Pipeline Share Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with **marketing-pipeline-share**, an end-to-end content automation system that handles research, scriptwriting, article generation, and video rendering. The pipeline crawls real-time data from sources like TechCrunch, Twitter/X, and LinkedIn, then uses Claude 3 or OpenAI to generate multi-format content (toplists, POVs, case studies, how-tos) in multiple languages, and finally renders videos/infographics via Remotion.

## What This Project Does

**marketing-pipeline-share** is a Next.js TypeScript application that automates:

1. **Auto-Scan Research**: Crawls fresh content from news sources and social platforms within the last 24 hours
2. **AI Content Generation**: Uses Claude (Anthropic) or OpenAI to create articles in multiple formats and languages
3. **Multi-Format Output**: Generates toplists, POV pieces, case studies, and how-to guides
4. **Automated Video Rendering**: Converts written content into videos and infographics using Remotion
5. **Platform Optimization**: Exports content optimized for Reels, TikTok, Shorts, and other social platforms

## Installation

### Prerequisites

```bash
# Node.js 18+ required
node --version

# Package manager
npm --version
# or
yarn --version
```

### Clone and Install

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
yarn install
```

### Environment Configuration

Create a `.env.local` file in the project root:

```bash
# AI Provider APIs
ANTHROPIC_API_KEY=your_claude_api_key_here
OPENAI_API_KEY=your_openai_api_key_here

# RapidAPI for web scraping/research
RAPIDAPI_KEY=your_rapidapi_key_here

# Next.js Configuration
NEXT_PUBLIC_APP_URL=http://localhost:3000

# Remotion Configuration (for video rendering)
REMOTION_AWS_ACCESS_KEY_ID=your_aws_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret
```

Reference environment variables in code:

```typescript
const anthropicKey = process.env.ANTHROPIC_API_KEY;
const openaiKey = process.env.OPENAI_API_KEY;
const rapidApiKey = process.env.RAPIDAPI_KEY;
```

## Key Architecture

### Project Structure

```
marketing-pipeline-share/
├── src/
│   ├── app/              # Next.js app router pages
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── crawlers/    # Web scraping modules
│   │   ├── generators/  # Content generation logic
│   │   └── video/       # Remotion video rendering
│   ├── types/           # TypeScript type definitions
│   └── utils/           # Utility functions
├── public/              # Static assets
├── remotion/            # Remotion video templates
└── package.json
```

## Core API Usage

### 1. Research Crawler Module

```typescript
// lib/crawlers/newsScanner.ts
import axios from 'axios';

interface NewsArticle {
  title: string;
  url: string;
  publishedAt: string;
  source: string;
  content: string;
}

export async function scanTechNews(keyword: string, hours: number = 24): Promise<NewsArticle[]> {
  const rapidApiKey = process.env.RAPIDAPI_KEY;
  
  const options = {
    method: 'GET',
    url: 'https://news-api14.p.rapidapi.com/search',
    params: {
      q: keyword,
      language: 'en',
      sortBy: 'publishedAt',
      pageSize: 10
    },
    headers: {
      'X-RapidAPI-Key': rapidApiKey,
      'X-RapidAPI-Host': 'news-api14.p.rapidapi.com'
    }
  };

  try {
    const response = await axios.request(options);
    const cutoffTime = new Date(Date.now() - hours * 60 * 60 * 1000);
    
    return response.data.articles
      .filter((article: any) => new Date(article.publishedAt) > cutoffTime)
      .map((article: any) => ({
        title: article.title,
        url: article.url,
        publishedAt: article.publishedAt,
        source: article.source.name,
        content: article.description || article.content
      }));
  } catch (error) {
    console.error('News scan error:', error);
    throw new Error('Failed to scan news sources');
  }
}
```

### 2. AI Content Generation with Claude

```typescript
// lib/ai/claudeGenerator.ts
import Anthropic from '@anthropic-ai/sdk';

interface ContentRequest {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  researchData: string[];
}

export async function generateContent(request: ContentRequest): Promise<string> {
  const anthropic = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY,
  });

  const formatPrompts = {
    'toplist': 'Create a compelling top 10 list article',
    'pov': 'Write an insightful point-of-view analysis',
    'case-study': 'Develop a detailed case study with real examples',
    'how-to': 'Create a step-by-step how-to guide'
  };

  const toneAdjustments = {
    'expert': 'Use professional, authoritative language with technical depth',
    'friendly': 'Use conversational, approachable language',
    'humorous': 'Include wit and engaging storytelling elements'
  };

  const prompt = `
${formatPrompts[request.format]} about "${request.keyword}" in ${request.language === 'vi' ? 'Vietnamese' : 'English'}.

Tone: ${toneAdjustments[request.tone]}

Research Data:
${request.researchData.join('\n\n')}

Requirements:
- Use only the latest information from the research data
- Include specific data points and statistics
- Make it engaging and shareable
- Optimize for ${request.language === 'vi' ? 'Vietnamese' : 'English'} readers
${request.format === 'toplist' ? '- Number each item clearly' : ''}
${request.format === 'how-to' ? '- Include actionable steps' : ''}
`;

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

  return message.content[0].type === 'text' ? message.content[0].text : '';
}
```

### 3. OpenAI Alternative Generator

```typescript
// lib/ai/openaiGenerator.ts
import OpenAI from 'openai';

export async function generateContentOpenAI(request: ContentRequest): Promise<string> {
  const openai = new OpenAI({
    apiKey: process.env.OPENAI_API_KEY,
  });

  const systemPrompt = `You are an expert content creator specializing in ${request.format} format. 
Write in ${request.language === 'vi' ? 'Vietnamese' : 'English'} with a ${request.tone} tone.`;

  const userPrompt = `Create ${request.format} content about "${request.keyword}" using this research:\n\n${request.researchData.join('\n\n')}`;

  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      { role: 'system', content: systemPrompt },
      { role: 'user', content: userPrompt }
    ],
    temperature: 0.7,
    max_tokens: 3000,
  });

  return completion.choices[0]?.message?.content || '';
}
```

### 4. Video Rendering with Remotion

```typescript
// lib/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  title: string;
  keyPoints: string[];
  duration: number; // in frames (30fps)
  format: 'vertical' | 'square' | 'horizontal';
}

export async function renderContentVideo(config: VideoConfig): Promise<string> {
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
    inputProps: {
      title: config.title,
      keyPoints: config.keyPoints,
    },
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
      keyPoints: config.keyPoints,
    },
  });

  return outputLocation;
}
```

### 5. Complete Pipeline Orchestration

```typescript
// lib/pipeline/orchestrator.ts
import { scanTechNews } from '../crawlers/newsScanner';
import { generateContent } from '../ai/claudeGenerator';
import { renderContentVideo } from '../video/renderer';

interface PipelineConfig {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  languages: ('en' | 'vi')[];
  includeVideo: boolean;
  videoFormat?: 'vertical' | 'square' | 'horizontal';
}

export async function runContentPipeline(config: PipelineConfig) {
  // Step 1: Research
  console.log('🔍 Scanning news sources...');
  const articles = await scanTechNews(config.keyword, 24);
  
  if (articles.length === 0) {
    throw new Error('No recent articles found for this keyword');
  }

  const researchData = articles.map(a => `${a.title}\n${a.content}\nSource: ${a.source}`);

  // Step 2: Generate content for each language
  console.log('✍️ Generating content...');
  const generatedContent: Record<string, string> = {};

  for (const language of config.languages) {
    const content = await generateContent({
      keyword: config.keyword,
      format: config.format,
      language,
      tone: 'expert',
      researchData,
    });
    generatedContent[language] = content;
  }

  // Step 3: Render video (if requested)
  let videoPath: string | null = null;
  
  if (config.includeVideo) {
    console.log('🎬 Rendering video...');
    
    // Extract key points from English content
    const keyPoints = extractKeyPoints(generatedContent['en'], config.format);
    
    videoPath = await renderContentVideo({
      title: config.keyword,
      keyPoints,
      duration: 900, // 30 seconds at 30fps
      format: config.videoFormat || 'vertical',
    });
  }

  return {
    articles,
    content: generatedContent,
    videoPath,
    timestamp: new Date().toISOString(),
  };
}

function extractKeyPoints(content: string, format: string): string[] {
  // Simple extraction - can be enhanced with AI
  const lines = content.split('\n').filter(line => line.trim());
  
  if (format === 'toplist') {
    return lines.filter(line => /^\d+\./.test(line)).slice(0, 5);
  }
  
  // For other formats, take first 5 substantial paragraphs
  return lines.filter(line => line.length > 50).slice(0, 5);
}
```

## Common Usage Patterns

### API Route for Content Generation

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline/orchestrator';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    
    const { keyword, format, languages, includeVideo, videoFormat } = body;

    // Validate input
    if (!keyword || !format) {
      return NextResponse.json(
        { error: 'Missing required fields: keyword, format' },
        { status: 400 }
      );
    }

    // Run pipeline
    const result = await runContentPipeline({
      keyword,
      format,
      languages: languages || ['en'],
      includeVideo: includeVideo || false,
      videoFormat: videoFormat || 'vertical',
    });

    return NextResponse.json(result);
  } catch (error: any) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { error: error.message || 'Pipeline execution failed' },
      { status: 500 }
    );
  }
}
```

### Frontend Component

```typescript
// components/ContentGenerator.tsx
'use client';

import { useState } from 'react';

export default function ContentGenerator() {
  const [keyword, setKeyword] = useState('');
  const [format, setFormat] = useState<'toplist' | 'pov' | 'case-study' | 'how-to'>('toplist');
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState<any>(null);

  const handleGenerate = async () => {
    setLoading(true);
    try {
      const response = await fetch('/api/generate', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          keyword,
          format,
          languages: ['en', 'vi'],
          includeVideo: true,
          videoFormat: 'vertical',
        }),
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
    <div className="p-6 max-w-4xl mx-auto">
      <h1 className="text-3xl font-bold mb-6">AI Content Pipeline</h1>
      
      <div className="space-y-4">
        <input
          type="text"
          placeholder="Enter keyword (e.g., AI marketing tools)"
          value={keyword}
          onChange={(e) => setKeyword(e.target.value)}
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
          <option value="how-to">How-to Guide</option>
        </select>

        <button
          onClick={handleGenerate}
          disabled={loading || !keyword}
          className="w-full bg-blue-600 text-white p-3 rounded hover:bg-blue-700 disabled:opacity-50"
        >
          {loading ? 'Generating...' : 'Generate Content'}
        </button>
      </div>

      {result && (
        <div className="mt-8 space-y-6">
          <div>
            <h2 className="text-xl font-semibold mb-2">Research Sources ({result.articles.length})</h2>
            <ul className="list-disc pl-5">
              {result.articles.map((article: any, i: number) => (
                <li key={i}>{article.title} - {article.source}</li>
              ))}
            </ul>
          </div>

          <div>
            <h2 className="text-xl font-semibold mb-2">Generated Content (EN)</h2>
            <div className="prose max-w-none bg-gray-50 p-4 rounded">
              {result.content.en}
            </div>
          </div>

          {result.content.vi && (
            <div>
              <h2 className="text-xl font-semibold mb-2">Generated Content (VI)</h2>
              <div className="prose max-w-none bg-gray-50 p-4 rounded">
                {result.content.vi}
              </div>
            </div>
          )}

          {result.videoPath && (
            <div>
              <h2 className="text-xl font-semibold mb-2">Generated Video</h2>
              <video controls className="w-full max-w-md">
                <source src={result.videoPath} type="video/mp4" />
              </video>
            </div>
          )}
        </div>
      )}
    </div>
  );
}
```

## Configuration

### TypeScript Types

```typescript
// types/content.ts
export type ContentFormat = 'toplist' | 'pov' | 'case-study' | 'how-to';
export type Language = 'en' | 'vi';
export type Tone = 'expert' | 'friendly' | 'humorous';
export type VideoFormat = 'vertical' | 'square' | 'horizontal';

export interface ResearchArticle {
  title: string;
  url: string;
  publishedAt: string;
  source: string;
  content: string;
}

export interface GeneratedContent {
  format: ContentFormat;
  language: Language;
  content: string;
  metadata: {
    wordCount: number;
    generatedAt: string;
    aiModel: string;
  };
}

export interface PipelineResult {
  articles: ResearchArticle[];
  content: Record<Language, string>;
  videoPath: string | null;
  timestamp: string;
}
```

### Remotion Video Template

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, interpolate } from 'remotion';

interface ContentVideoProps {
  title: string;
  keyPoints: string[];
}

export const ContentVideo: React.FC<ContentVideoProps> = ({ title, keyPoints }) => {
  const frame = useCurrentFrame();
  
  const opacity = interpolate(frame, [0, 30], [0, 1], {
    extrapolateRight: 'clamp',
  });

  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a1a' }}>
      <div style={{ 
        opacity, 
        padding: 60, 
        color: 'white',
        display: 'flex',
        flexDirection: 'column',
        justifyContent: 'center',
      }}>
        <h1 style={{ fontSize: 48, marginBottom: 40 }}>{title}</h1>
        <ul style={{ fontSize: 24, lineHeight: 1.8 }}>
          {keyPoints.map((point, i) => (
            <li key={i} style={{ marginBottom: 20 }}>{point}</li>
          ))}
        </ul>
      </div>
    </AbsoluteFill>
  );
};
```

## Troubleshooting

### API Key Issues

```typescript
// lib/utils/validateEnv.ts
export function validateEnvironment() {
  const required = [
    'ANTHROPIC_API_KEY',
    'OPENAI_API_KEY',
    'RAPIDAPI_KEY',
  ];

  const missing = required.filter(key => !process.env[key]);

  if (missing.length > 0) {
    throw new Error(`Missing required environment variables: ${missing.join(', ')}`);
  }
}

// Call at app startup
// app/layout.tsx
import { validateEnvironment } from '@/lib/utils/validateEnv';

if (process.env.NODE_ENV === 'development') {
  validateEnvironment();
}
```

### Rate Limiting Handler

```typescript
// lib/utils/rateLimiter.ts
export class RateLimiter {
  private lastCall: number = 0;
  private minInterval: number;

  constructor(callsPerMinute: number) {
    this.minInterval = (60 / callsPerMinute) * 1000;
  }

  async throttle() {
    const now = Date.now();
    const timeSinceLastCall = now - this.lastCall;
    
    if (timeSinceLastCall < this.minInterval) {
      await new Promise(resolve => 
        setTimeout(resolve, this.minInterval - timeSinceLastCall)
      );
    }
    
    this.lastCall = Date.now();
  }
}

// Usage
const anthropicLimiter = new RateLimiter(50); // 50 calls/min

export async function generateWithRateLimit(request: ContentRequest) {
  await anthropicLimiter.throttle();
  return generateContent(request);
}
```

### Error Recovery

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
const content = await retryWithBackoff(() => 
  generateContent(request)
);
```

This skill covers the complete automation pipeline from research to video generation, enabling AI agents to effectively integrate and extend this marketing content automation system.
