---
name: marketing-pipeline-share-ai-content
description: Automated AI content pipeline for research, script generation, video rendering and multi-platform publishing
triggers:
  - automate my content creation pipeline
  - generate videos from blog posts automatically
  - research and write articles with AI
  - create content from keyword to video
  - build an AI marketing content system
  - scrape news and generate content automatically
  - set up automated content workflow
  - render videos from written content
---

# Marketing Pipeline Share - AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI agents to work with **marketing-pipeline-share**, an end-to-end AI content automation system that handles research (web scraping), content generation (Claude/OpenAI), and video rendering (Remotion). It transforms a single keyword into complete multi-format content including articles, social posts, and videos.

## What This Project Does

Marketing Pipeline Share is a TypeScript-based Next.js application that:

- **Auto-scans** news sources (TechCrunch, a16z, X/Twitter, LinkedIn) for fresh data
- **Generates content** in multiple formats (toplist, POV, case study, how-to) using Claude 3 or OpenAI
- **Renders videos** automatically using Remotion from written content
- **Supports multi-language** output (English/Vietnamese)
- **Optimizes for platforms** (Reels, TikTok, Shorts)

## Installation

### Prerequisites

```bash
node >= 18.0.0
npm or yarn
```

### Clone and Setup

```bash
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
yarn install
```

### Environment Configuration

Create `.env.local` file:

```bash
# AI Providers
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Next.js
NEXT_PUBLIC_API_URL=http://localhost:3000

# Optional: Database
DATABASE_URL=postgresql://user:password@localhost:5432/marketing_pipeline

# Remotion (for video rendering)
REMOTION_REGION=us-east-1
```

### Run Development Server

```bash
npm run dev
# or
yarn dev
```

Access at `http://localhost:3000`

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app routes
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI service integrations
│   │   ├── scraper/     # Web scraping logic
│   │   ├── render/      # Video rendering
│   │   └── utils/       # Helpers
│   └── types/           # TypeScript definitions
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Key API Routes and Usage

### 1. Research API (Content Scraping)

```typescript
// src/app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';

export async function POST(req: NextRequest) {
  const { keyword, sources } = await req.json();
  
  // Example: Scrape TechCrunch
  const response = await fetch('https://rapidapi.com/techcrunch/search', {
    headers: {
      'X-RapidAPI-Key': process.env.RAPIDAPI_KEY!,
    },
    method: 'POST',
    body: JSON.stringify({ query: keyword })
  });
  
  const articles = await response.json();
  
  return NextResponse.json({
    keyword,
    articles,
    timestamp: new Date().toISOString()
  });
}
```

**Client Usage:**

```typescript
const researchContent = async (keyword: string) => {
  const response = await fetch('/api/research', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      keyword,
      sources: ['techcrunch', 'a16z', 'twitter']
    })
  });
  
  return await response.json();
};
```

### 2. Content Generation API (Claude/OpenAI)

```typescript
// src/lib/ai/generate-content.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

export interface ContentRequest {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  researchData: any[];
}

export async function generateWithClaude(request: ContentRequest) {
  const anthropic = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY,
  });

  const prompt = `You are a content marketing expert. Create a ${request.format} article about "${request.keyword}" in ${request.language} with a ${request.tone} tone.

Research data:
${JSON.stringify(request.researchData, null, 2)}

Generate a complete article with:
- Engaging headline
- Introduction
- Main sections with data-backed insights
- Conclusion with call-to-action
- Suggested social media captions`;

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [{
      role: 'user',
      content: prompt
    }]
  });

  return message.content[0].text;
}

export async function generateWithOpenAI(request: ContentRequest) {
  const openai = new OpenAI({
    apiKey: process.env.OPENAI_API_KEY,
  });

  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [{
      role: 'system',
      content: 'You are an expert content marketer.'
    }, {
      role: 'user',
      content: `Create ${request.format} about ${request.keyword}`
    }],
    temperature: 0.7,
  });

  return completion.choices[0].message.content;
}
```

**API Route:**

```typescript
// src/app/api/generate/route.ts
import { generateWithClaude } from '@/lib/ai/generate-content';

export async function POST(req: NextRequest) {
  const contentRequest: ContentRequest = await req.json();
  
  const content = await generateWithClaude(contentRequest);
  
  return NextResponse.json({
    content,
    metadata: {
      format: contentRequest.format,
      language: contentRequest.language,
      generatedAt: new Date().toISOString()
    }
  });
}
```

### 3. Video Rendering (Remotion Integration)

```typescript
// src/lib/render/video-generator.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

export interface VideoConfig {
  title: string;
  content: string[];
  bgColor: string;
  duration: number; // in frames
}

export async function renderContentVideo(config: VideoConfig) {
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: config,
  });

  const outputLocation = path.join(
    process.cwd(),
    'public',
    'videos',
    `${Date.now()}.mp4`
  );

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: config,
  });

  return outputLocation;
}
```

**Remotion Component:**

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig } from 'remotion';

interface ContentVideoProps {
  title: string;
  content: string[];
  bgColor: string;
}

export const ContentVideo: React.FC<ContentVideoProps> = ({
  title,
  content,
  bgColor
}) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  const opacity = Math.min(1, frame / 30);

  return (
    <AbsoluteFill style={{ backgroundColor: bgColor }}>
      <div style={{ 
        padding: 60, 
        opacity,
        fontFamily: 'Arial, sans-serif'
      }}>
        <h1 style={{ fontSize: 72, marginBottom: 40 }}>
          {title}
        </h1>
        {content.map((text, i) => (
          <p key={i} style={{ fontSize: 36, marginBottom: 20 }}>
            {text}
          </p>
        ))}
      </div>
    </AbsoluteFill>
  );
};
```

**Remotion Config:**

```typescript
// remotion/index.ts
import { registerRoot } from 'remotion';
import { ContentVideo } from './ContentVideo';

