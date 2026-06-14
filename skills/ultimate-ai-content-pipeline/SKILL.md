---
name: ultimate-ai-content-pipeline
description: Automated content pipeline from research to video generation using Claude, OpenAI, and Remotion
triggers:
  - how do I set up the AI content automation pipeline
  - generate content with automatic research and video creation
  - use the marketing pipeline to create posts and videos
  - automate content from research to video rendering
  - configure Claude and OpenAI for content generation
  - create automated social media content with AI
  - set up the content automation workflow
  - generate videos from AI-written content
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This project is a comprehensive AI-powered content automation system that handles the entire content creation pipeline: from researching trending topics, generating multi-format written content in multiple languages, to automatically rendering videos and graphics. Built with Next.js, TypeScript, Claude/OpenAI APIs, and Remotion.

## What It Does

The Ultimate AI Content Pipeline automates:
- **Auto-Research**: Crawls and analyzes real-time data from TechCrunch, a16z, Twitter/X, LinkedIn
- **Multi-Format Content**: Generates articles in various formats (toplist, POV, case study, how-to)
- **Multilingual**: Creates content in both English and Vietnamese simultaneously
- **Video Generation**: Automatically renders videos and infographics using Remotion
- **Platform Optimization**: Exports content optimized for Reels, TikTok, Shorts

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

```bash
# AI Providers
OPENAI_API_KEY=your_openai_key_here
ANTHROPIC_API_KEY=your_claude_key_here

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key_here
TWITTER_BEARER_TOKEN=your_twitter_token_here

# Database (if applicable)
DATABASE_URL=your_database_url_here

# Remotion (for video rendering)
REMOTION_AWS_ACCESS_KEY_ID=your_aws_key_here
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret_here

# App Configuration
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

## Key Commands

```bash
# Development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Run content research crawler
npm run research

# Generate video renders
npm run render-video

# Export static content
npm run export
```

## Core API Usage

### 1. Research Module

```typescript
// lib/research/crawler.ts
import { ResearchCrawler } from '@/lib/research';

const crawler = new ResearchCrawler({
  sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
  timeframe: '24h',
  keywords: ['AI', 'marketing automation']
});

const researchData = await crawler.scan();
// Returns: { articles: [], insights: [], trending: [] }
```

### 2. Content Generation

```typescript
// lib/content/generator.ts
import { ContentGenerator } from '@/lib/content';

const generator = new ContentGenerator({
  provider: 'claude', // or 'openai'
  model: 'claude-3-opus-20240229',
  apiKey: process.env.ANTHROPIC_API_KEY
});

const content = await generator.create({
  topic: 'AI Marketing Trends 2026',
  format: 'toplist', // toplist | pov | case-study | how-to
  languages: ['en', 'vi'],
  tone: 'professional', // professional | friendly | humorous
  researchData: researchData
});

// Returns:
// {
//   en: { title, content, metadata },
//   vi: { title, content, metadata }
// }
```

### 3. Video Rendering with Remotion

```typescript
// lib/video/render.ts
import { renderVideo } from '@/lib/video';
import { VideoComposition } from '@/remotion/compositions';

const videoConfig = {
  composition: VideoComposition,
  inputProps: {
    title: content.en.title,
    keyPoints: content.en.keyPoints,
    bgColor: '#1a1a1a',
    accentColor: '#00ff88'
  },
  outputFormat: 'mp4',
  dimensions: {
    width: 1080,
    height: 1920 // Vertical for Reels/TikTok/Shorts
  }
};

const videoPath = await renderVideo(videoConfig);
// Returns: path to rendered video file
```

## Complete Workflow Example

```typescript
// pages/api/generate-content.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { ResearchCrawler } from '@/lib/research';
import { ContentGenerator } from '@/lib/content';
import { renderVideo } from '@/lib/video';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  try {
    const { keyword, format, platform } = req.body;

    // Step 1: Research
    const crawler = new ResearchCrawler({
      sources: ['techcrunch', 'a16z', 'twitter'],
      timeframe: '24h',
      keywords: [keyword]
    });
    const research = await crawler.scan();

    // Step 2: Generate Content
    const generator = new ContentGenerator({
      provider: 'claude',
      model: 'claude-3-opus-20240229',
      apiKey: process.env.ANTHROPIC_API_KEY
    });
    
    const content = await generator.create({
      topic: keyword,
      format: format,
      languages: ['en', 'vi'],
      tone: 'professional',
      researchData: research
    });

    // Step 3: Render Video
    const videoDimensions = {
      'reels': { width: 1080, height: 1920 },
      'youtube': { width: 1920, height: 1080 },
      'tiktok': { width: 1080, height: 1920 }
    };

    const video = await renderVideo({
      composition: 'ContentVideo',
      inputProps: {
        title: content.en.title,
        keyPoints: content.en.keyPoints.slice(0, 5),
        bgColor: '#0a0a0a',
        accentColor: '#00d9ff'
      },
      outputFormat: 'mp4',
      dimensions: videoDimensions[platform] || videoDimensions.reels
    });

    res.status(200).json({
      success: true,
      content: content,
      videoPath: video,
      research: research.insights
    });

  } catch (error) {
    console.error('Content generation failed:', error);
    res.status(500).json({ 
      success: false, 
      error: error.message 
    });
  }
}
```

## Frontend Integration

```typescript
// components/ContentPipeline.tsx
import { useState } from 'react';

