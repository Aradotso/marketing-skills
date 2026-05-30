---
name: ultimate-ai-content-pipeline
description: Automated content creation pipeline from research to video generation using Claude/OpenAI and Remotion
triggers:
  - how do I automate content creation from research to video
  - set up AI content pipeline with Claude and Remotion
  - generate social media videos automatically from articles
  - crawl news sources and create content with AI
  - build automated marketing content workflow
  - create multilingual content with AI and auto-post
  - generate reels and shorts from text content
  - automate content research and script writing
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to help developers use the Ultimate AI Content Pipeline - a comprehensive TypeScript-based automation system that transforms keywords into complete content pieces including research, scripts, and rendered videos. The pipeline integrates Claude 3, OpenAI, web scraping, and Remotion for end-to-end content automation.

## What This Project Does

Ultimate AI Content Pipeline is an all-in-one content automation system that:

- **Auto-scans research sources** - Crawls TechCrunch, a16z, Twitter/X, LinkedIn for latest news within 24h
- **Generates multi-format content** - Creates Toplists, POV pieces, Case Studies, How-tos in multiple languages (EN/VN)
- **Renders videos automatically** - Converts text content into Reels/TikTok/Shorts using Remotion
- **Optimizes for platforms** - Outputs content in formats optimized for different social media platforms
- **Supports multiple AI providers** - Works with OpenAI, Anthropic Claude, and RapidAPI integrations

## Installation

### Prerequisites

```bash
node >= 18.0.0
npm or yarn
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

# Copy environment template
cp .env.example .env
```

### Environment Configuration

Create `.env` file with the following required variables:

```bash
# AI Provider Keys
OPENAI_API_KEY=your_openai_key_here
ANTHROPIC_API_KEY=your_claude_key_here
RAPIDAPI_KEY=your_rapidapi_key_here

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Application Settings
NEXT_PUBLIC_APP_URL=http://localhost:3000
NODE_ENV=development

# Content Research
RESEARCH_DEPTH=deep
RESEARCH_TIMEFRAME=24h

# Video Rendering
REMOTION_OUTPUT_DIR=./public/videos
VIDEO_QUALITY=high
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration modules
│   │   ├── crawler/     # Web scraping utilities
│   │   ├── video/       # Remotion video rendering
│   │   └── utils/       # Helper functions
│   └── types/           # TypeScript type definitions
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Core API & Usage Patterns

### 1. Content Research Module

```typescript
import { researchTopic } from '@/lib/crawler/research';

async function gatherResearch(keyword: string) {
  const research = await researchTopic({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeframe: '24h',
    depth: 'deep'
  });

  return {
    insights: research.insights,
    dataPoints: research.statistics,
    trends: research.trends,
    sources: research.citations
  };
}

// Usage
const aiResearch = await gatherResearch("AI automation trends");
```

### 2. AI Content Generation

```typescript
import { generateContent } from '@/lib/ai/content-generator';

async function createArticle(research: any, format: string) {
  const content = await generateContent({
    provider: 'claude', // or 'openai'
    model: 'claude-3-opus-20240229',
    research,
    format, // 'toplist' | 'pov' | 'case-study' | 'how-to'
    languages: ['en', 'vi'],
    tone: 'professional', // 'professional' | 'friendly' | 'humorous'
    targetAudience: 'marketers'
  });

  return {
    title: content.title,
    body: content.body,
    metadata: content.metadata,
    translations: content.translations
  };
}

// Usage with Claude
const article = await createArticle(aiResearch, 'toplist');
```

### 3. Multi-Provider AI Integration

```typescript
import { AIProvider } from '@/lib/ai/providers';

// Claude provider
const claudeProvider = new AIProvider({
  type: 'anthropic',
  apiKey: process.env.ANTHROPIC_API_KEY,
  model: 'claude-3-opus-20240229'
});

