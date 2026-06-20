---
name: marketing-pipeline-ai-content-automation
description: Automated AI content pipeline for research, scriptwriting, Facebook posting, and video generation using Claude/OpenAI and Remotion
triggers:
  - automate content creation with AI research and video generation
  - set up AI content pipeline for social media
  - create automated marketing content workflow
  - generate videos from AI-written content
  - build AI-powered content automation system
  - automate Facebook posts with AI research
  - create content pipeline with Claude and OpenAI
  - set up automated video rendering for marketing
---

# Marketing Pipeline AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

Marketing Pipeline Share is a comprehensive TypeScript-based content automation system that handles the entire content lifecycle: from researching trending topics, generating scripts with AI (Claude/OpenAI), automatically posting to Facebook, to rendering videos with Remotion. It crawls real-time data from sources like TechCrunch, a16z, Twitter, and LinkedIn to create data-backed, trending content in multiple formats and languages.

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
# AI API Keys
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research & Data Crawling
RAPIDAPI_KEY=your_rapidapi_key

# Social Media
FACEBOOK_ACCESS_TOKEN=your_facebook_token
FACEBOOK_PAGE_ID=your_page_id

# Remotion Video Rendering
REMOTION_AWS_ACCESS_KEY_ID=your_aws_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret

# Next.js
NEXT_PUBLIC_API_URL=http://localhost:3000
```

## Running the Application

```bash
# Development mode
npm run dev

# Production build
npm run build
npm start

# Remotion video preview (if applicable)
npm run remotion:preview
```

## Core Architecture

The pipeline follows this workflow:

1. **Research Module**: Crawls trending topics from news sources
2. **AI Content Generation**: Uses Claude/OpenAI to create scripts in multiple formats
3. **Auto-posting**: Publishes content to Facebook pages
4. **Video Rendering**: Generates videos using Remotion from the written content

## Key API Endpoints

### 1. Research & Topic Discovery

```typescript
// pages/api/research/trending.ts
import type { NextApiRequest, NextApiResponse } from 'next';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  const { keyword, sources = ['techcrunch', 'a16z'] } = req.body;

  const response = await fetch('https://rapidapi.com/example/search', {
    headers: {
      'X-RapidAPI-Key': process.env.RAPIDAPI_KEY!,
    },
  });

  const trendingData = await response.json();

  res.status(200).json({
    keyword,
    insights: trendingData.articles,
    timestamp: new Date().toISOString(),
  });
}
```

### 2. AI Content Generation

```typescript
// lib/ai/generateContent.ts
import Anthropic from '@anthropic-ai/sdk';
import { OpenAI } from 'openai';

interface ContentOptions {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  researchData?: any[];
}

export async function generateWithClaude(options: ContentOptions) {
  const anthropic = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY,
  });

  const prompt = `
    Create a ${options.format} article about "${options.keyword}" in ${options.language}.
    Tone: ${options.tone}
    Research data: ${JSON.stringify(options.researchData)}
    
    Include:
    - Engaging headline
    - Data-backed insights
    - Actionable takeaways
    - SEO-optimized structure
  `;

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4000,
    messages: [{ role: 'user', content: prompt }],
  });

  return message.content[0].type === 'text' ? message.content[0].text : '';
}

export async function generateWithOpenAI(options: ContentOptions) {
  const openai = new OpenAI({
    apiKey: process.env.OPENAI_API_KEY,
  });

  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are a ${options.tone} content writer specializing in ${options.format} format.`,
      },
      {
        role: 'user',
        content: `Write about ${options.keyword} in ${options.language}. Research: ${JSON.stringify(options.researchData)}`,
      },
    ],
    temperature: 0.7,
  });

  return completion.choices[0].message.content || '';
}
```

### 3. Facebook Auto-Posting

```typescript
// lib/social/facebookPost.ts
interface FacebookPostOptions {
  pageId: string;
  message: string;
  link?: string;
  imageUrl?: string;
}

export async function postToFacebook(options: FacebookPostOptions) {
  const { pageId, message, link, imageUrl } = options;
  const accessToken = process.env.FACEBOOK_ACCESS_TOKEN;

  const endpoint = `https://graph.facebook.com/v18.0/${pageId}/feed`;

  const body: any = {
    message,
    access_token: accessToken,
  };

  if (link) body.link = link;
  if (imageUrl) body.picture = imageUrl;

  const response = await fetch(endpoint, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(body),
  });

  const result = await response.json();

  if (result.error) {
    throw new Error(`Facebook API Error: ${result.error.message}`);
  }

  return result;
}
```

### 4. Video Generation with Remotion

```typescript
// remotion/compositions/ContentVideo.tsx
import { AbsoluteFill, Sequence, useCurrentFrame } from 'remotion';

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

  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a1a' }}>
      <Sequence from={0} durationInFrames={60}>
        <AbsoluteFill
          style={{
            justifyContent: 'center',
            alignItems: 'center',
            fontSize: 60,
            color: 'white',
            fontWeight: 'bold',
            padding: 40,
            textAlign: 'center',
          }}
        >
          {title}
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
              alignItems: 'flex-start',
              fontSize: 40,
              color: 'white',
              padding: 60,
            }}
          >
            <div>✓ {point}</div>
          </AbsoluteFill>
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};
```

```typescript
// lib/video/renderVideo.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface RenderOptions {
  compositionId: string;
  inputProps: any;
  outputPath: string;
}

