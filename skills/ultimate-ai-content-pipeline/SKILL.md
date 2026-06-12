---
name: ultimate-ai-content-pipeline
description: Automated Vietnamese/English content creation pipeline with AI research, script generation, and video rendering using Claude/OpenAI and Remotion
triggers:
  - how do I set up the AI content pipeline for automated marketing
  - help me generate content with AI research and video rendering
  - configure Claude or OpenAI for automated content creation
  - create marketing content from research to video automatically
  - use the content pipeline to generate multilingual posts
  - automate content creation with AI crawling and video generation
  - set up Remotion video rendering for marketing content
  - integrate Claude API for Vietnamese and English content generation
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

Ultimate AI Content Pipeline is a comprehensive TypeScript-based system that automates the entire content creation workflow: from researching trending topics (crawling TechCrunch, a16z, Twitter/X, LinkedIn), to generating AI-written articles in multiple formats (toplist, POV, case study, how-to), to automatically rendering videos and graphics using Remotion. It supports both Vietnamese and English content generation using Claude 3 and OpenAI APIs.

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
# AI APIs
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs (RapidAPI)
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Next.js
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── app/                    # Next.js app directory
├── components/             # React components
├── lib/
│   ├── ai/                # AI integration (Claude, OpenAI)
│   ├── crawler/           # Content crawling modules
│   ├── research/          # Research automation
│   └── video/             # Remotion video rendering
├── public/                # Static assets
└── remotion/              # Remotion video templates
```

## Core Modules

### 1. AI Content Generation

#### Using Claude API

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

interface ContentRequest {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'vi' | 'en';
  tone: 'expert' | 'friendly' | 'humorous';
  researchData: string;
}

async function generateContent(request: ContentRequest): Promise<string> {
  const prompt = `
You are an expert content writer. Create a ${request.format} article about "${request.keyword}" in ${request.language === 'vi' ? 'Vietnamese' : 'English'}.

Tone: ${request.tone}
Research Data: ${request.researchData}

Generate a comprehensive, engaging article with:
- Catchy headline
- Introduction hook
- 5-7 main points with data-backed insights
- Conclusion with call-to-action
`;

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [
      {
        role: 'user',
        content: prompt,
      },
    ],
  });

  return message.content[0].type === 'text' ? message.content[0].text : '';
}
```

#### Using OpenAI API

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
        content: `You are an expert ${request.tone} content writer specializing in ${request.format} articles.`,
      },
      {
        role: 'user',
        content: `Create a ${request.language} article about "${request.keyword}" using this research: ${request.researchData}`,
      },
    ],
    temperature: 0.7,
    max_tokens: 3000,
  });

  return completion.choices[0].message.content || '';
}
```

### 2. Auto-Research & Crawling

```typescript
import axios from 'axios';

interface NewsSource {
  name: string;
  url: string;
  selector?: string;
}

const NEWS_SOURCES: NewsSource[] = [
  { name: 'TechCrunch', url: 'https://techcrunch.com' },
  { name: 'a16z', url: 'https://a16z.com/blog' },
];

async function crawlRecentNews(keyword: string, hours: number = 24): Promise<string[]> {
  const options = {
    method: 'GET',
    url: 'https://google-news13.p.rapidapi.com/search',
    params: {
      keyword: keyword,
      lr: 'en-US',
    },
    headers: {
      'X-RapidAPI-Key': process.env.RAPIDAPI_KEY,
      'X-RapidAPI-Host': 'google-news13.p.rapidapi.com',
    },
  };

  try {
    const response = await axios.request(options);
    const articles = response.data.items || [];
    
    // Filter by time and extract relevant data
    const recentArticles = articles
      .filter((article: any) => {
        const publishTime = new Date(article.timestamp);
        const hoursAgo = (Date.now() - publishTime.getTime()) / (1000 * 60 * 60);
        return hoursAgo <= hours;
      })
      .map((article: any) => ({
        title: article.title,
        snippet: article.snippet,
        source: article.source,
        url: article.url,
      }));

    return recentArticles;
  } catch (error) {
    console.error('Crawling error:', error);
    return [];
  }
}

async function synthesizeResearch(keyword: string): Promise<string> {
  const news = await crawlRecentNews(keyword, 24);
  
  const researchSummary = news
    .map((item: any) => `**${item.source}**: ${item.title}\n${item.snippet}`)
    .join('\n\n');

  return `Recent research on "${keyword}" (last 24h):\n\n${researchSummary}`;
}
```

### 3. Remotion Video Generation

#### Video Composition Setup

```typescript
// remotion/Composition.tsx
import { Composition } from 'remotion';
import { ContentVideo } from './ContentVideo';

