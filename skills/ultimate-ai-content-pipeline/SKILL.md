---
name: ultimate-ai-content-pipeline
description: Automated content creation pipeline from research to video generation using Claude, OpenAI, and Remotion
triggers:
  - how do I automate content creation from research to video
  - set up AI content pipeline with Claude and OpenAI
  - generate video content automatically from articles
  - crawl news sources and create content with AI
  - build automated marketing content workflow
  - create videos from text using Remotion
  - automate content research and script generation
  - set up multilingual content generation pipeline
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with Ultimate AI Content Pipeline, a comprehensive TypeScript-based system that automates the entire content creation workflow: from researching trending topics, generating multilingual articles, to rendering videos automatically using Claude, OpenAI, and Remotion.

## What This Project Does

Ultimate AI Content Pipeline is an all-in-one content automation system that:

- **Auto-scans research sources**: Crawls TechCrunch, a16z, Twitter/X, LinkedIn for trending content within 24 hours
- **Generates diverse content formats**: Creates Toplists, POV pieces, Case Studies, How-tos using Claude/OpenAI
- **Multilingual support**: Produces content in both English and Vietnamese simultaneously
- **Video rendering**: Converts text content into videos and infographics using Remotion
- **Multi-platform optimization**: Exports videos optimized for Reels, TikTok, Shorts

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

Create a `.env.local` file in the root directory:

```env
# AI API Keys
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key
TWITTER_BEARER_TOKEN=your_twitter_bearer_token

# Remotion Video Rendering
REMOTION_LICENSE_KEY=your_remotion_license

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Next.js
NEXT_PUBLIC_API_URL=http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── crawler/     # Web scraping modules
│   │   ├── content/     # Content generation logic
│   │   └── video/       # Remotion video rendering
│   ├── types/           # TypeScript type definitions
│   └── utils/           # Utility functions
├── public/              # Static assets
└── remotion/            # Remotion video templates
```

## Key Components and Usage

### 1. Research Crawler

```typescript
import { NewsGatherer } from '@/lib/crawler/news-gatherer';

// Initialize the news gatherer
const gatherer = new NewsGatherer({
  sources: ['techcrunch', 'a16z', 'twitter'],
  rapidApiKey: process.env.RAPIDAPI_KEY,
  twitterToken: process.env.TWITTER_BEARER_TOKEN,
});

// Gather trending topics
async function gatherTrendingContent(keyword: string) {
  const results = await gatherer.search({
    keyword,
    timeRange: '24h',
    limit: 20,
  });

  return results;
}

// Example usage
const trendingAI = await gatherTrendingContent('artificial intelligence');
console.log(`Found ${trendingAI.length} articles`);
```

### 2. AI Content Generation with Claude

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

interface ContentConfig {
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  researchData: any[];
}

async function generateContent(config: ContentConfig) {
  const systemPrompt = `You are an expert content creator specializing in ${config.format} articles. 
Write in a ${config.tone} tone in ${config.language === 'en' ? 'English' : 'Vietnamese'}.`;

  const userPrompt = `Based on this research data:
${JSON.stringify(config.researchData, null, 2)}

Create a comprehensive ${config.format} article that:
- Incorporates latest trends and data
- Provides actionable insights
- Is engaging and well-structured`;

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    temperature: 0.7,
    system: systemPrompt,
    messages: [
      {
        role: 'user',
        content: userPrompt,
      },
    ],
  });

  return message.content[0].text;
}

// Generate bilingual content
async function generateBilingualContent(researchData: any[]) {
  const [englishContent, vietnameseContent] = await Promise.all([
    generateContent({
      format: 'toplist',
      language: 'en',
      tone: 'expert',
      researchData,
    }),
    generateContent({
      format: 'toplist',
      language: 'vi',
      tone: 'friendly',
      researchData,
    }),
  ]);

  return { englishContent, vietnameseContent };
}
```

### 3. OpenAI Integration

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateContentWithGPT(prompt: string, format: string) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are a professional content writer creating ${format} content for marketing purposes.`,
      },
      {
        role: 'user',
        content: prompt,
      },
    ],
    temperature: 0.8,
    max_tokens: 3000,
  });

  return completion.choices[0].message.content;
}

