---
name: marketing-pipeline-share-automation
description: AI-powered content automation pipeline for research, scriptwriting, and video generation using Claude/OpenAI and Remotion
triggers:
  - automate content creation from research to video
  - generate marketing content with AI pipeline
  - create videos from blog posts automatically
  - set up content automation workflow
  - research and generate social media content
  - build AI content generation system
  - use marketing pipeline for content creation
  - automate video rendering from text content
---

# Marketing Pipeline Share - AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to help developers use **Marketing Pipeline Share**, a complete AI-powered content automation system that handles research, scriptwriting, content generation, and video rendering. The pipeline integrates Claude 3, OpenAI, and Remotion to transform keywords into multi-format content including blog posts and videos.

## What It Does

Marketing Pipeline Share automates the entire content creation workflow:

1. **Auto-Research**: Crawls and analyzes recent data from sources like TechCrunch, Twitter, LinkedIn
2. **AI Content Generation**: Creates content in multiple formats (toplist, POV, case study, how-to) using Claude or OpenAI
3. **Multi-language Support**: Generates Vietnamese and English versions simultaneously
4. **Video Rendering**: Automatically creates videos and infographics using Remotion
5. **Multi-platform Optimization**: Exports content optimized for Reels, TikTok, Shorts

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

Create a `.env.local` file in the root directory:

```bash
# AI Provider API Keys
OPENAI_API_KEY=your_openai_key_here
ANTHROPIC_API_KEY=your_claude_api_key_here

# RapidAPI for research/scraping
RAPIDAPI_KEY=your_rapidapi_key_here

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Next.js Configuration
NEXT_PUBLIC_API_URL=http://localhost:3000

# Remotion Configuration
REMOTION_AWS_ACCESS_KEY_ID=your_aws_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret
```

## Running the Application

```bash
# Development mode
npm run dev

# Production build
npm run build
npm run start

# Remotion video rendering
npm run remotion
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/             # Core libraries
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── scraper/     # Web scraping utilities
│   │   └── video/       # Video generation
│   ├── remotion/        # Remotion video compositions
│   └── utils/           # Utility functions
├── public/              # Static assets
└── .env.local          # Environment variables
```

## Core API Usage

### 1. AI Content Generation

```typescript
import { generateContent } from '@/lib/ai/content-generator';

interface ContentRequest {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi' | 'both';
  tone: 'expert' | 'friendly' | 'humorous';
  provider: 'openai' | 'claude';
}

// Generate content using Claude
const content = await generateContent({
  keyword: 'AI marketing automation',
  format: 'toplist',
  language: 'both',
  tone: 'expert',
  provider: 'claude'
});

console.log(content.english);
console.log(content.vietnamese);
```

### 2. Research & Data Scraping

```typescript
import { researchTopic } from '@/lib/scraper/research';

interface ResearchOptions {
  keyword: string;
  sources: string[];
  timeframe: '24h' | '7d' | '30d';
  maxResults: number;
}

// Scrape recent articles and insights
const research = await researchTopic({
  keyword: 'AI content creation',
  sources: ['techcrunch', 'twitter', 'linkedin'],
  timeframe: '24h',
  maxResults: 20
});

// Process research data
const insights = research.insights.map(insight => ({
  title: insight.title,
  summary: insight.summary,
  url: insight.url,
  publishedAt: insight.publishedAt
}));
```

### 3. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import { VideoComposition } from '@/remotion/compositions/ContentVideo';

interface VideoConfig {
  content: string;
  title: string;
  format: 'vertical' | 'square' | 'horizontal';
  duration: number;
}

async function generateVideo(config: VideoConfig) {
  // Bundle Remotion composition
  const bundled = await bundle({
    entryPoint: './src/remotion/index.ts',
    webpackOverride: (config) => config,
  });

  // Select composition
  const composition = await selectComposition({
    serveUrl: bundled,
    id: 'ContentVideo',
    inputProps: {
      title: config.title,
      content: config.content,
      format: config.format,
    },
  });

  // Render video
  await renderMedia({
    composition,
    serveUrl: bundled,
    codec: 'h264',
    outputLocation: `out/${config.title}.mp4`,
  });
}

