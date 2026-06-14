---
name: marketing-pipeline-share-automation
description: AI-powered content automation pipeline that researches, generates scripts, creates videos, and auto-posts to social media using Claude/OpenAI and Remotion
triggers:
  - "automate content creation from research to video"
  - "build an AI marketing content pipeline"
  - "generate social media posts and videos automatically"
  - "create automated content workflow with AI"
  - "research trends and generate marketing content"
  - "set up content automation with Claude and OpenAI"
  - "auto-generate videos from written content"
  - "build end-to-end content production system"
---

# Marketing Pipeline Share Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

An all-in-one AI content automation system that handles the complete content lifecycle: automated research, multi-format content generation, video rendering, and social media publishing. Built with TypeScript, Next.js, Claude 3, OpenAI, and Remotion.

## What It Does

This pipeline automates the entire content creation process:

1. **Auto-Research**: Crawls recent content from TechCrunch, a16z, Twitter/X, LinkedIn within 24h
2. **AI Content Generation**: Creates posts in multiple formats (Toplist, POV, Case Study, How-to) using Claude/OpenAI
3. **Multi-language Support**: Generates both English and Vietnamese content simultaneously
4. **Video Generation**: Automatically renders infographics and short-form videos using Remotion
5. **Platform Optimization**: Exports content optimized for Reels, TikTok, Shorts
6. **Auto-Publishing**: Schedules and posts content to social platforms

## Installation

### Prerequisites

```bash
# Node.js 18+ required
node --version

# pnpm recommended (or npm/yarn)
npm install -g pnpm
```

### Setup

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
pnpm install

# Copy environment template
cp .env.example .env.local
```

### Required Environment Variables

```bash
# .env.local
ANTHROPIC_API_KEY=your_claude_key_here
OPENAI_API_KEY=your_openai_key_here
RAPIDAPI_KEY=your_rapidapi_key_here

# Database (if using)
DATABASE_URL=your_database_connection_string

# Social Media APIs (optional)
FACEBOOK_ACCESS_TOKEN=your_facebook_token
TWITTER_API_KEY=your_twitter_key
LINKEDIN_ACCESS_TOKEN=your_linkedin_token

# Remotion (for video generation)
REMOTION_LICENSE_KEY=your_remotion_license
```

## Key Architecture

### Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app router
│   ├── components/       # React components
│   ├── lib/             
│   │   ├── ai/          # AI service integrations
│   │   ├── crawlers/    # Content research crawlers
│   │   ├── generators/  # Content generators
│   │   └── video/       # Remotion video rendering
│   ├── types/           # TypeScript definitions
│   └── utils/           # Helper functions
├── remotion/            # Video templates
└── public/              # Static assets
```

## Core Usage Patterns

### 1. Research & Data Collection

```typescript
// src/lib/crawlers/research.ts
import { fetchTechCrunchArticles } from './sources/techcrunch';
import { fetchTwitterTrends } from './sources/twitter';
import { fetchLinkedInPosts } from './sources/linkedin';

interface ResearchConfig {
  keyword: string;
  sources: string[];
  timeframe: '24h' | '7d' | '30d';
  language?: 'en' | 'vi' | 'both';
}

export async function conductResearch(config: ResearchConfig) {
  const results = await Promise.all([
    fetchTechCrunchArticles(config.keyword, config.timeframe),
    fetchTwitterTrends(config.keyword),
    fetchLinkedInPosts(config.keyword, config.timeframe)
  ]);

  return {
    articles: results[0],
    trends: results[1],
    posts: results[2],
    insights: await extractInsights(results)
  };
}

// Usage
const research = await conductResearch({
  keyword: 'AI automation',
  sources: ['techcrunch', 'twitter', 'linkedin'],
  timeframe: '24h',
  language: 'both'
});
```

### 2. AI Content Generation

