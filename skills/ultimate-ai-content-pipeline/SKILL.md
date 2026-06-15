---
name: ultimate-ai-content-pipeline
description: Full-stack AI content automation pipeline for research, scriptwriting, publishing, and video generation using Claude/OpenAI and Remotion
triggers:
  - "set up AI content automation pipeline"
  - "automate content research and generation workflow"
  - "create automated marketing content system"
  - "build content pipeline with Claude and OpenAI"
  - "generate videos from written content automatically"
  - "crawl news and create AI-powered articles"
  - "set up Remotion video rendering pipeline"
  - "automate social media content creation"
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

A comprehensive TypeScript-based content automation system that handles the entire content lifecycle: from research (web scraping news sources), to AI-powered scriptwriting (using Claude/OpenAI), to automated video generation (using Remotion). Built with Next.js for a smooth UI experience.

## What This Project Does

This pipeline automates:
1. **Research Phase**: Auto-crawls news from TechCrunch, a16z, X (Twitter), LinkedIn for fresh data
2. **Content Generation**: Uses Claude 3/OpenAI to create multi-format content (toplist, POV, case study, how-to)
3. **Multi-language Support**: Generates content in both English and Vietnamese with customizable tone
4. **Video Rendering**: Converts written content into infographics and short-form videos via Remotion
5. **Auto-Publishing**: Schedules and publishes to multiple platforms

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

### Required Environment Variables

```bash
# .env.local
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key
RAPIDAPI_KEY=your_rapidapi_key

# Optional: Database
DATABASE_URL=postgresql://...

# Optional: Social Media Publishing
FACEBOOK_ACCESS_TOKEN=your_token
TWITTER_API_KEY=your_key
LINKEDIN_ACCESS_TOKEN=your_token
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── scraper/     # Web scraping modules
│   │   ├── video/       # Remotion video rendering
│   │   └── publisher/   # Social media publishing
│   └── types/           # TypeScript definitions
├── remotion/            # Remotion video compositions
└── public/              # Static assets
```

## Core API Usage

### 1. Research & Content Scraping

```typescript
import { scrapeNews } from '@/lib/scraper/news-scraper';

// Scrape latest news from multiple sources
async function researchTopic(keyword: string) {
  const sources = ['techcrunch', 'a16z', 'twitter', 'linkedin'];
  
  const newsData = await scrapeNews({
    keyword,
    sources,
    timeframe: '24h',
    limit: 50
  });

  return newsData;
}

// Example usage
const insights = await researchTopic('AI marketing automation');
console.log(insights); // Array of articles with metadata
```

### 2. AI Content Generation with Claude

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

