---
name: marketing-pipeline-share-automation
description: AI-powered content automation pipeline from research to video generation with auto-posting capabilities
triggers:
  - automate content creation with AI pipeline
  - generate videos from content automatically
  - auto research and create marketing content
  - set up content automation workflow
  - build AI content generation pipeline
  - create automated social media posts with video
  - research and publish content automatically
  - generate multilingual content with AI
---

# Marketing Pipeline Share Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the Ultimate AI Content Pipeline - a complete automation system that handles content research, scriptwriting, auto-posting, and video generation. The system integrates Claude 3, OpenAI, and Remotion to create a fully automated content production line.

## What This Project Does

Marketing Pipeline Share is an end-to-end content automation platform that:

- **Auto-scans and researches** trending news from TechCrunch, a16z, Twitter/X, LinkedIn within 24 hours
- **Generates diverse content formats** (toplist, POV, case studies, how-to) in multiple languages (English/Vietnamese)
- **Renders videos and graphics automatically** using Remotion integration
- **Posts to social platforms** with optimized formatting for Reels, TikTok, Shorts
- **Personalizes tone and voice** for different target audiences

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
# AI API Keys
OPENAI_API_KEY=your_openai_key_here
ANTHROPIC_API_KEY=your_claude_key_here

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key_here

# Video Rendering
REMOTION_LICENSE_KEY=your_remotion_key_here

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Social Media APIs (optional)
FACEBOOK_PAGE_TOKEN=your_facebook_token
LINKEDIN_ACCESS_TOKEN=your_linkedin_token
```

### Running the Development Server

```bash
npm run dev
# or
yarn dev
```

Access at `http://localhost:3000`

## Core Architecture

```typescript
// Directory structure
/src
  /app               # Next.js app router pages
  /components        # React components
  /lib
    /ai              # AI integration (Claude, OpenAI)
    /research        # Content research modules
    /video           # Remotion video generation
    /publishers      # Social media posting
  /remotion          # Video templates
  /utils             # Helpers and utilities
```

## Key Features & Usage

### 1. Content Research Module

```typescript
// lib/research/scanner.ts
import { researchTopic } from '@/lib/research/scanner';

interface ResearchResult {
  sources: Array<{
    url: string;
    title: string;
    publishedAt: string;
    summary: string;
  }>;
  insights: string[];
  trendingTopics: string[];
}

async function runResearch(keyword: string): Promise<ResearchResult> {
  const result = await researchTopic({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeRange: '24h',
    maxResults: 20
  });
  
  return result;
}

// Usage
const research = await runResearch('AI automation');
console.log(research.insights);
```

### 2. AI Content Generation

```typescript
// lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

interface ContentConfig {
  topic: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  researchData: any;
}

async function generateContent(config: ContentConfig): Promise<string> {
  const prompt = `
Based on this research data: ${JSON.stringify(config.researchData)}

Create a ${config.format} article about "${config.topic}" in ${config.language}.
Tone: ${config.tone}
Include data-backed insights and recent trends.
`;

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [
      { role: 'user', content: prompt }
    ],
  });

  return message.content[0].type === 'text' ? message.content[0].text : '';
}

// Alternative with OpenAI
async function generateContentOpenAI(config: ContentConfig): Promise<string> {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are an expert content creator specializing in ${config.format} articles.`
      },
      {
        role: 'user',
        content: `Create content about ${config.topic} using this research: ${JSON.stringify(config.researchData)}`
      }
    ],
    temperature: 0.7,
  });

  return completion.choices[0].message.content || '';
}
```

### 3. Video Generation with Remotion

```typescript
// lib/video/generator.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  content: string;
  title: string;
  format: 'reels' | 'tiktok' | 'shorts'; // 9:16 aspect ratio
  duration: number; // in frames
}

async function generateVideo(config: VideoConfig): Promise<string> {
  const compositionId = 'ContentVideo';
  const bundleLocation = await bundle(
    path.join(process.cwd(), 'src/remotion/index.ts')
  );

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: {
      content: config.content,
      title: config.title,
    },
  });

  const outputLocation = path.join(
    process.cwd(),
    `out/${Date.now()}-${config.format}.mp4`
  );

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: {
      content: config.content,
      title: config.title,
    },
  });

  return outputLocation;
}

