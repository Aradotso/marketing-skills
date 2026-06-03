---
name: ultimate-ai-content-pipeline
description: Automated content creation pipeline from research to video generation using Claude, OpenAI, and Remotion
triggers:
  - "create an automated content pipeline with AI"
  - "generate videos from blog posts automatically"
  - "scrape news and create content with AI"
  - "build a content automation system"
  - "set up AI-powered content research and writing"
  - "automate content creation from research to video"
  - "create multi-format content with Claude and OpenAI"
  - "generate social media videos from articles"
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

An end-to-end content automation system that researches trending topics, generates multi-format content (articles, scripts), and renders videos automatically. Built with Next.js, TypeScript, Claude/OpenAI APIs, and Remotion.

## What It Does

This project automates the entire content creation workflow:

1. **Auto-Research**: Crawls news sources (TechCrunch, a16z, Twitter, LinkedIn) for latest trends
2. **AI Content Generation**: Creates articles in multiple formats (toplist, POV, case study, how-to) using Claude/OpenAI
3. **Multi-language Output**: Generates content in both English and Vietnamese with customizable tone
4. **Video Generation**: Automatically renders infographic videos and social media clips using Remotion
5. **Platform Optimization**: Exports video formats for Reels, TikTok, Shorts

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

## Environment Configuration

Create a `.env.local` file with the following variables:

```bash
# AI APIs
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Application
NEXT_PUBLIC_APP_URL=http://localhost:3000

# Database (if applicable)
DATABASE_URL=your_database_url
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/              # Core libraries
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── research/    # Research/scraping modules
│   │   └── video/       # Remotion video rendering
│   ├── types/           # TypeScript types
│   └── utils/           # Utility functions
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Core Usage Patterns

### 1. Research Module

Automatically scrape and analyze content from news sources:

```typescript
import { researchTopic } from '@/lib/research/scraper';

async function gatherInsights(keyword: string) {
  const research = await researchTopic({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeframe: '24h',
    maxResults: 20
  });

  return {
    articles: research.articles,
    insights: research.insights,
    trends: research.trends,
    statistics: research.statistics
  };
}
```

### 2. AI Content Generation with Claude

Generate content using Claude API:

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

async function generateArticle(topic: string, format: string, tone: string) {
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [
      {
        role: 'user',
        content: `Write a ${format} article about ${topic} in a ${tone} tone. Include data-backed insights and recent trends.`
      }
    ],
  });

  return message.content[0].type === 'text' ? message.content[0].text : '';
}

// Usage
const article = await generateArticle(
  'AI in marketing automation',
  'case-study',
  'professional'
);
```

### 3. Multi-Language Content Generation

Generate content in multiple languages:

```typescript
interface ContentRequest {
  topic: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  languages: ('en' | 'vi')[];
  tone: 'expert' | 'friendly' | 'humorous';
}

async function generateMultiLanguageContent(request: ContentRequest) {
  const results = await Promise.all(
    request.languages.map(async (lang) => {
      const prompt = buildPrompt(request, lang);
      
      const content = await anthropic.messages.create({
        model: 'claude-3-5-sonnet-20241022',
        max_tokens: 4096,
        messages: [{ role: 'user', content: prompt }],
      });

      return {
        language: lang,
        content: content.content[0].type === 'text' ? content.content[0].text : '',
      };
    })
  );

  return results;
}

function buildPrompt(request: ContentRequest, lang: string): string {
  const languageInstructions = {
    en: 'Write in English',
    vi: 'Viết bằng tiếng Việt'
  };

  return `${languageInstructions[lang]}. Format: ${request.format}. Tone: ${request.tone}. Topic: ${request.topic}`;
}
```

### 4. Video Generation with Remotion

Render videos from content:

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  title: string;
  content: string;
  format: 'reels' | 'tiktok' | 'shorts';
  duration: number;
}

async function generateVideo(config: VideoConfig) {
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title: config.title,
      content: config.content,
    },
  });

  const aspectRatios = {
    reels: { width: 1080, height: 1920 }, // 9:16
    tiktok: { width: 1080, height: 1920 },
    shorts: { width: 1080, height: 1920 },
  };

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: `out/${config.format}-${Date.now()}.mp4`,
    ...aspectRatios[config.format],
  });
}
```

### 5. Complete Pipeline Flow

End-to-end content creation:

```typescript
import { researchTopic } from '@/lib/research/scraper';
import { generateContent } from '@/lib/ai/content-generator';
import { renderVideo } from '@/lib/video/renderer';

interface PipelineConfig {
  keyword: string;
  contentFormat: 'toplist' | 'pov' | 'case-study' | 'how-to';
  languages: ('en' | 'vi')[];
  generateVideo: boolean;
  videoFormat?: 'reels' | 'tiktok' | 'shorts';
}

