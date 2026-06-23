---
name: marketing-pipeline-share-automation
description: AI-powered content automation pipeline that researches, generates scripts, and creates videos automatically using Claude/OpenAI and Remotion
triggers:
  - how do I set up the AI content pipeline
  - automate content research and video generation
  - generate marketing content with Claude and OpenAI
  - create automated video content from scripts
  - build a content automation workflow
  - set up Remotion video rendering pipeline
  - crawl news sources for content research
  - automate social media content creation
---

# Marketing Pipeline Share - AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

**Marketing Pipeline Share** is a comprehensive content automation system that handles the entire content creation pipeline: from researching trending topics across news sources (TechCrunch, a16z, Twitter, LinkedIn), to generating multi-format content (articles, scripts) in multiple languages, to rendering videos automatically using Remotion. Built with Next.js and TypeScript, it integrates Claude 3, OpenAI, and various content APIs to create a fully automated content factory.

## Installation

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
yarn install

# Set up environment variables
cp .env.example .env.local
```

### Required Environment Variables

```bash
# AI Models
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Content Research APIs
RAPIDAPI_KEY=your_rapidapi_key
TWITTER_BEARER_TOKEN=your_twitter_bearer_token

# Next.js Configuration
NEXT_PUBLIC_API_URL=http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/                    # Next.js app router
│   ├── components/             # React components
│   ├── lib/
│   │   ├── ai/                # AI integration (Claude, OpenAI)
│   │   ├── crawlers/          # Content crawlers
│   │   ├── generators/        # Content generators
│   │   └── video/             # Remotion video rendering
│   └── types/                 # TypeScript types
├── remotion/                   # Remotion video templates
└── public/                     # Static assets
```

## Core Features & Usage

### 1. Content Research & Crawling

The system automatically crawls multiple sources for trending topics:

```typescript
// src/lib/crawlers/news-crawler.ts
import { fetchTechCrunchArticles } from './sources/techcrunch';
import { fetchTwitterTrends } from './sources/twitter';
import { fetchLinkedInPosts } from './sources/linkedin';

export interface ResearchResult {
  keyword: string;
  articles: Article[];
  trends: Trend[];
  insights: string[];
}

export async function conductResearch(keyword: string): Promise<ResearchResult> {
  const [articles, trends, posts] = await Promise.all([
    fetchTechCrunchArticles(keyword),
    fetchTwitterTrends(keyword),
    fetchLinkedInPosts(keyword)
  ]);

  return {
    keyword,
    articles,
    trends,
    insights: extractInsights(articles, trends, posts)
  };
}

// Usage in API route
// src/app/api/research/route.ts
import { conductResearch } from '@/lib/crawlers/news-crawler';

export async function POST(req: Request) {
  const { keyword } = await req.json();
  const research = await conductResearch(keyword);
  
  return Response.json(research);
}
```

### 2. AI Content Generation

Generate content in multiple formats using Claude or OpenAI:

```typescript
// src/lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY
});

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY
});

export type ContentFormat = 'toplist' | 'pov' | 'case-study' | 'how-to';
export type Language = 'en' | 'vi';
export type Tone = 'expert' | 'friendly' | 'humorous';

interface GenerateContentParams {
  research: ResearchResult;
  format: ContentFormat;
  language: Language;
  tone: Tone;
  provider?: 'claude' | 'openai';
}

export async function generateContent({
  research,
  format,
  language,
  tone,
  provider = 'claude'
}: GenerateContentParams): Promise<string> {
  const prompt = buildPrompt(research, format, language, tone);

  if (provider === 'claude') {
    const message = await anthropic.messages.create({
      model: 'claude-3-5-sonnet-20241022',
      max_tokens: 4096,
      messages: [
        { role: 'user', content: prompt }
      ]
    });
    
    return message.content[0].type === 'text' 
      ? message.content[0].text 
      : '';
  } else {
    const completion = await openai.chat.completions.create({
      model: 'gpt-4-turbo-preview',
      messages: [
        { role: 'user', content: prompt }
      ]
    });
    
    return completion.choices[0].message.content || '';
  }
}

function buildPrompt(
  research: ResearchResult,
  format: ContentFormat,
  language: Language,
  tone: Tone
): string {
  const formatInstructions = {
    'toplist': 'Create a numbered list article with engaging headlines',
    'pov': 'Write from a unique perspective or viewpoint',
    'case-study': 'Analyze with data, examples, and conclusions',
    'how-to': 'Provide step-by-step actionable instructions'
  };

  const toneInstructions = {
    'expert': 'Professional, authoritative, data-driven',
    'friendly': 'Conversational, approachable, warm',
    'humorous': 'Light, entertaining, with wit'
  };

  return `
You are a content creator. Using the following research data, create content in ${language.toUpperCase()}.

Format: ${formatInstructions[format]}
Tone: ${toneInstructions[tone]}

Research Data:
Keyword: ${research.keyword}
Articles: ${JSON.stringify(research.articles.slice(0, 5))}
Trends: ${JSON.stringify(research.trends)}
Key Insights: ${research.insights.join(', ')}

Generate a complete, engaging article (800-1200 words) that:
- Uses data from the research
- Follows the ${format} format
- Maintains a ${tone} tone
- Is optimized for social media sharing
`;
}
```

### 3. Script Generation for Videos

Create video scripts from generated content:

```typescript
// src/lib/generators/script-generator.ts
export interface VideoScript {
  title: string;
  scenes: Scene[];
  totalDuration: number;
}

