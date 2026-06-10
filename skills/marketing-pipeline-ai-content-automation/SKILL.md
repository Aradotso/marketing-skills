---
name: marketing-pipeline-ai-content-automation
description: AI-powered content pipeline that automates research, scriptwriting, Facebook posting, and video generation using Claude/OpenAI and Remotion
triggers:
  - automate content creation with AI research and video generation
  - set up automated marketing content pipeline with Claude
  - generate social media content from research to video
  - build AI content automation workflow with Remotion
  - create automated Facebook posting with AI-generated content
  - implement content pipeline with auto-research and video rendering
  - use marketing pipeline share for content automation
  - integrate Claude and OpenAI for content generation pipeline
---

# Marketing Pipeline AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with **marketing-pipeline-share**, a comprehensive TypeScript-based content automation system that handles the entire content lifecycle: from automated research scanning (TechCrunch, Twitter, LinkedIn), to AI-powered scriptwriting with Claude/OpenAI, to automated video generation with Remotion, and finally auto-posting to Facebook pages.

## What This Project Does

The marketing pipeline automates:
1. **Auto-scan research**: Crawls recent news from major sources (24h window)
2. **AI content generation**: Creates articles in multiple formats (toplist, POV, case study, how-to) using Claude 3 or OpenAI
3. **Multi-language support**: Generates content in both English and Vietnamese
4. **Video rendering**: Converts written content into infographics and short videos using Remotion
5. **Auto-posting**: Publishes content directly to Facebook pages
6. **Platform optimization**: Exports videos in formats suitable for Reels, TikTok, Shorts

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

Create a `.env.local` file in the project root:

```env
# AI Provider Keys
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research/Crawling APIs
RAPIDAPI_KEY=your_rapidapi_key

# Facebook Integration
FACEBOOK_PAGE_ACCESS_TOKEN=your_fb_page_token
FACEBOOK_PAGE_ID=your_fb_page_id

# Remotion (Video Rendering)
REMOTION_LICENSE_KEY=your_remotion_license

# Application
NEXT_PUBLIC_API_URL=http://localhost:3000
```

## Running the Application

```bash
# Development mode
npm run dev

# Production build
npm run build
npm run start

# Remotion video rendering
npm run remotion
```

## Core Architecture & Usage Patterns

### 1. Research Pipeline

The system automatically crawls and analyzes content from multiple sources:

```typescript
// lib/research/scanner.ts
import { RapidAPIClient } from './rapidapi-client';

interface ResearchSource {
  source: string;
  url: string;
  timeframe: '24h' | '7d' | '30d';
}

export async function scanResearch(keyword: string, sources: ResearchSource[]) {
  const client = new RapidAPIClient(process.env.RAPIDAPI_KEY!);
  
  const results = await Promise.all(
    sources.map(async (src) => {
      const data = await client.search({
        query: keyword,
        source: src.source,
        timeframe: src.timeframe
      });
      return {
        source: src.source,
        insights: extractInsights(data),
        metrics: extractMetrics(data)
      };
    })
  );
  
  return aggregateResearch(results);
}

function extractInsights(data: any) {
  // Extract key insights, quotes, statistics
  return data.articles.map((article: any) => ({
    title: article.title,
    summary: article.summary,
    url: article.url,
    publishedAt: article.publishedAt,
    metrics: article.metrics
  }));
}
```

### 2. AI Content Generation

Generate content using Claude or OpenAI with customizable formats:

