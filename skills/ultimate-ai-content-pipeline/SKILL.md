---
name: ultimate-ai-content-pipeline
description: Automated content pipeline from research to video generation using Claude/OpenAI and Remotion
triggers:
  - how do I automate content creation from research to video
  - set up an AI content pipeline with video generation
  - create automated marketing content with Claude and Remotion
  - build a content automation system with AI research
  - generate videos from AI-written content automatically
  - configure an end-to-end content marketing pipeline
  - automate blog posts and video creation with AI
  - use Remotion to render videos from AI content
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## What This Project Does

Ultimate AI Content Pipeline is a complete content automation system that transforms a single keyword into fully-formed articles and videos. It handles:

1. **Auto-Research**: Crawls recent content from TechCrunch, a16z, Twitter/X, LinkedIn
2. **AI Content Generation**: Uses Claude 3 or OpenAI to create articles in multiple formats (toplist, POV, case study, how-to)
3. **Multi-language Support**: Generates English and Vietnamese content simultaneously
4. **Video Rendering**: Automatically creates infographics and short videos using Remotion
5. **Platform Optimization**: Outputs optimized for Reels, TikTok, Shorts

This is a Next.js-based application built with TypeScript that integrates multiple AI services and Remotion for video generation.

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

```env
# AI Services
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# RapidAPI for web scraping/research
RAPIDAPI_KEY=your_rapidapi_key

# Next.js
NEXT_PUBLIC_APP_URL=http://localhost:3000

# Optional: Video rendering
REMOTION_LICENSE_KEY=your_remotion_license_key
```

## Running the Application

```bash
# Development mode
npm run dev

# Production build
npm run build
npm start

# Remotion video rendering (if separate)
npm run remotion
```

## Key API Routes & Usage

### 1. Research Endpoint

The research endpoint crawls and analyzes content from multiple sources:

```typescript
// pages/api/research.ts
import type { NextApiRequest, NextApiResponse } from 'next';

interface ResearchRequest {
  keyword: string;
  sources?: string[];
  timeframe?: '24h' | '7d' | '30d';
}

interface ResearchResult {
  insights: string[];
  articles: Array<{
    title: string;
    url: string;
    summary: string;
    sentiment: string;
  }>;
  trends: string[];
}

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse<ResearchResult>
) {
  const { keyword, sources = ['techcrunch', 'a16z', 'twitter'], timeframe = '24h' } = req.body as ResearchRequest;
  
  // Research implementation would use RapidAPI
  const research = await performResearch(keyword, sources, timeframe);
  
  res.status(200).json(research);
}
```

**Usage from client:**

```typescript
const performResearch = async (keyword: string) => {
  const response = await fetch('/api/research', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      keyword,
      sources: ['techcrunch', 'a16z', 'twitter'],
      timeframe: '24h'
    })
  });
  
  return await response.json();
};
```

### 2. Content Generation Endpoint

Generate content using Claude or OpenAI:

```typescript
// pages/api/generate-content.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

interface ContentRequest {
  research: ResearchResult;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi' | 'both';
  tone: 'expert' | 'friendly' | 'humorous';
  provider?: 'claude' | 'openai';
}

export default async function handler(req: NextApiRequest, res: NextApiResponse) {
  const { research, format, language, tone, provider = 'claude' } = req.body as ContentRequest;
  
  if (provider === 'claude') {
    const anthropic = new Anthropic({
      apiKey: process.env.ANTHROPIC_API_KEY,
    });
    
    const message = await anthropic.messages.create({
      model: 'claude-3-5-sonnet-20241022',
      max_tokens: 4096,
      messages: [{
        role: 'user',
        content: buildContentPrompt(research, format, language, tone)
      }]
    });
    
    res.status(200).json({ content: message.content });
  } else {
    const openai = new OpenAI({
      apiKey: process.env.OPENAI_API_KEY,
    });
    
    const completion = await openai.chat.completions.create({
      model: 'gpt-4-turbo-preview',
      messages: [{
        role: 'user',
        content: buildContentPrompt(research, format, language, tone)
      }]
    });
    
    res.status(200).json({ content: completion.choices[0].message.content });
  }
}
```

### 3. Video Generation with Remotion

Render videos from generated content:

```typescript
// remotion/compositions.tsx
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
          title: 'AI Content Title',
          points: [],
          backgroundColor: '#1a1a1a',
        }}
      />
    </>
  );
};
```

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig, interpolate } from 'remotion';

interface ContentVideoProps {
  title: string;
  points: string[];
  backgroundColor: string;
}

