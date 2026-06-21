---
name: marketing-pipeline-share-automation
description: AI-powered content automation pipeline for research, scriptwriting, and video generation using Claude, OpenAI, and Remotion
triggers:
  - automate content creation with AI pipeline
  - generate videos from blog posts automatically
  - research and write marketing content with AI
  - create multi-language content with Claude
  - set up automated content workflow
  - build AI content generation system
  - use Remotion for automated video rendering
  - scrape news and generate articles with AI
---

# Marketing Pipeline Share - Ultimate AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

Marketing Pipeline Share is a comprehensive TypeScript-based content automation system that transforms keywords into complete content packages. It automatically:

- Scrapes and analyzes real-time data from news sources (TechCrunch, Twitter, LinkedIn)
- Generates articles in multiple formats (listicles, POV, case studies, how-tos)
- Creates content in both English and Vietnamese
- Renders videos and infographics using Remotion
- Optimizes content for multiple platforms (Reels, TikTok, Shorts)

The system integrates Claude 3, OpenAI, and RapidAPI to create a fully automated content factory.

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

### Environment Variables

Create a `.env.local` file in the root directory:

```bash
# AI Services
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# Data Sources
RAPIDAPI_KEY=your_rapidapi_key

# Remotion (for video rendering)
REMOTION_AWS_ACCESS_KEY_ID=your_aws_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret

# Next.js
NEXT_PUBLIC_API_URL=http://localhost:3000
```

### Project Structure

```
├── src/
│   ├── app/              # Next.js app router
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI service integrations
│   │   ├── scrapers/    # Web scraping modules
│   │   └── video/       # Remotion video generation
│   └── types/           # TypeScript definitions
├── public/              # Static assets
└── remotion/            # Remotion video templates
```

## Core Features & Usage

### 1. Content Research & Scraping

The system automatically scrapes fresh data from multiple sources:

```typescript
// lib/scrapers/news-scraper.ts
import axios from 'axios';

interface NewsSource {
  url: string;
  selector: string;
  apiEndpoint?: string;
}

export async function scrapeNews(keyword: string, hours: number = 24) {
  const sources: NewsSource[] = [
    {
      url: 'https://techcrunch.com',
      apiEndpoint: '/api/search'
    },
    {
      url: 'https://a16z.com/blog',
      apiEndpoint: '/api/posts'
    }
  ];

  const results = await Promise.all(
    sources.map(source => fetchFromSource(source, keyword, hours))
  );

  return results.flat();
}

async function fetchFromSource(
  source: NewsSource,
  keyword: string,
  hours: number
) {
  const config = {
    headers: {
      'X-RapidAPI-Key': process.env.RAPIDAPI_KEY,
      'X-RapidAPI-Host': 'news-aggregator.p.rapidapi.com'
    },
    params: {
      q: keyword,
      time_range: `${hours}h`
    }
  };

  const response = await axios.get(source.apiEndpoint!, config);
  return response.data.articles;
}
```

### 2. AI Content Generation

Generate content in multiple formats using Claude or OpenAI:

```typescript
// lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

type ContentFormat = 'toplist' | 'pov' | 'case-study' | 'how-to';
type Language = 'en' | 'vi';
type Tone = 'expert' | 'friendly' | 'humorous';

interface ContentOptions {
  keyword: string;
  format: ContentFormat;
  language: Language;
  tone: Tone;
  researchData: any[];
}

export async function generateContent(
  options: ContentOptions,
  provider: 'claude' | 'openai' = 'claude'
) {
  const prompt = buildPrompt(options);

  if (provider === 'claude') {
    return generateWithClaude(prompt, options);
  } else {
    return generateWithOpenAI(prompt, options);
  }
}

async function generateWithClaude(prompt: string, options: ContentOptions) {
  const anthropic = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY
  });

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [
      {
        role: 'user',
        content: prompt
      }
    ]
  });

  return parseContentResponse(message.content[0].text, options);
}

async function generateWithOpenAI(prompt: string, options: ContentOptions) {
  const openai = new OpenAI({
    apiKey: process.env.OPENAI_API_KEY
  });

  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: 'You are an expert content creator and marketer.'
      },
      {
        role: 'user',
        content: prompt
      }
    ],
    temperature: 0.7
  });

  return parseContentResponse(completion.choices[0].message.content, options);
}

function buildPrompt(options: ContentOptions): string {
  const { keyword, format, language, tone, researchData } = options;

  const formatInstructions = {
    'toplist': 'Create a numbered list article with detailed explanations',
    'pov': 'Write a thought-leadership piece with a unique perspective',
    'case-study': 'Analyze real examples with data and outcomes',
    'how-to': 'Provide step-by-step actionable instructions'
  };

  const toneGuide = {
    'expert': 'authoritative, data-driven, professional',
    'friendly': 'conversational, approachable, warm',
    'humorous': 'witty, entertaining, light-hearted'
  };

  return `
Write a ${formatInstructions[format]} about "${keyword}" in ${language === 'en' ? 'English' : 'Vietnamese'}.

