---
name: ultimate-ai-content-pipeline
description: Automated content pipeline from research to script generation to video creation using Claude/OpenAI and Remotion
triggers:
  - how do I generate automated content with AI
  - set up content automation pipeline
  - create videos from articles automatically
  - research and generate content with Claude
  - automate content creation workflow
  - build AI-powered marketing content
  - generate multilingual content with AI
  - create social media videos from text
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This is a comprehensive TypeScript-based content automation system that handles the entire content creation workflow: from researching trending topics, generating scripts in multiple formats and languages, to automatically rendering videos. Built with Next.js, Claude/OpenAI, and Remotion.

## What It Does

The Ultimate AI Content Pipeline is an all-in-one content factory that:

- **Auto-scans & researches** trending topics from TechCrunch, a16z, Twitter/X, LinkedIn in the last 24 hours
- **Generates content** in multiple formats (toplist, POV, case study, how-to) using Claude 3 or OpenAI
- **Supports multilingual output** (English & Vietnamese) with customizable tone of voice
- **Renders videos & infographics** automatically using Remotion integration
- **Optimizes for platforms** like Reels, TikTok, and Shorts

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

```bash
# AI Provider Keys
ANTHROPIC_API_KEY=your_claude_api_key_here
OPENAI_API_KEY=your_openai_api_key_here

# Research API Keys
RAPIDAPI_KEY=your_rapidapi_key_here

# Remotion Configuration
REMOTION_GITHUB_TOKEN=your_github_token_here

# Application Configuration
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── app/                    # Next.js app directory
├── components/             # React components
├── lib/                    # Core utilities and services
│   ├── ai/                # AI integration (Claude, OpenAI)
│   ├── research/          # Content research modules
│   └── video/             # Remotion video generation
├── public/                # Static assets
└── remotion/              # Remotion video templates
```

## Core API Usage

### 1. Research Module

The research module crawls and analyzes trending content:

```typescript
import { researchTrends } from '@/lib/research/scanner';

async function getTrendingTopics(keyword: string) {
  const trends = await researchTrends({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeframe: '24h',
    language: 'en'
  });
  
  return trends;
}

// Example response structure
interface TrendData {
  title: string;
  source: string;
  url: string;
  summary: string;
  insights: string[];
  publishedAt: Date;
  engagement: {
    likes: number;
    shares: number;
    comments: number;
  };
}
```

### 2. Content Generation with AI

Generate content using Claude or OpenAI:

```typescript
import { generateContent } from '@/lib/ai/generator';

async function createArticle(topic: string) {
  const content = await generateContent({
    provider: 'claude', // or 'openai'
    model: 'claude-3-opus-20240229',
    topic,
    format: 'toplist', // 'pov', 'case-study', 'how-to'
    language: 'vi', // 'en' or 'vi'
    tone: 'expert', // 'friendly', 'humorous'
    researchData: await getTrendingTopics(topic)
  });
  
  return content;
}

// Example output structure
interface GeneratedContent {
  title: string;
  introduction: string;
  body: Array<{
    heading: string;
    content: string;
    data?: any;
  }>;
  conclusion: string;
  metadata: {
    wordCount: number;
    readingTime: number;
    keywords: string[];
  };
}
```

### 3. Video Generation with Remotion

Convert text content to video:

```typescript
import { renderVideo } from '@/lib/video/renderer';

async function createVideoFromContent(content: GeneratedContent) {
  const video = await renderVideo({
    template: 'infographic', // 'shorts', 'reels', 'tiktok'
    content: {
      title: content.title,
      points: content.body.map(b => b.heading),
      duration: 60 // seconds
    },
    format: {
      width: 1080,
      height: 1920, // 9:16 for vertical video
      fps: 30
    },
    style: {
      theme: 'modern',
      colors: {
        primary: '#3B82F6',
        secondary: '#8B5CF6',
        background: '#1F2937'
      },
      font: 'Inter'
    }
  });
  
  return video.url;
}
```

## Common Patterns

### Full Content Pipeline

End-to-end workflow from research to video:

```typescript
import { researchTrends } from '@/lib/research/scanner';
import { generateContent } from '@/lib/ai/generator';
import { renderVideo } from '@/lib/video/renderer';

async function runFullPipeline(keyword: string) {
  try {
    // Step 1: Research
    console.log('🔍 Researching trending topics...');
    const trends = await researchTrends({
      keyword,
      sources: ['techcrunch', 'twitter'],
      timeframe: '24h'
    });
    
    // Step 2: Generate Content
    console.log('✍️ Generating content...');
    const content = await generateContent({
      provider: 'claude',
      topic: keyword,
      format: 'toplist',
      language: 'vi',
      tone: 'expert',
      researchData: trends
    });
    
    // Step 3: Create Video
    console.log('🎬 Rendering video...');
    const videoUrl = await renderVideo({
      template: 'reels',
      content: {
        title: content.title,
        points: content.body.slice(0, 5).map(b => b.heading)
      },
      format: { width: 1080, height: 1920, fps: 30 }
    });
    
    return {
      article: content,
      video: videoUrl,
      metadata: {
        keyword,
        generatedAt: new Date(),
        sources: trends.length
      }
    };
  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}
```

### Batch Content Generation

Generate multiple pieces of content:

```typescript
async function generateBatchContent(keywords: string[]) {
  const results = await Promise.allSettled(
    keywords.map(async (keyword) => {
      const content = await generateContent({
        provider: 'openai',
        model: 'gpt-4-turbo-preview',
        topic: keyword,
        format: 'how-to',
        language: 'en'
      });
      
      return { keyword, content };
    })
  );
  
  const successful = results
    .filter(r => r.status === 'fulfilled')
    .map(r => (r as PromiseFulfilledResult<any>).value);
    
  const failed = results
    .filter(r => r.status === 'rejected')
    .map(r => (r as PromiseRejectedResult).reason);
  
  return { successful, failed };
}
```

### Custom Video Template

Create custom Remotion video compositions:

```typescript
import { Composition } from 'remotion';
import { z } from 'zod';

export const ContentVideoSchema = z.object({
  title: z.string(),
  points: z.array(z.string()),
  brandColor: z.string()
});

export const ContentVideo: React.FC<z.infer<typeof ContentVideoSchema>> = ({
  title,
  points,
  brandColor
}) => {
  return (
    <div style={{ 
      flex: 1, 
      backgroundColor: 'white',
      padding: 60 
    }}>
      <h1 style={{ 
        fontSize: 72, 
        color: brandColor,
        marginBottom: 40 
      }}>
        {title}
      </h1>
      <ul style={{ fontSize: 48, lineHeight: 1.6 }}>
        {points.map((point, i) => (
          <li key={i}>{point}</li>
        ))}
      </ul>
    </div>
  );
};

// Register composition
export const RemotionRoot = () => {
  return (
    <Composition
      id="ContentVideo"
      component={ContentVideo}
      durationInFrames={1800}
      fps={30}
      width={1080}
      height={1920}
      schema={ContentVideoSchema}
    />
  );
};
```

## API Routes

### Next.js API Endpoints

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { generateContent } from '@/lib/ai/generator';

export async function POST(request: NextRequest) {
  const body = await request.json();
  
  const content = await generateContent({
    provider: body.provider || 'claude',
    topic: body.topic,
    format: body.format || 'toplist',
    language: body.language || 'vi'
  });
  
  return NextResponse.json({ content });
}

// app/api/research/route.ts
export async function GET(request: NextRequest) {
  const { searchParams } = new URL(request.url);
  const keyword = searchParams.get('keyword');
  
  const trends = await researchTrends({
    keyword: keyword || '',
    sources: ['techcrunch', 'twitter']
  });
  
  return NextResponse.json({ trends });
}
```

## CLI Commands (if applicable)

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Run production server
npm start

# Render Remotion video
npx remotion render ContentVideo output.mp4

# Preview Remotion composition
npx remotion preview
```

## Configuration

### AI Model Selection

```typescript
// lib/config/ai.ts
export const AI_PROVIDERS = {
  claude: {
    models: ['claude-3-opus-20240229', 'claude-3-sonnet-20240229'],
    defaultModel: 'claude-3-opus-20240229',
    maxTokens: 4096
  },
  openai: {
    models: ['gpt-4-turbo-preview', 'gpt-4', 'gpt-3.5-turbo'],
    defaultModel: 'gpt-4-turbo-preview',
    maxTokens: 4096
  }
};

export function getAIConfig(provider: 'claude' | 'openai') {
  return AI_PROVIDERS[provider];
}
```

### Content Format Templates

```typescript
// lib/config/templates.ts
export const CONTENT_FORMATS = {
  toplist: {
    structure: ['intro', 'items', 'conclusion'],
    minItems: 5,
    maxItems: 10
  },
  pov: {
    structure: ['hook', 'thesis', 'arguments', 'conclusion'],
    tone: 'opinionated'
  },
  'case-study': {
    structure: ['background', 'challenge', 'solution', 'results'],
    dataRequired: true
  },
  'how-to': {
    structure: ['intro', 'prerequisites', 'steps', 'tips', 'conclusion'],
    stepByStep: true
  }
};
```

## Troubleshooting

### API Key Issues

```typescript
// Verify API keys are loaded
function checkAPIKeys() {
  const required = [
    'ANTHROPIC_API_KEY',
    'OPENAI_API_KEY',
    'RAPIDAPI_KEY'
  ];
  
  const missing = required.filter(key => !process.env[key]);
  
  if (missing.length > 0) {
    throw new Error(`Missing API keys: ${missing.join(', ')}`);
  }
}
```

### Rate Limiting

```typescript
import pLimit from 'p-limit';

// Limit concurrent API calls
const limit = pLimit(3);

async function generateWithRateLimit(topics: string[]) {
  return Promise.all(
    topics.map(topic =>
      limit(() => generateContent({ topic, provider: 'claude' }))
    )
  );
}
```

### Video Rendering Errors

```typescript
// Handle Remotion rendering errors
async function safeRenderVideo(config: any) {
  try {
    return await renderVideo(config);
  } catch (error) {
    if (error.message.includes('timeout')) {
      // Retry with lower quality
      return await renderVideo({
        ...config,
        format: { ...config.format, fps: 24 }
      });
    }
    throw error;
  }
}
```

### Memory Management for Large Batches

```typescript
// Process in chunks to avoid memory issues
async function processInChunks<T>(
  items: T[],
  processor: (item: T) => Promise<any>,
  chunkSize = 5
) {
  const results = [];
  
  for (let i = 0; i < items.length; i += chunkSize) {
    const chunk = items.slice(i, i + chunkSize);
    const chunkResults = await Promise.all(chunk.map(processor));
    results.push(...chunkResults);
    
    // Optional: Add delay between chunks
    await new Promise(resolve => setTimeout(resolve, 1000));
  }
  
  return results;
}
```
