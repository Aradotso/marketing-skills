---
name: marketing-pipeline-share-content-automation
description: Automate content creation from research to video generation using AI-powered pipeline with Claude, OpenAI, and Remotion
triggers:
  - "automate content creation with AI pipeline"
  - "generate video content from research automatically"
  - "set up marketing content automation workflow"
  - "create AI-powered content from trending news"
  - "build automated content pipeline with Claude and OpenAI"
  - "generate social media videos from blog posts"
  - "scrape news and create content automatically"
  - "set up Remotion video rendering for marketing"
---

# Marketing Pipeline Share - Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

Complete AI-powered content automation system that handles the entire pipeline: from researching trending news, generating multi-format content (blog posts, scripts), to rendering videos automatically. Built with TypeScript, Next.js, Claude/OpenAI APIs, and Remotion.

## What It Does

This system automates the complete content creation workflow:

1. **Auto-Research**: Scrapes recent articles from TechCrunch, a16z, X (Twitter), LinkedIn
2. **AI Content Generation**: Creates content in multiple formats (toplist, POV, case study, how-to) using Claude/OpenAI
3. **Multi-language Support**: Generates both English and Vietnamese versions
4. **Video Rendering**: Automatically renders videos and infographics using Remotion
5. **Platform Optimization**: Outputs content optimized for Reels, TikTok, Shorts

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
cp .env.example .env.local
```

## Environment Configuration

Create a `.env.local` file with the following variables:

```bash
# AI Providers
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_url

# Remotion (for video rendering)
REMOTION_LICENSE_KEY=your_remotion_license_key

# Application
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app router pages
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── scraper/     # News scraping logic
│   │   ├── content/     # Content generation
│   │   └── video/       # Remotion video rendering
│   ├── types/           # TypeScript types
│   └── utils/           # Utility functions
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Core Usage Patterns

### 1. Research & Scraping

```typescript
import { scrapeNews } from '@/lib/scraper/news-scraper';

// Scrape recent news for a topic
async function gatherResearch(topic: string) {
  const articles = await scrapeNews({
    query: topic,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeframe: '24h',
    limit: 20
  });
  
  return articles.map(article => ({
    title: article.title,
    url: article.url,
    summary: article.summary,
    publishedAt: article.publishedAt
  }));
}
```

### 2. AI Content Generation with Claude

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

async function generateContentWithClaude(
  topic: string,
  research: any[],
  format: 'toplist' | 'pov' | 'case-study' | 'how-to'
) {
  const prompt = `
You are a content marketing expert. Based on the following research data, 
create a ${format} article about "${topic}".

Research Data:
${JSON.stringify(research, null, 2)}

Requirements:
- Make it engaging and data-backed
- Include specific examples and insights
- Optimize for SEO
- Target audience: marketers and content creators
`;

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4000,
    messages: [{
      role: 'user',
      content: prompt
    }]
  });

  return message.content[0].text;
}
```

### 3. AI Content Generation with OpenAI

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateContentWithOpenAI(
  topic: string,
  research: any[],
  language: 'en' | 'vi'
) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are a content marketing expert writing in ${language === 'vi' ? 'Vietnamese' : 'English'}.`
      },
      {
        role: 'user',
        content: `Create engaging content about "${topic}" based on this research: ${JSON.stringify(research)}`
      }
    ],
    temperature: 0.7,
    max_tokens: 3000
  });

  return completion.choices[0].message.content;
}
```

### 4. Complete Content Pipeline

```typescript
import { scrapeNews } from '@/lib/scraper/news-scraper';
import { generateContentWithClaude } from '@/lib/ai/claude';
import { generateContentWithOpenAI } from '@/lib/ai/openai';
import { renderVideo } from '@/lib/video/remotion-render';

async function runContentPipeline(
  topic: string,
  format: 'toplist' | 'pov' | 'case-study' | 'how-to'
) {
  // Step 1: Research
  console.log('🔍 Gathering research...');
  const research = await scrapeNews({
    query: topic,
    sources: ['techcrunch', 'a16z'],
    timeframe: '24h',
    limit: 15
  });

  // Step 2: Generate English content
  console.log('✍️ Generating English content...');
  const englishContent = await generateContentWithClaude(
    topic,
    research,
    format
  );

  // Step 3: Generate Vietnamese content
  console.log('✍️ Generating Vietnamese content...');
  const vietnameseContent = await generateContentWithOpenAI(
    topic,
    research,
    'vi'
  );

  // Step 4: Render video
  console.log('🎬 Rendering video...');
  const videoUrl = await renderVideo({
    title: topic,
    content: englishContent,
    format: 'shorts', // reels, tiktok, shorts
    aspectRatio: '9:16'
  });

  return {
    english: englishContent,
    vietnamese: vietnameseContent,
    video: videoUrl,
    research: research
  };
}

// Usage
const result = await runContentPipeline(
  'AI Marketing Trends 2024',
  'toplist'
);
```

### 5. Remotion Video Rendering

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

async function renderVideo(config: {
  title: string;
  content: string;
  format: 'reels' | 'tiktok' | 'shorts';
  aspectRatio: string;
}) {
  const compositionId = 'ContentVideo';
  const bundleLocation = await bundle(
    path.resolve('./remotion/index.ts')
  );

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
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
    inputProps: {
      title: config.title,
      content: config.content,
      format: config.format
    },
  });

  return outputLocation;
}
```

### 6. Next.js API Route Example

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/content/pipeline';

