---
name: marketing-pipeline-share-content-automation
description: Automate content creation from research to video generation using Claude/OpenAI AI with auto-posting capabilities
triggers:
  - how do I automate content creation with AI research
  - generate marketing content from keywords automatically
  - create video content using remotion and AI
  - set up automated content pipeline with Claude
  - crawl news and generate content posts
  - automate social media content with AI research
  - build end-to-end content automation system
  - create AI-powered marketing content pipeline
---

# Marketing Pipeline Share - Content Automation Skill

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

**Marketing Pipeline Share** is a complete TypeScript-based content automation system that transforms keywords into finished content through AI-powered research, scriptwriting, and video generation. It crawls real-time data from sources like TechCrunch, a16z, Twitter/X, and LinkedIn, then uses Claude 3 or OpenAI to generate multi-format content (articles, videos, infographics) with automatic posting capabilities.

**Key capabilities:**
- Auto-scan research from news sources (last 24h)
- AI-powered content generation (Claude/OpenAI) in multiple formats (Toplist, POV, Case Study, How-to)
- Bilingual support (English/Vietnamese) with customizable tone
- Automatic video rendering with Remotion
- Multi-platform optimization (Reels, TikTok, Shorts)

## Installation

### Prerequisites

```bash
# Node.js 18+ required
node --version

# pnpm recommended (or npm/yarn)
npm install -g pnpm
```

### Setup

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
pnpm install

# Copy environment template
cp .env.example .env
```

### Environment Configuration

Create `.env` file with required API keys:

```bash
# AI Models
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=postgresql://user:password@localhost:5432/content_pipeline

# Video Rendering
REMOTION_BUCKET=your_s3_bucket_name
AWS_ACCESS_KEY_ID=your_aws_key
AWS_SECRET_ACCESS_KEY=your_aws_secret

# Social Media Auto-Post (optional)
FACEBOOK_PAGE_TOKEN=your_fb_token
LINKEDIN_ACCESS_TOKEN=your_linkedin_token
```

### Run Development Server

```bash
# Start Next.js development server
pnpm dev

# Server runs on http://localhost:3000
```

## Core Architecture

### Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app router pages
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── research/    # Web crawling & data extraction
│   │   ├── content/     # Content generation logic
│   │   └── video/       # Remotion video rendering
│   ├── services/        # API services
│   └── utils/           # Helper functions
├── remotion/            # Video templates
└── public/              # Static assets
```

## Key Features & Usage

### 1. Research Auto-Scan

Crawl latest news from multiple sources:

```typescript
import { scanSources } from '@/lib/research/scanner';
import { extractInsights } from '@/lib/research/insights';

async function performResearch(keyword: string) {
  // Scan sources from last 24 hours
  const sources = ['techcrunch', 'a16z', 'twitter', 'linkedin'];
  
  const rawData = await scanSources({
    keyword,
    sources,
    timeRange: '24h'
  });
  
  // Extract actionable insights
  const insights = await extractInsights(rawData, {
    minRelevance: 0.7,
    includeStats: true,
    language: 'en'
  });
  
  return insights;
}

// Usage
const data = await performResearch('AI automation');
console.log(data.articles); // Crawled articles
console.log(data.trends);   // Identified trends
console.log(data.stats);    // Data points & metrics
```

### 2. AI Content Generation

Generate content using Claude or OpenAI:

```typescript
import { generateContent } from '@/lib/ai/content-generator';
import { ContentFormat, ToneStyle } from '@/types';

async function createBlogPost(keyword: string, insights: any) {
  const content = await generateContent({
    keyword,
    insights,
    
    // Format options
    format: ContentFormat.CASE_STUDY, // or TOPLIST, POV, HOW_TO
    
    // Language & tone
    language: 'vi', // 'en' or 'vi'
    tone: ToneStyle.PROFESSIONAL, // FRIENDLY, HUMOROUS, EXPERT
    
    // AI provider
    provider: 'claude', // or 'openai'
    model: 'claude-3-sonnet-20240229',
    
    // Content parameters
    wordCount: 1500,
    includeImages: true,
    includeSEO: true
  });
  
  return {
    title: content.title,
    body: content.body,
    meta: content.seoMetadata,
    images: content.suggestedImages,
    cta: content.callToAction
  };
}
```

