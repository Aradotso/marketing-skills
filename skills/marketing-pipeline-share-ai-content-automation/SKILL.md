---
name: marketing-pipeline-share-ai-content-automation
description: AI-powered content pipeline for automated research, scriptwriting, social media posting, and video generation using Claude, OpenAI, and Remotion
triggers:
  - automate content creation from research to video
  - generate marketing content with AI pipeline
  - create social media posts and videos automatically
  - build content automation workflow with AI
  - research and generate videos with Remotion
  - set up AI content generation pipeline
  - automate blog posts and video creation
  - use Claude and OpenAI for content marketing
---

# Marketing Pipeline Share - AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the **Ultimate AI Content Pipeline**, a TypeScript-based system that automates the entire content creation workflow: from research and scriptwriting to automatic social media posting and video generation using Claude 3, OpenAI, and Remotion.

## What This Project Does

The Marketing Pipeline Share is an all-in-one content automation system that:

- **Auto-crawls** recent news from sources like TechCrunch, a16z, X (Twitter), LinkedIn (last 24 hours)
- **Generates** multi-format content (Toplist, POV, Case Study, How-to) in multiple languages (English/Vietnamese)
- **Renders** videos and infographics automatically using Remotion
- **Posts** content automatically to social media platforms
- **Optimizes** output for Reels, TikTok, Shorts with proper aspect ratios

## Installation

### Prerequisites

- Node.js 18+ and npm/yarn/pnpm
- API keys for OpenAI, Anthropic (Claude), and RapidAPI

### Setup Steps

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

### Environment Configuration

Create a `.env` file with the following variables:

```env
# AI Services
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_url

# Social Media APIs
FACEBOOK_ACCESS_TOKEN=your_fb_token
TWITTER_API_KEY=your_twitter_key
LINKEDIN_ACCESS_TOKEN=your_linkedin_token

# Remotion Configuration
REMOTION_AWS_ACCESS_KEY=your_aws_key
REMOTION_AWS_SECRET_KEY=your_aws_secret
```

### Run Development Server

```bash
npm run dev
# or
yarn dev
# or
pnpm dev
```

The application will be available at `http://localhost:3000`

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/              # Core libraries
│   │   ├── ai/          # AI integration (OpenAI, Claude)
│   │   ├── crawler/     # News crawling logic
│   │   ├── content/     # Content generation
│   │   └── video/       # Remotion video rendering
│   ├── services/        # API services
│   └── types/           # TypeScript definitions
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Core API Usage

### 1. Research & Data Crawling

```typescript
import { crawlNews } from '@/lib/crawler/news-crawler';
import { analyzeContent } from '@/lib/ai/content-analyzer';

// Crawl recent news based on keyword
async function researchTopic(keyword: string) {
  const newsArticles = await crawlNews({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeRange: '24h',
    limit: 20
  });

  // Analyze and extract insights using AI
  const insights = await analyzeContent(newsArticles, {
    model: 'claude-3-opus',
    extractStats: true,
    findTrends: true
  });

  return insights;
}
```

### 2. Content Generation with AI

```typescript
import { generateContent } from '@/lib/content/generator';
import { ContentFormat, ContentTone } from '@/types/content';

// Generate multi-format content
async function createContent(topic: string, research: any) {
  const content = await generateContent({
    topic,
    research,
    format: ContentFormat.TOPLIST, // or POV, CASE_STUDY, HOW_TO
    languages: ['en', 'vi'],
    tone: ContentTone.EXPERT, // or FRIENDLY, HUMOROUS
    aiProvider: 'anthropic', // or 'openai'
    model: 'claude-3-opus-20240229'
  });

  return content;
}

// Example output structure
interface GeneratedContent {
  title: string;
  summary: string;
  mainContent: string;
  keyPoints: string[];
  language: string;
  metadata: {
    wordCount: number;
    readingTime: number;
    seoScore: number;
  };
}
```

### 3. Video Generation with Remotion

