---
name: marketing-pipeline-share-automation
description: Automated AI content pipeline for research, script generation, social media posting, and video creation using Claude/OpenAI and Remotion
triggers:
  - how do I automatically generate content with AI research
  - set up automated marketing content pipeline
  - create videos from blog posts automatically
  - use marketing pipeline share for content automation
  - generate multilingual content with AI crawling
  - automate social media content creation workflow
  - build AI-powered content generation system
  - create automated content from trending topics
---

# Marketing Pipeline Share Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the **Ultimate AI Content Pipeline** (marketing-pipeline-share), a TypeScript-based automation system that handles the complete content creation workflow: from web scraping research, to AI-powered script generation, automated social media posting, and video rendering using Remotion.

## What It Does

The marketing-pipeline-share project is an all-in-one content automation pipeline that:

- **Auto-scans trending topics** from sources like TechCrunch, a16z, Twitter/X, and LinkedIn
- **Generates content** in multiple formats (toplist, POV, case study, how-to) using Claude 3 or OpenAI
- **Supports multilingual output** (English and Vietnamese by default)
- **Renders videos and infographics** automatically using Remotion
- **Optimizes for multiple platforms** (Reels, TikTok, Shorts)

## Project Structure

```
marketing-pipeline-share/
├── src/
│   ├── app/              # Next.js application routes
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI service integrations (Claude, OpenAI)
│   │   ├── scraper/     # Web scraping modules
│   │   ├── content/     # Content generation logic
│   │   └── video/       # Remotion video rendering
│   ├── config/          # Configuration files
│   └── types/           # TypeScript definitions
├── remotion/            # Video templates
└── public/              # Static assets
```

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
cp .env.example .env.local
```

## Environment Configuration

Configure the following environment variables in `.env.local`:

```bash
# AI Provider Keys
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Scraping API (RapidAPI)
RAPIDAPI_KEY=your_rapidapi_key

# Social Media APIs (optional)
FACEBOOK_ACCESS_TOKEN=your_facebook_token
LINKEDIN_ACCESS_TOKEN=your_linkedin_token

# Remotion (for video rendering)
REMOTION_LICENSE_KEY=your_remotion_license

# Database (if using)
DATABASE_URL=your_database_connection_string

# Next.js
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

## Key Components & Usage

### 1. Content Research Scraper

```typescript
// src/lib/scraper/research-crawler.ts
import { RapidAPIClient } from './rapidapi-client';

interface ResearchResult {
  title: string;
  url: string;
  summary: string;
  publishedAt: Date;
  source: string;
}

export class ResearchCrawler {
  private apiClient: RapidAPIClient;

  constructor(apiKey: string) {
    this.apiClient = new RapidAPIClient(apiKey);
  }

  async searchTrendingTopics(
    keyword: string,
    sources: string[] = ['techcrunch', 'a16z', 'twitter'],
    timeRange: '24h' | '7d' = '24h'
  ): Promise<ResearchResult[]> {
    const results: ResearchResult[] = [];

    for (const source of sources) {
      const articles = await this.apiClient.fetchArticles({
        source,
        keyword,
        timeRange,
      });

      results.push(
        ...articles.map((article) => ({
          title: article.title,
          url: article.url,
          summary: article.description || '',
          publishedAt: new Date(article.publishedAt),
          source,
        }))
      );
    }

    return results.sort(
      (a, b) => b.publishedAt.getTime() - a.publishedAt.getTime()
    );
  }

  async extractInsights(results: ResearchResult[]): Promise<string[]> {
    // Extract key insights from research results
    const insights = results.map((r) => `${r.title}: ${r.summary}`);
    return insights;
  }
}

// Usage
const crawler = new ResearchCrawler(process.env.RAPIDAPI_KEY!);
const trends = await crawler.searchTrendingTopics('AI automation', [
  'techcrunch',
  'a16z',
]);
```

### 2. AI Content Generation

```typescript
// src/lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

type AIProvider = 'claude' | 'openai';
type ContentFormat = 'toplist' | 'pov' | 'case-study' | 'how-to';
type ToneOfVoice = 'expert' | 'friendly' | 'humorous';

interface ContentGenerationParams {
  keyword: string;
  format: ContentFormat;
  tone: ToneOfVoice;
  language: 'en' | 'vi';
  researchData: string[];
  provider?: AIProvider;
}

export class ContentGenerator {
  private claude: Anthropic;
  private openai: OpenAI;

  constructor() {
    this.claude = new Anthropic({
      apiKey: process.env.ANTHROPIC_API_KEY,
    });
    this.openai = new OpenAI({
      apiKey: process.env.OPENAI_API_KEY,
    });
  }

  async generate(params: ContentGenerationParams): Promise<string> {
    const prompt = this.buildPrompt(params);

    if (params.provider === 'openai') {
      return this.generateWithOpenAI(prompt);
    }

    return this.generateWithClaude(prompt);
  }

  private buildPrompt(params: ContentGenerationParams): string {
    const formatInstructions = {
      toplist: 'Create a top 10 list article',
      pov: 'Write from a unique point of view perspective',
      'case-study': 'Develop a detailed case study analysis',
      'how-to': 'Write a step-by-step how-to guide',
    };

    const toneInstructions = {
      expert: 'Use professional, authoritative language',
      friendly: 'Use casual, approachable language',
      humorous: 'Use witty, entertaining language',
    };

    return `
