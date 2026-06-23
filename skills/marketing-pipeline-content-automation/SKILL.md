---
name: marketing-pipeline-content-automation
description: Automated AI content pipeline from research to video generation using Claude, OpenAI, and Remotion for multi-format content creation
triggers:
  - automate content creation from research to video
  - generate multi-format marketing content with AI
  - create content pipeline with Claude and OpenAI
  - automate video generation from written content
  - build AI-powered content automation system
  - set up automated content research and publishing
  - generate videos and articles from keywords automatically
  - create end-to-end marketing content pipeline
---

# Marketing Pipeline Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

Marketing Pipeline Content Automation is a comprehensive TypeScript-based system that automates the entire content creation workflow from research to video generation. It automatically crawls news sources (TechCrunch, a16z, Twitter, LinkedIn), generates AI-powered articles in multiple formats using Claude 3/OpenAI, and renders videos using Remotion - all from a single keyword input.

**Key capabilities:**
- Auto-scan research from real-time news sources
- Multi-format content generation (toplist, POV, case study, how-to)
- Bilingual content (English & Vietnamese)
- Automated video rendering with Remotion
- Next.js web interface for content management

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
# AI Services
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Remotion (Video Generation)
REMOTION_LICENSE_KEY=your_remotion_license_key

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Next.js
NEXT_PUBLIC_API_URL=http://localhost:3000
```

## Development Setup

```bash
# Run development server
npm run dev

# Build for production
npm run build

# Start production server
npm run start

# Run Remotion studio (for video editing)
npm run remotion:studio
```

## Core Architecture

### 1. Research Pipeline

The research module automatically crawls and aggregates content from multiple sources:

```typescript
// lib/research/scanner.ts
import { RapidAPI } from '@/lib/api/rapid';

interface ResearchSource {
  source: string;
  url: string;
  title: string;
  summary: string;
  publishedAt: Date;
}

export async function scanNewsSource(
  keyword: string,
  timeframe: '24h' | '7d' | '30d' = '24h'
): Promise<ResearchSource[]> {
  const sources = [
    'techcrunch',
    'a16z',
    'twitter',
    'linkedin'
  ];
  
  const results: ResearchSource[] = [];
  
  for (const source of sources) {
    try {
      const data = await RapidAPI.search({
        source,
        query: keyword,
        timeframe
      });
      
      results.push(...data.articles.map(article => ({
        source,
        url: article.url,
        title: article.title,
        summary: article.description,
        publishedAt: new Date(article.publishedAt)
      })));
    } catch (error) {
      console.error(`Failed to scan ${source}:`, error);
    }
  }
  
  return results;
}

export async function aggregateInsights(
  sources: ResearchSource[]
): Promise<string> {
  // Combine and deduplicate insights
  const insights = sources
    .map(s => `[${s.source}] ${s.title}: ${s.summary}`)
    .join('\n\n');
  
  return insights;
}
```

### 2. AI Content Generation

Generate content in multiple formats using Claude or OpenAI:

```typescript
// lib/ai/content-generator.ts
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

interface ContentRequest {
  keyword: string;
  research: string;
  format: ContentFormat;
  language: Language;
  tone: Tone;
}

export async function generateContentWithClaude(
  request: ContentRequest
): Promise<string> {
  const formatPrompts = {
    'toplist': 'Create a comprehensive top list article',
    'pov': 'Write a point-of-view opinion piece',
    'case-study': 'Develop a detailed case study analysis',
    'how-to': 'Write a step-by-step how-to guide'
  };
  
  const toneInstructions = {
    'expert': 'professional and authoritative tone',
    'friendly': 'conversational and approachable tone',
    'humorous': 'engaging and witty tone'
  };
  
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [{
      role: 'user',
      content: `${formatPrompts[request.format]} about "${request.keyword}" in ${request.language === 'vi' ? 'Vietnamese' : 'English'} with a ${toneInstructions[request.tone]}.

Research data:
${request.research}

Requirements:
- SEO-optimized with relevant keywords
- Data-backed insights from the research
- Engaging introduction and strong conclusion
- Include actionable takeaways
- ${request.format === 'toplist' ? 'Number items clearly' : ''}
- ${request.language === 'vi' ? 'Use natural Vietnamese expressions' : 'Use clear English'}`
    }]
  });
  
  return message.content[0].type === 'text' ? message.content[0].text : '';
}

