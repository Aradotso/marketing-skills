---
name: marketing-pipeline-automation
description: AI-powered content automation pipeline that researches, writes, and generates video from keywords using Claude, OpenAI, and Remotion
triggers:
  - automate content creation with ai pipeline
  - generate blog posts and videos automatically
  - research and write content using claude
  - create marketing content from keywords
  - build automated content workflow
  - use remotion to render marketing videos
  - scrape news and generate articles with ai
  - set up content automation pipeline
---

# Marketing Pipeline Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

Marketing Pipeline Automation is an end-to-end content production system that transforms keywords into polished blog posts and videos. It automatically:

1. **Researches** - Crawls recent news from TechCrunch, a16z, Twitter, LinkedIn
2. **Writes** - Generates multi-format content (toplist, POV, case study, how-to) in English and Vietnamese using Claude/OpenAI
3. **Renders** - Creates videos and infographics using Remotion for social media platforms

Perfect for marketers, content creators, and agencies who need to scale content production 10x without sacrificing quality.

## Installation

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
pnpm install
# or
yarn install
```

## Environment Configuration

Create a `.env.local` file in the root directory:

```bash
# AI Providers
ANTHROPIC_API_KEY=your_claude_api_key_here
OPENAI_API_KEY=your_openai_api_key_here

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key_here

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Remotion (for video rendering)
REMOTION_LICENSE_KEY=your_remotion_license_key

# Application
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

## Running the Application

```bash
# Development mode
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Render videos with Remotion
npm run remotion
```

## Core Architecture

### 1. Research Module (Auto-Scan)

The research module crawls and aggregates content from multiple sources:

```typescript
// lib/research/crawler.ts
import axios from 'axios';

interface NewsSource {
  name: string;
  url: string;
  parser: (data: any) => Article[];
}

export async function crawlNews(keyword: string, sources: NewsSource[]) {
  const articles: Article[] = [];
  
  for (const source of sources) {
    try {
      const response = await axios.get(source.url, {
        params: { q: keyword, hours: 24 },
        headers: {
          'X-RapidAPI-Key': process.env.RAPIDAPI_KEY,
        }
      });
      
      const parsed = source.parser(response.data);
      articles.push(...parsed);
    } catch (error) {
      console.error(`Failed to crawl ${source.name}:`, error);
    }
  }
  
  return articles;
}

// Usage
const techArticles = await crawlNews('AI automation', [
  {
    name: 'TechCrunch',
    url: 'https://api.techcrunch.com/search',
    parser: (data) => data.articles.map(a => ({
      title: a.title,
      url: a.url,
      summary: a.description,
      publishedAt: a.publishedAt
    }))
  }
]);
```

### 2. Content Generation with Claude/OpenAI

```typescript
// lib/ai/content-generator.ts
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
  tone: 'expert' | 'friendly' | 'humorous';
  language: 'en' | 'vi';
  research: Article[];
}

export async function generateContent(
  keyword: string,
  config: ContentConfig
): Promise<string> {
  const researchContext = config.research
    .map(a => `Title: ${a.title}\nSummary: ${a.summary}`)
    .join('\n\n');

  const prompt = `
You are an expert content writer. Create a ${config.format} article about "${keyword}".

Research Context (last 24h):
${researchContext}

Requirements:
- Format: ${config.format}
- Tone: ${config.tone}
- Language: ${config.language}
- Include data and insights from the research
- Make it engaging and actionable
- 1000-1500 words

Generate the complete article:
`;

  // Use Claude for longer, analytical content
  if (config.format === 'case-study' || config.format === 'pov') {
    const response = await anthropic.messages.create({
      model: 'claude-3-sonnet-20240229',
      max_tokens: 4096,
      messages: [{ role: 'user', content: prompt }],
    });
    
    return response.content[0].text;
  }

  // Use OpenAI for structured formats
  const response = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [{ role: 'user', content: prompt }],
    temperature: 0.7,
  });

  return response.choices[0].message.content;
}
```

### 3. Video Generation with Remotion

```typescript
// remotion/VideoComposition.tsx
import { AbsoluteFill, Sequence, useCurrentFrame } from 'remotion';
import { z } from 'zod';

export const videoSchema = z.object({
  title: z.string(),
  points: z.array(z.string()),
  duration: z.number(),
});

export const VideoComposition: React.FC<z.infer<typeof videoSchema>> = ({
  title,
  points,
  duration,
}) => {
  const frame = useCurrentFrame();
  const opacity = Math.min(1, frame / 30);

  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a1a' }}>
      <Sequence from={0} durationInFrames={60}>
        <AbsoluteFill
          style={{
            justifyContent: 'center',
            alignItems: 'center',
            opacity,
          }}
        >
          <h1 style={{ color: 'white', fontSize: 64 }}>{title}</h1>
        </AbsoluteFill>
      </Sequence>

      {points.map((point, index) => (
        <Sequence
          key={index}
          from={60 + index * 90}
          durationInFrames={90}
        >
          <AbsoluteFill
            style={{
              justifyContent: 'center',
              alignItems: 'center',
              padding: 60,
            }}
          >
            <div style={{ color: 'white', fontSize: 36 }}>
              {point}
            </div>
          </AbsoluteFill>
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};
```

```typescript
// lib/video/render.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

export async function renderContentVideo(
  content: string,
  outputPath: string
) {
  // Extract key points from content
  const points = extractKeyPoints(content);
  
  const bundleLocation = await bundle(
    path.join(process.cwd(), 'remotion', 'index.ts')
  );

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'VideoComposition',
    inputProps: {
      title: 'AI Content Automation',
      points,
      duration: 300, // 10 seconds per point
    },
  });

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: composition.defaultProps,
  });

  return outputPath;
}

function extractKeyPoints(content: string): string[] {
  // Simple extraction - you can enhance with AI
  const sentences = content.split(/[.!?]\s+/);
  return sentences
    .filter(s => s.length > 50 && s.length < 150)
    .slice(0, 5);
}
```

