---
name: ultimate-ai-content-pipeline
description: Automated content creation pipeline from research to video generation using Claude, OpenAI, and Remotion
triggers:
  - how do I set up the AI content pipeline for automated marketing
  - generate automated content from research to video with this pipeline
  - create marketing content automatically using Claude and OpenAI
  - set up the content automation system with video generation
  - how to configure the AI content research and scriptwriting pipeline
  - automate content creation from keywords to published videos
  - use the marketing pipeline for automatic content generation
  - configure AI-powered content workflow with Remotion
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This TypeScript-based system automates the entire content creation workflow: from researching trending topics to generating scripts and rendering videos. It crawls real-time data from sources like TechCrunch, a16z, Twitter/X, and LinkedIn, then uses Claude/OpenAI to create multi-format content (articles, videos, infographics) optimized for different platforms.

## What It Does

- **Auto-Research**: Scrapes and analyzes fresh content from major news sources within the last 24 hours
- **AI Content Generation**: Creates articles in multiple formats (toplist, POV, case study, how-to) using Claude 3 or OpenAI
- **Multilingual Support**: Generates content in both English and Vietnamese with customizable tone
- **Video Rendering**: Automatically creates short-form videos using Remotion for Reels/TikTok/Shorts
- **Visual Assets**: Generates infographics and images from written content

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
pnpm install
```

### Environment Configuration

Create a `.env.local` file in the project root:

```env
# AI APIs
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Optional: Custom endpoints
NEXT_PUBLIC_API_URL=http://localhost:3000

# Remotion (for video generation)
REMOTION_AWS_ACCESS_KEY_ID=your_aws_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret
```

### Start Development Server

```bash
npm run dev
# Server runs on http://localhost:3000
```

## Core Architecture

### Project Structure

```
├── app/                    # Next.js app router
│   ├── api/               # API routes
│   │   ├── research/      # Content research endpoints
│   │   ├── generate/      # Content generation endpoints
│   │   └── render/        # Video rendering endpoints
│   └── page.tsx           # Main UI
├── lib/                   # Core logic
│   ├── ai/               # AI integration (Claude, OpenAI)
│   ├── scrapers/         # Web scraping utilities
│   └── utils/            # Helper functions
├── remotion/             # Video templates
└── public/               # Static assets
```

## Key APIs & Usage

### 1. Research API

Crawl and analyze trending topics:

```typescript
// app/api/research/route.ts
import { NextResponse } from 'next/server';
import { researchTopic } from '@/lib/scrapers/news-scraper';

