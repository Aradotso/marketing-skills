---
name: marketing-pipeline-share-automation
description: Automated AI content pipeline for research, scriptwriting, Facebook posting, and video generation using Claude/OpenAI and Remotion
triggers:
  - how do I automate content creation with AI
  - set up automated marketing pipeline with video generation
  - create AI-powered content workflow from research to video
  - integrate Claude and OpenAI for automated content generation
  - build automated social media posting pipeline
  - generate videos automatically from AI-written content
  - use Remotion to render marketing videos programmatically
  - automate content research and scriptwriting workflow
---

# Marketing Pipeline Share Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

Marketing Pipeline Share is an end-to-end automated content creation system built with TypeScript/Next.js. It automates the entire content lifecycle: web scraping for research, AI-powered scriptwriting (Claude 3/OpenAI), automatic Facebook page posting, and video generation using Remotion.

The pipeline crawls news sources (TechCrunch, a16z, Twitter/X, LinkedIn), extracts insights, generates multi-format content (toplist, POV, case study, how-to) in multiple languages, and renders videos/infographics optimized for Reels, TikTok, and Shorts.

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

### Required Environment Variables

```bash
# AI Services
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research/Scraping APIs
RAPIDAPI_KEY=your_rapidapi_key

# Facebook Integration (optional)
FACEBOOK_PAGE_ACCESS_TOKEN=your_facebook_token
FACEBOOK_PAGE_ID=your_page_id

# Remotion (for video rendering)
REMOTION_GITHUB_TOKEN=your_github_token
```

## Core Architecture

The pipeline consists of four main stages:

1. **Research Stage**: Automated web scraping and data collection
2. **Content Generation**: AI-powered scriptwriting with Claude/OpenAI
3. **Publishing**: Automatic Facebook page posting
4. **Video Rendering**: Remotion-based video generation

## Key Components

### 1. Research & Data Collection

```typescript
// lib/research/scraper.ts
import axios from 'axios';

interface ResearchSource {
  url: string;
  type: 'tech' | 'social' | 'news';
  timeframe: '24h' | '7d';
}

export async function scrapeResearch(keyword: string, sources: ResearchSource[]) {
  const results = await Promise.all(
    sources.map(async (source) => {
      const response = await axios.get(source.url, {
        params: { q: keyword, timeframe: source.timeframe },
        headers: {
          'X-RapidAPI-Key': process.env.RAPIDAPI_KEY,
        },
      });
      return parseResearchData(response.data, source.type);
    })
  );
  
  return aggregateInsights(results);
}

function parseResearchData(data: any, sourceType: string) {
  // Extract relevant insights, headlines, and data points
  return {
    headlines: data.articles?.map((a: any) => a.title) || [],
    insights: extractInsights(data),
    metrics: extractMetrics(data),
    sourceType,
  };
}
```

### 2. AI Content Generation

```typescript
// lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

interface ContentConfig {
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  targetAudience: string;
}

export async function generateContent(
  researchData: any,
  config: ContentConfig,
  provider: 'claude' | 'openai' = 'claude'
) {
  const prompt = buildPrompt(researchData, config);
  
  if (provider === 'claude') {
    const message = await anthropic.messages.create({
      model: 'claude-3-5-sonnet-20241022',
      max_tokens: 4096,
      messages: [{
        role: 'user',
        content: prompt,
      }],
    });
    
    return parseAIResponse(message.content);
  } else {
    const completion = await openai.chat.completions.create({
      model: 'gpt-4-turbo-preview',
      messages: [{
        role: 'user',
        content: prompt,
      }],
      max_tokens: 4096,
    });
    
    return parseAIResponse(completion.choices[0].message.content);
  }
}

function buildPrompt(researchData: any, config: ContentConfig): string {
  const basePrompt = `Create a ${config.format} article in ${config.language} with a ${config.tone} tone.
Target audience: ${config.targetAudience}

Research data:
${JSON.stringify(researchData, null, 2)}

Requirements:
- Use real data and insights from the research
- Include specific metrics and examples
- Optimize for ${config.format} format
- Make it engaging and actionable`;

  return basePrompt;
}
```

### 3. Content Structure & Formatting

```typescript
// lib/content/formatter.ts
interface ContentOutput {
  title: string;
  subtitle?: string;
  body: string;
  sections: ContentSection[];
  callToAction: string;
  hashtags: string[];
  videoScript?: VideoScript;
}

interface ContentSection {
  heading: string;
  content: string;
  visualData?: any; // For infographic generation
}

interface VideoScript {
  scenes: Array<{
    duration: number;
    text: string;
    visualType: 'text' | 'image' | 'chart';
    animation: string;
  }>;
  totalDuration: number;
}

export function formatContent(rawContent: string, format: string): ContentOutput {
  // Parse AI-generated content into structured format
  const parsed = parseMarkdown(rawContent);
  
  return {
    title: parsed.title,
    subtitle: parsed.subtitle,
    body: parsed.body,
    sections: extractSections(parsed),
    callToAction: generateCTA(format),
    hashtags: extractHashtags(parsed),
    videoScript: generateVideoScript(parsed),
  };
}
```

