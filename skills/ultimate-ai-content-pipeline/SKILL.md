---
name: ultimate-ai-content-pipeline
description: Automated content creation pipeline that researches, generates scripts, creates videos, and publishes content using AI (Claude/OpenAI) and Remotion
triggers:
  - how do I automate content creation with AI
  - set up automated content pipeline
  - generate videos from text automatically
  - research and write content with Claude
  - create marketing content pipeline
  - automate social media content generation
  - build AI-powered content workflow
  - generate multilingual content with AI
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

Ultimate AI Content Pipeline is a comprehensive TypeScript-based system that automates the entire content creation workflow. It crawls news sources for research, generates multi-format content using Claude/OpenAI, and renders videos/graphics using Remotion. The pipeline supports Vietnamese and English content with customizable tone and style.

**Key capabilities:**
- Auto-research from TechCrunch, a16z, Twitter, LinkedIn (last 24h)
- Multi-format content generation (toplist, POV, case study, how-to)
- Bilingual support (English/Vietnamese)
- Automatic video rendering with Remotion
- Next.js frontend for easy management

## Installation

### Prerequisites

```bash
node >= 18.0.0
npm or yarn
```

### Setup

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

### Required Environment Variables

```bash
# AI Providers
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Remotion (for video rendering)
REMOTION_LICENSE_KEY=your_remotion_license

# Optional: Database
DATABASE_URL=postgresql://user:password@localhost:5432/content_db

# Next.js
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

### Start Development Server

```bash
npm run dev
# or
yarn dev
```

Visit `http://localhost:3000`

## Core Components

### 1. Research Module

Auto-crawls news sources and extracts insights:

```typescript
import { ResearchEngine } from './lib/research/engine';

const research = new ResearchEngine({
  apiKey: process.env.RAPIDAPI_KEY,
  sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
  timeRange: '24h'
});

// Fetch latest content on a topic
const insights = await research.scan({
  keyword: 'AI automation',
  language: 'en',
  maxResults: 10
});

console.log(insights);
// {
//   articles: [...],
//   trends: [...],
//   keyInsights: [...],
//   dataPoints: [...]
// }
```

### 2. Content Generation

Generate content in multiple formats using AI:

```typescript
import { ContentGenerator } from './lib/content/generator';

const generator = new ContentGenerator({
  provider: 'claude', // or 'openai'
  apiKey: process.env.ANTHROPIC_API_KEY,
  model: 'claude-3-opus-20240229'
});

// Generate article
const article = await generator.create({
  topic: 'AI in Marketing',
  format: 'toplist', // 'pov', 'case-study', 'how-to'
  language: 'vi', // or 'en'
  tone: 'professional', // 'friendly', 'humorous'
  researchData: insights,
  wordCount: 1500
});

console.log(article);
// {
//   title: "Top 10 AI Tools for Marketing in 2024",
//   content: "...",
//   metadata: { ... },
//   keywords: [...],
//   summary: "..."
// }
```

### 3. Bilingual Content

Generate content in both languages simultaneously:

```typescript
import { BilingualGenerator } from './lib/content/bilingual';

const bilingualGen = new BilingualGenerator({
  claudeKey: process.env.ANTHROPIC_API_KEY,
  openaiKey: process.env.OPENAI_API_KEY
});

const dualContent = await bilingualGen.generate({
  topic: 'Marketing Automation',
  format: 'pov',
  tone: 'expert',
  researchData: insights
});

console.log(dualContent);
// {
//   en: { title: "...", content: "..." },
//   vi: { title: "...", content: "..." },
//   sharedMetadata: { ... }
// }
```

### 4. Video Generation with Remotion

Render videos from generated content:

```typescript
import { VideoRenderer } from './lib/video/renderer';
import { bundle } from '@remotion/bundler';
import { renderMedia } from '@remotion/renderer';

const videoRenderer = new VideoRenderer({
  remotionLicense: process.env.REMOTION_LICENSE_KEY
});

// Prepare video from article
const videoData = await videoRenderer.prepareFromArticle({
  article: article,
  template: 'infographic', // 'slideshow', 'animated'
  aspectRatio: '9:16', // '16:9', '1:1'
  duration: 60 // seconds
});

// Render video
const bundleLocation = await bundle({
  entryPoint: './video/templates/Infographic.tsx',
  webpackOverride: (config) => config
});

await renderMedia({
  composition: videoData.composition,
  serveUrl: bundleLocation,
  codec: 'h264',
  outputLocation: `./output/${article.id}.mp4`,
  inputProps: videoData.props
});
```

### 5. Complete Pipeline Execution

Run the entire pipeline end-to-end:

