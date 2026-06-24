---
name: marketing-pipeline-auto-content
description: Automated AI content pipeline for research, scriptwriting, auto-posting and video generation using Claude, OpenAI and Remotion
triggers:
  - how do I set up the AI content automation pipeline
  - automate content creation from research to video
  - use Claude and OpenAI for content generation
  - create automated marketing content pipeline
  - generate videos from content automatically with Remotion
  - crawl news sources and create content posts
  - build an AI content factory for social media
  - set up auto-posting content workflow
---

# Marketing Pipeline Auto Content

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the Ultimate AI Content Pipeline - a complete automated content creation system that handles research (crawling news sources), content generation (using Claude/OpenAI), and video rendering (via Remotion). The pipeline transforms keywords into ready-to-publish content across multiple formats and languages.

## What This Project Does

The Marketing Pipeline is an all-in-one content automation system that:

- **Auto-scans research sources**: Crawls TechCrunch, a16z, Twitter/X, LinkedIn for fresh data within 24 hours
- **Generates multi-format content**: Creates toplist, POV, case studies, how-to articles using Claude 3 or OpenAI
- **Supports bilingual output**: Produces Vietnamese and English content simultaneously
- **Renders videos automatically**: Uses Remotion to transform written content into Reels/TikTok/Shorts videos
- **Optimizes for platforms**: Exports videos in proper aspect ratios for different social platforms

## Installation

### Prerequisites

```bash
# Node.js 18+ and npm/yarn required
node --version  # Should be 18.x or higher
```

### Clone and Install

```bash
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
# AI Provider Keys
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Crawler/Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Remotion (Video Rendering)
REMOTION_LICENSE_KEY=your_remotion_license

# Next.js Configuration
NEXT_PUBLIC_API_URL=http://localhost:3000
```

### Start Development Server

```bash
npm run dev
# or
yarn dev
```

Access the application at `http://localhost:3000`

## Project Structure

```
marketing-pineline-share/
├── app/                    # Next.js app directory
│   ├── api/               # API routes
│   │   ├── research/      # Content research endpoints
│   │   ├── generate/      # Content generation endpoints
│   │   └── render/        # Video rendering endpoints
│   ├── components/        # React components
│   └── page.tsx          # Main page
├── lib/                   # Core utilities
│   ├── ai/               # AI provider integrations
│   │   ├── claude.ts     # Claude API wrapper
│   │   └── openai.ts     # OpenAI API wrapper
│   ├── crawler/          # Web scraping modules
│   ├── content/          # Content generation logic
│   └── video/            # Remotion video templates
├── remotion/             # Remotion video configurations
└── public/               # Static assets
```

## Key API Endpoints

### Research Endpoint

```typescript
// app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { crawlSources } from '@/lib/crawler';

export async function POST(req: NextRequest) {
  const { keyword, sources } = await req.json();
  
  // Crawl multiple news sources
  const results = await crawlSources({
    keyword,
    sources: sources || ['techcrunch', 'a16z', 'twitter'],
    timeframe: '24h'
  });
  
  return NextResponse.json({ data: results });
}
```

### Content Generation Endpoint

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { generateContent } from '@/lib/content/generator';

export async function POST(req: NextRequest) {
  const { research, format, language, tone, aiProvider } = await req.json();
  
  const content = await generateContent({
    researchData: research,
    format: format || 'toplist', // toplist, pov, casestudy, howto
    language: language || 'vi', // vi, en, both
    tone: tone || 'professional', // professional, friendly, humorous
    provider: aiProvider || 'claude' // claude, openai
  });
  
  return NextResponse.json({ content });
}
```

### Video Rendering Endpoint

```typescript
// app/api/render/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { renderVideo } from '@/lib/video/renderer';

export async function POST(req: NextRequest) {
  const { content, platform, template } = await req.json();
  
  const video = await renderVideo({
    content,
    platform: platform || 'reels', // reels, tiktok, shorts
    template: template || 'infographic',
    aspectRatio: platform === 'reels' ? '9:16' : '1:1'
  });
  
  return NextResponse.json({ videoUrl: video.url });
}
```

## Core Modules Usage

### AI Content Generation with Claude

```typescript
// lib/ai/claude.ts
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

export async function generateWithClaude(prompt: string, systemPrompt?: string) {
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    system: systemPrompt || 'You are an expert content writer.',
    messages: [
      {
        role: 'user',
        content: prompt
      }
    ]
  });
  
  return message.content[0].text;
}

