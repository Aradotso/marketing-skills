---
name: marketing-pipeline-share-ai-content
description: Automated AI content pipeline for research, scripting, auto-posting, and video generation using Claude, OpenAI, and Remotion
triggers:
  - "set up automated content pipeline with AI"
  - "generate content from research to video automatically"
  - "create AI-powered marketing content workflow"
  - "automate content creation with Claude and OpenAI"
  - "build content pipeline with video generation"
  - "scrape news and generate articles with AI"
  - "set up Remotion video rendering for content"
  - "create automated social media content pipeline"
---

# Marketing Pipeline Share - AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This project is an end-to-end AI-powered content automation system that handles research (web scraping), content generation (Claude/OpenAI), and video rendering (Remotion). It automatically crawls news sources, generates articles in multiple formats and languages, and produces videos optimized for social platforms.

## What It Does

- **Auto-Research**: Crawls TechCrunch, a16z, Twitter/X, LinkedIn for trending topics
- **AI Content Generation**: Creates articles in multiple formats (toplist, POV, case study, how-to)
- **Multi-language Support**: Generates Vietnamese and English content simultaneously
- **Video Rendering**: Converts content to videos using Remotion for Reels/TikTok/Shorts
- **Auto-Publishing**: Schedules and posts content to social platforms

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

## Environment Configuration

Create a `.env` file with the following variables:

```env
# AI Services
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# Web Scraping
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_url

# Video Rendering
REMOTION_LICENSE_KEY=your_remotion_license

# Social Media APIs
FACEBOOK_ACCESS_TOKEN=your_fb_token
LINKEDIN_ACCESS_TOKEN=your_linkedin_token
```

## Running the Project

```bash
# Development mode
npm run dev

# Production build
npm run build
npm start

# Run video rendering
npm run render
```

## Core API Usage

### 1. Research & Scraping Module

```typescript
import { researchTopic } from './lib/research';

async function gatherContent(keyword: string) {
  const research = await researchTopic({
    keyword: keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeframe: '24h',
    maxResults: 20
  });
  
  return {
    articles: research.articles,
    insights: research.insights,
    trending: research.trendingTopics
  };
}
```

### 2. AI Content Generation

```typescript
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

// Using Claude for Vietnamese content
async function generateWithClaude(
  prompt: string,
  format: 'toplist' | 'pov' | 'case-study' | 'how-to'
) {
  const anthropic = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY,
  });

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [
      {
        role: 'user',
        content: `Viết bài ${format} về chủ đề: ${prompt}\n\n
        Yêu cầu:
        - Dựa trên dữ liệu research đính kèm
        - Giọng văn chuyên gia nhưng dễ hiểu
        - Có số liệu và insight cụ thể
        - Độ dài 1500-2000 từ`
      }
    ],
  });

  return message.content[0].text;
}

// Using OpenAI for English content
async function generateWithOpenAI(
  prompt: string,
  tone: 'expert' | 'friendly' | 'humorous'
) {
  const openai = new OpenAI({
    apiKey: process.env.OPENAI_API_KEY,
  });

  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are a marketing content expert. Write in ${tone} tone.`
      },
      {
        role: 'user',
        content: prompt
      }
    ],
    temperature: 0.7,
    max_tokens: 3000,
  });

  return completion.choices[0].message.content;
}
```

### 3. Parallel Content Generation

```typescript
async function createMultiLanguageContent(topic: string, researchData: any) {
  const [vietnameseContent, englishContent] = await Promise.all([
    generateWithClaude(
      `${topic}\n\nResearch data: ${JSON.stringify(researchData)}`,
      'toplist'
    ),
    generateWithOpenAI(
      `Create a toplist article about ${topic} based on: ${JSON.stringify(researchData)}`,
      'expert'
    )
  ]);

  return {
    vi: vietnameseContent,
    en: englishContent,
    metadata: {
      topic,
      generatedAt: new Date(),
      sources: researchData.sources
    }
  };
}
```

### 4. Video Rendering with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import { VideoComposition } from './remotion/Composition';

async function renderContentVideo(
  contentData: {
    title: string;
    points: string[];
    images: string[];
  },
  platform: 'reels' | 'tiktok' | 'shorts'
) {
  const platformConfig = {
    reels: { width: 1080, height: 1920, fps: 30 },
    tiktok: { width: 1080, height: 1920, fps: 30 },
    shorts: { width: 1080, height: 1920, fps: 30 }
  };

  const config = platformConfig[platform];
  
  const bundleLocation = await bundle({
    entryPoint: './remotion/index.ts',
    webpackOverride: (config) => config,
  });

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: contentData,
  });

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: `out/${platform}-${Date.now()}.mp4`,
    inputProps: contentData,
  });
}
```

### 5. Complete Pipeline Workflow

