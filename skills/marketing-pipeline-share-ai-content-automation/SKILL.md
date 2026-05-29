---
name: marketing-pipeline-share-ai-content-automation
description: Automated content pipeline from research to video generation using Claude/OpenAI and Remotion
triggers:
  - how do I set up the AI content pipeline automation
  - generate automated content with research and video
  - create content from keyword to published video
  - use marketing pipeline share for content automation
  - configure Claude and OpenAI for content generation
  - automate content research and video rendering
  - set up remotion video generation pipeline
  - build AI-powered content creation workflow
---

# Marketing Pipeline Share - AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

Ultimate AI Content Pipeline is an end-to-end content automation system that handles research, scriptwriting, and video generation. It crawls real-time data from sources like TechCrunch, a16z, Twitter, and LinkedIn, then uses Claude 3 or OpenAI to generate content in multiple formats (toplist, POV, case study, how-to), and finally renders videos using Remotion.

## What It Does

- **Auto-Research**: Crawls and analyzes real-time data from major news sources (24h window)
- **Multi-Format Content**: Generates content in various formats with customizable tone and language
- **Bilingual Support**: Automatically creates English and Vietnamese versions
- **Video Generation**: Converts written content into infographics and short-form videos using Remotion
- **Platform Optimization**: Exports videos in formats suitable for Reels, TikTok, Shorts

## Installation

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
yarn install

# Set up environment variables
cp .env.example .env
```

## Configuration

Create a `.env` file in the root directory with the following variables:

```bash
# AI API Keys
ANTHROPIC_API_KEY=your_anthropic_key_here
OPENAI_API_KEY=your_openai_key_here

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key_here

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Next.js Configuration
NEXT_PUBLIC_API_URL=http://localhost:3000

# Remotion Configuration
REMOTION_AWS_ACCESS_KEY_ID=your_aws_access_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret_key
```

## Key Components & Usage

### 1. Research Module - Auto Content Crawling

```typescript
import { ResearchService } from '@/services/research';

// Initialize research service
const researchService = new ResearchService({
  rapidApiKey: process.env.RAPIDAPI_KEY!,
});

// Crawl content by keyword
async function performResearch(keyword: string) {
  const results = await researchService.crawlSources({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeWindow: '24h',
    maxResults: 50
  });
  
  return results;
}

// Example usage
const insights = await performResearch('AI automation');
console.log(`Found ${insights.length} articles`);
```

### 2. Content Generation with Claude/OpenAI

```typescript
import { ContentGenerator } from '@/services/content-generator';
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

// Initialize AI clients
const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

// Generate content with Claude
async function generateWithClaude(
  research: any[], 
  format: 'toplist' | 'pov' | 'case-study' | 'how-to'
) {
  const prompt = `Based on this research data: ${JSON.stringify(research)}
  Create a ${format} article in both English and Vietnamese.
  Tone: Professional yet engaging.
  Include data-backed insights and recent trends.`;

  const message = await anthropic.messages.create({
    model: 'claude-3-opus-20240229',
    max_tokens: 4096,
    messages: [{
      role: 'user',
      content: prompt
    }]
  });

  return message.content;
}

// Generate content with OpenAI
async function generateWithOpenAI(research: any[], format: string) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [{
      role: 'system',
      content: 'You are an expert content creator specializing in marketing content.'
    }, {
      role: 'user',
      content: `Create a ${format} article based on: ${JSON.stringify(research)}`
    }],
    temperature: 0.7,
  });

  return completion.choices[0].message.content;
}
```

### 3. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import { Video } from '@/remotion/Video';

// Render video from content
async function renderContentVideo(
  content: string,
  outputPath: string,
  platform: 'reels' | 'tiktok' | 'shorts'
) {
  // Platform-specific dimensions
  const dimensions = {
    reels: { width: 1080, height: 1920 },
    tiktok: { width: 1080, height: 1920 },
    shorts: { width: 1080, height: 1920 }
  };

  const bundled = await bundle({
    entryPoint: './src/remotion/index.ts',
    webpackOverride: (config) => config,
  });

  const composition = await selectComposition({
    serveUrl: bundled,
    id: 'ContentVideo',
    inputProps: {
      content,
      ...dimensions[platform]
    },
  });

  await renderMedia({
    composition,
    serveUrl: bundled,
    codec: 'h264',
    outputLocation: outputPath,
  });

  return outputPath;
}

// Example usage
const videoPath = await renderContentVideo(
  generatedContent,
  'output/video.mp4',
  'reels'
);
```

### 4. Complete Pipeline Integration

