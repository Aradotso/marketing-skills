---
name: marketing-pipeline-ai-content-automation
description: Automated AI content pipeline for research, scriptwriting, and video generation using Claude, OpenAI, and Remotion
triggers:
  - automate content creation with AI research
  - generate video content from text automatically
  - create marketing content pipeline with Claude
  - scrape news and generate articles with AI
  - build automated content workflow with Remotion
  - set up AI-powered content generation system
  - create multi-format content with OpenAI
  - automate research to video pipeline
---

# Marketing Pipeline AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the Ultimate AI Content Pipeline - an end-to-end automated content creation system that handles research, scriptwriting, article generation, and video rendering using Claude 3, OpenAI, and Remotion.

## What This Project Does

The Marketing Pipeline automates the entire content creation workflow:

1. **Auto-Research**: Crawls real-time data from TechCrunch, a16z, Twitter/X, LinkedIn
2. **AI Content Generation**: Creates articles in multiple formats (listicles, POV, case studies, how-tos) using Claude/OpenAI
3. **Multi-language Support**: Generates content in English and Vietnamese simultaneously
4. **Video Generation**: Automatically renders videos and infographics from written content using Remotion
5. **Platform Optimization**: Exports videos optimized for Reels, TikTok, and Shorts

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
# AI Provider Keys
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research API Keys
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_url

# Remotion License (for video rendering)
REMOTION_LICENSE_KEY=your_remotion_license_key

# Application Settings
NEXT_PUBLIC_APP_URL=http://localhost:3000
NODE_ENV=development
```

## Running the Application

```bash
# Development mode
npm run dev

# Production build
npm run build
npm start

# Remotion video rendering
npm run remotion:render
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/             # Core library functions
│   │   ├── ai/          # AI provider integrations
│   │   ├── research/    # Content research modules
│   │   └── video/       # Remotion video generation
│   ├── types/           # TypeScript type definitions
│   └── utils/           # Utility functions
├── public/              # Static assets
├── remotion/            # Remotion video templates
└── .env.local          # Environment variables
```

## Core API Usage

### Content Research Module

```typescript
import { ResearchEngine } from '@/lib/research/engine';

// Initialize research engine
const researchEngine = new ResearchEngine({
  sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
  timeRange: '24h',
  keywords: ['AI', 'marketing', 'automation']
});

// Perform research
const research = await researchEngine.scan({
  topic: 'AI content automation trends',
  depth: 'detailed',
  includeStats: true
});

console.log(research.insights);
console.log(research.sources);
```

### AI Content Generation with Claude

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

interface ContentRequest {
  topic: string;
  format: 'toplist' | 'pov' | 'casestudy' | 'howto';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  researchData: any;
}

async function generateContent(request: ContentRequest): Promise<string> {
  const prompt = `
    Create a ${request.format} article about "${request.topic}" in ${request.language}.
    Tone: ${request.tone}
    
    Research data:
    ${JSON.stringify(request.researchData, null, 2)}
    
    Requirements:
    - Include real statistics and insights from research data
    - Make it data-backed and credible
    - Optimize for engagement
    - Length: 1500-2000 words
  `;

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

// Usage
const article = await generateContent({
  topic: 'The Future of AI in Marketing',
  format: 'toplist',
  language: 'en',
  tone: 'expert',
  researchData: research
});
```

### AI Content Generation with OpenAI

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateContentOpenAI(request: ContentRequest): Promise<string> {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are an expert content creator specializing in ${request.format} articles with a ${request.tone} tone.`
      },
      {
        role: 'user',
        content: `Write a comprehensive ${request.format} article about "${request.topic}" in ${request.language}.
        
        Research data: ${JSON.stringify(request.researchData, null, 2)}
        
        Make it engaging, data-backed, and optimized for social media sharing.`
      }
    ],
    temperature: 0.7,
    max_tokens: 3000
  });

  return completion.choices[0].message.content || '';
}
```

### Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  title: string;
  content: string;
  format: 'reel' | 'tiktok' | 'short';
  duration: number;
}

