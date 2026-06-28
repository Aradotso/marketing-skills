---
name: marketing-content-pipeline-automation
description: Automated AI content pipeline for research, scriptwriting, and video generation using Claude, OpenAI, and Remotion
triggers:
  - how do i automate content creation with AI
  - set up an automated marketing content pipeline
  - generate videos from text content automatically
  - create content pipeline with Claude and OpenAI
  - automate research and content generation workflow
  - build AI-powered content automation system
  - use Remotion for automated video rendering
  - set up content research and publishing pipeline
---

# Marketing Content Pipeline Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the **Ultimate AI Content Pipeline** - a comprehensive TypeScript-based automation system that handles the entire content creation workflow from research and scriptwriting to automated video generation and publishing.

## What This Project Does

The Marketing Content Pipeline is an all-in-one content automation system that:

- **Auto-crawls** latest news from TechCrunch, a16z, Twitter/X, LinkedIn within 24 hours
- **Generates content** in multiple formats (toplist, POV, case study, how-to) using Claude 3 and OpenAI
- **Supports multilingual** output (English and Vietnamese) with customizable tone
- **Renders videos** automatically using Remotion for social media platforms
- **Publishes automatically** to Facebook Pages and other platforms
- Built with **Next.js, TypeScript, and modern AI APIs**

## Installation

### Prerequisites

```bash
# Node.js 18+ required
node --version

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

### Environment Configuration

Create a `.env.local` file in the root directory:

```env
# AI Provider Keys
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Social Media
FACEBOOK_PAGE_ACCESS_TOKEN=your_fb_token
FACEBOOK_PAGE_ID=your_page_id

# Optional: Video Rendering
REMOTION_LICENSE_KEY=your_remotion_license

# Database (if applicable)
DATABASE_URL=your_database_url

# Application
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

### Run Development Server

```bash
npm run dev
# or
yarn dev
# or
pnpm dev
```

Navigate to `http://localhost:3000` to access the application.

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/             
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── research/    # Content research & crawling
│   │   ├── video/       # Remotion video generation
│   │   └── publishing/  # Auto-posting logic
│   ├── types/           # TypeScript definitions
│   └── utils/           # Helper functions
├── public/              # Static assets
├── remotion/            # Remotion video templates
└── .env.local          # Environment variables
```

## Core APIs and Usage

### 1. Research & Content Crawling

```typescript
// src/lib/research/crawler.ts
import { RapidAPIClient } from '@/lib/research/rapid-api';

interface ResearchOptions {
  keyword: string;
  sources?: string[];
  timeRange?: '24h' | '7d' | '30d';
  language?: 'en' | 'vi';
}

export async function researchTopic(options: ResearchOptions) {
  const client = new RapidAPIClient(process.env.RAPIDAPI_KEY!);
  
  const results = await client.searchNews({
    query: options.keyword,
    sources: options.sources || ['techcrunch', 'a16z', 'twitter'],
    timeRange: options.timeRange || '24h'
  });
  
  return {
    articles: results.articles,
    insights: await extractInsights(results),
    trending: results.trending
  };
}

async function extractInsights(data: any) {
  // Use AI to extract key insights
  const { Anthropic } = await import('@anthropic-ai/sdk');
  const client = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY
  });
  
  const response = await client.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 2000,
    messages: [{
      role: 'user',
      content: `Analyze these articles and extract key insights: ${JSON.stringify(data)}`
    }]
  });
  
  return response.content[0].text;
}
```

### 2. AI Content Generation

```typescript
// src/lib/ai/content-generator.ts
import OpenAI from 'openai';
import Anthropic from '@anthropic-ai/sdk';

type ContentFormat = 'toplist' | 'pov' | 'case-study' | 'how-to';
type ContentTone = 'expert' | 'friendly' | 'humorous';

interface GenerateContentOptions {
  topic: string;
  format: ContentFormat;
  tone: ContentTone;
  language: 'en' | 'vi';
  researchData?: any;
}

