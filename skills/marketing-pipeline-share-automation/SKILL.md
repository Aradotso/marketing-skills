---
name: marketing-pipeline-share-automation
description: AI-powered content automation pipeline from research to video generation using Claude, OpenAI, and Remotion
triggers:
  - automate content creation with AI research and video generation
  - set up marketing pipeline with Claude and OpenAI
  - create automated content workflow from research to video
  - build AI content pipeline with Remotion video rendering
  - generate content automatically from keyword to published video
  - implement content automation with crawling and AI writing
  - use marketing-pipeline-share for content generation
  - configure AI content pipeline with multi-language support
---

# Marketing Pipeline Share Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to help developers use **marketing-pipeline-share**, a comprehensive content automation system that handles research, scriptwriting, and video generation. The pipeline crawls real-time data from TechCrunch, a16z, Twitter, and LinkedIn, generates content in multiple formats and languages using Claude/OpenAI, and renders videos using Remotion.

## What This Project Does

Marketing Pipeline Share is an end-to-end content automation pipeline that:

- **Auto-crawls** recent news/data from major tech sources (last 24h)
- **Generates AI content** in multiple formats (Toplist, POV, Case Study, How-to) using Claude 3 or OpenAI
- **Supports multi-language** output (English & Vietnamese) with customizable tone
- **Renders videos automatically** using Remotion for social media platforms
- **Exports ready-to-publish** content for Reels, TikTok, Shorts

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

### Environment Configuration

Create a `.env.local` file in the root directory:

```bash
# AI Providers
ANTHROPIC_API_KEY=your_anthropic_key_here
OPENAI_API_KEY=your_openai_key_here

# Research APIs (RapidAPI)
RAPIDAPI_KEY=your_rapidapi_key_here

# Optional: Custom API endpoints
NEXT_PUBLIC_API_BASE_URL=http://localhost:3000

# Remotion Configuration
REMOTION_AWS_ACCESS_KEY_ID=your_aws_access_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret_key
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI provider integrations
│   │   ├── crawler/     # Web scraping modules
│   │   ├── content/     # Content generation logic
│   │   └── video/       # Remotion video rendering
│   └── types/           # TypeScript type definitions
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Core APIs and Usage

### 1. Content Research API

The research module crawls and analyzes recent content from multiple sources:

```typescript
// src/lib/crawler/research.ts
import { fetchTechCrunchArticles } from './sources/techcrunch';
import { fetchTwitterTrends } from './sources/twitter';
import { fetchLinkedInPosts } from './sources/linkedin';

interface ResearchResult {
  keyword: string;
  articles: Article[];
  insights: Insight[];
  timestamp: Date;
}

export async function conductResearch(
  keyword: string,
  options?: {
    sources?: ('techcrunch' | 'twitter' | 'linkedin' | 'a16z')[];
    timeRange?: '24h' | '7d' | '30d';
  }
): Promise<ResearchResult> {
  const sources = options?.sources || ['techcrunch', 'twitter', 'linkedin'];
  const timeRange = options?.timeRange || '24h';

  const results = await Promise.all([
    sources.includes('techcrunch') ? fetchTechCrunchArticles(keyword, timeRange) : [],
    sources.includes('twitter') ? fetchTwitterTrends(keyword, timeRange) : [],
    sources.includes('linkedin') ? fetchLinkedInPosts(keyword, timeRange) : [],
  ]);

  return {
    keyword,
    articles: results.flat(),
    insights: await extractInsights(results.flat()),
    timestamp: new Date(),
  };
}
```

**Usage Example:**

```typescript
import { conductResearch } from '@/lib/crawler/research';

const research = await conductResearch('AI automation', {
  sources: ['techcrunch', 'twitter'],
  timeRange: '24h'
});

console.log(`Found ${research.articles.length} relevant articles`);
```

### 2. AI Content Generation

Generate content using Claude or OpenAI with customizable formats:

```typescript
// src/lib/ai/generate-content.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

interface ContentOptions {
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  provider: 'claude' | 'openai';
}

