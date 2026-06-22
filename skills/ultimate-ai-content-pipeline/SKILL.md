```markdown
---
name: ultimate-ai-content-pipeline
description: Automated content creation pipeline from research to script generation and video rendering using AI
triggers:
  - create automated content pipeline with AI
  - generate video content from research
  - automate content workflow from research to video
  - build AI-powered marketing content system
  - create content with Claude and OpenAI integration
  - set up automated video rendering pipeline
  - generate multilingual content with AI
  - automate content creation from trending news
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the Ultimate AI Content Pipeline, a comprehensive TypeScript-based system that automates the entire content creation workflow: from researching trending topics, generating scripts in multiple formats and languages, to rendering videos automatically.

## Overview

Ultimate AI Content Pipeline is an all-in-one content automation system that:

- **Auto-scans research** from sources like TechCrunch, a16z, Twitter/X, LinkedIn for the latest trends
- **Generates content** in multiple formats (toplist, POV, case study, how-to) using Claude/OpenAI
- **Supports multilingual output** (English & Vietnamese) with customizable tone
- **Renders videos automatically** using Remotion for social media platforms
- **Provides Next.js interface** for easy content management and scheduling

## Installation

### Prerequisites

- Node.js 18+ and npm/yarn/pnpm
- API keys for: OpenAI, Anthropic Claude, RapidAPI (for web scraping)

### Setup

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

### Environment Configuration

Create a `.env` file with the following variables:

```bash
# AI Service APIs
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# Research/Scraping APIs
RAPIDAPI_KEY=your_rapidapi_key

# Application Configuration
NEXT_PUBLIC_APP_URL=http://localhost:3000
DATABASE_URL=your_database_url

# Video Rendering (optional)
REMOTION_LICENSE_KEY=your_remotion_key
```

### Run the Application

```bash
# Development mode
npm run dev

# Production build
npm run build
npm start

# Remotion video rendering server
npm run remotion:dev
```

## Core Architecture

The pipeline consists of several key modules:

### 1. Research Module

Automatically scrapes and analyzes content from configured sources:

```typescript
// src/lib/research/scraper.ts
import { ResearchService } from '@/lib/research';

const researchService = new ResearchService({
  rapidApiKey: process.env.RAPIDAPI_KEY,
});

// Fetch trending content by keyword
const research = await researchService.fetchTrendingContent({
  keyword: 'AI automation',
  sources: ['techcrunch', 'twitter', 'linkedin'],
  timeframe: '24h',
  limit: 10,
});

console.log(research.insights);
console.log(research.dataPoints);
```

### 2. Content Generation Module

Generate content in various formats using AI:

```typescript
// src/lib/content/generator.ts
import { ContentGenerator } from '@/lib/content';

const generator = new ContentGenerator({
  provider: 'claude', // or 'openai'
  apiKey: process.env.ANTHROPIC_API_KEY,
});

// Generate article from research
const content = await generator.generateArticle({
  format: 'toplist', // 'pov', 'case-study', 'how-to'
  topic: 'AI Content Automation Tools',
  research: researchData,
  language: 'vi', // or 'en'
  tone: 'professional', // 'friendly', 'humorous'
  targetAudience: 'marketers',
});

console.log(content.title);
console.log(content.body);
console.log(content.metadata);
```

### 3. Multilingual Support

Generate content in multiple languages simultaneously:

```typescript
// src/lib/content/multilingual.ts
import { MultilingualGenerator } from '@/lib/content';

const mlGenerator = new MultilingualGenerator({
  openaiKey: process.env.OPENAI_API_KEY,
  claudeKey: process.env.ANTHROPIC_API_KEY,
});

// Generate in both languages
const multiContent = await mlGenerator.generateMultilingual({
  topic: 'Marketing Automation Trends 2026',
  languages: ['en', 'vi'],
  format: 'case-study',
  research: researchData,
});

console.log(multiContent.en.title);
console.log(multiContent.vi.title);
```

### 4. Video Rendering Module

Automatically render videos from content using Remotion:

```typescript
// src/lib/video/renderer.ts
import { VideoRenderer } from '@/lib/video';

const renderer = new VideoRenderer({
  remotionLicenseKey: process.env.REMOTION_LICENSE_KEY,
});

// Render video from article content
const video = await renderer.renderFromContent({
  content: articleContent,
  template: 'infographic', // 'talking-head', 'slideshow'
  platform: 'reels', // 'tiktok', 'shorts', 'linkedin'
  aspectRatio: '9:16',
  duration: 60, // seconds
});

