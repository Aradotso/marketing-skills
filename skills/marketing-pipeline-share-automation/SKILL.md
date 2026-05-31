---
name: marketing-pipeline-share-automation
description: Automated AI content pipeline for research, script generation, and video rendering using Claude, OpenAI, and Remotion
triggers:
  - "generate automated marketing content with AI"
  - "create content pipeline from research to video"
  - "automate blog posts and video generation"
  - "scrape news and generate content automatically"
  - "build AI-powered content workflow"
  - "generate videos from text content using Remotion"
  - "create multilingual content with Claude and OpenAI"
  - "set up automated content research pipeline"
---

# Marketing Pipeline Share Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with **Ultimate AI Content Pipeline**, a TypeScript-based automated content generation system that handles the complete workflow from research to video generation. The pipeline crawls news sources, generates content in multiple formats and languages using Claude/OpenAI, and renders videos using Remotion.

## What This Project Does

The Marketing Pipeline Share project is an end-to-end content automation system that:

- **Auto-scans and researches** trending topics from TechCrunch, a16z, Twitter/X, LinkedIn
- **Generates content** in multiple formats (toplist, POV, case study, how-to) using Claude 3 and OpenAI
- **Creates multilingual content** (English and Vietnamese) with customizable tone
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

# Set up environment variables
cp .env.example .env
```

### Required Environment Variables

```bash
# .env
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key
RAPIDAPI_KEY=your_rapidapi_key

# Optional
DATABASE_URL=postgresql://...
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

### Start Development Server

```bash
npm run dev
# Server runs on http://localhost:3000
```

## Core Architecture

The project follows a modular pipeline structure:

```
src/
├── app/              # Next.js app directory
├── lib/
│   ├── ai/          # AI integrations (Claude, OpenAI)
│   ├── scraper/     # News crawling modules
│   ├── generator/   # Content generation logic
│   └── video/       # Remotion video rendering
├── components/       # React components
└── remotion/        # Video templates
```

## Key APIs and Usage Patterns

### 1. Research & Scraping Module

```typescript
import { scrapeNews } from '@/lib/scraper/news-crawler';
import { analyzeInsights } from '@/lib/ai/insight-analyzer';

// Scrape trending news from multiple sources
const scrapeSources = async (keyword: string) => {
  const sources = [
    'techcrunch',
    'a16z',
    'twitter',
    'linkedin'
  ];
  
  const results = await scrapeNews({
    keyword,
    sources,
    timeRange: '24h',
    maxResults: 50
  });
  
  return results;
};

// Analyze and extract insights
const getInsights = async (articles: Article[]) => {
  const insights = await analyzeInsights({
    articles,
    model: 'claude-3-opus-20240229',
    apiKey: process.env.ANTHROPIC_API_KEY
  });
  
  return insights;
};
```

### 2. Content Generation with AI

```typescript
import { generateContent } from '@/lib/generator/content-generator';

// Generate content in multiple formats
const createContent = async (topic: string, format: string) => {
  const content = await generateContent({
    topic,
    format, // 'toplist' | 'pov' | 'case-study' | 'how-to'
    language: 'both', // 'en' | 'vi' | 'both'
    tone: 'professional', // 'professional' | 'friendly' | 'humorous'
    provider: 'claude', // 'claude' | 'openai'
    apiKey: process.env.ANTHROPIC_API_KEY
  });
  
  return content;
};

// Example: Generate a toplist article
const generateToplist = async () => {
  const insights = await getInsights(articles);
  
  const content = await generateContent({
    topic: 'AI Marketing Tools 2024',
    format: 'toplist',
    language: 'both',
    tone: 'professional',
    provider: 'claude',
    apiKey: process.env.ANTHROPIC_API_KEY,
    context: insights
  });
  
  // Returns both English and Vietnamese versions
  console.log(content.en);
  console.log(content.vi);
};
```

### 3. OpenAI Integration Pattern

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY
});

// Generate content with GPT-4
const generateWithOpenAI = async (prompt: string) => {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: 'You are an expert marketing content writer.'
      },
      {
        role: 'user',
        content: prompt
      }
    ],
    temperature: 0.7,
    max_tokens: 2000
  });
  
  return completion.choices[0].message.content;
};
```

### 4. Claude Integration Pattern

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY
});

// Generate content with Claude
const generateWithClaude = async (prompt: string) => {
  const message = await anthropic.messages.create({
    model: 'claude-3-opus-20240229',
    max_tokens: 4096,
    messages: [
      {
        role: 'user',
        content: prompt
      }
    ]
  });
  
  return message.content[0].text;
};
```

### 5. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import { VideoTemplate } from '@/remotion/VideoTemplate';

// Render video from content
const generateVideo = async (content: ContentData) => {
  const bundleLocation = await bundle({
    entryPoint: './remotion/index.ts',
    webpackOverride: (config) => config
  });
  
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: content
  });
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: `out/${content.id}.mp4`,
    inputProps: content
  });
};

