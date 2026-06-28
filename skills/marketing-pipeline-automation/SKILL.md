---
name: marketing-pipeline-automation
description: Automated AI content pipeline from research to video generation using Claude, OpenAI, and Remotion
triggers:
  - how do I automate content creation with AI
  - help me set up the marketing pipeline project
  - generate video content from text automatically
  - crawl news and create content with Claude
  - automate research to video workflow
  - build AI content generation pipeline
  - create automated marketing content system
  - set up remotion video rendering for content
---

# Marketing Pipeline Automation Skill

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to help developers use the **Ultimate AI Content Pipeline** - an automated system that takes a keyword and produces complete content pieces including research, written articles, and rendered videos. The pipeline integrates Claude 3, OpenAI, web scraping, and Remotion for video generation.

## What This Project Does

The Marketing Pipeline automates the entire content creation workflow:

1. **Auto-Research**: Crawls recent news from TechCrunch, a16z, Twitter/X, LinkedIn (last 24h)
2. **AI Content Generation**: Creates articles in multiple formats (toplist, POV, case study, how-to) using Claude/OpenAI
3. **Multi-language**: Generates content in English and Vietnamese simultaneously
4. **Video Rendering**: Automatically creates infographics and short videos using Remotion
5. **Platform Optimization**: Exports videos for Reels, TikTok, Shorts

## Installation

### Prerequisites

```bash
# Node.js 18+ required
node --version

# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
yarn install
```

### Environment Configuration

Create a `.env.local` file in the project root:

```bash
# AI APIs
OPENAI_API_KEY=
ANTHROPIC_API_KEY=

# Web Scraping
RAPIDAPI_KEY=

# Database (optional)
DATABASE_URL=

# Remotion (for video rendering)
REMOTION_LICENSE_KEY=

# Application
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

### Development Server

```typescript
// Start the Next.js development server
npm run dev
// or
yarn dev

// Access at http://localhost:3000
```

## Core Architecture

### Project Structure

```
marketing-pineline-share/
├── app/                    # Next.js app directory
├── components/             # React components
├── lib/                    # Core utilities
│   ├── ai/                # AI integrations
│   ├── scraper/           # Web scraping modules
│   └── video/             # Remotion video generation
├── remotion/              # Video templates
└── public/                # Static assets
```

## Key Components and Usage

### 1. Content Research Module

```typescript
// lib/scraper/news-crawler.ts
import { fetchNewsFromSources } from '@/lib/scraper/news-crawler';

interface NewsSource {
  name: string;
  url: string;
  selector: string;
}

async function gatherResearch(keyword: string) {
  const sources: NewsSource[] = [
    { name: 'TechCrunch', url: 'https://techcrunch.com', selector: '.post-block' },
    { name: 'a16z', url: 'https://a16z.com/posts', selector: '.article' }
  ];
  
  const results = await fetchNewsFromSources(keyword, sources, {
    timeRange: '24h',
    maxResults: 20
  });
  
  return results;
}

// Usage
const research = await gatherResearch('AI automation');
console.log(research.articles); // Array of crawled articles
```

### 2. AI Content Generation

```typescript
// lib/ai/content-generator.ts
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
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  researchData: any[];
}

async function generateContentWithClaude(request: ContentRequest) {
  const prompt = buildPrompt(request);
  
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

async function generateContentWithOpenAI(request: ContentRequest) {
  const prompt = buildPrompt(request);
  
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      { role: 'system', content: 'You are an expert content creator.' },
      { role: 'user', content: prompt }
    ],
    max_tokens: 4096
  });
  
  return completion.choices[0].message.content;
}

function buildPrompt(request: ContentRequest): string {
  const { keyword, format, language, tone, researchData } = request;
  
  return `
Create a ${format} article about "${keyword}" in ${language}.
Tone: ${tone}
Based on this research data: ${JSON.stringify(researchData)}

Requirements:
- Include data-backed insights
- Use recent statistics (last 24h)
- Format for ${language === 'vi' ? 'Vietnamese' : 'English'} audience
- Optimize for SEO
`;
}
```

### 3. Video Generation with Remotion

```typescript
// lib/video/render-video.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  title: string;
  content: string[];
  style: 'infographic' | 'text-animation' | 'slideshow';
  platform: 'reels' | 'tiktok' | 'shorts';
}

