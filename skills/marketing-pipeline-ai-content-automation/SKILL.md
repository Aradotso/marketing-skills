---
name: marketing-pipeline-ai-content-automation
description: Automated AI content pipeline from research to video generation using Claude, OpenAI, and Remotion
triggers:
  - "set up automated content pipeline"
  - "create AI-driven content marketing system"
  - "automate content research and video generation"
  - "build content automation with Claude and Remotion"
  - "generate content from keywords automatically"
  - "create multi-format content with AI"
  - "automate marketing content pipeline"
  - "set up AI content generation workflow"
---

# Marketing Pipeline AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the Ultimate AI Content Pipeline - an automated content creation system that handles research, scriptwriting, and video generation using Claude 3, OpenAI, and Remotion.

## What This Project Does

The marketing-pipeline-share project is a comprehensive TypeScript-based content automation system that:

- **Auto-crawls research data** from sources like TechCrunch, a16z, Twitter/X, LinkedIn for real-time insights
- **Generates multi-format content** (toplists, POV articles, case studies, how-tos) using Claude/OpenAI
- **Produces bilingual content** (English & Vietnamese) with customizable voice tones
- **Renders videos and infographics** automatically using Remotion
- **Optimizes for multiple platforms** (Reels, TikTok, Shorts)

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

```env
# AI Services
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Remotion (for video generation)
REMOTION_LICENSE_KEY=your_remotion_license

# Next.js
NEXT_PUBLIC_API_URL=http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app routes
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integrations (Claude, OpenAI)
│   │   ├── research/    # Content research crawlers
│   │   ├── content/     # Content generation logic
│   │   └── video/       # Remotion video rendering
│   ├── types/           # TypeScript types
│   └── utils/           # Utility functions
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Key API Patterns

### 1. Research & Data Crawling

```typescript
// src/lib/research/crawler.ts
import { RapidAPIClient } from '@/lib/api/rapidapi';

interface ResearchQuery {
  keyword: string;
  sources: ('techcrunch' | 'twitter' | 'linkedin' | 'a16z')[];
  timeframe: '24h' | '7d' | '30d';
}

export async function crawlResearch(query: ResearchQuery) {
  const client = new RapidAPIClient(process.env.RAPIDAPI_KEY!);
  
  const results = await Promise.all(
    query.sources.map(source => 
      client.fetchFromSource(source, {
        keyword: query.keyword,
        since: query.timeframe
      })
    )
  );
  
  return {
    articles: results.flat(),
    insights: extractInsights(results),
    trends: analyzeTrends(results)
  };
}

function extractInsights(data: any[]) {
  // Extract key insights from crawled data
  return data.map(article => ({
    title: article.title,
    summary: article.summary,
    source: article.source,
    publishedAt: article.publishedAt,
    keyPoints: article.keyPoints
  }));
}
```

### 2. AI Content Generation with Claude

```typescript
// src/lib/ai/claude-generator.ts
import Anthropic from '@anthropic-ai/sdk';

interface ContentConfig {
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  tone: 'expert' | 'friendly' | 'humorous';
  language: 'en' | 'vi';
  researchData: any;
}

