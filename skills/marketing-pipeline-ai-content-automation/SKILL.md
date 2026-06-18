---
name: marketing-pipeline-ai-content-automation
description: Ultimate AI content pipeline for automated research, scriptwriting, video generation, and multi-platform publishing using Claude, OpenAI, and Remotion
triggers:
  - automate my content creation pipeline
  - set up AI content research and generation
  - build automated marketing content system
  - create content with AI from research to video
  - generate social media content automatically
  - automate blog posts and video creation
  - set up content pipeline with Claude and OpenAI
  - create multi-format content with AI automation
---

# Marketing Pipeline AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with **Ultimate AI Content Pipeline**, a comprehensive TypeScript-based system that automates the entire content creation workflow from research to video generation and publishing.

## What This Project Does

The Marketing Pipeline automates:
- **Auto-Scan Research**: Crawls news sources (TechCrunch, a16z, Twitter, LinkedIn) for fresh data
- **AI Content Generation**: Creates multi-format content (toplist, POV, case study, how-to) using Claude 3 and OpenAI
- **Multi-Language Support**: Generates content in English and Vietnamese simultaneously
- **Video Rendering**: Converts text content to video/infographics using Remotion
- **Auto-Publishing**: Schedules and posts content to social platforms

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
# AI Services
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_url

# Video Rendering
REMOTION_LICENSE_KEY=your_remotion_license

# Social Platform APIs (optional)
FACEBOOK_ACCESS_TOKEN=your_fb_token
TWITTER_API_KEY=your_twitter_key
```

### Start Development Server

```bash
npm run dev
# or
yarn dev
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app router pages
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI service integrations
│   │   ├── crawler/     # Content research crawlers
│   │   ├── generator/   # Content generation logic
│   │   └── video/       # Remotion video rendering
│   └── types/           # TypeScript type definitions
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Core APIs and Usage

### 1. Content Research & Crawling

```typescript
import { ContentCrawler } from '@/lib/crawler';

// Initialize crawler
const crawler = new ContentCrawler({
  sources: ['techcrunch', 'a16z', 'twitter'],
  timeRange: '24h'
});

// Fetch latest trends
async function researchTopic(keyword: string) {
  const results = await crawler.scan({
    keyword,
    maxResults: 20,
    filters: {
      language: 'en',
      minEngagement: 100
    }
  });

  return results;
}

// Example usage
const aiTrends = await researchTopic('artificial intelligence');
console.log(aiTrends.insights);
```

### 2. AI Content Generation with Claude/OpenAI

```typescript
import { ContentGenerator } from '@/lib/generator';
import { Anthropic } from '@anthropic-ai/sdk';
import OpenAI from 'openai';

// Initialize AI clients
const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY
});

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY
});

// Generate content in multiple formats
async function generateContent(
  topic: string,
  format: 'toplist' | 'pov' | 'case-study' | 'how-to',
  language: 'en' | 'vi'
) {
  const generator = new ContentGenerator({ anthropic, openai });

  const content = await generator.create({
    topic,
    format,
    language,
    tone: 'professional', // or 'friendly', 'humorous'
    includeData: true,
    sources: await researchTopic(topic)
  });

  return content;
}

// Generate bilingual content
async function generateBilingual(topic: string) {
  const [enContent, viContent] = await Promise.all([
    generateContent(topic, 'toplist', 'en'),
    generateContent(topic, 'toplist', 'vi')
  ]);

  return { en: enContent, vi: viContent };
}
```

### 3. Advanced Content Generation with Templates

```typescript
import { AIContentPipeline } from '@/lib/ai/pipeline';

const pipeline = new AIContentPipeline({
  anthropicKey: process.env.ANTHROPIC_API_KEY!,
  openaiKey: process.env.OPENAI_API_KEY!
});

// Generate POV article
async function generatePOVArticle(topic: string) {
  const article = await pipeline.generate({
    type: 'pov',
    topic,
    parameters: {
      perspective: 'expert',
      wordCount: 1500,
      includeStats: true,
      citeSources: true,
      seoOptimized: true
    }
  });

  return {
    title: article.title,
    content: article.body,
    meta: article.seoMeta,
    hashtags: article.socialHashtags
  };
}

// Generate case study with data
async function generateCaseStudy(company: string, topic: string) {
  const caseStudy = await pipeline.generate({
    type: 'case-study',
    topic: `${company} ${topic}`,
    parameters: {
      structure: ['problem', 'solution', 'results', 'takeaways'],
      includeMetrics: true,
      visualData: true
    }
  });

  return caseStudy;
}
```

