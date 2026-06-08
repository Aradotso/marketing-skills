---
name: marketing-pipeline-ai-content-automation
description: Automate content creation from research to video generation using AI (Claude, OpenAI) and Remotion for marketing pipelines
triggers:
  - how do I set up the AI content pipeline
  - generate automated marketing content with research
  - create videos from articles using Remotion
  - scrape news and generate content automatically
  - set up Claude and OpenAI for content automation
  - build marketing content pipeline with AI
  - automate content from research to video
  - configure the marketing pipeline system
---

# Marketing Pipeline AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables you to use the Ultimate AI Content Pipeline system - an end-to-end automation platform that handles research, content generation, and video creation for marketing workflows. The system automatically crawls news sources, generates multi-format content using Claude/OpenAI, and renders videos using Remotion.

## What This Project Does

The marketing-pipeline-share project automates the entire content creation workflow:

1. **Auto-Research**: Crawls recent news from TechCrunch, a16z, Twitter/X, LinkedIn
2. **AI Content Generation**: Creates articles in multiple formats (toplist, POV, case study, how-to) using Claude 3 or OpenAI
3. **Multi-language Support**: Generates content in English and Vietnamese simultaneously
4. **Video Rendering**: Automatically creates videos and infographics from content using Remotion
5. **Platform Optimization**: Outputs video in formats suitable for Reels, TikTok, Shorts

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
# AI Provider Keys
ANTHROPIC_API_KEY=your_claude_api_key_here
OPENAI_API_KEY=your_openai_api_key_here

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key_here

# Optional: Database
DATABASE_URL=postgresql://user:password@localhost:5432/content_pipeline

# Next.js Configuration
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

### Running the Development Server

```bash
npm run dev
# or
yarn dev
# or
pnpm dev
```

Access the application at `http://localhost:3000`

## Key API Routes and Usage

### 1. Content Research Endpoint

```typescript
// app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';

export async function POST(req: NextRequest) {
  const { keyword, sources, timeRange } = await req.json();
  
  // Crawl news from specified sources
  const results = await crawlNews({
    keyword,
    sources: sources || ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeRange: timeRange || '24h'
  });
  
  return NextResponse.json(results);
}
```

**Client-side usage:**

```typescript
// Example: Trigger research
async function startResearch(keyword: string) {
  const response = await fetch('/api/research', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      keyword: 'AI automation',
      sources: ['techcrunch', 'twitter'],
      timeRange: '24h'
    })
  });
  
  const data = await response.json();
  return data;
}
```

### 2. Content Generation Endpoint

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

export async function POST(req: NextRequest) {
  const { researchData, format, language, tone, provider } = await req.json();
  
  if (provider === 'claude') {
    const anthropic = new Anthropic({
      apiKey: process.env.ANTHROPIC_API_KEY,
    });
    
    const message = await anthropic.messages.create({
      model: 'claude-3-5-sonnet-20241022',
      max_tokens: 4096,
      messages: [{
        role: 'user',
        content: `Generate a ${format} article in ${language} with ${tone} tone based on: ${JSON.stringify(researchData)}`
      }]
    });
    
    return NextResponse.json({ 
      content: message.content[0].text,
      provider: 'claude'
    });
  } else {
    const openai = new OpenAI({
      apiKey: process.env.OPENAI_API_KEY,
    });
    
    const completion = await openai.chat.completions.create({
      model: 'gpt-4-turbo-preview',
      messages: [{
        role: 'system',
        content: 'You are a professional content writer.'
      }, {
        role: 'user',
        content: `Generate a ${format} article in ${language} with ${tone} tone based on: ${JSON.stringify(researchData)}`
      }]
    });
    
    return NextResponse.json({ 
      content: completion.choices[0].message.content,
      provider: 'openai'
    });
  }
}
```

**Client-side usage:**

```typescript
// Generate content from research
async function generateContent(researchData: any) {
  const response = await fetch('/api/generate', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      researchData,
      format: 'toplist', // or 'pov', 'case-study', 'how-to'
      language: 'vi', // or 'en'
      tone: 'professional', // or 'friendly', 'humorous'
      provider: 'claude' // or 'openai'
    })
  });
  
  return await response.json();
}
```

### 3. Video Rendering with Remotion

```typescript
// app/api/render-video/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import { webpackOverride } from './webpack-override';
import path from 'path';

