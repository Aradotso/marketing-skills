---
name: ultimate-ai-content-pipeline
description: Automated content pipeline from research to video generation using Claude/OpenAI and Remotion
triggers:
  - how do i automate content creation with AI
  - generate social media videos from articles automatically
  - crawl news sources and create content
  - build an AI content pipeline with video rendering
  - automate marketing content research and writing
  - create multi-language content with Claude API
  - generate reels and shorts from text automatically
  - set up automated content workflow with Remotion
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

Ultimate AI Content Pipeline is a comprehensive TypeScript-based automation system that handles the entire content creation workflow: from crawling news sources for research, to generating written content in multiple formats and languages using Claude/OpenAI, to rendering videos automatically with Remotion. Perfect for content creators and marketers looking to scale their output.

## What This Project Does

1. **Auto-Scan Research**: Crawls news sources (TechCrunch, Twitter, LinkedIn) for real-time data within 24 hours
2. **AI Content Generation**: Creates diverse content formats (toplist, POV, case studies, how-to) in multiple languages using Claude 3 and OpenAI
3. **Video Rendering**: Automatically generates infographics and short videos from written content using Remotion
4. **Multi-Platform Optimization**: Exports videos in formats optimized for Reels, TikTok, and YouTube Shorts

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

Create a `.env.local` file in the project root:

```env
# AI Provider APIs
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research/Crawling APIs
RAPIDAPI_KEY=your_rapidapi_key

# Remotion Configuration
REMOTION_AWS_ACCESS_KEY_ID=your_aws_access_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret_key

# Next.js Configuration
NEXT_PUBLIC_API_URL=http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app router pages
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── crawlers/    # News crawling modules
│   │   ├── generators/  # Content generation logic
│   │   └── video/       # Remotion video rendering
│   └── utils/           # Utility functions
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Core APIs and Usage

### 1. Content Research (Crawling)

```typescript
// src/lib/crawlers/news-crawler.ts
import axios from 'axios';

interface NewsSource {
  title: string;
  url: string;
  publishedAt: string;
  content: string;
}

export async function crawlTechCrunch(keyword: string): Promise<NewsSource[]> {
  const options = {
    method: 'GET',
    url: 'https://techcrunch-api.p.rapidapi.com/articles',
    params: { query: keyword, limit: '10' },
    headers: {
      'X-RapidAPI-Key': process.env.RAPIDAPI_KEY,
      'X-RapidAPI-Host': 'techcrunch-api.p.rapidapi.com'
    }
  };

  try {
    const response = await axios.request(options);
    return response.data.articles.map((article: any) => ({
      title: article.title,
      url: article.url,
      publishedAt: article.publishedAt,
      content: article.content
    }));
  } catch (error) {
    console.error('Error crawling TechCrunch:', error);
    return [];
  }
}

export async function aggregateResearch(keyword: string) {
  const sources = await Promise.all([
    crawlTechCrunch(keyword),
    // Add other sources as needed
  ]);
  
  return sources.flat();
}
```

### 2. AI Content Generation with Claude

```typescript
// src/lib/ai/claude-generator.ts
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

interface ContentRequest {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  research: string[];
}

export async function generateContent(request: ContentRequest): Promise<string> {
  const prompt = buildPrompt(request);
  
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

  return message.content[0].type === 'text' ? message.content[0].text : '';
}

function buildPrompt(request: ContentRequest): string {
  const formatInstructions = {
    'toplist': 'Create a numbered list article with rankings and explanations',
    'pov': 'Write from a specific perspective with strong opinions',
    'case-study': 'Analyze a real example with data and insights',
    'how-to': 'Create a step-by-step tutorial'
  };

  const toneInstructions = {
    'expert': 'Use professional language with industry terminology',
    'friendly': 'Write conversationally and approachably',
    'humorous': 'Include light humor while maintaining value'
  };

  return `
You are a content marketing expert. Create a ${request.format} article about "${request.keyword}" in ${request.language === 'vi' ? 'Vietnamese' : 'English'}.

Format: ${formatInstructions[request.format]}
Tone: ${toneInstructions[request.tone]}

Use the following research data:
${request.research.join('\n\n')}

Requirements:
- Include specific data points and statistics
- Make it actionable and valuable
- Optimize for social media engagement
- Length: 800-1200 words
`;
}
```

### 3. OpenAI Alternative

```typescript
// src/lib/ai/openai-generator.ts
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

