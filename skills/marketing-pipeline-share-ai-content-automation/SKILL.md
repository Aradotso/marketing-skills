---
name: marketing-pipeline-share-ai-content-automation
description: Automated AI content pipeline for research, scriptwriting, and video generation using Claude, OpenAI, and Remotion
triggers:
  - how do I automate content creation with AI research
  - generate video content from text automatically
  - set up AI content pipeline with Claude and OpenAI
  - crawl news sources and create content with AI
  - automate social media content with video generation
  - research and generate multi-format content with AI
  - build automated content marketing pipeline
  - create videos from blog posts using Remotion
---

# Marketing Pipeline Share - AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

Marketing Pipeline Share is a comprehensive AI-powered content automation system that handles the complete content creation lifecycle: from research and data crawling, to multi-format content generation, to automated video rendering. It integrates Claude 3, OpenAI, RapidAPI for news crawling, and Remotion for video generation to create a complete content production pipeline.

**Key capabilities:**
- Auto-crawl trending news from TechCrunch, a16z, Twitter/X, LinkedIn
- Generate content in multiple formats (toplist, POV, case studies, how-to)
- Bilingual support (English/Vietnamese) with customizable tone
- Automatic video rendering optimized for Reels/TikTok/Shorts
- Next.js-based UI for easy management

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

### Environment Configuration

Create a `.env.local` file in the root directory:

```env
# AI Model APIs
OPENAI_API_KEY=your_openai_api_key
ANTHROPIC_API_KEY=your_claude_api_key

# News/Research APIs
RAPIDAPI_KEY=your_rapidapi_key
TECHCRUNCH_API_KEY=your_techcrunch_key

# Remotion (Video Generation)
REMOTION_LICENSE_KEY=your_remotion_license

# Application
NEXT_PUBLIC_APP_URL=http://localhost:3000
DATABASE_URL=your_database_connection_string

# Optional: Social Media Auto-posting
FACEBOOK_PAGE_TOKEN=your_facebook_token
LINKEDIN_ACCESS_TOKEN=your_linkedin_token
```

## Running the Application

```bash
# Development mode
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Render videos (Remotion)
npm run render
```

Access the application at `http://localhost:3000`

## Core Architecture

### 1. Research & Crawling Module

The system automatically scrapes news sources for fresh content:

```typescript
// lib/research/crawler.ts
import { NewsAPIClient } from './apis/newsapi';
import { TechCrunchScraper } from './scrapers/techcrunch';

interface ResearchParams {
  keyword: string;
  timeRange?: '24h' | '7d' | '30d';
  sources?: string[];
  language?: 'en' | 'vi';
}

export async function conductResearch(params: ResearchParams) {
  const { keyword, timeRange = '24h', sources = ['techcrunch', 'twitter'], language = 'en' } = params;
  
  // Fetch from multiple sources
  const results = await Promise.all([
    fetchTechCrunchArticles(keyword, timeRange),
    fetchTwitterPosts(keyword, timeRange),
    fetchLinkedInPosts(keyword, timeRange),
  ]);

  // Aggregate and analyze
  const insights = await analyzeResearchData(results.flat(), language);
  
  return {
    rawData: results.flat(),
    insights,
    trends: extractTrends(results.flat()),
    statistics: compileStatistics(results.flat()),
  };
}

async function fetchTechCrunchArticles(keyword: string, timeRange: string) {
  const scraper = new TechCrunchScraper(process.env.RAPIDAPI_KEY);
  return await scraper.search(keyword, {
    publishedWithin: timeRange,
    limit: 20,
  });
}
```

### 2. AI Content Generation

Generate multi-format content using Claude or OpenAI:

