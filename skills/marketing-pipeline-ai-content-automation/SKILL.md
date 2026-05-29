---
name: marketing-pipeline-ai-content-automation
description: Automate content creation from research to video generation using AI (Claude/OpenAI) and Remotion for multi-platform marketing content
triggers:
  - how do I automate content creation with AI
  - set up an AI content pipeline
  - generate marketing videos from text automatically
  - crawl news and create content with Claude
  - build automated content workflow with remotion
  - create multilingual marketing content with AI
  - automate social media content generation
  - research and write content using AI agents
---

# Marketing Pipeline AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the Ultimate AI Content Pipeline - an end-to-end automated content creation system that handles research, scriptwriting, posting, and video generation using Claude 3, OpenAI, and Remotion.

## What It Does

The Marketing Pipeline automates the entire content creation workflow:

1. **Auto-Research**: Crawls recent news from TechCrunch, a16z, Twitter/X, LinkedIn
2. **AI Content Generation**: Creates articles in multiple formats (toplist, POV, case study, how-to) using Claude/OpenAI
3. **Multilingual Support**: Generates content in both English and Vietnamese
4. **Video Rendering**: Automatically creates videos and infographics using Remotion
5. **Multi-Platform Output**: Exports content optimized for Reels, TikTok, Shorts

## Installation

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
yarn install

# Set up environment variables
cp .env.example .env
```

## Configuration

Create a `.env` file with the following variables:

```bash
# AI Providers
ANTHROPIC_API_KEY=your_claude_api_key_here
OPENAI_API_KEY=your_openai_api_key_here

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key_here
TWITTER_API_KEY=your_twitter_api_key_here

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Next.js
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── crawler/     # News crawling logic
│   │   ├── video/       # Remotion video generation
│   │   └── utils/       # Helper functions
│   └── types/           # TypeScript type definitions
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Running the Application

```bash
# Development mode
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Render videos with Remotion
npm run remotion
```

## Core APIs and Usage

### 1. Research & Crawling

```typescript
import { crawlLatestNews } from '@/lib/crawler';
import { analyzeInsights } from '@/lib/ai/research';

async function gatherResearch(topic: string) {
  // Crawl news from multiple sources
  const newsData = await crawlLatestNews({
    keyword: topic,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeframe: '24h'
  });

  // Extract insights using AI
  const insights = await analyzeInsights(newsData, {
    model: 'claude-3-opus-20240229',
    language: 'en'
  });

  return insights;
}
```

### 2. Content Generation

```typescript
import { generateContent } from '@/lib/ai/content-generator';

async function createArticle(topic: string, format: string) {
  const content = await generateContent({
    topic,
    format, // 'toplist' | 'pov' | 'case-study' | 'how-to'
    provider: 'claude', // or 'openai'
    languages: ['en', 'vi'],
    tone: 'professional', // 'friendly' | 'humorous' | 'expert'
    includeData: true,
    researchContext: await gatherResearch(topic)
  });

  return content;
}

// Usage
const article = await createArticle(
  'AI in Marketing Automation',
  'toplist'
);

console.log(article.en); // English version
console.log(article.vi); // Vietnamese version
```

### 3. Multi-Format Content Generation

```typescript
import { ContentPipeline } from '@/lib/pipeline';

const pipeline = new ContentPipeline({
  anthropicKey: process.env.ANTHROPIC_API_KEY,
  openaiKey: process.env.OPENAI_API_KEY
});

async function generateMultiFormatContent(keyword: string) {
  const job = await pipeline.create({
    keyword,
    formats: ['toplist', 'pov', 'how-to'],
    languages: ['en', 'vi'],
    includeVideo: true,
    includeInfographic: true
  });

  // Monitor progress
  pipeline.on('progress', (status) => {
    console.log(`${status.step}: ${status.progress}%`);
  });

  const result = await pipeline.execute(job);

  return {
    articles: result.content,
    videos: result.videos,
    images: result.images
  };
}
```

### 4. Video Generation with Remotion

```typescript
import { renderVideo } from '@/lib/video/renderer';
import { createVideoScript } from '@/lib/ai/video-script';

async function generateMarketingVideo(content: any) {
  // Create video script from content
  const script = await createVideoScript({
    content,
    duration: 60, // seconds
    platform: 'tiktok' // 'reels' | 'shorts' | 'tiktok'
  });

  // Render video using Remotion
  const videoUrl = await renderVideo({
    script,
    template: 'modern-infographic',
    aspectRatio: '9:16', // vertical for mobile
    outputFormat: 'mp4',
    quality: 'high'
  });

  return videoUrl;
}
```

### 5. Complete End-to-End Pipeline

```typescript
import { AutoContentPipeline } from '@/lib/auto-pipeline';

async function runFullPipeline(topic: string) {
  const pipeline = new AutoContentPipeline({
    anthropicKey: process.env.ANTHROPIC_API_KEY,
    openaiKey: process.env.OPENAI_API_KEY,
    rapidApiKey: process.env.RAPIDAPI_KEY
  });

  const result = await pipeline.run({
    topic,
    steps: [
      'research',      // Crawl latest news
      'analyze',       // Extract insights
      'write',         // Generate articles
      'translate',     // Create multilingual versions
      'visualize',     // Create infographics
      'render-video'   // Generate videos
    ],
    config: {
      articleFormats: ['toplist', 'case-study'],
      languages: ['en', 'vi'],
      videoFormats: ['reels', 'tiktok'],
      autoPublish: false
    }
  });

  return {
    articles: result.articles,
    videos: result.videos,
    infographics: result.images,
    metadata: result.metadata
  };
}

// Execute pipeline
const output = await runFullPipeline('AI Marketing Trends 2024');
```

