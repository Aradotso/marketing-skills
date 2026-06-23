---
name: marketing-pipeline-ai-content-automation
description: Automated AI content pipeline for research, scriptwriting, posting, and video generation using Claude/OpenAI and Remotion
triggers:
  - set up automated content pipeline with AI
  - create AI-powered marketing content automation
  - configure content research and video generation system
  - build automated social media content workflow
  - implement AI content creation with Claude and OpenAI
  - automate content from research to video using Remotion
  - set up multi-language content generation pipeline
  - create automated marketing content with video rendering
---

# Marketing Pipeline AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

Marketing Pipeline is a complete AI-powered content automation system that handles the entire content lifecycle: from researching trending topics, generating multilingual articles, to creating videos. Built with Next.js and TypeScript, it integrates Claude 3, OpenAI, and Remotion to create a fully automated content factory.

**Key capabilities:**
- Auto-crawl and research from TechCrunch, a16z, Twitter/X, LinkedIn
- Generate content in multiple formats (Top Lists, POV, Case Studies, How-To)
- Bilingual content generation (English & Vietnamese)
- Automatic video rendering with Remotion
- Social media optimization for Reels, TikTok, Shorts

## Installation

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
pnpm install

# Set up environment variables
cp .env.example .env.local
```

## Environment Configuration

Create `.env.local` with the following variables:

```bash
# AI Models
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_url

# Remotion (for video rendering)
REMOTION_LICENSE_KEY=your_remotion_license

# Optional: Social Media APIs
FACEBOOK_ACCESS_TOKEN=your_fb_token
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── research/    # Content research crawlers
│   │   ├── content/     # Content generation logic
│   │   └── video/       # Remotion video rendering
│   ├── types/           # TypeScript definitions
│   └── utils/           # Utility functions
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Core Usage Patterns

### 1. Content Research & Crawling

```typescript
import { ResearchService } from '@/lib/research/research-service';

// Initialize research service
const researchService = new ResearchService({
  rapidApiKey: process.env.RAPIDAPI_KEY!,
});

// Research a topic
async function researchTopic(keyword: string) {
  const results = await researchService.crawlSources({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeRange: '24h',
    maxResults: 20,
  });

  // Extract insights
  const insights = await researchService.extractInsights(results);
  
  return {
    rawData: results,
    insights,
    trends: insights.trends,
    dataPoints: insights.statistics,
  };
}
```

### 2. AI Content Generation with Claude

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY!,
});

interface ContentGenerationOptions {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  tone: 'expert' | 'friendly' | 'humorous';
  language: 'en' | 'vi';
  researchData: any;
}

async function generateContent(options: ContentGenerationOptions) {
  const { keyword, format, tone, language, researchData } = options;

  const prompt = `
You are an expert content creator. Using the following research data:
${JSON.stringify(researchData, null, 2)}

Create a ${format} article about "${keyword}" in ${language}.
Tone: ${tone}
Include:
- Compelling headline
- Data-backed insights
- Actionable takeaways
- SEO-optimized structure
`;

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4000,
    messages: [
      {
        role: 'user',
        content: prompt,
      },
    ],
  });

  return message.content[0].text;
}
```

### 3. Bilingual Content Generation

```typescript
interface BilingualContentResult {
  english: string;
  vietnamese: string;
}

async function generateBilingualContent(
  keyword: string,
  researchData: any
): Promise<BilingualContentResult> {
  const [englishContent, vietnameseContent] = await Promise.all([
    generateContent({
      keyword,
      format: 'toplist',
      tone: 'expert',
      language: 'en',
      researchData,
    }),
    generateContent({
      keyword,
      format: 'toplist',
      tone: 'expert',
      language: 'vi',
      researchData,
    }),
  ]);

  return {
    english: englishContent,
    vietnamese: vietnameseContent,
  };
}
```

### 4. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoGenerationOptions {
  content: string;
  title: string;
  outputFormat: 'reels' | 'tiktok' | 'shorts';
}

async function generateVideo(options: VideoGenerationOptions) {
  const { content, title, outputFormat } = options;

  // Video dimensions based on format
  const dimensions = {
    reels: { width: 1080, height: 1920 },
    tiktok: { width: 1080, height: 1920 },
    shorts: { width: 1080, height: 1920 },
  };

  const { width, height } = dimensions[outputFormat];

  // Bundle Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.join(process.cwd(), 'remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  // Get composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title,
      content: content.substring(0, 500), // Limit for video
      branding: {
        logo: '/logo.png',
        colors: {
          primary: '#FF6B6B',
          secondary: '#4ECDC4',
        },
      },
    },
  });

  // Render video
  const outputPath = path.join(
    process.cwd(),
    'output',
    `${Date.now()}-${outputFormat}.mp4`
  );

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: composition.defaultProps,
  });

  return outputPath;
}
```

### 5. Complete Pipeline Orchestration

