---
name: ultimate-ai-content-pipeline
description: Automated content pipeline for research, scriptwriting, and video generation using AI (Claude, OpenAI) and Remotion
triggers:
  - automate content creation with ai
  - generate video content from text automatically
  - scrape news and create marketing content
  - build ai content pipeline with remotion
  - create automated social media content workflow
  - research and generate multilingual content with ai
  - turn articles into videos with remotion
  - set up automated marketing content system
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This project is a comprehensive TypeScript-based content automation system that handles the entire content creation workflow: from researching trending topics across news sources, generating multilingual articles in multiple formats, to rendering videos automatically using Remotion. It integrates Claude 3, OpenAI, web scraping APIs, and video rendering to create a complete "content factory" for marketers and creators.

## What It Does

- **Auto-Research**: Crawls and analyzes real-time data from sources like TechCrunch, a16z, Twitter/X, LinkedIn
- **AI Content Generation**: Creates content in multiple formats (toplist, POV, case study, how-to) using Claude/OpenAI
- **Multilingual Output**: Generates content in both English and Vietnamese with customizable tone
- **Video Rendering**: Automatically converts text content into videos/infographics using Remotion
- **Multi-Platform Optimization**: Exports videos optimized for Reels, TikTok, Shorts

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

### Environment Configuration

Create a `.env.local` file in the root directory:

```env
# AI Services
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Web Scraping
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_connection_string

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
│   │   ├── scraper/     # Web scraping modules
│   │   └── video/       # Remotion video generation
│   ├── types/           # TypeScript type definitions
│   └── utils/           # Utility functions
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Running the Application

```bash
# Development mode
npm run dev

# Production build
npm run build
npm run start

# Remotion studio (for video preview)
npm run remotion
```

## Core Modules

### 1. Content Research & Scraping

```typescript
// src/lib/scraper/news-scraper.ts
import axios from 'axios';

interface NewsArticle {
  title: string;
  url: string;
  source: string;
  publishedAt: string;
  content: string;
}

export async function scrapeNewsArticles(
  keyword: string,
  timeframe: '24h' | '7d' = '24h'
): Promise<NewsArticle[]> {
  const rapidApiKey = process.env.RAPIDAPI_KEY;
  
  const options = {
    method: 'GET',
    url: 'https://news-api14.p.rapidapi.com/v2/search/articles',
    params: {
      query: keyword,
      language: 'en',
      sortBy: 'publishedAt',
      timeRange: timeframe
    },
    headers: {
      'X-RapidAPI-Key': rapidApiKey,
      'X-RapidAPI-Host': 'news-api14.p.rapidapi.com'
    }
  };

  try {
    const response = await axios.request(options);
    return response.data.articles.map((article: any) => ({
      title: article.title,
      url: article.url,
      source: article.source.name,
      publishedAt: article.publishedAt,
      content: article.description || article.content
    }));
  } catch (error) {
    console.error('Error scraping news:', error);
    throw error;
  }
}

// Extract insights from scraped articles
export function extractInsights(articles: NewsArticle[]): string {
  const insights = articles.map(article => 
    `- ${article.title} (${article.source}, ${article.publishedAt})`
  ).join('\n');
  
  return `Recent insights from ${articles.length} sources:\n${insights}`;
}
```

### 2. AI Content Generation with Claude

```typescript
// src/lib/ai/claude-content-generator.ts
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

export type ContentFormat = 'toplist' | 'pov' | 'case-study' | 'how-to';
export type ContentTone = 'expert' | 'friendly' | 'humorous';

interface ContentRequest {
  keyword: string;
  format: ContentFormat;
  tone: ContentTone;
  language: 'en' | 'vi';
  researchData: string;
}