export async function generateWithOpenAI(
  keyword: string,
  research: string[]
): Promise<string> {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: 'You are an expert content marketer who creates engaging, data-driven content.'
      },
      {
        role: 'user',
        content: `Create an article about ${keyword} using this research:\n${research.join('\n\n')}`
      }
    ],
    temperature: 0.7,
    max_tokens: 3000
  });

  return completion.choices[0].message.content || '';
}
```

### 4. Video Generation with Remotion

```typescript
// src/lib/video/video-renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  title: string;
  keyPoints: string[];
  duration: number;
  format: 'reels' | 'tiktok' | 'shorts';
}

const formatSpecs = {
  reels: { width: 1080, height: 1920, fps: 30 },
  tiktok: { width: 1080, height: 1920, fps: 30 },
  shorts: { width: 1080, height: 1920, fps: 30 }
};

export async function renderContentVideo(config: VideoConfig): Promise<string> {
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
      keyPoints: config.keyPoints,
    },
  });

  const specs = formatSpecs[config.format];
  const outputLocation = `out/${config.format}-${Date.now()}.mp4`;

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: {
      title: config.title,
      keyPoints: config.keyPoints,
    },
    ...specs,
  });

  return outputLocation;
}
```

### 5. Remotion Video Component

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, interpolate, Sequence } from 'remotion';
import React from 'react';

interface ContentVideoProps {
  title: string;
  keyPoints: string[];
}

export const ContentVideo: React.FC<ContentVideoProps> = ({ title, keyPoints }) => {
  const frame = useCurrentFrame();

  const titleOpacity = interpolate(frame, [0, 30], [0, 1], {
    extrapolateRight: 'clamp',
  });

  return (
    <AbsoluteFill style={{ backgroundColor: '#000' }}>
      <Sequence from={0} durationInFrames={90}>
        <AbsoluteFill
          style={{
            justifyContent: 'center',
            alignItems: 'center',
            opacity: titleOpacity,
          }}
        >
          <h1 style={{ color: 'white', fontSize: 60, textAlign: 'center', padding: 40 }}>
            {title}
          </h1>
        </AbsoluteFill>
      </Sequence>

      {keyPoints.map((point, index) => (
        <Sequence key={index} from={90 + index * 90} durationInFrames={90}>
          <KeyPointSlide point={point} index={index + 1} />
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};

const KeyPointSlide: React.FC<{ point: string; index: number }> = ({ point, index }) => {
  const frame = useCurrentFrame();
  const opacity = interpolate(frame, [0, 20], [0, 1]);

  return (
    <AbsoluteFill
      style={{
        justifyContent: 'center',
        alignItems: 'center',
        opacity,
        padding: 40,
      }}
    >
      <div style={{ color: '#00ff00', fontSize: 40, marginBottom: 20 }}>
        #{index}
      </div>
      <div style={{ color: 'white', fontSize: 36, textAlign: 'center' }}>
        {point}
      </div>
    </AbsoluteFill>
  );
};
```

## Complete Workflow Example

```typescript
// src/lib/pipeline/content-pipeline.ts
import { aggregateResearch } from '../crawlers/news-crawler';
import { generateContent } from '../ai/claude-generator';
import { renderContentVideo } from '../video/video-renderer';

export async function runContentPipeline(keyword: string) {
  try {
    // Step 1: Research
    console.log('Starting research phase...');
    const research = await aggregateResearch(keyword);
    const researchSummaries = research.map(r => `${r.title}: ${r.content}`);

    // Step 2: Generate Content
    console.log('Generating content...');
    const content = await generateContent({
      keyword,
      format: 'toplist',
      language: 'en',
      tone: 'expert',
      research: researchSummaries
    });

    // Extract key points for video
    const keyPoints = extractKeyPoints(content);

    // Step 3: Generate Video
    console.log('Rendering video...');
    const videoPath = await renderContentVideo({
      title: keyword,
      keyPoints,
      duration: 60,
      format: 'reels'
    });

    return {
      content,
      videoPath,
      research: research.length
    };
  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}

function extractKeyPoints(content: string): string[] {
  // Simple extraction - improve based on your content structure
  const lines = content.split('\n');
  return lines
    .filter(line => /^\d+\./.test(line.trim()))
    .slice(0, 5)
    .map(line => line.replace(/^\d+\.\s*/, ''));
}
```

