---
name: marketing-pipeline-share-ai-content-automation
description: Automated AI content pipeline for research, scriptwriting, video generation, and multi-platform publishing using Claude, OpenAI, and Remotion
triggers:
  - "automate content creation pipeline with AI"
  - "research and generate marketing content automatically"
  - "create videos from text using Remotion"
  - "set up AI content automation system"
  - "build automated marketing content workflow"
  - "generate multilingual content with Claude and OpenAI"
  - "crawl news and create content automatically"
  - "automate content research and video generation"
---

# Marketing Pipeline Share AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to use the Ultimate AI Content Pipeline, a comprehensive TypeScript-based system that automates the entire content creation workflow: from researching trending topics, generating scripts in multiple formats and languages, to rendering videos and publishing to social platforms.

## What This Project Does

The Marketing Pipeline Share is an end-to-end content automation system that:

- **Auto-scans research sources**: Crawls TechCrunch, a16z, Twitter/X, LinkedIn for trending topics within the last 24 hours
- **Generates AI-powered content**: Creates articles in multiple formats (toplist, POV, case study, how-to) using Claude 3 and OpenAI
- **Multilingual support**: Produces content in both English and Vietnamese with customizable tone
- **Video automation**: Renders videos and infographics from text using Remotion
- **Platform optimization**: Exports content optimized for Reels, TikTok, Shorts, and other platforms

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

## Environment Configuration

Create a `.env.local` file in the project root:

```env
# AI Provider Keys
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research API Keys
RAPIDAPI_KEY=your_rapidapi_key

# Database (if using)
DATABASE_URL=your_database_url

# Remotion Configuration
REMOTION_LAMBDA_ROLE_ARN=your_aws_role_arn
REMOTION_LAMBDA_FUNCTION_NAME=your_function_name

# Application Settings
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

## Running the Application

```bash
# Development mode
npm run dev

# Production build
npm run build
npm run start

# Remotion video preview
npm run remotion
```

## Core Architecture

### Content Generation Pipeline

The system follows this workflow:

1. **Research Phase**: Crawl and analyze trending topics
2. **Content Generation**: AI creates scripts based on templates
3. **Video Rendering**: Remotion transforms text to video
4. **Publishing**: Auto-post to configured platforms

## Key APIs and Usage Patterns

### 1. Research Content from Sources

```typescript
import { researchTrends } from '@/lib/research';

async function getTrendingTopics(keyword: string) {
  const trends = await researchTrends({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeframe: '24h',
    language: 'en'
  });
  
  return trends.map(trend => ({
    title: trend.title,
    summary: trend.summary,
    url: trend.url,
    publishedAt: trend.publishedAt
  }));
}
```

### 2. Generate Content with AI

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

async function generateArticle(
  topic: string, 
  format: 'toplist' | 'pov' | 'case-study' | 'how-to',
  language: 'en' | 'vi'
) {
  const systemPrompt = `You are an expert content creator specializing in ${format} articles.
Create engaging, data-backed content in ${language === 'vi' ? 'Vietnamese' : 'English'}.`;

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    system: systemPrompt,
    messages: [
      {
        role: 'user',
        content: `Create a ${format} article about: ${topic}`
      }
    ]
  });

  return message.content[0].type === 'text' 
    ? message.content[0].text 
    : '';
}
```

### 3. Alternative: Using OpenAI

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
        content: 'You are a professional content marketer.'
      },
      {
        role: 'user',
        content: prompt
      }
    ],
    temperature: 0.7,
  });

  return completion.choices[0].message.content;
}
```

### 4. Video Rendering with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoContent {
  title: string;
  points: string[];
  duration: number;
}

async function renderContentVideo(content: VideoContent) {
  const bundleLocation = await bundle({
    entryPoint: path.join(process.cwd(), 'src/remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: content,
  });

  const outputLocation = path.join(
    process.cwd(), 
    'public/videos', 
    `${Date.now()}.mp4`
  );

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: content,
  });

  return outputLocation;
}
```

### 5. Complete Content Pipeline

```typescript
import { researchTrends } from '@/lib/research';
import { generateArticle } from '@/lib/ai/generate';
import { renderContentVideo } from '@/lib/video/render';
import { publishToSocial } from '@/lib/publish';

interface ContentPipelineOptions {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  languages: ('en' | 'vi')[];
  includeVideo: boolean;
  platforms: string[];
}

async function runContentPipeline(options: ContentPipelineOptions) {
  // Step 1: Research
  const trends = await researchTrends({
    keyword: options.keyword,
    sources: ['techcrunch', 'a16z', 'twitter'],
    timeframe: '24h',
    language: 'en'
  });

  const topTrend = trends[0];

  // Step 2: Generate content in multiple languages
  const articles = await Promise.all(
    options.languages.map(lang => 
      generateArticle(topTrend.title, options.format, lang)
    )
  );

  // Step 3: Create video if requested
  let videoPath: string | null = null;
  if (options.includeVideo) {
    const videoContent = {
      title: topTrend.title,
      points: articles[0].split('\n').filter(line => line.startsWith('-')),
      duration: 30
    };
    videoPath = await renderContentVideo(videoContent);
  }

  // Step 4: Publish to platforms
  const publishResults = await Promise.all(
    options.platforms.map(platform =>
      publishToSocial({
        platform,
        content: articles[0],
        media: videoPath ? [videoPath] : []
      })
    )
  );

  return {
    trend: topTrend,
    articles,
    video: videoPath,
    published: publishResults
  };
}
```

## Common Content Formats

### Toplist Article

```typescript
const toplistPrompt = `Create a toplist article with:
- Catchy headline
- 5-7 numbered items
- Each item has a title and 2-3 sentence explanation
- Data-backed insights
- Conclusion with actionable takeaway`;
```

