---
name: marketing-pipeline-ai-content-automation
description: Automated AI content pipeline for research, scriptwriting, social posting, and video generation using Claude, OpenAI, and Remotion
triggers:
  - automate content creation with AI research and video generation
  - build automated marketing pipeline with Claude and OpenAI
  - create AI-powered content workflow from research to video
  - set up automated social media content generation
  - generate videos from articles using Remotion and AI
  - automate content research and scriptwriting pipeline
  - build end-to-end AI content automation system
  - create automated marketing content with video rendering
---

# Marketing Pipeline AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI agents to use the Ultimate AI Content Pipeline - a comprehensive automation system that handles content creation from research through video generation. The pipeline automatically crawls news sources, generates multi-format content using Claude/OpenAI, and renders videos using Remotion.

## What This Project Does

The marketing-pipeline-share project provides:

- **Auto-Research**: Crawls TechCrunch, a16z, Twitter/X, LinkedIn for recent content (24h)
- **AI Content Generation**: Creates articles in multiple formats (toplist, POV, case study, how-to) using Claude 3 and OpenAI
- **Multi-language Support**: Generates content in both English and Vietnamese
- **Video Rendering**: Automatically creates infographics and short videos from content using Remotion
- **Social Media Automation**: Posts content automatically to pages/platforms
- **Content Pipeline Architecture**: Next.js frontend with API integrations for OpenAI, Anthropic, and RapidAPI

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
cp .env.example .env.local
```

## Configuration

Create a `.env.local` file with the following variables:

```bash
# AI API Keys
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# RapidAPI for content crawling
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_url

# Remotion Configuration
REMOTION_AWS_ACCESS_KEY_ID=your_aws_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret

# Next.js Configuration
NEXT_PUBLIC_API_URL=http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/              # Core utilities
│   │   ├── ai/           # AI integration (Claude, OpenAI)
│   │   ├── crawler/      # Content crawling logic
│   │   ├── video/        # Remotion video generation
│   │   └── utils/        # Helper functions
│   └── types/            # TypeScript type definitions
├── remotion/             # Remotion video templates
└── public/               # Static assets
```

## Core API Usage

### 1. Content Research & Crawling

```typescript
import { crawlRecentContent } from '@/lib/crawler';

interface CrawlResult {
  title: string;
  url: string;
  content: string;
  publishedAt: Date;
  source: string;
}

async function researchTopic(keyword: string): Promise<CrawlResult[]> {
  const sources = ['techcrunch', 'a16z', 'twitter', 'linkedin'];
  
  const results = await crawlRecentContent({
    keyword,
    sources,
    timeRange: '24h',
    limit: 20
  });
  
  return results;
}

// Usage
const insights = await researchTopic('AI marketing automation');
```

### 2. AI Content Generation with Claude

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

interface ContentGenerationParams {
  topic: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  researchData: CrawlResult[];
}

async function generateContent(params: ContentGenerationParams): Promise<string> {
  const { topic, format, language, tone, researchData } = params;
  
  const researchSummary = researchData.map(r => 
    `Source: ${r.source}\nTitle: ${r.title}\nContent: ${r.content.slice(0, 500)}`
  ).join('\n\n');
  
  const prompt = `You are a ${tone} content writer specializing in ${format} format.
  
Topic: ${topic}
Language: ${language}
Format: ${format}

Recent Research Data:
${researchSummary}

Create a comprehensive ${format} article that:
1. Uses the latest insights from the research data
2. Maintains a ${tone} tone
3. Is optimized for ${language === 'en' ? 'English' : 'Vietnamese'} audience
4. Includes data-backed claims and examples
5. Is engaging and actionable

Write the complete article now:`;

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4000,
    messages: [
      { role: 'user', content: prompt }
    ],
  });

  return message.content[0].type === 'text' ? message.content[0].text : '';
}

// Usage
const article = await generateContent({
  topic: 'AI Marketing Trends 2024',
  format: 'toplist',
  language: 'en',
  tone: 'expert',
  researchData: insights
});
```

### 3. Alternative: OpenAI Content Generation

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateContentOpenAI(params: ContentGenerationParams): Promise<string> {
  const { topic, format, language, tone, researchData } = params;
  
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are an expert content writer specializing in ${format} format with a ${tone} tone.`
      },
      {
        role: 'user',
        content: `Create a ${format} article about "${topic}" in ${language} using this research: ${JSON.stringify(researchData)}`
      }
    ],
    temperature: 0.7,
    max_tokens: 3000,
  });

  return completion.choices[0].message.content || '';
}
```

### 4. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoGenerationParams {
  article: string;
  title: string;
  outputPath: string;
  format: 'reels' | 'tiktok' | 'shorts'; // 9:16 aspect ratio
}