async function renderContentVideo(config: VideoConfig) {
  const { title, content, style, platform } = config;
  
  // Platform-specific dimensions
  const dimensions = {
    reels: { width: 1080, height: 1920 },
    tiktok: { width: 1080, height: 1920 },
    shorts: { width: 1080, height: 1920 }
  };
  
  const { width, height } = dimensions[platform];
  
  // Bundle Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.join(process.cwd(), 'remotion/index.ts'),
    webpackOverride: (config) => config,
  });
  
  // Select composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: style,
    inputProps: {
      title,
      content,
      width,
      height
    }
  });
  
  // Render video
  const outputPath = path.join(process.cwd(), 'public/videos', `${Date.now()}.mp4`);
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: {
      title,
      content
    }
  });
  
  return outputPath;
}
```

### 4. Remotion Video Template

```typescript
// remotion/compositions/Infographic.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig, interpolate } from 'remotion';

interface InfographicProps {
  title: string;
  content: string[];
}

export const Infographic: React.FC<InfographicProps> = ({ title, content }) => {
  const frame = useCurrentFrame();
  const { fps, durationInFrames } = useVideoConfig();
  
  const titleOpacity = interpolate(
    frame,
    [0, 30],
    [0, 1],
    { extrapolateRight: 'clamp' }
  );
  
  const titleScale = interpolate(
    frame,
    [0, 30],
    [0.8, 1],
    { extrapolateRight: 'clamp' }
  );
  
  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a1a' }}>
      <div style={{
        display: 'flex',
        flexDirection: 'column',
        justifyContent: 'center',
        alignItems: 'center',
        padding: '40px',
        height: '100%'
      }}>
        <h1 style={{
          color: 'white',
          fontSize: '72px',
          fontWeight: 'bold',
          textAlign: 'center',
          opacity: titleOpacity,
          transform: `scale(${titleScale})`
        }}>
          {title}
        </h1>
        
        <div style={{ marginTop: '60px' }}>
          {content.map((item, index) => {
            const itemFrame = 40 + (index * 20);
            const itemOpacity = interpolate(
              frame,
              [itemFrame, itemFrame + 15],
              [0, 1],
              { extrapolateRight: 'clamp' }
            );
            
            return (
              <p key={index} style={{
                color: '#ffffff',
                fontSize: '36px',
                marginBottom: '30px',
                opacity: itemOpacity
              }}>
                {item}
              </p>
            );
          })}
        </div>
      </div>
    </AbsoluteFill>
  );
};
```

### 5. Complete Pipeline Example

```typescript
// app/api/generate-content/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { gatherResearch } from '@/lib/scraper/news-crawler';
import { generateContentWithClaude } from '@/lib/ai/content-generator';
import { renderContentVideo } from '@/lib/video/render-video';

