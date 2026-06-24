---
name: ultimate-ai-content-pipeline
description: Automated content pipeline for research, scriptwriting, auto-posting, and video generation using Claude, OpenAI, and Remotion
triggers:
  - how do I set up the AI content pipeline
  - create automated content with research and video
  - generate content from research to video automatically
  - use the marketing content automation pipeline
  - set up remotion video generation with AI
  - automate content research and social media posting
  - build an AI-powered content workflow
  - integrate Claude and OpenAI for content generation
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This TypeScript project is an end-to-end AI-powered content automation system that handles research (crawling news sources), scriptwriting (using Claude/OpenAI), automatic social media posting, and video generation (using Remotion). It enables content creators and marketers to automate up to 90% of their content production workflow.

## What It Does

- **Auto-Research**: Crawls real-time data from TechCrunch, a16z, Twitter/X, LinkedIn within the last 24 hours
- **AI Content Generation**: Creates articles in multiple formats (listicles, POV, case studies, how-tos) using Claude 3 and OpenAI
- **Multi-language Support**: Generates content in both English and Vietnamese with customizable tone
- **Video Rendering**: Automatically creates infographics and short videos from written content using Remotion
- **Social Media Integration**: Auto-posts to Facebook pages and other platforms
- **Platform Optimization**: Exports videos in formats optimized for Reels, TikTok, and Shorts

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

## Environment Configuration

Create a `.env.local` file in the root directory:

```env
# AI Provider APIs
OPENAI_API_KEY=your_openai_key_here
ANTHROPIC_API_KEY=your_claude_key_here

# Research/Crawling APIs
RAPIDAPI_KEY=your_rapidapi_key_here

# Social Media APIs
FACEBOOK_PAGE_ACCESS_TOKEN=your_fb_token_here
FACEBOOK_PAGE_ID=your_page_id_here

# Remotion Configuration
REMOTION_LICENSE_KEY=your_remotion_license_here

# Application Settings
NEXT_PUBLIC_APP_URL=http://localhost:3000
NODE_ENV=development
```

## Running the Application

```bash
# Development mode
npm run dev

# Build for production
npm run build

# Start production server
npm run start

# Render videos (Remotion)
npm run remotion:render
```

## Core Architecture

The pipeline follows this flow:

1. **Research Module** → Crawls and aggregates data
2. **AI Generation Module** → Creates content using LLMs
3. **Rendering Module** → Generates videos/images
4. **Publishing Module** → Auto-posts to social platforms

## Key Usage Patterns

### 1. Research & Data Crawling

```typescript
import { crawlTechNews } from '@/lib/research/crawler';
import { analyzeInsights } from '@/lib/research/analyzer';

async function gatherResearch(keyword: string) {
  // Crawl latest news from multiple sources
  const techCrunchData = await crawlTechNews({
    source: 'techcrunch',
    keyword,
    timeframe: '24h'
  });
  
  const twitterData = await crawlTechNews({
    source: 'twitter',
    keyword,
    limit: 50
  });
  
  // Analyze and extract insights
  const insights = await analyzeInsights({
    sources: [techCrunchData, twitterData],
    extractMetrics: true,
    summaryLength: 'detailed'
  });
  
  return insights;
}
```

### 2. AI Content Generation with Claude

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY
});

async function generateContent(research: any, format: string) {
  const prompt = `Based on this research data: ${JSON.stringify(research)}
  
  Create a ${format} article that:
  - Uses data-backed insights
  - Includes specific metrics and examples
  - Has an engaging hook
  - Optimized for social media sharing
  
  Format: ${format}
  Language: Vietnamese and English versions
  Tone: Professional yet conversational`;
  
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [{
      role: 'user',
      content: prompt
    }]
  });
  
  return message.content[0].text;
}
```

### 3. OpenAI Alternative

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY
});

async function generateWithOpenAI(research: any, format: string) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: 'You are an expert content marketer who creates engaging, data-driven articles.'
      },
      {
        role: 'user',
        content: `Create a ${format} article from this research: ${JSON.stringify(research)}`
      }
    ],
    temperature: 0.7,
    max_tokens: 3000
  });
  
  return completion.choices[0].message.content;
}
```

### 4. Video Rendering with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import { webpack } from '@remotion/bundler';
import path from 'path';

async function renderContentVideo(content: any) {
  // Bundle the Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: (config) => config
  });
  
  // Select composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title: content.title,
      keyPoints: content.keyPoints,
      stats: content.statistics,
      branding: {
        logo: '/logo.png',
        colors: ['#FF6B6B', '#4ECDC4']
      }
    }
  });
  
  // Render video
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: `out/${content.id}.mp4`,
    inputProps: composition.inputProps
  });
  
  return `out/${content.id}.mp4`;
}
```

### 5. Remotion Composition Example

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig } from 'remotion';
import { interpolate } from 'remotion';

export const ContentVideo: React.FC<{
  title: string;
  keyPoints: string[];
  stats: { label: string; value: string }[];
}> = ({ title, keyPoints, stats }) => {
  const frame = useCurrentFrame();
  const { fps, durationInFrames } = useVideoConfig();
  
  const opacity = interpolate(
    frame,
    [0, 30],
    [0, 1],
    { extrapolateRight: 'clamp' }
  );
  
  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a1a' }}>
      <div style={{ 
        opacity, 
        padding: 60,
        color: 'white',
        fontFamily: 'Inter, sans-serif'
      }}>
        <h1 style={{ fontSize: 48, marginBottom: 40 }}>{title}</h1>
        
        {keyPoints.map((point, idx) => {
          const pointOpacity = interpolate(
            frame,
            [30 + (idx * 20), 50 + (idx * 20)],
            [0, 1],
            { extrapolateRight: 'clamp' }
          );
          
          return (
            <div key={idx} style={{ opacity: pointOpacity, marginBottom: 20 }}>
              <p style={{ fontSize: 24 }}>• {point}</p>
            </div>
          );
        })}
        
        <div style={{ marginTop: 60 }}>
          {stats.map((stat, idx) => (
            <div key={idx} style={{ marginBottom: 15 }}>
              <span style={{ fontSize: 36, fontWeight: 'bold' }}>
                {stat.value}
              </span>
              <span style={{ fontSize: 18, marginLeft: 10 }}>
                {stat.label}
              </span>
            </div>
          ))}
        </div>
      </div>
    </AbsoluteFill>
  );
};
```

