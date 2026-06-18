---
name: marketing-content-pipeline-automation
description: Automated AI-powered content pipeline for research, scriptwriting, video generation, and multi-platform publishing with Claude/OpenAI integration
triggers:
  - automate content creation with AI research
  - generate videos from blog content automatically
  - create multilingual marketing content pipeline
  - build automated content workflow with Claude
  - scrape trending news and generate content
  - automate social media content with AI
  - create video content from text using Remotion
  - set up AI-powered marketing automation system
---

# Marketing Content Pipeline Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

Marketing Content Pipeline Automation is a comprehensive TypeScript-based system that automates the entire content creation workflow from research to video generation. It combines AI (Claude 3, OpenAI) for content generation with Remotion for video rendering, creating an end-to-end pipeline that:

- **Auto-researches** trending topics from TechCrunch, a16z, Twitter/X, LinkedIn
- **Generates** content in multiple formats (listicles, POV, case studies, how-tos)
- **Produces** bilingual content (English & Vietnamese)
- **Renders** videos and infographics automatically using Remotion
- **Optimizes** output for multiple platforms (Reels, TikTok, Shorts)

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

## Configuration

### Environment Variables

Create a `.env.local` file with the following variables:

```bash
# AI Providers
OPENAI_API_KEY=your_openai_key_here
ANTHROPIC_API_KEY=your_claude_key_here

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key_here

# Content Settings
DEFAULT_LANGUAGE=en
SECONDARY_LANGUAGE=vi
CONTENT_TONE=professional

# Video Rendering (Remotion)
REMOTION_AWS_ACCESS_KEY_ID=your_aws_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret

# Database (if applicable)
DATABASE_URL=your_database_connection_string
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── research/          # Content research & scraping
│   ├── generation/        # AI content generation
│   ├── video/            # Remotion video rendering
│   ├── api/              # API routes
│   └── utils/            # Helper functions
├── pages/                # Next.js pages
├── public/               # Static assets
└── remotion/             # Video templates
```

## Core Components

### 1. Content Research Module

Automatically scrapes and analyzes trending content:

```typescript
import { ContentResearcher } from './src/research/researcher';

async function researchTopic(keyword: string) {
  const researcher = new ContentResearcher({
    apiKey: process.env.RAPIDAPI_KEY,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeRange: '24h'
  });

  const insights = await researcher.research({
    keyword,
    depth: 'comprehensive',
    includeStats: true,
    language: 'en'
  });

  return insights;
}

// Usage
const data = await researchTopic('AI marketing automation');
console.log(data.articles, data.trends, data.statistics);
```

### 2. AI Content Generation

Generate content in multiple formats using Claude or OpenAI:

```typescript
import { ContentGenerator } from './src/generation/generator';
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

async function generateContent(researchData: any, format: string) {
  const generator = new ContentGenerator({
    provider: 'claude', // or 'openai'
    model: 'claude-3-5-sonnet-20241022',
  });

  const content = await generator.create({
    format, // 'toplist', 'pov', 'case-study', 'how-to'
    researchData,
    tone: 'professional', // 'friendly', 'humorous', 'expert'
    languages: ['en', 'vi'],
    includeSEO: true,
    includeVisuals: true
  });

  return content;
}

// Example with Claude
async function generateWithClaude(topic: string, research: any) {
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [{
      role: 'user',
      content: `Create a comprehensive marketing article about ${topic}.
      
Research data: ${JSON.stringify(research)}

Format: Professional blog post with:
- Engaging headline
- Introduction with hook
- 5 main sections with data-backed insights
- Actionable takeaways
- SEO-optimized structure

Output in both English and Vietnamese.`
    }]
  });

  return message.content;
}
```

### 3. Multi-Format Content Creation

```typescript
interface ContentFormat {
  type: 'toplist' | 'pov' | 'case-study' | 'how-to';
  structure: {
    sections: number;
    wordCount: number;
    includeStats: boolean;
    includeExamples: boolean;
  };
}

async function createMultiFormatContent(keyword: string) {
  const formats: ContentFormat[] = [
    {
      type: 'toplist',
      structure: { sections: 10, wordCount: 2000, includeStats: true, includeExamples: true }
    },
    {
      type: 'how-to',
      structure: { sections: 7, wordCount: 1500, includeStats: false, includeExamples: true }
    }
  ];

  const results = await Promise.all(
    formats.map(format => 
      generateContent({ keyword }, format.type)
    )
  );

  return results;
}
```

