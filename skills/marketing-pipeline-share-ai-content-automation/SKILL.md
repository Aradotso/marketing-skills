---
name: marketing-pipeline-share-ai-content-automation
description: AI-powered content pipeline that auto-researches, generates scripts, and creates videos using Claude/OpenAI and Remotion
triggers:
  - automate content creation with AI research and video generation
  - generate marketing content from keyword to video automatically
  - create multi-format content with AI research pipeline
  - build automated content workflow with Claude and OpenAI
  - set up AI content generation with auto research crawling
  - generate social media videos from AI-written scripts
  - automate marketing content pipeline end to end
  - create AI-driven content with news research automation
---

# Marketing Pipeline Share - AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

An end-to-end AI content automation system that handles research (crawling TechCrunch, Twitter, LinkedIn), script generation (Claude/OpenAI), multi-format content creation, and video rendering (Remotion). Built with TypeScript/Next.js.

## What It Does

This project automates the entire content creation pipeline:

1. **Auto-Research**: Crawls news sources and social media for fresh data (last 24h)
2. **AI Content Generation**: Creates multiple formats (Top List, POV, Case Study, How-to) in English and Vietnamese
3. **Video Rendering**: Automatically generates infographics and short videos using Remotion
4. **Multi-Platform Export**: Outputs optimized content for Reels, TikTok, Shorts

## Installation

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
yarn install
```

## Environment Configuration

Create a `.env.local` file in the root directory:

```bash
# AI Providers
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_anthropic_key

# Research/Crawling APIs
RAPIDAPI_KEY=your_rapidapi_key

# Optional: Database
DATABASE_URL=your_database_url

# Optional: Social Media APIs
TWITTER_BEARER_TOKEN=your_twitter_token
LINKEDIN_ACCESS_TOKEN=your_linkedin_token
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app router pages
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI generation logic (Claude/OpenAI)
│   │   ├── research/    # Auto-crawling and data gathering
│   │   ├── video/       # Remotion video rendering
│   │   └── utils/       # Helper functions
│   └── types/           # TypeScript type definitions
├── remotion/            # Video templates and compositions
└── public/              # Static assets
```

## Core Usage Patterns

### 1. Research & Data Crawling

```typescript
import { researchTopic } from '@/lib/research/crawler';

interface ResearchResult {
  sources: Array<{
    url: string;
    title: string;
    content: string;
    publishedAt: Date;
    platform: 'techcrunch' | 'twitter' | 'linkedin' | 'a16z';
  }>;
  insights: string[];
  trends: string[];
}

async function gatherResearch(keyword: string): Promise<ResearchResult> {
  const research = await researchTopic({
    keyword,
    timeframe: '24h', // Last 24 hours
    sources: ['techcrunch', 'twitter', 'linkedin'],
    maxResults: 20
  });
  
  return research;
}
```

### 2. AI Content Generation with Claude

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

interface ContentRequest {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  tone: 'professional' | 'friendly' | 'humorous';
  language: 'en' | 'vi';
  research: ResearchResult;
}

async function generateContent(req: ContentRequest) {
  const prompt = `
Based on this research data: ${JSON.stringify(req.research)}

Create a ${req.format} article about "${req.keyword}" in ${req.language}.
Tone: ${req.tone}
Include data-backed insights and recent trends from the research.
Format with proper headings, bullet points, and call-to-action.
`;

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [{
      role: 'user',
      content: prompt
    }]
  });

  return message.content[0].text;
}
```

### 3. AI Content Generation with OpenAI

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateWithGPT(req: ContentRequest) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are an expert content creator specializing in ${req.format} articles.`
      },
      {
        role: 'user',
        content: `Create content about "${req.keyword}" using this research: ${JSON.stringify(req.research)}`
      }
    ],
    temperature: 0.7,
    max_tokens: 3000
  });

  return completion.choices[0].message.content;
}
```

### 4. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  title: string;
  keyPoints: string[];
  backgroundColor: string;
  platform: 'reels' | 'tiktok' | 'shorts';
}

