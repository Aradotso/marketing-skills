---
name: ultimate-ai-content-pipeline
description: Automated content creation pipeline from research to video generation using Claude/OpenAI APIs
triggers:
  - "set up the AI content pipeline"
  - "create automated content workflow"
  - "generate content with research and video"
  - "configure the marketing pipeline"
  - "help me use the content automation system"
  - "build AI-powered content creation"
  - "automate content from research to video"
  - "set up remotion video generation for content"
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to help developers use the Ultimate AI Content Pipeline — a comprehensive TypeScript-based system that automates content creation from research gathering, scriptwriting, to video generation using Claude 3/OpenAI and Remotion.

## What This Project Does

The Ultimate AI Content Pipeline is an all-in-one content automation system that:

1. **Auto-Research**: Crawls recent news from TechCrunch, a16z, Twitter/X, LinkedIn
2. **AI Content Generation**: Creates content in multiple formats (toplist, POV, case study, how-to) using Claude/OpenAI
3. **Multi-language Support**: Generates parallel Vietnamese and English versions
4. **Video Rendering**: Automatically creates infographics and short videos using Remotion
5. **Platform Optimization**: Exports videos optimized for Reels, TikTok, Shorts

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
# AI APIs
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# Research/Crawling APIs
RAPIDAPI_KEY=your_rapidapi_key

# Optional: Database
DATABASE_URL=your_database_connection_string

# Optional: Storage for videos
STORAGE_BUCKET=your_storage_bucket
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app router
│   ├── components/       # React components
│   ├── lib/              # Core libraries
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── research/    # Research/crawling modules
│   │   └── video/       # Remotion video generation
│   ├── api/             # API routes
│   └── utils/           # Utility functions
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Core API Integration

### OpenAI Integration

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateContent(topic: string, format: string) {
  const completion = await openai.chat.completions.create({
    model: "gpt-4-turbo-preview",
    messages: [
      {
        role: "system",
        content: `You are a content creator specializing in ${format} format.`
      },
      {
        role: "user",
        content: `Create a ${format} article about: ${topic}`
      }
    ],
    temperature: 0.7,
    max_tokens: 2000
  });

  return completion.choices[0].message.content;
}
```

### Claude (Anthropic) Integration

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

async function generateWithClaude(
  topic: string, 
  researchData: string,
  language: 'en' | 'vi'
) {
  const message = await anthropic.messages.create({
    model: "claude-3-opus-20240229",
    max_tokens: 4096,
    messages: [
      {
        role: "user",
        content: `Based on this research data:\n${researchData}\n\nCreate a comprehensive article about ${topic} in ${language === 'vi' ? 'Vietnamese' : 'English'}.`
      }
    ]
  });

  return message.content[0].text;
}
```

## Research Module

### Auto-Crawling News Sources

```typescript
interface ResearchSource {
  name: string;
  url: string;
  selector: string;
}

async function crawlNewsSources(keyword: string): Promise<Article[]> {
  const sources: ResearchSource[] = [
    {
      name: 'TechCrunch',
      url: `https://techcrunch.com/search/${keyword}`,
      selector: '.post-block'
    },
    // Add more sources
  ];

  const articles: Article[] = [];

  for (const source of sources) {
    try {
      const response = await fetch(source.url, {
        headers: {
          'X-RapidAPI-Key': process.env.RAPIDAPI_KEY || '',
        }
      });
      
      const data = await response.json();
      // Parse and extract articles
      articles.push(...parseArticles(data, source.name));
    } catch (error) {
      console.error(`Failed to crawl ${source.name}:`, error);
    }
  }

  return articles;
}

interface Article {
  title: string;
  url: string;
  excerpt: string;
  publishedAt: Date;
  source: string;
}

function parseArticles(data: any, sourceName: string): Article[] {
  // Implementation depends on API response structure
  return data.articles?.map((item: any) => ({
    title: item.title,
    url: item.url,
    excerpt: item.description,
    publishedAt: new Date(item.publishedAt),
    source: sourceName
  })) || [];
}
```

## Content Generation Pipeline

### Complete Workflow

```typescript
interface ContentRequest {
  topic: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi' | 'both';
  includeVideo: boolean;
}

