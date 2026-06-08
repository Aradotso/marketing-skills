---
name: marketing-pipeline-auto-content
description: Automated AI content pipeline that researches, writes multi-format articles, and generates videos from scratch using Claude/OpenAI and Remotion
triggers:
  - "set up automated content generation pipeline"
  - "create AI-powered content from research to video"
  - "automate content research and script generation"
  - "build content pipeline with Claude and OpenAI"
  - "generate videos from written content automatically"
  - "crawl trending news and create articles"
  - "set up multi-language content automation"
  - "create infographics and videos from blog posts"
---

# Marketing Pipeline Auto Content

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

**Marketing Pipeline Auto Content** is an end-to-end automated content creation system that transforms a single keyword into polished articles and videos. The pipeline handles:

- **Auto Research**: Crawls real-time data from TechCrunch, a16z, Twitter, LinkedIn
- **AI Writing**: Generates multi-format content (listicles, POV, case studies, how-tos) in multiple languages
- **Video Generation**: Automatically renders videos and infographics using Remotion
- **Multi-platform Export**: Optimized for Reels, TikTok, Shorts

Built with Next.js, TypeScript, and integrates with OpenAI, Anthropic (Claude), RapidAPI, and Remotion.

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

### Required Environment Variables

```bash
# AI Services
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Optional: Video Rendering
REMOTION_LICENSE_KEY=your_remotion_key
```

### Development Server

```bash
npm run dev
# Server runs on http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI service integrations
│   │   ├── research/    # Content research crawlers
│   │   ├── video/       # Remotion video generation
│   │   └── utils/       # Helper functions
│   └── types/           # TypeScript definitions
├── public/              # Static assets
└── remotion/            # Remotion video templates
```

## Core Components

### 1. Content Research Pipeline

```typescript
import { researchTopic } from '@/lib/research/crawler';

// Crawl and analyze trending content
async function gatherResearch(keyword: string) {
  const research = await researchTopic({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeframe: '24h',
    maxResults: 20
  });

  return {
    articles: research.articles,
    insights: research.insights,
    statistics: research.statistics,
    trends: research.trends
  };
}
```

### 2. AI Content Generation

```typescript
import { generateContent } from '@/lib/ai/content-generator';

// Generate multi-format content
async function createArticle(topic: string, research: any) {
  const content = await generateContent({
    topic,
    research,
    format: 'toplist', // 'toplist' | 'pov' | 'case-study' | 'how-to'
    language: 'en', // 'en' | 'vi' | 'both'
    tone: 'professional', // 'professional' | 'friendly' | 'humorous'
    provider: 'claude' // 'claude' | 'openai'
  });

  return content;
}
```

### 3. Claude API Integration

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

export async function generateWithClaude(
  prompt: string,
  systemPrompt: string
) {
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    system: systemPrompt,
    messages: [
      {
        role: 'user',
        content: prompt
      }
    ]
  });

  return message.content[0].text;
}
```

### 4. OpenAI Integration

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

export async function generateWithOpenAI(
  prompt: string,
  systemPrompt: string
) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      { role: 'system', content: systemPrompt },
      { role: 'user', content: prompt }
    ],
    temperature: 0.7,
    max_tokens: 3000
  });

  return completion.choices[0].message.content;
}
```

### 5. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

export async function generateVideo(
  content: any,
  template: string = 'default'
) {
  const bundleLocation = await bundle(
    path.join(process.cwd(), 'remotion/index.ts')
  );

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: template,
    inputProps: {
      title: content.title,
      sections: content.sections,
      stats: content.statistics,
      branding: {
        logo: '/logo.png',
        colors: ['#FF6B6B', '#4ECDC4']
      }
    }
  });

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: `out/${content.slug}.mp4`,
    inputProps: composition.defaultProps
  });

  return `out/${content.slug}.mp4`;
}
```

## Complete Workflow Example

```typescript
import { researchTopic } from '@/lib/research/crawler';
import { generateContent } from '@/lib/ai/content-generator';
import { generateVideo } from '@/lib/video/render';