const claudeResponse = await claudeProvider.generate({
  prompt: 'Create a compelling intro for AI automation article',
  maxTokens: 500,
  temperature: 0.7
});

// OpenAI provider
const openaiProvider = new AIProvider({
  type: 'openai',
  apiKey: process.env.OPENAI_API_KEY,
  model: 'gpt-4-turbo-preview'
});

const openaiResponse = await openaiProvider.generate({
  prompt: 'Summarize key trends in marketing automation',
  maxTokens: 300
});
```

### 4. Video Generation with Remotion

```typescript
import { renderVideo } from '@/lib/video/renderer';
import { VideoComposition } from '@/remotion/compositions';

async function generateVideoContent(article: any) {
  const videoData = {
    title: article.title,
    keyPoints: article.keyPoints,
    duration: 60, // seconds
    aspectRatio: '9:16', // for Reels/TikTok/Shorts
    style: 'modern'
  };

  const video = await renderVideo({
    composition: VideoComposition.SocialMediaShort,
    data: videoData,
    outputPath: `${process.env.REMOTION_OUTPUT_DIR}/${article.slug}.mp4`,
    quality: 'high',
    codec: 'h264'
  });

  return video.outputPath;
}

// Usage
const videoPath = await generateVideoContent(article);
```

### 5. Complete Pipeline Workflow

```typescript
import { ContentPipeline } from '@/lib/pipeline';

