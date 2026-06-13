---
name: ultimate-ai-content-pipeline
description: Automated content creation pipeline from research to video generation using Claude/OpenAI and Remotion
triggers:
  - automate content creation with AI pipeline
  - generate blog posts and videos automatically
  - research and create marketing content with AI
  - build automated content workflow
  - create AI-powered content generation system
  - generate videos from text content automatically
  - set up content automation pipeline
  - build marketing content automation with Claude
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

Ultimate AI Content Pipeline is a comprehensive TypeScript-based content automation system that handles the entire content lifecycle: from researching trending topics, generating multi-format articles (Vietnamese & English), to automatically rendering videos and graphics using Remotion. It integrates Claude 3, OpenAI, and various data sources (TechCrunch, Twitter, LinkedIn) to create data-backed, trend-leading content.

## What It Does

- **Auto-Research**: Crawls and analyzes real-time data from major tech news sources (TechCrunch, a16z, X/Twitter, LinkedIn)
- **AI Content Generation**: Creates content in multiple formats (Toplist, POV, Case Study, How-to) using Claude/OpenAI
- **Multi-Language Output**: Generates both English and Vietnamese versions simultaneously
- **Video Rendering**: Automatically converts written content into videos/infographics using Remotion
- **Platform Optimization**: Exports video in formats optimized for Reels, TikTok, and YouTube Shorts

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

## Environment Configuration

Create a `.env.local` file in the root directory:

```env
# AI APIs
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Data Sources
RAPIDAPI_KEY=your_rapidapi_key
TWITTER_BEARER_TOKEN=your_twitter_token

# Database (if applicable)
DATABASE_URL=your_database_url

# Remotion Configuration
REMOTION_AWS_ACCESS_KEY_ID=your_aws_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app router pages
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── crawler/     # Web scraping modules
│   │   ├── content/     # Content generation logic
│   │   └── video/       # Remotion video rendering
│   └── utils/           # Utility functions
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Core API Usage

### 1. Research & Data Crawling

```typescript
import { crawlTechNews } from '@/lib/crawler/tech-sources';
import { analyzeTrends } from '@/lib/ai/trend-analyzer';

async function researchTopic(keyword: string) {
  // Crawl latest news from multiple sources
  const articles = await crawlTechNews({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter'],
    timeframe: '24h'
  });

  // Analyze trends using AI
  const insights = await analyzeTrends(articles, {
    model: 'claude-3-sonnet',
    extractMetrics: true,
    identifyPatterns: true
  });

  return {
    articles,
    insights,
    dataPoints: insights.metrics
  };
}
```

### 2. AI Content Generation

```typescript
import { generateContent } from '@/lib/content/generator';
import { Anthropic } from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY
});

async function createBlogPost(topic: string, format: 'toplist' | 'pov' | 'case-study' | 'how-to') {
  // Research phase
  const research = await researchTopic(topic);

  // Generate content in both languages
  const content = await generateContent({
    topic,
    format,
    research: research.insights,
    languages: ['en', 'vi'],
    tone: 'professional', // or 'friendly', 'humorous'
    client: anthropic,
    model: 'claude-3-sonnet-20240229'
  });

  return {
    english: content.en,
    vietnamese: content.vi,
    metadata: {
      wordCount: content.wordCount,
      readingTime: content.readingTime,
      keywords: content.extractedKeywords
    }
  };
}
```

### 3. Multi-Format Content Generation

```typescript
import { ContentFormatter } from '@/lib/content/formatter';

const formatter = new ContentFormatter();

// Generate Toplist format
const toplist = await formatter.generateToplist({
  title: 'Top 10 AI Tools for Marketing in 2024',
  items: research.insights.topItems,
  includeRatings: true,
  addProsAndCons: true
});

// Generate POV/Opinion piece
const povArticle = await formatter.generatePOV({
  topic: 'Why AI Won\'t Replace Human Marketers',
  stance: 'balanced',
  arguments: research.insights.keyArguments,
  includeCounterpoints: true
});

// Generate Case Study
const caseStudy = await formatter.generateCaseStudy({
  company: 'Company Name',
  challenge: research.insights.problem,
  solution: research.insights.solution,
  results: research.insights.metrics
});
```

### 4. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import { VideoComposition } from '@/remotion/compositions/ContentVideo';

async function generateVideo(content: any) {
  // Bundle Remotion project
  const bundleLocation = await bundle({
    entryPoint: './remotion/index.ts',
    webpackOverride: (config) => config
  });

  // Select composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title: content.title,
      points: content.keyPoints,
      style: 'modern',
      duration: 60 // seconds
    }
  });

  // Render video
  const outputPath = `./output/video-${Date.now()}.mp4`;
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: composition.inputProps
  });

  return outputPath;
}
```

### 5. Complete Pipeline Workflow

