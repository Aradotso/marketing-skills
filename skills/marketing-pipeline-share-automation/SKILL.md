---
name: marketing-pipeline-share-automation
description: Automated AI content pipeline from research to video generation using Claude, OpenAI, and Remotion
triggers:
  - how do I automate content creation with AI
  - generate video from blog posts automatically
  - set up AI marketing content pipeline
  - create automated content research workflow
  - build AI content generation system
  - automate social media content with AI
  - generate videos from articles using remotion
  - set up automated content crawling and writing
---

# Marketing Pipeline Share - AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill helps you work with **marketing-pipeline-share**, an end-to-end AI-powered content automation system that handles research, scriptwriting, and video generation. It crawls real-time news from sources like TechCrunch, a16z, Twitter, and LinkedIn, then generates multilingual content and renders videos using Remotion.

## What It Does

The marketing-pipeline-share project is a TypeScript/Next.js application that:

- **Auto-crawls** trending news and insights from major tech sources
- **Generates content** in multiple formats (listicles, POV, case studies, how-tos) using Claude 3 and OpenAI
- **Creates bilingual content** (English and Vietnamese) with customizable tone
- **Renders videos** automatically from written content using Remotion
- **Optimizes for platforms** like Reels, TikTok, and Shorts

## Installation

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pipeline-share.git
cd marketing-pipeline-share

# Install dependencies
npm install
# or
yarn install
# or
pnpm install

# Set up environment variables
cp .env.example .env.local
```

## Configuration

Create a `.env.local` file with the following required variables:

```bash
# AI APIs
OPENAI_API_KEY=your_openai_key_here
ANTHROPIC_API_KEY=your_claude_key_here

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key_here

# Database (if using)
DATABASE_URL=your_database_connection_string

# Remotion (for video rendering)
REMOTION_LICENSE_KEY=your_remotion_license_key

# Optional: Social Media APIs
FACEBOOK_PAGE_TOKEN=your_facebook_token
TWITTER_API_KEY=your_twitter_key
```

## Core Architecture

The pipeline consists of three main stages:

1. **Research Stage**: Crawl and analyze real-time data
2. **Content Generation Stage**: Create articles in multiple formats
3. **Video Rendering Stage**: Convert content to video using Remotion

## Usage Patterns

### 1. Automated Research & Crawling

```typescript
import { ResearchService } from '@/services/research';

const researchService = new ResearchService({
  apiKey: process.env.RAPIDAPI_KEY,
  sources: ['techcrunch', 'a16z', 'twitter', 'linkedin']
});

// Crawl recent articles by keyword
async function gatherResearch(keyword: string) {
  const articles = await researchService.crawlNews({
    keyword,
    timeframe: '24h',
    limit: 20
  });
  
  const insights = await researchService.extractInsights(articles);
  
  return {
    articles,
    insights,
    trends: insights.filter(i => i.score > 0.7)
  };
}
```

### 2. AI Content Generation with Claude

```typescript
import { ContentGenerator } from '@/services/content-generator';
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY
});

const generator = new ContentGenerator(anthropic);

// Generate content in specific format
async function generateArticle(
  keyword: string, 
  format: 'toplist' | 'pov' | 'case-study' | 'how-to',
  language: 'en' | 'vi'
) {
  const research = await gatherResearch(keyword);
  
  const article = await generator.create({
    format,
    language,
    tone: 'professional', // or 'friendly', 'humorous'
    research: research.insights,
    wordCount: 1500
  });
  
  return article;
}
```

### 3. OpenAI Alternative

```typescript
import { OpenAI } from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY
});

async function generateWithGPT(prompt: string, research: any[]) {
  const completion = await openai.chat.completions.create({
    model: "gpt-4-turbo-preview",
    messages: [
      {
        role: "system",
        content: "You are an expert content writer creating data-backed marketing content."
      },
      {
        role: "user",
        content: `Research data: ${JSON.stringify(research)}\n\nPrompt: ${prompt}`
      }
    ],
    temperature: 0.7,
    max_tokens: 2000
  });
  
  return completion.choices[0].message.content;
}
```

### 4. Video Rendering with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import { VideoComposition } from '@/remotion/compositions/article-video';

async function renderArticleVideo(article: any) {
  // Bundle the Remotion project
  const bundleLocation = await bundle({
    entryPoint: './remotion/index.ts',
    webpackOverride: (config) => config
  });
  
  // Select composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ArticleVideo',
    inputProps: {
      title: article.title,
      content: article.content,
      highlights: article.keyPoints,
      duration: 60 // seconds
    }
  });
  
  // Render video
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: `out/${article.id}.mp4`,
    inputProps: composition.inputProps
  });
  
  return `out/${article.id}.mp4`;
}
```

### 5. Complete Pipeline Example

