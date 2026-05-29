---
name: marketing-pipeline-automation
description: AI-powered content automation pipeline from research to video generation using Claude, OpenAI, and Remotion
triggers:
  - automate content creation with AI pipeline
  - generate videos from blog posts automatically
  - crawl news and create content
  - set up marketing automation pipeline
  - create AI content workflow
  - build automated content generation system
  - research and generate marketing content
  - turn articles into videos with Remotion
---

# Marketing Pipeline Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This project is a comprehensive AI-powered content automation system that handles the entire content lifecycle: from researching trending news, generating multi-format articles in multiple languages, to automatically rendering videos and graphics. Built with Next.js, TypeScript, and integrations with Claude 3, OpenAI, and Remotion.

## What It Does

The Ultimate AI Content Pipeline automates:

1. **Research**: Crawls and analyzes real-time data from TechCrunch, a16z, Twitter/X, LinkedIn
2. **Content Generation**: Creates articles in multiple formats (toplist, POV, case study, how-to) in English and Vietnamese
3. **Video Rendering**: Converts written content into infographics and short-form videos optimized for Reels, TikTok, Shorts
4. **Multi-platform Publishing**: Prepares content for distribution across social platforms

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
# AI APIs
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# Research/Crawling APIs
RAPIDAPI_KEY=your_rapidapi_key

# Video Rendering
REMOTION_LICENSE_KEY=your_remotion_license

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Next.js
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app router pages
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration modules
│   │   ├── crawlers/    # News crawling services
│   │   ├── generators/  # Content generation logic
│   │   └── video/       # Remotion video rendering
│   └── types/           # TypeScript type definitions
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Core API Usage

### 1. Content Research

```typescript
import { researchTopic } from '@/lib/crawlers/newsScanner';

async function scanLatestNews(keyword: string) {
  const research = await researchTopic({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter'],
    timeRange: '24h',
    language: 'en'
  });

  return {
    articles: research.articles,
    insights: research.insights,
    dataPoints: research.statistics
  };
}
```

### 2. AI Content Generation with Claude

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

async function generateContent(
  researchData: any,
  format: 'toplist' | 'pov' | 'case-study' | 'how-to',
  language: 'en' | 'vi'
) {
  const prompt = `Generate a ${format} article in ${language} based on this research: ${JSON.stringify(researchData)}`;

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [{
      role: 'user',
      content: prompt
    }]
  });

  return message.content[0].text;
}
```

### 3. OpenAI Integration

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function enhanceContent(baseContent: string) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: 'You are a professional content editor and marketer.'
      },
      {
        role: 'user',
        content: `Enhance this content with SEO optimization and engagement hooks: ${baseContent}`
      }
    ],
    temperature: 0.7,
  });

  return completion.choices[0].message.content;
}
```

### 4. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

async function generateVideo(contentData: any) {
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title: contentData.title,
      points: contentData.keyPoints,
      branding: contentData.branding
    },
  });

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: `out/${contentData.id}.mp4`,
    inputProps: composition.inputProps,
  });

  return `out/${contentData.id}.mp4`;
}
```

## Common Workflows

### Full Content Pipeline

```typescript
import { researchTopic } from '@/lib/crawlers/newsScanner';
import { generateContent } from '@/lib/ai/contentGenerator';
import { renderVideo } from '@/lib/video/videoRenderer';
import { publishToSocial } from '@/lib/publishers/socialMedia';