```typescript
import { renderVideo } from '@/lib/video/remotion-renderer';
import { VideoTemplate } from '@/types/video';

// Render video from content
async function generateVideo(content: GeneratedContent) {
  const video = await renderVideo({
    template: VideoTemplate.INFOGRAPHIC, // or SHORT_FORM, REELS
    content: {
      title: content.title,
      points: content.keyPoints,
      stats: content.metadata
    },
    aspectRatio: '9:16', // TikTok/Reels format
    duration: 30, // seconds
    outputFormat: 'mp4',
    quality: 'high'
  });

  return video;
}
```

### 4. Automated Social Media Posting

```typescript
import { postToSocialMedia } from '@/services/social-media';
import { Platform } from '@/types/social';

// Post content to multiple platforms
async function publishContent(content: GeneratedContent, video: any) {
  const results = await postToSocialMedia({
    platforms: [Platform.FACEBOOK, Platform.TWITTER, Platform.LINKEDIN],
    content: {
      text: content.summary,
      media: [video.url],
      hashtags: content.metadata.hashtags,
      schedule: new Date(Date.now() + 3600000) // 1 hour from now
    }
  });

  return results;
}
```

## Common Patterns

### Full Pipeline Execution

```typescript
import { runFullPipeline } from '@/lib/pipeline/orchestrator';

async function automateContentCreation(keyword: string) {
  try {
    const result = await runFullPipeline({
      keyword,
      steps: {
        research: true,
        contentGeneration: true,
        videoCreation: true,
        autoPost: true
      },
      config: {
        contentFormats: ['toplist', 'case-study'],
        languages: ['en', 'vi'],
        videoTemplates: ['infographic', 'short-form'],
        platforms: ['facebook', 'twitter', 'linkedin']
      }
    });

    console.log('Pipeline completed:', result);
    return result;
  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}
```

### Batch Content Generation

```typescript
import { batchGenerate } from '@/lib/content/batch-processor';

async function createMultipleContents(topics: string[]) {
  const results = await batchGenerate({
    topics,
    concurrency: 3, // Process 3 at a time
    retryOnFailure: true,
    saveProgress: true,
    onProgress: (progress) => {
      console.log(`Progress: ${progress.completed}/${progress.total}`);
    }
  });

  return results;
}
```

### Custom Content Template

```typescript
import { createCustomTemplate } from '@/lib/content/templates';

const myTemplate = createCustomTemplate({
  name: 'product-launch',
  structure: [
    { section: 'hook', maxWords: 50 },
    { section: 'problem', maxWords: 150 },
    { section: 'solution', maxWords: 200 },
    { section: 'benefits', format: 'list', items: 5 },
    { section: 'cta', maxWords: 30 }
  ],
  tone: 'persuasive',
  includeStats: true
});

const content = await generateContent({
  topic: 'New SaaS Product Launch',
  template: myTemplate,
  aiProvider: 'openai',
  model: 'gpt-4-turbo'
});
```

## CLI Commands (if available)

```bash
# Generate content from keyword
npm run generate -- --keyword "AI automation" --format toplist

# Crawl and analyze news
npm run research -- --topic "marketing trends" --sources techcrunch,a16z

# Render video from content
npm run render -- --input content.json --template infographic

# Run full pipeline
npm run pipeline -- --keyword "content marketing" --auto-post

# Schedule content
npm run schedule -- --config schedule.json --platforms facebook,twitter
```

## Configuration

### AI Provider Configuration

```typescript
// config/ai.config.ts
export const aiConfig = {
  providers: {
    openai: {
      model: 'gpt-4-turbo',
      temperature: 0.7,
      maxTokens: 2000
    },
    anthropic: {
      model: 'claude-3-opus-20240229',
      temperature: 0.8,
      maxTokens: 4000
    }
  },
  defaultProvider: 'anthropic',
  fallbackProvider: 'openai'
};
```

### Crawler Configuration