Tone: ${toneGuide[tone]}

Use the following recent research data:
${JSON.stringify(researchData, null, 2)}

Requirements:
- Include specific data points and statistics
- Reference recent trends (last 24-48 hours)
- Add actionable insights
- Optimize for ${language === 'en' ? 'English-speaking' : 'Vietnamese'} audience
- Include clear headings and structure
`;
}

function parseContentResponse(text: string, options: ContentOptions) {
  return {
    title: extractTitle(text),
    content: text,
    language: options.language,
    format: options.format,
    metadata: {
      wordCount: text.split(' ').length,
      generatedAt: new Date().toISOString()
    }
  };
}

function extractTitle(text: string): string {
  const titleMatch = text.match(/^#\s+(.+)$/m);
  return titleMatch ? titleMatch[1] : 'Untitled';
}
```

### 3. Bilingual Content Creation

Generate content in both English and Vietnamese simultaneously:

```typescript
// lib/ai/bilingual-generator.ts
export async function generateBilingualContent(
  keyword: string,
  format: ContentFormat,
  researchData: any[]
) {
  const [english, vietnamese] = await Promise.all([
    generateContent({
      keyword,
      format,
      language: 'en',
      tone: 'expert',
      researchData
    }),
    generateContent({
      keyword,
      format,
      language: 'vi',
      tone: 'friendly',
      researchData
    })
  ]);

  return {
    en: english,
    vi: vietnamese,
    metadata: {
      keyword,
      format,
      generatedAt: new Date().toISOString()
    }
  };
}
```

### 4. Video Generation with Remotion

Transform content into videos automatically:

```typescript
// lib/video/video-renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  title: string;
  content: string;
  format: 'reels' | 'tiktok' | 'shorts';
  style: 'infographic' | 'text-animation' | 'slideshow';
}

export async function renderContentVideo(config: VideoConfig) {
  const { title, content, format, style } = config;

  // Determine video dimensions based on platform
  const dimensions = {
    reels: { width: 1080, height: 1920 },
    tiktok: { width: 1080, height: 1920 },
    shorts: { width: 1080, height: 1920 }
  }[format];

  // Bundle Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.tsx'),
    webpackOverride: (config) => config
  });

  // Select composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: `${style}-template`,
    inputProps: {
      title,
      content: parseContentForVideo(content),
      ...dimensions
    }
  });

  // Render video
  const outputLocation = path.join(
    process.cwd(),
    'public',
    'videos',
    `${Date.now()}-${format}.mp4`
  );

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: {
      title,
      content: parseContentForVideo(content)
    }
  });

  return {
    videoPath: outputLocation,
    duration: composition.durationInFrames / composition.fps,
    dimensions
  };
}

function parseContentForVideo(content: string) {
  // Extract key points for video slides
  const lines = content.split('\n').filter(line => line.trim());
  const keyPoints = lines
    .filter(line => line.match(/^[-•*\d.]/))
    .slice(0, 5)
    .map(line => line.replace(/^[-•*\d.]\s*/, ''));

  return keyPoints;
}
```

### 5. Remotion Video Template Example

```tsx
// remotion/templates/InfographicTemplate.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig, interpolate } from 'remotion';

interface InfographicProps {
  title: string;
  content: string[];
}

export const InfographicTemplate: React.FC<InfographicProps> = ({
  title,
  content
}) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  const titleOpacity = interpolate(frame, [0, 30], [0, 1], {
    extrapolateRight: 'clamp'
  });

  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a2e' }}>
      <div
        style={{
          display: 'flex',
          flexDirection: 'column',
          justifyContent: 'center',
          alignItems: 'center',
          padding: 60,
          color: 'white'
        }}
      >
        <h1
          style={{
            fontSize: 72,
            fontWeight: 'bold',
            textAlign: 'center',
            opacity: titleOpacity,
            marginBottom: 60
          }}
        >
          {title}
        </h1>

        {content.map((point, index) => {
          const startFrame = 60 + index * 90;
          const opacity = interpolate(
            frame,
            [startFrame, startFrame + 30],
            [0, 1],
            { extrapolateRight: 'clamp' }
          );

          return (
            <div
              key={index}
              style={{
                fontSize: 48,
                marginBottom: 40,
                opacity,
                padding: 20,
                backgroundColor: 'rgba(255, 255, 255, 0.1)',
                borderRadius: 10
              }}
            >
              {index + 1}. {point}
            </div>
          );
        })}
      </div>
    </AbsoluteFill>
  );
};
```

### 6. Complete Pipeline API Route

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { scrapeNews } from '@/lib/scrapers/news-scraper';
import { generateBilingualContent } from '@/lib/ai/bilingual-generator';
import { renderContentVideo } from '@/lib/video/video-renderer';

