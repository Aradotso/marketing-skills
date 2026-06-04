---
name: marketing-pipeline-ai-content-automation
description: Automated AI content pipeline for research, scriptwriting, posting, and video generation using Claude, OpenAI, and Remotion
triggers:
  - how do I automate content creation with AI research and video generation
  - set up automated content pipeline with Claude and OpenAI
  - create AI-powered content workflow from research to video
  - automate social media content with AI research and rendering
  - build content automation system with Remotion video generation
  - implement AI content pipeline with auto-research and posting
  - configure automated marketing content workflow
  - generate videos automatically from AI-written content
---

# Marketing Pipeline AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with **Ultimate AI Content Pipeline** (marketing-pipeline-share), a fully automated content production system that handles everything from research and scriptwriting to automatic posting and video generation. The pipeline uses Claude 3, OpenAI for content creation, and Remotion for video rendering.

## What This Project Does

Marketing Pipeline AI Content Automation is an end-to-end content factory that:

- **Auto-crawls** latest news from TechCrunch, a16z, Twitter/X, LinkedIn (last 24h)
- **Generates content** in multiple formats (toplist, POV, case study, how-to) using Claude/OpenAI
- **Creates bilingual content** (English & Vietnamese) with customizable tone
- **Renders videos & infographics** automatically using Remotion
- **Optimizes for platforms** (Reels, TikTok, Shorts)
- **Auto-posts** to social media pages

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

### Required Environment Variables

```bash
# AI Services
ANTHROPIC_API_KEY=your_claude_key_here
OPENAI_API_KEY=your_openai_key_here

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key_here
TWITTER_BEARER_TOKEN=your_twitter_token_here

# Social Media
FACEBOOK_PAGE_TOKEN=your_fb_token_here
FACEBOOK_PAGE_ID=your_page_id_here

# Video Rendering
REMOTION_LICENSE_KEY=your_remotion_key_here

# Database (if using)
DATABASE_URL=your_database_url_here
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app router
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI service integrations
│   │   ├── research/    # Content research & crawling
│   │   ├── content/     # Content generation
│   │   ├── video/       # Remotion video rendering
│   │   └── publish/     # Auto-posting services
│   ├── types/           # TypeScript types
│   └── utils/           # Utility functions
├── remotion/            # Video templates
└── public/              # Static assets
```

## Core API Usage

### 1. Research & Data Collection

```typescript
import { researchTopic } from '@/lib/research/crawler';

// Auto-crawl latest news on a topic
const researchData = await researchTopic({
  keyword: 'AI automation',
  sources: ['techcrunch', 'a16z', 'twitter'],
  timeframe: '24h',
  language: 'en'
});

// Returns:
// {
//   articles: Article[],
//   insights: string[],
//   trending: TrendData[],
//   sources: SourceInfo[]
// }
```

### 2. AI Content Generation

```typescript
import { generateContent } from '@/lib/content/generator';

// Generate content using Claude/OpenAI
const content = await generateContent({
  topic: 'AI automation trends',
  format: 'toplist', // 'pov' | 'case-study' | 'how-to'
  tone: 'professional', // 'friendly' | 'humorous' | 'expert'
  language: 'en', // 'vi' for Vietnamese
  researchData: researchData,
  aiProvider: 'claude', // 'openai'
  model: 'claude-3-5-sonnet-20241022'
});

// Returns:
// {
//   title: string,
//   content: string,
//   summary: string,
//   tags: string[],
//   metadata: ContentMetadata
// }
```

### 3. Bilingual Content Generation

```typescript
import { generateBilingualContent } from '@/lib/content/bilingual';

const bilingualContent = await generateBilingualContent({
  topic: 'AI automation',
  format: 'toplist',
  tone: 'professional',
  researchData: researchData
});

// Returns:
// {
//   en: { title, content, summary, tags },
//   vi: { title, content, summary, tags }
// }
```

### 4. Video Generation with Remotion