### 4. Complete Pipeline API

```typescript
// app/api/pipeline/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { crawlNews } from '@/lib/research/crawler';
import { generateContent } from '@/lib/ai/content-generator';
import { renderContentVideo } from '@/lib/video/render';

export async function POST(req: NextRequest) {
  try {
    const { keyword, format, language, tone } = await req.json();

    // Step 1: Research
    const research = await crawlNews(keyword, [
      /* your sources */
    ]);

    // Step 2: Generate Content
    const content = await generateContent(keyword, {
      format,
      tone,
      language,
      research,
    });

    // Step 3: Render Video
    const videoPath = await renderContentVideo(
      content,
      `./public/videos/${Date.now()}.mp4`
    );

    return NextResponse.json({
      success: true,
      content,
      videoUrl: videoPath.replace('./public', ''),
      research: research.length,
    });
  } catch (error) {
    return NextResponse.json(
      { error: error.message },
      { status: 500 }
    );
  }
}
```

### 5. Frontend Integration

```typescript
// app/dashboard/page.tsx
'use client';

import { useState } from 'react';

export default function Dashboard() {
  const [keyword, setKeyword] = useState('');
  const [result, setResult] = useState(null);
  const [loading, setLoading] = useState(false);

  const handleGenerate = async () => {
    setLoading(true);
    try {
      const response = await fetch('/api/pipeline', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          keyword,
          format: 'toplist',
          language: 'en',
          tone: 'expert',
        }),
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
    <div className="container mx-auto p-8">
      <h1 className="text-4xl font-bold mb-8">
        AI Content Pipeline
      </h1>

      <div className="space-y-4">
        <input
          type="text"
          value={keyword}
          onChange={(e) => setKeyword(e.target.value)}
          placeholder="Enter keyword (e.g., 'AI automation')"
          className="w-full p-4 border rounded"
        />

        <button
          onClick={handleGenerate}
          disabled={loading}
          className="px-6 py-3 bg-blue-600 text-white rounded"
        >
          {loading ? 'Generating...' : 'Generate Content'}
        </button>

        {result && (
          <div className="mt-8 space-y-4">
            <div className="p-6 bg-gray-50 rounded">
              <h3 className="font-bold mb-2">Generated Content</h3>
              <p className="whitespace-pre-wrap">{result.content}</p>
            </div>

            {result.videoUrl && (
              <video
                src={result.videoUrl}
                controls
                className="w-full max-w-2xl"
              />
            )}

            <p className="text-sm text-gray-600">
              Based on {result.research} recent articles
            </p>
          </div>
        )}
      </div>
    </div>
  );
}
```

## Common Patterns

### Batch Content Generation

```typescript
// scripts/batch-generate.ts
import { crawlNews } from '@/lib/research/crawler';
import { generateContent } from '@/lib/ai/content-generator';

const keywords = ['AI automation', 'Marketing tools', 'Content strategy'];

async function batchGenerate() {
  for (const keyword of keywords) {
    console.log(`Processing: ${keyword}`);
    
    const research = await crawlNews(keyword, sources);
    const content = await generateContent(keyword, {
      format: 'toplist',
      tone: 'expert',
      language: 'en',
      research,
    });

    // Save to database or file
    await saveContent(keyword, content);
  }
}

batchGenerate();
```

### Custom Research Sources

```typescript
// Add your own news sources
const customSources: NewsSource[] = [
  {
    name: 'Reddit',
    url: 'https://reddit-api.com/search',
    parser: (data) => data.posts.map(p => ({
      title: p.title,
      url: p.permalink,
      summary: p.selftext.slice(0, 200),
      publishedAt: new Date(p.created_utc * 1000),
    })),
  },
];
```

## Troubleshooting

### API Rate Limits

```typescript
// lib/utils/rate-limiter.ts
export async function withRetry<T>(
  fn: () => Promise<T>,
  maxRetries = 3
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      await new Promise(r => setTimeout(r, 2000 * (i + 1)));
    }
  }
  throw new Error('Max retries exceeded');
}

// Usage
const content = await withRetry(() =>
  generateContent(keyword, config)
);
```

### Video Rendering Timeout

```typescript
// Increase timeout for longer videos
await renderMedia({
  composition,
  serveUrl: bundleLocation,
  codec: 'h264',
  outputLocation: outputPath,
  inputProps: composition.defaultProps,
  timeoutInMilliseconds: 120000, // 2 minutes
});
```

### Memory Issues with Large Content

```typescript
// Process in chunks
function chunkContent(content: string, chunkSize = 500) {
  const words = content.split(' ');
  const chunks = [];
  
  for (let i = 0; i < words.length; i += chunkSize) {
    chunks.push(words.slice(i, i + chunkSize).join(' '));
  }
  
  return chunks;
}
```

## Advanced Configuration

### Multi-language Support

```typescript
const languagePrompts = {
  en: 'Write in professional English',
  vi: 'Viết bằng tiếng Việt chuyên nghiệp',
  es: 'Escribe en español profesional',
};

const prompt = `${languagePrompts[config.language]}\n\n${basePrompt}`;
```

### Custom Video Templates

```typescript
// remotion/templates/SocialMedia.tsx
export const SocialMediaTemplate: React.FC = ({ title, stats }) => (
  <AbsoluteFill style={{ background: 'linear-gradient(...)' }}>
    {/* Custom branding, animations, transitions */}
  </AbsoluteFill>
);
```

This skill enables AI agents to orchestrate complex content workflows combining web scraping, AI writing, and video generation in a unified TypeScript pipeline.