// Extract key points for video
async function extractVideoKeyPoints(article: string) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content:
          'Extract 5-7 key points from the article that can be visualized in a short video. Return as JSON array.',
      },
      {
        role: 'user',
        content: article,
      },
    ],
    response_format: { type: 'json_object' },
  });

  return JSON.parse(completion.choices[0].message.content || '{}');
}
```

### 4. Remotion Video Rendering

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import { webpack } from '@remotion/bundler';
import path from 'path';

interface VideoConfig {
  keyPoints: string[];
  title: string;
  duration: number;
  platform: 'reels' | 'tiktok' | 'shorts';
}

// Platform-specific dimensions
const PLATFORM_SPECS = {
  reels: { width: 1080, height: 1920, fps: 30 },
  tiktok: { width: 1080, height: 1920, fps: 30 },
  shorts: { width: 1080, height: 1920, fps: 30 },
};

async function renderContentVideo(config: VideoConfig) {
  const specs = PLATFORM_SPECS[config.platform];

  // Bundle Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  // Get composition
  const compositions = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      keyPoints: config.keyPoints,
      title: config.title,
    },
  });

  // Render video
  const outputPath = path.resolve(
    `./output/video-${Date.now()}-${config.platform}.mp4`
  );

  await renderMedia({
    composition: compositions,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: {
      keyPoints: config.keyPoints,
      title: config.title,
    },
    ...specs,
  });

  return outputPath;
}

// Render for multiple platforms
async function renderMultiPlatform(keyPoints: string[], title: string) {
  const platforms: Array<'reels' | 'tiktok' | 'shorts'> = [
    'reels',
    'tiktok',
    'shorts',
  ];

  const videos = await Promise.all(
    platforms.map((platform) =>
      renderContentVideo({
        keyPoints,
        title,
        duration: 60,
        platform,
      })
    )
  );

  return videos;
}
```

### 5. Complete Pipeline Orchestration

```typescript
import { ContentPipeline } from '@/lib/pipeline';

interface PipelineConfig {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  languages: Array<'en' | 'vi'>;
  generateVideo: boolean;
  platforms?: Array<'reels' | 'tiktok' | 'shorts'>;
}

async function runContentPipeline(config: PipelineConfig) {
  const pipeline = new ContentPipeline();

  try {
    // Step 1: Research
    console.log('🔍 Gathering research data...');
    const research = await pipeline.research(config.keyword);

    // Step 2: Generate Content
    console.log('✍️ Generating content...');
    const content = await pipeline.generateContent({
      research,
      format: config.format,
      languages: config.languages,
    });

    // Step 3: Extract insights for video
    if (config.generateVideo) {
      console.log('🎬 Extracting video key points...');
      const keyPoints = await pipeline.extractKeyPoints(
        content[config.languages[0]]
      );

      // Step 4: Render videos
      console.log('🎥 Rendering videos...');
      const videos = await pipeline.renderVideos({
        keyPoints,
        title: config.keyword,
        platforms: config.platforms || ['reels'],
      });

      return { content, videos };
    }

    return { content };
  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}

// Example: Full pipeline execution
const result = await runContentPipeline({
  keyword: 'AI Marketing Trends 2024',
  format: 'toplist',
  languages: ['en', 'vi'],
  generateVideo: true,
  platforms: ['reels', 'tiktok', 'shorts'],
});

console.log('✅ Pipeline complete!');
console.log('Content generated in:', Object.keys(result.content));
console.log('Videos rendered:', result.videos?.length);
```

## API Routes (Next.js)

### Create Content API

```typescript
// app/api/content/create/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const { keyword, format, languages, generateVideo } = body;

    // Validate input
    if (!keyword || !format) {
      return NextResponse.json(
        { error: 'Missing required fields' },
        { status: 400 }
      );
    }

    // Run pipeline
    const result = await runContentPipeline({
      keyword,
      format,
      languages: languages || ['en'],
      generateVideo: generateVideo || false,
    });

    return NextResponse.json({
      success: true,
      data: result,
    });
  } catch (error) {
    console.error('Content creation error:', error);
    return NextResponse.json(
      { error: 'Failed to create content' },
      { status: 500 }
    );
  }
}
```

### Research API

```typescript
// app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { NewsGatherer } from '@/lib/crawler/news-gatherer';

export async function GET(request: NextRequest) {
  const searchParams = request.nextUrl.searchParams;
  const keyword = searchParams.get('keyword');

  if (!keyword) {
    return NextResponse.json(
      { error: 'Keyword is required' },
      { status: 400 }
    );
  }

  const gatherer = new NewsGatherer({
    sources: ['techcrunch', 'a16z', 'twitter'],
    rapidApiKey: process.env.RAPIDAPI_KEY,
  });

  const results = await gatherer.search({
    keyword,
    timeRange: '24h',
    limit: 20,
  });

  return NextResponse.json({
    success: true,
    count: results.length,
    data: results,
  });
}
```

## Common Patterns

### Pattern 1: Batch Content Creation

```typescript
async function batchCreateContent(keywords: string[]) {
  const results = await Promise.allSettled(
    keywords.map((keyword) =>
      runContentPipeline({
        keyword,
        format: 'toplist',
        languages: ['en', 'vi'],
        generateVideo: false,
      })
    )
  );

  const successful = results.filter((r) => r.status === 'fulfilled');
  const failed = results.filter((r) => r.status === 'rejected');

  return {
    successful: successful.length,
    failed: failed.length,
    results,
  };
}
```