export async function POST(request: NextRequest) {
  try {
    const { topic, format } = await request.json();

    if (!topic || !format) {
      return NextResponse.json(
        { error: 'Topic and format are required' },
        { status: 400 }
      );
    }

    const result = await runContentPipeline(topic, format);

    return NextResponse.json({
      success: true,
      data: result
    });
  } catch (error) {
    console.error('Content generation error:', error);
    return NextResponse.json(
      { error: 'Failed to generate content' },
      { status: 500 }
    );
  }
}
```

### 7. React Component for Content Generation

```typescript
'use client';

import { useState } from 'react';

export default function ContentGenerator() {
  const [topic, setTopic] = useState('');
  const [format, setFormat] = useState<'toplist' | 'pov' | 'case-study' | 'how-to'>('toplist');
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState<any>(null);

  const generateContent = async () => {
    setLoading(true);
    try {
      const response = await fetch('/api/generate', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ topic, format })
      });

      const data = await response.json();
      setResult(data.data);
    } catch (error) {
      console.error('Error:', error);
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="p-6">
      <input
        type="text"
        value={topic}
        onChange={(e) => setTopic(e.target.value)}
        placeholder="Enter topic..."
        className="border p-2 w-full mb-4"
      />
      
      <select
        value={format}
        onChange={(e) => setFormat(e.target.value as any)}
        className="border p-2 mb-4"
      >
        <option value="toplist">Top List</option>
        <option value="pov">Point of View</option>
        <option value="case-study">Case Study</option>
        <option value="how-to">How To</option>
      </select>

      <button
        onClick={generateContent}
        disabled={loading}
        className="bg-blue-500 text-white px-4 py-2 rounded"
      >
        {loading ? 'Generating...' : 'Generate Content'}
      </button>

      {result && (
        <div className="mt-6">
          <h3 className="font-bold mb-2">English Version:</h3>
          <div className="mb-4 whitespace-pre-wrap">{result.english}</div>
          
          <h3 className="font-bold mb-2">Vietnamese Version:</h3>
          <div className="mb-4 whitespace-pre-wrap">{result.vietnamese}</div>
          
          {result.video && (
            <div>
              <h3 className="font-bold mb-2">Video:</h3>
              <video src={result.video} controls className="w-full max-w-md" />
            </div>
          )}
        </div>
      )}
    </div>
  );
}
```

## Development Workflow

```bash
# Run development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Render video (if using Remotion CLI)
npx remotion render ContentVideo output.mp4
```

## Common Patterns

### Batch Content Generation

```typescript
async function batchGenerateContent(topics: string[]) {
  const results = await Promise.all(
    topics.map(topic => 
      runContentPipeline(topic, 'toplist')
    )
  );
  
  return results;
}

// Usage
const topics = [
  'AI Marketing Tools 2024',
  'Content Automation Best Practices',
  'Video Marketing Trends'
];

const allContent = await batchGenerateContent(topics);
```

### Content Scheduling

```typescript
interface ScheduledContent {
  topic: string;
  format: string;
  scheduledFor: Date;
  platforms: ('facebook' | 'twitter' | 'linkedin')[];
}

async function scheduleContent(config: ScheduledContent) {
  const content = await runContentPipeline(config.topic, config.format as any);
  
  // Store in database with schedule time
  await db.scheduledPosts.create({
    data: {
      content: content.english,
      videoUrl: content.video,
      scheduledFor: config.scheduledFor,
      platforms: config.platforms,
      status: 'pending'
    }
  });
  
  return content;
}
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement retry logic with exponential backoff
async function withRetry<T>(
  fn: () => Promise<T>,
  maxRetries = 3,
  delay = 1000
): Promise<T> {
  try {
    return await fn();
  } catch (error: any) {
    if (maxRetries === 0 || !error.message?.includes('rate limit')) {
      throw error;
    }
    
    await new Promise(resolve => setTimeout(resolve, delay));
    return withRetry(fn, maxRetries - 1, delay * 2);
  }
}

// Usage
const content = await withRetry(() => 
  generateContentWithClaude(topic, research, 'toplist')
);
```

### Video Rendering Timeout

```typescript
// Increase timeout for complex videos
await renderMedia({
  composition,
  serveUrl: bundleLocation,
  codec: 'h264',
  outputLocation,
  inputProps: config,
  timeoutInMilliseconds: 120000, // 2 minutes
  chromiumOptions: {
    headless: true,
    gl: 'swiftshader'
  }
});
```

### Memory Issues with Large Content

```typescript
// Process in chunks
async function processLargeResearch(articles: any[]) {
  const chunkSize = 10;
  const results = [];
  
  for (let i = 0; i < articles.length; i += chunkSize) {
    const chunk = articles.slice(i, i + chunkSize);
    const processed = await generateContentWithClaude(
      'Topic',
      chunk,
      'toplist'
    );
    results.push(processed);
  }
  
  return results.join('\n\n');
}
```

### Environment Variable Validation

```typescript
function validateEnv() {
  const required = [
    'ANTHROPIC_API_KEY',
    'OPENAI_API_KEY',
    'RAPIDAPI_KEY'
  ];
  
  const missing = required.filter(key => !process.env[key]);
  
  if (missing.length > 0) {
    throw new Error(
      `Missing required environment variables: ${missing.join(', ')}`
    );
  }
}

// Call at app startup
validateEnv();
```

## Best Practices

1. **Rate Limiting**: Always implement retry logic for API calls
2. **Caching**: Cache research results to avoid repeated scraping
3. **Error Handling**: Wrap AI calls in try-catch blocks
4. **Cost Monitoring**: Track API usage to manage costs
5. **Content Validation**: Validate generated content before publishing
6. **Video Optimization**: Use appropriate video codecs and resolutions for target platforms
