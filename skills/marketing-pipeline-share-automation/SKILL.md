---
name: marketing-pipeline-share-automation
description: AI-powered content automation pipeline from research to video generation using Claude, OpenAI, and Remotion
triggers:
  - automate content creation with AI pipeline
  - generate videos from articles automatically
  - research and write content with Claude
  - create marketing content pipeline
  - auto-generate social media videos
  - scrape news and create content
  - build automated content workflow
  - generate multilingual marketing content
---

# Marketing Pipeline Share - AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the Ultimate AI Content Pipeline, a complete TypeScript-based system that automates content creation from research through video generation. The pipeline integrates Claude 3, OpenAI, web scraping, and Remotion for end-to-end content automation.

## What This Project Does

Marketing Pipeline Share is a fully automated content production system that:

- **Auto-scans and researches** live data from TechCrunch, a16z, Twitter/X, LinkedIn within 24 hours
- **Generates content** in multiple formats (toplist, POV, case studies, how-to) using Claude/OpenAI
- **Creates multilingual content** (English & Vietnamese) with customizable tone
- **Renders videos automatically** using Remotion for Reels, TikTok, Shorts
- **Provides Next.js interface** for managing the entire pipeline

## Installation

### Prerequisites

```bash
# Node.js 18+ required
node --version

# Install dependencies
npm install
# or
yarn install
# or
pnpm install
```

### Environment Configuration

Create a `.env.local` file in the project root:

```bash
# AI Services
ANTHROPIC_API_KEY=your_anthropic_key_here
OPENAI_API_KEY=your_openai_key_here

# RapidAPI for web scraping
RAPIDAPI_KEY=your_rapidapi_key_here

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Remotion (for video rendering)
REMOTION_LICENSE_KEY=your_remotion_license

# Next.js
NEXT_PUBLIC_API_URL=http://localhost:3000
```

### Running the Application

```bash
# Development mode
npm run dev

# Production build
npm run build
npm start

# Video rendering server
npm run remotion
```

## Project Structure

```
marketing-pipeline-share/
├── src/
│   ├── api/          # API routes for content generation
│   ├── components/   # React/Next.js UI components
│   ├── lib/          # Core utilities and services
│   │   ├── ai/       # Claude & OpenAI integrations
│   │   ├── scraper/  # Web scraping modules
│   │   └── video/    # Remotion video generation
│   ├── pages/        # Next.js pages
│   └── types/        # TypeScript type definitions
├── remotion/         # Video templates and compositions
└── public/           # Static assets
```

## Core API Usage

### 1. Research & Content Scraping

```typescript
// src/lib/scraper/research.ts
import axios from 'axios';

interface ResearchResult {
  title: string;
  content: string;
  source: string;
  publishedAt: Date;
  insights: string[];
}

export async function scrapeLatestNews(
  topic: string,
  sources: string[] = ['techcrunch', 'a16z']
): Promise<ResearchResult[]> {
  const results: ResearchResult[] = [];
  
  for (const source of sources) {
    try {
      const response = await axios.get(
        `https://api.rapidapi.com/news/${source}`,
        {
          headers: {
            'X-RapidAPI-Key': process.env.RAPIDAPI_KEY!,
            'X-RapidAPI-Host': 'news-api.rapidapi.com'
          },
          params: {
            q: topic,
            timeframe: '24h',
            lang: 'en'
          }
        }
      );
      
      results.push(...response.data.articles);
    } catch (error) {
      console.error(`Failed to scrape ${source}:`, error);
    }
  }
  
  return results;
}

// Extract insights from research data
export function extractInsights(results: ResearchResult[]): string[] {
  const insights = results.flatMap(r => r.insights);
  return [...new Set(insights)]; // Remove duplicates
}
```

### 2. AI Content Generation with Claude

```typescript
// src/lib/ai/claude-generator.ts
import Anthropic from '@anthropic-ai/sdk';

const client = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY!,
});

interface ContentConfig {
  topic: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  tone: 'expert' | 'friendly' | 'humorous';
  language: 'en' | 'vi';
  research: string[];
}

export async function generateContent(
  config: ContentConfig
): Promise<string> {
  const systemPrompt = buildSystemPrompt(config);
  const userPrompt = buildUserPrompt(config);
  
  const message = await client.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    temperature: 0.7,
    system: systemPrompt,
    messages: [
      {
        role: 'user',
        content: userPrompt
      }
    ]
  });
  
  return message.content[0].type === 'text' 
    ? message.content[0].text 
    : '';
}

