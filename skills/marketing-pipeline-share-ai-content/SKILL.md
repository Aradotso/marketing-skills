---
name: marketing-pipeline-share-ai-content
description: Automate content research, scriptwriting, and video generation with AI-powered marketing pipeline
triggers:
  - how do I set up the AI content pipeline
  - generate content with automated research
  - create video content from text automatically
  - use Claude and OpenAI for content automation
  - crawl news sources for content research
  - render videos with Remotion integration
  - automate marketing content workflow
  - build AI-powered content generation system
---

# Marketing Pipeline Share - AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the Ultimate AI Content Pipeline, a TypeScript-based system that automates the entire content creation workflow from research and scriptwriting to video generation. It integrates Claude 3, OpenAI, web scraping, and Remotion for end-to-end content automation.

## What It Does

The Marketing Pipeline Share automates:

1. **Auto-Research**: Crawls live data from TechCrunch, a16z, Twitter/X, LinkedIn for fresh insights
2. **AI Content Generation**: Creates multi-format content (lists, POV, case studies, how-tos) in multiple languages
3. **Video Rendering**: Automatically generates infographics and short-form videos using Remotion
4. **Multi-Platform Optimization**: Outputs content optimized for Reels, TikTok, Shorts

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

## Configuration

Create a `.env.local` file in the project root:

```bash
# AI Model Configuration
OPENAI_API_KEY=your_openai_key_here
ANTHROPIC_API_KEY=your_claude_key_here

# Research & Scraping APIs
RAPIDAPI_KEY=your_rapidapi_key_here

# Database (if applicable)
DATABASE_URL=your_database_url_here

# Remotion Configuration
REMOTION_LICENSE_KEY=your_remotion_license_here

# Optional: Content Settings
DEFAULT_LANGUAGE=en
CONTENT_TONE=professional
```

## Key Components

### 1. Research Module (Content Crawling)

The research system crawls news sources and extracts insights:

```typescript
// lib/research/crawler.ts
import axios from 'axios';

interface ResearchConfig {
  sources: string[];
  keywords: string[];
  timeframe: '24h' | '7d' | '30d';
}

export async function crawlContent(config: ResearchConfig) {
  const articles: any[] = [];
  
  for (const source of config.sources) {
    try {
      const response = await axios.get(`https://api.source.com/search`, {
        params: {
          q: config.keywords.join(' OR '),
          timeframe: config.timeframe,
        },
        headers: {
          'X-RapidAPI-Key': process.env.RAPIDAPI_KEY,
        },
      });
      
      articles.push(...response.data.articles);
    } catch (error) {
      console.error(`Failed to crawl ${source}:`, error);
    }
  }
  
  return articles;
}

export async function extractInsights(articles: any[]) {
  const insights = articles.map(article => ({
    title: article.title,
    summary: article.description,
    url: article.url,
    publishedAt: article.publishedAt,
    sentiment: analyzeSentiment(article.content),
  }));
  
  return insights;
}
```

### 2. AI Content Generation

Generate content using Claude or OpenAI:

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

interface ContentRequest {
  topic: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  tone: 'expert' | 'friendly' | 'humorous';
  language: 'en' | 'vi';
  researchData?: any[];
}

export async function generateWithClaude(request: ContentRequest) {
  const prompt = buildPrompt(request);
  
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4000,
    messages: [
      {
        role: 'user',
        content: prompt,
      },
    ],
  });
  
  return message.content[0].type === 'text' 
    ? message.content[0].text 
    : '';
}

export async function generateWithOpenAI(request: ContentRequest) {
  const prompt = buildPrompt(request);
  
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: 'You are an expert content creator specializing in marketing content.',
      },
      {
        role: 'user',
        content: prompt,
      },
    ],
    temperature: 0.7,
  });
  
  return completion.choices[0].message.content || '';
}

function buildPrompt(request: ContentRequest): string {
  const formatInstructions = {
    'toplist': 'Create a numbered list article with clear points',
    'pov': 'Write from a unique perspective with strong opinions',
    'case-study': 'Analyze with data, examples, and takeaways',
    'how-to': 'Provide step-by-step actionable instructions',
  };
  
  let prompt = `Create a ${request.format} article about: ${request.topic}\n`;
  prompt += `Tone: ${request.tone}\n`;
  prompt += `Language: ${request.language}\n`;
  prompt += `Format instructions: ${formatInstructions[request.format]}\n\n`;
  
  if (request.researchData && request.researchData.length > 0) {
    prompt += 'Use these recent insights:\n';
    request.researchData.forEach((item, idx) => {
      prompt += `${idx + 1}. ${item.title}: ${item.summary}\n`;
    });
  }
  
  return prompt;
}
```

