---
name: marketing-pipeline-share-automation
description: AI-powered content automation pipeline from research to video generation using Claude, OpenAI, and Remotion
triggers:
  - automate content creation with AI research and video
  - set up marketing pipeline with automated posting
  - generate content from research to video automatically
  - build AI content workflow with Claude and OpenAI
  - create automated video content from articles
  - implement content automation pipeline with Remotion
  - use marketing pipeline share for content generation
  - automate social media content with AI research
---

# Marketing Pipeline Share Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This project is an end-to-end AI content automation system that transforms keywords into complete content packages: research, scripting, article generation, and video rendering. It crawls recent news from sources like TechCrunch, a16z, Twitter/X, and LinkedIn, then uses AI (Claude 3, OpenAI) to generate multi-format content in multiple languages, and finally renders videos using Remotion.

## What It Does

The **Ultimate AI Content Pipeline** automates the entire content creation workflow:

1. **Auto-Research**: Crawls and analyzes real-time data from major news sources within the last 24 hours
2. **AI Content Generation**: Creates articles in various formats (Toplist, POV, Case Study, How-to) using Claude/OpenAI
3. **Multi-language Support**: Generates content in both English and Vietnamese with customizable tone
4. **Video Rendering**: Automatically creates infographics and short videos from article content using Remotion
5. **Platform Optimization**: Exports videos in formats suitable for Reels, TikTok, Shorts

## Installation

### Prerequisites

```bash
# Required
node >= 18.x
npm or yarn or pnpm
```

### Setup

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

### Environment Configuration

Create a `.env.local` file in the root directory:

```bash
# AI API Keys
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research/Crawling APIs
RAPIDAPI_KEY=your_rapidapi_key

# Optional: Database
DATABASE_URL=postgresql://user:password@localhost:5432/content_pipeline

# Optional: Social Media Posting
FACEBOOK_ACCESS_TOKEN=your_fb_token
LINKEDIN_ACCESS_TOKEN=your_linkedin_token

# Remotion Video Config
REMOTION_OUTPUT_DIR=./output/videos
```

### Start Development Server

```bash
npm run dev
# or
yarn dev
# or
pnpm dev
```

The application will be available at `http://localhost:3000`

## Key Commands

### Development

```bash
# Start Next.js dev server
npm run dev

# Build for production
npm run build

# Start production server
npm run start

# Run linter
npm run lint
```

### Content Generation

```bash
# Run research pipeline (if CLI exists)
npm run research -- --keyword "AI automation"

# Generate content from research
npm run generate-content -- --format toplist --lang en

# Render video from article
npm run render-video -- --article-id 123
```

### Remotion Commands

```bash
# Preview Remotion compositions
npm run remotion:preview

# Render a specific composition
npm run remotion:render -- --composition=ArticleVideo --props='{"articleId":"123"}'

# List all compositions
npm run remotion:compositions
```

## Architecture & Core Modules

### Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── crawler/     # News crawling modules
│   │   ├── content/     # Content generation logic
│   │   └── video/       # Remotion video rendering
│   ├── remotion/         # Remotion video templates
│   └── utils/           # Shared utilities
├── public/              # Static assets
└── output/              # Generated content & videos
```

## Core API Usage

### 1. Research & Crawling

```typescript
import { ResearchEngine } from '@/lib/crawler/research-engine';

async function performResearch(keyword: string) {
  const engine = new ResearchEngine({
    apiKey: process.env.RAPIDAPI_KEY,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeRange: '24h'
  });

  const results = await engine.search({
    keyword,
    limit: 20,
    includeInsights: true
  });

  return results;
}

// Example: Get AI automation news
const research = await performResearch('AI automation');
console.log(research.articles);
console.log(research.insights);
```

### 2. AI Content Generation with Claude

```typescript
import { ContentGenerator } from '@/lib/ai/content-generator';
import Anthropic from '@anthropic-ai/sdk';

