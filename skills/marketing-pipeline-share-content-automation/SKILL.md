---
name: marketing-pipeline-share-content-automation
description: Automate content creation from research to video generation using AI-powered pipeline with Claude, OpenAI, and Remotion
triggers:
  - automate content creation from research to video
  - set up AI content pipeline with Claude and OpenAI
  - generate videos from articles automatically
  - crawl news and create content with AI
  - build automated marketing content system
  - create multi-format content with AI pipeline
  - generate social media videos from text
  - automate research and scriptwriting workflow
---

# Marketing Pipeline Share - AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection

This skill enables AI coding agents to help developers use **Marketing Pipeline Share**, an end-to-end AI content automation system that handles research, scriptwriting, and video generation. The pipeline crawls news sources, generates content in multiple formats using Claude/OpenAI, and automatically renders videos using Remotion.

## What This Project Does

Marketing Pipeline Share is a TypeScript-based content automation pipeline that:

- **Auto-crawls** news from TechCrunch, a16z, Twitter, LinkedIn for fresh insights
- **Generates content** in multiple formats (toplist, POV, case study, how-to) using Claude 3 or OpenAI
- **Multi-language support** with parallel English/Vietnamese content generation
- **Auto-renders videos** from text content using Remotion for Reels, TikTok, Shorts
- **Flexible architecture** with Next.js frontend and modular backend services

## Installation

### Prerequisites

```bash
# Node.js 18+ and pnpm required
node --version  # Should be 18+
pnpm --version  # Install with: npm i -g pnpm
```

### Setup Steps

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
pnpm install

# Set up environment variables
cp .env.example .env
```

### Environment Configuration

Create `.env` file with required API keys:

```bash
# AI Services
ANTHROPIC_API_KEY=your_claude_key_here
OPENAI_API_KEY=your_openai_key_here

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key_here

# Database (if applicable)
DATABASE_URL=postgresql://user:password@localhost:5432/content_db

# Next.js
NEXT_PUBLIC_API_URL=http://localhost:3000
```

## Key Commands

### Development

```bash
# Start development server
pnpm dev

# Build for production
pnpm build

# Start production server
pnpm start

# Run research crawler
pnpm research:crawl

# Generate content from research
pnpm content:generate

# Render videos
pnpm video:render
```

## Core Architecture

### Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app router
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI service integrations
│   │   ├── crawler/     # News crawling modules
│   │   ├── content/     # Content generation
│   │   └── video/       # Remotion video rendering
│   └── types/           # TypeScript definitions
├── remotion/            # Video templates
└── public/              # Static assets
```

## Research & Crawling

### News Crawler Implementation

```typescript
// src/lib/crawler/news-crawler.ts
import axios from 'axios';

interface NewsSource {
  name: string;
  url: string;
  selector: string;
}

export class NewsCrawler {
  private sources: NewsSource[] = [
    { name: 'TechCrunch', url: 'https://techcrunch.com', selector: '.post-block' },
    { name: 'a16z', url: 'https://a16z.com/posts', selector: '.post' }
  ];

  async crawlLatestNews(keyword: string, hours: number = 24): Promise<Article[]> {
    const articles: Article[] = [];
    const cutoffTime = Date.now() - (hours * 60 * 60 * 1000);

    for (const source of this.sources) {
      try {
        const response = await axios.get(source.url, {
          headers: { 'User-Agent': 'Mozilla/5.0' }
        });
        
        // Parse and filter articles
        const parsed = this.parseArticles(response.data, source.selector);
        const filtered = parsed.filter(a => 
          a.timestamp > cutoffTime && 
          this.matchesKeyword(a.content, keyword)
        );
        
        articles.push(...filtered);
      } catch (error) {
        console.error(`Failed to crawl ${source.name}:`, error);
      }
    }

    return articles;
  }

  private matchesKeyword(content: string, keyword: string): boolean {
    return content.toLowerCase().includes(keyword.toLowerCase());
  }

  private parseArticles(html: string, selector: string): Article[] {
    // Implement HTML parsing logic
    return [];
  }
}

interface Article {
  title: string;
  content: string;
  url: string;
  timestamp: number;
  source: string;
}
```