// Usage
const videoPath = await generateVideo({
  content: 'Your generated content here',
  title: 'AI Automation Trends 2026',
  format: 'reels',
  duration: 300, // 10 seconds at 30fps
});
```

### 4. Remotion Video Component

```typescript
// src/remotion/ContentVideo.tsx
import { AbsoluteFill, Sequence, useCurrentFrame, interpolate } from 'remotion';

interface ContentVideoProps {
  title: string;
  content: string;
}

export const ContentVideo: React.FC<ContentVideoProps> = ({ title, content }) => {
  const frame = useCurrentFrame();
  
  const opacity = interpolate(
    frame,
    [0, 30],
    [0, 1],
    { extrapolateRight: 'clamp' }
  );

  const scale = interpolate(
    frame,
    [0, 30],
    [0.8, 1],
    { extrapolateRight: 'clamp' }
  );

  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a2e' }}>
      <Sequence from={0} durationInFrames={90}>
        <AbsoluteFill
          style={{
            justifyContent: 'center',
            alignItems: 'center',
            opacity,
            transform: `scale(${scale})`,
          }}
        >
          <h1 style={{ 
            color: 'white', 
            fontSize: 60, 
            textAlign: 'center',
            padding: '0 40px',
          }}>
            {title}
          </h1>
        </AbsoluteFill>
      </Sequence>
      
      <Sequence from={90}>
        <AbsoluteFill style={{ 
          padding: 40, 
          justifyContent: 'center',
        }}>
          <p style={{ 
            color: 'white', 
            fontSize: 24, 
            lineHeight: 1.6,
          }}>
            {content}
          </p>
        </AbsoluteFill>
      </Sequence>
    </AbsoluteFill>
  );
};
```

### 5. Auto-Publishing to Social Media

```typescript
// lib/publishers/facebook.ts
interface PublishConfig {
  content: string;
  videoPath?: string;
  scheduledTime?: Date;
}

async function publishToFacebook(config: PublishConfig): Promise<string> {
  const pageToken = process.env.FACEBOOK_PAGE_TOKEN;
  const pageId = process.env.FACEBOOK_PAGE_ID;
  
  const formData = new FormData();
  formData.append('message', config.content);
  formData.append('access_token', pageToken!);
  
  if (config.videoPath) {
    const videoFile = await fetch(`file://${config.videoPath}`);
    const blob = await videoFile.blob();
    formData.append('source', blob);
  }

  const response = await fetch(
    `https://graph.facebook.com/v18.0/${pageId}/videos`,
    {
      method: 'POST',
      body: formData,
    }
  );

  const data = await response.json();
  return data.id;
}
```

## Complete Workflow Example

```typescript
// lib/workflow/full-pipeline.ts
import { researchTopic } from '@/lib/research/scanner';
import { generateContent } from '@/lib/ai/content-generator';
import { generateVideo } from '@/lib/video/generator';
import { publishToFacebook } from '@/lib/publishers/facebook';

interface PipelineConfig {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  generateVideo: boolean;
  autoPublish: boolean;
}

async function runFullPipeline(config: PipelineConfig) {
  // Step 1: Research
  console.log('🔍 Starting research...');
  const research = await researchTopic({
    keyword: config.keyword,
    sources: ['techcrunch', 'a16z', 'twitter'],
    timeRange: '24h',
    maxResults: 15,
  });

  // Step 2: Generate Content
  console.log('✍️ Generating content...');
  const content = await generateContent({
    topic: config.keyword,
    format: config.format,
    language: config.language,
    tone: 'expert',
    researchData: research,
  });

  // Step 3: Generate Video (optional)
  let videoPath: string | undefined;
  if (config.generateVideo) {
    console.log('🎬 Rendering video...');
    videoPath = await generateVideo({
      content: content.substring(0, 500), // First 500 chars for video
      title: config.keyword,
      format: 'reels',
      duration: 300,
    });
  }

  // Step 4: Publish (optional)
  if (config.autoPublish) {
    console.log('📤 Publishing...');
    const postId = await publishToFacebook({
      content,
      videoPath,
    });
    console.log(`✅ Published: ${postId}`);
  }

  return {
    content,
    videoPath,
    research,
  };
}

// Usage
const result = await runFullPipeline({
  keyword: 'AI automation trends 2026',
  format: 'toplist',
  language: 'en',
  generateVideo: true,
  autoPublish: false, // Set to true to auto-post
});

