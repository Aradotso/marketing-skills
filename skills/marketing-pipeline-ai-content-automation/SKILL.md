---
name: marketing-pipeline-ai-content-automation
description: Automate content creation from research to video generation using AI (Claude, OpenAI) and Remotion for Vietnamese and English marketing content
triggers:
  - set up automated content pipeline with AI
  - create marketing content from keyword to video
  - automate content research and script generation
  - generate videos from blog posts automatically
  - crawl news sources and create content with AI
  - build AI-powered content automation system
  - use remotion for marketing video generation
  - automate multilingual content creation pipeline
---

# Marketing Pipeline AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to help developers use the **Ultimate AI Content Pipeline** - an end-to-end content automation system that transforms keywords into fully-researched articles and videos. The pipeline crawls real-time data from sources like TechCrunch, a16z, Twitter/X, and LinkedIn, then uses Claude 3 or OpenAI to generate content in multiple formats (toplist, POV, case study, how-to), and finally renders videos using Remotion.

## What This Project Does

The marketing-pipeline-share project is a TypeScript/Next.js application that automates:

1. **Research Phase**: Crawls latest news and insights from tech publications and social media
2. **Content Generation**: Uses AI (Claude/OpenAI) to create articles in Vietnamese and English with customizable tone
3. **Video Rendering**: Automatically generates infographics and short-form videos using Remotion
4. **Multi-format Output**: Produces content optimized for various platforms (Reels, TikTok, Shorts)

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
cp .env.example .env.local
```

## Configuration

Create a `.env.local` file with the following required variables:

```bash
# AI Provider Keys (choose one or both)
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research/Crawling APIs
RAPIDAPI_KEY=your_rapidapi_key

# Optional: Database & Storage
DATABASE_URL=your_database_connection
STORAGE_BUCKET=your_storage_bucket

# Remotion Configuration
REMOTION_TIMEOUT=120000
VIDEO_OUTPUT_DIR=./public/videos
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI provider integrations
│   │   ├── crawler/     # Web scraping modules
│   │   ├── content/     # Content generation logic
│   │   └── video/       # Remotion video rendering
│   └── types/           # TypeScript definitions
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Key API Usage Patterns

### 1. Research & Data Crawling

```typescript
import { crawlNewsData } from '@/lib/crawler/news-scraper';
import { analyzeInsights } from '@/lib/ai/insight-analyzer';

// Crawl recent news for a keyword
async function researchTopic(keyword: string) {
  const newsData = await crawlNewsData({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeframe: '24h'
  });

  // Extract insights using AI
  const insights = await analyzeInsights(newsData, {
    provider: 'claude', // or 'openai'
    model: 'claude-3-opus-20240229'
  });

  return {
    rawData: newsData,
    insights: insights
  };
}
```

### 2. Content Generation with AI

```typescript
import { generateContent } from '@/lib/content/generator';
import { ContentFormat, Language, Tone } from '@/types/content';

// Generate article in multiple formats
async function createArticle(topic: string, research: any) {
  const content = await generateContent({
    topic,
    research,
    format: ContentFormat.TOPLIST, // TOPLIST, POV, CASE_STUDY, HOWTO
    languages: [Language.VIETNAMESE, Language.ENGLISH],
    tone: Tone.EXPERT, // EXPERT, FRIENDLY, HUMOROUS
    provider: 'claude',
    apiKey: process.env.ANTHROPIC_API_KEY
  });

  return content;
}

// Example output structure
interface GeneratedContent {
  vietnamese: {
    title: string;
    body: string;
    summary: string;
    tags: string[];
  };
  english: {
    title: string;
    body: string;
    summary: string;
    tags: string[];
  };
  metadata: {
    format: ContentFormat;
    tone: Tone;
    generatedAt: Date;
  };
}
```

### 3. Video Rendering with Remotion

```typescript
import { renderVideo } from '@/lib/video/renderer';
import { VideoTemplate } from '@/types/video';

// Render video from content
async function generateVideoFromContent(content: GeneratedContent) {
  const video = await renderVideo({
    template: VideoTemplate.INFOGRAPHIC, // INFOGRAPHIC, SHORT_FORM, SLIDESHOW
    content: {
      title: content.vietnamese.title,
      points: extractKeyPoints(content.vietnamese.body),
      branding: {
        logo: '/logo.png',
        colors: ['#FF6B6B', '#4ECDC4']
      }
    },
    format: {
      width: 1080,
      height: 1920, // For Reels/TikTok/Shorts
      fps: 30
    },
    outputPath: process.env.VIDEO_OUTPUT_DIR
  });

  return video.path;
}
```

### 4. Complete Pipeline Workflow

```typescript
import { ContentPipeline } from '@/lib/pipeline';

// End-to-end content creation
async function runContentPipeline(keyword: string) {
  const pipeline = new ContentPipeline({
    aiProvider: 'claude',
    apiKey: process.env.ANTHROPIC_API_KEY
  });

  try {
    // Step 1: Research
    const research = await pipeline.research(keyword);
    console.log(`Found ${research.insights.length} insights`);

    // Step 2: Generate content
    const articles = await pipeline.generateContent({
      topic: keyword,
      research,
      formats: ['toplist', 'howto'],
      languages: ['vi', 'en']
    });

    // Step 3: Render videos
    const videos = await pipeline.renderVideos(articles, {
      templates: ['infographic', 'short_form']
    });

    // Step 4: Schedule publishing (optional)
    await pipeline.schedulePosts({
      articles,
      videos,
      platforms: ['facebook', 'tiktok', 'youtube']
    });

    return {
      articles,
      videos
    };
  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}
```

