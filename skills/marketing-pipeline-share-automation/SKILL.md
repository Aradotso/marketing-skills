---
name: marketing-pipeline-share-automation
description: Automate content creation from research to video generation using AI (Claude/OpenAI) and Remotion
triggers:
  - how do I automate content creation from research to video
  - set up an AI content pipeline with Claude and OpenAI
  - generate videos automatically from written content
  - crawl news sources and create content with AI
  - build an automated marketing content workflow
  - create multilingual content with AI video generation
  - automate social media content production end-to-end
  - research and generate content using AI agents
---

# Marketing Pipeline Share Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This project is an **all-in-one AI content automation pipeline** that handles research, content generation, and video production. It crawls real-time news from sources like TechCrunch, Twitter, and LinkedIn, generates content in multiple formats (Toplist, POV, Case Study, How-to) using Claude/OpenAI, and automatically renders videos using Remotion.

## What It Does

- **Auto-Research**: Crawls and analyzes fresh content from major tech news sources within 24 hours
- **AI Content Generation**: Creates multilingual content (English/Vietnamese) in various formats with customizable tone
- **Video Rendering**: Automatically converts written content into videos/infographics optimized for Reels, TikTok, Shorts
- **Full Pipeline**: Takes a keyword input and outputs complete content packages ready for publishing

## Installation

### Prerequisites

```bash
# Node.js 18+ required
node --version

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

```env
# AI APIs
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research/Crawling APIs
RAPIDAPI_KEY=your_rapidapi_key

# Optional: Social Media APIs
TWITTER_BEARER_TOKEN=your_twitter_token
LINKEDIN_API_KEY=your_linkedin_key

# Database (if applicable)
DATABASE_URL=your_database_url

# Next.js
NEXT_PUBLIC_API_URL=http://localhost:3000
```

### Start Development Server

```bash
npm run dev
# Server runs on http://localhost:3000
```

### Build for Production

```bash
npm run build
npm start
```

## Project Structure

```
marketing-pineline-share/
├── app/                    # Next.js app directory
├── components/             # React components
├── lib/
│   ├── ai/                # AI integration (Claude, OpenAI)
│   ├── crawler/           # Web scraping modules
│   ├── video/             # Remotion video generation
│   └── utils/             # Helper functions
├── public/                # Static assets
└── remotion/              # Remotion video templates
```

## Core Modules

### 1. Research & Crawling

```typescript
import { ResearchEngine } from '@/lib/crawler/research-engine';

// Initialize research engine
const researcher = new ResearchEngine({
  sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
  timeRange: '24h'
});

// Crawl content by keyword
async function researchTopic(keyword: string) {
  const results = await researcher.crawl({
    keyword: keyword,
    maxArticles: 10,
    includeInsights: true
  });
  
  return {
    articles: results.articles,
    trends: results.extractedTrends,
    dataPoints: results.statistics
  };
}

// Example usage
const aiResearch = await researchTopic('artificial intelligence trends');
console.log(aiResearch.trends);
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

// Generate content with Claude
async function generateContentClaude(
  topic: string,
  format: 'toplist' | 'pov' | 'case-study' | 'how-to',
  language: 'en' | 'vi'
) {
  const generator = new ContentGenerator(anthropic);
  
  const content = await generator.create({
    topic: topic,
    format: format,
    language: language,
    tone: 'professional', // or 'friendly', 'humorous'
    researchData: await researchTopic(topic),
    includeStats: true
  });
  
  return {
    title: content.title,
    body: content.body,
    keywords: content.keywords,
    metadata: content.metadata
  };
}

