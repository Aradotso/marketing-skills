---
name: marketing-pipeline-automation
description: Automated AI content pipeline for research, scriptwriting, social posting, and video generation using Claude, OpenAI, and Remotion
triggers:
  - automate my content creation workflow
  - set up AI content pipeline with video generation
  - create automated marketing content system
  - build research to video content pipeline
  - integrate Claude and OpenAI for content automation
  - generate videos from written content automatically
  - scrape and analyze trending content for marketing
  - automate social media content with AI
---

# Marketing Pipeline Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the Ultimate AI Content Pipeline, a TypeScript-based system that automates the entire content creation workflow from research and scriptwriting to video generation and social media posting.

## What This Project Does

The marketing-pipeline-share project is an end-to-end content automation system that:

- **Auto-scans and researches** trending content from sources like TechCrunch, a16z, Twitter/X, and LinkedIn
- **Generates multi-format content** (toplist, POV, case studies, how-to) in multiple languages using Claude 3 and OpenAI
- **Renders videos automatically** using Remotion to convert text content into visual media
- **Optimizes for platforms** like Reels, TikTok, and YouTube Shorts
- **Schedules and publishes** content to social media pages

## Installation

### Prerequisites

```bash
# Node.js 18+ required
node --version

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

### Environment Configuration

Create a `.env.local` file in the root directory:

```bash
# AI APIs
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research/Scraping APIs
RAPIDAPI_KEY=your_rapidapi_key

# Social Media (optional)
FACEBOOK_PAGE_ACCESS_TOKEN=your_token
FACEBOOK_PAGE_ID=your_page_id

# Video Rendering
REMOTION_LICENSE_KEY=your_remotion_license

# Database (if applicable)
DATABASE_URL=your_database_connection_string
```

### Development Server

```bash
npm run dev
# Server starts at http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app router pages
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── scraper/     # Content research/scraping
│   │   ├── video/       # Remotion video generation
│   │   └── social/      # Social media posting
│   ├── services/        # Business logic services
│   └── utils/           # Helper functions
├── remotion/            # Remotion video templates
├── public/              # Static assets
└── package.json
```

## Core API Usage

### 1. Content Research & Scraping

```typescript
import { researchContent } from '@/lib/scraper/research';

async function gatherTrendingTopics(keyword: string) {
  const research = await researchContent({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeframe: '24h',
    language: 'en'
  });

  return {
    articles: research.articles,
    insights: research.insights,
    statistics: research.statistics,
    trends: research.trends
  };
}

// Usage
const data = await gatherTrendingTopics('AI marketing automation');
console.log(data.insights);
```

### 2. AI Content Generation

```typescript
import { generateContent } from '@/lib/ai/generator';
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY
});

async function createMarketingContent(topic: string, format: string) {
  const content = await generateContent({
    client: anthropic,
    model: 'claude-3-5-sonnet-20241022',
    topic,
    format, // 'toplist', 'pov', 'case-study', 'how-to'
    languages: ['en', 'vi'],
    tone: 'professional', // 'friendly', 'humorous', 'expert'
    includeData: true,
    research: await researchContent({ keyword: topic })
  });

  return {
    title: content.title,
    body: content.body,
    metadata: content.metadata,
    translations: content.translations
  };
}

// Generate POV article
const article = await createMarketingContent(
  'The Future of AI in Content Marketing',
  'pov'
);
```

### 3. OpenAI Integration Alternative

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY
});

async function generateWithOpenAI(prompt: string) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: 'You are a professional content marketing writer.'
      },
      {
        role: 'user',
        content: prompt
      }
    ],
    temperature: 0.7,
    max_tokens: 2000
  });

  return completion.choices[0].message.content;
}
```

### 4. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

async function generateContentVideo(
  content: {
    title: string;
    points: string[];
    images: string[];
  },
  platform: 'reels' | 'tiktok' | 'shorts'
) {
  // Platform dimensions
  const dimensions = {
    reels: { width: 1080, height: 1920 },
    tiktok: { width: 1080, height: 1920 },
    shorts: { width: 1080, height: 1920 }
  };

  const bundleLocation = await bundle({
    entryPoint: path.join(process.cwd(), 'remotion/index.ts'),
    webpackOverride: (config) => config
  });

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title: content.title,
      points: content.points,
      images: content.images
    }
  });

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: `out/${platform}-video.mp4`,
    inputProps: {
      title: content.title,
      points: content.points,
      images: content.images
    },
    ...dimensions[platform]
  });

  return `out/${platform}-video.mp4`;
}

