---
name: marketing-pipeline-share-ai-content
description: Automated AI content pipeline for research, scriptwriting, and video generation using Claude/OpenAI
triggers:
  - "set up automated content pipeline"
  - "generate content from research to video"
  - "automate content creation with AI"
  - "create marketing content pipeline"
  - "generate videos from articles automatically"
  - "build AI-powered content workflow"
  - "scrape and generate content with Claude"
  - "automate social media content production"
---

# Marketing Pipeline Share - AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This project is an end-to-end AI-powered content automation pipeline that handles research, scriptwriting, article generation, and video rendering. It crawls news sources, generates content in multiple formats using Claude/OpenAI, and automatically renders videos using Remotion.

## What It Does

- **Auto-Research**: Crawls TechCrunch, a16z, Twitter/X, LinkedIn for trending content within 24h
- **AI Content Generation**: Creates articles in multiple formats (toplist, POV, case study, how-to) using Claude 3 or OpenAI
- **Multi-language**: Generates content in both English and Vietnamese with customizable tone
- **Video Rendering**: Automatically renders infographics and short-form videos from articles using Remotion
- **Platform Optimization**: Exports videos optimized for Reels, TikTok, and Shorts

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

### Environment Configuration

```bash
# .env file
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_url

# Next.js
NEXT_PUBLIC_API_URL=http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/              # Core utilities
│   │   ├── ai/           # AI integration (Claude, OpenAI)
│   │   ├── scraper/      # Content scraping logic
│   │   ├── generator/    # Content generation
│   │   └── video/        # Remotion video rendering
│   ├── api/              # API routes
│   └── types/            # TypeScript types
├── public/               # Static assets
└── remotion/             # Remotion video templates
```

## Core Usage

### 1. Content Research & Scraping

```typescript
import { ContentScraper } from '@/lib/scraper';

// Initialize scraper
const scraper = new ContentScraper({
  sources: ['techcrunch', 'a16z', 'twitter'],
  timeframe: '24h'
});

// Scrape content by keyword
const results = await scraper.scrapeByKeyword('AI marketing tools');

console.log(results);
// {
//   articles: [...],
//   insights: [...],
//   trends: [...]
// }
```

### 2. AI Content Generation

```typescript
import { ContentGenerator } from '@/lib/generator';
import { Anthropic } from '@anthropic-ai/sdk';

// Initialize with Claude
const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY
});

const generator = new ContentGenerator({
  client: anthropic,
  model: 'claude-3-opus-20240229'
});

// Generate article from research
const article = await generator.generate({
  topic: 'AI Marketing Trends 2024',
  format: 'toplist',
  language: 'en',
  tone: 'professional',
  researchData: results
});

console.log(article);
// {
//   title: "Top 10 AI Marketing Trends...",
//   content: "...",
//   metadata: {...}
// }
```

### 3. Multi-Format Content Generation

```typescript
import { FormatGenerator } from '@/lib/generator/formats';

const formatGen = new FormatGenerator();

// Generate different formats
const formats = await formatGen.generateAll({
  topic: 'Content Marketing Automation',
  research: results,
  formats: ['toplist', 'pov', 'case-study', 'how-to']
});

// Access specific format
const toplist = formats.toplist;
const caseStudy = formats['case-study'];
```

### 4. Bilingual Content Generation

```typescript
import { BilingualGenerator } from '@/lib/generator';

const bilingualGen = new BilingualGenerator({
  primaryLanguage: 'en',
  secondaryLanguage: 'vi'
});

const content = await bilingualGen.generate({
  topic: 'Social Media Strategy',
  format: 'how-to',
  researchData: results
});

console.log(content.en); // English version
console.log(content.vi); // Vietnamese version
```

### 5. Video Rendering with Remotion

```typescript
import { renderVideo } from '@/lib/video';
import { ArticleToVideo } from '@/remotion/compositions';

// Render video from article
const videoPath = await renderVideo({
  composition: ArticleToVideo,
  props: {
    title: article.title,
    content: article.content,
    style: 'infographic'
  },
  outputFormat: 'reels', // 'reels' | 'tiktok' | 'shorts'
  outputPath: './output/video.mp4'
});

console.log(`Video rendered: ${videoPath}`);
```

## API Routes

### Generate Content Endpoint

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { ContentPipeline } from '@/lib/pipeline';

