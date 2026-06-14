---
name: marketing-pipeline-share-automation
description: AI-powered content automation pipeline for research, scriptwriting, video generation, and multi-platform publishing using Claude/OpenAI and Remotion
triggers:
  - automate content creation pipeline
  - generate marketing content with AI
  - create videos from text automatically
  - build content automation workflow
  - set up AI content research pipeline
  - use remotion for video generation
  - automate social media content creation
  - create multilingual marketing content
---

# Marketing Pipeline Share Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the Ultimate AI Content Pipeline, a TypeScript-based system that automates the entire content creation workflow from research to video generation. The pipeline crawls news sources, generates AI-powered content in multiple formats and languages, and renders videos automatically using Remotion.

## What This Project Does

The Marketing Pipeline Share is an end-to-end content automation system that:

- **Auto-Research**: Crawls news from TechCrunch, a16z, Twitter/X, LinkedIn for trending topics
- **AI Content Generation**: Creates content using Claude 3 and OpenAI in multiple formats (toplists, POV articles, case studies, how-tos)
- **Multilingual Output**: Generates simultaneous English and Vietnamese content
- **Video Rendering**: Automatically creates infographics and short-form videos using Remotion
- **Multi-Platform Optimization**: Exports content formatted for Reels, TikTok, Shorts

## Installation

### Prerequisites

```bash
# Node.js 18+ required
node --version

# Package manager (npm, yarn, or pnpm)
npm --version
```

### Setup Steps

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

# Create environment file
cp .env.example .env
```

### Environment Configuration

Create a `.env` file with the following required keys:

```bash
# AI Provider API Keys
OPENAI_API_KEY=your_openai_key_here
ANTHROPIC_API_KEY=your_claude_key_here

# Research/Scraping APIs
RAPIDAPI_KEY=your_rapidapi_key_here

# Database (if applicable)
DATABASE_URL=postgresql://user:password@localhost:5432/content_pipeline

# Video Rendering
REMOTION_LICENSE_KEY=your_remotion_license_here

# Application Settings
NEXT_PUBLIC_APP_URL=http://localhost:3000
NODE_ENV=development
```

### Start Development Server

```bash
npm run dev
# or
yarn dev

# Access at http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── scraper/     # Web scraping modules
│   │   ├── content/     # Content generation logic
│   │   └── video/       # Remotion video rendering
│   ├── types/           # TypeScript type definitions
│   └── utils/           # Utility functions
├── remotion/            # Remotion video templates
├── public/              # Static assets
└── .env                 # Environment variables
```

## Core API Usage

### 1. Research & Scraping

```typescript
import { scrapeNews } from '@/lib/scraper/news-scraper';
import { analyzeTrends } from '@/lib/ai/trend-analyzer';

// Scrape latest news from configured sources
async function gatherResearch(keyword: string) {
  const sources = ['techcrunch', 'a16z', 'twitter', 'linkedin'];
  
  const newsData = await scrapeNews({
    keyword,
    sources,
    timeRange: '24h',
    limit: 50
  });

  // AI-powered trend analysis
  const insights = await analyzeTrends(newsData, {
    provider: 'claude', // or 'openai'
    model: 'claude-3-5-sonnet-20241022'
  });

  return {
    rawData: newsData,
    insights,
    topTopics: insights.trending,
    dataPoints: insights.statistics
  };
}
```

### 2. AI Content Generation

```typescript
import { generateContent } from '@/lib/ai/content-generator';

// Generate content in multiple formats
async function createContent(research: ResearchData) {
  const contentFormats = ['toplist', 'pov', 'case-study', 'how-to'];
  
  const content = await generateContent({
    research,
    format: 'toplist',
    languages: ['en', 'vi'],
    tone: 'professional', // 'friendly', 'humorous'
    provider: 'claude',
    model: 'claude-3-5-sonnet-20241022',
    temperature: 0.7
  });

  return content;
}

// Example with OpenAI
async function generateWithOpenAI(prompt: string) {
  const { Configuration, OpenAIApi } = await import('openai');
  
  const configuration = new Configuration({
    apiKey: process.env.OPENAI_API_KEY,
  });
  
  const openai = new OpenAIApi(configuration);
  
  const response = await openai.createChatCompletion({
    model: 'gpt-4-turbo-preview',
    messages: [
      { role: 'system', content: 'You are a professional content writer.' },
      { role: 'user', content: prompt }
    ],
    temperature: 0.7,
    max_tokens: 2000
  });

  return response.data.choices[0].message?.content;
}
```

### 3. Claude Integration

```typescript
import Anthropic from '@anthropic-ai/sdk';

async function generateWithClaude(prompt: string, context?: string) {
  const anthropic = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY,
  });

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [
      {
        role: 'user',
        content: context 
          ? `Context: ${context}\n\nTask: ${prompt}`
          : prompt
      }
    ],
    temperature: 0.7,
  });

  return message.content[0].type === 'text' 
    ? message.content[0].text 
    : '';
}

