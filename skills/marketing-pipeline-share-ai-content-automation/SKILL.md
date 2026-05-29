---
name: marketing-pipeline-share-ai-content-automation
description: Automated AI content pipeline for research, scriptwriting, posting, and video generation using Claude, OpenAI, and Remotion
triggers:
  - automate content creation with AI research and video generation
  - set up marketing pipeline for auto-posting content
  - generate videos from AI-written scripts automatically
  - crawl news sources and create content with Claude
  - build automated content workflow with Remotion rendering
  - create multi-format AI content from keywords
  - automate research to video content pipeline
  - set up TypeScript AI content automation system
---

# Marketing Pipeline Share - AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with **marketing-pipeline-share**, an all-in-one automated content pipeline that transforms keywords into published content and videos. The system handles research (crawling TechCrunch, Twitter, LinkedIn), AI content generation (Claude/OpenAI), and video rendering (Remotion) — all automatically.

## What This Project Does

Marketing Pipeline Share is a TypeScript-based automation system that:

- **Auto-crawls** latest news from major sources (TechCrunch, a16z, Twitter, LinkedIn)
- **Generates content** in multiple formats (toplist, POV, case study, how-to) using Claude 3 or OpenAI
- **Supports bilingual** content (English & Vietnamese) with customizable tone
- **Renders videos** and infographics automatically using Remotion
- **Optimizes** for social platforms (Reels, TikTok, Shorts)
- **Automates posting** to social media pages

Perfect for content creators, marketers, and businesses wanting to scale content production 10x.

## Installation

### Prerequisites

```bash
# Node.js 18+ required
node --version

# Install pnpm (recommended) or npm
npm install -g pnpm
```

### Setup Steps

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
pnpm install
# or
npm install

# Copy environment template
cp .env.example .env
```

### Environment Configuration

Create `.env` file with required API keys:

```bash
# AI Provider Keys
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# RapidAPI for News Crawling
RAPIDAPI_KEY=your_rapidapi_key

# Remotion (for video rendering)
REMOTION_LICENSE_KEY=your_remotion_key

# Database (if using)
DATABASE_URL=postgresql://user:password@localhost:5432/marketing_pipeline

# Social Media Auto-posting (optional)
FACEBOOK_PAGE_ACCESS_TOKEN=your_fb_token
TWITTER_API_KEY=your_twitter_key
LINKEDIN_ACCESS_TOKEN=your_linkedin_token

# Application Settings
NEXT_PUBLIC_APP_URL=http://localhost:3000
NODE_ENV=development
```

### Start Development Server

```bash
# Run Next.js development server
pnpm dev
# or
npm run dev

# Access at http://localhost:3000
```

## Core Architecture

### Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app router pages
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI providers (Claude, OpenAI)
│   │   ├── crawlers/    # News scraping modules
│   │   ├── generators/  # Content generators
│   │   └── video/       # Remotion video rendering
│   ├── services/        # Business logic
│   └── types/           # TypeScript definitions
├── remotion/            # Video templates
└── public/              # Static assets
```

## Key Usage Patterns

### 1. Content Generation from Keywords

```typescript
// src/lib/generators/content-generator.ts
import { Anthropic } from '@anthropic-ai/sdk';
import { crawlLatestNews } from '../crawlers/news-crawler';

interface ContentConfig {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
}

export async function generateContent(config: ContentConfig) {
  const anthropic = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY,
  });

  // Step 1: Research - crawl latest news
  const researchData = await crawlLatestNews({
    keyword: config.keyword,
    sources: ['techcrunch', 'twitter', 'linkedin'],
    timeframe: '24h',
  });

  // Step 2: Generate content with Claude
  const prompt = buildPrompt(config, researchData);
  
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [
      {
        role: 'user',
        content: prompt,
      },
    ],
  });

  const content = message.content[0].type === 'text' 
    ? message.content[0].text 
    : '';

  return {
    title: extractTitle(content),
    body: content,
    metadata: {
      sources: researchData.sources,
      generatedAt: new Date().toISOString(),
      format: config.format,
    },
  };
}

function buildPrompt(config: ContentConfig, research: any): string {
  return `You are a professional content writer. Create a ${config.format} article about "${config.keyword}".

Language: ${config.language}
Tone: ${config.tone}

Research data from the last 24 hours:
${JSON.stringify(research, null, 2)}

Requirements:
- Use real data and statistics from the research
- ${config.language === 'vi' ? 'Write in Vietnamese' : 'Write in English'}
- Make it engaging and actionable
- Include specific examples and insights
- Format: ${config.format}

Generate the complete article now:`;
}

function extractTitle(content: string): string {
  const match = content.match(/^#\s+(.+)$/m);
  return match ? match[1] : 'Untitled';
}
```

### 2. News Crawling System

