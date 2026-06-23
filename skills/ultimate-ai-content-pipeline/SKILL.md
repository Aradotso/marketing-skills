---
name: ultimate-ai-content-pipeline
description: Automated content pipeline from research to script to video generation using Claude, OpenAI, and Remotion
triggers:
  - how do I generate content with the AI pipeline
  - automate content creation from research to video
  - use the marketing content automation system
  - create automated blog posts and videos
  - set up the content generation pipeline
  - research and generate content automatically
  - build an AI content workflow
  - generate videos from text content
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

Ultimate AI Content Pipeline is a comprehensive TypeScript-based system that automates the entire content creation workflow: from researching trending topics, generating multi-format content (blog posts, case studies, how-tos), to rendering videos and infographics. It leverages Claude 3, OpenAI, web scraping, and Remotion for video generation.

## What It Does

- **Auto-Research**: Crawls and analyzes real-time data from TechCrunch, a16z, Twitter/X, LinkedIn
- **Content Generation**: Creates content in multiple formats (toplist, POV, case study, how-to) in English and Vietnamese
- **Video Rendering**: Automatically generates infographics and short videos from content using Remotion
- **Multi-Platform**: Optimized output for Reels, TikTok, Shorts
- **Flexible Architecture**: Integrates with OpenAI, Anthropic Claude, and RapidAPI

## Installation

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install

# Set up environment variables
cp .env.example .env
```

## Configuration

Create a `.env` file with the following variables:

```bash
# AI Services
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key
TWITTER_BEARER_TOKEN=your_twitter_token

# Database (if applicable)
DATABASE_URL=your_database_url

# Remotion (Video Generation)
REMOTION_CONCURRENCY=4

# Application
NEXT_PUBLIC_API_URL=http://localhost:3000
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

## Core Architecture

### 1. Research Module

The research module crawls and analyzes content from multiple sources:

```typescript
// services/research/crawler.ts
import axios from 'axios';

interface ResearchSource {
  url: string;
  type: 'news' | 'social' | 'blog';
}

export async function crawlSources(keyword: string): Promise<any[]> {
  const sources: ResearchSource[] = [
    { url: 'https://techcrunch.com/search', type: 'news' },
    { url: 'https://api.twitter.com/2/tweets/search/recent', type: 'social' }
  ];
  
  const results = await Promise.all(
    sources.map(async (source) => {
      try {
        const response = await axios.get(source.url, {
          params: { q: keyword, hours: 24 },
          headers: {
            'Authorization': `Bearer ${process.env.TWITTER_BEARER_TOKEN}`
          }
        });
        return {
          source: source.type,
          data: response.data
        };
      } catch (error) {
        console.error(`Error crawling ${source.url}:`, error);
        return null;
      }
    })
  );
  
  return results.filter(r => r !== null);
}

export function extractInsights(rawData: any[]): string[] {
  const insights: string[] = [];
  
  rawData.forEach(source => {
    if (source.type === 'news') {
      // Extract headlines and key points
      insights.push(...source.data.articles.map((a: any) => a.title));
    } else if (source.type === 'social') {
      // Extract trending topics
      insights.push(...source.data.tweets.map((t: any) => t.text));
    }
  });
  
  return insights;
}
```

### 2. Content Generation

Generate content using AI models with multiple format support:

```typescript
// services/content/generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY
});

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY
});

interface ContentConfig {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  insights: string[];
}

export async function generateContent(config: ContentConfig): Promise<string> {
  const prompt = buildPrompt(config);
  
  // Use Claude for long-form content
  const message = await anthropic.messages.create({
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

function buildPrompt(config: ContentConfig): string {
  const formatInstructions = {
    'toplist': 'Create a numbered list article with detailed explanations',
    'pov': 'Write from a unique perspective with personal insights',
    'case-study': 'Analyze a real-world example with data and outcomes',
    'how-to': 'Provide step-by-step instructions with practical examples'
  };
  
  const toneInstructions = {
    'expert': 'Use professional, authoritative language',
    'friendly': 'Use conversational, approachable language',
    'humorous': 'Include wit and humor while maintaining value'
  };
  
  return `
You are a content creator writing about: ${config.keyword}