async function generateVideo(config: VideoConfig): Promise<string> {
  // Bundle Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  // Get composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: config.format === 'reel' ? 'InstagramReel' : 'TikTokVideo',
    inputProps: {
      title: config.title,
      content: config.content,
      duration: config.duration
    }
  });

  // Render video
  const outputLocation = path.resolve(
    `./output/video-${Date.now()}.mp4`
  );

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: {
      title: config.title,
      content: config.content
    }
  });

  return outputLocation;
}

// Usage
const videoPath = await generateVideo({
  title: 'Top 5 AI Marketing Trends',
  content: article,
  format: 'reel',
  duration: 60
});
```

## Complete Pipeline Workflow

```typescript
import { ContentPipeline } from '@/lib/pipeline';

interface PipelineConfig {
  topic: string;
  contentFormat: 'toplist' | 'pov' | 'casestudy' | 'howto';
  languages: ('en' | 'vi')[];
  generateVideo: boolean;
  videoFormats?: ('reel' | 'tiktok' | 'short')[];
  aiProvider: 'claude' | 'openai';
}

async function runContentPipeline(config: PipelineConfig) {
  const pipeline = new ContentPipeline();

  // Step 1: Research
  console.log('🔍 Starting research...');
  const research = await pipeline.research({
    topic: config.topic,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeRange: '24h'
  });

  // Step 2: Generate content for each language
  console.log('✍️ Generating content...');
  const articles = await Promise.all(
    config.languages.map(lang =>
      pipeline.generateContent({
        topic: config.topic,
        format: config.contentFormat,
        language: lang,
        aiProvider: config.aiProvider,
        researchData: research
      })
    )
  );

  // Step 3: Generate videos (if enabled)
  const videos = [];
  if (config.generateVideo && config.videoFormats) {
    console.log('🎬 Rendering videos...');
    for (const format of config.videoFormats) {
      const videoPath = await pipeline.generateVideo({
        title: config.topic,
        content: articles[0], // Use English version
        format,
        duration: format === 'reel' ? 60 : 90
      });
      videos.push(videoPath);
    }
  }

  return {
    research,
    articles,
    videos
  };
}

// Execute full pipeline
const result = await runContentPipeline({
  topic: 'AI-Powered Marketing Automation in 2024',
  contentFormat: 'toplist',
  languages: ['en', 'vi'],
  generateVideo: true,
  videoFormats: ['reel', 'tiktok'],
  aiProvider: 'claude'
});

console.log('✅ Pipeline complete!');
console.log('Articles:', result.articles.length);
console.log('Videos:', result.videos.length);
```

## API Endpoints (Next.js)

### Research Endpoint

```typescript
// app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { ResearchEngine } from '@/lib/research/engine';

