---
name: marketing-pipeline-ai-content-automation
description: Automate content creation from research to video generation using AI (Claude/OpenAI) and Remotion
triggers:
  - how do I automatically generate content with AI research
  - set up automated marketing content pipeline
  - create AI-powered content from research to video
  - automate blog posts and video generation with Claude
  - build content automation workflow with AI
  - generate multi-format content automatically
  - use Remotion to create videos from articles
  - crawl news and generate content with AI
---

# Marketing Pipeline AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This project is an end-to-end content automation pipeline that crawls news sources, generates articles in multiple formats using Claude/OpenAI, and automatically renders videos using Remotion. It's designed for marketers, content creators, and businesses to reduce content production time by up to 90%.

## What It Does

The **Ultimate AI Content Pipeline** automates:

1. **Research Phase**: Auto-crawls trending news from TechCrunch, a16z, Twitter/X, LinkedIn
2. **Content Generation**: Uses Claude 3/OpenAI to create articles in various formats (Top Lists, POV, Case Studies, How-To)
3. **Multi-language Support**: Generates content in both English and Vietnamese
4. **Video Rendering**: Automatically creates infographics and short videos using Remotion
5. **Multi-platform Optimization**: Exports videos optimized for Reels, TikTok, and YouTube Shorts

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

# Set up environment variables
cp .env.example .env
```

## Configuration

Create a `.env` file with the following variables:

```bash
# AI API Keys
ANTHROPIC_API_KEY=your_claude_api_key_here
OPENAI_API_KEY=your_openai_api_key_here

# Research API Keys
RAPID_API_KEY=your_rapidapi_key_here

# Twitter/X API (optional)
TWITTER_BEARER_TOKEN=your_twitter_bearer_token_here

# Application Settings
NEXT_PUBLIC_APP_URL=http://localhost:3000
NODE_ENV=development

# Remotion Settings
REMOTION_AWS_ACCESS_KEY_ID=your_aws_access_key_here
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret_key_here
```

## Project Structure

```typescript
// Typical project structure
src/
├── app/                    # Next.js app directory
│   ├── api/               # API routes
│   │   ├── research/      # News crawling endpoints
│   │   ├── generate/      # Content generation endpoints
│   │   └── render/        # Video rendering endpoints
│   └── page.tsx           # Main UI
├── lib/
│   ├── ai/                # AI integration (Claude, OpenAI)
│   ├── crawlers/          # News source crawlers
│   └── utils/             # Utility functions
├── remotion/              # Remotion video templates
│   ├── compositions/      # Video compositions
│   └── assets/            # Video assets
└── types/                 # TypeScript type definitions
```

## Core Usage Patterns

### 1. Research & Crawl News

```typescript
// lib/crawlers/news-crawler.ts
import axios from 'axios';

interface NewsArticle {
  title: string;
  url: string;
  publishedAt: string;
  source: string;
  summary?: string;
}

export async function crawlTechCrunch(
  keyword: string,
  hours: number = 24
): Promise<NewsArticle[]> {
  const response = await axios.get(
    'https://techcrunch.com/wp-json/wp/v2/posts',
    {
      params: {
        search: keyword,
        per_page: 10,
        after: new Date(Date.now() - hours * 60 * 60 * 1000).toISOString()
      }
    }
  );

  return response.data.map((article: any) => ({
    title: article.title.rendered,
    url: article.link,
    publishedAt: article.date,
    source: 'TechCrunch',
    summary: article.excerpt.rendered
  }));
}

// Using RapidAPI for news aggregation
export async function crawlNewsAPI(keyword: string): Promise<NewsArticle[]> {
  const options = {
    method: 'GET',
    url: 'https://news-api14.p.rapidapi.com/v2/search/articles',
    params: { q: keyword, lang: 'en' },
    headers: {
      'X-RapidAPI-Key': process.env.RAPID_API_KEY,
      'X-RapidAPI-Host': 'news-api14.p.rapidapi.com'
    }
  };

  const response = await axios.request(options);
  return response.data.articles;
}
```

### 2. Generate Content with Claude/OpenAI

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

interface ContentRequest {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  tone: 'expert' | 'friendly' | 'humorous';
  language: 'en' | 'vi';
  researchData: NewsArticle[];
}

export async function generateContentWithClaude(
  request: ContentRequest
): Promise<string> {
  const prompt = buildPrompt(request);

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

  return message.content[0].type === 'text' 
    ? message.content[0].text 
    : '';
}

export async function generateContentWithOpenAI(
  request: ContentRequest
): Promise<string> {
  const prompt = buildPrompt(request);

  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: 'You are an expert content writer specializing in marketing and technology.'
      },
      {
        role: 'user',
        content: prompt
      }
    ],
    max_tokens: 4096
  });

  return completion.choices[0].message.content || '';
}

function buildPrompt(request: ContentRequest): string {
  const researchSummary = request.researchData
    .map(article => `- ${article.title} (${article.source}): ${article.summary}`)
    .join('\n');

  return `