### Using the Crawler

```typescript
// Example usage in API route or service
import { NewsCrawler } from '@/lib/crawler/news-crawler';

const crawler = new NewsCrawler();
const articles = await crawler.crawlLatestNews('AI automation', 24);

console.log(`Found ${articles.length} articles`);
```

## AI Content Generation

### Claude Integration

```typescript
// src/lib/ai/claude-generator.ts
import Anthropic from '@anthropic-ai/sdk';

export class ClaudeContentGenerator {
  private client: Anthropic;

  constructor() {
    this.client = new Anthropic({
      apiKey: process.env.ANTHROPIC_API_KEY
    });
  }

  async generateContent(
    articles: Article[], 
    format: ContentFormat,
    language: 'en' | 'vi' = 'en'
  ): Promise<string> {
    const prompt = this.buildPrompt(articles, format, language);

    const message = await this.client.messages.create({
      model: 'claude-3-5-sonnet-20241022',
      max_tokens: 4096,
      messages: [{
        role: 'user',
        content: prompt
      }]
    });

    return message.content[0].type === 'text' 
      ? message.content[0].text 
      : '';
  }

  private buildPrompt(
    articles: Article[], 
    format: ContentFormat,
    language: string
  ): string {
    const languageInstruction = language === 'vi' 
      ? 'Write in Vietnamese' 
      : 'Write in English';

    const formatInstructions = {
      toplist: 'Create a numbered list ranking the top insights',
      pov: 'Write from a first-person perspective with strong opinions',
      casestudy: 'Structure as: Problem, Solution, Results',
      howto: 'Create a step-by-step tutorial format'
    };

    return `
${languageInstruction}.

Format: ${formatInstructions[format]}

Based on these recent articles:
${articles.map(a => `- ${a.title}: ${a.content.slice(0, 200)}...`).join('\n')}

Generate comprehensive content that:
1. Synthesizes key insights from all sources
2. Includes specific data points and quotes
3. Provides actionable takeaways
4. Maintains an engaging, professional tone
`;
  }
}

type ContentFormat = 'toplist' | 'pov' | 'casestudy' | 'howto';
```

### OpenAI Alternative

```typescript
// src/lib/ai/openai-generator.ts
import OpenAI from 'openai';

export class OpenAIContentGenerator {
  private client: OpenAI;

  constructor() {
    this.client = new OpenAI({
      apiKey: process.env.OPENAI_API_KEY
    });
  }

  async generateContent(
    articles: Article[],
    format: string,
    language: 'en' | 'vi' = 'en'
  ): Promise<string> {
    const completion = await this.client.chat.completions.create({
      model: 'gpt-4-turbo-preview',
      messages: [
        {
          role: 'system',
          content: `You are an expert content writer specializing in ${format} format.`
        },
        {
          role: 'user',
          content: this.buildPrompt(articles, format, language)
        }
      ],
      temperature: 0.7,
      max_tokens: 3000
    });

    return completion.choices[0]?.message?.content || '';
  }

  private buildPrompt(articles: Article[], format: string, language: string): string {
    // Similar to Claude implementation
    return `Generate ${language} content in ${format} format...`;
  }
}
```

## Video Generation with Remotion

### Video Composition Setup

```typescript
// remotion/compositions/ArticleVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig } from 'remotion';

interface ArticleVideoProps {
  title: string;
  points: string[];
  backgroundColor: string;
  textColor: string;
}

export const ArticleVideo: React.FC<ArticleVideoProps> = ({
  title,
  points,
  backgroundColor,
  textColor
}) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  const opacity = Math.min(1, frame / (fps * 0.5));

  return (
    <AbsoluteFill style={{ backgroundColor }}>
      <div style={{ 
        padding: 60, 
        opacity,
        color: textColor 
      }}>
        <h1 style={{ fontSize: 72, marginBottom: 40 }}>
          {title}
        </h1>
        
        {points.map((point, index) => {
          const pointStart = fps * (2 + index * 3);
          const pointOpacity = frame > pointStart 
            ? Math.min(1, (frame - pointStart) / fps) 
            : 0;
          
          return (
            <div 
              key={index}
              style={{ 
                fontSize: 36, 
                marginBottom: 20,
                opacity: pointOpacity,
                transform: `translateY(${Math.max(0, 20 - (frame - pointStart))}px)`
              }}
            >
              {index + 1}. {point}
            </div>
          );
        })}
      </div>
    </AbsoluteFill>
  );
};
```

