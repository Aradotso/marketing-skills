---
name: ultimate-ai-content-pipeline
description: Automated content creation pipeline with AI research, multilingual script generation, and video rendering using Claude/OpenAI and Remotion
triggers:
  - how do I automate content creation with AI research
  - set up automated video generation pipeline
  - create multilingual content with Claude and OpenAI
  - build AI content automation system
  - generate videos from text with Remotion
  - automate research and content writing workflow
  - set up AI marketing content pipeline
  - create automated social media content system
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This project is a complete automated content creation pipeline that transforms keywords into fully-researched, multilingual articles and videos. It crawls recent data from sources like TechCrunch and Twitter, generates content using Claude/OpenAI, and renders videos using Remotion.

## What It Does

- **Auto-Research**: Crawls and analyzes real-time data from news sources and social media
- **AI Content Generation**: Creates articles in multiple formats (toplist, POV, case study, how-to)
- **Multilingual Support**: Generates content in English and Vietnamese simultaneously
- **Video Rendering**: Automatically creates infographics and short-form videos from content
- **Multi-Platform Export**: Optimized for Reels, TikTok, Shorts

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
# AI APIs
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_url

# Video Rendering
REMOTION_LICENSE_KEY=your_remotion_license_key
```

## Project Structure

```
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── research/    # Web scraping and data collection
│   │   ├── content/     # Content generation logic
│   │   └── video/       # Remotion video rendering
│   └── utils/           # Helper functions
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Core Usage Patterns

### 1. Research & Data Collection

```typescript
import { autoResearch } from '@/lib/research/auto-scan';

// Crawl recent data on a topic
const researchData = await autoResearch({
  keyword: 'AI automation',
  sources: ['techcrunch', 'twitter', 'linkedin'],
  timeframe: '24h',
  language: 'en'
});

// Returns structured data with insights
console.log(researchData.insights);
console.log(researchData.sources);
console.log(researchData.statistics);
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
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4000,
    messages: [
      {
        role: 'user',
        content: `Create a ${format} article about ${topic} in ${language}. 
        Include data-backed insights and current trends.`
      }
    ],
  });

  return message.content[0].text;
}

// Generate content
const article = await generateContent('AI Marketing Tools', 'toplist', 'en');
```

### 3. Multilingual Content Generation

```typescript
import { generateMultilingualContent } from '@/lib/content/multilingual';

// Generate content in multiple languages simultaneously
const multilingualContent = await generateMultilingualContent({
  topic: 'Content Automation Trends 2026',
  format: 'how-to',
  languages: ['en', 'vi'],
  tone: 'professional', // or 'friendly', 'humorous'
  researchData: researchData
});

console.log(multilingualContent.en); // English version
console.log(multilingualContent.vi); // Vietnamese version
```

### 4. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import { webpackOverride } from './remotion/webpack-override';

async function renderContentVideo(content: any) {
  // Bundle the Remotion project
  const bundleLocation = await bundle({
    entryPoint: './remotion/index.ts',
    webpackOverride,
  });

  // Select composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title: content.title,
      points: content.keyPoints,
      duration: 30, // seconds
    },
  });

  // Render video
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: `out/${content.slug}.mp4`,
  });
}
```

### 5. Complete Content Pipeline

```typescript
import { ContentPipeline } from '@/lib/pipeline';

const pipeline = new ContentPipeline({
  aiProvider: 'claude', // or 'openai'
  languages: ['en', 'vi'],
  outputFormats: ['article', 'video', 'infographic']
});

// Run full pipeline
const result = await pipeline.run({
  keyword: 'AI Marketing Automation',
  contentFormat: 'toplist',
  videoAspectRatio: '9:16', // for Reels/TikTok
  autoPublish: false
});

console.log(result.articles);  // Generated articles
console.log(result.videos);    // Rendered video paths
console.log(result.metadata);  // SEO and social metadata
```

## API Integration Examples

### Research API Integration

```typescript
import axios from 'axios';

async function fetchTrendingTopics() {
  const options = {
    method: 'GET',
    url: 'https://trending-topics-api.p.rapidapi.com/topics',
    params: { category: 'technology' },
    headers: {
      'X-RapidAPI-Key': process.env.RAPIDAPI_KEY,
      'X-RapidAPI-Host': 'trending-topics-api.p.rapidapi.com'
    }
  };

  const response = await axios.request(options);
  return response.data;
}
```

### OpenAI Integration Alternative

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateWithGPT(prompt: string) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: 'You are an expert content writer specializing in marketing.'
      },
      {
        role: 'user',
        content: prompt
      }
    ],
    temperature: 0.7,
  });

  return completion.choices[0].message.content;
}
```

## Running the Application

### Development Server

```bash
# Start Next.js development server
npm run dev

# Access at http://localhost:3000
```

### Video Rendering

```bash
# Render a specific composition
npx remotion render remotion/index.ts ContentVideo out/video.mp4

# Preview in Remotion Studio
npx remotion studio
```

### Build for Production

```bash
# Build Next.js app
npm run build

# Start production server
npm run start
```

## Common Workflows