export async function generateContent(config: ContentConfig) {
  const anthropic = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY,
  });

  const systemPrompt = buildSystemPrompt(config);
  const userPrompt = buildUserPrompt(config.researchData);

  const message = await anthropic.messages.create({
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

  return {
    content: message.content[0].type === 'text' 
      ? message.content[0].text 
      : '',
    metadata: {
      format: config.format,
      tone: config.tone,
      language: config.language,
      tokensUsed: message.usage.output_tokens
    }
  };
}

function buildSystemPrompt(config: ContentConfig): string {
  const formatInstructions = {
    'toplist': 'Create a compelling numbered list article with detailed explanations for each point',
    'pov': 'Write from a unique perspective with personal insights and expert analysis',
    'case-study': 'Present a detailed case study with background, challenges, solutions, and results',
    'how-to': 'Create a step-by-step tutorial with clear instructions and actionable advice'
  };

  return `You are an expert content creator specializing in ${config.format} articles.
Write in a ${config.tone} tone for ${config.language === 'en' ? 'English' : 'Vietnamese'} readers.
${formatInstructions[config.format]}
Use the research data provided to create data-backed, insightful content.`;
}
```

### 3. OpenAI Integration Alternative

```typescript
// src/lib/ai/openai-generator.ts
import OpenAI from 'openai';

export async function generateWithOpenAI(
  prompt: string,
  context: string
): Promise<string> {
  const openai = new OpenAI({
    apiKey: process.env.OPENAI_API_KEY,
  });

  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: 'You are a professional content marketing specialist.'
      },
      {
        role: 'user',
        content: `Context: ${context}\n\nTask: ${prompt}`
      }
    ],
    temperature: 0.8,
    max_tokens: 3000
  });

  return completion.choices[0].message.content || '';
}
```

### 4. Video Generation with Remotion

```typescript
// src/lib/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  template: 'infographic' | 'short' | 'reel';
  content: {
    title: string;
    points: string[];
    images?: string[];
  };
  platform: 'tiktok' | 'instagram' | 'youtube';
}

export async function renderContentVideo(config: VideoConfig) {
  // Bundle the Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.join(process.cwd(), 'remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  // Get composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: config.template,
    inputProps: {
      title: config.content.title,
      points: config.content.points,
      images: config.content.images || []
    },
  });

  // Platform-specific dimensions
  const dimensions = {
    tiktok: { width: 1080, height: 1920 },
    instagram: { width: 1080, height: 1920 },
    youtube: { width: 1080, height: 1920 }
  }[config.platform];

  // Render video
  const outputPath = path.join(
    process.cwd(), 
    'output', 
    `${Date.now()}-${config.template}.mp4`
  );

  await renderMedia({
    composition: {
      ...composition,
      ...dimensions
    },
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: composition.defaultProps,
  });

  return { outputPath, dimensions };
}
```

### 5. Complete Pipeline Workflow

```typescript
// src/lib/pipeline/orchestrator.ts
import { crawlResearch } from '@/lib/research/crawler';
import { generateContent } from '@/lib/ai/claude-generator';
import { renderContentVideo } from '@/lib/video/renderer';

interface PipelineInput {
  keyword: string;
  contentFormat: 'toplist' | 'pov' | 'case-study' | 'how-to';
  languages: ('en' | 'vi')[];
  generateVideo: boolean;
  platform?: 'tiktok' | 'instagram' | 'youtube';
}

