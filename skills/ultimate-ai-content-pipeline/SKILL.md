---
name: ultimate-ai-content-pipeline
description: Automated content creation system with AI research, multi-format generation, and video rendering capabilities using Claude, OpenAI, and Remotion
triggers:
  - how do I set up the AI content pipeline
  - generate content with automated research
  - create videos from text content automatically
  - use Claude and OpenAI for content generation
  - configure the content pipeline system
  - render videos with Remotion from articles
  - automate content creation with AI research
  - build multi-format content with this pipeline
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

Ultimate AI Content Pipeline is a comprehensive content automation system that handles the entire workflow from research to video generation. It automatically crawls news sources, generates multi-format content using Claude/OpenAI, and renders videos using Remotion—all from a single keyword input.

## What This Project Does

This TypeScript-based pipeline automates:
- **Auto-Research**: Crawls recent content from TechCrunch, a16z, X (Twitter), LinkedIn
- **Multi-Format Generation**: Creates Toplists, POV articles, Case Studies, How-tos in multiple languages
- **Video Rendering**: Converts written content into infographics and short videos optimized for social platforms
- **Workflow Automation**: End-to-end content production with minimal manual intervention

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
# AI Provider Keys
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Optional: Social Media APIs
TWITTER_API_KEY=your_twitter_key
LINKEDIN_API_KEY=your_linkedin_key

# Database (if using)
DATABASE_URL=your_database_connection_string

# Remotion License (for video rendering)
REMOTION_LICENSE_KEY=your_remotion_license
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI provider integrations
│   │   ├── research/    # Content crawling modules
│   │   └── video/       # Remotion video generation
│   ├── types/           # TypeScript type definitions
│   └── utils/           # Helper functions
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Core Usage Patterns

### 1. Automated Content Research

```typescript
import { researchTopic } from '@/lib/research/crawler';

// Crawl and analyze recent content
async function gatherResearch(keyword: string) {
  const research = await researchTopic({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeframe: '24h',
    maxResults: 50
  });

  return {
    insights: research.insights,
    trending: research.trendingTopics,
    dataPoints: research.statistics
  };
}

// Example usage
const data = await gatherResearch('AI automation');
console.log(data.insights);
```

### 2. Multi-Format Content Generation

```typescript
import { generateContent } from '@/lib/ai/content-generator';
import { ContentFormat, Language, Tone } from '@/types/content';

interface ContentRequest {
  keyword: string;
  format: ContentFormat;
  language: Language;
  tone: Tone;
  researchData?: any;
}

async function createContent(request: ContentRequest) {
  const content = await generateContent({
    keyword: request.keyword,
    format: request.format, // 'toplist' | 'pov' | 'case-study' | 'how-to'
    language: request.language, // 'en' | 'vi'
    tone: request.tone, // 'expert' | 'friendly' | 'humorous'
    provider: 'claude', // or 'openai'
    researchData: request.researchData
  });

  return content;
}

// Example: Generate a toplist in Vietnamese
const article = await createContent({
  keyword: 'marketing automation tools',
  format: 'toplist',
  language: 'vi',
  tone: 'expert'
});
```

### 3. Using Claude API

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

async function generateWithClaude(prompt: string, systemPrompt: string) {
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    system: systemPrompt,
    messages: [
      {
        role: 'user',
        content: prompt
      }
    ]
  });

  return message.content[0].type === 'text' 
    ? message.content[0].text 
    : '';
}

// Example system prompt for content generation
const systemPrompt = `You are an expert content writer specializing in marketing automation.
Create engaging, data-backed content that:
- Uses insights from recent research
- Maintains the specified tone and format
- Includes actionable takeaways
- Optimizes for readability`;

const content = await generateWithClaude(
  'Write a toplist of 5 marketing automation tools based on this research: ...',
  systemPrompt
);
```

### 4. Using OpenAI API

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateWithOpenAI(prompt: string, systemPrompt: string) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: systemPrompt
      },
      {
        role: 'user',
        content: prompt
      }
    ],
    temperature: 0.7,
    max_tokens: 3000
  });

  return completion.choices[0].message.content;
}
```

### 5. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  title: string;
  keyPoints: string[];
  duration: number;
  format: 'reels' | 'tiktok' | 'shorts';
}

async function generateVideo(content: VideoConfig) {
  // Bundle the Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  // Select composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title: content.title,
      keyPoints: content.keyPoints,
      duration: content.duration
    },
  });

  // Aspect ratios for different platforms
  const aspectRatios = {
    reels: { width: 1080, height: 1920 },
    tiktok: { width: 1080, height: 1920 },
    shorts: { width: 1080, height: 1920 }
  };

  const { width, height } = aspectRatios[content.format];

  // Render video
  const outputPath = path.resolve(`./output/video-${Date.now()}.mp4`);
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    overwrite: true,
    imageFormat: 'jpeg',
    width,
    height,
  });

  return outputPath;
}

// Example usage
const videoPath = await generateVideo({
  title: 'Top 5 Marketing Automation Tools',
  keyPoints: [
    'HubSpot - All-in-one platform',
    'Mailchimp - Email marketing',
    'ActiveCampaign - Customer experience',
    'Marketo - Enterprise solution',
    'Drip - E-commerce focused'
  ],
  duration: 30,
  format: 'reels'
});
```

### 6. Complete Pipeline Workflow

```typescript
import { researchTopic } from '@/lib/research/crawler';
import { generateContent } from '@/lib/ai/content-generator';
import { generateVideo } from '@/lib/video/renderer';
import { publishContent } from '@/lib/publish/distributor';

