---
name: marketing-pipeline-automation
description: Automated content pipeline from research to video generation using AI (Claude, OpenAI) and Remotion
triggers:
  - automate content creation from research to video
  - set up AI content pipeline with Claude and OpenAI
  - generate videos from blog posts automatically
  - crawl trending topics and create content
  - build automated marketing content workflow
  - create social media videos from articles
  - implement content research automation
  - configure Remotion video rendering pipeline
---

# Marketing Pipeline Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the Ultimate AI Content Pipeline, a TypeScript-based system that automates the entire content creation workflow: from research and crawling trending topics, to generating multi-format content (blog posts, scripts), to rendering videos with Remotion. The pipeline integrates Claude 3, OpenAI, and various data sources (TechCrunch, Twitter, LinkedIn) to create data-backed content at scale.

## What This Project Does

The Marketing Pipeline is an end-to-end content automation system that:

- **Auto-crawls** trending topics from news sources and social media (24h rolling window)
- **Generates content** in multiple formats (Toplist, POV, Case Study, How-to) using Claude/OpenAI
- **Supports multi-language** output (English & Vietnamese) with customizable tone
- **Renders videos** automatically using Remotion for social media (Reels, TikTok, Shorts)
- **Provides a Next.js interface** for managing the entire pipeline

## Installation

### Prerequisites

```bash
node >= 18.x
npm or yarn
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

# Set up environment variables
cp .env.example .env
```

### Required Environment Variables

```bash
# .env file
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Video rendering
REMOTION_LICENSE_KEY=your_remotion_license

# Optional: Social media integration
TWITTER_API_KEY=your_twitter_api_key
LINKEDIN_API_KEY=your_linkedin_api_key
```

### Run Development Server

```bash
npm run dev
# or
yarn dev
```

The application will be available at `http://localhost:3000`

## Core Architecture

### Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── crawler/      # News crawling logic
│   │   ├── ai/           # Claude/OpenAI integrations
│   │   ├── video/        # Remotion video generation
│   │   └── utils/        # Utility functions
│   ├── types/            # TypeScript type definitions
│   └── config/           # Configuration files
├── public/               # Static assets
└── remotion/             # Remotion video templates
```

## Key Workflows

### 1. Content Research Pipeline

```typescript
// src/lib/crawler/researchPipeline.ts
import { crawlTechCrunch, crawlTwitter, crawlLinkedIn } from './sources';
import { analyzeInsights } from '../ai/analyzer';

export async function researchTopic(keyword: string) {
  // Crawl multiple sources
  const [techCrunchData, twitterData, linkedInData] = await Promise.all([
    crawlTechCrunch(keyword, { hours: 24 }),
    crawlTwitter(keyword, { hours: 24 }),
    crawlLinkedIn(keyword, { hours: 24 })
  ]);

  // Combine and deduplicate
  const allData = [...techCrunchData, ...twitterData, ...linkedInData];
  
  // Extract insights using AI
  const insights = await analyzeInsights(allData, {
    model: 'claude-3-opus',
    focusAreas: ['trends', 'statistics', 'expert-opinions']
  });

  return {
    rawData: allData,
    insights,
    sources: allData.map(d => d.source)
  };
}
```

### 2. Content Generation with AI

```typescript
// src/lib/ai/contentGenerator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY
});

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY
});

export interface ContentOptions {
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  wordCount?: number;
}

export async function generateContent(
  topic: string,
  insights: any[],
  options: ContentOptions
) {
  const prompt = buildPrompt(topic, insights, options);

  // Use Claude for long-form content
  if (options.wordCount && options.wordCount > 2000) {
    const response = await anthropic.messages.create({
      model: 'claude-3-opus-20240229',
      max_tokens: 4096,
      messages: [{
        role: 'user',
        content: prompt
      }]
    });

    return response.content[0].text;
  }

  // Use OpenAI for shorter content
  const response = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [{
      role: 'system',
      content: 'You are an expert content creator specializing in marketing.'
    }, {
      role: 'user',
      content: prompt
    }],
    temperature: 0.7
  });

  return response.choices[0].message.content;
}

