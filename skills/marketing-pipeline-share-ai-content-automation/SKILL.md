---
name: marketing-pipeline-share-ai-content-automation
description: Automated content pipeline for research, scriptwriting, video generation, and social posting using AI (Claude/OpenAI) and Remotion
triggers:
  - automate content creation with AI research and video generation
  - set up marketing pipeline for automated social media posts
  - generate AI content from research to video using Claude
  - create automated content workflow with Remotion video rendering
  - build AI-powered content pipeline with auto-posting
  - configure marketing automation for research and script generation
  - use marketing-pipeline-share for content automation
  - integrate AI content research with video generation
---

# Marketing Pipeline Share - AI Content Automation Skill

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## What This Project Does

**marketing-pipeline-share** is a complete AI-powered content automation pipeline that transforms a single keyword into fully-realized content across multiple formats. It combines:

- **Auto-Research**: Crawls real-time data from TechCrunch, a16z, Twitter/X, LinkedIn (last 24h)
- **AI Script Generation**: Creates content in multiple formats (toplist, POV, case study, how-to) using Claude 3 or OpenAI
- **Multi-language Support**: Generates English and Vietnamese versions simultaneously
- **Video Rendering**: Automatically creates infographics and short videos using Remotion
- **Social Auto-Posting**: Schedules and publishes to Facebook Pages automatically

Built with Next.js (TypeScript), integrated with Anthropic Claude, OpenAI, RapidAPI, and Remotion.

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

## Configuration

### Environment Variables

Create a `.env.local` file with the following:

```bash
# AI Provider Keys
ANTHROPIC_API_KEY=your_claude_api_key_here
OPENAI_API_KEY=your_openai_api_key_here

# Research & Data APIs
RAPIDAPI_KEY=your_rapidapi_key_here

# Social Media Integration
FACEBOOK_PAGE_ACCESS_TOKEN=your_fb_page_token_here
FACEBOOK_PAGE_ID=your_page_id_here

# Database (if applicable)
DATABASE_URL=your_database_url_here

# Next.js
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

### Start Development Server

```bash
npm run dev
# or
yarn dev
```

The application will be available at `http://localhost:3000`.

## Core Components

### 1. Research Module (Auto-Scan)

Automatically scrapes and analyzes content from news sources:

```typescript
// lib/research/auto-scanner.ts
import { Anthropic } from '@anthropic-ai/sdk';

interface ResearchResult {
  sources: Array<{
    url: string;
    title: string;
    summary: string;
    publishedAt: string;
  }>;
  insights: string[];
  keywords: string[];
}

export async function autoResearch(
  keyword: string,
  sources: string[] = ['techcrunch', 'a16z', 'twitter', 'linkedin']
): Promise<ResearchResult> {
  const rapidApiKey = process.env.RAPIDAPI_KEY;
  
  // Fetch data from multiple sources
  const newsData = await Promise.all(
    sources.map(source => fetchFromSource(source, keyword, rapidApiKey))
  );
  
  // Use Claude to analyze and extract insights
  const anthropic = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY,
  });
  
  const message = await anthropic.messages.create({
    model: 'claude-3-sonnet-20240229',
    max_tokens: 2048,
    messages: [{
      role: 'user',
      content: `Analyze these articles about "${keyword}" and extract key insights:\n\n${JSON.stringify(newsData)}`
    }]
  });
  
  return {
    sources: newsData.flat(),
    insights: parseInsights(message.content),
    keywords: extractKeywords(message.content)
  };
}

async function fetchFromSource(
  source: string,
  keyword: string,
  apiKey: string
): Promise<any[]> {
  const response = await fetch(`https://rapidapi.com/v1/${source}/search`, {
    headers: {
      'X-RapidAPI-Key': apiKey,
      'Content-Type': 'application/json'
    },
    method: 'POST',
    body: JSON.stringify({ query: keyword, timeframe: '24h' })
  });
  
  return response.json();
}
```

### 2. Content Generation Module

Creates content in multiple formats and languages:

```typescript
// lib/content/generator.ts
import { Anthropic } from '@anthropic-ai/sdk';
import OpenAI from 'openai';