```typescript
// src/lib/crawlers/news-crawler.ts
import axios from 'axios';

interface CrawlConfig {
  keyword: string;
  sources: string[];
  timeframe: string;
}

interface NewsArticle {
  title: string;
  url: string;
  source: string;
  publishedAt: string;
  content: string;
}

export async function crawlLatestNews(config: CrawlConfig): Promise<NewsArticle[]> {
  const articles: NewsArticle[] = [];

  for (const source of config.sources) {
    switch (source) {
      case 'techcrunch':
        articles.push(...await crawlTechCrunch(config.keyword));
        break;
      case 'twitter':
        articles.push(...await crawlTwitter(config.keyword));
        break;
      case 'linkedin':
        articles.push(...await crawlLinkedIn(config.keyword));
        break;
    }
  }

  return filterByTimeframe(articles, config.timeframe);
}

async function crawlTechCrunch(keyword: string): Promise<NewsArticle[]> {
  try {
    const response = await axios.get('https://techcrunch.com/wp-json/wp/v2/posts', {
      params: {
        search: keyword,
        per_page: 10,
        _embed: true,
      },
    });

    return response.data.map((post: any) => ({
      title: post.title.rendered,
      url: post.link,
      source: 'TechCrunch',
      publishedAt: post.date,
      content: post.excerpt.rendered.replace(/<[^>]*>/g, ''),
    }));
  } catch (error) {
    console.error('TechCrunch crawl failed:', error);
    return [];
  }
}

async function crawlTwitter(keyword: string): Promise<NewsArticle[]> {
  // Using RapidAPI for Twitter search
  try {
    const response = await axios.get('https://twitter-api45.p.rapidapi.com/search.php', {
      params: { query: keyword, search_type: 'Latest' },
      headers: {
        'X-RapidAPI-Key': process.env.RAPIDAPI_KEY,
        'X-RapidAPI-Host': 'twitter-api45.p.rapidapi.com',
      },
    });

    return response.data.timeline.map((tweet: any) => ({
      title: tweet.text.substring(0, 100),
      url: `https://twitter.com/i/web/status/${tweet.tweet_id}`,
      source: 'Twitter',
      publishedAt: tweet.created_at,
      content: tweet.text,
    }));
  } catch (error) {
    console.error('Twitter crawl failed:', error);
    return [];
  }
}

function filterByTimeframe(articles: NewsArticle[], timeframe: string): NewsArticle[] {
  const now = new Date();
  const hours = parseInt(timeframe);
  const cutoff = new Date(now.getTime() - hours * 60 * 60 * 1000);

  return articles.filter(article => 
    new Date(article.publishedAt) > cutoff
  );
}
```

### 3. Video Generation with Remotion

```typescript
// src/lib/video/video-generator.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';
import { writeFile } from 'fs/promises';

interface VideoConfig {
  title: string;
  content: string;
  format: 'reels' | 'tiktok' | 'shorts';
  duration: number;
}

export async function generateVideo(config: VideoConfig): Promise<string> {
  const compositionId = 'ContentVideo';
  
  // Bundle Remotion composition
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  // Get composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: {
      title: config.title,
      content: config.content,
      format: config.format,
    },
  });

  // Render video
  const outputLocation = path.resolve(
    `./output/video-${Date.now()}.mp4`
  );

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: {
      title: config.title,
      content: config.content,
      format: config.format,
    },
  });

  return outputLocation;
}

// Get video dimensions based on platform
export function getVideoDimensions(format: string) {
  const formats = {
    reels: { width: 1080, height: 1920 }, // 9:16
    tiktok: { width: 1080, height: 1920 }, // 9:16
    shorts: { width: 1080, height: 1920 }, // 9:16
    landscape: { width: 1920, height: 1080 }, // 16:9
  };
  
  return formats[format as keyof typeof formats] || formats.reels;
}
```

### 4. Remotion Video Template

```tsx
// remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig, interpolate } from 'remotion';

interface ContentVideoProps {
  title: string;
  content: string;
  format: string;
}

export const ContentVideo: React.FC<ContentVideoProps> = ({
  title,
  content,
  format,
}) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  // Fade in animation
  const opacity = interpolate(frame, [0, 30], [0, 1], {
    extrapolateRight: 'clamp',
  });

  // Scale animation
  const scale = interpolate(frame, [0, 30], [0.8, 1], {
    extrapolateRight: 'clamp',
  });

  return (
    <AbsoluteFill
      style={{
        backgroundColor: '#1a1a1a',
        justifyContent: 'center',
        alignItems: 'center',
        fontFamily: 'Arial, sans-serif',
      }}
    >
      <div
        style={{
          opacity,
          transform: `scale(${scale})`,
          textAlign: 'center',
          padding: '40px',
          maxWidth: '80%',
        }}
      >
        <h1
          style={{
            fontSize: '48px',
            fontWeight: 'bold',
            color: '#ffffff',
            marginBottom: '20px',
            textShadow: '2px 2px 4px rgba(0,0,0,0.5)',
          }}
        >
          {title}
        </h1>
        <p
          style={{
            fontSize: '24px',
            color: '#e0e0e0',
            lineHeight: 1.6,
          }}
        >
          {content.substring(0, 200)}...
        </p>
      </div>
    </AbsoluteFill>
  );
};
```

### 5. Complete Pipeline Orchestration

```typescript
// src/services/pipeline-service.ts
import { generateContent } from '../lib/generators/content-generator';
import { generateVideo } from '../lib/video/video-generator';
import { postToSocialMedia } from '../lib/social/poster';

