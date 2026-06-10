---
name: marketing-pipeline-share-ai-content-automation
description: Automate content creation from research to video generation using AI (Claude/OpenAI) and Remotion
triggers:
  - how do I automate content creation with AI
  - set up AI content pipeline with research and video generation
  - create automated marketing content workflow
  - generate videos from content using Remotion
  - build AI content automation system
  - automate content research and video creation
  - use Claude and OpenAI for content generation
  - create AI-powered content pipeline
---

# Marketing Pipeline Share - AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

Marketing Pipeline Share is a complete AI-powered content automation system that handles the entire content creation workflow: from research and script writing to automated video generation. It crawls recent news from sources like TechCrunch, a16z, Twitter/X, and LinkedIn, then uses Claude 3 or OpenAI to generate content in multiple formats (toplists, POV pieces, case studies, how-tos) in both English and Vietnamese, and finally renders videos using Remotion.

This is a Next.js + TypeScript application that integrates multiple AI services and automation tools into a unified content production pipeline.

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

Create a `.env.local` file with the following environment variables:

```bash
# AI Services
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research & Scraping
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Remotion (for video rendering)
REMOTION_AWS_ACCESS_KEY_ID=your_aws_access_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret_key

# Application
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

## Core Components

### 1. Research & Content Scraping

The system automatically crawls and analyzes content from multiple sources:

```typescript
// services/research/scraper.ts
import axios from 'axios';

interface ResearchSource {
  source: string;
  url: string;
  content: string;
  publishedAt: Date;
}

export async function scrapeRecentNews(
  topic: string,
  sources: string[] = ['techcrunch', 'a16z', 'twitter', 'linkedin']
): Promise<ResearchSource[]> {
  const results: ResearchSource[] = [];
  
  for (const source of sources) {
    try {
      const response = await axios.get(`https://api.rapidapi.com/news/${source}`, {
        headers: {
          'X-RapidAPI-Key': process.env.RAPIDAPI_KEY,
          'X-RapidAPI-Host': 'news-scraper.p.rapidapi.com'
        },
        params: {
          query: topic,
          timeframe: '24h'
        }
      });
      
      results.push(...response.data.articles);
    } catch (error) {
      console.error(`Failed to scrape ${source}:`, error);
    }
  }
  
  return results;
}

export async function analyzeResearch(sources: ResearchSource[]): Promise<string> {
  // Extract key insights, statistics, and trends
  const insights = sources.map(s => ({
    source: s.source,
    keyPoints: extractKeyPoints(s.content),
    stats: extractStatistics(s.content)
  }));
  
  return JSON.stringify(insights, null, 2);
}
```

### 2. AI Content Generation

Generate content using Claude or OpenAI with multiple format options:

```typescript
// services/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

type ContentFormat = 'toplist' | 'pov' | 'case-study' | 'how-to';
type Language = 'en' | 'vi';
type Tone = 'expert' | 'friendly' | 'humorous';

interface ContentGenerationParams {
  topic: string;
  researchData: string;
  format: ContentFormat;
  language: Language;
  tone: Tone;
  provider?: 'claude' | 'openai';
}

export async function generateContent(params: ContentGenerationParams): Promise<string> {
  const {
    topic,
    researchData,
    format,
    language,
    tone,
    provider = 'claude'
  } = params;
  
  const prompt = buildPrompt(topic, researchData, format, language, tone);
  
  if (provider === 'claude') {
    return await generateWithClaude(prompt);
  } else {
    return await generateWithOpenAI(prompt);
  }
}

async function generateWithClaude(prompt: string): Promise<string> {
  const anthropic = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY,
  });
  
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [{
      role: 'user',
      content: prompt
    }]
  });
  
  return message.content[0].type === 'text' ? message.content[0].text : '';
}

async function generateWithOpenAI(prompt: string): Promise<string> {
  const openai = new OpenAI({
    apiKey: process.env.OPENAI_API_KEY,
  });
  
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [{
      role: 'user',
      content: prompt
    }],
    max_tokens: 4096
  });
  
  return completion.choices[0]?.message?.content || '';
}

