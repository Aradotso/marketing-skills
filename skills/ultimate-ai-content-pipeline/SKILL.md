---
name: ultimate-ai-content-pipeline
description: Automated content pipeline from research to video generation using AI (Claude/OpenAI) and Remotion for Vietnamese and English content creation
triggers:
  - how do I automate content creation with AI
  - generate videos from text content automatically
  - create multilingual content with Claude and OpenAI
  - crawl news and generate social media content
  - build automated marketing content pipeline
  - use Remotion to render content videos
  - auto-post content to Facebook pages
  - research and write articles with AI agents
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

A comprehensive TypeScript-based automation system that handles the complete content lifecycle: researching trending topics from news sources, generating multilingual articles (Vietnamese & English), and automatically rendering videos with Remotion. Perfect for marketers and content creators who need to scale production.

## What This Project Does

Ultimate AI Content Pipeline is an end-to-end content automation system that:

- **Auto-crawls** news from TechCrunch, a16z, Twitter/X, LinkedIn for fresh insights
- **Generates content** in multiple formats (toplist, POV, case study, how-to) using Claude 3 or OpenAI
- **Creates dual-language** content (English + Vietnamese) with customizable tone
- **Renders videos** automatically using Remotion for Reels, TikTok, Shorts
- **Auto-publishes** to Facebook pages and social platforms

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
```

### Environment Setup

Create a `.env.local` file in the root directory:

```bash
# AI Services
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# News/Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Social Media Integration
FACEBOOK_ACCESS_TOKEN=your_facebook_token
FACEBOOK_PAGE_ID=your_page_id

# Optional: Custom configurations
CONTENT_LANGUAGE=both # 'en', 'vi', or 'both'
DEFAULT_TONE=professional # 'professional', 'friendly', 'humorous'
```

### Running the Development Server

```bash
npm run dev
```

Navigate to `http://localhost:3000` to access the interface.

## Core Architecture

The pipeline consists of four main stages:

1. **Research Module** - Crawls and aggregates news
2. **Content Generation** - AI-powered article creation
3. **Video Rendering** - Remotion-based video generation
4. **Publishing** - Auto-post to social platforms

## Key API Usage Patterns

### 1. Research & Crawl News

```typescript
import { crawlNews } from '@/lib/research/crawler';
import { analyzeInsights } from '@/lib/research/analyzer';

async function researchTopic(keyword: string) {
  // Crawl recent articles
  const articles = await crawlNews({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter'],
    timeframe: '24h'
  });

  // Extract insights
  const insights = await analyzeInsights(articles, {
    apiKey: process.env.ANTHROPIC_API_KEY,
    model: 'claude-3-sonnet-20240229'
  });

  return {
    articles,
    insights,
    trending: insights.trendingTopics
  };
}
```

### 2. Generate Content with AI

```typescript
import { generateContent } from '@/lib/content/generator';

async function createArticle(topic: string, format: string) {
  const content = await generateContent({
    topic,
    format: format, // 'toplist' | 'pov' | 'case-study' | 'how-to'
    language: 'both', // Generates both English and Vietnamese
    tone: 'professional',
    aiProvider: 'claude', // or 'openai'
    apiKey: process.env.ANTHROPIC_API_KEY,
    researchData: await researchTopic(topic)
  });

  return content;
  // Returns: { en: {...}, vi: {...}, metadata: {...} }
}
```

### 3. Customize Content Format

```typescript
import { ContentGenerator } from '@/lib/content/ContentGenerator';

const generator = new ContentGenerator({
  apiKey: process.env.OPENAI_API_KEY,
  provider: 'openai',
  model: 'gpt-4-turbo-preview'
});

// Generate toplist format
const toplist = await generator.generate({
  type: 'toplist',
  topic: 'AI Marketing Tools 2024',
  itemCount: 10,
  includeStats: true,
  language: 'vi',
  tone: 'friendly'
});

// Generate POV/Opinion piece
const povArticle = await generator.generate({
  type: 'pov',
  topic: 'The Future of AI in Content Marketing',
  perspective: 'expert',
  language: 'en',
  wordCount: 1500
});
```