```typescript
import { ContentPipeline } from '@/lib/pipeline';

async function runFullPipeline(keyword: string) {
  const pipeline = new ContentPipeline({
    aiProvider: 'claude', // or 'openai'
    outputFormats: ['blog', 'video', 'social'],
    languages: ['en', 'vi']
  });

  try {
    // Step 1: Research
    const research = await pipeline.research(keyword);
    
    // Step 2: Generate content
    const content = await pipeline.generateContent({
      research,
      format: 'toplist',
      tone: 'professional'
    });

    // Step 3: Create visuals
    const video = await pipeline.renderVideo(content);
    const infographic = await pipeline.createInfographic(content);

    // Step 4: Export for platforms
    const exports = await pipeline.exportForPlatforms({
      content,
      video,
      platforms: ['reels', 'tiktok', 'youtube-shorts']
    });

    return {
      content,
      video,
      infographic,
      exports
    };
  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}
```

## API Routes (Next.js)

### Create Content API

```typescript
// src/app/api/content/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { createBlogPost } from '@/lib/content/generator';

export async function POST(request: NextRequest) {
  try {
    const { topic, format, language } = await request.json();

    const result = await createBlogPost(topic, format);

    return NextResponse.json({
      success: true,
      content: language === 'vi' ? result.vietnamese : result.english,
      metadata: result.metadata
    });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

### Video Generation API

```typescript
// src/app/api/video/render/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { generateVideo } from '@/lib/video/renderer';

export async function POST(request: NextRequest) {
  try {
    const { contentId, style, duration } = await request.json();

    const videoPath = await generateVideo({
      contentId,
      style,
      duration
    });

    return NextResponse.json({
      success: true,
      videoUrl: videoPath,
      status: 'completed'
    });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

## Common Patterns

### Custom AI Prompts

```typescript
import { createCustomPrompt } from '@/lib/ai/prompts';

const customPrompt = createCustomPrompt({
  systemMessage: `You are an expert marketing content writer specializing in ${topic}.`,
  userMessage: `Create a detailed ${format} article about ${topic}.
    Include:
    - Latest data and statistics
    - Real-world examples
    - Actionable insights
    - SEO-optimized headers
    
    Target audience: ${targetAudience}
    Tone: ${tone}
    Word count: ${wordCount}`,
  temperature: 0.7,
  maxTokens: 4000
});

const response = await anthropic.messages.create({
  model: 'claude-3-sonnet-20240229',
  ...customPrompt
});
```

### Batch Content Generation

```typescript
async function generateBatch(topics: string[]) {
  const results = await Promise.allSettled(
    topics.map(topic => 
      createBlogPost(topic, 'toplist')
    )
  );

  return results.map((result, index) => ({
    topic: topics[index],
    status: result.status,
    data: result.status === 'fulfilled' ? result.value : null,
    error: result.status === 'rejected' ? result.reason : null
  }));
}
```

### Content Scheduling

```typescript
import { scheduleContent } from '@/lib/scheduler';

await scheduleContent({
  content: generatedContent,
  platforms: ['facebook', 'linkedin', 'twitter'],
  scheduleTime: new Date('2024-12-25T10:00:00'),
  autoPost: true,
  includeVideo: true
});
```

## Running the Application

```bash
# Development mode
npm run dev

# Build for production
npm run build

# Start production server
npm run start

# Render Remotion videos
npm run remotion:render
```

## Troubleshooting

### API Rate Limits

```typescript
import { RateLimiter } from '@/lib/utils/rate-limiter';

const limiter = new RateLimiter({
  maxRequests: 10,
  windowMs: 60000 // 1 minute
});

await limiter.execute(async () => {
  return await anthropic.messages.create({
    model: 'claude-3-sonnet-20240229',
    messages: [{ role: 'user', content: prompt }]
  });
});
```

### Video Rendering Fails

```typescript
// Use smaller compositions or increase timeout
const composition = await selectComposition({
  serveUrl: bundleLocation,
  id: 'ContentVideo',
  timeoutInMilliseconds: 90000 // 90 seconds
});

// Reduce video quality for faster rendering
await renderMedia({
  composition,
  serveUrl: bundleLocation,
  codec: 'h264',
  crf: 23, // Lower = better quality, higher = faster
  scale: 0.5 // Reduce resolution
});
```

### Memory Issues with Large Crawls

```typescript
// Implement pagination for crawling
async function crawlInBatches(keyword: string, batchSize = 10) {
  const allResults = [];
  let page = 0;

  while (true) {
    const batch = await crawlTechNews({
      keyword,
      limit: batchSize,
      offset: page * batchSize
    });

    if (batch.length === 0) break;
    
    allResults.push(...batch);
    page++;
    
    // Prevent memory overflow
    if (allResults.length >= 100) break;
  }

  return allResults;
}
```

### OpenAI vs Claude Selection

```typescript
// Dynamic provider selection based on task
function selectAIProvider(taskType: string) {
  const providers = {
    'long-form': 'claude', // Better for detailed content
    'quick-summary': 'openai', // Faster responses
    'creative': 'claude', // More creative outputs
    'structured-data': 'openai' // Better JSON outputs
  };

  return providers[taskType] || 'claude';
}
```

## Best Practices

1. **Always validate research data** before passing to AI generators
2. **Cache API responses** to avoid redundant calls and reduce costs
3. **Use streaming responses** for long content generation to improve UX
4. **Implement retry logic** for API failures with exponential backoff
5. **Monitor token usage** to control costs across Claude/OpenAI APIs
6. **Test video rendering locally** before deploying to production
7. **Sanitize user inputs** before passing to crawlers or AI prompts
