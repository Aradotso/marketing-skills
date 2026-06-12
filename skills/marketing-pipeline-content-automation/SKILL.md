---
name: marketing-pipeline-content-automation
description: Ultimate AI content pipeline for automated research, script generation, video creation, and multi-platform content publishing
triggers:
  - automate content creation with AI research and video generation
  - build an AI-powered marketing content pipeline
  - create automated content workflow from research to video
  - set up AI content generation with Claude and OpenAI
  - generate marketing videos automatically from articles
  - automate social media content research and publishing
  - build content automation pipeline with Remotion
  - create AI-driven content factory for marketing
---

# Marketing Pipeline Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to help developers use the **Ultimate AI Content Pipeline** - a comprehensive TypeScript-based system that automates the entire content creation workflow from research and scriptwriting to video generation and publishing.

## What This Project Does

The Marketing Pipeline is an end-to-end content automation system that:

- **Auto-crawls and researches** trending news from sources like TechCrunch, a16z, Twitter/X, and LinkedIn
- **Generates AI-written content** in multiple formats (top lists, POV articles, case studies, how-to guides) using Claude 3 and OpenAI
- **Supports bilingual output** (English and Vietnamese) with customizable tone and voice
- **Auto-renders videos and infographics** using Remotion for social media platforms (Reels, TikTok, Shorts)
- **Provides a Next.js interface** for managing the entire pipeline with minimal clicks

This transforms a single keyword input into publication-ready content across multiple formats and platforms.

## Installation

### Prerequisites

```bash
# Required
Node.js 18+ or 20+
npm or yarn or pnpm
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
```

### Environment Configuration

Create a `.env.local` file in the project root:

```bash
# AI Model APIs
ANTHROPIC_API_KEY=your_claude_api_key_here
OPENAI_API_KEY=your_openai_api_key_here

# Research/Crawling APIs
RAPIDAPI_KEY=your_rapidapi_key_here

# Optional: Custom endpoints
NEXT_PUBLIC_API_URL=http://localhost:3000

# Remotion rendering
REMOTION_LICENSE_KEY=your_remotion_license_key_here
```

### Running the Application

```bash
# Development mode
npm run dev

# Production build
npm run build
npm run start

# Remotion video preview (if applicable)
npm run remotion:preview
```

The application will be available at `http://localhost:3000`.

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app router pages
│   ├── components/       # React components
│   ├── lib/             # Core business logic
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── crawler/     # Research and data crawling
│   │   ├── content/     # Content generation logic
│   │   └── video/       # Remotion video generation
│   ├── services/        # External API services
│   └── types/           # TypeScript type definitions
├── public/              # Static assets
├── remotion/            # Remotion video templates
└── .env.local          # Environment variables
```

## Core API Usage

### 1. Content Research & Crawling

```typescript
import { researchTopic } from '@/lib/crawler/research';

async function gatherResearch(keyword: string) {
  const research = await researchTopic({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeRange: '24h',
    maxResults: 20
  });

  return {
    articles: research.articles,
    insights: research.insights,
    trends: research.trends,
    dataSources: research.sources
  };
}

// Usage
const data = await gatherResearch('AI marketing automation');
console.log(`Found ${data.articles.length} relevant articles`);
```

### 2. AI Content Generation with Claude

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

async function generateContent(
  research: any,
  format: 'toplist' | 'pov' | 'case-study' | 'how-to',
  language: 'en' | 'vi'
) {
  const prompt = `Based on the following research data, create a ${format} article in ${language}.

Research Data:
${JSON.stringify(research, null, 2)}

Requirements:
- Tone: Professional yet engaging
- Include data-backed insights
- Format for web publishing
- Length: 800-1200 words`;

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4000,
    messages: [{
      role: 'user',
      content: prompt
    }]
  });

  return message.content[0].text;
}
```

### 3. AI Content Generation with OpenAI

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateWithGPT(
  topic: string,
  format: string,
  tone: 'expert' | 'friendly' | 'humorous'
) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are a professional content writer specializing in ${format} format with a ${tone} tone.`
      },
      {
        role: 'user',
        content: `Create an article about: ${topic}`
      }
    ],
    temperature: 0.7,
    max_tokens: 2000
  });

  return completion.choices[0].message.content;
}
```

### 4. Bilingual Content Pipeline

