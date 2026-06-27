---
name: marketing-pipeline-share-automation
description: Automate content creation from research to video generation using AI-powered content pipeline with Claude, OpenAI, and Remotion
triggers:
  - how do I generate automated content with marketing pipeline
  - set up AI content automation workflow
  - create videos from text using remotion and AI
  - crawl news and generate content automatically
  - build multi-language content with Claude API
  - automate social media content creation pipeline
  - generate infographics and videos from articles
  - research and write content with AI tools
---

# Marketing Pipeline Share - AI Content Automation Skill

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

Marketing Pipeline Share is a comprehensive content automation system that transforms a single keyword into fully-formatted articles and videos. The pipeline handles:

1. **Research Phase**: Auto-crawls recent news from TechCrunch, a16z, Twitter/X, LinkedIn
2. **Content Generation**: Uses Claude 3/OpenAI to write articles in multiple formats (toplist, POV, case study, how-to)
3. **Multi-language Output**: Generates both English and Vietnamese content
4. **Video Generation**: Uses Remotion to create videos, infographics, and social media content

Built with TypeScript, Next.js, and integrated with Claude API, OpenAI API, and RapidAPI.

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

## Configuration

Create a `.env.local` file in the project root:

```bash
# AI Model APIs
ANTHROPIC_API_KEY=your_anthropic_key_here
OPENAI_API_KEY=your_openai_key_here

# RapidAPI for news crawling
RAPIDAPI_KEY=your_rapidapi_key_here

# Next.js Config
NEXT_PUBLIC_API_URL=http://localhost:3000

# Remotion Config (optional)
REMOTION_AWS_ACCESS_KEY_ID=your_aws_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret
```

Start the development server:

```bash
npm run dev
```

Access the application at `http://localhost:3000`

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app router pages
│   ├── components/       # React components
│   ├── lib/             # Core utilities
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── crawler/     # News crawling logic
│   │   └── video/       # Remotion video generation
│   └── types/           # TypeScript definitions
├── public/              # Static assets
└── remotion/            # Remotion video templates
```

## Core API Usage

### 1. Research & Content Crawling

```typescript
// lib/crawler/newsResearch.ts
import axios from 'axios';

interface NewsSource {
  title: string;
  url: string;
  publishedAt: string;
  summary: string;
}

export async function crawlRecentNews(
  keyword: string,
  sources: string[] = ['techcrunch', 'a16z']
): Promise<NewsSource[]> {
  const results: NewsSource[] = [];
  
  for (const source of sources) {
    try {
      const response = await axios.get(`https://api.rapidapi.com/news/search`, {
        params: {
          q: keyword,
          source: source,
          from: new Date(Date.now() - 24 * 60 * 60 * 1000).toISOString()
        },
        headers: {
          'X-RapidAPI-Key': process.env.RAPIDAPI_KEY!,
          'X-RapidAPI-Host': 'news-api.rapidapi.com'
        }
      });
      
      results.push(...response.data.articles);
    } catch (error) {
      console.error(`Failed to crawl ${source}:`, error);
    }
  }
  
  return results;
}
```

### 2. AI Content Generation with Claude

```typescript
// lib/ai/claudeGenerator.ts
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

interface ContentRequest {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  research: string[];
}

export async function generateContent(request: ContentRequest): Promise<string> {
  const prompt = buildPrompt(request);
  
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [
      {
        role: 'user',
        content: prompt
      }
    ],
  });
  
  return message.content[0].type === 'text' 
    ? message.content[0].text 
    : '';
}

function buildPrompt(request: ContentRequest): string {
  const formatInstructions = {
    'toplist': 'Create a numbered list article with clear rankings',
    'pov': 'Write from a unique perspective or opinion',
    'case-study': 'Analyze real examples with data and insights',
    'how-to': 'Create step-by-step instructional content'
  };
  
  return `
You are a ${request.tone} content writer specializing in ${request.keyword}.

Format: ${formatInstructions[request.format]}
Language: ${request.language === 'en' ? 'English' : 'Vietnamese'}

Recent Research Data:
${request.research.join('\n\n')}

Write a comprehensive article based on the above research. Include:
- Engaging headline
- Data-backed insights
- Actionable takeaways
- SEO-optimized structure
`;
}
```

### 3. OpenAI Alternative Integration

```typescript
// lib/ai/openaiGenerator.ts
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

