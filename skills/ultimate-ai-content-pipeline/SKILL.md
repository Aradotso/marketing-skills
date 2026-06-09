---
name: ultimate-ai-content-pipeline
description: Automated content pipeline with AI research, scriptwriting, auto-posting and video generation using Claude, OpenAI and Remotion
triggers:
  - how do I automate content creation with AI research and video generation
  - set up automated content pipeline with Claude and Remotion
  - create content automation from research to video publishing
  - build AI content workflow with auto posting to social media
  - generate videos automatically from AI written content
  - implement end-to-end content automation with OpenAI
  - automate content research and video creation pipeline
  - set up marketing content automation with AI and Remotion
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

An end-to-end content automation system that handles research, scriptwriting, publishing, and video generation. The pipeline crawls news from TechCrunch, a16z, X (Twitter), and LinkedIn, generates content in multiple formats using Claude/OpenAI, and renders videos using Remotion.

## What This Project Does

Ultimate AI Content Pipeline automates the entire content creation workflow:

1. **Auto-Research**: Crawls real-time data from news sources (last 24h)
2. **AI Content Generation**: Creates content in multiple formats (Toplist, POV, Case Study, How-to) using Claude 3 or OpenAI
3. **Multi-language Support**: Generates content in English and Vietnamese simultaneously
4. **Video Rendering**: Automatically converts content to videos/infographics using Remotion
5. **Auto-Publishing**: Schedules and posts content to various platforms

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
cp .env.example .env
```

## Configuration

Create a `.env` file with the following variables:

```bash
# AI Providers
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key
TWITTER_BEARER_TOKEN=your_twitter_bearer_token

# Database (if using)
DATABASE_URL=postgresql://user:password@localhost:5432/content_pipeline

# Remotion Configuration
REMOTION_AWS_ACCESS_KEY_ID=your_aws_access_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret_key

# Content Configuration
DEFAULT_LANGUAGE=vi
CONTENT_TONE=professional # or friendly, humorous
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── research/          # Web scraping & data collection
│   ├── content/           # AI content generation
│   ├── video/             # Remotion video rendering
│   ├── publisher/         # Auto-posting logic
│   └── api/               # API routes
├── remotion/              # Remotion video templates
├── public/                # Static assets
└── pages/                 # Next.js pages
```

## Key Usage Patterns

### 1. Research & Data Collection

```typescript
import { researchTopic } from './src/research/crawler';

// Crawl latest news on a topic
const researchData = await researchTopic({
  keyword: 'AI automation',
  sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
  timeRange: '24h',
  maxResults: 50
});

console.log(researchData);
// {
//   articles: [...],
//   insights: [...],
//   statistics: {...},
//   trendingTopics: [...]
// }
```

### 2. AI Content Generation with Claude

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

async function generateContent(research: any, format: string) {
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [
      {
        role: 'user',
        content: `Generate a ${format} article based on this research data:
        
${JSON.stringify(research, null, 2)}

Requirements:
- Tone: Professional but engaging
- Include statistics and data points
- Optimize for SEO
- Add actionable insights`
      }
    ],
  });

  return message.content[0].text;
}

// Generate different content formats
const toplistArticle = await generateContent(researchData, 'toplist');
const caseStudy = await generateContent(researchData, 'case-study');
const howTo = await generateContent(researchData, 'how-to');
```

### 3. Multi-language Content Generation

```typescript
interface ContentGenerationOptions {
  research: any;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  languages: string[];
  tone: 'professional' | 'friendly' | 'humorous';
}

async function generateMultilingualContent(options: ContentGenerationOptions) {
  const { research, format, languages, tone } = options;
  const results: Record<string, string> = {};

  for (const lang of languages) {
    const prompt = `Generate a ${format} article in ${lang === 'vi' ? 'Vietnamese' : 'English'}.
    
Tone: ${tone}
Research data: ${JSON.stringify(research)}

