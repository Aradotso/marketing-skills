---
name: marketing-content-pipeline-automation
description: Automated AI content pipeline from research to video generation using Claude/OpenAI and Remotion
triggers:
  - how do I automate content creation with AI
  - set up marketing content pipeline
  - generate videos from text automatically
  - create content with Claude and OpenAI
  - build automated content workflow
  - research and generate marketing content
  - use Remotion for video automation
  - crawl news and create content
---

# Marketing Content Pipeline Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill helps you use the Ultimate AI Content Pipeline - an automated content creation system that handles research, scriptwriting, and video generation using Claude 3, OpenAI, and Remotion. The pipeline crawls fresh data from sources like TechCrunch, a16z, Twitter, and LinkedIn, generates multilingual content in various formats, and automatically renders videos.

## What This Project Does

The marketing-content-pipeline automates the entire content creation workflow:

1. **Auto-Research**: Crawls recent news and data from major tech sources (last 24h)
2. **AI Content Generation**: Creates articles in multiple formats (toplists, POV, case studies, how-to) using Claude/OpenAI
3. **Multilingual Support**: Generates content in English and Vietnamese simultaneously
4. **Video Rendering**: Automatically converts content to videos using Remotion
5. **Multi-Platform Optimization**: Exports videos in formats suitable for Reels, TikTok, and Shorts

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
# AI API Keys
OPENAI_API_KEY=your_openai_key_here
ANTHROPIC_API_KEY=your_claude_key_here

# RapidAPI for data scraping
RAPIDAPI_KEY=your_rapidapi_key_here

# Next.js Configuration
NEXT_PUBLIC_API_URL=http://localhost:3000

# Remotion Configuration (optional)
REMOTION_LICENSE_KEY=your_remotion_license_here
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/                 # Next.js app router
│   ├── components/          # React components
│   ├── lib/
│   │   ├── ai/             # AI integration (Claude, OpenAI)
│   │   ├── scraper/        # Content scraping logic
│   │   └── video/          # Remotion video generation
│   └── types/              # TypeScript types
├── remotion/               # Remotion video templates
└── public/                 # Static assets
```

## Key Commands

```bash
# Development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Render video with Remotion
npm run remotion:render

# Type checking
npm run type-check

# Linting
npm run lint
```

## Core API Usage

### 1. Research & Scraping

```typescript
// src/lib/scraper/news-scraper.ts
import axios from 'axios';

interface NewsSource {
  url: string;
  title: string;
  publishedAt: string;
  content: string;
}

export async function scrapeLatestNews(
  keyword: string,
  sources: string[] = ['techcrunch', 'a16z']
): Promise<NewsSource[]> {
  const rapidApiKey = process.env.RAPIDAPI_KEY;
  
  const options = {
    method: 'GET',
    url: 'https://news-api.rapidapi.com/v1/search',
    params: {
      q: keyword,
      lang: 'en',
      from: new Date(Date.now() - 24 * 60 * 60 * 1000).toISOString(),
      sortBy: 'publishedAt'
    },
    headers: {
      'X-RapidAPI-Key': rapidApiKey,
      'X-RapidAPI-Host': 'news-api.rapidapi.com'
    }
  };

  const response = await axios.request(options);
  return response.data.articles;
}
```

### 2. AI Content Generation with Claude

```typescript
// src/lib/ai/claude-generator.ts
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

export async function generateContent(
  topic: string,
  format: 'toplist' | 'pov' | 'case-study' | 'how-to',
  research: string,
  language: 'en' | 'vi' = 'en'
): Promise<string> {
  const prompts = {
    toplist: `Create a top 10 list article about ${topic}`,
    pov: `Write a point-of-view article about ${topic}`,
    'case-study': `Write a detailed case study about ${topic}`,
    'how-to': `Create a comprehensive how-to guide about ${topic}`
  };

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4000,
    messages: [
      {
        role: 'user',
        content: `${prompts[format]} in ${language === 'vi' ? 'Vietnamese' : 'English'}.
        
Use this research data:
${research}

Requirements:
- Data-backed insights
- Engaging tone
- SEO optimized
- Include statistics and quotes from research`
      }
    ]
  });

  return message.content[0].type === 'text' ? message.content[0].text : '';
}
```

### 3. AI Content Generation with OpenAI

```typescript
// src/lib/ai/openai-generator.ts
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

