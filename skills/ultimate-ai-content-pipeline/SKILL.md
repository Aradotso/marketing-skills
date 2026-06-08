---
name: ultimate-ai-content-pipeline
description: Automated content creation pipeline from research to video generation using AI (Claude, OpenAI) and Remotion
triggers:
  - how do I automate content creation with AI research
  - generate video content from text automatically
  - set up AI content pipeline with Claude and OpenAI
  - crawl news and create social media content
  - automate blog post and video generation
  - create content from keyword to video
  - build automated marketing content system
  - generate multi-language content with AI
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

Ultimate AI Content Pipeline is a comprehensive TypeScript-based content automation system that transforms a single keyword into fully-formed content across multiple formats. It handles:

- **Auto-Research**: Crawls real-time data from TechCrunch, a16z, Twitter/X, LinkedIn
- **AI Content Generation**: Creates articles in multiple formats (toplist, POV, case study, how-to) using Claude 3 and OpenAI
- **Multi-language Support**: Generates content in English and Vietnamese simultaneously
- **Video/Image Rendering**: Automatically creates videos and infographics using Remotion
- **Platform Optimization**: Exports content optimized for Reels, TikTok, Shorts

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
# AI Services
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Content Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Next.js Configuration
NEXT_PUBLIC_APP_URL=http://localhost:3000

# Video Rendering (Remotion)
REMOTION_LICENSE_KEY=your_remotion_license
```

## Project Structure

```
marketing-pineline-share/
├── app/                    # Next.js app directory
├── components/            # React components
├── lib/                   # Core libraries
│   ├── ai/               # AI integration (Claude, OpenAI)
│   ├── crawler/          # Web scraping modules
│   ├── content/          # Content generation logic
│   └── video/            # Remotion video rendering
├── remotion/             # Remotion video templates
├── public/               # Static assets
└── types/                # TypeScript type definitions
```

## Core API Usage

### 1. Content Research Module

```typescript
import { ContentResearcher } from '@/lib/crawler/researcher';

async function researchTopic(keyword: string) {
  const researcher = new ContentResearcher({
    apiKey: process.env.RAPIDAPI_KEY,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin']
  });

  const insights = await researcher.scan({
    keyword,
    timeRange: '24h',
    language: ['en', 'vi'],
    minRelevance: 0.7
  });

  return insights;
}

// Usage
const data = await researchTopic('AI automation tools');
console.log(data.articles); // Array of researched articles
console.log(data.insights); // Key insights extracted
```

### 2. AI Content Generation

```typescript
import { ContentGenerator } from '@/lib/ai/generator';
import { Anthropic } from '@anthropic-ai/sdk';
import OpenAI from 'openai';

// Using Claude
async function generateWithClaude(research: any, format: string) {
  const anthropic = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY
  });

  const generator = new ContentGenerator(anthropic);
  
  const content = await generator.create({
    research,
    format, // 'toplist' | 'pov' | 'case-study' | 'how-to'
    tone: 'professional',
    language: 'en',
    targetAudience: 'marketers'
  });

  return content;
}

// Using OpenAI
async function generateWithOpenAI(research: any) {
  const openai = new OpenAI({
    apiKey: process.env.OPENAI_API_KEY
  });

  const completion = await openai.chat.completions.create({
    model: "gpt-4-turbo-preview",
    messages: [
      {
        role: "system",
        content: "You are an expert content marketer creating engaging articles."
      },
      {
        role: "user",
        content: `Create a toplist article based on: ${JSON.stringify(research)}`
      }
    ],
    temperature: 0.7
  });

  return completion.choices[0].message.content;
}
```

### 3. Multi-Language Content Pipeline

```typescript
import { MultiLanguageGenerator } from '@/lib/content/multilang';

async function createMultilingualContent(keyword: string) {
  const generator = new MultiLanguageGenerator({
    claudeKey: process.env.ANTHROPIC_API_KEY,
    openaiKey: process.env.OPENAI_API_KEY
  });

  // Generate content in both languages simultaneously
  const contents = await generator.generateParallel({
    keyword,
    languages: ['en', 'vi'],
    formats: ['toplist', 'how-to'],
    includeMetadata: true
  });

  return {
    english: contents.en,
    vietnamese: contents.vi,
    metadata: contents.metadata
  };
}
```

### 4. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

async function generateContentVideo(article: any) {
  const bundleLocation = await bundle(
    path.join(process.cwd(), 'remotion/index.ts')
  );

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title: article.title,
      content: article.body,
      style: 'modern',
      platform: 'tiktok' // 'reels' | 'shorts' | 'tiktok'
    }
  });

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: `out/${article.slug}.mp4`,
    inputProps: composition.defaultProps
  });

  return `out/${article.slug}.mp4`;
}
```

## Complete Content Pipeline

```typescript
import { ContentPipeline } from '@/lib/pipeline';

async function runFullPipeline(keyword: string) {
  const pipeline = new ContentPipeline({
    anthropicKey: process.env.ANTHROPIC_API_KEY,
    openaiKey: process.env.OPENAI_API_KEY,
    rapidApiKey: process.env.RAPIDAPI_KEY
  });

  // Step 1: Research
  const research = await pipeline.research(keyword);
  
  // Step 2: Generate content
  const article = await pipeline.generate({
    research,
    format: 'toplist',
    languages: ['en', 'vi']
  });
  
  // Step 3: Create visuals
  const video = await pipeline.renderVideo({
    article,
    platform: 'tiktok',
    duration: 60
  });
  
  // Step 4: Export
  const output = await pipeline.export({
    article,
    video,
    formats: ['markdown', 'html', 'json']
  });

  return output;
}

// Execute pipeline
const result = await runFullPipeline('AI marketing automation');
console.log(result.article.url);
console.log(result.video.path);
```

