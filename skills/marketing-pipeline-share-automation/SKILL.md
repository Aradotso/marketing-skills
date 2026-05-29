---
name: marketing-pipeline-share-automation
description: Automated AI content pipeline for research, scriptwriting, and video generation using Claude/OpenAI and Remotion
triggers:
  - how do I automate content creation with AI
  - set up marketing pipeline for auto-generating videos
  - create automated content workflow from research to video
  - use Claude and OpenAI for content automation
  - build AI-powered content generation pipeline
  - automate research and scriptwriting with marketing pipeline
  - generate videos automatically from content research
  - integrate Remotion for automated video rendering
---

# Marketing Pipeline Share Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the Ultimate AI Content Pipeline, an end-to-end content automation system that handles research, scriptwriting, and video generation using Claude 3, OpenAI, and Remotion.

## What This Project Does

Marketing Pipeline Share automates the entire content creation workflow:

1. **Auto-Research**: Crawls recent content from TechCrunch, a16z, X (Twitter), LinkedIn within 24 hours
2. **AI Content Generation**: Creates multi-format content (Toplist, POV, Case Study, How-to) in multiple languages using Claude/OpenAI
3. **Video Rendering**: Automatically generates infographics and short videos using Remotion
4. **Multi-Platform Export**: Outputs optimized videos for Reels, TikTok, Shorts

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
# AI Services
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# RapidAPI for content scraping
RAPIDAPI_KEY=your_rapidapi_key

# Next.js Configuration
NEXT_PUBLIC_API_URL=http://localhost:3000

# Remotion Configuration (optional)
REMOTION_AWS_ACCESS_KEY_ID=your_aws_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret
```

## Key Commands

### Development

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Run Remotion studio for video editing
npm run remotion:studio

# Render video
npm run remotion:render
```

## Core API Routes

### 1. Research Content API

**Endpoint**: `POST /api/research`

```typescript
// app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';

export async function POST(request: NextRequest) {
  const { keyword, sources } = await request.json();
  
  const response = await fetch('https://rapidapi.com/api/news', {
    method: 'GET',
    headers: {
      'X-RapidAPI-Key': process.env.RAPIDAPI_KEY!,
      'X-RapidAPI-Host': 'news-api.p.rapidapi.com'
    },
    params: {
      q: keyword,
      sources: sources.join(','),
      from: new Date(Date.now() - 24 * 60 * 60 * 1000).toISOString()
    }
  });
  
  const data = await response.json();
  return NextResponse.json(data);
}
```

**Usage**:

```typescript
const researchContent = async (keyword: string) => {
  const response = await fetch('/api/research', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      keyword,
      sources: ['techcrunch', 'a16z', 'twitter', 'linkedin']
    })
  });
  
  return await response.json();
};
```

### 2. Generate Content API

**Endpoint**: `POST /api/generate-content`

```typescript
// app/api/generate-content/route.ts
import { NextRequest, NextResponse } from 'next/server';
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

export async function POST(request: NextRequest) {
  const { 
    researchData, 
    format, 
    language, 
    tone, 
    provider = 'claude' 
  } = await request.json();
  
  const prompt = `Based on this research data: ${JSON.stringify(researchData)}
  
Create a ${format} format article in ${language} with a ${tone} tone.
Include:
- Compelling headline
- Data-backed insights
- Actionable takeaways
- SEO-optimized structure`;

  let content;
  
  if (provider === 'claude') {
    const message = await anthropic.messages.create({
      model: 'claude-3-sonnet-20240229',
      max_tokens: 4096,
      messages: [{
        role: 'user',
        content: prompt
      }]
    });
    
    content = message.content[0].type === 'text' 
      ? message.content[0].text 
      : '';
  } else {
    const completion = await openai.chat.completions.create({
      model: 'gpt-4-turbo-preview',
      messages: [{
        role: 'user',
        content: prompt
      }]
    });
    
    content = completion.choices[0].message.content;
  }
  
  return NextResponse.json({ content });
}
```

**Usage**:

```typescript
const generateContent = async (
  researchData: any,
  format: 'toplist' | 'pov' | 'case-study' | 'how-to',
  language: 'en' | 'vi',
  tone: 'expert' | 'friendly' | 'humorous'
) => {
  const response = await fetch('/api/generate-content', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      researchData,
      format,
      language,
      tone,
      provider: 'claude' // or 'openai'
    })
  });
  
  return await response.json();
};
```