### 4. Video Generation with Remotion

```typescript
import { renderVideo } from '@/lib/video/renderer';
import { prepareVideoScript } from '@/lib/video/script';

async function createContentVideo(article: any) {
  // Prepare video script from article
  const script = await prepareVideoScript({
    content: article.content,
    format: 'short', // 'short' (15-60s) | 'medium' (1-3min)
    platform: 'tiktok', // 'tiktok' | 'reels' | 'shorts'
    voiceOver: true
  });

  // Render video
  const video = await renderVideo({
    script,
    composition: 'ContentVideo',
    props: {
      title: article.title,
      highlights: article.highlights,
      backgroundColor: '#1a1a1a',
      textColor: '#ffffff'
    },
    outputFormat: 'mp4',
    codec: 'h264',
    fps: 30,
    resolution: {
      width: 1080,
      height: 1920 // Vertical format for social
    }
  });

  return video.url;
}
```

### 5. Auto-Publishing Pipeline

```typescript
import { publishToFacebook } from '@/lib/publish/facebook';
import { schedulePost } from '@/lib/publish/scheduler';

async function publishContent(content: any, video?: string) {
  // Direct publish
  const post = await publishToFacebook({
    accessToken: process.env.FACEBOOK_ACCESS_TOKEN,
    pageId: process.env.FACEBOOK_PAGE_ID,
    message: content.vi.excerpt,
    link: content.articleUrl,
    videoUrl: video,
    published: true
  });

  // Or schedule for later
  const scheduled = await schedulePost({
    platform: 'facebook',
    content: content.vi,
    videoUrl: video,
    publishAt: new Date('2024-06-15T10:00:00Z'),
    credentials: {
      accessToken: process.env.FACEBOOK_ACCESS_TOKEN,
      pageId: process.env.FACEBOOK_PAGE_ID
    }
  });

  return { post, scheduled };
}
```

## Complete Workflow Example

```typescript
import { runContentPipeline } from '@/lib/pipeline';

async function automatedContentCreation() {
  const pipeline = await runContentPipeline({
    // Step 1: Research
    research: {
      keyword: 'AI Marketing Automation',
      sources: ['techcrunch', 'a16z', 'linkedin'],
      depth: 'deep' // 'quick' | 'deep'
    },

    // Step 2: Content Generation
    content: {
      format: 'toplist',
      language: 'both',
      tone: 'professional',
      aiProvider: 'claude',
      apiKey: process.env.ANTHROPIC_API_KEY,
      customPrompt: 'Focus on practical tools with pricing'
    },

    // Step 3: Video Creation
    video: {
      enabled: true,
      platform: 'reels',
      duration: 45,
      includeSubtitles: true,
      music: 'upbeat'
    },

    // Step 4: Publishing
    publish: {
      platforms: ['facebook', 'linkedin'],
      schedule: new Date('2024-06-15T14:00:00Z'),
      credentials: {
        facebook: {
          accessToken: process.env.FACEBOOK_ACCESS_TOKEN,
          pageId: process.env.FACEBOOK_PAGE_ID
        }
      }
    }
  });

  return pipeline;
  // Returns: { article, video, publications, analytics }
}
```

## Advanced Configuration

### Custom Content Templates

```typescript
import { registerTemplate } from '@/lib/content/templates';

registerTemplate('custom-review', {
  structure: [
    { section: 'introduction', minWords: 100 },
    { section: 'pros', format: 'bullet', count: 5 },
    { section: 'cons', format: 'bullet', count: 3 },
    { section: 'verdict', minWords: 150 }
  ],
  seo: {
    keywordDensity: 2.5,
    includeSchema: true
  }
});

const review = await generateContent({
  topic: 'Jasper AI Review',
  format: 'custom-review',
  language: 'vi'
});
```

### Multi-Source Research Aggregation