## Next.js API Routes

### Create Content API

```typescript
// app/api/content/create/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { ContentPipeline } from '@/lib/pipeline';

export async function POST(request: NextRequest) {
  try {
    const { keyword, format, languages, includeVideo } = await request.json();

    const pipeline = new ContentPipeline({
      anthropicKey: process.env.ANTHROPIC_API_KEY,
      openaiKey: process.env.OPENAI_API_KEY,
      rapidApiKey: process.env.RAPIDAPI_KEY
    });

    const research = await pipeline.research(keyword);
    const content = await pipeline.generate({
      research,
      format,
      languages
    });

    let video = null;
    if (includeVideo) {
      video = await pipeline.renderVideo({
        article: content,
        platform: 'tiktok'
      });
    }

    return NextResponse.json({
      success: true,
      content,
      video
    });
  } catch (error) {
    return NextResponse.json(
      { error: error.message },
      { status: 500 }
    );
  }
}
```

### Research API

```typescript
// app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { ContentResearcher } from '@/lib/crawler/researcher';

export async function GET(request: NextRequest) {
  const searchParams = request.nextUrl.searchParams;
  const keyword = searchParams.get('keyword');
  const timeRange = searchParams.get('timeRange') || '24h';

  if (!keyword) {
    return NextResponse.json(
      { error: 'Keyword is required' },
      { status: 400 }
    );
  }

  const researcher = new ContentResearcher({
    apiKey: process.env.RAPIDAPI_KEY
  });

  const insights = await researcher.scan({
    keyword,
    timeRange,
    language: ['en', 'vi']
  });

  return NextResponse.json(insights);
}
```

## Common Patterns

### Batch Content Generation

```typescript
async function batchGenerateContent(keywords: string[]) {
  const pipeline = new ContentPipeline({
    anthropicKey: process.env.ANTHROPIC_API_KEY,
    openaiKey: process.env.OPENAI_API_KEY
  });

  const results = await Promise.all(
    keywords.map(async (keyword) => {
      try {
        const research = await pipeline.research(keyword);
        const content = await pipeline.generate({
          research,
          format: 'toplist'
        });
        return { keyword, success: true, content };
      } catch (error) {
        return { keyword, success: false, error: error.message };
      }
    })
  );

  return results;
}
```

### Scheduled Content Creation

```typescript
import cron from 'node-cron';

// Run content generation daily at 9 AM
cron.schedule('0 9 * * *', async () => {
  const trendingKeywords = await getTrendingKeywords();
  
  for (const keyword of trendingKeywords) {
    await runFullPipeline(keyword);
  }
});
```

### Custom Video Templates

```typescript
// remotion/templates/CustomTemplate.tsx
import { AbsoluteFill, useCurrentFrame } from 'remotion';

export const CustomContentVideo: React.FC<{
  title: string;
  content: string;
}> = ({ title, content }) => {
  const frame = useCurrentFrame();
  
  return (
    <AbsoluteFill
      style={{
        backgroundColor: '#000',
        justifyContent: 'center',
        alignItems: 'center'
      }}
    >
      <h1 style={{ opacity: frame / 30 }}>{title}</h1>
      <p>{content}</p>
    </AbsoluteFill>
  );
};
```

## Troubleshooting

### API Rate Limits

```typescript
import { RateLimiter } from '@/lib/utils/rate-limiter';

const limiter = new RateLimiter({
  requestsPerMinute: 10
});

async function safeAPICall(fn: () => Promise<any>) {
  await limiter.wait();
  return await fn();
}
```

### Content Quality Validation

```typescript
import { ContentValidator } from '@/lib/validation';

async function validateGeneratedContent(content: any) {
  const validator = new ContentValidator();
  
  const checks = await validator.validate(content, {
    minWordCount: 500,
    checkGrammar: true,
    checkPlagiarism: true,
    requireSources: true
  });

  if (!checks.passed) {
    throw new Error(`Validation failed: ${checks.errors.join(', ')}`);
  }

  return content;
}
```

### Video Rendering Errors

```typescript
async function safeVideoRender(article: any) {
  try {
    return await generateContentVideo(article);
  } catch (error) {
    console.error('Video rendering failed:', error);
    
    // Fallback to image generation
    const fallbackImage = await generateStaticImage(article);
    return fallbackImage;
  }
}
```

## Development Commands

```bash
# Run development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Run Remotion preview
npm run remotion:preview

# Render video
npm run remotion:render

# Type checking
npm run type-check

# Linting
npm run lint
```

## Best Practices

1. **Always validate API keys before processing**
2. **Implement retry logic for external API calls**
3. **Cache research results to minimize API calls**
4. **Use queue system for batch video rendering**
5. **Monitor token usage for AI APIs to control costs**
6. **Store generated content in database for reuse**
7. **Implement proper error logging and monitoring**