### 4. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import { VideoTemplate } from '@/remotion/templates';

async function generateVideo(content: any, format: 'reels' | 'tiktok' | 'shorts') {
  // Aspect ratios for different platforms
  const dimensions = {
    reels: { width: 1080, height: 1920 },
    tiktok: { width: 1080, height: 1920 },
    shorts: { width: 1080, height: 1920 }
  };

  const bundleLocation = await bundle({
    entryPoint: './remotion/index.ts',
    webpackOverride: (config) => config
  });

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      content: content.body,
      title: content.title,
      style: 'modern',
      ...dimensions[format]
    }
  });

  const outputPath = `./output/video-${Date.now()}.mp4`;

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath
  });

  return outputPath;
}

// Generate video from article
async function articleToVideo(article: any) {
  const videos = await Promise.all([
    generateVideo(article, 'reels'),
    generateVideo(article, 'tiktok'),
    generateVideo(article, 'shorts')
  ]);

  return videos;
}
```

### 5. Complete Content Pipeline

```typescript
import { FullPipeline } from '@/lib/pipeline';

// End-to-end content creation
async function createCompleteContent(keyword: string) {
  const pipeline = new FullPipeline({
    anthropicKey: process.env.ANTHROPIC_API_KEY!,
    openaiKey: process.env.OPENAI_API_KEY!,
    rapidApiKey: process.env.RAPIDAPI_KEY!
  });

  // Step 1: Research
  const research = await pipeline.research(keyword);

  // Step 2: Generate content in multiple formats
  const content = await pipeline.generateMultiFormat({
    research,
    formats: ['toplist', 'how-to', 'pov'],
    languages: ['en', 'vi']
  });

  // Step 3: Generate videos
  const videos = await pipeline.renderVideos(content, {
    platforms: ['reels', 'tiktok', 'shorts']
  });

  // Step 4: Schedule publishing (optional)
  await pipeline.schedulePublish({
    content,
    videos,
    platforms: ['facebook', 'twitter', 'linkedin'],
    schedule: new Date(Date.now() + 86400000) // 24h later
  });

  return {
    articles: content,
    videos,
    analytics: await pipeline.getAnalytics()
  };
}

// Usage
const result = await createCompleteContent('AI automation trends 2026');
```

## Configuration

### AI Model Settings

```typescript
// lib/config/ai.ts
export const AI_CONFIG = {
  claude: {
    model: 'claude-3-opus-20240229',
    maxTokens: 4096,
    temperature: 0.7
  },
  openai: {
    model: 'gpt-4-turbo-preview',
    maxTokens: 4096,
    temperature: 0.7
  }
};
```

### Content Templates

```typescript
// lib/config/templates.ts
export const CONTENT_TEMPLATES = {
  toplist: {
    structure: ['intro', 'items', 'conclusion'],
    itemCount: { min: 5, max: 10 },
    includeImages: true
  },
  pov: {
    structure: ['hook', 'perspective', 'arguments', 'conclusion'],
    tone: 'thought-leadership',
    citations: true
  },
  'case-study': {
    structure: ['background', 'challenge', 'solution', 'results', 'takeaways'],
    includeMetrics: true,
    visualizeData: true
  },
  'how-to': {
    structure: ['intro', 'prerequisites', 'steps', 'tips', 'conclusion'],
    stepFormat: 'numbered',
    includeScreenshots: false
  }
};
```

### Crawler Configuration

```typescript
// lib/config/crawler.ts
export const CRAWLER_CONFIG = {
  sources: {
    techcrunch: {
      url: 'https://techcrunch.com/feed/',
      selector: 'article',
      rateLimit: 100
    },
    a16z: {
      url: 'https://a16z.com/feed/',
      selector: '.post',
      rateLimit: 50
    },
    twitter: {
      endpoint: 'https://api.twitter.com/2/tweets/search/recent',
      maxResults: 100
    }
  },
  caching: {
    enabled: true,
    ttl: 3600 // 1 hour
  }
};
```

## Common Patterns

### Pattern 1: Research-to-Article Workflow

```typescript
async function researchToArticle(topic: string, targetAudience: string) {
  // 1. Gather research
  const research = await researchTopic(topic);

  // 2. Analyze insights
  const insights = research.insights.filter(
    i => i.relevance > 0.7
  );

  // 3. Generate tailored content
  const article = await generateContent(topic, 'toplist', 'en');

  // 4. Optimize for SEO
  const optimized = await optimizeForSEO(article, {
    keyword: topic,
    audience: targetAudience
  });

  return optimized;
}
```

### Pattern 2: Multi-Platform Video Distribution

```typescript
async function distributeVideoContent(article: any) {
  // Generate platform-specific videos
  const videoConfigs = [
    { platform: 'reels', duration: 60, style: 'fast-paced' },
    { platform: 'tiktok', duration: 30, style: 'trendy' },
    { platform: 'shorts', duration: 45, style: 'educational' }
  ];

  const videos = await Promise.all(
    videoConfigs.map(config =>
      generateVideo(article, config.platform)
    )
  );

  return videos;
}
```

### Pattern 3: Scheduled Content Calendar

```typescript
import { ContentCalendar } from '@/lib/scheduler';

