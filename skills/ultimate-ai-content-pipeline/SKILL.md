---
name: ultimate-ai-content-pipeline
description: Automated content creation pipeline from research to video generation using Claude, OpenAI, and Remotion
triggers:
  - "set up automated content pipeline"
  - "generate content from research to video"
  - "auto-crawl news and create social media posts"
  - "build AI-powered marketing automation"
  - "create video content from blog posts automatically"
  - "configure Claude for content generation"
  - "automate content workflow with AI"
  - "generate multilingual marketing content"
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

An end-to-end automated content creation system that transforms keywords into fully researched articles and videos. Built with TypeScript, Next.js, Claude 3, OpenAI, and Remotion for video rendering.

## What It Does

This system automates the entire content creation workflow:

1. **Auto-Research**: Crawls recent news from TechCrunch, a16z, Twitter/X, LinkedIn (last 24h)
2. **Content Generation**: Creates articles in multiple formats (toplist, POV, case study, how-to) using Claude/OpenAI
3. **Multilingual Output**: Generates both English and Vietnamese versions
4. **Video Rendering**: Automatically creates videos and infographics using Remotion
5. **Multi-Platform Export**: Outputs optimized videos for Reels, TikTok, Shorts

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
# AI Services
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Optional: Database (if using)
DATABASE_URL=your_database_connection_string

# App Configuration
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI service integrations
│   │   ├── crawler/     # News crawling logic
│   │   ├── video/       # Remotion video generation
│   │   └── utils/       # Utility functions
│   └── types/           # TypeScript type definitions
├── public/              # Static assets
└── remotion/            # Remotion video templates
```

## Core API Usage

### 1. Research & Crawling

```typescript
import { crawlNews } from '@/lib/crawler/news-crawler';

// Crawl news from multiple sources
async function researchTopic(keyword: string) {
  const sources = ['techcrunch', 'a16z', 'twitter', 'linkedin'];
  
  const newsData = await crawlNews({
    keyword,
    sources,
    timeRange: '24h',
    maxResults: 20
  });
  
  return newsData;
}
```

### 2. Content Generation with Claude

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

async function generateContent(researchData: any, format: string) {
  const prompt = `
Based on the following research data, create a ${format} article:

Research: ${JSON.stringify(researchData)}

Requirements:
- Engaging and data-backed
- Include recent statistics
- Professional tone
- 1000-1500 words
`;

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [
      { role: 'user', content: prompt }
    ],
  });

  return message.content[0].text;
}
```

