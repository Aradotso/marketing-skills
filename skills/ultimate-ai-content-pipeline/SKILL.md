---
name: ultimate-ai-content-pipeline
description: Automate content creation from research to video generation using AI (Claude/OpenAI) and Remotion
triggers:
  - "help me set up the AI content pipeline"
  - "how do I generate automated content with this pipeline"
  - "create content using Claude and OpenAI research"
  - "automate video generation from text content"
  - "crawl and research content with AI"
  - "generate multilingual marketing content"
  - "set up Remotion video rendering for content"
  - "build an automated content workflow"
---

# Ultimate AI Content Pipeline Skill

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

A comprehensive TypeScript-based content automation system that handles the entire content creation pipeline: from crawling fresh data sources (TechCrunch, a16z, Twitter/X, LinkedIn), generating AI-written articles in multiple formats and languages (Vietnamese & English), to automatically rendering videos with Remotion.

## What This Project Does

Ultimate AI Content Pipeline is an all-in-one content automation system that:

- **Auto-crawls** real-time data from major tech news sources and social platforms
- **Generates content** using Claude 3 and OpenAI in multiple formats (toplist, POV, case study, how-to)
- **Supports multilingual** content creation (English & Vietnamese)
- **Renders videos** automatically using Remotion for social media (Reels, TikTok, Shorts)
- **Customizes tone** (expert, friendly, humorous) based on target audience

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

## Configuration

Create a `.env.local` file in the root directory:

```bash
# AI API Keys
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research API Keys
RAPIDAPI_KEY=your_rapidapi_key

# Optional: Video Generation
REMOTION_API_KEY=your_remotion_api_key

# Database (if applicable)
DATABASE_URL=your_database_url

# Application
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app router pages
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI service integrations
│   │   ├── crawlers/    # Content crawling modules
│   │   ├── generators/  # Content generation logic
│   │   └── video/       # Remotion video rendering
│   ├── types/           # TypeScript type definitions
│   └── utils/           # Utility functions
├── public/              # Static assets
└── remotion/            # Remotion video templates
```

## Core Usage Patterns

### 1. Content Research & Crawling

```typescript
import { crawlTechNews } from '@/lib/crawlers/tech-news';
import { crawlSocialMedia } from '@/lib/crawlers/social-media';

async function researchTopic(keyword: string) {
  // Crawl from multiple sources
  const [techNews, socialPosts] = await Promise.all([
    crawlTechNews(keyword, {
      sources: ['techcrunch', 'a16z'],
      timeframe: '24h'
    }),
    crawlSocialMedia(keyword, {
      platforms: ['twitter', 'linkedin'],
      limit: 50
    })
  ]);

  return {
    techNews,
    socialPosts,
    timestamp: new Date().toISOString()
  };
}
```

### 2. AI Content Generation with Claude

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

async function generateContent(
  topic: string,
  format: 'toplist' | 'pov' | 'case-study' | 'how-to',
  language: 'en' | 'vi',
  tone: 'expert' | 'friendly' | 'humorous'
) {
  const systemPrompt = `You are a ${tone} content writer creating ${format} content in ${language}.`;
  
  const userPrompt = `
    Topic: ${topic}
    Format: ${format}
    Language: ${language}
    
    Create engaging, data-backed content that is SEO-optimized and ready for publication.
  `;

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    system: systemPrompt,
    messages: [
      {
        role: 'user',
        content: userPrompt
      }
    ]
  });

  return message.content[0].type === 'text' 
    ? message.content[0].text 
    : '';
}
```

### 3. OpenAI Alternative Implementation

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateWithOpenAI(
  prompt: string,
  researchData: any
) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: 'You are an expert content creator specializing in tech and marketing content.'
      },
      {
        role: 'user',
        content: `${prompt}\n\nResearch Data: ${JSON.stringify(researchData)}`
      }
    ],
    temperature: 0.7,
    max_tokens: 3000
  });

  return completion.choices[0].message.content;
}
```

### 4. Multilingual Content Pipeline

```typescript
interface ContentRequest {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  languages: ('en' | 'vi')[];
  tone: 'expert' | 'friendly' | 'humorous';
}

async function createMultilingualContent(request: ContentRequest) {
  // Step 1: Research
  const research = await researchTopic(request.keyword);
  
  // Step 2: Generate content for each language
  const contentPromises = request.languages.map(async (lang) => {
    const content = await generateContent(
      request.keyword,
      request.format,
      lang,
      request.tone
    );
    
    return {
      language: lang,
      content,
      metadata: {
        wordCount: content.split(' ').length,
        createdAt: new Date().toISOString()
      }
    };
  });
  
  const contents = await Promise.all(contentPromises);
  
  return {
    research,
    contents
  };
}
```

### 5. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  content: string;
  title: string;
  format: 'reels' | 'tiktok' | 'shorts';
}

