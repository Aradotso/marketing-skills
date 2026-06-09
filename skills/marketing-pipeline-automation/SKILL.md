---
name: marketing-pipeline-automation
description: Automated AI content pipeline for research, scriptwriting, and video generation using Claude, OpenAI, and Remotion
triggers:
  - how do I automate content research and creation
  - generate marketing content from keywords automatically
  - create videos from text content using AI
  - set up automated content pipeline with Claude
  - research and write articles with AI automation
  - build content generation workflow with Remotion
  - crawl news and generate social media content
  - automate marketing content from research to video
---

# Marketing Pipeline Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This project is an end-to-end automated content pipeline that transforms keywords into complete marketing assets including research, written content in multiple formats, and rendered videos. It leverages Claude 3/OpenAI for content generation and Remotion for video rendering.

## What It Does

The Ultimate AI Content Pipeline automates the entire content creation workflow:

1. **Auto-Research**: Crawls real-time data from TechCrunch, a16z, Twitter/X, LinkedIn for fresh insights
2. **Content Generation**: Creates articles in multiple formats (top lists, POV pieces, case studies, how-tos) using Claude/OpenAI
3. **Multi-Language**: Generates content in both English and Vietnamese with customizable tones
4. **Video Rendering**: Automatically converts written content into videos and infographics using Remotion
5. **Platform Optimization**: Exports content optimized for Reels, TikTok, and Shorts

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

# Set up environment variables
cp .env.example .env
```

## Configuration

Create a `.env` file with the following variables:

```env
# AI API Keys
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_url

# Remotion Configuration
REMOTION_BROWSER_EXECUTABLE=/path/to/chrome

# Content Settings
DEFAULT_LANGUAGE=en
CONTENT_TONE=professional
```

## Project Structure

```typescript
// Typical project structure
├── app/                    # Next.js app directory
├── components/            # React components
├── lib/                   # Core utilities
│   ├── ai/               # AI integration modules
│   ├── crawler/          # Web scraping logic
│   └── video/            # Remotion video generation
├── remotion/             # Remotion compositions
└── public/               # Static assets
```

## Core API Usage

### 1. Content Research Module

```typescript
import { researchTopic } from '@/lib/crawler/research';
import { analyzeInsights } from '@/lib/ai/analyzer';

async function autoResearch(keyword: string) {
  // Crawl latest news and data
  const rawData = await researchTopic({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeframe: '24h'
  });
  
  // Analyze with AI
  const insights = await analyzeInsights(rawData, {
    apiKey: process.env.ANTHROPIC_API_KEY,
    model: 'claude-3-opus-20240229'
  });
  
  return insights;
}

// Usage
const insights = await autoResearch('AI marketing automation');
```

### 2. Content Generation

```typescript
import { generateContent } from '@/lib/ai/content-generator';

async function createArticle(
  keyword: string,
  insights: any,
  format: 'toplist' | 'pov' | 'case-study' | 'how-to'
) {
  const content = await generateContent({
    keyword,
    insights,
    format,
    language: 'en',
    tone: 'professional',
    apiKey: process.env.OPENAI_API_KEY,
    model: 'gpt-4-turbo-preview'
  });
  
  return content;
}

// Generate multiple formats
const formats = ['toplist', 'pov', 'case-study', 'how-to'] as const;
const articles = await Promise.all(
  formats.map(format => createArticle('AI tools', insights, format))
);
```

### 3. Bilingual Content Generation

```typescript
import { generateBilingualContent } from '@/lib/ai/bilingual';

async function createMultiLanguageContent(keyword: string) {
  const bilingualContent = await generateBilingualContent({
    keyword,
    languages: ['en', 'vi'],
    format: 'toplist',
    apiKey: process.env.ANTHROPIC_API_KEY
  });
  
  return {
    english: bilingualContent.en,
    vietnamese: bilingualContent.vi
  };
}
```

### 4. Video Generation with Remotion

```typescript
import { renderVideo } from '@/lib/video/renderer';
import { createVideoComposition } from '@/remotion/compositions';

async function generateVideoFromContent(article: {
  title: string;
  content: string;
  keyPoints: string[];
}) {
  // Create Remotion composition
  const composition = createVideoComposition({
    title: article.title,
    keyPoints: article.keyPoints,
    duration: 60, // seconds
    format: 'vertical' // for Reels/TikTok/Shorts
  });
  
  // Render video
  const videoPath = await renderVideo({
    composition,
    outputPath: './output/video.mp4',
    format: '1080x1920', // vertical format
    fps: 30
  });
  
  return videoPath;
}
```

## Complete Pipeline Example

```typescript
import { runContentPipeline } from '@/lib/pipeline';

