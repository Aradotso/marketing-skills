---
name: marketing-pipeline-content-automation
description: Automated AI content pipeline for research, scriptwriting, and video generation using Claude/OpenAI and Remotion
triggers:
  - how do I automate content creation with AI research
  - set up the marketing pipeline for auto content generation
  - create AI-powered content workflow from research to video
  - configure Claude and OpenAI for automated content writing
  - generate videos automatically from written content
  - build content automation pipeline with Remotion
  - crawl news sources and generate marketing content
  - automate social media content creation end-to-end
---

# Marketing Pipeline Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the Ultimate AI Content Pipeline - a complete automation system that handles content research, scriptwriting, multi-format generation, and video rendering. The pipeline crawls real-time data from sources like TechCrunch, Twitter/X, and LinkedIn, then uses Claude/OpenAI to generate content in multiple formats and languages, finally rendering videos via Remotion.

## What This Project Does

The Marketing Pipeline is an end-to-end content automation system that:

- **Auto-researches** trending topics by crawling news sources and social media in real-time
- **Generates content** in multiple formats (listicles, POV articles, case studies, how-tos)
- **Multi-language support** (English & Vietnamese) with customizable tone
- **Auto-renders videos** and infographics using Remotion for social media platforms
- **Manages scheduling** for automated posting to Facebook pages and social channels

## Installation

### Prerequisites

```bash
# Node.js 18+ required
node --version

# Package manager (npm, yarn, or pnpm)
npm --version
```

### Setup Steps

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

# Copy environment template
cp .env.example .env
```

### Environment Configuration

Create a `.env` file with the following variables:

```bash
# AI APIs
OPENAI_API_KEY=your_openai_key_here
ANTHROPIC_API_KEY=your_claude_key_here

# Research/Crawling APIs
RAPIDAPI_KEY=your_rapidapi_key_here

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Remotion Configuration
REMOTION_AWS_ACCESS_KEY_ID=your_aws_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret

# Next.js
NEXT_PUBLIC_API_URL=http://localhost:3000
```

### Start Development Server

```bash
npm run dev
# or
yarn dev
# or
pnpm dev
```

Access the application at `http://localhost:3000`

## Key Components & Architecture

### 1. Research Module (Content Crawling)

```typescript
// lib/research/crawler.ts
import axios from 'axios';

interface NewsSource {
  url: string;
  selector: string;
  source: string;
}

export class ContentCrawler {
  private sources: NewsSource[] = [
    { url: 'https://techcrunch.com', selector: 'article', source: 'TechCrunch' },
    // Add more sources
  ];

  async crawlLatestNews(keyword: string, hours: number = 24): Promise<Article[]> {
    const articles: Article[] = [];
    
    for (const source of this.sources) {
      try {
        const response = await axios.get(source.url, {
          params: { q: keyword, timeframe: `${hours}h` },
          headers: {
            'X-RapidAPI-Key': process.env.RAPIDAPI_KEY,
            'X-RapidAPI-Host': 'news-api.rapidapi.com'
          }
        });
        
        articles.push(...this.parseArticles(response.data, source.source));
      } catch (error) {
        console.error(`Failed to crawl ${source.source}:`, error);
      }
    }
    
    return articles;
  }

  private parseArticles(data: any, source: string): Article[] {
    // Parse and structure article data
    return data.articles?.map((article: any) => ({
      title: article.title,
      url: article.url,
      publishedAt: article.publishedAt,
      source: source,
      summary: article.description
    })) || [];
  }
}
```

### 2. AI Content Generation