```typescript
import { ContentPipeline } from './lib/pipeline';

const pipeline = new ContentPipeline({
  research: {
    apiKey: process.env.RAPIDAPI_KEY,
    sources: ['techcrunch', 'linkedin']
  },
  generation: {
    provider: 'claude',
    apiKey: process.env.ANTHROPIC_API_KEY
  },
  video: {
    enabled: true,
    remotionLicense: process.env.REMOTION_LICENSE_KEY
  }
});

// Execute full pipeline
const result = await pipeline.execute({
  keyword: 'AI Marketing Trends',
  outputFormats: ['article', 'video'],
  languages: ['en', 'vi'],
  publish: false // set true to auto-publish
});

console.log(result);
// {
//   research: { ... },
//   articles: { en: { ... }, vi: { ... } },
//   videos: [{ url: '...', format: '9:16' }],
//   status: 'completed'
// }
```

## API Routes (Next.js)

### Research Endpoint

```typescript
// pages/api/research.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { ResearchEngine } from '@/lib/research/engine';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  const { keyword, sources, timeRange } = req.body;

  const research = new ResearchEngine({
    apiKey: process.env.RAPIDAPI_KEY,
    sources: sources || ['techcrunch', 'a16z'],
    timeRange: timeRange || '24h'
  });

  try {
    const insights = await research.scan({ keyword });
    res.status(200).json(insights);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
}
```

### Content Generation Endpoint

```typescript
// pages/api/generate.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { ContentGenerator } from '@/lib/content/generator';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  const { topic, format, language, tone, researchData } = req.body;

  const generator = new ContentGenerator({
    provider: 'claude',
    apiKey: process.env.ANTHROPIC_API_KEY
  });

  try {
    const article = await generator.create({
      topic,
      format,
      language,
      tone,
      researchData
    });
    res.status(200).json(article);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
}
```

### Video Render Endpoint

```typescript
// pages/api/render-video.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { VideoRenderer } from '@/lib/video/renderer';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  const { article, template, aspectRatio } = req.body;

  const renderer = new VideoRenderer({
    remotionLicense: process.env.REMOTION_LICENSE_KEY
  });

  try {
    const videoUrl = await renderer.renderAndUpload({
      article,
      template,
      aspectRatio
    });
    res.status(200).json({ videoUrl });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
}
```

## Configuration

### Content Formats

Configure available content formats:

```typescript
// config/content-formats.ts
export const contentFormats = {
  toplist: {
    structure: ['intro', 'items', 'conclusion'],
    minItems: 5,
    maxItems: 15
  },
  pov: {
    structure: ['hook', 'perspective', 'evidence', 'conclusion'],
    toneRange: ['expert', 'thought-leader']
  },
  caseStudy: {
    structure: ['problem', 'solution', 'results', 'takeaways'],
    requiresData: true
  },
  howTo: {
    structure: ['intro', 'steps', 'tips', 'conclusion'],
    minSteps: 3,
    maxSteps: 10
  }
};
```

### Research Sources

Configure crawl sources:

```typescript
// config/research-sources.ts
export const researchSources = {
  techcrunch: {
    baseUrl: 'https://techcrunch.com',
    rssFeeds: ['/feed/'],
    categories: ['ai', 'marketing', 'startups']
  },
  a16z: {
    baseUrl: 'https://a16z.com',
    apiEndpoint: '/api/posts',
    filters: ['recent', 'trending']
  },
  twitter: {
    apiVersion: 'v2',
    searchParams: {
      maxResults: 100,
      tweetFields: ['created_at', 'public_metrics']
    }
  }
};
```

### Video Templates

Configure Remotion video templates:

```typescript
// video/config.ts
export const videoTemplates = {
  infographic: {
    composition: 'Infographic',
    durationPerSlide: 5,
    transitions: ['fade', 'slide'],
    bgMusic: true
  },
  slideshow: {
    composition: 'Slideshow',
    durationPerSlide: 3,
    transitions: ['wipe', 'zoom']
  },
  animated: {
    composition: 'Animated',
    fps: 30,
    complexAnimations: true
  }
};
```

## Common Patterns

### Pattern 1: Daily Content Automation

```typescript
import { ContentPipeline } from './lib/pipeline';
import cron from 'node-cron';

// Run daily at 6 AM
cron.schedule('0 6 * * *', async () => {
  const pipeline = new ContentPipeline({ /* config */ });
  
  const topics = [
    'AI Marketing Trends',
    'Social Media Strategy',
    'Content Automation'
  ];
  
  for (const topic of topics) {
    await pipeline.execute({
      keyword: topic,
      outputFormats: ['article', 'video'],
      languages: ['en', 'vi'],
      publish: true
    });
  }
});
```

### Pattern 2: Batch Content Generation

