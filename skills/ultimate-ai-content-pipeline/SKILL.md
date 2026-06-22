---
name: ultimate-ai-content-pipeline
description: AI-powered content automation pipeline for research, scriptwriting, and video generation using Claude, OpenAI, and Remotion
triggers:
  - how do I set up the AI content pipeline for automated content creation
  - configure content automation with Claude and OpenAI
  - generate automated video content from research data
  - create multilingual content with AI content pipeline
  - automate content research and video rendering
  - set up Remotion for automated video generation
  - build AI-powered marketing content workflow
  - configure auto-scan research for content creation
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

Ultimate AI Content Pipeline is a comprehensive TypeScript-based content automation system that transforms a single keyword into complete content pieces with automated research, scriptwriting, and video generation. It integrates Claude 3, OpenAI, and Remotion to create a closed-loop content production system.

**Key capabilities:**
- Auto-scan research from TechCrunch, a16z, Twitter, LinkedIn
- Multi-format content generation (Toplist, POV, Case Study, How-to)
- Multilingual support (English/Vietnamese)
- Automated video rendering with Remotion
- Next.js web interface

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

## Configuration

Create a `.env.local` file in the root directory:

```bash
# OpenAI Configuration
OPENAI_API_KEY=your_openai_key_here

# Anthropic Claude Configuration
ANTHROPIC_API_KEY=your_anthropic_key_here

# RapidAPI for content scraping
RAPIDAPI_KEY=your_rapidapi_key_here

# Remotion Configuration
REMOTION_LICENSE_KEY=your_remotion_license_key_here

# Database (if applicable)
DATABASE_URL=your_database_url_here

# Next.js Configuration
NEXT_PUBLIC_API_URL=http://localhost:3000/api
```

## Running the Application

```bash
# Development mode
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Remotion development
npm run remotion:dev

# Render video
npm run remotion:render
```

## Core Components & Usage

### 1. Content Research Service

```typescript
// lib/research/autoScan.ts
import { OpenAI } from 'openai';
import Anthropic from '@anthropic-ai/sdk';

interface ResearchConfig {
  keyword: string;
  sources: string[];
  timeRange: '24h' | '7d' | '30d';
  language: 'en' | 'vi';
}

export async function autoScanResearch(config: ResearchConfig) {
  const openai = new OpenAI({
    apiKey: process.env.OPENAI_API_KEY,
  });

  // Scrape recent content from sources
  const scrapedData = await scrapeContentSources({
    keyword: config.keyword,
    sources: config.sources,
    timeRange: config.timeRange,
  });

  // Analyze and extract insights using OpenAI
  const insights = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: 'You are a content research analyst. Extract key insights, trends, and data points from the provided content.',
      },
      {
        role: 'user',
        content: `Analyze this content and extract insights for keyword: ${config.keyword}\n\n${scrapedData}`,
      },
    ],
    temperature: 0.7,
  });

  return {
    insights: insights.choices[0].message.content,
    sources: scrapedData.sources,
    dataPoints: extractDataPoints(scrapedData),
  };
}
```

### 2. Content Generation with Claude

```typescript
// lib/generation/contentGenerator.ts
import Anthropic from '@anthropic-ai/sdk';

interface ContentConfig {
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  topic: string;
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  researchData: any;
}

export async function generateContent(config: ContentConfig) {
  const anthropic = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY,
  });

  const formatPrompts = {
    'toplist': 'Create a comprehensive top list article',
    'pov': 'Write a thought-provoking point of view piece',
    'case-study': 'Develop a detailed case study',
    'how-to': 'Create a step-by-step how-to guide',
  };

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [
      {
        role: 'user',
        content: `${formatPrompts[config.format]} about "${config.topic}" in ${config.language} with a ${config.tone} tone.
        
Use this research data:
${JSON.stringify(config.researchData, null, 2)}

Include:
- Compelling headline
- Introduction with hook
- Main content sections
- Data-backed insights
- Actionable takeaways`,
      },
    ],
  });

  return {
    content: message.content[0].text,
    metadata: {
      format: config.format,
      language: config.language,
      wordCount: message.content[0].text.split(' ').length,
    },
  };
}
```

