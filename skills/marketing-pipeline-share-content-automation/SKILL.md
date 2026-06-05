---
name: marketing-pipeline-share-content-automation
description: Automated AI content pipeline from research to script generation to video using Claude, OpenAI, and Remotion
triggers:
  - automate content creation from research to video
  - generate content with AI pipeline automation
  - create marketing content with Claude and OpenAI
  - build automated content workflow with research
  - generate videos from text content automatically
  - set up AI content generation pipeline
  - crawl news and generate content scripts
  - automate social media content with AI
---

# Marketing Pipeline Share - AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## What This Project Does

Marketing Pipeline Share is an end-to-end AI-powered content automation system that:
- **Auto-researches** trending topics by crawling news sources (TechCrunch, a16z, Twitter/X, LinkedIn)
- **Generates content** in multiple formats (toplist, POV, case study, how-to) using Claude 3 or OpenAI
- **Creates bilingual content** (English & Vietnamese) with customizable tone
- **Renders videos** automatically using Remotion for social media platforms (Reels, TikTok, Shorts)
- **Manages workflow** through a Next.js dashboard

This is a complete content factory that transforms keywords into publication-ready articles and videos.

## Installation

### Prerequisites
```bash
node >= 18.0.0
npm or yarn
```

### Clone and Setup
```bash
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share
npm install
# or
yarn install
```

### Environment Configuration

Create `.env.local` in the project root:

```bash
# AI Providers
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research/Crawling API
RAPIDAPI_KEY=your_rapidapi_key

# Next.js Configuration
NEXT_PUBLIC_APP_URL=http://localhost:3000

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Remotion (Video Generation)
REMOTION_AWS_ACCESS_KEY_ID=your_aws_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret
```

### Run Development Server
```bash
npm run dev
# or
yarn dev
```

Access at `http://localhost:3000`

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app router pages
│   ├── components/       # React components
│   ├── lib/              # Core utilities
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── crawler/     # News crawling logic
│   │   ├── generator/   # Content generation
│   │   └── video/       # Remotion video rendering
│   ├── types/           # TypeScript type definitions
│   └── config/          # Configuration files
├── public/              # Static assets
├── remotion/            # Remotion video templates
└── .env.local          # Environment variables
```

## Core API & Usage Patterns

### 1. Content Research Pipeline

```typescript
import { ResearchService } from '@/lib/crawler/research-service';

// Initialize research service
const researchService = new ResearchService({
  rapidApiKey: process.env.RAPIDAPI_KEY!,
  sources: ['techcrunch', 'a16z', 'twitter', 'linkedin']
});

// Crawl trending topics
async function researchTopic(keyword: string) {
  const results = await researchService.crawl({
    keyword,
    timeframe: '24h',
    limit: 20
  });

  return {
    articles: results.articles,
    insights: results.insights,
    dataPoints: results.statistics
  };
}

// Usage
const research = await researchTopic('AI automation');
console.log(research.insights);
```

### 2. AI Content Generation with Claude

```typescript
import { AnthropicService } from '@/lib/ai/anthropic';
import type { ContentFormat, Tone } from '@/types/content';

const claudeService = new AnthropicService(
  process.env.ANTHROPIC_API_KEY!
);

// Generate content
async function generateContent(
  researchData: any,
  format: ContentFormat = 'toplist',
  tone: Tone = 'professional'
) {
  const prompt = `
You are a content creator. Based on this research data:
${JSON.stringify(researchData, null, 2)}

Create a ${format} article with a ${tone} tone.
Include:
- Engaging headline
- 5-7 key points with data-backed insights
- Actionable takeaways
- SEO-optimized structure

Language: English and Vietnamese (bilingual)
`;

  const response = await claudeService.generate({
    model: 'claude-3-sonnet-20240229',
    prompt,
    maxTokens: 4000,
    temperature: 0.7
  });

  return response.content;
}
```

### 3. OpenAI Alternative

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY
});

async function generateWithOpenAI(
  researchData: any,
  format: string
) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: 'You are an expert content marketer specializing in data-driven articles.'
      },
      {
        role: 'user',
        content: `Create a ${format} article from: ${JSON.stringify(researchData)}`
      }
    ],
    temperature: 0.7,
    max_tokens: 3000
  });

  return completion.choices[0].message.content;
}
```

### 4. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