```typescript
// lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

interface ContentConfig {
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  tone: 'expert' | 'friendly' | 'humorous';
  language: 'en' | 'vi';
  researchData: any;
}

export class ContentGenerator {
  private anthropic: Anthropic;
  private openai: OpenAI;

  constructor() {
    this.anthropic = new Anthropic({
      apiKey: process.env.ANTHROPIC_API_KEY,
    });
    this.openai = new OpenAI({
      apiKey: process.env.OPENAI_API_KEY,
    });
  }

  async generateContent(config: ContentConfig): Promise<string> {
    const prompt = this.buildPrompt(config);

    // Use Claude for Vietnamese, OpenAI for English (or customize)
    if (config.language === 'vi') {
      return await this.generateWithClaude(prompt);
    } else {
      return await this.generateWithOpenAI(prompt);
    }
  }

  private async generateWithClaude(prompt: string): Promise<string> {
    const message = await this.anthropic.messages.create({
      model: 'claude-3-5-sonnet-20241022',
      max_tokens: 4096,
      messages: [
        {
          role: 'user',
          content: prompt,
        },
      ],
    });

    return message.content[0].type === 'text' ? message.content[0].text : '';
  }

  private async generateWithOpenAI(prompt: string): Promise<string> {
    const completion = await this.openai.chat.completions.create({
      model: 'gpt-4-turbo-preview',
      messages: [
        {
          role: 'system',
          content: 'You are an expert content writer specializing in marketing and technical content.',
        },
        {
          role: 'user',
          content: prompt,
        },
      ],
      max_tokens: 4096,
    });

    return completion.choices[0].message.content || '';
  }

  private buildPrompt(config: ContentConfig): string {
    const formatInstructions = {
      'toplist': 'Create a compelling top 10 list article',
      'pov': 'Write a thought-provoking point-of-view article',
      'case-study': 'Develop a detailed case study with data and analysis',
      'how-to': 'Write a step-by-step tutorial guide',
    };

    return `
${formatInstructions[config.format]} based on this research data:

${JSON.stringify(config.researchData.insights, null, 2)}

Requirements:
- Tone: ${config.tone}
- Language: ${config.language}
- Include statistics and data points from the research
- Make it engaging and actionable
- Add relevant examples and case studies
- Format with proper headings and bullet points
`;
  }
}
```

### 3. Video Generation with Remotion

Automatically render videos from content:

```typescript
// lib/video/render.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  content: string;
  title: string;
  format: 'reels' | 'tiktok' | 'shorts'; // 9:16 aspect ratio
  duration?: number;
}

export async function renderContentVideo(config: VideoConfig): Promise<string> {
  const { content, title, format, duration = 30 } = config;

  // Bundle Remotion composition
  const bundleLocation = await bundle({
    entryPoint: path.join(process.cwd(), 'remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  // Select composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title,
      content: parseContentForVideo(content),
      format,
    },
  });

  // Render video
  const outputLocation = path.join(
    process.cwd(),
    'public',
    'videos',
    `${sanitizeFilename(title)}-${Date.now()}.mp4`
  );

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: {
      title,
      content: parseContentForVideo(content),
      format,
    },
  });

  return outputLocation;
}

function parseContentForVideo(content: string) {
  // Extract key points for video scenes
  const lines = content.split('\n').filter(line => line.trim());
  const scenes = [];
  
  for (let i = 0; i < Math.min(lines.length, 10); i++) {
    scenes.push({
      text: lines[i],
      duration: 3, // seconds per scene
    });
  }
  
  return scenes;
}

function sanitizeFilename(name: string): string {
  return name.replace(/[^a-z0-9]/gi, '-').toLowerCase();
}
```

### 4. Remotion Video Component

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, Sequence, useCurrentFrame, interpolate } from 'remotion';
import React from 'react';

interface Scene {
  text: string;
  duration: number;
}

interface ContentVideoProps {
  title: string;
  content: Scene[];
  format: 'reels' | 'tiktok' | 'shorts';
}