```typescript
// lib/ai/contentGenerator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

export type ContentFormat = 'toplist' | 'pov' | 'case-study' | 'how-to';
export type Language = 'en' | 'vi';
export type Tone = 'professional' | 'friendly' | 'humorous';

interface GenerationConfig {
  format: ContentFormat;
  language: Language;
  tone: Tone;
  targetAudience: string;
}

export class AIContentGenerator {
  private claude: Anthropic;
  private openai: OpenAI;

  constructor() {
    this.claude = new Anthropic({
      apiKey: process.env.ANTHROPIC_API_KEY,
    });
    
    this.openai = new OpenAI({
      apiKey: process.env.OPENAI_API_KEY,
    });
  }

  async generateContent(
    research: Article[],
    config: GenerationConfig
  ): Promise<string> {
    const prompt = this.buildPrompt(research, config);
    
    // Use Claude for long-form content
    if (config.format === 'case-study' || config.format === 'how-to') {
      return this.generateWithClaude(prompt, config);
    }
    
    // Use OpenAI for shorter formats
    return this.generateWithOpenAI(prompt, config);
  }

  private async generateWithClaude(
    prompt: string,
    config: GenerationConfig
  ): Promise<string> {
    const message = await this.claude.messages.create({
      model: 'claude-3-5-sonnet-20241022',
      max_tokens: 4096,
      messages: [{
        role: 'user',
        content: prompt
      }],
      temperature: 0.7,
    });

    return message.content[0].type === 'text' 
      ? message.content[0].text 
      : '';
  }

  private async generateWithOpenAI(
    prompt: string,
    config: GenerationConfig
  ): Promise<string> {
    const completion = await this.openai.chat.completions.create({
      model: 'gpt-4-turbo-preview',
      messages: [
        {
          role: 'system',
          content: `You are a ${config.tone} content writer creating ${config.format} content in ${config.language}.`
        },
        {
          role: 'user',
          content: prompt
        }
      ],
      temperature: 0.8,
      max_tokens: 2048,
    });

    return completion.choices[0]?.message?.content || '';
  }

  private buildPrompt(research: Article[], config: GenerationConfig): string {
    const articlesContext = research
      .map(a => `- ${a.title} (${a.source}): ${a.summary}`)
      .join('\n');

    const formatInstructions = {
      'toplist': 'Create a numbered list article with clear benefits for each point',
      'pov': 'Write from a unique perspective with strong opinions backed by data',
      'case-study': 'Analyze real examples with concrete results and takeaways',
      'how-to': 'Provide step-by-step instructions with actionable tips'
    };

    return `
Based on the following recent articles:

${articlesContext}

Create a ${config.format} article in ${config.language} with a ${config.tone} tone.
Target audience: ${config.targetAudience}

Instructions: ${formatInstructions[config.format]}

Include:
- Compelling headline
- Introduction hook
- Main content with data/examples from research
- Clear conclusion with call-to-action
`;
  }
}
```

### 3. Video Generation with Remotion

```typescript
// lib/video/videoRenderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import { getCompositions } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  title: string;
  content: string;
  style: 'reels' | 'tiktok' | 'shorts';
  duration: number;
}

export class VideoRenderer {
  private bundleLocation: string | null = null;

  async renderContentVideo(config: VideoConfig): Promise<string> {
    // Bundle Remotion composition
    if (!this.bundleLocation) {
      this.bundleLocation = await bundle({
        entryPoint: path.join(process.cwd(), 'remotion/index.ts'),
        webpackOverride: (config) => config,
      });
    }

    // Get composition
    const compositions = await getCompositions(this.bundleLocation);
    const composition = compositions.find(
      (c) => c.id === config.style
    );

    if (!composition) {
      throw new Error(`Composition ${config.style} not found`);
    }

    // Prepare output path
    const outputPath = path.join(
      process.cwd(),
      'public/videos',
      `${Date.now()}-${config.style}.mp4`
    );

    // Render video
    await renderMedia({
      composition,
      serveUrl: this.bundleLocation,
      codec: 'h264',
      outputLocation: outputPath,
      inputProps: {
        title: config.title,
        content: this.parseContentForVideo(config.content),
        duration: config.duration,
      },
    });

    return outputPath;
  }

  private parseContentForVideo(content: string): VideoSlide[] {
    // Parse content into video slides
    const sections = content.split('\n\n');
    return sections.map((section, index) => ({
      id: index,
      text: section.trim(),
      duration: 3, // seconds per slide
    }));
  }
}
```

### 4. Remotion Video Composition