### 4. Video Rendering with Remotion

Transform content into video automatically:

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import { VideoComposition } from './remotion/Composition';

async function renderContentVideo(content: any) {
  const bundled = await bundle({
    entryPoint: './src/video/index.ts',
    webpackOverride: (config) => config,
  });

  const composition = await selectComposition({
    serveUrl: bundled,
    id: 'ContentVideo',
    inputProps: {
      title: content.title,
      sections: content.sections,
      branding: {
        logo: '/assets/logo.png',
        colors: {
          primary: '#FF6B6B',
          secondary: '#4ECDC4'
        }
      }
    },
  });

  await renderMedia({
    composition,
    serveUrl: bundled,
    codec: 'h264',
    outputLocation: `out/${content.slug}.mp4`,
    inputProps: composition.inputProps,
  });

  return `out/${content.slug}.mp4`;
}

// Render for multiple platforms
async function renderMultiPlatform(content: any) {
  const platforms = [
    { name: 'reels', width: 1080, height: 1920, fps: 30 },
    { name: 'youtube', width: 1920, height: 1080, fps: 30 },
    { name: 'tiktok', width: 1080, height: 1920, fps: 30 }
  ];

  return Promise.all(
    platforms.map(platform => 
      renderMedia({
        composition: {
          ...composition,
          width: platform.width,
          height: platform.height,
          fps: platform.fps
        },
        serveUrl: bundled,
        codec: 'h264',
        outputLocation: `out/${content.slug}-${platform.name}.mp4`,
      })
    )
  );
}
```

### 5. Complete Pipeline Execution

```typescript
import { ContentPipeline } from './src/pipeline';

async function runFullPipeline(keyword: string) {
  const pipeline = new ContentPipeline({
    researchConfig: {
      sources: ['techcrunch', 'a16z', 'twitter'],
      timeRange: '24h'
    },
    generationConfig: {
      provider: 'claude',
      model: 'claude-3-5-sonnet-20241022',
      formats: ['toplist', 'how-to'],
      languages: ['en', 'vi']
    },
    videoConfig: {
      platforms: ['reels', 'tiktok', 'youtube'],
      template: 'modern'
    }
  });

  const result = await pipeline.execute(keyword);

  return {
    research: result.research,
    content: result.content, // Array of generated articles
    videos: result.videos,   // Array of rendered videos
    metadata: result.metadata
  };
}

// Execute pipeline
const output = await runFullPipeline('AI marketing trends 2024');
console.log(`Generated ${output.content.length} articles`);
console.log(`Rendered ${output.videos.length} videos`);
```

## API Routes (Next.js)

### Research Endpoint

```typescript
// pages/api/research.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { ContentResearcher } from '@/src/research/researcher';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  const { keyword, sources, timeRange } = req.body;

  try {
    const researcher = new ContentResearcher({
      apiKey: process.env.RAPIDAPI_KEY,
      sources: sources || ['techcrunch', 'a16z'],
      timeRange: timeRange || '24h'
    });

    const insights = await researcher.research({ keyword });

    res.status(200).json({ success: true, data: insights });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
}
```

### Generate Content Endpoint

```typescript
// pages/api/generate.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { ContentGenerator } from '@/src/generation/generator';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  const { researchData, format, languages, tone } = req.body;

  try {
    const generator = new ContentGenerator({
      provider: 'claude',
      model: 'claude-3-5-sonnet-20241022',
    });

    const content = await generator.create({
      format,
      researchData,
      tone: tone || 'professional',
      languages: languages || ['en'],
    });

    res.status(200).json({ success: true, content });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
}
```

### Render Video Endpoint

```typescript
// pages/api/render-video.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { renderContentVideo } from '@/src/video/renderer';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  const { content, platform } = req.body;

  try {
    const videoPath = await renderContentVideo(content, platform);
    
    res.status(200).json({ 
      success: true, 
      videoUrl: `/videos/${videoPath}` 
    });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
}
```

## Common Patterns

### Batch Content Generation

```typescript
async function batchGenerateContent(keywords: string[]) {
  const results = [];

  for (const keyword of keywords) {
    const research = await researchTopic(keyword);
    const content = await generateContent(research, 'toplist');
    const video = await renderContentVideo(content);

    results.push({ keyword, content, video });

    // Rate limiting
    await new Promise(resolve => setTimeout(resolve, 2000));
  }

  return results;
}
```

### Content Scheduling

```typescript
interface ScheduledContent {
  content: any;
  publishDate: Date;
  platforms: string[];
}