async function generateVideo(params: VideoGenerationParams): Promise<string> {
  const { article, title, outputPath, format } = params;
  
  // Extract key points from article for video
  const keyPoints = await extractKeyPoints(article);
  
  // Bundle Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.join(process.cwd(), 'remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  // Get composition
  const compositions = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title,
      keyPoints,
      format,
    },
  });

  // Render video
  await renderMedia({
    composition: compositions,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: {
      title,
      keyPoints,
      format,
    },
  });

  return outputPath;
}

async function extractKeyPoints(article: string): Promise<string[]> {
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 1000,
    messages: [
      {
        role: 'user',
        content: `Extract 5-7 key points from this article for a short video. Each point should be 10-15 words:\n\n${article}`
      }
    ],
  });

  const points = message.content[0].type === 'text' 
    ? message.content[0].text.split('\n').filter(p => p.trim())
    : [];
  
  return points;
}
```

### 5. Remotion Video Component Example

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig, interpolate } from 'remotion';
import React from 'react';

interface ContentVideoProps {
  title: string;
  keyPoints: string[];
  format: 'reels' | 'tiktok' | 'shorts';
}

export const ContentVideo: React.FC<ContentVideoProps> = ({ title, keyPoints, format }) => {
  const frame = useCurrentFrame();
  const { fps, durationInFrames } = useVideoConfig();
  
  const titleOpacity = interpolate(frame, [0, 30], [0, 1], {
    extrapolateRight: 'clamp',
  });

  const framesPerPoint = Math.floor(durationInFrames / keyPoints.length);

  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a1a' }}>
      {/* Title */}
      <div
        style={{
          position: 'absolute',
          top: 80,
          left: 40,
          right: 40,
          opacity: titleOpacity,
        }}
      >
        <h1 style={{ color: 'white', fontSize: 48, fontWeight: 'bold' }}>
          {title}
        </h1>
      </div>

      {/* Key Points */}
      {keyPoints.map((point, index) => {
        const startFrame = 60 + index * framesPerPoint;
        const endFrame = startFrame + framesPerPoint;
        
        const pointOpacity = interpolate(
          frame,
          [startFrame, startFrame + 20, endFrame - 20, endFrame],
          [0, 1, 1, 0],
          { extrapolateLeft: 'clamp', extrapolateRight: 'clamp' }
        );

        return (
          <div
            key={index}
            style={{
              position: 'absolute',
              top: '50%',
              left: 40,
              right: 40,
              transform: 'translateY(-50%)',
              opacity: pointOpacity,
            }}
          >
            <p style={{ color: '#00ff88', fontSize: 36, lineHeight: 1.6 }}>
              {point}
            </p>
          </div>
        );
      })}
    </AbsoluteFill>
  );
};
```

## Complete Pipeline Example

```typescript
import { researchTopic } from '@/lib/crawler';
import { generateContent } from '@/lib/ai/claude';
import { generateVideo } from '@/lib/video/remotion';
import { postToSocialMedia } from '@/lib/social';

interface PipelineConfig {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  languages: ('en' | 'vi')[];
  generateVideo: boolean;
  autoPost: boolean;
}

async function runContentPipeline(config: PipelineConfig) {
  const { keyword, format, languages, generateVideo: shouldGenerateVideo, autoPost } = config;
  
  console.log(`🔍 Step 1: Researching "${keyword}"...`);
  const researchData = await researchTopic(keyword);
  console.log(`✅ Found ${researchData.length} sources`);
  
  const results = [];
  
  for (const language of languages) {
    console.log(`📝 Step 2: Generating ${language} content...`);
    const article = await generateContent({
      topic: keyword,
      format,
      language,
      tone: 'expert',
      researchData,
    });
    
    console.log(`✅ ${language} article generated (${article.length} chars)`);
    
    let videoPath: string | null = null;
    
    if (shouldGenerateVideo) {
      console.log(`🎬 Step 3: Generating video for ${language}...`);
      videoPath = await generateVideo({
        article,
        title: keyword,
        outputPath: `./output/${keyword}-${language}-${Date.now()}.mp4`,
        format: 'reels',
      });
      console.log(`✅ Video generated: ${videoPath}`);
    }
    
    if (autoPost) {
      console.log(`📤 Step 4: Posting to social media...`);
      await postToSocialMedia({
        content: article,
        videoPath,
        platform: 'facebook',
        language,
      });
      console.log(`✅ Posted to social media`);
    }
    
    results.push({ language, article, videoPath });
  }
  
  return results;
}

// Usage
const results = await runContentPipeline({
  keyword: 'AI Marketing Automation 2024',
  format: 'toplist',
  languages: ['en', 'vi'],
  generateVideo: true,
  autoPost: false, // Set to true to auto-post
});
```

## API Routes (Next.js)

