---
name: marketing-pipeline-share-automation
description: Automated AI content pipeline from research to video generation using Claude, OpenAI, and Remotion
triggers:
  - how do I automate content creation with AI research
  - set up marketing pipeline for auto-generated videos
  - create content from keyword to video automatically
  - use Claude and OpenAI for content automation
  - build automated marketing content workflow
  - generate videos from blog posts using Remotion
  - crawl news sources and create AI content
  - automate social media content pipeline
---

# Marketing Pipeline Share - AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

**marketing-pipeline-share** is a TypeScript-based automated content pipeline that transforms a single keyword into complete content packages including research, written content, and rendered videos. The system crawls real-time data from sources like TechCrunch, a16z, Twitter/X, and LinkedIn, then uses Claude 3 or OpenAI to generate content in multiple formats and languages, finally rendering videos using Remotion.

**Core workflow:**
1. **Auto Research** - Crawls latest news/data from major tech sources
2. **AI Content Generation** - Creates articles in various formats (toplist, POV, case study, how-to)
3. **Multi-language Output** - Generates English and Vietnamese versions
4. **Video Rendering** - Automatically creates infographics and short-form videos

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

### Environment Configuration

Create a `.env.local` file in the root directory:

```bash
# AI API Keys
OPENAI_API_KEY=your_openai_key_here
ANTHROPIC_API_KEY=your_claude_key_here

# Research/Crawling APIs
RAPIDAPI_KEY=your_rapidapi_key_here

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Video Rendering
REMOTION_LICENSE_KEY=your_remotion_license_key

# Application
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

### Running the Application

```bash
# Development mode
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Render videos (Remotion)
npm run remotion
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app router pages
│   ├── components/       # React components
│   ├── lib/              # Core libraries
│   │   ├── ai/           # AI integration (Claude, OpenAI)
│   │   ├── crawlers/     # News/data crawling modules
│   │   ├── content/      # Content generation logic
│   │   └── video/        # Remotion video rendering
│   ├── types/            # TypeScript type definitions
│   └── utils/            # Utility functions
├── remotion/             # Remotion video templates
├── public/               # Static assets
└── package.json
```

## Core APIs & Usage

### 1. Content Research Module

```typescript
import { ResearchCrawler } from '@/lib/crawlers/research-crawler';

// Initialize crawler
const crawler = new ResearchCrawler({
  sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
  timeRange: '24h',
  apiKey: process.env.RAPIDAPI_KEY
});

// Crawl data for a keyword
async function researchTopic(keyword: string) {
  const results = await crawler.crawl({
    keyword,
    maxResults: 20,
    language: 'en'
  });
  
  return {
    articles: results.articles,
    insights: results.insights,
    dataPoints: results.statistics
  };
}

// Usage
const research = await researchTopic('AI automation');
console.log(research.insights);
```

### 2. AI Content Generation

```typescript
import { ContentGenerator } from '@/lib/ai/content-generator';
import { Anthropic } from '@anthropic-ai/sdk';
import OpenAI from 'openai';

// Initialize AI clients
const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY
});

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY
});

// Generate content using Claude
async function generateWithClaude(
  research: any,
  format: 'toplist' | 'pov' | 'case-study' | 'how-to'
) {
  const generator = new ContentGenerator(anthropic);
  
  const content = await generator.generate({
    model: 'claude-3-5-sonnet-20241022',
    research,
    format,
    tone: 'professional', // 'friendly', 'humorous'
    language: 'en', // 'vi' for Vietnamese
    wordCount: 1500
  });
  
  return content;
}

// Generate bilingual content
async function generateBilingual(research: any) {
  const [englishContent, vietnameseContent] = await Promise.all([
    generateWithClaude(research, 'toplist'),
    generateWithClaude(
      { ...research, language: 'vi' },
      'toplist'
    )
  ]);
  
  return {
    en: englishContent,
    vi: vietnameseContent
  };
}
```

### 3. Content Format Templates

```typescript
import { formatContent } from '@/lib/content/formatter';