async function generateArticle(
  research: any,
  format: 'toplist' | 'pov' | 'casestudy' | 'howto'
) {
  const anthropic = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY
  });

  const generator = new ContentGenerator(anthropic);

  const article = await generator.create({
    format,
    research,
    language: 'en',
    tone: 'professional',
    targetLength: 1500
  });

  return article;
}

// Example: Generate toplist article
const article = await generateArticle(research, 'toplist');
```

### 3. Multi-language Content Generation

```typescript
import { MultiLanguageGenerator } from '@/lib/content/multi-lang';

async function generateBilingualContent(research: any) {
  const mlGen = new MultiLanguageGenerator({
    claudeKey: process.env.ANTHROPIC_API_KEY,
    openaiKey: process.env.OPENAI_API_KEY
  });

  const content = await mlGen.generateParallel({
    research,
    languages: ['en', 'vi'],
    format: 'pov',
    tone: 'friendly'
  });

  return {
    english: content.en,
    vietnamese: content.vi
  };
}
```

### 4. Video Rendering with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import { ArticleVideoComposition } from '@/remotion/ArticleVideo';

async function renderArticleVideo(article: any) {
  const bundled = await bundle({
    entryPoint: './src/remotion/index.ts',
    webpackOverride: (config) => config
  });

  const composition = await selectComposition({
    serveUrl: bundled,
    id: 'ArticleVideo',
    inputProps: {
      title: article.title,
      points: article.keyPoints,
      duration: 60
    }
  });

  await renderMedia({
    composition,
    serveUrl: bundled,
    codec: 'h264',
    outputLocation: `${process.env.REMOTION_OUTPUT_DIR}/${article.id}.mp4`,
    inputProps: composition.inputProps
  });

  return `${article.id}.mp4`;
}
```

### 5. Full Pipeline Example

```typescript
import { ContentPipeline } from '@/lib/content/pipeline';

async function runFullPipeline(keyword: string) {
  const pipeline = new ContentPipeline({
    anthropicKey: process.env.ANTHROPIC_API_KEY,
    openaiKey: process.env.OPENAI_API_KEY,
    rapidApiKey: process.env.RAPIDAPI_KEY
  });

  // Step 1: Research
  const research = await pipeline.research(keyword);

  // Step 2: Generate content in both languages
  const content = await pipeline.generateContent({
    research,
    formats: ['toplist', 'pov'],
    languages: ['en', 'vi']
  });

  // Step 3: Render videos
  const videos = await pipeline.renderVideos({
    content,
    platforms: ['reels', 'tiktok', 'shorts']
  });

  // Step 4: Optional - Auto-post
  if (process.env.FACEBOOK_ACCESS_TOKEN) {
    await pipeline.publishToSocial({
      content,
      videos,
      platforms: ['facebook', 'linkedin']
    });
  }

  return {
    research,
    content,
    videos
  };
}

// Execute pipeline
const result = await runFullPipeline('marketing automation');
```

## Common Patterns

### Custom Content Templates

```typescript
import { TemplateManager } from '@/lib/content/templates';

const templateManager = new TemplateManager();

// Define custom template
templateManager.register('viral-thread', {
  structure: [
    { type: 'hook', maxLength: 280 },
    { type: 'problem', maxLength: 280 },
    { type: 'solution', maxLength: 280 },
    { type: 'cta', maxLength: 280 }
  ],
  tone: 'engaging',
  platform: 'twitter'
});

// Use template
const thread = await templateManager.generate('viral-thread', {
  research: myResearch
});
```

### Custom Video Compositions

```typescript
// src/remotion/CustomVideo.tsx
import { AbsoluteFill, useCurrentFrame, interpolate } from 'remotion';

export const CustomVideo: React.FC<{
  title: string;
  points: string[];
}> = ({ title, points }) => {
  const frame = useCurrentFrame();
  const opacity = interpolate(frame, [0, 30], [0, 1]);

  return (
    <AbsoluteFill style={{ backgroundColor: '#000' }}>
      <h1 style={{ opacity, color: '#fff' }}>{title}</h1>
      {points.map((point, i) => (
        <p key={i} style={{ color: '#fff' }}>{point}</p>
      ))}
    </AbsoluteFill>
  );
};
```