```typescript
// src/app/api/pipeline/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const { keyword, format, languages, generateVideo, autoPost } = body;
    
    if (!keyword || !format || !languages) {
      return NextResponse.json(
        { error: 'Missing required fields' },
        { status: 400 }
      );
    }
    
    const results = await runContentPipeline({
      keyword,
      format,
      languages,
      generateVideo: generateVideo ?? false,
      autoPost: autoPost ?? false,
    });
    
    return NextResponse.json({ success: true, results });
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { error: 'Pipeline execution failed' },
      { status: 500 }
    );
  }
}
```

## Frontend Integration

```typescript
// src/components/ContentPipelineForm.tsx
'use client';

import { useState } from 'react';

export default function ContentPipelineForm() {
  const [keyword, setKeyword] = useState('');
  const [format, setFormat] = useState<'toplist' | 'pov' | 'case-study' | 'how-to'>('toplist');
  const [languages, setLanguages] = useState<('en' | 'vi')[]>(['en']);
  const [loading, setLoading] = useState(false);
  const [results, setResults] = useState<any>(null);

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    setLoading(true);
    
    try {
      const response = await fetch('/api/pipeline', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          keyword,
          format,
          languages,
          generateVideo: true,
          autoPost: false,
        }),
      });
      
      const data = await response.json();
      setResults(data.results);
    } catch (error) {
      console.error('Error:', error);
    } finally {
      setLoading(false);
    }
  };

  return (
    <form onSubmit={handleSubmit} className="space-y-4">
      <div>
        <label className="block mb-2">Keyword</label>
        <input
          type="text"
          value={keyword}
          onChange={(e) => setKeyword(e.target.value)}
          className="w-full p-2 border rounded"
          placeholder="AI Marketing Automation"
        />
      </div>
      
      <div>
        <label className="block mb-2">Format</label>
        <select
          value={format}
          onChange={(e) => setFormat(e.target.value as any)}
          className="w-full p-2 border rounded"
        >
          <option value="toplist">Top List</option>
          <option value="pov">POV</option>
          <option value="case-study">Case Study</option>
          <option value="how-to">How-to</option>
        </select>
      </div>
      
      <button
        type="submit"
        disabled={loading}
        className="px-6 py-2 bg-blue-600 text-white rounded disabled:opacity-50"
      >
        {loading ? 'Processing...' : 'Generate Content'}
      </button>
      
      {results && (
        <div className="mt-4 p-4 bg-gray-100 rounded">
          <h3 className="font-bold mb-2">Results:</h3>
          <pre>{JSON.stringify(results, null, 2)}</pre>
        </div>
      )}
    </form>
  );
}
```

## Common Patterns

### Rate Limiting for AI APIs

```typescript
import pLimit from 'p-limit';

const limit = pLimit(3); // Max 3 concurrent AI requests

async function generateMultipleArticles(topics: string[]) {
  const promises = topics.map(topic =>
    limit(() => generateContent({
      topic,
      format: 'toplist',
      language: 'en',
      tone: 'expert',
      researchData: [],
    }))
  );
  
  return Promise.all(promises);
}
```

### Error Handling & Retry Logic

```typescript
async function generateContentWithRetry(
  params: ContentGenerationParams,
  maxRetries = 3
): Promise<string> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await generateContent(params);
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      console.log(`Retry ${i + 1}/${maxRetries}...`);
      await new Promise(resolve => setTimeout(resolve, 1000 * (i + 1)));
    }
  }
  throw new Error('Max retries exceeded');
}
```

## Troubleshooting

### API Key Issues

```typescript
// Validate environment variables at startup
function validateEnv() {
  const required = ['OPENAI_API_KEY', 'ANTHROPIC_API_KEY', 'RAPIDAPI_KEY'];
  const missing = required.filter(key => !process.env[key]);
  
  if (missing.length > 0) {
    throw new Error(`Missing environment variables: ${missing.join(', ')}`);
  }
}

validateEnv();
```

### Remotion Rendering Issues

If video rendering fails:

1. Check AWS credentials are correctly set
2. Ensure sufficient disk space
3. Verify Remotion version compatibility

```bash
# Update Remotion packages
npm update @remotion/bundler @remotion/renderer @remotion/cli
```

### Memory Issues with Large Content

```typescript
// Stream large responses instead of loading all at once
async function generateContentStream(params: ContentGenerationParams) {
  const stream = await anthropic.messages.stream({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4000,
    messages: [
      { role: 'user', content: 'Generate content...' }
    ],
  });

  let fullText = '';
  for await (const chunk of stream) {
    if (chunk.type === 'content_block_delta' && chunk.delta.type === 'text_delta') {
      fullText += chunk.delta.text;
    }
  }
  
  return fullText;
}
```

This skill provides comprehensive automation for content marketing pipelines with AI-powered research, generation, and video creation capabilities.
