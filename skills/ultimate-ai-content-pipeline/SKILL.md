---
name: ultimate-ai-content-pipeline
description: Automate content creation from research to video generation using AI-powered pipeline with Claude, OpenAI, and Remotion
triggers:
  - how do I set up the AI content pipeline
  - generate content automatically with AI research
  - create videos from written content using remotion
  - automate content research and scriptwriting
  - use Claude and OpenAI for content generation
  - build automated marketing content workflow
  - crawl news and generate content from it
  - set up the ultimate content automation system
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to help developers use the Ultimate AI Content Pipeline - a comprehensive TypeScript-based system that automates content creation from initial research through video generation. The pipeline crawls news sources, generates content in multiple formats using Claude/OpenAI, and renders videos automatically using Remotion.

## What This Project Does

The Ultimate AI Content Pipeline is an end-to-end content automation system that:

1. **Auto-researches** by crawling real-time data from TechCrunch, a16z, Twitter/X, LinkedIn
2. **Generates content** in multiple formats (listicles, POV, case studies, how-tos) using Claude 3 and OpenAI
3. **Supports multi-language** output (English and Vietnamese) with customizable tone
4. **Renders videos** automatically using Remotion for social media platforms
5. **Optimizes for platforms** like Reels, TikTok, and Shorts

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
# AI Provider APIs
OPENAI_API_KEY=your_openai_key_here
ANTHROPIC_API_KEY=your_claude_key_here

# Research & Crawling
RAPIDAPI_KEY=your_rapidapi_key_here

# Optional: Database (if using)
DATABASE_URL=your_database_url_here

# Application Settings
NEXT_PUBLIC_APP_URL=http://localhost:3000
NODE_ENV=development
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app router pages
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI provider integrations
│   │   ├── crawler/     # Content crawling logic
│   │   ├── generators/  # Content generation modules
│   │   └── video/       # Remotion video rendering
│   └── utils/           # Utility functions
├── public/              # Static assets
├── remotion/            # Remotion video templates
└── package.json
```

## Running the Application

```bash
# Development mode
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Render video (if separate command)
npm run render
```

## Core API Usage

### 1. Content Research & Crawling

```typescript
// src/lib/crawler/research.ts
import axios from 'axios';

interface ResearchSource {
  title: string;
  url: string;
  content: string;
  publishedAt: Date;
}

export async function crawlLatestNews(
  keyword: string,
  sources: string[] = ['techcrunch', 'a16z', 'twitter']
): Promise<ResearchSource[]> {
  const results: ResearchSource[] = [];
  
  for (const source of sources) {
    const response = await axios.get(`https://api.rapidapi.com/news/${source}`, {
      headers: {
        'X-RapidAPI-Key': process.env.RAPIDAPI_KEY,
      },
      params: {
        q: keyword,
        timeframe: '24h',
      },
    });
    
    results.push(...response.data.articles.map((article: any) => ({
      title: article.title,
      url: article.url,
      content: article.content,
      publishedAt: new Date(article.publishedAt),
    })));
  }
  
  return results;
}
```

### 2. AI Content Generation with Claude

```typescript
// src/lib/ai/claude-generator.ts
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

interface ContentRequest {
  topic: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  researchData: string[];
}

export async function generateContentWithClaude(
  request: ContentRequest
): Promise<string> {
  const prompt = buildPrompt(request);
  
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
  
  return message.content[0].type === 'text' 
    ? message.content[0].text 
    : '';
}

function buildPrompt(request: ContentRequest): string {
  const formatInstructions = {
    'toplist': 'Create a numbered list article with clear points',
    'pov': 'Write from a personal perspective with unique insights',
    'case-study': 'Analyze with data, examples, and conclusions',
    'how-to': 'Provide step-by-step instructions',
  };
  
  return `
You are a professional content writer. Create a ${request.format} article about "${request.topic}".

Format: ${formatInstructions[request.format]}
Language: ${request.language === 'en' ? 'English' : 'Vietnamese'}
Tone: ${request.tone}

Research data:
${request.researchData.join('\n\n')}

Requirements:
- Use the latest data from research
- Include specific examples and statistics
- Make it engaging and actionable
- Optimize for social media sharing
  `.trim();
}
```

### 3. OpenAI Integration Alternative

```typescript
// src/lib/ai/openai-generator.ts
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