Format: ${formatInstructions[config.format]}
Tone: ${toneInstructions[config.tone]}
Language: ${config.language === 'en' ? 'English' : 'Vietnamese'}

Recent insights and data:
${config.insights.join('\n')}

Create comprehensive, engaging content that:
1. Uses the provided insights and data
2. Follows the specified format strictly
3. Matches the tone perfectly
4. Is SEO-optimized and reader-friendly
5. Includes actionable takeaways
`;
}
```

### 3. Multi-Language Support

Generate content in both English and Vietnamese simultaneously:

```typescript
// services/content/multilingual.ts
export async function generateMultilingualContent(
  keyword: string,
  format: string,
  insights: string[]
): Promise<{ en: string; vi: string }> {
  const [enContent, viContent] = await Promise.all([
    generateContent({
      keyword,
      format: format as any,
      language: 'en',
      tone: 'expert',
      insights
    }),
    generateContent({
      keyword,
      format: format as any,
      language: 'vi',
      tone: 'expert',
      insights
    })
  ]);
  
  return { en: enContent, vi: viContent };
}
```

### 4. Video Generation with Remotion

Transform text content into video format:

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, interpolate } from 'remotion';
import React from 'react';

interface ContentVideoProps {
  title: string;
  points: string[];
  duration: number;
}

export const ContentVideo: React.FC<ContentVideoProps> = ({
  title,
  points,
  duration
}) => {
  const frame = useCurrentFrame();
  const fps = 30;
  
  const opacity = interpolate(
    frame,
    [0, fps, duration * fps - fps, duration * fps],
    [0, 1, 1, 0]
  );
  
  return (
    <AbsoluteFill
      style={{
        backgroundColor: '#1a1a2e',
        justifyContent: 'center',
        alignItems: 'center',
        opacity
      }}
    >
      <div style={{ color: 'white', fontSize: 60, fontWeight: 'bold', marginBottom: 40 }}>
        {title}
      </div>
      <div style={{ fontSize: 32, color: '#e0e0e0' }}>
        {points.map((point, i) => (
          <div key={i} style={{ marginBottom: 20 }}>
            {i + 1}. {point}
          </div>
        ))}
      </div>
    </AbsoluteFill>
  );
};
```

Render video programmatically:

```typescript
// services/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoContent {
  title: string;
  points: string[];
}

export async function renderContentVideo(
  content: VideoContent,
  outputPath: string
): Promise<string> {
  const compositionId = 'ContentVideo';
  const bundleLocation = await bundle(
    path.join(process.cwd(), 'remotion/index.ts')
  );
  
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: content
  });
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: content,
    concurrency: parseInt(process.env.REMOTION_CONCURRENCY || '4')
  });
  
  return outputPath;
}
```

### 5. Complete Pipeline Integration

Orchestrate the entire workflow:

```typescript
// services/pipeline/orchestrator.ts
export interface PipelineConfig {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  generateVideo: boolean;
  languages: ('en' | 'vi')[];
}

export interface PipelineResult {
  content: { en?: string; vi?: string };
  videoPath?: string;
  metadata: {
    keyword: string;
    format: string;
    generatedAt: Date;
  };
}

export async function runContentPipeline(
  config: PipelineConfig
): Promise<PipelineResult> {
  console.log(`Starting pipeline for keyword: ${config.keyword}`);
  
  // Step 1: Research
  console.log('Step 1: Researching...');
  const rawData = await crawlSources(config.keyword);
  const insights = extractInsights(rawData);
  
  // Step 2: Generate Content
  console.log('Step 2: Generating content...');
  const content: { en?: string; vi?: string } = {};
  
  for (const lang of config.languages) {
    content[lang] = await generateContent({
      keyword: config.keyword,
      format: config.format,
      language: lang,
      tone: 'expert',
      insights
    });
  }
  
  // Step 3: Generate Video (if enabled)
  let videoPath: string | undefined;
  if (config.generateVideo && content.en) {
    console.log('Step 3: Rendering video...');
    const videoContent = extractVideoContent(content.en);
    videoPath = await renderContentVideo(
      videoContent,
      `./output/${config.keyword}-${Date.now()}.mp4`
    );
  }
  
  return {
    content,
    videoPath,
    metadata: {
      keyword: config.keyword,
      format: config.format,
      generatedAt: new Date()
    }
  };
}

