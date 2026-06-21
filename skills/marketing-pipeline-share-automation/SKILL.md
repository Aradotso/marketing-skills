---
name: marketing-pipeline-share-automation
description: AI-powered content automation pipeline from research to video generation using Claude, OpenAI, and Remotion
triggers:
  - how do I automate content creation with AI research
  - generate blog posts with automatic news crawling
  - create videos from blog content automatically
  - set up AI content pipeline with Claude and OpenAI
  - build automated marketing content workflow
  - how to use marketing pipeline for content generation
  - automate research to video content creation
  - create multilingual content with AI automation
---

# Marketing Pipeline Share Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

Marketing Pipeline Share is an end-to-end AI content automation system that transforms keywords into published content and videos. It automatically crawls recent news from sources like TechCrunch, a16z, Twitter, and LinkedIn, generates content in multiple formats and languages using Claude 3 and OpenAI, and renders videos using Remotion.

**Key Capabilities:**
- Auto-crawl news and research data from multiple sources
- Generate content in multiple formats (toplist, POV, case study, how-to)
- Bilingual support (English & Vietnamese)
- Automatic video rendering from content
- Multi-platform optimization (Reels, TikTok, Shorts)

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

```bash
# AI Services
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Optional: Video Rendering
REMOTION_LICENSE_KEY=your_remotion_license_key
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/             # Core libraries
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── crawlers/    # News crawling modules
│   │   ├── generators/  # Content generators
│   │   └── video/       # Video rendering (Remotion)
│   └── utils/           # Utility functions
├── public/              # Static assets
└── remotion/           # Remotion video templates
```

## Core API Usage

### 1. Content Research & Crawling

```typescript
import { Newscrawler } from '@/lib/crawlers/news-crawler';
import { analyzeResearch } from '@/lib/ai/research-analyzer';

// Crawl recent news on a topic
async function researchTopic(keyword: string) {
  const crawler = new NewsCrawler({
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeRange: '24h'
  });
  
  const rawData = await crawler.crawl(keyword);
  
  // Analyze with AI
  const insights = await analyzeResearch(rawData, {
    provider: 'claude', // or 'openai'
    model: 'claude-3-sonnet-20240229'
  });
  
  return insights;
}
```

### 2. Content Generation

```typescript
import { ContentGenerator } from '@/lib/generators/content-generator';

async function generateContent(keyword: string, format: string) {
  const generator = new ContentGenerator({
    aiProvider: 'claude',
    apiKey: process.env.ANTHROPIC_API_KEY
  });
  
  const content = await generator.generate({
    keyword,
    format, // 'toplist' | 'pov' | 'case-study' | 'how-to'
    languages: ['en', 'vi'],
    tone: 'professional', // 'professional' | 'friendly' | 'humorous'
    includeData: true // Include statistics and sources
  });
  
  return content;
}
```

### 3. Multilingual Content Generation

```typescript
import { MultilingualGenerator } from '@/lib/generators/multilingual';

async function createBilingualPost(topic: string) {
  const generator = new MultilingualGenerator();
  
  const result = await generator.generate({
    topic,
    languages: {
      en: {
        tone: 'expert',
        style: 'informative'
      },
      vi: {
        tone: 'friendly',
        style: 'conversational'
      }
    },
    format: 'toplist',
    itemCount: 7
  });
  
  return {
    english: result.en,
    vietnamese: result.vi,
    metadata: result.metadata
  };
}
```

### 4. Video Generation with Remotion

```typescript
import { renderVideo } from '@/lib/video/renderer';
import { createVideoScript } from '@/lib/video/script-generator';

async function generateVideoFromContent(content: any) {
  // Generate video script from content
  const script = await createVideoScript(content, {
    duration: 60, // seconds
    platform: 'reels', // 'reels' | 'tiktok' | 'shorts'
    aspectRatio: '9:16'
  });
  
  // Render video
  const video = await renderVideo({
    composition: 'ContentVideo',
    script,
    outputPath: './public/videos',
    props: {
      title: content.title,
      points: content.keyPoints,
      branding: {
        logo: '/logo.png',
        colors: {
          primary: '#FF6B6B',
          secondary: '#4ECDC4'
        }
      }
    }
  });
  
  return video;
}
```

## Complete Workflow Example

```typescript
import { ContentPipeline } from '@/lib/pipeline';

async function runCompletePipeline(keyword: string) {
  const pipeline = new ContentPipeline({
    anthropicKey: process.env.ANTHROPIC_API_KEY,
    openaiKey: process.env.OPENAI_API_KEY,
    rapidApiKey: process.env.RAPIDAPI_KEY
  });
  
  // Step 1: Research
  console.log('Starting research...');
  const research = await pipeline.research(keyword, {
    sources: ['techcrunch', 'a16z', 'linkedin'],
    depth: 'comprehensive'
  });
  
  // Step 2: Generate content
  console.log('Generating content...');
  const content = await pipeline.generateContent({
    research,
    formats: ['toplist', 'case-study'],
    languages: ['en', 'vi']
  });
  
  // Step 3: Create visuals
  console.log('Creating infographics...');
  const infographic = await pipeline.createInfographic(content.en);
  
  // Step 4: Generate video
  console.log('Rendering video...');
  const video = await pipeline.renderVideo({
    content: content.en,
    template: 'modern',
    platform: 'all' // Generates for all platforms
  });
  
  return {
    content,
    infographic,
    videos: video
  };
}

// Usage
runCompletePipeline('AI Marketing Automation 2026')
  .then(result => console.log('Pipeline complete:', result))
  .catch(err => console.error('Pipeline error:', err));
```

## Next.js API Routes

### Create Content Endpoint