export interface Scene {
  id: number;
  text: string;
  duration: number;
  visualType: 'text' | 'image' | 'chart';
  visualData?: any;
}

export async function generateVideoScript(
  content: string
): Promise<VideoScript> {
  const prompt = `
Convert the following article into a short-form video script (30-60 seconds).
Break it into 5-7 scenes with clear visual instructions.

Article:
${content}

Output as JSON with this structure:
{
  "title": "Video title",
  "scenes": [
    {
      "id": 1,
      "text": "Scene narration",
      "duration": 5,
      "visualType": "text|image|chart",
      "visualData": {}
    }
  ]
}
`;

  const response = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 2048,
    messages: [{ role: 'user', content: prompt }]
  });

  const scriptText = response.content[0].type === 'text' 
    ? response.content[0].text 
    : '';
  
  const script = JSON.parse(scriptText);
  script.totalDuration = script.scenes.reduce(
    (sum: number, scene: Scene) => sum + scene.duration, 
    0
  );

  return script;
}
```

### 4. Video Rendering with Remotion

Set up Remotion video templates and render:

```typescript
// remotion/Video.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig } from 'remotion';

export const VideoComposition: React.FC<{
  script: VideoScript;
}> = ({ script }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  const currentTime = frame / fps;
  const currentScene = getCurrentScene(script.scenes, currentTime);

  return (
    <AbsoluteFill style={{ backgroundColor: '#000' }}>
      {currentScene && (
        <SceneRenderer scene={currentScene} />
      )}
    </AbsoluteFill>
  );
};

const SceneRenderer: React.FC<{ scene: Scene }> = ({ scene }) => {
  return (
    <AbsoluteFill
      style={{
        justifyContent: 'center',
        alignItems: 'center',
        padding: 40
      }}
    >
      <h1 style={{ color: '#fff', fontSize: 60, textAlign: 'center' }}>
        {scene.text}
      </h1>
    </AbsoluteFill>
  );
};

function getCurrentScene(scenes: Scene[], time: number): Scene | null {
  let elapsed = 0;
  for (const scene of scenes) {
    if (time < elapsed + scene.duration) {
      return scene;
    }
    elapsed += scene.duration;
  }
  return scenes[scenes.length - 1];
}
```

```typescript
// src/lib/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

export async function renderVideo(
  script: VideoScript,
  outputPath: string
): Promise<string> {
  const compositionId = 'VideoComposition';
  const bundleLocation = await bundle(
    path.join(process.cwd(), 'remotion/index.ts')
  );

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: { script }
  });

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: { script }
  });

  return outputPath;
}

// Usage in API route
// src/app/api/render/route.ts
export async function POST(req: Request) {
  const { script } = await req.json();
  const outputPath = path.join(process.cwd(), 'public', 'videos', `${Date.now()}.mp4`);
  
  await renderVideo(script, outputPath);
  
  return Response.json({ 
    videoUrl: `/videos/${path.basename(outputPath)}` 
  });
}
```

### 5. Complete Pipeline API

Orchestrate the entire workflow:

```typescript
// src/app/api/pipeline/route.ts
import { conductResearch } from '@/lib/crawlers/news-crawler';
import { generateContent } from '@/lib/ai/content-generator';
import { generateVideoScript } from '@/lib/generators/script-generator';
import { renderVideo } from '@/lib/video/renderer';

export async function POST(req: Request) {
  const { 
    keyword, 
    format, 
    language, 
    tone,
    generateVideo = false 
  } = await req.json();

  // Step 1: Research
  const research = await conductResearch(keyword);

  // Step 2: Generate Content
  const content = await generateContent({
    research,
    format,
    language,
    tone
  });

  // Step 3: Generate Script (if video requested)
  let script = null;
  let videoUrl = null;

  if (generateVideo) {
    script = await generateVideoScript(content);
    const outputPath = path.join(
      process.cwd(), 
      'public', 
      'videos', 
      `${Date.now()}.mp4`
    );
    await renderVideo(script, outputPath);
    videoUrl = `/videos/${path.basename(outputPath)}`;
  }

  return Response.json({
    success: true,
    data: {
      keyword,
      research,
      content,
      script,
      videoUrl
    }
  });
}
```

## Frontend Integration

```typescript
// src/components/ContentPipeline.tsx
'use client';

import { useState } from 'react';