You are a content writer creating a ${params.format} article about "${params.keyword}".

${formatInstructions[params.format]}.
${toneInstructions[params.tone]}.
Write in ${params.language === 'en' ? 'English' : 'Vietnamese'}.

Use the following research data as reference:
${params.researchData.join('\n\n')}

Create a comprehensive article with:
- Engaging headline
- Clear introduction
- Well-structured body with data-backed insights
- Actionable conclusion
- SEO-optimized content
`;
  }

  private async generateWithClaude(prompt: string): Promise<string> {
    const message = await this.claude.messages.create({
      model: 'claude-3-5-sonnet-20241022',
      max_tokens: 4096,
      messages: [{ role: 'user', content: prompt }],
    });

    return message.content[0].type === 'text'
      ? message.content[0].text
      : '';
  }

  private async generateWithOpenAI(prompt: string): Promise<string> {
    const completion = await this.openai.chat.completions.create({
      model: 'gpt-4-turbo-preview',
      messages: [{ role: 'user', content: prompt }],
      max_tokens: 4096,
    });

    return completion.choices[0]?.message?.content || '';
  }
}

// Usage
const generator = new ContentGenerator();
const article = await generator.generate({
  keyword: 'AI automation trends 2026',
  format: 'toplist',
  tone: 'expert',
  language: 'en',
  researchData: trends.map((t) => t.summary),
  provider: 'claude',
});
```

### 3. Video Generation with Remotion

```typescript
// src/lib/video/video-renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  title: string;
  content: string[];
  aspectRatio: '9:16' | '16:9' | '1:1';
  platform: 'reels' | 'tiktok' | 'youtube';
}

export class VideoRenderer {
  async renderContentVideo(config: VideoConfig): Promise<string> {
    const compositionId = this.getCompositionId(config.platform);
    const bundleLocation = await bundle(
      path.join(process.cwd(), 'remotion/index.ts')
    );

    const composition = await selectComposition({
      serveUrl: bundleLocation,
      id: compositionId,
      inputProps: {
        title: config.title,
        content: config.content,
        aspectRatio: config.aspectRatio,
      },
    });

    const outputPath = path.join(
      process.cwd(),
      'output',
      `${Date.now()}-${config.platform}.mp4`
    );

    await renderMedia({
      composition,
      serveUrl: bundleLocation,
      codec: 'h264',
      outputLocation: outputPath,
      inputProps: {
        title: config.title,
        content: config.content,
      },
    });

    return outputPath;
  }

  private getCompositionId(platform: string): string {
    const compositions = {
      reels: 'InstagramReels',
      tiktok: 'TikTokVideo',
      youtube: 'YouTubeShorts',
    };
    return compositions[platform as keyof typeof compositions];
  }
}

// Usage
const renderer = new VideoRenderer();
const videoPath = await renderer.renderContentVideo({
  title: 'Top 10 AI Automation Trends',
  content: article.split('\n').filter((line) => line.trim()),
  aspectRatio: '9:16',
  platform: 'reels',
});
```

### 4. Complete Pipeline Orchestration

```typescript
// src/lib/pipeline/content-pipeline.ts
import { ResearchCrawler } from '../scraper/research-crawler';
import { ContentGenerator } from '../ai/content-generator';
import { VideoRenderer } from '../video/video-renderer';

export class ContentPipeline {
  private crawler: ResearchCrawler;
  private generator: ContentGenerator;
  private renderer: VideoRenderer;

  constructor() {
    this.crawler = new ResearchCrawler(process.env.RAPIDAPI_KEY!);
    this.generator = new ContentGenerator();
    this.renderer = new VideoRenderer();
  }

  async execute(keyword: string) {
    console.log(`🔍 Starting pipeline for: ${keyword}`);

    // Step 1: Research
    console.log('📡 Crawling trending topics...');
    const research = await this.crawler.searchTrendingTopics(keyword);
    const insights = await this.crawler.extractInsights(research);

    // Step 2: Generate Content (English & Vietnamese)
    console.log('🧠 Generating content...');
    const [englishContent, vietnameseContent] = await Promise.all([
      this.generator.generate({
        keyword,
        format: 'toplist',
        tone: 'expert',
        language: 'en',
        researchData: insights,
        provider: 'claude',
      }),
      this.generator.generate({
        keyword,
        format: 'toplist',
        tone: 'expert',
        language: 'vi',
        researchData: insights,
        provider: 'claude',
      }),
    ]);

    // Step 3: Render Videos
    console.log('🎬 Rendering videos...');
    const videos = await Promise.all([
      this.renderer.renderContentVideo({
        title: keyword,
        content: englishContent.split('\n').slice(0, 10),
        aspectRatio: '9:16',
        platform: 'reels',
      }),
      this.renderer.renderContentVideo({
        title: keyword,
        content: vietnameseContent.split('\n').slice(0, 10),
        aspectRatio: '9:16',
        platform: 'tiktok',
      }),
    ]);

    console.log('✅ Pipeline completed!');
    return {
      research,
      content: { english: englishContent, vietnamese: vietnameseContent },
      videos,
    };
  }
}

// Usage in API route or script
const pipeline = new ContentPipeline();
const result = await pipeline.execute('AI content automation');
```

