---
name: marketing-pipeline-ai-content
description: AI-powered content automation pipeline that researches, generates scripts, and creates videos automatically using Claude/OpenAI and Remotion
triggers:
  - how do I automate content creation with AI research
  - set up the marketing pipeline for auto-posting
  - generate videos from content automatically
  - use Claude to research and write content
  - create content pipeline with Remotion rendering
  - automate social media content generation
  - build AI content workflow with research crawling
  - configure the marketing automation pipeline
---

# Marketing Pipeline AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the **Ultimate AI Content Pipeline** - a TypeScript-based content automation system that handles research, scriptwriting, and video generation using AI (Claude 3, OpenAI) and Remotion for video rendering.

## What This Project Does

The Marketing Pipeline is an end-to-end content automation system that:

- **Auto-crawls research** from sources like TechCrunch, a16z, Twitter/X, and LinkedIn
- **Generates content** in multiple formats (toplists, POV, case studies, how-tos) using Claude/OpenAI
- **Creates multilingual content** (English & Vietnamese) with customizable tone
- **Renders videos automatically** using Remotion for social media platforms
- **Provides a Next.js interface** for managing the entire pipeline

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

## Configuration

Create a `.env.local` file in the root directory:

```env
# AI Services
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Content Settings
DEFAULT_LANGUAGE=en
TONE=professional
```

## Project Structure

```
marketing-pipeline/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── research/     # Research crawling modules
│   │   ├── ai/           # AI generation (Claude/OpenAI)
│   │   ├── render/       # Remotion video rendering
│   │   └── utils/        # Helper functions
│   └── types/            # TypeScript type definitions
├── public/               # Static assets
└── remotion/            # Remotion compositions
```

## Core API Usage

### 1. Research Content Crawling

```typescript
import { crawlResearch } from '@/lib/research/crawler';

interface ResearchOptions {
  keyword: string;
  sources: ('techcrunch' | 'a16z' | 'twitter' | 'linkedin')[];
  timeframe: '24h' | '7d' | '30d';
}

async function gatherResearch(options: ResearchOptions) {
  const research = await crawlResearch({
    keyword: options.keyword,
    sources: options.sources,
    timeframe: options.timeframe,
  });
  
  return research; // Returns { articles: [], insights: [], data: [] }
}

// Example usage
const data = await gatherResearch({
  keyword: 'AI automation',
  sources: ['techcrunch', 'twitter'],
  timeframe: '24h',
});
```

### 2. AI Content Generation

```typescript
import { generateContent } from '@/lib/ai/generator';
import { Anthropic } from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

interface ContentConfig {
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  tone: 'professional' | 'friendly' | 'humorous';
  language: 'en' | 'vi';
  research: any;
}

async function createContent(config: ContentConfig) {
  const prompt = `
Based on this research: ${JSON.stringify(config.research)}

Create a ${config.format} article in ${config.language} with a ${config.tone} tone.
Include data-backed insights and real examples.
  `;

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [{
      role: 'user',
      content: prompt,
    }],
  });

  return message.content[0].text;
}
```

### 3. OpenAI Alternative

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateWithOpenAI(prompt: string) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: 'You are an expert content creator specializing in marketing and social media.',
      },
      {
        role: 'user',
        content: prompt,
      },
    ],
    temperature: 0.7,
    max_tokens: 3000,
  });

  return completion.choices[0].message.content;
}
```

### 4. Video Rendering with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  title: string;
  content: string;
  duration: number;
  format: 'reels' | 'tiktok' | 'shorts';
}

async function renderContentVideo(config: VideoConfig) {
  const bundleLocation = await bundle({
    entryPoint: path.join(process.cwd(), 'remotion/index.ts'),
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

  const dimensions = {
    reels: { width: 1080, height: 1920 },
    tiktok: { width: 1080, height: 1920 },
    shorts: { width: 1080, height: 1920 },
  };

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: `out/${config.title}.mp4`,
    ...dimensions[config.format],
  });
}
```

## Common Patterns

### Complete Content Pipeline