export async function generateContentWithGPT(
  request: ContentRequest
): Promise<string> {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: 'You are an expert content creator specializing in data-driven marketing content.',
      },
      {
        role: 'user',
        content: buildPrompt(request),
      },
    ],
    temperature: 0.7,
    max_tokens: 3000,
  });
  
  return completion.choices[0]?.message?.content || '';
}
```

### 4. Video Generation with Remotion

```typescript
// src/lib/video/render-content.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  title: string;
  keyPoints: string[];
  platform: 'reels' | 'tiktok' | 'shorts';
  duration: number;
}

export async function renderContentVideo(
  config: VideoConfig,
  outputPath: string
): Promise<string> {
  // Platform-specific dimensions
  const dimensions = {
    reels: { width: 1080, height: 1920 },
    tiktok: { width: 1080, height: 1920 },
    shorts: { width: 1080, height: 1920 },
  };
  
  const { width, height } = dimensions[config.platform];
  
  // Bundle the Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: (config) => config,
  });
  
  // Get composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: config,
  });
  
  // Render video
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: config,
  });
  
  return outputPath;
}
```

### 5. Remotion Video Component

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, Sequence, useCurrentFrame, useVideoConfig } from 'remotion';
import React from 'react';

interface ContentVideoProps {
  title: string;
  keyPoints: string[];
  duration: number;
}

export const ContentVideo: React.FC<ContentVideoProps> = ({
  title,
  keyPoints,
  duration,
}) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  
  const opacity = Math.min(1, frame / 30);
  
  return (
    <AbsoluteFill style={{ backgroundColor: '#000' }}>
      <Sequence from={0} durationInFrames={fps * 3}>
        <AbsoluteFill
          style={{
            justifyContent: 'center',
            alignItems: 'center',
            opacity,
          }}
        >
          <h1 style={{ 
            color: 'white', 
            fontSize: 60, 
            textAlign: 'center',
            padding: 40,
          }}>
            {title}
          </h1>
        </AbsoluteFill>
      </Sequence>
      
      {keyPoints.map((point, index) => (
        <Sequence
          key={index}
          from={fps * (3 + index * 4)}
          durationInFrames={fps * 4}
        >
          <AbsoluteFill
            style={{
              justifyContent: 'center',
              alignItems: 'center',
              backgroundColor: '#000',
            }}
          >
            <div style={{ 
              color: 'white', 
              fontSize: 40, 
              padding: 60,
              textAlign: 'center',
            }}>
              <div style={{ fontSize: 80, marginBottom: 20 }}>
                {index + 1}
              </div>
              <p>{point}</p>
            </div>
          </AbsoluteFill>
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};
```

## Complete Workflow Example

```typescript
// src/lib/pipeline/content-pipeline.ts
import { crawlLatestNews } from '../crawler/research';
import { generateContentWithClaude } from '../ai/claude-generator';
import { renderContentVideo } from '../video/render-content';

export async function runContentPipeline(
  keyword: string,
  format: 'toplist' | 'pov' | 'case-study' | 'how-to',
  language: 'en' | 'vi' = 'en'
) {
  console.log('🔍 Step 1: Researching latest news...');
  const researchData = await crawlLatestNews(keyword);
  
  console.log('✍️ Step 2: Generating content...');
  const content = await generateContentWithClaude({
    topic: keyword,
    format,
    language,
    tone: 'expert',
    researchData: researchData.map(r => `${r.title}\n${r.content}`),
  });
  
  console.log('🎬 Step 3: Rendering video...');
  const keyPoints = extractKeyPoints(content, 5);
  const videoPath = await renderContentVideo(
    {
      title: keyword,
      keyPoints,
      platform: 'reels',
      duration: 30,
    },
    `./output/${keyword.replace(/\s+/g, '-')}.mp4`
  );
  
  return {
    content,
    videoPath,
    sources: researchData.map(r => r.url),
  };
}

function extractKeyPoints(content: string, count: number): string[] {
  // Simple extraction - can be enhanced with AI
  const lines = content.split('\n').filter(line => 
    line.trim().match(/^[\d\-\*]\.?\s+/)
  );
  return lines.slice(0, count).map(line => 
    line.replace(/^[\d\-\*]\.?\s+/, '').trim()
  );
}
```