async function fullContentPipeline(keyword: string) {
  try {
    // Step 1: Research
    console.log('🔍 Researching topic...');
    const research = await researchTopic({
      keyword,
      sources: ['techcrunch', 'twitter'],
      timeframe: '24h',
      maxResults: 15
    });

    // Step 2: Generate Content (English & Vietnamese)
    console.log('✍️ Generating content...');
    const content = await generateContent({
      topic: keyword,
      research,
      format: 'toplist',
      language: 'both',
      tone: 'professional',
      provider: 'claude'
    });

    // Step 3: Generate Video
    console.log('🎬 Rendering video...');
    const videoPath = await generateVideo(content.en, 'social-media');

    // Step 4: Generate Thumbnail
    const thumbnail = await generateThumbnail(content.en);

    return {
      articles: {
        en: content.en,
        vi: content.vi
      },
      video: videoPath,
      thumbnail,
      metadata: {
        keyword,
        generatedAt: new Date(),
        sources: research.sources
      }
    };
  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}

// Usage
const result = await fullContentPipeline('AI content automation');
```

## API Routes

### Research Endpoint

```typescript
// app/api/research/route.ts
import { NextResponse } from 'next/server';
import { researchTopic } from '@/lib/research/crawler';

export async function POST(request: Request) {
  const { keyword, sources, timeframe } = await request.json();

  const research = await researchTopic({
    keyword,
    sources: sources || ['techcrunch', 'a16z'],
    timeframe: timeframe || '24h',
    maxResults: 20
  });

  return NextResponse.json(research);
}
```

### Content Generation Endpoint

```typescript
// app/api/generate/route.ts
import { NextResponse } from 'next/server';
import { generateContent } from '@/lib/ai/content-generator';

export async function POST(request: Request) {
  const { topic, research, format, language, provider } = await request.json();

  const content = await generateContent({
    topic,
    research,
    format: format || 'toplist',
    language: language || 'en',
    tone: 'professional',
    provider: provider || 'claude'
  });

  return NextResponse.json(content);
}
```

### Video Rendering Endpoint

```typescript
// app/api/video/route.ts
import { NextResponse } from 'next/server';
import { generateVideo } from '@/lib/video/render';

export async function POST(request: Request) {
  const { content, template } = await request.json();

  const videoPath = await generateVideo(content, template || 'default');

  return NextResponse.json({ 
    success: true, 
    videoUrl: `/videos/${path.basename(videoPath)}` 
  });
}
```

## Content Format Templates

### Toplist Format

```typescript
const toplistPrompt = `
Create a toplist article about ${topic}.

Structure:
1. Compelling headline (numbered list format)
2. Brief introduction with hook
3. ${research.insights.length} main points, each with:
   - Subheading
   - 2-3 paragraphs explanation
   - Real data/statistics from research
   - Practical example
4. Conclusion with actionable takeaway

Use insights: ${JSON.stringify(research.insights)}
Include statistics: ${JSON.stringify(research.statistics)}

Tone: ${tone}
Language: ${language}
`;
```

### Case Study Format

```typescript
const caseStudyPrompt = `
Write a case study about ${topic}.

Structure:
1. Context: Problem/Challenge
2. Solution: Approach taken
3. Implementation: Step-by-step process
4. Results: Quantifiable outcomes
5. Lessons Learned

Use real examples from: ${JSON.stringify(research.articles)}
Include metrics: ${JSON.stringify(research.statistics)}

Focus on storytelling and practical insights.
`;
```

## Configuration

### Custom Content Templates

```typescript
// lib/config/templates.ts
export const contentTemplates = {
  toplist: {
    minPoints: 5,
    maxPoints: 10,
    includeStats: true,
    includeImages: true
  },
  pov: {
    perspective: 'first-person',
    includePersonalAnecdotes: true,
    tone: 'conversational'
  },
  caseStudy: {
    sections: ['context', 'solution', 'implementation', 'results'],
    includeMetrics: true,
    dataVisualization: true
  },
  howTo: {
    format: 'step-by-step',
    includeScreenshots: true,
    difficulty: 'beginner' // 'beginner' | 'intermediate' | 'advanced'
  }
};
```

### Video Templates

```typescript
// remotion/config/templates.ts
export const videoTemplates = {
  'social-media': {
    duration: 30, // seconds
    aspectRatio: '9:16', // vertical for Reels/TikTok
    fps: 30,
    animations: 'fast'
  },
  'youtube-short': {
    duration: 60,
    aspectRatio: '9:16',
    fps: 60,
    animations: 'medium'
  },
  'linkedin': {
    duration: 45,
    aspectRatio: '1:1',
    fps: 30,
    animations: 'professional'
  }
};
```

## Common Patterns

### Batch Processing

```typescript
async function batchGenerateContent(keywords: string[]) {
  const results = await Promise.allSettled(
    keywords.map(async (keyword) => {
      const research = await researchTopic({ keyword });
      const content = await generateContent({ 
        topic: keyword, 
        research,
        provider: 'claude' 
      });
      return { keyword, content };
    })
  );

  return results
    .filter(r => r.status === 'fulfilled')
    .map(r => (r as PromiseFulfilledResult<any>).value);
}
```

### Content Scheduling

```typescript
import { schedulePost } from '@/lib/social/scheduler';

async function createAndSchedule(keyword: string, publishDate: Date) {
  const result = await fullContentPipeline(keyword);

  await schedulePost({
    content: result.articles.en,
    video: result.video,
    thumbnail: result.thumbnail,
    platforms: ['facebook', 'linkedin', 'twitter'],
    scheduledFor: publishDate
  });
}
```

### Multi-language Generation

```typescript
async function generateMultiLingual(topic: string, languages: string[]) {
  const research = await researchTopic({ keyword: topic });

  const contentByLanguage = await Promise.all(
    languages.map(async (lang) => {
      const content = await generateContent({
        topic,
        research,
        language: lang,
        provider: 'claude'
      });
      return { language: lang, content };
    })
  );

  return Object.fromEntries(
    contentByLanguage.map(({ language, content }) => [language, content])
  );
}
```

## Troubleshooting

### Rate Limiting

```typescript
import pLimit from 'p-limit';

// Limit concurrent API calls
const limit = pLimit(3);

async function rateLimitedGeneration(topics: string[]) {
  return Promise.all(
    topics.map(topic => 
      limit(() => fullContentPipeline(topic))
    )
  );
}
```

### API Errors

```typescript
import { retry } from '@/lib/utils/retry';

async function robustGeneration(keyword: string) {
  return retry(
    async () => {
      return await generateContent({
        topic: keyword,
        provider: 'claude'
      });
    },
    {
      retries: 3,
      minTimeout: 1000,
      onRetry: (error, attempt) => {
        console.log(`Retry attempt ${attempt}: ${error.message}`);
      }
    }
  );
}
```

### Video Rendering Issues

```typescript
// Ensure FFmpeg is installed
import { ensureFfmpeg } from '@remotion/renderer';

async function safeVideoRender(content: any) {
  try {
    await ensureFfmpeg();
    return await generateVideo(content);
  } catch (error) {
    if (error.message.includes('FFmpeg')) {
      console.error('FFmpeg not found. Install: npm install @remotion/renderer');
    }
    throw error;
  }
}
```

### Memory Management for Large Batches

```typescript
async function efficientBatchProcessing(keywords: string[]) {
  const batchSize = 5;
  const results = [];

  for (let i = 0; i < keywords.length; i += batchSize) {
    const batch = keywords.slice(i, i + batchSize);
    const batchResults = await Promise.all(
      batch.map(k => fullContentPipeline(k))
    );
    results.push(...batchResults);
    
    // Clear memory between batches
    if (global.gc) global.gc();
  }

  return results;
}
```

## Build and Deploy

```bash
# Production build
npm run build

# Start production server
npm start

# Export static site (if applicable)
npm run export
```

## Environment-Specific Configuration

```typescript
// lib/config/environment.ts
export const config = {
  development: {
    apiTimeout: 60000,
    maxRetries: 3,
    verboseLogging: true
  },
  production: {
    apiTimeout: 30000,
    maxRetries: 5,
    verboseLogging: false
  }
}[process.env.NODE_ENV || 'development'];
```