Format requirements:
- Engaging headline
- Clear structure with headers
- Data-backed insights
- Call-to-action at the end`;

    const message = await anthropic.messages.create({
      model: 'claude-3-5-sonnet-20241022',
      max_tokens: 4096,
      messages: [{ role: 'user', content: prompt }],
    });

    results[lang] = message.content[0].text;
  }

  return results;
}

// Usage
const content = await generateMultilingualContent({
  research: researchData,
  format: 'toplist',
  languages: ['en', 'vi'],
  tone: 'professional'
});
```

### 4. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import { webpackOverride } from './remotion/webpack-override';
import path from 'path';

interface VideoConfig {
  title: string;
  content: string;
  insights: string[];
  stats: Array<{ label: string; value: string }>;
}

async function generateVideo(config: VideoConfig) {
  const compositionId = 'ContentVideo';
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride,
  });

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: config,
  });

  const outputPath = `./output/video-${Date.now()}.mp4`;

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: config,
  });

  return outputPath;
}

// Generate video from content
const videoPath = await generateVideo({
  title: content.en.split('\n')[0], // First line as title
  content: content.en,
  insights: researchData.insights.slice(0, 3),
  stats: researchData.statistics
});
```

### 5. Remotion Video Template

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, interpolate, Sequence } from 'remotion';
import React from 'react';

interface ContentVideoProps {
  title: string;
  content: string;
  insights: string[];
  stats: Array<{ label: string; value: string }>;
}

export const ContentVideo: React.FC<ContentVideoProps> = ({
  title,
  insights,
  stats,
}) => {
  const frame = useCurrentFrame();

  const titleOpacity = interpolate(frame, [0, 30], [0, 1], {
    extrapolateRight: 'clamp',
  });

  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a2e' }}>
      <Sequence from={0} durationInFrames={90}>
        <AbsoluteFill style={{ 
          justifyContent: 'center', 
          alignItems: 'center',
          opacity: titleOpacity 
        }}>
          <h1 style={{ color: 'white', fontSize: 72, textAlign: 'center' }}>
            {title}
          </h1>
        </AbsoluteFill>
      </Sequence>

      <Sequence from={90} durationInFrames={150}>
        <AbsoluteFill style={{ padding: 60 }}>
          <h2 style={{ color: '#00d4ff', marginBottom: 30 }}>Key Insights:</h2>
          {insights.map((insight, i) => (
            <p key={i} style={{ color: 'white', fontSize: 32, marginBottom: 20 }}>
              • {insight}
            </p>
          ))}
        </AbsoluteFill>
      </Sequence>

      <Sequence from={240} durationInFrames={150}>
        <AbsoluteFill style={{ padding: 60 }}>
          <h2 style={{ color: '#00d4ff', marginBottom: 30 }}>Statistics:</h2>
          {stats.map((stat, i) => (
            <div key={i} style={{ marginBottom: 30 }}>
              <p style={{ color: '#aaa', fontSize: 24 }}>{stat.label}</p>
              <p style={{ color: 'white', fontSize: 48, fontWeight: 'bold' }}>
                {stat.value}
              </p>
            </div>
          ))}
        </AbsoluteFill>
      </Sequence>
    </AbsoluteFill>
  );
};
```

### 6. Complete Pipeline Execution

```typescript
import { researchTopic } from './src/research/crawler';
import { generateMultilingualContent } from './src/content/generator';
import { generateVideo } from './src/video/renderer';
import { publishContent } from './src/publisher/scheduler';

async function runContentPipeline(keyword: string) {
  console.log(`Starting pipeline for keyword: ${keyword}`);

  // Step 1: Research
  console.log('Step 1: Researching topic...');
  const research = await researchTopic({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeRange: '24h',
    maxResults: 50
  });

  // Step 2: Generate Content
  console.log('Step 2: Generating content...');
  const content = await generateMultilingualContent({
    research,
    format: 'toplist',
    languages: ['en', 'vi'],
    tone: 'professional'
  });

  // Step 3: Generate Video
  console.log('Step 3: Rendering video...');
  const videoPath = await generateVideo({
    title: content.en.split('\n')[0],
    content: content.en,
    insights: research.insights.slice(0, 3),
    stats: research.statistics
  });

  // Step 4: Publish
  console.log('Step 4: Publishing content...');
  await publishContent({
    content: content.en,
    contentVi: content.vi,
    videoPath,
    platforms: ['facebook', 'linkedin', 'twitter'],
    scheduleTime: new Date(Date.now() + 3600000) // 1 hour from now
  });

  console.log('Pipeline completed successfully!');
}

// Execute pipeline
runContentPipeline('AI automation trends 2024');
```