// Usage example
export async function createTopListArticle(research: any, language: string) {
  const prompt = `
Based on this research data: ${JSON.stringify(research)}

Create a toplist article in ${language} with:
- Engaging headline
- 5-7 items with data-backed insights
- Each item with title, description, and key metrics
- Conclusion with actionable takeaways
`;

  return await generateWithClaude(prompt);
}
```

### AI Content Generation with OpenAI

```typescript
// lib/ai/openai.ts
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

export async function generateWithOpenAI(prompt: string, systemPrompt?: string) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: systemPrompt || 'You are an expert content writer.'
      },
      {
        role: 'user',
        content: prompt
      }
    ],
    temperature: 0.7,
    max_tokens: 4096
  });
  
  return completion.choices[0].message.content;
}
```

### Web Crawler for Research

```typescript
// lib/crawler/index.ts
import axios from 'axios';

interface CrawlOptions {
  keyword: string;
  sources: string[];
  timeframe: string;
}

export async function crawlSources(options: CrawlOptions) {
  const { keyword, sources, timeframe } = options;
  const results = [];
  
  for (const source of sources) {
    try {
      let data;
      
      if (source === 'techcrunch') {
        data = await crawlTechCrunch(keyword, timeframe);
      } else if (source === 'a16z') {
        data = await crawlA16Z(keyword, timeframe);
      } else if (source === 'twitter') {
        data = await crawlTwitter(keyword, timeframe);
      }
      
      results.push({
        source,
        data,
        crawledAt: new Date().toISOString()
      });
    } catch (error) {
      console.error(`Failed to crawl ${source}:`, error);
    }
  }
  
  return results;
}

async function crawlTechCrunch(keyword: string, timeframe: string) {
  // Using RapidAPI or custom scraper
  const response = await axios.get('https://api.rapidapi.com/techcrunch/search', {
    params: { q: keyword, timeframe },
    headers: {
      'X-RapidAPI-Key': process.env.RAPIDAPI_KEY,
      'X-RapidAPI-Host': 'techcrunch.p.rapidapi.com'
    }
  });
  
  return response.data;
}

async function crawlTwitter(keyword: string, timeframe: string) {
  // Twitter/X API integration
  const response = await axios.get('https://api.twitter.com/2/tweets/search/recent', {
    params: { 
      query: keyword,
      max_results: 20,
      'tweet.fields': 'created_at,public_metrics'
    },
    headers: {
      'Authorization': `Bearer ${process.env.TWITTER_BEARER_TOKEN}`
    }
  });
  
  return response.data;
}
```

### Content Generator

```typescript
// lib/content/generator.ts
import { generateWithClaude } from '@/lib/ai/claude';
import { generateWithOpenAI } from '@/lib/ai/openai';

interface GenerateContentOptions {
  researchData: any;
  format: 'toplist' | 'pov' | 'casestudy' | 'howto';
  language: 'vi' | 'en' | 'both';
  tone: 'professional' | 'friendly' | 'humorous';
  provider: 'claude' | 'openai';
}

export async function generateContent(options: GenerateContentOptions) {
  const { researchData, format, language, tone, provider } = options;
  
  const systemPrompt = buildSystemPrompt(format, tone);
  const userPrompt = buildUserPrompt(researchData, format, language);
  
  let content;
  if (provider === 'claude') {
    content = await generateWithClaude(userPrompt, systemPrompt);
  } else {
    content = await generateWithOpenAI(userPrompt, systemPrompt);
  }
  
  // Parse and structure the content
  const structured = parseContent(content, format);
  
  // Generate bilingual if needed
  if (language === 'both') {
    const otherLang = await translateContent(structured);
    return { vi: structured, en: otherLang };
  }
  
  return structured;
}

function buildSystemPrompt(format: string, tone: string): string {
  const toneDescriptions = {
    professional: 'You write in a professional, authoritative style with data-driven insights.',
    friendly: 'You write in a warm, conversational style that connects with readers.',
    humorous: 'You write with wit and humor while maintaining credibility.'
  };
  
  return `You are an expert content writer specializing in ${format} format. ${toneDescriptions[tone]}`;
}

function buildUserPrompt(research: any, format: string, language: string): string {
  const formatInstructions = {
    toplist: 'Create a numbered list article with 5-7 items, each with a title, detailed description, and key metrics.',
    pov: 'Write a perspective piece that takes a strong stance on the topic with supporting evidence.',
    casestudy: 'Write a detailed case study with problem, solution, implementation, and results sections.',
    howto: 'Create a step-by-step tutorial with clear instructions and examples.'
  };
  
  return `
Research Data:
${JSON.stringify(research, null, 2)}

Instructions:
${formatInstructions[format]}

Language: ${language === 'vi' ? 'Vietnamese' : 'English'}

Include:
- Compelling headline
- Hook paragraph
- Detailed body sections
- Data and statistics from the research
- Actionable conclusion
- 3-5 relevant hashtags

Format the output as JSON with: headline, hook, body (array), conclusion, hashtags
`;
}

