---
name: marketing-pipeline-share-automation
description: Automated AI content pipeline for research, scriptwriting, and video generation using Claude, OpenAI, and Remotion
triggers:
  - how do I automate content creation with AI
  - set up marketing pipeline with video generation
  - create automated content workflow from research to video
  - generate content and videos automatically
  - build AI-powered content automation system
  - use Claude and OpenAI for content pipeline
  - automate social media content with AI research
  - create video content from AI-generated scripts
---

# Marketing Pipeline Share - AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

Marketing Pipeline Share is an end-to-end AI content automation system that transforms keywords into finished content and videos. It automatically crawls news sources (TechCrunch, Twitter, LinkedIn), generates articles in multiple formats using Claude/OpenAI, and renders videos with Remotion.

**Key capabilities:**
- Auto-crawl real-time data from news sources
- Generate content in multiple formats (toplist, POV, case study, how-to)
- Multi-language support (English & Vietnamese)
- Automatic video rendering for social media
- Next.js web interface for content management

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

## Environment Configuration

Create a `.env.local` file in the root directory:

```bash
# AI APIs
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# RapidAPI for news crawling
RAPIDAPI_KEY=your_rapidapi_key

# Database (if using)
DATABASE_URL=your_database_connection

# Remotion configuration
REMOTION_AWS_ACCESS_KEY_ID=your_aws_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret
```

## Running the Application

```bash
# Development mode
npm run dev

# Production build
npm run build
npm start

# Render videos (Remotion)
npm run remotion:render
```

Access the application at `http://localhost:3000`

## Core Components

### 1. Research & Data Crawling

The system automatically fetches trending content from multiple sources:

```typescript
// services/research/crawler.ts
import axios from 'axios';

interface ResearchData {
  source: string;
  title: string;
  content: string;
  publishedAt: Date;
}

export async function crawlTechNews(keyword: string): Promise<ResearchData[]> {
  const options = {
    method: 'GET',
    url: 'https://api.rapidapi.com/news/search',
    params: {
      q: keyword,
      language: 'en',
      sortBy: 'publishedAt',
      pageSize: 10
    },
    headers: {
      'X-RapidAPI-Key': process.env.RAPIDAPI_KEY,
      'X-RapidAPI-Host': 'news-api.rapidapi.com'
    }
  };

  try {
    const response = await axios.request(options);
    return response.data.articles.map((article: any) => ({
      source: article.source.name,
      title: article.title,
      content: article.description,
      publishedAt: new Date(article.publishedAt)
    }));
  } catch (error) {
    console.error('Crawl error:', error);
    return [];
  }
}
```

### 2. AI Content Generation

Generate content using Claude or OpenAI:

```typescript
// services/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

export interface ContentConfig {
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  researchData: ResearchData[];
}

export async function generateContentWithClaude(
  keyword: string,
  config: ContentConfig
): Promise<string> {
  const prompt = buildPrompt(keyword, config);

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [
      {
        role: 'user',
        content: prompt
      }
    ],
  });

  return message.content[0].type === 'text' 
    ? message.content[0].text 
    : '';
}

export async function generateContentWithOpenAI(
  keyword: string,
  config: ContentConfig
): Promise<string> {
  const prompt = buildPrompt(keyword, config);

  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: 'You are an expert content writer specializing in marketing and tech.'
      },
      {
        role: 'user',
        content: prompt
      }
    ],
    temperature: 0.7,
    max_tokens: 4000,
  });

  return completion.choices[0].message.content || '';
}

function buildPrompt(keyword: string, config: ContentConfig): string {
  const formatInstructions = {
    'toplist': 'Create a numbered list article highlighting the top items',
    'pov': 'Write from a specific point of view with personal insights',
    'case-study': 'Analyze a real-world example with data and outcomes',
    'how-to': 'Provide step-by-step instructions and actionable advice'
  };

  const toneStyle = {
    'expert': 'professional and authoritative',
    'friendly': 'conversational and approachable',
    'humorous': 'witty and entertaining'
  };

  const researchContext = config.researchData
    .map(r => `- ${r.title}: ${r.content}`)
    .join('\n');

  return `
Write a ${config.language === 'vi' ? 'Vietnamese' : 'English'} article about "${keyword}"
Format: ${formatInstructions[config.format]}
Tone: ${toneStyle[config.tone]}

Recent research data:
${researchContext}

