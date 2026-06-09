---
name: marketing-pipeline-share-automation
description: Ultimate AI content pipeline from research to video generation with automated crawling, multi-format content creation, and video rendering
triggers:
  - how do I automate content creation with marketing pipeline
  - set up AI content automation system
  - generate videos from content automatically
  - crawl news and create social media posts
  - use marketing pipeline for content generation
  - automate content research and video creation
  - create multi-language content with AI pipeline
  - render videos from text content automatically
---

# Marketing Pipeline Share - AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to help developers use **Marketing Pipeline Share**, a comprehensive content automation system that handles everything from research (crawling news sources) to content generation in multiple formats, and automatic video rendering using Remotion.

## What It Does

Marketing Pipeline Share is an end-to-end content pipeline that:

- **Auto-crawls** trending news from TechCrunch, a16z, Twitter/X, LinkedIn
- **Generates content** in multiple formats (toplist, POV, case study, how-to) using Claude/OpenAI
- **Multi-language support** (English & Vietnamese) with customizable tone
- **Renders videos** automatically using Remotion for social media platforms
- **Optimizes for platforms** like Reels, TikTok, Shorts with proper aspect ratios

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
```

### Environment Configuration

Create a `.env.local` file in the root directory:

```bash
# AI Model APIs
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# Content Crawling (RapidAPI)
RAPIDAPI_KEY=your_rapidapi_key

# Next.js Configuration
NEXT_PUBLIC_API_URL=http://localhost:3000

# Remotion Video Rendering
REMOTION_AWS_ACCESS_KEY_ID=your_aws_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret
```

### Run Development Server

```bash
npm run dev
# or
yarn dev
```

Access at `http://localhost:3000`

## Core API Endpoints

### 1. Content Research API

**Endpoint:** `POST /api/research`

Crawls and analyzes trending content from multiple sources.

```typescript
// app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';

export async function POST(request: NextRequest) {
  const { keyword, sources, timeRange } = await request.json();
  
  // Crawl from multiple sources
  const results = await crawlSources({
    keyword,
    sources: sources || ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    hours: timeRange || 24
  });
  
  // Analyze and extract insights
  const insights = await analyzeWithAI(results);
  
  return NextResponse.json({ insights, rawData: results });
}
```

**Usage Example:**

```typescript
const response = await fetch('/api/research', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    keyword: 'AI automation',
    sources: ['techcrunch', 'twitter'],
    timeRange: 24
  })
});

const { insights, rawData } = await response.json();
```

### 2. Content Generation API

**Endpoint:** `POST /api/generate-content`

Generates content in multiple formats using AI models.

```typescript
// app/api/generate-content/route.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

export async function POST(request: NextRequest) {
  const { 
    insights, 
    format, 
    language, 
    tone,
    model 
  } = await request.json();
  
  const prompt = buildPrompt({ insights, format, language, tone });
  
  let content;
  if (model === 'claude') {
    const anthropic = new Anthropic({
      apiKey: process.env.ANTHROPIC_API_KEY,
    });
    
    const message = await anthropic.messages.create({
      model: 'claude-3-5-sonnet-20241022',
      max_tokens: 4096,
      messages: [{
        role: 'user',
        content: prompt
      }]
    });
    
    content = message.content[0].text;
  } else {
    const openai = new OpenAI({
      apiKey: process.env.OPENAI_API_KEY,
    });
    
    const completion = await openai.chat.completions.create({
      model: 'gpt-4-turbo-preview',
      messages: [{ role: 'user', content: prompt }],
    });
    
    content = completion.choices[0].message.content;
  }
  
  return NextResponse.json({ content, format, language });
}
```

**Usage Example:**

```typescript
const response = await fetch('/api/generate-content', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    insights: researchData,
    format: 'toplist', // or 'pov', 'case-study', 'how-to'
    language: 'vi', // or 'en'
    tone: 'professional', // or 'friendly', 'humorous'
    model: 'claude' // or 'openai'
  })
});

const { content } = await response.json();
```