### Pattern 2: Content Scheduling

```typescript
import cron from 'node-cron';

// Schedule content generation daily at 9 AM
cron.schedule('0 9 * * *', async () => {
  console.log('Running scheduled content generation...');

  const keywords = ['AI trends', 'Marketing automation', 'SEO tips'];

  for (const keyword of keywords) {
    try {
      await runContentPipeline({
        keyword,
        format: 'toplist',
        languages: ['en'],
        generateVideo: true,
      });
    } catch (error) {
      console.error(`Failed to generate content for ${keyword}:`, error);
    }
  }
});
```

### Pattern 3: Custom Content Templates

```typescript
interface ContentTemplate {
  name: string;
  systemPrompt: string;
  structure: string[];
}

const templates: Record<string, ContentTemplate> = {
  toplist: {
    name: 'Top List',
    systemPrompt:
      'Create a numbered list article with detailed explanations for each point.',
    structure: ['Introduction', 'List Items (5-10)', 'Conclusion', 'CTA'],
  },
  howto: {
    name: 'How-To Guide',
    systemPrompt: 'Write a step-by-step tutorial with actionable instructions.',
    structure: ['Overview', 'Prerequisites', 'Steps', 'Tips', 'Conclusion'],
  },
};

async function generateFromTemplate(
  templateName: string,
  researchData: any[]
) {
  const template = templates[templateName];

  const content = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    system: `${template.systemPrompt}\n\nStructure: ${template.structure.join(' → ')}`,
    messages: [
      {
        role: 'user',
        content: `Create content based on: ${JSON.stringify(researchData)}`,
      },
    ],
  });

  return content;
}
```

## Running the Development Server

```bash
# Start Next.js development server
npm run dev

# Start Remotion studio (for video editing)
npm run remotion

# Build for production
npm run build

# Start production server
npm start
```

## Troubleshooting

### Issue: API Rate Limits

```typescript
// Implement rate limiting and retries
import pRetry from 'p-retry';

async function callAPIWithRetry(apiCall: () => Promise<any>) {
  return pRetry(
    async () => {
      try {
        return await apiCall();
      } catch (error: any) {
        if (error?.status === 429) {
          throw error; // Retry on rate limit
        }
        throw new pRetry.AbortError(error); // Don't retry other errors
      }
    },
    {
      retries: 3,
      minTimeout: 2000,
      factor: 2,
    }
  );
}
```

### Issue: Video Rendering Timeout

```typescript
// Increase timeout for long videos
await renderMedia({
  composition: compositions,
  serveUrl: bundleLocation,
  codec: 'h264',
  outputLocation: outputPath,
  timeoutInMilliseconds: 300000, // 5 minutes
  onProgress: ({ progress }) => {
    console.log(`Rendering progress: ${Math.round(progress * 100)}%`);
  },
});
```

### Issue: Memory Issues with Large Crawls

```typescript
// Process in batches
async function crawlInBatches(keywords: string[], batchSize: number = 5) {
  const results = [];

  for (let i = 0; i < keywords.length; i += batchSize) {
    const batch = keywords.slice(i, i + batchSize);
    const batchResults = await Promise.all(
      batch.map((keyword) => gatherTrendingContent(keyword))
    );
    results.push(...batchResults);

    // Clear memory between batches
    if (global.gc) {
      global.gc();
    }
  }

  return results;
}
```

### Issue: Claude/OpenAI Token Limits

```typescript
// Chunk large content
function chunkContent(content: string, maxTokens: number = 3000): string[] {
  const words = content.split(' ');
  const chunks: string[] = [];
  let currentChunk: string[] = [];
  let currentLength = 0;

  for (const word of words) {
    // Rough token estimation (1 token ≈ 4 chars)
    if (currentLength + word.length > maxTokens * 4) {
      chunks.push(currentChunk.join(' '));
      currentChunk = [word];
      currentLength = word.length;
    } else {
      currentChunk.push(word);
      currentLength += word.length;
    }
  }

  if (currentChunk.length > 0) {
    chunks.push(currentChunk.join(' '));
  }

  return chunks;
}
```

## Best Practices

1. **Always validate environment variables on startup**
2. **Implement proper error handling and logging**
3. **Cache research results to avoid redundant API calls**
4. **Use queues for video rendering to prevent memory issues**
5. **Monitor API usage and costs with tracking middleware**
6. **Store generated content with metadata for analytics**
7. **Test video templates in Remotion studio before automation**

This skill provides comprehensive guidance for implementing and extending the Ultimate AI Content Pipeline in any AI-assisted development workflow.