Requirements:
- Use data-backed insights from the research
- Include specific examples and statistics
- Make it engaging and actionable
- Optimize for social media sharing
`;
}
```

### 3. Video Generation with Remotion

```typescript
// remotion/compositions/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, interpolate } from 'remotion';
import React from 'react';

export interface VideoProps {
  title: string;
  points: string[];
  duration: number;
}

export const ContentVideo: React.FC<VideoProps> = ({ title, points, duration }) => {
  const frame = useCurrentFrame();
  
  const opacity = interpolate(
    frame,
    [0, 30],
    [0, 1],
    { extrapolateRight: 'clamp' }
  );

  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a2e' }}>
      <div style={{ 
        padding: 60, 
        opacity,
        color: 'white',
        fontFamily: 'Arial, sans-serif'
      }}>
        <h1 style={{ fontSize: 48, marginBottom: 40 }}>
          {title}
        </h1>
        <div>
          {points.map((point, index) => {
            const pointOpacity = interpolate(
              frame,
              [30 + index * 20, 50 + index * 20],
              [0, 1],
              { extrapolateRight: 'clamp' }
            );
            
            return (
              <p 
                key={index}
                style={{ 
                  fontSize: 24, 
                  marginBottom: 20,
                  opacity: pointOpacity 
                }}
              >
                {index + 1}. {point}
              </p>
            );
          })}
        </div>
      </div>
    </AbsoluteFill>
  );
};
```

Register composition:

```typescript
// remotion/index.ts
import { registerRoot } from 'remotion';
import { Composition } from 'remotion';
import { ContentVideo } from './compositions/ContentVideo';

export const RemotionRoot: React.FC = () => {
  return (
    <>
      <Composition
        id="ContentVideo"
        component={ContentVideo}
        durationInFrames={300}
        fps={30}
        width={1080}
        height={1920}
        defaultProps={{
          title: 'Default Title',
          points: ['Point 1', 'Point 2', 'Point 3'],
          duration: 10
        }}
      />
    </>
  );
};

registerRoot(RemotionRoot);
```

### 4. Complete Pipeline Orchestration

```typescript
// services/pipeline/orchestrator.ts
import { crawlTechNews } from '../research/crawler';
import { generateContentWithClaude } from '../ai/content-generator';
import { renderVideo } from '../video/renderer';

export interface PipelineConfig {
  keyword: string;
  contentFormat: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  generateVideo: boolean;
}

export async function runContentPipeline(config: PipelineConfig) {
  console.log(`Starting pipeline for keyword: ${config.keyword}`);

  // Step 1: Research
  console.log('Step 1: Crawling research data...');
  const researchData = await crawlTechNews(config.keyword);
  console.log(`Found ${researchData.length} articles`);

  // Step 2: Generate content
  console.log('Step 2: Generating content with AI...');
  const content = await generateContentWithClaude(config.keyword, {
    format: config.contentFormat,
    language: config.language,
    tone: config.tone,
    researchData
  });

  // Step 3: Extract key points for video
  const keyPoints = extractKeyPoints(content);

  // Step 4: Generate video (optional)
  let videoUrl = null;
  if (config.generateVideo) {
    console.log('Step 3: Rendering video...');
    videoUrl = await renderVideo({
      title: config.keyword,
      points: keyPoints,
      duration: 10
    });
  }

  return {
    content,
    keyPoints,
    videoUrl,
    researchSources: researchData.map(r => r.source)
  };
}

function extractKeyPoints(content: string): string[] {
  // Simple extraction - look for numbered lists or headers
  const lines = content.split('\n');
  const points = lines
    .filter(line => /^\d+\./.test(line.trim()) || /^##/.test(line.trim()))
    .map(line => line.replace(/^\d+\.\s*/, '').replace(/^##\s*/, '').trim())
    .slice(0, 5);
  
  return points.length > 0 ? points : [content.slice(0, 200)];
}
```

### 5. API Routes (Next.js)

```typescript
// pages/api/generate-content.ts
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
    const { keyword, format, language, tone, generateVideo } = req.body;

    if (!keyword) {
      return res.status(400).json({ error: 'Keyword is required' });
    }

    const result = await runContentPipeline({
      keyword,
      contentFormat: format || 'toplist',
      language: language || 'en',
      tone: tone || 'friendly',
      generateVideo: generateVideo || false
    });

    res.status(200).json(result);
  } catch (error) {
    console.error('Pipeline error:', error);
    res.status(500).json({ 
      error: 'Failed to generate content',
      message: error instanceof Error ? error.message : 'Unknown error'
    });
  }
}
```

