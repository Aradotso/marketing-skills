---
name: marketing-content-pipeline-automation
description: Automated AI content pipeline for research, scriptwriting, and video generation using Claude/OpenAI and Remotion
triggers:
  - how do I automate content creation with AI research
  - set up marketing content pipeline with Claude and OpenAI
  - generate videos automatically from AI-written content
  - crawl news sources and create content with AI
  - build automated marketing pipeline with Remotion
  - create multi-language content posts with AI automation
  - research and generate social media content automatically
  - automate content from research to video rendering
---

# Marketing Content Pipeline Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with **Ultimate AI Content Pipeline** (pennydinh/marketing-pineline-share), an end-to-end automated content creation system that handles research, scriptwriting, multi-format content generation, and video rendering using Claude 3, OpenAI, and Remotion.

## What This Project Does

The Ultimate AI Content Pipeline is a TypeScript-based Next.js application that automates the entire content creation workflow:

1. **Auto-Research**: Crawls news sources (TechCrunch, a16z, Twitter/X, LinkedIn) for latest data
2. **AI Content Generation**: Creates multi-format content (toplist, POV, case study, how-to) in multiple languages
3. **Video Generation**: Renders infographics and short videos using Remotion
4. **Multi-Platform Optimization**: Outputs content optimized for Reels, TikTok, and Shorts

## Installation

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
yarn install

# Set up environment variables
cp .env.example .env.local
```

## Configuration

Create a `.env.local` file with the following variables:

```bash
# AI APIs
OPENAI_API_KEY=your_openai_key_here
ANTHROPIC_API_KEY=your_claude_api_key_here

# RapidAPI for news crawling
RAPIDAPI_KEY=your_rapidapi_key_here

# Next.js Configuration
NEXT_PUBLIC_API_URL=http://localhost:3000

# Remotion Configuration (optional)
REMOTION_AWS_ACCESS_KEY_ID=your_aws_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret
```

## Key API Endpoints

### Research Endpoint

```typescript
// pages/api/research.ts
import type { NextApiRequest, NextApiResponse } from 'next';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  const { keyword, sources = ['techcrunch', 'twitter'], timeframe = '24h' } = req.body;
  
  // Crawl news sources
  const researchData = await crawlSources(keyword, sources, timeframe);
  
  res.status(200).json({ data: researchData });
}
```

### Content Generation Endpoint

```typescript
// pages/api/generate-content.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

export default async function handler(req: NextApiRequest, res: NextApiResponse) {
  const { research, format, language, tone } = req.body;
  
  // Generate content with Claude
  const message = await anthropic.messages.create({
    model: "claude-3-5-sonnet-20241022",
    max_tokens: 4096,
    messages: [{
      role: "user",
      content: `Create ${format} content in ${language} with ${tone} tone based on: ${research}`
    }],
  });
  
  res.status(200).json({ content: message.content });
}
```

### Video Rendering Endpoint

```typescript
// pages/api/render-video.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

export default async function handler(req: NextApiRequest, res: NextApiResponse) {
  const { content, format = 'mp4', aspectRatio = '9:16' } = req.body;
  
  const compositionId = 'ContentVideo';
  const bundleLocation = await bundle(path.join(process.cwd(), 'src/remotion/index.ts'));
  const composition = await selectComposition({ serveUrl: bundleLocation, id: compositionId });
  
  const outputLocation = path.join(process.cwd(), 'public/videos', `output-${Date.now()}.${format}`);
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: { content, aspectRatio },
  });
  
  res.status(200).json({ videoUrl: outputLocation });
}
```

## Core Usage Patterns

### 1. Complete Content Pipeline

```typescript
// lib/contentPipeline.ts
import { crawlNewsData } from './research';
import { generateContent } from './ai-generation';
import { renderVideo } from './video-renderer';

export async function runContentPipeline(keyword: string, options: PipelineOptions) {
  // Step 1: Research
  const research = await crawlNewsData({
    keyword,
    sources: options.sources || ['techcrunch', 'twitter', 'linkedin'],
    timeframe: options.timeframe || '24h',
  });
  
  // Step 2: Generate content
  const content = await generateContent({
    research,
    format: options.format || 'toplist',
    language: options.language || 'en',
    tone: options.tone || 'professional',
    aiProvider: options.aiProvider || 'claude',
  });
  
  // Step 3: Create bilingual versions
  const translations = await Promise.all(
    options.languages.map(lang => 
      generateContent({ ...content, language: lang })
    )
  );
  
  // Step 4: Render video (optional)
  let video = null;
  if (options.generateVideo) {
    video = await renderVideo({
      content: content.text,
      aspectRatio: options.aspectRatio || '9:16',
      platform: options.platform || 'reels',
    });
  }
  
  return { research, content, translations, video };
}
```

### 2. Research Data Crawling

```typescript
// lib/research/crawler.ts
import axios from 'axios';

