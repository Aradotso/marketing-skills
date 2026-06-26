---
name: marketing-pipeline-ai-content-automation
description: Automate content creation from research to video generation using AI (Claude, OpenAI) and Remotion for multi-format output
triggers:
  - how do I automate content research and video generation
  - set up AI content pipeline with Claude and OpenAI
  - create automated marketing content workflow
  - generate videos from articles automatically with Remotion
  - build content automation system with research crawling
  - configure AI content generation pipeline
  - automate content from research to video publishing
  - set up multi-language content automation
---

# Marketing Pipeline AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to help developers use the Ultimate AI Content Pipeline - a TypeScript-based system that automates content creation from research gathering through video generation. The pipeline crawls news sources, generates multi-format content using Claude/OpenAI, and renders videos with Remotion.

## What This Project Does

The Marketing Pipeline automates the entire content creation workflow:

1. **Auto Research**: Crawls TechCrunch, a16z, Twitter/X, LinkedIn for recent news (24h window)
2. **AI Content Generation**: Creates articles in multiple formats (toplist, POV, case study, how-to) using Claude 3 or OpenAI
3. **Multi-language Output**: Generates English and Vietnamese versions simultaneously
4. **Video Rendering**: Automatically creates infographics and short videos from content using Remotion
5. **Platform Optimization**: Exports videos optimized for Reels, TikTok, Shorts

## Installation

### Prerequisites

```bash
node >= 18.0.0
npm or yarn
```

### Clone and Setup

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
yarn install

# Copy environment template
cp .env.example .env
```

### Environment Configuration

Edit `.env` with your API keys:

```env
# AI Providers
ANTHROPIC_API_KEY=your_claude_api_key_here
OPENAI_API_KEY=your_openai_api_key_here

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key_here
TWITTER_BEARER_TOKEN=your_twitter_bearer_token_here

# Database (if applicable)
DATABASE_URL=postgresql://user:password@localhost:5432/content_db

# Remotion Config
REMOTION_COMPOSITION_NAME=ContentVideo
REMOTION_OUTPUT_FORMAT=mp4
```

### Run Development Server

```bash
npm run dev
# or
yarn dev
```

Access at `http://localhost:3000`

## Core API Usage

### 1. Content Research Module

```typescript
import { researchContent } from '@/lib/research/crawler';

async function gatherResearch(topic: string) {
  const research = await researchContent({
    topic: topic,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeRange: '24h',
    maxResults: 20
  });
  
  return {
    articles: research.articles,
    insights: research.insights,
    dataPoints: research.statistics
  };
}

// Usage
const data = await gatherResearch('AI automation tools');
console.log(`Found ${data.articles.length} relevant articles`);
```

### 2. AI Content Generation

```typescript
import { generateContent } from '@/lib/ai/generator';
import { ContentFormat, Language } from '@/types';

async function createArticle(
  topic: string, 
  researchData: any,
  format: ContentFormat = 'toplist'
) {
  const content = await generateContent({
    provider: 'claude', // or 'openai'
    model: 'claude-3-opus-20240229',
    topic: topic,
    format: format,
    languages: ['en', 'vi'],
    tone: 'professional', // 'friendly', 'humorous'
    researchContext: researchData,
    wordCount: 1500
  });
  
  return {
    english: content.translations.en,
    vietnamese: content.translations.vi,
    metadata: content.metadata
  };
}

// Generate toplist article
const article = await createArticle(
  'Top 10 AI Tools for Marketing',
  researchData,
  'toplist'
);
```

### 3. Video Generation with Remotion

```typescript
import { renderVideo } from '@/lib/video/remotion-renderer';
import { VideoTemplate } from '@/types/video';

async function generateVideoFromContent(
  content: string,
  template: VideoTemplate = 'infographic'
) {
  const videoConfig = {
    composition: 'ContentVideo',
    template: template,
    content: {
      title: content.title,
      keyPoints: content.highlights,
      statistics: content.dataPoints,
      branding: {
        logo: '/assets/logo.png',
        colors: ['#FF6B6B', '#4ECDC4']
      }
    },
    format: {
      width: 1080,
      height: 1920, // Portrait for Reels/TikTok
      fps: 30,
      durationInFrames: 900 // 30 seconds
    },
    outputFormat: 'mp4'
  };
  
  const video = await renderVideo(videoConfig);
  return video.path;
}

// Render video
const videoPath = await generateVideoFromContent(article.english, 'infographic');
console.log(`Video rendered: ${videoPath}`);
```