export async function renderContentVideo(options: RenderOptions) {
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: options.compositionId,
    inputProps: options.inputProps,
  });

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: options.outputPath,
    inputProps: options.inputProps,
  });

  return options.outputPath;
}
```

## Complete Workflow Example

```typescript
// pages/api/pipeline/execute.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { generateWithClaude } from '@/lib/ai/generateContent';
import { postToFacebook } from '@/lib/social/facebookPost';
import { renderContentVideo } from '@/lib/video/renderVideo';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  const { keyword, format, language, tone } = req.body;

  try {
    // Step 1: Research
    const researchResponse = await fetch(
      `${process.env.NEXT_PUBLIC_API_URL}/api/research/trending`,
      {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ keyword }),
      }
    );
    const researchData = await researchResponse.json();

    // Step 2: Generate Content
    const content = await generateWithClaude({
      keyword,
      format,
      language,
      tone,
      researchData: researchData.insights,
    });

    // Step 3: Extract key points for video
    const contentLines = content.split('\n').filter((line) => line.trim());
    const title = contentLines[0];
    const points = contentLines
      .slice(1)
      .filter((line) => line.startsWith('-'))
      .map((line) => line.substring(1).trim())
      .slice(0, 5);

    // Step 4: Render Video
    const videoPath = await renderContentVideo({
      compositionId: 'ContentVideo',
      inputProps: { title, points, duration: 30 },
      outputPath: `./public/videos/${keyword}-${Date.now()}.mp4`,
    });

    // Step 5: Post to Facebook
    const fbPost = await postToFacebook({
      pageId: process.env.FACEBOOK_PAGE_ID!,
      message: content,
      link: `${process.env.NEXT_PUBLIC_API_URL}${videoPath.replace('./public', '')}`,
    });

    res.status(200).json({
      success: true,
      content,
      videoPath,
      facebookPostId: fbPost.id,
    });
  } catch (error: any) {
    res.status(500).json({ error: error.message });
  }
}
```

## Frontend Integration

```typescript
// components/ContentPipeline.tsx
'use client';

import { useState } from 'react';

export default function ContentPipeline() {
  const [keyword, setKeyword] = useState('');
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState<any>(null);

  const handleExecute = async () => {
    setLoading(true);
    try {
      const response = await fetch('/api/pipeline/execute', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          keyword,
          format: 'toplist',
          language: 'vi',
          tone: 'expert',
        }),
      });
      const data = await response.json();
      setResult(data);
    } catch (error) {
      console.error('Pipeline error:', error);
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="p-8">
      <h1 className="text-3xl font-bold mb-4">AI Content Pipeline</h1>
      <div className="flex gap-4 mb-4">
        <input
          type="text"
          value={keyword}
          onChange={(e) => setKeyword(e.target.value)}
          placeholder="Enter keyword..."
          className="border px-4 py-2 rounded flex-1"
        />
        <button
          onClick={handleExecute}
          disabled={loading}
          className="bg-blue-500 text-white px-6 py-2 rounded disabled:opacity-50"
        >
          {loading ? 'Processing...' : 'Execute Pipeline'}
        </button>
      </div>
      {result && (
        <div className="bg-gray-50 p-4 rounded">
          <h2 className="font-bold mb-2">Result:</h2>
          <p className="mb-2">Content generated ✓</p>
          <p className="mb-2">Video: {result.videoPath}</p>
          <p>Facebook Post ID: {result.facebookPostId}</p>
        </div>
      )}
    </div>
  );
}
```

## Common Patterns

### Batch Content Generation

```typescript
// lib/batch/generateBatch.ts
export async function generateBatchContent(keywords: string[]) {
  const results = await Promise.all(
    keywords.map(async (keyword) => {
      const response = await fetch('/api/pipeline/execute', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ keyword }),
      });
      return response.json();
    })
  );

  return results;
}
```

### Scheduled Posting

```typescript
// lib/scheduler/schedulePost.ts
import cron from 'node-cron';

export function scheduleContentGeneration(
  schedule: string,
  keywords: string[]
) {
  cron.schedule(schedule, async () => {
    console.log('Running scheduled content generation...');
    for (const keyword of keywords) {
      await fetch('/api/pipeline/execute', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ keyword }),
      });
    }
  });
}
```

## Troubleshooting

### API Rate Limits

```typescript
// lib/utils/rateLimiter.ts
export async function withRetry(fn: () => Promise<any>, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error: any) {
      if (error.status === 429 && i < maxRetries - 1) {
        await new Promise((resolve) => setTimeout(resolve, 2000 * (i + 1)));
        continue;
      }
      throw error;
    }
  }
}
```

### Video Rendering Timeouts

Set longer timeouts for Remotion rendering:

```typescript
await renderMedia({
  composition,
  serveUrl: bundleLocation,
  codec: 'h264',
  outputLocation: options.outputPath,
  inputProps: options.inputProps,
  timeoutInMilliseconds: 120000, // 2 minutes
});
```

### Facebook Token Expiration

Implement token refresh logic:

```typescript
// lib/social/refreshFacebookToken.ts
export async function refreshFacebookToken() {
  const response = await fetch(
    `https://graph.facebook.com/oauth/access_token?` +
      `grant_type=fb_exchange_token&` +
      `client_id=${process.env.FACEBOOK_APP_ID}&` +
      `client_secret=${process.env.FACEBOOK_APP_SECRET}&` +
      `fb_exchange_token=${process.env.FACEBOOK_ACCESS_TOKEN}`
  );

  const data = await response.json();
  // Update your environment or database with new token
  return data.access_token;
}
```
