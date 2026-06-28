---
name: marketing-content-pipeline-automation
description: Automate content creation from research to video generation using AI (Claude, OpenAI) and Remotion
triggers:
  - how do I automate content research and generation
  - create automated marketing content pipeline
  - generate videos from blog posts automatically
  - set up AI content automation with Claude and OpenAI
  - build content pipeline with research and video generation
  - automate social media content creation workflow
  - use Remotion for automated video content
  - create AI-powered content creation system
---

# Marketing Content Pipeline Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to help developers build and use an automated content creation pipeline that handles research, script generation, and video rendering using AI models (Claude 3, OpenAI) and Remotion.

## What This Project Does

**Ultimate AI Content Pipeline** is a complete content automation system that:

- **Auto-researches** trending topics from sources like TechCrunch, a16z, Twitter/X, LinkedIn
- **Generates multi-format content** (listicles, POV pieces, case studies, how-tos) in multiple languages
- **Renders videos automatically** from written content using Remotion
- **Optimizes for multiple platforms** (Reels, TikTok, Shorts)

The pipeline transforms a single keyword into complete, multi-format content ready for publication.

## Installation

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
yarn install
```

## Environment Configuration

Create a `.env.local` file in the root directory:

```bash
# AI Models
ANTHROPIC_API_KEY=your_anthropic_key_here
OPENAI_API_KEY=your_openai_key_here

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key_here

# Optional: Custom configurations
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app router
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── research/    # Content research crawlers
│   │   ├── video/       # Remotion video generation
│   │   └── utils/       # Utility functions
│   └── types/           # TypeScript types
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Core Workflows

### 1. Research & Data Collection

```typescript
import { researchTopic } from '@/lib/research/crawler';

// Crawl and analyze recent content
async function gatherResearch(keyword: string) {
  const research = await researchTopic({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeframe: '24h',
    maxResults: 20
  });

  return {
    insights: research.insights,
    dataPoints: research.statistics,
    trends: research.trends,
    references: research.sources
  };
}
```

### 2. AI Content Generation

```typescript
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

// Generate content using Claude
async function generateWithClaude(
  topic: string,
  research: any,
  format: 'toplist' | 'pov' | 'case-study' | 'how-to'
) {
  const anthropic = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY,
  });

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [
      {
        role: 'user',
        content: `Create a ${format} article about ${topic}.
        
Research data: ${JSON.stringify(research)}

Requirements:
- Use data-backed insights
- Include statistics and trends
- Write in both English and Vietnamese
- Optimize for engagement`
      }
    ],
  });

  return message.content;
}

// Generate content using OpenAI
async function generateWithOpenAI(
  topic: string,
  research: any,
  tone: 'expert' | 'friendly' | 'humorous'
) {
  const openai = new OpenAI({
    apiKey: process.env.OPENAI_API_KEY,
  });

  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are a ${tone} content writer. Create engaging content based on research data.`
      },
      {
        role: 'user',
        content: `Topic: ${topic}\nResearch: ${JSON.stringify(research)}`
      }
    ],
    temperature: 0.7,
  });

  return completion.choices[0].message.content;
}
```

### 3. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import { VideoTemplate } from '@/remotion/VideoTemplate';

async function generateVideo(content: {
  title: string;
  points: string[];
  images: string[];
}) {
  const bundled = await bundle({
    entryPoint: './remotion/index.ts',
    webpackOverride: (config) => config,
  });

  const composition = await selectComposition({
    serveUrl: bundled,
    id: 'ContentVideo',
    inputProps: {
      title: content.title,
      points: content.points,
      images: content.images,
      duration: 30, // seconds
    },
  });

  await renderMedia({
    composition,
    serveUrl: bundled,
    codec: 'h264',
    outputLocation: `./output/${content.title}.mp4`,
    imageFormat: 'jpeg',
  });

  return `./output/${content.title}.mp4`;
}
```

### 4. Complete Pipeline Example

```typescript
import { ContentPipeline } from '@/lib/pipeline';

interface PipelineConfig {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  languages: string[];
  generateVideo: boolean;
  videoFormat: 'reels' | 'tiktok' | 'shorts';
}

async function runContentPipeline(config: PipelineConfig) {
  const pipeline = new ContentPipeline();

  // Step 1: Research
  console.log('🔍 Researching topic...');
  const research = await pipeline.research({
    keyword: config.keyword,
    timeframe: '24h',
  });

  // Step 2: Generate content
  console.log('✍️ Generating content...');
  const content = await pipeline.generate({
    research,
    format: config.format,
    languages: config.languages,
    aiModel: 'claude-3-5-sonnet', // or 'gpt-4'
  });

  // Step 3: Create video (optional)
  let video = null;
  if (config.generateVideo) {
    console.log('🎬 Rendering video...');
    video = await pipeline.renderVideo({
      content,
      format: config.videoFormat,
      aspectRatio: config.videoFormat === 'reels' ? '9:16' : '16:9',
    });
  }

  return {
    article: content,
    video,
    metadata: {
      keyword: config.keyword,
      generatedAt: new Date(),
      sources: research.references,
    },
  };
}

// Usage
const result = await runContentPipeline({
  keyword: 'AI automation trends 2026',
  format: 'toplist',
  languages: ['en', 'vi'],
  generateVideo: true,
  videoFormat: 'reels',
});

console.log('✅ Content pipeline complete!');
console.log('Article:', result.article);
console.log('Video:', result.video);
```