function extractVideoContent(content: string): VideoContent {
  const lines = content.split('\n').filter(l => l.trim());
  const title = lines[0].replace(/^#+ /, '');
  const points = lines
    .filter(l => l.match(/^\d+\./))
    .map(l => l.replace(/^\d+\.\s*/, ''))
    .slice(0, 5);
  
  return { title, points };
}
```

### 6. API Routes (Next.js)

Expose the pipeline via API endpoints:

```typescript
// pages/api/generate.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { runContentPipeline } from '../../services/pipeline/orchestrator';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }
  
  try {
    const { keyword, format, generateVideo, languages } = req.body;
    
    if (!keyword || !format) {
      return res.status(400).json({ error: 'Missing required fields' });
    }
    
    const result = await runContentPipeline({
      keyword,
      format,
      generateVideo: generateVideo ?? false,
      languages: languages ?? ['en']
    });
    
    res.status(200).json(result);
  } catch (error) {
    console.error('Pipeline error:', error);
    res.status(500).json({ 
      error: 'Pipeline execution failed',
      message: error instanceof Error ? error.message : 'Unknown error'
    });
  }
}
```

## Common Workflows

### Generate Content with Research

```typescript
import { runContentPipeline } from './services/pipeline/orchestrator';

async function example() {
  const result = await runContentPipeline({
    keyword: 'AI Marketing Automation',
    format: 'toplist',
    generateVideo: true,
    languages: ['en', 'vi']
  });
  
  console.log('English content:', result.content.en);
  console.log('Vietnamese content:', result.content.vi);
  console.log('Video path:', result.videoPath);
}
```

### Custom Research Sources

```typescript
// Add custom sources to the crawler
const customSources = [
  { url: 'https://news.ycombinator.com', type: 'news' as const },
  { url: 'https://www.producthunt.com', type: 'news' as const }
];

// Extend the crawler
export async function crawlCustomSources(keyword: string) {
  // Implementation similar to crawlSources
}
```

### Batch Content Generation

```typescript
async function batchGenerate(keywords: string[]) {
  const results = await Promise.all(
    keywords.map(keyword =>
      runContentPipeline({
        keyword,
        format: 'how-to',
        generateVideo: false,
        languages: ['en']
      })
    )
  );
  
  return results;
}
```

## Troubleshooting

### API Rate Limits

If you hit rate limits with AI APIs:

```typescript
// Add retry logic with exponential backoff
async function generateWithRetry(config: ContentConfig, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await generateContent(config);
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      await new Promise(resolve => setTimeout(resolve, 2 ** i * 1000));
    }
  }
}
```

### Video Rendering Issues

If Remotion rendering fails:

```bash
# Install system dependencies
sudo apt-get install ffmpeg

# Reduce concurrency in .env
REMOTION_CONCURRENCY=2
```

### Memory Issues with Large Content

```typescript
// Stream content generation for large batches
import { pipeline } from 'stream/promises';

async function streamGeneration(keywords: string[]) {
  for (const keyword of keywords) {
    const result = await runContentPipeline({
      keyword,
      format: 'toplist',
      generateVideo: false,
      languages: ['en']
    });
    
    // Process and save immediately
    await saveResult(result);
  }
}
```

### Missing Environment Variables

```typescript
// Validate environment on startup
function validateEnv() {
  const required = [
    'OPENAI_API_KEY',
    'ANTHROPIC_API_KEY',
    'RAPIDAPI_KEY'
  ];
  
  const missing = required.filter(key => !process.env[key]);
  
  if (missing.length > 0) {
    throw new Error(`Missing required environment variables: ${missing.join(', ')}`);
  }
}

validateEnv();
```

## Best Practices

1. **Cache Research Data**: Store crawled data to avoid redundant API calls
2. **Queue Video Rendering**: Use a job queue for video generation to avoid blocking
3. **Rate Limit Protection**: Implement request throttling for AI APIs
4. **Content Validation**: Verify generated content meets quality standards before publishing
5. **Error Logging**: Track failures in research, generation, and rendering stages
