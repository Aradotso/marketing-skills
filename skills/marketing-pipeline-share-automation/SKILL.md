---
name: marketing-pipeline-share-automation
description: Automated AI content pipeline from research to video generation using Claude, OpenAI, and Remotion
triggers:
  - how do I automate content creation with AI
  - set up marketing content pipeline automation
  - generate videos from blog posts automatically
  - crawl news and create content with AI
  - use Claude and OpenAI for content automation
  - configure Remotion for automated video rendering
  - build AI-powered content research pipeline
  - automate multilingual content generation
---

# Marketing Pipeline Share - AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI agents to work with the Marketing Pipeline Share project - a complete automated content pipeline that handles research, script writing, and video generation using AI (Claude 3, OpenAI) and Remotion.

## What This Project Does

Marketing Pipeline Share is an all-in-one content automation system that:

- **Auto-crawls** fresh content from TechCrunch, a16z, Twitter/X, LinkedIn within the last 24h
- **Generates** blog posts in multiple formats (Toplist, POV, Case Study, How-to) using Claude/OpenAI
- **Multilingual support** with automatic English and Vietnamese content generation
- **Renders videos** automatically using Remotion for social media (Reels, TikTok, Shorts)
- **Customizable tone** (expert, friendly, humorous) based on target audience

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
```

## Environment Configuration

Create a `.env.local` file in the project root:

```bash
# AI API Keys
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Remotion Configuration
REMOTION_AWS_ACCESS_KEY_ID=your_aws_access_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret_key
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI service integrations
│   │   ├── crawler/     # Content crawling logic
│   │   ├── video/       # Remotion video rendering
│   │   └── utils/       # Utility functions
│   └── types/           # TypeScript type definitions
├── public/              # Static assets
└── remotion/            # Remotion video templates
```

## Core API Usage

### 1. Content Research & Crawling

```typescript
import { crawlLatestNews } from '@/lib/crawler/news-crawler';
import { analyzeContent } from '@/lib/ai/content-analyzer';

async function performResearch(keyword: string) {
  // Crawl fresh content from multiple sources
  const newsData = await crawlLatestNews({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeframe: '24h'
  });

  // Analyze and extract insights using AI
  const insights = await analyzeContent({
    data: newsData,
    aiProvider: 'claude', // or 'openai'
    apiKey: process.env.ANTHROPIC_API_KEY
  });

  return insights;
}
```

### 2. AI Content Generation

```typescript
import { generateArticle } from '@/lib/ai/content-generator';

async function createContent(topic: string, format: string) {
  const article = await generateArticle({
    topic,
    format: 'toplist', // 'pov', 'case-study', 'how-to'
    language: ['en', 'vi'], // Generate both English and Vietnamese
    tone: 'expert', // 'friendly', 'humorous'
    aiProvider: 'claude',
    apiKey: process.env.ANTHROPIC_API_KEY,
    model: 'claude-3-opus-20240229'
  });

  return {
    english: article.en,
    vietnamese: article.vi,
    metadata: article.meta
  };
}
```

### 3. Video Rendering with Remotion

```typescript
import { renderVideo } from '@/lib/video/remotion-renderer';
import { bundle } from '@remotion/bundler';
import { renderMedia } from '@remotion/renderer';

async function generateVideoFromArticle(article: Article) {
  // Prepare video composition from article content
  const videoData = {
    title: article.title,
    highlights: article.keyPoints,
    images: article.images,
    duration: 60 // seconds
  };

  // Bundle Remotion composition
  const bundled = await bundle({
    entryPoint: './remotion/index.tsx',
    webpackOverride: (config) => config
  });

  // Render video
  const video = await renderMedia({
    composition: {
      id: 'ContentVideo',
      width: 1080,
      height: 1920, // TikTok/Reels format
      fps: 30,
      durationInFrames: videoData.duration * 30
    },
    serveUrl: bundled,
    codec: 'h264',
    outputLocation: `./output/${article.slug}.mp4`,
    inputProps: videoData
  });

  return video;
}
```

### 4. Complete Pipeline Example

```typescript
import { runContentPipeline } from '@/lib/pipeline';

async function automateContentCreation(keyword: string) {
  const pipeline = await runContentPipeline({
    // Step 1: Research
    research: {
      keyword,
      sources: ['techcrunch', 'a16z', 'linkedin'],
      depth: 'deep' // 'quick', 'medium', 'deep'
    },

    // Step 2: Content Generation
    generation: {
      formats: ['toplist', 'case-study'],
      languages: ['en', 'vi'],
      tone: 'expert',
      aiProvider: 'claude',
      apiKey: process.env.ANTHROPIC_API_KEY
    },

    // Step 3: Video Rendering
    video: {
      enabled: true,
      aspectRatio: '9:16', // TikTok/Reels
      platforms: ['tiktok', 'reels', 'shorts'],
      style: 'infographic'
    },

    // Step 4: Auto-publish (optional)
    publish: {
      enabled: false, // Set to true to auto-post
      platforms: ['facebook', 'linkedin'],
      schedule: new Date('2024-06-15T10:00:00')
    }
  });

  return {
    articles: pipeline.content,
    videos: pipeline.videos,
    analytics: pipeline.stats
  };
}
```

## Common Patterns

### Multi-Language Content Generation

```typescript
import { generateMultilingualContent } from '@/lib/ai/multilingual';