## API Endpoints

### Next.js API Routes

```typescript
// app/api/content/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';

export async function POST(request: NextRequest) {
  const { keyword, format, languages } = await request.json();

  try {
    // Research phase
    const research = await researchTopic(keyword);
    
    // Generation phase
    const content = await generateContent({
      keyword,
      research,
      format,
      languages,
    });

    return NextResponse.json({
      success: true,
      data: content,
    });
  } catch (error) {
    return NextResponse.json(
      { error: 'Content generation failed' },
      { status: 500 }
    );
  }
}

// app/api/video/render/route.ts
export async function POST(request: NextRequest) {
  const { content, format } = await request.json();

  try {
    const videoPath = await generateVideo({
      ...content,
      format,
    });

    return NextResponse.json({
      success: true,
      videoUrl: videoPath,
    });
  } catch (error) {
    return NextResponse.json(
      { error: 'Video rendering failed' },
      { status: 500 }
    );
  }
}
```

## Common Patterns

### Multi-Language Content Generation

```typescript
async function generateMultiLanguage(
  topic: string,
  research: any,
  languages: string[]
) {
  const contents: Record<string, string> = {};

  for (const lang of languages) {
    const prompt = `Generate content in ${lang} about ${topic}`;
    contents[lang] = await generateWithClaude(topic, research, 'toplist');
  }

  return contents;
}
```

### Batch Content Processing

```typescript
async function processBatch(keywords: string[]) {
  const results = await Promise.all(
    keywords.map(async (keyword) => {
      try {
        return await runContentPipeline({
          keyword,
          format: 'toplist',
          languages: ['en', 'vi'],
          generateVideo: true,
          videoFormat: 'reels',
        });
      } catch (error) {
        console.error(`Failed for ${keyword}:`, error);
        return null;
      }
    })
  );

  return results.filter(Boolean);
}
```

### Custom Video Templates

```typescript
// remotion/CustomTemplate.tsx
import { AbsoluteFill, useCurrentFrame } from 'remotion';

export const CustomTemplate: React.FC<{
  title: string;
  points: string[];
}> = ({ title, points }) => {
  const frame = useCurrentFrame();

  return (
    <AbsoluteFill style={{ backgroundColor: '#000' }}>
      <h1 style={{ opacity: frame / 30 }}>{title}</h1>
      {points.map((point, i) => (
        <p key={i} style={{ opacity: Math.max(0, (frame - i * 30) / 30) }}>
          {point}
        </p>
      ))}
    </AbsoluteFill>
  );
};
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement retry logic with exponential backoff
async function callAIWithRetry(
  fn: () => Promise<any>,
  maxRetries = 3
) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error: any) {
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

### Video Rendering Memory Issues

```typescript
// Use Remotion's memory-efficient settings
await renderMedia({
  composition,
  serveUrl: bundled,
  codec: 'h264',
  outputLocation: output,
  chromiumOptions: {
    headless: true,
    args: ['--no-sandbox', '--disable-dev-shm-usage'],
  },
  concurrency: 2, // Limit concurrent renders
});
```

### Research Data Quality

```typescript
// Validate and filter research results
function validateResearch(research: any) {
  return research.insights.filter((insight: any) => {
    return (
      insight.date &&
      new Date(insight.date) > new Date(Date.now() - 86400000) &&
      insight.source &&
      insight.content.length > 100
    );
  });
}
```

## Development Commands

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Render Remotion video locally
npm run remotion:render

# Type checking
npm run type-check

# Linting
npm run lint
```

## Best Practices

1. **Cache research data** to avoid redundant API calls
2. **Use streaming responses** for real-time content generation feedback
3. **Implement queue systems** for video rendering (CPU-intensive)
4. **Store generated content** in a database for reuse
5. **Monitor API costs** with usage tracking
6. **Version your prompts** for reproducible results
7. **Test video templates** before batch processing

This skill enables AI agents to effectively implement and customize automated content pipelines using this TypeScript-based system.
