---
name: ultimate-ai-content-pipeline
description: Automated content creation pipeline from research to script to video generation using Claude, OpenAI, and Remotion
triggers:
  - how do I automate content creation with AI
  - generate video content from text automatically
  - crawl news and create content with Claude
  - setup AI marketing content pipeline
  - create automated social media videos
  - research and generate content with OpenAI
  - build content automation workflow
  - use Remotion for video generation
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the Ultimate AI Content Pipeline - a comprehensive TypeScript-based system that automates content creation from research through video generation. The pipeline crawls news sources, generates multi-format content using Claude/OpenAI, and renders videos with Remotion.

## What This Project Does

Ultimate AI Content Pipeline is an all-in-one content automation system that:

- **Auto-crawls** news sources (TechCrunch, a16z, Twitter, LinkedIn) for fresh data
- **Generates content** in multiple formats (listicles, POV, case studies, how-tos) using Claude 3 and OpenAI
- **Supports bilingual** output (English and Vietnamese)
- **Renders videos** automatically from content using Remotion
- **Optimizes for platforms** like Reels, TikTok, and Shorts

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
cp .env.example .env
```

### Required Environment Variables

```bash
# AI Models
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_connection

# Video Rendering
REMOTION_LICENSE_KEY=your_remotion_key
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── research/       # News crawling and data extraction
│   ├── generation/     # AI content generation
│   ├── video/          # Remotion video rendering
│   ├── api/            # API routes (Next.js)
│   └── utils/          # Helper functions
├── public/             # Static assets
└── remotion/           # Video templates
```

## Core Components

### 1. Research Module (Auto-Crawling)

The research module crawls news sources and extracts insights.

```typescript
import { researchTopic } from './src/research/crawler';

// Crawl news for a specific topic
async function gatherResearch(keyword: string) {
  const research = await researchTopic({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeframe: '24h', // Last 24 hours
    maxResults: 20
  });

  return {
    articles: research.articles,
    insights: research.extractedInsights,
    statistics: research.dataPoints
  };
}

// Usage
const data = await gatherResearch('AI automation');
console.log(`Found ${data.articles.length} articles`);
```

### 2. Content Generation with AI

Generate content in multiple formats using Claude or OpenAI.

```typescript
import { generateContent } from './src/generation/ai-writer';
import { ContentFormat, Language, Tone } from './src/types';

async function createContent(research: any, format: ContentFormat) {
  const content = await generateContent({
    researchData: research,
    format: format, // 'toplist' | 'pov' | 'case-study' | 'how-to'
    language: 'en', // 'en' | 'vi'
    tone: 'expert', // 'expert' | 'friendly' | 'humorous'
    aiModel: 'claude-3-opus', // or 'gpt-4'
    wordCount: 1500
  });

  return content;
}

// Generate a top list article
const article = await createContent(researchData, 'toplist');
```

### 3. Bilingual Content Generation

Create content in both English and Vietnamese simultaneously.

```typescript
import { generateBilingualContent } from './src/generation/bilingual';

async function createBilingualPost(topic: string) {
  const research = await researchTopic({ keyword: topic });
  
  const content = await generateBilingualContent({
    research,
    format: 'how-to',
    tone: 'friendly',
    includeImages: true
  });

  return {
    english: content.en,
    vietnamese: content.vi,
    metadata: content.metadata
  };
}

const post = await createBilingualPost('Content marketing automation');
```

### 4. Video Rendering with Remotion

Transform text content into videos automatically.

```typescript
import { renderVideo } from './src/video/renderer';
import { VideoTemplate } from './src/types';

async function createVideoFromContent(content: any) {
  const video = await renderVideo({
    template: 'reels', // 'reels' | 'tiktok' | 'shorts' | 'story'
    content: {
      title: content.title,
      subtitle: content.subtitle,
      keyPoints: content.bulletPoints,
      backgroundMusic: 'upbeat',
      voiceOver: false
    },
    duration: 30, // seconds
    aspectRatio: '9:16', // vertical for mobile
    outputFormat: 'mp4'
  });

  return video.outputPath;
}

