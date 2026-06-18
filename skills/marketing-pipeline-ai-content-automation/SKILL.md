---
name: marketing-pipeline-ai-content-automation
description: Vietnamese AI content automation pipeline for research, scriptwriting, and video generation using Claude, OpenAI, and Remotion
triggers:
  - automate content creation with AI research
  - build marketing pipeline with Claude and OpenAI
  - generate video content from text automatically
  - scrape news and create social media posts
  - set up AI content automation workflow
  - create Vietnamese and English content with AI
  - render videos from article content using Remotion
  - build end-to-end content generation system
---

# Marketing Pipeline AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

Marketing Pipeline AI Content Automation is a comprehensive TypeScript-based content automation system that:
- **Auto-researches** trending topics from TechCrunch, a16z, Twitter, LinkedIn
- **Generates content** in multiple formats (toplist, POV, case study, how-to) using Claude/OpenAI
- **Creates bilingual content** (Vietnamese & English) with customizable tone
- **Renders videos** automatically using Remotion for social media platforms
- **Manages scheduling** and publishing workflow

Built with Next.js, this pipeline transforms a single keyword into fully-formatted articles, graphics, and videos.

## Installation

### Prerequisites

```bash
node >= 18.0.0
npm or pnpm
```

### Clone and Setup

```bash
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share
npm install
# or
pnpm install
```

### Environment Configuration

Create `.env.local` in the project root:

```bash
# AI APIs
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key
TWITTER_BEARER_TOKEN=your_twitter_token

# Video Rendering
REMOTION_LICENSE_KEY=your_remotion_key

# Database (if using)
DATABASE_URL=your_database_url

# Next.js
NEXT_PUBLIC_API_URL=http://localhost:3000
```

### Run Development Server

```bash
npm run dev
# or
pnpm dev
```

Access at `http://localhost:3000`

## Core Architecture

### Key Directories

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app router pages
│   ├── components/       # React components
│   ├── lib/              # Core utilities
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── research/    # Content scraping & analysis
│   │   ├── render/      # Remotion video generation
│   │   └── utils/       # Helpers
│   ├── types/           # TypeScript definitions
│   └── config/          # Configuration files
├── public/              # Static assets
└── remotion/            # Video templates
```

## Content Generation Workflow

### 1. Research Phase

Scrape and analyze content from multiple sources:

```typescript
import { researchTopic } from '@/lib/research/scraper';

async function gatherInsights(keyword: string) {
  const research = await researchTopic({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeRange: '24h',
    language: 'en'
  });

  return {
    articles: research.articles,
    trends: research.trends,
    insights: research.insights,
    dataPoints: research.statistics
  };
}
```

### 2. Content Generation with AI

Generate bilingual content using Claude or OpenAI:

```typescript
import { generateContent } from '@/lib/ai/content-generator';

async function createArticle(research: ResearchData, format: ContentFormat) {
  const content = await generateContent({
    research,
    format, // 'toplist' | 'pov' | 'case-study' | 'how-to'
    languages: ['vi', 'en'],
    tone: 'professional', // 'professional' | 'friendly' | 'humorous'
    aiProvider: 'claude', // 'claude' | 'openai'
    model: 'claude-3-opus-20240229'
  });

  return {
    vietnamese: content.vi,
    english: content.en,
    metadata: content.meta
  };
}
```

### 3. Video Rendering

Generate videos from content using Remotion:

```typescript
import { renderVideo } from '@/lib/render/video-generator';

async function createVideoFromArticle(article: Article) {
  const video = await renderVideo({
    content: article.vietnamese,
    template: 'reels', // 'reels' | 'tiktok' | 'shorts'
    aspectRatio: '9:16',
    duration: 60,
    style: {
      theme: 'modern',
      colors: ['#FF6B6B', '#4ECDC4'],
      font: 'Inter'
    }
  });

  return {
    videoUrl: video.url,
    thumbnailUrl: video.thumbnail,
    format: video.format
  };
}
```

## API Routes

### Content Generation Endpoint

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { ContentPipeline } from '@/lib/pipeline';

export async function POST(req: NextRequest) {
  const { keyword, format, languages } = await req.json();

  const pipeline = new ContentPipeline({
    aiProvider: process.env.AI_PROVIDER || 'claude'
  });

  const result = await pipeline.execute({
    keyword,
    format,
    languages,
    includeVideo: true
  });

  return NextResponse.json({
    success: true,
    data: result
  });
}
```

### Usage from Frontend

```typescript
async function generateContentFromUI(keyword: string) {
  const response = await fetch('/api/generate', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      keyword,
      format: 'toplist',
      languages: ['vi', 'en']
    })
  });

  const { data } = await response.json();
  return data;
}
```

## Common Patterns

### Full Pipeline Execution

```typescript
import { ContentPipeline } from '@/lib/pipeline';

async function runFullPipeline(keyword: string) {
  const pipeline = new ContentPipeline({
    aiProvider: 'claude',
    researchSources: ['techcrunch', 'a16z', 'twitter'],
    videoEnabled: true
  });

  // Step 1: Research
  const research = await pipeline.research(keyword);
  console.log(`Found ${research.articles.length} articles`);

  // Step 2: Generate content
  const content = await pipeline.generateContent({
    research,
    format: 'toplist',
    languages: ['vi', 'en']
  });

  // Step 3: Render video
  const video = await pipeline.renderVideo({
    content: content.vietnamese,
    template: 'reels'
  });

  // Step 4: Schedule publishing
  await pipeline.schedule({
    content,
    video,
    platforms: ['facebook', 'instagram'],
    publishAt: new Date('2024-12-01T10:00:00Z')
  });

  return { content, video };
}
```

