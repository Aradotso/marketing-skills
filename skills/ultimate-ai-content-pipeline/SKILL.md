---
name: ultimate-ai-content-pipeline
description: Automate content creation from research to video generation with AI-powered crawling, multi-format writing, and automatic video rendering
triggers:
  - how do I automate content creation with AI research
  - generate blog posts with automatic research crawling
  - create videos from content automatically with Remotion
  - set up AI content pipeline for marketing
  - automate TechCrunch news crawling for content
  - generate multilingual content with Claude and OpenAI
  - build automated content generation workflow
  - create social media videos from blog posts
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

A comprehensive TypeScript-based content automation system that combines AI-powered research, multi-format content generation, and automatic video rendering. This pipeline crawls fresh data from sources like TechCrunch, Twitter, and LinkedIn, generates content in multiple languages and formats using Claude/OpenAI, and renders videos using Remotion.

## What It Does

This system automates the entire content creation workflow:

1. **Auto-Research**: Crawls and analyzes real-time data from news sources and social media
2. **AI Content Generation**: Creates content in multiple formats (toplist, POV, case study, how-to) and languages (English/Vietnamese)
3. **Video Rendering**: Automatically generates infographics and short-form videos from written content
4. **Multi-Platform Export**: Outputs optimized content for Reels, TikTok, Shorts, and blog posts

## Installation

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
pnpm install
```

### Environment Setup

Create a `.env.local` file in the root directory:

```env
# AI APIs
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# Content Crawling
RAPIDAPI_KEY=your_rapidapi_key

# Remotion (Video Rendering)
REMOTION_LICENSE_KEY=your_remotion_key

# Database (if applicable)
DATABASE_URL=your_database_connection

# Next.js
NEXT_PUBLIC_API_URL=http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI service integrations
│   │   ├── crawler/     # Content crawling logic
│   │   └── video/       # Remotion video generation
│   ├── types/           # TypeScript type definitions
│   └── utils/           # Utility functions
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Core Features & Usage

### 1. Content Research & Crawling

The system automatically crawls fresh content from multiple sources:

```typescript
// lib/crawler/sources.ts
import { RapidAPIClient } from '@/lib/api/rapidapi';

interface NewsSource {
  name: string;
  url: string;
  selector: string;
}

export async function crawlTechNews(keyword: string) {
  const rapidAPI = new RapidAPIClient(process.env.RAPIDAPI_KEY!);
  
  const sources = [
    { name: 'TechCrunch', endpoint: 'techcrunch/articles' },
    { name: 'a16z', endpoint: 'a16z/posts' },
  ];
  
  const results = await Promise.all(
    sources.map(source => 
      rapidAPI.fetchArticles({
        source: source.endpoint,
        keyword,
        timeRange: '24h',
        limit: 10
      })
    )
  );
  
  return results.flat();
}

// Usage
const articles = await crawlTechNews('AI automation');
```

### 2. AI Content Generation

Generate multi-format content with Claude or OpenAI:

```typescript
// lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

interface ContentConfig {
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  targetAudience: string;
}

export class ContentGenerator {
  private claude: Anthropic;
  private openai: OpenAI;
  
  constructor() {
    this.claude = new Anthropic({
      apiKey: process.env.ANTHROPIC_API_KEY!
    });
    this.openai = new OpenAI({
      apiKey: process.env.OPENAI_API_KEY!
    });
  }
  
  async generateContent(
    topic: string,
    researchData: any[],
    config: ContentConfig
  ) {
    const prompt = this.buildPrompt(topic, researchData, config);
    
    // Use Claude for long-form content
    const response = await this.claude.messages.create({
      model: 'claude-3-5-sonnet-20241022',
      max_tokens: 4096,
      messages: [{
        role: 'user',
        content: prompt
      }]
    });
    
    return this.parseContent(response.content[0].text);
  }
  
  private buildPrompt(
    topic: string,
    research: any[],
    config: ContentConfig
  ): string {
    const researchContext = research
      .map(r => `- ${r.title}: ${r.summary}`)
      .join('\n');
    
    return `
