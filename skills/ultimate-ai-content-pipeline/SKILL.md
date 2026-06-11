---
name: ultimate-ai-content-pipeline
description: Automated content pipeline with AI research, multi-format content generation, and video rendering using Claude/OpenAI and Remotion
triggers:
  - how do I set up the AI content pipeline
  - generate automated content with research
  - create videos from content automatically
  - use Claude to write marketing content
  - automate content creation workflow
  - render videos with Remotion
  - crawl and research trending topics
  - generate multi-language content posts
---

# Ultimate AI Content Pipeline Skill

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to help developers use the **Ultimate AI Content Pipeline** - a TypeScript-based system that automates content creation from research to video generation. The pipeline crawls trending topics, generates multi-format content using Claude/OpenAI, and renders videos using Remotion.

## What This Project Does

The Ultimate AI Content Pipeline is an end-to-end content automation system that:

1. **Auto-Research**: Crawls real-time data from TechCrunch, a16z, Twitter/X, LinkedIn
2. **AI Content Generation**: Creates content in multiple formats (toplist, POV, case study, how-to) using Claude 3 or OpenAI
3. **Multi-Language Support**: Generates parallel English and Vietnamese versions
4. **Video Rendering**: Automatically creates infographics and short videos using Remotion
5. **Platform Optimization**: Exports videos for Reels, TikTok, Shorts

## Installation

### Prerequisites

```bash
# Node.js 18+ required
node --version

# Install dependencies
npm install
# or
yarn install
# or
pnpm install
```

### Environment Configuration

Create a `.env.local` file in the root directory:

```bash
# AI Models
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Remotion (for video rendering)
REMOTION_WEBHOOK_SECRET=your_webhook_secret
```

### Development Server

```bash
npm run dev
# Server runs on http://localhost:3000
```

### Build for Production

```bash
npm run build
npm start
```

## Project Structure

```
marketing-pipeline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── crawler/     # Web scraping modules
│   │   ├── content/     # Content generation logic
│   │   └── video/       # Remotion video rendering
│   └── utils/           # Utility functions
├── public/              # Static assets
└── remotion/            # Remotion video templates
```

## Core API Usage

### 1. Research & Crawling

```typescript
import { crawlTrendingTopics } from '@/lib/crawler/trending';
import { analyzeSources } from '@/lib/crawler/analyzer';

// Crawl trending topics from multiple sources
async function researchTopic(keyword: string) {
  const sources = await crawlTrendingTopics({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeRange: '24h'
  });

  const insights = await analyzeSources(sources);
  
  return {
    rawData: sources,
    insights,
    trends: insights.topTrends,
    statistics: insights.stats
  };
}

// Usage
const research = await researchTopic('AI automation');
console.log(research.insights);
```

### 2. AI Content Generation with Claude

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

async function generateContent(
  research: any,
  format: 'toplist' | 'pov' | 'case-study' | 'how-to',
  language: 'en' | 'vi'
) {
  const prompt = buildPrompt(research, format, language);
  
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [
      {
        role: 'user',
        content: prompt
      }
    ],
    temperature: 0.7
  });

  return message.content[0].text;
}

function buildPrompt(research: any, format: string, language: string) {
  const formatInstructions = {
    'toplist': 'Create a numbered list article with rankings',
    'pov': 'Write from a personal perspective with opinions',
    'case-study': 'Analyze with data-backed insights',
    'how-to': 'Provide step-by-step actionable guide'
  };

  return `
Based on this research data: ${JSON.stringify(research.insights)}

Create a ${format} article in ${language} language.
Format instructions: ${formatInstructions[format]}

Include:
- Attention-grabbing headline
- Data-backed insights from the research
- Actionable takeaways
- SEO-optimized structure

Tone: Professional yet engaging
`;
}
```

### 3. OpenAI Alternative

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateWithOpenAI(
  research: any,
  format: string,
  language: string
) {
  const prompt = buildPrompt(research, format, language);
  
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: 'You are an expert content marketer who creates engaging, data-driven articles.'
      },
      {
        role: 'user',
        content: prompt
      }
    ],
    temperature: 0.7,
    max_tokens: 4000
  });

  return completion.choices[0].message.content;
}
```

### 4. Video Rendering with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