export async function generateContent(request: ContentRequest): Promise<string> {
  const formatPrompts = {
    'toplist': 'Create a numbered list article with detailed explanations for each point.',
    'pov': 'Write from a personal perspective sharing unique insights and opinions.',
    'case-study': 'Present a detailed analysis with problem, solution, and results.',
    'how-to': 'Write a step-by-step tutorial with actionable instructions.'
  };

  const tonePrompts = {
    'expert': 'Use professional language with industry terminology and data-driven insights.',
    'friendly': 'Write in a conversational, approachable style that connects with readers.',
    'humorous': 'Include wit and light humor while maintaining informativeness.'
  };

  const prompt = `
You are a professional content creator. Generate a ${request.format} article about "${request.keyword}".

Research Data:
${request.researchData}

Requirements:
- Format: ${formatPrompts[request.format]}
- Tone: ${tonePrompts[request.tone]}
- Language: ${request.language === 'en' ? 'English' : 'Vietnamese'}
- Include real data and statistics from the research
- Make it engaging and valuable for the target audience
- Length: 1000-1500 words

Generate the complete article now:
`;

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [
      { role: 'user', content: prompt }
    ],
  });

  return message.content[0].type === 'text' ? message.content[0].text : '';
}
```

### 3. Alternative: OpenAI Content Generation

```typescript
// src/lib/ai/openai-content-generator.ts
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

export async function generateWithOpenAI(request: ContentRequest): Promise<string> {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: 'You are an expert content creator specializing in marketing and social media content.'
      },
      {
        role: 'user',
        content: `Create a ${request.format} article about "${request.keyword}" in ${request.language} with a ${request.tone} tone. Use this research: ${request.researchData}`
      }
    ],
    temperature: 0.7,
    max_tokens: 3000,
  });

  return completion.choices[0]?.message?.content || '';
}
```

### 4. Video Generation with Remotion

```typescript
// src/lib/video/video-generator.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

export interface VideoConfig {
  title: string;
  content: string[];
  duration: number;
  format: 'reels' | 'tiktok' | 'shorts'; // 9:16 aspect ratio
}

export async function generateVideo(config: VideoConfig): Promise<string> {
  const compositionId = 'ContentVideo';
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: {
      title: config.title,
      content: config.content,
      duration: config.duration,
    },
  });

  const outputPath = path.join(process.cwd(), 'public', 'videos', `${Date.now()}.mp4`);

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: {
      title: config.title,
      content: config.content,
    },
  });

  return outputPath;
}
```

```typescript
// remotion/index.ts
import { Composition } from 'remotion';
import { ContentVideo } from './ContentVideo';

export const RemotionRoot: React.FC = () => {
  return (
    <>
      <Composition
        id="ContentVideo"
        component={ContentVideo}
        durationInFrames={300}
        fps={30}
        width={1080}
        height={1920} // 9:16 for vertical video
      />
    </>
  );
};
```

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, interpolate, Sequence } from 'remotion';
import React from 'react';

interface ContentVideoProps {
  title: string;
  content: string[];
  duration: number;
}

export const ContentVideo: React.FC<ContentVideoProps> = ({ title, content }) => {
  const frame = useCurrentFrame();
  
  const opacity = interpolate(frame, [0, 30], [0, 1], {
    extrapolateRight: 'clamp',
  });

  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a1a' }}>
      <Sequence from={0} durationInFrames={90}>
        <AbsoluteFill style={{ 
          opacity,
          justifyContent: 'center',
          alignItems: 'center',
          padding: 60 
        }}>
          <h1 style={{ 
            color: 'white', 
            fontSize: 72,
            textAlign: 'center',
            fontWeight: 'bold'
          }}>
            {title}
          </h1>
        </AbsoluteFill>
      </Sequence>
      
      {content.map((text, index) => (
        <Sequence key={index} from={90 + (index * 60)} durationInFrames={60}>
          <AbsoluteFill style={{ 
            justifyContent: 'center',
            alignItems: 'center',
            padding: 80 
          }}>
            <p style={{ 
              color: 'white', 
              fontSize: 48,
              textAlign: 'center',
              lineHeight: 1.5
            }}>
              {text}
            </p>
          </AbsoluteFill>
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};
```