### Custom AI Prompts

```typescript
import { createCustomPrompt } from '@/lib/ai/prompts';

const customPrompt = createCustomPrompt({
  role: 'expert-marketer',
  task: 'create-engaging-toplist',
  context: {
    audience: 'Vietnamese marketers',
    tone: 'professional-friendly',
    includeCTA: true
  },
  constraints: {
    maxLength: 2000,
    minDataPoints: 5,
    includeEmojis: true
  }
});

const content = await generateContent({
  research: myResearch,
  customPrompt
});
```

### Batch Processing

```typescript
async function processBatchKeywords(keywords: string[]) {
  const pipeline = new ContentPipeline();
  
  const results = await Promise.all(
    keywords.map(async (keyword) => {
      try {
        return await pipeline.execute({
          keyword,
          format: 'auto', // Auto-detect best format
          languages: ['vi', 'en'],
          includeVideo: true
        });
      } catch (error) {
        console.error(`Failed for ${keyword}:`, error);
        return null;
      }
    })
  );

  return results.filter(Boolean);
}
```

## Configuration

### Content Formats

```typescript
// src/config/formats.ts
export const CONTENT_FORMATS = {
  toplist: {
    structure: 'numbered-list',
    minItems: 5,
    maxItems: 10,
    includeIntro: true,
    includeConclusion: true
  },
  pov: {
    structure: 'opinion-based',
    includeSources: true,
    tone: 'personal'
  },
  'case-study': {
    structure: 'problem-solution',
    sections: ['background', 'challenge', 'solution', 'results'],
    requireDataPoints: true
  },
  'how-to': {
    structure: 'step-by-step',
    minSteps: 3,
    includeImages: true
  }
};
```

### AI Model Selection

```typescript
// src/config/ai.ts
export const AI_CONFIG = {
  claude: {
    model: 'claude-3-opus-20240229',
    maxTokens: 4000,
    temperature: 0.7
  },
  openai: {
    model: 'gpt-4-turbo-preview',
    maxTokens: 4000,
    temperature: 0.7
  }
};
```

### Video Templates

```typescript
// remotion/templates/reels.tsx
import { Composition } from 'remotion';

export const ReelsTemplate: React.FC<{
  title: string;
  points: string[];
  duration: number;
}> = ({ title, points, duration }) => {
  return (
    <div className="w-full h-full bg-gradient-to-br from-purple-500 to-pink-500">
      <h1 className="text-4xl font-bold text-white text-center p-8">
        {title}
      </h1>
      <ul className="space-y-4 p-8">
        {points.map((point, idx) => (
          <li key={idx} className="text-2xl text-white">
            {idx + 1}. {point}
          </li>
        ))}
      </ul>
    </div>
  );
};
```

## Troubleshooting

### API Rate Limits

```typescript
import { RateLimiter } from '@/lib/utils/rate-limiter';

const limiter = new RateLimiter({
  openai: { requestsPerMinute: 50 },
  claude: { requestsPerMinute: 40 },
  rapidapi: { requestsPerMinute: 100 }
});

await limiter.checkAndWait('openai');
const result = await callOpenAI();
```

### Research Scraping Failures

```typescript
async function robustResearch(keyword: string) {
  const fallbackSources = ['techcrunch', 'a16z', 'twitter'];
  let research = null;

  for (const source of fallbackSources) {
    try {
      research = await researchTopic({
        keyword,
        sources: [source],
        timeout: 10000
      });
      if (research.articles.length > 0) break;
    } catch (error) {
      console.warn(`Source ${source} failed:`, error);
      continue;
    }
  }

  return research || { articles: [], trends: [], insights: [] };
}
```

### Video Rendering Memory Issues

```typescript
// Use streaming for large videos
import { renderVideoStream } from '@/lib/render/streaming';

const videoStream = await renderVideoStream({
  content: largeContent,
  template: 'reels',
  chunkSize: 10, // Process 10 seconds at a time
  quality: 'medium' // Reduce memory usage
});
```

### Environment Variable Validation

```typescript
// src/lib/utils/validate-env.ts
export function validateEnv() {
  const required = [
    'OPENAI_API_KEY',
    'ANTHROPIC_API_KEY',
    'RAPIDAPI_KEY'
  ];

  const missing = required.filter(key => !process.env[key]);

  if (missing.length > 0) {
    throw new Error(
      `Missing required environment variables: ${missing.join(', ')}`
    );
  }
}
```

## Production Deployment

### Build

```bash
npm run build
# or
pnpm build
```

### Docker Support

```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
RUN npm run build
EXPOSE 3000
CMD ["npm", "start"]
```

### Performance Optimization

```typescript
// Cache research results
import { cacheResearch } from '@/lib/cache';

const cachedResearch = await cacheResearch(keyword, async () => {
  return await researchTopic({ keyword });
}, { ttl: 3600 }); // Cache for 1 hour
```