export async function generateContent(
  research: ResearchResult,
  options: ContentOptions
): Promise<string> {
  const prompt = buildPrompt(research, options);

  if (options.provider === 'claude') {
    const anthropic = new Anthropic({
      apiKey: process.env.ANTHROPIC_API_KEY,
    });

    const message = await anthropic.messages.create({
      model: 'claude-3-5-sonnet-20241022',
      max_tokens: 4096,
      messages: [
        {
          role: 'user',
          content: prompt,
        },
      ],
    });

    return message.content[0].type === 'text' 
      ? message.content[0].text 
      : '';
  } else {
    const openai = new OpenAI({
      apiKey: process.env.OPENAI_API_KEY,
    });

    const completion = await openai.chat.completions.create({
      model: 'gpt-4-turbo-preview',
      messages: [
        {
          role: 'system',
          content: 'You are an expert content creator specializing in marketing content.',
        },
        {
          role: 'user',
          content: prompt,
        },
      ],
      max_tokens: 4096,
    });

    return completion.choices[0].message.content || '';
  }
}

function buildPrompt(research: ResearchResult, options: ContentOptions): string {
  const formatInstructions = {
    'toplist': 'Create a numbered list article with clear headings and explanations',
    'pov': 'Write from a strong perspective with personal insights and opinions',
    'case-study': 'Structure as a detailed case study with problem, solution, and results',
    'how-to': 'Create a step-by-step tutorial with actionable instructions',
  };

  const toneInstructions = {
    'expert': 'Use professional, authoritative language with industry terminology',
    'friendly': 'Write in a conversational, approachable tone',
    'humorous': 'Include wit and light humor while maintaining credibility',
  };

  return `
Create a ${options.format} article in ${options.language === 'en' ? 'English' : 'Vietnamese'} about "${research.keyword}".

FORMAT: ${formatInstructions[options.format]}
TONE: ${toneInstructions[options.tone]}

Use these recent insights and data:
${research.insights.map(i => `- ${i.summary}`).join('\n')}

Source articles:
${research.articles.slice(0, 5).map(a => `- ${a.title} (${a.source})`).join('\n')}

Requirements:
- Be data-driven with specific examples
- Include current trends and statistics
- Make it engaging and valuable
- Length: 1200-1500 words
`;
}
```

**Usage Example:**

```typescript
import { generateContent } from '@/lib/ai/generate-content';

const content = await generateContent(research, {
  format: 'toplist',
  language: 'en',
  tone: 'expert',
  provider: 'claude'
});

console.log('Generated content:', content);
```

### 3. Video Generation with Remotion

Create videos automatically from generated content:

```typescript
// src/lib/video/render.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import { webpack } from '@remotion/bundler';
import path from 'path';

interface VideoConfig {
  title: string;
  content: string;
  style: 'modern' | 'minimal' | 'dynamic';
  platform: 'reels' | 'tiktok' | 'shorts';
}

const platformDimensions = {
  reels: { width: 1080, height: 1920, fps: 30 },
  tiktok: { width: 1080, height: 1920, fps: 30 },
  shorts: { width: 1080, height: 1920, fps: 30 },
};

export async function renderContentVideo(
  config: VideoConfig
): Promise<string> {
  const dimensions = platformDimensions[config.platform];
  
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
      title: config.title,
      content: config.content,
      style: config.style,
    },
  });

  // Render video
  const outputLocation = path.join(
    process.cwd(),
    'public/videos',
    `${Date.now()}-${config.platform}.mp4`
  );

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: {
      title: config.title,
      content: config.content,
      style: config.style,
    },
  });

  return outputLocation;
}
```

**Remotion Video Component:**

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig, spring } from 'remotion';

interface ContentVideoProps {
  title: string;
  content: string;
  style: 'modern' | 'minimal' | 'dynamic';
}

export const ContentVideo: React.FC<ContentVideoProps> = ({ title, content, style }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  const titleAnimation = spring({
    frame,
    fps,
    config: {
      damping: 100,
    },
  });

  const contentLines = content.split('\n').slice(0, 5);

  return (
    <AbsoluteFill
      style={{
        backgroundColor: style === 'modern' ? '#1a1a2e' : '#ffffff',
        padding: 60,
        fontFamily: 'Arial, sans-serif',
      }}
    >
      <div
        style={{
          fontSize: 64,
          fontWeight: 'bold',
          color: style === 'modern' ? '#fff' : '#000',
          marginBottom: 40,
          opacity: titleAnimation,
          transform: `translateY(${(1 - titleAnimation) * 50}px)`,
        }}
      >
        {title}
      </div>

      {contentLines.map((line, index) => {
        const lineAnimation = spring({
          frame: frame - (index + 1) * 10,
          fps,
          config: {
            damping: 100,
          },
        });

        return (
          <div
            key={index}
            style={{
              fontSize: 32,
              color: style === 'modern' ? '#eee' : '#333',
              marginBottom: 20,
              opacity: lineAnimation,
              transform: `translateX(${(1 - lineAnimation) * 30}px)`,
            }}
          >
            {line}
          </div>
        );
      })}
    </AbsoluteFill>
  );
};
```