### 3. Video Generation API

**Endpoint**: `POST /api/generate-video`

```typescript
// app/api/generate-video/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

export async function POST(request: NextRequest) {
  const { content, platform, duration } = await request.json();
  
  // Bundle Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: (config) => config,
  });
  
  // Select composition based on platform
  const compositionId = platform === 'reels' ? 'ReelsVideo' : 
                        platform === 'tiktok' ? 'TikTokVideo' : 
                        'ShortsVideo';
  
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: {
      content,
      duration
    }
  });
  
  // Render video
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
    inputProps: {
      content,
      duration
    }
  });
  
  return NextResponse.json({ 
    videoUrl: outputLocation.replace(process.cwd() + '/public', '')
  });
}
```

## Remotion Video Components

### Basic Video Composition

```typescript
// remotion/ReelsVideo.tsx
import { AbsoluteFill, Sequence, useCurrentFrame, useVideoConfig } from 'remotion';
import React from 'react';

interface ReelsVideoProps {
  content: {
    title: string;
    points: string[];
    author: string;
  };
  duration: number;
}

export const ReelsVideo: React.FC<ReelsVideoProps> = ({ content, duration }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  
  const opacity = Math.min(1, frame / 30);
  
  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a1a' }}>
      <Sequence from={0} durationInFrames={fps * 3}>
        <AbsoluteFill
          style={{
            justifyContent: 'center',
            alignItems: 'center',
            opacity
          }}
        >
          <h1 style={{ 
            color: 'white', 
            fontSize: 60, 
            textAlign: 'center',
            padding: '0 40px'
          }}>
            {content.title}
          </h1>
        </AbsoluteFill>
      </Sequence>
      
      {content.points.map((point, index) => (
        <Sequence 
          key={index}
          from={fps * (3 + index * 2)} 
          durationInFrames={fps * 2}
        >
          <AbsoluteFill
            style={{
              justifyContent: 'center',
              alignItems: 'center',
              padding: '0 60px'
            }}
          >
            <div style={{
              backgroundColor: '#2a2a2a',
              padding: 40,
              borderRadius: 20
            }}>
              <p style={{ 
                color: 'white', 
                fontSize: 40,
                lineHeight: 1.5
              }}>
                {index + 1}. {point}
              </p>
            </div>
          </AbsoluteFill>
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};
```

### Register Composition

```typescript
// remotion/index.ts
import { registerRoot } from 'remotion';
import { ReelsVideo } from './ReelsVideo';
import { Composition } from 'remotion';

export const RemotionRoot: React.FC = () => {
  return (
    <>
      <Composition
        id="ReelsVideo"
        component={ReelsVideo}
        durationInFrames={900}
        fps={30}
        width={1080}
        height={1920}
        defaultProps={{
          content: {
            title: 'Sample Title',
            points: ['Point 1', 'Point 2', 'Point 3'],
            author: 'Author Name'
          },
          duration: 30
        }}
      />
    </>
  );
};

registerRoot(RemotionRoot);
```

## Full Workflow Example

```typescript
// lib/content-pipeline.ts
export class ContentPipeline {
  async run(keyword: string, options: {
    format: 'toplist' | 'pov' | 'case-study' | 'how-to';
    language: 'en' | 'vi';
    tone: 'expert' | 'friendly' | 'humorous';
    generateVideo: boolean;
    platform?: 'reels' | 'tiktok' | 'shorts';
  }) {
    // Step 1: Research
    console.log('🔍 Researching content...');
    const researchData = await fetch('/api/research', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        keyword,
        sources: ['techcrunch', 'a16z', 'twitter', 'linkedin']
      })
    }).then(res => res.json());
    
    // Step 2: Generate Content
    console.log('✍️ Generating content...');
    const { content } = await fetch('/api/generate-content', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        researchData,
        format: options.format,
        language: options.language,
        tone: options.tone,
        provider: 'claude'
      })
    }).then(res => res.json());
    
    // Step 3: Generate Video (optional)
    let videoUrl = null;
    if (options.generateVideo && options.platform) {
      console.log('🎬 Rendering video...');
      const videoResult = await fetch('/api/generate-video', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          content: JSON.parse(content),
          platform: options.platform,
          duration: 30
        })
      }).then(res => res.json());
      
      videoUrl = videoResult.videoUrl;
    }
    
    return {
      content,
      videoUrl,
      researchData
    };
  }
}
```