```typescript
// lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

interface ContentConfig {
  provider: 'claude' | 'openai';
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  researchData: any;
}

export async function generateContent(config: ContentConfig) {
  const prompt = buildPrompt(config);
  
  if (config.provider === 'claude') {
    return await generateWithClaude(prompt, config);
  } else {
    return await generateWithOpenAI(prompt, config);
  }
}

async function generateWithClaude(prompt: string, config: ContentConfig) {
  const anthropic = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY
  });
  
  const message = await anthropic.messages.create({
    model: 'claude-3-opus-20240229',
    max_tokens: 4096,
    messages: [{
      role: 'user',
      content: prompt
    }],
    system: getSystemPrompt(config.format, config.tone)
  });
  
  return {
    content: message.content[0].text,
    metadata: {
      provider: 'claude',
      model: message.model,
      usage: message.usage
    }
  };
}

async function generateWithOpenAI(prompt: string, config: ContentConfig) {
  const openai = new OpenAI({
    apiKey: process.env.OPENAI_API_KEY
  });
  
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      { role: 'system', content: getSystemPrompt(config.format, config.tone) },
      { role: 'user', content: prompt }
    ],
    temperature: 0.7,
    max_tokens: 3000
  });
  
  return {
    content: completion.choices[0].message.content,
    metadata: {
      provider: 'openai',
      model: completion.model,
      usage: completion.usage
    }
  };
}

function buildPrompt(config: ContentConfig): string {
  return `
Create a ${config.format} article in ${config.language} with a ${config.tone} tone.

Research Data:
${JSON.stringify(config.researchData, null, 2)}

Requirements:
- Use data-backed insights from the research
- Include specific metrics and statistics
- Make it engaging and shareable
- Optimize for ${config.language === 'vi' ? 'Vietnamese' : 'English'} audience
`;
}
```

### 3. Video Generation with Remotion

Convert content into videos automatically:

```typescript
// lib/video/generator.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  content: string;
  style: 'infographic' | 'text-animation' | 'slide-show';
  platform: 'reels' | 'tiktok' | 'shorts' | 'square';
  duration: number;
}

export async function generateVideo(config: VideoConfig) {
  // Bundle Remotion project
  const bundled = await bundle(
    path.join(process.cwd(), 'remotion/index.ts')
  );
  
  // Get composition
  const composition = await selectComposition({
    serveUrl: bundled,
    id: config.style,
    inputProps: {
      content: config.content,
      platform: config.platform
    }
  });
  
  // Render video
  const outputPath = path.join(
    process.cwd(),
    'public/videos',
    `${Date.now()}.mp4`
  );
  
  await renderMedia({
    composition,
    serveUrl: bundled,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: {
      content: config.content,
      platform: config.platform
    }
  });
  
  return {
    path: outputPath,
    url: `/videos/${path.basename(outputPath)}`,
    dimensions: getVideoDimensions(config.platform)
  };
}

function getVideoDimensions(platform: string) {
  const dimensions = {
    'reels': { width: 1080, height: 1920 },
    'tiktok': { width: 1080, height: 1920 },
    'shorts': { width: 1080, height: 1920 },
    'square': { width: 1080, height: 1080 }
  };
  return dimensions[platform] || dimensions.square;
}
```

### 4. Remotion Video Component

```typescript
// remotion/compositions/Infographic.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig } from 'remotion';
import React from 'react';

interface InfographicProps {
  content: string;
  platform: string;
}

export const Infographic: React.FC<InfographicProps> = ({ content, platform }) => {
  const frame = useCurrentFrame();
  const { fps, durationInFrames } = useVideoConfig();
  
  const opacity = Math.min(1, frame / 30);
  const scale = Math.min(1, frame / 20);
  
  const contentBlocks = parseContent(content);
  
  return (
    <AbsoluteFill
      style={{
        backgroundColor: '#1a1a1a',
        justifyContent: 'center',
        alignItems: 'center'
      }}
    >
      <div
        style={{
          opacity,
          transform: `scale(${scale})`,
          maxWidth: '80%',
          textAlign: 'center'
        }}
      >
        {contentBlocks.map((block, index) => {
          const blockFrame = frame - (index * fps);
          const blockOpacity = Math.min(1, Math.max(0, blockFrame / 20));
          
          return (
            <div
              key={index}
              style={{
                opacity: blockOpacity,
                marginBottom: '2rem',
                transition: 'opacity 0.5s'
              }}
            >
              <h2 style={{ color: '#fff', fontSize: '3rem' }}>
                {block.title}
              </h2>
              <p style={{ color: '#ccc', fontSize: '1.5rem' }}>
                {block.text}
              </p>
            </div>
          );
        })}
      </div>
    </AbsoluteFill>
  );
};

function parseContent(content: string) {
  // Parse content into title/text blocks
  const lines = content.split('\n').filter(l => l.trim());
  const blocks = [];
  
  for (let i = 0; i < lines.length; i += 2) {
    blocks.push({
      title: lines[i] || '',
      text: lines[i + 1] || ''
    });
  }
  
  return blocks.slice(0, 5); // Limit to 5 blocks
}
```

