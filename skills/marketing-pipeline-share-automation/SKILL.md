---
name: marketing-pipeline-share-automation
description: Automated content pipeline from research to video generation using AI (Claude/OpenAI) and Remotion
triggers:
  - automate content creation from research to video
  - generate marketing content with AI pipeline
  - create videos from text using Remotion
  - build automated content workflow
  - scrape and generate content with Claude
  - automate social media content generation
  - create multilingual marketing content
  - build AI-powered content assembly line
---

# Marketing Pipeline Share - AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with Marketing Pipeline Share, a complete AI-powered content automation system that handles research, script writing, and video generation. The pipeline crawls recent news from sources like TechCrunch, a16z, Twitter/X, and LinkedIn, then uses Claude/OpenAI to generate content in multiple formats and languages, finally rendering videos using Remotion.

## What It Does

Marketing Pipeline Share automates the entire content creation workflow:

1. **Auto-Research**: Crawls and analyzes real-time data from news sources (last 24h)
2. **AI Content Generation**: Creates content in multiple formats (Toplist, POV, Case Study, How-to) using Claude 3 or OpenAI
3. **Multilingual Support**: Generates content in both English and Vietnamese with customizable tone
4. **Video Rendering**: Automatically converts content to videos/infographics using Remotion
5. **Multi-Platform Export**: Outputs video in formats optimized for Reels, TikTok, Shorts

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

### Required Environment Variables

```bash
# AI API Keys
ANTHROPIC_API_KEY=your_claude_api_key_here
OPENAI_API_KEY=your_openai_api_key_here

# RapidAPI for content scraping
RAPIDAPI_KEY=your_rapidapi_key_here

# Optional: Database connection
DATABASE_URL=your_database_connection_string

# Next.js configuration
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── app/                    # Next.js app directory
├── components/             # React components
├── lib/                    # Core utilities
│   ├── ai/                # AI integration (Claude, OpenAI)
│   ├── scraper/           # Content scraping modules
│   └── remotion/          # Video generation
├── public/                # Static assets
└── remotion/              # Remotion video templates
```

## Core API Usage

### 1. Content Research & Scraping

```typescript
import { scrapeNewsFromSources } from '@/lib/scraper/news-scraper';

// Scrape recent news on a topic
async function researchTopic(keyword: string) {
  const sources = ['techcrunch', 'a16z', 'twitter', 'linkedin'];
  
  const articles = await scrapeNewsFromSources({
    keyword,
    sources,
    timeRange: '24h',
    limit: 20
  });
  
  return articles;
}

// Example usage
const aiNews = await researchTopic('artificial intelligence');
console.log(`Found ${aiNews.length} recent articles`);
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
  language: 'en' | 'vi',
  tone: 'expert' | 'friendly' | 'humorous'
) {
  const prompt = buildPrompt(research, format, language, tone);
  
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
  
  return message.content[0].text;
}

function buildPrompt(research: any[], format: string, language: string, tone: string) {
  const researchText = research.map(r => 
    `Title: ${r.title}\nSummary: ${r.summary}\nSource: ${r.url}`
  ).join('\n\n');
  
  return `You are a ${tone} content writer. Create a ${format} article in ${language} based on this research:

${researchText}

Requirements:
- Use data-backed insights
- Include specific examples and statistics
- Make it engaging and actionable
- Format: ${format}
- Language: ${language}
- Tone: ${tone}`;
}
```