### 4. Facebook Auto-Posting

```typescript
// lib/social/facebook-publisher.ts
import axios from 'axios';

interface FacebookPostConfig {
  pageId: string;
  accessToken: string;
  content: ContentOutput;
  scheduleTime?: Date;
}

export async function publishToFacebook(config: FacebookPostConfig) {
  const { pageId, accessToken, content, scheduleTime } = config;
  
  const postData = {
    message: formatPostMessage(content),
    link: content.externalLink,
    scheduled_publish_time: scheduleTime 
      ? Math.floor(scheduleTime.getTime() / 1000) 
      : undefined,
    published: !scheduleTime,
  };
  
  const response = await axios.post(
    `https://graph.facebook.com/v18.0/${pageId}/feed`,
    postData,
    {
      params: { access_token: accessToken },
    }
  );
  
  return response.data;
}

function formatPostMessage(content: ContentOutput): string {
  return `${content.title}

${content.subtitle || ''}

${content.body}

${content.callToAction}

${content.hashtags.join(' ')}`;
}
```

### 5. Video Generation with Remotion

```typescript
// lib/video/remotion-renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  script: VideoScript;
  style: 'modern' | 'minimal' | 'bold';
  aspectRatio: '16:9' | '9:16' | '1:1';
  platform: 'reels' | 'tiktok' | 'youtube-shorts';
}

export async function renderVideo(config: VideoConfig) {
  // Bundle the Remotion project
  const bundleLocation = await bundle(
    path.join(process.cwd(), 'src/remotion/index.ts')
  );
  
  // Select composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      script: config.script,
      style: config.style,
      aspectRatio: config.aspectRatio,
    },
  });
  
  // Render video
  const outputLocation = path.join(
    process.cwd(),
    'output',
    `video-${Date.now()}.mp4`
  );
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
  });
  
  return outputLocation;
}
```

### 6. Remotion Video Component

```typescript
// src/remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, interpolate, Sequence } from 'remotion';
import React from 'react';

interface ContentVideoProps {
  script: VideoScript;
  style: string;
}