async function renderContentVideo(
  content: string,
  title: string,
  format: 'reel' | 'tiktok' | 'shorts'
) {
  const dimensions = {
    reel: { width: 1080, height: 1920 },
    tiktok: { width: 1080, height: 1920 },
    shorts: { width: 1080, height: 1920 }
  };

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
      title,
      content: content.split('\n').slice(0, 5), // First 5 lines
      ...dimensions[format]
    },
  });

  // Render video
  const outputPath = path.join(process.cwd(), 'output', `${Date.now()}.mp4`);
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: composition.inputProps,
  });

  return outputPath;
}

// Usage
const videoPath = await renderContentVideo(
  generatedContent,
  'Top 10 AI Tools 2024',
  'reel'
);
```

## Complete Content Pipeline

```typescript
// src/lib/pipeline/complete-pipeline.ts

interface PipelineConfig {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  languages: ('en' | 'vi')[];
  generateVideo: boolean;
  videoFormat?: 'reel' | 'tiktok' | 'shorts';
  aiProvider: 'claude' | 'openai';
}

export async function runContentPipeline(config: PipelineConfig) {
  console.log(`🚀 Starting pipeline for: ${config.keyword}`);

  // Step 1: Research
  console.log('📡 Researching trending topics...');
  const research = await researchTopic(config.keyword);

  // Step 2: Generate content in multiple languages
  console.log('🧠 Generating content...');
  const contents = {};
  
  for (const lang of config.languages) {
    if (config.aiProvider === 'claude') {
      contents[lang] = await generateContent(research, config.format, lang);
    } else {
      contents[lang] = await generateWithOpenAI(research, config.format, lang);
    }
  }

  // Step 3: Render video if requested
  let videoPath = null;
  if (config.generateVideo && config.videoFormat) {
    console.log('🎬 Rendering video...');
    videoPath = await renderContentVideo(
      contents['en'],
      `${config.keyword} - ${config.format}`,
      config.videoFormat
    );
  }

  console.log('✅ Pipeline complete!');
  
  return {
    research,
    contents,
    videoPath,
    metadata: {
      keyword: config.keyword,
      format: config.format,
      generatedAt: new Date().toISOString()
    }
  };
}

// Example usage
const result = await runContentPipeline({
  keyword: 'AI automation trends 2024',
  format: 'toplist',
  languages: ['en', 'vi'],
  generateVideo: true,
  videoFormat: 'reel',
  aiProvider: 'claude'
});
```

## API Routes (Next.js)

```typescript
// src/app/api/generate/route.ts

import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline/complete-pipeline';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    
    const result = await runContentPipeline({
      keyword: body.keyword,
      format: body.format || 'toplist',
      languages: body.languages || ['en'],
      generateVideo: body.generateVideo || false,
      videoFormat: body.videoFormat || 'reel',
      aiProvider: body.aiProvider || 'claude'
    });

    return NextResponse.json({
      success: true,
      data: result
    });
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { success: false, error: error.message },
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

export function ContentGenerator() {
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState(null);

  async function handleGenerate(formData: FormData) {
    setLoading(true);
    
    const response = await fetch('/api/generate', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        keyword: formData.get('keyword'),
        format: formData.get('format'),
        languages: ['en', 'vi'],
        generateVideo: formData.get('generateVideo') === 'on',
        videoFormat: formData.get('videoFormat'),
        aiProvider: formData.get('aiProvider')
      })
    });

    const data = await response.json();
    setResult(data);
    setLoading(false);
  }

  return (
    <div className="content-generator">
      <form action={handleGenerate}>
        <input 
          type="text" 
          name="keyword" 
          placeholder="Enter topic keyword"
          required
        />
        
        <select name="format">
          <option value="toplist">Top List</option>
          <option value="pov">POV</option>
          <option value="case-study">Case Study</option>
          <option value="how-to">How-to Guide</option>
        </select>

        <select name="aiProvider">
          <option value="claude">Claude 3</option>
          <option value="openai">OpenAI GPT-4</option>
        </select>

        <label>
          <input type="checkbox" name="generateVideo" />
          Generate Video
        </label>

        <select name="videoFormat">
          <option value="reel">Instagram Reel</option>
          <option value="tiktok">TikTok</option>
          <option value="shorts">YouTube Shorts</option>
        </select>

        <button type="submit" disabled={loading}>
          {loading ? 'Generating...' : 'Generate Content'}
        </button>
      </form>

      {result && (
        <div className="results">
          <h3>Generated Content</h3>
          <pre>{JSON.stringify(result, null, 2)}</pre>
        </div>
      )}
    </div>
  );
}
```

## Configuration Patterns

### Custom Content Formats

```typescript
// src/lib/content/formats.ts

