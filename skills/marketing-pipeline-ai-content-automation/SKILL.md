---
name: marketing-pipeline-ai-content-automation
description: Automate content creation from research to video generation using AI (Claude, OpenAI) and Remotion
triggers:
  - how do I automate content creation with AI
  - set up the marketing pipeline for automated posts
  - generate video content from text automatically
  - create AI-powered content research pipeline
  - build automated social media content workflow
  - use Claude and OpenAI for content generation
  - render videos from blog posts with Remotion
  - scrape news sources for content ideas with AI
---

# Marketing Pipeline AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI agents to help developers build and use an automated content pipeline that handles research, scriptwriting, and video generation using Claude 3, OpenAI, and Remotion.

## What This Project Does

The Ultimate AI Content Pipeline is a TypeScript-based automation system that:

- **Auto-researches** trending topics by crawling news sources (TechCrunch, a16z, Twitter, LinkedIn)
- **Generates content** in multiple formats (top lists, POV articles, case studies, how-tos) using Claude/OpenAI
- **Supports bilingual output** (English and Vietnamese) with customizable tone
- **Renders videos** automatically from written content using Remotion
- **Optimizes for platforms** like Reels, TikTok, and YouTube Shorts

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

Create a `.env.local` file in the project root:

```bash
# AI Services
ANTHROPIC_API_KEY=your_claude_api_key_here
OPENAI_API_KEY=your_openai_api_key_here

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key_here

# Optional: Video Rendering
REMOTION_LICENSE_KEY=your_remotion_license_here

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Next.js Config
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── research/    # Content scraping logic
│   │   ├── video/       # Remotion video generation
│   │   └── utils/       # Helper functions
│   └── types/           # TypeScript type definitions
├── public/              # Static assets
└── remotion/           # Remotion video templates
```

## Core Usage Patterns

### 1. Research & Content Scraping

```typescript
// src/lib/research/scraper.ts
import axios from 'axios';

interface ResearchSource {
  url: string;
  title: string;
  content: string;
  publishedAt: Date;
}

export async function scrapeNewsForTopic(
  topic: string,
  sources: string[] = ['techcrunch', 'a16z']
): Promise<ResearchSource[]> {
  const results: ResearchSource[] = [];
  
  const options = {
    method: 'GET',
    url: 'https://news-api.p.rapidapi.com/search',
    params: {
      q: topic,
      time: '24h',
      sources: sources.join(',')
    },
    headers: {
      'X-RapidAPI-Key': process.env.RAPIDAPI_KEY,
      'X-RapidAPI-Host': 'news-api.p.rapidapi.com'
    }
  };

  try {
    const response = await axios.request(options);
    return response.data.articles.map((article: any) => ({
      url: article.url,
      title: article.title,
      content: article.description,
      publishedAt: new Date(article.publishedAt)
    }));
  } catch (error) {
    console.error('Research scraping error:', error);
    return [];
  }
}
```

### 2. AI Content Generation with Claude

```typescript
// src/lib/ai/claude-generator.ts
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

interface ContentParams {
  topic: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  tone: 'expert' | 'friendly' | 'humorous';
  language: 'en' | 'vi';
  researchData: string;
}

export async function generateContentWithClaude(
  params: ContentParams
): Promise<string> {
  const { topic, format, tone, language, researchData } = params;

  const systemPrompt = `You are a professional content writer specializing in ${format} articles. 
Your tone should be ${tone}. Write in ${language === 'en' ? 'English' : 'Vietnamese'}.`;

  const userPrompt = `Based on this research data:
${researchData}

Create a ${format} article about "${topic}". 
Include:
- Engaging headline
- Data-backed insights
- Actionable takeaways
- SEO-optimized structure`;

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4000,
    system: systemPrompt,
    messages: [
      {
        role: 'user',
        content: userPrompt
      }
    ]
  });

  return message.content[0].type === 'text' 
    ? message.content[0].text 
    : '';
}
```

### 3. Alternative: OpenAI Content Generation