```typescript
// remotion/compositions/Reels.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig } from 'remotion';
import { interpolate, spring } from 'remotion';

interface ReelsProps {
  title: string;
  content: VideoSlide[];
}

export const ReelsComposition: React.FC<ReelsProps> = ({ title, content }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  const titleOpacity = interpolate(frame, [0, 30], [0, 1], {
    extrapolateRight: 'clamp',
  });

  return (
    <AbsoluteFill style={{ backgroundColor: '#000' }}>
      {/* Title Section */}
      <div
        style={{
          position: 'absolute',
          top: '10%',
          width: '100%',
          textAlign: 'center',
          opacity: titleOpacity,
        }}
      >
        <h1 style={{ color: '#fff', fontSize: 48, fontWeight: 'bold' }}>
          {title}
        </h1>
      </div>

      {/* Content Slides */}
      <div
        style={{
          position: 'absolute',
          top: '30%',
          width: '100%',
          padding: '0 40px',
        }}
      >
        {content.map((slide, index) => {
          const startFrame = index * fps * slide.duration + 30;
          const endFrame = startFrame + fps * slide.duration;
          
          const slideOpacity = interpolate(
            frame,
            [startFrame, startFrame + 15, endFrame - 15, endFrame],
            [0, 1, 1, 0],
            { extrapolateLeft: 'clamp', extrapolateRight: 'clamp' }
          );

          const slideY = spring({
            frame: frame - startFrame,
            fps,
            config: { damping: 100 },
          });

          return (
            <div
              key={slide.id}
              style={{
                opacity: slideOpacity,
                transform: `translateY(${interpolate(slideY, [0, 1], [50, 0])}px)`,
                color: '#fff',
                fontSize: 32,
                marginBottom: 20,
                lineHeight: 1.5,
              }}
            >
              {slide.text}
            </div>
          );
        })}
      </div>
    </AbsoluteFill>
  );
};
```

## Complete Pipeline Workflow

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { ContentCrawler } from '@/lib/research/crawler';
import { AIContentGenerator } from '@/lib/ai/contentGenerator';
import { VideoRenderer } from '@/lib/video/videoRenderer';

