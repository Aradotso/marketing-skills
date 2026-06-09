---
name: marketing-pipeline-share-ai-content
description: Automated AI content pipeline for research, scripting, posting, and video generation using Claude/OpenAI and Remotion
triggers:
  - how do I automate content creation with AI research
  - set up automated content pipeline with video generation
  - configure AI content workflow with Claude and OpenAI
  - build automated marketing content system
  - create content pipeline from research to video
  - implement AI-powered content automation
  - set up Remotion video generation for content
  - automate content research and social media posting
---

# Marketing Pipeline Share - AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to help developers build and customize an end-to-end automated content pipeline that handles research, scripting, posting, and video generation using AI (Claude 3, OpenAI) and Remotion.

## What This Project Does

Ultimate AI Content Pipeline is a TypeScript-based automation system that:

- **Auto-researches** trending topics by crawling news sources (TechCrunch, a16z, Twitter, LinkedIn)
- **Generates content** in multiple formats (Toplist, POV, Case Study, How-to) using Claude/OpenAI
- **Creates bilingual content** (English & Vietnamese) with customizable tone
- **Renders videos** automatically using Remotion for Reels/TikTok/Shorts
- **Publishes content** to social platforms automatically

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

### Required Environment Variables

```bash
# AI Services
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Social Media (optional)
FACEBOOK_ACCESS_TOKEN=your_facebook_token
TWITTER_API_KEY=your_twitter_key

# Remotion (for video rendering)
REMOTION_LICENSE_KEY=your_remotion_license
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI service integrations
│   │   ├── crawler/     # Content research/scraping
│   │   ├── content/     # Content generation logic
│   │   └── video/       # Remotion video rendering
│   ├── remotion/        # Remotion video templates
│   └── types/           # TypeScript definitions
├── public/              # Static assets
└── package.json
```

## Core API Usage

### 1. Content Research & Crawling

```typescript
import { researchTopic } from '@/lib/crawler/research';
import { crawlNews } from '@/lib/crawler/news-scraper';

// Research a topic from multiple sources
async function gatherResearch(keyword: string) {
  const research = await researchTopic({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeframe: '24h',
    maxResults: 20
  });

  return {
    insights: research.insights,
    trends: research.trends,
    dataPoints: research.statistics,
    sources: research.sourceUrls
  };
}

// Crawl specific news sources
async function crawlLatestNews(topic: string) {
  const articles = await crawlNews({
    query: topic,
    rapidApiKey: process.env.RAPIDAPI_KEY,
    limit: 10
  });

  return articles.map(article => ({
    title: article.title,
    summary: article.summary,
    url: article.url,
    publishedAt: article.publishedAt
  }));
}
```

### 2. AI Content Generation

```typescript
import { generateContent } from '@/lib/ai/content-generator';
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

// Generate content with Claude
async function generateWithClaude(research: any, format: string) {
  const anthropic = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY
  });

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [
      {
        role: 'user',
        content: `Create a ${format} article based on this research:\n\n${JSON.stringify(research)}\n\nFormat: ${format}\nTone: Professional\nLanguages: English and Vietnamese`
      }
    ]
  });

  return message.content[0].text;
}

// Generate content with OpenAI
async function generateWithOpenAI(research: any, format: string) {
  const openai = new OpenAI({
    apiKey: process.env.OPENAI_API_KEY
  });

  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: 'You are an expert content creator specializing in marketing content.'
      },
      {
        role: 'user',
        content: `Create a ${format} article based on this research data: ${JSON.stringify(research)}`
      }
    ],
    temperature: 0.7
  });

  return completion.choices[0].message.content;
}

// Full content generation pipeline
async function createContent(keyword: string, format: 'toplist' | 'pov' | 'case-study' | 'how-to') {
  const research = await gatherResearch(keyword);
  
  const content = await generateContent({
    research,
    format,
    tone: 'professional',
    languages: ['en', 'vi'],
    aiProvider: 'claude' // or 'openai'
  });

  return {
    english: content.en,
    vietnamese: content.vi,
    metadata: {
      format,
      sources: research.sources,
      generatedAt: new Date()
    }
  };
}
```

### 3. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import { getCompositions } from '@remotion/renderer';

// Define video composition
interface VideoProps {
  title: string;
  points: string[];
  bgColor: string;
  duration: number;
}