function buildPrompt(topic: string, insights: any[], options: ContentOptions): string {
  const formatInstructions = {
    'toplist': 'Create a numbered list article with clear headings',
    'pov': 'Write from a unique perspective with strong opinions',
    'case-study': 'Analyze a real-world example with data',
    'how-to': 'Provide step-by-step actionable instructions'
  };

  return `
Topic: ${topic}

Format: ${options.format}
${formatInstructions[options.format]}

Language: ${options.language === 'vi' ? 'Vietnamese' : 'English'}
Tone: ${options.tone}

Research Insights:
${insights.map((i, idx) => `${idx + 1}. ${i.summary}`).join('\n')}

Requirements:
- Use data and statistics from the insights
- Include specific examples
- Write in ${options.tone} tone
- Target word count: ${options.wordCount || 1500} words
`;
}
```

### 3. Video Generation with Remotion

```typescript
// src/lib/video/videoGenerator.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

export interface VideoConfig {
  title: string;
  keyPoints: string[];
  backgroundMusic?: string;
  aspectRatio: '9:16' | '16:9' | '1:1'; // For different platforms
}

export async function generateVideo(
  content: string,
  config: VideoConfig
) {
  // Extract key points from content
  const keyPoints = config.keyPoints || await extractKeyPoints(content);

  // Bundle Remotion composition
  const bundleLocation = await bundle({
    entryPoint: path.join(process.cwd(), 'remotion/index.ts'),
    webpackOverride: (config) => config
  });

  // Select composition based on aspect ratio
  const compositionId = getCompositionId(config.aspectRatio);
  
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: {
      title: config.title,
      keyPoints,
      backgroundMusic: config.backgroundMusic
    }
  });

  // Render video
  const outputPath = path.join(process.cwd(), 'output', `${Date.now()}.mp4`);
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: {
      title: config.title,
      keyPoints,
      backgroundMusic: config.backgroundMusic
    }
  });

  return outputPath;
}

function getCompositionId(aspectRatio: string): string {
  const compositions = {
    '9:16': 'ReelsTemplate',
    '16:9': 'YouTubeTemplate',
    '1:1': 'InstagramTemplate'
  };
  return compositions[aspectRatio] || 'ReelsTemplate';
}

async function extractKeyPoints(content: string): Promise<string[]> {
  // Use AI to extract 3-5 key points
  const openai = new OpenAI({ apiKey: process.env.OPENAI_API_KEY });
  
  const response = await openai.chat.completions.create({
    model: 'gpt-4',
    messages: [{
      role: 'user',
      content: `Extract 3-5 key points from this content for a short video:\n\n${content}`
    }],
    temperature: 0.3
  });

  return response.choices[0].message.content
    .split('\n')
    .filter(line => line.trim())
    .slice(0, 5);
}
```

### 4. Remotion Video Template

```typescript
// remotion/ReelsTemplate.tsx
import React from 'react';
import { AbsoluteFill, useCurrentFrame, useVideoConfig, interpolate, Sequence } from 'remotion';

export interface ReelsProps {
  title: string;
  keyPoints: string[];
  backgroundMusic?: string;
}