export async function generateContentOpenAI(
  topic: string,
  research: string,
  tone: 'expert' | 'friendly' | 'humorous' = 'expert'
): Promise<string> {
  const toneInstructions = {
    expert: 'Use professional, authoritative language',
    friendly: 'Use conversational, approachable language',
    humorous: 'Use engaging, witty language with appropriate humor'
  };

  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are a content marketing expert. ${toneInstructions[tone]}.`
      },
      {
        role: 'user',
        content: `Create an article about ${topic} using this research:\n\n${research}`
      }
    ],
    temperature: 0.7,
    max_tokens: 3000
  });

  return completion.choices[0].message.content || '';
}
```

### 4. Video Generation with Remotion

```typescript
// src/lib/video/render-video.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

export async function renderContentVideo(
  content: string,
  title: string,
  format: 'reels' | 'tiktok' | 'shorts' = 'reels'
): Promise<string> {
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
      title,
      content,
      ...dimensions[format]
    },
  });

  const outputPath = path.join(process.cwd(), 'public', 'videos', `${Date.now()}.mp4`);

  await renderMedia({
    composition,
    serveUrl: bundled,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: {
      title,
      content,
    },
  });

  return outputPath;
}
```

### 5. Complete Pipeline Integration

```typescript
// src/lib/pipeline/content-pipeline.ts
import { scrapeLatestNews } from '../scraper/news-scraper';
import { generateContent } from '../ai/claude-generator';
import { renderContentVideo } from '../video/render-video';

export async function runContentPipeline(
  keyword: string,
  format: 'toplist' | 'pov' | 'case-study' | 'how-to',
  language: 'en' | 'vi' = 'en',
  generateVideo: boolean = true
) {
  // Step 1: Research
  console.log('🔍 Scraping latest news...');
  const newsData = await scrapeLatestNews(keyword);
  const researchSummary = newsData
    .map(article => `${article.title}: ${article.content}`)
    .join('\n\n');

  // Step 2: Generate Content
  console.log('✍️ Generating content with AI...');
  const content = await generateContent(keyword, format, researchSummary, language);

  // Step 3: Generate Video (optional)
  let videoPath = null;
  if (generateVideo) {
    console.log('🎬 Rendering video...');
    videoPath = await renderContentVideo(content, keyword, 'reels');
  }

  return {
    content,
    videoPath,
    research: newsData,
    metadata: {
      keyword,
      format,
      language,
      generatedAt: new Date().toISOString()
    }
  };
}
```

## API Route Examples

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline/content-pipeline';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const { keyword, format, language, generateVideo } = body;

    if (!keyword) {
      return NextResponse.json(
        { error: 'Keyword is required' },
        { status: 400 }
      );
    }

    const result = await runContentPipeline(
      keyword,
      format || 'toplist',
      language || 'en',
      generateVideo ?? false
    );

    return NextResponse.json(result);
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { error: 'Failed to generate content' },
      { status: 500 }
    );
  }
}
```

## Common Patterns

### Batch Content Generation

```typescript
// Generate multiple articles at once
async function batchGenerateContent(keywords: string[]) {
  const results = await Promise.all(
    keywords.map(keyword => 
      runContentPipeline(keyword, 'toplist', 'en', false)
    )
  );
  
  return results;
}
```

### Scheduled Content Creation

```typescript
// Use with node-cron or similar
import cron from 'node-cron';

// Run every day at 9 AM
cron.schedule('0 9 * * *', async () => {
  const trendingTopics = ['AI', 'Marketing', 'SaaS'];
  
  for (const topic of trendingTopics) {
    await runContentPipeline(topic, 'toplist', 'vi', true);
  }
});
```

### Custom Video Templates

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, interpolate } from 'remotion';

export const ContentVideo: React.FC<{
  title: string;
  content: string;
}> = ({ title, content }) => {
  const frame = useCurrentFrame();
  const opacity = interpolate(frame, [0, 30], [0, 1]);

  return (
    <AbsoluteFill style={{ backgroundColor: '#000' }}>
      <div style={{ opacity, padding: 60, color: '#fff' }}>
        <h1 style={{ fontSize: 72, fontWeight: 'bold' }}>{title}</h1>
        <p style={{ fontSize: 32, marginTop: 40 }}>{content.slice(0, 200)}...</p>
      </div>
    </AbsoluteFill>
  );
};
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement exponential backoff
async function fetchWithRetry(fn: () => Promise<any>, retries = 3) {
  for (let i = 0; i < retries; i++) {
    try {
      return await fn();
    } catch (error: any) {
      if (error.status === 429 && i < retries - 1) {
        await new Promise(resolve => setTimeout(resolve, Math.pow(2, i) * 1000));
      } else {
        throw error;
      }
    }
  }
}
```

### Video Rendering Memory Issues

```typescript
// Reduce concurrency for video rendering
import pLimit from 'p-limit';

const limit = pLimit(2); // Only 2 concurrent renders

const videoPromises = topics.map(topic =>
  limit(() => renderContentVideo(topic.content, topic.title))
);

await Promise.all(videoPromises);
```

### Missing Environment Variables

```typescript
// Validate environment variables at startup
function validateEnv() {
  const required = ['OPENAI_API_KEY', 'ANTHROPIC_API_KEY', 'RAPIDAPI_KEY'];
  const missing = required.filter(key => !process.env[key]);
  
  if (missing.length > 0) {
    throw new Error(`Missing required environment variables: ${missing.join(', ')}`);
  }
}

validateEnv();
```

### Content Quality Issues

```typescript
// Add content validation
function validateContent(content: string): boolean {
  return (
    content.length > 500 &&
    content.split(' ').length > 100 &&
    !content.includes('[ERROR]') &&
    !content.includes('I cannot')
  );
}

// Use in pipeline
const content = await generateContent(keyword, format, research, language);
if (!validateContent(content)) {
  throw new Error('Generated content did not meet quality standards');
}
```

This skill enables AI agents to help developers build and customize automated content marketing pipelines with AI-powered research, generation, and video creation capabilities.
