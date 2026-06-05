---
name: ultimate-ai-content-pipeline
description: Automate content creation from research to video generation using AI (Claude/OpenAI) and Remotion
triggers:
  - how do I set up the AI content pipeline
  - generate automated content with research and video
  - create content automatically with Claude and OpenAI
  - use Remotion to render videos from articles
  - automate content research and scripting
  - set up marketing content automation pipeline
  - crawl news and generate content with AI
  - build automated video content from text
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This project is a complete content automation system that handles research (crawling news sources), content generation (using Claude/OpenAI), and video rendering (using Remotion). It transforms a single keyword into multi-format content including articles, scripts, and videos optimized for various platforms.

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
# AI APIs
OPENAI_API_KEY=your_openai_key_here
ANTHROPIC_API_KEY=your_anthropic_key_here

# Research/Crawling APIs
RAPIDAPI_KEY=your_rapidapi_key_here

# Optional: Remotion rendering
REMOTION_LICENSE_KEY=your_remotion_license_here

# Database (if using)
DATABASE_URL=your_database_connection_string

# Next.js
NEXT_PUBLIC_API_URL=http://localhost:3000
```

## Development Setup

```bash
# Run the development server
npm run dev

# Build for production
npm run build

# Start production server
npm start
```

The application will be available at `http://localhost:3000`.

## Core Architecture

### 1. Research Module (Content Crawling)

The system automatically crawls news sources to gather fresh, real-time data:

```typescript
// lib/research/crawler.ts
import axios from 'axios';

interface NewsSource {
  url: string;
  selector: string;
  source: string;
}

export async function crawlNews(keyword: string): Promise<ArticleData[]> {
  const sources: NewsSource[] = [
    { url: 'https://techcrunch.com', selector: 'article', source: 'TechCrunch' },
    // Add more sources
  ];

  const articles: ArticleData[] = [];
  
  for (const source of sources) {
    try {
      const response = await axios.get(`${source.url}/search?q=${keyword}`, {
        headers: {
          'X-RapidAPI-Key': process.env.RAPIDAPI_KEY,
        }
      });
      
      const parsed = parseArticles(response.data, source);
      articles.push(...parsed);
    } catch (error) {
      console.error(`Failed to crawl ${source.source}:`, error);
    }
  }

  return articles;
}

interface ArticleData {
  title: string;
  content: string;
  source: string;
  publishedAt: Date;
  url: string;
}
```

### 2. AI Content Generation

Generate content using Claude or OpenAI based on researched data:

```typescript
// lib/ai/contentGenerator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

interface ContentRequest {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  researchData: ArticleData[];
}

export async function generateContent(request: ContentRequest): Promise<string> {
  const prompt = buildPrompt(request);

  // Using Claude
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [
      { role: 'user', content: prompt }
    ],
  });

  return message.content[0].type === 'text' ? message.content[0].text : '';
}

export async function generateContentOpenAI(request: ContentRequest): Promise<string> {
  const prompt = buildPrompt(request);

  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      { role: 'system', content: 'You are an expert content creator specializing in data-backed articles.' },
      { role: 'user', content: prompt }
    ],
    temperature: 0.7,
  });

  return completion.choices[0].message.content || '';
}

function buildPrompt(request: ContentRequest): string {
  const researchSummary = request.researchData
    .map(article => `- ${article.title} (${article.source}): ${article.content.substring(0, 200)}...`)
    .join('\n');

  return `
Create a ${request.format} article about "${request.keyword}" in ${request.language === 'en' ? 'English' : 'Vietnamese'}.
Tone: ${request.tone}

Research data from the last 24 hours:
${researchSummary}

Requirements:
- Use data-backed insights from the research
- Include specific examples and statistics
- Structure according to ${request.format} format
- Write in a ${request.tone} tone
- Make it engaging and actionable
`;
}
```

### 3. Video Generation with Remotion

Transform content into videos using Remotion:

```typescript
// remotion/VideoComposition.tsx
import React from 'react';
import { AbsoluteFill, Sequence, useCurrentFrame, interpolate } from 'remotion';

interface VideoProps {
  title: string;
  points: string[];
  duration: number;
}

export const ContentVideo: React.FC<VideoProps> = ({ title, points, duration }) => {
  const frame = useCurrentFrame();
  
  const titleOpacity = interpolate(frame, [0, 30], [0, 1], {
    extrapolateRight: 'clamp',
  });

  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a2e' }}>
      <Sequence from={0} durationInFrames={60}>
        <AbsoluteFill style={{ 
          justifyContent: 'center', 
          alignItems: 'center',
          opacity: titleOpacity 
        }}>
          <h1 style={{ color: 'white', fontSize: 60, textAlign: 'center', padding: 40 }}>
            {title}
          </h1>
        </AbsoluteFill>
      </Sequence>

      {points.map((point, index) => (
        <Sequence key={index} from={60 + index * 90} durationInFrames={90}>
          <PointSlide point={point} index={index + 1} />
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};

const PointSlide: React.FC<{ point: string; index: number }> = ({ point, index }) => {
  const frame = useCurrentFrame();
  const opacity = interpolate(frame, [0, 20], [0, 1]);
  
  return (
    <AbsoluteFill style={{ 
      justifyContent: 'center', 
      alignItems: 'center', 
      opacity,
      padding: 60 
    }}>
      <div style={{ color: '#00ff88', fontSize: 40, fontWeight: 'bold', marginBottom: 20 }}>
        #{index}
      </div>
      <p style={{ color: 'white', fontSize: 32, textAlign: 'center', lineHeight: 1.5 }}>
        {point}
      </p>
    </AbsoluteFill>
  );
};
```

Render the video:

```typescript
// lib/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

export async function renderContentVideo(
  title: string,
  points: string[],
  outputPath: string
): Promise<string> {
  const compositionId = 'ContentVideo';
  const bundleLocation = await bundle(path.join(process.cwd(), 'remotion/index.ts'));
  
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: { title, points, duration: 300 },
  });

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: { title, points, duration: 300 },
  });

  return outputPath;
}
```

## API Routes (Next.js)

### Content Generation Endpoint

```typescript
// pages/api/generate-content.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { crawlNews } from '@/lib/research/crawler';
import { generateContent } from '@/lib/ai/contentGenerator';
import { renderContentVideo } from '@/lib/video/renderer';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  try {
    const { keyword, format, language, tone, includeVideo } = req.body;

    // Step 1: Research
    const researchData = await crawlNews(keyword);

    // Step 2: Generate content
    const content = await generateContent({
      keyword,
      format,
      language,
      tone,
      researchData,
    });

    // Step 3: Generate video (optional)
    let videoUrl = null;
    if (includeVideo) {
      const points = extractKeyPoints(content);
      const videoPath = `./public/videos/${Date.now()}.mp4`;
      await renderContentVideo(keyword, points, videoPath);
      videoUrl = videoPath.replace('./public', '');
    }

    res.status(200).json({
      content,
      videoUrl,
      research: researchData.map(r => ({
        title: r.title,
        source: r.source,
        url: r.url,
      })),
    });
  } catch (error) {
    console.error('Content generation error:', error);
    res.status(500).json({ error: 'Failed to generate content' });
  }
}

function extractKeyPoints(content: string): string[] {
  // Simple extraction - improve based on your format
  const lines = content.split('\n').filter(line => 
    line.trim().match(/^[\d\-\*]/) || line.includes('##')
  );
  return lines.slice(0, 5).map(line => line.replace(/^[\d\-\*#\s]+/, ''));
}
```

## Frontend Integration

