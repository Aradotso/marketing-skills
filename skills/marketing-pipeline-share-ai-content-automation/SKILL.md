---
name: marketing-pipeline-share-ai-content-automation
description: Automated AI content pipeline for research, scriptwriting, video generation and publishing with Claude/OpenAI
triggers:
  - how do I automate content creation with AI
  - set up automated content pipeline with Claude
  - generate videos from text automatically
  - create AI content workflow for marketing
  - automate research and scriptwriting pipeline
  - build content automation with Remotion video
  - set up AI marketing content generator
  - crawl news and generate content automatically
---

# Marketing Pipeline Share AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

Marketing Pipeline Share is an end-to-end AI content automation system that handles the complete content lifecycle: from research and data crawling, to scriptwriting in multiple formats, to automated video generation. Built with TypeScript/Next.js, it integrates Claude 3, OpenAI, and Remotion for a complete content production pipeline.

The system automatically:
- Crawls fresh data from TechCrunch, a16z, X (Twitter), LinkedIn
- Generates content in multiple formats (toplist, POV, case study, how-to)
- Produces bilingual content (English & Vietnamese)
- Renders videos and infographics via Remotion
- Optimizes output for multiple platforms (Reels, TikTok, Shorts)

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

# Set up environment variables
cp .env.example .env.local
```

## Configuration

Create a `.env.local` file with the following variables:

```bash
# AI Provider Keys
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research & Data Crawling
RAPIDAPI_KEY=your_rapidapi_key

# Next.js Configuration
NEXT_PUBLIC_API_URL=http://localhost:3000

# Remotion Configuration (for video rendering)
REMOTION_AWS_ACCESS_KEY_ID=your_aws_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── crawler/     # News crawling logic
│   │   ├── content/     # Content generation
│   │   └── video/       # Remotion video generation
│   └── types/           # TypeScript types
├── public/              # Static assets
└── remotion/            # Video templates
```

## Core APIs and Usage

### 1. Content Research & Crawling

```typescript
import { crawlLatestNews } from '@/lib/crawler/newsCrawler';
import { analyzeResearch } from '@/lib/ai/research';

// Crawl news from multiple sources
async function gatherResearch(keyword: string) {
  const sources = ['techcrunch', 'a16z', 'twitter', 'linkedin'];
  
  const newsData = await crawlLatestNews({
    keyword,
    sources,
    timeframe: '24h',
    limit: 20
  });
  
  // Analyze with AI to extract insights
  const insights = await analyzeResearch({
    rawData: newsData,
    provider: 'claude', // or 'openai'
    model: 'claude-3-opus-20240229'
  });
  
  return insights;
}
```

### 2. Content Generation

```typescript
import { generateContent } from '@/lib/content/generator';
import { ContentFormat, ToneOfVoice } from '@/types/content';

// Generate multi-format content
async function createContent(topic: string, research: any) {
  const formats: ContentFormat[] = ['toplist', 'how-to', 'case-study', 'pov'];
  const languages = ['en', 'vi'];
  
  const content = await generateContent({
    topic,
    research,
    formats,
    languages,
    tone: 'professional' as ToneOfVoice,
    aiProvider: 'claude',
    options: {
      includeDataPoints: true,
      generateImages: true,
      seoOptimized: true
    }
  });
  
  return content;
}
```

### 3. Video Generation with Remotion

```typescript
import { renderVideo } from '@/lib/video/renderer';
import { VideoTemplate } from '@/types/video';

// Render video from content
async function generateVideo(content: any) {
  const videoConfig = {
    template: 'infographic' as VideoTemplate,
    content: {
      title: content.title,
      keyPoints: content.keyPoints,
      data: content.statistics
    },
    format: {
      platform: 'reels', // or 'tiktok', 'youtube-shorts'
      aspectRatio: '9:16',
      duration: 30 // seconds
    },
    styling: {
      theme: 'modern',
      brandColors: ['#FF6B6B', '#4ECDC4'],
      fontFamily: 'Inter'
    }
  };
  
  const video = await renderVideo(videoConfig);
  return video.url;
}
```

### 4. Complete Pipeline

```typescript
import { runContentPipeline } from '@/lib/pipeline';

// Full automation pipeline
async function automateContentCreation() {
  const pipeline = await runContentPipeline({
    keyword: 'AI automation',
    steps: [
      'research',
      'content-generation',
      'video-render',
      'publish'
    ],
    config: {
      research: {
        sources: ['techcrunch', 'a16z', 'twitter'],
        timeframe: '24h'
      },
      content: {
        formats: ['toplist', 'how-to'],
        languages: ['en', 'vi'],
        tone: 'expert'
      },
      video: {
        platforms: ['reels', 'tiktok'],
        autoPublish: false
      }
    }
  });
  
  return pipeline.results;
}
```

## API Routes

### Research Endpoint

```typescript
// pages/api/research.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { crawlLatestNews } from '@/lib/crawler/newsCrawler';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }
  
  const { keyword, sources, timeframe } = req.body;
  
  try {
    const research = await crawlLatestNews({
      keyword,
      sources: sources || ['techcrunch', 'a16z'],
      timeframe: timeframe || '24h',
      limit: 20
    });
    
    res.status(200).json({ success: true, data: research });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
}
```

### Content Generation Endpoint

```typescript
// pages/api/generate.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { generateContent } from '@/lib/content/generator';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  const { topic, research, formats, languages, tone } = req.body;
  
  try {
    const content = await generateContent({
      topic,
      research,
      formats: formats || ['toplist'],
      languages: languages || ['en'],
      tone: tone || 'professional',
      aiProvider: process.env.AI_PROVIDER || 'claude'
    });
    
    res.status(200).json({ success: true, content });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
}
```

## Common Patterns

### Custom AI Provider Configuration

```typescript
import { Anthropic } from '@anthropic-ai/sdk';
import OpenAI from 'openai';