### 3. Bilingual Content Generation

Create parallel English and Vietnamese versions:

```typescript
import { generateBilingual } from '@/lib/content/bilingual';

async function createBilingualPost(keyword: string) {
  const { english, vietnamese } = await generateBilingual({
    keyword,
    format: 'toplist',
    tone: 'friendly',
    
    // Shared context
    research: await performResearch(keyword),
    
    // Language-specific customization
    enConfig: {
      targetAudience: 'international marketers',
      useImperial: true
    },
    viConfig: {
      targetAudience: 'doanh nghiệp Việt Nam',
      useLocalExamples: true
    }
  });
  
  return { english, vietnamese };
}
```

### 4. Video Rendering with Remotion

Generate videos from content:

```typescript
import { renderVideo } from '@/lib/video/renderer';
import { VideoTemplate } from '@/remotion/templates';

async function createContentVideo(content: any) {
  const videoConfig = {
    template: VideoTemplate.INFOGRAPHIC, // or SHORT_FORM, TUTORIAL
    
    // Content data
    title: content.title,
    keyPoints: content.highlights,
    statistics: content.stats,
    
    // Visual settings
    aspectRatio: '9:16', // For Reels/TikTok/Shorts
    duration: 60, // seconds
    
    // Branding
    logo: '/assets/logo.png',
    colorScheme: {
      primary: '#3B82F6',
      secondary: '#10B981',
      background: '#1F2937'
    },
    
    // Audio
    voiceover: content.voiceoverScript,
    backgroundMusic: 'upbeat-corporate'
  };
  
  const video = await renderVideo(videoConfig);
  
  return {
    url: video.publicUrl,
    thumbnail: video.thumbnailUrl,
    metadata: video.metadata
  };
}
```

### 5. Complete Pipeline Example

Full workflow from keyword to published content:

```typescript
import { ContentPipeline } from '@/lib/pipeline';

async function runCompletePipeline() {
  const pipeline = new ContentPipeline({
    aiProvider: 'claude',
    autoPost: true,
    platforms: ['facebook', 'linkedin']
  });
  
  // Execute full pipeline
  const result = await pipeline.execute({
    keyword: 'AI marketing automation 2026',
    
    // Research phase
    research: {
      sources: ['techcrunch', 'twitter'],
      timeRange: '24h',
      minArticles: 10
    },
    
    // Content generation phase
    content: {
      formats: ['article', 'video'],
      languages: ['en', 'vi'],
      tone: 'expert'
    },
    
    // Video rendering phase
    video: {
      enabled: true,
      templates: ['infographic', 'short-form'],
      aspectRatios: ['9:16', '1:1']
    },
    
    // Publishing phase
    publish: {
      schedule: new Date('2026-06-01T10:00:00Z'),
      platforms: {
        facebook: {
          pageId: process.env.FB_PAGE_ID,
          includeVideo: true
        },
        linkedin: {
          companyId: process.env.LINKEDIN_COMPANY_ID,
          includeArticle: true
        }
      }
    }
  });
  
  return {
    researchData: result.research,
    generatedContent: result.content,
    renderedVideos: result.videos,
    publishedPosts: result.published
  };
}
```

## API Routes

### Content Generation Endpoint

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { generateContent } from '@/lib/ai/content-generator';