export async function runContentPipeline(input: PipelineInput) {
  try {
    // Step 1: Research
    console.log('🔍 Starting research phase...');
    const researchData = await crawlResearch({
      keyword: input.keyword,
      sources: ['techcrunch', 'twitter', 'linkedin'],
      timeframe: '24h'
    });

    // Step 2: Generate content in multiple languages
    console.log('✍️ Generating content...');
    const contentResults = await Promise.all(
      input.languages.map(lang =>
        generateContent({
          format: input.contentFormat,
          tone: 'expert',
          language: lang,
          researchData: researchData.insights
        })
      )
    );

    const contents = contentResults.reduce((acc, result, index) => {
      acc[input.languages[index]] = result.content;
      return acc;
    }, {} as Record<string, string>);

    // Step 3: Generate video if requested
    let videoOutput = null;
    if (input.generateVideo && input.platform) {
      console.log('🎬 Rendering video...');
      
      // Extract key points from content for video
      const points = extractKeyPoints(contents['en'] || contents['vi']);
      
      videoOutput = await renderContentVideo({
        template: 'infographic',
        content: {
          title: input.keyword,
          points: points.slice(0, 5)
        },
        platform: input.platform
      });
    }

    return {
      success: true,
      research: researchData,
      content: contents,
      video: videoOutput,
      metadata: {
        keyword: input.keyword,
        format: input.contentFormat,
        timestamp: new Date().toISOString()
      }
    };
  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}

function extractKeyPoints(content: string): string[] {
  // Extract bullet points or numbered items from content
  const lines = content.split('\n');
  return lines
    .filter(line => /^[\d\-\*]/.test(line.trim()))
    .map(line => line.replace(/^[\d\-\*]+\.?\s*/, '').trim())
    .filter(Boolean);
}
```

### 6. Next.js API Route Example

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline/orchestrator';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    
    const result = await runContentPipeline({
      keyword: body.keyword,
      contentFormat: body.format || 'toplist',
      languages: body.languages || ['en'],
      generateVideo: body.generateVideo || false,
      platform: body.platform
    });

    return NextResponse.json(result);
  } catch (error) {
    return NextResponse.json(
      { error: 'Pipeline execution failed', details: error.message },
      { status: 500 }
    );
  }
}
```

### 7. Frontend Component Usage

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
        body: JSON.stringify({
          keyword,
          format: 'toplist',
          languages: ['en', 'vi'],
          generateVideo: true,
          platform: 'instagram'
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
    <div className="p-6">
      <input
        type="text"
        value={keyword}
        onChange={(e) => setKeyword(e.target.value)}
        placeholder="Enter keyword..."
        className="border p-2 rounded w-full"
      />
      <button
        onClick={handleGenerate}
        disabled={loading}
        className="mt-4 bg-blue-500 text-white px-6 py-2 rounded"
      >
        {loading ? 'Generating...' : 'Generate Content'}
      </button>
      
      {result && (
        <div className="mt-6">
          <h3>Results:</h3>
          <pre className="bg-gray-100 p-4 rounded overflow-auto">
            {JSON.stringify(result, null, 2)}
          </pre>
        </div>
      )}
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
npm run start

# Render Remotion video (standalone)
npm run remotion:render
```

## Common Patterns

### Batch Content Generation

```typescript
async function generateBatchContent(keywords: string[]) {
  const results = [];
  
  for (const keyword of keywords) {
    const result = await runContentPipeline({
      keyword,
      contentFormat: 'toplist',
      languages: ['en'],
      generateVideo: false
    });
    
    results.push(result);
    
    // Rate limiting
    await new Promise(resolve => setTimeout(resolve, 2000));
  }
  
  return results;
}
```

### Scheduled Content Generation

```typescript
// Using node-cron or similar
import cron from 'node-cron';

cron.schedule('0 9 * * *', async () => {
  console.log('Running daily content generation...');
  
  const trendingTopics = await fetchTrendingTopics();
  
  for (const topic of trendingTopics.slice(0, 3)) {
    await runContentPipeline({
      keyword: topic,
      contentFormat: 'pov',
      languages: ['en', 'vi'],
      generateVideo: true,
      platform: 'instagram'
    });
  }
});
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement retry logic with exponential backoff
async function withRetry<T>(
  fn: () => Promise<T>,
  maxRetries = 3
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      await new Promise(resolve => 
        setTimeout(resolve, Math.pow(2, i) * 1000)
      );
    }
  }
  throw new Error('Max retries reached');
}
```

### Video Rendering Errors

```typescript
// Add error handling and cleanup
try {
  await renderContentVideo(config);
} catch (error) {
  console.error('Video rendering failed:', error);
  // Cleanup temporary files
  await cleanupTempFiles();
  throw error;
}
```

### Missing Environment Variables

```typescript
// Validate environment at startup
const requiredEnvVars = [
  'ANTHROPIC_API_KEY',
  'OPENAI_API_KEY',
  'RAPIDAPI_KEY'
];

for (const envVar of requiredEnvVars) {
  if (!process.env[envVar]) {
    throw new Error(`Missing required environment variable: ${envVar}`);
  }
}
```

This skill provides comprehensive guidance for working with the marketing-pipeline-share project's automated content generation system.