// Toplist format
const toplistContent = await formatContent({
  type: 'toplist',
  data: research,
  structure: {
    introduction: true,
    items: 10,
    conclusion: true,
    cta: 'Learn more about AI automation'
  }
});

// POV (Point of View) format
const povContent = await formatContent({
  type: 'pov',
  data: research,
  perspective: 'industry-expert',
  includePersonalAnecdotes: true
});

// Case Study format
const caseStudyContent = await formatContent({
  type: 'case-study',
  data: research,
  structure: {
    challenge: true,
    solution: true,
    results: true,
    metrics: true
  }
});

// How-to format
const howToContent = await formatContent({
  type: 'how-to',
  data: research,
  steps: 7,
  difficulty: 'intermediate',
  includeScreenshots: false
});
```

### 4. Video Rendering with Remotion

```typescript
import { renderVideo } from '@/lib/video/renderer';
import { VideoComposition } from '@/remotion/compositions';

// Render video from content
async function createContentVideo(content: any) {
  const videoConfig = {
    composition: 'ContentVideo',
    inputProps: {
      title: content.title,
      highlights: content.keyPoints,
      style: 'modern', // 'minimal', 'energetic'
      duration: 60, // seconds
      format: 'vertical' // 'horizontal', 'square'
    },
    outputLocation: `out/${content.slug}.mp4`
  };
  
  const result = await renderVideo(videoConfig);
  return result.outputPath;
}

// Platform-specific rendering
async function renderForPlatforms(content: any) {
  const platforms = {
    reels: { width: 1080, height: 1920, duration: 60 },
    tiktok: { width: 1080, height: 1920, duration: 60 },
    youtube: { width: 1920, height: 1080, duration: 120 }
  };
  
  const videos = await Promise.all(
    Object.entries(platforms).map(([platform, specs]) =>
      renderVideo({
        composition: 'ContentVideo',
        inputProps: {
          ...content,
          ...specs
        },
        outputLocation: `out/${content.slug}-${platform}.mp4`
      })
    )
  );
  
  return videos;
}
```

## Complete Workflow Example

```typescript
import { Pipeline } from '@/lib/pipeline';