```typescript
import { generateContent } from '@/lib/content/generator';
import { translateContent } from '@/lib/content/translator';

async function createBilingualContent(keyword: string) {
  // Step 1: Research
  const research = await researchTopic({ keyword });
  
  // Step 2: Generate English version
  const englishContent = await generateContent(research, 'toplist', 'en');
  
  // Step 3: Generate or translate to Vietnamese
  const vietnameseContent = await generateContent(research, 'toplist', 'vi');
  
  return {
    en: {
      title: englishContent.title,
      body: englishContent.body,
      metadata: englishContent.metadata
    },
    vi: {
      title: vietnameseContent.title,
      body: vietnameseContent.body,
      metadata: vietnameseContent.metadata
    }
  };
}
```

### 5. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

async function generateVideo(content: any, platform: 'reels' | 'tiktok' | 'shorts') {
  const dimensions = {
    reels: { width: 1080, height: 1920 },
    tiktok: { width: 1080, height: 1920 },
    shorts: { width: 1080, height: 1920 }
  };

  const { width, height } = dimensions[platform];

  // Bundle Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  // Get composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title: content.title,
      points: content.keyPoints,
      branding: content.branding
    },
  });

  // Render video
  const outputLocation = `./output/${platform}-${Date.now()}.mp4`;
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: composition.defaultProps,
  });

  return outputLocation;
}
```

### 6. Complete Pipeline Integration

```typescript
import { researchTopic } from '@/lib/crawler/research';
import { generateContent } from '@/lib/content/generator';
import { generateVideo } from '@/lib/video/renderer';
import { publishContent } from '@/lib/publishing/publisher';

async function runFullPipeline(keyword: string) {
  try {
    console.log('🔍 Step 1: Researching topic...');
    const research = await researchTopic({
      keyword,
      sources: ['techcrunch', 'a16z', 'twitter'],
      timeRange: '24h'
    });

    console.log('✍️ Step 2: Generating content...');
    const content = await generateContent(research, 'toplist', 'en');

    console.log('🎬 Step 3: Creating video...');
    const videoPath = await generateVideo(content, 'reels');

    console.log('📤 Step 4: Publishing...');
    const published = await publishContent({
      article: content,
      video: videoPath,
      platforms: ['facebook', 'instagram', 'tiktok']
    });

    return {
      success: true,
      content,
      video: videoPath,
      published
    };
  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}

// Execute pipeline
runFullPipeline('AI content automation trends 2024')
  .then(result => console.log('✅ Pipeline completed:', result))
  .catch(err => console.error('❌ Pipeline failed:', err));
```

## Configuration Patterns

### Custom AI Model Settings

```typescript
// lib/config/ai.ts
export const aiConfig = {
  claude: {
    model: 'claude-3-5-sonnet-20241022',
    maxTokens: 4000,
    temperature: 0.7,
    topP: 0.9
  },
  openai: {
    model: 'gpt-4-turbo-preview',
    maxTokens: 2000,
    temperature: 0.7,
    frequencyPenalty: 0.3,
    presencePenalty: 0.3
  }
};

// Usage in generation
const message = await anthropic.messages.create({
  model: aiConfig.claude.model,
  max_tokens: aiConfig.claude.maxTokens,
  temperature: aiConfig.claude.temperature,
  messages: [/* ... */]
});
```

### Content Format Templates

```typescript
// lib/templates/formats.ts
export const contentFormats = {
  toplist: {
    structure: ['intro', 'items', 'conclusion'],
    itemCount: 10,
    includeNumbers: true,
    includeImages: true
  },
  pov: {
    structure: ['hook', 'context', 'perspective', 'evidence', 'conclusion'],
    tone: 'opinionated',
    includeCounterArguments: true
  },
  caseStudy: {
    structure: ['background', 'challenge', 'solution', 'results', 'takeaways'],
    includeMetrics: true,
    includeQuotes: true
  },
  howTo: {
    structure: ['intro', 'prerequisites', 'steps', 'tips', 'conclusion'],
    stepByStep: true,
    includeVisuals: true
  }
};
```

### Multi-Platform Video Specs

```typescript
// lib/config/video.ts
export const videoSpecs = {
  reels: {
    width: 1080,
    height: 1920,
    fps: 30,
    duration: 60, // seconds
    codec: 'h264',
    bitrate: '5M'
  },
  tiktok: {
    width: 1080,
    height: 1920,
    fps: 30,
    duration: 60,
    codec: 'h264',
    bitrate: '5M'
  },
  shorts: {
    width: 1080,
    height: 1920,
    fps: 30,
    duration: 60,
    codec: 'h264',
    bitrate: '5M'
  },
  landscape: {
    width: 1920,
    height: 1080,
    fps: 30,
    duration: 300,
    codec: 'h264',
    bitrate: '8M'
  }
};
```

## Common Patterns

### Error Handling and Retry Logic

```typescript
async function resilientAPICall<T>(
  fn: () => Promise<T>,
  maxRetries: number = 3,
  delay: number = 1000
): Promise<T> {
  let lastError: Error;
  
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      lastError = error as Error;
      console.warn(`Attempt ${i + 1} failed:`, error.message);
      
      if (i < maxRetries - 1) {
        await new Promise(resolve => setTimeout(resolve, delay * (i + 1)));
      }
    }
  }
  
  throw lastError!;
}