## Complete Workflow Example

```typescript
// src/app/api/generate-content/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { scrapeNewsArticles, extractInsights } from '@/lib/scraper/news-scraper';
import { generateContent } from '@/lib/ai/claude-content-generator';
import { generateVideo } from '@/lib/video/video-generator';

export async function POST(request: NextRequest) {
  try {
    const { keyword, format, tone, language } = await request.json();

    // Step 1: Research
    console.log('Starting research phase...');
    const articles = await scrapeNewsArticles(keyword, '24h');
    const researchData = extractInsights(articles);

    // Step 2: Generate Content
    console.log('Generating content with AI...');
    const content = await generateContent({
      keyword,
      format,
      tone,
      language,
      researchData,
    });

    // Step 3: Extract key points for video
    const contentPoints = content
      .split('\n')
      .filter(line => line.trim().length > 0)
      .slice(0, 5); // First 5 points

    // Step 4: Generate Video
    console.log('Rendering video...');
    const videoPath = await generateVideo({
      title: keyword,
      content: contentPoints,
      duration: 300,
      format: 'reels',
    });

    return NextResponse.json({
      success: true,
      data: {
        content,
        videoPath,
        researchSources: articles.length,
      },
    });
  } catch (error) {
    console.error('Content generation error:', error);
    return NextResponse.json(
      { success: false, error: 'Failed to generate content' },
      { status: 500 }
    );
  }
}
```

## Frontend Component Example

```typescript
// src/components/ContentGenerator.tsx
'use client';

import { useState } from 'react';

export default function ContentGenerator() {
  const [keyword, setKeyword] = useState('');
  const [format, setFormat] = useState<'toplist' | 'pov' | 'case-study' | 'how-to'>('toplist');
  const [tone, setTone] = useState<'expert' | 'friendly' | 'humorous'>('friendly');
  const [language, setLanguage] = useState<'en' | 'vi'>('en');
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState<any>(null);

  const handleGenerate = async () => {
    setLoading(true);
    try {
      const response = await fetch('/api/generate-content', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ keyword, format, tone, language }),
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
    <div className="max-w-4xl mx-auto p-8">
      <h1 className="text-3xl font-bold mb-8">AI Content Pipeline</h1>
      
      <div className="space-y-4">
        <input
          type="text"
          placeholder="Enter keyword (e.g., AI Marketing)"
          value={keyword}
          onChange={(e) => setKeyword(e.target.value)}
          className="w-full p-3 border rounded"
        />
        
        <div className="grid grid-cols-3 gap-4">
          <select value={format} onChange={(e) => setFormat(e.target.value as any)} className="p-3 border rounded">
            <option value="toplist">Top List</option>
            <option value="pov">POV</option>
            <option value="case-study">Case Study</option>
            <option value="how-to">How-to</option>
          </select>
          
          <select value={tone} onChange={(e) => setTone(e.target.value as any)} className="p-3 border rounded">
            <option value="expert">Expert</option>
            <option value="friendly">Friendly</option>
            <option value="humorous">Humorous</option>
          </select>
          
          <select value={language} onChange={(e) => setLanguage(e.target.value as any)} className="p-3 border rounded">
            <option value="en">English</option>
            <option value="vi">Vietnamese</option>
          </select>
        </div>
        
        <button
          onClick={handleGenerate}
          disabled={loading || !keyword}
          className="w-full bg-blue-600 text-white p-3 rounded hover:bg-blue-700 disabled:bg-gray-400"
        >
          {loading ? 'Generating...' : 'Generate Content & Video'}
        </button>
        
        {result?.success && (
          <div className="mt-8 space-y-4">
            <div className="p-4 bg-gray-100 rounded">
              <h2 className="font-bold mb-2">Generated Content:</h2>
              <pre className="whitespace-pre-wrap">{result.data.content}</pre>
            </div>
            
            {result.data.videoPath && (
              <div>
                <h2 className="font-bold mb-2">Generated Video:</h2>
                <video src={result.data.videoPath} controls className="w-full rounded" />
              </div>
            )}
          </div>
        )}
      </div>
    </div>
  );
}
```