// Usage
const videoPath = await generateContentVideo(
  {
    title: 'Top 5 AI Marketing Tools',
    points: ['Tool 1', 'Tool 2', 'Tool 3'],
    images: ['/img1.jpg', '/img2.jpg', '/img3.jpg']
  },
  'reels'
);
```

### 5. Social Media Publishing

```typescript
import { publishToFacebook } from '@/lib/social/facebook';

async function publishContent(
  content: string,
  videoPath?: string
) {
  const result = await publishToFacebook({
    pageId: process.env.FACEBOOK_PAGE_ID!,
    accessToken: process.env.FACEBOOK_PAGE_ACCESS_TOKEN!,
    message: content,
    videoPath,
    scheduledTime: new Date(Date.now() + 3600000) // 1 hour from now
  });

  return {
    postId: result.id,
    postUrl: result.url,
    status: result.status
  };
}
```

## Common Workflow Patterns

### Full Pipeline: Research to Published Video

```typescript
import { researchContent } from '@/lib/scraper/research';
import { generateContent } from '@/lib/ai/generator';
import { generateContentVideo } from '@/lib/video/remotion';
import { publishToFacebook } from '@/lib/social/facebook';

async function fullContentPipeline(keyword: string) {
  try {
    // Step 1: Research
    console.log('🔍 Researching...');
    const research = await researchContent({
      keyword,
      sources: ['techcrunch', 'twitter'],
      timeframe: '24h'
    });

    // Step 2: Generate content with Claude
    console.log('✍️ Generating content...');
    const content = await generateContent({
      topic: keyword,
      format: 'toplist',
      languages: ['en', 'vi'],
      tone: 'expert',
      research
    });

    // Step 3: Generate video
    console.log('🎬 Rendering video...');
    const videoPath = await generateContentVideo(
      {
        title: content.title,
        points: content.mainPoints,
        images: content.imageUrls
      },
      'reels'
    );

    // Step 4: Publish
    console.log('📤 Publishing...');
    const published = await publishToFacebook({
      pageId: process.env.FACEBOOK_PAGE_ID!,
      accessToken: process.env.FACEBOOK_PAGE_ACCESS_TOKEN!,
      message: content.body,
      videoPath
    });

    return {
      success: true,
      content,
      videoPath,
      postUrl: published.url
    };
  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}

// Execute pipeline
await fullContentPipeline('AI content automation');
```

### Batch Content Generation

```typescript
async function batchGenerateContent(keywords: string[]) {
  const results = await Promise.allSettled(
    keywords.map(async (keyword) => {
      const research = await researchContent({ keyword });
      const content = await generateContent({
        topic: keyword,
        format: 'how-to',
        languages: ['en'],
        research
      });
      return { keyword, content };
    })
  );

  const successful = results
    .filter((r) => r.status === 'fulfilled')
    .map((r) => (r as PromiseFulfilledResult<any>).value);

  const failed = results
    .filter((r) => r.status === 'rejected')
    .map((r) => (r as PromiseRejectedResult).reason);

  return { successful, failed };
}
```

### Custom Content Format Template

```typescript
interface ContentTemplate {
  format: string;
  structure: string[];
  tone: string;
  minWords: number;
}

async function generateCustomFormat(
  topic: string,
  template: ContentTemplate
) {
  const prompt = `
Create a ${template.format} article about "${topic}".

Structure:
${template.structure.map((s, i) => `${i + 1}. ${s}`).join('\n')}

Tone: ${template.tone}
Minimum words: ${template.minWords}

Include real data, statistics, and actionable insights.
  `;

  const content = await generateWithOpenAI(prompt);
  return content;
}

// Usage
const listicle = await generateCustomFormat(
  'Email marketing best practices',
  {
    format: 'Listicle',
    structure: [
      'Engaging hook',
      '10 numbered tips with explanations',
      'Real-world examples',
      'Call to action'
    ],
    tone: 'friendly',
    minWords: 1500
  }
);
```

## API Routes (Next.js)

### Generate Content Endpoint

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { generateContent } from '@/lib/ai/generator';

export async function POST(request: NextRequest) {
  try {
    const { keyword, format, languages } = await request.json();

    const content = await generateContent({
      topic: keyword,
      format,
      languages,
      tone: 'professional'
    });

    return NextResponse.json({
      success: true,
      data: content
    });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: (error as Error).message },
      { status: 500 }
    );
  }
}
```

### Video Generation Endpoint

```typescript
// app/api/video/render/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { generateContentVideo } from '@/lib/video/remotion';

export async function POST(request: NextRequest) {
  try {
    const { title, points, images, platform } = await request.json();

    const videoPath = await generateContentVideo(
      { title, points, images },
      platform
    );

    return NextResponse.json({
      success: true,
      videoPath,
      url: `${process.env.NEXT_PUBLIC_URL}/${videoPath}`
    });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: (error as Error).message },
      { status: 500 }
    );
  }
}
```

## Configuration

### Customize AI Models

```typescript
// lib/config/ai.ts
export const AI_CONFIG = {
  claude: {
    model: 'claude-3-5-sonnet-20241022',
    maxTokens: 4096,
    temperature: 0.7
  },
  openai: {
    model: 'gpt-4-turbo-preview',
    maxTokens: 2000,
    temperature: 0.7
  },
  formats: ['toplist', 'pov', 'case-study', 'how-to'],
  languages: ['en', 'vi'],
  tones: ['professional', 'friendly', 'humorous', 'expert']
};
```

### Video Templates Configuration

```typescript
// remotion/config.ts
export const VIDEO_CONFIG = {
  fps: 30,
  durationInFrames: 900, // 30 seconds at 30fps
  platforms: {
    reels: { width: 1080, height: 1920 },
    tiktok: { width: 1080, height: 1920 },
    shorts: { width: 1080, height: 1920 },
    youtube: { width: 1920, height: 1080 }
  },
  fonts: {
    primary: 'Inter',
    secondary: 'Roboto'
  }
};
```

## Troubleshooting

### API Rate Limits

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
        console.log(`Rate limited. Retrying in ${delay}ms...`);
        await new Promise((resolve) => setTimeout(resolve, delay));
      } else {
        throw error;
      }
    }
  }
  throw new Error('Max retries reached');
}
```

### Video Rendering Memory Issues

```typescript
// Adjust Remotion rendering settings
import { renderMedia } from '@remotion/renderer';