type ContentFormat = 'toplist' | 'pov' | 'case-study' | 'how-to';
type Language = 'en' | 'vi';
type Tone = 'expert' | 'friendly' | 'humorous';

interface ContentConfig {
  format: ContentFormat;
  language: Language;
  tone: Tone;
  researchData: ResearchResult;
}

export async function generateContent(
  config: ContentConfig,
  provider: 'claude' | 'openai' = 'claude'
): Promise<string> {
  const prompt = buildPrompt(config);
  
  if (provider === 'claude') {
    const anthropic = new Anthropic({
      apiKey: process.env.ANTHROPIC_API_KEY,
    });
    
    const message = await anthropic.messages.create({
      model: 'claude-3-opus-20240229',
      max_tokens: 4096,
      messages: [{ role: 'user', content: prompt }]
    });
    
    return message.content[0].text;
  } else {
    const openai = new OpenAI({
      apiKey: process.env.OPENAI_API_KEY,
    });
    
    const completion = await openai.chat.completions.create({
      model: 'gpt-4-turbo-preview',
      messages: [{ role: 'user', content: prompt }],
      max_tokens: 4096
    });
    
    return completion.choices[0].message.content || '';
  }
}

function buildPrompt(config: ContentConfig): string {
  const formatInstructions = {
    'toplist': 'Create a numbered list article with engaging headlines',
    'pov': 'Write from a unique perspective or opinion piece',
    'case-study': 'Analyze a specific example with data and outcomes',
    'how-to': 'Provide step-by-step instructions'
  };
  
  const toneGuide = {
    'expert': 'Use professional terminology and authoritative voice',
    'friendly': 'Conversational and approachable language',
    'humorous': 'Light-hearted with witty observations'
  };
  
  return `
You are a content creator. Create a ${config.format} article in ${config.language}.

Tone: ${toneGuide[config.tone]}
Format: ${formatInstructions[config.format]}

Research Data:
${JSON.stringify(config.researchData, null, 2)}

Requirements:
- Use data and insights from the research
- Include specific examples and numbers
- Optimize for engagement
- ${config.language === 'vi' ? 'Write in Vietnamese' : 'Write in English'}
  `;
}
```

### 3. Bilingual Content Generation

Generate English and Vietnamese simultaneously:

```typescript
// lib/content/bilingual.ts
interface BilingualContent {
  en: string;
  vi: string;
}

export async function generateBilingualContent(
  config: Omit<ContentConfig, 'language'>
): Promise<BilingualContent> {
  const [enContent, viContent] = await Promise.all([
    generateContent({ ...config, language: 'en' }),
    generateContent({ ...config, language: 'vi' })
  ]);
  
  return { en: enContent, vi: viContent };
}
```

### 4. Video Rendering with Remotion

Automatically create videos from content:

```typescript
// lib/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  content: string;
  title: string;
  platform: 'reels' | 'tiktok' | 'shorts';
}

const platformSpecs = {
  reels: { width: 1080, height: 1920, fps: 30 },
  tiktok: { width: 1080, height: 1920, fps: 30 },
  shorts: { width: 1080, height: 1920, fps: 30 }
};

export async function renderContentVideo(
  config: VideoConfig
): Promise<string> {
  const { width, height, fps } = platformSpecs[config.platform];
  
  // Bundle Remotion project
  const bundleLocation = await bundle(
    path.join(process.cwd(), 'remotion/index.ts')
  );
  
  // Select composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      content: config.content,
      title: config.title
    }
  });
  
  // Render video
  const outputLocation = path.join(
    process.cwd(),
    'public',
    'videos',
    `${Date.now()}-${config.platform}.mp4`
  );
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: {
      content: config.content,
      title: config.title
    }
  });
  
  return outputLocation;
}
```

### 5. Remotion Video Component

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig } from 'remotion';
import React from 'react';

interface ContentVideoProps {
  title: string;
  content: string;
}

export const ContentVideo: React.FC<ContentVideoProps> = ({ title, content }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  
  const opacity = Math.min(1, frame / 30);
  const scale = 0.8 + (opacity * 0.2);
  
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
          transform: `scale(${scale})`,
          opacity,
          padding: 60,
          maxWidth: 900
        }}
      >
        <h1
          style={{
            fontSize: 72,
            color: '#ffffff',
            fontWeight: 'bold',
            marginBottom: 40,
            textAlign: 'center'
          }}
        >
          {title}
        </h1>
        <p
          style={{
            fontSize: 36,
            color: '#cccccc',
            lineHeight: 1.6,
            textAlign: 'center'
          }}
        >
          {content.substring(0, 200)}...
        </p>
      </div>
    </AbsoluteFill>
  );
};
```

