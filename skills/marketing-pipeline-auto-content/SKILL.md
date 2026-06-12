---
name: marketing-pipeline-auto-content
description: Automated AI content pipeline for research, scriptwriting, and video generation using Claude/OpenAI and Remotion
triggers:
  - how do I auto-generate content with AI research
  - set up automated marketing content pipeline
  - create content from keyword to video automatically
  - build AI content generation workflow
  - integrate Claude and OpenAI for content automation
  - generate videos from blog posts automatically
  - automate social media content creation pipeline
  - research and write content using AI crawlers
---

# Marketing Pipeline Auto Content

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the Ultimate AI Content Pipeline, a TypeScript-based automated content creation system that handles research, scriptwriting, multi-format content generation, and video rendering using Claude 3, OpenAI, and Remotion.

## What This Project Does

The Marketing Pipeline is an end-to-end content automation system that:

- **Auto-scans research** from TechCrunch, a16z, Twitter/X, LinkedIn for fresh insights
- **Generates multi-format content** (listicles, POV pieces, case studies, how-tos) in multiple languages
- **Renders videos and infographics** automatically using Remotion
- **Optimizes for multiple platforms** (Reels, TikTok, Shorts)

The workflow: Keyword → Research → Script → Content → Video → Ready to publish

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

### Environment Setup

Create a `.env.local` file in the root directory:

```bash
# AI Provider Keys
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# Research API Keys
RAPIDAPI_KEY=your_rapidapi_key

# Optional: Video Rendering
REMOTION_LICENSE_KEY=your_remotion_key

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Application Config
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app router pages
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration modules
│   │   ├── crawlers/    # Research crawlers
│   │   ├── generators/  # Content generators
│   │   └── video/       # Remotion video rendering
│   ├── types/           # TypeScript types
│   └── utils/           # Utility functions
├── remotion/            # Video templates
└── public/              # Static assets
```

## Core Modules & Usage

### 1. Research Crawler Module

```typescript
// src/lib/crawlers/research-crawler.ts
import { crawlTechCrunch, crawlTwitter, crawlLinkedIn } from '@/lib/crawlers';

interface ResearchData {
  title: string;
  url: string;
  snippet: string;
  publishedAt: Date;
  source: string;
}

export async function performResearch(keyword: string): Promise<ResearchData[]> {
  const rapidApiKey = process.env.RAPIDAPI_KEY;
  
  const [techNews, socialData, linkedInPosts] = await Promise.all([
    crawlTechCrunch(keyword, rapidApiKey),
    crawlTwitter(keyword, rapidApiKey),
    crawlLinkedIn(keyword, rapidApiKey)
  ]);

  return [...techNews, ...socialData, ...linkedInPosts]
    .filter(item => isRecent(item.publishedAt, 24)) // Last 24 hours
    .sort((a, b) => b.publishedAt.getTime() - a.publishedAt.getTime());
}

// Usage in your code
const research = await performResearch('AI automation');
console.log(`Found ${research.length} recent articles`);
```

### 2. AI Content Generation

```typescript
// src/lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

interface ContentConfig {
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  tone: 'professional' | 'friendly' | 'humorous';
  language: 'en' | 'vi';
  researchData: ResearchData[];
}

export async function generateContent(
  keyword: string,
  config: ContentConfig
): Promise<string> {
  const prompt = buildPrompt(keyword, config);

  // Use Claude for long-form content
  if (config.format === 'case-study') {
    const response = await anthropic.messages.create({
      model: 'claude-3-5-sonnet-20241022',
      max_tokens: 4096,
      messages: [{
        role: 'user',
        content: prompt
      }]
    });

    return response.content[0].type === 'text' 
      ? response.content[0].text 
      : '';
  }

  // Use OpenAI for shorter formats
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [{ role: 'user', content: prompt }],
    temperature: 0.7,
  });

  return completion.choices[0].message.content || '';
}

function buildPrompt(keyword: string, config: ContentConfig): string {
  const { format, tone, language, researchData } = config;
  
  const context = researchData
    .map(r => `- ${r.title} (${r.source}): ${r.snippet}`)
    .join('\n');

  return `Write a ${format} article about "${keyword}" in ${language === 'vi' ? 'Vietnamese' : 'English'}.
Tone: ${tone}
Use these recent insights:
${context}

Requirements:
- Data-backed with specific examples
- Engaging and actionable
- SEO-optimized
- ${format === 'toplist' ? 'Include numbered points' : ''}
- ${format === 'how-to' ? 'Step-by-step instructions' : ''}`;
}
```

### 3. Video Generation with Remotion

```typescript
// src/lib/video/video-renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import { WebpackOverrideFn } from '@remotion/bundler';
import path from 'path';

interface VideoConfig {
  content: string;
  format: 'reels' | 'tiktok' | 'shorts';
  title: string;
}

const DIMENSIONS = {
  reels: { width: 1080, height: 1920 },
  tiktok: { width: 1080, height: 1920 },
  shorts: { width: 1080, height: 1920 },
};

export async function renderVideo(config: VideoConfig): Promise<string> {
  const { content, format, title } = config;
  const { width, height } = DIMENSIONS[format];

  // Bundle Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.join(process.cwd(), 'remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  // Get composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      content,
      title,
    },
  });

  // Render video
  const outputPath = path.join(
    process.cwd(),
    'public',
    'videos',
    `${Date.now()}-${format}.mp4`
  );

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: {
      content,
      title,
    },
  });

  return outputPath;
}
```