Create a ${request.format} article about "${request.keyword}" in ${request.language} language.
Tone: ${request.tone}

Recent research data:
${researchSummary}

Requirements:
- Include data-backed insights from the research
- Make it engaging and SEO-optimized
- Use proper formatting (headings, bullet points)
- Include a compelling introduction and conclusion
${request.language === 'vi' ? '- Write in Vietnamese' : '- Write in English'}
`;
}
```

### 3. API Route for Content Generation

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { crawlNewsAPI } from '@/lib/crawlers/news-crawler';
import { generateContentWithClaude } from '@/lib/ai/content-generator';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const { keyword, format, tone, language } = body;

    // Step 1: Research
    const researchData = await crawlNewsAPI(keyword);

    // Step 2: Generate content
    const content = await generateContentWithClaude({
      keyword,
      format,
      tone,
      language,
      researchData
    });

    return NextResponse.json({
      success: true,
      content,
      sources: researchData.length
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

### 4. Remotion Video Generation

```typescript
// remotion/compositions/ArticleVideo.tsx
import { AbsoluteFill, useCurrentFrame, interpolate } from 'remotion';
import React from 'react';

interface ArticleVideoProps {
  title: string;
  keyPoints: string[];
  bgColor?: string;
}

export const ArticleVideo: React.FC<ArticleVideoProps> = ({
  title,
  keyPoints,
  bgColor = '#1a1a1a'
}) => {
  const frame = useCurrentFrame();

  const titleOpacity = interpolate(frame, [0, 30], [0, 1], {
    extrapolateRight: 'clamp'
  });

  return (
    <AbsoluteFill style={{ backgroundColor: bgColor }}>
      <div style={{ padding: 60 }}>
        <h1
          style={{
            color: 'white',
            fontSize: 72,
            fontWeight: 'bold',
            opacity: titleOpacity,
            marginBottom: 40
          }}
        >
          {title}
        </h1>
        
        {keyPoints.map((point, index) => {
          const pointOpacity = interpolate(
            frame,
            [60 + index * 30, 90 + index * 30],
            [0, 1],
            { extrapolateRight: 'clamp' }
          );

          return (
            <div
              key={index}
              style={{
                color: 'white',
                fontSize: 36,
                marginBottom: 20,
                opacity: pointOpacity
              }}
            >
              • {point}
            </div>
          );
        })}
      </div>
    </AbsoluteFill>
  );
};
```

```typescript
// remotion/index.ts
import { registerRoot } from 'remotion';
import { ArticleVideo } from './compositions/ArticleVideo';

export const RemotionRoot = () => {
  return (
    <>
      <Composition
        id="ArticleVideo"
        component={ArticleVideo}
        durationInFrames={300}
        fps={30}
        width={1080}
        height={1920}
        defaultProps={{
          title: 'Your Article Title',
          keyPoints: [
            'First key insight',
            'Second key insight',
            'Third key insight'
          ]
        }}
      />
    </>
  );
};

registerRoot(RemotionRoot);
```

### 5. Render Video API

```typescript
// app/api/render/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

export async function POST(request: NextRequest) {
  try {
    const { title, keyPoints } = await request.json();

    const compositionId = 'ArticleVideo';
    const bundleLocation = await bundle({
      entryPoint: path.resolve('./remotion/index.ts'),
      webpackOverride: (config) => config
    });

    const composition = await selectComposition({
      serveUrl: bundleLocation,
      id: compositionId,
      inputProps: { title, keyPoints }
    });

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
      inputProps: { title, keyPoints }
    });

    return NextResponse.json({
      success: true,
      videoUrl: `/videos/${path.basename(outputLocation)}`
    });
  } catch (error) {
    console.error('Video rendering error:', error);
    return NextResponse.json(
      { success: false, error: 'Failed to render video' },
      { status: 500 }
    );
  }
}
```

### 6. Frontend Integration

```typescript
// app/page.tsx
'use client';

import { useState } from 'react';

export default function Home() {
  const [keyword, setKeyword] = useState('');
  const [format, setFormat] = useState<'toplist' | 'pov' | 'case-study' | 'how-to'>('toplist');
  const [loading, setLoading] = useState(false);
  const [content, setContent] = useState('');
  const [videoUrl, setVideoUrl] = useState('');

  const handleGenerate = async () => {
    setLoading(true);
    try {
      // Generate content
      const contentRes = await fetch('/api/generate', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          keyword,
          format,
          tone: 'expert',
          language: 'en'
        })
      });
      const contentData = await contentRes.json();
      setContent(contentData.content);

      // Extract key points and generate video
      const keyPoints = extractKeyPoints(contentData.content);
      const videoRes = await fetch('/api/render', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          title: keyword,
          keyPoints
        })
      });
      const videoData = await videoRes.json();
      setVideoUrl(videoData.videoUrl);
    } catch (error) {
      console.error('Generation error:', error);
    } finally {
      setLoading(false);
    }
  };

  const extractKeyPoints = (content: string): string[] => {
    // Simple extraction - improve based on your content structure
    const lines = content.split('\n').filter(line => 
      line.trim().startsWith('-') || line.trim().startsWith('•')
    );
    return lines.slice(0, 5).map(line => line.replace(/^[-•]\s*/, ''));
  };

  return (
    <main className="container mx-auto p-8">
      <h1 className="text-4xl font-bold mb-8">AI Content Pipeline</h1>
      
      <div className="space-y-4 mb-8">
        <input
          type="text"
          placeholder="Enter keyword..."
          value={keyword}
          onChange={(e) => setKeyword(e.target.value)}
          className="w-full p-3 border rounded"
        />
        
        <select
          value={format}
          onChange={(e) => setFormat(e.target.value as any)}
          className="w-full p-3 border rounded"
        >
          <option value="toplist">Top List</option>
          <option value="pov">Point of View</option>
          <option value="case-study">Case Study</option>
          <option value="how-to">How-To Guide</option>
        </select>
        
        <button
          onClick={handleGenerate}
          disabled={loading || !keyword}
          className="w-full bg-blue-600 text-white p-3 rounded disabled:opacity-50"
        >
          {loading ? 'Generating...' : 'Generate Content & Video'}
        </button>
      </div>

      {content && (
        <div className="mb-8">
          <h2 className="text-2xl font-bold mb-4">Generated Content</h2>
          <div className="prose max-w-none bg-gray-50 p-6 rounded">
            {content}
          </div>
        </div>
      )}

      {videoUrl && (
        <div>
          <h2 className="text-2xl font-bold mb-4">Generated Video</h2>
          <video controls className="w-full max-w-md">
            <source src={videoUrl} type="video/mp4" />
          </video>
        </div>
      )}
    </main>
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
npm run start

# Render a specific Remotion video
npx remotion render ArticleVideo output.mp4
```

## Common Patterns

### Multi-language Content Generation

```typescript
async function generateBilingualContent(keyword: string) {
  const researchData = await crawlNewsAPI(keyword);
  
  const [englishContent, vietnameseContent] = await Promise.all([
    generateContentWithClaude({
      keyword,
      format: 'toplist',
      tone: 'expert',
      language: 'en',
      researchData
    }),
    generateContentWithClaude({
      keyword,
      format: 'toplist',
      tone: 'expert',
      language: 'vi',
      researchData
    })
  ]);

  return { english: englishContent, vietnamese: vietnameseContent };
}
```

### Batch Processing

```typescript
async function batchGenerateContent(keywords: string[]) {
  const results = await Promise.allSettled(
    keywords.map(async (keyword) => {
      const researchData = await crawlNewsAPI(keyword);
      const content = await generateContentWithClaude({
        keyword,
        format: 'toplist',
        tone: 'expert',
        language: 'en',
        researchData
      });
      return { keyword, content };
    })
  );

  return results
    .filter(r => r.status === 'fulfilled')
    .map(r => (r as PromiseFulfilledResult<any>).value);
}
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement retry logic with exponential backoff
async function retryWithBackoff<T>(
  fn: () => Promise<T>,
  maxRetries = 3
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error: any) {
      if (error?.status === 429 && i < maxRetries - 1) {
        const waitTime = Math.pow(2, i) * 1000;
        await new Promise(resolve => setTimeout(resolve, waitTime));
        continue;
      }
      throw error;
    }
  }
  throw new Error('Max retries exceeded');
}
```

### Video Rendering Performance

- Use `--concurrency` flag to control parallel rendering
- Optimize composition complexity for faster rendering
- Consider using Remotion Lambda for cloud rendering at scale

### Environment Variables Not Loading

Ensure `.env` file is in the project root and restart the dev server after changes.

```bash
# Verify environment variables are loaded
npm run dev -- --debug
```