console.log(video.url);
console.log(video.metadata);
```

## API Routes

The Next.js application provides several API endpoints:

### Research Endpoint

```typescript
// app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { ResearchService } from '@/lib/research';

export async function POST(req: NextRequest) {
  const { keyword, sources, timeframe } = await req.json();
  
  const researchService = new ResearchService({
    rapidApiKey: process.env.RAPIDAPI_KEY,
  });
  
  const results = await researchService.fetchTrendingContent({
    keyword,
    sources,
    timeframe,
  });
  
  return NextResponse.json(results);
}
```

### Content Generation Endpoint

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { ContentGenerator } from '@/lib/content';

export async function POST(req: NextRequest) {
  const { format, topic, language, tone, research } = await req.json();
  
  const generator = new ContentGenerator({
    provider: 'claude',
    apiKey: process.env.ANTHROPIC_API_KEY,
  });
  
  const content = await generator.generateArticle({
    format,
    topic,
    research,
    language,
    tone,
  });
  
  return NextResponse.json(content);
}
```

### Video Rendering Endpoint

```typescript
// app/api/render/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { VideoRenderer } from '@/lib/video';

export async function POST(req: NextRequest) {
  const { content, template, platform } = await req.json();
  
  const renderer = new VideoRenderer({
    remotionLicenseKey: process.env.REMOTION_LICENSE_KEY,
  });
  
  const video = await renderer.renderFromContent({
    content,
    template,
    platform,
  });
  
  return NextResponse.json(video);
}
```

## Common Patterns

### Full Pipeline Automation

```typescript
// src/workflows/full-pipeline.ts
import { ResearchService } from '@/lib/research';
import { ContentGenerator } from '@/lib/content';
import { VideoRenderer } from '@/lib/video';

async function runFullPipeline(keyword: string) {
  // Step 1: Research
  const research = new ResearchService({
    rapidApiKey: process.env.RAPIDAPI_KEY,
  });
  
  const researchData = await research.fetchTrendingContent({
    keyword,
    sources: ['techcrunch', 'twitter'],
    timeframe: '24h',
  });
  
  // Step 2: Generate Content
  const generator = new ContentGenerator({
    provider: 'claude',
    apiKey: process.env.ANTHROPIC_API_KEY,
  });
  
  const content = await generator.generateArticle({
    format: 'toplist',
    topic: keyword,
    research: researchData,
    language: 'vi',
    tone: 'professional',
  });
  
  // Step 3: Render Video
  const renderer = new VideoRenderer({
    remotionLicenseKey: process.env.REMOTION_LICENSE_KEY,
  });
  
  const video = await renderer.renderFromContent({
    content,
    template: 'infographic',
    platform: 'reels',
  });
  
  return {
    research: researchData,
    content,
    video,
  };
}

// Execute pipeline
const result = await runFullPipeline('AI Marketing Tools');
```

### Scheduled Content Generation

```typescript
// src/workflows/scheduled.ts
import cron from 'node-cron';

// Run pipeline daily at 9 AM
cron.schedule('0 9 * * *', async () => {
  const keywords = ['AI trends', 'Marketing automation', 'Content strategy'];
  
  for (const keyword of keywords) {
    try {
      const result = await runFullPipeline(keyword);
      
      // Save to database or publish
      await saveToDatabase(result);
      
      console.log(`Generated content for: ${keyword}`);
    } catch (error) {
      console.error(`Failed for ${keyword}:`, error);
    }
  }
});
```

### Custom Template Configuration

```typescript
// src/lib/content/templates.ts
export interface ContentTemplate {
  name: string;
  structure: string[];
  promptModifiers: string[];
}

const templates: Record<string, ContentTemplate> = {
  toplist: {
    name: 'Top List',
    structure: ['intro', 'items', 'conclusion'],
    promptModifiers: ['numbered format', 'comparative analysis'],
  },
  pov: {
    name: 'Point of View',
    structure: ['hook', 'argument', 'evidence', 'conclusion'],
    promptModifiers: ['personal perspective', 'strong opinion'],
  },
  'case-study': {
    name: 'Case Study',
    structure: ['background', 'challenge', 'solution', 'results'],
    promptModifiers: ['data-driven', 'specific metrics'],
  },
  'how-to': {
    name: 'How-to Guide',
    structure: ['intro', 'steps', 'tips', 'conclusion'],
    promptModifiers: ['step-by-step', 'actionable'],
  },
};
```

## Configuration

### Content Settings

