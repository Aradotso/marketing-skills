---
name: marketing-pipeline-automation
description: AI-powered content automation pipeline from research to video generation with auto-posting
triggers:
  - automate content creation with AI research
  - generate marketing content from keyword research
  - create automated content pipeline with video
  - build AI content workflow from research to video
  - setup automated marketing content generation
  - create content automation with Claude and OpenAI
  - generate social media content automatically
  - build content research and video pipeline
---

# Marketing Pipeline Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

Master the Ultimate AI Content Pipeline - an automated system that transforms keywords into complete content pieces with research, scriptwriting, and video generation capabilities using Claude 3, OpenAI, and Remotion.

## What This Project Does

This TypeScript-based system provides a complete content automation pipeline:

- **Auto-Research**: Crawls and analyzes real-time data from TechCrunch, a16z, X (Twitter), and LinkedIn
- **AI Content Generation**: Creates content in multiple formats (toplist, POV, case studies, how-to) using Claude/OpenAI
- **Multi-Language Support**: Generates content in both English and Vietnamese with customizable tone
- **Video Rendering**: Automatically creates infographics and short videos using Remotion
- **Auto-Posting**: Publishes content directly to social platforms

## Installation

### Prerequisites

```bash
# Required Node.js version
node --version  # Should be 18.x or higher
npm --version   # Should be 8.x or higher
```

### Setup Steps

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
yarn install
```

### Environment Configuration

Create a `.env.local` file in the project root:

```bash
# AI Provider APIs
ANTHROPIC_API_KEY=your_claude_api_key_here
OPENAI_API_KEY=your_openai_api_key_here

# Research APIs (RapidAPI)
RAPIDAPI_KEY=your_rapidapi_key_here

# Social Media Integration
FACEBOOK_ACCESS_TOKEN=your_facebook_token_here
TWITTER_API_KEY=your_twitter_key_here
TWITTER_API_SECRET=your_twitter_secret_here

# Remotion Configuration
REMOTION_AWS_ACCESS_KEY_ID=your_aws_access_key_here
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret_key_here

# Next.js Configuration
NEXT_PUBLIC_API_URL=http://localhost:3000
```

### Development Server

```bash
# Start the development server
npm run dev
# or
yarn dev

# Access at http://localhost:3000
```

### Production Build

```bash
# Build for production
npm run build

# Start production server
npm start
```

## Core Architecture

### Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/             
│   │   ├── ai/          # AI integration modules
│   │   ├── research/    # Content research scrapers
│   │   ├── video/       # Remotion video generation
│   │   └── social/      # Social media posting
│   ├── types/           # TypeScript type definitions
│   └── utils/           # Utility functions
├── public/              # Static assets
└── remotion/            # Remotion video templates
```

## Key Features & Usage

### 1. Content Research Module

```typescript
// src/lib/research/scraper.ts
import { ResearchScraper } from '@/lib/research/scraper';

interface ResearchResult {
  title: string;
  url: string;
  content: string;
  publishedAt: Date;
  source: string;
}

// Initialize scraper
const scraper = new ResearchScraper({
  apiKey: process.env.RAPIDAPI_KEY,
  sources: ['techcrunch', 'a16z', 'twitter', 'linkedin']
});

// Perform research on a topic
async function researchTopic(keyword: string): Promise<ResearchResult[]> {
  const results = await scraper.search({
    query: keyword,
    timeRange: '24h',
    limit: 20,
    language: 'en'
  });
  
  return results.filter(r => r.relevanceScore > 0.7);
}

// Usage
const articles = await researchTopic('AI marketing automation');
```

### 2. AI Content Generation