```typescript
import { ContentPipeline } from '@/services/pipeline';

// Full automation pipeline
async function runContentPipeline(keyword: string) {
  const pipeline = new ContentPipeline({
    anthropicKey: process.env.ANTHROPIC_API_KEY!,
    openaiKey: process.env.OPENAI_API_KEY!,
    rapidApiKey: process.env.RAPIDAPI_KEY!,
  });

  // Step 1: Research
  const research = await pipeline.research(keyword);
  console.log('Research completed:', research.length, 'sources');

  // Step 2: Generate content
  const content = await pipeline.generateContent({
    research,
    format: 'toplist',
    languages: ['en', 'vi'],
    tone: 'professional',
  });

  // Step 3: Render video
  const videos = await pipeline.renderVideos({
    content,
    platforms: ['reels', 'tiktok', 'shorts'],
  });

  // Step 4: Save results
  await pipeline.save({
    keyword,
    research,
    content,
    videos,
  });

  return {
    content,
    videos,
  };
}

// Execute pipeline
const result = await runContentPipeline('AI marketing automation');
```

## API Routes (Next.js)

### Content Generation Endpoint

```typescript
// pages/api/generate-content.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { ContentPipeline } from '@/services/pipeline';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  const { keyword, format, languages, platforms } = req.body;

  try {
    const pipeline = new ContentPipeline({
      anthropicKey: process.env.ANTHROPIC_API_KEY!,
      openaiKey: process.env.OPENAI_API_KEY!,
      rapidApiKey: process.env.RAPIDAPI_KEY!,
    });

    const result = await pipeline.run({
      keyword,
      format,
      languages,
      platforms,
    });

    res.status(200).json(result);
  } catch (error) {
    console.error('Pipeline error:', error);
    res.status(500).json({ error: 'Content generation failed' });
  }
}
```

### Research Endpoint

```typescript
// pages/api/research.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { ResearchService } from '@/services/research';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  const { keyword, sources, timeWindow } = req.query;

  const research = new ResearchService({
    rapidApiKey: process.env.RAPIDAPI_KEY!,
  });

  const results = await research.crawlSources({
    keyword: keyword as string,
    sources: (sources as string).split(','),
    timeWindow: timeWindow as string || '24h',
  });

  res.status(200).json({ results });
}
```

## Common Patterns

### Batch Content Generation

```typescript
async function batchGenerateContent(keywords: string[]) {
  const pipeline = new ContentPipeline({
    anthropicKey: process.env.ANTHROPIC_API_KEY!,
    openaiKey: process.env.OPENAI_API_KEY!,
    rapidApiKey: process.env.RAPIDAPI_KEY!,
  });

  const results = await Promise.all(
    keywords.map(keyword => 
      pipeline.run({
        keyword,
        format: 'toplist',
        languages: ['en', 'vi'],
        platforms: ['reels'],
      })
    )
  );

  return results;
}
```

### Custom Tone and Voice

```typescript
const customContent = await pipeline.generateContent({
  research,
  format: 'pov',
  tone: {
    style: 'friendly',
    emoji: true,
    callToAction: 'Subscribe for more insights!',
  },
  languages: ['en'],
});
```

### Scheduled Content Production

```typescript
import cron from 'node-cron';

// Run pipeline daily at 9 AM
cron.schedule('0 9 * * *', async () => {
  const keywords = ['AI trends', 'marketing automation', 'content strategy'];
  
  for (const keyword of keywords) {
    await runContentPipeline(keyword);
  }
});
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
      if (error?.status === 429 && i < maxRetries - 1) {
        await new Promise(resolve => 
          setTimeout(resolve, Math.pow(2, i) * 1000)
        );
        continue;
      }
      throw error;
    }
  }
  throw new Error('Max retries exceeded');
}

const content = await withRetry(() => 
  generateWithClaude(research, 'toplist')
);
```

### Video Rendering Issues

```typescript
// Handle Remotion memory issues
const video = await renderContentVideo(content, outputPath, 'reels', {
  concurrency: 1, // Reduce concurrent rendering
  chromiumOptions: {
    headless: true,
    args: ['--no-sandbox', '--disable-setuid-sandbox'],
  },
});
```

### Environment Variable Validation

```typescript
function validateEnv() {
  const required = [
    'ANTHROPIC_API_KEY',
    'OPENAI_API_KEY',
    'RAPIDAPI_KEY',
  ];

  for (const key of required) {
    if (!process.env[key]) {
      throw new Error(`Missing required environment variable: ${key}`);
    }
  }
}

validateEnv();
```

## Running the Development Server

```bash
# Start Next.js development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Render a single video
npm run render -- --props='{"keyword": "AI marketing"}'
```

## Best Practices

1. **Cache Research Data**: Store crawled data to avoid redundant API calls
2. **Queue Video Rendering**: Use a job queue (Bull, BullMQ) for video generation
3. **Monitor API Usage**: Track API costs and implement usage limits
4. **Content Versioning**: Save content drafts before rendering videos
5. **Error Logging**: Implement comprehensive error tracking (Sentry, LogRocket)

This skill enables AI coding agents to effectively implement and extend the Marketing Pipeline Share system for automated content creation at scale.