```typescript
// config/content.config.ts
export const contentConfig = {
  ai: {
    defaultProvider: 'claude', // 'openai' or 'claude'
    model: {
      claude: 'claude-3-opus-20240229',
      openai: 'gpt-4-turbo-preview',
    },
    maxTokens: 4000,
    temperature: 0.7,
  },
  
  languages: {
    supported: ['en', 'vi'],
    default: 'vi',
  },
  
  formats: ['toplist', 'pov', 'case-study', 'how-to'],
  
  tones: ['professional', 'friendly', 'humorous', 'expert'],
};
```

### Video Settings

```typescript
// config/video.config.ts
export const videoConfig = {
  platforms: {
    reels: {
      aspectRatio: '9:16',
      maxDuration: 90,
      resolution: { width: 1080, height: 1920 },
    },
    tiktok: {
      aspectRatio: '9:16',
      maxDuration: 180,
      resolution: { width: 1080, height: 1920 },
    },
    shorts: {
      aspectRatio: '9:16',
      maxDuration: 60,
      resolution: { width: 1080, height: 1920 },
    },
    linkedin: {
      aspectRatio: '16:9',
      maxDuration: 600,
      resolution: { width: 1920, height: 1080 },
    },
  },
  
  templates: ['infographic', 'talking-head', 'slideshow'],
  
  fps: 30,
  codec: 'h264',
};
```

## Troubleshooting

### API Rate Limits

```typescript
// src/lib/utils/rate-limiter.ts
import pLimit from 'p-limit';

const limit = pLimit(5); // Max 5 concurrent requests

async function batchGenerate(topics: string[]) {
  const promises = topics.map(topic =>
    limit(async () => {
      try {
        return await runFullPipeline(topic);
      } catch (error) {
        if (error.message.includes('rate limit')) {
          // Wait and retry
          await new Promise(resolve => setTimeout(resolve, 5000));
          return await runFullPipeline(topic);
        }
        throw error;
      }
    })
  );
  
  return await Promise.all(promises);
}
```

### Error Handling

```typescript
// src/lib/utils/error-handler.ts
export class PipelineError extends Error {
  constructor(
    public stage: 'research' | 'generation' | 'rendering',
    message: string,
    public originalError?: Error
  ) {
    super(message);
    this.name = 'PipelineError';
  }
}

export async function safeExecutePipeline(keyword: string) {
  try {
    return await runFullPipeline(keyword);
  } catch (error) {
    if (error instanceof PipelineError) {
      console.error(`Pipeline failed at ${error.stage}:`, error.message);
      
      // Implement fallback or retry logic
      if (error.stage === 'research') {
        // Use cached data or alternative source
        return await runWithCachedResearch(keyword);
      }
    }
    throw error;
  }
}
```

### Video Rendering Issues

```typescript
// src/lib/video/troubleshooting.ts
export async function debugVideoRender(content: any) {
  const renderer = new VideoRenderer({
    remotionLicenseKey: process.env.REMOTION_LICENSE_KEY,
  });
  
  // Validate content structure
  const validation = renderer.validateContent(content);
  if (!validation.valid) {
    console.error('Content validation failed:', validation.errors);
    return null;
  }
  
  // Try rendering with lower quality first
  try {
    return await renderer.renderFromContent({
      content,
      template: 'infographic',
      platform: 'reels',
      quality: 'low', // Test with lower quality
    });
  } catch (error) {
    console.error('Rendering failed:', error);
    
    // Check Remotion logs
    const logs = await renderer.getLogs();
    console.log('Remotion logs:', logs);
    
    return null;
  }
}
```

## Advanced Usage

### Custom AI Prompts

```typescript
// src/lib/content/custom-prompts.ts
export function buildCustomPrompt(params: {
  format: string;
  topic: string;
  research: any;
  language: string;
  tone: string;
  customInstructions?: string;
}) {
  const basePrompt = `
    Create a ${params.format} article about "${params.topic}" in ${params.language}.
    Tone: ${params.tone}
    
    Research data:
    ${JSON.stringify(params.research)}
    
    ${params.customInstructions || ''}
  `;
  
  return basePrompt;
}
```

### Integration with CMS

```typescript
// src/integrations/cms.ts
import { WordPressClient } from '@/lib/integrations/wordpress';

export async function publishToWordPress(content: any) {
  const wp = new WordPressClient({
    url: process.env.WP_URL,
    username: process.env.WP_USERNAME,
    password: process.env.WP_APP_PASSWORD,
  });
  
  const post = await wp.createPost({
    title: content.title,
    content: content.body,
    status: 'draft',
    categories: content.metadata.categories,
    tags: content.metadata.tags,
  });
  
  return post;
}
```

This skill provides comprehensive coverage of the Ultimate AI Content Pipeline, enabling AI agents to effectively assist developers in building automated content creation workflows.
```