### 3. Video Rendering API

**Endpoint:** `POST /api/render-video`

Renders videos from content using Remotion.

```typescript
// app/api/render-video/route.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

export async function POST(request: NextRequest) {
  const { content, platform, style } = await request.json();
  
  // Bundle Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: (config) => config,
  });
  
  // Select composition based on platform
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: platform === 'reels' ? 'ReelsVideo' : 'ShortsVideo',
    inputProps: {
      content,
      style
    },
  });
  
  // Render video
  const outputLocation = `./public/videos/output-${Date.now()}.mp4`;
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: { content, style },
  });
  
  return NextResponse.json({ 
    videoUrl: outputLocation.replace('./public', ''),
    duration: composition.durationInFrames / composition.fps
  });
}
```

**Usage Example:**

```typescript
const response = await fetch('/api/render-video', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    content: generatedContent,
    platform: 'reels', // or 'tiktok', 'shorts'
    style: 'modern' // or 'minimal', 'energetic'
  })
});

const { videoUrl, duration } = await response.json();
```

## Common Patterns

### Full Pipeline Automation

```typescript
// lib/content-pipeline.ts
export async function runFullPipeline(keyword: string) {
  try {
    // Step 1: Research
    console.log('🔍 Starting research...');
    const researchRes = await fetch('/api/research', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ keyword, timeRange: 24 })
    });
    const { insights } = await researchRes.json();
    
    // Step 2: Generate Content (Multiple Formats)
    console.log('✍️ Generating content...');
    const formats = ['toplist', 'pov', 'how-to'];
    const contentPromises = formats.map(format =>
      fetch('/api/generate-content', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          insights,
          format,
          language: 'vi',
          tone: 'professional',
          model: 'claude'
        })
      }).then(r => r.json())
    );
    
    const contents = await Promise.all(contentPromises);
    
    // Step 3: Render Videos
    console.log('🎬 Rendering videos...');
    const videoPromises = contents.map((content, idx) =>
      fetch('/api/render-video', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          content: content.content,
          platform: ['reels', 'tiktok', 'shorts'][idx],
          style: 'modern'
        })
      }).then(r => r.json())
    );
    
    const videos = await Promise.all(videoPromises);
    
    console.log('✅ Pipeline complete!');
    return { contents, videos };
    
  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}
```

### Custom Remotion Composition

```typescript
// remotion/compositions/ReelsVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig } from 'remotion';
import { interpolate } from 'remotion';

export const ReelsVideo: React.FC<{ content: string; style: string }> = ({
  content,
  style
}) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  
  const opacity = interpolate(frame, [0, 30], [0, 1], {
    extrapolateRight: 'clamp',
  });
  
  const scale = interpolate(frame, [0, 30], [0.8, 1], {
    extrapolateRight: 'clamp',
  });
  
  return (
    <AbsoluteFill style={{ backgroundColor: '#000' }}>
      <div
        style={{
          opacity,
          transform: `scale(${scale})`,
          color: 'white',
          fontSize: '48px',
          textAlign: 'center',
          padding: '40px',
        }}
      >
        {content}
      </div>
    </AbsoluteFill>
  );
};
```

### Prompt Builder Utility

```typescript
// lib/prompt-builder.ts
export function buildPrompt({
  insights,
  format,
  language,
  tone
}: {
  insights: any;
  format: string;
  language: string;
  tone: string;
}) {
  const toneMap = {
    professional: 'chuyên nghiệp, có dẫn chứng',
    friendly: 'thân thiện, gần gũi',
    humorous: 'hài hước, sáng tạo'
  };
  
  const formatInstructions = {
    toplist: 'Tạo bài viết dạng Top 10/Top 5 với các mục được đánh số rõ ràng',
    pov: 'Viết theo góc nhìn cá nhân, thể hiện quan điểm độc đáo',
    'case-study': 'Phân tích case study chi tiết với số liệu cụ thể',
    'how-to': 'Hướng dẫn từng bước rõ ràng, dễ thực hiện'
  };
  
  return `