// Generate vertical video for TikTok/Reels
await generateVideo({
  content: 'Your generated content here',
  title: 'AI Marketing Tips',
  format: 'vertical',
  duration: 30
});
```

## Common Patterns

### End-to-End Content Pipeline

```typescript
import { contentPipeline } from '@/lib/pipeline';

async function runFullPipeline(keyword: string) {
  // Step 1: Research
  const research = await contentPipeline.research({
    keyword,
    sources: ['techcrunch', 'twitter'],
    timeframe: '24h'
  });

  // Step 2: Generate content
  const content = await contentPipeline.generate({
    keyword,
    research: research.insights,
    format: 'toplist',
    language: 'both',
    provider: 'claude'
  });

  // Step 3: Create video
  const video = await contentPipeline.renderVideo({
    content: content.english,
    title: content.title,
    format: 'vertical'
  });

  return {
    content,
    video,
    research
  };
}

// Execute full pipeline
const result = await runFullPipeline('AI marketing tools 2026');
```

### Custom AI Prompt Configuration

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

async function generateWithCustomPrompt(
  keyword: string,
  customInstructions: string
) {
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [
      {
        role: 'user',
        content: `Create content about "${keyword}".
        
        Additional instructions: ${customInstructions}
        
        Format: Toplist
        Tone: Expert
        Include: Data-backed insights, recent trends, actionable tips`
      }
    ]
  });

  return message.content[0].text;
}
```

### Multi-format Content Export

```typescript
import { exportContent } from '@/lib/export';

interface ExportOptions {
  content: any;
  formats: Array<'markdown' | 'html' | 'json' | 'pdf'>;
  destination: string;
}

async function exportMultiFormat(options: ExportOptions) {
  const exports = await Promise.all(
    options.formats.map(async (format) => {
      const exported = await exportContent({
        content: options.content,
        format,
        destination: `${options.destination}/${options.content.title}.${format}`
      });
      return exported;
    })
  );

  return exports;
}

// Export to multiple formats
await exportMultiFormat({
  content: generatedContent,
  formats: ['markdown', 'html', 'pdf'],
  destination: './output'
});
```

## API Route Examples

### Content Generation Endpoint

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { generateContent } from '@/lib/ai/content-generator';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const { keyword, format, language, tone, provider } = body;

    // Validate input
    if (!keyword) {
      return NextResponse.json(
        { error: 'Keyword is required' },
        { status: 400 }
      );
    }

    // Generate content
    const content = await generateContent({
      keyword,
      format: format || 'toplist',
      language: language || 'both',
      tone: tone || 'expert',
      provider: provider || 'claude'
    });

    return NextResponse.json({
      success: true,
      data: content
    });
  } catch (error) {
    console.error('Content generation error:', error);
    return NextResponse.json(
      { error: 'Failed to generate content' },
      { status: 500 }
    );
  }
}
```

### Research Endpoint

```typescript
// src/app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { researchTopic } from '@/lib/scraper/research';

export async function POST(request: NextRequest) {
  try {
    const { keyword, sources, timeframe } = await request.json();

    const research = await researchTopic({
      keyword,
      sources: sources || ['techcrunch', 'twitter'],
      timeframe: timeframe || '24h',
      maxResults: 20
    });

    return NextResponse.json({
      success: true,
      data: research
    });
  } catch (error) {
    console.error('Research error:', error);
    return NextResponse.json(
      { error: 'Failed to research topic' },
      { status: 500 }
    );
  }
}
```

## Remotion Video Composition Example

```typescript
// src/remotion/compositions/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig } from 'remotion';

