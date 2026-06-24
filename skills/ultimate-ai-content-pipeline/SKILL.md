---
name: ultimate-ai-content-pipeline
description: Automated content creation pipeline from research to video generation using AI (Claude/OpenAI) and Remotion
triggers:
  - how do I automate content creation with AI research
  - generate blog posts from trending news automatically
  - create videos from written content using Remotion
  - set up AI content pipeline with Claude and OpenAI
  - automate research and content generation workflow
  - build content marketing automation system
  - generate multilingual content with AI
  - create social media videos from blog posts
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

Ultimate AI Content Pipeline is an end-to-end content automation system that transforms keywords into complete content pieces including research, articles, and videos. It automatically crawls trending news from sources like TechCrunch, a16z, Twitter/X, and LinkedIn, generates multilingual content using Claude 3 and OpenAI, and renders videos using Remotion.

## Core Capabilities

- **Auto-Research**: Crawls real-time data from news sources and social media (24h window)
- **AI Content Generation**: Creates multiple content formats (Toplist, POV, Case Study, How-to) in multiple languages
- **Video Rendering**: Automatically generates infographics and short-form videos from content
- **Multi-Platform**: Optimized output for Reels, TikTok, Shorts, and blog posts

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

```bash
# AI Models
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Optional: Database
DATABASE_URL=your_database_connection_string

# Next.js
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── app/                    # Next.js app directory
├── components/             # React components
├── lib/                    # Core utilities
│   ├── ai/                # AI integration (Claude, OpenAI)
│   ├── crawler/           # Web scraping modules
│   └── video/             # Remotion video generation
├── remotion/              # Remotion video templates
└── public/                # Static assets
```

## Core API Usage

### 1. Research & Data Collection

```typescript
import { crawlTrendingNews } from '@/lib/crawler/news-crawler';
import { analyzeTrends } from '@/lib/ai/trend-analyzer';

async function gatherResearch(keyword: string) {
  // Crawl trending news from multiple sources
  const newsData = await crawlTrendingNews({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeframe: '24h'
  });

  // Analyze trends with AI
  const insights = await analyzeTrends({
    data: newsData,
    model: 'claude-3-opus-20240229'
  });

  return {
    rawData: newsData,
    insights,
    timestamp: new Date().toISOString()
  };
}
```

### 2. AI Content Generation

```typescript
import { Anthropic } from '@anthropic-ai/sdk';
import { OpenAI } from 'openai';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY
});

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY
});

// Generate content with Claude
async function generateArticle(research: any, format: 'toplist' | 'pov' | 'case-study' | 'how-to') {
  const prompt = `
Based on the following research data, create a ${format} article:

${JSON.stringify(research, null, 2)}

Requirements:
- Engaging headline
- Data-backed insights
- Bilingual output (English & Vietnamese)
- Optimized for SEO
- Include actionable takeaways
`;

  const message = await anthropic.messages.create({
    model: 'claude-3-opus-20240229',
    max_tokens: 4096,
    messages: [
      { role: 'user', content: prompt }
    ]
  });

  return message.content[0].text;
}

// Alternative: Generate with OpenAI
async function generateWithOpenAI(research: any, tone: 'expert' | 'friendly' | 'humorous') {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are a ${tone} content writer specializing in marketing and tech trends.`
      },
      {
        role: 'user',
        content: `Create an article based on: ${JSON.stringify(research)}`
      }
    ],
    temperature: 0.7
  });

  return completion.choices[0].message.content;
}
```

### 3. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import { webpackOverride } from './remotion/webpack-override';
import path from 'path';

async function generateVideo(content: string, type: 'reel' | 'tiktok' | 'short') {
  const compositionId = `${type}-template`;
  
  // Bundle the Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride
  });

  // Get composition details
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: {
      content,
      duration: 30, // seconds
      aspectRatio: type === 'tiktok' ? '9:16' : '1:1'
    }
  });

  // Render the video
  const outputPath = path.join('./public/videos', `${Date.now()}.mp4`);
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: composition.defaultProps
  });

  return outputPath;
}
```

### 4. Complete Pipeline Workflow

```typescript
interface PipelineConfig {
  keyword: string;
  contentFormat: 'toplist' | 'pov' | 'case-study' | 'how-to';
  tone: 'expert' | 'friendly' | 'humorous';
  videoTypes: ('reel' | 'tiktok' | 'short')[];
  languages: ('en' | 'vi')[];
}

async function runContentPipeline(config: PipelineConfig) {
  try {
    // Step 1: Research
    console.log('🔍 Gathering research...');
    const research = await gatherResearch(config.keyword);

    // Step 2: Generate content
    console.log('✍️ Generating content...');
    const article = await generateArticle(research, config.contentFormat);

    // Step 3: Create videos
    console.log('🎬 Rendering videos...');
    const videos = await Promise.all(
      config.videoTypes.map(type => generateVideo(article, type))
    );

    // Step 4: Return complete package
    return {
      research: research.insights,
      article,
      videos,
      metadata: {
        keyword: config.keyword,
        generatedAt: new Date().toISOString(),
        format: config.contentFormat
      }
    };
  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}

// Usage
const result = await runContentPipeline({
  keyword: 'AI marketing automation',
  contentFormat: 'how-to',
  tone: 'expert',
  videoTypes: ['reel', 'tiktok'],
  languages: ['en', 'vi']
});
```