export async function generateContentWithOpenAI(
  request: ContentRequest
): Promise<string> {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [{
      role: 'system',
      content: `You are an expert content writer specializing in ${request.format} articles.`
    }, {
      role: 'user',
      content: `Write a ${request.format} article about "${request.keyword}" based on this research:\n\n${request.research}`
    }],
    temperature: 0.7,
    max_tokens: 3000
  });
  
  return completion.choices[0].message.content || '';
}
```

### 3. Bilingual Content Generation

Generate content in both English and Vietnamese simultaneously:

```typescript
// lib/ai/bilingual-generator.ts
import { generateContentWithClaude, ContentRequest } from './content-generator';

interface BilingualContent {
  en: string;
  vi: string;
}

export async function generateBilingualContent(
  request: Omit<ContentRequest, 'language'>
): Promise<BilingualContent> {
  const [enContent, viContent] = await Promise.all([
    generateContentWithClaude({ ...request, language: 'en' }),
    generateContentWithClaude({ ...request, language: 'vi' })
  ]);
  
  return {
    en: enContent,
    vi: viContent
  };
}
```

### 4. Video Generation with Remotion

Transform written content into videos:

```typescript
// remotion/compositions/ContentVideo.tsx
import { AbsoluteFill, Sequence, useCurrentFrame, useVideoConfig } from 'remotion';
import React from 'react';

interface ContentVideoProps {
  title: string;
  points: string[];
  backgroundColor: string;
  textColor: string;
}

export const ContentVideo: React.FC<ContentVideoProps> = ({
  title,
  points,
  backgroundColor,
  textColor
}) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  
  const titleOpacity = Math.min(1, frame / (fps * 0.5));
  
  return (
    <AbsoluteFill style={{ backgroundColor }}>
      {/* Title Sequence */}
      <Sequence from={0} durationInFrames={fps * 2}>
        <AbsoluteFill
          style={{
            justifyContent: 'center',
            alignItems: 'center',
            padding: '40px'
          }}
        >
          <h1
            style={{
              fontSize: '60px',
              fontWeight: 'bold',
              color: textColor,
              opacity: titleOpacity,
              textAlign: 'center'
            }}
          >
            {title}
          </h1>
        </AbsoluteFill>
      </Sequence>
      
      {/* Content Points */}
      {points.map((point, index) => (
        <Sequence
          key={index}
          from={fps * (2 + index * 3)}
          durationInFrames={fps * 3}
        >
          <AbsoluteFill
            style={{
              justifyContent: 'center',
              alignItems: 'center',
              padding: '60px'
            }}
          >
            <div
              style={{
                fontSize: '36px',
                color: textColor,
                lineHeight: 1.5,
                textAlign: 'center'
              }}
            >
              {point}
            </div>
          </AbsoluteFill>
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};
```

```typescript
// lib/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoRenderOptions {
  title: string;
  content: string;
  outputPath: string;
}

export async function renderContentVideo(
  options: VideoRenderOptions
): Promise<string> {
  // Extract key points from content
  const points = extractKeyPoints(options.content);
  
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: (config) => config
  });
  
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title: options.title,
      points,
      backgroundColor: '#1a1a2e',
      textColor: '#ffffff'
    }
  });
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: options.outputPath,
    inputProps: composition.defaultProps
  });
  
  return options.outputPath;
}

function extractKeyPoints(content: string): string[] {
  // Simple extraction - can be enhanced with AI
  const lines = content.split('\n').filter(line => 
    line.trim().length > 20 && 
    (line.match(/^[\d\-•]/) || line.includes(':'))
  );
  
  return lines.slice(0, 5).map(line => 
    line.replace(/^[\d\-•]+\s*/, '').trim()
  );
}
```

### 5. Complete Pipeline API

```typescript
// app/api/content-pipeline/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { scanNewsSource, aggregateInsights } from '@/lib/research/scanner';
import { generateBilingualContent } from '@/lib/ai/bilingual-generator';
import { renderContentVideo } from '@/lib/video/renderer';
import path from 'path';