### 4. Complete Pipeline Workflow

```typescript
// src/lib/pipeline/content-pipeline.ts
import { performResearch } from '@/lib/crawlers/research-crawler';
import { generateContent } from '@/lib/ai/content-generator';
import { renderVideo } from '@/lib/video/video-renderer';

export async function runContentPipeline(keyword: string) {
  try {
    // Step 1: Research
    console.log('🔍 Starting research phase...');
    const research = await performResearch(keyword);
    
    if (research.length === 0) {
      throw new Error('No recent content found for this keyword');
    }

    // Step 2: Generate Content (English & Vietnamese)
    console.log('✍️ Generating content...');
    const [englishContent, vietnameseContent] = await Promise.all([
      generateContent(keyword, {
        format: 'toplist',
        tone: 'professional',
        language: 'en',
        researchData: research,
      }),
      generateContent(keyword, {
        format: 'toplist',
        tone: 'professional',
        language: 'vi',
        researchData: research,
      }),
    ]);

    // Step 3: Render Videos
    console.log('🎬 Rendering videos...');
    const videoPromises = ['reels', 'tiktok', 'shorts'].map((format) =>
      renderVideo({
        content: englishContent,
        format: format as 'reels' | 'tiktok' | 'shorts',
        title: keyword,
      })
    );

    const videoPaths = await Promise.all(videoPromises);

    return {
      research,
      content: {
        en: englishContent,
        vi: vietnameseContent,
      },
      videos: videoPaths,
    };
  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}
```

## API Routes (Next.js)

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

    return NextResponse.json({
      success: true,
      data: result,
    });
  } catch (error) {
    console.error('API Error:', error);
    return NextResponse.json(
      { error: 'Failed to generate content' },
      { status: 500 }
    );
  }
}
```

## Frontend Integration

```typescript
// src/components/ContentGenerator.tsx
'use client';

import { useState } from 'react';

export default function ContentGenerator() {
  const [keyword, setKeyword] = useState('');
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState(null);

  const handleGenerate = async () => {
    setLoading(true);
    try {
      const response = await fetch('/api/generate', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ keyword }),
      });

      const data = await response.json();
      setResult(data.data);
    } catch (error) {
      console.error('Generation failed:', error);
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="max-w-4xl mx-auto p-6">
      <h1 className="text-3xl font-bold mb-6">AI Content Pipeline</h1>
      
      <div className="space-y-4">
        <input
          type="text"
          value={keyword}
          onChange={(e) => setKeyword(e.target.value)}
          placeholder="Enter keyword..."
          className="w-full px-4 py-2 border rounded-lg"
        />
        
        <button
          onClick={handleGenerate}
          disabled={loading || !keyword}
          className="px-6 py-2 bg-blue-600 text-white rounded-lg disabled:opacity-50"
        >
          {loading ? 'Generating...' : 'Generate Content'}
        </button>

        {result && (
          <div className="mt-6 space-y-4">
            <h2 className="text-xl font-semibold">Results:</h2>
            <div className="bg-gray-50 p-4 rounded-lg">
              <pre className="whitespace-pre-wrap">
                {JSON.stringify(result, null, 2)}
              </pre>
            </div>
          </div>
        )}
      </div>
    </div>
  );
}
```

## Running the Application

```bash
# Development mode
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Render specific video template
npm run remotion:render
```

## Common Patterns

### Batch Content Generation

```typescript
const keywords = ['AI automation', 'Marketing tools', 'Content creation'];

const results = await Promise.allSettled(
  keywords.map(keyword => runContentPipeline(keyword))
);

results.forEach((result, index) => {
  if (result.status === 'fulfilled') {
    console.log(`✅ ${keywords[index]}: Success`);
  } else {
    console.error(`❌ ${keywords[index]}: ${result.reason}`);
  }
});
```

### Custom Content Formats

```typescript
const customFormats = {
  'product-review': {
    format: 'pov',
    tone: 'friendly',
    additionalPrompt: 'Focus on pros, cons, and verdict',
  },
  'trend-analysis': {
    format: 'case-study',
    tone: 'professional',
    additionalPrompt: 'Include market data and predictions',
  },
};
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement exponential backoff
async function retryWithBackoff<T>(
  fn: () => Promise<T>,
  maxRetries = 3
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error: any) {
      if (error.status === 429 && i < maxRetries - 1) {
        await new Promise(resolve => setTimeout(resolve, 2 ** i * 1000));
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
// Increase Node memory limit
// package.json
{
  "scripts": {
    "dev": "NODE_OPTIONS='--max-old-space-size=4096' next dev",
    "build": "NODE_OPTIONS='--max-old-space-size=4096' next build"
  }
}
```

### Missing Environment Variables

```typescript
// src/lib/config/validate-env.ts
export function validateEnv() {
  const required = [
    'OPENAI_API_KEY',
    'ANTHROPIC_API_KEY',
    'RAPIDAPI_KEY',
  ];

  const missing = required.filter(key => !process.env[key]);

  if (missing.length > 0) {
    throw new Error(
      `Missing required environment variables: ${missing.join(', ')}`
    );
  }
}

// Call at app startup
validateEnv();
```

This skill provides comprehensive knowledge for AI agents to help developers implement and extend the Marketing Pipeline Auto Content system.