// Multi-language content generation
async function generateMultilingualContent(
  research: ResearchData,
  format: ContentFormat
) {
  const englishPrompt = `Create a ${format} article based on this research: ${JSON.stringify(research)}`;
  const vietnamesePrompt = `Tạo bài viết dạng ${format} bằng tiếng Việt dựa trên nghiên cứu: ${JSON.stringify(research)}`;

  const [enContent, viContent] = await Promise.all([
    generateWithClaude(englishPrompt),
    generateWithClaude(vietnamesePrompt)
  ]);

  return { en: enContent, vi: viContent };
}
```

### 4. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import { webpack } from '@remotion/webpack-config';
import path from 'path';

// Video rendering configuration
async function renderContentVideo(content: GeneratedContent) {
  const bundleLocation = await bundle({
    entryPoint: path.join(process.cwd(), 'remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo', // Your Remotion composition ID
    inputProps: {
      title: content.title,
      script: content.script,
      language: content.language,
      style: 'modern' // 'minimal', 'vibrant'
    },
  });

  const outputLocation = path.join(
    process.cwd(), 
    'public/videos',
    `${content.id}-${Date.now()}.mp4`
  );

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: composition.inputProps,
  });

  return outputLocation;
}

// Platform-specific video formats
async function renderForPlatforms(content: GeneratedContent) {
  const platforms = [
    { name: 'reels', width: 1080, height: 1920 },
    { name: 'tiktok', width: 1080, height: 1920 },
    { name: 'youtube-shorts', width: 1080, height: 1920 },
    { name: 'square', width: 1080, height: 1080 }
  ];

  const videos = await Promise.all(
    platforms.map(async (platform) => {
      const video = await renderMedia({
        composition: {
          ...composition,
          width: platform.width,
          height: platform.height,
        },
        serveUrl: bundleLocation,
        codec: 'h264',
        outputLocation: `public/videos/${content.id}-${platform.name}.mp4`,
      });
      
      return { platform: platform.name, path: video };
    })
  );

  return videos;
}
```

### 5. Complete Pipeline Workflow

```typescript
import { Pipeline } from '@/lib/pipeline';

// End-to-end content pipeline
async function runContentPipeline(keyword: string) {
  const pipeline = new Pipeline({
    aiProvider: 'claude',
    languages: ['en', 'vi'],
    formats: ['toplist', 'case-study'],
    renderVideo: true,
    autoPublish: false
  });

  try {
    // Step 1: Research
    console.log('🔍 Gathering research...');
    const research = await pipeline.research(keyword);

    // Step 2: Generate content
    console.log('✍️ Generating content...');
    const content = await pipeline.generate(research);

    // Step 3: Render videos
    console.log('🎬 Rendering videos...');
    const videos = await pipeline.renderVideos(content);

    // Step 4: Export results
    console.log('💾 Exporting results...');
    const output = await pipeline.export({
      content,
      videos,
      format: 'json', // or 'markdown', 'html'
      destination: './output'
    });

    return {
      success: true,
      content,
      videos,
      output
    };
  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}

// Usage
runContentPipeline('AI automation trends 2026')
  .then(result => console.log('✅ Pipeline completed:', result))
  .catch(err => console.error('❌ Pipeline failed:', err));
```

## Common Patterns

### Custom Content Templates

```typescript
import { ContentTemplate } from '@/lib/content/templates';

// Define custom content template
const customTemplate: ContentTemplate = {
  name: 'Product Launch',
  structure: {
    hook: 'Attention-grabbing opening',
    problem: 'Pain point identification',
    solution: 'Product introduction',
    features: 'Key benefits (3-5 points)',
    social_proof: 'Testimonials or data',
    cta: 'Call to action'
  },
  tone: 'persuasive',
  length: 'medium' // 500-800 words
};

async function generateFromTemplate(
  research: ResearchData,
  template: ContentTemplate
) {
  const prompt = `
    Create content following this template:
    ${JSON.stringify(template.structure, null, 2)}
    
    Research data: ${JSON.stringify(research)}
    Tone: ${template.tone}
    Length: ${template.length}
  `;

  return await generateWithClaude(prompt);
}
```

### Batch Content Generation

```typescript
async function batchGenerateContent(keywords: string[]) {
  const batchSize = 5; // Process 5 at a time to avoid rate limits
  const results = [];

  for (let i = 0; i < keywords.length; i += batchSize) {
    const batch = keywords.slice(i, i + batchSize);
    
    const batchResults = await Promise.all(
      batch.map(async (keyword) => {
        try {
          const research = await gatherResearch(keyword);
          const content = await createContent(research);
          
          return { keyword, success: true, content };
        } catch (error) {
          return { keyword, success: false, error: error.message };
        }
      })
    );

    results.push(...batchResults);
    
    // Rate limiting delay
    if (i + batchSize < keywords.length) {
      await new Promise(resolve => setTimeout(resolve, 2000));
    }
  }

  return results;
}
```

