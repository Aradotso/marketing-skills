---
name: ultimate-ai-content-pipeline
description: Automated content pipeline from research to video generation using AI (Claude/OpenAI) and Remotion
triggers:
  - how do I automate content creation with AI
  - set up automated content pipeline from research to video
  - use Claude and OpenAI to generate marketing content
  - create videos automatically from text content
  - build content automation system with remotion
  - crawl news and generate content with AI
  - automate content research and video rendering
  - generate multilingual content with AI pipeline
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI agents to use the Ultimate AI Content Pipeline - a complete automation system that researches content, generates articles in multiple formats using Claude/OpenAI, and automatically renders videos using Remotion.

## What It Does

The Ultimate AI Content Pipeline is a TypeScript-based Next.js application that:

1. **Auto-Research**: Crawls news from TechCrunch, a16z, Twitter/X, LinkedIn for fresh insights
2. **AI Content Generation**: Creates articles in multiple formats (toplist, POV, case study, how-to) using Claude 3 or OpenAI
3. **Multilingual Support**: Generates content in both English and Vietnamese
4. **Video Generation**: Automatically renders infographics and short-form videos using Remotion
5. **Multi-Platform Export**: Optimizes video output for Reels, TikTok, Shorts

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

### Required Environment Variables

```bash
# AI API Keys
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research API Keys
RAPIDAPI_KEY=your_rapidapi_key

# Optional: Database and Storage
DATABASE_URL=your_database_url
NEXT_PUBLIC_API_URL=http://localhost:3000
```

### Run Development Server

```bash
npm run dev
# or
yarn dev
# or
pnpm dev
```

Access at `http://localhost:3000`

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/             
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── research/    # Content research/crawling
│   │   └── video/       # Remotion video generation
│   ├── remotion/        # Remotion video templates
│   └── types/           # TypeScript types
├── public/              # Static assets
└── package.json
```

## Core APIs and Usage

### 1. Content Research API

```typescript
// src/lib/research/crawler.ts
import { RapidAPIClient } from '@/lib/research/rapidapi-client';

interface ResearchResult {
  title: string;
  url: string;
  excerpt: string;
  source: string;
  publishedAt: Date;
}

export async function researchTopic(
  keyword: string,
  sources: string[] = ['techcrunch', 'a16z', 'twitter']
): Promise<ResearchResult[]> {
  const client = new RapidAPIClient(process.env.RAPIDAPI_KEY!);
  
  const results: ResearchResult[] = [];
  
  for (const source of sources) {
    const data = await client.search({
      query: keyword,
      source: source,
      timeRange: '24h'
    });
    
    results.push(...data.articles);
  }
  
  return results;
}

// Usage in API route
export async function POST(request: Request) {
  const { keyword } = await request.json();
  
  const research = await researchTopic(keyword);
  
  return Response.json({ research });
}
```

### 2. AI Content Generation with Claude

```typescript
// src/lib/ai/claude-generator.ts
import Anthropic from '@anthropic-ai/sdk';

interface ContentOptions {
  topic: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  research: ResearchResult[];
}

export async function generateContent(options: ContentOptions): Promise<string> {
  const anthropic = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY!,
  });

  const researchContext = options.research
    .map((r) => `- ${r.title} (${r.source}): ${r.excerpt}`)
    .join('\n');

  const prompt = `You are an expert content writer. Generate a ${options.format} article about "${options.topic}" in ${options.language} with a ${options.tone} tone.

Use this recent research data:
${researchContext}

Requirements:
- Length: 1000-1500 words
- Include data and statistics from the research
- Make it engaging and actionable
- Use proper formatting (headings, bullets, etc.)`;

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
```

### 3. AI Content Generation with OpenAI

```typescript
// src/lib/ai/openai-generator.ts
import OpenAI from 'openai';