```typescript
// src/lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import { OpenAI } from 'openai';

interface ContentConfig {
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  tone: 'expert' | 'friendly' | 'humorous';
  language: 'en' | 'vi';
  targetAudience: string;
}

class ContentGenerator {
  private claude: Anthropic;
  private openai: OpenAI;
  
  constructor() {
    this.claude = new Anthropic({
      apiKey: process.env.ANTHROPIC_API_KEY
    });
    
    this.openai = new OpenAI({
      apiKey: process.env.OPENAI_API_KEY
    });
  }
  
  async generateContent(
    research: ResearchResult[],
    config: ContentConfig
  ): Promise<string> {
    const prompt = this.buildPrompt(research, config);
    
    // Use Claude for content generation
    const message = await this.claude.messages.create({
      model: 'claude-3-sonnet-20240229',
      max_tokens: 4096,
      messages: [{
        role: 'user',
        content: prompt
      }]
    });
    
    return message.content[0].type === 'text' 
      ? message.content[0].text 
      : '';
  }
  
  private buildPrompt(
    research: ResearchResult[],
    config: ContentConfig
  ): string {
    const researchData = research.map(r => 
      `Title: ${r.title}\nSource: ${r.source}\nContent: ${r.content}`
    ).join('\n\n');
    
    return `
Create a ${config.format} article in ${config.language} with a ${config.tone} tone.
Target audience: ${config.targetAudience}

Research data:
${researchData}

Requirements:
- Use data-backed insights
- Include specific examples and statistics
- Make it engaging and actionable
- Optimize for social media sharing
`;
  }
}

// Usage
const generator = new ContentGenerator();
const content = await generator.generateContent(articles, {
  format: 'how-to',
  tone: 'friendly',
  language: 'en',
  targetAudience: 'marketing professionals'
});
```

### 3. Multi-Language Content Generation

```typescript
// src/lib/ai/multilingual.ts
interface BilingualContent {
  en: string;
  vi: string;
  metadata: {
    keywords: string[];
    hashtags: string[];
    suggestedTitle: string;
  };
}

async function generateBilingualContent(
  keyword: string
): Promise<BilingualContent> {
  const research = await researchTopic(keyword);
  const generator = new ContentGenerator();
  
  // Generate English version
  const englishContent = await generator.generateContent(research, {
    format: 'toplist',
    tone: 'expert',
    language: 'en',
    targetAudience: 'global marketers'
  });
  
  // Generate Vietnamese version
  const vietnameseContent = await generator.generateContent(research, {
    format: 'toplist',
    tone: 'expert',
    language: 'vi',
    targetAudience: 'Vietnamese marketers'
  });
  
  return {
    en: englishContent,
    vi: vietnameseContent,
    metadata: {
      keywords: [keyword, ...extractKeywords(research)],
      hashtags: generateHashtags(keyword),
      suggestedTitle: `Ultimate Guide to ${keyword}`
    }
  };
}
```

### 4. Video Generation with Remotion

```typescript
// src/lib/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import { webpack } from '@remotion/bundler';
import path from 'path';

interface VideoConfig {
  content: string;
  style: 'infographic' | 'slideshow' | 'animated';
  duration: number;
  aspectRatio: '16:9' | '9:16' | '1:1';
  platform: 'reels' | 'tiktok' | 'youtube-shorts';
}

class VideoRenderer {
  async renderVideo(config: VideoConfig): Promise<string> {
    // Bundle the Remotion project
    const bundleLocation = await bundle({
      entryPoint: path.resolve('./remotion/index.ts'),
      webpackOverride: (config) => config,
    });
    
    // Get composition
    const composition = await selectComposition({
      serveUrl: bundleLocation,
      id: config.style,
      inputProps: {
        content: config.content,
        duration: config.duration
      }
    });
    
    // Render video
    const outputPath = path.resolve(`./output/video-${Date.now()}.mp4`);
    
    await renderMedia({
      composition,
      serveUrl: bundleLocation,
      codec: 'h264',
      outputLocation: outputPath,
      inputProps: {
        content: config.content,
        aspectRatio: config.aspectRatio
      }
    });
    
    return outputPath;
  }
}

// Remotion composition example
// remotion/Infographic.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig } from 'remotion';

export const Infographic: React.FC<{
  content: string;
  aspectRatio: string;
}> = ({ content, aspectRatio }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  
  const opacity = Math.min(1, frame / (fps * 0.5));
  
  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a1a' }}>
      <div style={{ 
        opacity,
        padding: '40px',
        color: 'white',
        fontSize: '32px'
      }}>
        {content}
      </div>
    </AbsoluteFill>
  );
};
```