## Common Patterns

### Batch Content Generation

```typescript
// Generate multiple content pieces at once
export async function batchGenerateContent(keywords: string[]) {
  const results = await Promise.all(
    keywords.map(async (keyword) => {
      const articles = await scrapeNewsArticles(keyword, '24h');
      const researchData = extractInsights(articles);
      
      return {
        keyword,
        content: await generateContent({
          keyword,
          format: 'toplist',
          tone: 'friendly',
          language: 'en',
          researchData,
        }),
      };
    })
  );
  
  return results;
}
```

### Scheduled Content Pipeline

```typescript
// src/lib/scheduler/content-scheduler.ts
import cron from 'node-cron';

export function scheduleContentGeneration(keywords: string[]) {
  // Run every day at 8 AM
  cron.schedule('0 8 * * *', async () => {
    console.log('Starting scheduled content generation...');
    
    for (const keyword of keywords) {
      try {
        await generateAndPublish(keyword);
      } catch (error) {
        console.error(`Failed for keyword: ${keyword}`, error);
      }
    }
  });
}

async function generateAndPublish(keyword: string) {
  // Research -> Generate -> Render Video -> Auto-publish
  // Implementation here
}
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement rate limiting
import pLimit from 'p-limit';

const limit = pLimit(3); // Max 3 concurrent requests

export async function scrapeWithRateLimit(keywords: string[]) {
  return Promise.all(
    keywords.map((keyword) =>
      limit(() => scrapeNewsArticles(keyword, '24h'))
    )
  );
}
```

### Video Rendering Memory Issues

If Remotion renders fail due to memory:

```bash
# Increase Node.js memory limit
NODE_OPTIONS="--max-old-space-size=4096" npm run build
```

### Claude API Errors

```typescript
// Add retry logic for Claude API
async function generateWithRetry(request: ContentRequest, retries = 3): Promise<string> {
  for (let i = 0; i < retries; i++) {
    try {
      return await generateContent(request);
    } catch (error: any) {
      if (error.status === 429 && i < retries - 1) {
        await new Promise(resolve => setTimeout(resolve, 2000 * (i + 1)));
        continue;
      }
      throw error;
    }
  }
  throw new Error('Max retries exceeded');
}
```

### Missing Environment Variables

```typescript
// src/lib/config/validate-env.ts
export function validateEnv() {
  const required = [
    'ANTHROPIC_API_KEY',
    'OPENAI_API_KEY',
    'RAPIDAPI_KEY',
  ];
  
  const missing = required.filter(key => !process.env[key]);
  
  if (missing.length > 0) {
    throw new Error(`Missing environment variables: ${missing.join(', ')}`);
  }
}
```

## Performance Optimization

```typescript
// Cache research results to avoid redundant API calls
import { Redis } from '@upstash/redis';

const redis = new Redis({
  url: process.env.UPSTASH_REDIS_URL,
  token: process.env.UPSTASH_REDIS_TOKEN,
});

export async function getCachedResearch(keyword: string) {
  const cacheKey = `research:${keyword}:${new Date().toISOString().split('T')[0]}`;
  
  const cached = await redis.get(cacheKey);
  if (cached) return cached;
  
  const articles = await scrapeNewsArticles(keyword, '24h');
  await redis.set(cacheKey, articles, { ex: 86400 }); // 24h cache
  
  return articles;
}
```

This skill enables AI agents to help developers build and customize automated content pipelines with research, AI generation, and video rendering capabilities.