### 3. Video Generation with Remotion

Render videos from generated content:

```typescript
// lib/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  title: string;
  content: string[];
  style: 'minimal' | 'bold' | 'professional';
  aspectRatio: '9:16' | '16:9' | '1:1';
}

export async function renderContentVideo(config: VideoConfig) {
  const compositionId = 'ContentVideo';
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: (config) => config,
  });
  
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: {
      title: config.title,
      content: config.content,
      style: config.style,
    },
  });
  
  const outputLocation = `out/${Date.now()}-${config.title.toLowerCase().replace(/\s+/g, '-')}.mp4`;
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: {
      title: config.title,
      content: config.content,
      style: config.style,
    },
  });
  
  return outputLocation;
}
```

### 4. Remotion Video Component

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig, interpolate } from 'remotion';
import React from 'react';

interface ContentVideoProps {
  title: string;
  content: string[];
  style: 'minimal' | 'bold' | 'professional';
}

export const ContentVideo: React.FC<ContentVideoProps> = ({ title, content, style }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  
  const titleOpacity = interpolate(frame, [0, 30], [0, 1], {
    extrapolateRight: 'clamp',
  });
  
  const styleConfig = {
    minimal: { bg: '#ffffff', fg: '#000000', font: 'Arial' },
    bold: { bg: '#000000', fg: '#ffff00', font: 'Impact' },
    professional: { bg: '#1a1a2e', fg: '#eaeaea', font: 'Helvetica' },
  };
  
  const colors = styleConfig[style];
  
  return (
    <AbsoluteFill style={{ backgroundColor: colors.bg }}>
      <div style={{
        display: 'flex',
        flexDirection: 'column',
        justifyContent: 'center',
        alignItems: 'center',
        padding: '40px',
        fontFamily: colors.font,
      }}>
        <h1 style={{
          fontSize: '48px',
          color: colors.fg,
          opacity: titleOpacity,
          textAlign: 'center',
          marginBottom: '40px',
        }}>
          {title}
        </h1>
        
        {content.map((text, idx) => {
          const startFrame = 30 + (idx * 60);
          const opacity = interpolate(frame, [startFrame, startFrame + 20], [0, 1], {
            extrapolateRight: 'clamp',
            extrapolateLeft: 'clamp',
          });
          
          return (
            <p key={idx} style={{
              fontSize: '32px',
              color: colors.fg,
              opacity,
              marginBottom: '20px',
              textAlign: 'center',
            }}>
              {text}
            </p>
          );
        })}
      </div>
    </AbsoluteFill>
  );
};
```

### 5. Complete Pipeline Orchestration

```typescript
// lib/pipeline/orchestrator.ts
import { crawlContent, extractInsights } from '../research/crawler';
import { generateWithClaude } from '../ai/content-generator';
import { renderContentVideo } from '../video/renderer';

interface PipelineConfig {
  keyword: string;
  contentFormat: 'toplist' | 'pov' | 'case-study' | 'how-to';
  languages: ('en' | 'vi')[];
  generateVideo: boolean;
}