### 6. Frontend Usage

```typescript
// components/ContentGenerator.tsx
import { useState } from 'react';

export default function ContentGenerator() {
  const [keyword, setKeyword] = useState('');
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState<any>(null);

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
          tone: 'friendly',
          generateVideo: true
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
      <h1 className="text-3xl font-bold mb-4">AI Content Generator</h1>
      
      <div className="mb-4">
        <input
          type="text"
          value={keyword}
          onChange={(e) => setKeyword(e.target.value)}
          placeholder="Enter keyword..."
          className="w-full p-3 border rounded"
        />
      </div>

      <button
        onClick={handleGenerate}
        disabled={loading || !keyword}
        className="bg-blue-600 text-white px-6 py-3 rounded disabled:opacity-50"
      >
        {loading ? 'Generating...' : 'Generate Content'}
      </button>

      {result && (
        <div className="mt-8">
          <h2 className="text-2xl font-bold mb-4">Generated Content</h2>
          <div className="prose max-w-none">
            {result.content}
          </div>
          
          {result.videoUrl && (
            <div className="mt-6">
              <h3 className="text-xl font-bold mb-2">Video</h3>
              <video src={result.videoUrl} controls className="w-full max-w-md" />
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
// scripts/batch-generate.ts
import { runContentPipeline } from '../services/pipeline/orchestrator';

const keywords = ['AI trends 2024', 'Marketing automation', 'Content strategy'];

async function batchGenerate() {
  for (const keyword of keywords) {
    console.log(`Processing: ${keyword}`);
    
    const result = await runContentPipeline({
      keyword,
      contentFormat: 'toplist',
      language: 'en',
      tone: 'expert',
      generateVideo: true
    });

    // Save to database or file
    console.log(`Completed: ${keyword}`);
    
    // Rate limiting
    await new Promise(resolve => setTimeout(resolve, 5000));
  }
}

batchGenerate();
```

### Custom Video Templates

```typescript
// remotion/compositions/CustomTemplate.tsx
import { AbsoluteFill, Sequence, useVideoConfig } from 'remotion';

export const CustomTemplate: React.FC<VideoProps> = ({ title, points }) => {
  const { fps } = useVideoConfig();
  
  return (
    <AbsoluteFill>
      <Sequence from={0} durationInFrames={fps * 2}>
        <TitleSlide title={title} />
      </Sequence>
      
      {points.map((point, i) => (
        <Sequence 
          key={i} 
          from={fps * (2 + i * 3)} 
          durationInFrames={fps * 3}
        >
          <PointSlide point={point} index={i + 1} />
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};
```

## Troubleshooting

### API Rate Limits

If you hit rate limits, implement queuing:

```typescript
// services/queue/rate-limiter.ts
class RateLimiter {
  private queue: Array<() => Promise<any>> = [];
  private processing = false;
  private delay = 1000; // ms between requests

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
      this.process();
    });
  }

  private async process() {
    if (this.processing || this.queue.length === 0) return;
    
    this.processing = true;
    const fn = this.queue.shift();
    
    if (fn) {
      await fn();
      await new Promise(resolve => setTimeout(resolve, this.delay));
    }
    
    this.processing = false;
    this.process();
  }
}

export const rateLimiter = new RateLimiter();
```

### Video Rendering Errors

Check Remotion configuration and AWS credentials:

```bash
# Test Remotion setup
npx remotion preview

# Check composition
npx remotion compositions
```

### Memory Issues with Large Content

Use streaming for large content generation:

```typescript
import { Anthropic } from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

async function streamContent(prompt: string) {
  const stream = await anthropic.messages.stream({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [{ role: 'user', content: prompt }],
  });

  let fullContent = '';
  
  for await (const chunk of stream) {
    if (chunk.type === 'content_block_delta' && 
        chunk.delta.type === 'text_delta') {
      fullContent += chunk.delta.text;
      // Process chunks as they arrive
    }
  }
  
  return fullContent;
}
```

## Performance Optimization

- Cache research data for repeated keywords
- Use database to store generated content
- Implement CDN for video distribution
- Use background jobs for video rendering
- Batch API requests where possible
