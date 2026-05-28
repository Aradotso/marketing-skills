---
name: marketing-content-pipeline-automation
description: Automated AI content pipeline for research, scriptwriting, social media posting, and video generation using Claude, OpenAI, and Remotion
triggers:
  - how do I automate content creation with AI
  - set up automated marketing content pipeline
  - generate videos from content automatically
  - create AI-powered content workflow
  - automate social media content research and posting
  - build content automation system with Claude and OpenAI
  - use Remotion for automated video rendering
  - implement AI content research and generation
---

# Marketing Content Pipeline Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to help developers use the **Ultimate AI Content Pipeline** - an all-in-one automated content creation system that handles research, scriptwriting, social media posting, and video generation.

## Overview

The Marketing Content Pipeline is a TypeScript/Next.js application that automates the entire content creation workflow:

1. **Auto-Research**: Crawls and analyzes real-time data from TechCrunch, a16z, Twitter/X, LinkedIn
2. **AI Content Generation**: Creates multi-format content (toplist, POV, case studies, how-to) using Claude/OpenAI
3. **Multi-language Support**: Generates content in English and Vietnamese with customizable tone
4. **Video Generation**: Automatically renders videos and infographics using Remotion
5. **Social Media Integration**: Auto-posts content to Facebook pages and other platforms

## Installation

### Prerequisites

```bash
# Node.js 18+ required
node --version

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
# AI Provider Keys
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research & Data Sources
RAPIDAPI_KEY=your_rapidapi_key

# Social Media
FACEBOOK_PAGE_ACCESS_TOKEN=your_facebook_token
FACEBOOK_PAGE_ID=your_page_id

# Video Rendering (Remotion)
REMOTION_LICENSE_KEY=your_remotion_license

# Database (if applicable)
DATABASE_URL=your_database_url

# Application
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

### Start Development Server

```bash
npm run dev
# or
yarn dev
```

Access the application at `http://localhost:3000`

## Core Components

### 1. Research Module

The research module crawls and analyzes content from multiple sources:

```typescript
// lib/research/crawler.ts
import { AnthropicClient } from '@anthropic-ai/sdk';

interface ResearchSource {
  name: string;
  url: string;
  type: 'news' | 'social' | 'blog';
}

export async function researchTopic(keyword: string, timeframe: '24h' | '7d' = '24h') {
  const sources: ResearchSource[] = [
    { name: 'TechCrunch', url: 'https://techcrunch.com/search/', type: 'news' },
    { name: 'a16z', url: 'https://a16z.com/posts/', type: 'blog' },
  ];

  const results = await Promise.all(
    sources.map(source => fetchSourceData(source, keyword, timeframe))
  );

  return aggregateInsights(results);
}

async function fetchSourceData(
  source: ResearchSource,
  keyword: string,
  timeframe: string
) {
  // Use RapidAPI for web scraping
  const response = await fetch('https://web-scraping-api.rapidapi.com/scrape', {
    method: 'POST',
    headers: {
      'X-RapidAPI-Key': process.env.RAPIDAPI_KEY!,
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({
      url: `${source.url}?q=${keyword}`,
      timeframe,
    }),
  });

  return response.json();
}

async function aggregateInsights(results: any[]) {
  const client = new AnthropicClient({
    apiKey: process.env.ANTHROPIC_API_KEY,
  });

  const prompt = `Analyze these research results and extract key insights, trends, and data points:
${JSON.stringify(results, null, 2)}

Provide structured insights with:
1. Main trends
2. Key statistics
3. Notable quotes
4. Emerging patterns`;

  const message = await client.messages.create({
    model: 'claude-3-opus-20240229',
    max_tokens: 4096,
    messages: [{ role: 'user', content: prompt }],
  });

  return message.content;
}
```

### 2. Content Generation Module

Generate content in multiple formats and languages:

```typescript
// lib/content/generator.ts
import OpenAI from 'openai';
import Anthropic from '@anthropic-ai/sdk';

type ContentFormat = 'toplist' | 'pov' | 'case-study' | 'how-to';
type ContentTone = 'expert' | 'friendly' | 'humorous';
type Language = 'en' | 'vi';

interface ContentOptions {
  keyword: string;
  format: ContentFormat;
  tone: ContentTone;
  language: Language;
  researchData?: any;
}

export async function generateContent(options: ContentOptions) {
  const { keyword, format, tone, language, researchData } = options;

  const systemPrompt = buildSystemPrompt(format, tone, language);
  const userPrompt = buildUserPrompt(keyword, researchData);

  // Use Claude for high-quality content
  const anthropic = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY,
  });

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 8192,
    system: systemPrompt,
    messages: [{ role: 'user', content: userPrompt }],
  });

  const content = extractTextContent(message.content);

  // Generate dual language if needed
  if (language === 'en') {
    const viVersion = await translateContent(content, 'vi');
    return { en: content, vi: viVersion };
  }

  return { [language]: content };
}

function buildSystemPrompt(format: ContentFormat, tone: ContentTone, language: Language) {
  const formatInstructions = {
    'toplist': 'Create a numbered list format with detailed explanations for each item',
    'pov': 'Write from a specific perspective or viewpoint, sharing opinions and insights',
    'case-study': 'Present a detailed case study with problem, solution, and results',
    'how-to': 'Create a step-by-step tutorial with actionable instructions',
  };

  const toneInstructions = {
    'expert': 'Use professional, authoritative language with industry terminology',
    'friendly': 'Use conversational, approachable language that connects with readers',
    'humorous': 'Incorporate wit and humor while maintaining informativeness',
  };

  const lang = language === 'vi' ? 'Vietnamese' : 'English';

  return `You are an expert content creator specializing in ${format} articles.