async function generateContent(
  research: any[],
  format: 'toplist' | 'pov' | 'case-study' | 'how-to',
  language: 'en' | 'vi' = 'en'
) {
  const prompt = `
Based on the following research data, create a ${format} article in ${language}:

Research Data:
${JSON.stringify(research, null, 2)}

Requirements:
- Engaging headline
- Data-backed insights
- ${format === 'toplist' ? 'Numbered list format' : ''}
- ${language === 'vi' ? 'Vietnamese tone: professional yet friendly' : 'Professional English'}
- Include statistics and quotes from research
  `;

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
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

async function generateWithGPT(
  topic: string,
  tone: 'expert' | 'friendly' | 'humorous'
) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are a ${tone} content creator specializing in marketing content.`
      },
      {
        role: 'user',
        content: `Create a comprehensive article about: ${topic}`
      }
    ],
    temperature: 0.7,
  });

  return completion.choices[0].message.content;
}
```

### 4. Video Generation with Remotion

```typescript
// remotion/Composition.tsx
import { AbsoluteFill, Sequence, useCurrentFrame } from 'remotion';

export const ArticleVideo: React.FC<{
  title: string;
  keyPoints: string[];
  stats: { label: string; value: string }[];
}> = ({ title, keyPoints, stats }) => {
  const frame = useCurrentFrame();

  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a2e' }}>
      <Sequence from={0} durationInFrames={90}>
        <div style={{ 
          fontSize: 60, 
          color: 'white', 
          padding: 60,
          opacity: Math.min(1, frame / 30)
        }}>
          {title}
        </div>
      </Sequence>
      
      {keyPoints.map((point, i) => (
        <Sequence key={i} from={90 + i * 120} durationInFrames={120}>
          <div style={{ padding: 60, color: 'white', fontSize: 40 }}>
            • {point}
          </div>
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};
```

```typescript
// lib/video/render.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

export async function renderArticleVideo(
  contentData: {
    title: string;
    keyPoints: string[];
    stats: any[];
  },
  outputPath: string
) {
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ArticleVideo',
    inputProps: contentData,
  });

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: contentData,
  });

  return outputPath;
}
```

## Complete Pipeline Example

```typescript
// app/api/generate-content/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { scrapeNews } from '@/lib/scraper/news-scraper';
import { generateContent } from '@/lib/ai/claude';
import { renderArticleVideo } from '@/lib/video/render';
import { publishToSocial } from '@/lib/publisher/social';

export async function POST(req: NextRequest) {
  const { keyword, format, language, platforms } = await req.json();

  try {
    // Step 1: Research
    const research = await scrapeNews({
      keyword,
      sources: ['techcrunch', 'a16z'],
      timeframe: '24h'
    });

    // Step 2: Generate content
    const article = await generateContent(research, format, language);
    
    // Parse article for video creation
    const parsed = parseArticle(article);

    // Step 3: Render video
    const videoPath = await renderArticleVideo({
      title: parsed.title,
      keyPoints: parsed.keyPoints,
      stats: parsed.stats
    }, `./output/video-${Date.now()}.mp4`);

    // Step 4: Publish
    const published = await publishToSocial({
      text: article,
      video: videoPath,
      platforms: platforms || ['facebook', 'twitter', 'linkedin']
    });

    return NextResponse.json({
      success: true,
      article,
      videoPath,
      published
    });
  } catch (error) {
    return NextResponse.json(
      { error: error.message },
      { status: 500 }
    );
  }
}

function parseArticle(article: string) {
  // Extract title, key points, and stats from article
  const lines = article.split('\n');
  const title = lines[0].replace(/^#\s*/, '');
  const keyPoints = lines
    .filter(l => l.match(/^-\s|^\d+\./))
    .map(l => l.replace(/^-\s|^\d+\.\s/, ''));
  
  return { title, keyPoints, stats: [] };
}
```

## Frontend Integration

```typescript
// app/page.tsx
'use client';

import { useState } from 'react';

export default function Home() {
  const [keyword, setKeyword] = useState('');
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState(null);

  async function handleGenerate() {
    setLoading(true);
    
    const response = await fetch('/api/generate-content', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        keyword,
        format: 'toplist',
        language: 'vi',
        platforms: ['facebook', 'twitter']
      })
    });

    const data = await response.json();
    setResult(data);
    setLoading(false);
  }

  return (
    <div className="p-8">
      <h1 className="text-4xl font-bold mb-8">
        AI Content Pipeline
      </h1>
      
      <input
        type="text"
        value={keyword}
        onChange={(e) => setKeyword(e.target.value)}
        placeholder="Enter keyword..."
        className="border p-2 w-full mb-4"
      />
      
      <button
        onClick={handleGenerate}
        disabled={loading}
        className="bg-blue-500 text-white px-6 py-2 rounded"
      >
        {loading ? 'Generating...' : 'Generate Content'}
      </button>

      {result && (
        <div className="mt-8">
          <h2 className="text-2xl font-bold mb-4">Result</h2>
          <pre className="bg-gray-100 p-4 rounded overflow-auto">
            {JSON.stringify(result, null, 2)}
          </pre>
        </div>
      )}
    </div>
  );
}
```

## Configuration Patterns

### Custom Scraper Configuration

```typescript
// lib/scraper/config.ts
export const scraperConfig = {
  techcrunch: {
    baseUrl: 'https://techcrunch.com',
    selectors: {
      article: '.post-block',
      title: '.post-block__title',
      content: '.article-content'
    }
  },
  a16z: {
    baseUrl: 'https://a16z.com',
    selectors: {
      article: '.article-card',
      title: 'h2',
      content: '.content'
    }
  }
};
```

### AI Model Configuration

```typescript
// lib/ai/config.ts
export const aiConfig = {
  claude: {
    model: 'claude-3-5-sonnet-20241022',
    maxTokens: 4096,
    temperature: 0.7
  },
  openai: {
    model: 'gpt-4-turbo-preview',
    maxTokens: 4096,
    temperature: 0.7
  }
};

export const contentFormats = {
  toplist: {
    structure: 'numbered-list',
    minItems: 5,
    includeStats: true
  },
  pov: {
    structure: 'narrative',
    perspective: 'first-person',
    includeExamples: true
  },
  'case-study': {
    structure: 'problem-solution',
    includeMetrics: true,
    includeTimeline: true
  },
  'how-to': {
    structure: 'step-by-step',
    includeVisuals: true,
    difficulty: 'beginner'
  }
};
```

## Common Patterns

### Batch Content Generation

```typescript
async function batchGenerateContent(keywords: string[]) {
  const results = await Promise.all(
    keywords.map(async (keyword) => {
      const research = await scrapeNews({ keyword, timeframe: '24h' });
      const content = await generateContent(research, 'toplist', 'vi');
      return { keyword, content };
    })
  );

  return results;
}
```

### Scheduled Content Pipeline

```typescript
import cron from 'node-cron';

// Run every day at 9 AM
cron.schedule('0 9 * * *', async () => {
  const trendingTopics = await fetchTrendingTopics();
  
  for (const topic of trendingTopics) {
    const research = await scrapeNews({ keyword: topic, timeframe: '24h' });
    const article = await generateContent(research, 'toplist', 'vi');
    const videoPath = await renderArticleVideo(parseArticle(article), `./output/${topic}.mp4`);
    
    await publishToSocial({
      text: article,
      video: videoPath,
      platforms: ['facebook', 'linkedin']
    });
  }
});
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
    const fn = this.queue.shift()!;
    await fn();
    
    // Wait 1 second between requests
    await new Promise(resolve => setTimeout(resolve, 1000));
    this.processing = false;
    this.process();
  }
}

export const rateLimiter = new RateLimiter();
```

### Error Handling for Scraping

```typescript
async function safeScrape(url: string) {
  try {
    const response = await fetch(url, {
      headers: {
        'User-Agent': 'Mozilla/5.0 (compatible; ContentBot/1.0)'
      }
    });
    
    if (!response.ok) {
      throw new Error(`HTTP ${response.status}`);
    }
    
    return await response.text();
  } catch (error) {
    console.error(`Failed to scrape ${url}:`, error);
    return null;
  }
}
```

### Video Rendering Timeout

```typescript
async function renderWithTimeout(inputProps: any, timeout = 300000) {
  return Promise.race([
    renderArticleVideo(inputProps, './output/video.mp4'),
    new Promise((_, reject) => 
      setTimeout(() => reject(new Error('Render timeout')), timeout)
    )
  ]);
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

# Render videos (if using Remotion CLI)
npx remotion render ArticleVideo output/video.mp4
```

This pipeline provides a complete, production-ready system for automated content marketing at scale.