## Next.js API Routes

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline/content-pipeline';

export async function POST(request: NextRequest) {
  try {
    const { keyword } = await request.json();

    if (!keyword) {
      return NextResponse.json(
        { error: 'Keyword is required' },
        { status: 400 }
      );
    }

    const result = await runContentPipeline(keyword);

    return NextResponse.json(result);
  } catch (error) {
    console.error('Generation error:', error);
    return NextResponse.json(
      { error: 'Failed to generate content' },
      { status: 500 }
    );
  }
}
```

## Frontend Component Example

```typescript
// src/components/ContentGenerator.tsx
'use client';

import { useState } from 'react';

export function ContentGenerator() {
  const [keyword, setKeyword] = useState('');
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState<any>(null);

  const handleGenerate = async () => {
    setLoading(true);
    try {
      const response = await fetch('/api/generate', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ keyword })
      });

      const data = await response.json();
      setResult(data);
    } catch (error) {
      console.error('Error:', error);
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="p-6">
      <input
        type="text"
        value={keyword}
        onChange={(e) => setKeyword(e.target.value)}
        placeholder="Enter keyword..."
        className="border p-2 rounded w-full mb-4"
      />
      <button
        onClick={handleGenerate}
        disabled={loading}
        className="bg-blue-500 text-white px-4 py-2 rounded"
      >
        {loading ? 'Generating...' : 'Generate Content'}
      </button>

      {result && (
        <div className="mt-4">
          <h3 className="font-bold">Generated Content:</h3>
          <pre className="whitespace-pre-wrap">{result.content}</pre>
          <p>Video: {result.videoPath}</p>
          <p>Sources researched: {result.research}</p>
        </div>
      )}
    </div>
  );
}
```

## Running the Development Server

```bash
# Start Next.js development server
npm run dev

# Access at http://localhost:3000
```

## Common Patterns

### Multi-Language Support

```typescript
export async function generateBilingualContent(keyword: string, research: string[]) {
  const [englishContent, vietnameseContent] = await Promise.all([
    generateContent({
      keyword,
      format: 'toplist',
      language: 'en',
      tone: 'expert',
      research
    }),
    generateContent({
      keyword,
      format: 'toplist',
      language: 'vi',
      tone: 'friendly',
      research
    })
  ]);

  return { englishContent, vietnameseContent };
}
```

### Batch Processing

```typescript
export async function processBatchKeywords(keywords: string[]) {
  const results = await Promise.allSettled(
    keywords.map(keyword => runContentPipeline(keyword))
  );

  return results.map((result, index) => ({
    keyword: keywords[index],
    status: result.status,
    data: result.status === 'fulfilled' ? result.value : null,
    error: result.status === 'rejected' ? result.reason : null
  }));
}
```

## Troubleshooting

### API Rate Limits

If you hit rate limits, implement exponential backoff:

```typescript
async function withRetry<T>(
  fn: () => Promise<T>,
  maxRetries = 3
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error: any) {
      if (i === maxRetries - 1) throw error;
      if (error.status === 429) {
        await new Promise(resolve => setTimeout(resolve, Math.pow(2, i) * 1000));
      } else {
        throw error;
      }
    }
  }
  throw new Error('Max retries exceeded');
}
```

### Video Rendering Memory Issues

For large videos, increase Node.js memory:

```json
// package.json
{
  "scripts": {
    "dev": "NODE_OPTIONS='--max-old-space-size=4096' next dev",
    "build": "NODE_OPTIONS='--max-old-space-size=4096' next build"
  }
}
```

### Missing Environment Variables

Always validate environment variables at startup:

```typescript
// src/lib/config.ts
const requiredEnvVars = [
  'ANTHROPIC_API_KEY',
  'OPENAI_API_KEY',
  'RAPIDAPI_KEY'
];

export function validateEnv() {
  const missing = requiredEnvVars.filter(key => !process.env[key]);
  if (missing.length > 0) {
    throw new Error(`Missing required environment variables: ${missing.join(', ')}`);
  }
}
```

This system enables fully automated content creation from research to video, perfect for scaling marketing operations.
