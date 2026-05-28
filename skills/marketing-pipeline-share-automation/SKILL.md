---
name: marketing-pipeline-share-automation
description: Automate content creation from research to video generation using AI-powered pipeline with Claude, OpenAI, and Remotion
triggers:
  - automate content creation pipeline
  - generate AI-powered marketing content
  - research and create video content automatically
  - build automated content workflow
  - create content from keyword to video
  - setup AI content automation system
  - use marketing pipeline for content generation
  - automate research to video pipeline
---

# Marketing Pipeline Share - AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to help developers build and use the Ultimate AI Content Pipeline - an end-to-end automated content creation system that handles research, scriptwriting, and video generation using Claude 3, OpenAI, and Remotion.

## What It Does

Marketing Pipeline Share is a complete content automation pipeline that:

- **Auto-researches** trending topics from sources like TechCrunch, a16z, Twitter/X, LinkedIn
- **Generates content** in multiple formats (toplists, POV, case studies, how-tos)
- **Creates bilingual content** (English & Vietnamese) with customizable tone
- **Renders videos** automatically using Remotion for social media platforms
- **Optimizes for multi-platform** delivery (Reels, TikTok, Shorts)

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

# Setup environment variables
cp .env.example .env.local
```

## Environment Configuration

Create a `.env.local` file with the required API keys:

```env
# AI Service APIs
ANTHROPIC_API_KEY=your_claude_api_key_here
OPENAI_API_KEY=your_openai_api_key_here

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key_here

# Optional: Custom endpoints
NEXT_PUBLIC_API_URL=http://localhost:3000/api

# Video rendering
REMOTION_LICENSE_KEY=your_remotion_license_here
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/             # Core utilities
│   │   ├── ai/          # AI service integrations
│   │   ├── research/    # Content research modules
│   │   └── video/       # Remotion video generation
│   ├── types/           # TypeScript definitions
│   └── config/          # Configuration files
├── public/              # Static assets
└── remotion/           # Remotion video templates
```

## Core API Usage

### 1. Research & Content Scraping

```typescript
import { researchTopic } from '@/lib/research/scraper';

// Automatically research a topic from multiple sources
const research = await researchTopic({
  keyword: 'AI automation trends',
  sources: ['techcrunch', 'twitter', 'linkedin'],
  timeframe: '24h',
  language: 'en'
});

console.log(research.insights);
// Returns: { articles: [...], trends: [...], data: [...] }
```

### 2. AI Content Generation with Claude

```typescript
import { Anthropic } from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

async function generateContent(topic: string, format: 'toplist' | 'pov' | 'case-study' | 'how-to') {
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [{
      role: 'user',
      content: `Create a ${format} article about ${topic}. 
                Include data-backed insights and make it engaging.
                Format: Bilingual (English & Vietnamese)`
    }],
  });

  return message.content;
}

// Usage
const content = await generateContent('Marketing Automation', 'toplist');
```

### 3. Multi-Format Content Pipeline

```typescript
import { ContentPipeline } from '@/lib/content-pipeline';

const pipeline = new ContentPipeline({
  aiProvider: 'claude', // or 'openai'
  language: ['en', 'vi'],
  tone: 'professional' // or 'friendly', 'humorous'
});

// Full pipeline execution
const result = await pipeline.execute({
  keyword: 'AI Content Marketing',
  format: 'toplist',
  includeVideo: true,
  videoAspectRatio: '9:16' // for Reels/TikTok
});

console.log(result);
// {
//   content: { en: '...', vi: '...' },
//   metadata: { sources: [...], keywords: [...] },
//   video: { url: '...', thumbnail: '...' }
// }
```

### 4. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

async function generateVideo(contentData: any) {
  // Bundle Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.join(process.cwd(), 'remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  // Select composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title: contentData.title,
      points: contentData.points,
      style: 'modern'
    },
  });

  // Render video
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: `out/${contentData.slug}.mp4`,
  });
}

// Usage
await generateVideo({
  title: 'Top 5 AI Tools',
  points: ['Tool 1', 'Tool 2', '...'],
  slug: 'ai-tools-2024'
});
```

## API Routes

### Research Endpoint

```typescript
// app/api/research/route.ts
import { NextResponse } from 'next/server';
import { researchTopic } from '@/lib/research/scraper';

export async function POST(request: Request) {
  const { keyword, sources, timeframe } = await request.json();

  try {
    const data = await researchTopic({
      keyword,
      sources: sources || ['techcrunch', 'twitter'],
      timeframe: timeframe || '24h'
    });

    return NextResponse.json({ success: true, data });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

### Content Generation Endpoint

```typescript
// app/api/generate/route.ts
import { NextResponse } from 'next/server';
import { Anthropic } from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

export async function POST(request: Request) {
  const { topic, format, language, tone } = await request.json();

  const prompt = `Create a ${format} article about ${topic}.
    Language: ${language}
    Tone: ${tone}
    Include recent data and insights.`;

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [{ role: 'user', content: prompt }],
  });

  return NextResponse.json({
    content: message.content,
    usage: message.usage
  });
}
```

## Common Patterns

### Complete Content Creation Workflow

```typescript
import { ContentWorkflow } from '@/lib/workflow';

