---
name: marketing-pipeline-ai-content-automation
description: Automated AI content pipeline for research, scriptwriting, and video generation using Claude/OpenAI and Remotion
triggers:
  - how do I generate automated content with AI
  - set up AI content pipeline for social media
  - create automated video content from text
  - build content automation workflow with Claude
  - generate videos and posts automatically
  - automate content research and creation
  - use Remotion for video generation
  - create AI-powered marketing content pipeline
---

# Marketing Pipeline AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill provides expertise in using the **Ultimate AI Content Pipeline** project, a comprehensive TypeScript-based system that automates content creation from research to video generation. The pipeline integrates Claude 3, OpenAI, and Remotion to create a complete content production workflow.

## What This Project Does

The marketing-pipeline-share project is an end-to-end content automation system that:

- **Auto-crawls** news and insights from sources like TechCrunch, a16z, Twitter/X, and LinkedIn
- **Generates content** in multiple formats (toplist, POV, case study, how-to) using Claude or OpenAI
- **Creates bilingual content** (English and Vietnamese) with customizable tone
- **Renders videos automatically** using Remotion for social media platforms (Reels, TikTok, Shorts)
- **Provides a Next.js interface** for managing the entire pipeline

## Installation

### Prerequisites

```bash
# Node.js 18+ required
node --version

# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
pnpm install
# or
yarn install
```

### Environment Configuration

Create a `.env.local` file in the root directory:

```bash
# AI Provider Keys
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Remotion Configuration
REMOTION_LICENSE_KEY=your_remotion_license_key

# Next.js Configuration
NEXT_PUBLIC_API_URL=http://localhost:3000
```

### Start the Development Server

```bash
npm run dev
# or
pnpm dev
# or
yarn dev
```

Access the application at `http://localhost:3000`

## Key API & Commands

### Content Research Module

```typescript
// src/lib/research/crawler.ts
import { crawlNews } from '@/lib/research/crawler';

interface ResearchConfig {
  keyword: string;
  sources: string[];
  timeRange: '24h' | '7d' | '30d';
  language: 'en' | 'vi';
}

async function gatherResearch(config: ResearchConfig) {
  const results = await crawlNews({
    keyword: config.keyword,
    sources: config.sources,
    timeRange: config.timeRange,
  });
  
  return results;
}

// Example usage
const research = await gatherResearch({
  keyword: 'AI marketing automation',
  sources: ['techcrunch', 'a16z', 'twitter'],
  timeRange: '24h',
  language: 'en',
});
```

### Content Generation with Claude

```typescript
// src/lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

interface ContentParams {
  topic: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  tone: 'expert' | 'friendly' | 'humorous';
  language: 'en' | 'vi';
  researchData: any[];
}

async function generateContent(params: ContentParams) {
  const prompt = buildPrompt(params);
  
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [
      {
        role: 'user',
        content: prompt,
      },
    ],
  });
  
  return message.content[0].text;
}

function buildPrompt(params: ContentParams): string {
  const formatInstructions = {
    'toplist': 'Create a numbered list article with clear rankings',
    'pov': 'Write from a unique perspective with personal insights',
    'case-study': 'Analyze a real-world example with data and outcomes',
    'how-to': 'Provide step-by-step instructions with actionable tips',
  };
  
  return `
You are a ${params.tone} content writer. Create a ${params.format} article in ${params.language} about "${params.topic}".

${formatInstructions[params.format]}

Use the following research data:
${JSON.stringify(params.researchData, null, 2)}

Requirements:
- Data-backed insights
- Engaging headlines
- SEO-optimized
- Platform-ready formatting
`;
}
```

### Content Generation with OpenAI

```typescript
// src/lib/ai/openai-generator.ts
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateWithGPT(params: ContentParams) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are a ${params.tone} content creator specializing in ${params.format} articles.`,
      },
      {
        role: 'user',
        content: buildPrompt(params),
      },
    ],
    temperature: 0.7,
    max_tokens: 3000,
  });
  
  return completion.choices[0].message.content;
}
```

### Video Generation with Remotion

```typescript
// src/lib/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  content: string;
  format: 'reels' | 'tiktok' | 'shorts';
  outputPath: string;
}