async function generateVideo(config: VideoConfig) {
  const bundleLocation = await bundle(
    path.join(process.cwd(), 'remotion/index.ts')
  );

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title: config.title,
      content: config.content,
      format: config.format
    }
  });

  const outputLocation = path.join(
    process.cwd(),
    'public/videos',
    `${Date.now()}.mp4`
  );

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
  });

  return {
    videoPath: outputLocation,
    duration: composition.durationInFrames / composition.fps
  };
}
```

### 6. Complete Content Pipeline API Route

```typescript
// app/api/content/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const { keyword, format, languages, generateVideo } = body;

    // Step 1: Research
    const research = await researchTopic(keyword);

    // Step 2: Generate content
    const contentResult = await createMultilingualContent({
      keyword,
      format,
      languages,
      tone: 'expert'
    });

    // Step 3: Generate video (optional)
    let video = null;
    if (generateVideo) {
      const primaryContent = contentResult.contents.find(c => c.language === 'en');
      video = await generateVideo({
        content: primaryContent?.content || '',
        title: keyword,
        format: 'reels'
      });
    }

    return NextResponse.json({
      success: true,
      data: {
        research: research,
        content: contentResult.contents,
        video: video
      }
    });
  } catch (error) {
    console.error('Content generation error:', error);
    return NextResponse.json(
      { success: false, error: 'Failed to generate content' },
      { status: 500 }
    );
  }
}
```

### 7. Client-Side Usage Example

```typescript
'use client';

import { useState } from 'react';

export default function ContentGenerator() {
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState(null);

  async function handleGenerate() {
    setLoading(true);
    try {
      const response = await fetch('/api/content/generate', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          keyword: 'AI in Marketing 2024',
          format: 'toplist',
          languages: ['en', 'vi'],
          generateVideo: true
        })
      });

      const data = await response.json();
      setResult(data);
    } catch (error) {
      console.error('Error:', error);
    } finally {
      setLoading(false);
    }
  }

  return (
    <div>
      <button onClick={handleGenerate} disabled={loading}>
        {loading ? 'Generating...' : 'Generate Content'}
      </button>
      
      {result && (
        <div>
          <h2>Generated Content</h2>
          {/* Render content here */}
        </div>
      )}
    </div>
  );
}
```

## Running the Application

```bash
# Development mode
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Run Remotion Studio (for video template editing)
npm run remotion
```

## Common Patterns

### Rate Limiting for API Calls

```typescript
import pLimit from 'p-limit';

const limit = pLimit(3); // Max 3 concurrent requests

async function batchGenerateContent(topics: string[]) {
  const tasks = topics.map(topic =>
    limit(() => generateContent(topic, 'toplist', 'en', 'expert'))
  );
  
  return await Promise.all(tasks);
}
```

### Caching Research Results

```typescript
import { Redis } from '@upstash/redis';

const redis = new Redis({
  url: process.env.UPSTASH_REDIS_URL,
  token: process.env.UPSTASH_REDIS_TOKEN,
});

async function getCachedResearch(keyword: string) {
  const cacheKey = `research:${keyword}`;
  const cached = await redis.get(cacheKey);
  
  if (cached) {
    return cached;
  }
  
  const fresh = await researchTopic(keyword);
  await redis.setex(cacheKey, 3600, fresh); // Cache for 1 hour
  
  return fresh;
}
```

### Error Handling & Retry Logic

```typescript
async function generateWithRetry(
  topic: string,
  maxRetries = 3
) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await generateContent(topic, 'toplist', 'en', 'expert');
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      await new Promise(resolve => setTimeout(resolve, 1000 * (i + 1)));
    }
  }
}
```

## Troubleshooting

### API Rate Limits
- Implement exponential backoff for API calls
- Use caching to reduce redundant requests
- Consider upgrading API tier for higher limits

### Video Rendering Issues
- Ensure sufficient disk space for video output
- Check Remotion composition configuration
- Verify FFmpeg installation for server deployments

### Content Quality
- Adjust temperature parameter (0.7-0.9) for creativity vs consistency
- Provide more context in prompts for better results
- Use few-shot examples in system prompts

### TypeScript Errors
```bash
# Regenerate types
npm run type-check

# Clear Next.js cache
rm -rf .next
npm run dev
```

### Environment Variables Not Loading
- Ensure `.env.local` is in project root
- Restart dev server after adding new variables
- Use `NEXT_PUBLIC_` prefix for client-side variables

## Advanced Features

### Custom Video Templates

Create custom Remotion compositions in `remotion/` directory:

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig } from 'remotion';

export const ContentVideo: React.FC<{
  title: string;
  content: string;
}> = ({ title, content }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  return (
    <AbsoluteFill style={{ backgroundColor: '#000', padding: 60 }}>
      <h1 style={{ color: '#fff', fontSize: 80 }}>{title}</h1>
      <p style={{ color: '#fff', fontSize: 40 }}>{content}</p>
    </AbsoluteFill>
  );
};
```

### Webhook Integration for Auto-Publishing

```typescript
async function publishToSocialMedia(content: any, video: any) {
  // Facebook/Instagram
  await fetch('https://graph.facebook.com/v18.0/me/feed', {
    method: 'POST',
    headers: { 'Authorization': `Bearer ${process.env.FB_ACCESS_TOKEN}` },
    body: JSON.stringify({ message: content.text, video_url: video.url })
  });
  
  // LinkedIn
  await fetch('https://api.linkedin.com/v2/ugcPosts', {
    method: 'POST',
    headers: { 'Authorization': `Bearer ${process.env.LINKEDIN_TOKEN}` },
    body: JSON.stringify({ /* content */ })
  });
}
```