### 5. Facebook Auto-Posting

```typescript
// lib/social/facebook-poster.ts
import axios from 'axios';

interface FacebookPostConfig {
  pageId: string;
  accessToken: string;
  content: string;
  videoUrl?: string;
  imageUrl?: string;
}

export async function postToFacebook(config: FacebookPostConfig) {
  const baseUrl = `https://graph.facebook.com/v18.0/${config.pageId}`;
  
  if (config.videoUrl) {
    return await postVideo(baseUrl, config);
  } else if (config.imageUrl) {
    return await postImage(baseUrl, config);
  } else {
    return await postText(baseUrl, config);
  }
}

async function postVideo(baseUrl: string, config: FacebookPostConfig) {
  const response = await axios.post(`${baseUrl}/videos`, {
    access_token: config.accessToken,
    description: config.content,
    file_url: config.videoUrl,
    published: true
  });
  
  return {
    postId: response.data.id,
    url: `https://facebook.com/${response.data.id}`
  };
}

async function postImage(baseUrl: string, config: FacebookPostConfig) {
  const response = await axios.post(`${baseUrl}/photos`, {
    access_token: config.accessToken,
    message: config.content,
    url: config.imageUrl,
    published: true
  });
  
  return {
    postId: response.data.id,
    url: `https://facebook.com/${response.data.id}`
  };
}

async function postText(baseUrl: string, config: FacebookPostConfig) {
  const response = await axios.post(`${baseUrl}/feed`, {
    access_token: config.accessToken,
    message: config.content
  });
  
  return {
    postId: response.data.id,
    url: `https://facebook.com/${response.data.id}`
  };
}
```

### 6. Complete Pipeline Orchestration

```typescript
// lib/pipeline/orchestrator.ts
import { scanResearch } from '../research/scanner';
import { generateContent } from '../ai/content-generator';
import { generateVideo } from '../video/generator';
import { postToFacebook } from '../social/facebook-poster';

interface PipelineConfig {
  keyword: string;
  contentFormat: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  includeVideo: boolean;
  autoPost: boolean;
}