### 5. Social Media Auto-Posting

```typescript
// src/lib/social/publisher.ts
import { FacebookAPI } from './facebook';
import { TwitterAPI } from './twitter';

interface PostConfig {
  platforms: ('facebook' | 'twitter' | 'linkedin')[];
  content: string;
  mediaUrl?: string;
  scheduledTime?: Date;
}

class SocialPublisher {
  private facebook: FacebookAPI;
  private twitter: TwitterAPI;
  
  constructor() {
    this.facebook = new FacebookAPI({
      accessToken: process.env.FACEBOOK_ACCESS_TOKEN
    });
    
    this.twitter = new TwitterAPI({
      apiKey: process.env.TWITTER_API_KEY,
      apiSecret: process.env.TWITTER_API_SECRET
    });
  }
  
  async publish(config: PostConfig): Promise<void> {
    const tasks = config.platforms.map(async (platform) => {
      switch (platform) {
        case 'facebook':
          return this.facebook.post({
            message: config.content,
            mediaUrl: config.mediaUrl
          });
        case 'twitter':
          return this.twitter.tweet({
            text: config.content,
            media: config.mediaUrl
          });
        default:
          throw new Error(`Unsupported platform: ${platform}`);
      }
    });
    
    await Promise.all(tasks);
  }
}

// Usage
const publisher = new SocialPublisher();
await publisher.publish({
  platforms: ['facebook', 'twitter'],
  content: 'Check out our latest AI marketing insights!',
  mediaUrl: videoPath
});
```

## Complete Workflow Example

```typescript
// src/workflows/complete-pipeline.ts
import { ResearchScraper } from '@/lib/research/scraper';
import { ContentGenerator } from '@/lib/ai/content-generator';
import { VideoRenderer } from '@/lib/video/renderer';
import { SocialPublisher } from '@/lib/social/publisher';

interface PipelineConfig {
  keyword: string;
  contentFormat: 'toplist' | 'pov' | 'case-study' | 'how-to';
  languages: ('en' | 'vi')[];
  generateVideo: boolean;
  platforms: ('facebook' | 'twitter' | 'linkedin')[];
}

async function runCompleteContentPipeline(
  config: PipelineConfig
): Promise<void> {
  console.log(`🚀 Starting pipeline for keyword: ${config.keyword}`);
  
  // Step 1: Research
  console.log('📡 Performing research...');
  const scraper = new ResearchScraper({
    apiKey: process.env.RAPIDAPI_KEY,
    sources: ['techcrunch', 'a16z', 'twitter']
  });
  
  const research = await scraper.search({
    query: config.keyword,
    timeRange: '24h',
    limit: 20
  });
  
  // Step 2: Generate Content
  console.log('🧠 Generating content...');
  const generator = new ContentGenerator();
  
  const contents = await Promise.all(
    config.languages.map(lang =>
      generator.generateContent(research, {
        format: config.contentFormat,
        tone: 'expert',
        language: lang,
        targetAudience: 'marketing professionals'
      })
    )
  );
  
  // Step 3: Generate Video (if enabled)
  let videoPath: string | undefined;
  
  if (config.generateVideo) {
    console.log('🎬 Rendering video...');
    const renderer = new VideoRenderer();
    
    videoPath = await renderer.renderVideo({
      content: contents[0].substring(0, 500),
      style: 'infographic',
      duration: 30,
      aspectRatio: '9:16',
      platform: 'reels'
    });
  }
  
  // Step 4: Publish
  console.log('📤 Publishing to social media...');
  const publisher = new SocialPublisher();
  
  await publisher.publish({
    platforms: config.platforms,
    content: contents[0],
    mediaUrl: videoPath
  });
  
  console.log('✅ Pipeline completed successfully!');
}

// Execute pipeline
runCompleteContentPipeline({
  keyword: 'AI marketing automation 2024',
  contentFormat: 'how-to',
  languages: ['en', 'vi'],
  generateVideo: true,
  platforms: ['facebook', 'twitter']
});
```

