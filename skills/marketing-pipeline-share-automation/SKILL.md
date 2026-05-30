---
name: marketing-pipeline-share-automation
description: AI-powered content pipeline that automates research, script writing, posting, and video generation using Claude/OpenAI and Remotion
triggers:
  - automate content creation with AI pipeline
  - set up automated marketing content workflow
  - generate videos from content automatically
  - scrape news and create articles with AI
  - build content automation system
  - create multi-format content with Claude
  - set up Remotion video rendering pipeline
  - automate social media content generation
---

# Marketing Pipeline Share - AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI agents to work with the Marketing Pipeline Share system, an end-to-end content automation pipeline that handles research, content generation, and video rendering. The system combines web scraping, AI content generation (Claude/OpenAI), and video rendering (Remotion) to create a complete content factory.

## What This Project Does

Marketing Pipeline Share automates the entire content creation lifecycle:

1. **Auto-Research**: Crawls recent news from TechCrunch, a16z, X (Twitter), LinkedIn
2. **AI Content Generation**: Creates multi-format content (toplist, POV, case study, how-to) in multiple languages
3. **Video Rendering**: Automatically generates infographics and videos using Remotion
4. **Multi-Platform Output**: Exports content optimized for Reels, TikTok, Shorts

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
cp .env.example .env
```

## Configuration

Create a `.env` file with the following variables:

```bash
# AI Services
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# RapidAPI for web scraping
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_url

# Next.js
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

## Key Architecture

The system is built with:
- **Next.js**: Frontend and API routes
- **TypeScript**: Type-safe development
- **Remotion**: Video rendering engine
- **Claude/OpenAI**: Content generation
- **RapidAPI**: News scraping

## Core Components

### 1. Research & Scraping Module

The research module automatically fetches recent content from various sources:

```typescript
// lib/research/scraper.ts
interface NewsSource {
  name: string;
  url: string;
  scrapeInterval: number;
}

interface ScrapedArticle {
  title: string;
  content: string;
  url: string;
  publishedAt: Date;
  source: string;
  insights?: string[];
}

export async function scrapeNewsSources(
  sources: NewsSource[],
  keywords: string[]
): Promise<ScrapedArticle[]> {
  const articles: ScrapedArticle[] = [];
  
  for (const source of sources) {
    const response = await fetch(`https://api.rapidapi.com/scrape`, {
      method: 'POST',
      headers: {
        'X-RapidAPI-Key': process.env.RAPIDAPI_KEY!,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        url: source.url,
        keywords: keywords
      })
    });
    
    const data = await response.json();
    articles.push(...data.articles);
  }
  
  return articles;
}

export async function extractInsights(
  articles: ScrapedArticle[]
): Promise<ScrapedArticle[]> {
  return articles.map(article => ({
    ...article,
    insights: analyzeContent(article.content)
  }));
}

function analyzeContent(content: string): string[] {
  // Extract key insights, statistics, trends
  const insights: string[] = [];
  
  // Pattern matching for statistics
  const stats = content.match(/\d+%|\$\d+[MBK]?/g);
  if (stats) insights.push(...stats);
  
  return insights;
}
```

### 2. AI Content Generation

Generate content in multiple formats using Claude or OpenAI:

```typescript
// lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

type ContentFormat = 'toplist' | 'pov' | 'case-study' | 'how-to';
type Language = 'en' | 'vi';
type Tone = 'expert' | 'friendly' | 'humorous';

interface ContentConfig {
  format: ContentFormat;
  language: Language;
  tone: Tone;
  targetAudience: string;
}

interface GeneratedContent {
  title: string;
  body: string;
  summary: string;
  hashtags: string[];
  metadata: {
    wordCount: number;
    readingTime: number;
  };
}

export class ContentGenerator {
  private anthropic: Anthropic;
  private openai: OpenAI;
  
  constructor() {
    this.anthropic = new Anthropic({
      apiKey: process.env.ANTHROPIC_API_KEY
    });
    this.openai = new OpenAI({
      apiKey: process.env.OPENAI_API_KEY
    });
  }
  
  async generateWithClaude(
    research: ScrapedArticle[],
    config: ContentConfig
  ): Promise<GeneratedContent> {
    const prompt = this.buildPrompt(research, config);
    
    const message = await this.anthropic.messages.create({
      model: 'claude-3-5-sonnet-20241022',
      max_tokens: 4096,
      messages: [{
        role: 'user',
        content: prompt
      }]
    });
    
    const content = message.content[0].text;
    return this.parseGeneratedContent(content);
  }
  
  async generateWithOpenAI(
    research: ScrapedArticle[],
    config: ContentConfig
  ): Promise<GeneratedContent> {
    const prompt = this.buildPrompt(research, config);
    
    const completion = await this.openai.chat.completions.create({
      model: 'gpt-4-turbo-preview',
      messages: [{
        role: 'system',
        content: 'You are an expert content creator specializing in marketing content.'
      }, {
        role: 'user',
        content: prompt
      }],
      temperature: 0.7,
      max_tokens: 4096
    });
    
    const content = completion.choices[0].message.content || '';
    return this.parseGeneratedContent(content);
  }
  