## Usage in React Components

```typescript
// app/components/ContentGenerator.tsx
'use client';

import { useState } from 'react';
import { ContentPipeline } from '@/lib/content-pipeline';

export default function ContentGenerator() {
  const [keyword, setKeyword] = useState('');
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState<any>(null);
  
  const pipeline = new ContentPipeline();
  
  const handleGenerate = async () => {
    setLoading(true);
    try {
      const result = await pipeline.run(keyword, {
        format: 'toplist',
        language: 'en',
        tone: 'expert',
        generateVideo: true,
        platform: 'reels'
      });
      setResult(result);
    } catch (error) {
      console.error('Pipeline error:', error);
    } finally {
      setLoading(false);
    }
  };
  
  return (
    <div>
      <input
        type="text"
        value={keyword}
        onChange={(e) => setKeyword(e.target.value)}
        placeholder="Enter keyword..."
      />
      <button onClick={handleGenerate} disabled={loading}>
        {loading ? 'Generating...' : 'Generate Content'}
      </button>
      
      {result && (
        <div>
          <h2>Content</h2>
          <pre>{result.content}</pre>
          
          {result.videoUrl && (
            <video src={result.videoUrl} controls />
          )}
        </div>
      )}
    </div>
  );
}
```

## Common Patterns

### 1. Batch Content Generation

```typescript
const generateBatch = async (keywords: string[]) => {
  const pipeline = new ContentPipeline();
  
  const results = await Promise.all(
    keywords.map(keyword => 
      pipeline.run(keyword, {
        format: 'toplist',
        language: 'en',
        tone: 'expert',
        generateVideo: false
      })
    )
  );
  
  return results;
};
```

### 2. Multi-Language Content

```typescript
const generateMultiLanguage = async (keyword: string) => {
  const pipeline = new ContentPipeline();
  
  const [enContent, viContent] = await Promise.all([
    pipeline.run(keyword, {
      format: 'pov',
      language: 'en',
      tone: 'friendly',
      generateVideo: false
    }),
    pipeline.run(keyword, {
      format: 'pov',
      language: 'vi',
      tone: 'friendly',
      generateVideo: false
    })
  ]);
  
  return { en: enContent, vi: viContent };
};
```

### 3. Custom Video Templates

```typescript
// remotion/CustomTemplate.tsx
export const CustomTemplate: React.FC<{
  background: string;
  textColor: string;
  content: any;
}> = ({ background, textColor, content }) => {
  return (
    <AbsoluteFill style={{ backgroundColor: background }}>
      {/* Custom video layout */}
    </AbsoluteFill>
  );
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
    throw new Error(
      `Missing required environment variables: ${missing.join(', ')}`
    );
  }
}
```

### Rate Limiting

```typescript
// lib/rate-limiter.ts
export class RateLimiter {
  private queue: Array<() => Promise<any>> = [];
  private processing = false;
  
  async add<T>(fn: () => Promise<T>): Promise<T> {
    return new Promise((resolve, reject) => {
      this.queue.push(async () => {
        try {
          const result = await fn();
          resolve(result);
        } catch (error) {
          reject(error);
        }
      });
      
      this.process();
    });
  }
  
  private async process() {
    if (this.processing || this.queue.length === 0) return;
    
    this.processing = true;
    const fn = this.queue.shift()!;
    await fn();
    await new Promise(resolve => setTimeout(resolve, 1000)); // 1s delay
    this.processing = false;
    this.process();
  }
}
```

### Video Rendering Errors

```typescript
// Check Remotion bundle
if (!existsSync(bundleLocation)) {
  throw new Error('Remotion bundle not found. Run: npm run remotion:bundle');
}

// Verify composition exists
const compositions = await getCompositions(bundleLocation);
if (!compositions.find(c => c.id === compositionId)) {
  throw new Error(`Composition ${compositionId} not found`);
}
```

This skill provides comprehensive coverage of the Marketing Pipeline Share automation system, enabling AI agents to effectively assist developers in setting up and using this powerful content automation tool.