### 4. Complete Pipeline Orchestration

Combine all steps in a single workflow:

```typescript
// src/lib/pipeline/orchestrator.ts
import { conductResearch } from '../crawler/research';
import { generateContent } from '../ai/generate-content';
import { renderContentVideo } from '../video/render';

interface PipelineConfig {
  keyword: string;
  contentFormat: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  aiProvider: 'claude' | 'openai';
  generateVideo: boolean;
  videoPlatform?: 'reels' | 'tiktok' | 'shorts';
}

export async function runContentPipeline(config: PipelineConfig) {
  // Step 1: Research
  console.log('🔍 Starting research phase...');
  const research = await conductResearch(config.keyword, {
    timeRange: '24h'
  });

  // Step 2: Generate Content
  console.log('✍️ Generating content...');
  const content = await generateContent(research, {
    format: config.contentFormat,
    language: config.language,
    tone: config.tone,
    provider: config.aiProvider,
  });

  // Step 3: Render Video (optional)
  let videoPath: string | null = null;
  if (config.generateVideo && config.videoPlatform) {
    console.log('🎬 Rendering video...');
    videoPath = await renderContentVideo({
      title: research.keyword,
      content: content.slice(0, 500),
      style: 'modern',
      platform: config.videoPlatform,
    });
  }

  return {
    research,
    content,
    videoPath,
    timestamp: new Date(),
  };
}
```

**API Route Example:**

```typescript
// src/app/api/pipeline/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline/orchestrator';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    
    const result = await runContentPipeline({
      keyword: body.keyword,
      contentFormat: body.format || 'toplist',
      language: body.language || 'en',
      tone: body.tone || 'expert',
      aiProvider: body.provider || 'claude',
      generateVideo: body.generateVideo || false,
      videoPlatform: body.videoPlatform,
    });

    return NextResponse.json({
      success: true,
      data: result,
    });
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { success: false, error: 'Pipeline execution failed' },
      { status: 500 }
    );
  }
}
```

## Common Patterns

### Pattern 1: Batch Content Generation

Generate multiple content pieces from a single research phase:

```typescript
async function batchGenerate(keyword: string) {
  const research = await conductResearch(keyword);
  
  const formats: Array<'toplist' | 'pov' | 'case-study' | 'how-to'> = [
    'toplist',
    'how-to',
    'case-study'
  ];

  const contents = await Promise.all(
    formats.map(format =>
      generateContent(research, {
        format,
        language: 'en',
        tone: 'expert',
        provider: 'claude'
      })
    )
  );

  return formats.map((format, index) => ({
    format,
    content: contents[index]
  }));
}
```

### Pattern 2: Multi-Language Content

Generate the same content in multiple languages:

```typescript
async function generateMultiLingual(research: ResearchResult) {
  const languages: Array<'en' | 'vi'> = ['en', 'vi'];
  
  return await Promise.all(
    languages.map(lang =>
      generateContent(research, {
        format: 'toplist',
        language: lang,
        tone: 'friendly',
        provider: 'claude'
      })
    )
  );
}
```

### Pattern 3: Scheduled Pipeline Execution

Run the pipeline on a schedule using cron jobs:

```typescript
// src/lib/cron/scheduled-pipeline.ts
import { runContentPipeline } from '../pipeline/orchestrator';

export async function dailyContentGeneration() {
  const keywords = ['AI automation', 'Marketing trends', 'Content creation'];
  
  for (const keyword of keywords) {
    try {
      await runContentPipeline({
        keyword,
        contentFormat: 'toplist',
        language: 'en',
        tone: 'expert',
        aiProvider: 'claude',
        generateVideo: true,
        videoPlatform: 'reels',
      });
      
      console.log(`✅ Completed pipeline for: ${keyword}`);
    } catch (error) {
      console.error(`❌ Failed for ${keyword}:`, error);
    }
  }
}
```