```typescript
import { renderContentVideo } from '@/lib/video/render';

// Render video from content
const video = await renderContentVideo({
  content: content,
  template: 'infographic', // 'toplist' | 'quote' | 'stats'
  platform: 'reels', // 'tiktok' | 'shorts'
  aspectRatio: '9:16',
  duration: 30,
  style: {
    primaryColor: '#FF6B6B',
    font: 'Inter',
    animation: 'smooth'
  }
});

// Returns:
// {
//   videoUrl: string,
//   thumbnailUrl: string,
//   duration: number,
//   size: number
// }
```

### 5. Auto-Publishing

```typescript
import { publishToFacebook } from '@/lib/publish/facebook';

// Auto-post to Facebook page
const post = await publishToFacebook({
  content: content.content,
  mediaUrl: video.videoUrl,
  pageId: process.env.FACEBOOK_PAGE_ID,
  accessToken: process.env.FACEBOOK_PAGE_TOKEN,
  scheduledTime: new Date('2024-06-05T10:00:00Z') // Optional
});

// Returns:
// {
//   postId: string,
//   url: string,
//   status: 'published' | 'scheduled'
// }
```

## Complete Workflow Example

```typescript
import { ContentPipeline } from '@/lib/pipeline';

async function runFullPipeline() {
  const pipeline = new ContentPipeline({
    aiProvider: 'claude',
    languages: ['en', 'vi'],
    platforms: ['facebook', 'twitter'],
    videoEnabled: true
  });

  // Execute full pipeline
  const result = await pipeline.execute({
    keyword: 'AI automation trends 2024',
    format: 'toplist',
    tone: 'expert',
    autoPublish: true,
    generateVideo: true
  });

  return result;
  // {
  //   research: ResearchData,
  //   content: { en: Content, vi: Content },
  //   videos: { en: Video, vi: Video },
  //   posts: { facebook: Post, twitter: Post },
  //   analytics: PipelineAnalytics
  // }
}
```

## Configuration

### Content Formats

```typescript
type ContentFormat = 
  | 'toplist'      // Top 10 lists, rankings
  | 'pov'          // Opinion/perspective pieces
  | 'case-study'   // In-depth analysis
  | 'how-to'       // Tutorial/guide format
  | 'news'         // News article format
  | 'comparison';  // A vs B format
```

### Research Sources Configuration

```typescript
// src/lib/research/config.ts
export const RESEARCH_SOURCES = {
  techcrunch: {
    enabled: true,
    weight: 0.3,
    categories: ['startups', 'ai', 'apps']
  },
  a16z: {
    enabled: true,
    weight: 0.2,
    feed: 'https://a16z.com/feed/'
  },
  twitter: {
    enabled: true,
    weight: 0.25,
    accounts: ['@sama', '@elonmusk', '@naval']
  },
  linkedin: {
    enabled: true,
    weight: 0.25
  }
};
```

### Video Templates (Remotion)

```typescript
// remotion/compositions.ts
import { Composition } from 'remotion';
import { ToplistTemplate } from './templates/Toplist';
import { InfographicTemplate } from './templates/Infographic';

export const RemotionRoot: React.FC = () => {
  return (
    <>
      <Composition
        id="toplist"
        component={ToplistTemplate}
        durationInFrames={900}
        fps={30}
        width={1080}
        height={1920}
      />
      <Composition
        id="infographic"
        component={InfographicTemplate}
        durationInFrames={600}
        fps={30}
        width={1080}
        height={1920}
      />
    </>
  );
};
```

## API Routes (Next.js)

### Research API

```typescript
// app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { researchTopic } from '@/lib/research/crawler';

export async function POST(request: NextRequest) {
  const { keyword, sources, timeframe } = await request.json();
  
  const data = await researchTopic({
    keyword,
    sources,
    timeframe
  });
  
  return NextResponse.json(data);
}
```

### Content Generation API

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { generateContent } from '@/lib/content/generator';

export async function POST(request: NextRequest) {
  const { topic, format, tone, language, researchData } = await request.json();
  
  const content = await generateContent({
    topic,
    format,
    tone,
    language,
    researchData,
    aiProvider: 'claude'
  });
  
  return NextResponse.json(content);
}
```

### Video Rendering API

```typescript
// app/api/render-video/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { renderContentVideo } from '@/lib/video/render';