async function runContentPipeline(config: PipelineConfig) {
  // Step 1: Research
  console.log('🔍 Researching topic...');
  const research = await researchTopic({
    keyword: config.keyword,
    sources: ['techcrunch', 'a16z', 'twitter'],
    timeframe: '24h',
  });

  // Step 2: Generate content
  console.log('✍️ Generating content...');
  const content = await generateContent({
    topic: config.keyword,
    format: config.contentFormat,
    languages: config.languages,
    research: research.insights,
    tone: 'professional',
  });

  // Step 3: Generate video (optional)
  if (config.generateVideo && config.videoFormat) {
    console.log('🎬 Rendering video...');
    await renderVideo({
      title: content.title,
      content: content.body,
      format: config.videoFormat,
      duration: 60,
    });
  }

  return {
    research,
    content,
    status: 'completed',
  };
}

// Usage
const result = await runContentPipeline({
  keyword: 'AI marketing automation 2024',
  contentFormat: 'toplist',
  languages: ['en', 'vi'],
  generateVideo: true,
  videoFormat: 'reels',
});
```

## API Endpoints

### Create Content Pipeline

```typescript
// app/api/pipeline/route.ts
import { NextRequest, NextResponse } from 'next/server';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    
    const result = await runContentPipeline({
      keyword: body.keyword,
      contentFormat: body.format,
      languages: body.languages || ['en'],
      generateVideo: body.generateVideo || false,
      videoFormat: body.videoFormat,
    });

    return NextResponse.json(result, { status: 200 });
  } catch (error) {
    return NextResponse.json(
      { error: 'Pipeline failed', details: error.message },
      { status: 500 }
    );
  }
}
```

### Research Only

```typescript
// app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { researchTopic } from '@/lib/research/scraper';

export async function POST(request: NextRequest) {
  const { keyword, sources, timeframe } = await request.json();

  const research = await researchTopic({
    keyword,
    sources: sources || ['techcrunch', 'a16z'],
    timeframe: timeframe || '24h',
  });

  return NextResponse.json(research);
}
```

## Common Patterns

### Custom Content Templates

```typescript
interface ContentTemplate {
  name: string;
  structure: string[];
  promptTemplate: string;
}

const templates: Record<string, ContentTemplate> = {
  toplist: {
    name: 'Top List',
    structure: ['intro', 'items', 'conclusion'],
    promptTemplate: 'Create a top 10 list about {topic}. Each item should have a title, description, and example.',
  },
  'case-study': {
    name: 'Case Study',
    structure: ['problem', 'solution', 'results', 'lessons'],
    promptTemplate: 'Write a detailed case study about {topic}. Include real-world examples and data.',
  },
  'how-to': {
    name: 'How-To Guide',
    structure: ['overview', 'steps', 'tips', 'conclusion'],
    promptTemplate: 'Create a comprehensive how-to guide for {topic}. Include step-by-step instructions.',
  },
};

function getTemplate(format: string): ContentTemplate {
  return templates[format] || templates['toplist'];
}
```

### Batch Processing

```typescript
async function batchGenerateContent(keywords: string[]) {
  const results = [];

  for (const keyword of keywords) {
    try {
      const result = await runContentPipeline({
        keyword,
        contentFormat: 'toplist',
        languages: ['en', 'vi'],
        generateVideo: false,
      });
      results.push({ keyword, status: 'success', data: result });
    } catch (error) {
      results.push({ keyword, status: 'failed', error: error.message });
    }

    // Rate limiting
    await new Promise(resolve => setTimeout(resolve, 2000));
  }

  return results;
}
```

## Development

```bash
# Run development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Type checking
npm run type-check

# Lint code
npm run lint
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement retry logic with exponential backoff
async function callAIWithRetry(prompt: string, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await anthropic.messages.create({
        model: 'claude-3-5-sonnet-20241022',
        max_tokens: 4096,
        messages: [{ role: 'user', content: prompt }],
      });
    } catch (error) {
      if (error.status === 429 && i < maxRetries - 1) {
        const delay = Math.pow(2, i) * 1000;
        await new Promise(resolve => setTimeout(resolve, delay));
        continue;
      }
      throw error;
    }
  }
}
```

### Video Rendering Issues

- Ensure Remotion dependencies are installed: `npm install @remotion/bundler @remotion/renderer`
- Check that ffmpeg is available on your system
- Verify video composition IDs match your Remotion project
- Use smaller video dimensions for faster rendering during development

### Research Scraping Failures

- Verify RapidAPI key is valid and has credits
- Check source-specific API endpoints are accessible
- Implement fallback sources if primary sources fail
- Cache research results to reduce API calls

### Memory Issues with Large Content

```typescript
// Stream large content instead of loading all at once
async function* streamContent(topic: string) {
  const stream = await anthropic.messages.stream({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [{ role: 'user', content: `Write about ${topic}` }],
  });

  for await (const chunk of stream) {
    if (chunk.type === 'content_block_delta' && chunk.delta.type === 'text_delta') {
      yield chunk.delta.text;
    }
  }
}
```

## Best Practices

1. **Always use environment variables** for API keys
2. **Implement rate limiting** when calling external APIs
3. **Cache research results** to avoid redundant API calls
4. **Use streaming** for large content generation
5. **Validate user input** before processing
6. **Handle errors gracefully** with proper fallbacks
7. **Monitor API usage** to avoid unexpected costs
8. **Test video rendering** locally before deploying
