---
name: ultimate-ai-content-pipeline
description: Automated content creation pipeline from research to video generation using AI (Claude/OpenAI) and Remotion
triggers:
  - "how do I automate content creation with AI"
  - "generate blog posts and videos automatically"
  - "set up AI content pipeline with research"
  - "create automated marketing content workflow"
  - "build AI-powered content generation system"
  - "scrape news and generate content with Claude"
  - "automate video generation from blog posts"
  - "use Remotion for content automation"
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

An end-to-end TypeScript-based content automation system that researches trending topics, generates multi-format articles in multiple languages, and produces videos automatically. Built with Next.js, Claude 3, OpenAI, and Remotion.

## What This Project Does

This pipeline automates the entire content creation workflow:

1. **Auto-Research**: Crawls recent news from TechCrunch, a16z, Twitter/X, LinkedIn (last 24h)
2. **AI Content Generation**: Creates articles in multiple formats (toplist, POV, case study, how-to) using Claude or OpenAI
3. **Multi-language Support**: Generates English and Vietnamese versions simultaneously
4. **Video Rendering**: Converts content to infographics and short videos using Remotion
5. **Platform Optimization**: Exports videos for Reels, TikTok, Shorts

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
cp .env.example .env.local
```

## Configuration

Create a `.env.local` file with the following variables:

```bash
# AI Provider Keys
ANTHROPIC_API_KEY=your_claude_key_here
OPENAI_API_KEY=your_openai_key_here

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key_here
TWITTER_BEARER_TOKEN=your_twitter_token_here

# Content Settings
DEFAULT_LANGUAGE=en
SECONDARY_LANGUAGE=vi

# Remotion Settings
REMOTION_CONCURRENCY=2
```

## Key Architecture

The project follows this structure:

```
/src
  /app              # Next.js app router pages
  /components       # React components
  /lib
    /ai             # AI provider integrations
    /research       # Content research modules
    /generators     # Content generators
    /video          # Remotion video generation
  /remotion         # Video templates
```

## Core Usage Patterns

### 1. Research Content from Sources

```typescript
// lib/research/crawler.ts
import { researchTopic } from '@/lib/research';

async function gatherInsights(keyword: string) {
  const research = await researchTopic({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter'],
    timeframe: '24h',
    maxResults: 10
  });

  return {
    articles: research.articles,
    insights: research.insights,
    trends: research.trends
  };
}
```

### 2. Generate Content with AI

```typescript
// lib/generators/content.ts
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

async function generateArticle(
  topic: string,
  format: 'toplist' | 'pov' | 'case-study' | 'how-to',
  language: 'en' | 'vi'
) {
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [
      {
        role: 'user',
        content: `Generate a ${format} article about ${topic} in ${language}.
        
Include:
- Engaging headline
- Introduction with hook
- Main content with data points
- Clear structure
- Call to action

Format: Return JSON with { title, content, metadata }`
      }
    ]
  });

  return JSON.parse(message.content[0].text);
}
```

### 3. Alternative: OpenAI Generation

```typescript
// lib/generators/openai-content.ts
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateWithGPT(prompt: string, format: string) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are an expert content creator specializing in ${format} articles.`
      },
      {
        role: 'user',
        content: prompt
      }
    ],
    response_format: { type: 'json_object' }
  });

  return JSON.parse(completion.choices[0].message.content);
}
```

### 4. Video Generation with Remotion

```typescript
// lib/video/render.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

async function generateVideo(content: ArticleContent) {
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./src/remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title: content.title,
      points: content.keyPoints,
      duration: 30, // seconds
    },
  });

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: `out/${content.slug}.mp4`,
    inputProps: composition.inputProps,
  });

  return `out/${content.slug}.mp4`;
}
```

### 5. Remotion Composition Example

```typescript
// src/remotion/compositions/ContentVideo.tsx
import { AbsoluteFill, Sequence, useCurrentFrame, useVideoConfig } from 'remotion';

interface ContentVideoProps {
  title: string;
  points: string[];
  duration: number;
}

export const ContentVideo: React.FC<ContentVideoProps> = ({ title, points }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  const opacity = Math.min(1, frame / 30);

  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a1a' }}>
      <Sequence from={0} durationInFrames={fps * 3}>
        <div style={{ 
          opacity, 
          fontSize: 60, 
          color: 'white',
          padding: 80 
        }}>
          {title}
        </div>
      </Sequence>

      {points.map((point, index) => (
        <Sequence
          key={index}
          from={fps * (3 + index * 2)}
          durationInFrames={fps * 2}
        >
          <div style={{
            fontSize: 40,
            color: 'white',
            padding: 80,
            opacity: Math.min(1, (frame - fps * (3 + index * 2)) / 20)
          }}>
            • {point}
          </div>
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};
```

### 6. API Route Example

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { researchTopic } from '@/lib/research';
import { generateArticle } from '@/lib/generators/content';
import { generateVideo } from '@/lib/video/render';