export async function POST(req: NextRequest) {
  const { content, title, style } = await req.json();
  
  try {
    // Bundle Remotion video
    const bundleLocation = await bundle({
      entryPoint: path.resolve('./remotion/index.ts'),
      webpackOverride,
    });
    
    // Get composition
    const composition = await selectComposition({
      serveUrl: bundleLocation,
      id: 'ContentVideo',
      inputProps: { content, title, style }
    });
    
    // Render video
    const outputLocation = path.join(process.cwd(), 'public', 'videos', `${Date.now()}.mp4`);
    
    await renderMedia({
      composition,
      serveUrl: bundleLocation,
      codec: 'h264',
      outputLocation,
      inputProps: { content, title, style }
    });
    
    return NextResponse.json({ 
      videoUrl: `/videos/${path.basename(outputLocation)}`
    });
  } catch (error) {
    return NextResponse.json({ error: error.message }, { status: 500 });
  }
}
```

### 4. Remotion Video Component

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig, interpolate } from 'remotion';

interface ContentVideoProps {
  title: string;
  content: string;
  style?: 'minimal' | 'dynamic' | 'professional';
}

export const ContentVideo: React.FC<ContentVideoProps> = ({ 
  title, 
  content, 
  style = 'professional' 
}) => {
  const frame = useCurrentFrame();
  const { fps, durationInFrames } = useVideoConfig();
  
  const opacity = interpolate(
    frame,
    [0, 30, durationInFrames - 30, durationInFrames],
    [0, 1, 1, 0]
  );
  
  const scale = interpolate(
    frame,
    [0, 30],
    [0.8, 1],
    { extrapolateRight: 'clamp' }
  );
  
  return (
    <AbsoluteFill
      style={{
        backgroundColor: style === 'minimal' ? '#ffffff' : '#0a0a0a',
        justifyContent: 'center',
        alignItems: 'center',
        fontFamily: 'Arial, sans-serif',
      }}
    >
      <div
        style={{
          opacity,
          transform: `scale(${scale})`,
          padding: '40px',
          maxWidth: '80%',
        }}
      >
        <h1
          style={{
            fontSize: '48px',
            color: style === 'minimal' ? '#000' : '#fff',
            marginBottom: '20px',
            textAlign: 'center',
          }}
        >
          {title}
        </h1>
        <p
          style={{
            fontSize: '24px',
            color: style === 'minimal' ? '#333' : '#ccc',
            lineHeight: '1.6',
            textAlign: 'center',
          }}
        >
          {content.substring(0, 200)}...
        </p>
      </div>
    </AbsoluteFill>
  );
};
```

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
        height={1920}
        defaultProps={{
          title: 'Your Content Title',
          content: 'Your generated content',
          style: 'professional'
        }}
      />
    </>
  );
});
```

## Common Workflow Patterns

### Full Pipeline Automation

```typescript
// lib/content-pipeline.ts
export class ContentPipeline {
  async runFullPipeline(keyword: string) {
    // Step 1: Research
    const researchData = await fetch('/api/research', {
      method: 'POST',
      body: JSON.stringify({ keyword })
    }).then(r => r.json());
    
    // Step 2: Generate content in both languages
    const [contentVi, contentEn] = await Promise.all([
      fetch('/api/generate', {
        method: 'POST',
        body: JSON.stringify({
          researchData,
          format: 'toplist',
          language: 'vi',
          provider: 'claude'
        })
      }).then(r => r.json()),
      
      fetch('/api/generate', {
        method: 'POST',
        body: JSON.stringify({
          researchData,
          format: 'toplist',
          language: 'en',
          provider: 'openai'
        })
      }).then(r => r.json())
    ]);
    
    // Step 3: Render videos
    const [videoVi, videoEn] = await Promise.all([
      fetch('/api/render-video', {
        method: 'POST',
        body: JSON.stringify({
          content: contentVi.content,
          title: `Top ${keyword}`,
          style: 'dynamic'
        })
      }).then(r => r.json()),
      
      fetch('/api/render-video', {
        method: 'POST',
        body: JSON.stringify({
          content: contentEn.content,
          title: `Top ${keyword}`,
          style: 'professional'
        })
      }).then(r => r.json())
    ]);
    
    return {
      research: researchData,
      content: { vi: contentVi, en: contentEn },
      videos: { vi: videoVi, en: videoEn }
    };
  }
}
```

### Using the Pipeline in a Component

```typescript
// app/components/ContentGenerator.tsx
'use client';

import { useState } from 'react';
import { ContentPipeline } from '@/lib/content-pipeline';