// Usage
const content = await resilientAPICall(() => 
  generateContent(research, 'toplist', 'en')
);
```

### Caching Research Data

```typescript
import NodeCache from 'node-cache';

const cache = new NodeCache({ stdTTL: 3600 }); // 1 hour

async function getCachedResearch(keyword: string) {
  const cacheKey = `research:${keyword}`;
  
  const cached = cache.get(cacheKey);
  if (cached) {
    console.log('📦 Using cached research data');
    return cached;
  }
  
  console.log('🔍 Fetching fresh research data');
  const research = await researchTopic({ keyword });
  cache.set(cacheKey, research);
  
  return research;
}
```

### Batch Content Generation

```typescript
async function generateBatchContent(keywords: string[]) {
  const results = await Promise.allSettled(
    keywords.map(async (keyword) => {
      const research = await researchTopic({ keyword });
      const content = await generateContent(research, 'toplist', 'en');
      return { keyword, content };
    })
  );

  const successful = results
    .filter(r => r.status === 'fulfilled')
    .map(r => (r as PromiseFulfilledResult<any>).value);

  const failed = results
    .filter(r => r.status === 'rejected')
    .map((r, i) => ({ keyword: keywords[i], error: (r as PromiseRejectedResult).reason }));

  return { successful, failed };
}
```

## Troubleshooting

### API Key Issues

```typescript
// Validate API keys on startup
function validateEnvironment() {
  const required = [
    'ANTHROPIC_API_KEY',
    'OPENAI_API_KEY',
    'RAPIDAPI_KEY'
  ];
  
  const missing = required.filter(key => !process.env[key]);
  
  if (missing.length > 0) {
    throw new Error(`Missing required environment variables: ${missing.join(', ')}`);
  }
  
  console.log('✅ All API keys configured');
}

validateEnvironment();
```

### Rate Limiting

```typescript
import pLimit from 'p-limit';

// Limit concurrent API calls
const limit = pLimit(3);

async function generateMultipleWithRateLimit(topics: string[]) {
  const promises = topics.map(topic =>
    limit(() => generateContent({ keyword: topic }, 'toplist', 'en'))
  );
  
  return Promise.all(promises);
}
```

### Video Rendering Issues

```bash
# Common Remotion issues

# Install system dependencies (Linux)
sudo apt-get install -y ffmpeg chromium-browser

# macOS
brew install ffmpeg

# Check Remotion configuration
npx remotion versions

# Clear Remotion cache
rm -rf node_modules/.remotion
```

### Memory Management for Large Operations

```typescript
// Process in chunks to avoid memory issues
async function processLargeDataset(items: any[], chunkSize: number = 10) {
  const chunks = [];
  
  for (let i = 0; i < items.length; i += chunkSize) {
    chunks.push(items.slice(i, i + chunkSize));
  }
  
  const results = [];
  
  for (const chunk of chunks) {
    const chunkResults = await Promise.all(
      chunk.map(item => generateContent(item, 'toplist', 'en'))
    );
    results.push(...chunkResults);
    
    // Force garbage collection between chunks if available
    if (global.gc) {
      global.gc();
    }
  }
  
  return results;
}
```

### Debug Mode

```typescript
// Enable verbose logging
const DEBUG = process.env.DEBUG === 'true';

function debugLog(...args: any[]) {
  if (DEBUG) {
    console.log('[DEBUG]', new Date().toISOString(), ...args);
  }
}

// Usage
debugLog('Research completed:', research.articles.length, 'articles found');
```

## Best Practices

1. **Always validate API responses** before processing
2. **Use environment variables** for all sensitive configuration
3. **Implement caching** for expensive research operations
4. **Monitor API quota usage** to avoid unexpected billing
5. **Test video rendering locally** before deploying to production
6. **Use TypeScript strict mode** to catch type errors early
7. **Implement proper error boundaries** in React components
8. **Version control your content templates** for reproducibility

This skill enables AI agents to effectively assist developers in building, configuring, and extending automated content marketing pipelines.