// Define video composition
interface VideoProps {
  title: string;
  keyPoints: string[];
  duration: number;
}

// Render video from content
async function renderContentVideo(
  content: VideoProps,
  outputPath: string
) {
  const compositionId = 'ContentVideo';
  
  // Bundle Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: (config) => config
  });

  // Get composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: content
  });

  // Render
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: content
  });

  return outputPath;
}

// Usage
const videoContent = {
  title: 'Top 5 AI Trends in 2024',
  keyPoints: [
    'Multimodal AI becomes mainstream',
    'AI agents automate workflows',
    'Open source models compete with closed'
  ],
  duration: 30
};

await renderContentVideo(videoContent, './output/video.mp4');
```

### 5. Complete Pipeline Workflow

```typescript
import { ContentPipeline } from '@/lib/generator/pipeline';

// Full automation pipeline
class ContentPipeline {
  constructor(
    private anthropic: AnthropicService,
    private research: ResearchService,
    private videoRenderer: VideoRenderer
  ) {}

  async execute(keyword: string, options: PipelineOptions) {
    // Step 1: Research
    console.log('🔍 Researching topic...');
    const research = await this.research.crawl({
      keyword,
      timeframe: options.timeframe || '24h'
    });

    // Step 2: Generate content
    console.log('✍️ Generating content...');
    const content = await this.anthropic.generate({
      prompt: this.buildPrompt(research, options.format),
      maxTokens: 4000
    });

    // Step 3: Parse structured content
    const structured = this.parseContent(content);

    // Step 4: Generate video
    console.log('🎬 Rendering video...');
    const videoPath = await this.videoRenderer.render({
      title: structured.title,
      keyPoints: structured.keyPoints,
      outputPath: `./videos/${keyword}-${Date.now()}.mp4`
    });

    return {
      article: structured,
      video: videoPath,
      metadata: {
        keyword,
        generatedAt: new Date().toISOString(),
        sources: research.sources
      }
    };
  }
}

// Execute pipeline
const pipeline = new ContentPipeline(
  claudeService,
  researchService,
  videoRenderer
);

const result = await pipeline.execute('AI marketing automation', {
  format: 'toplist',
  timeframe: '24h',
  tone: 'professional'
});
```

## API Routes (Next.js)

### Generate Content Endpoint

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { ContentPipeline } from '@/lib/generator/pipeline';

export async function POST(req: NextRequest) {
  try {
    const { keyword, format, language } = await req.json();

    // Validate
    if (!keyword) {
      return NextResponse.json(
        { error: 'Keyword is required' },
        { status: 400 }
      );
    }

    // Initialize pipeline
    const pipeline = new ContentPipeline(/* dependencies */);

    // Generate
    const result = await pipeline.execute(keyword, {
      format: format || 'toplist',
      language: language || 'en'
    });

    return NextResponse.json({
      success: true,
      data: result
    });
  } catch (error) {
    console.error('Generation error:', error);
    return NextResponse.json(
      { error: 'Failed to generate content' },
      { status: 500 }
    );
  }
}
```

### Research Endpoint

```typescript
// src/app/api/research/route.ts
export async function GET(req: NextRequest) {
  const { searchParams } = new URL(req.url);
  const keyword = searchParams.get('keyword');
  const timeframe = searchParams.get('timeframe') || '24h';

  const researchService = new ResearchService({
    rapidApiKey: process.env.RAPIDAPI_KEY!
  });

  const results = await researchService.crawl({
    keyword: keyword!,
    timeframe
  });

  return NextResponse.json(results);
}
```

## Configuration Options

### Content Format Types

```typescript
type ContentFormat = 
  | 'toplist'      // Top 5/10 format
  | 'pov'          // Opinion/perspective piece
  | 'case-study'   // Deep dive analysis
  | 'how-to'       // Tutorial/guide
  | 'news-roundup' // News aggregation

type Tone = 
  | 'professional'
  | 'casual'
  | 'humorous'
  | 'authoritative'
  | 'friendly'

interface ContentConfig {
  format: ContentFormat;
  tone: Tone;
  languages: ('en' | 'vi')[];
  includeVideo: boolean;
  videoAspectRatio: '16:9' | '9:16' | '1:1';
  seoOptimized: boolean;
}
```

### Research Configuration