  private buildPrompt(
    research: ScrapedArticle[],
    config: ContentConfig
  ): string {
    const researchContext = research.map(article => 
      `Title: ${article.title}\nInsights: ${article.insights?.join(', ')}\nURL: ${article.url}`
    ).join('\n\n');
    
    const formatInstructions = {
      'toplist': 'Create a numbered list article with detailed explanations for each item',
      'pov': 'Write from a unique perspective or opinion on the topic',
      'case-study': 'Analyze a specific example with data and outcomes',
      'how-to': 'Create a step-by-step guide with actionable instructions'
    };
    
    const toneGuide = {
      'expert': 'authoritative and data-driven',
      'friendly': 'conversational and approachable',
      'humorous': 'engaging with light humor'
    };
    
    return `
Based on this recent research:
${researchContext}

Create a ${config.format} article in ${config.language} with a ${toneGuide[config.tone]} tone.
Target audience: ${config.targetAudience}

Format: ${formatInstructions[config.format]}

Include:
- Compelling title
- Well-structured body with headings
- Key insights and statistics from research
- Summary/conclusion
- Relevant hashtags

Output in JSON format:
{
  "title": "...",
  "body": "...",
  "summary": "...",
  "hashtags": ["...", "..."]
}
    `.trim();
  }
  
  private parseGeneratedContent(content: string): GeneratedContent {
    const parsed = JSON.parse(content);
    const wordCount = parsed.body.split(/\s+/).length;
    const readingTime = Math.ceil(wordCount / 200); // 200 words per minute
    
    return {
      ...parsed,
      metadata: {
        wordCount,
        readingTime
      }
    };
  }
}
```

### 3. Video Rendering with Remotion

Automatically generate videos from content:

```typescript
// lib/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  width: number;
  height: number;
  fps: number;
  durationInFrames: number;
}

interface VideoAssets {
  title: string;
  subtitle?: string;
  bullets?: string[];
  backgroundImage?: string;
  brandColor: string;
}

export class VideoRenderer {
  private bundleLocation: string | null = null;
  
  async initialize() {
    // Bundle the Remotion project
    this.bundleLocation = await bundle({
      entryPoint: path.join(process.cwd(), 'remotion/index.ts'),
      webpackOverride: (config) => config
    });
  }
  
  async renderContentVideo(
    content: GeneratedContent,
    config: VideoConfig = {
      width: 1080,
      height: 1920, // Vertical for Reels/TikTok
      fps: 30,
      durationInFrames: 300 // 10 seconds at 30fps
    }
  ): Promise<string> {
    if (!this.bundleLocation) {
      await this.initialize();
    }
    
    const composition = await selectComposition({
      serveUrl: this.bundleLocation!,
      id: 'ContentVideo',
      inputProps: this.prepareVideoAssets(content)
    });
    
    const outputPath = path.join(
      process.cwd(),
      'public/videos',
      `${Date.now()}.mp4`
    );
    
    await renderMedia({
      composition,
      serveUrl: this.bundleLocation!,
      codec: 'h264',
      outputLocation: outputPath,
      inputProps: this.prepareVideoAssets(content)
    });
    
    return outputPath;
  }
  
  private prepareVideoAssets(content: GeneratedContent): VideoAssets {
    // Extract bullet points from content
    const bullets = content.body
      .split('\n')
      .filter(line => line.trim().match(/^[-*\d.]/))
      .slice(0, 5)
      .map(line => line.replace(/^[-*\d.]\s*/, ''));
    
    return {
      title: content.title,
      subtitle: content.summary,
      bullets,
      brandColor: '#3B82F6'
    };
  }
}
```

### 4. Remotion Video Component

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig, interpolate } from 'remotion';
import React from 'react';

interface ContentVideoProps {
  title: string;
  subtitle?: string;
  bullets?: string[];
  brandColor: string;
}

export const ContentVideo: React.FC<ContentVideoProps> = ({
  title,
  subtitle,
  bullets = [],
  brandColor
}) => {
  const frame = useCurrentFrame();
  const { fps, durationInFrames } = useVideoConfig();
  
  // Fade in animation
  const opacity = interpolate(frame, [0, 30], [0, 1], {
    extrapolateRight: 'clamp'
  });
  
  // Slide up animation for title
  const titleY = interpolate(frame, [0, 40], [100, 0], {
    extrapolateRight: 'clamp'
  });
  
  return (
    <AbsoluteFill
      style={{
        backgroundColor: '#1a1a1a',
        justifyContent: 'center',
        alignItems: 'center',
        padding: 60
      }}
    >
      <div
        style={{
          opacity,
          transform: `translateY(${titleY}px)`,
          width: '100%'
        }}
      >
        <h1
          style={{
            fontSize: 72,
            fontWeight: 'bold',
            color: brandColor,
            textAlign: 'center',
            marginBottom: 30,
            lineHeight: 1.2
          }}
        >
          {title}
        </h1>
        
        {subtitle && (
          <p
            style={{
              fontSize: 32,
              color: '#ffffff',
              textAlign: 'center',
              marginBottom: 50
            }}
          >
            {subtitle}
          </p>
        )}
        
        <div style={{ marginTop: 40 }}>
          {bullets.map((bullet, index) => {
            const bulletOpacity = interpolate(
              frame,
              [60 + index * 15, 75 + index * 15],
              [0, 1],
              { extrapolateRight: 'clamp' }
            );
            
            return (
              <div
                key={index}
                style={{
                  opacity: bulletOpacity,
                  fontSize: 28,
                  color: '#ffffff',
                  marginBottom: 20,
                  display: 'flex',
                  alignItems: 'center'
                }}
              >
                <span
                  style={{
                    width: 12,
                    height: 12,
                    borderRadius: '50%',
                    backgroundColor: brandColor,
                    marginRight: 20
                  }}
                />
                {bullet}
              </div>
            );
          })}
        </div>
      </div>
    </AbsoluteFill>
  );
};
```