// Generate bilingual content
async function generateBilingual(topic: string) {
  const [english, vietnamese] = await Promise.all([
    generateContentClaude(topic, 'toplist', 'en'),
    generateContentClaude(topic, 'toplist', 'vi')
  ]);
  
  return { english, vietnamese };
}
```

### 3. Alternative with OpenAI

```typescript
async function generateContentOpenAI(topic: string) {
  const response = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: 'You are an expert content creator specializing in tech marketing.'
      },
      {
        role: 'user',
        content: `Create a comprehensive toplist article about: ${topic}`
      }
    ],
    temperature: 0.7,
    max_tokens: 2000
  });
  
  return response.choices[0].message.content;
}
```

### 4. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import { VideoTemplate } from '@/remotion/templates/content-video';

async function generateVideo(content: any) {
  // Bundle Remotion project
  const bundled = await bundle({
    entryPoint: './remotion/index.ts',
    webpackOverride: (config) => config
  });
  
  // Select composition
  const composition = await selectComposition({
    serveUrl: bundled,
    id: 'ContentVideo',
    inputProps: {
      title: content.title,
      body: content.body,
      style: 'modern', // or 'minimal', 'vibrant'
      duration: 60 // seconds
    }
  });
  
  // Render video
  await renderMedia({
    composition,
    serveUrl: bundled,
    codec: 'h264',
    outputLocation: `./output/video-${Date.now()}.mp4`,
    inputProps: composition.inputProps
  });
  
  return { success: true, path: './output' };
}
```

### 5. Full Pipeline Execution

```typescript
import { ContentPipeline } from '@/lib/pipeline';

async function runFullPipeline(keyword: string) {
  const pipeline = new ContentPipeline({
    aiProvider: 'claude', // or 'openai'
    languages: ['en', 'vi'],
    outputFormats: ['article', 'video', 'infographic']
  });
  
  try {
    // Step 1: Research
    const research = await pipeline.research(keyword);
    console.log('Research completed:', research.insights);
    
    // Step 2: Generate content
    const content = await pipeline.generateContent({
      keyword,
      researchData: research,
      format: 'toplist',
      tone: 'professional'
    });
    console.log('Content generated:', content.title);
    
    // Step 3: Create video
    const video = await pipeline.renderVideo(content);
    console.log('Video rendered:', video.path);
    
    // Step 4: Optimize for platforms
    const optimized = await pipeline.optimizeForPlatforms(video, {
      platforms: ['tiktok', 'reels', 'youtube-shorts']
    });
    
    return {
      content,
      video,
      optimized,
      status: 'completed'
    };
  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}

// Execute pipeline
const result = await runFullPipeline('AI marketing automation 2024');
```

## API Routes

### Next.js API Endpoints

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';

export async function POST(req: NextRequest) {
  const { keyword, format, language } = await req.json();
  
  const pipeline = new ContentPipeline({
    aiProvider: process.env.AI_PROVIDER || 'claude'
  });
  
  const result = await pipeline.run({
    keyword,
    format,
    language
  });
  
  return NextResponse.json(result);
}

// Usage with fetch
const response = await fetch('/api/generate', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    keyword: 'content marketing trends',
    format: 'toplist',
    language: 'en'
  })
});

const data = await response.json();
```

## Configuration

### AI Model Configuration

```typescript
// lib/config/ai-config.ts
export const aiConfig = {
  claude: {
    model: 'claude-3-opus-20240229',
    maxTokens: 4096,
    temperature: 0.7
  },
  openai: {
    model: 'gpt-4-turbo-preview',
    maxTokens: 4000,
    temperature: 0.7
  }
};

// Use in content generation
const generator = new ContentGenerator(anthropic, {
  model: aiConfig.claude.model,
  maxTokens: aiConfig.claude.maxTokens
});
```

### Video Template Configuration

```typescript
// remotion/config.ts
export const videoConfig = {
  fps: 30,
  width: 1080,
  height: 1920, // 9:16 for vertical video
  durationInFrames: 1800, // 60 seconds at 30fps
  
  styles: {
    modern: {
      backgroundColor: '#000000',
      primaryColor: '#00ff00',
      fontFamily: 'Inter'
    },
    minimal: {
      backgroundColor: '#ffffff',
      primaryColor: '#333333',
      fontFamily: 'Helvetica'
    }
  }
};
```

## Common Patterns

### Batch Content Generation

```typescript
async function batchGenerateContent(keywords: string[]) {
  const results = await Promise.allSettled(
    keywords.map(keyword => runFullPipeline(keyword))
  );
  
  const successful = results
    .filter(r => r.status === 'fulfilled')
    .map(r => (r as PromiseFulfilledResult<any>).value);
  
  const failed = results
    .filter(r => r.status === 'rejected')
    .map(r => (r as PromiseRejectedResult).reason);
  
  return { successful, failed };
}