```typescript
// src/lib/ai/openai-generator.ts
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

export async function generateContentWithOpenAI(
  params: ContentParams
): Promise<string> {
  const { topic, format, tone, language, researchData } = params;

  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are a ${tone} content writer creating ${format} articles in ${language}.`
      },
      {
        role: 'user',
        content: `Research: ${researchData}\n\nCreate a ${format} about: ${topic}`
      }
    ],
    temperature: 0.7,
    max_tokens: 3000
  });

  return completion.choices[0].message.content || '';
}
```

### 4. Video Generation with Remotion

```typescript
// src/lib/video/render-video.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  title: string;
  content: string[];
  backgroundColor: string;
  platform: 'reels' | 'tiktok' | 'shorts';
}

const PLATFORM_DIMENSIONS = {
  reels: { width: 1080, height: 1920 },
  tiktok: { width: 1080, height: 1920 },
  shorts: { width: 1080, height: 1920 }
};

export async function renderContentVideo(
  config: VideoConfig,
  outputPath: string
): Promise<string> {
  const { platform, title, content, backgroundColor } = config;
  const dimensions = PLATFORM_DIMENSIONS[platform];

  // Bundle Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  // Get composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title,
      slides: content,
      backgroundColor
    },
  });

  // Render video
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: {
      title,
      slides: content,
      backgroundColor
    },
  });

  return outputPath;
}
```

### 5. Remotion Video Component

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, Sequence, useCurrentFrame, useVideoConfig } from 'remotion';
import React from 'react';

interface ContentVideoProps {
  title: string;
  slides: string[];
  backgroundColor: string;
}

export const ContentVideo: React.FC<ContentVideoProps> = ({
  title,
  slides,
  backgroundColor
}) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  const slideDuration = 3 * fps; // 3 seconds per slide

  return (
    <AbsoluteFill style={{ backgroundColor }}>
      <Sequence durationInFrames={2 * fps}>
        <AbsoluteFill style={{
          justifyContent: 'center',
          alignItems: 'center',
          padding: 40
        }}>
          <h1 style={{
            fontSize: 80,
            fontWeight: 'bold',
            color: 'white',
            textAlign: 'center',
            textShadow: '2px 2px 4px rgba(0,0,0,0.5)'
          }}>
            {title}
          </h1>
        </AbsoluteFill>
      </Sequence>

      {slides.map((slide, index) => (
        <Sequence
          key={index}
          from={(index + 1) * slideDuration}
          durationInFrames={slideDuration}
        >
          <AbsoluteFill style={{
            justifyContent: 'center',
            alignItems: 'center',
            padding: 60
          }}>
            <p style={{
              fontSize: 50,
              color: 'white',
              textAlign: 'center',
              lineHeight: 1.6
            }}>
              {slide}
            </p>
          </AbsoluteFill>
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};
```

### 6. Complete Pipeline Orchestration

```typescript
// src/lib/pipeline/orchestrator.ts
import { scrapeNewsForTopic } from '../research/scraper';
import { generateContentWithClaude } from '../ai/claude-generator';
import { renderContentVideo } from '../video/render-video';

interface PipelineConfig {
  topic: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  tone: 'expert' | 'friendly' | 'humorous';
  language: 'en' | 'vi';
  generateVideo: boolean;
  platform?: 'reels' | 'tiktok' | 'shorts';
}

export async function runContentPipeline(config: PipelineConfig) {
  // Step 1: Research
  console.log(`🔍 Researching topic: ${config.topic}`);
  const research = await scrapeNewsForTopic(config.topic);
  const researchData = research
    .map(r => `${r.title}: ${r.content}`)
    .join('\n\n');

  // Step 2: Generate Content
  console.log(`✍️ Generating ${config.format} content...`);
  const content = await generateContentWithClaude({
    topic: config.topic,
    format: config.format,
    tone: config.tone,
    language: config.language,
    researchData
  });

  // Step 3: Generate Video (Optional)
  let videoPath: string | null = null;
  if (config.generateVideo && config.platform) {
    console.log(`🎬 Rendering video for ${config.platform}...`);
    
    // Extract key points for video slides
    const slides = content
      .split('\n')
      .filter(line => line.trim().length > 0)
      .slice(0, 5); // First 5 paragraphs

    videoPath = await renderContentVideo(
      {
        title: config.topic,
        content: slides,
        backgroundColor: '#1a1a1a',
        platform: config.platform
      },
      `./output/${config.topic.replace(/\s+/g, '-')}.mp4`
    );
  }

  return {
    content,
    research,
    videoPath
  };
}
```