### 3. Alternative: OpenAI Content Generation

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateWithOpenAI(research: any[], format: string) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: 'You are an expert marketing content writer specializing in data-driven articles.'
      },
      {
        role: 'user',
        content: buildPrompt(research, format, 'en', 'expert')
      }
    ],
    temperature: 0.7,
    max_tokens: 3000,
  });
  
  return completion.choices[0].message.content;
}
```

### 4. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  title: string;
  content: string[];
  theme: 'dark' | 'light';
  platform: 'reels' | 'tiktok' | 'shorts';
}

async function generateVideo(config: VideoConfig) {
  // Bundle Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.join(process.cwd(), 'remotion/index.ts'),
    webpackOverride: (config) => config,
  });
  
  // Get composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: config,
  });
  
  // Render video
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
    inputProps: config,
  });
  
  return outputLocation;
}

// Example usage
const videoPath = await generateVideo({
  title: 'Top 5 AI Trends in 2024',
  content: [
    'Trend 1: Multimodal AI',
    'Trend 2: AI Agents',
    'Trend 3: Open Source Models',
    'Trend 4: Edge AI',
    'Trend 5: AI Regulation'
  ],
  theme: 'dark',
  platform: 'reels'
});
```

### 5. Complete Pipeline Example

```typescript
import { scrapeNewsFromSources } from '@/lib/scraper/news-scraper';
import { generateContent } from '@/lib/ai/claude';
import { generateVideo } from '@/lib/remotion/render';

async function runContentPipeline(keyword: string) {
  try {
    console.log(`Starting pipeline for: ${keyword}`);
    
    // Step 1: Research
    console.log('Researching...');
    const research = await scrapeNewsFromSources({
      keyword,
      sources: ['techcrunch', 'twitter'],
      timeRange: '24h',
      limit: 10
    });
    
    // Step 2: Generate content (bilingual)
    console.log('Generating content...');
    const [englishContent, vietnameseContent] = await Promise.all([
      generateContent(research, 'toplist', 'en', 'expert'),
      generateContent(research, 'toplist', 'vi', 'friendly')
    ]);
    
    // Step 3: Extract key points for video
    const keyPoints = extractKeyPoints(englishContent);
    
    // Step 4: Generate video
    console.log('Rendering video...');
    const videoPath = await generateVideo({
      title: `Top Insights: ${keyword}`,
      content: keyPoints,
      theme: 'dark',
      platform: 'reels'
    });
    
    return {
      research,
      content: {
        english: englishContent,
        vietnamese: vietnameseContent
      },
      video: videoPath
    };
  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}

function extractKeyPoints(content: string, maxPoints: number = 5): string[] {
  // Simple extraction - can be enhanced with AI
  const lines = content.split('\n');
  const points = lines
    .filter(line => line.match(/^\d+\.|^-|^•/))
    .slice(0, maxPoints)
    .map(line => line.replace(/^\d+\.\s*|^-\s*|^•\s*/, ''));
  
  return points;
}

// Run the pipeline
const result = await runContentPipeline('artificial intelligence');
console.log('Pipeline complete!', result);
```

## Next.js API Routes

### Create Content Endpoint

```typescript
// app/api/content/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline';

export async function POST(request: NextRequest) {
  try {
    const { keyword, format, language, tone } = await request.json();
    
    if (!keyword) {
      return NextResponse.json(
        { error: 'Keyword is required' },
        { status: 400 }
      );
    }
    
    const result = await runContentPipeline(keyword);
    
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

### Video Rendering Endpoint

```typescript
// app/api/video/render/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { generateVideo } from '@/lib/remotion/render';

export async function POST(request: NextRequest) {
  try {
    const config = await request.json();
    
    const videoPath = await generateVideo(config);
    
    return NextResponse.json({
      success: true,
      videoUrl: videoPath.replace('public', '')
    });
  } catch (error) {
    console.error('Video rendering error:', error);
    return NextResponse.json(
      { error: 'Failed to render video' },
      { status: 500 }
    );
  }
}
```

## Common Patterns

### Rate Limiting for API Calls

```typescript
class RateLimiter {
  private queue: Array<() => Promise<any>> = [];
  private running = 0;
  private maxConcurrent: number;
  private delay: number;
  
  constructor(maxConcurrent = 3, delayMs = 1000) {
    this.maxConcurrent = maxConcurrent;
    this.delay = delayMs;
  }
  
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
    if (this.running >= this.maxConcurrent || this.queue.length === 0) {
      return;
    }
    