// Full pipeline from keyword to video
async function runContentPipeline(keyword: string) {
  const pipeline = new Pipeline({
    aiProvider: 'claude', // or 'openai'
    languages: ['en', 'vi'],
    formats: ['toplist', 'how-to'],
    generateVideo: true
  });
  
  try {
    // Step 1: Research
    console.log('🔍 Researching topic...');
    const research = await pipeline.research(keyword);
    
    // Step 2: Generate content
    console.log('✍️ Generating content...');
    const content = await pipeline.generateContent(research, {
      format: 'toplist',
      tone: 'professional',
      includeDataPoints: true,
      includeCitations: true
    });
    
    // Step 3: Create bilingual versions
    console.log('🌍 Creating translations...');
    const translations = await pipeline.translate(content);
    
    // Step 4: Render videos
    console.log('🎬 Rendering videos...');
    const videos = await pipeline.renderVideos(content, {
      platforms: ['reels', 'tiktok', 'youtube'],
      style: 'modern'
    });
    
    // Step 5: Save to database
    await pipeline.save({
      content,
      translations,
      videos,
      metadata: {
        keyword,
        createdAt: new Date(),
        status: 'ready'
      }
    });
    
    return {
      success: true,
      content,
      videos: videos.map(v => v.outputPath)
    };
    
  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}

// Execute pipeline
const result = await runContentPipeline('AI content automation');
console.log('✅ Pipeline complete!', result);
```

## Configuration Patterns

### Custom AI Prompts

```typescript
// lib/ai/prompts.ts
export const customPrompts = {
  research: {
    system: `You are an expert content researcher. Analyze the provided data 
             and extract key insights, trends, and data points.`,
    user: (keyword: string) => `Research topic: ${keyword}
                                Find latest trends, statistics, and expert opinions.`
  },
  
  content: {
    toplist: `Create a comprehensive toplist article with:
             - Engaging introduction
             - 10 well-researched items
             - Data-backed claims
             - Actionable conclusion`,
    
    pov: `Write from an industry expert perspective:
          - Personal insights and experience
          - Contrarian or unique viewpoints
          - Real-world examples
          - Thought-provoking conclusion`
  }
};

// Use custom prompts
const content = await generator.generate({
  prompt: customPrompts.content.toplist,
  research,
  format: 'toplist'
});
```

### Video Templates

```typescript
// remotion/templates/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig } from 'remotion';

export const ContentVideo: React.FC<{
  title: string;
  highlights: string[];
  style: 'modern' | 'minimal';
}> = ({ title, highlights, style }) => {
  const frame = useCurrentFrame();
  const { fps, width, height } = useVideoConfig();
  
  const opacity = Math.min(1, frame / 30);
  
  return (
    <AbsoluteFill
      style={{
        backgroundColor: style === 'modern' ? '#0A0E27' : '#FFFFFF',
        fontFamily: 'Inter'
      }}
    >
      <div style={{ opacity, padding: 60 }}>
        <h1 style={{ fontSize: 72, fontWeight: 'bold', marginBottom: 40 }}>
          {title}
        </h1>
        {highlights.map((highlight, i) => (
          <div
            key={i}
            style={{
              fontSize: 36,
              marginBottom: 20,
              opacity: Math.min(1, (frame - i * 15) / 30)
            }}
          >
            ✓ {highlight}
          </div>
        ))}
      </div>
    </AbsoluteFill>
  );
};
```

## API Endpoints (Next.js)

```typescript
// app/api/content/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline';

export async function POST(request: NextRequest) {
  try {
    const { keyword, format, languages } = await request.json();
    
    if (!keyword) {
      return NextResponse.json(
        { error: 'Keyword is required' },
        { status: 400 }
      );
    }
    
    const result = await runContentPipeline(keyword);
    
    return NextResponse.json(result);
    
  } catch (error) {
    console.error('API Error:', error);
    return NextResponse.json(
      { error: 'Internal server error' },
      { status: 500 }
    );
  }
}

// Usage from frontend
const response = await fetch('/api/content/generate', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    keyword: 'AI automation',
    format: 'toplist',
    languages: ['en', 'vi']
  })
});

const data = await response.json();
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement rate limiting and retries
import { retry } from '@/utils/retry';

const contentWithRetry = await retry(
  () => generateWithClaude(research, 'toplist'),
  {
    maxAttempts: 3,
    delay: 1000,
    backoff: 'exponential'
  }
);
```

### Video Rendering Issues

```bash
# Check Remotion installation
npx remotion --version

# Clear Remotion cache
rm -rf node_modules/.cache/remotion

# Test video rendering
npx remotion preview remotion/index.ts
```

### Memory Issues with Large Content

```typescript
// Process content in batches
async function batchProcess(keywords: string[]) {
  const batchSize = 5;
  const results = [];
  
  for (let i = 0; i < keywords.length; i += batchSize) {
    const batch = keywords.slice(i, i + batchSize);
    const batchResults = await Promise.all(
      batch.map(kw => runContentPipeline(kw))
    );
    results.push(...batchResults);
    
    // Clear memory between batches
    if (global.gc) global.gc();
  }
  
  return results;
}
```

### Crawler Blocking

```typescript
// Add user agents and delays
const crawler = new ResearchCrawler({
  headers: {
    'User-Agent': 'Mozilla/5.0 (compatible; ContentBot/1.0)'
  },
  requestDelay: 2000, // 2 second delay between requests
  respectRobotsTxt: true
});
```

## Best Practices

1. **Always use environment variables** for API keys
2. **Implement caching** for research results to avoid redundant API calls
3. **Use batching** for multiple content generation to manage rate limits
4. **Monitor AI costs** by tracking token usage
5. **Version control video templates** for consistent branding
6. **Test prompts** on small datasets before full production runs
7. **Implement error tracking** (Sentry, LogRocket) for production use
