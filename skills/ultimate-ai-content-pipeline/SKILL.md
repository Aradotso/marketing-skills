---
name: ultimate-ai-content-pipeline
description: Automated Vietnamese/English content creation pipeline with AI research, script generation, and video rendering using Claude, OpenAI, and Remotion
triggers:
  - "generate content with AI research and video"
  - "create automated marketing content pipeline"
  - "set up AI content generation with Remotion"
  - "build Vietnamese English content automation"
  - "crawl news and generate blog posts automatically"
  - "create TikTok Reels from AI written articles"
  - "automate content from research to video"
  - "set up Claude OpenAI content workflow"
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

An end-to-end TypeScript content automation system that crawls real-time news, generates multilingual content (Vietnamese/English), and renders videos automatically. Built with Next.js, Claude/OpenAI APIs, and Remotion.

## What It Does

This pipeline automates the entire content creation workflow:

1. **Auto-Research**: Crawls TechCrunch, a16z, Twitter/X, LinkedIn for latest news (last 24h)
2. **AI Writing**: Generates content in multiple formats (toplist, POV, case study, how-to) using Claude 3 or OpenAI
3. **Bilingual Output**: Produces both English and Vietnamese versions
4. **Video Generation**: Renders infographics and short-form videos via Remotion for Reels/TikTok/Shorts

## Installation

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
pnpm install
# or
yarn install
```

## Environment Configuration

Create a `.env.local` file in the project root:

```env
# AI Provider Keys (choose one or both)
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research/Crawling APIs
RAPIDAPI_KEY=your_rapidapi_key

# Database (if using)
DATABASE_URL=your_database_connection_string

# Remotion (for video rendering)
REMOTION_LICENSE_KEY=your_remotion_license

# Application
NEXT_PUBLIC_API_URL=http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app router pages
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── research/    # Web crawling & data extraction
│   │   ├── content/     # Content generation logic
│   │   └── video/       # Remotion video rendering
│   └── utils/           # Helper functions
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Core Usage Patterns

### 1. Research & Content Generation

```typescript
import { researchTopic } from '@/lib/research/crawler';
import { generateContent } from '@/lib/ai/content-generator';

// Research a topic from multiple sources
async function createContentPipeline(keyword: string) {
  // Step 1: Research
  const researchData = await researchTopic(keyword, {
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeRange: '24h',
    maxArticles: 10
  });

  // Step 2: Generate content
  const content = await generateContent({
    keyword,
    researchData,
    format: 'toplist', // or 'pov', 'case-study', 'how-to'
    languages: ['en', 'vi'],
    tone: 'expert', // or 'friendly', 'humorous'
    aiProvider: 'claude' // or 'openai'
  });

  return content;
}
```

### 2. AI Content Generation with Claude

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

async function generateWithClaude(prompt: string, context: string) {
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [{
      role: 'user',
      content: `${context}\n\n${prompt}`
    }],
    system: 'You are an expert content creator specializing in marketing and technical content. Generate engaging, data-backed articles.'
  });

  return message.content[0].text;
}

// Usage
const article = await generateWithClaude(
  'Write a toplist article about AI trends',
  JSON.stringify(researchData)
);
```

### 3. AI Content Generation with OpenAI

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateWithOpenAI(prompt: string, context: string) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: 'You are an expert content creator for marketing and tech content.'
      },
      {
        role: 'user',
        content: `Context: ${context}\n\nTask: ${prompt}`
      }
    ],
    temperature: 0.7,
    max_tokens: 3000
  });

  return completion.choices[0].message.content;
}
```

### 4. Web Crawling & Research

```typescript
import axios from 'axios';

// Example: Crawl TechCrunch for latest AI news
async function crawlTechCrunch(keyword: string) {
  const options = {
    method: 'GET',
    url: 'https://techcrunch-api.p.rapidapi.com/search',
    params: {
      query: keyword,
      limit: '10',
      date_range: '24h'
    },
    headers: {
      'X-RapidAPI-Key': process.env.RAPIDAPI_KEY,
      'X-RapidAPI-Host': 'techcrunch-api.p.rapidapi.com'
    }
  };

  try {
    const response = await axios.request(options);
    return response.data.articles;
  } catch (error) {
    console.error('Crawl error:', error);
    return [];
  }
}

// Extract insights from crawled data
function extractInsights(articles: any[]) {
  return articles.map(article => ({
    title: article.title,
    summary: article.description,
    url: article.url,
    publishedAt: article.publishedAt,
    relevanceScore: calculateRelevance(article)
  }));
}
```

### 5. Bilingual Content Generation

```typescript
interface BilingualContent {
  en: {
    title: string;
    content: string;
    metadata: any;
  };
  vi: {
    title: string;
    content: string;
    metadata: any;
  };
}

async function generateBilingualContent(
  keyword: string,
  researchData: any
): Promise<BilingualContent> {
  // Generate English version
  const enContent = await generateContent({
    keyword,
    researchData,
    language: 'en',
    format: 'toplist'
  });

  // Generate Vietnamese version (not just translation!)
  const viContent = await generateContent({
    keyword,
    researchData,
    language: 'vi',
    format: 'toplist',
    culturalContext: 'vietnamese-market'
  });

  return {
    en: enContent,
    vi: viContent
  };
}
```

### 6. Video Rendering with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

