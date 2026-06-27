---
name: marketing-pipeline-automation
description: Automated AI content pipeline that researches, writes scripts, generates videos, and auto-posts using Claude/OpenAI and Remotion
triggers:
  - how do I automate content creation with AI research
  - set up an automated marketing content pipeline
  - generate videos from content automatically with Remotion
  - crawl and research news for AI content generation
  - create multilingual content with Claude and OpenAI
  - build an end-to-end content automation system
  - auto-post content to social media platforms
  - generate infographics and videos from text content
---

# Marketing Pipeline Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the Ultimate AI Content Pipeline - a comprehensive TypeScript-based system that automates the entire content creation workflow from research to video generation and publishing.

## What This Project Does

The Marketing Pipeline is an all-in-one content automation system that:

1. **Auto-Research**: Crawls news sources (TechCrunch, a16z, Twitter/X, LinkedIn) for fresh data
2. **AI Content Generation**: Creates articles in multiple formats (toplists, POV, case studies, how-tos) using Claude 3 and OpenAI
3. **Multilingual Support**: Generates content in both English and Vietnamese with customizable tones
4. **Video Rendering**: Automatically converts text content to videos and infographics using Remotion
5. **Auto-Publishing**: Posts content directly to social media platforms

## Installation

### Prerequisites

```bash
# Node.js 18+ required
node --version

# Install pnpm (recommended) or npm
npm install -g pnpm
```

### Setup

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
pnpm install
# or
npm install

# Copy environment variables
cp .env.example .env
```

### Environment Configuration

Edit `.env` file with your API keys:

```bash
# AI API Keys
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_anthropic_claude_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key
TWITTER_BEARER_TOKEN=your_twitter_bearer_token

# Remotion (Video Generation)
REMOTION_LICENSE_KEY=your_remotion_license

# Social Media Publishing
FACEBOOK_ACCESS_TOKEN=your_facebook_token
FACEBOOK_PAGE_ID=your_page_id

# Database
DATABASE_URL=postgresql://user:password@localhost:5432/marketing_pipeline
```

## Key Architecture

The pipeline follows this flow:

```
Keyword Input → Research Module → Content Generation → Video Rendering → Publishing
```

### Project Structure

```
/src
  /modules
    /research     # News crawling and data extraction
    /content      # AI content generation (Claude/OpenAI)
    /video        # Remotion video rendering
    /publisher    # Social media posting
  /lib
    /ai           # AI client wrappers
    /utils        # Helper functions
  /pages          # Next.js pages
    /api          # API routes
```

## Core Usage Patterns

### 1. Research & Data Crawling

```typescript
// src/modules/research/crawler.ts
import { researchTopic } from '@/modules/research';

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
    statistics: research.statistics
  };
}

// Example usage
const data = await gatherResearch('AI automation');
console.log(`Found ${data.articles.length} articles`);
```

### 2. AI Content Generation with Claude

```typescript
// src/modules/content/generator.ts
import Anthropic from '@anthropic-ai/sdk';

const client = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY
});

interface ContentConfig {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  researchData: any;
}

async function generateContent(config: ContentConfig) {
  const prompt = buildPrompt(config);
  
  const message = await client.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [
      {
        role: 'user',
        content: prompt
      }
    ]
  });

  return {
    title: extractTitle(message.content),
    body: extractBody(message.content),
    metadata: extractMetadata(message.content)
  };
}

function buildPrompt(config: ContentConfig): string {
  const formatInstructions = {
    'toplist': 'Create a numbered list article with compelling reasons',
    'pov': 'Write from a unique perspective with strong opinions',
    'case-study': 'Analyze real examples with data and outcomes',
    'how-to': 'Provide step-by-step actionable instructions'
  };

  return `
You are a ${config.tone} content writer specializing in ${config.keyword}.

Format: ${formatInstructions[config.format]}
Language: ${config.language}

Research Data:
${JSON.stringify(config.researchData, null, 2)}

Create a comprehensive article that:
1. Uses the latest research data provided
2. Follows the ${config.format} format strictly
3. Writes in ${config.language} language
4. Maintains a ${config.tone} tone
5. Includes specific statistics and examples from the research

Output as JSON:
{
  "title": "Article title",
  "subtitle": "Engaging subtitle",
  "body": "Full article content in markdown",
  "keyPoints": ["point1", "point2", "point3"],
  "callToAction": "CTA text"
}
`;
}
```

### 3. Multilingual Content Generation

```typescript
// src/modules/content/multilingual.ts
async function generateBilingualContent(keyword: string, researchData: any) {
  const [englishContent, vietnameseContent] = await Promise.all([
    generateContent({
      keyword,
      format: 'toplist',
      language: 'en',
      tone: 'expert',
      researchData
    }),
    generateContent({
      keyword,
      format: 'toplist',
      language: 'vi',
      tone: 'friendly',
      researchData
    })
  ]);

  return {
    en: englishContent,
    vi: vietnameseContent
  };
}
```

### 4. Video Generation with Remotion

```typescript
// src/modules/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  title: string;
  keyPoints: string[];
  duration: number;
  format: 'reel' | 'tiktok' | 'shorts'; // 9:16 aspect ratio
}

