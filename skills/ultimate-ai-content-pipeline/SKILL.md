---
name: ultimate-ai-content-pipeline
description: Automated content pipeline from research to video generation using Claude/OpenAI and Remotion
triggers:
  - how do I automate content creation with AI
  - set up AI content pipeline for marketing
  - generate videos from blog posts automatically
  - crawl news and create content with AI
  - use Remotion to render marketing videos
  - build automated content workflow with Claude
  - create multilingual content with AI pipeline
  - automate research and scriptwriting for content
---

# Ultimate AI Content Pipeline Skill

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to use the Ultimate AI Content Pipeline - a complete automation system that handles content research, scriptwriting, multilingual generation, and video rendering. Built with TypeScript, Next.js, Claude/OpenAI APIs, and Remotion.

## What This Project Does

The Ultimate AI Content Pipeline is an end-to-end content automation system that:

1. **Auto-Research**: Crawls news sources (TechCrunch, a16z, Twitter, LinkedIn) for trending topics
2. **AI Writing**: Generates content in multiple formats (listicles, POV, case studies, how-tos) using Claude 3 or OpenAI
3. **Multilingual**: Creates parallel English and Vietnamese versions with customizable tone
4. **Video Generation**: Automatically renders videos and infographics from content using Remotion
5. **Platform Optimization**: Exports video in formats for Reels, TikTok, and Shorts

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
DATABASE_URL=your_database_url

# Next.js
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

### Run the Development Server

```bash
npm run dev
# or
yarn dev
# or
pnpm dev
```

The app will be available at `http://localhost:3000`.

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app router pages
│   ├── components/       # React components
│   ├── lib/             # Core utilities
│   │   ├── ai/          # AI providers (Claude, OpenAI)
│   │   ├── crawler/     # Web scraping utilities
│   │   ├── render/      # Remotion video rendering
│   │   └── content/     # Content generation logic
│   └── types/           # TypeScript definitions
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Core API Usage

### 1. Research & Crawling

```typescript
import { crawlNewsFeeds } from '@/lib/crawler/news-crawler';

// Crawl trending topics from multiple sources
async function researchTopic(keyword: string) {
  const sources = await crawlNewsFeeds({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeRange: '24h',
    limit: 20
  });
  
  return sources.filter(article => article.relevanceScore > 0.7);
}
```

### 2. AI Content Generation with Claude

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

async function generateContent(research: any[], format: string, language: string) {
  const prompt = `
Based on this research data: ${JSON.stringify(research)}
Create a ${format} article in ${language}.
Tone: Professional yet engaging.
Include data-backed insights and trending angles.
`;

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [
      { role: 'user', content: prompt }
    ],
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

async function generateWithGPT(research: any[], format: string) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: 'You are an expert content marketer who creates engaging, data-driven articles.'
      },
      {
        role: 'user',
        content: `Research: ${JSON.stringify(research)}\nFormat: ${format}\nCreate an article.`
      }
    ],
    temperature: 0.7,
    max_tokens: 3000,
  });

  return completion.choices[0].message.content;
}
```

### 4. Multilingual Content Pipeline

```typescript
import { generateContent } from '@/lib/ai/content-generator';

async function createMultilingualContent(keyword: string) {
  const research = await researchTopic(keyword);
  
  const [englishVersion, vietnameseVersion] = await Promise.all([
    generateContent(research, 'toplist', 'en', {
      tone: 'professional',
      targetAudience: 'B2B marketers'
    }),
    generateContent(research, 'toplist', 'vi', {
      tone: 'friendly',
      targetAudience: 'Vietnamese startups'
    })
  ]);

  return { english: englishVersion, vietnamese: vietnameseVersion };
}
```

### 5. Video Rendering with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

async function renderContentVideo(content: string, format: 'reels' | 'tiktok' | 'shorts') {
  const compositions = {
    reels: { width: 1080, height: 1920, fps: 30 },
    tiktok: { width: 1080, height: 1920, fps: 30 },
    shorts: { width: 1080, height: 1920, fps: 30 },
  };

  const bundled = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  const composition = await selectComposition({
    serveUrl: bundled,
    id: 'ContentVideo',
    inputProps: {
      content,
      format,
    },
  });

  await renderMedia({
    composition,
    serveUrl: bundled,
    codec: 'h264',
    outputLocation: `out/video-${format}-${Date.now()}.mp4`,
    inputProps: {
      content,
      format,
    },
  });
}
```

## Common Patterns

### Full Content Pipeline

```typescript
import { ContentPipeline } from '@/lib/content/pipeline';

async function runFullPipeline(keyword: string) {
  const pipeline = new ContentPipeline({
    aiProvider: 'claude', // or 'openai'
    languages: ['en', 'vi'],
    formats: ['toplist', 'case-study'],
    videoFormats: ['reels', 'tiktok'],
  });

  const result = await pipeline.execute({
    keyword,
    crawlSources: ['techcrunch', 'a16z'],
    autoPublish: false,
  });

  return {
    articles: result.content,
    videos: result.videos,
    metadata: result.analytics,
  };
}
```

### Custom Content Format