function parseContent(content: string, format: string) {
  try {
    return JSON.parse(content);
  } catch {
    // Fallback parsing if AI doesn't return valid JSON
    return {
      headline: extractHeadline(content),
      hook: extractHook(content),
      body: extractBody(content, format),
      conclusion: extractConclusion(content),
      hashtags: extractHashtags(content)
    };
  }
}
```

### Video Rendering with Remotion

```typescript
// lib/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface RenderOptions {
  content: any;
  platform: 'reels' | 'tiktok' | 'shorts';
  template: string;
  aspectRatio: string;
}

export async function renderVideo(options: RenderOptions) {
  const { content, platform, template, aspectRatio } = options;
  
  // Bundle the Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.join(process.cwd(), 'remotion/index.ts'),
    webpackOverride: (config) => config,
  });
  
  // Select composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: template,
    inputProps: {
      content,
      platform,
      aspectRatio
    },
  });
  
  // Render video
  const outputLocation = path.join(
    process.cwd(),
    'public/videos',
    `${Date.now()}-${platform}.mp4`
  );
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: {
      content,
      platform,
      aspectRatio
    },
  });
  
  return {
    url: outputLocation.replace(path.join(process.cwd(), 'public'), ''),
    width: composition.width,
    height: composition.height,
    duration: composition.durationInFrames / composition.fps
  };
}
```

### Remotion Video Template

```typescript
// remotion/compositions/Infographic.tsx
import { AbsoluteFill, Sequence, useCurrentFrame, useVideoConfig } from 'remotion';
import React from 'react';

interface InfographicProps {
  content: {
    headline: string;
    body: Array<{ title: string; description: string }>;
  };
  platform: string;
  aspectRatio: string;
}