### Scheduling Content

```typescript
import { ContentScheduler } from '@/lib/content/scheduler';

const scheduler = new ContentScheduler();

// Schedule content generation
await scheduler.schedule({
  keyword: 'AI trends',
  frequency: 'daily',
  time: '09:00',
  timezone: 'UTC',
  autoPost: true
});

// List scheduled jobs
const jobs = await scheduler.listJobs();
```

## Configuration

### AI Model Selection

```typescript
// config/ai.ts
export const AI_CONFIG = {
  defaultModel: 'claude-3-5-sonnet-20241022',
  fallbackModel: 'gpt-4-turbo',
  maxTokens: 4096,
  temperature: 0.7,
  researchModel: 'claude-3-haiku-20240307' // Faster for research
};
```

### Content Format Presets

```typescript
// config/formats.ts
export const CONTENT_FORMATS = {
  toplist: {
    minItems: 5,
    maxItems: 10,
    includeIntro: true,
    includeConclusion: true
  },
  pov: {
    perspective: 'first-person',
    includePersonalExperience: true
  },
  casestudy: {
    sections: ['background', 'challenge', 'solution', 'results'],
    includeMetrics: true
  },
  howto: {
    stepByStep: true,
    includeVisuals: true,
    difficultyLevel: 'beginner'
  }
};
```

### Video Export Settings

```typescript
// config/video.ts
export const VIDEO_PRESETS = {
  reels: {
    width: 1080,
    height: 1920,
    fps: 30,
    codec: 'h264'
  },
  tiktok: {
    width: 1080,
    height: 1920,
    fps: 30,
    codec: 'h264'
  },
  shorts: {
    width: 1080,
    height: 1920,
    fps: 30,
    codec: 'h264'
  },
  landscape: {
    width: 1920,
    height: 1080,
    fps: 30,
    codec: 'h264'
  }
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

await limiter.execute(async () => {
  return await anthropic.messages.create({...});
});
```

### Crawler Timeout Issues

```typescript
import { ResearchEngine } from '@/lib/crawler/research-engine';

const engine = new ResearchEngine({
  timeout: 30000, // Increase timeout to 30s
  retries: 3,
  retryDelay: 2000
});
```

### Video Rendering Memory Issues

```bash
# Increase Node memory limit
NODE_OPTIONS="--max-old-space-size=4096" npm run remotion:render
```

### Missing Dependencies

```bash
# Ensure all Remotion dependencies are installed
npm install @remotion/bundler @remotion/renderer @remotion/cli

# For video encoding
npm install @remotion/lambda
```

### Content Quality Issues

```typescript
// Adjust AI parameters for better quality
const article = await generator.create({
  format: 'toplist',
  research,
  temperature: 0.5, // Lower for more focused content
  maxRetries: 3,
  qualityCheck: true // Enable quality validation
});
```

## Advanced Usage

### Custom Research Sources

```typescript
import { CustomCrawler } from '@/lib/crawler/custom';

class RedditCrawler extends CustomCrawler {
  async crawl(subreddit: string) {
    // Custom Reddit crawling logic
    return this.parse(await this.fetch(`/r/${subreddit}`));
  }
}

// Add to research engine
engine.addSource(new RedditCrawler());
```

### Webhook Integration

```typescript
import { WebhookManager } from '@/lib/integrations/webhooks';

const webhooks = new WebhookManager();

// Trigger webhook on content generation
webhooks.on('content:generated', async (content) => {
  await fetch(process.env.WEBHOOK_URL, {
    method: 'POST',
    body: JSON.stringify(content)
  });
});
```

This skill enables AI coding agents to help developers implement comprehensive content automation pipelines with research, AI generation, and video rendering capabilities.