// Render video from content
async function renderContentVideo(content: any, template: string = 'default') {
  const bundled = await bundle({
    entryPoint: './src/remotion/index.ts',
    webpackOverride: (config) => config
  });

  const compositions = await getCompositions(bundled);
  const composition = selectComposition({
    compositions,
    id: template
  });

  const videoProps: VideoProps = {
    title: content.title,
    points: content.keyPoints,
    bgColor: '#1a1a1a',
    duration: 30 // seconds
  };

  await renderMedia({
    composition,
    serveUrl: bundled,
    codec: 'h264',
    outputLocation: `./output/${content.id}.mp4`,
    inputProps: videoProps,
    envVariables: {
      REMOTION_LICENSE_KEY: process.env.REMOTION_LICENSE_KEY
    }
  });

  return `./output/${content.id}.mp4`;
}

// Create video for social media
async function createSocialVideo(content: any, platform: 'reels' | 'tiktok' | 'shorts') {
  const dimensions = {
    reels: { width: 1080, height: 1920 },
    tiktok: { width: 1080, height: 1920 },
    shorts: { width: 1080, height: 1920 }
  };

  const videoPath = await renderContentVideo(content, `${platform}-template`);
  
  return {
    path: videoPath,
    dimensions: dimensions[platform],
    platform
  };
}
```

### 4. Remotion Video Component Example

```typescript
// src/remotion/compositions/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig, interpolate } from 'remotion';
import React from 'react';

interface ContentVideoProps {
  title: string;
  points: string[];
  bgColor: string;
}

export const ContentVideo: React.FC<ContentVideoProps> = ({ title, points, bgColor }) => {
  const frame = useCurrentFrame();
  const { fps, durationInFrames } = useVideoConfig();

  const titleOpacity = interpolate(frame, [0, 30], [0, 1], {
    extrapolateRight: 'clamp'
  });

  return (
    <AbsoluteFill style={{ backgroundColor: bgColor }}>
      <div style={{ 
        padding: 60, 
        display: 'flex', 
        flexDirection: 'column',
        justifyContent: 'center'
      }}>
        <h1 style={{ 
          fontSize: 80, 
          color: 'white',
          opacity: titleOpacity,
          marginBottom: 40
        }}>
          {title}
        </h1>
        
        {points.map((point, index) => {
          const pointStart = 30 + (index * 20);
          const pointOpacity = interpolate(
            frame, 
            [pointStart, pointStart + 20], 
            [0, 1],
            { extrapolateRight: 'clamp' }
          );
          
          return (
            <p key={index} style={{ 
              fontSize: 40, 
              color: 'white',
              opacity: pointOpacity,
              marginBottom: 20
            }}>
              • {point}
            </p>
          );
        })}
      </div>
    </AbsoluteFill>
  );
};
```

### 5. Complete Pipeline Orchestration

```typescript
import { runContentPipeline } from '@/lib/pipeline/orchestrator';

// Full end-to-end pipeline
async function executeContentPipeline(config: {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  platforms: ('facebook' | 'twitter' | 'linkedin')[];
  generateVideo: boolean;
}) {
  // Step 1: Research
  console.log('🔍 Researching topic...');
  const research = await gatherResearch(config.keyword);

  // Step 2: Generate content
  console.log('✍️ Generating content...');
  const content = await createContent(config.keyword, config.format);

  // Step 3: Generate video (optional)
  let videos = [];
  if (config.generateVideo) {
    console.log('🎬 Rendering videos...');
    videos = await Promise.all([
      createSocialVideo(content, 'reels'),
      createSocialVideo(content, 'tiktok'),
      createSocialVideo(content, 'shorts')
    ]);
  }

  // Step 4: Schedule posts
  console.log('📤 Scheduling posts...');
  const scheduledPosts = await schedulePosts({
    content,
    videos,
    platforms: config.platforms,
    scheduleTime: new Date(Date.now() + 3600000) // 1 hour from now
  });

  return {
    research,
    content,
    videos,
    scheduledPosts,
    status: 'completed'
  };
}

