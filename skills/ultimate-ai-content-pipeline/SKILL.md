---
name: ultimate-ai-content-pipeline
description: Automate content creation from research to video generation using AI (Claude/OpenAI) and Remotion
triggers:
  - how do I automate content creation with AI
  - set up an AI content pipeline with video generation
  - create automated marketing content with Claude and OpenAI
  - generate videos from AI-written content using Remotion
  - build an AI-powered content automation system
  - scrape news and generate content automatically
  - automate social media content with AI research
  - create multilingual content with AI video rendering
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

A comprehensive TypeScript-based content automation system that handles the entire content lifecycle: automatic news research/scraping, AI-powered content generation in multiple formats and languages (using Claude 3 and OpenAI), and automatic video/image rendering with Remotion. Perfect for marketers, content creators, and agencies looking to scale content production.

## What This Project Does

Ultimate AI Content Pipeline is an all-in-one content automation system that:

- **Auto-scans research sources**: Crawls TechCrunch, a16z, Twitter/X, LinkedIn for fresh data within 24 hours
- **Generates diverse content formats**: Toplists, POV articles, case studies, how-to guides
- **Multilingual support**: Simultaneously creates English and Vietnamese content
- **Video generation**: Automatically renders infographics and short-form videos from written content using Remotion
- **Multi-platform optimization**: Exports videos for Reels, TikTok, Shorts

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

Create a `.env.local` file in the root directory:

```bash
# AI Services
OPENAI_API_KEY=your_openai_key_here
ANTHROPIC_API_KEY=your_claude_api_key_here

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key_here

# Next.js
NEXT_PUBLIC_API_URL=http://localhost:3000

# Optional: Database (if using)
DATABASE_URL=your_database_url_here

# Remotion Configuration
REMOTION_AWS_ACCESS_KEY_ID=your_aws_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app router pages
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI service integrations (Claude, OpenAI)
│   │   ├── research/    # Web scraping and research modules
│   │   └── video/       # Remotion video generation
│   └── utils/           # Helper functions
├── public/              # Static assets
├── remotion/            # Remotion video templates
└── package.json
```

## Running the Development Server

```bash
npm run dev
# or
yarn dev
# or
pnpm dev
```

Access the application at `http://localhost:3000`

## Key API Patterns

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

export async function scrapeRecentNews(
  keyword: string,
  sources: string[] = ['techcrunch', 'a16z']
): Promise<ResearchSource[]> {
  const results: ResearchSource[] = [];
  
  for (const source of sources) {
    try {
      const response = await axios.get(
        `https://api.rapidapi.com/search/${source}`,
        {
          params: { q: keyword, hours: 24 },
          headers: {
            'X-RapidAPI-Key': process.env.RAPIDAPI_KEY!,
            'X-RapidAPI-Host': 'news-scraper.p.rapidapi.com'
          }
        }
      );
      
      results.push(...response.data.articles);
    } catch (error) {
      console.error(`Failed to scrape ${source}:`, error);
    }
  }
  
  return results;
}

export function extractInsights(articles: ResearchSource[]): string {
  // Combine and summarize key points from articles
  return articles
    .map(a => `${a.title}: ${a.content.slice(0, 200)}`)
    .join('\n\n');
}
```

### 2. AI Content Generation with Claude

```typescript
// src/lib/ai/claude-generator.ts
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY!
});

interface ContentConfig {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  researchData: string;
}

export async function generateContent(config: ContentConfig): Promise<string> {
  const formatPrompts = {
    'toplist': 'Create a numbered list article with detailed analysis',
    'pov': 'Write from a unique perspective with strong opinions',
    'case-study': 'Analyze with concrete examples and data',
    'how-to': 'Provide step-by-step actionable instructions'
  };

  const tonePrompts = {
    'expert': 'Use professional, authoritative language',
    'friendly': 'Use conversational, approachable language',
    'humorous': 'Include wit and engaging storytelling'
  };

  const prompt = `
Research Data:
${config.researchData}

Task: ${formatPrompts[config.format]} about "${config.keyword}"
Language: ${config.language === 'vi' ? 'Vietnamese' : 'English'}
Tone: ${tonePrompts[config.tone]}

Requirements:
- Include data and statistics from the research
- Make it SEO-optimized
- Add practical insights
- Length: 1000-1500 words
`;

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [{
      role: 'user',
      content: prompt
    }]
  });

  return message.content[0].type === 'text' 
    ? message.content[0].text 
    : '';
}
```

### 3. OpenAI Alternative Implementation

```typescript
// src/lib/ai/openai-generator.ts
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY!
});

export async function generateContentOpenAI(config: ContentConfig): Promise<string> {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: 'You are an expert content creator specializing in marketing and tech content.'
      },
      {
        role: 'user',
        content: `Create a ${config.format} article about "${config.keyword}" using this research:\n\n${config.researchData}`
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
// src/lib/video/render.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  title: string;
  content: string;
  format: 'reel' | 'tiktok' | 'short';
}

const aspectRatios = {
  'reel': { width: 1080, height: 1920 },
  'tiktok': { width: 1080, height: 1920 },
  'short': { width: 1080, height: 1920 }
};

export async function generateVideo(config: VideoConfig): Promise<string> {
  // Bundle the Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.join(process.cwd(), 'remotion/index.ts'),
    webpackOverride: (config) => config
  });

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title: config.title,
      content: config.content,
      ...aspectRatios[config.format]
    }
  });

  const outputPath = path.join(
    process.cwd(), 
    'public/videos', 
    `${Date.now()}-${config.format}.mp4`
  );

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: composition.props
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
  content: string;
}