```typescript
import { BatchProcessor } from './lib/batch';

const batchProcessor = new BatchProcessor({
  concurrency: 3,
  retries: 2
});

const keywords = [
  'AI tools 2024',
  'Marketing automation',
  'Content strategy',
  'Video marketing'
];

const results = await batchProcessor.generateContent({
  keywords,
  format: 'toplist',
  languages: ['en', 'vi'],
  includeVideo: true
});

console.log(`Generated ${results.length} content pieces`);
```

### Pattern 3: Custom Content Workflow

```typescript
import { ResearchEngine } from './lib/research/engine';
import { ContentGenerator } from './lib/content/generator';
import { ContentOptimizer } from './lib/content/optimizer';

// Step 1: Research
const research = new ResearchEngine({ /* config */ });
const insights = await research.scan({ keyword: 'AI Marketing' });

// Step 2: Generate draft
const generator = new ContentGenerator({ /* config */ });
const draft = await generator.create({
  topic: 'AI Marketing',
  format: 'pov',
  researchData: insights
});

// Step 3: Optimize for SEO
const optimizer = new ContentOptimizer();
const optimized = await optimizer.enhance({
  content: draft,
  targetKeywords: ['AI marketing', 'automation', 'tools'],
  readabilityScore: 80
});

// Step 4: Save to database
await saveToDatabase(optimized);
```

### Pattern 4: Multi-Platform Video Export

```typescript
import { VideoRenderer } from './lib/video/renderer';

const renderer = new VideoRenderer({ /* config */ });

const platforms = [
  { name: 'reels', aspectRatio: '9:16', maxDuration: 60 },
  { name: 'youtube', aspectRatio: '16:9', maxDuration: 180 },
  { name: 'tiktok', aspectRatio: '9:16', maxDuration: 60 }
];

for (const platform of platforms) {
  const video = await renderer.renderAndUpload({
    article,
    template: 'infographic',
    aspectRatio: platform.aspectRatio,
    duration: platform.maxDuration,
    outputName: `${article.id}_${platform.name}.mp4`
  });
  
  console.log(`${platform.name}: ${video.url}`);
}
```

## CLI Commands

If the project includes CLI tools:

```bash
# Research command
npm run research -- --keyword "AI Marketing" --sources techcrunch,a16z

# Generate content
npm run generate -- --topic "Marketing Trends" --format toplist --lang vi

# Render video
npm run render -- --article-id 123 --template infographic --ratio 9:16

# Full pipeline
npm run pipeline -- --keyword "AI tools" --formats article,video --langs en,vi

# Batch processing
npm run batch -- --input keywords.txt --format pov
```

## Troubleshooting

### API Rate Limits

```typescript
import { RateLimiter } from './lib/utils/rate-limiter';

const limiter = new RateLimiter({
  maxRequests: 50,
  perMinutes: 1
});

await limiter.execute(async () => {
  return await generator.create({ /* params */ });
});
```

### Video Rendering Fails

```typescript
// Increase memory for large videos
// package.json
{
  "scripts": {
    "render": "node --max-old-space-size=4096 ./scripts/render.js"
  }
}

// Or use cloud rendering
import { renderMediaOnLambda } from '@remotion/lambda';

const result = await renderMediaOnLambda({
  region: 'us-east-1',
  functionName: 'remotion-render',
  composition: videoData.composition,
  serveUrl: bundleLocation,
  inputProps: videoData.props
});
```

### Content Quality Issues

```typescript
import { ContentValidator } from './lib/content/validator';

const validator = new ContentValidator({
  minWordCount: 800,
  readabilityThreshold: 70,
  requireSources: true
});

const validation = await validator.check(article);

if (!validation.passed) {
  // Regenerate with feedback
  const improved = await generator.create({
    ...originalParams,
    feedback: validation.issues
  });
}
```

### API Timeout Handling

```typescript
import axios from 'axios';

const apiClient = axios.create({
  timeout: 30000, // 30 seconds
  retry: 3,
  retryDelay: 1000
});

apiClient.interceptors.response.use(
  response => response,
  async error => {
    if (error.code === 'ECONNABORTED') {
      console.log('Request timeout, retrying...');
      return apiClient.request(error.config);
    }
    throw error;
  }
);
```

### Database Connection Issues

```typescript
// Use connection pooling
import { Pool } from 'pg';

const pool = new Pool({
  connectionString: process.env.DATABASE_URL,
  max: 20,
  idleTimeoutMillis: 30000,
  connectionTimeoutMillis: 2000
});

// Always use try-finally
const client = await pool.connect();
try {
  await client.query('INSERT INTO articles...');
} finally {
  client.release();
}
```

## Best Practices

1. **Always use environment variables** for API keys
2. **Implement rate limiting** to avoid API quota issues
3. **Cache research results** for 24 hours to reduce API calls
4. **Queue video rendering** for resource-intensive operations
5. **Validate generated content** before publishing
6. **Log all pipeline steps** for debugging
7. **Use TypeScript types** for better code safety