### 3. Multilingual Content Creation

```typescript
// lib/generation/multilingualGenerator.ts
export async function generateMultilingualContent(
  topic: string,
  format: string,
  researchData: any
) {
  const [englishContent, vietnameseContent] = await Promise.all([
    generateContent({
      format: format as any,
      topic,
      language: 'en',
      tone: 'expert',
      researchData,
    }),
    generateContent({
      format: format as any,
      topic,
      language: 'vi',
      tone: 'friendly',
      researchData,
    }),
  ]);

  return {
    en: englishContent,
    vi: vietnameseContent,
  };
}
```

### 4. Video Generation with Remotion

```typescript
// remotion/compositions/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig } from 'remotion';
import React from 'react';

interface ContentVideoProps {
  title: string;
  points: string[];
  duration: number;
}

export const ContentVideo: React.FC<ContentVideoProps> = ({
  title,
  points,
  duration,
}) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  const opacity = Math.min(1, frame / (fps * 0.5));

  return (
    <AbsoluteFill
      style={{
        backgroundColor: '#1a1a1a',
        justifyContent: 'center',
        alignItems: 'center',
        fontFamily: 'Arial, sans-serif',
      }}
    >
      <div style={{ opacity }}>
        <h1
          style={{
            fontSize: 60,
            color: 'white',
            textAlign: 'center',
            marginBottom: 40,
          }}
        >
          {title}
        </h1>
        {points.map((point, index) => {
          const pointFrame = fps * (index + 1);
          const pointOpacity = frame > pointFrame ? 1 : 0;
          return (
            <p
              key={index}
              style={{
                fontSize: 30,
                color: '#00ff88',
                opacity: pointOpacity,
                transition: 'opacity 0.3s',
                margin: '20px 0',
              }}
            >
              • {point}
            </p>
          );
        })}
      </div>
    </AbsoluteFill>
  );
};
```

```typescript
// lib/video/renderVideo.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

export async function renderContentVideo(
  contentData: {
    title: string;
    points: string[];
  },
  outputPath: string
) {
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: contentData,
  });

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: contentData,
  });

  return outputPath;
}
```

### 5. Complete Pipeline API

```typescript
// app/api/pipeline/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { autoScanResearch } from '@/lib/research/autoScan';
import { generateMultilingualContent } from '@/lib/generation/multilingualGenerator';
import { renderContentVideo } from '@/lib/video/renderVideo';

export async function POST(request: NextRequest) {
  try {
    const { keyword, format, sources } = await request.json();

    // Step 1: Research
    const research = await autoScanResearch({
      keyword,
      sources: sources || ['techcrunch', 'a16z', 'twitter'],
      timeRange: '24h',
      language: 'en',
    });

    // Step 2: Generate content
    const content = await generateMultilingualContent(
      keyword,
      format,
      research
    );

    // Step 3: Extract key points for video
    const keyPoints = extractKeyPoints(content.en.content);

    // Step 4: Render video
    const videoPath = await renderContentVideo(
      {
        title: keyword,
        points: keyPoints.slice(0, 5),
      },
      `./public/videos/${Date.now()}.mp4`
    );

    return NextResponse.json({
      success: true,
      data: {
        research,
        content,
        videoPath,
      },
    });
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}

function extractKeyPoints(content: string): string[] {
  // Simple extraction - can be enhanced with AI
  const lines = content.split('\n').filter((line) => line.trim().length > 0);
  return lines.slice(0, 10);
}
```

### 6. Frontend Integration