Tone: ${toneInstructions[tone]}
Language: ${lang}
Format: ${formatInstructions[format]}

Include:
- Compelling headline
- Clear structure with headings
- Data-backed insights
- Actionable takeaways
- SEO-optimized content`;
}

function buildUserPrompt(keyword: string, researchData?: any) {
  let prompt = `Create comprehensive content about: ${keyword}\n\n`;

  if (researchData) {
    prompt += `Use these research insights:\n${JSON.stringify(researchData, null, 2)}\n\n`;
  }

  prompt += `Ensure the content is:
- Original and valuable
- Well-researched with current data
- Engaging and shareable
- Optimized for social media distribution`;

  return prompt;
}

async function translateContent(content: string, targetLang: string) {
  const openai = new OpenAI({
    apiKey: process.env.OPENAI_API_KEY,
  });

  const response = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `Translate this content to ${targetLang} while maintaining tone, style, and formatting.`,
      },
      { role: 'user', content },
    ],
  });

  return response.choices[0].message.content;
}

function extractTextContent(content: any): string {
  if (Array.isArray(content)) {
    return content.map(block => block.text || '').join('\n');
  }
  return content.text || content.toString();
}
```

### 3. Video Generation with Remotion

Automatically render videos from content:

```typescript
// lib/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  title: string;
  content: string[];
  style: 'infographic' | 'talking-points' | 'slideshow';
  duration: number;
  aspectRatio: '16:9' | '9:16' | '1:1';
}

export async function generateVideo(config: VideoConfig) {
  const { title, content, style, duration, aspectRatio } = config;

  // Bundle Remotion composition
  const bundleLocation = await bundle({
    entryPoint: path.join(process.cwd(), 'remotion', 'index.tsx'),
    webpackOverride: (config) => config,
  });

  // Get composition details
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: style,
    inputProps: {
      title,
      content,
      aspectRatio,
    },
  });

  // Render video
  const outputPath = path.join(process.cwd(), 'public', 'videos', `${Date.now()}.mp4`);

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: {
      title,
      content,
      aspectRatio,
    },
  });

  return {
    path: outputPath,
    url: `/videos/${path.basename(outputPath)}`,
  };
}

// Example Remotion composition
// remotion/compositions/Infographic.tsx
import { AbsoluteFill, Sequence, useCurrentFrame, useVideoConfig } from 'remotion';

interface InfographicProps {
  title: string;
  content: string[];
}

export const Infographic: React.FC<InfographicProps> = ({ title, content }) => {
  const frame = useCurrentFrame();
  const { fps, durationInFrames } = useVideoConfig();

  const opacity = Math.min(1, frame / 30);

  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a1a' }}>
      <Sequence from={0} durationInFrames={60}>
        <AbsoluteFill
          style={{
            justifyContent: 'center',
            alignItems: 'center',
            opacity,
          }}
        >
          <h1 style={{ color: 'white', fontSize: 60, textAlign: 'center' }}>
            {title}
          </h1>
        </AbsoluteFill>
      </Sequence>

      {content.map((text, index) => (
        <Sequence
          key={index}
          from={60 + index * 90}
          durationInFrames={90}
        >
          <AbsoluteFill
            style={{
              justifyContent: 'center',
              alignItems: 'center',
              padding: 60,
            }}
          >
            <p style={{ color: 'white', fontSize: 40, textAlign: 'center' }}>
              {text}
            </p>
          </AbsoluteFill>
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};
```

### 4. Social Media Integration

Auto-post content to Facebook and other platforms:

```typescript
// lib/social/facebook.ts
interface FacebookPostOptions {
  message: string;
  link?: string;
  imageUrl?: string;
  videoUrl?: string;
  scheduledTime?: Date;
}

export async function postToFacebook(options: FacebookPostOptions) {
  const { message, link, imageUrl, videoUrl, scheduledTime } = options;

  const pageId = process.env.FACEBOOK_PAGE_ID;
  const accessToken = process.env.FACEBOOK_PAGE_ACCESS_TOKEN;

  let endpoint = `https://graph.facebook.com/v18.0/${pageId}/feed`;
  const params = new URLSearchParams({
    message,
    access_token: accessToken!,
  });

  if (link) params.append('link', link);

  // Handle media upload
  if (imageUrl) {
    endpoint = `https://graph.facebook.com/v18.0/${pageId}/photos`;
    params.append('url', imageUrl);
  } else if (videoUrl) {
    endpoint = `https://graph.facebook.com/v18.0/${pageId}/videos`;
    params.append('file_url', videoUrl);
  }

  // Schedule post if time provided
  if (scheduledTime) {
    params.append('published', 'false');
    params.append('scheduled_publish_time', Math.floor(scheduledTime.getTime() / 1000).toString());
  }

  const response = await fetch(endpoint, {
    method: 'POST',
    body: params,
  });

  const result = await response.json();

  if (!response.ok) {
    throw new Error(`Facebook API error: ${result.error?.message || 'Unknown error'}`);
  }

  return result;
}
```

## Complete Workflow Example

```typescript
// pages/api/pipeline/create.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { researchTopic } from '@/lib/research/crawler';
import { generateContent } from '@/lib/content/generator';
import { generateVideo } from '@/lib/video/renderer';
import { postToFacebook } from '@/lib/social/facebook';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  try {
    const { keyword, format, language, autoPost } = req.body;

    // Step 1: Research
    console.log('🔍 Researching topic...');
    const researchData = await researchTopic(keyword, '24h');

    // Step 2: Generate Content
    console.log('✍️ Generating content...');
    const content = await generateContent({
      keyword,
      format: format || 'toplist',
      tone: 'expert',
      language: language || 'en',
      researchData,
    });

    // Step 3: Generate Video
    console.log('🎬 Rendering video...');
    const contentPoints = content[language].split('\n').filter(Boolean).slice(0, 5);
    const video = await generateVideo({
      title: keyword,
      content: contentPoints,
      style: 'infographic',
      duration: 30,
      aspectRatio: '16:9',
    });

    // Step 4: Auto-post if requested
    let postResult = null;
    if (autoPost) {
      console.log('📤 Posting to social media...');
      postResult = await postToFacebook({
        message: content[language].substring(0, 500),
        videoUrl: `${process.env.NEXT_PUBLIC_APP_URL}${video.url}`,
      });
    }

    return res.status(200).json({
      success: true,
      data: {
        content,
        video,
        post: postResult,
      },
    });
  } catch (error) {
    console.error('Pipeline error:', error);
    return res.status(500).json({
      error: 'Pipeline execution failed',
      message: error instanceof Error ? error.message : 'Unknown error',
    });
  }
}
```

## Frontend Integration

```typescript
// components/ContentPipelineForm.tsx
'use client';

import { useState } from 'react';

export default function ContentPipelineForm() {
  const [keyword, setKeyword] = useState('');
  const [format, setFormat] = useState<'toplist' | 'pov' | 'case-study' | 'how-to'>('toplist');
  const [language, setLanguage] = useState<'en' | 'vi'>('en');
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState<any>(null);

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    setLoading(true);

    try {
      const response = await fetch('/api/pipeline/create', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          keyword,
          format,
          language,
          autoPost: true,
        }),
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
    <div className="max-w-2xl mx-auto p-6">
      <form onSubmit={handleSubmit} className="space-y-4">
        <div>
          <label className="block text-sm font-medium mb-2">
            Keyword / Topic
          </label>
          <input
            type="text"
            value={keyword}
            onChange={(e) => setKeyword(e.target.value)}
            className="w-full px-4 py-2 border rounded-lg"
            placeholder="e.g., AI Marketing Trends 2024"
            required
          />
        </div>

        <div>
          <label className="block text-sm font-medium mb-2">
            Content Format
          </label>
          <select
            value={format}
            onChange={(e) => setFormat(e.target.value as any)}
            className="w-full px-4 py-2 border rounded-lg"
          >
            <option value="toplist">Top List</option>
            <option value="pov">Point of View</option>
            <option value="case-study">Case Study</option>
            <option value="how-to">How-To Guide</option>
          </select>
        </div>

        <div>
          <label className="block text-sm font-medium mb-2">
            Language
          </label>
          <select
            value={language}
            onChange={(e) => setLanguage(e.target.value as any)}
            className="w-full px-4 py-2 border rounded-lg"
          >
            <option value="en">English</option>
            <option value="vi">Vietnamese</option>
          </select>
        </div>

        <button
          type="submit"
          disabled={loading}
          className="w-full bg-blue-600 text-white py-3 rounded-lg font-medium hover:bg-blue-700 disabled:opacity-50"
        >
          {loading ? 'Processing...' : 'Generate Content Pipeline'}
        </button>
      </form>

      {result && (
        <div className="mt-8 p-6 bg-green-50 rounded-lg">
          <h3 className="text-lg font-bold mb-4">✅ Pipeline Complete!</h3>
          <div className="space-y-2">
            <p><strong>Content:</strong> Generated ({Object.keys(result.data.content).join(', ')})</p>
            <p><strong>Video:</strong> <a href={result.data.video.url} target="_blank" className="text-blue-600">View Video</a></p>
            {result.data.post && (
              <p><strong>Posted:</strong> Facebook Post ID {result.data.post.id}</p>
            )}
          </div>
        </div>
      )}
    </div>
  );
}
```

## Common Patterns

### Batch Processing Multiple Keywords

```typescript
// lib/pipeline/batch.ts
export async function batchProcessKeywords(keywords: string[]) {
  const results = [];

  for (const keyword of keywords) {
    try {
      const result = await fetch('/api/pipeline/create', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          keyword,
          format: 'toplist',
          language: 'en',
          autoPost: false,
        }),
      });

      results.push({ keyword, success: true, data: await result.json() });
    } catch (error) {
      results.push({ keyword, success: false, error });
    }

    // Rate limiting delay
    await new Promise(resolve => setTimeout(resolve, 5000));
  }

  return results;
}
```

### Scheduled Content Publishing

```typescript
// lib/scheduler/cron.ts
import cron from 'node-cron';