## API Route Example (Next.js)

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline/content-pipeline';

export async function POST(request: NextRequest) {
  try {
    const { keyword, format, language } = await request.json();
    
    if (!keyword) {
      return NextResponse.json(
        { error: 'Keyword is required' },
        { status: 400 }
      );
    }
    
    const result = await runContentPipeline(keyword, format, language);
    
    return NextResponse.json({
      success: true,
      data: result,
    });
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { error: 'Failed to generate content' },
      { status: 500 }
    );
  }
}
```

## Frontend Component Example

```typescript
// src/components/ContentGenerator.tsx
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
      const response = await fetch('/api/generate', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ keyword, format, language: 'en' }),
      });
      
      const data = await response.json();
      setResult(data.data);
    } catch (error) {
      console.error('Error:', error);
    } finally {
      setLoading(false);
    }
  };
  
  return (
    <div className="max-w-4xl mx-auto p-6">
      <h1 className="text-3xl font-bold mb-6">AI Content Generator</h1>
      
      <div className="space-y-4">
        <input
          type="text"
          placeholder="Enter keyword..."
          value={keyword}
          onChange={(e) => setKeyword(e.target.value)}
          className="w-full p-3 border rounded"
        />
        
        <select
          value={format}
          onChange={(e) => setFormat(e.target.value as any)}
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
          className="w-full bg-blue-500 text-white p-3 rounded disabled:opacity-50"
        >
          {loading ? 'Generating...' : 'Generate Content'}
        </button>
      </div>
      
      {result && (
        <div className="mt-8 space-y-4">
          <div className="p-4 bg-gray-50 rounded">
            <h3 className="font-bold mb-2">Generated Content:</h3>
            <div className="whitespace-pre-wrap">{result.content}</div>
          </div>
          
          <div className="p-4 bg-gray-50 rounded">
            <h3 className="font-bold mb-2">Video:</h3>
            <p>{result.videoPath}</p>
          </div>
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
  const results = [];
  
  for (const keyword of keywords) {
    const result = await runContentPipeline(keyword, 'toplist', 'en');
    results.push(result);
    
    // Rate limiting
    await new Promise(resolve => setTimeout(resolve, 2000));
  }
  
  return results;
}
```

### Multilingual Content

```typescript
async function generateMultilingualContent(keyword: string) {
  const [enContent, viContent] = await Promise.all([
    generateContentWithClaude({
      topic: keyword,
      format: 'toplist',
      language: 'en',
      tone: 'expert',
      researchData: [],
    }),
    generateContentWithClaude({
      topic: keyword,
      format: 'toplist',
      language: 'vi',
      tone: 'expert',
      researchData: [],
    }),
  ]);
  
  return { en: enContent, vi: viContent };
}
```

## Troubleshooting

### API Key Issues
- Ensure all API keys are set in `.env.local`
- Verify keys have sufficient credits/quota
- Check for typos in environment variable names

### Crawling Failures
```typescript
// Add retry logic
async function crawlWithRetry(keyword: string, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await crawlLatestNews(keyword);
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      await new Promise(r => setTimeout(r, 1000 * (i + 1)));
    }
  }
}
```

### Video Rendering Errors
- Ensure ffmpeg is installed: `npm install @remotion/renderer`
- Check output directory permissions
- Verify Remotion composition ID matches

### Memory Issues with Large Content
```typescript
// Process in chunks
async function generateLargeContent(keyword: string) {
  const researchData = await crawlLatestNews(keyword);
  const chunks = chunkArray(researchData, 5);
  
  const contents = [];
  for (const chunk of chunks) {
    const content = await generateContentWithClaude({
      topic: keyword,
      format: 'toplist',
      language: 'en',
      tone: 'expert',
      researchData: chunk.map(r => r.content),
    });
    contents.push(content);
  }
  
  return contents.join('\n\n');
}
```

This skill provides comprehensive coverage of the Ultimate AI Content Pipeline system, enabling AI coding agents to effectively assist developers in implementing automated content generation workflows.