### 7. API Route Example (Next.js)

```typescript
// pages/api/generate-content.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { runContentPipeline } from '../../src/pipeline';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  const { keyword, format, languages, platforms } = req.body;

  if (!keyword) {
    return res.status(400).json({ error: 'Keyword is required' });
  }

  try {
    const result = await runContentPipeline(keyword);
    
    res.status(200).json({
      success: true,
      data: result
    });
  } catch (error) {
    console.error('Pipeline error:', error);
    res.status(500).json({ 
      error: 'Pipeline execution failed',
      details: error.message 
    });
  }
}
```

## Common Workflows

### Quick Content Generation

```typescript
// Generate content only (no video)
import { quickGenerate } from './src/content/quick';

const article = await quickGenerate({
  topic: 'Latest AI trends',
  format: 'toplist',
  language: 'vi'
});
```

### Scheduled Publishing

```typescript
// Schedule multiple posts
import { scheduleMultiplePosts } from './src/publisher/scheduler';

await scheduleMultiplePosts([
  {
    content: content.en,
    platform: 'linkedin',
    scheduleTime: new Date('2024-01-15T10:00:00')
  },
  {
    content: content.vi,
    platform: 'facebook',
    scheduleTime: new Date('2024-01-15T14:00:00')
  }
]);
```

### Batch Video Generation

```typescript
// Generate multiple video formats
const videos = await Promise.all([
  generateVideo({ ...config, aspectRatio: '16:9' }), // YouTube
  generateVideo({ ...config, aspectRatio: '9:16' }), // TikTok/Reels
  generateVideo({ ...config, aspectRatio: '1:1' }),  // Instagram
]);
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement retry logic with exponential backoff
async function retryWithBackoff(fn: () => Promise<any>, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (error.status === 429 && i < maxRetries - 1) {
        const delay = Math.pow(2, i) * 1000;
        console.log(`Rate limited. Retrying in ${delay}ms...`);
        await new Promise(resolve => setTimeout(resolve, delay));
      } else {
        throw error;
      }
    }
  }
}
```

### Video Rendering Issues

```typescript
// Check Remotion configuration
import { getCompositions } from '@remotion/renderer';

async function validateVideoSetup() {
  try {
    const compositions = await getCompositions(bundleLocation);
    console.log('Available compositions:', compositions);
  } catch (error) {
    console.error('Remotion setup error:', error);
    // Check: Node version, FFmpeg installation, disk space
  }
}
```

### Memory Issues with Large Research

```typescript
// Stream large datasets instead of loading all at once
async function streamResearch(keyword: string) {
  const stream = createResearchStream(keyword);
  
  for await (const chunk of stream) {
    // Process chunk by chunk
    await processResearchChunk(chunk);
  }
}
```

### Debug Mode

```typescript
// Enable detailed logging
process.env.DEBUG = 'content-pipeline:*';

// Or use custom logger
import debug from 'debug';
const log = debug('content-pipeline:main');

log('Starting pipeline with config:', config);
```

## Performance Optimization

### Cache Research Results

```typescript
import { cacheResearch, getCachedResearch } from './src/cache';

async function optimizedResearch(keyword: string) {
  const cached = await getCachedResearch(keyword);
  if (cached && Date.now() - cached.timestamp < 3600000) {
    return cached.data;
  }
  
  const fresh = await researchTopic({ keyword });
  await cacheResearch(keyword, fresh);
  return fresh;
}
```

### Parallel Processing

```typescript
// Generate content and video in parallel
const [content, video] = await Promise.all([
  generateContent(research),
  generateVideo(research)
]);
```

This skill enables AI agents to help developers implement automated content pipelines with AI-powered research, multi-format content generation, video rendering, and auto-publishing capabilities.