async function renderContentVideo(content: any) {
  // Bundle the Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.join(process.cwd(), 'remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  // Select composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title: content.title,
      points: content.keyPoints,
      background: content.brandColors
    },
  });

  // Render video
  const outputPath = path.join(process.cwd(), 'output', `${content.id}.mp4`);
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: composition.inputProps,
  });

  return outputPath;
}
```

### 7. Complete Pipeline Example

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { researchTopic } from '@/lib/research/crawler';
import { generateContent } from '@/lib/ai/content-generator';
import { renderContentVideo } from '@/lib/video/renderer';

export async function POST(req: NextRequest) {
  try {
    const { keyword, format, languages } = await req.json();

    // 1. Research phase
    const research = await researchTopic(keyword, {
      sources: ['techcrunch', 'a16z', 'twitter'],
      timeRange: '24h',
      maxArticles: 10
    });

    // 2. Content generation phase
    const content = await generateContent({
      keyword,
      researchData: research,
      format: format || 'toplist',
      languages: languages || ['en', 'vi'],
      tone: 'expert',
      aiProvider: 'claude'
    });

    // 3. Video rendering phase
    const videoPath = await renderContentVideo(content.en);

    // 4. Return results
    return NextResponse.json({
      success: true,
      content,
      videoUrl: `/videos/${path.basename(videoPath)}`
    });

  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { error: 'Pipeline failed', details: error.message },
      { status: 500 }
    );
  }
}
```

### 8. Frontend Component Example

```typescript
'use client';

import { useState } from 'react';

export default function ContentGenerator() {
  const [keyword, setKeyword] = useState('');
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
          format: 'toplist',
          languages: ['en', 'vi']
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
    <div className="content-generator">
      <input
        type="text"
        value={keyword}
        onChange={(e) => setKeyword(e.target.value)}
        placeholder="Enter topic keyword..."
      />
      <button onClick={handleGenerate} disabled={loading}>
        {loading ? 'Generating...' : 'Generate Content'}
      </button>

      {result && (
        <div className="results">
          <h2>{result.content.en.title}</h2>
          <div dangerouslySetInnerHTML={{ __html: result.content.en.content }} />
          {result.videoUrl && (
            <video src={result.videoUrl} controls />
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

# Render Remotion video (if separate)
npm run remotion:render
```

## Configuration Options

### Content Formats

- `toplist`: Numbered list articles (e.g., "Top 10 AI Tools")
- `pov`: Opinion/perspective pieces
- `case-study`: Detailed analysis with examples
- `how-to`: Step-by-step tutorials

### Tone Options

- `expert`: Professional, authoritative voice
- `friendly`: Conversational, approachable
- `humorous`: Light-hearted, entertaining

### Research Sources

Configure in `lib/research/sources.ts`:

```typescript
export const RESEARCH_SOURCES = {
  techcrunch: {
    enabled: true,
    apiEndpoint: 'https://techcrunch-api.p.rapidapi.com',
    weight: 1.0
  },
  a16z: {
    enabled: true,
    crawlUrl: 'https://a16z.com/latest',
    weight: 0.8
  },
  twitter: {
    enabled: true,
    hashtags: ['#AI', '#Marketing', '#Tech'],
    weight: 0.6
  }
};
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement rate limiting
import { Ratelimit } from '@upstash/ratelimit';
import { Redis } from '@upstash/redis';

const ratelimit = new Ratelimit({
  redis: Redis.fromEnv(),
  limiter: Ratelimit.slidingWindow(10, '1 h'),
});

// In API route
const { success } = await ratelimit.limit(userId);
if (!success) {
  return NextResponse.json({ error: 'Rate limit exceeded' }, { status: 429 });
}
```

### Claude/OpenAI Errors

```typescript
async function generateWithRetry(
  generateFn: () => Promise<string>,
  maxRetries = 3
) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await generateFn();
    } catch (error) {
      if (error.status === 429) {
        // Rate limited, wait and retry
        await new Promise(resolve => setTimeout(resolve, 2000 * (i + 1)));
      } else if (error.status === 500) {
        // Server error, retry
        continue;
      } else {
        throw error; // Other errors, don't retry
      }
    }
  }
  throw new Error('Max retries exceeded');
}
```

### Video Rendering Issues

```typescript
// Check Remotion configuration
import { getCompositions } from '@remotion/renderer';

async function debugRemotionSetup() {
  try {
    const compositions = await getCompositions(bundleLocation);
    console.log('Available compositions:', compositions);
  } catch (error) {
    console.error('Remotion setup error:', error);
    // Check: Node.js version, ffmpeg installation, license key
  }
}
```

### Crawling Blocked

```typescript
// Add user agent and delays
import axios from 'axios';

const crawlWithHeaders = async (url: string) => {
  return axios.get(url, {
    headers: {
      'User-Agent': 'Mozilla/5.0 (compatible; ContentBot/1.0)',
      'Accept': 'text/html,application/json',
    },
    timeout: 10000
  });
};

// Add delays between requests
async function crawlMultipleSources(sources: string[]) {
  const results = [];
  for (const source of sources) {
    results.push(await crawlWithHeaders(source));
    await new Promise(resolve => setTimeout(resolve, 1000)); // 1s delay
  }
  return results;
}
```

## Best Practices

1. **Cache Research Data**: Avoid re-crawling the same topic within 24h
2. **Batch API Calls**: Combine multiple AI requests when possible
3. **Queue Video Rendering**: Use a job queue (Bull, BullMQ) for video generation
4. **Monitor Costs**: Track API usage for Claude/OpenAI to manage expenses
5. **Validate Content**: Implement content moderation before publishing
6. **Version Control**: Keep track of prompts and AI model versions for consistency