export const ReelsTemplate: React.FC<ReelsProps> = ({ title, keyPoints, backgroundMusic }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  const titleOpacity = interpolate(frame, [0, 30], [0, 1], { extrapolateRight: 'clamp' });

  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a2e' }}>
      {/* Title Sequence */}
      <Sequence from={0} durationInFrames={fps * 2}>
        <AbsoluteFill style={{ justifyContent: 'center', alignItems: 'center' }}>
          <h1 
            style={{ 
              fontSize: 48,
              color: 'white',
              opacity: titleOpacity,
              textAlign: 'center',
              padding: 20
            }}
          >
            {title}
          </h1>
        </AbsoluteFill>
      </Sequence>

      {/* Key Points Sequences */}
      {keyPoints.map((point, idx) => (
        <Sequence 
          key={idx}
          from={fps * (2 + idx * 3)}
          durationInFrames={fps * 3}
        >
          <KeyPointSlide point={point} index={idx + 1} />
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};

const KeyPointSlide: React.FC<{ point: string; index: number }> = ({ point, index }) => {
  const frame = useCurrentFrame();
  const opacity = interpolate(frame, [0, 15, 60, 75], [0, 1, 1, 0]);
  const scale = interpolate(frame, [0, 15], [0.8, 1], { extrapolateRight: 'clamp' });

  return (
    <AbsoluteFill 
      style={{ 
        justifyContent: 'center', 
        alignItems: 'center',
        opacity,
        transform: `scale(${scale})`
      }}
    >
      <div style={{ maxWidth: '80%', textAlign: 'center' }}>
        <div style={{ fontSize: 72, color: '#00d4ff', marginBottom: 20 }}>
          {index}
        </div>
        <p style={{ fontSize: 32, color: 'white', lineHeight: 1.5 }}>
          {point}
        </p>
      </div>
    </AbsoluteFill>
  );
};
```

### 5. Complete Pipeline Orchestration

```typescript
// src/lib/pipeline/orchestrator.ts
import { researchTopic } from '../crawler/researchPipeline';
import { generateContent } from '../ai/contentGenerator';
import { generateVideo } from '../video/videoGenerator';

export interface PipelineConfig {
  keyword: string;
  contentFormat: 'toplist' | 'pov' | 'case-study' | 'how-to';
  languages: ('en' | 'vi')[];
  tone: 'expert' | 'friendly' | 'humorous';
  generateVideo: boolean;
  videoAspectRatio?: '9:16' | '16:9' | '1:1';
}

export async function runContentPipeline(config: PipelineConfig) {
  console.log(`Starting pipeline for keyword: ${config.keyword}`);

  // Step 1: Research
  console.log('Step 1: Researching topic...');
  const research = await researchTopic(config.keyword);
  
  // Step 2: Generate content for each language
  console.log('Step 2: Generating content...');
  const contents = await Promise.all(
    config.languages.map(lang => 
      generateContent(config.keyword, research.insights, {
        format: config.contentFormat,
        language: lang,
        tone: config.tone,
        wordCount: 2000
      })
    )
  );

  const results = {
    research,
    content: config.languages.reduce((acc, lang, idx) => {
      acc[lang] = contents[idx];
      return acc;
    }, {} as Record<string, string>),
    videos: {} as Record<string, string>
  };

  // Step 3: Generate videos if requested
  if (config.generateVideo) {
    console.log('Step 3: Generating videos...');
    
    for (let i = 0; i < config.languages.length; i++) {
      const lang = config.languages[i];
      const content = contents[i];
      
      const videoPath = await generateVideo(content, {
        title: config.keyword,
        keyPoints: [], // Will be extracted automatically
        aspectRatio: config.videoAspectRatio || '9:16'
      });
      
      results.videos[lang] = videoPath;
    }
  }

  console.log('Pipeline complete!');
  return results;
}
```

## API Routes (Next.js)

```typescript
// src/app/api/pipeline/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline/orchestrator';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    
    const result = await runContentPipeline({
      keyword: body.keyword,
      contentFormat: body.format || 'toplist',
      languages: body.languages || ['en'],
      tone: body.tone || 'expert',
      generateVideo: body.generateVideo || false,
      videoAspectRatio: body.videoAspectRatio || '9:16'
    });

    return NextResponse.json(result);
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { error: 'Pipeline execution failed' },
      { status: 500 }
    );
  }
}
```

## Client Usage Example

```typescript
// src/components/PipelineForm.tsx
'use client';

import { useState } from 'react';