export default function ContentPipeline() {
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState(null);

  const handleGenerate = async (formData: FormData) => {
    setLoading(true);
    
    const response = await fetch('/api/pipeline', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        keyword: formData.get('keyword'),
        format: formData.get('format'),
        language: formData.get('language'),
        tone: formData.get('tone'),
        generateVideo: formData.get('generateVideo') === 'true'
      })
    });

    const data = await response.json();
    setResult(data);
    setLoading(false);
  };

  return (
    <div className="pipeline-container">
      <form action={handleGenerate}>
        <input name="keyword" placeholder="Enter keyword" required />
        
        <select name="format">
          <option value="toplist">Top List</option>
          <option value="pov">POV</option>
          <option value="case-study">Case Study</option>
          <option value="how-to">How-To</option>
        </select>

        <select name="language">
          <option value="en">English</option>
          <option value="vi">Vietnamese</option>
        </select>

        <select name="tone">
          <option value="expert">Expert</option>
          <option value="friendly">Friendly</option>
          <option value="humorous">Humorous</option>
        </select>

        <label>
          <input type="checkbox" name="generateVideo" value="true" />
          Generate Video
        </label>

        <button type="submit" disabled={loading}>
          {loading ? 'Generating...' : 'Generate Content'}
        </button>
      </form>

      {result && (
        <div className="result">
          <h2>Generated Content</h2>
          <div dangerouslySetInnerHTML={{ __html: result.data.content }} />
          
          {result.data.videoUrl && (
            <video src={result.data.videoUrl} controls />
          )}
        </div>
      )}
    </div>
  );
}
```

## Development Commands

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Render a single video (development)
npx remotion render VideoComposition output.mp4

# Preview Remotion compositions
npx remotion preview
```

## Configuration

### Remotion Configuration

```typescript
// remotion.config.ts
import { Config } from '@remotion/cli/config';

Config.setVideoImageFormat('jpeg');
Config.setOverwriteOutput(true);
Config.setConcurrency(2);
Config.setCodec('h264');
```

### Content Crawler Configuration

```typescript
// src/lib/crawlers/config.ts
export const CRAWLER_CONFIG = {
  sources: [
    'techcrunch',
    'a16z',
    'twitter',
    'linkedin'
  ],
  maxArticlesPerSource: 10,
  timeRange: '24h',
  retryAttempts: 3,
  timeout: 30000
};
```

## Common Patterns

### Batch Content Generation

```typescript
// Generate multiple content pieces at once
export async function batchGenerate(keywords: string[]) {
  const results = await Promise.all(
    keywords.map(async (keyword) => {
      const research = await conductResearch(keyword);
      const content = await generateContent({
        research,
        format: 'toplist',
        language: 'en',
        tone: 'expert'
      });
      return { keyword, content };
    })
  );

  return results;
}
```

### Scheduled Content Pipeline

```typescript
// Run pipeline on a schedule (using cron or similar)
import cron from 'node-cron';

cron.schedule('0 9 * * *', async () => {
  const trendingKeywords = await fetchTrendingKeywords();
  
  for (const keyword of trendingKeywords) {
    await fetch('http://localhost:3000/api/pipeline', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        keyword,
        format: 'pov',
        language: 'vi',
        tone: 'friendly',
        generateVideo: true
      })
    });
  }
});
```

## Troubleshooting

### API Rate Limits

If you encounter rate limits with AI providers:

```typescript
// Implement exponential backoff
async function retryWithBackoff<T>(
  fn: () => Promise<T>,
  maxRetries = 3
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      await new Promise(resolve => 
        setTimeout(resolve, Math.pow(2, i) * 1000)
      );
    }
  }
  throw new Error('Max retries exceeded');
}
```

### Video Rendering Memory Issues

For large videos, adjust Node.js memory:

```bash
NODE_OPTIONS=--max-old-space-size=4096 npm run dev
```

### Missing Environment Variables

Validate environment variables on startup:

```typescript
// src/lib/config/validate-env.ts
const requiredEnvVars = [
  'ANTHROPIC_API_KEY',
  'OPENAI_API_KEY',
  'RAPIDAPI_KEY'
];

export function validateEnv() {
  const missing = requiredEnvVars.filter(key => !process.env[key]);
  
  if (missing.length > 0) {
    throw new Error(
      `Missing required environment variables: ${missing.join(', ')}`
    );
  }
}
```

## Performance Optimization

### Caching Research Results

```typescript
// Use Redis or similar for caching
import { Redis } from '@upstash/redis';

const redis = new Redis({
  url: process.env.UPSTASH_REDIS_URL,
  token: process.env.UPSTASH_REDIS_TOKEN
});

export async function conductResearchCached(keyword: string) {
  const cached = await redis.get(`research:${keyword}`);
  if (cached) return cached;

  const research = await conductResearch(keyword);
  await redis.setex(`research:${keyword}`, 86400, research); // 24h cache
  
  return research;
}
```

This skill provides comprehensive coverage of the Marketing Pipeline Share automation system, enabling AI agents to help developers implement end-to-end content automation workflows.