export async function generateWithOpenAI(
  prompt: string,
  model: string = 'gpt-4-turbo-preview'
): Promise<string> {
  const completion = await openai.chat.completions.create({
    model: model,
    messages: [
      {
        role: 'system',
        content: 'You are an expert content marketing writer.'
      },
      {
        role: 'user',
        content: prompt
      }
    ],
    temperature: 0.7,
    max_tokens: 3000,
  });
  
  return completion.choices[0].message.content || '';
}
```

### 4. Video Generation with Remotion

```typescript
// lib/video/renderVideo.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoContent {
  title: string;
  highlights: string[];
  stats: { label: string; value: string }[];
}

export async function generateContentVideo(
  content: VideoContent,
  format: 'reels' | 'tiktok' | 'shorts' = 'reels'
): Promise<string> {
  const compositionId = 'ContentVideo';
  const bundleLocation = await bundle(
    path.join(process.cwd(), 'remotion/index.ts')
  );
  
  const compositions = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
  });
  
  const outputLocation = path.join(
    process.cwd(), 
    'public/videos',
    `${Date.now()}.mp4`
  );
  
  await renderMedia({
    composition: compositions,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: {
      content,
      format,
    },
  });
  
  return outputLocation;
}
```

### 5. Complete Pipeline Orchestration

```typescript
// lib/pipeline/contentPipeline.ts
import { crawlRecentNews } from '../crawler/newsResearch';
import { generateContent } from '../ai/claudeGenerator';
import { generateContentVideo } from '../video/renderVideo';

interface PipelineConfig {
  keyword: string;
  contentFormat: 'toplist' | 'pov' | 'case-study' | 'how-to';
  languages: ('en' | 'vi')[];
  generateVideo: boolean;
  sources?: string[];
}

export async function runContentPipeline(config: PipelineConfig) {
  // Step 1: Research
  console.log('🔍 Starting research phase...');
  const newsData = await crawlRecentNews(config.keyword, config.sources);
  const researchSummaries = newsData.map(n => `${n.title}: ${n.summary}`);
  
  // Step 2: Generate content for each language
  console.log('✍️ Generating content...');
  const contents = await Promise.all(
    config.languages.map(lang => 
      generateContent({
        keyword: config.keyword,
        format: config.contentFormat,
        language: lang,
        tone: 'expert',
        research: researchSummaries,
      })
    )
  );
  
  const result = {
    research: newsData,
    articles: config.languages.reduce((acc, lang, i) => {
      acc[lang] = contents[i];
      return acc;
    }, {} as Record<string, string>),
    videos: [] as string[],
  };
  
  // Step 3: Generate videos (optional)
  if (config.generateVideo) {
    console.log('🎬 Rendering videos...');
    const videoContent = extractVideoContent(contents[0]);
    
    const videoFormats: Array<'reels' | 'tiktok' | 'shorts'> = ['reels', 'tiktok', 'shorts'];
    result.videos = await Promise.all(
      videoFormats.map(format => generateContentVideo(videoContent, format))
    );
  }
  
  console.log('✅ Pipeline complete!');
  return result;
}

function extractVideoContent(article: string): VideoContent {
  // Simple extraction logic - can be enhanced with AI
  const lines = article.split('\n').filter(l => l.trim());
  return {
    title: lines[0] || 'Content Video',
    highlights: lines.slice(1, 4),
    stats: [],
  };
}
```

## API Routes (Next.js)

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline/contentPipeline';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    
    const result = await runContentPipeline({
      keyword: body.keyword,
      contentFormat: body.format || 'how-to',
      languages: body.languages || ['en', 'vi'],
      generateVideo: body.generateVideo ?? true,
      sources: body.sources,
    });
    
    return NextResponse.json({
      success: true,
      data: result,
    });
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { success: false, error: 'Pipeline failed' },
      { status: 500 }
    );
  }
}
```

