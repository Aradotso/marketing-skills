---
name: marketing-pipeline-automation
description: AI-powered content automation pipeline from research to script writing to video generation with multi-language support
triggers:
  - automate content creation pipeline
  - build AI content workflow
  - generate video from articles automatically
  - crawl news and create content
  - setup marketing automation pipeline
  - create multilingual content with AI
  - build content research to video pipeline
  - automate social media content generation
---

# Marketing Pipeline Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with Ultimate AI Content Pipeline, a complete automation system that transforms keywords into researched articles and videos. The pipeline crawls real-time data from sources like TechCrunch, a16z, Twitter/X, and LinkedIn, generates content in multiple formats and languages using Claude/OpenAI, and renders videos using Remotion.

## What This Project Does

Marketing Pipeline Automation is an end-to-end content creation system that:

- **Auto-researches** trending topics from major news sources and social platforms
- **Generates content** in multiple formats (toplist, POV, case study, how-to) and languages (English, Vietnamese)
- **Renders videos** automatically from written content using Remotion
- **Optimizes for platforms** like Reels, TikTok, and YouTube Shorts

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
# AI Provider Keys
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# Research/Crawling APIs
RAPIDAPI_KEY=your_rapidapi_key

# Database (if using)
DATABASE_URL=your_database_connection_string

# Next.js
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/             
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── crawlers/    # News/social media crawlers
│   │   └── video/       # Remotion video rendering
│   ├── types/           # TypeScript type definitions
│   └── utils/           # Utility functions
├── public/              # Static assets
└── remotion/            # Video templates
```

## Key Commands

```bash
# Development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Run video rendering
npm run render

# Type checking
npm run type-check

# Linting
npm run lint
```

## Core API Usage

### 1. Research & Data Crawling

```typescript
import { crawlTechCrunch, crawlTwitter } from '@/lib/crawlers';

async function gatherResearch(keyword: string) {
  // Crawl multiple sources
  const [techcrunchData, twitterData] = await Promise.all([
    crawlTechCrunch(keyword),
    crawlTwitter(keyword, { hours: 24 })
  ]);

  return {
    articles: techcrunchData.articles,
    socialPosts: twitterData.posts,
    insights: [...techcrunchData.insights, ...twitterData.insights]
  };
}
```

### 2. AI Content Generation with Claude

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

async function generateContent(
  topic: string,
  format: 'toplist' | 'pov' | 'case-study' | 'how-to',
  language: 'en' | 'vi'
) {
  const prompt = `Create a ${format} article about ${topic} in ${language}.
  Use the following research data: ${researchData}
  
  Format requirements:
  - Engaging headline
  - Data-backed insights
  - ${format === 'toplist' ? 'Numbered list format' : 'Narrative structure'}
  - Call-to-action at the end
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

### 3. OpenAI Alternative

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateWithGPT(
  topic: string,
  researchData: any,
  tone: 'expert' | 'friendly' | 'humorous'
) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are a content creator with a ${tone} tone. Create engaging, data-driven content.`
      },
      {
        role: 'user',
        content: `Topic: ${topic}\nResearch: ${JSON.stringify(researchData)}`
      }
    ],
    temperature: 0.7,
  });

  return completion.choices[0].message.content;
}
```

### 4. Video Rendering with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

async function renderContentVideo(
  content: string,
  platform: 'reels' | 'tiktok' | 'shorts'
) {
  // Platform dimensions
  const dimensions = {
    reels: { width: 1080, height: 1920 },
    tiktok: { width: 1080, height: 1920 },
    shorts: { width: 1080, height: 1920 }
  };

  const bundleLocation = await bundle({
    entryPoint: path.join(process.cwd(), 'remotion/index.ts'),
  });

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      content,
      ...dimensions[platform]
    },
  });

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: `out/${platform}-video.mp4`,
  });

  return `out/${platform}-video.mp4`;
}
```

## Complete Pipeline Example

```typescript
import { crawlMultipleSources } from '@/lib/crawlers';
import { generateContent } from '@/lib/ai/claude';
import { renderVideo } from '@/lib/video/remotion';