export async function generateContentOpenAI(options: ContentOptions): Promise<string> {
  const openai = new OpenAI({
    apiKey: process.env.OPENAI_API_KEY!,
  });

  const researchContext = options.research
    .map((r) => `- ${r.title} (${r.source}): ${r.excerpt}`)
    .join('\n');

  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are an expert content writer specializing in ${options.format} format.`,
      },
      {
        role: 'user',
        content: `Create a ${options.format} article about "${options.topic}" in ${options.language}. Research data:\n${researchContext}`,
      },
    ],
    temperature: 0.7,
    max_tokens: 4096,
  });

  return completion.choices[0].message.content || '';
}
```

### 4. Video Generation with Remotion

```typescript
// src/lib/video/render.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoOptions {
  content: string;
  title: string;
  format: 'reels' | 'tiktok' | 'shorts'; // 9:16
  duration: number; // in seconds
}

export async function renderVideo(options: VideoOptions): Promise<string> {
  const compositionId = 'ContentVideo';
  
  // Bundle Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.join(process.cwd(), 'src/remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  // Get composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: {
      title: options.title,
      content: options.content,
      format: options.format,
    },
  });

  // Render video
  const outputLocation = path.join(
    process.cwd(),
    'public',
    'videos',
    `${Date.now()}-${options.format}.mp4`
  );

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: {
      title: options.title,
      content: options.content,
      format: options.format,
    },
  });

  return outputLocation;
}
```

### 5. Remotion Video Template

```typescript
// src/remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig } from 'remotion';
import { interpolate } from 'remotion';

interface ContentVideoProps {
  title: string;
  content: string;
  format: 'reels' | 'tiktok' | 'shorts';
}

export const ContentVideo: React.FC<ContentVideoProps> = ({
  title,
  content,
  format,
}) => {
  const frame = useCurrentFrame();
  const { fps, durationInFrames } = useVideoConfig();

  const opacity = interpolate(frame, [0, 30], [0, 1], {
    extrapolateRight: 'clamp',
  });

  const scale = interpolate(
    frame,
    [0, 30],
    [0.8, 1],
    {
      extrapolateRight: 'clamp',
    }
  );

  return (
    <AbsoluteFill
      style={{
        backgroundColor: '#1a1a1a',
        justifyContent: 'center',
        alignItems: 'center',
        padding: '40px',
      }}
    >
      <div
        style={{
          opacity,
          transform: `scale(${scale})`,
          textAlign: 'center',
          color: 'white',
        }}
      >
        <h1
          style={{
            fontSize: '48px',
            fontWeight: 'bold',
            marginBottom: '20px',
          }}
        >
          {title}
        </h1>
        <p
          style={{
            fontSize: '24px',
            lineHeight: '1.5',
            maxWidth: '800px',
          }}
        >
          {content.substring(0, 200)}...
        </p>
      </div>
    </AbsoluteFill>
  );
};
```

### 6. Complete Content Pipeline

```typescript
// src/app/api/generate-content/route.ts
import { researchTopic } from '@/lib/research/crawler';
import { generateContent } from '@/lib/ai/claude-generator';
import { renderVideo } from '@/lib/video/render';

export async function POST(request: Request) {
  const { keyword, format, language, generateVideo } = await request.json();

  // Step 1: Research
  const research = await researchTopic(keyword);

  // Step 2: Generate content
  const content = await generateContent({
    topic: keyword,
    format,
    language,
    tone: 'expert',
    research,
  });

  // Step 3: Generate video (optional)
  let videoUrl = null;
  if (generateVideo) {
    const videoPath = await renderVideo({
      content,
      title: keyword,
      format: 'reels',
      duration: 30,
    });
    videoUrl = `/videos/${path.basename(videoPath)}`;
  }

  return Response.json({
    content,
    videoUrl,
    research: research.slice(0, 5),
  });
}
```

## Frontend Integration

```typescript
// src/components/ContentGenerator.tsx
'use client';

import { useState } from 'react';

export default function ContentGenerator() {
  const [keyword, setKeyword] = useState('');
  const [format, setFormat] = useState('toplist');
  const [language, setLanguage] = useState('en');
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState<any>(null);

  const handleGenerate = async () => {
    setLoading(true);
    
    const response = await fetch('/api/generate-content', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        keyword,
        format,
        language,
        generateVideo: true,
      }),
    });

    const data = await response.json();
    setResult(data);
    setLoading(false);
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
          <option value="pov">POV</option>
          <option value="case-study">Case Study</option>
          <option value="how-to">How-to</option>
        </select>
        
        <select
          value={language}
          onChange={(e) => setLanguage(e.target.value)}
          className="w-full p-3 border rounded"
        >
          <option value="en">English</option>
          <option value="vi">Tiếng Việt</option>
        </select>
        
        <button
          onClick={handleGenerate}
          disabled={loading}
          className="w-full p-3 bg-blue-600 text-white rounded hover:bg-blue-700 disabled:bg-gray-400"
        >
          {loading ? 'Generating...' : 'Generate Content & Video'}
        </button>
      </div>

      {result && (
        <div className="mt-8 space-y-6">
          <div className="prose max-w-none">
            <h2>Generated Content</h2>
            <div dangerouslySetInnerHTML={{ __html: result.content }} />
          </div>
          
          {result.videoUrl && (
            <div>
              <h2 className="text-2xl font-bold mb-4">Generated Video</h2>
              <video controls className="w-full rounded">
                <source src={result.videoUrl} type="video/mp4" />
              </video>
            </div>
          )}
        </div>
      )}
    </div>
  );
}
```

## Common Patterns

### Pattern 1: Batch Content Generation

```typescript
// Generate multiple content pieces at once
async function batchGenerate(keywords: string[]) {
  const results = await Promise.all(
    keywords.map(async (keyword) => {
      const research = await researchTopic(keyword);
      const content = await generateContent({
        topic: keyword,
        format: 'toplist',
        language: 'en',
        tone: 'expert',
        research,
      });
      return { keyword, content };
    })
  );
  
  return results;
}
```

### Pattern 2: Content Scheduling

```typescript
// Schedule content for publishing
interface ScheduledContent {
  content: string;
  videoUrl?: string;
  publishAt: Date;
  platforms: ('facebook' | 'twitter' | 'linkedin')[];
}

async function scheduleContent(scheduled: ScheduledContent) {
  // Save to database with scheduled time
  await db.scheduledContent.create({
    data: scheduled,
  });
}
```

### Pattern 3: A/B Testing Content

```typescript
// Generate multiple versions for A/B testing
async function generateVariants(keyword: string, count: number = 3) {
  const research = await researchTopic(keyword);
  
  const variants = await Promise.all(
    Array.from({ length: count }).map(async (_, i) => {
      return await generateContent({
        topic: keyword,
        format: 'toplist',
        language: 'en',
        tone: ['expert', 'friendly', 'humorous'][i % 3] as any,
        research,
      });
    })
  );
  
  return variants;
}
```

## Configuration

### Remotion Config

```typescript
// remotion.config.ts
import { Config } from '@remotion/cli/config';

Config.setVideoImageFormat('jpeg');
Config.setOverwriteOutput(true);
Config.setConcurrency(4);
Config.setCodec('h264');
```

### Next.js Config for Video Support

```javascript
// next.config.js
/** @type {import('next').NextConfig} */
const nextConfig = {
  webpack: (config) => {
    config.externals.push({
      '@remotion/renderer': '@remotion/renderer',
    });
    return config;
  },
};

module.exports = nextConfig;
```

## Troubleshooting

### Issue: AI API Rate Limits

```typescript
// Implement rate limiting and retry logic
import { setTimeout } from 'timers/promises';

async function generateWithRetry(options: ContentOptions, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await generateContent(options);
    } catch (error: any) {
      if (error.status === 429 && i < maxRetries - 1) {
        await setTimeout(2000 * (i + 1)); // Exponential backoff
        continue;
      }
      throw error;
    }
  }
}
```

### Issue: Video Rendering Timeout

```typescript
// Increase timeout for long videos
await renderMedia({
  composition,
  serveUrl: bundleLocation,
  codec: 'h264',
  outputLocation,
  timeoutInMilliseconds: 120000, // 2 minutes
  inputProps: {...},
});
```

### Issue: Research API Failures

```typescript
// Fallback to cached data if API fails
async function researchWithFallback(keyword: string) {
  try {
    return await researchTopic(keyword);
  } catch (error) {
    console.error('Research API failed, using cached data');
    return await getCachedResearch(keyword);
  }
}
```

### Issue: Memory Issues with Large Content

```typescript
// Process content in chunks
function chunkContent(content: string, chunkSize: number = 1000) {
  const chunks = [];
  for (let i = 0; i < content.length; i += chunkSize) {
    chunks.push(content.slice(i, i + chunkSize));
  }
  return chunks;
}
```

## Best Practices

1. **Cache Research Data**: Store research results to avoid repeated API calls
2. **Queue Video Rendering**: Use a job queue for video generation to avoid blocking
3. **Monitor API Usage**: Track API calls to stay within rate limits
4. **Validate Content**: Review AI-generated content before publishing
5. **Optimize Video Size**: Compress videos for faster uploads to social platforms
6. **Use Environment Variables**: Never hardcode API keys
