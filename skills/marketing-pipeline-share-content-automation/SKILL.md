---
name: marketing-pipeline-share-content-automation
description: AI-powered content pipeline for automated research, scriptwriting, posting, and video generation using Claude/OpenAI and Remotion
triggers:
  - automate content creation from research to video
  - generate marketing content with AI pipeline
  - create automated content workflow with Claude
  - build content automation system with video generation
  - scrape news and generate content automatically
  - set up AI content pipeline with Remotion
  - automate social media content with AI research
  - create end-to-end content automation workflow
---

# Marketing Pipeline Share - Content Automation Skill

> Skill by [ara.so](https://ara.so) — Marketing Skills collection

## Overview

Marketing Pipeline Share is a complete AI-driven content automation system that handles the entire content lifecycle: from researching trending topics across news sources (TechCrunch, a16z, Twitter, LinkedIn), to generating scripts in multiple formats (toplist, POV, case study, how-to), to automatically rendering videos using Remotion. Built with Next.js and TypeScript, it integrates Claude 3, OpenAI, and RapidAPI for comprehensive content generation.

## Installation

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
pnpm install
# or
yarn install

# Set up environment variables
cp .env.example .env.local
```

## Environment Configuration

Create `.env.local` with the following required variables:

```bash
# AI APIs
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research/Scraping APIs
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Next.js
NEXT_PUBLIC_APP_URL=http://localhost:3000

# Remotion (for video generation)
REMOTION_AWS_ACCESS_KEY_ID=your_aws_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret
```

## Core Architecture

The pipeline consists of four main stages:

1. **Research Stage**: Auto-crawl news sources and extract trending topics
2. **Content Generation Stage**: Use AI (Claude/OpenAI) to create scripts in various formats
3. **Post Management Stage**: Schedule and manage content for different platforms
4. **Video Generation Stage**: Render videos using Remotion from generated content

## Key Components

### 1. Research/Scraping Module

```typescript
// lib/research/scraper.ts
import axios from 'axios';

interface ResearchSource {
  url: string;
  platform: 'techcrunch' | 'a16z' | 'twitter' | 'linkedin';
  timeframe: '24h' | '7d' | '30d';
}

export async function scrapeNewsData(
  keyword: string,
  sources: ResearchSource[]
): Promise<any[]> {
  const results = [];
  
  for (const source of sources) {
    try {
      const response = await axios.get(`/api/scrape`, {
        params: {
          keyword,
          source: source.platform,
          timeframe: source.timeframe
        },
        headers: {
          'X-RapidAPI-Key': process.env.RAPIDAPI_KEY
        }
      });
      
      results.push(...response.data);
    } catch (error) {
      console.error(`Failed to scrape ${source.platform}:`, error);
    }
  }
  
  return results;
}

export function extractInsights(rawData: any[]): string[] {
  // Process and extract key insights
  return rawData
    .map(item => item.content)
    .filter(content => content.length > 100)
    .slice(0, 10);
}
```

### 2. AI Content Generation

```typescript
// lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

export type ContentFormat = 'toplist' | 'pov' | 'case-study' | 'how-to';
export type Language = 'en' | 'vi';
export type Tone = 'expert' | 'friendly' | 'humorous';

interface GenerateContentOptions {
  keyword: string;
  format: ContentFormat;
  language: Language;
  tone: Tone;
  insights: string[];
  provider?: 'claude' | 'openai';
}

export async function generateContent(
  options: GenerateContentOptions
): Promise<string> {
  const { keyword, format, language, tone, insights, provider = 'claude' } = options;
  
  const prompt = buildPrompt(keyword, format, language, tone, insights);
  
  if (provider === 'claude') {
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
  } else {
    const completion = await openai.chat.completions.create({
      model: 'gpt-4-turbo-preview',
      messages: [
        {
          role: 'system',
          content: 'You are an expert content creator specializing in marketing content.'
        },
        {
          role: 'user',
          content: prompt
        }
      ],
      max_tokens: 4096
    });
    
    return completion.choices[0]?.message?.content || '';
  }
}

function buildPrompt(
  keyword: string,
  format: ContentFormat,
  language: Language,
  tone: Tone,
  insights: string[]
): string {
  const formatInstructions = {
    'toplist': 'Create a top 10 list article',
    'pov': 'Write from a unique point of view perspective',
    'case-study': 'Develop a detailed case study analysis',
    'how-to': 'Create a step-by-step how-to guide'
  };
  
  const toneInstructions = {
    'expert': 'Use professional, authoritative language',
    'friendly': 'Use conversational, approachable tone',
    'humorous': 'Include wit and humor while staying informative'
  };
  
  return `
Write a ${formatInstructions[format]} about "${keyword}" in ${language === 'en' ? 'English' : 'Vietnamese'}.

Tone: ${toneInstructions[tone]}

Use the following recent insights and data:
${insights.map((insight, i) => `${i + 1}. ${insight}`).join('\n')}

Requirements:
- Include data-backed points
- Make it SEO-friendly
- Add clear section headings
- Include actionable takeaways
- Length: 1500-2000 words
`;
}
```

### 3. Multi-Language Content Generation

```typescript
// lib/ai/multi-language.ts
export async function generateBilingualContent(
  keyword: string,
  format: ContentFormat,
  tone: Tone,
  insights: string[]
): Promise<{ en: string; vi: string }> {
  const [englishContent, vietnameseContent] = await Promise.all([
    generateContent({
      keyword,
      format,
      language: 'en',
      tone,
      insights
    }),
    generateContent({
      keyword,
      format,
      language: 'vi',
      tone,
      insights
    })
  ]);
  
  return {
    en: englishContent,
    vi: vietnameseContent
  };
}
```

### 4. Video Generation with Remotion

```typescript
// lib/video/remotion-renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import { mkdirSync } from 'fs';
import path from 'path';

export interface VideoConfig {
  title: string;
  content: string[];
  style: 'reels' | 'tiktok' | 'shorts';
}

export async function renderContentVideo(
  config: VideoConfig
): Promise<string> {
  const { title, content, style } = config;
  
  // Platform-specific dimensions
  const dimensions = {
    reels: { width: 1080, height: 1920 },
    tiktok: { width: 1080, height: 1920 },
    shorts: { width: 1080, height: 1920 }
  };
  
  const { width, height } = dimensions[style];
  
  // Bundle Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: (config) => config,
  });
  
  // Select composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title,
      content,
    },
  });
  
  // Create output directory
  const outputDir = path.join(process.cwd(), 'public', 'videos');
  mkdirSync(outputDir, { recursive: true });
  
  const outputPath = path.join(
    outputDir,
    `${Date.now()}-${style}.mp4`
  );
  
  // Render video
  await renderMedia({
    composition: {
      ...composition,
      width,
      height,
    },
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: {
      title,
      content,
    },
  });
  
  return outputPath;
}
```

### 5. Complete Pipeline Orchestration

```typescript
// lib/pipeline/orchestrator.ts
import { scrapeNewsData, extractInsights } from '../research/scraper';
import { generateBilingualContent } from '../ai/multi-language';
import { renderContentVideo } from '../video/remotion-renderer';