async function runContentPipeline(keyword: string) {
  try {
    // Step 1: Research
    console.log('🔍 Starting research...');
    const research = await researchTopic({
      keyword,
      sources: ['techcrunch', 'a16z', 'twitter'],
      timeframe: '24h'
    });

    // Step 2: Generate content in multiple formats
    console.log('✍️ Generating content...');
    const contentFormats = ['toplist', 'pov'] as const;
    const contents = await Promise.all(
      contentFormats.map(format =>
        generateContent({
          keyword,
          format,
          language: 'vi',
          tone: 'expert',
          researchData: research,
          provider: 'claude'
        })
      )
    );

    // Step 3: Generate video from toplist content
    console.log('🎬 Rendering video...');
    const videoPath = await generateVideo({
      title: contents[0].title,
      keyPoints: contents[0].keyPoints,
      duration: 30,
      format: 'reels'
    });

    // Step 4: Publish (optional)
    console.log('📤 Publishing content...');
    await publishContent({
      articles: contents,
      video: videoPath,
      platforms: ['facebook', 'linkedin']
    });

    console.log('✅ Pipeline completed successfully!');
    
    return {
      research,
      contents,
      videoPath
    };
  } catch (error) {
    console.error('❌ Pipeline error:', error);
    throw error;
  }
}

// Run the pipeline
const result = await runContentPipeline('AI content automation');
```

## API Routes (Next.js)

### Generate Content Endpoint

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { generateContent } from '@/lib/ai/content-generator';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const { keyword, format, language, tone } = body;

    if (!keyword) {
      return NextResponse.json(
        { error: 'Keyword is required' },
        { status: 400 }
      );
    }

    const content = await generateContent({
      keyword,
      format: format || 'toplist',
      language: language || 'vi',
      tone: tone || 'expert',
      provider: 'claude'
    });

    return NextResponse.json({ content });
  } catch (error) {
    return NextResponse.json(
      { error: 'Content generation failed' },
      { status: 500 }
    );
  }
}
```

### Research Endpoint

```typescript
// app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { researchTopic } from '@/lib/research/crawler';

export async function POST(request: NextRequest) {
  try {
    const { keyword, sources, timeframe } = await request.json();

    const research = await researchTopic({
      keyword,
      sources: sources || ['techcrunch', 'a16z'],
      timeframe: timeframe || '24h'
    });

    return NextResponse.json({ research });
  } catch (error) {
    return NextResponse.json(
      { error: 'Research failed' },
      { status: 500 }
    );
  }
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

# Render Remotion video locally
npm run remotion:preview

# Export Remotion video
npm run remotion:render
```

## Common Patterns

### Custom AI Prompts

```typescript
// lib/ai/prompts.ts
export const contentPrompts = {
  toplist: (keyword: string, research: any) => `
    Create a comprehensive toplist article about "${keyword}".
    
    Research insights:
    ${JSON.stringify(research.insights, null, 2)}
    
    Requirements:
    - 5-10 items ranked by relevance
    - Include data points and statistics
    - Add actionable takeaways
    - Vietnamese language, expert tone
  `,
  
  pov: (keyword: string, research: any) => `
    Write a thought-provoking POV article about "${keyword}".
    
    Base your perspective on:
    ${JSON.stringify(research.insights, null, 2)}
    
    Requirements:
    - Strong opening hook
    - Personal insights and opinions
    - Backed by data
    - Conversational yet authoritative
  `
};
```

### Error Handling and Retries

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
const content = await withRetry(() => 
  generateContent({ keyword: 'AI tools', format: 'toplist' })
);
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
  
  // Check cache
  const cached = await redis.get(cacheKey);
  if (cached) return cached;
  
  // Fetch new research
  const research = await researchTopic({ keyword });
  
  // Cache for 24 hours
  await redis.setex(cacheKey, 86400, JSON.stringify(research));
  
  return research;
}
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement rate limiting
import { Ratelimit } from '@upstash/ratelimit';

const ratelimit = new Ratelimit({
  redis,
  limiter: Ratelimit.slidingWindow(10, '1 m'),
});

async function rateLimitedGenerate(keyword: string) {
  const identifier = `api:${keyword}`;
  const { success } = await ratelimit.limit(identifier);
  
  if (!success) {
    throw new Error('Rate limit exceeded. Please try again later.');
  }
  
  return generateContent({ keyword });
}
```

### Remotion Rendering Issues

```bash
# Ensure required codecs are installed
npm install @remotion/lambda

# Increase memory for large videos
NODE_OPTIONS="--max-old-space-size=4096" npm run remotion:render
```

### Claude API Timeout

```typescript
// Increase timeout for long-form content
const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
  timeout: 120000, // 2 minutes
});
```

### Missing Environment Variables

```typescript
// Validate environment at startup
const requiredEnvVars = [
  'ANTHROPIC_API_KEY',
  'OPENAI_API_KEY',
  'RAPIDAPI_KEY'
];

for (const envVar of requiredEnvVars) {
  if (!process.env[envVar]) {
    throw new Error(`Missing required environment variable: ${envVar}`);
  }
}
```

## Performance Optimization

```typescript
// Parallel processing for multiple content formats
async function generateMultipleFormats(keyword: string) {
  const formats = ['toplist', 'pov', 'case-study'] as const;
  
  const contents = await Promise.allSettled(
    formats.map(format =>
      generateContent({ keyword, format, language: 'vi' })
    )
  );
  
  return contents
    .filter(result => result.status === 'fulfilled')
    .map(result => (result as PromiseFulfilledResult<any>).value);
}
```

This skill covers the essential patterns for using the Ultimate AI Content Pipeline system effectively. The pipeline integrates research automation, AI-powered content generation, and video rendering into a seamless workflow for marketing content creation.