interface ContentPipelineConfig {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  languages: ('en' | 'vi')[];
  platforms: ('reels' | 'tiktok' | 'shorts')[];
}

async function runContentPipeline(config: ContentPipelineConfig) {
  console.log(`🔍 Starting pipeline for: ${config.keyword}`);

  // Step 1: Research
  console.log('📡 Gathering research...');
  const research = await crawlMultipleSources({
    keyword: config.keyword,
    sources: ['techcrunch', 'twitter', 'linkedin'],
    timeframe: { hours: 24 }
  });

  const results = [];

  // Step 2: Generate content for each language
  for (const lang of config.languages) {
    console.log(`✍️  Generating ${lang} content...`);
    
    const content = await generateContent({
      topic: config.keyword,
      format: config.format,
      language: lang,
      research: research.insights,
      tone: 'expert'
    });

    results.push({
      language: lang,
      content,
      metadata: {
        wordCount: content.split(' ').length,
        generatedAt: new Date().toISOString()
      }
    });

    // Step 3: Render videos for each platform
    for (const platform of config.platforms) {
      console.log(`🎬 Rendering ${platform} video (${lang})...`);
      
      const videoPath = await renderVideo({
        content,
        platform,
        language: lang,
        outputDir: `./output/${config.keyword}/${lang}`
      });

      results[results.length - 1][platform] = videoPath;
    }
  }

  console.log('✅ Pipeline complete!');
  return results;
}

// Usage
const output = await runContentPipeline({
  keyword: 'AI marketing trends 2026',
  format: 'toplist',
  languages: ['en', 'vi'],
  platforms: ['reels', 'tiktok']
});
```

## Data Crawler Implementation

```typescript
import axios from 'axios';

interface CrawlerOptions {
  keyword: string;
  hours?: number;
  limit?: number;
}

export async function crawlTechCrunch(options: CrawlerOptions) {
  const { keyword, hours = 24, limit = 10 } = options;
  
  try {
    const response = await axios.get('https://techcrunch-api.p.rapidapi.com/search', {
      params: {
        query: keyword,
        limit,
      },
      headers: {
        'X-RapidAPI-Key': process.env.RAPIDAPI_KEY,
        'X-RapidAPI-Host': 'techcrunch-api.p.rapidapi.com'
      }
    });

    return {
      articles: response.data.articles.map((article: any) => ({
        title: article.title,
        summary: article.summary,
        url: article.url,
        publishedAt: article.publishedAt,
        insights: extractInsights(article.content)
      })),
      insights: response.data.articles.flatMap((a: any) => 
        extractInsights(a.content)
      )
    };
  } catch (error) {
    console.error('TechCrunch crawl error:', error);
    throw error;
  }
}

function extractInsights(content: string): string[] {
  // Extract key statistics, quotes, and data points
  const insights: string[] = [];
  
  const statRegex = /(\d+%|\$\d+[MBK]?|\d+\s+(million|billion))/gi;
  const stats = content.match(statRegex) || [];
  
  stats.forEach(stat => {
    const context = content.substring(
      Math.max(0, content.indexOf(stat) - 100),
      Math.min(content.length, content.indexOf(stat) + 100)
    );
    insights.push(context.trim());
  });

  return insights;
}
```

## Remotion Video Template

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, Sequence, useCurrentFrame } from 'remotion';
import { FC } from 'react';

interface ContentVideoProps {
  content: string;
  width: number;
  height: number;
}

export const ContentVideo: FC<ContentVideoProps> = ({ 
  content, 
  width, 
  height 
}) => {
  const frame = useCurrentFrame();
  
  // Split content into slides
  const slides = content.split('\n\n').filter(Boolean);
  const framesPerSlide = 60; // 2 seconds at 30fps

  return (
    <AbsoluteFill style={{ backgroundColor: '#000' }}>
      {slides.map((slide, index) => (
        <Sequence
          key={index}
          from={index * framesPerSlide}
          durationInFrames={framesPerSlide}
        >
          <AbsoluteFill
            style={{
              justifyContent: 'center',
              alignItems: 'center',
              padding: 60,
            }}
          >
            <h2
              style={{
                color: '#fff',
                fontSize: 48,
                textAlign: 'center',
                lineHeight: 1.4,
                opacity: Math.min(1, (frame - index * framesPerSlide) / 10),
              }}
            >
              {slide}
            </h2>
          </AbsoluteFill>
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};
```