registerRoot(() => {
  return (
    <>
      <Composition
        id="ContentVideo"
        component={ContentVideo}
        durationInFrames={300}
        fps={30}
        width={1080}
        height={1920} // Vertical for Reels/TikTok
      />
    </>
  );
});
```

## Complete Workflow Example

```typescript
// src/lib/pipeline/complete-workflow.ts
import { researchContent } from './research';
import { generateWithClaude } from '@/lib/ai/generate-content';
import { renderContentVideo } from '@/lib/render/video-generator';

export async function runCompletePipeline(keyword: string) {
  // Step 1: Research
  console.log('🔍 Researching:', keyword);
  const research = await fetch('/api/research', {
    method: 'POST',
    body: JSON.stringify({ keyword, sources: ['techcrunch'] })
  }).then(r => r.json());

  // Step 2: Generate Content
  console.log('✍️ Generating content...');
  const content = await generateWithClaude({
    keyword,
    format: 'toplist',
    language: 'en',
    tone: 'expert',
    researchData: research.articles
  });

  // Step 3: Parse for video
  const contentLines = content.split('\n').filter(l => l.trim());
  
  // Step 4: Render Video
  console.log('🎬 Rendering video...');
  const videoPath = await renderContentVideo({
    title: keyword,
    content: contentLines.slice(0, 5),
    bgColor: '#1a1a1a',
    duration: 300
  });

  return {
    research,
    content,
    videoPath
  };
}
```

**Usage in Component:**

```typescript
// src/components/ContentPipeline.tsx
'use client';

import { useState } from 'react';

export function ContentPipeline() {
  const [keyword, setKeyword] = useState('');
  const [result, setResult] = useState<any>(null);
  const [loading, setLoading] = useState(false);

  const handleGenerate = async () => {
    setLoading(true);
    
    const response = await fetch('/api/pipeline', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ keyword })
    });
    
    const data = await response.json();
    setResult(data);
    setLoading(false);
  };

  return (
    <div className="p-8">
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
        <div className="mt-8">
          <h2>Generated Content</h2>
          <pre>{result.content}</pre>
          
          {result.videoPath && (
            <video src={result.videoPath} controls className="mt-4" />
          )}
        </div>
      )}
    </div>
  );
}
```

## Configuration Patterns

### Multi-Provider AI Setup

```typescript
// src/lib/ai/provider.ts
type AIProvider = 'claude' | 'openai';

export const getAIProvider = (): AIProvider => {
  return (process.env.PREFERRED_AI_PROVIDER as AIProvider) || 'claude';
};

export async function generateContent(request: ContentRequest) {
  const provider = getAIProvider();
  
  switch (provider) {
    case 'claude':
      return generateWithClaude(request);
    case 'openai':
      return generateWithOpenAI(request);
    default:
      throw new Error(`Unknown provider: ${provider}`);
  }
}
```

### Rate Limiting for APIs

```typescript
// src/lib/utils/rate-limiter.ts
import pLimit from 'p-limit';

const limit = pLimit(3); // Max 3 concurrent requests

export async function batchResearch(keywords: string[]) {
  const promises = keywords.map(keyword =>
    limit(() => researchContent(keyword))
  );
  
  return Promise.all(promises);
}
```

## Common Troubleshooting

### Video Rendering Fails

```typescript
// Check Remotion installation
// Ensure ffmpeg is installed on system
import { getCompositions } from '@remotion/renderer';

async function debugRemotionSetup() {
  try {
    const compositions = await getCompositions(bundleLocation);
    console.log('Available compositions:', compositions);
  } catch (error) {
    console.error('Remotion setup error:', error);
    // Install ffmpeg: sudo apt-get install ffmpeg (Linux)
    // Or: brew install ffmpeg (Mac)
  }
}
```

### API Rate Limits

```typescript
// Implement retry logic
export async function fetchWithRetry(
  url: string,
  options: RequestInit,
  maxRetries = 3
) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      const response = await fetch(url, options);
      if (response.ok) return response;
      
      if (response.status === 429) {
        await new Promise(resolve => setTimeout(resolve, 2000 * (i + 1)));
        continue;
      }
      
      throw new Error(`HTTP ${response.status}`);
    } catch (error) {
      if (i === maxRetries - 1) throw error;
    }
  }
}
```

### Claude/OpenAI Token Limits

```typescript
// Chunk large content
export function chunkContent(text: string, maxTokens = 3000) {
  const chunks: string[] = [];
  const words = text.split(' ');
  let currentChunk = '';
  
  for (const word of words) {
    if ((currentChunk + word).length > maxTokens * 4) { // ~4 chars per token
      chunks.push(currentChunk);
      currentChunk = word;
    } else {
      currentChunk += ' ' + word;
    }
  }
  
  if (currentChunk) chunks.push(currentChunk);
  return chunks;
}
```

## Advanced Usage: Scheduled Content Generation

```typescript
// src/lib/scheduler/cron-job.ts
import cron from 'node-cron';

export function scheduleContentGeneration(keywords: string[]) {
  // Run every day at 9 AM
  cron.schedule('0 9 * * *', async () => {
    console.log('Starting scheduled content generation...');
    
    for (const keyword of keywords) {
      try {
        await runCompletePipeline(keyword);
        console.log(`✅ Completed: ${keyword}`);
      } catch (error) {
        console.error(`❌ Failed: ${keyword}`, error);
      }
    }
  });
}
```

This skill provides comprehensive coverage of the marketing-pipeline-share project for automated AI-driven content creation, from research through video rendering.
