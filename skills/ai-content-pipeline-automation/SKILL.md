---
name: ai-content-pipeline-automation
description: Complete AI-powered content automation pipeline from research to video generation using Claude, OpenAI, and Remotion
triggers:
  - automate content creation with AI research and video generation
  - set up AI content pipeline with auto-research and rendering
  - create automated marketing content workflow with Claude and OpenAI
  - build content automation system with web scraping and video
  - generate videos and articles automatically from keywords
  - implement AI content research and publishing pipeline
  - automate social media content with AI and Remotion
  - create end-to-end content generation workflow
---

# AI Content Pipeline Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the Ultimate AI Content Pipeline, a complete automation system that transforms keywords into polished content and videos through AI-powered research, scriptwriting, and video generation.

## What This Project Does

The AI Content Pipeline is an end-to-end content automation system that:

- **Auto-crawls** news sources (TechCrunch, a16z, Twitter/X, LinkedIn) for fresh data within 24 hours
- **Generates content** in multiple formats (listicles, POV articles, case studies, how-tos) using Claude 3 or OpenAI
- **Creates bilingual content** (English and Vietnamese) with customizable tone
- **Renders videos automatically** using Remotion for social media (Reels, TikTok, Shorts)
- **Provides a Next.js interface** for managing the entire workflow

## Installation

### Prerequisites

```bash
node >= 18.0.0
npm or yarn or pnpm
```

### Basic Setup

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

### Environment Configuration

Create a `.env.local` file in the root directory:

```bash
# AI API Keys
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research & Scraping
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Remotion Configuration
REMOTION_LICENSE_KEY=your_remotion_license

# Optional: Social Media APIs for auto-posting
FACEBOOK_ACCESS_TOKEN=your_facebook_token
TWITTER_API_KEY=your_twitter_key
```

### Run Development Server

```bash
npm run dev
# or
yarn dev
# or
pnpm dev
```

Navigate to `http://localhost:3000`

## Key Components & Architecture

### 1. Research Module (Auto-Scraping)

The research module automatically crawls content sources:

```typescript
// lib/research/scraper.ts
import axios from 'axios';

interface ResearchConfig {
  sources: string[];
  timeframe: number; // hours
  keywords: string[];
}

export async function conductResearch(config: ResearchConfig) {
  const { sources, timeframe, keywords } = config;
  
  const results = await Promise.all(
    sources.map(source => 
      fetchSourceData(source, timeframe, keywords)
    )
  );
  
  return aggregateInsights(results);
}

async function fetchSourceData(
  source: string, 
  timeframe: number, 
  keywords: string[]
) {
  const options = {
    method: 'GET',
    url: `https://api.rapidapi.com/${source}/search`,
    params: {
      q: keywords.join(' OR '),
      timeframe: `${timeframe}h`
    },
    headers: {
      'X-RapidAPI-Key': process.env.RAPIDAPI_KEY,
      'X-RapidAPI-Host': `${source}.rapidapi.com`
    }
  };
  
  const response = await axios.request(options);
  return response.data;
}
```

### 2. Content Generation with AI

Generate content using Claude or OpenAI:

```typescript
// lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

type ContentFormat = 'toplist' | 'pov' | 'case-study' | 'how-to';
type Language = 'en' | 'vi';
type Tone = 'expert' | 'friendly' | 'humorous';

interface ContentRequest {
  keyword: string;
  format: ContentFormat;
  language: Language;
  tone: Tone;
  researchData: any[];
}

export async function generateContent(
  request: ContentRequest,
  provider: 'claude' | 'openai' = 'claude'
) {
  const prompt = buildPrompt(request);
  
  if (provider === 'claude') {
    return generateWithClaude(prompt);
  } else {
    return generateWithOpenAI(prompt);
  }
}

async function generateWithClaude(prompt: string) {
  const client = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY
  });
  
  const message = await client.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [
      {
        role: 'user',
        content: prompt
      }
    ]
  });
  
  return message.content[0].text;
}

async function generateWithOpenAI(prompt: string) {
  const client = new OpenAI({
    apiKey: process.env.OPENAI_API_KEY
  });
  
  const completion = await client.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: 'You are an expert content creator specializing in marketing content.'
      },
      {
        role: 'user',
        content: prompt
      }
    ],
    max_tokens: 4096
  });
  
  return completion.choices[0].message.content;
}