// Render video
const videoPath = await createVideoFromContent(article);
console.log(`Video saved to: ${videoPath}`);
```

### 5. Complete Pipeline Workflow

End-to-end content creation pipeline.

```typescript
import { ContentPipeline } from './src/pipeline';

async function runFullPipeline(keyword: string) {
  const pipeline = new ContentPipeline({
    openaiKey: process.env.OPENAI_API_KEY!,
    claudeKey: process.env.ANTHROPIC_API_KEY!,
    rapidApiKey: process.env.RAPIDAPI_KEY!
  });

  // Step 1: Research
  console.log('Researching topic...');
  const research = await pipeline.research(keyword);

  // Step 2: Generate content
  console.log('Generating content...');
  const content = await pipeline.generate({
    research,
    format: 'toplist',
    languages: ['en', 'vi']
  });

  // Step 3: Create video
  console.log('Rendering video...');
  const video = await pipeline.renderVideo({
    content: content.en,
    template: 'reels'
  });

  // Step 4: Export results
  return {
    articles: content,
    video: video.path,
    metadata: {
      sources: research.sources.length,
      wordCount: content.en.wordCount,
      videoDuration: video.duration
    }
  };
}

// Run the complete pipeline
const result = await runFullPipeline('AI in marketing');
```

## Next.js API Integration

### API Route for Content Generation

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { ContentPipeline } from '@/pipeline';

export async function POST(req: NextRequest) {
  try {
    const { keyword, format, language } = await req.json();

    const pipeline = new ContentPipeline({
      openaiKey: process.env.OPENAI_API_KEY!,
      claudeKey: process.env.ANTHROPIC_API_KEY!,
      rapidApiKey: process.env.RAPIDAPI_KEY!
    });

    const research = await pipeline.research(keyword);
    const content = await pipeline.generate({
      research,
      format,
      languages: [language]
    });

    return NextResponse.json({
      success: true,
      data: content
    });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

### Frontend Usage

```typescript
// src/app/components/ContentGenerator.tsx
'use client';

import { useState } from 'react';

export default function ContentGenerator() {
  const [keyword, setKeyword] = useState('');
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState(null);

  async function generateContent() {
    setLoading(true);
    
    const response = await fetch('/api/generate', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        keyword,
        format: 'toplist',
        language: 'en'
      })
    });

    const data = await response.json();
    setResult(data);
    setLoading(false);
  }

  return (
    <div>
      <input
        value={keyword}
        onChange={(e) => setKeyword(e.target.value)}
        placeholder="Enter topic..."
      />
      <button onClick={generateContent} disabled={loading}>
        {loading ? 'Generating...' : 'Generate Content'}
      </button>
      {result && <pre>{JSON.stringify(result, null, 2)}</pre>}
    </div>
  );
}
```

## Custom Video Templates

Create custom Remotion video templates.

```typescript
// remotion/templates/CustomReels.tsx
import { AbsoluteFill, useCurrentFrame, interpolate } from 'remotion';

export const CustomReels: React.FC<{ content: any }> = ({ content }) => {
  const frame = useCurrentFrame();
  
  const opacity = interpolate(frame, [0, 30], [0, 1], {
    extrapolateRight: 'clamp'
  });

  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a1a' }}>
      <div style={{ 
        opacity, 
        padding: 40, 
        color: 'white',
        fontSize: 48,
        fontWeight: 'bold'
      }}>
        {content.title}
      </div>
      <div style={{ padding: 40, color: '#ccc', fontSize: 24 }}>
        {content.keyPoints.map((point: string, i: number) => (
          <div key={i} style={{ marginTop: 20 }}>• {point}</div>
        ))}
      </div>
    </AbsoluteFill>
  );
};
```

## Configuration Patterns

### Advanced Pipeline Configuration

```typescript
import { PipelineConfig } from './src/types';