async function renderContentVideo(config: VideoConfig) {
  const bundleLocation = await bundle({
    entryPoint: path.join(process.cwd(), 'src/remotion/index.ts'),
    webpackOverride: (config) => config,
  });
  
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      content: config.content,
      format: config.format,
    },
  });
  
  const dimensions = getFormatDimensions(config.format);
  
  await renderMedia({
    composition: {
      ...composition,
      width: dimensions.width,
      height: dimensions.height,
    },
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: config.outputPath,
    inputProps: {
      content: config.content,
      format: config.format,
    },
  });
  
  return config.outputPath;
}

function getFormatDimensions(format: string) {
  const formats = {
    reels: { width: 1080, height: 1920 },
    tiktok: { width: 1080, height: 1920 },
    shorts: { width: 1080, height: 1920 },
  };
  return formats[format] || formats.reels;
}
```

### Remotion Composition Example

```typescript
// src/remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig } from 'remotion';
import React from 'react';

interface ContentVideoProps {
  content: string;
  format: string;
}

export const ContentVideo: React.FC<ContentVideoProps> = ({ content, format }) => {
  const frame = useCurrentFrame();
  const { fps, durationInFrames } = useVideoConfig();
  
  const opacity = Math.min(1, frame / 30);
  
  return (
    <AbsoluteFill
      style={{
        backgroundColor: '#000',
        justifyContent: 'center',
        alignItems: 'center',
      }}
    >
      <div
        style={{
          fontSize: 60,
          color: '#fff',
          textAlign: 'center',
          padding: 40,
          opacity,
        }}
      >
        {content}
      </div>
    </AbsoluteFill>
  );
};
```

## Complete Pipeline Integration

```typescript
// src/lib/pipeline/orchestrator.ts
import { gatherResearch } from '@/lib/research/crawler';
import { generateContent } from '@/lib/ai/content-generator';
import { renderContentVideo } from '@/lib/video/renderer';

interface PipelineConfig {
  keyword: string;
  contentFormat: 'toplist' | 'pov' | 'case-study' | 'how-to';
  tone: 'expert' | 'friendly' | 'humorous';
  language: 'en' | 'vi';
  generateVideo: boolean;
  videoFormat?: 'reels' | 'tiktok' | 'shorts';
}