export async function runContentPipeline(config: PipelineConfig) {
  console.log('[Pipeline] Starting content generation pipeline...');
  
  // Step 1: Research
  console.log('[Pipeline] Step 1: Scanning research...');
  const researchData = await scanResearch(config.keyword, [
    { source: 'techcrunch', url: '', timeframe: '24h' },
    { source: 'twitter', url: '', timeframe: '24h' },
    { source: 'linkedin', url: '', timeframe: '24h' }
  ]);
  
  // Step 2: Generate Content
  console.log('[Pipeline] Step 2: Generating content...');
  const content = await generateContent({
    provider: 'claude',
    format: config.contentFormat,
    language: config.language,
    tone: 'expert',
    researchData
  });
  
  // Step 3: Generate Video (if enabled)
  let video = null;
  if (config.includeVideo) {
    console.log('[Pipeline] Step 3: Generating video...');
    video = await generateVideo({
      content: content.content,
      style: 'infographic',
      platform: 'reels',
      duration: 30
    });
  }
  
  // Step 4: Auto-post (if enabled)
  let post = null;
  if (config.autoPost) {
    console.log('[Pipeline] Step 4: Posting to Facebook...');
    post = await postToFacebook({
      pageId: process.env.FACEBOOK_PAGE_ID!,
      accessToken: process.env.FACEBOOK_PAGE_ACCESS_TOKEN!,
      content: content.content,
      videoUrl: video?.url
    });
  }
  
  console.log('[Pipeline] Pipeline completed successfully!');
  
  return {
    research: researchData,
    content,
    video,
    post,
    metadata: {
      keyword: config.keyword,
      completedAt: new Date().toISOString()
    }
  };
}
```

## API Routes (Next.js)

### Create Content Endpoint

```typescript
// pages/api/content/create.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { runContentPipeline } from '../../../lib/pipeline/orchestrator';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }
  
  try {
    const { keyword, format, language, includeVideo, autoPost } = req.body;
    
    if (!keyword) {
      return res.status(400).json({ error: 'Keyword is required' });
    }
    
    const result = await runContentPipeline({
      keyword,
      contentFormat: format || 'toplist',
      language: language || 'vi',
      includeVideo: includeVideo ?? true,
      autoPost: autoPost ?? false
    });
    
    res.status(200).json(result);
  } catch (error) {
    console.error('Pipeline error:', error);
    res.status(500).json({ 
      error: 'Pipeline failed', 
      message: error.message 
    });
  }
}
```

## Common Usage Examples

### Example 1: Generate Content Only (No Video)

```typescript
const result = await runContentPipeline({
  keyword: 'AI automation trends 2024',
  contentFormat: 'toplist',
  language: 'en',
  includeVideo: false,
  autoPost: false
});

console.log('Generated content:', result.content.content);
```

### Example 2: Full Pipeline with Video and Auto-Post

```typescript
const result = await runContentPipeline({
  keyword: 'Marketing automation tips',
  contentFormat: 'how-to',
  language: 'vi',
  includeVideo: true,
  autoPost: true
});

console.log('Content posted at:', result.post.url);
console.log('Video available at:', result.video.url);
```

### Example 3: Bilingual Content Generation

```typescript
const [englishContent, vietnameseContent] = await Promise.all([
  generateContent({
    provider: 'claude',
    format: 'case-study',
    language: 'en',
    tone: 'expert',
    researchData
  }),
  generateContent({
    provider: 'claude',
    format: 'case-study',
    language: 'vi',
    tone: 'friendly',
    researchData
  })
]);
```

## Troubleshooting

### API Rate Limits

If you encounter rate limits with Claude or OpenAI:

```typescript
// Add retry logic with exponential backoff
async function generateWithRetry(config: ContentConfig, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await generateContent(config);
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

### Video Rendering Memory Issues

For large videos, increase Node.js memory:

```json
// package.json
{
  "scripts": {
    "remotion": "NODE_OPTIONS=--max-old-space-size=4096 remotion render"
  }
}
```

### Facebook Token Expiration

Refresh long-lived page tokens:

```typescript
async function refreshPageToken(currentToken: string) {
  const response = await axios.get(
    'https://graph.facebook.com/v18.0/oauth/access_token',
    {
      params: {
        grant_type: 'fb_exchange_token',
        client_id: process.env.FACEBOOK_APP_ID,
        client_secret: process.env.FACEBOOK_APP_SECRET,
        fb_exchange_token: currentToken
      }
    }
  );
  
  return response.data.access_token;
}
```

### Research Data Quality

Filter and validate research results:

```typescript
function validateResearch(data: any) {
  return data.filter(article => 
    article.title &&
    article.summary &&
    article.publishedAt &&
    new Date(article.publishedAt) > new Date(Date.now() - 86400000) // 24h
  );
}
```

This skill covers the complete content automation pipeline from research to publication. Adjust AI providers, content formats, and video styles based on your specific requirements.