```typescript
// config/crawler.config.ts
export const crawlerConfig = {
  sources: {
    techcrunch: {
      enabled: true,
      baseUrl: 'https://techcrunch.com',
      rateLimit: 10 // requests per minute
    },
    twitter: {
      enabled: true,
      searchTypes: ['recent', 'popular'],
      maxResults: 50
    }
  },
  cacheExpiry: 3600, // 1 hour
  userAgent: 'ContentPipeline/1.0'
};
```

### Remotion Video Configuration

```typescript
// remotion/config.ts
export const videoConfig = {
  templates: {
    infographic: {
      fps: 30,
      duration: 30,
      width: 1080,
      height: 1920, // 9:16 for Reels/TikTok
      backgroundColor: '#ffffff'
    },
    shortForm: {
      fps: 60,
      duration: 15,
      width: 1080,
      height: 1920
    }
  },
  rendering: {
    codec: 'h264',
    concurrency: 4,
    imageFormat: 'png'
  }
};
```

## Troubleshooting

### API Rate Limits

```typescript
import { RateLimiter } from '@/lib/utils/rate-limiter';

// Implement rate limiting for API calls
const limiter = new RateLimiter({
  maxRequests: 50,
  perMilliseconds: 60000 // 50 requests per minute
});

async function safeApiCall(fn: () => Promise<any>) {
  await limiter.wait();
  return fn();
}
```

### Content Generation Failures

```typescript
import { retry } from '@/lib/utils/retry';

// Retry failed content generation
const content = await retry(
  () => generateContent(params),
  {
    maxAttempts: 3,
    delayMs: 1000,
    backoff: 'exponential',
    onRetry: (error, attempt) => {
      console.log(`Retry attempt ${attempt}: ${error.message}`);
    }
  }
);
```

### Video Rendering Issues

```typescript
// Check Remotion configuration
import { validateRemotionSetup } from '@/lib/video/validator';

async function ensureVideoSetup() {
  const validation = await validateRemotionSetup();
  
  if (!validation.success) {
    console.error('Remotion setup issues:', validation.errors);
    throw new Error('Video rendering not configured properly');
  }
}
```

### Memory Management for Large Batches

```typescript
import { processInChunks } from '@/lib/utils/chunk-processor';

// Process large batches without memory issues
async function processManyTopics(topics: string[]) {
  const results = await processInChunks(
    topics,
    async (chunk) => {
      return Promise.all(chunk.map(topic => createContent(topic)));
    },
    {
      chunkSize: 5,
      delayBetweenChunks: 2000 // 2 seconds
    }
  );

  return results.flat();
}
```

## Advanced Usage

### Custom AI Prompt Templates

```typescript
import { createPromptTemplate } from '@/lib/ai/prompt-builder';

const customPrompt = createPromptTemplate({
  system: `You are an expert content strategist specializing in ${process.env.NICHE}.`,
  user: `Create a {{format}} about {{topic}} using this research: {{research}}
         Target audience: {{audience}}
         Tone: {{tone}}
         Include: {{requirements}}`,
  variables: ['format', 'topic', 'research', 'audience', 'tone', 'requirements']
});

const content = await generateWithCustomPrompt(customPrompt, {
  format: 'case study',
  topic: 'AI automation',
  research: insights,
  audience: 'marketers',
  tone: 'professional',
  requirements: 'ROI statistics, real examples'
});
```

### Webhook Integration

```typescript
import { setupWebhooks } from '@/services/webhooks';

// Notify external systems on pipeline completion
setupWebhooks({
  onContentGenerated: async (content) => {
    await fetch(process.env.WEBHOOK_URL_CONTENT, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(content)
    });
  },
  onVideoRendered: async (video) => {
    await fetch(process.env.WEBHOOK_URL_VIDEO, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(video)
    });
  }
});
```

This skill provides comprehensive coverage for working with the Marketing Pipeline Share project, enabling AI agents to assist developers in automating content creation workflows from research through video generation and social media distribution.
