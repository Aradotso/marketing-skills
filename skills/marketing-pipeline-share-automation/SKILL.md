---
name: marketing-pipeline-share-automation
description: AI-powered content automation pipeline for research, scriptwriting, Facebook posting, and video generation using Claude/OpenAI and Remotion
triggers:
  - automate content creation pipeline
  - research and generate marketing content
  - auto-post to Facebook page
  - generate video from content script
  - crawl news for content ideas
  - create multilingual social media posts
  - build AI content workflow
  - use marketing pipeline automation
---

# Marketing Pipeline Share Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables you to work with **Ultimate AI Content Pipeline** (marketing-pipeline-share), a comprehensive TypeScript-based automation system that handles the entire content creation workflow: from researching trending topics, generating scripts in multiple formats and languages, to auto-posting on Facebook and rendering videos with Remotion.

## What This Project Does

Marketing Pipeline Share is an all-in-one content automation system that:

- **Auto-scans research sources**: Crawls TechCrunch, a16z, X (Twitter), LinkedIn for fresh content within 24h
- **Generates diverse content formats**: Creates Toplists, POV pieces, Case Studies, How-tos using Claude 3 or OpenAI
- **Multilingual support**: Simultaneously generates English and Vietnamese versions
- **Auto-posts to Facebook**: Publishes content directly to Facebook Pages
- **Renders videos automatically**: Uses Remotion to convert text content into infographics and short videos for Reels/TikTok/Shorts

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
# AI Providers
ANTHROPIC_API_KEY=your_claude_api_key_here
OPENAI_API_KEY=your_openai_api_key_here

# RapidAPI for research/crawling
RAPIDAPI_KEY=your_rapidapi_key_here

# Facebook Page Integration
FACEBOOK_PAGE_ACCESS_TOKEN=your_facebook_token_here
FACEBOOK_PAGE_ID=your_page_id_here

# Next.js
NEXT_PUBLIC_API_URL=http://localhost:3000

# Remotion (for video rendering)
REMOTION_AWS_ACCESS_KEY_ID=your_aws_key_here
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret_here
```

## Running the Application

```bash
# Development mode
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Render videos with Remotion
npm run remotion
```

## Project Structure

```
marketing-pipeline-share/
├── app/                    # Next.js app directory
│   ├── api/               # API routes
│   │   ├── research/      # Research crawling endpoints
│   │   ├── generate/      # Content generation endpoints
│   │   └── publish/       # Facebook posting endpoints
│   └── components/        # React components
├── lib/                   # Core libraries
│   ├── ai/               # AI integration (Claude, OpenAI)
│   ├── crawler/          # Web scraping utilities
│   ├── facebook/         # Facebook API integration
│   └── remotion/         # Video rendering
├── remotion/             # Remotion video templates
└── types/                # TypeScript definitions
```

## Key API Endpoints

### Research & Crawling

```typescript
// app/api/research/scan/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { crawlTechNews } from '@/lib/crawler/tech-sources';

export async function POST(request: NextRequest) {
  const { keyword, sources = ['techcrunch', 'a16z'] } = await request.json();
  
  const results = await crawlTechNews({
    keyword,
    sources,
    timeframe: '24h'
  });
  
  return NextResponse.json({ 
    success: true, 
    articles: results 
  });
}
```

### Content Generation

```typescript
// app/api/generate/content/route.ts
import { Anthropic } from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

export async function POST(request: NextRequest) {
  const { topic, format, language, tone } = await request.json();
  
  const prompt = `Generate a ${format} article about ${topic} in ${language} with a ${tone} tone.`;
  
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [
      { role: 'user', content: prompt }
    ],
  });
  
  const content = message.content[0].text;
  
  return NextResponse.json({ 
    success: true, 
    content 
  });
}
```

### Facebook Publishing

```typescript
// lib/facebook/publisher.ts
export async function publishToFacebookPage(
  content: string,
  imageUrl?: string
) {
  const pageAccessToken = process.env.FACEBOOK_PAGE_ACCESS_TOKEN;
  const pageId = process.env.FACEBOOK_PAGE_ID;
  
  const formData = new FormData();
  formData.append('message', content);
  formData.append('access_token', pageAccessToken);
  
  if (imageUrl) {
    formData.append('url', imageUrl);
  }
  
  const endpoint = imageUrl 
    ? `https://graph.facebook.com/v18.0/${pageId}/photos`
    : `https://graph.facebook.com/v18.0/${pageId}/feed`;
  
  const response = await fetch(endpoint, {
    method: 'POST',
    body: formData,
  });
  
  return response.json();
}
```

## Content Generation Workflow

### Complete Pipeline Example

```typescript
// lib/pipeline/content-workflow.ts
import { crawlTechNews } from '@/lib/crawler/tech-sources';
import { generateContent } from '@/lib/ai/content-generator';
import { publishToFacebookPage } from '@/lib/facebook/publisher';
import { renderVideo } from '@/lib/remotion/video-renderer';