```typescript
// src/lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY
});

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY
});

interface ContentConfig {
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  tone: 'expert' | 'friendly' | 'humorous';
  language: 'en' | 'vi';
  research: any;
}

export async function generateContent(config: ContentConfig) {
  const prompt = buildPrompt(config);
  
  // Use Claude for content generation
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [{
      role: 'user',
      content: prompt
    }]
  });

  const content = message.content[0].text;

  // Generate variations with OpenAI
  const variations = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [{
      role: 'system',
      content: 'Create 3 headline variations for this content'
    }, {
      role: 'user',
      content: content
    }]
  });

  return {
    mainContent: content,
    headlines: variations.choices.map(c => c.message.content),
    metadata: extractMetadata(content)
  };
}

function buildPrompt(config: ContentConfig): string {
  const formatInstructions = {
    'toplist': 'Create a numbered list format with clear benefits',
    'pov': 'Write from a personal perspective with strong opinions',
    'case-study': 'Structure as problem-solution-results',
    'how-to': 'Step-by-step tutorial format'
  };

  return `
    Create a ${config.format} post about the following research.
    Tone: ${config.tone}
    Language: ${config.language}
    Format: ${formatInstructions[config.format]}
    
    Research Data:
    ${JSON.stringify(config.research, null, 2)}
    
    Requirements:
    - Include data-backed insights
    - Use recent statistics from the research
    - Optimize for social media engagement
    - Add relevant hashtags
  `;
}
```

### 3. Video Generation with Remotion

```typescript
// src/lib/video/render.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import { webpackOverride } from './webpack-override';

interface VideoConfig {
  content: string;
  format: 'reels' | 'tiktok' | 'shorts';
  duration: number;
}

const ASPECT_RATIOS = {
  reels: { width: 1080, height: 1920 },
  tiktok: { width: 1080, height: 1920 },
  shorts: { width: 1080, height: 1920 }
};

export async function generateVideo(config: VideoConfig) {
  const { width, height } = ASPECT_RATIOS[config.format];
  
  // Bundle Remotion project
  const bundleLocation = await bundle({
    entryPoint: './remotion/index.ts',
    webpackOverride
  });

  // Select composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      content: config.content,
      format: config.format
    }
  });

  // Render video
  const outputPath = `./output/video-${Date.now()}.mp4`;
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: {
      content: config.content,
      format: config.format
    }
  });

  return {
    path: outputPath,
    width,
    height,
    duration: config.duration
  };
}
```

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig } from 'remotion';
import React from 'react';

interface ContentVideoProps {
  content: string;
  format: 'reels' | 'tiktok' | 'shorts';
}

export const ContentVideo: React.FC<ContentVideoProps> = ({ content, format }) => {
  const frame = useCurrentFrame();
  const { fps, durationInFrames } = useVideoConfig();

  const opacity = Math.min(1, frame / 30);
  const scale = 0.8 + (opacity * 0.2);

  return (
    <AbsoluteFill style={{ backgroundColor: '#000' }}>
      <div
        style={{
          flex: 1,
          display: 'flex',
          flexDirection: 'column',
          justifyContent: 'center',
          alignItems: 'center',
          padding: 60,
          opacity,
          transform: `scale(${scale})`
        }}
      >
        <h1 style={{ color: '#fff', fontSize: 72, textAlign: 'center' }}>
          {content.split('\n')[0]}
        </h1>
        <p style={{ color: '#ccc', fontSize: 36, marginTop: 40, textAlign: 'center' }}>
          {content.split('\n').slice(1, 3).join(' ')}
        </p>
      </div>
    </AbsoluteFill>
  );
};
```

### 4. Complete Pipeline Orchestration

```typescript
// src/lib/pipeline/orchestrator.ts
import { conductResearch } from '../crawlers/research';
import { generateContent } from '../ai/content-generator';
import { generateVideo } from '../video/render';
import { publishToSocial } from '../publishers/social';