async function createContentCalendar(topics: string[], days: number) {
  const calendar = new ContentCalendar();

  for (let i = 0; i < days; i++) {
    const topic = topics[i % topics.length];
    const publishDate = new Date(Date.now() + i * 86400000);

    await calendar.schedule({
      topic,
      date: publishDate,
      formats: ['article', 'video'],
      platforms: ['facebook', 'linkedin', 'twitter']
    });
  }

  return calendar.export();
}
```

## Troubleshooting

### API Rate Limits

```typescript
import { RateLimiter } from '@/lib/utils/rate-limiter';

const limiter = new RateLimiter({
  anthropic: { requestsPerMinute: 50 },
  openai: { requestsPerMinute: 60 },
  rapidapi: { requestsPerMinute: 100 }
});

// Use with automatic retry
async function safeAPICall(fn: () => Promise<any>, service: string) {
  return limiter.execute(service, fn);
}
```

### Video Rendering Memory Issues

```typescript
// Optimize Remotion rendering for large projects
export const REMOTION_CONFIG = {
  concurrency: 2, // Reduce if out of memory
  chromiumOptions: {
    args: ['--max-old-space-size=4096']
  }
};
```

### Content Quality Checks

```typescript
import { ContentValidator } from '@/lib/validator';

async function validateContent(content: any) {
  const validator = new ContentValidator();

  const checks = await validator.run(content, {
    minWordCount: 800,
    readabilityScore: 60,
    plagiarismCheck: true,
    factCheck: true
  });

  if (!checks.passed) {
    throw new Error(`Content validation failed: ${checks.errors.join(', ')}`);
  }

  return checks;
}
```

### Debugging AI Responses

```typescript
// Enable verbose logging
const generator = new ContentGenerator({
  debug: true,
  logPrompts: true,
  logResponses: true
});

// Log token usage
generator.on('completion', (data) => {
  console.log('Tokens used:', data.usage);
  console.log('Cost estimate:', data.cost);
});
```

## CLI Commands (if applicable)

```bash
# Generate content from CLI
npm run generate -- --topic "AI trends" --format toplist --lang en

# Crawl sources
npm run crawl -- --sources techcrunch,a16z --hours 24

# Render video
npm run render -- --input article.json --format reels

# Schedule posts
npm run schedule -- --calendar content-calendar.json

# Export analytics
npm run analytics -- --export --format csv
```

## Best Practices

1. **Always validate research sources** before generating content
2. **Use rate limiting** to avoid API quota exhaustion
3. **Cache research results** to reduce API calls
4. **Test video templates** before bulk rendering
5. **Monitor content quality** with automated validators
6. **Schedule content** during peak engagement hours
7. **Track analytics** to optimize future content

This skill enables complete automation of the content marketing workflow using cutting-edge AI and video generation technology.
