---
name: marketing-pipeline-share-automation
description: Automated AI content pipeline from research to video generation using Claude, OpenAI, and Remotion
triggers:
  - how do I automate content creation with AI
  - set up marketing content pipeline
  - generate videos from text automatically
  - create automated content research workflow
  - use Claude and OpenAI for content generation
  - build content automation with Remotion
  - automate social media video creation
  - integrate AI content pipeline in TypeScript
---

# Marketing Pipeline Share Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the Ultimate AI Content Pipeline, a comprehensive TypeScript-based system that automates content creation from research to video generation. The pipeline integrates Claude 3, OpenAI, and Remotion to crawl news sources, generate content in multiple formats, and render videos automatically.

## What This Project Does

The Marketing Pipeline Share is an all-in-one content automation system that:

- **Auto-crawls** real-time news from TechCrunch, a16z, Twitter/X, LinkedIn (last 24h)
- **Generates content** in multiple formats (toplist, POV, case study, how-to) using Claude/OpenAI
- **Supports bilingual** output (English and Vietnamese) with customizable tone
- **Renders videos** automatically using Remotion for social media (Reels, TikTok, Shorts)
- **Provides Next.js interface** for easy content scheduling and management

## Installation

### Prerequisites

```bash
# Node.js 18+ required
node --version

# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share
```

### Install Dependencies

```bash
# Install all packages
npm install
# or
yarn install
# or
pnpm install
```

### Environment Configuration

Create a `.env.local` file in the root directory:

```bash
# AI API Keys
ANTHROPIC_API_KEY=your_claude_api_key_here
OPENAI_API_KEY=your_openai_api_key_here

# RapidAPI for news crawling
RAPIDAPI_KEY=your_rapidapi_key_here

# Remotion configuration
REMOTION_LICENSE_KEY=your_remotion_license_key_here

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Next.js configuration
NEXT_PUBLIC_API_URL=http://localhost:3000
```

### Start Development Server

```bash
# Run the Next.js development server
npm run dev

# The app will be available at http://localhost:3000
```

## Key Commands

### Development

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Run linter
npm run lint

# Type checking
npm run type-check
```

### Remotion Video Rendering

```bash
# Preview Remotion compositions
npm run remotion:preview

# Render a specific composition
npm run remotion:render -- <composition-id> output.mp4

# Render with custom props
npm run remotion:render -- <composition-id> output.mp4 --props='{"title":"My Video"}'
```

## Core API Usage

### 1. Content Research & Crawling

```typescript
import { crawlNews } from '@/lib/research/crawler';
import { analyzeContent } from '@/lib/research/analyzer';

// Crawl news from multiple sources
async function researchTopic(keyword: string) {
  const sources = ['techcrunch', 'a16z', 'twitter', 'linkedin'];
  
  const crawlResults = await Promise.all(
    sources.map(source => 
      crawlNews({
        source,
        keyword,
        timeRange: '24h',
        limit: 10
      })
    )
  );

  // Analyze and extract insights
  const insights = await analyzeContent({
    articles: crawlResults.flat(),
    analysisType: 'deep',
    extractDataPoints: true
  });

  return insights;
}
```

### 2. AI Content Generation with Claude

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

async function generateContent(
  topic: string,
  format: 'toplist' | 'pov' | 'case-study' | 'how-to',
  language: 'en' | 'vi',
  tone: 'expert' | 'friendly' | 'humorous'
) {
  const prompt = `Create a ${format} article about ${topic} in ${language} with a ${tone} tone.
  
Include:
- Engaging headline
- Introduction with hook
- Main content with data points
- Call to action
- SEO metadata`;

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [{
      role: 'user',
      content: prompt
    }]
  });

  return message.content[0].text;
}
```

### 3. OpenAI Integration for Alternative Generation

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateVariation(content: string, style: string) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are a content rewriter. Rewrite in ${style} style.`
      },
      {
        role: 'user',
        content: content
      }
    ],
    temperature: 0.7,
  });

  return completion.choices[0].message.content;
}
```

### 4. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

async function generateVideo(contentData: {
  title: string;
  points: string[];
  bgColor: string;
}) {
  const compositionId = 'ContentVideo';
  const bundleLocation = await bundle(
    path.join(process.cwd(), 'src/remotion/index.ts')
  );

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
  });

  const outputLocation = path.join(
    process.cwd(), 
    'public/videos', 
    `${Date.now()}.mp4`
  );

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: contentData,
  });

  return outputLocation;
}
```

## Common Patterns

### Full Content Pipeline

```typescript
import { researchTopic } from '@/lib/research';
import { generateContent } from '@/lib/ai/claude';
import { generateVideo } from '@/lib/video/remotion';
import { publishToScheduler } from '@/lib/scheduler';