## Complete Pipeline Example

```typescript
import { ContentPipeline } from '@/lib/pipeline';

async function runFullPipeline(topic: string) {
  const pipeline = new ContentPipeline({
    aiProvider: process.env.ANTHROPIC_API_KEY ? 'claude' : 'openai',
    languages: ['en', 'vi'],
    generateVideo: true
  });
  
  try {
    // Step 1: Research
    const research = await pipeline.research(topic, {
      sources: ['techcrunch', 'twitter'],
      timeRange: '24h'
    });
    
    // Step 2: Generate content in multiple formats
    const contents = await pipeline.generateMultiFormat(research, {
      formats: ['toplist', 'howto', 'casestudy'],
      wordCount: 1500
    });
    
    // Step 3: Render videos for each format
    const videos = await pipeline.renderVideos(contents, {
      template: 'dynamic',
      platforms: ['reels', 'tiktok', 'shorts']
    });
    
    // Step 4: Export results
    await pipeline.export({
      outputDir: './output',
      includeMetadata: true
    });
    
    return {
      articles: contents.length,
      videos: videos.length,
      outputPath: './output'
    };
    
  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}

// Execute pipeline
runFullPipeline('AI Marketing Automation 2026')
  .then(result => console.log('Pipeline completed:', result));
```

## Custom Research Crawler

```typescript
import { CustomCrawler } from '@/lib/research/custom-crawler';

class MyCustomCrawler extends CustomCrawler {
  async crawlSource(url: string) {
    const response = await fetch(url, {
      headers: {
        'User-Agent': 'ContentPipeline/1.0'
      }
    });
    
    const html = await response.text();
    return this.parseHTML(html);
  }
  
  parseHTML(html: string) {
    // Custom parsing logic
    const articles = [];
    // Extract articles from HTML
    return articles;
  }
}

// Use custom crawler
const crawler = new MyCustomCrawler();
const results = await crawler.crawl(['https://example.com/news']);
```

## CLI Commands

### Generate Content

```bash
# Basic content generation
npm run generate -- --topic "AI Tools" --format toplist

# Multi-language with video
npm run generate -- --topic "Marketing Trends" --languages en,vi --video

# Specify AI provider
npm run generate -- --topic "SEO Tips" --provider claude --model claude-3-opus-20240229
```

### Run Research Only

```bash
npm run research -- --topic "Content Marketing" --sources techcrunch,twitter --days 2
```

### Render Video from Existing Content

```bash
npm run render-video -- --input ./content/article.json --template infographic --output ./videos
```

### Full Pipeline

```bash
npm run pipeline -- --topic "AI Automation" --formats toplist,howto --video --publish
```

## Configuration Patterns

### Format Configurations

```typescript
// config/formats.ts
export const contentFormats = {
  toplist: {
    structure: 'numbered',
    sections: ['intro', 'items', 'conclusion'],
    minItems: 5,
    maxItems: 15
  },
  howto: {
    structure: 'sequential',
    sections: ['intro', 'steps', 'tips', 'conclusion'],
    minSteps: 3
  },
  casestudy: {
    structure: 'narrative',
    sections: ['background', 'challenge', 'solution', 'results'],
    includeMetrics: true
  },
  pov: {
    structure: 'opinion',
    sections: ['hook', 'argument', 'evidence', 'conclusion'],
    tone: 'conversational'
  }
};
```

### AI Provider Configuration

```typescript
// config/ai-providers.ts
export const aiConfig = {
  claude: {
    model: 'claude-3-opus-20240229',
    maxTokens: 4000,
    temperature: 0.7,
    systemPrompt: 'You are an expert content creator...'
  },
  openai: {
    model: 'gpt-4-turbo-preview',
    maxTokens: 4000,
    temperature: 0.7,
    systemPrompt: 'You are an expert content creator...'
  }
};
```