export const ContentVideo: React.FC<ContentVideoProps> = ({ 
  title, 
  points, 
  backgroundColor 
}) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  
  const opacity = interpolate(frame, [0, 30], [0, 1], {
    extrapolateRight: 'clamp',
  });
  
  return (
    <AbsoluteFill style={{ backgroundColor }}>
      <div style={{ opacity, padding: 40 }}>
        <h1 style={{ fontSize: 60, color: 'white', marginBottom: 40 }}>
          {title}
        </h1>
        {points.map((point, idx) => {
          const pointStart = 30 + (idx * 60);
          const pointOpacity = interpolate(
            frame,
            [pointStart, pointStart + 20],
            [0, 1],
            { extrapolateRight: 'clamp' }
          );
          
          return (
            <p 
              key={idx} 
              style={{ 
                fontSize: 36, 
                color: 'white', 
                opacity: pointOpacity,
                marginBottom: 20 
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

**Render video programmatically:**

```typescript
// pages/api/render-video.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

export default async function handler(req: NextApiRequest, res: NextApiResponse) {
  const { title, points } = req.body;
  
  const compositionId = 'ContentVideo';
  const bundleLocation = await bundle(path.join(process.cwd(), 'remotion/index.ts'));
  
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: { title, points, backgroundColor: '#1a1a1a' },
  });
  
  const outputLocation = path.join(process.cwd(), 'public', 'videos', `${Date.now()}.mp4`);
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: { title, points, backgroundColor: '#1a1a1a' },
  });
  
  res.status(200).json({ videoUrl: `/videos/${path.basename(outputLocation)}` });
}
```

## Common Patterns

### Full Pipeline Execution

```typescript
// lib/contentPipeline.ts
export async function executeContentPipeline(keyword: string) {
  // Step 1: Research
  const research = await fetch('/api/research', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ keyword, sources: ['techcrunch', 'a16z'], timeframe: '24h' })
  }).then(r => r.json());
  
  // Step 2: Generate Content
  const content = await fetch('/api/generate-content', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      research,
      format: 'toplist',
      language: 'both',
      tone: 'expert',
      provider: 'claude'
    })
  }).then(r => r.json());
  
  // Step 3: Extract key points for video
  const points = extractKeyPoints(content.content);
  
  // Step 4: Render Video
  const video = await fetch('/api/render-video', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      title: keyword,
      points
    })
  }).then(r => r.json());
  
  return { research, content, video };
}
```

### Component Example

```typescript
// components/ContentGenerator.tsx
import { useState } from 'react';

export default function ContentGenerator() {
  const [keyword, setKeyword] = useState('');
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState<any>(null);
  
  const handleGenerate = async () => {
    setLoading(true);
    try {
      const pipeline = await executeContentPipeline(keyword);
      setResult(pipeline);
    } catch (error) {
      console.error('Pipeline error:', error);
    } finally {
      setLoading(false);
    }
  };
  
  return (
    <div className="content-generator">
      <input
        type="text"
        value={keyword}
        onChange={(e) => setKeyword(e.target.value)}
        placeholder="Enter keyword..."
        className="keyword-input"
      />
      <button onClick={handleGenerate} disabled={loading}>
        {loading ? 'Generating...' : 'Generate Content'}
      </button>
      
      {result && (
        <div className="results">
          <div className="research">
            <h3>Research Insights</h3>
            {result.research.insights.map((insight: string, i: number) => (
              <p key={i}>{insight}</p>
            ))}
          </div>
          
          <div className="content">
            <h3>Generated Content</h3>
            <div dangerouslySetInnerHTML={{ __html: result.content.content }} />
          </div>
          
          <div className="video">
            <h3>Generated Video</h3>
            <video src={result.video.videoUrl} controls />
          </div>
        </div>
      )}
    </div>
  );
}
```

## Troubleshooting

### API Rate Limits

If you encounter rate limit errors:

```typescript
// lib/rateLimiter.ts
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
    const fn = this.queue.shift();
    
    if (fn) {
      await fn();
      await new Promise(resolve => setTimeout(resolve, 1000)); // 1 second delay
    }
    
    this.processing = false;
    this.process();
  }
}
```

### Video Rendering Timeout

For long videos, increase timeout:

```typescript
await renderMedia({
  composition,
  serveUrl: bundleLocation,
  codec: 'h264',
  outputLocation,
  inputProps,
  timeoutInMilliseconds: 300000, // 5 minutes
});
```

### Memory Issues with Remotion

Use the CLI for large-scale rendering:

```bash
npx remotion render src/index.ts ContentVideo out/video.mp4 --props='{"title":"My Title","points":["Point 1","Point 2"]}'
```

### Claude API Errors

Handle streaming and errors properly:

```typescript
try {
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [{ role: 'user', content: prompt }]
  });
  return message.content;
} catch (error: any) {
  if (error.status === 429) {
    // Rate limit - wait and retry
    await new Promise(resolve => setTimeout(resolve, 5000));
    return generateContent(prompt); // Retry
  }
  throw error;
}
```