await renderMedia({
  composition,
  serveUrl: bundleLocation,
  codec: 'h264',
  concurrency: 1, // Reduce concurrency for lower memory
  enforceAudioTrack: false,
  chromiumOptions: {
    headless: true,
    args: ['--no-sandbox', '--disable-setuid-sandbox']
  }
});
```

### Content Quality Issues

```typescript
// Add validation layer
function validateContent(content: any): boolean {
  const checks = [
    content.body.length >= 500,
    content.title.length <= 100,
    content.mainPoints?.length >= 3,
    !content.body.includes('[INSERT'),
    !content.body.includes('Lorem ipsum')
  ];

  return checks.every((check) => check === true);
}

// Use in pipeline
const content = await generateContent({ topic, format });
if (!validateContent(content)) {
  console.warn('Content quality check failed, regenerating...');
  content = await generateContent({ topic, format, temperature: 0.8 });
}
```

### Environment Variable Issues

```typescript
// Validate environment on startup
function validateEnv() {
  const required = [
    'ANTHROPIC_API_KEY',
    'OPENAI_API_KEY',
    'RAPIDAPI_KEY'
  ];

  const missing = required.filter((key) => !process.env[key]);

  if (missing.length > 0) {
    throw new Error(
      `Missing required environment variables: ${missing.join(', ')}`
    );
  }
}

validateEnv();
```

This skill provides comprehensive coverage of the marketing pipeline automation system, enabling AI agents to effectively assist developers in implementing automated content workflows from research through video generation and publishing.