function buildPrompt(request: ContentRequest): string {
  const { keyword, format, language, tone, researchData } = request;
  
  const researchSummary = researchData
    .map(item => `- ${item.title}: ${item.summary}`)
    .join('\n');
  
  return `
Create a ${format} article about "${keyword}" in ${language} with a ${tone} tone.

Use this recent research data:
${researchSummary}

Requirements:
- Format: ${format}
- Language: ${language}
- Tone: ${tone}
- Include data-backed insights
- Make it engaging and actionable
- Optimize for social media sharing
`;
}
```

### 3. Video Generation with Remotion

Render videos from generated content:

```typescript
// lib/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  content: string;
  title: string;
  platform: 'reels' | 'tiktok' | 'shorts';
}

const PLATFORM_DIMENSIONS = {
  reels: { width: 1080, height: 1920 },
  tiktok: { width: 1080, height: 1920 },
  shorts: { width: 1080, height: 1920 }
};

export async function renderVideo(config: VideoConfig): Promise<string> {
  const { content, title, platform } = config;
  const dimensions = PLATFORM_DIMENSIONS[platform];
  
  // Bundle Remotion project
  const bundleLocation = await bundle(
    path.join(process.cwd(), 'remotion', 'index.ts')
  );
  
  // Get composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title,
      content: parseContentForVideo(content),
      ...dimensions
    }
  });
  
  // Render video
  const outputPath = path.join(
    process.cwd(),
    'public',
    'videos',
    `${Date.now()}-${platform}.mp4`
  );
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: {
      title,
      content: parseContentForVideo(content)
    }
  });
  
  return outputPath;
}

function parseContentForVideo(content: string) {
  // Extract key points for video scenes
  const lines = content.split('\n').filter(line => line.trim());
  const scenes = [];
  
  for (let i = 0; i < lines.length && scenes.length < 5; i++) {
    if (lines[i].match(/^#{1,3}\s/) || lines[i].match(/^\d+\./)) {
      scenes.push({
        text: lines[i].replace(/^#{1,3}\s/, '').replace(/^\d+\.\s/, ''),
        duration: 3 // seconds per scene
      });
    }
  }
  
  return scenes;
}
```

### 4. Remotion Video Component

```tsx
// remotion/ContentVideo.tsx
import { useCurrentFrame, useVideoConfig, spring, AbsoluteFill } from 'remotion';

interface ContentVideoProps {
  title: string;
  content: Array<{ text: string; duration: number }>;
}

export const ContentVideo: React.FC<ContentVideoProps> = ({ title, content }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  
  // Calculate which scene to show
  let currentScene = 0;
  let elapsedFrames = 0;
  
  for (let i = 0; i < content.length; i++) {
    const sceneDuration = content[i].duration * fps;
    if (frame < elapsedFrames + sceneDuration) {
      currentScene = i;
      break;
    }
    elapsedFrames += sceneDuration;
  }
  
  const scene = content[currentScene];
  const sceneFrame = frame - elapsedFrames;
  
  // Animation
  const scale = spring({
    frame: sceneFrame,
    fps,
    from: 0.8,
    to: 1,
    durationInFrames: 30
  });
  
  const opacity = spring({
    frame: sceneFrame,
    fps,
    from: 0,
    to: 1,
    durationInFrames: 20
  });
  
  return (
    <AbsoluteFill
      style={{
        backgroundColor: '#1a1a2e',
        justifyContent: 'center',
        alignItems: 'center',
        padding: 40
      }}
    >
      <div
        style={{
          transform: `scale(${scale})`,
          opacity,
          color: 'white',
          fontSize: 48,
          fontWeight: 'bold',
          textAlign: 'center',
          fontFamily: 'Arial, sans-serif',
          lineHeight: 1.4
        }}
      >
        {scene.text}
      </div>
    </AbsoluteFill>
  );
};
```

## Complete Workflow Example

```typescript
// pages/api/generate-content.ts
import { NextApiRequest, NextApiResponse } from 'next';
import { conductResearch } from '@/lib/research/scraper';
import { generateContent } from '@/lib/ai/content-generator';
import { renderVideo } from '@/lib/video/renderer';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }
  
  try {
    const { keyword, format, language, tone, platform } = req.body;
    
    // Step 1: Conduct research
    const researchData = await conductResearch({
      sources: ['techcrunch', 'a16z', 'twitter'],
      timeframe: 24,
      keywords: [keyword]
    });
    
    // Step 2: Generate content
    const content = await generateContent({
      keyword,
      format,
      language,
      tone,
      researchData
    }, 'claude');
    
    // Step 3: Render video
    const videoPath = await renderVideo({
      content,
      title: keyword,
      platform
    });
    
    // Step 4: Return results
    res.status(200).json({
      success: true,
      content,
      videoUrl: `/videos/${path.basename(videoPath)}`
    });
    
  } catch (error) {
    console.error('Content generation error:', error);
    res.status(500).json({ 
      error: 'Failed to generate content',
      details: error.message 
    });
  }
}
```

## Frontend Integration

```tsx
// components/ContentGenerator.tsx
import { useState } from 'react';

