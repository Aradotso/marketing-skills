---
name: ultimate-ai-content-pipeline
description: Automated content pipeline for research, scriptwriting, and video generation using Claude/OpenAI and Remotion
triggers:
  - how do I automate content creation with AI
  - set up an AI content pipeline for marketing
  - generate videos automatically from blog posts
  - create content workflow with Claude and OpenAI
  - automate research and content generation
  - build an AI-powered marketing content system
  - use Remotion to render videos from content
  - set up automated content research and publishing
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This project is an end-to-end AI-powered content automation system that handles research, content generation, and video rendering. It automatically scrapes news from sources like TechCrunch, a16z, Twitter/X, and LinkedIn, generates content in multiple formats and languages using Claude/OpenAI, and renders videos using Remotion.

## What This Project Does

- **Auto-Research**: Crawls and analyzes recent content from major tech news sources
- **AI Content Generation**: Creates articles in multiple formats (toplist, POV, case study, how-to) using Claude 3 and OpenAI
- **Multi-language Support**: Generates content in English and Vietnamese simultaneously
- **Video Rendering**: Converts written content into infographics and short-form videos using Remotion
- **Full Pipeline**: Keyword input → research → content generation → video output

## Installation

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
yarn install
```

## Environment Configuration

Create a `.env.local` file in the root directory:

```bash
# AI APIs
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research/Scraping APIs
RAPIDAPI_KEY=your_rapidapi_key

# Optional: Custom endpoints
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
│   │   ├── research/    # Content scraping and analysis
│   │   ├── video/       # Remotion video rendering
│   │   └── utils/       # Shared utilities
│   └── types/           # TypeScript type definitions
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Core API Patterns

### 1. Research Module

Automatically fetch and analyze recent content:

```typescript
import { researchContent } from '@/lib/research/crawler';
import { analyzeInsights } from '@/lib/research/analyzer';

async function gatherResearch(keyword: string) {
  // Fetch recent articles from multiple sources
  const articles = await researchContent({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeRange: '24h',
    limit: 20
  });

  // Extract insights and data points
  const insights = await analyzeInsights(articles, {
    extractStats: true,
    identifyTrends: true,
    summarize: true
  });

  return {
    articles,
    insights,
    timestamp: new Date()
  };
}
```

### 2. Content Generation with Claude/OpenAI

Generate content in multiple formats:

```typescript
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY
});

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY
});

// Using Claude for content generation
async function generateWithClaude(
  research: any,
  format: 'toplist' | 'pov' | 'case-study' | 'how-to',
  language: 'en' | 'vi'
) {
  const prompt = buildPrompt(research, format, language);

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [{
      role: 'user',
      content: prompt
    }],
    system: `You are an expert content writer specializing in ${format} articles. 
             Use the research data provided to create engaging, data-backed content.`
  });

  return message.content[0].text;
}

// Using OpenAI for alternative generation
async function generateWithOpenAI(
  research: any,
  format: string,
  tone: 'professional' | 'friendly' | 'humorous'
) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are a ${tone} content writer creating ${format} content.`
      },
      {
        role: 'user',
        content: buildPrompt(research, format, 'en')
      }
    ],
    temperature: 0.7,
    max_tokens: 3000
  });

  return completion.choices[0].message.content;
}

function buildPrompt(research: any, format: string, language: string): string {
  return `
    Research Data: ${JSON.stringify(research.insights)}
    
    Create a ${format} article in ${language} based on this research.
    Include:
    - Data-backed insights from the research
    - Recent trends and statistics
    - Actionable takeaways
    
    Format: ${format}
    Language: ${language}
    Word count: 1200-1500 words
  `;
}
```

### 3. Video Generation with Remotion

Convert content to video:

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

async function renderContentVideo(
  content: string,
  outputPath: string,
  platform: 'reels' | 'tiktok' | 'shorts'
) {
  // Platform-specific dimensions
  const dimensions = {
    reels: { width: 1080, height: 1920 },
    tiktok: { width: 1080, height: 1920 },
    shorts: { width: 1080, height: 1920 }
  };

  const { width, height } = dimensions[platform];

  // Bundle Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.join(process.cwd(), 'remotion/index.ts'),
    webpackOverride: (config) => config
  });

  // Select composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      content: parseContentForVideo(content),
      platform
    }
  });

  // Render video
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: {
      content: parseContentForVideo(content),
      platform
    }
  });

  return outputPath;
}

function parseContentForVideo(content: string) {
  // Extract key points, headlines, and stats for video scenes
  const sections = content.split('\n\n');
  return sections.map((section, index) => ({
    id: index,
    text: section.trim(),
    duration: 3, // seconds per scene
    type: detectSectionType(section)
  }));
}

function detectSectionType(section: string): 'headline' | 'stat' | 'quote' | 'text' {
  if (section.match(/^\d+%|\d+x|^\$\d+/)) return 'stat';
  if (section.match(/^"|^"/)) return 'quote';
  if (section.match(/^#|^\*\*/)) return 'headline';
  return 'text';
}
```

### 4. Full Pipeline Example

Complete workflow from keyword to video:

```typescript
import { gatherResearch } from '@/lib/research';
import { generateWithClaude } from '@/lib/ai/claude';
import { renderContentVideo } from '@/lib/video/remotion';

async function runContentPipeline(
  keyword: string,
  options: {
    format: 'toplist' | 'pov' | 'case-study' | 'how-to';
    languages: ('en' | 'vi')[];
    generateVideo: boolean;
    platform?: 'reels' | 'tiktok' | 'shorts';
  }
) {
  // Step 1: Research
  console.log(`🔍 Researching: ${keyword}`);
  const research = await gatherResearch(keyword);

  // Step 2: Generate content in multiple languages
  const contentVersions: Record<string, string> = {};
  
  for (const lang of options.languages) {
    console.log(`✍️ Generating ${lang.toUpperCase()} content...`);
    contentVersions[lang] = await generateWithClaude(
      research,
      options.format,
      lang
    );
  }

  // Step 3: Generate video if requested
  let videoPath: string | null = null;
  if (options.generateVideo && options.platform) {
    console.log(`🎬 Rendering video for ${options.platform}...`);
    videoPath = await renderContentVideo(
      contentVersions.en,
      `./output/${keyword}-${options.platform}.mp4`,
      options.platform
    );
  }

  return {
    keyword,
    research: research.insights,
    content: contentVersions,
    videoPath,
    createdAt: new Date()
  };
}

// Usage
const result = await runContentPipeline('AI marketing automation', {
  format: 'toplist',
  languages: ['en', 'vi'],
  generateVideo: true,
  platform: 'reels'
});

console.log('✅ Pipeline complete:', result);
```

## Next.js API Routes

### Create API endpoint for content generation:

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const { keyword, format, languages, generateVideo, platform } = body;

    if (!keyword || !format) {
      return NextResponse.json(
        { error: 'Missing required fields' },
        { status: 400 }
      );
    }

    const result = await runContentPipeline(keyword, {
      format,
      languages: languages || ['en'],
      generateVideo: generateVideo || false,
      platform
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

### Frontend component:

```typescript
// components/ContentGenerator.tsx
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
        body: JSON.stringify({
          keyword,
          format: 'toplist',
          languages: ['en', 'vi'],
          generateVideo: true,
          platform: 'reels'
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
    <div className="p-4">
      <input
        type="text"
        value={keyword}
        onChange={(e) => setKeyword(e.target.value)}
        placeholder="Enter keyword..."
        className="border p-2 rounded"
      />
      <button
        onClick={handleGenerate}
        disabled={loading}
        className="ml-2 bg-blue-500 text-white px-4 py-2 rounded"
      >
        {loading ? 'Generating...' : 'Generate Content'}
      </button>

      {result && (
        <div className="mt-4">
          <h3>Generated Content:</h3>
          <pre className="bg-gray-100 p-4 rounded overflow-auto">
            {JSON.stringify(result, null, 2)}
          </pre>
        </div>
      )}
    </div>
  );
}
```

## Common Patterns

### 1. Batch Content Generation

```typescript
async function batchGenerate(keywords: string[]) {
  const results = await Promise.all(
    keywords.map(keyword =>
      runContentPipeline(keyword, {
        format: 'toplist',
        languages: ['en'],
        generateVideo: false
      })
    )
  );
  return results;
}
```

### 2. Custom Research Sources

```typescript
import { addCustomSource } from '@/lib/research/sources';

addCustomSource({
  name: 'custom-blog',
  url: 'https://example.com/api/posts',
  parser: (data) => ({
    title: data.headline,
    content: data.body,
    publishedAt: new Date(data.date)
  })
});
```

### 3. Video Template Customization

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, Sequence, useCurrentFrame } from 'remotion';

export const ContentVideo: React.FC<{
  content: Array<{ text: string; duration: number }>;
  platform: string;
}> = ({ content, platform }) => {
  const frame = useCurrentFrame();

  return (
    <AbsoluteFill style={{ backgroundColor: '#000' }}>
      {content.map((scene, i) => (
        <Sequence
          key={i}
          from={i * scene.duration * 30}
          durationInFrames={scene.duration * 30}
        >
          <AbsoluteFill className="items-center justify-center p-12">
            <h1 className="text-white text-4xl font-bold text-center">
              {scene.text}
            </h1>
          </AbsoluteFill>
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement retry logic with exponential backoff
async function callAIWithRetry(fn: () => Promise<any>, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error: any) {
      if (error?.status === 429 && i < maxRetries - 1) {
        const delay = Math.pow(2, i) * 1000;
        console.log(`Rate limited. Retrying in ${delay}ms...`);
        await new Promise(resolve => setTimeout(resolve, delay));
        continue;
      }
      throw error;
    }
  }
}
```

### Video Rendering Memory Issues

```typescript
// Use serverless rendering for large videos
import { renderMediaOnLambda } from '@remotion/lambda';

async function renderOnCloud(composition: any) {
  const result = await renderMediaOnLambda({
    region: 'us-east-1',
    functionName: 'remotion-render',
    composition,
    codec: 'h264'
  });
  return result.outputFile;
}
```

### Research Scraping Failures

```typescript
// Graceful degradation when sources fail
async function robustResearch(keyword: string) {
  const sources = ['techcrunch', 'a16z', 'twitter'];
  const results = await Promise.allSettled(
    sources.map(source => fetchFromSource(source, keyword))
  );

  const successful = results
    .filter((r): r is PromiseFulfilledResult<any> => r.status === 'fulfilled')
    .map(r => r.value);

  if (successful.length === 0) {
    throw new Error('All research sources failed');
  }

  return successful.flat();
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

# Render video locally (if Remotion CLI is configured)
npm run render -- --props='{"keyword":"AI"}' --output=output.mp4

# Type checking
npm run type-check

# Linting
npm run lint
```

This system enables complete content automation from research to publication, perfect for marketers, content creators, and agencies looking to scale their content production.
