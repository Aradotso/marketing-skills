---
name: ultimate-ai-content-pipeline
description: TypeScript automation pipeline for AI-powered content research, script writing, and video generation using Claude, OpenAI, and Remotion
triggers:
  - generate content from trending topics automatically
  - research and create marketing scripts with AI
  - automate content pipeline from research to video
  - crawl tech news and generate social media posts
  - create video content from text using Remotion
  - set up AI content automation workflow
  - build automated content marketing system
  - generate multilingual content with Claude and OpenAI
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

The Ultimate AI Content Pipeline is a comprehensive TypeScript-based automation system that handles the entire content creation workflow: from researching trending topics across tech news sources, to generating scripts in multiple formats and languages using Claude/OpenAI, to rendering videos automatically with Remotion. Built with Next.js, it enables marketers and content creators to automate up to 90% of their content production workflow.

## What This Project Does

This pipeline automates four core stages:

1. **Auto-Research**: Crawls recent articles from TechCrunch, a16z, Twitter/X, LinkedIn within the last 24 hours
2. **AI Content Generation**: Creates content in multiple formats (toplist, POV, case study, how-to) in both English and Vietnamese using Claude 3 or OpenAI
3. **Video Rendering**: Automatically generates infographics and short-form videos from written content using Remotion
4. **Multi-Platform Publishing**: Optimizes content for Reels, TikTok, and YouTube Shorts

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

## Configuration

Create a `.env.local` file in the root directory:

```bash
# AI APIs
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Content Sources (optional configurations)
TWITTER_BEARER_TOKEN=your_twitter_token
LINKEDIN_ACCESS_TOKEN=your_linkedin_token

# Remotion (for video rendering)
REMOTION_LICENSE_KEY=your_remotion_license

# Next.js
NEXT_PUBLIC_API_URL=http://localhost:3000
```

## Running the Pipeline

```bash
# Development mode
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Render videos (Remotion)
npm run render
```

## Core API Usage

### 1. Research Module

The research module crawls and aggregates trending content:

```typescript
import { researchTrends } from '@/lib/research';

// Crawl trending topics from multiple sources
const trends = await researchTrends({
  keyword: 'AI automation',
  sources: ['techcrunch', 'a16z', 'twitter'],
  timeframe: '24h',
  limit: 10
});

console.log(trends);
// Returns: Array of {title, url, summary, source, publishedAt, insights}
```

### 2. Content Generation with Claude/OpenAI

Generate content in multiple formats:

```typescript
import { generateContent } from '@/lib/ai/content-generator';

const content = await generateContent({
  keyword: 'AI-powered marketing',
  format: 'toplist', // 'toplist' | 'pov' | 'case-study' | 'how-to'
  language: 'en', // 'en' | 'vi'
  tone: 'professional', // 'professional' | 'friendly' | 'humorous'
  provider: 'claude', // 'claude' | 'openai'
  researchData: trends // Pass research insights
});

console.log(content);
// Returns: {title, body, metadata, keywords, suggestedImages}
```

### 3. Using Claude API Directly

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

const message = await anthropic.messages.create({
  model: 'claude-3-opus-20240229',
  max_tokens: 4096,
  messages: [{
    role: 'user',
    content: `Create a professional marketing blog post about ${keyword} based on this research: ${JSON.stringify(trends)}`
  }],
  system: 'You are an expert content marketing writer specializing in tech and AI trends.'
});

const generatedContent = message.content[0].text;
```

### 4. Using OpenAI API

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

const completion = await openai.chat.completions.create({
  model: 'gpt-4-turbo-preview',
  messages: [
    {
      role: 'system',
      content: 'You are an expert content marketer who creates engaging, data-backed articles.'
    },
    {
      role: 'user',
      content: `Generate a ${format} article about ${keyword} in ${language}`
    }
  ],
  temperature: 0.7,
  max_tokens: 2000
});

const articleContent = completion.choices[0].message.content;
```

### 5. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