async function renderContentVideo(config: VideoConfig) {
  const compositionId = 'ContentVideo';
  
  // Bundle Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.join(process.cwd(), 'src/remotion/index.ts'),
    webpackOverride: (config) => config
  });

  // Get composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: {
      title: config.title,
      keyPoints: config.keyPoints,
      theme: 'modern'
    }
  });

  // Render video
  const outputPath = path.join(
    process.cwd(),
    'public/videos',
    `${Date.now()}.mp4`
  );

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: {
      title: config.title,
      keyPoints: config.keyPoints
    }
  });

  return outputPath;
}
```

### 5. Remotion Video Component

```typescript
// src/remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig, interpolate } from 'remotion';
import React from 'react';

interface Props {
  title: string;
  keyPoints: string[];
  theme: 'modern' | 'minimal';
}

export const ContentVideo: React.FC<Props> = ({ title, keyPoints, theme }) => {
  const frame = useCurrentFrame();
  const { fps, durationInFrames } = useVideoConfig();

  // Animate title entrance
  const titleOpacity = interpolate(
    frame,
    [0, 30],
    [0, 1],
    { extrapolateRight: 'clamp' }
  );

  return (
    <AbsoluteFill style={{ backgroundColor: '#000' }}>
      {/* Title Section */}
      <div
        style={{
          opacity: titleOpacity,
          color: '#fff',
          fontSize: 60,
          fontWeight: 'bold',
          padding: 100,
          textAlign: 'center'
        }}
      >
        {title}
      </div>

      {/* Key Points Animation */}
      {keyPoints.map((point, index) => {
        const pointStart = 60 + index * 90;
        const pointOpacity = interpolate(
          frame,
          [pointStart, pointStart + 20],
          [0, 1],
          { extrapolateRight: 'clamp', extrapolateLeft: 'clamp' }
        );

        return (
          <div
            key={index}
            style={{
              position: 'absolute',
              top: 300 + index * 120,
              left: 100,
              right: 100,
              opacity: pointOpacity,
              color: '#fff',
              fontSize: 40,
              padding: 20,
              backgroundColor: 'rgba(255,255,255,0.1)',
              borderRadius: 10
            }}
          >
            {index + 1}. {point}
          </div>
        );
      })}
    </AbsoluteFill>
  );
};
```

### 6. Auto-Publishing to Social Media

```typescript
// src/modules/publisher/facebook.ts
import axios from 'axios';

interface PublishConfig {
  pageId: string;
  accessToken: string;
  content: {
    message: string;
    videoPath?: string;
    link?: string;
  };
}

async function publishToFacebook(config: PublishConfig) {
  const { pageId, accessToken, content } = config;

  if (content.videoPath) {
    // Upload video
    const formData = new FormData();
    formData.append('file', fs.createReadStream(content.videoPath));
    formData.append('description', content.message);
    formData.append('access_token', accessToken);

    const response = await axios.post(
      `https://graph.facebook.com/v18.0/${pageId}/videos`,
      formData
    );

    return response.data;
  } else {
    // Post text/link
    const response = await axios.post(
      `https://graph.facebook.com/v18.0/${pageId}/feed`,
      {
        message: content.message,
        link: content.link,
        access_token: accessToken
      }
    );

    return response.data;
  }
}
```

### 7. Complete Pipeline Orchestration

```typescript
// src/modules/pipeline/orchestrator.ts
import { researchTopic } from '@/modules/research';
import { generateContent } from '@/modules/content/generator';
import { renderContentVideo } from '@/modules/video/renderer';
import { publishToFacebook } from '@/modules/publisher/facebook';

interface PipelineConfig {
  keyword: string;
  contentFormat: 'toplist' | 'pov' | 'case-study' | 'how-to';
  generateVideo: boolean;
  autoPublish: boolean;
}

async function runContentPipeline(config: PipelineConfig) {
  console.log(`Starting pipeline for: ${config.keyword}`);

  // Step 1: Research
  console.log('🔍 Researching...');
  const research = await researchTopic({
    keyword: config.keyword,
    sources: ['techcrunch', 'a16z', 'twitter'],
    timeRange: '24h',
    maxResults: 15
  });

  // Step 2: Generate Content
  console.log('✍️ Generating content...');
  const content = await generateContent({
    keyword: config.keyword,
    format: config.contentFormat,
    language: 'en',
    tone: 'expert',
    researchData: research
  });

  let videoPath: string | null = null;

  // Step 3: Generate Video (optional)
  if (config.generateVideo) {
    console.log('🎬 Rendering video...');
    videoPath = await renderContentVideo({
      title: content.title,
      keyPoints: content.keyPoints,
      duration: 30,
      format: 'reel'
    });
  }

  // Step 4: Publish (optional)
  if (config.autoPublish) {
    console.log('📤 Publishing...');
    await publishToFacebook({
      pageId: process.env.FACEBOOK_PAGE_ID!,
      accessToken: process.env.FACEBOOK_ACCESS_TOKEN!,
      content: {
        message: `${content.title}\n\n${content.subtitle}`,
        videoPath: videoPath || undefined
      }
    });
  }

  return {
    content,
    videoPath,
    published: config.autoPublish
  };
}

