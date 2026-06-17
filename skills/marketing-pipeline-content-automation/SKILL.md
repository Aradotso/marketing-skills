---
name: marketing-pipeline-content-automation
description: Automated AI content pipeline for research, scriptwriting, social media posting, and video generation using Claude/OpenAI and Remotion
triggers:
  - automate content creation with AI pipeline
  - generate videos from blog posts automatically
  - crawl trending news and create content
  - schedule social media posts with AI
  - create multi-format content with Claude
  - build automated marketing content workflow
  - scrape research data and generate articles
  - render videos from text using Remotion
---

# Marketing Pipeline Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with **marketing-pipeline-share**, a complete automated content pipeline that goes from research → scriptwriting → social media posting → video generation. The system uses AI (Claude 3, OpenAI) to crawl trending news, generate multi-format content in multiple languages, and automatically render videos using Remotion.

## What This Project Does

**marketing-pipeline-share** is an all-in-one content production system that:

- **Auto-scans research sources**: Crawls TechCrunch, a16z, Twitter/X, LinkedIn for trending topics in the last 24 hours
- **Generates diverse content formats**: Creates Toplists, POV articles, Case Studies, How-tos using Claude/OpenAI
- **Multi-language support**: Automatically generates English and Vietnamese versions
- **Auto-renders videos**: Converts written content into infographics and short videos using Remotion
- **Platform optimization**: Exports videos in formats optimized for Reels, TikTok, Shorts

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

# Set up environment variables
cp .env.example .env
```

### Required Environment Variables

```bash
# .env
OPENAI_API_KEY=your_openai_key_here
ANTHROPIC_API_KEY=your_claude_key_here
RAPIDAPI_KEY=your_rapidapi_key_here

# Database (if applicable)
DATABASE_URL=your_database_url

# Social Media APIs (optional)
FACEBOOK_ACCESS_TOKEN=your_token
TWITTER_API_KEY=your_key

# Remotion Configuration
REMOTION_AWS_ACCESS_KEY_ID=your_aws_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret
```

## Project Structure

```
marketing-pipeline-share/
├── src/
│   ├── app/              # Next.js app router pages
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── crawler/     # Web scraping modules
│   │   ├── content/     # Content generation logic
│   │   └── video/       # Remotion video rendering
│   ├── config/          # Configuration files
│   └── types/           # TypeScript type definitions
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Core Modules & Usage

### 1. Research Crawler

Automatically scrape trending topics from various sources:

```typescript
import { ResearchCrawler } from '@/lib/crawler/research-crawler';

const crawler = new ResearchCrawler({
  sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
  timeframe: '24h',
  keywords: ['AI', 'marketing', 'automation']
});

// Fetch trending topics
const trendingTopics = await crawler.fetchTrends();

// Get detailed articles
const articles = await crawler.getArticleDetails(trendingTopics[0].id);

console.log(articles);
// Output: Array of article objects with title, content, source, publishedAt
```

### 2. AI Content Generation

Generate content using Claude or OpenAI:

```typescript
import { ContentGenerator } from '@/lib/ai/content-generator';

const generator = new ContentGenerator({
  provider: 'claude', // or 'openai'
  model: 'claude-3-sonnet-20240229',
  apiKey: process.env.ANTHROPIC_API_KEY
});

// Generate a blog post
const blogPost = await generator.createContent({
  topic: 'AI Marketing Automation Trends 2024',
  format: 'toplist', // 'pov', 'case-study', 'how-to'
  language: 'en', // or 'vi'
  tone: 'professional', // 'friendly', 'humorous'
  researchData: trendingTopics,
  wordCount: 1500
});

console.log(blogPost);
// Output: { title, content, summary, tags, metadata }
```

### 3. Multi-Language Content

Generate content in multiple languages simultaneously:

```typescript
import { MultiLanguageGenerator } from '@/lib/content/multi-language';

const mlGenerator = new MultiLanguageGenerator({
  primaryLanguage: 'en',
  secondaryLanguages: ['vi']
});

const content = await mlGenerator.generate({
  topic: 'The Future of AI in Marketing',
  format: 'case-study',
  researchData: articles
});

console.log(content.en.title); // English version
console.log(content.vi.title); // Vietnamese version
```

### 4. Video Rendering with Remotion

Convert text content to video automatically:

```typescript
import { VideoRenderer } from '@/lib/video/renderer';
import { bundle } from '@remotion/bundler';
import { renderMedia } from '@remotion/renderer';

const renderer = new VideoRenderer();

// Create video from blog post
const videoConfig = await renderer.createVideoConfig({
  content: blogPost,
  template: 'infographic', // 'talking-head', 'slideshow'
  platform: 'reels', // 'tiktok', 'youtube-shorts'
  duration: 60 // seconds
});

// Render the video
const bundled = await bundle({
  entryPoint: './remotion/index.ts',
  webpackOverride: (config) => config
});

const { outputPath } = await renderMedia({
  composition: videoConfig.composition,
  serveUrl: bundled,
  codec: 'h264',
  outputLocation: `out/video-${Date.now()}.mp4`
});

console.log(`Video rendered: ${outputPath}`);
```

## API Routes (Next.js)

### Generate Content Endpoint

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { ContentGenerator } from '@/lib/ai/content-generator';

export async function POST(request: NextRequest) {
  try {
    const { topic, format, language } = await request.json();
    
    const generator = new ContentGenerator({
      provider: 'claude',
      apiKey: process.env.ANTHROPIC_API_KEY
    });
    
    const content = await generator.createContent({
      topic,
      format,
      language
    });
    
    return NextResponse.json(content);
  } catch (error) {
    return NextResponse.json(
      { error: error.message },
      { status: 500 }
    );
  }
}
```

### Usage in Frontend

```typescript
// src/components/ContentGenerator.tsx
'use client';

import { useState } from 'react';

export function ContentGeneratorForm() {
  const [loading, setLoading] = useState(false);
  
  const handleGenerate = async (e: React.FormEvent) => {
    e.preventDefault();
    setLoading(true);
    
    const response = await fetch('/api/generate', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        topic: 'AI Marketing Trends',
        format: 'toplist',
        language: 'en'
      })
    });
    
    const content = await response.json();
    console.log(content);
    setLoading(false);
  };
  
  return (
    <form onSubmit={handleGenerate}>
      {/* Form fields */}
      <button type="submit" disabled={loading}>
        {loading ? 'Generating...' : 'Generate Content'}
      </button>
    </form>
  );
}
```

## Common Workflow Patterns

### Complete Content Pipeline

End-to-end automation from research to video:

```typescript
import { ContentPipeline } from '@/lib/pipeline';

const pipeline = new ContentPipeline({
  aiProvider: 'claude',
  videoEnabled: true,
  autoPublish: false
});

// Run the complete pipeline
const result = await pipeline.run({
  keyword: 'AI marketing automation',
  steps: [
    'research',      // Crawl trending topics
    'generate',      // Create content
    'translate',     // Multi-language
    'render-video',  // Create videos
    'schedule'       // Queue for publishing
  ],
  config: {
    contentFormat: 'toplist',
    languages: ['en', 'vi'],
    videoFormats: ['reels', 'tiktok'],
    publishAt: new Date('2024-06-15T10:00:00Z')
  }
});

console.log(result);
// Output: {
//   content: { en: {...}, vi: {...} },
//   videos: ['path/to/reels.mp4', 'path/to/tiktok.mp4'],
//   scheduled: true,
//   publishAt: '2024-06-15T10:00:00Z'
// }
```

### Custom Research Sources

Add custom data sources for research:

```typescript
import { CustomSourceCrawler } from '@/lib/crawler/custom-source';

const customCrawler = new CustomSourceCrawler();

// Add RSS feed
customCrawler.addSource({
  type: 'rss',
  url: 'https://example.com/feed.xml',
  name: 'Industry Blog'
});

// Add API endpoint
customCrawler.addSource({
  type: 'api',
  endpoint: 'https://api.example.com/articles',
  headers: {
    'Authorization': `Bearer ${process.env.CUSTOM_API_KEY}`
  },
  parser: (data) => data.articles.map(a => ({
    title: a.headline,
    content: a.body,
    publishedAt: a.date
  }))
});