### 6. Social Media Auto-Posting

```typescript
// lib/social/facebook.ts
interface PostConfig {
  content: string;
  videoPath?: string;
  scheduledTime?: Date;
}

export async function postToFacebookPage(
  config: PostConfig
): Promise<string> {
  const pageAccessToken = process.env.FACEBOOK_PAGE_ACCESS_TOKEN;
  const pageId = process.env.FACEBOOK_PAGE_ID;
  
  const endpoint = `https://graph.facebook.com/v18.0/${pageId}/feed`;
  
  const formData = new FormData();
  formData.append('message', config.content);
  formData.append('access_token', pageAccessToken);
  
  if (config.scheduledTime) {
    const timestamp = Math.floor(config.scheduledTime.getTime() / 1000);
    formData.append('scheduled_publish_time', timestamp.toString());
    formData.append('published', 'false');
  }
  
  if (config.videoPath) {
    const videoEndpoint = `https://graph.facebook.com/v18.0/${pageId}/videos`;
    formData.append('source', config.videoPath);
    
    const response = await fetch(videoEndpoint, {
      method: 'POST',
      body: formData
    });
    
    return (await response.json()).id;
  }
  
  const response = await fetch(endpoint, {
    method: 'POST',
    body: formData
  });
  
  return (await response.json()).id;
}
```

## Complete Pipeline Example

```typescript
// lib/pipeline/complete-flow.ts
export async function runCompletePipeline(
  keyword: string,
  format: ContentFormat = 'toplist',
  tone: Tone = 'expert'
) {
  // Step 1: Research
  console.log('🔍 Starting research...');
  const researchData = await autoResearch(keyword);
  
  // Step 2: Generate bilingual content
  console.log('✍️ Generating content...');
  const content = await generateBilingualContent({
    format,
    tone,
    researchData
  });
  
  // Step 3: Render videos for both languages
  console.log('🎬 Rendering videos...');
  const [enVideo, viVideo] = await Promise.all([
    renderContentVideo({
      content: content.en,
      title: keyword,
      platform: 'reels'
    }),
    renderContentVideo({
      content: content.vi,
      title: keyword,
      platform: 'reels'
    })
  ]);
  
  // Step 4: Schedule posts
  console.log('📤 Scheduling posts...');
  const scheduleTime = new Date(Date.now() + 3600000); // 1 hour from now
  
  await Promise.all([
    postToFacebookPage({
      content: content.en,
      videoPath: enVideo,
      scheduledTime: scheduleTime
    }),
    postToFacebookPage({
      content: content.vi,
      videoPath: viVideo,
      scheduledTime: new Date(scheduleTime.getTime() + 1800000) // 30 min after EN
    })
  ]);
  
  console.log('✅ Pipeline complete!');
  
  return {
    research: researchData,
    content,
    videos: { en: enVideo, vi: viVideo }
  };
}
```

## API Routes (Next.js)

### Create Content Endpoint

```typescript
// pages/api/content/create.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { runCompletePipeline } from '@/lib/pipeline/complete-flow';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }
  
  try {
    const { keyword, format, tone } = req.body;
    
    if (!keyword) {
      return res.status(400).json({ error: 'Keyword is required' });
    }
    
    const result = await runCompletePipeline(keyword, format, tone);
    
    res.status(200).json({
      success: true,
      data: result
    });
  } catch (error) {
    console.error('Pipeline error:', error);
    res.status(500).json({
      error: 'Pipeline execution failed',
      message: error.message
    });
  }
}
```

## Common Patterns

### Pattern 1: Content Scheduling Queue

```typescript
// lib/queue/scheduler.ts
interface ScheduledContent {
  id: string;
  keyword: string;
  scheduledFor: Date;
  status: 'pending' | 'processing' | 'completed' | 'failed';
}