interface PipelineConfig {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  platforms: string[];
  autoPost: boolean;
}

export async function runContentPipeline(config: PipelineConfig) {
  console.log(`🚀 Starting pipeline for keyword: ${config.keyword}`);

  // Step 1: Generate content
  console.log('📝 Generating content...');
  const content = await generateContent({
    keyword: config.keyword,
    format: config.format,
    language: config.language,
    tone: config.tone,
  });

  // Step 2: Generate video
  console.log('🎬 Generating video...');
  const videoPath = await generateVideo({
    title: content.title,
    content: content.body,
    format: 'reels',
    duration: 30,
  });

  // Step 3: Auto-post if enabled
  if (config.autoPost) {
    console.log('📤 Posting to social media...');
    const results = await postToSocialMedia({
      platforms: config.platforms,
      content: {
        text: content.title,
        videoPath,
      },
    });
    
    return {
      content,
      videoPath,
      postResults: results,
    };
  }

  return {
    content,
    videoPath,
  };
}
```

### 6. API Route Example (Next.js)

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/services/pipeline-service';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    
    const { keyword, format, language, tone, platforms, autoPost } = body;

    if (!keyword) {
      return NextResponse.json(
        { error: 'Keyword is required' },
        { status: 400 }
      );
    }

    const result = await runContentPipeline({
      keyword,
      format: format || 'toplist',
      language: language || 'en',
      tone: tone || 'expert',
      platforms: platforms || ['facebook'],
      autoPost: autoPost || false,
    });

    return NextResponse.json({
      success: true,
      data: result,
    });
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { error: 'Pipeline execution failed' },
      { status: 500 }
    );
  }
}
```

## CLI Commands

```bash
# Development
pnpm dev              # Start development server
pnpm build            # Build for production
pnpm start            # Start production server

# Video Rendering
pnpm remotion render  # Render Remotion video
pnpm remotion preview # Preview video template

# Testing
pnpm test             # Run tests
pnpm lint             # Lint code
```

## Common Workflows

### Workflow 1: Generate Content Only

```typescript
import { generateContent } from './lib/generators/content-generator';

const content = await generateContent({
  keyword: 'AI automation trends 2024',
  format: 'toplist',
  language: 'en',
  tone: 'expert',
});

console.log(content.title);
console.log(content.body);
```

### Workflow 2: Full Pipeline with Video

```typescript
import { runContentPipeline } from './services/pipeline-service';

const result = await runContentPipeline({
  keyword: 'Marketing automation',
  format: 'how-to',
  language: 'vi',
  tone: 'friendly',
  platforms: ['facebook', 'tiktok'],
  autoPost: true,
});
```

### Workflow 3: Custom Research Source

```typescript
import { crawlLatestNews } from './lib/crawlers/news-crawler';

const research = await crawlLatestNews({
  keyword: 'SaaS marketing',
  sources: ['techcrunch', 'twitter'],
  timeframe: '48h',
});

// Use research data for custom processing
```

## Troubleshooting

### API Key Issues

**Problem**: `Error: Anthropic API key not found`

**Solution**:
```bash
# Verify .env file exists and has correct keys
cat .env | grep ANTHROPIC_API_KEY

# Restart development server after adding keys
pnpm dev
```

### Video Rendering Fails

**Problem**: `Remotion render error: ENOENT`

**Solution**:
```bash
# Ensure output directory exists
mkdir -p output

# Check Remotion license key
echo $REMOTION_LICENSE_KEY

# Install missing dependencies
pnpm install @remotion/bundler @remotion/renderer
```

### Crawling Returns Empty Results

**Problem**: News crawlers return no articles

**Solution**:
```typescript
// Add error handling and logging
async function crawlWithRetry(source: string, keyword: string) {
  let retries = 3;
  while (retries > 0) {
    try {
      const results = await crawlSource(source, keyword);
      if (results.length > 0) return results;
    } catch (error) {
      console.error(`Crawl attempt failed: ${error}`);
      retries--;
    }
  }
  return [];
}
```

### Rate Limiting

**Problem**: API rate limits exceeded

**Solution**:
```typescript
// Implement rate limiting
import pLimit from 'p-limit';

const limit = pLimit(2); // Max 2 concurrent requests

const promises = sources.map(source => 
  limit(() => crawlSource(source, keyword))
);

const results = await Promise.all(promises);
```

### TypeScript Errors

**Problem**: Type errors in generated code

**Solution**:
```bash
# Regenerate types
pnpm build

# Check tsconfig.json
cat tsconfig.json

# Install type definitions
pnpm add -D @types/node @types/react
```

## Best Practices

1. **Always validate environment variables on startup**
2. **Implement proper error handling for external API calls**
3. **Cache crawled data to avoid redundant requests**
4. **Use queue systems (Bull, BullMQ) for large-scale automation**
5. **Monitor API usage to stay within rate limits**
6. **Store generated content in database for tracking**
7. **Implement webhook notifications for pipeline completion**

This skill provides comprehensive guidance for AI agents to effectively use the marketing-pipeline-share automation system for content research, generation, and distribution.
