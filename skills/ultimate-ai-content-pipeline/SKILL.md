---
name: ultimate-ai-content-pipeline
description: Automated content pipeline from research to script to video generation using Claude, OpenAI, and Remotion
triggers:
  - automate content creation with AI
  - generate videos from blog posts automatically
  - research and create marketing content pipeline
  - use Claude and OpenAI for content automation
  - build automated content workflow
  - create videos with Remotion from content
  - scrape news and generate content scripts
  - set up AI marketing content system
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This project is an end-to-end AI-powered content automation system that handles research, script generation, and video rendering. It automatically crawls news sources (TechCrunch, a16z, Twitter, LinkedIn), generates content in multiple formats and languages using Claude/OpenAI, and creates videos using Remotion.

## What It Does

- **Auto-Research**: Scrapes real-time data from news sources within 24 hours
- **Multi-Format Content**: Generates Toplist, POV, Case Study, How-to formats
- **Bilingual**: Creates English and Vietnamese versions simultaneously
- **Video Generation**: Automatically renders videos and infographics from content
- **Platform Optimization**: Exports video for Reels, TikTok, Shorts

## Installation

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
yarn install

# Set up environment variables
cp .env.example .env
```

## Configuration

Create a `.env` file with the following required variables:

```bash
# AI APIs
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_url

# Optional: Video rendering
REMOTION_LICENSE_KEY=your_remotion_key
```

## Project Structure

```typescript
// Typical TypeScript project structure
src/
├── app/                  // Next.js app directory
├── components/           // React components
├── lib/
│   ├── ai/              // AI integrations (Claude, OpenAI)
│   ├── research/        // Web scraping & data collection
│   ├── content/         // Content generation logic
│   └── video/           // Remotion video rendering
├── types/               // TypeScript type definitions
└── utils/               // Helper functions
```

## Core API Usage

### 1. Research Module - Auto-Crawling

```typescript
import { researchTopic } from '@/lib/research/crawler';

// Crawl news sources for a topic
const researchData = await researchTopic({
  keyword: 'AI automation',
  sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
  timeRange: '24h',
  maxResults: 20
});

console.log(researchData);
// {
//   articles: [...],
//   insights: [...],
//   statistics: {...},
//   trendingTopics: [...]
// }
```

### 2. Content Generation with Claude/OpenAI

```typescript
import { generateContent } from '@/lib/ai/content-generator';

// Generate content in multiple formats
const content = await generateContent({
  topic: 'AI in Marketing',
  format: 'toplist', // 'pov' | 'case-study' | 'how-to'
  language: 'both', // 'en' | 'vi' | 'both'
  tone: 'professional', // 'friendly' | 'humorous' | 'expert'
  researchData: researchData,
  aiProvider: 'claude' // or 'openai'
});

console.log(content);
// {
//   english: { title: '...', body: '...', metadata: {...} },
//   vietnamese: { title: '...', body: '...', metadata: {...} }
// }
```

### 3. AI Provider Integration

```typescript
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

// Claude Integration
const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

async function generateWithClaude(prompt: string) {
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [
      {
        role: 'user',
        content: prompt
      }
    ],
  });
  
  return message.content[0].text;
}

// OpenAI Integration
const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateWithOpenAI(prompt: string) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: 'You are a professional content creator.'
      },
      {
        role: 'user',
        content: prompt
      }
    ],
  });
  
  return completion.choices[0].message.content;
}
```

### 4. Video Rendering with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

async function renderContentVideo(content: any) {
  const compositionId = 'ContentVideo';
  const bundleLocation = await bundle(
    path.join(process.cwd(), 'src/remotion/index.ts')
  );

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
  });

  const outputLocation = path.join(
    process.cwd(),
    'public/videos',
    `${content.id}.mp4`
  );

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: {
      title: content.title,
      body: content.body,
      format: 'vertical', // for Reels/TikTok/Shorts
    },
  });

  return outputLocation;
}
```

### 5. Complete Pipeline Example