interface PipelineConfig {
  keyword: string;
  contentFormats: Array<'toplist' | 'pov' | 'case-study' | 'how-to'>;
  languages: Array<'en' | 'vi'>;
  includeVideo: boolean;
  autoPublish: boolean;
  platforms: string[];
}

export async function runPipeline(config: PipelineConfig) {
  console.log('🔍 Starting research phase...');
  const research = await conductResearch({
    keyword: config.keyword,
    sources: ['techcrunch', 'twitter', 'linkedin'],
    timeframe: '24h',
    language: config.languages.length > 1 ? 'both' : config.languages[0]
  });

  const results = [];

  for (const format of config.contentFormats) {
    for (const language of config.languages) {
      console.log(`✍️ Generating ${format} content in ${language}...`);
      
      const content = await generateContent({
        format,
        tone: 'expert',
        language,
        research
      });

      let videoPath = null;
      if (config.includeVideo) {
        console.log('🎬 Rendering video...');
        const video = await generateVideo({
          content: content.mainContent,
          format: 'reels',
          duration: 30
        });
        videoPath = video.path;
      }

      const result = {
        format,
        language,
        content: content.mainContent,
        headlines: content.headlines,
        videoPath
      };

      if (config.autoPublish) {
        console.log('📤 Publishing to social media...');
        await publishToSocial({
          content: result.content,
          platforms: config.platforms,
          mediaPath: videoPath
        });
      }

      results.push(result);
    }
  }

  console.log('✅ Pipeline completed!');
  return results;
}

// Usage example
const results = await runPipeline({
  keyword: 'AI automation trends 2024',
  contentFormats: ['toplist', 'how-to'],
  languages: ['en', 'vi'],
  includeVideo: true,
  autoPublish: false,
  platforms: ['facebook', 'linkedin']
});
```

### 5. Social Media Publishing

```typescript
// src/lib/publishers/social.ts
interface PublishConfig {
  content: string;
  platforms: string[];
  mediaPath?: string;
  scheduledTime?: Date;
}

export async function publishToSocial(config: PublishConfig) {
  const results = [];

  for (const platform of config.platforms) {
    try {
      let result;
      
      switch (platform) {
        case 'facebook':
          result = await publishToFacebook(config);
          break;
        case 'linkedin':
          result = await publishToLinkedIn(config);
          break;
        case 'twitter':
          result = await publishToTwitter(config);
          break;
      }

      results.push({
        platform,
        success: true,
        postId: result?.id
      });
    } catch (error) {
      results.push({
        platform,
        success: false,
        error: error.message
      });
    }
  }

  return results;
}

async function publishToFacebook(config: PublishConfig) {
  const response = await fetch(
    `https://graph.facebook.com/v18.0/me/feed`,
    {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        message: config.content,
        access_token: process.env.FACEBOOK_ACCESS_TOKEN
      })
    }
  );

  return response.json();
}
```

## Development Workflow

### Running the Dev Server

```bash
# Start Next.js development server
pnpm dev

# Access at http://localhost:3000
```

### Testing Individual Components

```bash
# Test research crawler
pnpm test:research

# Test content generation
pnpm test:generate

# Test video rendering
pnpm test:video
```

### Building for Production

```bash
# Build Next.js app
pnpm build

# Start production server
pnpm start
```

## Configuration Files

### Next.js Config

```typescript
// next.config.js
/** @type {import('next').NextConfig} */
const nextConfig = {
  reactStrictMode: true,
  env: {
    ANTHROPIC_API_KEY: process.env.ANTHROPIC_API_KEY,
    OPENAI_API_KEY: process.env.OPENAI_API_KEY
  },
  webpack: (config) => {
    config.resolve.fallback = {
      ...config.resolve.fallback,
      fs: false
    };
    return config;
  }
};

