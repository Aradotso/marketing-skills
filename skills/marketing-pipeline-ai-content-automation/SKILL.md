---
name: marketing-pipeline-ai-content-automation
description: Automated content pipeline using AI (Claude/OpenAI) to research, generate scripts, and create videos with Remotion
triggers:
  - how do I automate content creation with AI
  - set up automated marketing pipeline
  - generate blog posts and videos automatically
  - use Claude and OpenAI for content automation
  - create video content from text with Remotion
  - build automated content research pipeline
  - schedule and publish AI-generated content
  - automate social media content creation
---

# Marketing Pipeline AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with **marketing-pipeline-share**, a comprehensive TypeScript-based content automation system that handles the entire content lifecycle: research, scriptwriting, content generation, and video rendering. It crawls fresh news from sources like TechCrunch and Twitter, uses Claude/OpenAI to generate multi-format content in multiple languages, and renders videos using Remotion.

## What It Does

The marketing pipeline automates:
- **Auto-research**: Crawls real-time news from major sources (24h freshness)
- **Multi-format content generation**: Toplists, POV articles, case studies, how-tos
- **Bilingual output**: Simultaneous English and Vietnamese content
- **Video rendering**: Converts text to infographics and short videos via Remotion
- **Platform optimization**: Exports for Reels, TikTok, Shorts

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

Create a `.env.local` file in the project root:

```bash
# AI Provider APIs
OPENAI_API_KEY=your_openai_key_here
ANTHROPIC_API_KEY=your_claude_key_here

# Research/Crawling APIs
RAPIDAPI_KEY=your_rapidapi_key_here

# Optional: Database
DATABASE_URL=your_database_connection_string

# Optional: Next.js Config
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/                 # Next.js app router
│   ├── components/          # React components
│   ├── lib/
│   │   ├── ai/             # AI integrations (Claude, OpenAI)
│   │   ├── crawler/        # Web scraping modules
│   │   ├── content/        # Content generation logic
│   │   └── video/          # Remotion video rendering
│   └── types/              # TypeScript definitions
├── remotion/               # Remotion video templates
└── public/                 # Static assets
```

## Core API Usage

### 1. Content Research & Crawling

```typescript
import { researchTopic } from '@/lib/crawler/research';

async function gatherInsights(keyword: string) {
  const research = await researchTopic({
    keyword,
    sources: ['techcrunch', 'twitter', 'linkedin'],
    timeframe: '24h',
    maxResults: 20
  });

  return research; // { articles: [], insights: [], trending: [] }
}
```

### 2. AI Content Generation with Claude

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

async function generateContent(research: any, format: string) {
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [{
      role: 'user',
      content: `Based on this research: ${JSON.stringify(research)}
      
Generate a ${format} article with:
- Engaging headline
- Data-backed insights
- Bilingual output (English & Vietnamese)
- SEO optimized structure`
    }]
  });

  return message.content[0].text;
}
```

### 3. OpenAI Integration Alternative

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateWithGPT(prompt: string, tone: string) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are a ${tone} content writer specializing in marketing content.`
      },
      {
        role: 'user',
        content: prompt
      }
    ],
    temperature: 0.7,
  });

  return completion.choices[0].message.content;
}
```

### 4. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

async function renderContentVideo(content: {
  title: string;
  points: string[];
  duration: number;
}) {
  const compositionId = 'ContentVideo';
  const bundleLocation = await bundle(
    path.join(process.cwd(), 'remotion/index.ts')
  );

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: content,
  });

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: `out/${content.title}.mp4`,
    inputProps: content,
  });

  return `out/${content.title}.mp4`;
}
```

## Common Patterns

### End-to-End Content Pipeline

```typescript
import { researchTopic } from '@/lib/crawler/research';
import { generateContent } from '@/lib/ai/claude';
import { renderContentVideo } from '@/lib/video/remotion';

async function fullContentPipeline(keyword: string) {
  try {
    // Step 1: Research
    console.log('🔍 Researching topic...');
    const research = await researchTopic({
      keyword,
      sources: ['techcrunch', 'twitter'],
      timeframe: '24h'
    });

    // Step 2: Generate content
    console.log('✍️ Generating content...');
    const content = await generateContent(research, 'toplist');
    
    // Step 3: Parse for video
    const videoData = {
      title: content.headline,
      points: content.keyPoints,
      duration: 30 // seconds
    };

    // Step 4: Render video
    console.log('🎬 Rendering video...');
    const videoPath = await renderContentVideo(videoData);

    return {
      article: content,
      video: videoPath,
      research
    };
  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}

// Usage
fullContentPipeline('AI marketing automation').then(result => {
  console.log('✅ Content ready:', result);
});
```