### POV (Point of View) Article

```typescript
const povPrompt = `Create a POV article with:
- Strong personal perspective
- Industry experience backing
- Contrarian or unique angle
- Supporting evidence
- Call to action`;
```

### Case Study

```typescript
const caseStudyPrompt = `Create a case study with:
- Company/project background
- Problem statement
- Solution implemented
- Results with specific metrics
- Key learnings`;
```

## API Routes (Next.js)

### Generate Content Endpoint

```typescript
// app/api/content/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { generateArticle } from '@/lib/ai/generate';

export async function POST(request: NextRequest) {
  try {
    const { topic, format, language } = await request.json();
    
    const article = await generateArticle(topic, format, language);
    
    return NextResponse.json({ 
      success: true, 
      article 
    });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

### Research Endpoint

```typescript
// app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { researchTrends } from '@/lib/research';

export async function POST(request: NextRequest) {
  try {
    const { keyword, sources, timeframe } = await request.json();
    
    const trends = await researchTrends({
      keyword,
      sources,
      timeframe,
      language: 'en'
    });
    
    return NextResponse.json({ 
      success: true, 
      trends 
    });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

## Remotion Video Components

### Basic Content Video Component

```typescript
// src/remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig } from 'remotion';

interface ContentVideoProps {
  title: string;
  points: string[];
}

export const ContentVideo: React.FC<ContentVideoProps> = ({ title, points }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  const opacity = Math.min(1, frame / (fps * 0.5));

  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a1a' }}>
      <div style={{ 
        padding: 60, 
        opacity,
        color: 'white',
        fontFamily: 'Arial, sans-serif'
      }}>
        <h1 style={{ fontSize: 48, marginBottom: 40 }}>
          {title}
        </h1>
        <ul style={{ fontSize: 24, lineHeight: 1.8 }}>
          {points.map((point, i) => (
            <li key={i} style={{ marginBottom: 20 }}>
              {point}
            </li>
          ))}
        </ul>
      </div>
    </AbsoluteFill>
  );
};
```

## Configuration Patterns

### AI Provider Configuration

```typescript
// lib/ai/config.ts
export const AI_PROVIDERS = {
  claude: {
    model: 'claude-3-5-sonnet-20241022',
    maxTokens: 4096,
    temperature: 0.7,
  },
  openai: {
    model: 'gpt-4-turbo-preview',
    maxTokens: 4096,
    temperature: 0.7,
  },
} as const;

export function getAIProvider(provider: keyof typeof AI_PROVIDERS) {
  return AI_PROVIDERS[provider];
}
```

### Content Format Templates

```typescript
// lib/content/templates.ts
export const CONTENT_TEMPLATES = {
  toplist: {
    structure: 'numbered-list',
    minItems: 5,
    maxItems: 10,
    includeIntro: true,
    includeConclusion: true,
  },
  pov: {
    structure: 'opinion-piece',
    includePerspective: true,
    includeEvidence: true,
    tone: 'authoritative',
  },
  'case-study': {
    structure: 'problem-solution',
    sections: ['background', 'problem', 'solution', 'results', 'learnings'],
    includeMetrics: true,
  },
  'how-to': {
    structure: 'step-by-step',
    includeOverview: true,
    includePrerequisites: true,
    stepFormat: 'detailed',
  },
} as const;
```

## Troubleshooting

### API Rate Limits

```typescript
// lib/utils/rate-limit.ts
class RateLimiter {
  private timestamps: number[] = [];
  
  constructor(
    private maxRequests: number,
    private windowMs: number
  ) {}
  
  async checkLimit(): Promise<boolean> {
    const now = Date.now();
    this.timestamps = this.timestamps.filter(
      ts => now - ts < this.windowMs
    );
    
    if (this.timestamps.length >= this.maxRequests) {
      const oldestRequest = this.timestamps[0];
      const waitTime = this.windowMs - (now - oldestRequest);
      await new Promise(resolve => setTimeout(resolve, waitTime));
    }
    
    this.timestamps.push(now);
    return true;
  }
}

export const anthropicLimiter = new RateLimiter(50, 60000); // 50 req/min
export const openaiLimiter = new RateLimiter(60, 60000); // 60 req/min
```

### Error Handling

```typescript
// lib/utils/error-handler.ts
export class ContentGenerationError extends Error {
  constructor(
    message: string,
    public provider: string,
    public originalError?: unknown
  ) {
    super(message);
    this.name = 'ContentGenerationError';
  }
}

export async function withRetry<T>(
  fn: () => Promise<T>,
  maxRetries = 3,
  delay = 1000
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      await new Promise(resolve => setTimeout(resolve, delay * (i + 1)));
    }
  }
  throw new Error('Max retries exceeded');
}
```

### Video Rendering Issues

```typescript
// Common fix for Remotion memory issues
export const VIDEO_CONFIG = {
  // Reduce concurrency for large videos
  concurrency: 2,
  
  // Enable frame caching
  frameRange: [0, 900], // 30 seconds at 30fps
  
  // Optimize codec settings
  codec: 'h264' as const,
  pixelFormat: 'yuv420p',
  
  // Quality settings
  crf: 18, // Lower = better quality
  videoBitrate: '5M',
};
```

## Best Practices

1. **Use environment variables** for all API keys and secrets
2. **Implement rate limiting** to avoid API quota exhaustion
3. **Cache research results** to reduce API calls
4. **Batch video rendering** to optimize resource usage
5. **Validate content** before publishing to platforms
6. **Monitor AI costs** across Claude and OpenAI usage
7. **Test video renders locally** before production deployment