### Registering Compositions

```typescript
// remotion/index.ts
import { registerRoot } from 'remotion';
import { ArticleVideo } from './compositions/ArticleVideo';

export const RemotionRoot: React.FC = () => {
  return (
    <>
      <Composition
        id="ArticleVideo"
        component={ArticleVideo}
        durationInFrames={450}
        fps={30}
        width={1080}
        height={1920}
        defaultProps={{
          title: 'Top 5 AI Trends',
          points: [
            'AI automation is transforming content creation',
            'Claude 3 offers superior reasoning capabilities',
            'Video content dominates social media',
            'Multi-language support is essential',
            'Integration beats standalone tools'
          ],
          backgroundColor: '#1a1a2e',
          textColor: '#eee'
        }}
      />
    </>
  );
};

registerRoot(RemotionRoot);
```

### Rendering Videos Programmatically

```typescript
// src/lib/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

export class VideoRenderer {
  async renderArticleVideo(
    content: GeneratedContent,
    outputPath: string
  ): Promise<string> {
    const compositionId = 'ArticleVideo';
    
    // Bundle the Remotion project
    const bundleLocation = await bundle({
      entryPoint: path.resolve('./remotion/index.ts'),
      webpackOverride: (config) => config
    });

    // Get composition
    const composition = await selectComposition({
      serveUrl: bundleLocation,
      id: compositionId,
      inputProps: {
        title: content.title,
        points: content.keyPoints,
        backgroundColor: '#1a1a2e',
        textColor: '#eee'
      }
    });

    // Render video
    await renderMedia({
      composition,
      serveUrl: bundleLocation,
      codec: 'h264',
      outputLocation: outputPath,
      inputProps: composition.defaultProps
    });

    return outputPath;
  }
}

interface GeneratedContent {
  title: string;
  keyPoints: string[];
  fullText: string;
}
```

## Complete Pipeline Workflow

### End-to-End Content Generation

```typescript
// src/lib/pipeline/content-pipeline.ts
import { NewsCrawler } from '@/lib/crawler/news-crawler';
import { ClaudeContentGenerator } from '@/lib/ai/claude-generator';
import { VideoRenderer } from '@/lib/video/renderer';

export class ContentPipeline {
  private crawler: NewsCrawler;
  private contentGenerator: ClaudeContentGenerator;
  private videoRenderer: VideoRenderer;

  constructor() {
    this.crawler = new NewsCrawler();
    this.contentGenerator = new ClaudeContentGenerator();
    this.videoRenderer = new VideoRenderer();
  }

  async runPipeline(
    keyword: string,
    format: ContentFormat,
    languages: ('en' | 'vi')[] = ['en']
  ): Promise<PipelineResult> {
    console.log(`Starting pipeline for keyword: ${keyword}`);

    // Step 1: Research - Crawl latest news
    const articles = await this.crawler.crawlLatestNews(keyword, 24);
    console.log(`Found ${articles.length} articles`);

    if (articles.length === 0) {
      throw new Error('No articles found for keyword');
    }

    // Step 2: Generate content in multiple languages
    const contentByLanguage: Record<string, string> = {};
    
    for (const lang of languages) {
      const content = await this.contentGenerator.generateContent(
        articles,
        format,
        lang
      );
      contentByLanguage[lang] = content;
      console.log(`Generated ${lang} content: ${content.length} chars`);
    }

    // Step 3: Extract key points for video
    const keyPoints = this.extractKeyPoints(contentByLanguage.en || contentByLanguage.vi);

    // Step 4: Render video
    const videoPath = `./output/video-${Date.now()}.mp4`;
    await this.videoRenderer.renderArticleVideo(
      {
        title: this.extractTitle(contentByLanguage.en || contentByLanguage.vi),
        keyPoints,
        fullText: contentByLanguage.en || contentByLanguage.vi
      },
      videoPath
    );

    console.log(`Video rendered: ${videoPath}`);

    return {
      articles,
      content: contentByLanguage,
      videoPath,
      keyPoints
    };
  }

  private extractKeyPoints(content: string): string[] {
    // Simple extraction - can be enhanced with AI
    const lines = content.split('\n');
    return lines
      .filter(line => line.match(/^\d+\./))
      .map(line => line.replace(/^\d+\.\s*/, '').trim())
      .slice(0, 5);
  }

  private extractTitle(content: string): string {
    const firstLine = content.split('\n')[0];
    return firstLine.replace(/^#\s*/, '').trim();
  }
}

interface PipelineResult {
  articles: Article[];
  content: Record<string, string>;
  videoPath: string;
  keyPoints: string[];
}
```