### Multi-format Content Generation

```typescript
type ContentFormat = 'toplist' | 'pov' | 'case-study' | 'how-to';

async function generateMultiFormat(
  keyword: string,
  formats: ContentFormat[]
) {
  const research = await researchTopic({ keyword });
  
  const contentPromises = formats.map(async (format) => {
    const content = await generateContent(research, format);
    return { format, content };
  });

  return Promise.all(contentPromises);
}

// Generate 4 different formats simultaneously
const contents = await generateMultiFormat('content marketing trends', [
  'toplist',
  'pov',
  'case-study',
  'how-to'
]);
```

### Scheduled Content Generation

```typescript
import cron from 'node-cron';

// Run daily at 6 AM
cron.schedule('0 6 * * *', async () => {
  const topics = ['AI marketing', 'social media trends', 'SEO tips'];
  
  for (const topic of topics) {
    try {
      const result = await fullContentPipeline(topic);
      // Auto-publish or save to CMS
      await publishToPlatform(result);
    } catch (error) {
      console.error(`Failed for ${topic}:`, error);
    }
  }
});
```

## Running the Application

```bash
# Development mode
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Render Remotion videos only
npm run render
```

## API Routes (Next.js)

### Generate Content API

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';

export async function POST(request: NextRequest) {
  const { keyword, format, language } = await request.json();
  
  try {
    const research = await researchTopic({ keyword });
    const content = await generateContent(research, format);
    
    return NextResponse.json({
      success: true,
      content,
      language
    });
  } catch (error) {
    return NextResponse.json(
      { error: 'Content generation failed' },
      { status: 500 }
    );
  }
}
```

### Usage from frontend:

```typescript
const response = await fetch('/api/generate', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    keyword: 'AI marketing',
    format: 'toplist',
    language: 'en'
  })
});

const { content } = await response.json();
```

## Remotion Video Templates

### Basic Content Video Component

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, interpolate } from 'remotion';

export const ContentVideo: React.FC<{
  title: string;
  points: string[];
}> = ({ title, points }) => {
  const frame = useCurrentFrame();
  
  const titleOpacity = interpolate(
    frame,
    [0, 30],
    [0, 1],
    { extrapolateRight: 'clamp' }
  );

  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a1a' }}>
      <div style={{ opacity: titleOpacity, padding: 60 }}>
        <h1 style={{ color: 'white', fontSize: 72 }}>{title}</h1>
        <ul style={{ color: 'white', fontSize: 36, marginTop: 40 }}>
          {points.map((point, i) => (
            <li key={i} style={{ marginBottom: 20 }}>{point}</li>
          ))}
        </ul>
      </div>
    </AbsoluteFill>
  );
};
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement retry logic with exponential backoff
async function retryWithBackoff<T>(
  fn: () => Promise<T>,
  maxRetries = 3
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error: any) {
      if (error.status === 429 && i < maxRetries - 1) {
        const delay = Math.pow(2, i) * 1000;
        await new Promise(resolve => setTimeout(resolve, delay));
        continue;
      }
      throw error;
    }
  }
  throw new Error('Max retries exceeded');
}
```

### Memory Issues with Video Rendering

```typescript
// Render videos in batches
async function renderVideoBatch(contents: any[], batchSize = 3) {
  const results = [];
  
  for (let i = 0; i < contents.length; i += batchSize) {
    const batch = contents.slice(i, i + batchSize);
    const batchResults = await Promise.all(
      batch.map(c => renderContentVideo(c))
    );
    results.push(...batchResults);
    
    // Clean up between batches
    if (global.gc) global.gc();
  }
  
  return results;
}
```

### Missing Environment Variables

```typescript
// Validate required env vars on startup
function validateEnv() {
  const required = [
    'OPENAI_API_KEY',
    'ANTHROPIC_API_KEY',
    'RAPIDAPI_KEY'
  ];
  
  const missing = required.filter(key => !process.env[key]);
  
  if (missing.length > 0) {
    throw new Error(
      `Missing required environment variables: ${missing.join(', ')}`
    );
  }
}

validateEnv();
```

## Best Practices

1. **Cache research results** to avoid redundant API calls
2. **Use streaming responses** for long content generation
3. **Implement proper error handling** for each pipeline stage
4. **Monitor API costs** - Claude and OpenAI can be expensive
5. **Batch video rendering** to manage memory usage
6. **Version control prompts** separately for easy iteration