You are an expert content writer. Create a ${config.format} article about "${topic}".

Research Data (last 24h):
${researchContext}

Requirements:
- Language: ${config.language === 'en' ? 'English' : 'Vietnamese'}
- Tone: ${config.tone}
- Target Audience: ${config.targetAudience}
- Include specific data points and statistics from research
- Make it engaging and actionable

${this.getFormatInstructions(config.format)}
    `.trim();
  }
  
  private getFormatInstructions(format: string): string {
    const formats = {
      'toplist': 'Create a numbered list with detailed explanations for each point.',
      'pov': 'Write from a unique perspective with strong opinions backed by data.',
      'case-study': 'Structure as: Problem → Solution → Results with real examples.',
      'how-to': 'Provide step-by-step instructions with actionable tips.'
    };
    return formats[format] || '';
  }
  
  private parseContent(rawContent: string) {
    // Extract structured content
    return {
      title: this.extractTitle(rawContent),
      introduction: this.extractSection(rawContent, 'introduction'),
      mainContent: this.extractSection(rawContent, 'body'),
      conclusion: this.extractSection(rawContent, 'conclusion'),
      keyPoints: this.extractKeyPoints(rawContent),
      metadata: {
        wordCount: rawContent.split(' ').length,
        readingTime: Math.ceil(rawContent.split(' ').length / 200)
      }
    };
  }
}

// Usage
const generator = new ContentGenerator();
const content = await generator.generateContent(
  'AI Content Automation Trends 2024',
  crawledArticles,
  {
    format: 'toplist',
    language: 'en',
    tone: 'expert',
    targetAudience: 'Marketing professionals'
  }
);
```

### 3. Bilingual Content Generation

Generate content in both English and Vietnamese simultaneously:

```typescript
// lib/ai/bilingual-generator.ts
export async function generateBilingualContent(
  topic: string,
  research: any[]
) {
  const generator = new ContentGenerator();
  
  const [enContent, viContent] = await Promise.all([
    generator.generateContent(topic, research, {
      format: 'toplist',
      language: 'en',
      tone: 'expert',
      targetAudience: 'Global marketers'
    }),
    generator.generateContent(topic, research, {
      format: 'toplist',
      language: 'vi',
      tone: 'friendly',
      targetAudience: 'Vietnamese content creators'
    })
  ]);
  
  return { enContent, viContent };
}
```

### 4. Video Generation with Remotion

Automatically render videos from content:

```typescript
// lib/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  template: 'infographic' | 'social-short' | 'explainer';
  platform: 'reels' | 'tiktok' | 'shorts' | 'youtube';
  duration: number;
}

export class VideoRenderer {
  async renderFromContent(
    content: any,
    config: VideoConfig
  ): Promise<string> {
    // Bundle Remotion composition
    const bundleLocation = await bundle({
      entryPoint: path.join(process.cwd(), 'remotion/index.ts'),
      webpackOverride: (config) => config
    });
    
    // Get composition
    const composition = await selectComposition({
      serveUrl: bundleLocation,
      id: this.getCompositionId(config.template),
      inputProps: this.prepareInputProps(content, config)
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
      inputProps: this.prepareInputProps(content, config)
    });
    
    return outputLocation;
  }
  
  private getCompositionId(template: string): string {
    const templates = {
      'infographic': 'InfographicVideo',
      'social-short': 'SocialShortVideo',
      'explainer': 'ExplainerVideo'
    };
    return templates[template] || 'DefaultVideo';
  }
  
  private prepareInputProps(content: any, config: VideoConfig) {
    return {
      title: content.title,
      keyPoints: content.keyPoints,
      aspectRatio: this.getAspectRatio(config.platform),
      duration: config.duration,
      theme: {
        primaryColor: '#3b82f6',
        backgroundColor: '#1f2937',
        textColor: '#ffffff'
      }
    };
  }
  
  private getAspectRatio(platform: string) {
    const ratios = {
      'reels': { width: 1080, height: 1920 },
      'tiktok': { width: 1080, height: 1920 },
      'shorts': { width: 1080, height: 1920 },
      'youtube': { width: 1920, height: 1080 }
    };
    return ratios[platform] || ratios.youtube;
  }
}
```