```typescript
import { Pipeline } from '@/services/pipeline';

const pipeline = new Pipeline({
  anthropic: process.env.ANTHROPIC_API_KEY,
  openai: process.env.OPENAI_API_KEY,
  rapidapi: process.env.RAPIDAPI_KEY
});

// Full automation workflow
async function runContentPipeline(keyword: string) {
  try {
    // Step 1: Research
    console.log('Starting research...');
    const research = await pipeline.research(keyword);
    
    // Step 2: Generate content in multiple formats
    console.log('Generating content...');
    const articles = await Promise.all([
      pipeline.generateContent(research, 'toplist', 'en'),
      pipeline.generateContent(research, 'toplist', 'vi'),
      pipeline.generateContent(research, 'how-to', 'en')
    ]);
    
    // Step 3: Render videos
    console.log('Rendering videos...');
    const videos = await Promise.all(
      articles.map(article => pipeline.renderVideo(article))
    );
    
    // Step 4: Schedule posts (optional)
    await pipeline.schedulePost({
      articles,
      videos,
      platforms: ['facebook', 'twitter', 'linkedin']
    });
    
    return { articles, videos };
  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}
```

## API Routes (Next.js)

### Generate Content Endpoint

```typescript
// pages/api/generate.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { ContentPipeline } from '@/lib/pipeline';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }
  
  const { keyword, format, language } = req.body;
  
  try {
    const pipeline = new ContentPipeline();
    const result = await pipeline.run({
      keyword,
      format,
      language
    });
    
    res.status(200).json(result);
  } catch (error) {
    res.status(500).json({ 
      error: 'Generation failed', 
      message: error.message 
    });
  }
}
```

### Research Endpoint

```typescript
// pages/api/research.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { ResearchService } from '@/services/research';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  const { keyword } = req.query;
  
  try {
    const service = new ResearchService({
      apiKey: process.env.RAPIDAPI_KEY
    });
    
    const data = await service.crawlNews({
      keyword: keyword as string,
      timeframe: '24h'
    });
    
    res.status(200).json(data);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
}
```

## CLI Commands

If the project includes CLI tools:

```bash
# Run research for a keyword
npm run research -- --keyword "AI automation" --sources techcrunch,a16z

# Generate content
npm run generate -- --keyword "AI automation" --format toplist --lang en

# Render video
npm run render -- --article-id abc123 --output ./videos

# Full pipeline
npm run pipeline -- --keyword "AI automation" --formats toplist,how-to --langs en,vi
```

## Common Patterns

### Custom Content Templates

```typescript
// Define custom template
const customTemplate = {
  name: 'startup-spotlight',
  structure: [
    'hook',
    'problem',
    'solution',
    'metrics',
    'insights',
    'cta'
  ],
  tone: 'inspirational',
  wordCount: 1200
};

// Use with generator
const article = await generator.createFromTemplate(
  customTemplate,
  researchData
);
```

### Batch Processing

```typescript
async function batchGenerateContent(keywords: string[]) {
  const results = [];
  
  for (const keyword of keywords) {
    const article = await pipeline.run({
      keyword,
      format: 'toplist',
      language: 'en'
    });
    
    results.push(article);
    
    // Rate limiting
    await new Promise(resolve => setTimeout(resolve, 2000));
  }
  
  return results;
}
```

### Error Handling & Retries

```typescript
async function generateWithRetry(params: any, maxRetries = 3) {
  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      return await pipeline.generateContent(params);
    } catch (error) {
      if (attempt === maxRetries) throw error;
      
      console.log(`Attempt ${attempt} failed, retrying...`);
      await new Promise(resolve => 
        setTimeout(resolve, 1000 * attempt)
      );
    }
  }
}
```

## Troubleshooting

### API Rate Limits

If you encounter rate limit errors:

```typescript
import { RateLimiter } from '@/utils/rate-limiter';

const limiter = new RateLimiter({
  requestsPerMinute: 10
});

async function throttledRequest(fn: () => Promise<any>) {
  await limiter.wait();
  return fn();
}
```

### Video Rendering Timeout

Increase timeout for large videos:

```typescript
await renderMedia({
  composition,
  serveUrl: bundleLocation,
  codec: 'h264',
  outputLocation: output,
  timeoutInMilliseconds: 120000 // 2 minutes
});
```

### Memory Issues with Large Crawls

Process in chunks:

```typescript
async function crawlInChunks(keywords: string[], chunkSize = 5) {
  const results = [];
  
  for (let i = 0; i < keywords.length; i += chunkSize) {
    const chunk = keywords.slice(i, i + chunkSize);
    const chunkResults = await Promise.all(
      chunk.map(k => researchService.crawlNews({ keyword: k }))
    );
    results.push(...chunkResults);
  }
  
  return results;
}
```

### Claude API Errors

Handle context length issues:

```typescript
async function generateWithContextWindow(content: string) {
  const maxTokens = 100000; // Claude's limit
  
  if (content.length > maxTokens * 4) { // rough estimate
    // Summarize research first
    const summary = await anthropic.messages.create({
      model: 'claude-3-haiku-20240307',
      max_tokens: 2000,
      messages: [{
        role: 'user',
        content: `Summarize this research: ${content}`
      }]
    });
    
    content = summary.content[0].text;
  }
  
  return await generator.create({ research: content });
}
```

## Development Server

```bash
# Run Next.js development server
npm run dev

# Access at http://localhost:3000

# Build for production
npm run build

# Start production server
npm start
```

## Best Practices

1. **Always validate research data** before generating content
2. **Cache research results** to avoid redundant API calls
3. **Use environment-specific configs** for different deployment stages
4. **Monitor API usage** to stay within rate limits
5. **Store generated content** in a database for tracking and reuse
6. **Test video rendering locally** before deploying to production