const content = await generateMultilingualContent({
  topic: 'AI Marketing Trends 2024',
  languages: {
    en: {
      tone: 'professional',
      targetAudience: 'marketing executives'
    },
    vi: {
      tone: 'friendly',
      targetAudience: 'startup founders'
    }
  },
  provider: 'claude'
});
```

### Custom Video Templates

```typescript
// remotion/templates/InfographicVideo.tsx
import { AbsoluteFill, Sequence } from 'remotion';

export const InfographicVideo: React.FC<{
  title: string;
  points: string[];
}> = ({ title, points }) => {
  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a1a' }}>
      <Sequence from={0} durationInFrames={60}>
        <h1 style={{ color: 'white', fontSize: 80 }}>{title}</h1>
      </Sequence>
      {points.map((point, index) => (
        <Sequence
          key={index}
          from={60 + index * 90}
          durationInFrames={90}
        >
          <div style={{ color: 'white', fontSize: 50 }}>{point}</div>
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};
```

### Batch Content Processing

```typescript
import { batchProcessKeywords } from '@/lib/pipeline/batch';

const keywords = [
  'AI marketing automation',
  'content pipeline tools',
  'video marketing trends'
];

const results = await batchProcessKeywords({
  keywords,
  maxConcurrent: 3,
  delayBetween: 5000, // 5 seconds
  onProgress: (processed, total) => {
    console.log(`Processed ${processed}/${total} keywords`);
  }
});
```

## CLI Commands

If the project includes CLI tools:

```bash
# Run development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Generate content via CLI (if available)
npm run generate -- --keyword "AI trends" --format toplist

# Render video from existing content
npm run render-video -- --input content.json --output video.mp4
```

## Configuration Options

### AI Provider Configuration

```typescript
// lib/config/ai-config.ts
export const aiConfig = {
  providers: {
    claude: {
      model: 'claude-3-opus-20240229',
      maxTokens: 4000,
      temperature: 0.7
    },
    openai: {
      model: 'gpt-4-turbo-preview',
      maxTokens: 3000,
      temperature: 0.8
    }
  },
  fallback: 'openai' // Use OpenAI if Claude fails
};
```

### Video Rendering Configuration

```typescript
// lib/config/video-config.ts
export const videoConfig = {
  formats: {
    tiktok: { width: 1080, height: 1920, fps: 30 },
    reels: { width: 1080, height: 1920, fps: 30 },
    shorts: { width: 1080, height: 1920, fps: 30 },
    landscape: { width: 1920, height: 1080, fps: 30 }
  },
  codec: 'h264',
  quality: 'high'
};
```

## Troubleshooting

### API Rate Limits

```typescript
import { RateLimiter } from '@/lib/utils/rate-limiter';

const limiter = new RateLimiter({
  maxRequests: 50,
  perInterval: 60000 // 50 requests per minute
});

await limiter.schedule(async () => {
  return await generateArticle(params);
});
```

### Error Handling in Pipeline

```typescript
try {
  const result = await runContentPipeline(config);
} catch (error) {
  if (error.code === 'API_QUOTA_EXCEEDED') {
    // Switch to fallback provider
    config.generation.aiProvider = 'openai';
    return await runContentPipeline(config);
  }
  
  if (error.code === 'RENDER_FAILED') {
    // Retry video rendering with lower quality
    config.video.quality = 'medium';
    return await runContentPipeline(config);
  }
  
  throw error;
}
```

### Memory Issues with Large Video Renders

```typescript
// Increase Node.js memory limit
// In package.json scripts:
{
  "scripts": {
    "render": "NODE_OPTIONS='--max-old-space-size=4096' tsx src/render.ts"
  }
}
```

### Crawling Blocked by Rate Limits

```typescript
import { crawlWithRetry } from '@/lib/crawler/retry';

const data = await crawlWithRetry({
  url: targetUrl,
  maxRetries: 3,
  backoffMs: 2000,
  userAgent: 'Mozilla/5.0...' // Use realistic user agent
});
```

## Best Practices

1. **Always use environment variables** for API keys and secrets
2. **Implement retry logic** for AI API calls (they can be flaky)
3. **Cache research results** to avoid redundant crawling
4. **Monitor token usage** to control AI API costs
5. **Use webhooks** for long-running video renders instead of blocking
6. **Validate input data** before sending to AI providers
7. **Store generated content** in a database for reuse and analytics

## Integration Examples

### With a CMS (e.g., Strapi)

```typescript
import { createEntry } from '@/lib/cms/strapi';

const article = await generateArticle(params);

await createEntry({
  collection: 'articles',
  data: {
    title: article.title,
    content: article.body,
    language: article.language,
    slug: article.slug,
    publishedAt: new Date()
  }
});
```

### With Social Media APIs

```typescript
import { postToSocial } from '@/lib/social/publisher';

await postToSocial({
  platforms: ['facebook', 'linkedin'],
  content: article.summary,
  video: videoPath,
  schedule: scheduledTime
});
```

This skill provides comprehensive guidance for AI coding agents to effectively use the Marketing Pipeline Share project for automated content creation, research, and video generation workflows.