async function executeFullPipeline(keyword: string) {
  const result = await runContentPipeline({
    keyword,
    
    // Research configuration
    research: {
      sources: ['techcrunch', 'a16z', 'twitter'],
      timeframe: '24h'
    },
    
    // Content generation
    content: {
      formats: ['toplist', 'how-to'],
      languages: ['en', 'vi'],
      tone: 'professional'
    },
    
    // Video generation
    video: {
      enabled: true,
      platforms: ['reels', 'tiktok', 'shorts'],
      duration: 60
    },
    
    // API keys from env
    apiKeys: {
      openai: process.env.OPENAI_API_KEY,
      anthropic: process.env.ANTHROPIC_API_KEY,
      rapidapi: process.env.RAPIDAPI_KEY
    }
  });
  
  return {
    research: result.insights,
    articles: result.content,
    videos: result.videos
  };
}

// Execute
const output = await executeFullPipeline('content marketing automation');
console.log('Generated:', output);
```

## API Routes (Next.js)

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline';

export async function POST(request: NextRequest) {
  try {
    const { keyword, formats, languages } = await request.json();
    
    const result = await runContentPipeline({
      keyword,
      content: { formats, languages },
      apiKeys: {
        openai: process.env.OPENAI_API_KEY,
        anthropic: process.env.ANTHROPIC_API_KEY
      }
    });
    
    return NextResponse.json({ success: true, data: result });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

## CLI Commands (if applicable)

```bash
# Run the development server
npm run dev

# Build for production
npm run build

# Start production server
npm run start

# Render Remotion video
npm run render -- --composition=ContentVideo --output=output.mp4

# Run content pipeline from CLI
npm run pipeline -- --keyword="AI marketing" --format=toplist
```

## Common Patterns

### Pattern 1: Scheduled Content Generation

```typescript
import cron from 'node-cron';

// Schedule daily content generation
cron.schedule('0 9 * * *', async () => {
  const keywords = ['AI trends', 'marketing automation', 'social media'];
  
  for (const keyword of keywords) {
    await executeFullPipeline(keyword);
  }
});
```

### Pattern 2: Batch Processing

```typescript
async function batchGenerateContent(keywords: string[]) {
  const results = await Promise.allSettled(
    keywords.map(keyword => 
      executeFullPipeline(keyword).catch(err => ({
        keyword,
        error: err.message
      }))
    )
  );
  
  return results;
}
```

### Pattern 3: Custom Content Workflow

```typescript
import { researchTopic } from '@/lib/crawler/research';
import { generateContent } from '@/lib/ai/content-generator';
import { renderVideo } from '@/lib/video/renderer';

async function customWorkflow(keyword: string) {
  // Step 1: Research
  const insights = await researchTopic({ keyword, sources: ['techcrunch'] });
  
  // Step 2: Generate content with custom prompt
  const content = await generateContent({
    keyword,
    insights,
    customPrompt: 'Write as a thought leader in the industry',
    apiKey: process.env.ANTHROPIC_API_KEY
  });
  
  // Step 3: Create video only if content is high quality
  if (content.qualityScore > 0.8) {
    await renderVideo({ content });
  }
  
  return content;
}
```

## Troubleshooting

### API Rate Limits

```typescript
import { RateLimiter } from '@/lib/utils/rate-limiter';

const limiter = new RateLimiter({
  maxRequests: 50,
  perMinutes: 1
});

async function generateWithRateLimit(keyword: string) {
  await limiter.waitForSlot();
  return await generateContent({ keyword });
}
```

### Error Handling

```typescript
async function safeContentGeneration(keyword: string) {
  try {
    return await executeFullPipeline(keyword);
  } catch (error) {
    if (error.code === 'RATE_LIMIT_EXCEEDED') {
      console.log('Rate limit hit, retrying in 60s...');
      await new Promise(resolve => setTimeout(resolve, 60000));
      return await executeFullPipeline(keyword);
    }
    
    if (error.code === 'INVALID_API_KEY') {
      throw new Error('Check your API keys in .env file');
    }
    
    throw error;
  }
}
```

### Video Rendering Issues

```typescript
// Ensure Chrome/Chromium is installed for Remotion
import { getChromiumPath } from '@/lib/video/chromium';

async function checkVideoSetup() {
  const chromiumPath = await getChromiumPath();
  if (!chromiumPath) {
    throw new Error(
      'Chromium not found. Install with: npx remotion browser ensure'
    );
  }
  return true;
}
```

### Memory Management for Large Batches

```typescript
async function processLargeKeywordList(keywords: string[]) {
  const chunkSize = 5;
  const results = [];
  
  for (let i = 0; i < keywords.length; i += chunkSize) {
    const chunk = keywords.slice(i, i + chunkSize);
    const chunkResults = await Promise.all(
      chunk.map(k => executeFullPipeline(k))
    );
    results.push(...chunkResults);
    
    // Give system time to clean up memory
    await new Promise(resolve => setTimeout(resolve, 5000));
  }
  
  return results;
}
```

## Best Practices

1. **Always use environment variables** for API keys
2. **Implement rate limiting** to avoid API quota exhaustion
3. **Cache research results** to reduce redundant API calls
4. **Use TypeScript types** for configuration objects
5. **Handle errors gracefully** with retries for transient failures
6. **Monitor API costs** by logging token usage
7. **Test video rendering locally** before deploying to production