async function runContentPipeline(request: ContentRequest) {
  // Step 1: Research
  console.log('🔍 Starting research phase...');
  const researchData = await crawlNewsSources(request.topic);
  const insights = await extractInsights(researchData);

  // Step 2: Generate content
  console.log('✍️ Generating content...');
  const content = await generateContent(
    request.topic,
    request.format,
    insights,
    request.language
  );

  // Step 3: Create video (if requested)
  let videoUrl = null;
  if (request.includeVideo) {
    console.log('🎬 Rendering video...');
    videoUrl = await renderVideo(content);
  }

  return {
    content,
    videoUrl,
    metadata: {
      topic: request.topic,
      format: request.format,
      createdAt: new Date(),
      sources: researchData.length
    }
  };
}

async function extractInsights(articles: Article[]): Promise<string> {
  const summaries = articles.map(a => `${a.title}: ${a.excerpt}`).join('\n\n');
  
  const completion = await openai.chat.completions.create({
    model: "gpt-4-turbo-preview",
    messages: [
      {
        role: "system",
        content: "Extract key insights and data points from these articles."
      },
      {
        role: "user",
        content: summaries
      }
    ]
  });

  return completion.choices[0].message.content || '';
}
```

## Remotion Video Generation

### Video Template Setup

```typescript
// remotion/ContentVideo.tsx
import { Composition } from 'remotion';
import { VideoTemplate } from './templates/VideoTemplate';

export const RemotionRoot: React.FC = () => {
  return (
    <>
      <Composition
        id="ContentVideo"
        component={VideoTemplate}
        durationInFrames={300}
        fps={30}
        width={1080}
        height={1920}
        defaultProps={{
          title: '',
          points: [],
          theme: 'modern'
        }}
      />
    </>
  );
};
```

### Render Video from Content

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoContent {
  title: string;
  points: string[];
  theme?: 'modern' | 'professional' | 'playful';
}

async function renderVideo(content: VideoContent): Promise<string> {
  const bundleLocation = await bundle(
    path.join(process.cwd(), 'remotion/index.ts')
  );

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: content,
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
  });

  return outputLocation;
}
```

## API Routes (Next.js)

### Generate Content Endpoint

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';

export async function POST(req: NextRequest) {
  try {
    const body = await req.json();
    const { topic, format, language, includeVideo } = body;

    if (!topic || !format) {
      return NextResponse.json(
        { error: 'Topic and format are required' },
        { status: 400 }
      );
    }

    const result = await runContentPipeline({
      topic,
      format,
      language: language || 'both',
      includeVideo: includeVideo || false,
    });

    return NextResponse.json(result);
  } catch (error) {
    console.error('Generation error:', error);
    return NextResponse.json(
      { error: 'Failed to generate content' },
      { status: 500 }
    );
  }
}
```

### Research Endpoint

```typescript
// src/app/api/research/route.ts
export async function GET(req: NextRequest) {
  const { searchParams } = new URL(req.url);
  const keyword = searchParams.get('keyword');

  if (!keyword) {
    return NextResponse.json(
      { error: 'Keyword is required' },
      { status: 400 }
    );
  }

  const articles = await crawlNewsSources(keyword);
  
  return NextResponse.json({
    keyword,
    count: articles.length,
    articles: articles.slice(0, 10), // Return top 10
  });
}
```

## Frontend Component Example

```typescript
// src/components/ContentGenerator.tsx
'use client';

import { useState } from 'react';

interface GenerateResult {
  content: {
    en?: string;
    vi?: string;
  };
  videoUrl?: string;
  metadata: any;
}