```typescript
// src/app/api/content/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { ContentPipeline } from '@/lib/pipeline';

export async function POST(req: NextRequest) {
  try {
    const { keyword, format, languages } = await req.json();
    
    const pipeline = new ContentPipeline({
      anthropicKey: process.env.ANTHROPIC_API_KEY,
      openaiKey: process.env.OPENAI_API_KEY
    });
    
    const result = await pipeline.generateContent({
      keyword,
      format,
      languages
    });
    
    return NextResponse.json(result);
  } catch (error) {
    return NextResponse.json(
      { error: error.message },
      { status: 500 }
    );
  }
}
```

### Research Endpoint

```typescript
// src/app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { NewsResearcher } from '@/lib/crawlers/news-researcher';

export async function POST(req: NextRequest) {
  try {
    const { keyword, sources, timeRange } = await req.json();
    
    const researcher = new NewsResearcher({
      rapidApiKey: process.env.RAPIDAPI_KEY
    });
    
    const insights = await researcher.research({
      keyword,
      sources: sources || ['techcrunch', 'a16z'],
      timeRange: timeRange || '24h'
    });
    
    return NextResponse.json({ insights });
  } catch (error) {
    return NextResponse.json(
      { error: error.message },
      { status: 500 }
    );
  }
}
```

## Component Usage

### Content Generator UI Component

```typescript
'use client';

import { useState } from 'react';
import { ContentForm } from '@/components/content-form';

export default function ContentGeneratorPage() {
  const [result, setResult] = useState(null);
  const [loading, setLoading] = useState(false);

  async function handleGenerate(formData: any) {
    setLoading(true);
    
    try {
      const response = await fetch('/api/content/generate', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(formData)
      });
      
      const data = await response.json();
      setResult(data);
    } catch (error) {
      console.error('Generation failed:', error);
    } finally {
      setLoading(false);
    }
  }

  return (
    <div className="container mx-auto p-8">
      <h1>AI Content Generator</h1>
      <ContentForm onSubmit={handleGenerate} loading={loading} />
      {result && <ContentPreview content={result} />}
    </div>
  );
}
```

## Common Patterns

### Batch Content Generation

```typescript
async function generateBatchContent(keywords: string[]) {
  const pipeline = new ContentPipeline({
    anthropicKey: process.env.ANTHROPIC_API_KEY
  });
  
  const results = await Promise.all(
    keywords.map(async (keyword) => {
      const research = await pipeline.research(keyword);
      const content = await pipeline.generateContent({
        research,
        formats: ['toplist'],
        languages: ['en']
      });
      
      return { keyword, content };
    })
  );
  
  return results;
}
```

### Content Scheduling

```typescript
import { schedulePost } from '@/lib/scheduler';

async function scheduleContent(content: any, platform: string) {
  const scheduled = await schedulePost({
    platform, // 'facebook' | 'linkedin' | 'twitter'
    content: content.text,
    media: content.images,
    scheduledTime: new Date(Date.now() + 3600000), // 1 hour from now
    metadata: {
      generatedBy: 'ai-pipeline',
      keywords: content.keywords
    }
  });
  
  return scheduled;
}
```

### Custom AI Prompts

```typescript
import { customPrompt } from '@/lib/ai/custom-prompts';

async function generateWithCustomPrompt(topic: string) {
  const prompt = customPrompt('expert-analysis', {
    topic,
    includeStats: true,
    tone: 'authoritative',
    length: 'long-form'
  });
  
  const result = await fetch('https://api.anthropic.com/v1/messages', {
    method: 'POST',
    headers: {
      'x-api-key': process.env.ANTHROPIC_API_KEY,
      'anthropic-version': '2023-06-01',
      'content-type': 'application/json'
    },
    body: JSON.stringify({
      model: 'claude-3-sonnet-20240229',
      max_tokens: 4096,
      messages: [{ role: 'user', content: prompt }]
    })
  });
  
  return result.json();
}
```

## Development Commands

```bash
# Run development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Run Remotion preview
npm run remotion:preview

# Render video
npm run remotion:render -- --composition=ContentVideo

# Type checking
npm run type-check

# Linting
npm run lint
```

## Troubleshooting

### API Rate Limits

```typescript
import { RateLimiter } from '@/lib/utils/rate-limiter';

const limiter = new RateLimiter({
  maxRequests: 50,
  windowMs: 60000 // 1 minute
});

async function safeApiCall(fn: () => Promise<any>) {
  await limiter.wait();
  return fn();
}
```

### Error Handling

```typescript
import { PipelineError } from '@/lib/errors';

async function robustGeneration(keyword: string) {
  try {
    return await pipeline.generateContent({ keyword });
  } catch (error) {
    if (error instanceof PipelineError) {
      // Handle specific pipeline errors
      console.error('Pipeline error:', error.stage, error.message);
      
      // Retry with fallback
      if (error.stage === 'research') {
        return await pipeline.generateContent({
          keyword,
          skipResearch: true,
          useCache: true
        });
      }
    }
    throw error;
  }
}
```

### Video Rendering Issues

```typescript
// Check Remotion configuration
import { getVideoMetadata } from '@remotion/renderer';

async function validateVideoSetup() {
  try {
    const metadata = await getVideoMetadata({
      serveUrl: 'http://localhost:3000/remotion',
      composition: 'ContentVideo'
    });
    console.log('Video setup valid:', metadata);
  } catch (error) {
    console.error('Video setup issue:', error);
    // Fallback to image generation
  }
}
```

## Best Practices

1. **Always use environment variables** for API keys
2. **Implement rate limiting** for external API calls
3. **Cache research results** to avoid redundant crawling
4. **Use streaming responses** for long-form content generation
5. **Validate content** before video rendering to save resources
6. **Store generated content** in a database for reuse
7. **Monitor API usage** to stay within quotas