export async function POST(request: NextRequest) {
  const { keyword, format, language } = await request.json();
  
  try {
    const content = await generateContent({
      keyword,
      format,
      language,
      provider: 'claude'
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
// src/app/api/render-video/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { renderVideo } from '@/lib/video/renderer';

export async function POST(request: NextRequest) {
  const { contentId, template, aspectRatio } = await request.json();
  
  const video = await renderVideo({
    contentId,
    template,
    aspectRatio,
    outputBucket: process.env.REMOTION_BUCKET
  });
  
  return NextResponse.json({ videoUrl: video.publicUrl });
}
```

## Common Patterns

### Custom Content Templates

```typescript
// Define custom content template
interface CustomTemplate {
  structure: string[];
  sections: Record<string, string>;
  style: string;
}

const caseStudyTemplate: CustomTemplate = {
  structure: [
    'introduction',
    'challenge',
    'solution',
    'results',
    'conclusion'
  ],
  sections: {
    introduction: 'Hook with problem statement',
    challenge: 'Detail the specific challenges faced',
    solution: 'Explain implementation with steps',
    results: 'Present data-backed outcomes',
    conclusion: 'Key takeaways and CTA'
  },
  style: 'professional'
};

// Use template in generation
const content = await generateContent({
  keyword: 'marketing automation success',
  template: caseStudyTemplate,
  provider: 'claude'
});
```

### Batch Processing

```typescript
import { batchProcess } from '@/lib/utils/batch';

async function generateMultipleContent(keywords: string[]) {
  const results = await batchProcess(keywords, async (keyword) => {
    const research = await performResearch(keyword);
    const content = await createBlogPost(keyword, research);
    const video = await createContentVideo(content);
    
    return { keyword, content, video };
  }, {
    concurrency: 3, // Process 3 at a time
    retries: 2
  });
  
  return results;
}
```

### Scheduled Content Calendar

```typescript
import { scheduleContent } from '@/lib/scheduling/calendar';

async function setupContentCalendar(keywords: string[]) {
  const calendar = await scheduleContent({
    keywords,
    frequency: 'daily', // daily, weekly, monthly
    time: '10:00',
    timezone: 'Asia/Ho_Chi_Minh',
    
    platforms: ['facebook', 'linkedin'],
    
    // Auto-generate content on schedule
    autoGenerate: true,
    
    // Content variety
    formatRotation: ['article', 'toplist', 'case-study']
  });
  
  return calendar;
}
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement rate limiting and retry logic
import { withRetry } from '@/lib/utils/retry';

const content = await withRetry(
  () => generateContent({ keyword, provider: 'claude' }),
  {
    maxRetries: 3,
    delayMs: 2000,
    backoff: 'exponential',
    onRetry: (attempt) => console.log(`Retry attempt ${attempt}`)
  }
);
```

### Video Rendering Issues

```typescript
// Debug video rendering
import { validateVideoConfig } from '@/lib/video/validator';

const config = {
  template: 'infographic',
  aspectRatio: '9:16'
};

const validation = validateVideoConfig(config);
if (!validation.valid) {
  console.error('Invalid config:', validation.errors);
  // Fix issues before rendering
}
```

### Content Quality Check

```typescript
import { validateContent } from '@/lib/content/validator';

const content = await generateContent({ keyword });

const quality = await validateContent(content, {
  minWordCount: 1000,
  checkGrammar: true,
  checkPlagiarism: true,
  requireImages: true
});

if (!quality.passed) {
  console.log('Issues found:', quality.issues);
  // Regenerate or manually fix
}
```

### Memory Issues with Large Batches

```typescript
// Use streaming for large datasets
import { createReadStream } from 'fs';
import { pipeline } from 'stream/promises';

async function processLargeKeywordList(filePath: string) {
  const stream = createReadStream(filePath, { encoding: 'utf-8' });
  
  await pipeline(
    stream,
    async function* (source) {
      for await (const chunk of source) {
        const keywords = chunk.split('\n');
        for (const keyword of keywords) {
          yield await generateContent({ keyword });
        }
      }
    },
    async function (contents) {
      for await (const content of contents) {
        await saveContent(content);
      }
    }
  );
}
```

## Advanced Configuration

### Custom AI Model Settings

```typescript
// Fine-tune AI behavior
const customSettings = {
  temperature: 0.7, // Creativity (0-1)
  maxTokens: 2000,
  topP: 0.9,
  frequencyPenalty: 0.5,
  presencePenalty: 0.5,
  
  // Custom system prompts
  systemPrompt: `You are an expert marketing content creator...`,
  
  // Context management
  includeResearch: true,
  contextWindow: 8000
};

const content = await generateContent({
  keyword,
  provider: 'openai',
  modelSettings: customSettings
});
```

This skill enables AI coding agents to help developers build, customize, and troubleshoot complete content automation pipelines using the Marketing Pipeline Share system.