export const RemotionRoot: React.FC = () => {
  return (
    <>
      <Composition
        id="ContentVideo"
        component={ContentVideo}
        durationInFrames={300}
        fps={30}
        width={1080}
        height={1920}
        defaultProps={{
          title: 'Your Title',
          points: [],
          language: 'vi',
        }}
      />
    </>
  );
};
```

#### Video Template Component

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, interpolate, Sequence } from 'remotion';
import React from 'react';

interface ContentVideoProps {
  title: string;
  points: string[];
  language: 'vi' | 'en';
}

export const ContentVideo: React.FC<ContentVideoProps> = ({ title, points, language }) => {
  const frame = useCurrentFrame();
  
  const titleOpacity = interpolate(frame, [0, 30], [0, 1], {
    extrapolateRight: 'clamp',
  });

  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a1a' }}>
      <Sequence from={0} durationInFrames={60}>
        <AbsoluteFill
          style={{
            justifyContent: 'center',
            alignItems: 'center',
            opacity: titleOpacity,
          }}
        >
          <h1
            style={{
              fontSize: 80,
              color: 'white',
              textAlign: 'center',
              fontWeight: 'bold',
              padding: '0 100px',
            }}
          >
            {title}
          </h1>
        </AbsoluteFill>
      </Sequence>

      {points.map((point, index) => (
        <Sequence key={index} from={60 + index * 40} durationInFrames={40}>
          <AbsoluteFill
            style={{
              justifyContent: 'center',
              alignItems: 'center',
              padding: '0 80px',
            }}
          >
            <div
              style={{
                fontSize: 50,
                color: 'white',
                backgroundColor: 'rgba(255,255,255,0.1)',
                padding: '40px',
                borderRadius: '20px',
              }}
            >
              {point}
            </div>
          </AbsoluteFill>
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};
```

#### Video Rendering

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoRenderParams {
  title: string;
  points: string[];
  language: 'vi' | 'en';
  outputPath: string;
}

async function renderContentVideo(params: VideoRenderParams): Promise<string> {
  const compositionId = 'ContentVideo';
  
  // Bundle the Remotion project
  const bundleLocation = await bundle(
    path.join(process.cwd(), 'remotion/index.ts')
  );

  // Select composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: {
      title: params.title,
      points: params.points,
      language: params.language,
    },
  });

  // Render video
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: params.outputPath,
    inputProps: {
      title: params.title,
      points: params.points,
      language: params.language,
    },
  });

  return params.outputPath;
}
```

### 4. Complete Pipeline Integration

```typescript
// lib/pipeline.ts
interface PipelineConfig {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  languages: ('vi' | 'en')[];
  tone: 'expert' | 'friendly' | 'humorous';
  generateVideo: boolean;
}

async function runContentPipeline(config: PipelineConfig) {
  console.log(`Starting pipeline for keyword: ${config.keyword}`);

  // Step 1: Research
  console.log('Step 1: Researching...');
  const researchData = await synthesizeResearch(config.keyword);

  // Step 2: Generate content for each language
  const contents: Record<string, string> = {};
  
  for (const language of config.languages) {
    console.log(`Step 2: Generating ${language} content...`);
    
    const content = await generateContent({
      keyword: config.keyword,
      format: config.format,
      language,
      tone: config.tone,
      researchData,
    });
    
    contents[language] = content;
  }

  // Step 3: Generate video (if requested)
  let videoPath: string | null = null;
  
  if (config.generateVideo) {
    console.log('Step 3: Rendering video...');
    
    // Extract key points from content
    const points = extractKeyPoints(contents['vi'] || contents['en']);
    
    videoPath = await renderContentVideo({
      title: config.keyword,
      points: points.slice(0, 5),
      language: config.languages[0],
      outputPath: `./output/video-${Date.now()}.mp4`,
    });
  }

  return {
    research: researchData,
    contents,
    videoPath,
  };
}