async function runContentPipeline(keyword: string) {
  try {
    // Step 1: Research
    console.log('📡 Researching topic...');
    const research = await researchTopic(keyword);

    // Step 2: Generate content (both languages)
    console.log('🧠 Generating content...');
    const [contentEN, contentVI] = await Promise.all([
      generateContent(keyword, 'toplist', 'en', 'expert'),
      generateContent(keyword, 'toplist', 'vi', 'friendly')
    ]);

    // Step 3: Create video
    console.log('🎬 Rendering video...');
    const videoPath = await generateVideo({
      title: research.mainTitle,
      points: research.keyPoints,
      bgColor: '#1a1a2e'
    });

    // Step 4: Schedule for publishing
    console.log('📅 Scheduling post...');
    await publishToScheduler({
      contentEN,
      contentVI,
      videoPath,
      scheduledTime: new Date(Date.now() + 3600000) // 1 hour from now
    });

    return {
      success: true,
      research,
      contentEN,
      contentVI,
      videoPath
    };
  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}
```

### API Route Example (Next.js)

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline';

export async function POST(request: NextRequest) {
  try {
    const { keyword, format, language } = await request.json();

    if (!keyword) {
      return NextResponse.json(
        { error: 'Keyword is required' },
        { status: 400 }
      );
    }

    const result = await runContentPipeline(keyword);

    return NextResponse.json({
      success: true,
      data: result
    });
  } catch (error) {
    return NextResponse.json(
      { error: error.message },
      { status: 500 }
    );
  }
}
```

### Client-Side Component

```typescript
// components/ContentGenerator.tsx
'use client';

import { useState } from 'react';

export function ContentGenerator() {
  const [keyword, setKeyword] = useState('');
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState(null);

  const handleGenerate = async () => {
    setLoading(true);
    try {
      const response = await fetch('/api/generate', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ 
          keyword,
          format: 'toplist',
          language: 'en'
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
    <div className="generator-container">
      <input
        type="text"
        value={keyword}
        onChange={(e) => setKeyword(e.target.value)}
        placeholder="Enter keyword..."
        className="keyword-input"
      />
      <button onClick={handleGenerate} disabled={loading}>
        {loading ? 'Generating...' : 'Generate Content'}
      </button>
      {result && (
        <div className="result">
          <pre>{JSON.stringify(result, null, 2)}</pre>
        </div>
      )}
    </div>
  );
}
```

## Configuration

### Remotion Composition

```typescript
// src/remotion/compositions/ContentVideo.tsx
import { AbsoluteFill, Sequence, useCurrentFrame } from 'remotion';

export const ContentVideo: React.FC<{
  title: string;
  points: string[];
  bgColor: string;
}> = ({ title, points, bgColor }) => {
  const frame = useCurrentFrame();

  return (
    <AbsoluteFill style={{ backgroundColor: bgColor }}>
      <Sequence from={0} durationInFrames={60}>
        <div style={{ 
          fontSize: 60, 
          textAlign: 'center',
          color: 'white',
          padding: 40
        }}>
          {title}
        </div>
      </Sequence>
      {points.map((point, index) => (
        <Sequence 
          key={index}
          from={60 + index * 90}
          durationInFrames={90}
        >
          <div style={{
            fontSize: 40,
            color: 'white',
            padding: 60
          }}>
            {point}
          </div>
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};
```

### Custom AI Prompts Configuration

```typescript
// config/prompts.ts
export const CONTENT_PROMPTS = {
  toplist: {
    system: 'You are an expert content creator specializing in listicles.',
    template: `Create a top 10 list about {topic}.
    Include:
    - Catchy headline
    - Brief intro
    - 10 items with descriptions
    - Data points where applicable
    - Conclusion with CTA`
  },
  pov: {
    system: 'You are a thought leader sharing unique perspectives.',
    template: `Write a POV article about {topic}.
    Express strong opinions backed by research.
    Use first-person narrative.
    Include personal anecdotes if relevant.`
  },
  caseStudy: {
    system: 'You are a business analyst writing case studies.',
    template: `Write a case study about {topic}.
    Structure: Problem, Solution, Results, Lessons.
    Include specific metrics and outcomes.`
  }
};
```

## Troubleshooting

### API Rate Limits

```typescript
// lib/utils/rateLimit.ts
class RateLimiter {
  private requests: number[] = [];
  private maxRequests: number;
  private timeWindow: number;

  constructor(maxRequests: number, timeWindowMs: number) {
    this.maxRequests = maxRequests;
    this.timeWindow = timeWindowMs;
  }

  async waitIfNeeded() {
    const now = Date.now();
    this.requests = this.requests.filter(
      time => now - time < this.timeWindow
    );

    if (this.requests.length >= this.maxRequests) {
      const oldestRequest = this.requests[0];
      const waitTime = this.timeWindow - (now - oldestRequest);
      await new Promise(resolve => setTimeout(resolve, waitTime));
    }

    this.requests.push(now);
  }
}

export const claudeLimiter = new RateLimiter(50, 60000); // 50 req/min
export const openaiLimiter = new RateLimiter(60, 60000); // 60 req/min
```

### Error Handling

```typescript
// lib/utils/errorHandler.ts
export class PipelineError extends Error {
  constructor(
    message: string,
    public stage: 'research' | 'generation' | 'video' | 'publish',
    public originalError?: Error
  ) {
    super(message);
    this.name = 'PipelineError';
  }
}

export async function withRetry<T>(
  fn: () => Promise<T>,
  maxRetries = 3,
  delay = 1000
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      await new Promise(resolve => setTimeout(resolve, delay * (i + 1)));
    }
  }
  throw new Error('Max retries reached');
}
```

### Video Rendering Issues

```typescript
// Check Remotion configuration
import { getCompositions } from '@remotion/renderer';

async function debugRemotionSetup() {
  try {
    const compositions = await getCompositions(bundleLocation);
    console.log('Available compositions:', compositions);
    
    // Verify composition exists
    const targetComp = compositions.find(c => c.id === 'ContentVideo');
    if (!targetComp) {
      throw new Error('Composition not found');
    }
    
    console.log('Composition config:', targetComp);
  } catch (error) {
    console.error('Remotion setup error:', error);
  }
}
```

This skill provides comprehensive guidance for working with the Marketing Pipeline Share automation system, covering all major features from content research to video generation.