interface CrawlOptions {
  keyword: string;
  sources: string[];
  timeframe: string;
}

export async function crawlNewsData(options: CrawlOptions) {
  const { keyword, sources, timeframe } = options;
  const results = [];
  
  for (const source of sources) {
    const apiUrl = getSourceApiUrl(source);
    
    const response = await axios.get(apiUrl, {
      params: { q: keyword, time: timeframe },
      headers: {
        'X-RapidAPI-Key': process.env.RAPIDAPI_KEY,
        'X-RapidAPI-Host': getSourceHost(source),
      },
    });
    
    results.push(...response.data.articles);
  }
  
  return analyzeResearch(results);
}

function analyzeResearch(articles: any[]) {
  return {
    insights: extractInsights(articles),
    trends: identifyTrends(articles),
    dataPoints: extractDataPoints(articles),
    sources: articles.map(a => ({ title: a.title, url: a.url })),
  };
}
```

### 3. AI Content Generation with Multiple Providers

```typescript
// lib/ai-generation/generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

interface GenerateOptions {
  research: any;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'professional' | 'friendly' | 'humorous';
  aiProvider: 'claude' | 'openai';
}

export async function generateContent(options: GenerateOptions) {
  const prompt = buildPrompt(options);
  
  if (options.aiProvider === 'claude') {
    return generateWithClaude(prompt);
  } else {
    return generateWithOpenAI(prompt);
  }
}

async function generateWithClaude(prompt: string) {
  const anthropic = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY,
  });
  
  const message = await anthropic.messages.create({
    model: "claude-3-5-sonnet-20241022",
    max_tokens: 4096,
    temperature: 0.7,
    messages: [{ role: "user", content: prompt }],
  });
  
  return parseAIResponse(message.content[0].text);
}

async function generateWithOpenAI(prompt: string) {
  const openai = new OpenAI({
    apiKey: process.env.OPENAI_API_KEY,
  });
  
  const completion = await openai.chat.completions.create({
    model: "gpt-4-turbo-preview",
    messages: [{ role: "user", content: prompt }],
    temperature: 0.7,
    max_tokens: 4096,
  });
  
  return parseAIResponse(completion.choices[0].message.content);
}

function buildPrompt(options: GenerateOptions) {
  const formatInstructions = {
    'toplist': 'Create a numbered list article',
    'pov': 'Write from a specific point of view',
    'case-study': 'Analyze a real-world case with data',
    'how-to': 'Create a step-by-step guide',
  };
  
  return `
${formatInstructions[options.format]} in ${options.language} with a ${options.tone} tone.

Research Data:
${JSON.stringify(options.research, null, 2)}

Requirements:
- Use data-backed insights from the research
- Include specific numbers and statistics
- Make it engaging and shareable
- Optimize for social media platforms
`;
}
```

### 4. Remotion Video Generation

```typescript
// src/remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig, interpolate } from 'remotion';

interface ContentVideoProps {
  content: string;
  aspectRatio: '9:16' | '16:9' | '1:1';
}

export const ContentVideo: React.FC<ContentVideoProps> = ({ content, aspectRatio }) => {
  const frame = useCurrentFrame();
  const { fps, durationInFrames } = useVideoConfig();
  
  const opacity = interpolate(frame, [0, 30], [0, 1], { extrapolateRight: 'clamp' });
  
  const segments = content.split('\n\n');
  const currentSegment = Math.floor((frame / durationInFrames) * segments.length);
  
  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a1a', opacity }}>
      <div style={{
        display: 'flex',
        flexDirection: 'column',
        justifyContent: 'center',
        alignItems: 'center',
        padding: '40px',
        color: 'white',
        fontSize: aspectRatio === '9:16' ? '28px' : '36px',
        textAlign: 'center',
      }}>
        <h1 style={{ marginBottom: '20px' }}>
          {segments[currentSegment]?.split('\n')[0]}
        </h1>
        <p style={{ fontSize: '0.8em', lineHeight: '1.5' }}>
          {segments[currentSegment]?.split('\n').slice(1).join('\n')}
        </p>
      </div>
    </AbsoluteFill>
  );
};
```

```typescript
// lib/video-renderer/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface RenderOptions {
  content: string;
  aspectRatio: '9:16' | '16:9' | '1:1';
  platform: 'reels' | 'tiktok' | 'shorts' | 'youtube';
}