```typescript
import { crawlResearch } from '@/lib/research/crawler';
import { generateContent } from '@/lib/ai/generator';
import { renderContentVideo } from '@/lib/render/video';

async function runContentPipeline(keyword: string) {
  try {
    // Step 1: Research
    console.log('Starting research...');
    const research = await crawlResearch({
      keyword,
      sources: ['techcrunch', 'twitter'],
      timeframe: '24h',
    });

    // Step 2: Generate content
    console.log('Generating content...');
    const content = await createContent({
      format: 'toplist',
      tone: 'professional',
      language: 'en',
      research,
    });

    // Step 3: Render video
    console.log('Rendering video...');
    await renderContentVideo({
      title: keyword,
      content,
      duration: 30,
      format: 'reels',
    });

    return {
      success: true,
      content,
      videoPath: `out/${keyword}.mp4`,
    };
  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}
```

### Bilingual Content Generation

```typescript
async function generateBilingualContent(research: any) {
  const [englishContent, vietnameseContent] = await Promise.all([
    createContent({
      format: 'pov',
      tone: 'professional',
      language: 'en',
      research,
    }),
    createContent({
      format: 'pov',
      tone: 'professional',
      language: 'vi',
      research,
    }),
  ]);

  return {
    en: englishContent,
    vi: vietnameseContent,
  };
}
```

### Batch Content Creation

```typescript
async function createMultipleFormats(keyword: string) {
  const research = await crawlResearch({
    keyword,
    sources: ['techcrunch', 'a16z'],
    timeframe: '7d',
  });

  const formats: Array<'toplist' | 'pov' | 'case-study' | 'how-to'> = [
    'toplist',
    'pov',
    'case-study',
    'how-to',
  ];

  const contents = await Promise.all(
    formats.map((format) =>
      createContent({
        format,
        tone: 'professional',
        language: 'en',
        research,
      })
    )
  );

  return formats.reduce((acc, format, index) => {
    acc[format] = contents[index];
    return acc;
  }, {} as Record<string, string>);
}
```

## Running the Application

### Development Server

```bash
npm run dev
# or
yarn dev
# or
pnpm dev
```

Visit `http://localhost:3000` to access the Next.js interface.

### Build for Production

```bash
npm run build
npm start
```

### Render Videos Only

```bash
# If the project has a dedicated video rendering script
npm run render
```

## API Routes (Next.js)

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline';

export async function POST(request: NextRequest) {
  const { keyword, format, language } = await request.json();

  try {
    const result = await runContentPipeline(keyword);
    
    return NextResponse.json({
      success: true,
      data: result,
    });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement retry logic with exponential backoff
async function retryWithBackoff<T>(
  fn: () => Promise<T>,
  maxRetries = 3
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      await new Promise(resolve => setTimeout(resolve, Math.pow(2, i) * 1000));
    }
  }
  throw new Error('Max retries exceeded');
}
```

### Video Rendering Memory Issues

```typescript
// Use smaller compositions or split rendering
const composition = await selectComposition({
  serveUrl: bundleLocation,
  id: 'ContentVideo',
  inputProps: {
    title: config.title,
    content: config.content.slice(0, 500), // Limit content length
  },
});
```

### Claude/OpenAI Token Limits

```typescript
function truncateContent(text: string, maxTokens = 3000): string {
  // Rough estimate: 1 token ≈ 4 characters
  const maxChars = maxTokens * 4;
  return text.length > maxChars ? text.slice(0, maxChars) : text;
}
```

## Environment Variables Reference

| Variable | Required | Description |
|----------|----------|-------------|
| `ANTHROPIC_API_KEY` | Yes | Claude API key from Anthropic |
| `OPENAI_API_KEY` | Optional | OpenAI API key (alternative to Claude) |
| `RAPIDAPI_KEY` | Yes | RapidAPI key for research crawling |
| `DEFAULT_LANGUAGE` | No | Default content language (en/vi) |
| `TONE` | No | Default content tone |

## Best Practices

1. **Always validate research data** before passing to AI generators
2. **Cache research results** to avoid redundant API calls
3. **Use environment-specific configs** for development vs production
4. **Monitor API usage** to stay within rate limits
5. **Test video renders locally** before batch processing
6. **Implement proper error logging** for production debugging