export class ContentScheduler {
  private queue: ScheduledContent[] = [];
  
  add(keyword: string, scheduledFor: Date): string {
    const id = `content-${Date.now()}`;
    this.queue.push({
      id,
      keyword,
      scheduledFor,
      status: 'pending'
    });
    return id;
  }
  
  async processQueue() {
    const now = new Date();
    const pending = this.queue.filter(
      item => item.status === 'pending' && item.scheduledFor <= now
    );
    
    for (const item of pending) {
      try {
        item.status = 'processing';
        await runCompletePipeline(item.keyword);
        item.status = 'completed';
      } catch (error) {
        item.status = 'failed';
        console.error(`Failed to process ${item.id}:`, error);
      }
    }
  }
}
```

### Pattern 2: Custom Content Templates

```typescript
// lib/templates/custom.ts
export const contentTemplates = {
  productLaunch: {
    format: 'case-study' as ContentFormat,
    tone: 'expert' as Tone,
    customPrompt: 'Focus on innovation and market impact'
  },
  trendAnalysis: {
    format: 'pov' as ContentFormat,
    tone: 'expert' as Tone,
    customPrompt: 'Analyze market trends with data'
  },
  tutorial: {
    format: 'how-to' as ContentFormat,
    tone: 'friendly' as Tone,
    customPrompt: 'Step-by-step beginner-friendly guide'
  }
};

export function applyTemplate(
  templateName: keyof typeof contentTemplates,
  researchData: ResearchResult
) {
  return {
    ...contentTemplates[templateName],
    researchData
  };
}
```

## Troubleshooting

### Issue: API Rate Limits

```typescript
// lib/utils/rate-limiter.ts
class RateLimiter {
  private lastCall: number = 0;
  private minInterval: number;
  
  constructor(callsPerMinute: number) {
    this.minInterval = 60000 / callsPerMinute;
  }
  
  async throttle() {
    const now = Date.now();
    const timeSinceLastCall = now - this.lastCall;
    
    if (timeSinceLastCall < this.minInterval) {
      await new Promise(resolve => 
        setTimeout(resolve, this.minInterval - timeSinceLastCall)
      );
    }
    
    this.lastCall = Date.now();
  }
}

const claudeLimiter = new RateLimiter(50); // 50 calls/min

// Use before API calls
await claudeLimiter.throttle();
const content = await generateContent(config);
```

### Issue: Video Rendering Timeouts

Increase timeout for long videos:

```typescript
await renderMedia({
  composition,
  serveUrl: bundleLocation,
  codec: 'h264',
  outputLocation,
  timeoutInMilliseconds: 300000, // 5 minutes
  inputProps: { content, title }
});
```

### Issue: Research Data Quality

Filter and validate research results:

```typescript
function validateResearchData(data: ResearchResult): boolean {
  return (
    data.sources.length >= 3 &&
    data.insights.length >= 5 &&
    data.keywords.length >= 3
  );
}

const research = await autoResearch(keyword);
if (!validateResearchData(research)) {
  throw new Error('Insufficient research data quality');
}
```

## Testing

```typescript
// __tests__/pipeline.test.ts
import { autoResearch, generateContent } from '@/lib';

describe('Content Pipeline', () => {
  it('should fetch research data', async () => {
    const result = await autoResearch('AI marketing');
    expect(result.sources.length).toBeGreaterThan(0);
    expect(result.insights.length).toBeGreaterThan(0);
  });
  
  it('should generate content in correct language', async () => {
    const content = await generateContent({
      format: 'toplist',
      language: 'vi',
      tone: 'friendly',
      researchData: mockResearchData
    });
    
    expect(content).toContain('Tiếng Việt content markers');
  });
});
```