export async function POST(request: NextRequest) {
  try {
    const { keyword, format, language, platform } = await request.json();
    
    // Step 1: Research
    console.log('🔍 Gathering research...');
    const research = await gatherResearch(keyword);
    
    // Step 2: Generate content
    console.log('✍️ Generating content...');
    const content = await generateContentWithClaude({
      keyword,
      format,
      language,
      tone: 'expert',
      researchData: research.articles
    });
    
    // Parse content into video-friendly format
    const contentLines = content.split('\n').filter(line => line.trim());
    
    // Step 3: Render video
    console.log('🎬 Rendering video...');
    const videoPath = await renderContentVideo({
      title: keyword,
      content: contentLines.slice(0, 5), // First 5 key points
      style: 'infographic',
      platform
    });
    
    return NextResponse.json({
      success: true,
      data: {
        content,
        videoUrl: videoPath.replace(process.cwd() + '/public', '')
      }
    });
    
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

### 6. Frontend Component

```typescript
// components/ContentPipeline.tsx
'use client';

import { useState } from 'react';

export function ContentPipeline() {
  const [keyword, setKeyword] = useState('');
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState<any>(null);
  
  const handleGenerate = async () => {
    setLoading(true);
    
    try {
      const response = await fetch('/api/generate-content', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          keyword,
          format: 'toplist',
          language: 'vi',
          platform: 'reels'
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
          placeholder="Enter keyword (e.g., 'AI automation')"
          className="w-full px-4 py-2 border rounded"
        />
        
        <button
          onClick={handleGenerate}
          disabled={loading || !keyword}
          className="px-6 py-2 bg-blue-600 text-white rounded disabled:opacity-50"
        >
          {loading ? 'Generating...' : 'Generate Content'}
        </button>
        
        {result && (
          <div className="mt-6 space-y-4">
            <div className="p-4 bg-gray-50 rounded">
              <h3 className="font-bold mb-2">Generated Content:</h3>
              <p className="whitespace-pre-wrap">{result.data.content}</p>
            </div>
            
            {result.data.videoUrl && (
              <div>
                <h3 className="font-bold mb-2">Generated Video:</h3>
                <video src={result.data.videoUrl} controls className="w-full" />
              </div>
            )}
          </div>
        )}
      </div>
    </div>
  );
}
```

## Configuration Options

### AI Model Selection

```typescript
// config/ai-config.ts
export const AI_CONFIG = {
  primaryModel: 'claude', // or 'openai'
  models: {
    claude: {
      model: 'claude-3-5-sonnet-20241022',
      maxTokens: 4096
    },
    openai: {
      model: 'gpt-4-turbo-preview',
      maxTokens: 4096
    }
  }
};
```

### Scraper Configuration

```typescript
// config/scraper-config.ts
export const SCRAPER_CONFIG = {
  sources: [
    { name: 'TechCrunch', url: 'https://techcrunch.com', enabled: true },
    { name: 'a16z', url: 'https://a16z.com/posts', enabled: true },
    { name: 'Twitter', url: 'https://twitter.com', enabled: false } // Requires auth
  ],
  timeRange: '24h',
  maxArticles: 20,
  timeout: 30000
};
```

### Video Configuration

```typescript
// config/video-config.ts
export const VIDEO_CONFIG = {
  defaultStyle: 'infographic',
  fps: 30,
  platforms: {
    reels: { width: 1080, height: 1920, duration: 30 },
    tiktok: { width: 1080, height: 1920, duration: 60 },
    shorts: { width: 1080, height: 1920, duration: 60 }
  }
};
```

## Common Patterns

### Batch Content Generation

```typescript
async function generateBatchContent(keywords: string[]) {
  const results = await Promise.all(
    keywords.map(async (keyword) => {
      const research = await gatherResearch(keyword);
      const content = await generateContentWithClaude({
        keyword,
        format: 'toplist',
        language: 'vi',
        tone: 'expert',
        researchData: research.articles
      });
      
      return { keyword, content };
    })
  );
  
  return results;
}
```

### Multi-language Generation

```typescript
async function generateMultiLanguage(keyword: string) {
  const research = await gatherResearch(keyword);
  
  const [english, vietnamese] = await Promise.all([
    generateContentWithClaude({
      keyword,
      format: 'toplist',
      language: 'en',
      tone: 'expert',
      researchData: research.articles
    }),
    generateContentWithClaude({
      keyword,
      format: 'toplist',
      language: 'vi',
      tone: 'expert',
      researchData: research.articles
    })
  ]);
  
  return { english, vietnamese };
}
```

## Troubleshooting

### API Rate Limits

```typescript
// lib/utils/rate-limiter.ts
class RateLimiter {
  private queue: Promise<any> = Promise.resolve();
  
  async throttle<T>(fn: () => Promise<T>, delay: number = 1000): Promise<T> {
    this.queue = this.queue.then(() => 
      new Promise(resolve => setTimeout(resolve, delay))
    );
    
    await this.queue;
    return fn();
  }
}

const limiter = new RateLimiter();

// Usage
const content = await limiter.throttle(() => 
  generateContentWithClaude(request)
);
```

### Video Rendering Memory Issues

```bash
# Increase Node.js memory limit
NODE_OPTIONS="--max-old-space-size=4096" npm run dev
```

### Scraper Blocked

```typescript
// Add user agent rotation
const USER_AGENTS = [
  'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36',
  'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36'
];

function getRandomUserAgent() {
  return USER_AGENTS[Math.floor(Math.random() * USER_AGENTS.length)];
}
```

### Environment Variables Not Loading

```typescript
// Check .env.local exists
if (!process.env.OPENAI_API_KEY) {
  throw new Error('OPENAI_API_KEY not found in environment variables');
}
```

## CLI Commands

```bash
# Development
npm run dev          # Start dev server
npm run build        # Build for production
npm run start        # Start production server

# Remotion
npm run remotion     # Open Remotion studio
npm run render       # Render video compositions

# Testing
npm run test         # Run tests
npm run lint         # Run linter
```