function buildSystemPrompt(config: ContentConfig): string {
  const toneMap = {
    expert: 'professional and authoritative',
    friendly: 'conversational and approachable',
    humorous: 'witty and entertaining'
  };
  
  return `You are a ${toneMap[config.tone]} content writer specializing in ${config.format} articles. 
Write in ${config.language === 'en' ? 'English' : 'Vietnamese'}.
Use data-backed insights and maintain high engagement.`;
}

function buildUserPrompt(config: ContentConfig): string {
  return `
Topic: ${config.topic}
Format: ${config.format}

Research Data:
${config.research.join('\n\n')}

Create a comprehensive ${config.format} article about ${config.topic} using the research data provided.
Include specific examples, data points, and actionable insights.
  `.trim();
}
```

### 3. OpenAI Integration (Alternative)

```typescript
// src/lib/ai/openai-generator.ts
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY!,
});

export async function generateWithGPT4(
  prompt: string,
  systemPrompt: string
): Promise<string> {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      { role: 'system', content: systemPrompt },
      { role: 'user', content: prompt }
    ],
    temperature: 0.7,
    max_tokens: 3000,
  });
  
  return completion.choices[0]?.message?.content || '';
}
```

### 4. Video Generation with Remotion

```typescript
// src/lib/video/render.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  title: string;
  content: string[];
  format: 'reel' | 'tiktok' | 'short';
}

export async function renderContentVideo(
  config: VideoConfig
): Promise<string> {
  // Bundle Remotion project
  const bundled = await bundle({
    entryPoint: path.join(process.cwd(), 'remotion/index.ts'),
    webpackOverride: (config) => config,
  });
  
  // Get composition
  const composition = await selectComposition({
    serveUrl: bundled,
    id: 'ContentVideo',
    inputProps: {
      title: config.title,
      slides: config.content,
      format: config.format,
    },
  });
  
  // Render video
  const outputPath = path.join(
    process.cwd(), 
    'public/videos',
    `${Date.now()}.mp4`
  );
  
  await renderMedia({
    composition,
    serveUrl: bundled,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: composition.defaultProps,
  });
  
  return outputPath;
}
```

```typescript
// remotion/Composition.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig } from 'remotion';

export const ContentVideo: React.FC<{
  title: string;
  slides: string[];
  format: 'reel' | 'tiktok' | 'short';
}> = ({ title, slides, format }) => {
  const frame = useCurrentFrame();
  const { fps, durationInFrames } = useVideoConfig();
  
  const slideIndex = Math.floor(
    (frame / durationInFrames) * slides.length
  );
  
  return (
    <AbsoluteFill style={{ backgroundColor: '#000' }}>
      <div style={{
        fontSize: format === 'reel' ? 48 : 40,
        color: 'white',
        textAlign: 'center',
        padding: 40,
      }}>
        <h1>{title}</h1>
        <p>{slides[slideIndex]}</p>
      </div>
    </AbsoluteFill>
  );
};
```

## Complete Pipeline Implementation

```typescript
// src/lib/pipeline/content-pipeline.ts
import { scrapeLatestNews, extractInsights } from '../scraper/research';
import { generateContent } from '../ai/claude-generator';
import { renderContentVideo } from '../video/render';

interface PipelineConfig {
  topic: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  tone: 'expert' | 'friendly' | 'humorous';
  languages: ('en' | 'vi')[];
  includeVideo: boolean;
  sources?: string[];
}

export async function runContentPipeline(
  config: PipelineConfig
) {
  console.log(`Starting pipeline for topic: ${config.topic}`);
  
  // Step 1: Research
  console.log('Step 1: Scraping latest news...');
  const research = await scrapeLatestNews(
    config.topic,
    config.sources
  );
  const insights = extractInsights(research);
  
  // Step 2: Generate content for each language
  console.log('Step 2: Generating content...');
  const articles: Record<string, string> = {};
  
  for (const lang of config.languages) {
    const content = await generateContent({
      topic: config.topic,
      format: config.format,
      tone: config.tone,
      language: lang,
      research: insights,
    });
    
    articles[lang] = content;
  }
  
  // Step 3: Generate video (if requested)
  let videoPath: string | null = null;
  if (config.includeVideo) {
    console.log('Step 3: Rendering video...');
    
    // Extract key points for video slides
    const slides = extractKeyPoints(articles.en || articles.vi);
    
    videoPath = await renderContentVideo({
      title: config.topic,
      content: slides,
      format: 'reel',
    });
  }
  
  return {
    articles,
    research: insights,
    video: videoPath,
    timestamp: new Date(),
  };
}