export const ContentVideo: React.FC<ContentVideoProps> = ({ title, content, format }) => {
  const frame = useCurrentFrame();
  const fps = 30;

  return (
    <AbsoluteFill style={{ backgroundColor: '#000' }}>
      {/* Title Scene */}
      <Sequence durationInFrames={fps * 3}>
        <TitleScene title={title} />
      </Sequence>

      {/* Content Scenes */}
      {content.map((scene, index) => (
        <Sequence
          key={index}
          from={(3 + index * scene.duration) * fps}
          durationInFrames={scene.duration * fps}
        >
          <ContentScene text={scene.text} index={index} />
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};

const TitleScene: React.FC<{ title: string }> = ({ title }) => {
  const frame = useCurrentFrame();
  const opacity = interpolate(frame, [0, 15, 60, 75], [0, 1, 1, 0]);
  const scale = interpolate(frame, [0, 15], [0.8, 1], { extrapolateRight: 'clamp' });

  return (
    <AbsoluteFill style={{ justifyContent: 'center', alignItems: 'center' }}>
      <h1
        style={{
          color: '#fff',
          fontSize: 60,
          textAlign: 'center',
          opacity,
          transform: `scale(${scale})`,
          padding: '0 40px',
        }}
      >
        {title}
      </h1>
    </AbsoluteFill>
  );
};

const ContentScene: React.FC<{ text: string; index: number }> = ({ text, index }) => {
  const frame = useCurrentFrame();
  const opacity = interpolate(frame, [0, 10, 80, 90], [0, 1, 1, 0]);

  return (
    <AbsoluteFill style={{ justifyContent: 'center', alignItems: 'center', padding: '0 60px' }}>
      <div style={{ opacity }}>
        <div
          style={{
            color: '#FFD700',
            fontSize: 40,
            fontWeight: 'bold',
            marginBottom: 20,
          }}
        >
          #{index + 1}
        </div>
        <p
          style={{
            color: '#fff',
            fontSize: 32,
            textAlign: 'center',
            lineHeight: 1.5,
          }}
        >
          {text}
        </p>
      </div>
    </AbsoluteFill>
  );
};
```

## Complete Workflow Example

```typescript
// pages/api/generate-content.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { conductResearch } from '@/lib/research/crawler';
import { ContentGenerator } from '@/lib/ai/content-generator';
import { renderContentVideo } from '@/lib/video/render';

export default async function handler(req: NextApiRequest, res: NextApiResponse) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  try {
    const { keyword, format, language, generateVideo } = req.body;

    // Step 1: Research
    console.log('🔍 Conducting research...');
    const research = await conductResearch({
      keyword,
      timeRange: '24h',
      language,
    });

    // Step 2: Generate Content
    console.log('✍️ Generating content...');
    const generator = new ContentGenerator();
    const content = await generator.generateContent({
      format,
      tone: 'expert',
      language,
      researchData: research,
    });

    // Step 3: Render Video (optional)
    let videoUrl = null;
    if (generateVideo) {
      console.log('🎬 Rendering video...');
      const videoPath = await renderContentVideo({
        content,
        title: keyword,
        format: 'reels',
      });
      videoUrl = `/videos/${path.basename(videoPath)}`;
    }

    return res.status(200).json({
      success: true,
      data: {
        content,
        videoUrl,
        research: {
          sources: research.rawData.length,
          insights: research.insights,
        },
      },
    });
  } catch (error) {
    console.error('Error in content generation:', error);
    return res.status(500).json({
      error: 'Content generation failed',
      message: error.message,
    });
  }
}
```

## Client-Side Usage

```typescript
// components/ContentGenerator.tsx
import { useState } from 'react';

export function ContentGeneratorForm() {
  const [keyword, setKeyword] = useState('');
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState(null);

  const handleGenerate = async () => {
    setLoading(true);
    try {
      const response = await fetch('/api/generate-content', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          keyword,
          format: 'toplist',
          language: 'en',
          generateVideo: true,
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
    <div className="p-6">
      <input
        type="text"
        value={keyword}
        onChange={(e) => setKeyword(e.target.value)}
        placeholder="Enter keyword..."
        className="border p-2 w-full mb-4"
      />
      <button
        onClick={handleGenerate}
        disabled={loading}
        className="bg-blue-500 text-white px-4 py-2 rounded"
      >
        {loading ? 'Generating...' : 'Generate Content'}
      </button>

      {result && (
        <div className="mt-6">
          <h3>Generated Content:</h3>
          <div className="prose">{result.data.content}</div>
          {result.data.videoUrl && (
            <video src={result.data.videoUrl} controls className="mt-4" />
          )}
        </div>
      )}
    </div>
  );
}
```

## Configuration Patterns

### Customizing AI Models

```typescript
// config/ai-models.ts
export const AI_MODEL_CONFIG = {
  research: {
    model: 'gpt-4-turbo-preview',
    temperature: 0.3,
    maxTokens: 2048,
  },
  contentGeneration: {
    english: {
      provider: 'openai',
      model: 'gpt-4-turbo-preview',
      temperature: 0.7,
    },
    vietnamese: {
      provider: 'anthropic',
      model: 'claude-3-5-sonnet-20241022',
      temperature: 0.7,
    },
  },
  formats: {
    toplist: { minLength: 1500, maxLength: 3000 },
    pov: { minLength: 1000, maxLength: 2000 },
    'case-study': { minLength: 2000, maxLength: 4000 },
    'how-to': { minLength: 1500, maxLength: 3000 },
  },
};
```

### News Source Configuration

```typescript
// config/news-sources.ts
export const NEWS_SOURCES = {
  techcrunch: {
    enabled: true,
    priority: 1,
    apiEndpoint: 'https://techcrunch-api.rapidapi.com',
    rateLimitPerHour: 100,
  },
  twitter: {
    enabled: true,
    priority: 2,
    hashtags: ['#tech', '#ai', '#startup'],
  },
  linkedin: {
    enabled: true,
    priority: 3,
    industries: ['technology', 'marketing', 'ai'],
  },
};
```

## Troubleshooting

### API Rate Limits

```typescript
// lib/utils/rate-limiter.ts
import { RateLimiter } from 'limiter';

export const rateLimiters = {
  openai: new RateLimiter({ tokensPerInterval: 50, interval: 'minute' }),
  anthropic: new RateLimiter({ tokensPerInterval: 50, interval: 'minute' }),
  rapidapi: new RateLimiter({ tokensPerInterval: 100, interval: 'hour' }),
};

export async function withRateLimit<T>(
  limiter: RateLimiter,
  fn: () => Promise<T>
): Promise<T> {
  await limiter.removeTokens(1);
  return fn();
}
```

### Video Rendering Issues

If videos fail to render:

```bash
# Install required dependencies
npm install @remotion/bundler @remotion/renderer @remotion/cli

# Check Remotion configuration
npx remotion preview remotion/index.ts

# Verify FFmpeg installation
ffmpeg -version
```

### Memory Issues with Large Content

```typescript
// Implement content chunking for large datasets
async function processLargeResearch(data: any[]) {
  const CHUNK_SIZE = 10;
  const results = [];

  for (let i = 0; i < data.length; i += CHUNK_SIZE) {
    const chunk = data.slice(i, i + CHUNK_SIZE);
    const processed = await processChunk(chunk);
    results.push(...processed);
    
    // Allow garbage collection
    await new Promise(resolve => setImmediate(resolve));
  }

  return results;
}
```

### Database Connection Issues

```typescript
// lib/db/connection.ts
import { PrismaClient } from '@prisma/client';

const globalForPrisma = global as unknown as { prisma: PrismaClient };

export const prisma =
  globalForPrisma.prisma ||
  new PrismaClient({
    log: process.env.NODE_ENV === 'development' ? ['query', 'error', 'warn'] : ['error'],
  });

if (process.env.NODE_ENV !== 'production') globalForPrisma.prisma = prisma;
```

## Best Practices

1. **Always validate research data** before passing to AI models
2. **Cache research results** to avoid redundant API calls
3. **Implement retry logic** for API failures
4. **Use streaming responses** for long content generation
5. **Optimize video rendering** by pre-processing content
6. **Monitor API costs** with usage tracking middleware
7. **Implement proper error boundaries** in React components
8. **Use background jobs** for video rendering to avoid timeouts