### API Route Implementation

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { ContentPipeline } from '@/lib/pipeline/content-pipeline';

export async function POST(req: NextRequest) {
  try {
    const { keyword, format, languages } = await req.json();

    if (!keyword) {
      return NextResponse.json(
        { error: 'Keyword is required' },
        { status: 400 }
      );
    }

    const pipeline = new ContentPipeline();
    const result = await pipeline.runPipeline(
      keyword,
      format || 'toplist',
      languages || ['en']
    );

    return NextResponse.json({
      success: true,
      data: {
        articlesCount: result.articles.length,
        content: result.content,
        videoUrl: `/videos/${result.videoPath.split('/').pop()}`,
        keyPoints: result.keyPoints
      }
    });

  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { error: error instanceof Error ? error.message : 'Pipeline failed' },
      { status: 500 }
    );
  }
}
```

## Frontend Usage

### React Component Example

```typescript
// src/components/ContentGenerator.tsx
'use client';

import { useState } from 'react';

export default function ContentGenerator() {
  const [keyword, setKeyword] = useState('');
  const [format, setFormat] = useState<'toplist' | 'pov' | 'casestudy' | 'howto'>('toplist');
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
          languages: ['en', 'vi']
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
    <div className="p-8">
      <h1 className="text-3xl font-bold mb-6">AI Content Pipeline</h1>
      
      <div className="space-y-4">
        <input
          type="text"
          value={keyword}
          onChange={(e) => setKeyword(e.target.value)}
          placeholder="Enter keyword (e.g., 'AI automation')"
          className="w-full p-3 border rounded"
        />

        <select
          value={format}
          onChange={(e) => setFormat(e.target.value as any)}
          className="w-full p-3 border rounded"
        >
          <option value="toplist">Top List</option>
          <option value="pov">Point of View</option>
          <option value="casestudy">Case Study</option>
          <option value="howto">How-to Guide</option>
        </select>

        <button
          onClick={handleGenerate}
          disabled={loading || !keyword}
          className="w-full bg-blue-600 text-white p-3 rounded disabled:bg-gray-400"
        >
          {loading ? 'Generating...' : 'Generate Content & Video'}
        </button>

        {result && (
          <div className="mt-6 space-y-4">
            <div className="p-4 bg-green-50 rounded">
              <p>✅ Found {result.data.articlesCount} articles</p>
              <p>✅ Generated content in {Object.keys(result.data.content).length} languages</p>
            </div>

            {result.data.videoUrl && (
              <video
                controls
                src={result.data.videoUrl}
                className="w-full rounded"
              />
            )}

            <div className="p-4 bg-gray-50 rounded">
              <h3 className="font-bold mb-2">Key Points:</h3>
              <ul className="list-disc pl-5">
                {result.data.keyPoints.map((point: string, i: number) => (
                  <li key={i}>{point}</li>
                ))}
              </ul>
            </div>
          </div>
        )}
      </div>
    </div>
  );
}
```

## Common Patterns

### Batch Processing Multiple Keywords

```typescript
// src/scripts/batch-generate.ts
import { ContentPipeline } from '@/lib/pipeline/content-pipeline';