    this.running++;
    const fn = this.queue.shift()!;
    
    await fn();
    await new Promise(resolve => setTimeout(resolve, this.delay));
    
    this.running--;
    this.process();
  }
}

// Usage
const limiter = new RateLimiter(2, 2000);

const results = await Promise.all(
  keywords.map(keyword => 
    limiter.add(() => scrapeNewsFromSources({ keyword }))
  )
);
```

### Content Caching

```typescript
import { Redis } from '@upstash/redis';

const redis = new Redis({
  url: process.env.UPSTASH_REDIS_URL,
  token: process.env.UPSTASH_REDIS_TOKEN,
});

async function getCachedOrGenerate(
  key: string,
  generator: () => Promise<any>,
  ttl: number = 3600
) {
  // Check cache
  const cached = await redis.get(key);
  if (cached) {
    return cached;
  }
  
  // Generate new content
  const result = await generator();
  
  // Cache result
  await redis.setex(key, ttl, JSON.stringify(result));
  
  return result;
}

// Usage
const content = await getCachedOrGenerate(
  `content:${keyword}:${format}`,
  () => generateContent(research, format, 'en', 'expert'),
  7200 // 2 hours
);
```

### Error Handling & Retry Logic

```typescript
async function withRetry<T>(
  fn: () => Promise<T>,
  maxRetries = 3,
  delayMs = 1000
): Promise<T> {
  let lastError: Error;
  
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      lastError = error as Error;
      console.log(`Attempt ${i + 1} failed, retrying...`);
      
      if (i < maxRetries - 1) {
        await new Promise(resolve => 
          setTimeout(resolve, delayMs * Math.pow(2, i))
        );
      }
    }
  }
  
  throw lastError!;
}

// Usage
const content = await withRetry(
  () => generateContent(research, 'toplist', 'en', 'expert'),
  3,
  2000
);
```

## Running the Application

```bash
# Development mode
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Render Remotion video (standalone)
npm run remotion:render
```

## Troubleshooting

### API Key Issues

```typescript
// Validate API keys on startup
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

validateEnv();
```

### Scraping Failures

- Check RapidAPI quota and subscription status
- Verify source URLs are accessible
- Implement fallback sources if primary fails
- Use proxy rotation for rate-limited sources

### Video Rendering Issues

```bash
# Install required system dependencies for Remotion
# On Ubuntu/Debian:
sudo apt-get update
sudo apt-get install -y chromium-browser ffmpeg

# On macOS:
brew install chromium ffmpeg

# Set Chrome path if needed
export CHROME_EXECUTABLE_PATH=/path/to/chromium
```

### Memory Issues with Large Videos

```typescript
// Optimize video rendering
await renderMedia({
  composition,
  serveUrl: bundleLocation,
  codec: 'h264',
  outputLocation,
  inputProps: config,
  // Optimize settings
  concurrency: 1,
  enforceAudioTrack: false,
  chromiumOptions: {
    args: ['--no-sandbox', '--disable-dev-shm-usage']
  }
});
```

### Claude API Rate Limits

```typescript
// Implement exponential backoff
async function callClaudeWithBackoff(prompt: string) {
  const maxRetries = 5;
  
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await anthropic.messages.create({
        model: 'claude-3-5-sonnet-20241022',
        max_tokens: 4096,
        messages: [{ role: 'user', content: prompt }],
      });
    } catch (error: any) {
      if (error.status === 429 && i < maxRetries - 1) {
        const delay = Math.pow(2, i) * 1000;
        console.log(`Rate limited, waiting ${delay}ms...`);
        await new Promise(resolve => setTimeout(resolve, delay));
        continue;
      }
      throw error;
    }
  }
}
```

This skill enables AI coding agents to effectively use Marketing Pipeline Share for automated content creation, from research through video generation, with proper error handling and optimization strategies.