export interface PipelineConfig {
  keyword: string;
  format: ContentFormat;
  tone: Tone;
  generateVideo: boolean;
  videoStyle?: 'reels' | 'tiktok' | 'shorts';
}

export async function runContentPipeline(
  config: PipelineConfig
) {
  const {
    keyword,
    format,
    tone,
    generateVideo,
    videoStyle = 'reels'
  } = config;
  
  console.log(`Starting pipeline for keyword: ${keyword}`);
  
  // Stage 1: Research
  console.log('Stage 1: Researching...');
  const rawData = await scrapeNewsData(keyword, [
    { url: '', platform: 'techcrunch', timeframe: '24h' },
    { url: '', platform: 'twitter', timeframe: '24h' },
  ]);
  
  const insights = extractInsights(rawData);
  console.log(`Extracted ${insights.length} insights`);
  
  // Stage 2: Generate Content
  console.log('Stage 2: Generating content...');
  const { en, vi } = await generateBilingualContent(
    keyword,
    format,
    tone,
    insights
  );
  
  console.log('Content generated in both languages');
  
  // Stage 3: Generate Video (optional)
  let videoPath: string | null = null;
  if (generateVideo) {
    console.log('Stage 3: Rendering video...');
    
    // Extract key points for video
    const contentPoints = en
      .split('\n\n')
      .filter(p => p.trim().length > 0)
      .slice(0, 5);
    
    videoPath = await renderContentVideo({
      title: keyword,
      content: contentPoints,
      style: videoStyle
    });
    
    console.log(`Video rendered: ${videoPath}`);
  }
  
  return {
    content: { en, vi },
    insights,
    videoPath,
    metadata: {
      keyword,
      format,
      tone,
      timestamp: new Date().toISOString()
    }
  };
}
```

## API Routes

### Research Endpoint

```typescript
// pages/api/research.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { scrapeNewsData, extractInsights } from '@/lib/research/scraper';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }
  
  const { keyword, sources } = req.body;
  
  try {
    const rawData = await scrapeNewsData(keyword, sources);
    const insights = extractInsights(rawData);
    
    res.status(200).json({ insights, count: insights.length });
  } catch (error) {
    console.error('Research error:', error);
    res.status(500).json({ error: 'Failed to research content' });
  }
}
```

### Content Generation Endpoint

```typescript
// pages/api/generate.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { generateContent } from '@/lib/ai/content-generator';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }
  
  const { keyword, format, language, tone, insights, provider } = req.body;
  
  try {
    const content = await generateContent({
      keyword,
      format,
      language,
      tone,
      insights,
      provider
    });
    
    res.status(200).json({ content });
  } catch (error) {
    console.error('Generation error:', error);
    res.status(500).json({ error: 'Failed to generate content' });
  }
}
```

### Pipeline Endpoint

```typescript
// pages/api/pipeline/run.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { runContentPipeline } from '@/lib/pipeline/orchestrator';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }
  
  const config = req.body;
  
  try {
    const result = await runContentPipeline(config);
    res.status(200).json(result);
  } catch (error) {
    console.error('Pipeline error:', error);
    res.status(500).json({ error: 'Pipeline execution failed' });
  }
}
```

## Usage Examples

### Running the Complete Pipeline

```typescript
// Example: pages/index.tsx
import { useState } from 'react';