async function scheduleContent(items: ScheduledContent[]) {
  const scheduled = items.map(item => ({
    ...item,
    status: 'scheduled',
    createdAt: new Date()
  }));

  // Save to database or scheduling service
  return scheduled;
}
```

### Bilingual Content Generation

```typescript
async function generateBilingual(keyword: string) {
  const research = await researchTopic(keyword);

  const [englishContent, vietnameseContent] = await Promise.all([
    generateContent(research, 'how-to', 'en'),
    generateContent(research, 'how-to', 'vi')
  ]);

  return {
    en: englishContent,
    vi: vietnameseContent,
    shared: {
      research,
      metadata: {
        keyword,
        generatedAt: new Date()
      }
    }
  };
}
```

## Troubleshooting

### API Rate Limits

```typescript
class RateLimiter {
  private queue: Array<() => Promise<any>> = [];
  private running = 0;
  
  constructor(private maxConcurrent: number, private delayMs: number) {}

  async add<T>(fn: () => Promise<T>): Promise<T> {
    while (this.running >= this.maxConcurrent) {
      await new Promise(resolve => setTimeout(resolve, this.delayMs));
    }
    
    this.running++;
    try {
      return await fn();
    } finally {
      this.running--;
    }
  }
}

const limiter = new RateLimiter(3, 1000);
await limiter.add(() => generateContent(data, 'toplist'));
```

### Video Rendering Memory Issues

```typescript
// Use streaming for large videos
import { renderMedia } from '@remotion/renderer';

await renderMedia({
  composition,
  serveUrl: bundled,
  codec: 'h264',
  outputLocation: output,
  chromiumOptions: {
    gl: 'angle', // Better GPU handling
    headless: true
  },
  // Reduce memory usage
  concurrency: 2,
  enforceAudioTrack: false
});
```

### Claude API Errors

```typescript
async function generateWithRetry(prompt: string, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      const response = await anthropic.messages.create({
        model: 'claude-3-5-sonnet-20241022',
        max_tokens: 4096,
        messages: [{ role: 'user', content: prompt }]
      });
      return response;
    } catch (error) {
      if (error.status === 529 && i < maxRetries - 1) {
        // Overloaded, wait and retry
        await new Promise(resolve => setTimeout(resolve, 2000 * (i + 1)));
        continue;
      }
      throw error;
    }
  }
}
```

### Research Data Quality

```typescript
function validateResearchData(data: any): boolean {
  return (
    data.articles?.length > 0 &&
    data.trends?.length > 0 &&
    data.statistics &&
    data.articles.every(a => a.title && a.source && a.publishedAt)
  );
}

async function researchWithValidation(keyword: string) {
  const data = await researchTopic(keyword);
  
  if (!validateResearchData(data)) {
    throw new Error('Insufficient research data quality');
  }
  
  return data;
}
```

## Development Commands

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Render video (Remotion)
npm run render

# Test pipeline
npm run test:pipeline

# Lint code
npm run lint
```

## Best Practices

1. **Always validate research data** before content generation
2. **Use environment variables** for all API keys
3. **Implement rate limiting** for external API calls
4. **Cache research results** to avoid redundant API calls
5. **Monitor video rendering** memory usage for large batches
6. **Version control content templates** for reproducibility
7. **Log pipeline execution** for debugging and optimization