async function runFullPipeline(keyword: string) {
  const pipeline = new ContentPipeline({
    aiProvider: 'claude',
    videoEnabled: true,
    autoPublish: false
  });

  try {
    // Step 1: Research
    const research = await pipeline.research(keyword);
    
    // Step 2: Generate content in multiple formats
    const contents = await pipeline.generateContent(research, {
      formats: ['toplist', 'case-study'],
      languages: ['en', 'vi']
    });
    
    // Step 3: Render videos
    const videos = await pipeline.renderVideos(contents, {
      platforms: ['reels', 'tiktok', 'youtube-shorts']
    });
    
    // Step 4: Schedule or publish
    const scheduled = await pipeline.schedule(contents, videos, {
      platforms: ['facebook', 'instagram', 'linkedin'],
      publishAt: new Date('2026-06-01T10:00:00Z')
    });

    return {
      research,
      contents,
      videos,
      scheduled
    };
  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}

// Execute full pipeline
const result = await runFullPipeline("AI marketing automation");
```

## Advanced Configuration

### Custom Content Formats

```typescript
import { ContentFormat } from '@/types/content';

const customFormat: ContentFormat = {
  name: 'product-review',
  structure: {
    sections: [
      { type: 'introduction', minLength: 100, maxLength: 200 },
      { type: 'features', items: 5 },
      { type: 'pros-cons', balanced: true },
      { type: 'verdict', minLength: 150 }
    ]
  },
  seo: {
    metaDescription: true,
    keywords: true,
    schema: 'Review'
  }
};

const reviewContent = await generateContent({
  provider: 'claude',
  research,
  customFormat,
  languages: ['en']
});
```

### Web Scraping Configuration

```typescript
import { WebCrawler } from '@/lib/crawler';

const crawler = new WebCrawler({
  sources: [
    {
      name: 'techcrunch',
      url: 'https://techcrunch.com',
      selectors: {
        articles: '.post-block',
        title: 'h2.post-block__title',
        content: '.article-content'
      }
    }
  ],
  rateLimit: {
    requestsPerMinute: 30,
    concurrent: 5
  },
  cache: {
    enabled: true,
    ttl: 3600 // 1 hour
  }
});

const articles = await crawler.scrape({
  keyword: 'AI trends',
  maxResults: 20
});
```

### Remotion Video Templates

```typescript
// remotion/compositions/SocialMediaShort.tsx
import { AbsoluteFill, useCurrentFrame } from 'remotion';

export const SocialMediaShort: React.FC<{
  title: string;
  keyPoints: string[];
}> = ({ title, keyPoints }) => {
  const frame = useCurrentFrame();
  
  return (
    <AbsoluteFill style={{ backgroundColor: '#000' }}>
      <div style={{ 
        color: 'white', 
        fontSize: 48,
        opacity: Math.min(1, frame / 30)
      }}>
        {title}
      </div>
      {keyPoints.map((point, i) => (
        <div key={i} style={{
          opacity: frame > 60 + (i * 30) ? 1 : 0,
          transform: `translateY(${Math.max(0, 60 - (frame - 60 - i * 30))}px)`
        }}>
          {point}
        </div>
      ))}
    </AbsoluteFill>
  );
};
```

## CLI Commands

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Run content pipeline
npm run pipeline -- --keyword "AI automation" --format toplist

# Render videos only
npm run render-videos -- --input ./content/articles

# Export static site
npm run export
```

## Common Patterns

### Error Handling with Retries

```typescript
import { withRetry } from '@/lib/utils/retry';

const contentWithRetry = await withRetry(
  () => generateContent({
    provider: 'claude',
    research,
    format: 'toplist'
  }),
  {
    maxAttempts: 3,
    delayMs: 1000,
    onRetry: (attempt, error) => {
      console.log(`Retry attempt ${attempt}:`, error.message);
    }
  }
);
```

### Batch Processing

```typescript
import { batchProcess } from '@/lib/utils/batch';

const keywords = ['AI automation', 'content marketing', 'video creation'];

const results = await batchProcess(keywords, async (keyword) => {
  const research = await researchTopic({ keyword });
  const content = await generateContent({ research });
  return content;
}, {
  concurrency: 3,
  onProgress: (completed, total) => {
    console.log(`Progress: ${completed}/${total}`);
  }
});
```

### Caching Strategy

```typescript
import { cacheResult } from '@/lib/utils/cache';

const cachedResearch = await cacheResult(
  `research-${keyword}`,
  () => researchTopic({ keyword }),
  { ttl: 3600 } // Cache for 1 hour
);
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement rate limiting
import { RateLimiter } from '@/lib/utils/rate-limiter';

const limiter = new RateLimiter({
  maxRequests: 50,
  windowMs: 60000 // 1 minute
});

await limiter.execute(async () => {
  return await claudeProvider.generate({ prompt });
});
```

### Video Rendering Issues

```bash
# Ensure Remotion dependencies are installed
npm install --save @remotion/cli @remotion/renderer

# Check ffmpeg installation
npx remotion versions

# Render with verbose logging
npm run render-videos -- --log=verbose
```

### Memory Management for Large Batches

```typescript
// Process in chunks to avoid memory issues
import { chunk } from 'lodash';

const keywordChunks = chunk(keywords, 10);

for (const chunkOfKeywords of keywordChunks) {
  await Promise.all(
    chunkOfKeywords.map(keyword => runFullPipeline(keyword))
  );
  // Allow garbage collection between chunks
  await new Promise(resolve => setTimeout(resolve, 1000));
}
```

### API Key Validation

```typescript
import { validateConfig } from '@/lib/utils/config';

try {
  validateConfig({
    OPENAI_API_KEY: process.env.OPENAI_API_KEY,
    ANTHROPIC_API_KEY: process.env.ANTHROPIC_API_KEY,
    RAPIDAPI_KEY: process.env.RAPIDAPI_KEY
  });
} catch (error) {
  console.error('Configuration error:', error.message);
  process.exit(1);
}
```

## Best Practices

1. **Always use environment variables** for API keys and sensitive data
2. **Implement retry logic** for AI API calls due to rate limits
3. **Cache research results** to reduce redundant API calls
4. **Process videos asynchronously** to avoid blocking the main thread
5. **Monitor token usage** to optimize costs with AI providers
6. **Validate content quality** before publishing or scheduling
7. **Use TypeScript strict mode** for type safety across the pipeline