export default function Home() {
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState(null);
  
  const runPipeline = async () => {
    setLoading(true);
    
    try {
      const response = await fetch('/api/pipeline/run', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          keyword: 'AI Marketing Automation',
          format: 'toplist',
          tone: 'expert',
          generateVideo: true,
          videoStyle: 'reels'
        })
      });
      
      const data = await response.json();
      setResult(data);
    } catch (error) {
      console.error('Pipeline failed:', error);
    } finally {
      setLoading(false);
    }
  };
  
  return (
    <div>
      <button onClick={runPipeline} disabled={loading}>
        {loading ? 'Running Pipeline...' : 'Generate Content'}
      </button>
      
      {result && (
        <div>
          <h2>English Content</h2>
          <pre>{result.content.en}</pre>
          
          <h2>Vietnamese Content</h2>
          <pre>{result.content.vi}</pre>
          
          {result.videoPath && (
            <div>
              <h2>Video</h2>
              <video src={result.videoPath} controls />
            </div>
          )}
        </div>
      )}
    </div>
  );
}
```

### Standalone Research

```typescript
import { scrapeNewsData, extractInsights } from '@/lib/research/scraper';

async function research() {
  const data = await scrapeNewsData('AI trends', [
    { url: '', platform: 'techcrunch', timeframe: '24h' }
  ]);
  
  const insights = extractInsights(data);
  console.log(insights);
}
```

### Generate Content Only

```typescript
import { generateContent } from '@/lib/ai/content-generator';

async function createArticle() {
  const content = await generateContent({
    keyword: 'Content Marketing Strategy',
    format: 'how-to',
    language: 'en',
    tone: 'friendly',
    insights: ['Insight 1', 'Insight 2'],
    provider: 'claude'
  });
  
  console.log(content);
}
```

## Development Commands

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Run linting
npm run lint

# Render Remotion video locally
npm run remotion:render
```

## Troubleshooting

### API Key Issues

If you encounter authentication errors:
- Verify all API keys in `.env.local`
- Check API key permissions and quotas
- Ensure keys are properly prefixed (no extra spaces)

### Scraping Failures

If scraping returns empty results:
- Check RapidAPI subscription status
- Verify source URLs are accessible
- Increase timeout values for slow sources

### Video Rendering Issues

If Remotion fails to render:
- Ensure FFmpeg is installed (`ffmpeg -version`)
- Check AWS credentials for cloud rendering
- Verify composition ID matches Remotion project

### Content Quality Issues

If generated content is low quality:
- Provide more detailed insights from research
- Adjust prompt engineering in `buildPrompt()`
- Switch between Claude and OpenAI providers
- Increase max_tokens for longer content

### Rate Limiting

To handle API rate limits:

```typescript
// lib/utils/rate-limiter.ts
export async function withRateLimit<T>(
  fn: () => Promise<T>,
  delayMs: number = 1000
): Promise<T> {
  await new Promise(resolve => setTimeout(resolve, delayMs));
  return fn();
}
```

## Best Practices

1. **Batch Processing**: Process multiple keywords in parallel with rate limiting
2. **Caching**: Cache research results to avoid redundant API calls
3. **Error Handling**: Always wrap API calls in try-catch blocks
4. **Monitoring**: Log pipeline stages for debugging
5. **Content Review**: Review AI-generated content before publishing
6. **Video Optimization**: Compress videos for faster uploads

## Integration Tips

- Store generated content in a database for version control
- Use a job queue (Bull, BullMQ) for long-running pipelines
- Implement webhooks for pipeline completion notifications
- Add authentication middleware for production APIs
- Set up monitoring with Sentry or similar tools