export const ContentVideo: React.FC<ContentVideoProps> = ({ script, style }) => {
  return (
    <AbsoluteFill style={{ backgroundColor: '#000' }}>
      {script.scenes.map((scene, index) => (
        <Sequence
          key={index}
          from={index * scene.duration * 30} // 30 fps
          durationInFrames={scene.duration * 30}
        >
          <SceneComponent scene={scene} style={style} />
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};

const SceneComponent: React.FC<{ scene: any; style: string }> = ({ scene, style }) => {
  const frame = useCurrentFrame();
  const opacity = interpolate(frame, [0, 10], [0, 1], { extrapolateRight: 'clamp' });
  
  return (
    <AbsoluteFill style={{ opacity }}>
      <div style={getStyleConfig(style)}>
        <h2>{scene.text}</h2>
        {scene.visualData && <Infographic data={scene.visualData} />}
      </div>
    </AbsoluteFill>
  );
};
```

## Complete Pipeline Workflow

```typescript
// lib/pipeline/orchestrator.ts
export async function runContentPipeline(keyword: string, config: PipelineConfig) {
  console.log('Starting content pipeline for:', keyword);
  
  // Stage 1: Research
  const researchData = await scrapeResearch(keyword, config.sources);
  console.log('Research completed:', researchData.insights.length, 'insights');
  
  // Stage 2: Generate Content
  const rawContent = await generateContent(researchData, {
    format: config.contentFormat,
    language: config.language,
    tone: config.tone,
    targetAudience: config.audience,
  }, config.aiProvider);
  
  const formattedContent = formatContent(rawContent, config.contentFormat);
  console.log('Content generated:', formattedContent.title);
  
  // Stage 3: Publish to Facebook (optional)
  if (config.autoPublish && config.facebookConfig) {
    const postResult = await publishToFacebook({
      pageId: process.env.FACEBOOK_PAGE_ID!,
      accessToken: process.env.FACEBOOK_PAGE_ACCESS_TOKEN!,
      content: formattedContent,
      scheduleTime: config.scheduleTime,
    });
    console.log('Published to Facebook:', postResult.id);
  }
  
  // Stage 4: Render Video
  if (config.generateVideo && formattedContent.videoScript) {
    const videoPath = await renderVideo({
      script: formattedContent.videoScript,
      style: config.videoStyle,
      aspectRatio: config.aspectRatio,
      platform: config.platform,
    });
    console.log('Video rendered:', videoPath);
    
    return {
      content: formattedContent,
      video: videoPath,
      facebookPost: postResult?.id,
    };
  }
  
  return {
    content: formattedContent,
    facebookPost: postResult?.id,
  };
}
```

## Next.js API Routes

```typescript
// pages/api/generate-content.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { runContentPipeline } from '@/lib/pipeline/orchestrator';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }
  
  try {
    const { keyword, config } = req.body;
    
    const result = await runContentPipeline(keyword, {
      sources: config.sources || [],
      contentFormat: config.format || 'toplist',
      language: config.language || 'vi',
      tone: config.tone || 'expert',
      audience: config.audience || 'marketers',
      aiProvider: config.aiProvider || 'claude',
      autoPublish: config.autoPublish || false,
      generateVideo: config.generateVideo || false,
      videoStyle: config.videoStyle || 'modern',
      aspectRatio: config.aspectRatio || '9:16',
      platform: config.platform || 'reels',
    });
    
    res.status(200).json(result);
  } catch (error) {
    console.error('Pipeline error:', error);
    res.status(500).json({ error: 'Pipeline execution failed' });
  }
}
```

## Usage Examples

### Basic Content Generation

```typescript
import { runContentPipeline } from './lib/pipeline/orchestrator';

const result = await runContentPipeline('AI Marketing Trends 2024', {
  sources: [
    { url: 'techcrunch.com', type: 'tech', timeframe: '24h' },
    { url: 'linkedin.com', type: 'social', timeframe: '7d' },
  ],
  contentFormat: 'toplist',
  language: 'vi',
  tone: 'expert',
  audience: 'digital marketers',
  aiProvider: 'claude',
  autoPublish: false,
  generateVideo: true,
  videoStyle: 'modern',
  aspectRatio: '9:16',
  platform: 'reels',
});

console.log('Content:', result.content.title);
console.log('Video:', result.video);
```

### Scheduled Publishing

```typescript
const scheduleTime = new Date();
scheduleTime.setHours(scheduleTime.getHours() + 2); // Publish in 2 hours

await runContentPipeline('Content Marketing Tips', {
  contentFormat: 'how-to',
  language: 'en',
  autoPublish: true,
  scheduleTime,
  facebookConfig: {
    pageId: process.env.FACEBOOK_PAGE_ID!,
    accessToken: process.env.FACEBOOK_PAGE_ACCESS_TOKEN!,
  },
});
```

## Common Patterns

### Multi-Language Content Generation

```typescript
const languages = ['en', 'vi'];
const results = await Promise.all(
  languages.map(lang => 
    runContentPipeline(keyword, {
      ...baseConfig,
      language: lang,
    })
  )
);
```

### Batch Video Rendering

```typescript
const videoConfigs = [
  { aspectRatio: '16:9', platform: 'youtube-shorts' },
  { aspectRatio: '9:16', platform: 'reels' },
  { aspectRatio: '1:1', platform: 'tiktok' },
];

for (const videoConfig of videoConfigs) {
  await renderVideo({
    script: content.videoScript,
    ...videoConfig,
  });
}
```

## Troubleshooting

### API Rate Limits

If you encounter rate limits with Claude or OpenAI:

```typescript
import pRetry from 'p-retry';

const generateWithRetry = pRetry(
  () => generateContent(researchData, config),
  {
    retries: 3,
    onFailedAttempt: error => {
      console.log(`Attempt ${error.attemptNumber} failed. Retrying...`);
    },
  }
);
```

### Remotion Rendering Issues

Ensure you have the correct codecs installed:

```bash
# Install FFmpeg
brew install ffmpeg  # macOS
apt-get install ffmpeg  # Ubuntu/Debian
```

### Memory Issues During Video Rendering

```typescript
// Increase Node memory limit
// package.json
{
  "scripts": {
    "render": "NODE_OPTIONS='--max-old-space-size=4096' node render-script.js"
  }
}
```

### Facebook API Errors

Verify your access token has the correct permissions:
- `pages_manage_posts`
- `pages_read_engagement`
- `publish_video` (if posting videos)

## Development

```bash
# Run development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Render videos (Remotion)
npm run remotion:render
```

## Best Practices

1. **Rate Limiting**: Implement exponential backoff for API calls
2. **Caching**: Cache research data to reduce API costs
3. **Error Handling**: Log all pipeline stages for debugging
4. **Content Review**: Add a review step before auto-publishing
5. **Video Optimization**: Compress videos for faster uploads
6. **Token Management**: Monitor AI token usage to control costs