const research = await customCrawler.fetch();
```

## Configuration

### Content Generation Config

```typescript
// src/config/content.config.ts
export const contentConfig = {
  ai: {
    defaultProvider: 'claude',
    models: {
      claude: 'claude-3-sonnet-20240229',
      openai: 'gpt-4-turbo-preview'
    },
    temperature: 0.7,
    maxTokens: 4000
  },
  formats: {
    toplist: {
      itemCount: 10,
      includeImages: true,
      includeSources: true
    },
    pov: {
      perspective: 'first-person',
      argumentative: true
    },
    caseStudy: {
      includeMetrics: true,
      includeTimeline: true
    }
  },
  languages: {
    supported: ['en', 'vi'],
    default: 'en'
  }
};
```

### Video Rendering Config

```typescript
// remotion/config.ts
export const videoConfig = {
  templates: {
    infographic: {
      fps: 30,
      durationInFrames: 1800, // 60 seconds at 30fps
      width: 1080,
      height: 1920 // Vertical for Reels/TikTok
    },
    slideshow: {
      fps: 30,
      durationInFrames: 2700, // 90 seconds
      width: 1920,
      height: 1080 // Horizontal for YouTube
    }
  },
  platforms: {
    reels: { width: 1080, height: 1920, maxDuration: 90 },
    tiktok: { width: 1080, height: 1920, maxDuration: 180 },
    shorts: { width: 1080, height: 1920, maxDuration: 60 }
  }
};
```

## Running the Development Server

```bash
# Start Next.js dev server
npm run dev

# Open browser
# Navigate to http://localhost:3000
```

## Troubleshooting

### AI Generation Issues

**Problem**: Claude/OpenAI API timeout or rate limit errors

```typescript
// Add retry logic
import { retry } from '@/lib/utils/retry';

const generator = new ContentGenerator({
  provider: 'claude',
  apiKey: process.env.ANTHROPIC_API_KEY,
  retryConfig: {
    maxRetries: 3,
    backoff: 'exponential',
    initialDelay: 1000
  }
});

const content = await retry(
  () => generator.createContent(params),
  { maxAttempts: 3, delay: 2000 }
);
```

### Video Rendering Memory Issues

**Problem**: Out of memory when rendering long videos

```typescript
// Split into chunks
import { VideoRenderer } from '@/lib/video/renderer';

const renderer = new VideoRenderer({
  chunkSize: 30, // Render in 30-second chunks
  parallelRenders: 2 // Limit concurrent renders
});

const video = await renderer.renderChunked({
  content: blogPost,
  platform: 'youtube-shorts'
});
```

### Crawler Rate Limiting

**Problem**: Getting blocked by websites

```typescript
// Add delays and user agent rotation
import { ResearchCrawler } from '@/lib/crawler/research-crawler';

const crawler = new ResearchCrawler({
  rateLimit: {
    requestsPerMinute: 10,
    delayBetweenRequests: 2000 // 2 seconds
  },
  userAgentRotation: true,
  proxy: process.env.PROXY_URL // Optional proxy
});
```

### Environment Variable Not Loading

Ensure `.env.local` is in root directory and variables are prefixed correctly:

```bash
# For Next.js public variables
NEXT_PUBLIC_API_URL=https://api.example.com

# For server-side only
ANTHROPIC_API_KEY=sk-ant-...
OPENAI_API_KEY=sk-...
```

Restart dev server after changing environment variables.

## Advanced Features

### Batch Processing

Process multiple topics in parallel:

```typescript
import { BatchProcessor } from '@/lib/pipeline/batch';

const processor = new BatchProcessor({
  concurrency: 3,
  onProgress: (completed, total) => {
    console.log(`Progress: ${completed}/${total}`);
  }
});

const topics = [
  'AI in Healthcare',
  'Marketing Automation Tools',
  'Social Media Trends 2024'
];

const results = await processor.processAll(topics, async (topic) => {
  return await pipeline.run({ keyword: topic });
});
```

### Custom Video Templates

Create custom Remotion compositions:

```typescript
// remotion/CustomTemplate.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig } from 'remotion';

export const CustomTemplate: React.FC<{ content: string }> = ({ content }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  
  const opacity = Math.min(1, frame / (fps * 0.5)); // Fade in
  
  return (
    <AbsoluteFill style={{ backgroundColor: '#000', opacity }}>
      <h1 style={{ color: '#fff', fontSize: 60 }}>{content}</h1>
    </AbsoluteFill>
  );
};
```

This skill provides comprehensive coverage of the marketing content automation pipeline, enabling AI coding agents to effectively assist developers in building automated content workflows.