// Configure Claude
export const claudeClient = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY
});

// Configure OpenAI
export const openaiClient = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY
});

// Unified AI interface
export async function callAI(prompt: string, provider: 'claude' | 'openai') {
  if (provider === 'claude') {
    const response = await claudeClient.messages.create({
      model: 'claude-3-opus-20240229',
      max_tokens: 4096,
      messages: [{ role: 'user', content: prompt }]
    });
    return response.content[0].text;
  } else {
    const response = await openaiClient.chat.completions.create({
      model: 'gpt-4-turbo-preview',
      messages: [{ role: 'user', content: prompt }]
    });
    return response.choices[0].message.content;
  }
}
```

### Content Format Templates

```typescript
// lib/content/templates.ts
export const contentTemplates = {
  toplist: {
    structure: [
      'introduction',
      'items',
      'conclusion'
    ],
    prompt: `Create a toplist article about {topic} based on this research: {research}.
    Include 5-10 items with data-backed insights.`
  },
  
  howTo: {
    structure: [
      'problem',
      'steps',
      'examples',
      'conclusion'
    ],
    prompt: `Write a step-by-step guide about {topic} using this research: {research}.
    Make it actionable and include real examples.`
  },
  
  caseStudy: {
    structure: [
      'background',
      'challenge',
      'solution',
      'results',
      'lessons'
    ],
    prompt: `Create a case study about {topic} based on: {research}.
    Focus on real data and measurable outcomes.`
  }
};
```

### Video Template Configuration

```typescript
// remotion/templates/infographic.tsx
import { AbsoluteFill, useCurrentFrame, interpolate } from 'remotion';

export const InfographicTemplate: React.FC<{
  title: string;
  data: any[];
}> = ({ title, data }) => {
  const frame = useCurrentFrame();
  
  const opacity = interpolate(frame, [0, 30], [0, 1]);
  
  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a1a' }}>
      <div style={{ opacity, padding: 40 }}>
        <h1 style={{ color: 'white', fontSize: 48 }}>{title}</h1>
        {data.map((item, i) => (
          <div key={i} style={{ marginTop: 20, color: 'white' }}>
            <h3>{item.label}</h3>
            <p>{item.value}</p>
          </div>
        ))}
      </div>
    </AbsoluteFill>
  );
};
```

## Running the Pipeline

### Development Mode

```bash
# Start the Next.js development server
npm run dev

# In another terminal, start Remotion preview (for video editing)
npm run remotion:preview
```

### Production Build

```bash
# Build the Next.js application
npm run build

# Start production server
npm start

# Render videos in production
npm run remotion:render
```

## Troubleshooting

### API Rate Limits

```typescript
// lib/utils/rateLimit.ts
export async function withRateLimit<T>(
  fn: () => Promise<T>,
  retries = 3,
  delay = 1000
): Promise<T> {
  try {
    return await fn();
  } catch (error) {
    if (error.status === 429 && retries > 0) {
      await new Promise(resolve => setTimeout(resolve, delay));
      return withRateLimit(fn, retries - 1, delay * 2);
    }
    throw error;
  }
}
```

### Handling Failed Video Renders

```typescript
// lib/video/errorHandling.ts
export async function safeRenderVideo(config: any) {
  try {
    return await renderVideo(config);
  } catch (error) {
    console.error('Video render failed:', error);
    
    // Fallback to image generation
    return await generateStaticImage(config.content);
  }
}
```

### Content Quality Validation

```typescript
// lib/content/validation.ts
export function validateContent(content: any): boolean {
  const checks = [
    content.title && content.title.length > 10,
    content.body && content.body.length > 200,
    content.keyPoints && content.keyPoints.length >= 3,
    !content.body.includes('[TODO]'),
    !content.body.includes('Lorem ipsum')
  ];
  
  return checks.every(Boolean);
}
```

### Debug Mode

```typescript
// Enable detailed logging
export const DEBUG_MODE = process.env.DEBUG === 'true';

export function debugLog(message: string, data?: any) {
  if (DEBUG_MODE) {
    console.log(`[DEBUG] ${message}`, data || '');
  }
}
```

## Best Practices

1. **Always validate API keys** before running the pipeline
2. **Use rate limiting** for external API calls
3. **Cache research data** to avoid redundant crawling
4. **Validate content quality** before video rendering
5. **Monitor API costs** for Claude/OpenAI usage
6. **Test video templates** in Remotion preview before production renders
7. **Use environment-specific configs** for different deployment stages