export default function ContentGenerator() {
  const [keyword, setKeyword] = useState('');
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState(null);
  
  const pipeline = new ContentPipeline();
  
  const handleGenerate = async () => {
    setLoading(true);
    try {
      const output = await pipeline.runFullPipeline(keyword);
      setResult(output);
    } catch (error) {
      console.error('Pipeline error:', error);
    } finally {
      setLoading(false);
    }
  };
  
  return (
    <div className="p-8">
      <input
        type="text"
        value={keyword}
        onChange={(e) => setKeyword(e.target.value)}
        placeholder="Enter keyword..."
        className="border p-2 w-full mb-4"
      />
      
      <button
        onClick={handleGenerate}
        disabled={loading}
        className="bg-blue-500 text-white px-6 py-2 rounded"
      >
        {loading ? 'Generating...' : 'Generate Content'}
      </button>
      
      {result && (
        <div className="mt-8">
          <h2 className="text-2xl font-bold mb-4">Results</h2>
          
          <div className="mb-4">
            <h3 className="font-semibold">Vietnamese Content:</h3>
            <p>{result.content.vi.content.substring(0, 200)}...</p>
            {result.videos.vi.videoUrl && (
              <video src={result.videos.vi.videoUrl} controls className="mt-2 w-full max-w-md" />
            )}
          </div>
          
          <div>
            <h3 className="font-semibold">English Content:</h3>
            <p>{result.content.en.content.substring(0, 200)}...</p>
            {result.videos.en.videoUrl && (
              <video src={result.videos.en.videoUrl} controls className="mt-2 w-full max-w-md" />
            )}
          </div>
        </div>
      )}
    </div>
  );
}
```

## Configuration

### Customizing Content Formats

```typescript
// lib/content-formats.ts
export const contentFormats = {
  toplist: {
    prompt: 'Create a numbered list article highlighting the top items',
    structure: ['intro', 'items', 'conclusion']
  },
  pov: {
    prompt: 'Write from a specific perspective or viewpoint',
    structure: ['context', 'perspective', 'arguments', 'conclusion']
  },
  caseStudy: {
    prompt: 'Analyze a real-world example with data and insights',
    structure: ['background', 'challenge', 'solution', 'results', 'takeaways']
  },
  howTo: {
    prompt: 'Provide step-by-step instructions',
    structure: ['intro', 'steps', 'tips', 'conclusion']
  }
};
```

### Video Style Presets

```typescript
// remotion/styles.ts
export const videoStyles = {
  minimal: {
    backgroundColor: '#ffffff',
    textColor: '#000000',
    fontSize: 32,
    animation: 'fade'
  },
  dynamic: {
    backgroundColor: '#ff0080',
    textColor: '#ffffff',
    fontSize: 48,
    animation: 'slide'
  },
  professional: {
    backgroundColor: '#0a0a0a',
    textColor: '#ffffff',
    fontSize: 36,
    animation: 'zoom'
  }
};
```

## Troubleshooting

### API Key Issues

```typescript
// lib/validate-env.ts
export function validateEnvironment() {
  const required = [
    'ANTHROPIC_API_KEY',
    'OPENAI_API_KEY',
    'RAPIDAPI_KEY'
  ];
  
  const missing = required.filter(key => !process.env[key]);
  
  if (missing.length > 0) {
    throw new Error(`Missing required environment variables: ${missing.join(', ')}`);
  }
}

// Call in your API routes
import { validateEnvironment } from '@/lib/validate-env';

export async function POST(req: NextRequest) {
  validateEnvironment();
  // ... rest of handler
}
```

### Remotion Rendering Errors

If video rendering fails, check:

```bash
# Install required dependencies
npm install @remotion/bundler @remotion/renderer @remotion/cli

# Ensure ffmpeg is installed
# macOS
brew install ffmpeg

# Ubuntu/Debian
sudo apt-get install ffmpeg

# Test Remotion setup
npx remotion preview remotion/index.ts
```

### Rate Limiting

```typescript
// lib/rate-limiter.ts
import { Ratelimit } from '@upstash/ratelimit';
import { Redis } from '@upstash/redis';

const ratelimit = new Ratelimit({
  redis: Redis.fromEnv(),
  limiter: Ratelimit.slidingWindow(10, '1 h'),
});

export async function checkRateLimit(identifier: string) {
  const { success, remaining } = await ratelimit.limit(identifier);
  
  if (!success) {
    throw new Error('Rate limit exceeded. Please try again later.');
  }
  
  return remaining;
}
```

### Memory Issues with Large Videos

```typescript
// next.config.js
module.exports = {
  experimental: {
    // Increase memory for video rendering
    workerThreads: false,
    cpus: 1
  },
  webpack: (config) => {
    config.externals.push({
      'utf-8-validate': 'commonjs utf-8-validate',
      'bufferutil': 'commonjs bufferutil',
    });
    return config;
  }
};
```

For large video rendering, consider using a worker queue:

```typescript
// lib/video-queue.ts
import Bull from 'bull';

export const videoQueue = new Bull('video-rendering', {
  redis: process.env.REDIS_URL
});

videoQueue.process(async (job) => {
  const { content, title, style } = job.data;
  
  // Render video in background worker
  const result = await renderVideo({ content, title, style });
  
  return result;
});
```

This skill covers the essential functionality of the marketing pipeline system, focusing on practical integration patterns for AI coding agents to help developers automate content creation workflows.