```typescript
import { AggregateResearch } from '@/lib/research';

const aggregator = new AggregateResearch({
  sources: {
    news: {
      apis: ['techcrunch', 'a16z'],
      apiKey: process.env.RAPIDAPI_KEY
    },
    social: {
      twitter: { enabled: true },
      linkedin: { enabled: true }
    },
    rss: [
      'https://blog.hubspot.com/feed',
      'https://moz.com/blog/feed'
    ]
  },
  filters: {
    minEngagement: 100,
    language: 'en',
    recency: '48h'
  }
});

const data = await aggregator.collect('content marketing trends');
```

### Video Customization

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia } from '@remotion/renderer';
import { webpackOverride } from './remotion.config';

async function customVideoRender(article: any) {
  const bundled = await bundle({
    entryPoint: './src/video/index.tsx',
    webpackOverride
  });

  const { url } = await renderMedia({
    composition: {
      id: 'ArticleVideo',
      width: 1080,
      height: 1920,
      fps: 30,
      durationInFrames: 1350 // 45 seconds at 30fps
    },
    serveUrl: bundled,
    codec: 'h264',
    inputProps: {
      title: article.title,
      sections: article.sections,
      theme: 'dark',
      font: 'Inter',
      brand: {
        logo: '/brand/logo.png',
        color: '#ff6b6b'
      }
    },
    outputLocation: `out/${article.id}.mp4`
  });

  return url;
}
```

## Common Patterns

### Batch Content Generation

```typescript
async function batchGenerate(topics: string[]) {
  const results = await Promise.all(
    topics.map(async (topic) => {
      try {
        return await createArticle(topic, 'toplist');
      } catch (error) {
        console.error(`Failed for ${topic}:`, error);
        return null;
      }
    })
  );

  return results.filter(Boolean);
}
```

### Content Optimization

```typescript
import { optimizeForSEO } from '@/lib/content/seo';

async function optimizedContent(topic: string) {
  const draft = await generateContent({ topic, format: 'how-to' });
  
  const optimized = await optimizeForSEO(draft, {
    targetKeyword: topic,
    relatedKeywords: await findRelatedKeywords(topic),
    readabilityScore: 60,
    includeMetaDescription: true,
    addInternalLinks: true
  });

  return optimized;
}
```

## Troubleshooting

### API Rate Limits

```typescript
import { RateLimiter } from '@/lib/utils/rateLimiter';

const limiter = new RateLimiter({
  maxRequests: 50,
  perMilliseconds: 60000 // 50 requests per minute
});

await limiter.execute(() => generateContent({ topic: 'AI' }));
```

### Content Generation Timeout

```typescript
const content = await generateContent({
  topic: 'Long Topic',
  timeout: 120000, // 2 minutes
  retries: 3,
  fallbackProvider: 'openai' // If Claude fails
});
```

### Video Rendering Memory Issues

```typescript
import { renderVideo } from '@/lib/video/renderer';

const video = await renderVideo({
  script,
  quality: 'medium', // Use 'medium' instead of 'high' for memory constraints
  concurrency: 2, // Limit concurrent rendering
  maxMemory: '2GB'
});
```

### Invalid Facebook Credentials

```typescript
import { validateCredentials } from '@/lib/publish/validator';

const isValid = await validateCredentials({
  platform: 'facebook',
  accessToken: process.env.FACEBOOK_ACCESS_TOKEN,
  pageId: process.env.FACEBOOK_PAGE_ID
});

if (!isValid) {
  throw new Error('Invalid Facebook credentials. Check token expiration.');
}
```

## CLI Usage (if available)

```bash
# Generate content from command line
npm run generate -- --topic "AI Marketing" --format toplist --lang vi

# Research only
npm run research -- --keyword "content automation" --sources techcrunch,a16z

# Render video from existing content
npm run render-video -- --input content.json --output video.mp4

# Publish to social media
npm run publish -- --content article.json --platforms facebook,linkedin
```

## Best Practices

1. **Always validate** research data before content generation
2. **Use environment variables** for all API keys and secrets
3. **Implement retry logic** for API calls to handle transient failures
4. **Cache research results** to avoid redundant API calls
5. **Monitor token usage** for Claude/OpenAI to control costs
6. **Test video renders** locally before batch processing
7. **Schedule posts** during peak engagement hours for your audience