module.exports = nextConfig;
```

### TypeScript Config

```json
// tsconfig.json
{
  "compilerOptions": {
    "target": "ES2020",
    "lib": ["ES2020", "DOM"],
    "jsx": "preserve",
    "module": "ESNext",
    "moduleResolution": "bundler",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "incremental": true,
    "paths": {
      "@/*": ["./src/*"],
      "@/components/*": ["./src/components/*"],
      "@/lib/*": ["./src/lib/*"]
    }
  },
  "include": ["next-env.d.ts", "**/*.ts", "**/*.tsx"],
  "exclude": ["node_modules"]
}
```

## Common Patterns

### Error Handling

```typescript
// src/lib/utils/error-handler.ts
export class PipelineError extends Error {
  constructor(
    message: string,
    public phase: string,
    public originalError?: Error
  ) {
    super(message);
    this.name = 'PipelineError';
  }
}

export async function withErrorHandling<T>(
  phase: string,
  fn: () => Promise<T>
): Promise<T> {
  try {
    return await fn();
  } catch (error) {
    console.error(`Error in ${phase}:`, error);
    throw new PipelineError(
      `Failed during ${phase}`,
      phase,
      error as Error
    );
  }
}

// Usage
const content = await withErrorHandling('content-generation', () =>
  generateContent(config)
);
```

### Rate Limiting

```typescript
// src/lib/utils/rate-limiter.ts
export class RateLimiter {
  private queue: Array<() => Promise<any>> = [];
  private running = 0;

  constructor(
    private maxConcurrent: number,
    private delayMs: number
  ) {}

  async add<T>(fn: () => Promise<T>): Promise<T> {
    return new Promise((resolve, reject) => {
      this.queue.push(async () => {
        try {
          this.running++;
          const result = await fn();
          resolve(result);
        } catch (error) {
          reject(error);
        } finally {
          this.running--;
          await this.delay(this.delayMs);
          this.processQueue();
        }
      });

      this.processQueue();
    });
  }

  private processQueue() {
    if (this.running < this.maxConcurrent && this.queue.length > 0) {
      const fn = this.queue.shift();
      fn?.();
    }
  }

  private delay(ms: number) {
    return new Promise(resolve => setTimeout(resolve, ms));
  }
}

// Usage with API calls
const limiter = new RateLimiter(5, 1000); // 5 concurrent, 1s delay

const results = await Promise.all(
  keywords.map(keyword =>
    limiter.add(() => conductResearch({ keyword, sources: ['techcrunch'], timeframe: '24h' }))
  )
);
```

## Troubleshooting

### API Rate Limits

```typescript
// Handle Anthropic rate limits
if (error.status === 429) {
  console.log('Rate limited, waiting 60s...');
  await new Promise(resolve => setTimeout(resolve, 60000));
  return retryRequest();
}
```

### Video Rendering Memory Issues

```bash
# Increase Node.js memory limit
NODE_OPTIONS="--max-old-space-size=4096" pnpm dev
```

### Missing Environment Variables

```typescript
// src/lib/utils/validate-env.ts
const requiredEnvVars = [
  'ANTHROPIC_API_KEY',
  'OPENAI_API_KEY',
  'RAPIDAPI_KEY'
];

export function validateEnvironment() {
  const missing = requiredEnvVars.filter(
    key => !process.env[key]
  );

  if (missing.length > 0) {
    throw new Error(
      `Missing required environment variables: ${missing.join(', ')}`
    );
  }
}
```

### Crawler Blocked/403 Errors

```typescript
// Add user agent and headers
const headers = {
  'User-Agent': 'Mozilla/5.0 (compatible; ContentBot/1.0)',
  'Accept': 'application/json',
  'Accept-Language': 'en-US,en;q=0.9'
};

const response = await fetch(url, { headers });
```

## Best Practices

1. **Always validate research data** before feeding to AI models
2. **Use rate limiters** for all external API calls
3. **Cache research results** to avoid redundant crawling
4. **Test video templates** with different content lengths
5. **Implement retry logic** for network failures
6. **Monitor API costs** - Claude and OpenAI can get expensive
7. **Version your content templates** for A/B testing
8. **Log all pipeline runs** for debugging and analytics