// Generate content for multiple topics
const keywords = [
  'AI in marketing',
  'Content automation trends',
  'Video marketing 2024'
];

const batchResults = await batchGenerateContent(keywords);
```

### Custom Content Format

```typescript
interface CustomFormat {
  structure: 'intro' | 'body' | 'conclusion';
  sectionCount: number;
  includeCallToAction: boolean;
}

async function generateCustomFormat(
  topic: string,
  format: CustomFormat
) {
  const prompt = `
    Create content about ${topic} with:
    - ${format.sectionCount} main sections
    - Structure: ${format.structure}
    ${format.includeCallToAction ? '- Include strong CTA at the end' : ''}
  `;
  
  const content = await anthropic.messages.create({
    model: 'claude-3-opus-20240229',
    max_tokens: 4096,
    messages: [{
      role: 'user',
      content: prompt
    }]
  });
  
  return content.content[0].text;
}
```

### Scheduled Content Pipeline

```typescript
import cron from 'node-cron';

// Run pipeline daily at 9 AM
cron.schedule('0 9 * * *', async () => {
  console.log('Running daily content generation...');
  
  const dailyKeywords = await fetchTrendingTopics();
  const results = await batchGenerateContent(dailyKeywords);
  
  // Auto-publish or save to CMS
  await publishToWordPress(results.successful);
  
  console.log(`Generated ${results.successful.length} pieces of content`);
});
```

## Troubleshooting

### API Rate Limits

```typescript
import pLimit from 'p-limit';

// Limit concurrent API calls
const limit = pLimit(3);

async function generateWithRateLimit(keywords: string[]) {
  return Promise.all(
    keywords.map(keyword => 
      limit(() => runFullPipeline(keyword))
    )
  );
}
```

### Video Rendering Errors

```typescript
// Add error handling and retries
async function renderVideoWithRetry(content: any, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await generateVideo(content);
    } catch (error) {
      console.error(`Render attempt ${i + 1} failed:`, error);
      
      if (i === maxRetries - 1) throw error;
      
      // Wait before retry
      await new Promise(resolve => setTimeout(resolve, 5000 * (i + 1)));
    }
  }
}
```

### Memory Issues with Large Content

```typescript
// Stream large content generation
import { Readable } from 'stream';

async function streamContent(topic: string) {
  const stream = await anthropic.messages.stream({
    model: 'claude-3-opus-20240229',
    max_tokens: 4096,
    messages: [{
      role: 'user',
      content: `Write about ${topic}`
    }]
  });
  
  let fullContent = '';
  
  for await (const chunk of stream) {
    if (chunk.type === 'content_block_delta') {
      fullContent += chunk.delta.text;
      // Process in chunks to avoid memory issues
    }
  }
  
  return fullContent;
}
```

### Environment-Specific Configuration

```typescript
// lib/config/index.ts
const isDev = process.env.NODE_ENV === 'development';

export const config = {
  ai: {
    provider: isDev ? 'openai' : 'claude',
    debug: isDev
  },
  video: {
    quality: isDev ? 'draft' : 'high',
    skipRendering: isDev && process.env.SKIP_VIDEO === 'true'
  }
};
```

## Advanced Usage

### Multi-Platform Optimization

```typescript
async function optimizeForAllPlatforms(content: any) {
  const platforms = {
    tiktok: { width: 1080, height: 1920, duration: 60 },
    reels: { width: 1080, height: 1920, duration: 90 },
    youtube: { width: 1920, height: 1080, duration: 120 },
    shorts: { width: 1080, height: 1920, duration: 60 }
  };
  
  return Promise.all(
    Object.entries(platforms).map(([platform, specs]) =>
      renderMedia({
        ...composition,
        width: specs.width,
        height: specs.height,
        durationInFrames: specs.duration * 30,
        outputLocation: `./output/${platform}-${Date.now()}.mp4`
      })
    )
  );
}
```

This skill enables AI agents to leverage the full power of automated content creation, from research through video generation, optimized for marketing workflows.
