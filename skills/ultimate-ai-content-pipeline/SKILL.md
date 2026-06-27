---
name: ultimate-ai-content-pipeline
description: Automated content creation pipeline from research to video generation using Claude, OpenAI, and Remotion
triggers:
  - how do I use the AI content pipeline
  - set up automated content generation
  - create content with AI research and video
  - generate content from keywords automatically
  - use remotion for video generation
  - configure claude and openai for content
  - automate social media content creation
  - build content pipeline with AI
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

Ultimate AI Content Pipeline is an end-to-end automated content creation system that handles research, scriptwriting, and video generation. It crawls real-time data from sources like TechCrunch and Twitter, uses Claude/OpenAI to generate content in multiple formats and languages, and renders videos using Remotion.

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
cp .env.example .env
```

## Configuration

Create a `.env` file in the project root with the following variables:

```bash
# AI Services
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Database (if using)
DATABASE_URL=your_database_url

# Next.js
NEXT_PUBLIC_API_URL=http://localhost:3000
```

## Core Components

### 1. Research & Data Crawling

The system automatically crawls fresh content from multiple sources:

```typescript
// src/services/research.ts
interface ResearchSource {
  name: string;
  url: string;
  timeframe: '24h' | '7d' | '30d';
}

export async function crawlContent(keyword: string, sources: ResearchSource[]) {
  const results = await Promise.all(
    sources.map(async (source) => {
      const response = await fetch(source.url, {
        headers: {
          'X-RapidAPI-Key': process.env.RAPIDAPI_KEY,
        },
      });
      return response.json();
    })
  );
  
  return aggregateInsights(results);
}

export function aggregateInsights(rawData: any[]) {
  return {
    trends: extractTrends(rawData),
    dataPoints: extractDataPoints(rawData),
    quotes: extractQuotes(rawData),
    sources: rawData.map(d => d.source),
  };
}
```

### 2. AI Content Generation

Generate content in multiple formats using Claude or OpenAI:

```typescript
// src/services/ai-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

interface ContentRequest {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  tone: 'expert' | 'friendly' | 'humorous';
  language: 'en' | 'vi';
  researchData: any;
}

export async function generateContent(request: ContentRequest) {
  const prompt = buildPrompt(request);
  
  // Using Claude
  const message = await anthropic.messages.create({
    model: 'claude-3-opus-20240229',
    max_tokens: 4096,
    messages: [
      {
        role: 'user',
        content: prompt,
      },
    ],
  });
  
  return {
    content: message.content[0].text,
    metadata: {
      model: 'claude-3-opus',
      tokens: message.usage,
    },
  };
}

function buildPrompt(request: ContentRequest): string {
  return `
You are a content creator specializing in ${request.format} articles.

Keyword: ${request.keyword}
Tone: ${request.tone}
Language: ${request.language}

Research Data:
${JSON.stringify(request.researchData, null, 2)}

Create a comprehensive article that:
1. Incorporates the latest trends and data
2. Uses specific examples and data points
3. Maintains the requested tone
4. Follows the ${request.format} structure
5. Is optimized for ${request.language === 'en' ? 'English' : 'Vietnamese'} readers

Article:
`;
}
```

### 3. Video Generation with Remotion

Transform content into videos automatically:

```typescript
// src/services/video-generator.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import { webpackOverride } from './webpack-override';

interface VideoConfig {
  title: string;
  content: string[];
  style: 'reels' | 'tiktok' | 'shorts';
  duration: number;
}

export async function generateVideo(config: VideoConfig) {
  const bundleLocation = await bundle({
    entryPoint: './src/remotion/index.ts',
    webpackOverride,
  });

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title: config.title,
      slides: config.content,
      style: config.style,
    },
  });

  const outputLocation = `./public/videos/${Date.now()}.mp4`;

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
  });

  return outputLocation;
}
```

### 4. Remotion Video Component

```typescript
// src/remotion/ContentVideo.tsx
import { AbsoluteFill, Sequence, useCurrentFrame, useVideoConfig } from 'remotion';

interface ContentVideoProps {
  title: string;
  slides: string[];
  style: 'reels' | 'tiktok' | 'shorts';
}