export async function POST(req: NextRequest) {
  const { keyword, format, languages } = await req.json();

  try {
    // Step 1: Research
    const research = await researchTopic({
      keyword,
      sources: ['techcrunch', 'twitter'],
      timeframe: '24h'
    });

    // Step 2: Generate content for each language
    const articles = await Promise.all(
      languages.map(lang => 
        generateArticle(keyword, format, lang)
      )
    );

    // Step 3: Generate video from primary article
    const videoPath = await generateVideo(articles[0]);

    return NextResponse.json({
      success: true,
      articles,
      videoPath,
      research: research.insights
    });

  } catch (error) {
    return NextResponse.json(
      { error: error.message },
      { status: 500 }
    );
  }
}
```

### 7. Full Pipeline Orchestration

```typescript
// lib/pipeline/orchestrator.ts
export class ContentPipeline {
  async run(config: PipelineConfig) {
    // 1. Research phase
    const research = await this.research(config.keyword);
    
    // 2. Content generation phase
    const content = await this.generateContent({
      research,
      format: config.format,
      languages: config.languages
    });
    
    // 3. Media generation phase
    const media = await this.generateMedia(content);
    
    // 4. Return complete package
    return {
      content,
      media,
      metadata: {
        generatedAt: new Date(),
        sources: research.sources,
        keywords: research.keywords
      }
    };
  }

  private async research(keyword: string) {
    return researchTopic({
      keyword,
      sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
      timeframe: '24h',
      maxResults: 20
    });
  }

  private async generateContent(params: ContentParams) {
    return Promise.all(
      params.languages.map(lang =>
        generateArticle(
          params.research.summary,
          params.format,
          lang
        )
      )
    );
  }

  private async generateMedia(content: Article[]) {
    return Promise.all([
      generateVideo(content[0]),
      generateInfographic(content[0])
    ]);
  }
}

// Usage
const pipeline = new ContentPipeline();
const result = await pipeline.run({
  keyword: 'AI automation',
  format: 'toplist',
  languages: ['en', 'vi']
});
```

## CLI Commands

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Render video (Remotion)
npm run render -- --props='{"title":"My Title"}'

# Type checking
npm run type-check

# Lint
npm run lint
```

## Common Patterns

### Multi-Provider AI Fallback

```typescript
async function generateWithFallback(prompt: string) {
  try {
    return await generateWithClaude(prompt);
  } catch (error) {
    console.warn('Claude failed, falling back to OpenAI', error);
    return await generateWithGPT(prompt);
  }
}
```

### Rate Limiting Research Requests

```typescript
import pLimit from 'p-limit';

const limit = pLimit(3); // Max 3 concurrent requests

async function batchResearch(keywords: string[]) {
  return Promise.all(
    keywords.map(keyword =>
      limit(() => researchTopic({ keyword }))
    )
  );
}
```

### Caching Research Results

```typescript
import { Redis } from '@upstash/redis';

const redis = new Redis({
  url: process.env.UPSTASH_REDIS_REST_URL,
  token: process.env.UPSTASH_REDIS_REST_TOKEN,
});

async function getCachedResearch(keyword: string) {
  const cached = await redis.get(`research:${keyword}`);
  if (cached) return cached;

  const fresh = await researchTopic({ keyword });
  await redis.set(`research:${keyword}`, fresh, { ex: 3600 }); // 1 hour
  return fresh;
}
```

## Troubleshooting

### AI Provider Rate Limits

**Problem**: Getting 429 errors from Claude or OpenAI

**Solution**: Implement exponential backoff:

```typescript
async function retryWithBackoff<T>(
  fn: () => Promise<T>,
  maxRetries = 3
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (error.status === 429 && i < maxRetries - 1) {
        await new Promise(resolve => 
          setTimeout(resolve, Math.pow(2, i) * 1000)
        );
        continue;
      }
      throw error;
    }
  }
}
```

### Remotion Memory Issues

**Problem**: Video rendering crashes with out-of-memory errors

**Solution**: Reduce concurrency and composition complexity:

```typescript
await renderMedia({
  // ... other options
  concurrency: 1, // Reduce from default
  timeoutInMilliseconds: 120000,
  chromiumOptions: {
    args: ['--disable-web-security', '--no-sandbox']
  }
});
```

### Research Crawling Fails

**Problem**: Cannot fetch data from news sources

**Solution**: Check API keys and implement fallback sources:

```typescript
const SOURCES_PRIORITY = ['techcrunch', 'a16z', 'twitter', 'linkedin'];

async function researchWithFallback(keyword: string) {
  for (const source of SOURCES_PRIORITY) {
    try {
      return await fetchFromSource(source, keyword);
    } catch (error) {
      console.warn(`${source} failed, trying next...`);
      continue;
    }
  }
  throw new Error('All research sources failed');
}
```

### TypeScript Configuration Issues

Ensure `tsconfig.json` includes:

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "lib": ["ES2022", "DOM"],
    "jsx": "preserve",
    "module": "ESNext",
    "moduleResolution": "bundler",
    "resolveJsonModule": true,
    "paths": {
      "@/*": ["./src/*"]
    }
  }
}
```

## Best Practices

1. **Always validate API keys** before running expensive operations
2. **Cache research results** to avoid redundant API calls
3. **Use streaming responses** for real-time content generation feedback
4. **Implement queue systems** for video rendering (use BullMQ or similar)
5. **Monitor token usage** to control costs with AI providers
6. **Store generated content** with metadata for analytics and reuse