### Workflow 1: Generate Daily Content

```typescript
import { scheduleContentGeneration } from '@/lib/scheduler';

// Schedule daily content generation
scheduleContentGeneration({
  cron: '0 9 * * *', // 9 AM daily
  topics: ['AI trends', 'Marketing automation', 'Content strategy'],
  formats: ['toplist', 'how-to'],
  languages: ['en', 'vi'],
  autoRender: true,
  autoPublish: false
});
```

### Workflow 2: Trend-Based Content

```typescript
async function generateTrendingContent() {
  // 1. Research trending topics
  const trends = await fetchTrendingTopics();
  
  // 2. Generate content for top 3 trends
  const contentPromises = trends.slice(0, 3).map(trend =>
    generateContent(trend.title, 'pov', 'en')
  );
  
  const articles = await Promise.all(contentPromises);
  
  // 3. Render videos for each article
  for (const article of articles) {
    await renderContentVideo(article);
  }
  
  return articles;
}
```

### Workflow 3: Custom Content Format

```typescript
interface CustomContentConfig {
  topic: string;
  targetAudience: string;
  contentGoal: 'awareness' | 'engagement' | 'conversion';
  includeDataPoints: boolean;
}

async function generateCustomContent(config: CustomContentConfig) {
  const researchData = await autoResearch({ 
    keyword: config.topic 
  });
  
  const prompt = `
    Create content about ${config.topic} for ${config.targetAudience}.
    Goal: ${config.contentGoal}
    ${config.includeDataPoints ? 'Include relevant statistics and data points.' : ''}
    
    Research insights: ${JSON.stringify(researchData.insights)}
  `;
  
  return await generateWithGPT(prompt);
}
```

## Configuration Options

### Content Generation Config

```typescript
interface ContentConfig {
  aiProvider: 'claude' | 'openai';
  model?: string;
  temperature?: number;
  maxTokens?: number;
  tone?: 'professional' | 'friendly' | 'humorous';
  includeImages?: boolean;
  seoOptimized?: boolean;
}

const config: ContentConfig = {
  aiProvider: 'claude',
  model: 'claude-3-5-sonnet-20241022',
  temperature: 0.7,
  maxTokens: 4000,
  tone: 'professional',
  includeImages: true,
  seoOptimized: true
};
```

### Video Rendering Config

```typescript
interface VideoConfig {
  fps: number;
  width: number;
  height: number;
  codec: 'h264' | 'h265';
  quality: 'low' | 'medium' | 'high';
  aspectRatio: '16:9' | '9:16' | '1:1';
}

const videoConfig: VideoConfig = {
  fps: 30,
  width: 1080,
  height: 1920,
  codec: 'h264',
  quality: 'high',
  aspectRatio: '9:16' // for TikTok/Reels
};
```

## Troubleshooting

### API Rate Limits

```typescript
import { RateLimiter } from '@/lib/utils/rate-limiter';

const limiter = new RateLimiter({
  maxRequests: 50,
  windowMs: 60000, // 1 minute
});

async function callAIWithRateLimit(prompt: string) {
  await limiter.wait();
  return await generateContent(prompt, 'toplist', 'en');
}
```

### Error Handling

```typescript
async function safeContentGeneration(topic: string) {
  try {
    const content = await generateContent(topic, 'toplist', 'en');
    return { success: true, content };
  } catch (error) {
    if (error.status === 429) {
      console.error('Rate limit exceeded, retry in 60s');
      await new Promise(resolve => setTimeout(resolve, 60000));
      return safeContentGeneration(topic);
    }
    
    if (error.status === 401) {
      console.error('Invalid API key');
      throw new Error('Check your API keys in .env.local');
    }
    
    console.error('Content generation failed:', error);
    return { success: false, error: error.message };
  }
}
```

### Video Rendering Issues

```typescript
// Check Remotion setup
import { getCompositions } from '@remotion/renderer';

async function debugRemotionSetup() {
  try {
    const bundleLocation = await bundle({
      entryPoint: './remotion/index.ts',
    });
    
    const compositions = await getCompositions(bundleLocation);
    console.log('Available compositions:', compositions);
  } catch (error) {
    console.error('Remotion setup error:', error);
    console.log('Check: 1) remotion/ directory exists, 2) index.ts is valid');
  }
}
```

### Memory Management for Large Content

```typescript
// Process content in batches
async function batchContentGeneration(topics: string[]) {
  const batchSize = 5;
  const results = [];
  
  for (let i = 0; i < topics.length; i += batchSize) {
    const batch = topics.slice(i, i + batchSize);
    const batchResults = await Promise.all(
      batch.map(topic => generateContent(topic, 'toplist', 'en'))
    );
    results.push(...batchResults);
    
    // Clear memory between batches
    if (global.gc) global.gc();
  }
  
  return results;
}
```

## Best Practices

1. **Always validate research data** before passing to AI
2. **Cache research results** to avoid redundant API calls
3. **Use streaming** for long-form content generation
4. **Implement retry logic** for API failures
5. **Monitor AI costs** with usage tracking
6. **Version control** your prompt templates
7. **Test video renders** before batch processing