// Usage
const result = await executeContentPipeline({
  keyword: 'AI marketing automation',
  format: 'toplist',
  platforms: ['facebook', 'linkedin'],
  generateVideo: true
});
```

## Configuration

### Content Format Templates

```typescript
// src/lib/content/templates.ts
export const contentTemplates = {
  toplist: {
    structure: ['intro', 'items', 'conclusion'],
    minItems: 5,
    maxItems: 10,
    tone: 'engaging'
  },
  pov: {
    structure: ['hook', 'perspective', 'evidence', 'call-to-action'],
    tone: 'opinionated',
    voiceOptions: ['expert', 'contrarian', 'enthusiast']
  },
  'case-study': {
    structure: ['problem', 'solution', 'results', 'learnings'],
    dataRequired: true,
    tone: 'analytical'
  },
  'how-to': {
    structure: ['intro', 'steps', 'tips', 'conclusion'],
    minSteps: 3,
    tone: 'instructional'
  }
};
```

### AI Provider Configuration

```typescript
// src/lib/ai/config.ts
export const aiConfig = {
  claude: {
    model: 'claude-3-5-sonnet-20241022',
    maxTokens: 4096,
    temperature: 0.7,
    useCase: ['long-form', 'creative', 'analysis']
  },
  openai: {
    model: 'gpt-4-turbo-preview',
    maxTokens: 4096,
    temperature: 0.7,
    useCase: ['quick-generation', 'structured-output']
  }
};
```

## Common Workflows

### Workflow 1: Daily Content Generation

```typescript
// Daily automated content generation
async function dailyContentJob() {
  const topics = ['AI trends', 'Marketing automation', 'Content strategy'];
  
  for (const topic of topics) {
    try {
      await executeContentPipeline({
        keyword: topic,
        format: 'toplist',
        platforms: ['facebook', 'linkedin'],
        generateVideo: true
      });
      
      console.log(`✅ Completed pipeline for: ${topic}`);
    } catch (error) {
      console.error(`❌ Failed for ${topic}:`, error);
    }
  }
}

// Schedule with cron or similar
// 0 9 * * * - Run daily at 9 AM
```

### Workflow 2: Trend-Based Content

```typescript
// Monitor trends and auto-generate content
async function trendMonitoringWorkflow() {
  const trends = await researchTopic({
    keyword: 'emerging tech',
    sources: ['twitter', 'techcrunch'],
    timeframe: '24h',
    minEngagement: 1000
  });

  for (const trend of trends.trending) {
    if (trend.score > 80) {
      await executeContentPipeline({
        keyword: trend.topic,
        format: 'pov',
        platforms: ['twitter', 'linkedin'],
        generateVideo: false
      });
    }
  }
}
```

### Workflow 3: Batch Video Generation

```typescript
// Generate multiple video variants
async function batchVideoGeneration(contentId: string) {
  const content = await getContentById(contentId);
  
  const platforms = ['reels', 'tiktok', 'shorts'] as const;
  const styles = ['minimal', 'bold', 'corporate'];
  
  const videos = [];
  
  for (const platform of platforms) {
    for (const style of styles) {
      const video = await renderContentVideo(content, `${platform}-${style}`);
      videos.push({
        platform,
        style,
        path: video
      });
    }
  }
  
  return videos;
}
```

## Troubleshooting

### AI API Rate Limits

```typescript
// Implement rate limiting and retry logic
import pRetry from 'p-retry';

async function generateWithRetry(prompt: string) {
  return pRetry(
    async () => {
      return await generateWithClaude(prompt, 'toplist');
    },
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
// Check Remotion setup and dependencies
import { getVideoMetadata } from '@remotion/renderer';

async function validateVideoSetup() {
  try {
    const bundled = await bundle({
      entryPoint: './src/remotion/index.ts'
    });
    
    const compositions = await getCompositions(bundled);
    console.log('✅ Available compositions:', compositions.map(c => c.id));
  } catch (error) {
    console.error('❌ Remotion setup error:', error);
  }
}
```

### Content Quality Checks

```typescript
// Validate generated content
function validateContent(content: any): boolean {
  const checks = {
    hasTitle: !!content.title,
    hasBody: content.body && content.body.length > 100,
    hasSources: content.sources && content.sources.length > 0,
    isOriginal: true // Implement plagiarism check if needed
  };

  const passed = Object.values(checks).every(Boolean);
  
  if (!passed) {
    console.warn('Content validation failed:', checks);
  }
  
  return passed;
}
```

### Environment Variable Validation

```typescript
// Validate all required env vars on startup
function validateEnvironment() {
  const required = [
    'ANTHROPIC_API_KEY',
    'OPENAI_API_KEY',
    'RAPIDAPI_KEY'
  ];

  const missing = required.filter(key => !process.env[key]);

  if (missing.length > 0) {
    throw new Error(`Missing required env vars: ${missing.join(', ')}`);
  }

  console.log('✅ Environment validation passed');
}
```

## Development Commands

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Start production server
npm run start

# Render Remotion video
npm run remotion:render

# Preview Remotion composition
npm run remotion:preview

# Type checking
npm run type-check

# Linting
npm run lint
```

This skill provides comprehensive guidance for AI agents to help developers implement, customize, and troubleshoot the Marketing Pipeline Share AI content automation system.