export function scheduleContentGeneration() {
  // Run every day at 9 AM
  cron.schedule('0 9 * * *', async () => {
    console.log('🕐 Running scheduled content generation...');

    const topics = ['AI trends', 'Marketing automation', 'Social media strategy'];

    for (const topic of topics) {
      await fetch(`${process.env.NEXT_PUBLIC_APP_URL}/api/pipeline/create`, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          keyword: topic,
          format: 'toplist',
          language: 'en',
          autoPost: true,
        }),
      });
    }
  });
}
```

## Troubleshooting

### API Rate Limits

```typescript
// lib/utils/rate-limiter.ts
export class RateLimiter {
  private queue: Array<() => Promise<any>> = [];
  private processing = false;

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

      if (!this.processing) {
        this.process();
      }
    });
  }

  private async process() {
    this.processing = true;

    while (this.queue.length > 0) {
      const fn = this.queue.shift();
      if (fn) {
        await fn();
        await new Promise(resolve => setTimeout(resolve, 1000)); // 1s delay
      }
    }

    this.processing = false;
  }
}

// Usage
const limiter = new RateLimiter();
await limiter.add(() => generateContent(options));
```

### Video Rendering Errors

```typescript
// Ensure FFmpeg is installed
// Install on Ubuntu/Debian:
// sudo apt-get install ffmpeg

// Install on macOS:
// brew install ffmpeg

// Check FFmpeg in code
import { execSync } from 'child_process';

try {
  execSync('ffmpeg -version');
} catch (error) {
  console.error('FFmpeg not found. Please install FFmpeg to render videos.');
}
```

### Memory Issues with Large Content

```typescript
// Increase Node.js memory limit in package.json
{
  "scripts": {
    "dev": "NODE_OPTIONS='--max-old-space-size=4096' next dev",
    "build": "NODE_OPTIONS='--max-old-space-size=4096' next build"
  }
}
```

### API Key Validation

```typescript
// lib/utils/validate-env.ts
export function validateEnvironment() {
  const required = [
    'ANTHROPIC_API_KEY',
    'OPENAI_API_KEY',
    'RAPIDAPI_KEY',
    'FACEBOOK_PAGE_ACCESS_TOKEN',
  ];

  const missing = required.filter(key => !process.env[key]);

  if (missing.length > 0) {
    throw new Error(`Missing required environment variables: ${missing.join(', ')}`);
  }
}

// Call in API routes
validateEnvironment();
```

## Performance Optimization

### Caching Research Results

```typescript
// lib/cache/research.ts
import { Redis } from '@upstash/redis';

const redis = new Redis({
  url: process.env.UPSTASH_REDIS_URL!,
  token: process.env.UPSTASH_REDIS_TOKEN!,
});

export async function getCachedResearch(keyword: string) {
  const cached = await redis.get(`research:${keyword}`);
  return cached;
}

export async function setCachedResearch(keyword: string, data: any) {
  await redis.set(`research:${keyword}`, JSON.stringify(data), {
    ex: 86400, // 24 hours
  });
}
```

This skill enables AI agents to help developers build comprehensive automated marketing content pipelines with research, generation, video rendering, and social media distribution capabilities.