## Configuration

### AI Provider Selection

Choose between Claude and OpenAI based on your needs:

```typescript
// config/ai-providers.ts
export const AI_PROVIDER_CONFIG = {
  claude: {
    model: 'claude-3-5-sonnet-20241022',
    maxTokens: 4096,
    temperature: 0.7,
    bestFor: ['long-form', 'analysis', 'nuanced-content'],
  },
  openai: {
    model: 'gpt-4-turbo-preview',
    maxTokens: 4096,
    temperature: 0.7,
    bestFor: ['creative', 'conversational', 'quick-iterations'],
  },
};
```

### Research Source Configuration

Customize which sources to crawl:

```typescript
// config/research-sources.ts
export const RESEARCH_CONFIG = {
  sources: {
    techcrunch: {
      enabled: true,
      apiEndpoint: 'https://api.techcrunch.com',
      priority: 1,
    },
    twitter: {
      enabled: true,
      apiKey: process.env.RAPIDAPI_KEY,
      priority: 2,
    },
    linkedin: {
      enabled: true,
      apiKey: process.env.RAPIDAPI_KEY,
      priority: 3,
    },
  },
  cacheTimeout: 3600, // 1 hour in seconds
};
```

## Troubleshooting

### Issue: API Rate Limits

**Problem:** Hitting rate limits on AI providers or research APIs.

**Solution:**
```typescript
// Implement exponential backoff
async function retryWithBackoff<T>(
  fn: () => Promise<T>,
  maxRetries = 3
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      const delay = Math.pow(2, i) * 1000;
      await new Promise(resolve => setTimeout(resolve, delay));
    }
  }
  throw new Error('Max retries exceeded');
}

// Usage
const content = await retryWithBackoff(() =>
  generateContent(research, options)
);
```

### Issue: Video Rendering Failures

**Problem:** Remotion rendering crashes or produces corrupted videos.

**Solution:**
```typescript
// Add error handling and logging
import { renderMedia } from '@remotion/renderer';

try {
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    onProgress: ({ progress }) => {
      console.log(`Rendering progress: ${Math.round(progress * 100)}%`);
    },
    onDownload: ({ progress }) => {
      console.log(`Download progress: ${Math.round(progress * 100)}%`);
    },
  });
} catch (error) {
  console.error('Rendering failed:', error);
  // Fallback: try with lower quality settings
  await renderMedia({
    ...config,
    scale: 0.5, // Reduce resolution
    codec: 'h264',
    crf: 23, // Higher CRF = lower quality but more stable
  });
}
```

### Issue: Memory Issues with Large Content

**Problem:** Node.js runs out of memory when processing large batches.

**Solution:**
```bash
# Increase Node.js memory limit
NODE_OPTIONS=--max-old-space-size=4096 npm run dev
```

Or process in smaller chunks:

```typescript
async function processBatch<T>(
  items: T[],
  processor: (item: T) => Promise<void>,
  batchSize = 3
) {
  for (let i = 0; i < items.length; i += batchSize) {
    const batch = items.slice(i, i + batchSize);
    await Promise.all(batch.map(processor));
  }
}
```

### Issue: Missing Environment Variables

**Problem:** Pipeline fails due to missing API keys.

**Solution:**
```typescript
// src/lib/utils/validate-env.ts
export function validateEnvironment() {
  const required = [
    'ANTHROPIC_API_KEY',
    'OPENAI_API_KEY',
    'RAPIDAPI_KEY',
  ];

  const missing = required.filter(key => !process.env[key]);

  if (missing.length > 0) {
    throw new Error(
      `Missing required environment variables: ${missing.join(', ')}\n` +
      'Please check your .env.local file.'
    );
  }
}

// Call at app startup
validateEnvironment();
```

## Development Commands

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Run type checking
npm run type-check

# Lint code
npm run lint

# Render a single video (Remotion)
npm run remotion render ContentVideo output.mp4
```

This skill provides comprehensive guidance for using marketing-pipeline-share to automate content creation workflows from research through video generation, with real TypeScript examples and practical troubleshooting for common issues.