### 5. API Routes (Next.js)

```typescript
// pages/api/generate-content.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { ContentGenerator } from '@/lib/ai/content-generator';
import { scrapeNewsSources } from '@/lib/research/scraper';
import { VideoRenderer } from '@/lib/video/renderer';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }
  
  try {
    const { keyword, format, language, tone, generateVideo } = req.body;
    
    // Step 1: Research
    const sources = [
      { name: 'TechCrunch', url: 'https://techcrunch.com', scrapeInterval: 24 },
      { name: 'a16z', url: 'https://a16z.com/blog', scrapeInterval: 24 }
    ];
    
    const articles = await scrapeNewsSources(sources, [keyword]);
    
    // Step 2: Generate content
    const generator = new ContentGenerator();
    const content = await generator.generateWithClaude(articles, {
      format,
      language,
      tone,
      targetAudience: 'marketers and content creators'
    });
    
    let videoPath = null;
    
    // Step 3: Render video (optional)
    if (generateVideo) {
      const renderer = new VideoRenderer();
      videoPath = await renderer.renderContentVideo(content);
    }
    
    res.status(200).json({
      success: true,
      content,
      videoUrl: videoPath ? `/videos/${path.basename(videoPath)}` : null,
      articlesAnalyzed: articles.length
    });
    
  } catch (error) {
    console.error('Content generation error:', error);
    res.status(500).json({ 
      error: 'Failed to generate content',
      details: error instanceof Error ? error.message : 'Unknown error'
    });
  }
}
```

### 6. Frontend Usage

```typescript
// components/ContentPipeline.tsx
import { useState } from 'react';

export function ContentPipeline() {
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState(null);
  
  const handleGenerate = async (formData: any) => {
    setLoading(true);
    
    try {
      const response = await fetch('/api/generate-content', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(formData)
      });
      
      const data = await response.json();
      setResult(data);
    } catch (error) {
      console.error(error);
    } finally {
      setLoading(false);
    }
  };
  
  return (
    <div>
      {/* Form UI here */}
      {result && (
        <div>
          <h2>{result.content.title}</h2>
          <div dangerouslySetInnerHTML={{ __html: result.content.body }} />
          {result.videoUrl && (
            <video src={result.videoUrl} controls />
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
  const generator = new ContentGenerator();
  const renderer = new VideoRenderer();
  
  const results = await Promise.all(
    keywords.map(async (keyword) => {
      const articles = await scrapeNewsSources(sources, [keyword]);
      const content = await generator.generateWithClaude(articles, config);
      const video = await renderer.renderContentVideo(content);
      
      return { keyword, content, video };
    })
  );
  
  return results;
}
```

### Scheduling Content

```typescript
// lib/scheduler/cron.ts
import cron from 'node-cron';

export function scheduleContentGeneration(
  schedule: string,
  keywords: string[]
) {
  cron.schedule(schedule, async () => {
    console.log('Running scheduled content generation...');
    await generateBatchContent(keywords);
  });
}

// Run daily at 9 AM
scheduleContentGeneration('0 9 * * *', ['AI', 'Marketing', 'Tech']);
```

## Troubleshooting

**API Rate Limits**: Implement retry logic and rate limiting:

```typescript
async function withRetry<T>(
  fn: () => Promise<T>,
  maxRetries = 3
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      await new Promise(resolve => setTimeout(resolve, 1000 * (i + 1)));
    }
  }
  throw new Error('Max retries exceeded');
}
```

**Video Rendering Fails**: Ensure Remotion bundle is initialized:

```typescript
const renderer = new VideoRenderer();
await renderer.initialize(); // Always initialize before rendering
```

**Memory Issues with Large Content**: Use streaming for large batches:

```typescript
import { createWriteStream } from 'fs';

async function streamContentGeneration(keywords: string[]) {
  const stream = createWriteStream('output.jsonl');
  
  for (const keyword of keywords) {
    const content = await generateContent(keyword);
    stream.write(JSON.stringify(content) + '\n');
  }
  
  stream.end();
}
```