## Common Patterns

### Custom AI Provider Configuration

```typescript
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

const claude = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY
});

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY
});

async function generateWithClaude(prompt: string) {
  const message = await claude.messages.create({
    model: 'claude-3-opus-20240229',
    max_tokens: 4000,
    messages: [{
      role: 'user',
      content: prompt
    }]
  });

  return message.content[0].text;
}

async function generateWithOpenAI(prompt: string) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [{
      role: 'user',
      content: prompt
    }],
    max_tokens: 4000
  });

  return completion.choices[0].message.content;
}
```

### Custom Crawler Integration

```typescript
import axios from 'axios';

interface CrawlerConfig {
  sources: string[];
  keyword: string;
  limit?: number;
}

async function customCrawler(config: CrawlerConfig) {
  const results = [];

  for (const source of config.sources) {
    try {
      const response = await axios.get(`https://api.${source}.com/search`, {
        params: {
          q: config.keyword,
          limit: config.limit || 10
        },
        headers: {
          'X-RapidAPI-Key': process.env.RAPIDAPI_KEY
        }
      });

      results.push({
        source,
        articles: response.data.articles
      });
    } catch (error) {
      console.error(`Failed to crawl ${source}:`, error);
    }
  }

  return results;
}
```

### Batch Processing

```typescript
async function batchGenerateContent(topics: string[]) {
  const results = await Promise.allSettled(
    topics.map(async (topic) => {
      const pipeline = new AutoContentPipeline({
        anthropicKey: process.env.ANTHROPIC_API_KEY,
        openaiKey: process.env.OPENAI_API_KEY
      });

      return await pipeline.run({
        topic,
        steps: ['research', 'write', 'render-video'],
        config: {
          articleFormats: ['toplist'],
          languages: ['en'],
          videoFormats: ['reels']
        }
      });
    })
  );

  return results
    .filter(r => r.status === 'fulfilled')
    .map(r => r.value);
}
```

### Scheduling Content Generation

```typescript
import cron from 'node-cron';

// Run content generation daily at 9 AM
cron.schedule('0 9 * * *', async () => {
  const trendingTopics = await fetchTrendingTopics();
  
  for (const topic of trendingTopics) {
    await runFullPipeline(topic);
  }
  
  console.log('Daily content generation completed');
});
```

## Remotion Video Templates

```typescript
// remotion/templates/InfographicVideo.tsx
import { AbsoluteFill, Sequence, useCurrentFrame } from 'remotion';

export const InfographicVideo: React.FC<{
  title: string;
  points: string[];
  duration: number;
}> = ({ title, points, duration }) => {
  const frame = useCurrentFrame();

  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a1a' }}>
      <Sequence from={0} durationInFrames={60}>
        <h1 style={{ color: 'white', fontSize: 64 }}>
          {title}
        </h1>
      </Sequence>
      
      {points.map((point, idx) => (
        <Sequence
          key={idx}
          from={60 + idx * 90}
          durationInFrames={90}
        >
          <div style={{ color: 'white', fontSize: 32 }}>
            {point}
          </div>
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};
```

## Troubleshooting

### API Rate Limits

```typescript
import { RateLimiter } from '@/lib/utils/rate-limiter';

const limiter = new RateLimiter({
  maxRequests: 50,
  windowMs: 60000 // 1 minute
});

async function apiCallWithRateLimit(fn: () => Promise<any>) {
  await limiter.wait();
  return await fn();
}
```

### Error Handling

```typescript
import { retry } from '@/lib/utils/retry';

async function robustGeneration(topic: string) {
  try {
    return await retry(
      () => generateContent({ topic }),
      {
        maxAttempts: 3,
        delayMs: 1000,
        onRetry: (error, attempt) => {
          console.log(`Attempt ${attempt} failed:`, error.message);
        }
      }
    );
  } catch (error) {
    console.error('All attempts failed:', error);
    // Fallback to simpler model or cached content
    return await generateWithFallback(topic);
  }
}
```

### Memory Management for Large Batches

```typescript
async function processLargeBatch(topics: string[]) {
  const chunkSize = 5;
  const results = [];

  for (let i = 0; i < topics.length; i += chunkSize) {
    const chunk = topics.slice(i, i + chunkSize);
    const chunkResults = await Promise.all(
      chunk.map(topic => runFullPipeline(topic))
    );
    
    results.push(...chunkResults);
    
    // Clear memory between chunks
    if (global.gc) {
      global.gc();
    }
  }

  return results;
}
```

### Video Rendering Issues

```bash
# If Remotion fails to render, ensure FFmpeg is installed
brew install ffmpeg  # macOS
apt-get install ffmpeg  # Ubuntu/Debian

# Increase memory for large video renders
NODE_OPTIONS="--max-old-space-size=8192" npm run remotion
```

### Debug Mode

```typescript
const pipeline = new AutoContentPipeline({
  anthropicKey: process.env.ANTHROPIC_API_KEY,
  openaiKey: process.env.OPENAI_API_KEY,
  debug: true,  // Enable detailed logging
  logLevel: 'verbose'
});

pipeline.on('log', (message) => {
  console.log(`[${new Date().toISOString()}] ${message}`);
});
```

This skill equips AI coding agents with comprehensive knowledge to implement, customize, and troubleshoot the Marketing Pipeline AI Content Automation system for automated content creation workflows.