export async function POST(request: NextRequest) {
  try {
    const { keyword, format, includeVideo } = await request.json();

    // Step 1: Research
    console.log('Scraping news for:', keyword);
    const researchData = await scrapeNews(keyword, 24);

    // Step 2: Generate content
    console.log('Generating content...');
    const content = await generateBilingualContent(
      keyword,
      format,
      researchData
    );

    // Step 3: Render video (optional)
    let video = null;
    if (includeVideo) {
      console.log('Rendering video...');
      video = await renderContentVideo({
        title: content.en.title,
        content: content.en.content,
        format: 'reels',
        style: 'infographic'
      });
    }

    return NextResponse.json({
      success: true,
      data: {
        content,
        video,
        research: researchData.slice(0, 5) // Top 5 sources
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

### 7. Frontend Usage Example

```tsx
// app/page.tsx
'use client';

import { useState } from 'react';

export default function HomePage() {
  const [keyword, setKeyword] = useState('');
  const [format, setFormat] = useState<ContentFormat>('toplist');
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState(null);

  const handleGenerate = async () => {
    setLoading(true);
    try {
      const response = await fetch('/api/generate', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          keyword,
          format,
          includeVideo: true
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
    <div className="container mx-auto p-8">
      <h1 className="text-4xl font-bold mb-8">
        AI Content Pipeline
      </h1>

      <div className="space-y-4">
        <input
          type="text"
          placeholder="Enter keyword..."
          value={keyword}
          onChange={(e) => setKeyword(e.target.value)}
          className="w-full p-4 border rounded"
        />

        <select
          value={format}
          onChange={(e) => setFormat(e.target.value as ContentFormat)}
          className="w-full p-4 border rounded"
        >
          <option value="toplist">Top List</option>
          <option value="pov">Point of View</option>
          <option value="case-study">Case Study</option>
          <option value="how-to">How To</option>
        </select>

        <button
          onClick={handleGenerate}
          disabled={loading || !keyword}
          className="w-full p-4 bg-blue-600 text-white rounded disabled:opacity-50"
        >
          {loading ? 'Generating...' : 'Generate Content'}
        </button>
      </div>

      {result && (
        <div className="mt-8 space-y-6">
          <div>
            <h2 className="text-2xl font-bold mb-4">English Version</h2>
            <div className="prose">{result.data.content.en.content}</div>
          </div>

          <div>
            <h2 className="text-2xl font-bold mb-4">Vietnamese Version</h2>
            <div className="prose">{result.data.content.vi.content}</div>
          </div>

          {result.data.video && (
            <div>
              <h2 className="text-2xl font-bold mb-4">Generated Video</h2>
              <video controls src={result.data.video.videoPath} />
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
async function generateBatchContent(keywords: string[]) {
  const results = await Promise.allSettled(
    keywords.map(keyword =>
      generateBilingualContent(keyword, 'toplist', [])
    )
  );

  return results
    .filter(r => r.status === 'fulfilled')
    .map(r => r.value);
}
```

### Custom AI Prompts

```typescript
const customPrompts = {
  'startup-news': `Focus on funding rounds, product launches, and market trends`,
  'tech-tutorial': `Provide code examples and step-by-step explanations`,
  'marketing-tips': `Include actionable tactics and real-world examples`
};

function buildCustomPrompt(keyword: string, category: string) {
  const basePrompt = buildPrompt({
    keyword,
    format: 'how-to',
    language: 'en',
    tone: 'expert',
    researchData: []
  });

  return `${basePrompt}\n\nAdditional context: ${customPrompts[category]}`;
}
```

## Troubleshooting

### API Rate Limits

```typescript
// lib/utils/rate-limiter.ts
export class RateLimiter {
  private queue: Array<() => Promise<any>> = [];
  private processing = false;

  async add<T>(fn: () => Promise<T>, delay: number = 1000): Promise<T> {
    return new Promise((resolve, reject) => {
      this.queue.push(async () => {
        try {
          const result = await fn();
          resolve(result);
        } catch (error) {
          reject(error);
        }
        await new Promise(r => setTimeout(r, delay));
      });

      if (!this.processing) {
        this.process();
      }
    });
  }

  private async process() {
    this.processing = true;
    while (this.queue.length > 0) {
      const fn = this.queue.shift()!;
      await fn();
    }
    this.processing = false;
  }
}
```

### Video Rendering Errors

If video rendering fails, check:
- Remotion dependencies are installed: `npm install @remotion/bundler @remotion/renderer`
- AWS credentials are properly configured for cloud rendering
- Sufficient disk space for temporary files

### Content Quality Issues

Fine-tune AI output:
```typescript
const qualitySettings = {
  temperature: 0.7, // Lower for more focused content
  max_tokens: 4096, // Increase for longer content
  top_p: 0.9 // Adjust creativity vs coherence
};
```

## Performance Optimization

```typescript
// Cache research data
import { redis } from '@/lib/redis';

async function getCachedResearch(keyword: string) {
  const cached = await redis.get(`research:${keyword}`);
  if (cached) return JSON.parse(cached);

  const fresh = await scrapeNews(keyword, 24);
  await redis.setex(`research:${keyword}`, 3600, JSON.stringify(fresh));
  return fresh;
}
```

This skill enables AI agents to fully utilize the Marketing Pipeline Share system for automated content creation, from research through video generation.
