---
name: marketing-pipeline-share-automation
description: AI-powered content automation pipeline for research, scriptwriting, and video generation using Claude/OpenAI and Remotion
triggers:
  - automate content creation with AI research
  - generate marketing videos from text automatically
  - build an AI content pipeline with research crawling
  - create multilingual content with Claude and OpenAI
  - render videos from blog posts using Remotion
  - set up automated content workflow from research to video
  - scrape news and generate AI-written articles
  - build a content automation system with TypeScript
---

# Marketing Pipeline Share Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the Ultimate AI Content Pipeline, a TypeScript-based automation system that handles the entire content creation workflow: from automated research and news crawling, to AI-powered scriptwriting in multiple formats and languages, to automatic video rendering using Remotion.

## What This Project Does

Marketing Pipeline Share is an all-in-one content automation system that:

1. **Crawls real-time data** from sources like TechCrunch, a16z, Twitter/X, and LinkedIn
2. **Generates AI-powered content** using Claude 3 or OpenAI in multiple formats (toplist, POV, case study, how-to)
3. **Creates bilingual content** (English & Vietnamese) with customizable tone
4. **Renders videos and infographics** automatically using Remotion
5. **Optimizes output** for multiple platforms (Reels, TikTok, Shorts)

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

## Environment Configuration

Create a `.env` file with the following variables:

```bash
# AI Provider Keys
ANTHROPIC_API_KEY=your_claude_api_key_here
OPENAI_API_KEY=your_openai_api_key_here

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key_here

# Social Media APIs (optional)
TWITTER_BEARER_TOKEN=your_twitter_token_here
LINKEDIN_ACCESS_TOKEN=your_linkedin_token_here

# Database (if using)
DATABASE_URL=your_database_connection_string

# Remotion Configuration
REMOTION_AWS_ACCESS_KEY_ID=your_aws_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret

# App Configuration
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app router pages
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI provider integrations
│   │   ├── crawler/     # News crawling logic
│   │   ├── content/     # Content generation
│   │   └── video/       # Remotion video rendering
│   ├── types/           # TypeScript type definitions
│   └── utils/           # Utility functions
├── remotion/            # Video templates
└── public/              # Static assets
```

## Core API Usage

### 1. Research & Crawling Module

```typescript
import { NewsResearcher } from '@/lib/crawler/researcher';

// Initialize researcher
const researcher = new NewsResearcher({
  rapidApiKey: process.env.RAPIDAPI_KEY!,
  sources: ['techcrunch', 'a16z', 'twitter', 'linkedin']
});

// Crawl latest news on a topic
async function researchTopic(keyword: string) {
  const results = await researcher.crawl({
    keyword,
    timeframe: '24h',
    maxResults: 20,
    languages: ['en', 'vi']
  });
  
  return results.articles.map(article => ({
    title: article.title,
    url: article.url,
    summary: article.summary,
    publishedAt: article.publishedAt,
    source: article.source
  }));
}

// Extract insights from research
const insights = await researcher.extractInsights(results);
```

### 2. AI Content Generation

```typescript
import { ContentGenerator } from '@/lib/ai/generator';
import { ClaudeProvider } from '@/lib/ai/providers/claude';
import { OpenAIProvider } from '@/lib/ai/providers/openai';

// Initialize with Claude
const generator = new ContentGenerator({
  provider: new ClaudeProvider({
    apiKey: process.env.ANTHROPIC_API_KEY!,
    model: 'claude-3-opus-20240229'
  })
});

// Or use OpenAI
const generatorOpenAI = new ContentGenerator({
  provider: new OpenAIProvider({
    apiKey: process.env.OPENAI_API_KEY!,
    model: 'gpt-4-turbo-preview'
  })
});

// Generate content in multiple formats
async function generateContent(research: any[], format: string) {
  const content = await generator.generate({
    research,
    format, // 'toplist' | 'pov' | 'case-study' | 'how-to'
    languages: ['en', 'vi'],
    tone: 'professional', // 'professional' | 'friendly' | 'humorous'
    targetAudience: 'marketers',
    includeStats: true
  });
  
  return {
    english: content.en,
    vietnamese: content.vi,
    metadata: content.metadata
  };
}
```

### 3. Bilingual Content Example

```typescript
import { BilingualWriter } from '@/lib/content/bilingual';

const writer = new BilingualWriter({
  primaryLanguage: 'en',
  secondaryLanguage: 'vi'
});

// Generate toplist article
const toplist = await writer.writeToplist({
  title: 'Top 5 AI Marketing Tools in 2024',
  items: [
    { name: 'Tool A', description: '...', stats: '...' },
    { name: 'Tool B', description: '...', stats: '...' }
  ],
  research: researchData,
  tone: 'expert'
});

// Generate POV article
const povArticle = await writer.writePOV({
  topic: 'The Future of AI in Content Marketing',
  perspective: 'industry-expert',
  research: researchData,
  includeCounterarguments: true
});

// Generate case study
const caseStudy = await writer.writeCaseStudy({
  company: 'Company X',
  challenge: 'Struggling with content production',
  solution: 'Implemented AI automation',
  results: 'Increased output by 90%',
  research: researchData
});
```

### 4. Video Rendering with Remotion