function buildPrompt(
  topic: string,
  researchData: string,
  format: ContentFormat,
  language: Language,
  tone: Tone
): string {
  const formatInstructions = {
    'toplist': 'Create a numbered list article with clear rankings',
    'pov': 'Write a perspective piece with personal insights and opinions',
    'case-study': 'Develop a detailed case study with data and analysis',
    'how-to': 'Create a step-by-step tutorial guide'
  };
  
  const languageInstructions = {
    'en': 'Write in English',
    'vi': 'Viết bằng tiếng Việt'
  };
  
  const toneInstructions = {
    'expert': 'Use professional, authoritative tone with technical depth',
    'friendly': 'Use conversational, approachable tone',
    'humorous': 'Use light, entertaining tone with humor'
  };
  
  return `
You are a professional content creator. Create an article about: ${topic}

Format: ${formatInstructions[format]}
Language: ${languageInstructions[language]}
Tone: ${toneInstructions[tone]}

Research data to incorporate:
${researchData}

Requirements:
- Include relevant statistics and data points from the research
- Make it engaging and actionable
- Optimize for SEO
- Include a strong introduction and conclusion
- Add proper headings and structure
`.trim();
}
```

### 3. Video Generation with Remotion

Automatically render videos from generated content:

```typescript
// services/video/remotion-renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  content: string;
  title: string;
  aspectRatio: '16:9' | '9:16' | '1:1';
  platform: 'youtube' | 'tiktok' | 'instagram';
}

export async function renderContentVideo(config: VideoConfig): Promise<string> {
  const {
    content,
    title,
    aspectRatio,
    platform
  } = config;
  
  // Bundle the Remotion project
  const bundleLocation = await bundle(
    path.join(process.cwd(), 'remotion', 'index.ts'),
    () => undefined,
    {
      webpackOverride: (config) => config,
    }
  );
  
  // Get composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      content,
      title,
      aspectRatio
    }
  });
  
  // Render video
  const outputLocation = path.join(
    process.cwd(),
    'output',
    `${platform}-${Date.now()}.mp4`
  );
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: {
      content,
      title,
      aspectRatio
    }
  });
  
  return outputLocation;
}
```

### 4. Complete Pipeline API Route

```typescript
// app/api/pipeline/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { scrapeRecentNews, analyzeResearch } from '@/services/research/scraper';
import { generateContent } from '@/services/ai/content-generator';
import { renderContentVideo } from '@/services/video/remotion-renderer';