export async function executeContentPipeline(
  keyword: string,
  options: {
    format: 'toplist' | 'pov' | 'case-study' | 'how-to';
    languages: ('en' | 'vi')[];
    autoPublish: boolean;
    generateVideo: boolean;
  }
) {
  // Step 1: Research
  const research = await crawlTechNews({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter'],
    timeframe: '24h'
  });
  
  // Step 2: Generate content for each language
  const contents = await Promise.all(
    options.languages.map(async (lang) => {
      return await generateContent({
        research,
        format: options.format,
        language: lang,
        tone: 'professional'
      });
    })
  );
  
  // Step 3: Auto-publish if enabled
  if (options.autoPublish) {
    for (const content of contents) {
      await publishToFacebookPage(content.text, content.imageUrl);
    }
  }
  
  // Step 4: Generate video if enabled
  if (options.generateVideo) {
    const videoUrl = await renderVideo({
      script: contents[0].text,
      style: 'infographic',
      aspectRatio: '9:16' // For Reels/TikTok
    });
    
    return { contents, videoUrl };
  }
  
  return { contents };
}
```

### Using the Pipeline

```typescript
// app/components/ContentGenerator.tsx
'use client';

import { useState } from 'react';
import { executeContentPipeline } from '@/lib/pipeline/content-workflow';

export default function ContentGenerator() {
  const [keyword, setKeyword] = useState('');
  const [loading, setLoading] = useState(false);
  
  const handleGenerate = async () => {
    setLoading(true);
    
    try {
      const result = await executeContentPipeline(keyword, {
        format: 'toplist',
        languages: ['en', 'vi'],
        autoPublish: true,
        generateVideo: true
      });
      
      console.log('Pipeline completed:', result);
    } catch (error) {
      console.error('Pipeline failed:', error);
    } finally {
      setLoading(false);
    }
  };
  
  return (
    <div className="p-4">
      <input
        type="text"
        value={keyword}
        onChange={(e) => setKeyword(e.target.value)}
        placeholder="Enter keyword..."
        className="border p-2 rounded"
      />
      <button
        onClick={handleGenerate}
        disabled={loading}
        className="ml-2 bg-blue-500 text-white px-4 py-2 rounded"
      >
        {loading ? 'Generating...' : 'Generate Content'}
      </button>
    </div>
  );
}
```

## Video Generation with Remotion

### Video Template Example

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, interpolate } from 'remotion';

export const ContentVideo: React.FC<{
  title: string;
  points: string[];
}> = ({ title, points }) => {
  const frame = useCurrentFrame();
  
  const opacity = interpolate(frame, [0, 30], [0, 1], {
    extrapolateRight: 'clamp',
  });
  
  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a1a' }}>
      <div style={{ opacity, padding: 60 }}>
        <h1 style={{ color: 'white', fontSize: 64 }}>{title}</h1>
        <ul style={{ color: 'white', fontSize: 32, marginTop: 40 }}>
          {points.map((point, i) => (
            <li key={i} style={{ marginBottom: 20 }}>{point}</li>
          ))}
        </ul>
      </div>
    </AbsoluteFill>
  );
};
```

### Render Video

```typescript
// lib/remotion/video-renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

export async function renderVideo(config: {
  script: string;
  style: string;
  aspectRatio: string;
}) {
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
  });
  
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title: config.script.split('\n')[0],
      points: config.script.split('\n').slice(1, 6),
    },
  });
  
  const outputLocation = `./public/videos/output-${Date.now()}.mp4`;
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
  });
  
  return outputLocation;
}
```

## Common Patterns

### Multi-Format Content Generation

```typescript
const formats = ['toplist', 'pov', 'case-study', 'how-to'];

const allContent = await Promise.all(
  formats.map(format => 
    generateContent({
      research,
      format,
      language: 'en',
      tone: 'professional'
    })
  )
);
```

### Scheduled Publishing

```typescript
// lib/scheduler/post-scheduler.ts
import { executeContentPipeline } from '@/lib/pipeline/content-workflow';

export async function scheduleDaily(keywords: string[]) {
  for (const keyword of keywords) {
    await executeContentPipeline(keyword, {
      format: 'toplist',
      languages: ['en', 'vi'],
      autoPublish: true,
      generateVideo: false
    });
    
    // Wait 1 hour between posts
    await new Promise(resolve => setTimeout(resolve, 3600000));
  }
}
```

## Troubleshooting

### API Rate Limits

If you hit Claude/OpenAI rate limits:

```typescript
// Add retry logic with exponential backoff
async function generateWithRetry(prompt: string, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await anthropic.messages.create({
        model: 'claude-3-5-sonnet-20241022',
        max_tokens: 4096,
        messages: [{ role: 'user', content: prompt }],
      });
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      await new Promise(resolve => setTimeout(resolve, 2 ** i * 1000));
    }
  }
}
```

### Facebook Token Expiration

Facebook tokens expire. Implement token refresh:

```typescript
// Check token validity before posting
async function isTokenValid(token: string) {
  const response = await fetch(
    `https://graph.facebook.com/debug_token?input_token=${token}&access_token=${token}`
  );
  const data = await response.json();
  return data.data?.is_valid ?? false;
}
```

### Video Rendering Memory Issues

For large videos, use cloud rendering:

```typescript
// Use Remotion Lambda for cloud rendering
import { renderMediaOnLambda } from '@remotion/lambda';

const { renderId, bucketName } = await renderMediaOnLambda({
  region: 'us-east-1',
  functionName: 'remotion-render',
  composition,
  serveUrl: bundleLocation,
  codec: 'h264',
});
```

This skill enables AI coding agents to help developers build and customize automated content marketing pipelines with AI-powered research, generation, publishing, and video creation capabilities.