async function runContentPipeline(config: PipelineConfig) {
  try {
    // Step 1: Research
    console.log('🔍 Starting research phase...');
    const research = await gatherResearch({
      keyword: config.keyword,
      sources: ['techcrunch', 'a16z', 'twitter'],
      timeRange: '24h',
      language: config.language,
    });
    
    // Step 2: Generate Content
    console.log('✍️ Generating content...');
    const content = await generateContent({
      topic: config.keyword,
      format: config.contentFormat,
      tone: config.tone,
      language: config.language,
      researchData: research,
    });
    
    // Step 3: Render Video (optional)
    let videoPath = null;
    if (config.generateVideo && config.videoFormat) {
      console.log('🎬 Rendering video...');
      videoPath = await renderContentVideo({
        content,
        format: config.videoFormat,
        outputPath: `./output/${Date.now()}.mp4`,
      });
    }
    
    return {
      content,
      videoPath,
      research,
      metadata: {
        keyword: config.keyword,
        format: config.contentFormat,
        language: config.language,
        generatedAt: new Date().toISOString(),
      },
    };
  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}

// Example usage
const result = await runContentPipeline({
  keyword: 'AI content automation 2024',
  contentFormat: 'how-to',
  tone: 'friendly',
  language: 'en',
  generateVideo: true,
  videoFormat: 'reels',
});

console.log('✅ Pipeline completed!', result);
```

## Next.js API Routes

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline/orchestrator';

export async function POST(req: NextRequest) {
  try {
    const body = await req.json();
    
    const result = await runContentPipeline({
      keyword: body.keyword,
      contentFormat: body.format || 'how-to',
      tone: body.tone || 'friendly',
      language: body.language || 'en',
      generateVideo: body.generateVideo || false,
      videoFormat: body.videoFormat || 'reels',
    });
    
    return NextResponse.json({
      success: true,
      data: result,
    });
  } catch (error) {
    return NextResponse.json(
      {
        success: false,
        error: error.message,
      },
      { status: 500 }
    );
  }
}
```

## Common Patterns

### Bilingual Content Generation

```typescript
async function generateBilingualContent(keyword: string) {
  const [englishContent, vietnameseContent] = await Promise.all([
    generateContent({
      topic: keyword,
      format: 'how-to',
      tone: 'expert',
      language: 'en',
      researchData: [],
    }),
    generateContent({
      topic: keyword,
      format: 'how-to',
      tone: 'expert',
      language: 'vi',
      researchData: [],
    }),
  ]);
  
  return { englishContent, vietnameseContent };
}
```

### Batch Content Generation

```typescript
async function batchGenerateContent(keywords: string[]) {
  const results = await Promise.all(
    keywords.map((keyword) =>
      runContentPipeline({
        keyword,
        contentFormat: 'toplist',
        tone: 'friendly',
        language: 'en',
        generateVideo: false,
      })
    )
  );
  
  return results;
}
```

### Custom Research Sources

```typescript
// src/lib/research/custom-crawler.ts
interface CustomSource {
  name: string;
  url: string;
  selector: string;
}

async function crawlCustomSource(source: CustomSource, keyword: string) {
  // Implement custom crawling logic
  const response = await fetch(`${source.url}/search?q=${keyword}`);
  const html = await response.text();
  
  // Parse and extract data
  // Return structured results
  return {
    source: source.name,
    articles: [],
  };
}
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement rate limiting
import pLimit from 'p-limit';

const limit = pLimit(3); // Max 3 concurrent requests

async function generateWithRateLimit(topics: string[]) {
  return Promise.all(
    topics.map((topic) =>
      limit(() =>
        generateContent({
          topic,
          format: 'how-to',
          tone: 'friendly',
          language: 'en',
          researchData: [],
        })
      )
    )
  );
}
```

### Video Rendering Memory Issues

```typescript
// Use concurrency control for video rendering
import { bundle } from '@remotion/bundler';

async function renderVideoSafely(config: VideoConfig) {
  try {
    // Clear cache before rendering
    process.env.REMOTION_CACHE = 'false';
    
    const result = await renderContentVideo(config);
    return result;
  } catch (error) {
    if (error.message.includes('out of memory')) {
      console.error('Memory error: Try reducing video resolution');
      // Implement fallback logic
    }
    throw error;
  }
}
```

### Research Data Quality

```typescript
// Validate and filter research data
function validateResearchData(data: any[]) {
  return data.filter((item) => {
    return (
      item.title &&
      item.content &&
      item.publishedAt &&
      new Date(item.publishedAt) > new Date(Date.now() - 7 * 24 * 60 * 60 * 1000)
    );
  });
}
```

### Error Handling Best Practices

```typescript
async function robustPipeline(config: PipelineConfig) {
  const maxRetries = 3;
  let attempt = 0;
  
  while (attempt < maxRetries) {
    try {
      return await runContentPipeline(config);
    } catch (error) {
      attempt++;
      console.error(`Attempt ${attempt} failed:`, error);
      
      if (attempt >= maxRetries) {
        throw new Error(`Pipeline failed after ${maxRetries} attempts`);
      }
      
      // Exponential backoff
      await new Promise((resolve) =>
        setTimeout(resolve, Math.pow(2, attempt) * 1000)
      );
    }
  }
}
```

## CLI Commands (if applicable)

```bash
# Generate content from command line
npm run generate -- --keyword "AI marketing" --format how-to

# Render video only
npm run render-video -- --input content.json --format reels

# Run full pipeline
npm run pipeline -- --config pipeline.config.json

# Build for production
npm run build

# Start production server
npm run start
```

This skill enables AI coding agents to effectively implement and customize the marketing pipeline automation system for automated content creation, research, and video generation workflows.