### 3. OpenAI Alternative

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateWithGPT(topic: string, language: 'en' | 'vi') {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are a professional content writer. Generate content in ${language === 'vi' ? 'Vietnamese' : 'English'}.`
      },
      {
        role: 'user',
        content: `Write a comprehensive article about: ${topic}`
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

async function generateVideo(content: string, outputPath: string) {
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
      description: content.description,
      keyPoints: content.keyPoints,
    },
  });

  // Render video
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: {
      title: content.title,
      description: content.description,
      keyPoints: content.keyPoints,
    },
  });

  return outputPath;
}
```

## Complete Workflow Example

```typescript
import { crawlNews } from '@/lib/crawler/news-crawler';
import { generateContent } from '@/lib/ai/content-generator';
import { translateContent } from '@/lib/ai/translator';
import { generateVideo } from '@/lib/video/video-renderer';

async function fullContentPipeline(keyword: string) {
  try {
    // Step 1: Research
    console.log('🔍 Researching topic...');
    const researchData = await crawlNews({
      keyword,
      sources: ['techcrunch', 'a16z', 'twitter'],
      timeRange: '24h',
    });

    // Step 2: Generate English content
    console.log('✍️ Generating English content...');
    const englishContent = await generateContent(researchData, {
      format: 'toplist',
      language: 'en',
      tone: 'professional',
    });

    // Step 3: Translate to Vietnamese
    console.log('🌐 Translating to Vietnamese...');
    const vietnameseContent = await translateContent(englishContent, {
      targetLang: 'vi',
      tone: 'friendly',
    });

    // Step 4: Generate video
    console.log('🎬 Rendering video...');
    const videoPath = await generateVideo({
      content: englishContent,
      format: 'vertical', // for TikTok/Reels
      duration: 60,
      outputPath: `./output/${keyword}-video.mp4`,
    });

    return {
      englishContent,
      vietnameseContent,
      videoPath,
      researchSources: researchData.sources,
    };
  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}

// Usage
fullContentPipeline('AI in Marketing 2024')
  .then((result) => {
    console.log('✅ Content pipeline completed!');
    console.log('Video saved at:', result.videoPath);
  });
```

## Content Format Types

```typescript
type ContentFormat = 
  | 'toplist'      // "Top 10 Ways to..."
  | 'pov'          // Point of view / opinion piece
  | 'case-study'   // In-depth analysis
  | 'how-to'       // Step-by-step guide
  | 'news'         // News summary
  | 'comparison';  // Product/service comparison

type ToneStyle = 
  | 'professional'
  | 'friendly'
  | 'humorous'
  | 'authoritative'
  | 'casual';

interface ContentConfig {
  format: ContentFormat;
  language: 'en' | 'vi';
  tone: ToneStyle;
  wordCount?: number;
  includeStats?: boolean;
  includeCTA?: boolean;
}
```

## Running the Development Server

```bash
# Start Next.js development server
npm run dev

# Access the web interface at:
# http://localhost:3000
```

## CLI Usage (if implemented)

```bash
# Generate content from command line
npm run generate -- --keyword "AI Marketing" --format toplist --lang en

# Crawl news only
npm run crawl -- --keyword "Web3" --sources techcrunch,a16z

# Render video from existing content
npm run render-video -- --input content.json --output video.mp4
```

## Common Patterns

### Pattern 1: Batch Content Generation

```typescript
async function batchGenerate(keywords: string[]) {
  const results = await Promise.allSettled(
    keywords.map(keyword => fullContentPipeline(keyword))
  );

  const successful = results.filter(r => r.status === 'fulfilled');
  const failed = results.filter(r => r.status === 'rejected');

  return {
    successful: successful.length,
    failed: failed.length,
    results,
  };
}
```

### Pattern 2: Schedule Automated Posts

```typescript
import schedule from 'node-schedule';

function scheduleContentGeneration(config: {
  keywords: string[];
  cronPattern: string;
}) {
  schedule.scheduleJob(config.cronPattern, async () => {
    const keyword = config.keywords[Math.floor(Math.random() * config.keywords.length)];
    
    console.log(`⏰ Scheduled job started for: ${keyword}`);
    await fullContentPipeline(keyword);
  });
}

// Run daily at 9 AM
scheduleContentGeneration({
  keywords: ['AI trends', 'Marketing automation', 'Social media'],
  cronPattern: '0 9 * * *',
});
```

### Pattern 3: Custom Video Templates

```typescript
// remotion/compositions/CustomTemplate.tsx
import { AbsoluteFill, Sequence, useCurrentFrame } from 'remotion';

export const CustomTemplate: React.FC<{
  title: string;
  points: string[];
}> = ({ title, points }) => {
  const frame = useCurrentFrame();

  return (
    <AbsoluteFill style={{ backgroundColor: '#000' }}>
      <Sequence from={0} durationInFrames={60}>
        <h1 style={{ color: '#fff', fontSize: 48 }}>{title}</h1>
      </Sequence>
      
      {points.map((point, i) => (
        <Sequence key={i} from={60 * (i + 1)} durationInFrames={90}>
          <div style={{ color: '#fff', fontSize: 32 }}>{point}</div>
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement exponential backoff
async function withRetry<T>(
  fn: () => Promise<T>,
  maxRetries = 3
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error: any) {
      if (error.status === 429 && i < maxRetries - 1) {
        const delay = Math.pow(2, i) * 1000;
        console.log(`Rate limited. Retrying in ${delay}ms...`);
        await new Promise(resolve => setTimeout(resolve, delay));
      } else {
        throw error;
      }
    }
  }
  throw new Error('Max retries exceeded');
}
```

### Memory Issues with Large Videos

```typescript
// Stream video rendering to avoid memory issues
import { renderFrames } from '@remotion/renderer';

async function renderLargeVideo(config: any) {
  // Use frame-by-frame rendering
  const frames = await renderFrames({
    ...config,
    frameRange: [0, 300], // Render in chunks
    concurrency: 1, // Reduce parallelism
  });
  
  return frames;
}
```

### Claude API Timeout

```typescript
// Set proper timeout for long content generation
const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
  timeout: 120000, // 2 minutes
});
```

### Missing Dependencies

```bash
# Install Remotion dependencies
npm install @remotion/bundler @remotion/renderer @remotion/cli

# Install AI SDKs
npm install @anthropic-ai/sdk openai

# Install crawling utilities
npm install axios cheerio
```

## Performance Optimization

```typescript
// Cache research data to avoid duplicate crawls
import NodeCache from 'node-cache';

const cache = new NodeCache({ stdTTL: 3600 }); // 1 hour cache

async function cachedCrawl(keyword: string) {
  const cached = cache.get(keyword);
  if (cached) return cached;

  const data = await crawlNews({ keyword });
  cache.set(keyword, data);
  return data;
}
```

## Integration with Social Platforms

```typescript
// Auto-post to social media (example)
async function publishContent(content: any) {
  // Facebook Page
  await postToFacebook({
    message: content.summary,
    link: content.url,
  });

  // LinkedIn
  await postToLinkedIn({
    text: content.linkedInVersion,
    media: content.videoPath,
  });

  // Twitter/X
  await postToTwitter({
    text: content.tweetVersion,
    media: [content.thumbnail],
  });
}
```

This skill enables AI agents to help developers build, configure, and use the Ultimate AI Content Pipeline for automated marketing content creation from research through video generation.