export async function runContentPipeline(config: PipelineConfig) {
  console.log('🔍 Step 1: Research phase...');
  const articles = await crawlContent({
    sources: ['techcrunch', 'a16z', 'twitter'],
    keywords: [config.keyword],
    timeframe: '24h',
  });
  
  const insights = await extractInsights(articles);
  console.log(`✅ Found ${insights.length} insights`);
  
  console.log('🤖 Step 2: Generating content...');
  const contentResults = [];
  
  for (const lang of config.languages) {
    const content = await generateWithClaude({
      topic: config.keyword,
      format: config.contentFormat,
      tone: 'expert',
      language: lang,
      researchData: insights.slice(0, 5),
    });
    
    contentResults.push({
      language: lang,
      content,
    });
    
    console.log(`✅ Generated ${lang} content`);
  }
  
  if (config.generateVideo) {
    console.log('🎬 Step 3: Rendering video...');
    const videoContent = contentResults[0].content
      .split('\n')
      .filter(line => line.trim().length > 0)
      .slice(0, 5);
    
    const videoPath = await renderContentVideo({
      title: config.keyword,
      content: videoContent,
      style: 'professional',
      aspectRatio: '9:16',
    });
    
    console.log(`✅ Video rendered: ${videoPath}`);
    
    return { contentResults, videoPath };
  }
  
  return { contentResults };
}
```

## API Routes (Next.js)

```typescript
// pages/api/generate-content.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { runContentPipeline } from '@/lib/pipeline/orchestrator';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }
  
  const { keyword, format, languages, includeVideo } = req.body;
  
  try {
    const result = await runContentPipeline({
      keyword,
      contentFormat: format || 'toplist',
      languages: languages || ['en'],
      generateVideo: includeVideo || false,
    });
    
    res.status(200).json(result);
  } catch (error) {
    console.error('Pipeline error:', error);
    res.status(500).json({ error: 'Failed to generate content' });
  }
}
```

## Usage Examples

### Basic Content Generation

```typescript
import { runContentPipeline } from './lib/pipeline/orchestrator';

// Generate bilingual content with video
const result = await runContentPipeline({
  keyword: 'AI Marketing Trends 2024',
  contentFormat: 'toplist',
  languages: ['en', 'vi'],
  generateVideo: true,
});

console.log(result.contentResults);
console.log(`Video saved to: ${result.videoPath}`);
```

### Custom Research + Content

```typescript
import { crawlContent, extractInsights } from './lib/research/crawler';
import { generateWithClaude } from './lib/ai/content-generator';

// Custom research workflow
const articles = await crawlContent({
  sources: ['techcrunch'],
  keywords: ['startup', 'funding', 'AI'],
  timeframe: '7d',
});

const insights = await extractInsights(articles);

const content = await generateWithClaude({
  topic: 'Top AI Startups Raising Funds This Week',
  format: 'toplist',
  tone: 'expert',
  language: 'en',
  researchData: insights,
});
```

## Common Patterns

### Scheduled Content Generation

```typescript
// lib/scheduler/cron.ts
import cron from 'node-cron';
import { runContentPipeline } from '../pipeline/orchestrator';

// Run daily at 9 AM
cron.schedule('0 9 * * *', async () => {
  console.log('Starting scheduled content generation...');
  
  await runContentPipeline({
    keyword: 'Daily Tech News',
    contentFormat: 'toplist',
    languages: ['en', 'vi'],
    generateVideo: true,
  });
});
```

### Batch Processing Multiple Topics

```typescript
const topics = [
  'AI Marketing Tools',
  'Social Media Trends',
  'Content Creation Tips',
];

const results = await Promise.all(
  topics.map(topic => runContentPipeline({
    keyword: topic,
    contentFormat: 'how-to',
    languages: ['en'],
    generateVideo: false,
  }))
);
```

## Troubleshooting

### API Rate Limits

```typescript
// Add retry logic with exponential backoff
async function withRetry<T>(
  fn: () => Promise<T>,
  maxRetries = 3
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error: any) {
      if (error.status === 429 && i < maxRetries - 1) {
        const delay = Math.pow(2, i) * 1000;
        await new Promise(resolve => setTimeout(resolve, delay));
        continue;
      }
      throw error;
    }
  }
  throw new Error('Max retries exceeded');
}
```

### Video Rendering Memory Issues

```typescript
// Increase Node.js memory limit
// In package.json scripts:
{
  "scripts": {
    "render": "NODE_OPTIONS='--max-old-space-size=4096' node render-script.js"
  }
}
```

### Missing Environment Variables

```typescript
function validateEnv() {
  const required = [
    'OPENAI_API_KEY',
    'ANTHROPIC_API_KEY',
    'RAPIDAPI_KEY',
  ];
  
  const missing = required.filter(key => !process.env[key]);
  
  if (missing.length > 0) {
    throw new Error(`Missing environment variables: ${missing.join(', ')}`);
  }
}

validateEnv();
```

## Development Commands

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Run video rendering locally
npm run render

# Type checking
npm run type-check

# Linting
npm run lint
```

This skill enables AI agents to effectively work with the Marketing Pipeline Share system for automated content creation, research, and video generation workflows.