### 5. Remotion Video Composition

Example Remotion composition for social media shorts:

```typescript
// remotion/SocialShortVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig, interpolate } from 'remotion';
import React from 'react';

interface Props {
  title: string;
  keyPoints: string[];
  theme: {
    primaryColor: string;
    backgroundColor: string;
    textColor: string;
  };
}

export const SocialShortVideo: React.FC<Props> = ({ title, keyPoints, theme }) => {
  const frame = useCurrentFrame();
  const { fps, durationInFrames } = useVideoConfig();
  
  const titleOpacity = interpolate(frame, [0, 30], [0, 1], {
    extrapolateRight: 'clamp'
  });
  
  const titleScale = interpolate(frame, [0, 30], [0.8, 1], {
    extrapolateRight: 'clamp'
  });
  
  return (
    <AbsoluteFill style={{ backgroundColor: theme.backgroundColor }}>
      {/* Title Animation */}
      <div
        style={{
          position: 'absolute',
          top: '20%',
          width: '100%',
          textAlign: 'center',
          opacity: titleOpacity,
          transform: `scale(${titleScale})`
        }}
      >
        <h1
          style={{
            color: theme.textColor,
            fontSize: 64,
            fontWeight: 'bold',
            padding: '0 40px',
            margin: 0
          }}
        >
          {title}
        </h1>
      </div>
      
      {/* Key Points */}
      <div style={{ position: 'absolute', top: '40%', width: '100%', padding: '0 60px' }}>
        {keyPoints.map((point, index) => {
          const pointFrame = 60 + index * 90;
          const opacity = interpolate(
            frame,
            [pointFrame, pointFrame + 30],
            [0, 1],
            { extrapolateRight: 'clamp' }
          );
          
          return (
            <div
              key={index}
              style={{
                opacity,
                marginBottom: 40,
                backgroundColor: theme.primaryColor,
                padding: 30,
                borderRadius: 20
              }}
            >
              <p style={{ color: theme.textColor, fontSize: 36, margin: 0 }}>
                {index + 1}. {point}
              </p>
            </div>
          );
        })}
      </div>
    </AbsoluteFill>
  );
};
```

## Complete Workflow Example

Here's how to use the entire pipeline together:

```typescript
// app/api/generate-content/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { crawlTechNews } from '@/lib/crawler/sources';
import { ContentGenerator } from '@/lib/ai/content-generator';
import { VideoRenderer } from '@/lib/video/renderer';

export async function POST(req: NextRequest) {
  try {
    const { keyword, format, generateVideo } = await req.json();
    
    // Step 1: Research
    console.log('🔍 Crawling latest news...');
    const research = await crawlTechNews(keyword);
    
    // Step 2: Generate Content
    console.log('✍️ Generating content...');
    const generator = new ContentGenerator();
    const content = await generator.generateContent(
      keyword,
      research,
      {
        format,
        language: 'en',
        tone: 'expert',
        targetAudience: 'Marketing professionals'
      }
    );
    
    let videoUrl = null;
    
    // Step 3: Generate Video (optional)
    if (generateVideo) {
      console.log('🎬 Rendering video...');
      const renderer = new VideoRenderer();
      videoUrl = await renderer.renderFromContent(content, {
        template: 'social-short',
        platform: 'reels',
        duration: 30
      });
    }
    
    return NextResponse.json({
      success: true,
      content,
      videoUrl,
      researchSources: research.length
    });
    
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { error: 'Content generation failed' },
      { status: 500 }
    );
  }
}
```