```typescript
// components/ContentGenerator.tsx
'use client';

import { useState } from 'react';

export default function ContentGenerator() {
  const [keyword, setKeyword] = useState('');
  const [format, setFormat] = useState<'toplist' | 'pov' | 'case-study' | 'how-to'>('toplist');
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState<any>(null);

  const handleGenerate = async () => {
    setLoading(true);
    try {
      const response = await fetch('/api/generate-content', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          keyword,
          format,
          language: 'vi',
          tone: 'expert',
          includeVideo: true,
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
    <div className="max-w-4xl mx-auto p-6">
      <h1 className="text-3xl font-bold mb-6">AI Content Pipeline</h1>
      
      <div className="space-y-4">
        <input
          type="text"
          value={keyword}
          onChange={(e) => setKeyword(e.target.value)}
          placeholder="Enter keyword..."
          className="w-full px-4 py-2 border rounded"
        />

        <select 
          value={format} 
          onChange={(e) => setFormat(e.target.value as any)}
          className="w-full px-4 py-2 border rounded"
        >
          <option value="toplist">Top List</option>
          <option value="pov">Point of View</option>
          <option value="case-study">Case Study</option>
          <option value="how-to">How-To Guide</option>
        </select>

        <button
          onClick={handleGenerate}
          disabled={loading || !keyword}
          className="w-full bg-blue-600 text-white py-3 rounded disabled:opacity-50"
        >
          {loading ? 'Generating...' : 'Generate Content'}
        </button>
      </div>

      {result && (
        <div className="mt-8 space-y-4">
          <div className="border rounded p-4">
            <h2 className="text-xl font-bold mb-2">Generated Content</h2>
            <div className="prose" dangerouslySetInnerHTML={{ __html: result.content }} />
          </div>

          {result.videoUrl && (
            <div className="border rounded p-4">
              <h2 className="text-xl font-bold mb-2">Generated Video</h2>
              <video controls src={result.videoUrl} className="w-full" />
            </div>
          )}

          <div className="border rounded p-4">
            <h2 className="text-xl font-bold mb-2">Research Sources</h2>
            <ul className="space-y-2">
              {result.research.map((item: any, i: number) => (
                <li key={i}>
                  <a href={item.url} target="_blank" rel="noopener" className="text-blue-600">
                    {item.title} ({item.source})
                  </a>
                </li>
              ))}
            </ul>
          </div>
        </div>
      )}
    </div>
  );
}
```

## Common Patterns

### Multi-Language Content Generation

```typescript
export async function generateBilingual(keyword: string, researchData: ArticleData[]) {
  const [englishContent, vietnameseContent] = await Promise.all([
    generateContent({
      keyword,
      format: 'toplist',
      language: 'en',
      tone: 'expert',
      researchData,
    }),
    generateContent({
      keyword,
      format: 'toplist',
      language: 'vi',
      tone: 'expert',
      researchData,
    }),
  ]);

  return { en: englishContent, vi: vietnameseContent };
}
```

### Batch Content Generation

```typescript
export async function generateBatch(keywords: string[]) {
  const results = await Promise.all(
    keywords.map(async (keyword) => {
      const research = await crawlNews(keyword);
      const content = await generateContent({
        keyword,
        format: 'toplist',
        language: 'vi',
        tone: 'expert',
        researchData: research,
      });
      return { keyword, content };
    })
  );

  return results;
}
```

## Troubleshooting

### Rate Limiting Issues

If you hit API rate limits, implement queuing:

```typescript
import pQueue from 'p-queue';

const queue = new pQueue({ concurrency: 2, interval: 1000, intervalCap: 2 });

export async function queuedGenerate(request: ContentRequest) {
  return queue.add(() => generateContent(request));
}
```

### Video Rendering Fails

Check Remotion configuration:

```bash
# Ensure ffmpeg is installed
# macOS
brew install ffmpeg

# Ubuntu/Debian
sudo apt-get install ffmpeg

# Test Remotion
npx remotion preview
```

### Memory Issues with Large Content

Stream responses instead of loading everything in memory:

```typescript
export async function streamContent(request: ContentRequest): AsyncGenerator<string> {
  const stream = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [{ role: 'user', content: buildPrompt(request) }],
    stream: true,
  });

  for await (const chunk of stream) {
    yield chunk.choices[0]?.delta?.content || '';
  }
}
```

### API Key Errors

Always validate environment variables on startup:

```typescript
// lib/config.ts
export function validateConfig() {
  const required = ['OPENAI_API_KEY', 'ANTHROPIC_API_KEY', 'RAPIDAPI_KEY'];
  const missing = required.filter(key => !process.env[key]);
  
  if (missing.length > 0) {
    throw new Error(`Missing required environment variables: ${missing.join(', ')}`);
  }
}
```