// Video composition component
export const VideoComposition: React.FC<{content: string}> = ({content}) => {
  return (
    <AbsoluteFill style={{backgroundColor: 'white'}}>
      <h1>{content}</h1>
    </AbsoluteFill>
  );
};

// Render video from content
async function renderContentVideo(content: any) {
  const bundled = await bundle(
    path.join(process.cwd(), 'src/remotion/index.ts')
  );
  
  const composition = await selectComposition({
    serveUrl: bundled,
    id: 'ContentVideo',
    inputProps: { content }
  });

  await renderMedia({
    composition,
    serveUrl: bundled,
    codec: 'h264',
    outputLocation: `out/${content.id}.mp4`,
    inputProps: { content }
  });
}
```

## Complete Workflow Example

```typescript
import { runFullPipeline } from '@/lib/pipeline';

async function automateContentCreation() {
  try {
    // Full automation: research → write → video
    const result = await runFullPipeline({
      keyword: 'marketing automation trends 2024',
      contentFormat: 'toplist',
      languages: ['en', 'vi'],
      generateVideo: true,
      videoFormat: 'reel', // 'reel' | 'tiktok' | 'short'
      autoPublish: false // Set true to auto-post to platforms
    });

    console.log('Pipeline complete:', result);
    // Returns: {
    //   research: {...},
    //   content: {en: {...}, vi: {...}},
    //   videos: [{url: '...', platform: 'reel'}],
    //   publishStatus: {...}
    // }
  } catch (error) {
    console.error('Pipeline error:', error);
  }
}
```

## API Routes Structure

### Research Endpoint

```typescript
// pages/api/research.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { researchTrends } from '@/lib/research';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  const { keyword, sources, timeframe } = req.body;

  try {
    const trends = await researchTrends({
      keyword,
      sources: sources || ['techcrunch', 'a16z'],
      timeframe: timeframe || '24h'
    });

    res.status(200).json({ success: true, data: trends });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
}
```

### Content Generation Endpoint

```typescript
// pages/api/generate.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { generateContent } from '@/lib/ai/content-generator';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  const { keyword, format, language, tone, provider, researchData } = req.body;

  try {
    const content = await generateContent({
      keyword,
      format,
      language,
      tone,
      provider,
      researchData
    });

    res.status(200).json({ success: true, content });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
}
```

## Common Patterns

### Pattern 1: Multi-Language Content Generation

```typescript
async function generateMultilingualContent(keyword: string, researchData: any) {
  const languages = ['en', 'vi'];
  const contents = {};

  for (const lang of languages) {
    contents[lang] = await generateContent({
      keyword,
      format: 'toplist',
      language: lang,
      tone: 'professional',
      provider: 'claude',
      researchData
    });
  }

  return contents;
}
```

### Pattern 2: Batch Processing Multiple Keywords

```typescript
async function batchProcessKeywords(keywords: string[]) {
  const results = await Promise.all(
    keywords.map(async (keyword) => {
      const research = await researchTrends({ keyword, sources: ['techcrunch'], timeframe: '24h' });
      const content = await generateContent({
        keyword,
        format: 'toplist',
        language: 'en',
        tone: 'professional',
        provider: 'claude',
        researchData: research
      });
      return { keyword, research, content };
    })
  );

  return results;
}
```

### Pattern 3: Content Scheduling

```typescript
import { scheduleContent } from '@/lib/scheduler';

async function scheduleWeeklyContent(keyword: string) {
  const content = await generateContent({
    keyword,
    format: 'how-to',
    language: 'en',
    tone: 'friendly',
    provider: 'openai'
  });

  await scheduleContent({
    content,
    platforms: ['facebook', 'linkedin', 'twitter'],
    scheduleTime: new Date(Date.now() + 24 * 60 * 60 * 1000), // Tomorrow
    autoPublish: true
  });
}
```

### Pattern 4: Custom Video Templates

```typescript
// src/remotion/templates/CustomTemplate.tsx
import { AbsoluteFill, useCurrentFrame, interpolate } from 'remotion';