// Usage
const result = await runContentPipeline({
  keyword: 'AI Marketing Automation',
  contentFormat: 'toplist',
  generateVideo: true,
  autoPublish: false // Set to true for auto-posting
});
```

## API Routes (Next.js)

### Content Generation Endpoint

```typescript
// src/pages/api/generate.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { runContentPipeline } from '@/modules/pipeline/orchestrator';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  const { keyword, format, generateVideo, autoPublish } = req.body;

  if (!keyword) {
    return res.status(400).json({ error: 'Keyword is required' });
  }

  try {
    const result = await runContentPipeline({
      keyword,
      contentFormat: format || 'toplist',
      generateVideo: generateVideo ?? false,
      autoPublish: autoPublish ?? false
    });

    res.status(200).json(result);
  } catch (error) {
    console.error('Pipeline error:', error);
    res.status(500).json({ 
      error: 'Pipeline execution failed',
      details: error instanceof Error ? error.message : 'Unknown error'
    });
  }
}
```

## CLI Usage (if available)

```bash
# Run the development server
pnpm dev

# Build the project
pnpm build

# Start production server
pnpm start

# Render a video composition
pnpm remotion render src/remotion/index.ts ContentVideo output.mp4

# Preview Remotion composition
pnpm remotion preview src/remotion/index.ts
```

## Common Patterns

### Rate Limiting for AI APIs

```typescript
// src/lib/utils/rate-limiter.ts
class RateLimiter {
  private queue: Array<() => Promise<any>> = [];
  private processing = false;
  private delay: number;

  constructor(requestsPerMinute: number) {
    this.delay = 60000 / requestsPerMinute;
  }

  async add<T>(fn: () => Promise<T>): Promise<T> {
    return new Promise((resolve, reject) => {
      this.queue.push(async () => {
        try {
          const result = await fn();
          resolve(result);
        } catch (error) {
          reject(error);
        }
      });

      if (!this.processing) {
        this.process();
      }
    });
  }

  private async process() {
    this.processing = true;
    while (this.queue.length > 0) {
      const fn = this.queue.shift();
      if (fn) await fn();
      await new Promise(resolve => setTimeout(resolve, this.delay));
    }
    this.processing = false;
  }
}

// Usage
const claudeLimiter = new RateLimiter(50); // 50 requests per minute

async function generateWithRateLimit(config: ContentConfig) {
  return claudeLimiter.add(() => generateContent(config));
}
```

### Error Handling & Retry Logic

```typescript
// src/lib/utils/retry.ts
async function withRetry<T>(
  fn: () => Promise<T>,
  maxRetries = 3,
  delayMs = 1000
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      
      console.log(`Retry ${i + 1}/${maxRetries} after error:`, error);
      await new Promise(resolve => setTimeout(resolve, delayMs * (i + 1)));
    }
  }
  throw new Error('Max retries exceeded');
}

// Usage
const content = await withRetry(() => 
  generateContent(config)
);
```

## Troubleshooting

### API Key Issues

```typescript
// Validate environment variables on startup
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

validateEnv();
```

### Remotion Rendering Errors

```bash
# If video rendering fails, check:
# 1. Install required dependencies
pnpm add @remotion/lambda @remotion/renderer

# 2. Ensure Chrome/Chromium is available
# On Linux:
sudo apt-get install chromium-browser

# 3. Check memory limits
# Increase Node memory if needed:
NODE_OPTIONS="--max-old-space-size=4096" pnpm dev
```

### Database Connection Issues

```typescript
// src/lib/db/client.ts
import { PrismaClient } from '@prisma/client';

const globalForPrisma = global as unknown as { prisma: PrismaClient };

export const prisma =
  globalForPrisma.prisma ||
  new PrismaClient({
    log: process.env.NODE_ENV === 'development' ? ['query', 'error', 'warn'] : ['error'],
  });

if (process.env.NODE_ENV !== 'production') {
  globalForPrisma.prisma = prisma;
}

// Test connection
async function testConnection() {
  try {
    await prisma.$connect();
    console.log('✅ Database connected');
  } catch (error) {
    console.error('❌ Database connection failed:', error);
    process.exit(1);
  }
}
```

### Performance Optimization

```typescript
// Parallel processing for multiple content pieces
async function batchGenerateContent(keywords: string[]) {
  const research = await Promise.all(
    keywords.map(keyword => researchTopic({ keyword, sources: ['techcrunch'], timeRange: '24h', maxResults: 10 }))
  );

  const contents = await Promise.all(
    keywords.map((keyword, i) =>
      generateContent({
        keyword,
        format: 'toplist',
        language: 'en',
        tone: 'expert',
        researchData: research[i]
      })
    )
  );

  return contents;
}
```

This skill covers the complete workflow for automating content creation from research to publishing using AI and video generation technologies.