export async function POST(request: NextRequest) {
  try {
    const { keyword, format, tone } = await request.json();
    
    // Step 1: Research
    console.log('🔍 Starting research phase...');
    const sources = await scanNewsSource(keyword, '24h');
    const research = await aggregateInsights(sources);
    
    // Step 2: Generate Content
    console.log('✍️ Generating bilingual content...');
    const content = await generateBilingualContent({
      keyword,
      research,
      format,
      tone
    });
    
    // Step 3: Render Video
    console.log('🎬 Rendering video...');
    const videoPath = path.join(
      process.cwd(),
      'public',
      'videos',
      `${keyword.replace(/\s+/g, '-')}-${Date.now()}.mp4`
    );
    
    const videoUrl = await renderContentVideo({
      title: keyword,
      content: content.en,
      outputPath: videoPath
    });
    
    return NextResponse.json({
      success: true,
      data: {
        research: sources.length,
        content: {
          english: content.en,
          vietnamese: content.vi
        },
        video: videoUrl.replace(process.cwd() + '/public', '')
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

## Usage Patterns

### Basic Content Generation

```typescript
// pages/create-content.tsx
import { useState } from 'react';

export default function CreateContent() {
  const [keyword, setKeyword] = useState('');
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState(null);
  
  const handleGenerate = async () => {
    setLoading(true);
    
    try {
      const response = await fetch('/api/content-pipeline', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          keyword,
          format: 'toplist',
          tone: 'expert'
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
    <div>
      <input 
        value={keyword}
        onChange={(e) => setKeyword(e.target.value)}
        placeholder="Enter keyword..."
      />
      <button onClick={handleGenerate} disabled={loading}>
        {loading ? 'Generating...' : 'Generate Content'}
      </button>
      
      {result && (
        <div>
          <h2>English Version</h2>
          <p>{result.data.content.english}</p>
          
          <h2>Vietnamese Version</h2>
          <p>{result.data.content.vietnamese}</p>
          
          <video src={result.data.video} controls />
        </div>
      )}
    </div>
  );
}
```

### Scheduled Content Generation

```typescript
// lib/scheduler/content-scheduler.ts
import cron from 'node-cron';
import { scanNewsSource, aggregateInsights } from '@/lib/research/scanner';
import { generateBilingualContent } from '@/lib/ai/bilingual-generator';

export function scheduleContentGeneration(keywords: string[]) {
  // Run daily at 9 AM
  cron.schedule('0 9 * * *', async () => {
    console.log('🤖 Starting scheduled content generation...');
    
    for (const keyword of keywords) {
      try {
        const sources = await scanNewsSource(keyword, '24h');
        
        if (sources.length > 5) {
          const research = await aggregateInsights(sources);
          const content = await generateBilingualContent({
            keyword,
            research,
            format: 'toplist',
            tone: 'expert'
          });
          
          // Save to database or publish
          console.log(`✅ Generated content for: ${keyword}`);
        }
      } catch (error) {
        console.error(`❌ Failed for ${keyword}:`, error);
      }
    }
  });
}
```

## Configuration

### Custom Remotion Compositions

```typescript
// remotion.config.ts
import { Config } from '@remotion/cli/config';

Config.setVideoImageFormat('jpeg');
Config.setOverwriteOutput(true);
Config.setConcurrency(4);
Config.setCodec('h264');
Config.setPixelFormat('yuv420p');

export default {
  compositions: [
    {
      id: 'ContentVideo',
      component: 'ContentVideo',
      durationInFrames: 300,
      fps: 30,
      width: 1080,
      height: 1920 // Vertical for Reels/TikTok
    }
  ]
};
```

### AI Provider Selection

```typescript
// lib/config/ai-config.ts
export const AI_CONFIG = {
  preferredProvider: process.env.AI_PROVIDER || 'claude', // 'claude' or 'openai'
  models: {
    claude: 'claude-3-5-sonnet-20241022',
    openai: 'gpt-4-turbo-preview'
  },
  maxTokens: {
    claude: 4096,
    openai: 3000
  }
};
```

## Troubleshooting

### API Rate Limits

```typescript
// lib/utils/rate-limiter.ts
class RateLimiter {
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
      await new Promise(resolve => setTimeout(resolve, 1000)); // 1s delay
    }
    
    this.processing = false;
    this.process();
  }
}

export const aiLimiter = new RateLimiter();
```

### Video Rendering Memory Issues

```typescript
// Increase Node.js memory limit
// package.json
{
  "scripts": {
    "render:video": "NODE_OPTIONS='--max-old-space-size=4096' tsx lib/video/renderer.ts"
  }
}
```

### Research Source Failures

```typescript
// Implement fallback mechanism
async function scanWithFallback(keyword: string) {
  const primarySources = ['techcrunch', 'a16z'];
  const fallbackSources = ['twitter', 'linkedin'];
  
  try {
    return await scanNewsSource(keyword, '24h');
  } catch (error) {
    console.warn('Primary sources failed, using fallback...');
    return await scanAlternativeSources(keyword, fallbackSources);
  }
}
```

## Best Practices

1. **Cache research results** to avoid redundant API calls
2. **Use queues** for video rendering to prevent memory overload
3. **Implement retry logic** for AI API calls
4. **Store generated content** in a database for reuse
5. **Monitor API usage** to stay within rate limits
6. **Validate content quality** before publishing
7. **A/B test different tones** and formats for your audience