## Common Patterns

### Multi-Language Content Generation

```typescript
async function generateMultilingualContent(
  topic: string,
  research: any
) {
  const languages = ['en', 'vi'];
  
  const contentByLanguage = await Promise.all(
    languages.map(async (lang) => {
      const content = await generateContent({
        topic,
        format: 'pov',
        language: lang,
        research,
        tone: 'expert'
      });

      return { language: lang, content };
    })
  );

  return Object.fromEntries(
    contentByLanguage.map(({ language, content }) => [language, content])
  );
}
```

### Batch Processing

```typescript
async function processBatch(keywords: string[]) {
  const batchSize = 3; // Process 3 at a time
  const results = [];

  for (let i = 0; i < keywords.length; i += batchSize) {
    const batch = keywords.slice(i, i + batchSize);
    
    const batchResults = await Promise.all(
      batch.map(keyword => runContentPipeline({
        keyword,
        format: 'toplist',
        languages: ['en', 'vi'],
        platforms: ['reels']
      }))
    );

    results.push(...batchResults);
    
    // Rate limiting pause
    await new Promise(resolve => setTimeout(resolve, 2000));
  }

  return results;
}
```

## Troubleshooting

### API Rate Limiting

```typescript
import pRetry from 'p-retry';

async function withRetry<T>(
  fn: () => Promise<T>,
  maxRetries = 3
): Promise<T> {
  return pRetry(fn, {
    retries: maxRetries,
    onFailedAttempt: (error) => {
      console.log(
        `Attempt ${error.attemptNumber} failed. ${error.retriesLeft} retries left.`
      );
    },
  });
}

// Usage
const content = await withRetry(() => 
  generateContent(topic, format, language)
);
```

### Video Rendering Memory Issues

```typescript
// Render in chunks for large content
async function renderLargeVideo(content: string, platform: string) {
  const maxSlides = 10;
  const slides = content.split('\n\n');
  
  if (slides.length <= maxSlides) {
    return renderVideo({ content, platform });
  }

  // Split into multiple videos
  const chunks = [];
  for (let i = 0; i < slides.length; i += maxSlides) {
    const chunk = slides.slice(i, i + maxSlides).join('\n\n');
    const videoPath = await renderVideo({
      content: chunk,
      platform,
      outputDir: `./output/chunk-${i}`
    });
    chunks.push(videoPath);
  }

  return chunks;
}
```

### Error Handling Pipeline

```typescript
async function safePipeline(config: ContentPipelineConfig) {
  try {
    return await runContentPipeline(config);
  } catch (error) {
    if (error.code === 'RATE_LIMIT') {
      console.warn('Rate limit hit, waiting 60s...');
      await new Promise(resolve => setTimeout(resolve, 60000));
      return safePipeline(config);
    }
    
    if (error.code === 'INSUFFICIENT_QUOTA') {
      console.error('API quota exceeded. Switch to backup provider.');
      // Fallback to alternative AI provider
      return runContentPipeline({
        ...config,
        aiProvider: 'openai' // switch from claude
      });
    }

    throw error;
  }
}
```

This skill enables comprehensive automation of content marketing workflows using TypeScript, AI models, and video rendering technology.