### Frontend Component

```typescript
// components/ContentPipeline.tsx
'use client';

import { useState } from 'react';

export default function ContentPipeline() {
  const [keyword, setKeyword] = useState('');
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState(null);
  
  const generateContent = async () => {
    setLoading(true);
    try {
      const response = await fetch('/api/generate-content', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          keyword,
          format: 'toplist',
          generateVideo: true
        })
      });
      
      const data = await response.json();
      setResult(data);
    } catch (error) {
      console.error('Error:', error);
    } finally {
      setLoading(false);
    }
  };
  
  return (
    <div className="max-w-4xl mx-auto p-6">
      <h1 className="text-3xl font-bold mb-6">AI Content Pipeline</h1>
      
      <div className="space-y-4">
        <input
          type="text"
          value={keyword}
          onChange={(e) => setKeyword(e.target.value)}
          placeholder="Enter keyword (e.g., AI automation)"
          className="w-full p-3 border rounded"
        />
        
        <button
          onClick={generateContent}
          disabled={loading || !keyword}
          className="bg-blue-600 text-white px-6 py-3 rounded disabled:opacity-50"
        >
          {loading ? 'Generating...' : 'Generate Content & Video'}
        </button>
        
        {result && (
          <div className="mt-6 space-y-4">
            <div className="bg-gray-100 p-4 rounded">
              <h2 className="font-bold text-xl">{result.content.title}</h2>
              <p className="text-sm text-gray-600">
                {result.researchSources} sources analyzed
              </p>
            </div>
            
            {result.videoUrl && (
              <video controls className="w-full rounded">
                <source src={result.videoUrl} type="video/mp4" />
              </video>
            )}
          </div>
        )}
      </div>
    </div>
  );
}
```

## Running the Project

```bash
# Development
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Render videos (if using separate Remotion CLI)
npm run remotion:render
```

## Configuration Tips

### API Rate Limits

Handle rate limits gracefully:

```typescript
// lib/utils/rate-limiter.ts
export class RateLimiter {
  private queue: Promise<any>[] = [];
  
  async throttle<T>(fn: () => Promise<T>, delayMs = 1000): Promise<T> {
    if (this.queue.length > 0) {
      await Promise.all(this.queue);
      await new Promise(resolve => setTimeout(resolve, delayMs));
    }
    
    const promise = fn();
    this.queue.push(promise);
    
    return promise;
  }
}
```

### Content Caching

Cache research results to reduce API calls:

```typescript
// lib/cache/research-cache.ts
import { Redis } from 'ioredis';

const redis = new Redis(process.env.REDIS_URL);

export async function getCachedResearch(keyword: string) {
  const cached = await redis.get(`research:${keyword}`);
  return cached ? JSON.parse(cached) : null;
}

export async function cacheResearch(keyword: string, data: any) {
  await redis.setex(`research:${keyword}`, 3600, JSON.stringify(data));
}
```

## Troubleshooting

### Video Rendering Fails

Check Remotion configuration and ensure FFmpeg is installed:

```bash
# Install FFmpeg
brew install ffmpeg  # macOS
sudo apt install ffmpeg  # Ubuntu
```

### AI API Timeouts

Increase timeout limits for long content generation:

```typescript
const response = await this.claude.messages.create({
  model: 'claude-3-5-sonnet-20241022',
  max_tokens: 4096,
  timeout: 60000,  // 60 seconds
  messages: [...]
});
```

### Memory Issues During Video Rendering

Optimize video settings:

```typescript
await renderMedia({
  composition,
  serveUrl: bundleLocation,
  codec: 'h264',
  outputLocation,
  chromiumOptions: {
    headless: true,
    args: ['--no-sandbox', '--disable-setuid-sandbox']
  },
  concurrency: 1  // Reduce concurrency for lower memory usage
});
```

This pipeline provides a complete automated content creation system from research to video generation, ideal for marketing teams and content creators looking to scale their output efficiently.