export async function generateContent(options: GenerateContentOptions) {
  const { topic, format, tone, language, researchData } = options;
  
  // Use Claude for content generation
  const anthropic = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY
  });
  
  const systemPrompt = buildSystemPrompt(format, tone, language);
  const userPrompt = buildUserPrompt(topic, researchData);
  
  const response = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4000,
    system: systemPrompt,
    messages: [{
      role: 'user',
      content: userPrompt
    }]
  });
  
  return {
    title: extractTitle(response.content[0].text),
    content: response.content[0].text,
    metadata: {
      format,
      tone,
      language,
      wordCount: countWords(response.content[0].text)
    }
  };
}

function buildSystemPrompt(format: ContentFormat, tone: ContentTone, lang: 'en' | 'vi'): string {
  const formatInstructions = {
    'toplist': 'Create a numbered list article with clear rankings',
    'pov': 'Write from a specific point of view with strong opinions',
    'case-study': 'Analyze a real case with data and insights',
    'how-to': 'Provide step-by-step actionable instructions'
  };
  
  const toneInstructions = {
    'expert': 'Use professional, authoritative language with industry terms',
    'friendly': 'Use conversational, approachable language',
    'humorous': 'Add wit and humor while staying informative'
  };
  
  return `You are a ${tone} content writer. ${formatInstructions[format]}. ${toneInstructions[tone]}. Write in ${lang === 'vi' ? 'Vietnamese' : 'English'}.`;
}
```

### 3. Video Generation with Remotion

```typescript
// src/lib/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  content: string;
  title: string;
  style?: 'minimal' | 'dynamic' | 'professional';
  platform?: 'reels' | 'tiktok' | 'shorts' | 'square';
}

export async function renderContentVideo(config: VideoConfig) {
  const { content, title, style = 'minimal', platform = 'reels' } = config;
  
  // Platform-specific dimensions
  const dimensions = {
    reels: { width: 1080, height: 1920 },
    tiktok: { width: 1080, height: 1920 },
    shorts: { width: 1080, height: 1920 },
    square: { width: 1080, height: 1080 }
  };
  
  const compositionId = `content-video-${style}`;
  const bundleLocation = await bundle(
    path.join(process.cwd(), 'remotion/index.ts')
  );
  
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: {
      title,
      content: parseContentForVideo(content),
      style
    }
  });
  
  const outputLocation = path.join(
    process.cwd(),
    'public/videos',
    `${Date.now()}-${platform}.mp4`
  );
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    ...dimensions[platform]
  });
  
  return outputLocation;
}

function parseContentForVideo(content: string) {
  // Parse content into video-friendly segments
  const lines = content.split('\n').filter(line => line.trim());
  return lines.map((line, index) => ({
    id: index,
    text: line,
    duration: calculateDuration(line)
  }));
}
```

### 4. Automated Publishing

```typescript
// src/lib/publishing/facebook.ts
interface PublishOptions {
  content: string;
  imageUrl?: string;
  videoUrl?: string;
  scheduledTime?: Date;
}

export async function publishToFacebook(options: PublishOptions) {
  const { content, imageUrl, videoUrl, scheduledTime } = options;
  
  const pageId = process.env.FACEBOOK_PAGE_ID!;
  const accessToken = process.env.FACEBOOK_PAGE_ACCESS_TOKEN!;
  
  const endpoint = `https://graph.facebook.com/v18.0/${pageId}/feed`;
  
  const payload: any = {
    message: content,
    access_token: accessToken
  };
  
  if (videoUrl) {
    payload.link = videoUrl;
  } else if (imageUrl) {
    payload.link = imageUrl;
  }
  
  if (scheduledTime) {
    payload.published = false;
    payload.scheduled_publish_time = Math.floor(scheduledTime.getTime() / 1000);
  }
  
  const response = await fetch(endpoint, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(payload)
  });
  
  return await response.json();
}
```

## Complete Workflow Example

```typescript
// src/app/api/pipeline/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { researchTopic } from '@/lib/research/crawler';
import { generateContent } from '@/lib/ai/content-generator';
import { renderContentVideo } from '@/lib/video/renderer';
import { publishToFacebook } from '@/lib/publishing/facebook';