export async function POST(request: NextRequest) {
  try {
    const {
      topic,
      format = 'toplist',
      language = 'en',
      tone = 'expert',
      generateVideo = false,
      platform = 'youtube'
    } = await request.json();
    
    // Step 1: Research
    console.log('Starting research phase...');
    const researchData = await scrapeRecentNews(topic);
    const insights = await analyzeResearch(researchData);
    
    // Step 2: Generate Content
    console.log('Generating content...');
    const content = await generateContent({
      topic,
      researchData: insights,
      format,
      language,
      tone
    });
    
    // Step 3: Optionally generate video
    let videoUrl = null;
    if (generateVideo) {
      console.log('Rendering video...');
      const videoPath = await renderContentVideo({
        content,
        title: topic,
        aspectRatio: platform === 'tiktok' ? '9:16' : '16:9',
        platform
      });
      videoUrl = `/output/${path.basename(videoPath)}`;
    }
    
    return NextResponse.json({
      success: true,
      data: {
        content,
        videoUrl,
        researchSources: researchData.length,
        generatedAt: new Date().toISOString()
      }
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

### 5. Client-Side Usage

```typescript
// components/ContentPipeline.tsx
'use client';

import { useState } from 'react';

export default function ContentPipeline() {
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState(null);
  
  async function runPipeline(formData: FormData) {
    setLoading(true);
    
    try {
      const response = await fetch('/api/pipeline', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify({
          topic: formData.get('topic'),
          format: formData.get('format'),
          language: formData.get('language'),
          tone: formData.get('tone'),
          generateVideo: formData.get('generateVideo') === 'on',
          platform: formData.get('platform')
        })
      });
      
      const data = await response.json();
      setResult(data);
    } catch (error) {
      console.error('Error:', error);
    } finally {
      setLoading(false);
    }
  }
  
  return (
    <div className="max-w-4xl mx-auto p-6">
      <h1 className="text-3xl font-bold mb-6">AI Content Pipeline</h1>
      
      <form action={runPipeline} className="space-y-4">
        <div>
          <label className="block mb-2">Topic</label>
          <input
            type="text"
            name="topic"
            className="w-full p-2 border rounded"
            placeholder="Enter your topic..."
            required
          />
        </div>
        
        <div>
          <label className="block mb-2">Format</label>
          <select name="format" className="w-full p-2 border rounded">
            <option value="toplist">Top List</option>
            <option value="pov">POV/Opinion</option>
            <option value="case-study">Case Study</option>
            <option value="how-to">How-To Guide</option>
          </select>
        </div>
        
        <div>
          <label className="block mb-2">Language</label>
          <select name="language" className="w-full p-2 border rounded">
            <option value="en">English</option>
            <option value="vi">Tiếng Việt</option>
          </select>
        </div>
        
        <div>
          <label className="block mb-2">Tone</label>
          <select name="tone" className="w-full p-2 border rounded">
            <option value="expert">Expert/Professional</option>
            <option value="friendly">Friendly/Casual</option>
            <option value="humorous">Humorous</option>
          </select>
        </div>
        
        <div className="flex items-center gap-2">
          <input type="checkbox" name="generateVideo" id="video" />
          <label htmlFor="video">Generate Video</label>
        </div>
        
        <button
          type="submit"
          disabled={loading}
          className="w-full bg-blue-600 text-white p-3 rounded hover:bg-blue-700 disabled:opacity-50"
        >
          {loading ? 'Processing...' : 'Generate Content'}
        </button>
      </form>
      
      {result && (
        <div className="mt-8 p-6 bg-gray-50 rounded">
          <h2 className="text-xl font-bold mb-4">Results</h2>
          <div className="prose max-w-none">
            {result.data.content}
          </div>
          {result.data.videoUrl && (
            <div className="mt-4">
              <video controls className="w-full">
                <source src={result.data.videoUrl} type="video/mp4" />
              </video>
            </div>
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
// services/batch-generator.ts
export async function generateBatchContent(
  topics: string[],
  config: Omit<ContentGenerationParams, 'topic' | 'researchData'>
) {
  const results = [];
  
  for (const topic of topics) {
    const researchData = await scrapeRecentNews(topic);
    const insights = await analyzeResearch(researchData);
    
    const content = await generateContent({
      topic,
      researchData: insights,
      ...config
    });
    
    results.push({
      topic,
      content,
      generatedAt: new Date()
    });
  }
  
  return results;
}
```

### Schedule Content Publication

```typescript
// services/scheduler.ts
export async function scheduleContentPost(
  content: string,
  publishAt: Date,
  platforms: string[]
) {
  // Integration with scheduling service
  const job = {
    content,
    publishAt,
    platforms,
    status: 'scheduled'
  };
  
  // Save to database or queue
  await saveScheduledJob(job);
  
  return job;
}
```

## Troubleshooting

### API Rate Limits

```typescript
// utils/rate-limiter.ts
export class RateLimiter {
  private lastCall = 0;
  private minInterval: number;
  
  constructor(callsPerMinute: number) {
    this.minInterval = 60000 / callsPerMinute;
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
const limiter = new RateLimiter(10); // 10 calls per minute
await limiter.throttle();
```

### Error Handling for AI Services

```typescript
// utils/ai-error-handler.ts
export async function withRetry<T>(
  fn: () => Promise<T>,
  maxRetries = 3,
  delay = 1000
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      
      console.log(`Retry ${i + 1}/${maxRetries} after error:`, error.message);
      await new Promise(resolve => setTimeout(resolve, delay * (i + 1)));
    }
  }
  
  throw new Error('Max retries exceeded');
}
```

### Video Rendering Issues

If Remotion rendering fails:

1. Check AWS credentials are properly configured
2. Ensure sufficient disk space for video output
3. Verify Remotion composition is properly defined
4. Check video codec compatibility

```typescript
// Verify Remotion setup
import { getCompositions } from '@remotion/renderer';

async function verifyRemotionSetup() {
  try {
    const compositions = await getCompositions(bundleLocation);
    console.log('Available compositions:', compositions);
    return true;
  } catch (error) {
    console.error('Remotion setup error:', error);
    return false;
  }
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

# Run linting
npm run lint

# Type checking
npm run type-check

# Render Remotion video locally
npm run remotion:render
```
