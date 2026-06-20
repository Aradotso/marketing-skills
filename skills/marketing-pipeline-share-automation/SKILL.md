---
name: marketing-pipeline-share-automation
description: Automated AI content pipeline for research, script generation, Facebook posting, and video creation using Claude, OpenAI, and Remotion
triggers:
  - how do I automate content creation with marketing pipeline
  - set up automated AI content research and posting
  - generate videos from content automatically
  - create content pipeline with Claude and OpenAI
  - automate Facebook posting with AI research
  - build content automation workflow with Remotion
  - configure marketing pipeline share for content generation
  - integrate AI content creation and video rendering
---

# Marketing Pipeline Share Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

Marketing Pipeline Share is an all-in-one AI-powered content automation system that handles the entire content lifecycle: from researching trending topics, generating multi-format content, to automatically posting on Facebook and rendering videos. Built with Next.js, TypeScript, and integrating Claude 3, OpenAI, and Remotion for video generation.

**Key capabilities:**
- Auto-crawl latest news from TechCrunch, a16z, Twitter, LinkedIn
- Generate content in multiple formats (toplist, POV, case study, how-to)
- Bilingual support (English & Vietnamese)
- Automatic video/infographic rendering with Remotion
- Facebook page auto-posting
- Customizable tone and audience targeting

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
# AI Providers
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Facebook Integration
FACEBOOK_ACCESS_TOKEN=your_fb_token
FACEBOOK_PAGE_ID=your_page_id

# Remotion (Video Rendering)
REMOTION_LICENSE_KEY=your_remotion_key

# Database (if applicable)
DATABASE_URL=your_database_url

# App Configuration
NEXT_PUBLIC_API_URL=http://localhost:3000
```

## Running the Application

```bash
# Development mode
npm run dev

# Production build
npm run build
npm start

# Remotion studio (for video editing)
npm run remotion
```

## Core Architecture

### 1. Research Module (Auto-Crawl)

**Typical file structure:**
```typescript
// src/services/research/crawler.ts
import axios from 'axios';

interface ResearchSource {
  name: string;
  url: string;
  selector: string;
}

export async function crawlLatestNews(keyword: string, hours: number = 24) {
  const sources: ResearchSource[] = [
    { name: 'TechCrunch', url: 'https://techcrunch.com/search', selector: '.article' },
    { name: 'a16z', url: 'https://a16z.com/posts', selector: '.post-card' }
  ];

  const articles = [];
  
  for (const source of sources) {
    try {
      const response = await axios.get(source.url, {
        params: { q: keyword },
        headers: {
          'X-RapidAPI-Key': process.env.RAPIDAPI_KEY
        }
      });
      
      // Parse and filter by time
      const filtered = response.data.filter((article: any) => 
        isWithinHours(article.publishedAt, hours)
      );
      
      articles.push(...filtered);
    } catch (error) {
      console.error(`Failed to crawl ${source.name}:`, error);
    }
  }

  return articles;
}

function isWithinHours(timestamp: string, hours: number): boolean {
  const articleDate = new Date(timestamp);
  const cutoff = new Date(Date.now() - hours * 60 * 60 * 1000);
  return articleDate >= cutoff;
}
```

### 2. AI Content Generation

**Claude/OpenAI integration:**

```typescript
// src/services/ai/contentGenerator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

interface ContentRequest {
  topic: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  researchData: any[];
}

export async function generateContent(request: ContentRequest) {
  const prompt = buildPrompt(request);
  
  // Using Claude for long-form content
  const response = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [{
      role: 'user',
      content: prompt
    }],
    temperature: 0.7,
  });

  const content = response.content[0].text;
  
  return {
    title: extractTitle(content),
    body: content,
    metadata: {
      format: request.format,
      language: request.language,
      generatedAt: new Date().toISOString()
    }
  };
}

