---
name: ultimate-ai-content-pipeline
description: Automated content creation pipeline from research to video generation using Claude, OpenAI, and Remotion
triggers:
  - how do I automate content creation with AI
  - generate video from text content automatically
  - scrape news and create social media posts
  - build an AI content pipeline with Claude
  - automate research and scriptwriting workflow
  - create videos for TikTok and Reels automatically
  - set up automated content generation system
  - use Remotion to render marketing videos
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

Ultimate AI Content Pipeline is a comprehensive TypeScript-based automation system that transforms keyword inputs into fully-researched articles and videos. It combines web scraping, AI content generation (Claude 3/OpenAI), and automated video rendering (Remotion) into a single workflow.

**Key capabilities:**
- Auto-scrapes fresh news from TechCrunch, a16z, Twitter, LinkedIn
- Generates multi-format content (toplist, POV, case study, how-to)
- Supports bilingual output (English/Vietnamese)
- Renders videos and infographics automatically
- Optimized for social media (Reels, TikTok, Shorts)

## Installation

```bash
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

## Environment Configuration

Create a `.env.local` file in the root directory:

```env
# AI Providers
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Web Scraping (RapidAPI)
RAPIDAPI_KEY=your_rapidapi_key

# Optional: Custom endpoints
CLAUDE_API_BASE_URL=https://api.anthropic.com
OPENAI_API_BASE_URL=https://api.openai.com/v1

# Content Settings
DEFAULT_LANGUAGE=en
ENABLE_VIDEO_GENERATION=true
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── research/          # Web scraping modules
│   ├── generators/        # AI content generation
│   ├── video/            # Remotion video rendering
│   ├── utils/            # Helper functions
│   └── types/            # TypeScript definitions
├── public/               # Static assets
├── remotion/            # Video templates
└── pages/               # Next.js pages
```

## Core Workflows

### 1. Research & Data Collection

The research module automatically scrapes recent news from multiple sources:

```typescript
import { ResearchEngine } from '@/research/engine';

async function collectResearch(keyword: string) {
  const engine = new ResearchEngine({
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeframe: '24h',
    maxResults: 20
  });

  const insights = await engine.search(keyword);
  
  return {
    articles: insights.articles,
    trends: insights.trends,
    dataPoints: insights.statistics
  };
}

// Usage
const research = await collectResearch('AI automation');
console.log(`Found ${research.articles.length} relevant articles`);
```

### 2. AI Content Generation

Generate content in multiple formats using Claude or OpenAI:

```typescript
import { ContentGenerator } from '@/generators/content';

async function generateContent(topic: string, format: 'toplist' | 'pov' | 'case-study' | 'how-to') {
  const generator = new ContentGenerator({
    provider: 'claude', // or 'openai'
    model: 'claude-3-opus-20240229',
    temperature: 0.7
  });

  const content = await generator.create({
    topic,
    format,
    language: 'en', // or 'vi' for Vietnamese
    tone: 'professional', // or 'friendly', 'humorous'
    researchData: research // from previous step
  });

  return {
    title: content.title,
    body: content.body,
    seo: content.seoMetadata,
    socialPosts: content.socialSnippets
  };
}

// Usage
const article = await generateContent(
  'Top AI Tools for Content Creators',
  'toplist'
);
```

### 3. Bilingual Content Creation

Generate parallel English/Vietnamese versions:

```typescript
import { BilingualGenerator } from '@/generators/bilingual';

async function createBilingualContent(topic: string) {
  const generator = new BilingualGenerator({
    primaryLanguage: 'en',
    secondaryLanguage: 'vi'
  });

  const content = await generator.generate({
    topic,
    format: 'how-to',
    syncStructure: true // Keep same headings/structure
  });

  return {
    en: {
      title: content.english.title,
      content: content.english.body
    },
    vi: {
      title: content.vietnamese.title,
      content: content.vietnamese.body
    }
  };
}
```

### 4. Video Generation with Remotion

Render videos from generated content:

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import { VideoComposition } from '@/video/compositions/ContentVideo';

async function renderContentVideo(content: any) {
  const bundleLocation = await bundle({
    entryPoint: './remotion/index.ts',
    webpackOverride: (config) => config
  });

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title: content.title,
      points: content.keyPoints,
      style: 'modern',
      duration: 30 // seconds
    }
  });

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: `out/${content.slug}.mp4`,
    inputProps: {
      title: content.title,
      points: content.keyPoints
    }
  });

  return `out/${content.slug}.mp4`;
}

// Usage
const videoPath = await renderContentVideo(article);
console.log(`Video rendered: ${videoPath}`);
```