## Frontend Usage Example

```typescript
// components/ContentGenerator.tsx
'use client';

import { useState } from 'react';

export default function ContentGenerator() {
  const [keyword, setKeyword] = useState('');
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState<any>(null);
  
  const handleGenerate = async () => {
    setLoading(true);
    
    try {
      const response = await fetch('/api/generate', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          keyword,
          format: 'toplist',
          languages: ['en', 'vi'],
          generateVideo: true,
        }),
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
    <div className="p-8">
      <input
        type="text"
        value={keyword}
        onChange={(e) => setKeyword(e.target.value)}
        placeholder="Enter keyword..."
        className="border p-2 rounded w-full"
      />
      
      <button
        onClick={handleGenerate}
        disabled={loading}
        className="mt-4 bg-blue-500 text-white px-6 py-2 rounded"
      >
        {loading ? 'Generating...' : 'Generate Content'}
      </button>
      
      {result && (
        <div className="mt-8">
          <h2 className="text-2xl font-bold">Results</h2>
          <div className="mt-4">
            <h3>English Article:</h3>
            <div className="whitespace-pre-wrap">{result.articles.en}</div>
          </div>
          <div className="mt-4">
            <h3>Vietnamese Article:</h3>
            <div className="whitespace-pre-wrap">{result.articles.vi}</div>
          </div>
          {result.videos.length > 0 && (
            <div className="mt-4">
              <h3>Generated Videos:</h3>
              {result.videos.map((v: string, i: number) => (
                <video key={i} src={v} controls className="mt-2" />
              ))}
            </div>
          )}
        </div>
      )}
    </div>
  );
}
```

## Common Patterns

### Batch Processing Multiple Keywords

```typescript
async function batchGenerate(keywords: string[]) {
  const results = [];
  
  for (const keyword of keywords) {
    const result = await runContentPipeline({
      keyword,
      contentFormat: 'toplist',
      languages: ['en'],
      generateVideo: false,
    });
    
    results.push({ keyword, ...result });
    
    // Rate limiting
    await new Promise(resolve => setTimeout(resolve, 2000));
  }
  
  return results;
}
```

### Custom Video Templates

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, interpolate } from 'remotion';

export const ContentVideo: React.FC<{
  content: VideoContent;
  format: string;
}> = ({ content, format }) => {
  const frame = useCurrentFrame();
  const opacity = interpolate(frame, [0, 30], [0, 1]);
  
  return (
    <AbsoluteFill style={{ backgroundColor: '#000' }}>
      <div style={{ opacity, color: 'white', padding: 40 }}>
        <h1>{content.title}</h1>
        <ul>
          {content.highlights.map((h, i) => (
            <li key={i}>{h}</li>
          ))}
        </ul>
      </div>
    </AbsoluteFill>
  );
};
```

## Troubleshooting

### API Rate Limits

If you encounter rate limiting errors:

```typescript
// Add exponential backoff
async function retryWithBackoff<T>(
  fn: () => Promise<T>,
  maxRetries: number = 3
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error: any) {
      if (i === maxRetries - 1) throw error;
      if (error.status === 429) {
        await new Promise(resolve => 
          setTimeout(resolve, Math.pow(2, i) * 1000)
        );
      } else {
        throw error;
      }
    }
  }
  throw new Error('Max retries exceeded');
}
```

### Video Rendering Issues

Ensure Remotion dependencies are properly installed:

```bash
npm install @remotion/bundler @remotion/renderer @remotion/cli
```

For server-side rendering, you may need Chrome/Chromium:

```bash
# Ubuntu/Debian
sudo apt-get install chromium-browser

# macOS
brew install chromium
```

### Memory Issues with Large Content

Process content in chunks:

```typescript
async function generateLargeContent(sections: string[]) {
  const results = [];
  
  for (const section of sections) {
    const content = await generateContent({
      keyword: section,
      format: 'how-to',
      language: 'en',
      tone: 'expert',
      research: [],
    });
    
    results.push(content);
    
    // Clear memory between generations
    if (global.gc) global.gc();
  }
  
  return results.join('\n\n');
}
```