## CLI Commands (if available)

```bash
# Start development server
npm run dev

# Generate content from keyword
npm run generate -- --keyword "AI marketing" --format toplist

# Render video only
npm run render:video -- --input ./content/article.json

# Run full pipeline
npm run pipeline -- --keyword "startup trends 2024" --languages vi,en

# Build for production
npm run build
```

## React Components Usage

```typescript
import { ContentGenerator } from '@/components/ContentGenerator';
import { VideoRenderer } from '@/components/VideoRenderer';
import { PipelineStatus } from '@/components/PipelineStatus';

// In your Next.js page
export default function DashboardPage() {
  return (
    <div>
      <ContentGenerator
        onGenerate={async (keyword) => {
          const result = await runContentPipeline(keyword);
          return result;
        }}
      />
      
      <PipelineStatus />
      
      <VideoRenderer
        templates={['infographic', 'short_form']}
        outputFormat="mp4"
      />
    </div>
  );
}
```

## Common Patterns

### Pattern 1: Batch Content Creation

```typescript
async function batchGenerateContent(keywords: string[]) {
  const results = await Promise.allSettled(
    keywords.map(keyword => runContentPipeline(keyword))
  );

  const successful = results
    .filter(r => r.status === 'fulfilled')
    .map(r => (r as PromiseFulfilledResult<any>).value);

  return successful;
}
```

### Pattern 2: Custom AI Prompts

```typescript
import { customPrompt } from '@/lib/ai/prompts';

const vietnameseMarketingPrompt = customPrompt({
  role: 'expert_marketer',
  context: 'Vietnamese startup ecosystem',
  style: 'data-driven, engaging',
  instructions: [
    'Include local market examples',
    'Use Vietnamese idioms naturally',
    'Add statistics from Vietnamese sources'
  ]
});

const content = await generateContent({
  topic: 'Fintech trends',
  customPrompt: vietnameseMarketingPrompt
});
```

### Pattern 3: Video Template Customization

```typescript
import { createCustomTemplate } from '@/lib/video/templates';

const brandedTemplate = createCustomTemplate({
  name: 'branded-toplist',
  composition: 'ToplistComposition',
  props: {
    theme: {
      primaryColor: '#FF6B6B',
      secondaryColor: '#4ECDC4',
      fontFamily: 'Inter'
    },
    animation: {
      transition: 'slide',
      duration: 30
    },
    branding: {
      logo: '/brand/logo.png',
      watermark: true
    }
  }
});

await renderVideo({
  template: brandedTemplate,
  content: myContent
});
```

## Troubleshooting

### AI API Rate Limits

```typescript
import { RateLimiter } from '@/lib/utils/rate-limiter';

const limiter = new RateLimiter({
  requestsPerMinute: 50,
  provider: 'claude'
});

await limiter.execute(async () => {
  return await generateContent(params);
});
```

### Video Rendering Timeout

```typescript
// Increase timeout in .env.local
REMOTION_TIMEOUT=300000

// Or in code
await renderVideo({
  ...config,
  timeout: 300000, // 5 minutes
  onProgress: (progress) => {
    console.log(`Rendering: ${progress}%`);
  }
});
```

### Crawler Blocked by Websites

```typescript
import { crawlWithProxy } from '@/lib/crawler/proxy';

const newsData = await crawlWithProxy({
  keyword,
  sources: ['techcrunch'],
  proxy: {
    enabled: true,
    rotating: true
  },
  retries: 3
});
```

### Memory Issues with Large Content

```typescript
// Process content in chunks
import { chunkContent } from '@/lib/utils/chunking';

const largeResearch = await researchTopic('broad keyword');
const chunks = chunkContent(largeResearch, { maxTokens: 4000 });

for (const chunk of chunks) {
  const partialContent = await generateContent({ research: chunk });
  await savePartialContent(partialContent);
}
```

## Integration Examples

### Facebook Auto-Posting

```typescript
import { FacebookPublisher } from '@/lib/publishers/facebook';

const publisher = new FacebookPublisher({
  pageAccessToken: process.env.FACEBOOK_PAGE_TOKEN
});

await publisher.post({
  content: article.vietnamese,
  video: videoPath,
  scheduledTime: new Date('2024-01-15T10:00:00Z')
});
```

### Webhook Notifications

```typescript
import { WebhookNotifier } from '@/lib/notifications/webhook';

const notifier = new WebhookNotifier(process.env.WEBHOOK_URL);

pipeline.on('complete', async (result) => {
  await notifier.send({
    event: 'pipeline_complete',
    data: result,
    timestamp: new Date()
  });
});
```

This skill provides comprehensive guidance for AI agents to help developers implement and customize the marketing pipeline automation system effectively.