function extractKeyPoints(content: string): string[] {
  // Simple extraction - split by paragraphs and take first sentence
  return content
    .split('\n\n')
    .filter(p => p.trim().length > 0)
    .map(p => p.split('.')[0] + '.')
    .slice(0, 5);
}
```

## API Routes (Next.js)

```typescript
// pages/api/generate-content.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { runContentPipeline } from '@/lib/pipeline/content-pipeline';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }
  
  try {
    const { topic, format, tone, languages, includeVideo } = req.body;
    
    const result = await runContentPipeline({
      topic,
      format,
      tone,
      languages: languages || ['en', 'vi'],
      includeVideo: includeVideo || false,
    });
    
    res.status(200).json(result);
  } catch (error) {
    console.error('Pipeline error:', error);
    res.status(500).json({ 
      error: 'Content generation failed',
      details: error instanceof Error ? error.message : 'Unknown error'
    });
  }
}
```

## Frontend Integration

```typescript
// src/components/ContentGenerator.tsx
'use client';

import { useState } from 'react';

export function ContentGenerator() {
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState<any>(null);
  
  const handleGenerate = async (e: React.FormEvent<HTMLFormElement>) => {
    e.preventDefault();
    setLoading(true);
    
    const formData = new FormData(e.currentTarget);
    const payload = {
      topic: formData.get('topic'),
      format: formData.get('format'),
      tone: formData.get('tone'),
      languages: ['en', 'vi'],
      includeVideo: formData.get('includeVideo') === 'on',
    };
    
    try {
      const response = await fetch('/api/generate-content', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(payload),
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
    <div className="content-generator">
      <form onSubmit={handleGenerate}>
        <input name="topic" placeholder="Enter topic..." required />
        
        <select name="format" required>
          <option value="toplist">Top List</option>
          <option value="pov">Point of View</option>
          <option value="case-study">Case Study</option>
          <option value="how-to">How-to Guide</option>
        </select>
        
        <select name="tone" required>
          <option value="expert">Expert</option>
          <option value="friendly">Friendly</option>
          <option value="humorous">Humorous</option>
        </select>
        
        <label>
          <input type="checkbox" name="includeVideo" />
          Generate Video
        </label>
        
        <button type="submit" disabled={loading}>
          {loading ? 'Generating...' : 'Generate Content'}
        </button>
      </form>
      
      {result && (
        <div className="results">
          <h3>Generated Content</h3>
          {Object.entries(result.articles).map(([lang, content]) => (
            <div key={lang}>
              <h4>{lang.toUpperCase()}</h4>
              <pre>{content as string}</pre>
            </div>
          ))}
          
          {result.video && (
            <video src={result.video} controls />
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
// Generate multiple pieces of content at once
const topics = ['AI in Marketing', 'SEO Trends 2024', 'Content Automation'];

const results = await Promise.all(
  topics.map(topic =>
    runContentPipeline({
      topic,
      format: 'toplist',
      tone: 'expert',
      languages: ['en'],
      includeVideo: false,
    })
  )
);
```

### Custom Research Sources

```typescript
// Add custom scraping sources
import { scrapeLatestNews } from '@/lib/scraper/research';

const customResults = await scrapeLatestNews('AI tools', [
  'techcrunch',
  'producthunt',
  'hackernews'
]);
```

### Video Format Optimization

```typescript
// Generate platform-specific videos
const formats = ['reel', 'tiktok', 'short'] as const;

for (const format of formats) {
  await renderContentVideo({
    title: 'AI Marketing Tips',
    content: slides,
    format,
  });
}
```

## Troubleshooting

### API Key Issues

```typescript
// Verify API keys are loaded
if (!process.env.ANTHROPIC_API_KEY) {
  throw new Error('ANTHROPIC_API_KEY is not set');
}
```

### Rate Limiting

```typescript
// Implement rate limiting for API calls
import pLimit from 'p-limit';

const limit = pLimit(3); // Max 3 concurrent requests

const results = await Promise.all(
  topics.map(topic => 
    limit(() => generateContent({ topic, /* ... */ }))
  )
);
```

### Video Rendering Memory Issues

```bash
# Increase Node.js memory limit
NODE_OPTIONS=--max-old-space-size=4096 npm run remotion
```

### Debugging Pipeline Steps

```typescript
// Add detailed logging
export async function runContentPipeline(config: PipelineConfig) {
  const startTime = Date.now();
  
  try {
    console.log('[Pipeline] Starting:', config);
    
    const research = await scrapeLatestNews(config.topic);
    console.log('[Pipeline] Research complete:', research.length, 'articles');
    
    // ... rest of pipeline
    
    console.log('[Pipeline] Completed in:', Date.now() - startTime, 'ms');
  } catch (error) {
    console.error('[Pipeline] Error:', error);
    throw error;
  }
}
```

This skill provides comprehensive guidance for working with the Marketing Pipeline Share automation system, covering research, content generation, and video rendering workflows.