export default function PipelineForm() {
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState(null);

  const handleSubmit = async (e: React.FormEvent<HTMLFormElement>) => {
    e.preventDefault();
    setLoading(true);

    const formData = new FormData(e.currentTarget);
    
    const response = await fetch('/api/pipeline', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        keyword: formData.get('keyword'),
        format: formData.get('format'),
        languages: ['en', 'vi'],
        tone: formData.get('tone'),
        generateVideo: formData.get('generateVideo') === 'on',
        videoAspectRatio: formData.get('aspectRatio')
      })
    });

    const data = await response.json();
    setResult(data);
    setLoading(false);
  };

  return (
    <form onSubmit={handleSubmit} className="space-y-4">
      <input
        name="keyword"
        placeholder="Enter keyword (e.g., AI Marketing Trends)"
        required
        className="w-full p-2 border rounded"
      />
      
      <select name="format" className="w-full p-2 border rounded">
        <option value="toplist">Top List</option>
        <option value="pov">Point of View</option>
        <option value="case-study">Case Study</option>
        <option value="how-to">How To</option>
      </select>

      <select name="tone" className="w-full p-2 border rounded">
        <option value="expert">Expert</option>
        <option value="friendly">Friendly</option>
        <option value="humorous">Humorous</option>
      </select>

      <div className="flex items-center gap-2">
        <input type="checkbox" name="generateVideo" id="generateVideo" />
        <label htmlFor="generateVideo">Generate Video</label>
      </div>

      <select name="aspectRatio" className="w-full p-2 border rounded">
        <option value="9:16">Reels/TikTok (9:16)</option>
        <option value="16:9">YouTube (16:9)</option>
        <option value="1:1">Instagram (1:1)</option>
      </select>

      <button
        type="submit"
        disabled={loading}
        className="w-full p-2 bg-blue-500 text-white rounded disabled:opacity-50"
      >
        {loading ? 'Processing...' : 'Generate Content'}
      </button>

      {result && (
        <div className="mt-4 p-4 bg-gray-100 rounded">
          <h3 className="font-bold">Results:</h3>
          <pre className="text-sm overflow-auto">
            {JSON.stringify(result, null, 2)}
          </pre>
        </div>
      )}
    </form>
  );
}
```

## Configuration

### Crawler Configuration

```typescript
// src/config/crawler.ts
export const crawlerConfig = {
  sources: {
    techcrunch: {
      enabled: true,
      url: 'https://techcrunch.com',
      rateLimit: 10 // requests per minute
    },
    twitter: {
      enabled: true,
      apiKey: process.env.TWITTER_API_KEY,
      maxTweets: 50
    },
    linkedin: {
      enabled: true,
      apiKey: process.env.LINKEDIN_API_KEY,
      maxPosts: 30
    }
  },
  timeWindow: 24, // hours
  cacheExpiry: 3600 // seconds
};
```

### AI Model Configuration

```typescript
// src/config/ai.ts
export const aiConfig = {
  claude: {
    model: 'claude-3-opus-20240229',
    maxTokens: 4096,
    temperature: 0.7,
    useFor: ['long-form', 'analysis']
  },
  openai: {
    model: 'gpt-4-turbo-preview',
    maxTokens: 2048,
    temperature: 0.7,
    useFor: ['short-form', 'extraction']
  }
};
```

## Common Patterns

### Rate Limiting and Retry Logic

```typescript
// src/lib/utils/rateLimiter.ts
export async function withRateLimit<T>(
  fn: () => Promise<T>,
  retries = 3,
  delay = 1000
): Promise<T> {
  for (let i = 0; i < retries; i++) {
    try {
      return await fn();
    } catch (error: any) {
      if (error.status === 429 && i < retries - 1) {
        await new Promise(resolve => setTimeout(resolve, delay * (i + 1)));
        continue;
      }
      throw error;
    }
  }
  throw new Error('Max retries reached');
}
```

### Content Caching

```typescript
// src/lib/utils/cache.ts
const cache = new Map<string, { data: any; timestamp: number }>();

export function getCached<T>(key: string, maxAge: number = 3600): T | null {
  const cached = cache.get(key);
  if (!cached) return null;
  
  if (Date.now() - cached.timestamp > maxAge * 1000) {
    cache.delete(key);
    return null;
  }
  
  return cached.data as T;
}

export function setCache(key: string, data: any): void {
  cache.set(key, { data, timestamp: Date.now() });
}
```

## Troubleshooting

### API Rate Limits

If you encounter rate limit errors:

```typescript
// Implement exponential backoff
import { withRateLimit } from '@/lib/utils/rateLimiter';

const result = await withRateLimit(
  () => anthropic.messages.create({...}),
  5, // retries
  2000 // initial delay
);
```

### Video Rendering Fails

Check Remotion setup:

```bash
# Ensure Remotion is properly installed
npm install @remotion/bundler @remotion/renderer @remotion/cli

# Test rendering locally
npx remotion preview remotion/index.ts
```

### Content Quality Issues

Adjust AI prompts for better results:

```typescript
// Add more specific instructions
const prompt = `
${basePrompt}

Additional requirements:
- Include at least 3 data points with sources
- Use active voice
- Add subheadings every 300 words
- Include a clear call-to-action
`;
```

### Memory Issues with Large Content

Stream responses for large content:

```typescript
const stream = await openai.chat.completions.create({
  model: 'gpt-4',
  messages: [...],
  stream: true
});

for await (const chunk of stream) {
  process.stdout.write(chunk.choices[0]?.delta?.content || '');
}
```

## Best Practices

1. **Always validate input** before sending to AI models
2. **Cache research results** to avoid redundant API calls
3. **Use environment-specific configs** for different deployment environments
4. **Monitor API costs** with usage tracking
5. **Implement proper error boundaries** in React components
6. **Test video templates** before bulk rendering
7. **Use TypeScript strict mode** for better type safety

This skill provides comprehensive coverage of the Marketing Pipeline automation project, enabling AI coding agents to effectively assist developers in implementing and customizing the content automation workflow.