```typescript
import { renderVideo } from '@/lib/video/renderer';
import { VideoComposition } from '@/remotion/compositions/ContentVideo';

// Render video from content
async function createContentVideo(content: any) {
  const videoData = {
    title: content.title,
    points: content.keyPoints,
    stats: content.statistics,
    duration: 60, // seconds
    format: '9:16' // For Reels/TikTok/Shorts
  };
  
  const video = await renderVideo({
    composition: VideoComposition,
    props: videoData,
    outputFormat: 'mp4',
    fps: 30,
    width: 1080,
    height: 1920
  });
  
  return video.url;
}

// Batch render for multiple platforms
async function renderForAllPlatforms(content: any) {
  const formats = [
    { name: 'reels', width: 1080, height: 1920 },
    { name: 'youtube', width: 1920, height: 1080 },
    { name: 'square', width: 1080, height: 1080 }
  ];
  
  const videos = await Promise.all(
    formats.map(format => 
      renderVideo({
        composition: VideoComposition,
        props: content,
        width: format.width,
        height: format.height
      })
    )
  );
  
  return videos;
}
```

## Common Workflow Pattern

```typescript
import { ContentPipeline } from '@/lib/pipeline';

// Complete end-to-end pipeline
async function runFullPipeline(keyword: string) {
  const pipeline = new ContentPipeline({
    anthropicKey: process.env.ANTHROPIC_API_KEY!,
    openaiKey: process.env.OPENAI_API_KEY!,
    rapidApiKey: process.env.RAPIDAPI_KEY!
  });
  
  // Step 1: Research
  const research = await pipeline.research({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter'],
    timeframe: '24h'
  });
  
  // Step 2: Generate content
  const content = await pipeline.generateContent({
    research,
    format: 'toplist',
    languages: ['en', 'vi'],
    tone: 'professional'
  });
  
  // Step 3: Create video
  const video = await pipeline.renderVideo({
    content: content.en,
    platform: 'reels'
  });
  
  // Step 4: Export results
  return {
    article: {
      english: content.en,
      vietnamese: content.vi
    },
    video: {
      url: video.url,
      thumbnail: video.thumbnail
    },
    metadata: {
      researchSources: research.sources.length,
      wordCount: content.en.wordCount,
      generatedAt: new Date()
    }
  };
}
```

## API Routes (Next.js)

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { ContentPipeline } from '@/lib/pipeline';

export async function POST(req: NextRequest) {
  try {
    const { keyword, format, languages } = await req.json();
    
    const pipeline = new ContentPipeline({
      anthropicKey: process.env.ANTHROPIC_API_KEY!,
      rapidApiKey: process.env.RAPIDAPI_KEY!
    });
    
    const result = await pipeline.run({
      keyword,
      format,
      languages
    });
    
    return NextResponse.json(result);
  } catch (error) {
    return NextResponse.json(
      { error: error.message },
      { status: 500 }
    );
  }
}
```

## Configuration Options

### Content Format Types

```typescript
type ContentFormat = 
  | 'toplist'      // Numbered list articles
  | 'pov'          // Opinion/perspective pieces
  | 'case-study'   // Detailed case studies
  | 'how-to';      // Step-by-step guides

type ToneStyle = 
  | 'professional'  // Expert, formal
  | 'friendly'      // Casual, approachable
  | 'humorous';     // Light, entertaining
```

### Research Configuration

```typescript
interface ResearchConfig {
  keyword: string;
  sources: ('techcrunch' | 'a16z' | 'twitter' | 'linkedin')[];
  timeframe: '24h' | '7d' | '30d';
  maxResults: number;
  languages: ('en' | 'vi')[];
  includeStats?: boolean;
  filterBy?: {
    minEngagement?: number;
    contentType?: string[];
  };
}
```

## Running the Development Server

```bash
# Start Next.js development server
npm run dev

# Start Remotion preview
npm run remotion

# Build for production
npm run build

# Start production server
npm start
```

## Troubleshooting

### API Rate Limits

```typescript
import { RateLimiter } from '@/lib/utils/rate-limiter';

const limiter = new RateLimiter({
  maxRequests: 50,
  windowMs: 60000 // 1 minute
});

async function safeApiCall(fn: () => Promise<any>) {
  await limiter.wait();
  return fn();
}
```

### Handle AI Provider Errors

```typescript
import { AIProviderError } from '@/lib/ai/errors';

try {
  const content = await generator.generate(config);
} catch (error) {
  if (error instanceof AIProviderError) {
    if (error.code === 'RATE_LIMIT') {
      // Switch to backup provider
      const backupGenerator = new ContentGenerator({
        provider: new OpenAIProvider({ 
          apiKey: process.env.OPENAI_API_KEY! 
        })
      });
      return backupGenerator.generate(config);
    }
  }
  throw error;
}
```

### Video Rendering Timeout

```typescript
const video = await renderVideo({
  composition: VideoComposition,
  props: data,
  timeout: 300000, // 5 minutes
  onProgress: (progress) => {
    console.log(`Rendering: ${progress}%`);
  }
});
```

### Memory Management for Large Batches

```typescript
import { chunk } from '@/lib/utils/array';

async function processBatch(keywords: string[]) {
  const batches = chunk(keywords, 5); // Process 5 at a time
  
  for (const batch of batches) {
    await Promise.all(
      batch.map(keyword => runFullPipeline(keyword))
    );
    
    // Allow garbage collection between batches
    await new Promise(resolve => setTimeout(resolve, 1000));
  }
}
```

## Best Practices

1. **Always use environment variables** for API keys
2. **Implement retry logic** for external API calls
3. **Cache research results** to avoid redundant crawling
4. **Use streaming responses** for real-time content generation
5. **Validate input** before processing to save API costs
6. **Monitor rate limits** across all providers
7. **Store generated content** in a database for reuse
8. **Optimize video rendering** by caching common assets