export default function ContentGenerator() {
  const [keyword, setKeyword] = useState('');
  const [result, setResult] = useState(null);
  const [loading, setLoading] = useState(false);
  
  const handleGenerate = async () => {
    setLoading(true);
    
    try {
      const response = await fetch('/api/generate-content', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          keyword,
          format: 'toplist',
          language: 'en',
          tone: 'expert',
          platform: 'reels'
        })
      });
      
      const data = await response.json();
      setResult(data);
    } catch (error) {
      console.error('Generation failed:', error);
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
          <h2 className="text-2xl font-bold mb-4">Generated Content</h2>
          <div className="prose mb-8">
            {result.content}
          </div>
          
          {result.videoUrl && (
            <div>
              <h3 className="text-xl font-bold mb-2">Video</h3>
              <video src={result.videoUrl} controls className="w-full max-w-md" />
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
// lib/batch/processor.ts
async function batchGenerate(keywords: string[]) {
  const results = await Promise.allSettled(
    keywords.map(keyword => 
      generateContent({
        keyword,
        format: 'toplist',
        language: 'en',
        tone: 'expert',
        researchData: []
      })
    )
  );
  
  return results
    .filter(r => r.status === 'fulfilled')
    .map(r => r.value);
}
```

### Scheduled Content Creation

```typescript
// lib/scheduler/cron.ts
import cron from 'node-cron';

// Run every day at 9 AM
cron.schedule('0 9 * * *', async () => {
  const trendingTopics = await fetchTrendingTopics();
  
  for (const topic of trendingTopics) {
    await generateContent({
      keyword: topic,
      format: 'pov',
      language: 'en',
      tone: 'friendly',
      researchData: []
    });
  }
});
```

## Troubleshooting

### API Rate Limits

```typescript
// lib/utils/rate-limiter.ts
import pLimit from 'p-limit';

const limit = pLimit(5); // Max 5 concurrent requests

export async function rateLimitedGenerate(requests: ContentRequest[]) {
  return Promise.all(
    requests.map(req => 
      limit(() => generateContent(req))
    )
  );
}
```

### Video Rendering Errors

```typescript
// Check Remotion logs
console.log('Rendering video with config:', config);

// Ensure output directory exists
import fs from 'fs';
const outputDir = path.join(process.cwd(), 'public', 'videos');
if (!fs.existsSync(outputDir)) {
  fs.mkdirSync(outputDir, { recursive: true });
}
```

### Memory Issues with Large Content

```typescript
// Use streaming for large content
import { Readable } from 'stream';

async function streamContent(content: string) {
  const chunks = content.match(/.{1,1000}/g) || [];
  return Readable.from(chunks);
}
```

### Claude API Timeout

```typescript
// Implement retry logic
async function generateWithRetry(prompt: string, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await generateWithClaude(prompt);
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      await new Promise(resolve => setTimeout(resolve, 2000 * (i + 1)));
    }
  }
}
```

This skill enables comprehensive content automation from research through video generation, leveraging multiple AI providers and modern video rendering technology.