### 6. Auto-Posting to Social Media

```typescript
import axios from 'axios';

async function postToFacebook(content: any, videoPath: string) {
  const { FACEBOOK_PAGE_ACCESS_TOKEN, FACEBOOK_PAGE_ID } = process.env;
  
  // Upload video
  const formData = new FormData();
  formData.append('file', fs.createReadStream(videoPath));
  formData.append('description', content.caption);
  formData.append('access_token', FACEBOOK_PAGE_ACCESS_TOKEN);
  
  const response = await axios.post(
    `https://graph.facebook.com/v18.0/${FACEBOOK_PAGE_ID}/videos`,
    formData,
    {
      headers: formData.getHeaders()
    }
  );
  
  return response.data;
}
```

### 7. Complete Pipeline Example

```typescript
import { NextApiRequest, NextApiResponse } from 'next';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  const { keyword, format, language } = req.body;
  
  try {
    // Step 1: Research
    const research = await gatherResearch(keyword);
    
    // Step 2: Generate Content
    const content = await generateContent(research, format);
    
    // Step 3: Parse and structure
    const structuredContent = parseContent(content, language);
    
    // Step 4: Render Video
    const videoPath = await renderContentVideo(structuredContent);
    
    // Step 5: Post to Social Media
    const postResult = await postToFacebook(structuredContent, videoPath);
    
    res.status(200).json({
      success: true,
      contentId: structuredContent.id,
      videoPath,
      socialPostId: postResult.id
    });
    
  } catch (error) {
    console.error('Pipeline error:', error);
    res.status(500).json({ error: error.message });
  }
}
```

## Content Format Types

```typescript
type ContentFormat = 
  | 'toplist'      // Top 10 lists
  | 'pov'          // Point of view / opinion
  | 'case-study'   // In-depth case analysis
  | 'how-to'       // Tutorial/guide
  | 'news-recap'   // News summary
  | 'comparison';  // Product/service comparison

interface ContentConfig {
  format: ContentFormat;
  language: 'en' | 'vi' | 'both';
  tone: 'professional' | 'casual' | 'humorous';
  length: 'short' | 'medium' | 'long';
  includeVideo: boolean;
  targetPlatform: 'facebook' | 'linkedin' | 'twitter' | 'all';
}
```

## Configuration Files

### Next.js Config

```typescript
// next.config.js
module.exports = {
  reactStrictMode: true,
  env: {
    OPENAI_API_KEY: process.env.OPENAI_API_KEY,
    ANTHROPIC_API_KEY: process.env.ANTHROPIC_API_KEY,
  },
  webpack: (config, { isServer }) => {
    if (!isServer) {
      config.resolve.fallback = {
        ...config.resolve.fallback,
        fs: false,
      };
    }
    return config;
  },
};
```

### Remotion Config

```typescript
// remotion.config.ts
import { Config } from '@remotion/cli/config';

Config.setVideoImageFormat('jpeg');
Config.setOverwriteOutput(true);
Config.setPixelFormat('yuv420p');
Config.setConcurrency(4);
Config.setCodec('h264');
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement rate limiting
import { Ratelimit } from '@upstash/ratelimit';
import { Redis } from '@upstash/redis';

const ratelimit = new Ratelimit({
  redis: Redis.fromEnv(),
  limiter: Ratelimit.slidingWindow(10, '1 h'),
});

async function withRateLimit(identifier: string, fn: () => Promise<any>) {
  const { success } = await ratelimit.limit(identifier);
  
  if (!success) {
    throw new Error('Rate limit exceeded');
  }
  
  return fn();
}
```

### Video Rendering Errors

```typescript
// Add retry logic for video rendering
async function renderWithRetry(content: any, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await renderContentVideo(content);
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      console.log(`Retry ${i + 1}/${maxRetries}`);
      await new Promise(resolve => setTimeout(resolve, 2000 * (i + 1)));
    }
  }
}
```

### Memory Issues with Large Videos

```typescript
// Stream video uploads instead of loading into memory
import { createReadStream } from 'fs';
import FormData from 'form-data';

async function uploadLargeVideo(filePath: string) {
  const form = new FormData();
  form.append('video', createReadStream(filePath), {
    knownLength: fs.statSync(filePath).size
  });
  
  return axios.post(uploadUrl, form, {
    headers: form.getHeaders(),
    maxContentLength: Infinity,
    maxBodyLength: Infinity
  });
}
```

## Best Practices

1. **Cache Research Data**: Store crawled data for 24h to avoid redundant API calls
2. **Batch Processing**: Process multiple content pieces in parallel with proper concurrency control
3. **Error Handling**: Implement comprehensive error logging for each pipeline stage
4. **Content Validation**: Validate AI-generated content before posting
5. **A/B Testing**: Generate multiple variations and track performance metrics