### 5. Full Pipeline Execution

Complete end-to-end workflow:

```typescript
import { ContentPipeline } from '@/pipeline';

async function runFullPipeline(keyword: string) {
  const pipeline = new ContentPipeline({
    research: {
      sources: ['techcrunch', 'twitter', 'linkedin'],
      timeframe: '24h'
    },
    content: {
      provider: 'claude',
      formats: ['toplist', 'how-to'],
      languages: ['en', 'vi']
    },
    video: {
      enabled: process.env.ENABLE_VIDEO_GENERATION === 'true',
      aspectRatio: '9:16', // For Reels/TikTok
      duration: 30
    }
  });

  const results = await pipeline.execute(keyword);

  return {
    research: results.researchData,
    articles: results.generatedContent,
    videos: results.renderedVideos,
    socialPosts: results.socialMediaSnippets
  };
}

// Usage
const output = await runFullPipeline('AI content automation');
console.log(`Generated ${output.articles.length} articles`);
console.log(`Rendered ${output.videos.length} videos`);
```

## API Routes (Next.js)

### Research Endpoint

```typescript
// pages/api/research.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { ResearchEngine } from '@/research/engine';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  const { keyword, sources, timeframe } = req.body;

  try {
    const engine = new ResearchEngine({
      sources: sources || ['techcrunch', 'twitter'],
      timeframe: timeframe || '24h',
      apiKey: process.env.RAPIDAPI_KEY
    });

    const insights = await engine.search(keyword);
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
import { ContentGenerator } from '@/generators/content';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  const { topic, format, language, researchData } = req.body;

  try {
    const generator = new ContentGenerator({
      provider: 'claude',
      apiKey: process.env.ANTHROPIC_API_KEY
    });

    const content = await generator.create({
      topic,
      format,
      language,
      researchData
    });

    res.status(200).json(content);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
}
```

### Video Render Endpoint

```typescript
// pages/api/render-video.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { renderContentVideo } from '@/video/renderer';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  const { content, aspectRatio, duration } = req.body;

  try {
    const videoPath = await renderContentVideo({
      ...content,
      aspectRatio: aspectRatio || '9:16',
      duration: duration || 30
    });

    res.status(200).json({ 
      success: true, 
      videoUrl: `/videos/${videoPath}` 
    });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
}
```

## Common Patterns

### Custom Content Templates

```typescript
import { ContentTemplate } from '@/generators/templates';

// Define a custom template
const customTemplate: ContentTemplate = {
  name: 'product-review',
  structure: {
    intro: { tone: 'engaging', length: 'short' },
    body: {
      sections: [
        { type: 'features', format: 'bullet-points' },
        { type: 'pros-cons', format: 'comparison' },
        { type: 'verdict', format: 'summary' }
      ]
    },
    cta: { style: 'soft-sell' }
  }
};

// Use the template
const generator = new ContentGenerator({
  provider: 'claude',
  template: customTemplate
});
```

### Batch Processing

```typescript
async function batchGenerateContent(keywords: string[]) {
  const pipeline = new ContentPipeline(config);
  
  const results = await Promise.all(
    keywords.map(async (keyword) => {
      try {
        return await pipeline.execute(keyword);
      } catch (error) {
        console.error(`Failed for ${keyword}:`, error);
        return null;
      }
    })
  );

  return results.filter(r => r !== null);
}

// Usage
const keywords = ['AI automation', 'content marketing', 'video tools'];
const allContent = await batchGenerateContent(keywords);
```