```typescript
interface PipelineConfig {
  keyword: string;
  contentFormats: Array<'toplist' | 'pov' | 'case-study' | 'how-to'>;
  languages: Array<'vi' | 'en'>;
  generateVideo: boolean;
  platforms: Array<'reels' | 'tiktok' | 'shorts'>;
}

async function runContentPipeline(config: PipelineConfig) {
  // Step 1: Research
  console.log('🔍 Starting research...');
  const research = await researchTopic({
    keyword: config.keyword,
    sources: ['techcrunch', 'a16z', 'twitter'],
    timeframe: '24h'
  });

  // Step 2: Generate content for each format
  console.log('✍️ Generating content...');
  const contents = await Promise.all(
    config.contentFormats.map(async (format) => {
      const content: any = {};
      
      if (config.languages.includes('vi')) {
        content.vi = await generateWithClaude(
          `${config.keyword}\n${JSON.stringify(research)}`,
          format
        );
      }
      
      if (config.languages.includes('en')) {
        content.en = await generateWithOpenAI(
          `Write ${format} about ${config.keyword}`,
          'expert'
        );
      }
      
      return { format, content };
    })
  );

  // Step 3: Generate videos if enabled
  if (config.generateVideo) {
    console.log('🎬 Rendering videos...');
    
    for (const platform of config.platforms) {
      await renderContentVideo(
        {
          title: config.keyword,
          points: extractKeyPoints(contents[0].content.vi),
          images: research.images || []
        },
        platform
      );
    }
  }

  // Step 4: Return results
  return {
    research,
    contents,
    videos: config.generateVideo ? config.platforms : [],
    timestamp: new Date()
  };
}

// Helper function
function extractKeyPoints(content: string): string[] {
  // Extract bullet points or key sentences from content
  return content
    .split('\n')
    .filter(line => line.trim().startsWith('-') || line.trim().startsWith('•'))
    .map(line => line.replace(/^[-•]\s*/, '').trim())
    .slice(0, 5);
}
```

## Common Usage Patterns

### Quick Single Article Generation

```typescript
import { runContentPipeline } from './lib/pipeline';

// Generate single toplist article in Vietnamese
const result = await runContentPipeline({
  keyword: 'AI Marketing Trends 2024',
  contentFormats: ['toplist'],
  languages: ['vi'],
  generateVideo: false,
  platforms: []
});

console.log(result.contents[0].content.vi);
```

### Full Multi-Platform Campaign

```typescript
// Complete campaign with videos
const campaign = await runContentPipeline({
  keyword: 'Startup Fundraising Strategy',
  contentFormats: ['toplist', 'case-study', 'how-to'],
  languages: ['vi', 'en'],
  generateVideo: true,
  platforms: ['reels', 'tiktok', 'shorts']
});

// Schedule posts
for (const content of campaign.contents) {
  await schedulePost({
    content: content.content.vi,
    platform: 'facebook',
    scheduledTime: new Date(Date.now() + 3600000) // 1 hour from now
  });
}
```

### Custom Tone & Style

```typescript
// Generate with specific tone
async function generateCustomContent(topic: string, style: {
  tone: 'expert' | 'friendly' | 'humorous';
  length: 'short' | 'medium' | 'long';
  audience: string;
}) {
  const wordCounts = { short: 800, medium: 1500, long: 2500 };
  
  const content = await generateWithOpenAI(
    `Write about ${topic} for ${style.audience} audience. 
    Target length: ${wordCounts[style.length]} words.`,
    style.tone
  );
  
  return content;
}
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement retry logic for API calls
async function retryableAPICall<T>(
  fn: () => Promise<T>,
  maxRetries: number = 3
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error: any) {
      if (error?.status === 429 && i < maxRetries - 1) {
        await new Promise(resolve => setTimeout(resolve, 2000 * (i + 1)));
        continue;
      }
      throw error;
    }
  }
  throw new Error('Max retries exceeded');
}

// Usage
const content = await retryableAPICall(() => 
  generateWithClaude(prompt, 'toplist')
);
```

### Video Rendering Errors

```typescript
// Check Remotion setup
import { validateComposition } from './lib/video-utils';

try {
  await renderContentVideo(data, 'reels');
} catch (error) {
  console.error('Video render failed:', error);
  
  // Fallback: generate static image
  await generateStaticImage(data);
}
```

### Memory Issues with Large Research

```typescript
// Process research in chunks
async function processLargeResearch(keyword: string) {
  const sources = ['techcrunch', 'a16z', 'twitter', 'linkedin'];
  const results = [];
  
  for (const source of sources) {
    const data = await researchTopic({
      keyword,
      sources: [source],
      maxResults: 10
    });
    results.push(data);
    
    // Clear memory between requests
    await new Promise(resolve => setTimeout(resolve, 1000));
  }
  
  return results;
}
```

## Best Practices

1. **Always validate API keys** before starting pipeline
2. **Cache research results** to avoid duplicate scraping
3. **Use streaming** for long content generation
4. **Batch video rendering** during off-peak hours
5. **Monitor API costs** with usage tracking middleware
6. **Version control prompts** for reproducible results