export async function POST(request: NextRequest) {
  try {
    const { keyword, format, language, autoPublish } = await request.json();
    
    // Step 1: Research
    console.log('Starting research...');
    const research = await researchTopic({
      keyword,
      timeRange: '24h',
      language
    });
    
    // Step 2: Generate Content
    console.log('Generating content...');
    const content = await generateContent({
      topic: keyword,
      format: format || 'toplist',
      tone: 'expert',
      language: language || 'en',
      researchData: research
    });
    
    // Step 3: Render Video
    console.log('Rendering video...');
    const videoPath = await renderContentVideo({
      content: content.content,
      title: content.title,
      style: 'dynamic',
      platform: 'reels'
    });
    
    // Step 4: Publish (if enabled)
    let publishResult = null;
    if (autoPublish) {
      console.log('Publishing to Facebook...');
      publishResult = await publishToFacebook({
        content: content.content,
        videoUrl: `${process.env.NEXT_PUBLIC_APP_URL}/videos/${path.basename(videoPath)}`
      });
    }
    
    return NextResponse.json({
      success: true,
      data: {
        research: research.insights,
        content,
        videoPath,
        publishResult
      }
    });
    
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

## Frontend Integration

```typescript
// src/components/ContentPipeline.tsx
'use client';

import { useState } from 'react';

export default function ContentPipeline() {
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState(null);
  
  async function runPipeline(formData: FormData) {
    setLoading(true);
    
    try {
      const response = await fetch('/api/pipeline', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          keyword: formData.get('keyword'),
          format: formData.get('format'),
          language: formData.get('language'),
          autoPublish: formData.get('autoPublish') === 'on'
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
    <div className="max-w-4xl mx-auto p-6">
      <h1 className="text-3xl font-bold mb-6">AI Content Pipeline</h1>
      
      <form action={runPipeline} className="space-y-4">
        <input
          name="keyword"
          placeholder="Enter topic or keyword..."
          className="w-full p-3 border rounded"
          required
        />
        
        <select name="format" className="w-full p-3 border rounded">
          <option value="toplist">Top List</option>
          <option value="pov">Point of View</option>
          <option value="case-study">Case Study</option>
          <option value="how-to">How-To Guide</option>
        </select>
        
        <select name="language" className="w-full p-3 border rounded">
          <option value="en">English</option>
          <option value="vi">Vietnamese</option>
        </select>
        
        <label className="flex items-center gap-2">
          <input type="checkbox" name="autoPublish" />
          <span>Auto-publish to Facebook</span>
        </label>
        
        <button
          type="submit"
          disabled={loading}
          className="w-full bg-blue-600 text-white p-3 rounded disabled:opacity-50"
        >
          {loading ? 'Processing...' : 'Generate Content'}
        </button>
      </form>
      
      {result && (
        <div className="mt-8 p-6 bg-gray-50 rounded">
          <h2 className="text-xl font-bold mb-4">Result</h2>
          <pre className="whitespace-pre-wrap">{JSON.stringify(result, null, 2)}</pre>
        </div>
      )}
    </div>
  );
}
```

## Common Patterns

### Error Handling with Retries

```typescript
async function withRetry<T>(
  fn: () => Promise<T>,
  maxRetries: number = 3,
  delay: number = 1000
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      await new Promise(resolve => setTimeout(resolve, delay * (i + 1)));
    }
  }
  throw new Error('Max retries exceeded');
}

// Usage
const content = await withRetry(() => generateContent(options));
```

### Rate Limiting

```typescript
import pLimit from 'p-limit';

const limit = pLimit(3); // Max 3 concurrent requests

const results = await Promise.all(
  keywords.map(keyword =>
    limit(() => researchTopic({ keyword, timeRange: '24h' }))
  )
);
```

## Troubleshooting

### API Key Issues
- Verify all API keys are set in `.env.local`
- Check API key permissions (especially Facebook tokens)
- Ensure Anthropic/OpenAI accounts have sufficient credits

### Video Rendering Fails
- Check Remotion license key if using licensed features
- Ensure ffmpeg is installed: `brew install ffmpeg` (macOS)
- Verify sufficient disk space for video output

### Content Quality Issues
- Adjust AI prompts in `buildSystemPrompt()` function
- Provide more detailed research data to AI models
- Experiment with different models (Claude vs GPT-4)

### Publishing Errors
- Verify Facebook Page access token hasn't expired
- Check page permissions for posting
- Ensure scheduled publish time is at least 10 minutes in future

### Memory Issues
- Limit concurrent API calls with `pLimit`
- Process large batches in chunks
- Clear video files after publishing to save disk space