```typescript
// app/components/ContentPipeline.tsx
'use client';

import { useState } from 'react';

export default function ContentPipeline() {
  const [keyword, setKeyword] = useState('');
  const [format, setFormat] = useState('toplist');
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState(null);

  const handleGenerate = async () => {
    setLoading(true);
    try {
      const response = await fetch('/api/pipeline', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          keyword,
          format,
          sources: ['techcrunch', 'a16z', 'twitter'],
        }),
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
    <div className="max-w-4xl mx-auto p-6">
      <h1 className="text-3xl font-bold mb-6">AI Content Pipeline</h1>

      <div className="space-y-4">
        <input
          type="text"
          value={keyword}
          onChange={(e) => setKeyword(e.target.value)}
          placeholder="Enter keyword..."
          className="w-full p-3 border rounded"
        />

        <select
          value={format}
          onChange={(e) => setFormat(e.target.value)}
          className="w-full p-3 border rounded"
        >
          <option value="toplist">Top List</option>
          <option value="pov">POV / Opinion</option>
          <option value="case-study">Case Study</option>
          <option value="how-to">How-to Guide</option>
        </select>

        <button
          onClick={handleGenerate}
          disabled={loading || !keyword}
          className="w-full bg-blue-600 text-white p-3 rounded hover:bg-blue-700 disabled:bg-gray-400"
        >
          {loading ? 'Generating...' : 'Generate Content'}
        </button>

        {result && (
          <div className="mt-6 p-4 bg-gray-50 rounded">
            <h2 className="text-xl font-bold mb-4">Results</h2>
            <div className="space-y-4">
              <div>
                <h3 className="font-semibold">English Content</h3>
                <p className="text-sm text-gray-600">
                  {result.data.content.en.content.substring(0, 200)}...
                </p>
              </div>
              <div>
                <h3 className="font-semibold">Vietnamese Content</h3>
                <p className="text-sm text-gray-600">
                  {result.data.content.vi.content.substring(0, 200)}...
                </p>
              </div>
              {result.data.videoPath && (
                <div>
                  <h3 className="font-semibold">Video</h3>
                  <video controls className="w-full">
                    <source src={result.data.videoPath} type="video/mp4" />
                  </video>
                </div>
              )}
            </div>
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
export async function batchGenerateContent(keywords: string[]) {
  const results = await Promise.allSettled(
    keywords.map(async (keyword) => {
      const research = await autoScanResearch({
        keyword,
        sources: ['techcrunch', 'a16z'],
        timeRange: '24h',
        language: 'en',
      });

      return await generateContent({
        format: 'toplist',
        topic: keyword,
        language: 'en',
        tone: 'expert',
        researchData: research,
      });
    })
  );

  return results;
}
```

### Custom Video Templates

```typescript
// remotion/compositions/CustomTemplate.tsx
export const CustomBrandedVideo: React.FC<{
  content: any;
  brandColors: { primary: string; secondary: string };
}> = ({ content, brandColors }) => {
  // Implement custom branded video template
  return (
    <AbsoluteFill
      style={{
        background: `linear-gradient(135deg, ${brandColors.primary}, ${brandColors.secondary})`,
      }}
    >
      {/* Custom video layout */}
    </AbsoluteFill>
  );
};
```

## Troubleshooting

### API Rate Limits

```typescript
// lib/utils/rateLimiter.ts
import pLimit from 'p-limit';

const limit = pLimit(5); // 5 concurrent requests max

export async function rateLimitedRequests<T>(
  requests: (() => Promise<T>)[]
): Promise<T[]> {
  return Promise.all(requests.map((request) => limit(request)));
}
```

### Error Handling

```typescript
export async function safeGenerateContent(config: ContentConfig) {
  try {
    return await generateContent(config);
  } catch (error) {
    if (error.status === 429) {
      // Rate limit - retry after delay
      await new Promise((resolve) => setTimeout(resolve, 5000));
      return await generateContent(config);
    }
    throw error;
  }
}
```

### Video Rendering Issues

- Ensure Remotion license key is set: `REMOTION_LICENSE_KEY`
- Check ffmpeg installation: `ffmpeg -version`
- Verify output directory permissions
- Use lower quality settings for faster rendering: `quality: 50`

### Memory Management for Large Batches

```typescript
async function processInChunks<T>(
  items: T[],
  chunkSize: number,
  processor: (item: T) => Promise<any>
) {
  const results = [];
  for (let i = 0; i < items.length; i += chunkSize) {
    const chunk = items.slice(i, i + chunkSize);
    const chunkResults = await Promise.all(chunk.map(processor));
    results.push(...chunkResults);
  }
  return results;
}
```