export async function POST(request: Request) {
  const { keyword, sources, timeframe } = await request.json();
  
  try {
    const results = await researchTopic({
      keyword,
      sources: sources || ['techcrunch', 'a16z', 'twitter'],
      hours: timeframe || 24
    });
    
    return NextResponse.json({
      success: true,
      data: results,
      insights: results.insights,
      articles: results.articles
    });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

**Client Usage:**

```typescript
// Trigger research
const response = await fetch('/api/research', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    keyword: 'AI automation',
    sources: ['techcrunch', 'linkedin', 'twitter'],
    timeframe: 24
  })
});

const { data, insights } = await response.json();
```

### 2. Content Generation API

Generate content using AI:

```typescript
// lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

export interface ContentRequest {
  topic: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  researchData?: any;
}

export async function generateContent(
  request: ContentRequest,
  provider: 'claude' | 'openai' = 'claude'
) {
  if (provider === 'claude') {
    const anthropic = new Anthropic({
      apiKey: process.env.ANTHROPIC_API_KEY
    });
    
    const message = await anthropic.messages.create({
      model: 'claude-3-5-sonnet-20241022',
      max_tokens: 4096,
      messages: [{
        role: 'user',
        content: buildPrompt(request)
      }]
    });
    
    return parseContentResponse(message.content);
  } else {
    const openai = new OpenAI({
      apiKey: process.env.OPENAI_API_KEY
    });
    
    const completion = await openai.chat.completions.create({
      model: 'gpt-4-turbo-preview',
      messages: [{
        role: 'user',
        content: buildPrompt(request)
      }]
    });
    
    return parseContentResponse(completion.choices[0].message.content);
  }
}

function buildPrompt(request: ContentRequest): string {
  const formatInstructions = {
    'toplist': 'Create a top 10 list format with compelling headlines',
    'pov': 'Write from a unique perspective with strong opinions',
    'case-study': 'Analyze with data, examples, and takeaways',
    'how-to': 'Step-by-step tutorial with actionable tips'
  };
  
  return `
Topic: ${request.topic}
Format: ${formatInstructions[request.format]}
Language: ${request.language === 'vi' ? 'Vietnamese' : 'English'}
Tone: ${request.tone}

${request.researchData ? `Research Context:\n${JSON.stringify(request.researchData, null, 2)}` : ''}

Create comprehensive, engaging content following the specified format.
Include:
- Attention-grabbing headline
- Well-structured body with subheadings
- Data points and statistics where relevant
- Actionable takeaways
- SEO-optimized structure
`;
}
```

**API Route:**

```typescript
// app/api/generate/route.ts
export async function POST(request: Request) {
  const contentRequest = await request.json();
  
  const content = await generateContent(
    contentRequest,
    contentRequest.provider || 'claude'
  );
  
  return NextResponse.json({
    success: true,
    content: content.text,
    metadata: content.metadata
  });
}
```

### 3. Video Rendering with Remotion

```typescript
// lib/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

export interface VideoConfig {
  contentText: string;
  title: string;
  platform: 'reels' | 'tiktok' | 'shorts';
  style?: 'minimal' | 'dynamic' | 'infographic';
}

export async function renderContentVideo(config: VideoConfig) {
  const bundleLocation = await bundle(
    path.join(process.cwd(), 'remotion/index.ts')
  );
  
  const compositions = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo'
  });
  
  const dimensions = getPlatformDimensions(config.platform);
  
  await renderMedia({
    composition: compositions,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: `out/${config.title}-${Date.now()}.mp4`,
    inputProps: {
      title: config.title,
      content: config.contentText,
      style: config.style || 'minimal'
    },
    ...dimensions
  });
}

function getPlatformDimensions(platform: string) {
  const configs = {
    'reels': { width: 1080, height: 1920 },
    'tiktok': { width: 1080, height: 1920 },
    'shorts': { width: 1080, height: 1920 }
  };
  return configs[platform] || configs.reels;
}
```

**Remotion Composition:**

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig } from 'remotion';
import React from 'react';

interface ContentVideoProps {
  title: string;
  content: string;
  style: 'minimal' | 'dynamic' | 'infographic';
}

export const ContentVideo: React.FC<ContentVideoProps> = ({
  title,
  content,
  style
}) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  
  const opacity = Math.min(1, frame / (fps * 0.5));
  
  return (
    <AbsoluteFill style={{ backgroundColor: '#000' }}>
      <div style={{
        opacity,
        padding: 60,
        color: '#fff',
        fontFamily: 'Inter, sans-serif'
      }}>
        <h1 style={{
          fontSize: 72,
          fontWeight: 'bold',
          marginBottom: 40
        }}>
          {title}
        </h1>
        <p style={{
          fontSize: 36,
          lineHeight: 1.6
        }}>
          {content.slice(0, 200)}...
        </p>
      </div>
    </AbsoluteFill>
  );
};
```

## Common Workflows

### Full Pipeline: Keyword to Video

```typescript
// lib/pipeline/full-automation.ts
export async function runFullPipeline(keyword: string) {
  // Step 1: Research
  const research = await fetch('/api/research', {
    method: 'POST',
    body: JSON.stringify({ keyword, timeframe: 24 })
  }).then(r => r.json());
  
  // Step 2: Generate Content
  const content = await fetch('/api/generate', {
    method: 'POST',
    body: JSON.stringify({
      topic: keyword,
      format: 'toplist',
      language: 'en',
      tone: 'expert',
      researchData: research.data,
      provider: 'claude'
    })
  }).then(r => r.json());
  
  // Step 3: Render Video
  const video = await fetch('/api/render', {
    method: 'POST',
    body: JSON.stringify({
      contentText: content.content,
      title: content.metadata.title,
      platform: 'reels',
      style: 'dynamic'
    })
  }).then(r => r.json());
  
  return {
    research,
    content,
    video
  };
}
```

### Batch Content Generation

```typescript
// Generate multiple formats from one research
async function generateMultipleFormats(keyword: string) {
  const research = await researchTopic({ keyword });
  
  const formats = ['toplist', 'how-to', 'case-study'] as const;
  
  const contents = await Promise.all(
    formats.map(format =>
      generateContent({
        topic: keyword,
        format,
        language: 'en',
        tone: 'expert',
        researchData: research
      })
    )
  );
  
  return contents;
}
```

## Configuration

### AI Provider Selection

```typescript
// lib/config/ai-config.ts
export const aiConfig = {
  defaultProvider: process.env.AI_PROVIDER || 'claude',
  fallbackProvider: 'openai',
  
  models: {
    claude: 'claude-3-5-sonnet-20241022',
    openai: 'gpt-4-turbo-preview'
  },
  
  maxTokens: {
    research: 2048,
    generation: 4096,
    summary: 1024
  }
};
```

### Content Templates

```typescript
// lib/templates/content-templates.ts
export const templates = {
  toplist: {
    structure: ['headline', 'intro', 'items', 'conclusion'],
    itemCount: 10,
    includeStats: true
  },
  
  howTo: {
    structure: ['problem', 'solution', 'steps', 'tips', 'conclusion'],
    stepCount: 5,
    includeVisuals: true
  },
  
  caseStudy: {
    structure: ['background', 'challenge', 'solution', 'results', 'takeaways'],
    includeMetrics: true,
    dataRequired: true
  }
};
```

## Troubleshooting

### API Rate Limits

```typescript
// lib/utils/rate-limiter.ts
import { RateLimiter } from 'limiter';

const claudeLimiter = new RateLimiter({
  tokensPerInterval: 50,
  interval: 'minute'
});

export async function callClaudeWithLimit(fn: () => Promise<any>) {
  await claudeLimiter.removeTokens(1);
  return fn();
}
```

### Error Handling

```typescript
// lib/utils/error-handler.ts
export async function safeAPICall<T>(
  fn: () => Promise<T>,
  fallback?: () => Promise<T>
): Promise<T> {
  try {
    return await fn();
  } catch (error) {
    console.error('API call failed:', error);
    
    if (fallback) {
      console.log('Attempting fallback...');
      return await fallback();
    }
    
    throw error;
  }
}

// Usage
const content = await safeAPICall(
  () => generateContent(request, 'claude'),
  () => generateContent(request, 'openai') // Fallback to OpenAI
);
```

### Video Rendering Issues

If Remotion rendering fails:

```bash
# Install required system dependencies
npm install @remotion/renderer @remotion/bundler

# For Lambda rendering (production)
npm install @remotion/lambda

# Verify FFmpeg installation
npx remotion versions
```

### Common Errors

**"Anthropic API key not found"**
- Ensure `ANTHROPIC_API_KEY` is set in `.env.local`
- Restart dev server after adding env vars

**"Research returned no results"**
- Check `RAPIDAPI_KEY` is valid
- Verify source endpoints are accessible
- Try different timeframe (24h → 48h)

**"Video rendering timeout"**
- Reduce video length or complexity
- Use Remotion Lambda for production workloads
- Check available disk space for output files