export async function renderVideo(options: RenderOptions) {
  const { content, aspectRatio, platform } = options;
  
  const bundleLocation = await bundle({
    entryPoint: path.join(process.cwd(), 'src/remotion/index.ts'),
    webpackOverride: (config) => config,
  });
  
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: { content, aspectRatio },
  });
  
  const outputLocation = path.join(
    process.cwd(),
    'public/videos',
    `${platform}-${Date.now()}.mp4`
  );
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: { content, aspectRatio },
  });
  
  return { videoPath: outputLocation, platform };
}
```

## Frontend Integration Example

```typescript
// pages/index.tsx
import { useState } from 'react';

export default function Home() {
  const [keyword, setKeyword] = useState('');
  const [result, setResult] = useState(null);
  const [loading, setLoading] = useState(false);
  
  const handleGenerate = async () => {
    setLoading(true);
    
    const response = await fetch('/api/pipeline', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        keyword,
        format: 'toplist',
        languages: ['en', 'vi'],
        generateVideo: true,
        aspectRatio: '9:16',
        sources: ['techcrunch', 'twitter'],
      }),
    });
    
    const data = await response.json();
    setResult(data);
    setLoading(false);
  };
  
  return (
    <div className="container">
      <h1>AI Content Pipeline</h1>
      <input
        type="text"
        value={keyword}
        onChange={(e) => setKeyword(e.target.value)}
        placeholder="Enter keyword..."
      />
      <button onClick={handleGenerate} disabled={loading}>
        {loading ? 'Generating...' : 'Generate Content'}
      </button>
      
      {result && (
        <div className="results">
          <h2>Research Insights</h2>
          <pre>{JSON.stringify(result.research.insights, null, 2)}</pre>
          
          <h2>Generated Content</h2>
          <div>{result.content.text}</div>
          
          {result.video && (
            <div>
              <h2>Video</h2>
              <video src={result.video.videoPath} controls />
            </div>
          )}
        </div>
      )}
    </div>
  );
}
```

## Troubleshooting

### API Rate Limits

```typescript
// lib/utils/rate-limiter.ts
export class RateLimiter {
  private queue: Array<() => Promise<any>> = [];
  private processing = false;
  
  async add<T>(fn: () => Promise<T>, delay: number = 1000): Promise<T> {
    return new Promise((resolve, reject) => {
      this.queue.push(async () => {
        try {
          const result = await fn();
          resolve(result);
        } catch (error) {
          reject(error);
        }
        await new Promise(r => setTimeout(r, delay));
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
      if (fn) await fn();
    }
    this.processing = false;
  }
}
```

### Video Rendering Errors

If Remotion rendering fails, ensure FFmpeg is installed:

```bash
# macOS
brew install ffmpeg

# Ubuntu/Debian
sudo apt-get install ffmpeg

# Windows (using Chocolatey)
choco install ffmpeg
```

### Memory Issues with Large Content

```typescript
// Implement streaming for large content generation
async function generateLargeContent(options: GenerateOptions) {
  const stream = await anthropic.messages.stream({
    model: "claude-3-5-sonnet-20241022",
    max_tokens: 8192,
    messages: [{ role: "user", content: buildPrompt(options) }],
  });
  
  let fullContent = '';
  for await (const chunk of stream) {
    if (chunk.type === 'content_block_delta') {
      fullContent += chunk.delta.text;
    }
  }
  
  return fullContent;
}
```

### Environment Variable Issues

```typescript
// lib/config/validate-env.ts
export function validateEnvironment() {
  const required = [
    'OPENAI_API_KEY',
    'ANTHROPIC_API_KEY',
    'RAPIDAPI_KEY',
  ];
  
  const missing = required.filter(key => !process.env[key]);
  
  if (missing.length > 0) {
    throw new Error(`Missing required environment variables: ${missing.join(', ')}`);
  }
}
```

## Common Workflows

### Daily Content Generation

```typescript
// scripts/daily-generation.ts
import { runContentPipeline } from '../lib/contentPipeline';

async function dailyContentGeneration() {
  const keywords = ['AI trends', 'Marketing automation', 'Social media'];
  
  for (const keyword of keywords) {
    const result = await runContentPipeline(keyword, {
      format: 'toplist',
      languages: ['en', 'vi'],
      generateVideo: true,
      sources: ['techcrunch', 'twitter', 'linkedin'],
      timeframe: '24h',
    });
    
    // Save to database or CMS
    await saveContent(result);
  }
}
```

This skill provides comprehensive coverage of the marketing content pipeline automation system, enabling AI agents to effectively assist developers in setting up, configuring, and using this powerful content generation tool.