### 7. Next.js API Route Example

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline/orchestrator';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    
    const {
      topic,
      format = 'toplist',
      tone = 'friendly',
      language = 'en',
      generateVideo = false,
      platform = 'reels'
    } = body;

    if (!topic) {
      return NextResponse.json(
        { error: 'Topic is required' },
        { status: 400 }
      );
    }

    const result = await runContentPipeline({
      topic,
      format,
      tone,
      language,
      generateVideo,
      platform
    });

    return NextResponse.json({
      success: true,
      data: result
    });
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { error: 'Failed to generate content' },
      { status: 500 }
    );
  }
}
```

### 8. Frontend Component Usage

```typescript
// src/components/ContentGenerator.tsx
'use client';

import { useState } from 'react';

export default function ContentGenerator() {
  const [topic, setTopic] = useState('');
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState<any>(null);

  const handleGenerate = async () => {
    setLoading(true);
    try {
      const response = await fetch('/api/generate', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          topic,
          format: 'toplist',
          tone: 'friendly',
          language: 'en',
          generateVideo: true,
          platform: 'reels'
        })
      });

      const data = await response.json();
      setResult(data.data);
    } catch (error) {
      console.error('Generation failed:', error);
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="max-w-4xl mx-auto p-6">
      <h1 className="text-3xl font-bold mb-6">AI Content Generator</h1>
      
      <div className="space-y-4">
        <input
          type="text"
          value={topic}
          onChange={(e) => setTopic(e.target.value)}
          placeholder="Enter your topic..."
          className="w-full p-3 border rounded-lg"
        />
        
        <button
          onClick={handleGenerate}
          disabled={loading || !topic}
          className="px-6 py-3 bg-blue-600 text-white rounded-lg disabled:opacity-50"
        >
          {loading ? 'Generating...' : 'Generate Content'}
        </button>

        {result && (
          <div className="mt-6 space-y-4">
            <div className="p-4 bg-gray-50 rounded-lg">
              <h3 className="font-bold mb-2">Generated Content:</h3>
              <pre className="whitespace-pre-wrap">{result.content}</pre>
            </div>
            
            {result.videoPath && (
              <div className="p-4 bg-gray-50 rounded-lg">
                <h3 className="font-bold mb-2">Video:</h3>
                <p>Generated at: {result.videoPath}</p>
              </div>
            )}
          </div>
        )}
      </div>
    </div>
  );
}
```

## Running the Application

```bash
# Development mode
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Render Remotion video standalone
npx remotion render ContentVideo output.mp4
```

## Common Troubleshooting

### API Key Issues
```typescript
// Verify environment variables are loaded
if (!process.env.ANTHROPIC_API_KEY) {
  throw new Error('ANTHROPIC_API_KEY is not set');
}
```

### Rate Limiting
```typescript
// Implement retry logic with exponential backoff
async function withRetry<T>(
  fn: () => Promise<T>,
  maxRetries = 3
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error: any) {
      if (error.status === 429 && i < maxRetries - 1) {
        await new Promise(resolve => 
          setTimeout(resolve, Math.pow(2, i) * 1000)
        );
        continue;
      }
      throw error;
    }
  }
  throw new Error('Max retries exceeded');
}
```

### Video Rendering Memory Issues
```bash
# Increase Node.js memory limit
NODE_OPTIONS="--max-old-space-size=4096" npm run dev
```

### Remotion Bundle Errors
```typescript
// Ensure webpack override is correct
const bundleLocation = await bundle({
  entryPoint: path.resolve('./remotion/index.ts'),
  webpackOverride: (config) => {
    return {
      ...config,
      resolve: {
        ...config.resolve,
        alias: {
          ...config.resolve?.alias,
          '@': path.resolve('./src'),
        },
      },
    };
  },
});
```

This skill provides comprehensive guidance for integrating and using the marketing pipeline automation system with AI-powered content generation and video rendering capabilities.