const config: PipelineConfig = {
  research: {
    sources: ['techcrunch', 'a16z', 'producthunt', 'twitter'],
    depth: 'comprehensive', // 'quick' | 'standard' | 'comprehensive'
    cacheDuration: 3600, // 1 hour
    filterKeywords: ['AI', 'automation', 'marketing']
  },
  generation: {
    primaryModel: 'claude-3-opus',
    fallbackModel: 'gpt-4',
    temperature: 0.7,
    maxTokens: 4000,
    retryAttempts: 3
  },
  video: {
    defaultTemplate: 'reels',
    quality: 'high', // 'low' | 'medium' | 'high'
    codec: 'h264',
    fps: 30
  }
};

const pipeline = new ContentPipeline(config);
```

## Common Patterns

### Batch Content Generation

```typescript
async function generateBatchContent(keywords: string[]) {
  const results = [];

  for (const keyword of keywords) {
    const research = await researchTopic({ keyword });
    const content = await generateContent({
      researchData: research,
      format: 'toplist',
      language: 'en'
    });
    
    results.push({
      keyword,
      content,
      timestamp: new Date()
    });

    // Rate limiting
    await new Promise(resolve => setTimeout(resolve, 2000));
  }

  return results;
}
```

### Schedule Content Publishing

```typescript
import { schedule } from './src/scheduler';

async function scheduleContentSeries(topic: string, dates: Date[]) {
  const research = await researchTopic({ keyword: topic });

  for (const date of dates) {
    const content = await generateContent({
      researchData: research,
      format: 'how-to'
    });

    await schedule.create({
      content,
      publishAt: date,
      platforms: ['facebook', 'linkedin', 'twitter']
    });
  }
}
```

### Content with SEO Optimization

```typescript
import { optimizeForSEO } from './src/generation/seo';

async function createSEOContent(keyword: string) {
  const research = await researchTopic({ keyword });
  const content = await generateContent({
    researchData: research,
    format: 'toplist'
  });

  const optimized = await optimizeForSEO({
    content,
    targetKeyword: keyword,
    relatedKeywords: research.keywords,
    metaDescription: true,
    internalLinks: true
  });

  return optimized;
}
```

## Troubleshooting

### API Rate Limits

```typescript
import { RateLimiter } from './src/utils/rate-limiter';

const limiter = new RateLimiter({
  openai: { requestsPerMinute: 50 },
  claude: { requestsPerMinute: 40 },
  rapidapi: { requestsPerMinute: 100 }
});

async function generateWithRateLimit(keyword: string) {
  await limiter.waitForSlot('claude');
  return await generateContent({ keyword });
}
```

### Error Handling

```typescript
import { PipelineError } from './src/errors';

async function safeGeneration(keyword: string) {
  try {
    return await runFullPipeline(keyword);
  } catch (error) {
    if (error instanceof PipelineError) {
      console.error(`Pipeline failed at: ${error.stage}`);
      console.error(`Reason: ${error.message}`);
      
      // Retry with fallback
      if (error.stage === 'generation' && error.retryable) {
        return await runFullPipeline(keyword, { useFallback: true });
      }
    }
    throw error;
  }
}
```

### Video Rendering Issues

```typescript
// Check Remotion installation
import { getVideoMetadata } from '@remotion/renderer';

async function debugVideoRender(compositionId: string) {
  try {
    const metadata = await getVideoMetadata({
      compositionId,
      inputProps: {}
    });
    console.log('Video metadata:', metadata);
  } catch (error) {
    console.error('Remotion error:', error);
    // Ensure @remotion/cli is installed
    // npm install @remotion/cli @remotion/renderer
  }
}
```

### Memory Management for Large Batches

```typescript
async function generateLargeBatch(keywords: string[]) {
  const chunkSize = 5;
  const results = [];

  for (let i = 0; i < keywords.length; i += chunkSize) {
    const chunk = keywords.slice(i, i + chunkSize);
    const chunkResults = await Promise.all(
      chunk.map(keyword => runFullPipeline(keyword))
    );
    
    results.push(...chunkResults);
    
    // Clear memory
    if (global.gc) global.gc();
  }

  return results;
}
```

## Running the Application

```bash
# Development mode
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Render video (Remotion CLI)
npm run remotion render src/video/index.tsx <composition-id> output.mp4
```

This skill enables comprehensive automation of content marketing workflows using cutting-edge AI models and video generation technology.