export const contentFormats = {
  toplist: {
    structure: 'numbered-list',
    minItems: 5,
    maxItems: 15,
    includeIntro: true,
    includeConclusion: true
  },
  pov: {
    structure: 'opinion-based',
    personalVoice: true,
    includeExamples: true,
    controversialOk: true
  },
  'case-study': {
    structure: 'analysis',
    requireData: true,
    includeCharts: true,
    citations: true
  },
  'how-to': {
    structure: 'step-by-step',
    actionable: true,
    includeScreenshots: false,
    difficultyLevel: 'intermediate'
  }
};

export function getFormatConfig(format: string) {
  return contentFormats[format] || contentFormats.toplist;
}
```

### Voice & Tone Customization

```typescript
// src/lib/content/voice.ts

export interface VoiceConfig {
  tone: 'professional' | 'friendly' | 'humorous' | 'authoritative';
  formality: 'formal' | 'casual';
  audience: 'beginners' | 'intermediate' | 'experts';
  emoji: boolean;
}

export function applyVoice(content: string, voice: VoiceConfig): string {
  const systemPrompt = `
Rewrite this content with the following characteristics:
- Tone: ${voice.tone}
- Formality: ${voice.formality}
- Target audience: ${voice.audience}
- Use emojis: ${voice.emoji ? 'yes' : 'no'}

Original content:
${content}
  `;

  // Process through AI to adjust voice
  return systemPrompt;
}
```

## Troubleshooting

### API Rate Limits

```typescript
// src/lib/utils/rate-limit.ts

import { Ratelimit } from '@upstash/ratelimit';
import { Redis } from '@upstash/redis';

const redis = new Redis({
  url: process.env.UPSTASH_REDIS_URL,
  token: process.env.UPSTASH_REDIS_TOKEN,
});

const ratelimit = new Ratelimit({
  redis,
  limiter: Ratelimit.slidingWindow(10, '1 h'),
});

export async function checkRateLimit(identifier: string) {
  const { success, limit, remaining } = await ratelimit.limit(identifier);
  
  if (!success) {
    throw new Error(`Rate limit exceeded. Try again later. Remaining: ${remaining}/${limit}`);
  }
  
  return { remaining, limit };
}
```

### Error Handling

```typescript
// src/lib/utils/error-handler.ts

export class PipelineError extends Error {
  constructor(
    message: string,
    public stage: 'research' | 'generation' | 'rendering',
    public originalError?: Error
  ) {
    super(message);
    this.name = 'PipelineError';
  }
}

export async function withErrorHandling<T>(
  fn: () => Promise<T>,
  stage: string
): Promise<T> {
  try {
    return await fn();
  } catch (error) {
    console.error(`Error in ${stage}:`, error);
    throw new PipelineError(
      `Failed at ${stage}: ${error.message}`,
      stage as any,
      error
    );
  }
}
```

### Debugging Tips

```typescript
// Enable debug logging
process.env.DEBUG = 'pipeline:*';

// Log research data
console.log('Research insights:', JSON.stringify(research, null, 2));

// Test AI generation without full pipeline
const testContent = await generateContent(mockResearch, 'toplist', 'en');

// Verify video rendering locally
const testVideo = await renderContentVideo('Test content', 'Test', 'reel');
```

## Performance Optimization

```typescript
// Parallel content generation
async function generateMultiLanguage(research: any, format: string) {
  const [englishContent, vietnameseContent] = await Promise.all([
    generateContent(research, format, 'en'),
    generateContent(research, format, 'vi')
  ]);
  
  return { en: englishContent, vi: vietnameseContent };
}

// Cache research results
import { unstable_cache } from 'next/cache';

const cachedResearch = unstable_cache(
  async (keyword: string) => researchTopic(keyword),
  ['research-cache'],
  { revalidate: 3600 } // 1 hour
);
```

This skill provides comprehensive guidance for using the Ultimate AI Content Pipeline to automate content creation workflows with AI-powered research, generation, and video rendering capabilities.