```typescript
import { researchTopic } from '@/lib/research/crawler';
import { generateContent } from '@/lib/ai/content-generator';
import { renderContentVideo } from '@/lib/video/renderer';
import { publishToSocial } from '@/lib/social/publisher';

async function runContentPipeline(keyword: string) {
  try {
    // Step 1: Research
    console.log('🔍 Starting research...');
    const research = await researchTopic({
      keyword,
      sources: ['techcrunch', 'twitter'],
      timeRange: '24h',
    });

    // Step 2: Generate Content
    console.log('✍️ Generating content...');
    const content = await generateContent({
      topic: keyword,
      format: 'toplist',
      language: 'both',
      tone: 'professional',
      researchData: research,
      aiProvider: 'claude',
    });

    // Step 3: Render Video
    console.log('🎬 Rendering video...');
    const videoPath = await renderContentVideo(content.english);

    // Step 4: Publish (optional)
    console.log('📤 Publishing...');
    await publishToSocial({
      content: content.english,
      video: videoPath,
      platforms: ['facebook', 'instagram', 'tiktok'],
    });

    console.log('✅ Pipeline completed successfully!');
    return { content, videoPath };
  } catch (error) {
    console.error('❌ Pipeline error:', error);
    throw error;
  }
}

// Execute
runContentPipeline('AI automation tools 2024');
```

## Next.js API Routes

### Create Content API

```typescript
// app/api/content/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { generateContent } from '@/lib/ai/content-generator';
import { researchTopic } from '@/lib/research/crawler';

export async function POST(request: NextRequest) {
  try {
    const { keyword, format, language } = await request.json();

    // Research phase
    const research = await researchTopic({
      keyword,
      sources: ['techcrunch', 'a16z'],
      timeRange: '24h',
    });

    // Generation phase
    const content = await generateContent({
      topic: keyword,
      format,
      language,
      researchData: research,
      aiProvider: 'claude',
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

### Render Video API

```typescript
// app/api/video/render/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { renderContentVideo } from '@/lib/video/renderer';

export async function POST(request: NextRequest) {
  try {
    const { contentId, content } = await request.json();

    const videoPath = await renderContentVideo({
      id: contentId,
      ...content,
    });

    return NextResponse.json({ 
      success: true, 
      videoUrl: `/videos/${contentId}.mp4` 
    });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

## Common Content Formats

### Toplist Format

```typescript
const toplistPrompt = `
Create a toplist article about ${topic}.
Format:
- Engaging title with number (e.g., "Top 10...")
- Brief introduction
- Each item with: Title, Description, Why it matters
- Data-backed insights from: ${JSON.stringify(researchData)}
- Conclusion with actionable takeaways
Tone: ${tone}
Language: ${language}
`;
```

### POV (Point of View) Format

```typescript
const povPrompt = `
Write a POV article about ${topic}.
Format:
- Controversial or unique angle
- Personal narrative style
- Strong opinion backed by data: ${JSON.stringify(researchData)}
- Counter-arguments addressed
- Call to action
Tone: ${tone}
Language: ${language}
`;
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
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      
      const waitTime = Math.pow(2, i) * 1000;
      console.log(`Retry ${i + 1}/${maxRetries} after ${waitTime}ms`);
      await new Promise(resolve => setTimeout(resolve, waitTime));
    }
  }
  throw new Error('Max retries exceeded');
}

// Usage
const content = await withRetry(() => 
  generateContent({ topic, format, language, researchData })
);
```

### Video Rendering Issues

```typescript
// Check Remotion configuration
import { Config } from '@remotion/cli/config';

Config.setConcurrency(1); // Reduce if memory issues
Config.setVideoImageFormat('jpeg'); // Use JPEG for smaller size
Config.setOverwriteOutput(true); // Overwrite existing files
```

### Missing Environment Variables

```typescript
// Validate environment on startup
function validateEnv() {
  const required = [
    'ANTHROPIC_API_KEY',
    'OPENAI_API_KEY',
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

## Performance Optimization

```typescript
// Parallel processing for multi-language content
async function generateMultiLanguageContent(config: any) {
  const [english, vietnamese] = await Promise.all([
    generateContent({ ...config, language: 'en' }),
    generateContent({ ...config, language: 'vi' }),
  ]);
  
  return { english, vietnamese };
}

// Cache research results
import { LRUCache } from 'lru-cache';

const researchCache = new LRUCache({
  max: 100,
  ttl: 1000 * 60 * 60, // 1 hour
});

async function cachedResearch(keyword: string) {
  const cached = researchCache.get(keyword);
  if (cached) return cached;
  
  const data = await researchTopic({ keyword });
  researchCache.set(keyword, data);
  return data;
}
```