async function generateVideo(content: string, config: VideoConfig) {
  // Platform-specific dimensions
  const dimensions = {
    reels: { width: 1080, height: 1920 },
    tiktok: { width: 1080, height: 1920 },
    shorts: { width: 1080, height: 1920 }
  };

  const bundled = await bundle({
    entryPoint: path.join(process.cwd(), 'remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  const composition = await selectComposition({
    serveUrl: bundled,
    id: 'ContentVideo',
    inputProps: {
      title: config.title,
      keyPoints: config.keyPoints,
      backgroundColor: config.backgroundColor
    }
  });

  await renderMedia({
    composition,
    serveUrl: bundled,
    codec: 'h264',
    outputLocation: `out/${config.platform}-video.mp4`,
    ...dimensions[config.platform]
  });
}
```

### 5. Complete Pipeline Integration

```typescript
import { researchTopic } from '@/lib/research/crawler';
import { generateContent } from '@/lib/ai/claude';
import { generateVideo } from '@/lib/video/remotion';
import { extractKeyPoints } from '@/lib/utils/parser';

interface PipelineConfig {
  keyword: string;
  contentFormat: 'toplist' | 'pov' | 'case-study' | 'how-to';
  languages: Array<'en' | 'vi'>;
  generateVideo: boolean;
  platform?: 'reels' | 'tiktok' | 'shorts';
}

async function runContentPipeline(config: PipelineConfig) {
  // Step 1: Research
  console.log('🔍 Researching topic...');
  const research = await researchTopic({
    keyword: config.keyword,
    timeframe: '24h',
    sources: ['techcrunch', 'twitter', 'linkedin']
  });

  // Step 2: Generate content in multiple languages
  const contents = await Promise.all(
    config.languages.map(async (lang) => {
      console.log(`✍️ Generating ${lang} content...`);
      return {
        language: lang,
        text: await generateContent({
          keyword: config.keyword,
          format: config.contentFormat,
          tone: 'professional',
          language: lang,
          research
        })
      };
    })
  );

  // Step 3: Generate video if requested
  let videoPath: string | null = null;
  if (config.generateVideo && config.platform) {
    console.log('🎬 Rendering video...');
    const keyPoints = extractKeyPoints(contents[0].text);
    videoPath = await generateVideo(contents[0].text, {
      title: config.keyword,
      keyPoints,
      backgroundColor: '#1a1a1a',
      platform: config.platform
    });
  }

  return {
    research,
    contents,
    videoPath
  };
}

// Example usage
const result = await runContentPipeline({
  keyword: 'AI Marketing Trends 2024',
  contentFormat: 'toplist',
  languages: ['en', 'vi'],
  generateVideo: true,
  platform: 'reels'
});
```

## API Routes (Next.js)

### Research Endpoint

```typescript
// app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { researchTopic } from '@/lib/research/crawler';

export async function POST(req: NextRequest) {
  try {
    const { keyword, timeframe, sources } = await req.json();
    
    const research = await researchTopic({
      keyword,
      timeframe: timeframe || '24h',
      sources: sources || ['techcrunch', 'twitter']
    });
    
    return NextResponse.json({ success: true, data: research });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

### Content Generation Endpoint

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { generateContent } from '@/lib/ai/claude';

export async function POST(req: NextRequest) {
  try {
    const body = await req.json();
    
    const content = await generateContent({
      keyword: body.keyword,
      format: body.format,
      tone: body.tone || 'professional',
      language: body.language || 'en',
      research: body.research
    });
    
    return NextResponse.json({ success: true, content });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

### Video Rendering Endpoint

```typescript
// app/api/video/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { generateVideo } from '@/lib/video/remotion';

export async function POST(req: NextRequest) {
  try {
    const { content, config } = await req.json();
    
    const videoPath = await generateVideo(content, config);
    
    return NextResponse.json({ 
      success: true, 
      videoUrl: `/videos/${path.basename(videoPath)}` 
    });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

## Development Commands

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Start production server
npm run start

# Render Remotion video locally
npm run remotion:render

# Preview Remotion compositions
npm run remotion:preview
```

## Common Patterns

### Multi-Language Content Generation

```typescript
async function generateMultiLanguageContent(keyword: string, research: ResearchResult) {
  const languages: Array<'en' | 'vi'> = ['en', 'vi'];
  
  const contentPromises = languages.map(async (lang) => ({
    language: lang,
    content: await generateContent({
      keyword,
      format: 'toplist',
      tone: 'professional',
      language: lang,
      research
    })
  }));

  return Promise.all(contentPromises);
}
```

### Batch Processing Multiple Keywords

```typescript
async function processBatch(keywords: string[]) {
  const results = [];
  
  for (const keyword of keywords) {
    const research = await researchTopic({ keyword, timeframe: '24h' });
    const content = await generateContent({
      keyword,
      format: 'how-to',
      tone: 'friendly',
      language: 'en',
      research
    });
    
    results.push({ keyword, research, content });
    
    // Rate limiting
    await new Promise(resolve => setTimeout(resolve, 2000));
  }
  
  return results;
}
```

### Error Handling & Retries

```typescript
async function generateWithRetry(req: ContentRequest, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await generateContent(req);
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      console.log(`Retry ${i + 1}/${maxRetries}...`);
      await new Promise(resolve => setTimeout(resolve, 1000 * (i + 1)));
    }
  }
}
```

## Troubleshooting

### API Rate Limits

**Issue**: Getting 429 errors from OpenAI/Anthropic
**Solution**: Implement rate limiting and exponential backoff

```typescript
import pLimit from 'p-limit';

const limit = pLimit(3); // Max 3 concurrent requests

async function generateMultiple(requests: ContentRequest[]) {
  return Promise.all(
    requests.map(req => limit(() => generateContent(req)))
  );
}
```

### Remotion Rendering Fails

**Issue**: Video rendering crashes or times out
**Solution**: Increase timeout and handle memory limits

```typescript
await renderMedia({
  composition,
  serveUrl: bundled,
  codec: 'h264',
  timeoutInMilliseconds: 120000, // 2 minutes
  chromiumOptions: {
    args: ['--no-sandbox', '--disable-setuid-sandbox']
  }
});
```

### Research Crawler Blocked

**Issue**: News sites blocking crawlers
**Solution**: Use proper headers and respect robots.txt

```typescript
const headers = {
  'User-Agent': 'Mozilla/5.0 (compatible; ContentBot/1.0)',
  'Accept': 'text/html,application/json',
  'Accept-Language': 'en-US,en;q=0.9'
};
```

### Large Content Exceeds Token Limits

**Issue**: Generated content too long for AI context
**Solution**: Chunk content and summarize

```typescript
function chunkResearch(research: ResearchResult, maxChunkSize = 2000) {
  const chunks = [];
  let currentChunk = '';
  
  for (const source of research.sources) {
    if ((currentChunk + source.content).length > maxChunkSize) {
      chunks.push(currentChunk);
      currentChunk = source.content;
    } else {
      currentChunk += '\n' + source.content;
    }
  }
  
  if (currentChunk) chunks.push(currentChunk);
  return chunks;
}
```

## Best Practices

1. **Always validate research data** before feeding to AI to avoid hallucinations
2. **Cache research results** to avoid redundant API calls
3. **Use streaming responses** for real-time content generation feedback
4. **Implement job queues** for video rendering to avoid blocking requests
5. **Store generated content** in database with metadata for analytics
6. **Use TypeScript strictly** to catch errors early in the pipeline