```typescript
interface ContentConfig {
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  tone: 'professional' | 'friendly' | 'humorous';
  length: 'short' | 'medium' | 'long';
  includeStats: boolean;
  includeCTA: boolean;
}

async function generateCustomContent(
  research: any[],
  config: ContentConfig
) {
  const systemPrompt = buildSystemPrompt(config);
  const userPrompt = buildUserPrompt(research, config);

  const content = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: config.length === 'long' ? 6000 : 3000,
    messages: [
      { role: 'system', content: systemPrompt },
      { role: 'user', content: userPrompt }
    ],
  });

  return content.content[0].text;
}

function buildSystemPrompt(config: ContentConfig): string {
  const toneMap = {
    professional: 'Write in a professional, authoritative tone',
    friendly: 'Write in a conversational, approachable tone',
    humorous: 'Write with wit and humor while staying informative',
  };

  return `You are an expert content writer. ${toneMap[config.tone]}.
${config.includeStats ? 'Include relevant statistics and data points.' : ''}
${config.includeCTA ? 'End with a clear call-to-action.' : ''}`;
}
```

### Scheduling & Auto-Publishing

```typescript
import { scheduleContent } from '@/lib/scheduler';

async function scheduleMultiplatform(content: any) {
  const schedule = await scheduleContent({
    platforms: ['facebook', 'linkedin', 'twitter'],
    content: {
      text: content.article,
      video: content.videoUrl,
      images: content.images,
    },
    publishTime: new Date(Date.now() + 3600000), // 1 hour from now
    autoPost: true,
  });

  return schedule;
}
```

## CLI Commands (if applicable)

```bash
# Generate content from keyword
npm run generate -- --keyword "AI marketing trends" --format toplist --lang en,vi

# Crawl and analyze trending topics
npm run research -- --sources techcrunch,a16z --hours 24

# Render video from existing content
npm run render -- --input content.json --format reels

# Run full pipeline
npm run pipeline -- --keyword "SaaS growth" --all
```

## Remotion Video Templates

### Basic Content Video Template

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, Sequence, useCurrentFrame } from 'remotion';

export const ContentVideo: React.FC<{
  content: string;
  format: string;
}> = ({ content, format }) => {
  const frame = useCurrentFrame();

  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a1a' }}>
      <Sequence from={0} durationInFrames={90}>
        <Title text={content.split('\n')[0]} frame={frame} />
      </Sequence>
      <Sequence from={90} durationInFrames={180}>
        <MainContent text={content} frame={frame} />
      </Sequence>
      <Sequence from={270} durationInFrames={60}>
        <CTA format={format} frame={frame} />
      </Sequence>
    </AbsoluteFill>
  );
};
```

## Configuration

### AI Provider Configuration

```typescript
// config/ai-providers.ts
export const aiConfig = {
  claude: {
    model: 'claude-3-5-sonnet-20241022',
    maxTokens: 4096,
    temperature: 0.7,
  },
  openai: {
    model: 'gpt-4-turbo-preview',
    maxTokens: 3000,
    temperature: 0.7,
  },
};

// Usage
import { aiConfig } from '@/config/ai-providers';

const message = await anthropic.messages.create({
  ...aiConfig.claude,
  messages: [{ role: 'user', content: prompt }],
});
```

### Crawler Configuration

```typescript
// config/crawler.ts
export const crawlerConfig = {
  sources: {
    techcrunch: {
      url: 'https://techcrunch.com/feed/',
      type: 'rss',
      rateLimit: 10, // requests per minute
    },
    a16z: {
      url: 'https://a16z.com/feed/',
      type: 'rss',
      rateLimit: 10,
    },
    twitter: {
      apiEndpoint: 'https://api.twitter.com/2/tweets/search/recent',
      requiresAuth: true,
      rateLimit: 5,
    },
  },
  defaultTimeRange: '24h',
  maxArticles: 50,
};
```

## Troubleshooting

### API Rate Limits

If you encounter rate limit errors:

```typescript
import { retry } from '@/lib/utils/retry';

async function generateWithRetry(prompt: string) {
  return retry(
    async () => {
      return await anthropic.messages.create({
        model: 'claude-3-5-sonnet-20241022',
        max_tokens: 4096,
        messages: [{ role: 'user', content: prompt }],
      });
    },
    {
      retries: 3,
      delay: 2000,
      backoff: 2,
    }
  );
}
```

### Video Rendering Memory Issues

For large video renders:

```typescript
await renderMedia({
  composition,
  serveUrl: bundled,
  codec: 'h264',
  outputLocation: `out/video.mp4`,
  // Reduce memory usage
  concurrency: 2, // Lower concurrency
  imageFormat: 'jpeg', // Use JPEG instead of PNG
  quality: 80, // Reduce quality slightly
});
```

### Crawler Blocked/Failed

Use RapidAPI proxy or add delays:

```typescript
import { delay } from '@/lib/utils/delay';

async function crawlWithDelay(urls: string[]) {
  const results = [];
  
  for (const url of urls) {
    try {
      const data = await fetch(url);
      results.push(await data.json());
      await delay(1000); // Wait 1 second between requests
    } catch (error) {
      console.error(`Failed to crawl ${url}:`, error);
    }
  }
  
  return results;
}
```

### Multilingual Character Encoding

Ensure proper UTF-8 encoding:

```typescript
async function saveContent(content: string, language: string) {
  const fs = require('fs').promises;
  
  await fs.writeFile(
    `content-${language}.txt`,
    content,
    { encoding: 'utf8' }
  );
}
```

## Best Practices

1. **Always use environment variables** for API keys
2. **Implement retry logic** for AI API calls
3. **Cache research results** to avoid redundant crawling
4. **Rate limit your requests** to external APIs
5. **Validate content quality** before auto-publishing
6. **Monitor token usage** to control costs
7. **Use TypeScript types** for all data structures
8. **Test video templates** before batch rendering