export async function POST(request: NextRequest) {
  try {
    const { keyword, format, language, tone, includeVideo } = await request.json();

    // Step 1: Research
    const crawler = new ContentCrawler();
    const articles = await crawler.crawlLatestNews(keyword, 24);

    if (articles.length === 0) {
      return NextResponse.json(
        { error: 'No articles found for keyword' },
        { status: 404 }
      );
    }

    // Step 2: Generate Content
    const generator = new AIContentGenerator();
    const content = await generator.generateContent(articles, {
      format,
      language,
      tone,
      targetAudience: 'marketers and content creators',
    });

    // Step 3: Generate Video (optional)
    let videoPath = null;
    if (includeVideo) {
      const renderer = new VideoRenderer();
      videoPath = await renderer.renderContentVideo({
        title: `${keyword} - ${format}`,
        content,
        style: 'reels',
        duration: 30,
      });
    }

    return NextResponse.json({
      success: true,
      data: {
        content,
        videoPath,
        sourceArticles: articles.length,
      },
    });
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
// app/page.tsx
'use client';

import { useState } from 'react';

export default function ContentPipeline() {
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState<any>(null);

  const handleGenerate = async (e: React.FormEvent<HTMLFormElement>) => {
    e.preventDefault();
    setLoading(true);

    const formData = new FormData(e.currentTarget);
    
    try {
      const response = await fetch('/api/generate', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          keyword: formData.get('keyword'),
          format: formData.get('format'),
          language: formData.get('language'),
          tone: formData.get('tone'),
          includeVideo: formData.get('includeVideo') === 'on',
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
    <div className="container mx-auto p-8">
      <h1 className="text-4xl font-bold mb-8">AI Content Pipeline</h1>
      
      <form onSubmit={handleGenerate} className="space-y-4">
        <input
          name="keyword"
          placeholder="Enter keyword (e.g., AI Marketing)"
          className="w-full p-3 border rounded"
          required
        />
        
        <select name="format" className="w-full p-3 border rounded">
          <option value="toplist">Top List</option>
          <option value="pov">POV Article</option>
          <option value="case-study">Case Study</option>
          <option value="how-to">How-To Guide</option>
        </select>

        <select name="language" className="w-full p-3 border rounded">
          <option value="en">English</option>
          <option value="vi">Vietnamese</option>
        </select>

        <select name="tone" className="w-full p-3 border rounded">
          <option value="professional">Professional</option>
          <option value="friendly">Friendly</option>
          <option value="humorous">Humorous</option>
        </select>

        <label className="flex items-center space-x-2">
          <input type="checkbox" name="includeVideo" />
          <span>Generate Video</span>
        </label>

        <button
          type="submit"
          disabled={loading}
          className="w-full bg-blue-600 text-white p-3 rounded disabled:bg-gray-400"
        >
          {loading ? 'Generating...' : 'Generate Content'}
        </button>
      </form>

      {result?.data && (
        <div className="mt-8 space-y-4">
          <div className="p-6 bg-white rounded shadow">
            <h2 className="text-2xl font-bold mb-4">Generated Content</h2>
            <div className="prose max-w-none">
              {result.data.content}
            </div>
          </div>

          {result.data.videoPath && (
            <div className="p-6 bg-white rounded shadow">
              <h2 className="text-2xl font-bold mb-4">Video</h2>
              <video controls className="w-full">
                <source src={result.data.videoPath} type="video/mp4" />
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
async function generateBatchContent(keywords: string[]) {
  const crawler = new ContentCrawler();
  const generator = new AIContentGenerator();
  
  const results = await Promise.all(
    keywords.map(async (keyword) => {
      const articles = await crawler.crawlLatestNews(keyword, 24);
      const content = await generator.generateContent(articles, {
        format: 'toplist',
        language: 'en',
        tone: 'professional',
        targetAudience: 'marketers',
      });
      
      return { keyword, content, articles: articles.length };
    })
  );
  
  return results;
}
```

### Scheduled Content Pipeline

```typescript
// lib/scheduler/contentScheduler.ts
import cron from 'node-cron';

export class ContentScheduler {
  start() {
    // Run every day at 9 AM
    cron.schedule('0 9 * * *', async () => {
      console.log('Running daily content generation...');
      
      const keywords = ['AI Marketing', 'Content Strategy', 'SEO Tips'];
      const results = await generateBatchContent(keywords);
      
      // Auto-post to social media or save to database
      await this.publishContent(results);
    });
  }

  private async publishContent(results: any[]) {
    // Implementation for posting to Facebook, etc.
  }
}
```

## Troubleshooting

### API Rate Limits

```typescript
// lib/utils/rateLimiter.ts
export class RateLimiter {
  private queue: Promise<any> = Promise.resolve();
  
  async throttle<T>(fn: () => Promise<T>, delayMs: number = 1000): Promise<T> {
    this.queue = this.queue.then(
      () => new Promise((resolve) => setTimeout(resolve, delayMs))
    );
    
    return this.queue.then(() => fn());
  }
}

// Usage
const limiter = new RateLimiter();
const content = await limiter.throttle(() => 
  generator.generateContent(articles, config),
  2000
);
```

### Video Rendering Timeout

```typescript
// Increase timeout for large videos
await renderMedia({
  composition,
  serveUrl: bundleLocation,
  codec: 'h264',
  outputLocation: outputPath,
  timeoutInMilliseconds: 120000, // 2 minutes
  inputProps: videoProps,
});
```

### Memory Issues with Large Content

```typescript
// Stream content generation in chunks
async function* generateContentStream(research: Article[]) {
  const chunks = chunkArray(research, 5);
  
  for (const chunk of chunks) {
    const partial = await generator.generateContent(chunk, config);
    yield partial;
  }
}
```

This skill provides comprehensive coverage of the marketing pipeline automation system, enabling AI agents to help developers implement automated content workflows with research, AI generation, and video rendering capabilities.