export const ContentVideo: React.FC<{
  title: string;
  content: string;
  format: 'vertical' | 'square' | 'horizontal';
}> = ({ title, content, format }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  const opacity = Math.min(1, frame / 30);

  const dimensions = {
    vertical: { width: 1080, height: 1920 },
    square: { width: 1080, height: 1080 },
    horizontal: { width: 1920, height: 1080 }
  };

  return (
    <AbsoluteFill
      style={{
        backgroundColor: '#1a1a1a',
        justifyContent: 'center',
        alignItems: 'center',
        ...dimensions[format]
      }}
    >
      <div style={{ opacity, padding: '40px', textAlign: 'center' }}>
        <h1 style={{ color: 'white', fontSize: '48px', marginBottom: '20px' }}>
          {title}
        </h1>
        <p style={{ color: '#ccc', fontSize: '24px', maxWidth: '80%' }}>
          {content}
        </p>
      </div>
    </AbsoluteFill>
  );
};
```

## Configuration Best Practices

### Rate Limiting for AI APIs

```typescript
import { Ratelimit } from '@upstash/ratelimit';
import { Redis } from '@upstash/redis';

const ratelimit = new Ratelimit({
  redis: Redis.fromEnv(),
  limiter: Ratelimit.slidingWindow(10, '1 m'), // 10 requests per minute
});

export async function rateLimitedGenerate(userId: string, options: any) {
  const { success } = await ratelimit.limit(userId);
  
  if (!success) {
    throw new Error('Rate limit exceeded');
  }

  return await generateContent(options);
}
```

### Error Handling Pattern

```typescript
import { logger } from '@/lib/logger';

export async function safeContentGeneration(options: any) {
  try {
    const content = await generateContent(options);
    return { success: true, data: content };
  } catch (error) {
    logger.error('Content generation failed', {
      error: error.message,
      options,
      timestamp: new Date().toISOString()
    });

    // Fallback to alternative provider
    if (options.provider === 'claude') {
      logger.info('Falling back to OpenAI');
      return await generateContent({ ...options, provider: 'openai' });
    }

    return { success: false, error: error.message };
  }
}
```

## Troubleshooting

### API Key Issues

```typescript
// Check if API keys are properly configured
function validateApiKeys() {
  const required = ['OPENAI_API_KEY', 'ANTHROPIC_API_KEY', 'RAPIDAPI_KEY'];
  const missing = required.filter(key => !process.env[key]);
  
  if (missing.length > 0) {
    console.error('Missing API keys:', missing.join(', '));
    throw new Error(`Please configure: ${missing.join(', ')}`);
  }
}

validateApiKeys();
```

### Video Rendering Timeout

```typescript
// Increase timeout for long video renders
await renderMedia({
  composition,
  serveUrl: bundled,
  codec: 'h264',
  outputLocation: 'out/video.mp4',
  timeoutInMilliseconds: 300000, // 5 minutes
  onProgress: ({ progress }) => {
    console.log(`Rendering: ${Math.round(progress * 100)}%`);
  }
});
```

### Content Quality Validation

```typescript
function validateGeneratedContent(content: any) {
  const minLength = 500;
  const requiredFields = ['title', 'body', 'summary'];
  
  // Check length
  if (content.body.length < minLength) {
    throw new Error('Content too short');
  }
  
  // Check required fields
  for (const field of requiredFields) {
    if (!content[field]) {
      throw new Error(`Missing field: ${field}`);
    }
  }
  
  return true;
}
```

## Performance Optimization

### Caching Research Results

```typescript
import { Redis } from '@upstash/redis';

const redis = Redis.fromEnv();

async function cachedResearch(keyword: string) {
  const cacheKey = `research:${keyword}`;
  
  // Check cache
  const cached = await redis.get(cacheKey);
  if (cached) {
    return JSON.parse(cached as string);
  }
  
  // Fetch fresh data
  const research = await researchTopic({ keyword });
  
  // Cache for 6 hours
  await redis.setex(cacheKey, 21600, JSON.stringify(research));
  
  return research;
}
```

This skill provides comprehensive guidance for using Marketing Pipeline Share to automate content creation workflows with AI-powered research, generation, and video rendering capabilities.