export async function POST(request: NextRequest) {
  try {
    const { topic, sources, timeRange } = await request.json();

    const engine = new ResearchEngine({ sources, timeRange });
    const research = await engine.scan({ topic });

    return NextResponse.json({
      success: true,
      data: research
    });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

### Content Generation Endpoint

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { generateContent } from '@/lib/ai/content-generator';

export async function POST(request: NextRequest) {
  try {
    const params = await request.json();
    const content = await generateContent(params);

    return NextResponse.json({
      success: true,
      content
    });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

### Video Rendering Endpoint

```typescript
// app/api/video/render/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { generateVideo } from '@/lib/video/renderer';

export async function POST(request: NextRequest) {
  try {
    const config = await request.json();
    const videoPath = await generateVideo(config);

    return NextResponse.json({
      success: true,
      videoUrl: `/output/${path.basename(videoPath)}`
    });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

## Frontend Integration

```typescript
// components/ContentGenerator.tsx
'use client';

import { useState } from 'react';

export default function ContentGenerator() {
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState(null);

  const handleGenerate = async (topic: string) => {
    setLoading(true);
    try {
      // Research
      const researchRes = await fetch('/api/research', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          topic,
          sources: ['techcrunch', 'twitter'],
          timeRange: '24h'
        })
      });
      const research = await researchRes.json();

      // Generate content
      const contentRes = await fetch('/api/generate', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          topic,
          format: 'toplist',
          language: 'en',
          tone: 'expert',
          aiProvider: 'claude',
          researchData: research.data
        })
      });
      const content = await contentRes.json();

      // Generate video
      const videoRes = await fetch('/api/video/render', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          title: topic,
          content: content.content,
          format: 'reel',
          duration: 60
        })
      });
      const video = await videoRes.json();

      setResult({ content, video });
    } catch (error) {
      console.error('Generation failed:', error);
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="content-generator">
      <input
        type="text"
        placeholder="Enter topic..."
        onKeyPress={(e) => {
          if (e.key === 'Enter') {
            handleGenerate(e.currentTarget.value);
          }
        }}
      />
      {loading && <p>Generating content...</p>}
      {result && (
        <div>
          <h3>Generated Content</h3>
          <div>{result.content.content}</div>
          <video src={result.video.videoUrl} controls />
        </div>
      )}
    </div>
  );
}
```

## Common Patterns

### Batch Content Generation

```typescript
async function batchGenerate(topics: string[]) {
  const results = [];
  
  for (const topic of topics) {
    const result = await runContentPipeline({
      topic,
      contentFormat: 'toplist',
      languages: ['en', 'vi'],
      generateVideo: true,
      videoFormats: ['reel'],
      aiProvider: 'claude'
    });
    
    results.push(result);
    
    // Rate limiting
    await new Promise(resolve => setTimeout(resolve, 2000));
  }
  
  return results;
}
```

### Custom Research Sources

```typescript
import { RapidAPIClient } from '@/lib/research/rapidapi';

const customResearch = async (keyword: string) => {
  const rapidAPI = new RapidAPIClient(process.env.RAPIDAPI_KEY);
  
  const [news, social, trends] = await Promise.all([
    rapidAPI.searchNews(keyword),
    rapidAPI.searchSocial(keyword, ['twitter', 'linkedin']),
    rapidAPI.getTrends(keyword)
  ]);
  
  return { news, social, trends };
};
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

// Usage
const content = await withRetry(() => generateContent(params));
```

### Video Rendering Issues

```typescript
// Check Remotion version compatibility
import { getRemotionEnvironment } from '@remotion/renderer';

const env = getRemotionEnvironment();
if (env.isRendering) {
  console.log('Rendering in progress...');
}

// Ensure proper codec settings
const renderConfig = {
  codec: 'h264',
  crf: 18,
  pixelFormat: 'yuv420p',
  proResProfile: undefined,
  videoBitrate: '8M'
};
```

### Memory Management for Large Batches

```typescript
async function processLargeBatch(topics: string[]) {
  const chunkSize = 5;
  const results = [];
  
  for (let i = 0; i < topics.length; i += chunkSize) {
    const chunk = topics.slice(i, i + chunkSize);
    const chunkResults = await Promise.all(
      chunk.map(topic => runContentPipeline({ topic, /* ... */ }))
    );
    results.push(...chunkResults);
    
    // Force garbage collection hint
    if (global.gc) global.gc();
  }
  
  return results;
}
```

### Error Handling Best Practices

```typescript
import { logger } from '@/lib/logger';

async function safePipelineRun(config: PipelineConfig) {
  try {
    return await runContentPipeline(config);
  } catch (error) {
    logger.error('Pipeline failed', {
      error: error.message,
      config,
      stack: error.stack
    });
    
    // Implement fallback or partial results
    return {
      success: false,
      error: error.message,
      partialResults: null
    };
  }
}
```

This skill provides comprehensive coverage of the marketing content automation pipeline, enabling AI agents to assist developers in implementing automated content research, generation, and video rendering workflows.