### Caching Research Data

```typescript
import { Redis } from 'ioredis';

const redis = new Redis(process.env.REDIS_URL);

async function getCachedResearch(keyword: string) {
  const cacheKey = `research:${keyword}:24h`;
  const cached = await redis.get(cacheKey);

  if (cached) {
    console.log('📦 Using cached research');
    return JSON.parse(cached);
  }

  console.log('🔍 Fetching fresh research');
  const research = await gatherResearch(keyword);
  
  // Cache for 24 hours
  await redis.setex(cacheKey, 86400, JSON.stringify(research));
  
  return research;
}
```

## Configuration

### AI Provider Settings

```typescript
// src/config/ai.config.ts
export const aiConfig = {
  claude: {
    model: 'claude-3-5-sonnet-20241022',
    maxTokens: 4096,
    temperature: 0.7,
    defaultLanguage: 'en'
  },
  openai: {
    model: 'gpt-4-turbo-preview',
    maxTokens: 2000,
    temperature: 0.7,
    topP: 1
  },
  fallback: 'claude' // Fallback if primary fails
};
```

### Scraper Configuration

```typescript
// src/config/scraper.config.ts
export const scraperConfig = {
  sources: {
    techcrunch: {
      enabled: true,
      endpoint: 'https://techcrunch.com/feed/',
      limit: 20
    },
    a16z: {
      enabled: true,
      endpoint: 'https://a16z.com/feed/',
      limit: 15
    },
    twitter: {
      enabled: true,
      apiKey: process.env.RAPIDAPI_KEY,
      limit: 30
    }
  },
  timeout: 10000,
  retries: 3
};
```

### Video Rendering Settings

```typescript
// remotion/config.ts
export const videoConfig = {
  fps: 30,
  durationInFrames: 300, // 10 seconds at 30fps
  compositions: {
    reels: { width: 1080, height: 1920 },
    square: { width: 1080, height: 1080 },
    landscape: { width: 1920, height: 1080 }
  },
  defaultStyle: 'modern',
  audioEnabled: false
};
```

## CLI Commands

While this is primarily a Next.js application, you can create CLI scripts:

```typescript
// scripts/generate.ts
import { Command } from 'commander';

const program = new Command();

program
  .name('content-pipeline')
  .description('AI Content Pipeline CLI')
  .version('1.0.0');

program
  .command('generate')
  .description('Generate content from keyword')
  .argument('<keyword>', 'Topic keyword')
  .option('-f, --format <type>', 'Content format', 'toplist')
  .option('-l, --lang <languages>', 'Languages (comma-separated)', 'en,vi')
  .option('-v, --video', 'Generate video', false)
  .action(async (keyword, options) => {
    const result = await runContentPipeline(keyword);
    console.log(result);
  });

program.parse();
```

Run with:

```bash
npx tsx scripts/generate.ts "AI trends" --format toplist --video
```

## Troubleshooting

### API Rate Limiting

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
      console.log(`⏳ Retry ${i + 1}/${maxRetries} after ${delay}ms`);
      await new Promise(resolve => setTimeout(resolve, delay));
    }
  }
  throw new Error('Max retries exceeded');
}
```

### Memory Issues with Large Videos

```typescript
// Use streaming for large video renders
import { renderFrames } from '@remotion/renderer';

async function renderLargeVideo(composition) {
  const { assetsInfo } = await renderFrames({
    composition,
    serveUrl: bundleLocation,
    onFrameUpdate: (frame) => {
      console.log(`Rendering frame ${frame}/${composition.durationInFrames}`);
    },
    imageFormat: 'jpeg',
    frameRange: [0, composition.durationInFrames],
  });

  return assetsInfo;
}
```

### Scraping Failures

```typescript
// Fallback to alternative sources
async function resilientScrape(keyword: string) {
  const sources = ['techcrunch', 'a16z', 'twitter'];
  const results = [];

  for (const source of sources) {
    try {
      const data = await scrapeNews({ keyword, sources: [source] });
      results.push(...data);
    } catch (error) {
      console.warn(`⚠️ Failed to scrape ${source}:`, error.message);
      continue; // Try next source
    }
  }

  if (results.length === 0) {
    throw new Error('All scraping sources failed');
  }

  return results;
}
```

### Environment Variable Validation

```typescript
// Validate required env vars on startup
function validateEnv() {
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

// Call in your app initialization
validateEnv();
```

This skill provides comprehensive guidance for AI coding agents to effectively use the Marketing Pipeline Share automation system for content creation, research automation, and video generation workflows.