async function createContentWorkflow(keyword: string) {
  const workflow = new ContentWorkflow();

  // Step 1: Research
  const research = await workflow.research({
    keyword,
    depth: 'comprehensive'
  });

  // Step 2: Generate content in multiple formats
  const contents = await Promise.all([
    workflow.generate({ research, format: 'toplist' }),
    workflow.generate({ research, format: 'how-to' }),
    workflow.generate({ research, format: 'case-study' })
  ]);

  // Step 3: Create videos for each piece
  const videos = await Promise.all(
    contents.map(content => 
      workflow.renderVideo({
        content,
        aspectRatio: '9:16',
        platform: 'tiktok'
      })
    )
  );

  // Step 4: Schedule publishing
  await workflow.schedule({
    contents,
    videos,
    platforms: ['facebook', 'tiktok', 'youtube'],
    timing: 'optimal'
  });

  return { contents, videos };
}
```

### Custom AI Provider Integration

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
        content: 'You are an expert content marketer creating engaging, data-driven articles.'
      },
      {
        role: 'user',
        content: prompt
      }
    ],
    temperature: 0.7,
    max_tokens: 3000
  });

  return completion.choices[0].message.content;
}
```

### Bilingual Content Generation

```typescript
async function generateBilingualContent(topic: string) {
  const formats = {
    en: {
      tone: 'professional',
      audience: 'B2B marketers',
      length: 'comprehensive'
    },
    vi: {
      tone: 'friendly',
      audience: 'general audience',
      length: 'concise'
    }
  };

  const [englishContent, vietnameseContent] = await Promise.all([
    generateContent(topic, formats.en),
    generateContent(topic, formats.vi)
  ]);

  return {
    en: englishContent,
    vi: vietnameseContent,
    metadata: {
      topic,
      generatedAt: new Date().toISOString()
    }
  };
}
```

## Development Commands

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Start production server
npm run start

# Run type checking
npm run type-check

# Lint code
npm run lint

# Render Remotion video locally
npm run remotion:preview

# Build Remotion video
npm run remotion:render
```

## Configuration Examples

### Content Pipeline Config

```typescript
// config/pipeline.config.ts
export const pipelineConfig = {
  research: {
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    maxArticles: 10,
    timeframe: '24h'
  },
  ai: {
    provider: 'claude', // or 'openai'
    model: 'claude-3-5-sonnet-20241022',
    temperature: 0.7,
    maxTokens: 4096
  },
  content: {
    formats: ['toplist', 'pov', 'case-study', 'how-to'],
    languages: ['en', 'vi'],
    tones: ['professional', 'friendly', 'expert']
  },
  video: {
    defaultAspectRatio: '9:16',
    codec: 'h264',
    fps: 30,
    quality: 'high'
  }
};
```

### Remotion Composition Config

```typescript
// remotion/Root.tsx
import { Composition } from 'remotion';
import { ContentVideo } from './ContentVideo';

export const RemotionRoot: React.FC = () => {
  return (
    <>
      <Composition
        id="ContentVideo"
        component={ContentVideo}
        durationInFrames={900} // 30 seconds at 30fps
        fps={30}
        width={1080}
        height={1920}
        defaultProps={{
          title: 'Sample Title',
          points: [],
          theme: 'modern'
        }}
      />
    </>
  );
};
```

## Troubleshooting

### API Key Issues

```typescript
// Check if API keys are properly configured
function validateApiKeys() {
  const required = [
    'ANTHROPIC_API_KEY',
    'OPENAI_API_KEY',
    'RAPIDAPI_KEY'
  ];

  const missing = required.filter(key => !process.env[key]);

  if (missing.length > 0) {
    throw new Error(`Missing API keys: ${missing.join(', ')}`);
  }
}
```

### Rate Limiting Handling

```typescript
import { setTimeout } from 'timers/promises';

async function withRetry<T>(
  fn: () => Promise<T>,
  maxRetries = 3
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (error.status === 429 && i < maxRetries - 1) {
        // Rate limited, wait and retry
        await setTimeout(Math.pow(2, i) * 1000);
        continue;
      }
      throw error;
    }
  }
  throw new Error('Max retries exceeded');
}

// Usage
const content = await withRetry(() => 
  generateContent('AI trends', 'toplist')
);
```

### Video Rendering Issues

```typescript
// Ensure proper video rendering configuration
async function renderWithErrorHandling(composition: any) {
  try {
    await renderMedia({
      composition,
      serveUrl: bundleLocation,
      codec: 'h264',
      onProgress: ({ progress }) => {
        console.log(`Rendering progress: ${Math.round(progress * 100)}%`);
      },
    });
  } catch (error) {
    console.error('Video rendering failed:', error);
    // Fallback to lower quality or different codec
    await renderMedia({
      composition,
      serveUrl: bundleLocation,
      codec: 'h264',
      quality: 'low',
    });
  }
}
```

## Best Practices

1. **Always validate API keys** before making requests
2. **Implement retry logic** for API calls to handle rate limits
3. **Cache research results** to avoid redundant API calls
4. **Use TypeScript types** for content structures to ensure consistency
5. **Monitor token usage** to stay within API limits
6. **Optimize video rendering** by adjusting quality based on platform requirements
7. **Implement error boundaries** in React components for graceful degradation
8. **Use environment variables** for all configuration and secrets