function extractKeyPoints(content: string): string[] {
  // Simple extraction - split by numbered lists or bullet points
  const lines = content.split('\n');
  const points: string[] = [];
  
  for (const line of lines) {
    if (/^\d+\.|^-|^•/.test(line.trim())) {
      const cleaned = line.replace(/^\d+\.|^-|^•/, '').trim();
      if (cleaned.length > 0) {
        points.push(cleaned);
      }
    }
  }
  
  return points;
}
```

### 5. Next.js API Route Example

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    
    const result = await runContentPipeline({
      keyword: body.keyword,
      format: body.format || 'toplist',
      languages: body.languages || ['vi', 'en'],
      tone: body.tone || 'friendly',
      generateVideo: body.generateVideo || false,
    });

    return NextResponse.json({
      success: true,
      data: result,
    });
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { success: false, error: 'Pipeline execution failed' },
      { status: 500 }
    );
  }
}
```

### 6. Frontend Component Example

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
          languages: ['vi', 'en'],
          tone: 'friendly',
          generateVideo: true,
        }),
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
    <div className="p-8">
      <h1 className="text-3xl font-bold mb-6">AI Content Pipeline</h1>
      
      <div className="mb-4">
        <input
          type="text"
          value={keyword}
          onChange={(e) => setKeyword(e.target.value)}
          placeholder="Enter keyword..."
          className="w-full p-3 border rounded"
        />
      </div>

      <button
        onClick={handleGenerate}
        disabled={loading || !keyword}
        className="bg-blue-600 text-white px-6 py-3 rounded disabled:opacity-50"
      >
        {loading ? 'Generating...' : 'Generate Content'}
      </button>

      {result && (
        <div className="mt-8">
          <h2 className="text-2xl font-bold mb-4">Results</h2>
          
          <div className="mb-6">
            <h3 className="text-xl font-semibold mb-2">Vietnamese Content</h3>
            <div className="bg-gray-100 p-4 rounded whitespace-pre-wrap">
              {result.contents.vi}
            </div>
          </div>

          <div className="mb-6">
            <h3 className="text-xl font-semibold mb-2">English Content</h3>
            <div className="bg-gray-100 p-4 rounded whitespace-pre-wrap">
              {result.contents.en}
            </div>
          </div>

          {result.videoPath && (
            <div>
              <h3 className="text-xl font-semibold mb-2">Video</h3>
              <p>Video generated: {result.videoPath}</p>
            </div>
          )}
        </div>
      )}
    </div>
  );
}
```

## Common Patterns

### Batch Content Generation

```typescript
async function batchGenerateContent(keywords: string[]) {
  const results = await Promise.all(
    keywords.map(keyword =>
      runContentPipeline({
        keyword,
        format: 'toplist',
        languages: ['vi', 'en'],
        tone: 'friendly',
        generateVideo: false,
      })
    )
  );

  return results;
}
```

### Scheduled Content Pipeline

```typescript
// Using node-cron or similar
import cron from 'node-cron';

// Run pipeline daily at 9 AM
cron.schedule('0 9 * * *', async () => {
  const trendingKeywords = await fetchTrendingKeywords();
  
  for (const keyword of trendingKeywords.slice(0, 3)) {
    await runContentPipeline({
      keyword,
      format: 'toplist',
      languages: ['vi'],
      tone: 'expert',
      generateVideo: true,
    });
  }
});
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement rate limiting
import pLimit from 'p-limit';

const limit = pLimit(2); // Max 2 concurrent requests

async function generateMultipleContents(requests: ContentRequest[]) {
  return Promise.all(
    requests.map(req => limit(() => generateContent(req)))
  );
}
```

### Video Rendering Errors

```typescript
// Add error handling and retry logic
async function safeRenderVideo(params: VideoRenderParams, retries = 3) {
  for (let i = 0; i < retries; i++) {
    try {
      return await renderContentVideo(params);
    } catch (error) {
      console.error(`Render attempt ${i + 1} failed:`, error);
      if (i === retries - 1) throw error;
      await new Promise(resolve => setTimeout(resolve, 5000));
    }
  }
}
```

### Memory Issues with Large Content

```typescript
// Stream large content processing
import { pipeline } from 'stream/promises';

async function processLargeContent(content: string) {
  const chunks = content.match(/.{1,5000}/g) || [];
  const processed: string[] = [];
  
  for (const chunk of chunks) {
    const result = await generateContent({
      keyword: chunk,
      format: 'pov',
      language: 'vi',
      tone: 'friendly',
      researchData: '',
    });
    processed.push(result);
  }
  
  return processed.join('\n\n');
}
```

## Development

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Render Remotion video (standalone)
npx remotion render ContentVideo output.mp4
```