Dựa trên insights sau: ${JSON.stringify(insights)}

Hãy viết một bài ${format} bằng ${language === 'vi' ? 'Tiếng Việt' : 'English'}.
Giọng văn: ${toneMap[tone as keyof typeof toneMap]}
Format: ${formatInstructions[format as keyof typeof formatInstructions]}

Yêu cầu:
- Độ dài: 800-1200 từ
- Có emoji phù hợp
- Trích dẫn số liệu từ insights
- Kết thúc với CTA mạnh mẽ
`;
}
```

## Configuration Files

### Next.js Configuration

```typescript
// next.config.js
/** @type {import('next').NextConfig} */
const nextConfig = {
  reactStrictMode: true,
  swcMinify: true,
  images: {
    domains: ['pbs.twimg.com', 'techcrunch.com'],
  },
  env: {
    NEXT_PUBLIC_API_URL: process.env.NEXT_PUBLIC_API_URL,
  },
};

module.exports = nextConfig;
```

### Remotion Configuration

```typescript
// remotion.config.ts
import { Config } from '@remotion/cli/config';

Config.setVideoImageFormat('jpeg');
Config.setOverwriteOutput(true);
Config.setConcurrency(4);
Config.setCodec('h264');
```

## Troubleshooting

### API Rate Limiting

```typescript
// lib/rate-limiter.ts
const rateLimits = new Map<string, number[]>();

export function checkRateLimit(apiKey: string, maxRequests = 10, windowMs = 60000) {
  const now = Date.now();
  const requests = rateLimits.get(apiKey) || [];
  
  // Remove old requests outside window
  const recentRequests = requests.filter(time => now - time < windowMs);
  
  if (recentRequests.length >= maxRequests) {
    throw new Error('Rate limit exceeded. Please try again later.');
  }
  
  recentRequests.push(now);
  rateLimits.set(apiKey, recentRequests);
}
```

### Video Rendering Timeouts

```typescript
// Increase timeout for large videos
export const config = {
  api: {
    bodyParser: {
      sizeLimit: '50mb',
    },
    responseLimit: false,
  },
  maxDuration: 300, // 5 minutes
};
```

### Content Quality Validation

```typescript
// lib/validators.ts
export function validateContent(content: string): boolean {
  const minLength = 500;
  const hasEmoji = /[\u{1F300}-\u{1F9FF}]/u.test(content);
  const hasStructure = content.includes('\n\n');
  
  if (content.length < minLength) {
    throw new Error(`Content too short. Minimum ${minLength} characters.`);
  }
  
  if (!hasStructure) {
    console.warn('Content may lack proper structure');
  }
  
  return true;
}
```

### Memory Management for Large Crawls

```typescript
// lib/crawler-optimizer.ts
export async function batchCrawl(sources: string[], batchSize = 3) {
  const results = [];
  
  for (let i = 0; i < sources.length; i += batchSize) {
    const batch = sources.slice(i, i + batchSize);
    const batchResults = await Promise.all(
      batch.map(source => crawlSource(source))
    );
    results.push(...batchResults);
    
    // Allow garbage collection between batches
    if (i + batchSize < sources.length) {
      await new Promise(resolve => setTimeout(resolve, 1000));
    }
  }
  
  return results;
}
```

## Best Practices

1. **Always validate environment variables** before running pipelines
2. **Use batch processing** for multiple content generations to avoid API limits
3. **Cache research results** to avoid redundant crawling within the same day
4. **Monitor video rendering** progress with webhooks or polling
5. **Implement retry logic** for API calls with exponential backoff
6. **Store generated content** in a database for version control and reuse

This skill enables comprehensive automation of content marketing workflows from research to final video output.