export default function ContentGenerator() {
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState<GenerateResult | null>(null);

  const handleGenerate = async (e: React.FormEvent<HTMLFormElement>) => {
    e.preventDefault();
    setLoading(true);

    const formData = new FormData(e.currentTarget);
    const payload = {
      topic: formData.get('topic'),
      format: formData.get('format'),
      language: formData.get('language'),
      includeVideo: formData.get('includeVideo') === 'on',
    };

    try {
      const response = await fetch('/api/generate', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(payload),
      });

      const data = await response.json();
      setResult(data);
    } catch (error) {
      console.error('Generation failed:', error);
      alert('Failed to generate content');
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="max-w-4xl mx-auto p-6">
      <form onSubmit={handleGenerate} className="space-y-4">
        <input
          name="topic"
          placeholder="Enter topic or keyword"
          className="w-full p-3 border rounded"
          required
        />
        
        <select name="format" className="w-full p-3 border rounded" required>
          <option value="toplist">Top List</option>
          <option value="pov">POV / Opinion</option>
          <option value="case-study">Case Study</option>
          <option value="how-to">How-to Guide</option>
        </select>

        <select name="language" className="w-full p-3 border rounded">
          <option value="both">Both (EN + VI)</option>
          <option value="en">English only</option>
          <option value="vi">Vietnamese only</option>
        </select>

        <label className="flex items-center gap-2">
          <input type="checkbox" name="includeVideo" />
          Generate video
        </label>

        <button
          type="submit"
          disabled={loading}
          className="w-full bg-blue-600 text-white p-3 rounded disabled:opacity-50"
        >
          {loading ? 'Generating...' : 'Generate Content'}
        </button>
      </form>

      {result && (
        <div className="mt-8 space-y-4">
          <h2 className="text-2xl font-bold">Generated Content</h2>
          {result.content.en && (
            <div className="p-4 bg-gray-50 rounded">
              <h3 className="font-semibold mb-2">English Version</h3>
              <pre className="whitespace-pre-wrap">{result.content.en}</pre>
            </div>
          )}
          {result.content.vi && (
            <div className="p-4 bg-gray-50 rounded">
              <h3 className="font-semibold mb-2">Vietnamese Version</h3>
              <pre className="whitespace-pre-wrap">{result.content.vi}</pre>
            </div>
          )}
          {result.videoUrl && (
            <div>
              <h3 className="font-semibold mb-2">Generated Video</h3>
              <video src={result.videoUrl} controls className="w-full" />
            </div>
          )}
        </div>
      )}
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

# Render Remotion video only
npm run remotion
```

## Common Patterns

### Batch Content Generation

```typescript
async function batchGenerate(topics: string[]) {
  const results = await Promise.all(
    topics.map(topic =>
      runContentPipeline({
        topic,
        format: 'toplist',
        language: 'both',
        includeVideo: true,
      })
    )
  );

  return results;
}
```

### Scheduled Content Generation

```typescript
import cron from 'node-cron';

// Run every day at 9 AM
cron.schedule('0 9 * * *', async () => {
  console.log('Starting scheduled content generation...');
  
  const trendingTopics = await fetchTrendingTopics();
  await batchGenerate(trendingTopics);
});
```

### Custom Content Tone

```typescript
async function generateWithTone(
  topic: string,
  tone: 'expert' | 'friendly' | 'humorous'
) {
  const tonePrompts = {
    expert: 'Write in an authoritative, data-driven style with industry jargon.',
    friendly: 'Write in a conversational, approachable tone.',
    humorous: 'Write with wit and humor, making complex topics entertaining.'
  };

  const completion = await openai.chat.completions.create({
    model: "gpt-4-turbo-preview",
    messages: [
      {
        role: "system",
        content: tonePrompts[tone]
      },
      {
        role: "user",
        content: `Write about: ${topic}`
      }
    ]
  });

  return completion.choices[0].message.content;
}
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement retry logic with exponential backoff
async function apiCallWithRetry<T>(
  fn: () => Promise<T>,
  maxRetries = 3
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error: any) {
      if (error.status === 429 && i < maxRetries - 1) {
        const delay = Math.pow(2, i) * 1000;
        console.log(`Rate limited. Retrying in ${delay}ms...`);
        await new Promise(resolve => setTimeout(resolve, delay));
      } else {
        throw error;
      }
    }
  }
  throw new Error('Max retries exceeded');
}
```

### Video Rendering Issues

```typescript
// Check Remotion dependencies
// Ensure ffmpeg is installed: brew install ffmpeg (macOS) or apt-get install ffmpeg (Linux)

// Debug video rendering
import { getCompositions } from '@remotion/renderer';

async function debugVideoSetup() {
  const bundleLocation = await bundle(
    path.join(process.cwd(), 'remotion/index.ts')
  );
  
  const compositions = await getCompositions(bundleLocation);
  console.log('Available compositions:', compositions);
}
```

### Memory Issues with Large Content

```typescript
// Stream large responses
async function generateLargeContent(topic: string) {
  const stream = await openai.chat.completions.create({
    model: "gpt-4-turbo-preview",
    messages: [{ role: "user", content: topic }],
    stream: true,
  });

  let fullContent = '';
  for await (const chunk of stream) {
    fullContent += chunk.choices[0]?.delta?.content || '';
  }

  return fullContent;
}
```

## Best Practices

1. **Always use environment variables** for API keys
2. **Implement error handling** for all AI API calls
3. **Cache research results** to avoid redundant crawling
4. **Rate limit your requests** to avoid hitting API limits
5. **Validate user input** before sending to AI APIs
6. **Store generated content** in a database for reuse
7. **Monitor API costs** especially for video generation
8. **Use TypeScript types** for all data structures