```typescript
interface ResearchConfig {
  sources: Array<'techcrunch' | 'a16z' | 'twitter' | 'linkedin'>;
  timeframe: '24h' | '7d' | '30d';
  limit: number;
  minEngagement?: number;
  language?: string;
}
```

## Common Workflows

### Workflow 1: Quick Article Generation

```typescript
import { quickGenerate } from '@/lib/shortcuts';

// One-liner for quick content
const article = await quickGenerate({
  keyword: 'AI automation tools',
  format: 'toplist'
});

console.log(article.title);
console.log(article.content);
```

### Workflow 2: Scheduled Content Pipeline

```typescript
// Cron job or scheduled task
import { schedule } from 'node-cron';

schedule('0 9 * * *', async () => {
  const keywords = ['AI trends', 'marketing automation', 'content strategy'];
  
  for (const keyword of keywords) {
    const result = await pipeline.execute(keyword, {
      format: 'news-roundup',
      includeVideo: true
    });
    
    // Save to database or publish
    await publishContent(result);
  }
});
```

### Workflow 3: Batch Video Generation

```typescript
async function batchGenerateVideos(articles: Article[]) {
  const videos = await Promise.all(
    articles.map(article => 
      renderContentVideo({
        title: article.title,
        keyPoints: article.keyPoints,
        duration: 30
      }, `./videos/${article.slug}.mp4`)
    )
  );
  
  return videos;
}
```

## Troubleshooting

### Issue: API Rate Limits

```typescript
// Implement retry logic with exponential backoff
async function retryWithBackoff<T>(
  fn: () => Promise<T>,
  maxRetries = 3
): Promise<T> {
  let lastError;
  
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error: any) {
      lastError = error;
      
      if (error.status === 429) {
        const delay = Math.pow(2, i) * 1000;
        console.log(`Rate limited. Retrying in ${delay}ms...`);
        await new Promise(resolve => setTimeout(resolve, delay));
        continue;
      }
      
      throw error;
    }
  }
  
  throw lastError;
}

// Usage
const content = await retryWithBackoff(() =>
  claudeService.generate({ prompt: '...' })
);
```

### Issue: Video Rendering Memory Errors

```typescript
// Use Remotion Lambda for heavy rendering
import { renderMediaOnLambda } from '@remotion/lambda';

const result = await renderMediaOnLambda({
  region: 'us-east-1',
  functionName: 'remotion-render',
  composition: compositionId,
  serveUrl: bundleLocation,
  codec: 'h264',
  inputProps: videoContent
});
```

### Issue: Crawler Blocked/Captcha

```typescript
// Use proxy rotation
const researchService = new ResearchService({
  rapidApiKey: process.env.RAPIDAPI_KEY!,
  proxyList: [
    process.env.PROXY_1,
    process.env.PROXY_2,
    process.env.PROXY_3
  ],
  rotateProxy: true
});
```

### Issue: Content Quality Inconsistent

```typescript
// Add validation layer
function validateContent(content: string): boolean {
  const checks = {
    minLength: content.length > 500,
    hasHeadline: /^#/.test(content),
    hasKeyPoints: (content.match(/\n-/g) || []).length >= 3,
    noPlaceholders: !content.includes('[INSERT]')
  };
  
  return Object.values(checks).every(Boolean);
}

// Regenerate if invalid
let content = await generateContent(research, format);
let attempts = 0;

while (!validateContent(content) && attempts < 3) {
  content = await generateContent(research, format);
  attempts++;
}
```

## CLI Commands (if applicable)

```bash
# Generate content from keyword
npm run generate -- --keyword "AI trends" --format toplist

# Research only
npm run research -- --keyword "marketing automation" --timeframe 7d

# Batch process from CSV
npm run batch -- --input keywords.csv --output ./generated

# Render video from JSON
npm run render-video -- --input content.json --output video.mp4
```

## Testing

```typescript
// Example test
import { ContentPipeline } from '@/lib/generator/pipeline';

describe('ContentPipeline', () => {
  it('should generate complete content from keyword', async () => {
    const result = await pipeline.execute('test keyword', {
      format: 'toplist'
    });
    
    expect(result.article).toBeDefined();
    expect(result.article.title).toBeTruthy();
    expect(result.video).toMatch(/\.mp4$/);
  });
});
```

Run tests:
```bash
npm test
# or
yarn test
```