export async function POST(request: NextRequest) {
  const { content, template, platform } = await request.json();
  
  const video = await renderContentVideo({
    content,
    template,
    platform,
    aspectRatio: platform === 'reels' ? '9:16' : '16:9'
  });
  
  return NextResponse.json(video);
}
```

## Common Patterns

### Pattern 1: Scheduled Content Pipeline

```typescript
import { schedule } from '@/lib/scheduler';

// Run pipeline every day at 9 AM
schedule.daily('09:00', async () => {
  const pipeline = new ContentPipeline({
    aiProvider: 'claude',
    languages: ['en', 'vi'],
    autoPublish: true
  });

  await pipeline.execute({
    keyword: 'daily AI news',
    format: 'news',
    tone: 'professional'
  });
});
```

### Pattern 2: Batch Content Generation

```typescript
async function generateBatchContent(topics: string[]) {
  const results = await Promise.all(
    topics.map(topic => 
      pipeline.execute({
        keyword: topic,
        format: 'toplist',
        generateVideo: true,
        autoPublish: false // Review before publishing
      })
    )
  );

  return results;
}

const topics = [
  'AI automation tools',
  'Marketing trends 2024',
  'Content creation tips'
];

const batch = await generateBatchContent(topics);
```

### Pattern 3: Custom AI Prompts

```typescript
import { createCustomPrompt } from '@/lib/ai/prompts';

const customPrompt = createCustomPrompt({
  system: 'You are a marketing expert specializing in B2B SaaS',
  template: `
    Based on this research data: {researchData}
    
    Create a {format} article about {topic}
    Tone: {tone}
    Language: {language}
    
    Include:
    - Data-backed insights
    - Real examples
    - Actionable takeaways
  `,
  variables: {
    format: 'case-study',
    tone: 'expert',
    language: 'en'
  }
});

const content = await generateContent({
  customPrompt,
  researchData,
  topic: 'AI in marketing'
});
```

## CLI Commands

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Render Remotion video locally
npm run render -- --composition=toplist --output=video.mp4

# Run content pipeline from CLI
npm run pipeline -- --keyword="AI trends" --format=toplist

# Test research crawling
npm run test:research -- --keyword="AI automation"
```

## Troubleshooting

### API Rate Limits

```typescript
import { RateLimiter } from '@/lib/utils/rate-limiter';

const limiter = new RateLimiter({
  maxRequests: 10,
  perMilliseconds: 60000 // 10 requests per minute
});

await limiter.execute(async () => {
  return await generateContent({ /* ... */ });
});
```

### Video Rendering Timeout

```typescript
// Increase timeout for complex videos
const video = await renderContentVideo({
  content,
  template: 'infographic',
  timeout: 300000, // 5 minutes
  quality: 'medium' // Lower quality for faster rendering
});
```

### Claude/OpenAI Context Length

```typescript
// Split long content into chunks
import { chunkContent } from '@/lib/utils/chunk';

const chunks = chunkContent(researchData.articles, {
  maxTokens: 4000,
  overlap: 200
});

const results = await Promise.all(
  chunks.map(chunk => generateContent({ 
    researchData: chunk,
    /* ... */
  }))
);
```

### Memory Issues with Video Rendering

```typescript
// Use streaming for large videos
import { renderVideoStream } from '@/lib/video/stream';

const stream = await renderVideoStream({
  content,
  template: 'toplist',
  onProgress: (progress) => {
    console.log(`Rendering: ${progress}%`);
  }
});

// Save to file
await pipeline(
  stream,
  fs.createWriteStream('output.mp4')
);
```

## Development Server

```bash
# Start Next.js dev server
npm run dev
# Server runs at http://localhost:3000

# Start Remotion studio
npm run remotion
# Studio at http://localhost:3001
```

This skill enables comprehensive automation of content marketing workflows using AI research, generation, and video production capabilities.