export default function ContentPipeline() {
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState(null);

  const generateContent = async (keyword: string) => {
    setLoading(true);
    
    const response = await fetch('/api/generate-content', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        keyword: keyword,
        format: 'toplist',
        platform: 'reels'
      })
    });

    const data = await response.json();
    setResult(data);
    setLoading(false);
  };

  return (
    <div className="pipeline-container">
      <input 
        type="text" 
        placeholder="Enter keyword..."
        onKeyPress={(e) => {
          if (e.key === 'Enter') {
            generateContent(e.currentTarget.value);
          }
        }}
      />
      
      {loading && <div>Generating content...</div>}
      
      {result?.success && (
        <div>
          <h2>{result.content.en.title}</h2>
          <div dangerouslySetInnerHTML={{ 
            __html: result.content.en.content 
          }} />
          <video src={result.videoPath} controls />
        </div>
      )}
    </div>
  );
}
```

## Configuration Patterns

### Custom Research Sources

```typescript
// config/research-sources.ts
export const customSources = {
  techcrunch: {
    url: 'https://techcrunch.com/feed/',
    parser: 'rss',
    selector: 'item'
  },
  linkedin: {
    url: 'https://www.linkedin.com/feed/',
    parser: 'api',
    endpoint: process.env.LINKEDIN_API_ENDPOINT
  },
  custom: {
    url: 'https://your-custom-source.com',
    parser: 'html',
    selector: '.article-content'
  }
};
```

### Content Templates

```typescript
// config/content-templates.ts
export const contentTemplates = {
  toplist: {
    structure: [
      'introduction',
      'list-items',
      'conclusion',
      'cta'
    ],
    prompt: 'Create a numbered list of {count} items about {topic}...'
  },
  pov: {
    structure: [
      'hook',
      'perspective',
      'arguments',
      'counterpoints',
      'conclusion'
    ],
    prompt: 'Write a perspective piece on {topic} from the viewpoint of...'
  },
  'case-study': {
    structure: [
      'challenge',
      'approach',
      'results',
      'lessons'
    ],
    prompt: 'Analyze a case study about {topic} including...'
  }
};
```

## Remotion Video Compositions

```typescript
// remotion/compositions/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, interpolate } from 'remotion';

export const ContentVideo: React.FC<{
  title: string;
  keyPoints: string[];
  bgColor: string;
  accentColor: string;
}> = ({ title, keyPoints, bgColor, accentColor }) => {
  const frame = useCurrentFrame();
  
  const opacity = interpolate(
    frame,
    [0, 30],
    [0, 1],
    { extrapolateRight: 'clamp' }
  );

  return (
    <AbsoluteFill style={{ backgroundColor: bgColor }}>
      <div style={{ opacity, padding: 60 }}>
        <h1 style={{ 
          fontSize: 72, 
          color: accentColor,
          marginBottom: 40 
        }}>
          {title}
        </h1>
        
        {keyPoints.map((point, idx) => {
          const pointOpacity = interpolate(
            frame,
            [30 + idx * 30, 60 + idx * 30],
            [0, 1],
            { extrapolateRight: 'clamp' }
          );
          
          return (
            <div key={idx} style={{ 
              opacity: pointOpacity,
              fontSize: 36,
              marginBottom: 20,
              color: '#ffffff'
            }}>
              {idx + 1}. {point}
            </div>
          );
        })}
      </div>
    </AbsoluteFill>
  );
};
```

## Troubleshooting

### API Rate Limits

```typescript
// lib/utils/rate-limiter.ts
import pLimit from 'p-limit';

const limit = pLimit(3); // Max 3 concurrent requests

const rateLimitedRequests = await Promise.all(
  urls.map(url => limit(() => fetch(url)))
);
```

### Video Rendering Timeouts

```typescript
// Increase timeout for large videos
const video = await renderVideo(config, {
  timeout: 300000, // 5 minutes
  concurrency: 2
});
```

### Memory Issues with Large Content

```typescript
// Process content in chunks
const chunkedContent = chunkArray(content, 10);
for (const chunk of chunkedContent) {
  await processChunk(chunk);
}
```

### Claude/OpenAI API Errors

```typescript
// Implement retry logic
import { retry } from '@/lib/utils/retry';

const content = await retry(
  () => generator.create(config),
  { retries: 3, delay: 1000 }
);
```

## Best Practices

1. **Cache research data** to avoid redundant API calls
2. **Use streaming** for long-form content generation
3. **Optimize video rendering** by pre-processing assets
4. **Monitor API quotas** for Claude/OpenAI
5. **Version control** your content templates
6. **A/B test** different content formats and tones