export const Infographic: React.FC<InfographicProps> = ({ content, platform }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  
  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a1a' }}>
      {/* Intro sequence */}
      <Sequence from={0} durationInFrames={fps * 2}>
        <AbsoluteFill style={{
          justifyContent: 'center',
          alignItems: 'center',
          opacity: frame / (fps * 2)
        }}>
          <h1 style={{ color: 'white', fontSize: 48, textAlign: 'center', padding: 20 }}>
            {content.headline}
          </h1>
        </AbsoluteFill>
      </Sequence>
      
      {/* Body items */}
      {content.body.map((item, index) => (
        <Sequence
          key={index}
          from={fps * (2 + index * 3)}
          durationInFrames={fps * 3}
        >
          <AbsoluteFill style={{
            justifyContent: 'center',
            alignItems: 'center',
            padding: 40
          }}>
            <div style={{ color: 'white', maxWidth: '80%' }}>
              <h2 style={{ fontSize: 36, marginBottom: 20 }}>
                {index + 1}. {item.title}
              </h2>
              <p style={{ fontSize: 24, lineHeight: 1.6 }}>
                {item.description}
              </p>
            </div>
          </AbsoluteFill>
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};
```

## Frontend Component Example

```typescript
// app/components/ContentPipeline.tsx
'use client';

import { useState } from 'react';

export default function ContentPipeline() {
  const [keyword, setKeyword] = useState('');
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState<any>(null);

  const runPipeline = async () => {
    setLoading(true);
    
    try {
      // Step 1: Research
      const researchRes = await fetch('/api/research', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          keyword,
          sources: ['techcrunch', 'twitter']
        })
      });
      const research = await researchRes.json();
      
      // Step 2: Generate Content
      const contentRes = await fetch('/api/generate', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          research: research.data,
          format: 'toplist',
          language: 'vi',
          tone: 'professional',
          aiProvider: 'claude'
        })
      });
      const content = await contentRes.json();
      
      // Step 3: Render Video
      const videoRes = await fetch('/api/render', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          content: content.content,
          platform: 'reels',
          template: 'infographic'
        })
      });
      const video = await videoRes.json();
      
      setResult({ research, content, video });
    } catch (error) {
      console.error('Pipeline error:', error);
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="p-8">
      <h1 className="text-3xl font-bold mb-6">AI Content Pipeline</h1>
      
      <div className="mb-4">
        <input
          type="text"
          value={keyword}
          onChange={(e) => setKeyword(e.target.value)}
          placeholder="Enter keyword (e.g., AI marketing)"
          className="w-full p-3 border rounded"
        />
      </div>
      
      <button
        onClick={runPipeline}
        disabled={loading || !keyword}
        className="bg-blue-600 text-white px-6 py-3 rounded hover:bg-blue-700 disabled:bg-gray-400"
      >
        {loading ? 'Processing...' : 'Generate Content'}
      </button>
      
      {result && (
        <div className="mt-8 space-y-6">
          <div className="border p-4 rounded">
            <h2 className="font-bold mb-2">Research Results</h2>
            <pre className="text-sm overflow-auto">
              {JSON.stringify(result.research, null, 2)}
            </pre>
          </div>
          
          <div className="border p-4 rounded">
            <h2 className="font-bold mb-2">Generated Content</h2>
            <div className="prose">
              <h3>{result.content.content.headline}</h3>
              <p>{result.content.content.hook}</p>
            </div>
          </div>
          
          <div className="border p-4 rounded">
            <h2 className="font-bold mb-2">Rendered Video</h2>
            <video controls className="w-full max-w-md">
              <source src={result.video.videoUrl} type="video/mp4" />
            </video>
          </div>
        </div>
      )}
    </div>
  );
}
```

## Common Patterns

### Full Pipeline Automation

```typescript
// lib/pipeline/automation.ts
export async function runFullPipeline(keyword: string, config: any) {
  // 1. Research Phase
  const research = await crawlSources({
    keyword,
    sources: config.sources || ['techcrunch', 'twitter'],
    timeframe: '24h'
  });
  
  // 2. Content Generation Phase
  const content = await generateContent({
    researchData: research,
    format: config.format || 'toplist',
    language: config.language || 'both',
    tone: config.tone || 'professional',
    provider: config.aiProvider || 'claude'
  });
  
  // 3. Video Rendering Phase
  const video = await renderVideo({
    content,
    platform: config.platform || 'reels',
    template: config.template || 'infographic',
    aspectRatio: '9:16'
  });
  
  // 4. Optional: Auto-post to platforms
  if (config.autoPost) {
    await postToSocialMedia({
      content,
      video,
      platforms: config.postTo || ['facebook', 'instagram']
    });
  }
  
  return { research, content, video };
}
```

### Batch Processing

```typescript
// Process multiple keywords
async function batchProcess(keywords: string[]) {
  const results = await Promise.all(
    keywords.map(keyword => 
      runFullPipeline(keyword, {
        format: 'toplist',
        language: 'vi',
        platform: 'reels'
      })
    )
  );
  
  return results;
}
```

## Troubleshooting

### API Rate Limits

```typescript
// lib/utils/rateLimit.ts
import pLimit from 'p-limit';

const limit = pLimit(3); // Max 3 concurrent requests

export async function batchWithRateLimit<T>(
  items: T[],
  fn: (item: T) => Promise<any>
) {
  return Promise.all(
    items.map(item => limit(() => fn(item)))
  );
}

// Usage
const results = await batchWithRateLimit(keywords, async (keyword) => {
  return await crawlSources({ keyword, sources: ['techcrunch'], timeframe: '24h' });
});
```

### Video Rendering Memory Issues

If video rendering fails with memory errors:

```typescript
// Increase Node.js heap size
// package.json
{
  "scripts": {
    "render": "NODE_OPTIONS='--max-old-space-size=4096' node render.js"
  }
}
```

### Content Quality Improvement

```typescript
// Add validation and retry logic
async function generateWithRetry(options: any, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    const content = await generateContent(options);
    
    if (validateContent(content)) {
      return content;
    }
    
    console.log(`Retry ${i + 1}/${maxRetries}`);
  }
  
  throw new Error('Failed to generate valid content');
}

function validateContent(content: any): boolean {
  return (
    content.headline?.length > 10 &&
    content.body?.length >= 3 &&
    content.conclusion?.length > 50
  );
}
```

### Environment Variables Missing

```typescript
// lib/utils/config.ts
export function validateEnv() {
  const required = [
    'ANTHROPIC_API_KEY',
    'OPENAI_API_KEY',
    'RAPIDAPI_KEY'
  ];
  
  const missing = required.filter(key => !process.env[key]);
  
  if (missing.length > 0) {
    throw new Error(`Missing environment variables: ${missing.join(', ')}`);
  }
}

// Call at app startup
validateEnv();
```

This pipeline system enables complete content automation from research to publication, significantly reducing content creation time while maintaining quality and consistency.