console.log('Pipeline completed!', result);
```

## API Routes (Next.js)

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runFullPipeline } from '@/lib/workflow/full-pipeline';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const { keyword, format, language, generateVideo, autoPublish } = body;

    const result = await runFullPipeline({
      keyword,
      format,
      language,
      generateVideo: generateVideo ?? true,
      autoPublish: autoPublish ?? false,
    });

    return NextResponse.json({
      success: true,
      data: result,
    });
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { success: false, error: (error as Error).message },
      { status: 500 }
    );
  }
}
```

## Configuration Patterns

### Research Sources Configuration

```typescript
// config/research.ts
export const RESEARCH_SOURCES = {
  techcrunch: {
    enabled: true,
    apiEndpoint: 'https://techcrunch.com/wp-json/wp/v2/posts',
    priority: 1,
  },
  a16z: {
    enabled: true,
    apiEndpoint: 'https://a16z.com/feed/',
    priority: 2,
  },
  twitter: {
    enabled: true,
    rapidApiEndpoint: 'twitter-api45.p.rapidapi.com',
    priority: 3,
  },
};

export const CONTENT_FORMATS = {
  toplist: {
    minItems: 5,
    maxItems: 10,
    includeImages: true,
  },
  pov: {
    sections: ['intro', 'opinion', 'evidence', 'conclusion'],
    wordCount: { min: 800, max: 1500 },
  },
  'case-study': {
    structure: ['problem', 'solution', 'results', 'takeaways'],
    includeMetrics: true,
  },
  'how-to': {
    stepFormat: 'numbered',
    includeVisuals: true,
  },
};
```

## Troubleshooting

### API Rate Limits

```typescript
// lib/utils/rate-limiter.ts
class RateLimiter {
  private requests: Map<string, number[]> = new Map();

  async checkLimit(key: string, maxRequests: number, windowMs: number): Promise<boolean> {
    const now = Date.now();
    const requests = this.requests.get(key) || [];
    
    // Remove old requests outside window
    const validRequests = requests.filter(time => now - time < windowMs);
    
    if (validRequests.length >= maxRequests) {
      return false; // Rate limit exceeded
    }
    
    validRequests.push(now);
    this.requests.set(key, validRequests);
    return true;
  }
}

export const rateLimiter = new RateLimiter();

// Usage in API calls
if (!await rateLimiter.checkLimit('openai', 60, 60000)) {
  throw new Error('Rate limit exceeded. Please wait.');
}
```

### Video Rendering Issues

```typescript
// Common fixes for Remotion errors
// 1. Memory issues - increase Node memory
// Run with: NODE_OPTIONS=--max_old_space_size=4096 npm run dev

// 2. Missing dependencies
// Ensure ffmpeg is installed:
// brew install ffmpeg (macOS)
// sudo apt-get install ffmpeg (Ubuntu)

// 3. Timeout issues - increase timeout
await renderMedia({
  composition,
  serveUrl: bundleLocation,
  codec: 'h264',
  outputLocation,
  timeoutInMilliseconds: 120000, // 2 minutes
});
```

### AI API Errors

```typescript
// lib/utils/ai-retry.ts
async function withRetry<T>(
  fn: () => Promise<T>,
  retries: number = 3
): Promise<T> {
  for (let i = 0; i < retries; i++) {
    try {
      return await fn();
    } catch (error: any) {
      if (i === retries - 1) throw error;
      
      // Wait exponentially before retry
      await new Promise(resolve => 
        setTimeout(resolve, Math.pow(2, i) * 1000)
      );
    }
  }
  throw new Error('Max retries exceeded');
}

// Usage
const content = await withRetry(() => 
  generateContent(config)
);
```

## Common Patterns

### Batch Content Generation

```typescript
async function generateBatchContent(keywords: string[]) {
  const results = await Promise.allSettled(
    keywords.map(keyword => 
      runFullPipeline({
        keyword,
        format: 'toplist',
        language: 'en',
        generateVideo: true,
        autoPublish: false,
      })
    )
  );

  const successful = results.filter(r => r.status === 'fulfilled');
  const failed = results.filter(r => r.status === 'rejected');

  return { successful, failed };
}
```

### Scheduled Publishing

```typescript
// lib/scheduler/cron.ts
import cron from 'node-cron';

// Schedule content generation daily at 9 AM
cron.schedule('0 9 * * *', async () => {
  console.log('Running scheduled content generation...');
  await runFullPipeline({
    keyword: 'daily tech trends',
    format: 'toplist',
    language: 'en',
    generateVideo: true,
    autoPublish: true,
  });
});
```

This skill enables comprehensive automation of content marketing workflows from research to publication.