## API Routes (Next.js)

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runCompleteContentPipeline } from '@/workflows/complete-pipeline';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    
    const { keyword, contentFormat, languages, generateVideo, platforms } = body;
    
    // Validate input
    if (!keyword || !contentFormat) {
      return NextResponse.json(
        { error: 'Missing required fields' },
        { status: 400 }
      );
    }
    
    // Run pipeline
    await runCompleteContentPipeline({
      keyword,
      contentFormat,
      languages: languages || ['en'],
      generateVideo: generateVideo || false,
      platforms: platforms || ['facebook']
    });
    
    return NextResponse.json({ 
      success: true,
      message: 'Content pipeline executed successfully'
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

## Common Patterns

### Batch Content Generation

```typescript
// Generate multiple content pieces in parallel
async function batchGenerate(keywords: string[]): Promise<void> {
  const batches = keywords.map(keyword => 
    runCompleteContentPipeline({
      keyword,
      contentFormat: 'toplist',
      languages: ['en', 'vi'],
      generateVideo: true,
      platforms: ['facebook', 'twitter']
    })
  );
  
  await Promise.all(batches);
}
```

### Scheduled Content Generation

```typescript
// Use node-cron for scheduling
import cron from 'node-cron';

// Run daily at 9 AM
cron.schedule('0 9 * * *', async () => {
  const todayKeywords = await fetchTrendingKeywords();
  await batchGenerate(todayKeywords);
});
```

### Content Personalization

```typescript
// Customize content for different audience segments
const segments = [
  { audience: 'beginners', tone: 'friendly' },
  { audience: 'experts', tone: 'technical' },
  { audience: 'executives', tone: 'business-focused' }
];

for (const segment of segments) {
  await generator.generateContent(research, {
    format: 'how-to',
    tone: segment.tone,
    language: 'en',
    targetAudience: segment.audience
  });
}
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement rate limiting with retry logic
import pRetry from 'p-retry';

async function generateWithRetry(config: ContentConfig) {
  return pRetry(
    () => generator.generateContent(research, config),
    {
      retries: 3,
      onFailedAttempt: error => {
        console.log(`Attempt ${error.attemptNumber} failed. Retrying...`);
      }
    }
  );
}
```

### Video Rendering Issues

```typescript
// Check Remotion dependencies
// Ensure ffmpeg is installed
import { execSync } from 'child_process';

try {
  execSync('ffmpeg -version');
} catch (error) {
  console.error('ffmpeg not found. Install: brew install ffmpeg');
}
```

### Memory Management for Large Batches

```typescript
// Process in chunks to avoid memory issues
import pLimit from 'p-limit';

const limit = pLimit(3); // Max 3 concurrent operations

const tasks = keywords.map(keyword =>
  limit(() => runCompleteContentPipeline({ 
    keyword,
    /* ... config ... */
  }))
);

await Promise.all(tasks);
```

### Environment Variable Validation

```typescript
// src/lib/config/validate.ts
function validateConfig(): void {
  const required = [
    'ANTHROPIC_API_KEY',
    'OPENAI_API_KEY',
    'RAPIDAPI_KEY'
  ];
  
  const missing = required.filter(key => !process.env[key]);
  
  if (missing.length > 0) {
    throw new Error(
      `Missing required environment variables: ${missing.join(', ')}\n` +
      'Please check your .env.local file'
    );
  }
}

// Call during initialization
validateConfig();
```

## Performance Optimization

```typescript
// Cache research results to reduce API calls
import NodeCache from 'node-cache';

const cache = new NodeCache({ stdTTL: 3600 }); // 1 hour TTL

async function cachedResearch(keyword: string): Promise<ResearchResult[]> {
  const cacheKey = `research:${keyword}`;
  const cached = cache.get<ResearchResult[]>(cacheKey);
  
  if (cached) {
    console.log('Using cached research data');
    return cached;
  }
  
  const results = await researchTopic(keyword);
  cache.set(cacheKey, results);
  
  return results;
}
```

This skill enables AI coding agents to help developers build automated content marketing pipelines with AI-powered research, multi-format content generation, video rendering, and social media publishing capabilities.