export const ContentVideo: React.FC<ContentVideoProps> = ({ title, slides, style }) => {
  const frame = useCurrentFrame();
  const { fps, durationInFrames } = useVideoConfig();
  
  const slideFrames = Math.floor(durationInFrames / slides.length);

  return (
    <AbsoluteFill style={{ backgroundColor: '#000' }}>
      <Sequence from={0} durationInFrames={fps * 2}>
        <Title text={title} />
      </Sequence>
      
      {slides.map((slide, index) => (
        <Sequence
          key={index}
          from={(index + 1) * slideFrames}
          durationInFrames={slideFrames}
        >
          <Slide text={slide} index={index} />
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};

const Title: React.FC<{ text: string }> = ({ text }) => {
  const frame = useCurrentFrame();
  const opacity = Math.min(1, frame / 30);
  
  return (
    <div style={{ 
      fontSize: 60, 
      color: 'white', 
      textAlign: 'center',
      opacity 
    }}>
      {text}
    </div>
  );
};

const Slide: React.FC<{ text: string; index: number }> = ({ text, index }) => {
  return (
    <div style={{ 
      fontSize: 40, 
      color: 'white', 
      padding: 40 
    }}>
      {text}
    </div>
  );
};
```

## API Routes

### Create Content Pipeline

```typescript
// pages/api/content/generate.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { crawlContent } from '@/services/research';
import { generateContent } from '@/services/ai-generator';
import { generateVideo } from '@/services/video-generator';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  const { keyword, format, tone, language, includeVideo } = req.body;

  try {
    // Step 1: Research
    const researchData = await crawlContent(keyword, [
      { name: 'TechCrunch', url: 'https://api.techcrunch.com/search', timeframe: '24h' },
      { name: 'Twitter', url: 'https://api.twitter.com/search', timeframe: '24h' },
    ]);

    // Step 2: Generate Content
    const content = await generateContent({
      keyword,
      format,
      tone,
      language,
      researchData,
    });

    // Step 3: Generate Video (optional)
    let videoUrl = null;
    if (includeVideo) {
      const slides = extractKeyPoints(content.content);
      const videoPath = await generateVideo({
        title: keyword,
        content: slides,
        style: 'reels',
        duration: 60,
      });
      videoUrl = `/videos/${videoPath}`;
    }

    res.status(200).json({
      success: true,
      content: content.content,
      metadata: content.metadata,
      videoUrl,
    });
  } catch (error) {
    console.error('Content generation error:', error);
    res.status(500).json({ error: 'Failed to generate content' });
  }
}

function extractKeyPoints(content: string): string[] {
  // Extract 5-7 key points for video slides
  const sentences = content.split('. ');
  return sentences.slice(0, 7);
}
```

## Usage Examples

### From Next.js Frontend

```typescript
// components/ContentGenerator.tsx
import { useState } from 'react';

export default function ContentGenerator() {
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState(null);

  const generateContent = async () => {
    setLoading(true);
    
    const response = await fetch('/api/content/generate', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        keyword: 'AI Marketing Trends 2024',
        format: 'toplist',
        tone: 'expert',
        language: 'en',
        includeVideo: true,
      }),
    });

    const data = await response.json();
    setResult(data);
    setLoading(false);
  };

  return (
    <div>
      <button onClick={generateContent} disabled={loading}>
        {loading ? 'Generating...' : 'Generate Content'}
      </button>
      
      {result && (
        <div>
          <h2>Generated Content</h2>
          <div dangerouslySetInnerHTML={{ __html: result.content }} />
          
          {result.videoUrl && (
            <video src={result.videoUrl} controls />
          )}
        </div>
      )}
    </div>
  );
}
```

### Batch Content Generation

```typescript
// scripts/batch-generate.ts
import { crawlContent } from './src/services/research';
import { generateContent } from './src/services/ai-generator';

const keywords = [
  'AI Marketing',
  'Social Media Automation',
  'Content Strategy 2024',
];

async function batchGenerate() {
  for (const keyword of keywords) {
    console.log(`Processing: ${keyword}`);
    
    const research = await crawlContent(keyword, [
      { name: 'TechCrunch', url: 'https://api.techcrunch.com/search', timeframe: '24h' },
    ]);

    const content = await generateContent({
      keyword,
      format: 'toplist',
      tone: 'expert',
      language: 'en',
      researchData: research,
    });

    // Save to database or file
    console.log(`Generated content for ${keyword}`);
  }
}

batchGenerate();
```

## Common Patterns

### Multi-language Content Generation

```typescript
async function generateMultilingualContent(keyword: string) {
  const research = await crawlContent(keyword, sources);
  
  const [englishContent, vietnameseContent] = await Promise.all([
    generateContent({
      keyword,
      format: 'how-to',
      tone: 'friendly',
      language: 'en',
      researchData: research,
    }),
    generateContent({
      keyword,
      format: 'how-to',
      tone: 'friendly',
      language: 'vi',
      researchData: research,
    }),
  ]);

  return { en: englishContent, vi: vietnameseContent };
}
```

### Scheduled Content Pipeline

```typescript
// lib/scheduler.ts
import cron from 'node-cron';

export function scheduleContentGeneration() {
  // Run every day at 9 AM
  cron.schedule('0 9 * * *', async () => {
    console.log('Starting daily content generation...');
    
    const trendingTopics = await fetchTrendingTopics();
    
    for (const topic of trendingTopics) {
      await generateAndPublish(topic);
    }
  });
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

# Run video rendering locally
npm run remotion:render

# Preview Remotion compositions
npm run remotion:preview
```

## Troubleshooting

### API Rate Limits

If you encounter rate limits:

```typescript
// Add retry logic with exponential backoff
async function generateWithRetry(request: ContentRequest, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await generateContent(request);
    } catch (error) {
      if (error.status === 429 && i < maxRetries - 1) {
        await new Promise(resolve => setTimeout(resolve, 2 ** i * 1000));
        continue;
      }
      throw error;
    }
  }
}
```

### Video Rendering Memory Issues

For large video projects:

```typescript
// Adjust Remotion settings
await renderMedia({
  composition,
  serveUrl: bundleLocation,
  codec: 'h264',
  outputLocation,
  chromiumOptions: {
    headless: true,
    args: ['--no-sandbox', '--disable-setuid-sandbox'],
  },
  concurrency: 1, // Reduce concurrency for memory-constrained environments
});
```

### Missing Research Data

```typescript
// Implement fallback sources
async function crawlWithFallback(keyword: string) {
  const primarySources = [...];
  const fallbackSources = [...];
  
  try {
    return await crawlContent(keyword, primarySources);
  } catch (error) {
    console.warn('Primary sources failed, using fallback');
    return await crawlContent(keyword, fallbackSources);
  }
}
```