async function runFullPipeline(keyword: string) {
  // Step 1: Research
  const research = await researchTopic({
    keyword,
    sources: ['techcrunch', 'twitter'],
    timeRange: '24h'
  });

  // Step 2: Generate content in multiple formats
  const contentVariants = await Promise.all([
    generateContent(research, 'toplist', 'en'),
    generateContent(research, 'toplist', 'vi'),
    generateContent(research, 'pov', 'en'),
  ]);

  // Step 3: Create videos
  const videos = await Promise.all(
    contentVariants.map(content => renderVideo({
      content,
      format: 'reels', // or 'tiktok', 'shorts'
      aspectRatio: '9:16'
    }))
  );

  // Step 4: Schedule publishing
  await publishToSocial({
    content: contentVariants,
    videos,
    platforms: ['facebook', 'instagram', 'tiktok'],
    schedule: 'auto'
  });

  return {
    research,
    contentVariants,
    videos
  };
}
```

### Custom Content Format

```typescript
interface ContentConfig {
  tone: 'professional' | 'friendly' | 'humorous';
  length: 'short' | 'medium' | 'long';
  includeStats: boolean;
  targetAudience: string;
}

async function generateCustomContent(
  topic: string,
  config: ContentConfig
) {
  const research = await researchTopic({ keyword: topic });
  
  const systemPrompt = `
    You are a ${config.tone} content writer.
    Target audience: ${config.targetAudience}
    ${config.includeStats ? 'Include data and statistics.' : ''}
    Length: ${config.length}
  `;

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: config.length === 'long' ? 8192 : 2048,
    system: systemPrompt,
    messages: [{
      role: 'user',
      content: `Write about: ${topic}\n\nResearch data: ${JSON.stringify(research)}`
    }]
  });

  return message.content[0].text;
}
```

## Running the Application

```bash
# Development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Render Remotion videos
npm run render
```

## API Routes

The Next.js app provides REST endpoints:

### Generate Content

```typescript
// POST /api/generate
// Body: { keyword: string, format: string, language: string }

fetch('/api/generate', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    keyword: 'AI marketing trends',
    format: 'toplist',
    language: 'en'
  })
});
```

### Render Video

```typescript
// POST /api/render-video
// Body: { contentId: string, format: string }

fetch('/api/render-video', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    contentId: 'article-123',
    format: 'reels'
  })
});
```

## Configuration

### Content Templates

Customize templates in `src/lib/templates/`:

```typescript
export const contentTemplates = {
  toplist: {
    structure: ['intro', 'items', 'conclusion'],
    minItems: 5,
    maxItems: 10
  },
  pov: {
    structure: ['hook', 'argument', 'evidence', 'conclusion'],
    tone: 'opinionated'
  },
  caseStudy: {
    structure: ['background', 'challenge', 'solution', 'results'],
    includeMetrics: true
  }
};
```

### Video Templates

Customize Remotion templates in `remotion/`:

```typescript
import { Composition } from 'remotion';

export const RemotionRoot = () => {
  return (
    <>
      <Composition
        id="ContentVideo"
        component={ContentVideoTemplate}
        durationInFrames={300}
        fps={30}
        width={1080}
        height={1920}
      />
    </>
  );
};
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement retry logic with exponential backoff
async function apiCallWithRetry(apiFunction: Function, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await apiFunction();
    } catch (error) {
      if (error.status === 429) {
        await new Promise(resolve => setTimeout(resolve, Math.pow(2, i) * 1000));
        continue;
      }
      throw error;
    }
  }
}
```

### Video Rendering Memory Issues

```typescript
// Reduce concurrency for video rendering
import pLimit from 'p-limit';

const limit = pLimit(2); // Only 2 concurrent renders

const videos = await Promise.all(
  contents.map(content => 
    limit(() => renderVideo(content))
  )
);
```

### Content Quality Issues

```typescript
// Add validation layer
function validateContent(content: string): boolean {
  return (
    content.length > 500 &&
    !content.includes('[INSERT]') &&
    /\d/.test(content) // Contains at least one statistic
  );
}
```

## Best Practices

1. **Cache research data** to avoid redundant API calls
2. **Use queues** (Bull, BullMQ) for video rendering jobs
3. **Implement content moderation** before auto-publishing
4. **Monitor API usage** to stay within rate limits
5. **A/B test** different content formats and tones
6. **Store generated content** in a database for analytics