export async function POST(req: NextRequest) {
  const { keyword, format, language } = await req.json();

  const pipeline = new ContentPipeline();
  
  const result = await pipeline.run({
    keyword,
    format,
    language,
    includeVideo: true
  });

  return NextResponse.json(result);
}
```

### Using the API

```typescript
// Client-side usage
const response = await fetch('/api/generate', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    keyword: 'AI content tools',
    format: 'toplist',
    language: 'en'
  })
});

const { article, video } = await response.json();
```

## Complete Pipeline Example

```typescript
import { ContentPipeline } from '@/lib/pipeline';

// Initialize full pipeline
const pipeline = new ContentPipeline({
  scraper: {
    sources: ['techcrunch', 'a16z', 'twitter'],
    timeframe: '24h'
  },
  generator: {
    provider: 'claude', // or 'openai'
    model: 'claude-3-opus-20240229'
  },
  video: {
    enabled: true,
    formats: ['reels', 'tiktok']
  }
});

// Run complete pipeline
const output = await pipeline.execute({
  keyword: 'marketing automation',
  contentFormat: 'case-study',
  languages: ['en', 'vi'],
  tone: 'professional'
});

console.log(output);
// {
//   research: {...},
//   articles: {
//     en: {...},
//     vi: {...}
//   },
//   videos: [
//     { format: 'reels', path: '...' },
//     { format: 'tiktok', path: '...' }
//   ]
// }
```

## Custom Tone Configuration

```typescript
import { ToneConfig } from '@/lib/generator/tone';

const customTone: ToneConfig = {
  name: 'casual-expert',
  description: 'Expert knowledge with friendly delivery',
  systemPrompt: `
    Write as an industry expert who explains complex topics 
    in simple, approachable language. Use examples and analogies.
  `,
  temperature: 0.7
};

const generator = new ContentGenerator({
  tone: customTone
});
```

## Scheduling Content

```typescript
import { ContentScheduler } from '@/lib/scheduler';

const scheduler = new ContentScheduler();

// Schedule daily content generation
scheduler.schedule({
  frequency: 'daily',
  time: '09:00',
  config: {
    keywords: ['AI', 'marketing', 'automation'],
    format: 'toplist',
    autoPost: true,
    platforms: ['facebook', 'linkedin']
  }
});
```

## Troubleshooting

### API Rate Limits

```typescript
import { RateLimiter } from '@/lib/utils/rate-limit';

const limiter = new RateLimiter({
  requestsPerMinute: 50,
  provider: 'anthropic'
});

// Wrap API calls
const result = await limiter.execute(async () => {
  return await generator.generate({...});
});
```

### Error Handling

```typescript
import { PipelineError } from '@/lib/errors';

try {
  const result = await pipeline.execute({...});
} catch (error) {
  if (error instanceof PipelineError) {
    console.error('Pipeline stage:', error.stage);
    console.error('Error:', error.message);
    
    // Retry logic
    if (error.retryable) {
      await pipeline.retry(error.stage);
    }
  }
}
```

### Video Rendering Issues

```typescript
// Check Remotion configuration
import { getVideoMetadata } from '@remotion/renderer';

const metadata = await getVideoMetadata({
  composition: ArticleToVideo,
  props: {...}
});

console.log('Duration:', metadata.durationInFrames);
console.log('FPS:', metadata.fps);
```

## Development Commands

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Run Remotion Studio (video preview)
npm run remotion:studio

# Render video directly
npm run remotion:render
```

## Common Patterns

### Batch Content Generation

```typescript
const topics = ['AI tools', 'Marketing tips', 'Growth hacks'];

const batchResults = await Promise.all(
  topics.map(topic => 
    pipeline.execute({
      keyword: topic,
      contentFormat: 'toplist',
      languages: ['en']
    })
  )
);
```

### Custom Scraper Integration

```typescript
import { CustomScraper } from '@/lib/scraper/custom';

class LinkedInScraper extends CustomScraper {
  async scrape(keyword: string) {
    // Custom LinkedIn scraping logic
    const posts = await this.fetchLinkedInPosts(keyword);
    return this.parseResults(posts);
  }
}

const scraper = new ContentScraper({
  custom: [new LinkedInScraper()]
});
```

This skill provides comprehensive automation for marketing content creation, from research to final video assets.