## Next.js API Routes

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { ContentPipeline } from '@/lib/pipeline/content-pipeline';

export async function POST(request: NextRequest) {
  try {
    const { keyword, format, language } = await request.json();

    if (!keyword) {
      return NextResponse.json(
        { error: 'Keyword is required' },
        { status: 400 }
      );
    }

    const pipeline = new ContentPipeline();
    const result = await pipeline.execute(keyword);

    return NextResponse.json({
      success: true,
      data: result,
    });
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { error: 'Failed to generate content' },
      { status: 500 }
    );
  }
}
```

## Running the Application

```bash
# Development mode
npm run dev

# Build for production
npm run build

# Start production server
npm run start

# Render videos only (if separate script)
npm run render
```

## Common Patterns

### Pattern 1: Scheduled Content Generation

```typescript
// src/scripts/scheduled-pipeline.ts
import cron from 'node-cron';
import { ContentPipeline } from '../lib/pipeline/content-pipeline';

const keywords = [
  'AI automation',
  'Marketing trends',
  'Content creation tools',
];

// Run every day at 6 AM
cron.schedule('0 6 * * *', async () => {
  const pipeline = new ContentPipeline();

  for (const keyword of keywords) {
    try {
      await pipeline.execute(keyword);
      console.log(`✅ Completed: ${keyword}`);
    } catch (error) {
      console.error(`❌ Failed: ${keyword}`, error);
    }
  }
});
```

### Pattern 2: Batch Processing with Queue

```typescript
// src/lib/queue/content-queue.ts
import Bull from 'bull';
import { ContentPipeline } from '../pipeline/content-pipeline';

const contentQueue = new Bull('content-generation', {
  redis: {
    host: process.env.REDIS_HOST || 'localhost',
    port: parseInt(process.env.REDIS_PORT || '6379'),
  },
});

contentQueue.process(async (job) => {
  const { keyword } = job.data;
  const pipeline = new ContentPipeline();
  return await pipeline.execute(keyword);
});

// Add job to queue
export async function queueContentGeneration(keyword: string) {
  return await contentQueue.add({ keyword });
}
```

### Pattern 3: Custom Content Templates

```typescript
// src/lib/ai/templates/custom-template.ts
export const customTemplates = {
  productReview: (product: string, features: string[]) => `
Write a comprehensive product review for ${product}.

Key features to cover:
${features.map((f, i) => `${i + 1}. ${f}`).join('\n')}

Include:
- Overview and first impressions
- Detailed feature analysis
- Pros and cons
- Value for money assessment
- Final verdict and rating
`,

  trendAnalysis: (trend: string, data: string[]) => `
Analyze the trend: ${trend}

Based on these data points:
${data.join('\n')}

Provide:
- Current state of the trend
- Key drivers and factors
- Future predictions
- Actionable insights for businesses
`,
};
```

## Troubleshooting

### Issue: AI API Rate Limits

```typescript
// Implement retry logic with exponential backoff
async function withRetry<T>(
  fn: () => Promise<T>,
  maxRetries = 3
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error: any) {
      if (error.status === 429 && i < maxRetries - 1) {
        const delay = Math.pow(2, i) * 1000;
        await new Promise((resolve) => setTimeout(resolve, delay));
        continue;
      }
      throw error;
    }
  }
  throw new Error('Max retries reached');
}
```

### Issue: Remotion Memory Errors

Reduce composition complexity or increase Node memory:

```bash
# In package.json scripts
"render": "NODE_OPTIONS=--max-old-space-size=4096 node render-script.js"
```

### Issue: Scraping Blocks

Rotate user agents or use proxy:

```typescript
const scraperConfig = {
  headers: {
    'User-Agent':
      'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36',
  },
  proxy: process.env.PROXY_URL, // Optional
};
```

### Issue: Missing Environment Variables

Add validation at startup:

```typescript
// src/lib/config/validate-env.ts
const requiredEnvVars = [
  'ANTHROPIC_API_KEY',
  'OPENAI_API_KEY',
  'RAPIDAPI_KEY',
];

export function validateEnv() {
  const missing = requiredEnvVars.filter((key) => !process.env[key]);

  if (missing.length > 0) {
    throw new Error(`Missing required env vars: ${missing.join(', ')}`);
  }
}
```

## Best Practices

1. **Cache Research Results**: Store crawled data to avoid redundant API calls
2. **Use Queue Systems**: Process heavy tasks (video rendering) asynchronously
3. **Implement Rate Limiting**: Respect API limits with proper throttling
4. **Monitor Costs**: Track API usage for Claude/OpenAI to manage expenses
5. **Version Content**: Keep track of generated content for A/B testing
6. **Optimize Remotion**: Use lazy loading and code splitting in video compositions