## Next.js API Routes

### Create Content Endpoint

```typescript
// app/api/content/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';

export async function POST(request: NextRequest) {
  try {
    const { keyword, format, videoEnabled } = await request.json();

    const config = {
      keyword,
      contentFormat: format,
      tone: 'expert',
      videoTypes: videoEnabled ? ['reel', 'tiktok'] : [],
      languages: ['en', 'vi']
    };

    const result = await runContentPipeline(config);

    return NextResponse.json({
      success: true,
      data: result
    });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

### Research Endpoint

```typescript
// app/api/research/route.ts
export async function GET(request: NextRequest) {
  const searchParams = request.nextUrl.searchParams;
  const keyword = searchParams.get('keyword');

  if (!keyword) {
    return NextResponse.json(
      { error: 'Keyword is required' },
      { status: 400 }
    );
  }

  const research = await gatherResearch(keyword);

  return NextResponse.json({
    success: true,
    data: research
  });
}
```

## Running the Application

```bash
# Development mode
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Render Remotion video (standalone)
npm run render -- --props='{"content":"Your content here"}'
```

## Common Patterns

### Batch Content Generation

```typescript
async function batchGenerate(keywords: string[]) {
  const results = [];

  for (const keyword of keywords) {
    const result = await runContentPipeline({
      keyword,
      contentFormat: 'toplist',
      tone: 'friendly',
      videoTypes: ['reel'],
      languages: ['en', 'vi']
    });

    results.push(result);

    // Rate limiting
    await new Promise(resolve => setTimeout(resolve, 2000));
  }

  return results;
}
```

### Custom Video Templates

```typescript
// remotion/CustomTemplate.tsx
import { AbsoluteFill, Sequence, useCurrentFrame } from 'remotion';

export const CustomTemplate: React.FC<{
  content: string;
  duration: number;
}> = ({ content, duration }) => {
  const frame = useCurrentFrame();

  return (
    <AbsoluteFill style={{ backgroundColor: '#000' }}>
      <Sequence from={0} durationInFrames={duration * 30}>
        <div style={{ 
          color: 'white', 
          fontSize: 48,
          padding: 40,
          opacity: frame / 30
        }}>
          {content}
        </div>
      </Sequence>
    </AbsoluteFill>
  );
};
```

### Error Handling & Retries

```typescript
async function generateWithRetry(
  fn: () => Promise<any>,
  maxRetries = 3
): Promise<any> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      
      console.log(`Attempt ${i + 1} failed, retrying...`);
      await new Promise(resolve => setTimeout(resolve, 1000 * (i + 1)));
    }
  }
}

// Usage
const article = await generateWithRetry(() =>
  generateArticle(research, 'toplist')
);
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement rate limiting
import pLimit from 'p-limit';

const limit = pLimit(2); // Max 2 concurrent requests

const results = await Promise.all(
  keywords.map(keyword =>
    limit(() => gatherResearch(keyword))
  )
);
```

### Video Rendering Memory Issues

```bash
# Increase Node.js memory limit
NODE_OPTIONS=--max-old-space-size=4096 npm run render
```

### Claude/OpenAI Timeouts

```typescript
// Set custom timeout
const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
  timeout: 60000, // 60 seconds
  maxRetries: 3
});
```

### Crawler Blocking

```typescript
// Add user agent and delays
const crawlerConfig = {
  headers: {
    'User-Agent': 'Mozilla/5.0 (compatible; ContentBot/1.0)'
  },
  delay: 1000, // 1 second between requests
  respectRobotsTxt: true
};
```

## Advanced Configuration

### Custom AI Prompts

```typescript
// lib/prompts/templates.ts
export const PROMPT_TEMPLATES = {
  toplist: `Create a top 10 list about {topic}...`,
  pov: `Write a thought-provoking POV on {topic}...`,
  caseStudy: `Analyze this case study about {topic}...`,
  howTo: `Create a step-by-step guide on {topic}...`
};

// Use custom prompt
const customPrompt = PROMPT_TEMPLATES.toplist.replace('{topic}', keyword);
```

### Multi-Language Support

```typescript
const LANGUAGE_CONFIGS = {
  en: { model: 'gpt-4', tone: 'professional' },
  vi: { model: 'claude-3-opus-20240229', tone: 'friendly' }
};

async function generateMultilingual(content: any) {
  return await Promise.all(
    Object.entries(LANGUAGE_CONFIGS).map(([lang, config]) =>
      generateInLanguage(content, lang, config)
    )
  );
}
```

## Performance Optimization

- Use streaming responses for real-time content generation
- Implement caching for research data (Redis recommended)
- Queue video rendering jobs for background processing
- Use CDN for serving generated videos
- Implement database for content versioning and history