### Custom Video Compositions

```typescript
// remotion/compositions/CustomVideo.tsx
import { AbsoluteFill, Sequence, useCurrentFrame } from 'remotion';

export const CustomVideo: React.FC<{
  title: string;
  points: string[];
}> = ({ title, points }) => {
  const frame = useCurrentFrame();

  return (
    <AbsoluteFill style={{ backgroundColor: '#000' }}>
      <Sequence from={0} durationInFrames={60}>
        <h1 style={{ color: '#fff', fontSize: 48 }}>{title}</h1>
      </Sequence>
      
      {points.map((point, i) => (
        <Sequence key={i} from={60 + i * 90} durationInFrames={90}>
          <div style={{ color: '#fff', fontSize: 32 }}>{point}</div>
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};
```

## Configuration Options

### Research Engine Settings

```typescript
interface ResearchConfig {
  sources: ('techcrunch' | 'a16z' | 'twitter' | 'linkedin')[];
  timeframe: '24h' | '7d' | '30d';
  maxResults: number;
  language?: 'en' | 'vi';
  includeStatistics?: boolean;
}
```

### Content Generator Settings

```typescript
interface GeneratorConfig {
  provider: 'claude' | 'openai';
  model?: string;
  temperature?: number; // 0-1
  maxTokens?: number;
  tone?: 'professional' | 'friendly' | 'humorous';
  format?: 'toplist' | 'pov' | 'case-study' | 'how-to';
}
```

### Video Renderer Settings

```typescript
interface VideoConfig {
  aspectRatio: '16:9' | '9:16' | '1:1';
  duration: number; // seconds
  fps: number;
  codec: 'h264' | 'h265';
  quality: number; // 0-100
  outputFormat: 'mp4' | 'webm';
}
```

## Troubleshooting

### API Rate Limits

```typescript
import { RateLimiter } from '@/utils/rate-limiter';

const limiter = new RateLimiter({
  maxRequests: 50,
  timeWindow: 60000 // 1 minute
});

async function safeClaude Call(prompt: string) {
  await limiter.wait();
  return await generator.create({ topic: prompt });
}
```

### Video Rendering Fails

```typescript
// Ensure ffmpeg is installed
import { getExecutableBinary } from '@remotion/renderer';

try {
  const ffmpegPath = await getExecutableBinary('ffmpeg');
  console.log('FFmpeg found at:', ffmpegPath);
} catch (error) {
  console.error('FFmpeg not found. Install with: npm install @remotion/ffmpeg');
}
```

### Memory Issues with Large Videos

```typescript
// Use lower quality settings for development
const devVideoConfig = {
  quality: 50,
  scale: 0.5,
  concurrency: 2
};

// Production settings
const prodVideoConfig = {
  quality: 90,
  scale: 1,
  concurrency: null // Use all cores
};

const config = process.env.NODE_ENV === 'production' 
  ? prodVideoConfig 
  : devVideoConfig;
```

### API Key Validation

```typescript
function validateEnvVars() {
  const required = [
    'ANTHROPIC_API_KEY',
    'OPENAI_API_KEY',
    'RAPIDAPI_KEY'
  ];

  const missing = required.filter(key => !process.env[key]);

  if (missing.length > 0) {
    throw new Error(
      `Missing required environment variables: ${missing.join(', ')}`
    );
  }
}

// Call at startup
validateEnvVars();
```

## Running the Development Server

```bash
# Start Next.js dev server
npm run dev

# Run in production mode
npm run build
npm run start

# Render videos in watch mode
npm run remotion:dev

# Render a specific composition
npm run remotion:render -- ContentVideo out/video.mp4
```

## Best Practices

1. **Cache research data** to avoid redundant API calls
2. **Use webhooks** for long-running video renders
3. **Implement retry logic** for AI API calls
4. **Validate content** before rendering videos
5. **Store rendered videos** in CDN for faster delivery
6. **Monitor API usage** to stay within limits
7. **Test video compositions** before batch rendering