### Video Templates

```typescript
// config/video-templates.ts
export const videoTemplates = {
  infographic: {
    composition: 'InfographicVideo',
    duration: 30,
    transitions: 'fade',
    textStyle: 'bold'
  },
  dynamic: {
    composition: 'DynamicVideo',
    duration: 60,
    transitions: 'slide',
    includeAnimations: true
  },
  minimal: {
    composition: 'MinimalVideo',
    duration: 15,
    transitions: 'none',
    textStyle: 'clean'
  }
};
```

## Troubleshooting

### API Rate Limits

```typescript
import { RateLimiter } from '@/lib/utils/rate-limiter';

const limiter = new RateLimiter({
  requestsPerMinute: 10,
  retryAfter: 60000 // 1 minute
});

async function fetchWithRateLimit(url: string) {
  await limiter.waitForSlot();
  return fetch(url);
}
```

### Claude API Errors

```typescript
try {
  const content = await generateContent({ provider: 'claude', ... });
} catch (error) {
  if (error.message.includes('rate_limit')) {
    console.log('Rate limit hit, waiting...');
    await new Promise(resolve => setTimeout(resolve, 60000));
    // Retry
  } else if (error.message.includes('invalid_api_key')) {
    console.error('Check ANTHROPIC_API_KEY in .env');
  }
}
```

### Remotion Rendering Issues

```bash
# Install required system dependencies for video rendering
sudo apt-get install ffmpeg chromium

# Test Remotion setup
npm run remotion preview
```

```typescript
// Handle rendering errors
import { renderMedia } from '@remotion/renderer';

try {
  await renderMedia({
    composition: compositionId,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: output,
    onProgress: ({ progress }) => {
      console.log(`Rendering: ${Math.round(progress * 100)}%`);
    }
  });
} catch (error) {
  if (error.message.includes('Chrome')) {
    console.error('Chrome/Chromium not found. Install: apt-get install chromium');
  }
  throw error;
}
```

### Memory Issues with Large Crawls

```typescript
// config/crawler.ts
export const crawlerConfig = {
  maxConcurrent: 5, // Limit concurrent requests
  batchSize: 10,    // Process in batches
  timeout: 30000,   // 30 second timeout
  retries: 3
};

// Implement batching
async function crawlInBatches(urls: string[]) {
  const batches = chunk(urls, crawlerConfig.batchSize);
  
  for (const batch of batches) {
    await Promise.all(batch.map(url => crawlUrl(url)));
    // Clear memory between batches
    if (global.gc) global.gc();
  }
}
```

### Database Connection Issues

```typescript
// lib/db/connection.ts
import { Pool } from 'pg';

const pool = new Pool({
  connectionString: process.env.DATABASE_URL,
  max: 20,
  idleTimeoutMillis: 30000,
  connectionTimeoutMillis: 2000,
});

pool.on('error', (err) => {
  console.error('Unexpected database error:', err);
  process.exit(-1);
});

export default pool;
```

## Advanced Patterns

### Custom Content Filters

```typescript
import { ContentFilter } from '@/lib/filters';

const filter = new ContentFilter({
  minQualityScore: 0.7,
  requiredKeywords: ['AI', 'automation'],
  excludeKeywords: ['spam', 'clickbait'],
  minWordCount: 1000
});

const filteredContent = filter.apply(generatedContent);
```

### Scheduling and Queue Management

```typescript
import { ContentQueue } from '@/lib/queue';

const queue = new ContentQueue({
  maxConcurrent: 3,
  retryAttempts: 3
});

queue.add('research', { topic: 'AI Tools' });
queue.add('generate', { researchId: 'abc123' });
queue.add('render', { contentId: 'xyz789' });

queue.on('completed', (job) => {
  console.log(`Job ${job.id} completed`);
});

await queue.process();
```

This skill provides comprehensive coverage of the Marketing Pipeline AI Content Automation system, enabling AI coding agents to effectively assist developers in implementing automated content workflows.