function buildPrompt(request: ContentRequest): string {
  const formatInstructions = {
    'toplist': 'Create a numbered list article with clear rankings',
    'pov': 'Write from a unique perspective with personal insights',
    'case-study': 'Analyze with data, examples, and actionable takeaways',
    'how-to': 'Provide step-by-step instructions with examples'
  };

  const toneGuide = {
    'expert': 'professional, authoritative, data-driven',
    'friendly': 'conversational, approachable, easy to understand',
    'humorous': 'engaging, witty, entertaining while informative'
  };

  return `
You are a ${toneGuide[request.tone]} content writer.

Topic: ${request.topic}
Format: ${formatInstructions[request.format]}
Language: ${request.language === 'vi' ? 'Vietnamese' : 'English'}

Research Data:
${JSON.stringify(request.researchData, null, 2)}

Create compelling, SEO-optimized content that:
1. Uses the latest research data provided
2. Follows the ${request.format} format strictly
3. Maintains a ${request.tone} tone throughout
4. Includes relevant statistics and sources
5. Is ${request.language === 'vi' ? '800-1200 words in Vietnamese' : '800-1200 words in English'}
`;
}

function extractTitle(content: string): string {
  const lines = content.split('\n');
  return lines[0].replace(/^#\s*/, '').trim();
}
```

**Bilingual content generation:**

```typescript
// src/services/ai/bilingualGenerator.ts
export async function generateBilingualContent(request: Omit<ContentRequest, 'language'>) {
  const [englishContent, vietnameseContent] = await Promise.all([
    generateContent({ ...request, language: 'en' }),
    generateContent({ ...request, language: 'vi' })
  ]);

  return {
    en: englishContent,
    vi: vietnameseContent
  };
}
```

### 3. Facebook Auto-Posting

```typescript
// src/services/social/facebookPoster.ts
import axios from 'axios';

interface FacebookPost {
  message: string;
  link?: string;
  image?: string;
  scheduledTime?: Date;
}

export async function postToFacebook(post: FacebookPost) {
  const pageId = process.env.FACEBOOK_PAGE_ID;
  const accessToken = process.env.FACEBOOK_ACCESS_TOKEN;
  
  const endpoint = `https://graph.facebook.com/v18.0/${pageId}/feed`;
  
  try {
    const response = await axios.post(endpoint, {
      message: post.message,
      link: post.link,
      access_token: accessToken,
      ...(post.scheduledTime && {
        published: false,
        scheduled_publish_time: Math.floor(post.scheduledTime.getTime() / 1000)
      })
    });

    return {
      success: true,
      postId: response.data.id
    };
  } catch (error) {
    console.error('Facebook posting failed:', error);
    throw new Error('Failed to post to Facebook');
  }
}

export async function uploadPhotoAndPost(imagePath: string, caption: string) {
  const pageId = process.env.FACEBOOK_PAGE_ID;
  const accessToken = process.env.FACEBOOK_ACCESS_TOKEN;
  
  const FormData = require('form-data');
  const fs = require('fs');
  
  const form = new FormData();
  form.append('source', fs.createReadStream(imagePath));
  form.append('message', caption);
  form.append('access_token', accessToken);
  
  const endpoint = `https://graph.facebook.com/v18.0/${pageId}/photos`;
  
  const response = await axios.post(endpoint, form, {
    headers: form.getHeaders()
  });
  
  return response.data;
}
```

### 4. Video Generation with Remotion

```typescript
// src/remotion/ContentVideo.tsx
import { AbsoluteFill, Sequence, useCurrentFrame, useVideoConfig } from 'remotion';
import { interpolate } from 'remotion';

interface ContentVideoProps {
  title: string;
  points: string[];
  brandColor: string;
}

export const ContentVideo: React.FC<ContentVideoProps> = ({ 
  title, 
  points, 
  brandColor = '#0066ff' 
}) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  const titleOpacity = interpolate(frame, [0, 30], [0, 1], {
    extrapolateRight: 'clamp',
  });

  return (
    <AbsoluteFill style={{ backgroundColor: 'white' }}>
      {/* Title sequence */}
      <Sequence from={0} durationInFrames={fps * 3}>
        <AbsoluteFill
          style={{
            justifyContent: 'center',
            alignItems: 'center',
            opacity: titleOpacity,
          }}
        >
          <h1 style={{ 
            fontSize: 60, 
            color: brandColor,
            textAlign: 'center',
            padding: '0 100px',
            fontWeight: 'bold'
          }}>
            {title}
          </h1>
        </AbsoluteFill>
      </Sequence>

      {/* Points sequences */}
      {points.map((point, index) => (
        <Sequence
          key={index}
          from={fps * (3 + index * 4)}
          durationInFrames={fps * 4}
        >
          <PointSlide point={point} index={index + 1} brandColor={brandColor} />
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};

const PointSlide: React.FC<{ point: string; index: number; brandColor: string }> = ({
  point,
  index,
  brandColor,
}) => {
  const frame = useCurrentFrame();
  
  const scale = interpolate(frame, [0, 20], [0.8, 1], {
    extrapolateRight: 'clamp',
  });

  return (
    <AbsoluteFill
      style={{
        justifyContent: 'center',
        alignItems: 'center',
        transform: `scale(${scale})`,
      }}
    >
      <div style={{ maxWidth: '80%' }}>
        <div style={{
          fontSize: 80,
          color: brandColor,
          fontWeight: 'bold',
          marginBottom: 20,
        }}>
          {index}
        </div>
        <p style={{
          fontSize: 40,
          color: '#333',
          lineHeight: 1.5,
        }}>
          {point}
        </p>
      </div>
    </AbsoluteFill>
  );
};
```

**Render video programmatically:**

```typescript
// src/services/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

export async function renderContentVideo(
  title: string,
  points: string[],
  outputPath: string
) {
  const compositionId = 'ContentVideo';
  const bundleLocation = await bundle({
    entryPoint: path.join(process.cwd(), 'src/remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: { title, points, brandColor: '#0066ff' },
  });

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: { title, points, brandColor: '#0066ff' },
  });

  return outputPath;
}
```

### 5. Complete Pipeline API Route

```typescript
// src/app/api/pipeline/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { crawlLatestNews } from '@/services/research/crawler';
import { generateContent } from '@/services/ai/contentGenerator';
import { postToFacebook } from '@/services/social/facebookPoster';
import { renderContentVideo } from '@/services/video/renderer';

export async function POST(request: NextRequest) {
  try {
    const { keyword, format, language, tone, includeVideo } = await request.json();

    // Step 1: Research
    console.log('Step 1: Researching latest news...');
    const researchData = await crawlLatestNews(keyword, 24);

    // Step 2: Generate content
    console.log('Step 2: Generating AI content...');
    const content = await generateContent({
      topic: keyword,
      format,
      language,
      tone,
      researchData,
    });

    // Step 3: Generate video (if requested)
    let videoPath = null;
    if (includeVideo) {
      console.log('Step 3: Rendering video...');
      const points = extractKeyPoints(content.body);
      videoPath = await renderContentVideo(
        content.title,
        points,
        `./public/videos/${Date.now()}.mp4`
      );
    }

    // Step 4: Post to Facebook
    console.log('Step 4: Posting to Facebook...');
    const fbPost = await postToFacebook({
      message: `${content.title}\n\n${content.body.substring(0, 500)}...`,
      link: videoPath ? `${process.env.NEXT_PUBLIC_API_URL}${videoPath}` : undefined,
    });

    return NextResponse.json({
      success: true,
      content,
      videoPath,
      facebookPostId: fbPost.postId,
    });
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { error: 'Pipeline execution failed' },
      { status: 500 }
    );
  }
}

function extractKeyPoints(content: string): string[] {
  const lines = content.split('\n').filter(line => 
    line.match(/^(\d+\.|\*|\-)\s/) || line.match(/^#{2,3}\s/)
  );
  return lines.slice(0, 5).map(line => 
    line.replace(/^(\d+\.|\*|\-|#{2,3})\s*/, '').trim()
  );
}
```

## Usage Examples

### Basic Content Generation Workflow

```typescript
// Example: Generate and post content
import { crawlLatestNews } from './services/research/crawler';
import { generateContent } from './services/ai/contentGenerator';
import { postToFacebook } from './services/social/facebookPoster';

async function createAndPostContent() {
  // 1. Research
  const news = await crawlLatestNews('AI marketing automation', 24);
  
  // 2. Generate
  const content = await generateContent({
    topic: 'AI Marketing Automation Trends 2026',
    format: 'toplist',
    language: 'en',
    tone: 'expert',
    researchData: news,
  });
  
  // 3. Post
  await postToFacebook({
    message: content.body,
    scheduledTime: new Date(Date.now() + 3600000), // 1 hour from now
  });
}
```

### Frontend Integration

```typescript
// src/app/dashboard/page.tsx
'use client';

import { useState } from 'react';

export default function Dashboard() {
  const [loading, setLoading] = useState(false);
  
  const handleGenerate = async (e: React.FormEvent<HTMLFormElement>) => {
    e.preventDefault();
    setLoading(true);
    
    const formData = new FormData(e.currentTarget);
    
    const response = await fetch('/api/pipeline', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        keyword: formData.get('keyword'),
        format: formData.get('format'),
        language: formData.get('language'),
        tone: formData.get('tone'),
        includeVideo: formData.get('includeVideo') === 'on',
      }),
    });
    
    const result = await response.json();
    setLoading(false);
    
    if (result.success) {
      alert('Content generated and posted successfully!');
    }
  };
  
  return (
    <form onSubmit={handleGenerate}>
      <input name="keyword" placeholder="Enter topic keyword" required />
      
      <select name="format">
        <option value="toplist">Top List</option>
        <option value="pov">Point of View</option>
        <option value="case-study">Case Study</option>
        <option value="how-to">How To</option>
      </select>
      
      <select name="language">
        <option value="en">English</option>
        <option value="vi">Vietnamese</option>
      </select>
      
      <select name="tone">
        <option value="expert">Expert</option>
        <option value="friendly">Friendly</option>
        <option value="humorous">Humorous</option>
      </select>
      
      <label>
        <input type="checkbox" name="includeVideo" />
        Generate Video
      </label>
      
      <button type="submit" disabled={loading}>
        {loading ? 'Processing...' : 'Generate & Post'}
      </button>
    </form>
  );
}
```

## Common Patterns

### Scheduled Content Pipeline

```typescript
// src/jobs/scheduledPipeline.ts
import cron from 'node-cron';

// Run every day at 9 AM
cron.schedule('0 9 * * *', async () => {
  const topics = ['AI automation', 'Content marketing', 'Social media trends'];
  
  for (const topic of topics) {
    try {
      await fetch(`${process.env.NEXT_PUBLIC_API_URL}/api/pipeline`, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          keyword: topic,
          format: 'toplist',
          language: 'en',
          tone: 'expert',
          includeVideo: true,
        }),
      });
      
      console.log(`Generated content for: ${topic}`);
    } catch (error) {
      console.error(`Failed for ${topic}:`, error);
    }
  }
});
```

### Batch Processing

```typescript
// Process multiple keywords in parallel
async function batchGenerateContent(keywords: string[]) {
  const results = await Promise.allSettled(
    keywords.map(keyword => 
      generateContent({
        topic: keyword,
        format: 'pov',
        language: 'vi',
        tone: 'friendly',
        researchData: [],
      })
    )
  );
  
  return results
    .filter(r => r.status === 'fulfilled')
    .map(r => (r as PromiseFulfilledResult<any>).value);
}
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement exponential backoff
async function apiCallWithRetry(fn: () => Promise<any>, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error: any) {
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

```typescript
// For large video renders, use chunks
import { getCompositions } from '@remotion/renderer';

const compositions = await getCompositions(bundleLocation);
const composition = compositions.find(c => c.id === 'ContentVideo');

if (composition.durationInFrames > 1000) {
  // Render in chunks or reduce quality
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    scale: 0.5, // Reduce to 50% for memory
  });
}
```

### Facebook Token Expiration

```typescript
// Check token validity before posting
async function validateFacebookToken() {
  const response = await axios.get(
    `https://graph.facebook.com/v18.0/me?access_token=${process.env.FACEBOOK_ACCESS_TOKEN}`
  );
  
  if (!response.data.id) {
    throw new Error('Facebook token is invalid or expired');
  }
}
```

## Performance Optimization

```typescript
// Cache research results
import NodeCache from 'node-cache';

const cache = new NodeCache({ stdTTL: 3600 }); // 1 hour

export async function getCachedResearch(keyword: string) {
  const cached = cache.get(keyword);
  if (cached) return cached;
  
  const fresh = await crawlLatestNews(keyword, 24);
  cache.set(keyword, fresh);
  return fresh;
}
```

This skill enables AI agents to effectively utilize the Marketing Pipeline Share system for automated content creation, from research through video generation and social posting.