export const CustomTemplate: React.FC<{title: string, points: string[]}> = ({title, points}) => {
  const frame = useCurrentFrame();
  
  const opacity = interpolate(frame, [0, 30], [0, 1], {
    extrapolateRight: 'clamp',
  });

  return (
    <AbsoluteFill style={{
      backgroundColor: '#1a1a2e',
      padding: 80,
      justifyContent: 'center',
      alignItems: 'center',
    }}>
      <h1 style={{ opacity, color: 'white', fontSize: 72 }}>
        {title}
      </h1>
      <ul style={{ marginTop: 40 }}>
        {points.map((point, i) => (
          <li key={i} style={{ color: 'white', fontSize: 36, marginBottom: 20 }}>
            {point}
          </li>
        ))}
      </ul>
    </AbsoluteFill>
  );
};
```

## Troubleshooting

### Issue: Claude API Rate Limits

```typescript
// Add retry logic with exponential backoff
async function generateWithRetry(params: any, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await generateContent(params);
    } catch (error) {
      if (error.status === 429 && i < maxRetries - 1) {
        await new Promise(resolve => setTimeout(resolve, Math.pow(2, i) * 1000));
        continue;
      }
      throw error;
    }
  }
}
```

### Issue: Research Crawling Fails

```typescript
// Add fallback sources
async function researchWithFallback(keyword: string) {
  const primarySources = ['techcrunch', 'a16z'];
  const fallbackSources = ['reddit', 'producthunt'];
  
  try {
    return await researchTrends({ keyword, sources: primarySources, timeframe: '24h' });
  } catch (error) {
    console.warn('Primary sources failed, trying fallback');
    return await researchTrends({ keyword, sources: fallbackSources, timeframe: '48h' });
  }
}
```

### Issue: Remotion Rendering Memory Issues

```bash
# Increase Node.js memory limit
NODE_OPTIONS="--max-old-space-size=4096" npm run render
```

Or in code:

```typescript
// Render in chunks for large videos
async function renderInChunks(content: any) {
  const chunks = splitContentIntoChunks(content, 30); // 30 second chunks
  
  const renderedChunks = await Promise.all(
    chunks.map((chunk, i) => 
      renderMedia({
        composition: getComposition(chunk),
        outputLocation: `out/chunk-${i}.mp4`,
        // ... other options
      })
    )
  );
  
  // Concatenate chunks using ffmpeg
  await concatenateVideos(renderedChunks, 'out/final.mp4');
}
```

### Issue: Missing Environment Variables

```typescript
// Validate environment on startup
function validateEnvironment() {
  const required = [
    'ANTHROPIC_API_KEY',
    'OPENAI_API_KEY',
    'RAPIDAPI_KEY'
  ];
  
  const missing = required.filter(key => !process.env[key]);
  
  if (missing.length > 0) {
    throw new Error(`Missing required environment variables: ${missing.join(', ')}`);
  }
}

// Call at app initialization
validateEnvironment();
```

## Advanced Configuration

### Custom Content Formats

```typescript
// lib/content-formats.ts
export const contentFormats = {
  toplist: {
    structure: ['intro', 'items', 'conclusion'],
    minItems: 5,
    maxItems: 10,
    prompt: 'Create a numbered list article highlighting the top {count} {topic}'
  },
  pov: {
    structure: ['hook', 'perspective', 'evidence', 'counterpoint', 'conclusion'],
    tone: 'opinionated',
    prompt: 'Share a unique perspective on {topic} with supporting evidence'
  },
  'case-study': {
    structure: ['background', 'challenge', 'solution', 'results', 'lessons'],
    dataRequired: true,
    prompt: 'Analyze a real-world case of {topic} with measurable outcomes'
  }
};
```

This skill equips AI agents with comprehensive knowledge to help developers implement automated content marketing workflows using this TypeScript pipeline.