```typescript
interface PipelineConfig {
  keyword: string;
  contentFormat: 'toplist' | 'pov' | 'case-study' | 'how-to';
  generateVideo: boolean;
  autoPost: boolean;
}

class ContentPipeline {
  private researchService: ResearchService;

  constructor() {
    this.researchService = new ResearchService({
      rapidApiKey: process.env.RAPIDAPI_KEY!,
    });
  }

  async execute(config: PipelineConfig) {
    console.log(`🚀 Starting pipeline for: ${config.keyword}`);

    // Step 1: Research
    console.log('📡 Researching topic...');
    const researchData = await this.researchService.crawlSources({
      keyword: config.keyword,
      sources: ['techcrunch', 'twitter'],
      timeRange: '24h',
      maxResults: 15,
    });

    const insights = await this.researchService.extractInsights(researchData);

    // Step 2: Generate Content
    console.log('🧠 Generating bilingual content...');
    const content = await generateBilingualContent(config.keyword, insights);

    // Step 3: Generate Video (if enabled)
    let videoPath: string | null = null;
    if (config.generateVideo) {
      console.log('🎬 Rendering video...');
      videoPath = await generateVideo({
        content: content.english,
        title: config.keyword,
        outputFormat: 'reels',
      });
    }

    // Step 4: Auto-post (if enabled)
    if (config.autoPost && videoPath) {
      console.log('📤 Publishing to social media...');
      await this.postToSocialMedia({
        content: content.english,
        videoPath,
      });
    }

    console.log('✅ Pipeline completed!');

    return {
      content,
      videoPath,
      insights,
    };
  }

  private async postToSocialMedia(data: {
    content: string;
    videoPath: string;
  }) {
    // Implementation for Facebook, Instagram, etc.
    // Using respective APIs with tokens from env
  }
}
```

## API Routes (Next.js)

### Generate Content API

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { ContentPipeline } from '@/lib/pipeline';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const { keyword, contentFormat, generateVideo } = body;

    if (!keyword) {
      return NextResponse.json(
        { error: 'Keyword is required' },
        { status: 400 }
      );
    }

    const pipeline = new ContentPipeline();
    const result = await pipeline.execute({
      keyword,
      contentFormat: contentFormat || 'toplist',
      generateVideo: generateVideo || false,
      autoPost: false,
    });

    return NextResponse.json({
      success: true,
      data: result,
    });
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { error: 'Failed to generate content' },
      { status: 500 }
    );
  }
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

# Render Remotion video locally (for testing)
npm run remotion:render
```

## CLI Usage (if available)

```bash
# Generate content from command line
npm run generate -- --keyword "AI trends 2024" --format toplist --video

# Research only
npm run research -- --keyword "AI trends 2024" --sources techcrunch,twitter

# Render video from existing content
npm run render-video -- --content-id abc123 --format reels
```

## Common Patterns

### Custom Content Templates

```typescript
// src/lib/content/templates.ts
export const contentTemplates = {
  toplist: `
# {title}

{introduction}

## Top {count} {topic}

{items}

## Key Takeaways

{takeaways}
  `,
  
  pov: `
# {title}

## My Take on {topic}

{perspective}

## Why This Matters

{analysis}

## What's Next

{predictions}
  `,
};

export function formatContent(
  template: keyof typeof contentTemplates,
  data: Record<string, any>
): string {
  let content = contentTemplates[template];
  
  Object.keys(data).forEach((key) => {
    content = content.replace(new RegExp(`{${key}}`, 'g'), data[key]);
  });
  
  return content;
}
```

### Scheduled Content Generation

```typescript
import cron from 'node-cron';

// Schedule content generation daily at 9 AM
cron.schedule('0 9 * * *', async () => {
  const pipeline = new ContentPipeline();
  
  const topics = ['AI trends', 'Marketing automation', 'Social media'];
  
  for (const topic of topics) {
    await pipeline.execute({
      keyword: topic,
      contentFormat: 'toplist',
      generateVideo: true,
      autoPost: true,
    });
  }
});
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement rate limiting for AI APIs
import pLimit from 'p-limit';

const limit = pLimit(2); // Max 2 concurrent requests

async function batchGenerate(keywords: string[]) {
  const promises = keywords.map((keyword) =>
    limit(() => generateContent({ keyword, /* ... */ }))
  );
  
  return Promise.all(promises);
}
```

### Video Rendering Memory Issues

```bash
# Increase Node.js memory limit
NODE_OPTIONS="--max-old-space-size=4096" npm run remotion:render
```

### Claude API Errors

```typescript
async function generateWithRetry(options: ContentGenerationOptions, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await generateContent(options);
    } catch (error: any) {
      if (error.status === 429) {
        // Rate limited - wait and retry
        await new Promise(resolve => setTimeout(resolve, 2000 * (i + 1)));
        continue;
      }
      throw error;
    }
  }
  throw new Error('Max retries exceeded');
}
```

### Research Data Quality

```typescript
// Filter and validate research results
function validateResearchData(data: any[]) {
  return data.filter((item) => {
    return (
      item.content &&
      item.content.length > 100 &&
      item.publishedDate &&
      new Date(item.publishedDate) > new Date(Date.now() - 7 * 24 * 60 * 60 * 1000)
    );
  });
}
```

## Best Practices

1. **Always validate API keys** before running the pipeline
2. **Cache research data** to avoid redundant API calls
3. **Use queue systems** (Bull, BeeQueue) for large-scale content generation
4. **Monitor API costs** - set up alerts for Claude/OpenAI usage
5. **Test video templates** locally before rendering at scale
6. **Version your prompts** for reproducible content generation
7. **Implement content moderation** before auto-posting