// Create video for multiple platforms
const renderMultiPlatform = async (content: ContentData) => {
  const platforms = [
    { name: 'reels', width: 1080, height: 1920 },
    { name: 'tiktok', width: 1080, height: 1920 },
    { name: 'youtube', width: 1920, height: 1080 }
  ];
  
  for (const platform of platforms) {
    await generateVideo({
      ...content,
      dimensions: { width: platform.width, height: platform.height },
      outputPath: `out/${platform.name}/${content.id}.mp4`
    });
  }
};
```

### 6. Complete Pipeline Workflow

```typescript
import { ContentPipeline } from '@/lib/pipeline';

// Run complete automation pipeline
const runContentPipeline = async (keyword: string) => {
  const pipeline = new ContentPipeline({
    openaiKey: process.env.OPENAI_API_KEY,
    claudeKey: process.env.ANTHROPIC_API_KEY,
    rapidApiKey: process.env.RAPIDAPI_KEY
  });
  
  // Step 1: Research
  const research = await pipeline.research({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter'],
    timeRange: '24h'
  });
  
  // Step 2: Generate content
  const content = await pipeline.generate({
    research,
    format: 'toplist',
    languages: ['en', 'vi'],
    tone: 'professional'
  });
  
  // Step 3: Create videos
  const videos = await pipeline.renderVideos({
    content,
    platforms: ['reels', 'tiktok', 'shorts']
  });
  
  return {
    research,
    content,
    videos
  };
};
```

## Common Configuration Patterns

### Custom Content Templates

```typescript
// lib/generator/templates.ts
export const contentTemplates = {
  toplist: {
    structure: [
      'intro',
      'items',
      'conclusion',
      'cta'
    ],
    toneVariations: {
      professional: 'formal, data-driven',
      friendly: 'conversational, accessible',
      humorous: 'witty, engaging'
    }
  },
  pov: {
    structure: [
      'hook',
      'context',
      'opinion',
      'evidence',
      'conclusion'
    ]
  },
  caseStudy: {
    structure: [
      'problem',
      'solution',
      'implementation',
      'results',
      'takeaways'
    ]
  }
};

// Use in content generation
const generateFromTemplate = async (
  template: keyof typeof contentTemplates,
  data: any
) => {
  const config = contentTemplates[template];
  // Generate based on structure
};
```

### Video Template Configuration

```typescript
// remotion/VideoTemplate.tsx
import { AbsoluteFill, useCurrentFrame } from 'remotion';

export const VideoTemplate: React.FC<{
  title: string;
  content: string[];
  branding: {
    logo: string;
    colors: string[];
  };
}> = ({ title, content, branding }) => {
  const frame = useCurrentFrame();
  
  return (
    <AbsoluteFill style={{ backgroundColor: branding.colors[0] }}>
      <h1 style={{ opacity: Math.min(1, frame / 30) }}>
        {title}
      </h1>
      {/* Render content sections */}
    </AbsoluteFill>
  );
};
```

## API Routes (Next.js)

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { ContentPipeline } from '@/lib/pipeline';

export async function POST(req: NextRequest) {
  const { keyword, format, languages } = await req.json();
  
  const pipeline = new ContentPipeline({
    openaiKey: process.env.OPENAI_API_KEY,
    claudeKey: process.env.ANTHROPIC_API_KEY
  });
  
  try {
    const result = await pipeline.generate({
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

## Troubleshooting

### API Rate Limits

```typescript
// Implement retry logic with exponential backoff
const retryWithBackoff = async (
  fn: () => Promise<any>,
  maxRetries = 3
) => {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (error.status === 429) {
        const delay = Math.pow(2, i) * 1000;
        await new Promise(resolve => setTimeout(resolve, delay));
        continue;
      }
      throw error;
    }
  }
};
```

### Video Rendering Errors

```typescript
// Check Remotion configuration
if (!process.env.REMOTION_BUNDLE) {
  throw new Error('Remotion bundle not configured');
}

// Ensure proper codec support
const renderWithFallback = async (composition) => {
  try {
    await renderMedia({ ...composition, codec: 'h264' });
  } catch (error) {
    console.warn('H264 failed, trying VP8');
    await renderMedia({ ...composition, codec: 'vp8' });
  }
};
```

### Memory Issues with Large Datasets

```typescript
// Process in batches
const processBatch = async (items: any[], batchSize = 10) => {
  const results = [];
  
  for (let i = 0; i < items.length; i += batchSize) {
    const batch = items.slice(i, i + batchSize);
    const batchResults = await Promise.all(
      batch.map(item => processItem(item))
    );
    results.push(...batchResults);
  }
  
  return results;
};
```

### Environment Variables Not Loading

```bash
# Verify .env file location (project root)
# For Next.js, prefix client-side vars with NEXT_PUBLIC_
NEXT_PUBLIC_APP_URL=http://localhost:3000

# Server-side only (no prefix needed)
OPENAI_API_KEY=sk-...
ANTHROPIC_API_KEY=sk-ant-...
```

## Best Practices

1. **Always validate API keys** before running pipelines
2. **Cache research results** to avoid redundant API calls
3. **Use batch processing** for video generation
4. **Implement proper error handling** for AI API failures
5. **Monitor token usage** to stay within API limits
6. **Version control your templates** for consistent output
7. **Test video renders** on target platforms before bulk generation