export const ContentVideo: React.FC<ContentVideoProps> = ({ title, content }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  const opacity = Math.min(1, frame / 30);

  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a1a' }}>
      <Sequence from={0} durationInFrames={fps * 3}>
        <AbsoluteFill style={{ 
          justifyContent: 'center', 
          alignItems: 'center',
          opacity 
        }}>
          <h1 style={{ 
            color: 'white', 
            fontSize: 80,
            textAlign: 'center',
            padding: '0 100px'
          }}>
            {title}
          </h1>
        </AbsoluteFill>
      </Sequence>
      
      <Sequence from={fps * 3} durationInFrames={fps * 5}>
        <AbsoluteFill style={{ 
          justifyContent: 'center', 
          alignItems: 'center',
          padding: '100px'
        }}>
          <p style={{ 
            color: 'white', 
            fontSize: 40,
            lineHeight: 1.6
          }}>
            {content.slice(0, 300)}...
          </p>
        </AbsoluteFill>
      </Sequence>
    </AbsoluteFill>
  );
};
```

## Complete Pipeline Example

```typescript
// src/app/api/generate-content/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { scrapeRecentNews, extractInsights } from '@/lib/research/scraper';
import { generateContent } from '@/lib/ai/claude-generator';
import { generateVideo } from '@/lib/video/render';

export async function POST(req: NextRequest) {
  try {
    const { keyword, format, language, tone, videoFormat } = await req.json();

    // Step 1: Research
    const articles = await scrapeRecentNews(keyword);
    const researchData = extractInsights(articles);

    // Step 2: Generate Content
    const content = await generateContent({
      keyword,
      format,
      language,
      tone,
      researchData
    });

    // Step 3: Generate Video (optional)
    let videoPath;
    if (videoFormat) {
      videoPath = await generateVideo({
        title: keyword,
        content,
        format: videoFormat
      });
    }

    return NextResponse.json({
      success: true,
      content,
      videoUrl: videoPath ? `/videos/${path.basename(videoPath)}` : null,
      sources: articles.map(a => ({ title: a.title, url: a.url }))
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

## Frontend Integration

```typescript
// src/components/ContentGenerator.tsx
'use client';

import { useState } from 'react';

export default function ContentGenerator() {
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState<any>(null);

  const handleGenerate = async (formData: FormData) => {
    setLoading(true);
    
    const response = await fetch('/api/generate-content', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        keyword: formData.get('keyword'),
        format: formData.get('format'),
        language: formData.get('language'),
        tone: formData.get('tone'),
        videoFormat: formData.get('videoFormat')
      })
    });

    const data = await response.json();
    setResult(data);
    setLoading(false);
  };

  return (
    <div className="max-w-4xl mx-auto p-8">
      <form action={handleGenerate}>
        <input 
          name="keyword" 
          placeholder="Enter keyword..." 
          required 
          className="w-full p-3 border rounded"
        />
        
        <select name="format" className="w-full p-3 mt-4 border rounded">
          <option value="toplist">Top List</option>
          <option value="pov">POV Article</option>
          <option value="case-study">Case Study</option>
          <option value="how-to">How-to Guide</option>
        </select>

        <select name="language" className="w-full p-3 mt-4 border rounded">
          <option value="en">English</option>
          <option value="vi">Vietnamese</option>
        </select>

        <button 
          type="submit" 
          disabled={loading}
          className="w-full mt-6 bg-blue-600 text-white p-3 rounded"
        >
          {loading ? 'Generating...' : 'Generate Content'}
        </button>
      </form>

      {result && (
        <div className="mt-8 p-6 bg-gray-50 rounded">
          <div className="prose max-w-none">
            {result.content}
          </div>
          
          {result.videoUrl && (
            <video controls className="mt-6 w-full">
              <source src={result.videoUrl} type="video/mp4" />
            </video>
          )}
        </div>
      )}
    </div>
  );
}
```

## Remotion Configuration

```typescript
// remotion.config.ts
import { Config } from '@remotion/cli/config';

Config.setVideoImageFormat('jpeg');
Config.setOverwriteOutput(true);
Config.setConcurrency(4);
Config.setCodec('h264');
```

## Common Patterns

### Batch Content Generation

```typescript
async function batchGenerate(keywords: string[]) {
  const results = await Promise.all(
    keywords.map(async (keyword) => {
      const articles = await scrapeRecentNews(keyword);
      const content = await generateContent({
        keyword,
        format: 'toplist',
        language: 'en',
        tone: 'expert',
        researchData: extractInsights(articles)
      });
      return { keyword, content };
    })
  );
  
  return results;
}
```

### Scheduling Content Publication

```typescript
import { schedule } from 'node-cron';

// Schedule content generation every day at 8 AM
schedule('0 8 * * *', async () => {
  const trendingTopics = await fetchTrendingTopics();
  await batchGenerate(trendingTopics.slice(0, 5));
});
```

## Troubleshooting

**API Rate Limits**: Implement retry logic with exponential backoff:

```typescript
async function withRetry<T>(fn: () => Promise<T>, retries = 3): Promise<T> {
  try {
    return await fn();
  } catch (error) {
    if (retries > 0 && error.status === 429) {
      await new Promise(resolve => setTimeout(resolve, 2000 * (4 - retries)));
      return withRetry(fn, retries - 1);
    }
    throw error;
  }
}
```

**Remotion Rendering Memory Issues**: Reduce concurrency in config or use cloud rendering:

```typescript
Config.setConcurrency(1); // For low-memory environments
```

**Missing Research Data**: Add fallback to AI knowledge when scraping fails:

```typescript
const researchData = articles.length > 0 
  ? extractInsights(articles)
  : `Use your knowledge about ${keyword}`;
```