async function batchGenerate(keywords: string[]) {
  const pipeline = new ContentPipeline();
  const results = [];

  for (const keyword of keywords) {
    try {
      console.log(`Processing: ${keyword}`);
      const result = await pipeline.runPipeline(keyword, 'toplist', ['en', 'vi']);
      results.push({ keyword, success: true, result });
      
      // Rate limiting
      await new Promise(resolve => setTimeout(resolve, 5000));
    } catch (error) {
      console.error(`Failed for ${keyword}:`, error);
      results.push({ keyword, success: false, error });
    }
  }

  return results;
}

// Usage
const keywords = ['AI automation', 'content marketing', 'video creation'];
batchGenerate(keywords).then(results => {
  console.log(`Completed ${results.filter(r => r.success).length}/${results.length}`);
});
```

### Custom Video Templates

```typescript
// Add new composition for different social media formats
export const ReelsVideo: React.FC<VideoProps> = (props) => {
  return (
    <AbsoluteFill style={{ backgroundColor: '#000' }}>
      {/* 9:16 format optimized for Instagram Reels */}
    </AbsoluteFill>
  );
};

export const YouTubeShortsVideo: React.FC<VideoProps> = (props) => {
  return (
    <AbsoluteFill style={{ backgroundColor: '#fff' }}>
      {/* YouTube Shorts specific layout */}
    </AbsoluteFill>
  );
};
```

## Troubleshooting

### Common Issues

**API Rate Limits**
```typescript
// Implement retry logic with exponential backoff
async function withRetry<T>(
  fn: () => Promise<T>,
  maxRetries: number = 3
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error: any) {
      if (error.status === 429 && i < maxRetries - 1) {
        const delay = Math.pow(2, i) * 1000;
        console.log(`Rate limited, retrying in ${delay}ms`);
        await new Promise(resolve => setTimeout(resolve, delay));
      } else {
        throw error;
      }
    }
  }
  throw new Error('Max retries exceeded');
}
```

**Video Rendering Fails**
```bash
# Ensure Remotion dependencies are installed
pnpm add @remotion/bundler @remotion/renderer @remotion/cli

# Check FFmpeg is available
ffmpeg -version

# Install if missing (Mac)
brew install ffmpeg

# Install if missing (Ubuntu)
sudo apt-get install ffmpeg
```

**Memory Issues with Large Content**
```typescript
// Process articles in chunks
function chunkArray<T>(array: T[], size: number): T[][] {
  const chunks: T[][] = [];
  for (let i = 0; i < array.length; i += size) {
    chunks.push(array.slice(i, i + size));
  }
  return chunks;
}

const articleChunks = chunkArray(articles, 10);
for (const chunk of articleChunks) {
  await processChunk(chunk);
}
```

**Environment Variables Not Loading**
```typescript
// Validate environment setup
function validateEnv() {
  const required = [
    'ANTHROPIC_API_KEY',
    'OPENAI_API_KEY',
    'RAPIDAPI_KEY'
  ];

  const missing = required.filter(key => !process.env[key]);
  
  if (missing.length > 0) {
    throw new Error(`Missing required env vars: ${missing.join(', ')}`);
  }
}

// Call at app startup
validateEnv();
```

## Performance Optimization

### Caching Research Results

```typescript
// src/lib/cache/redis-cache.ts
import { Redis } from 'ioredis';

export class CacheManager {
  private redis: Redis;

  constructor() {
    this.redis = new Redis(process.env.REDIS_URL);
  }

  async getCachedArticles(keyword: string): Promise<Article[] | null> {
    const cached = await this.redis.get(`articles:${keyword}`);
    return cached ? JSON.parse(cached) : null;
  }

  async cacheArticles(keyword: string, articles: Article[]) {
    await this.redis.setex(
      `articles:${keyword}`,
      3600, // 1 hour TTL
      JSON.stringify(articles)
    );
  }
}
```

This skill equips AI coding agents with comprehensive knowledge to help developers implement, customize, and troubleshoot the Marketing Pipeline Share content automation system.
