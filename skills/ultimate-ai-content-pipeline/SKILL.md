---
name: ultimate-ai-content-pipeline
description: Vietnamese-focused AI content automation system for research, scriptwriting, and video generation using Claude/OpenAI and Remotion
triggers:
  - how do I set up the AI content pipeline system
  - automate content creation with research and video generation
  - use the marketing pipeline to generate posts
  - configure Claude and OpenAI for content automation
  - generate videos from content using Remotion
  - crawl news sources for content research
  - create multilingual content with AI pipeline
  - troubleshoot the content generation workflow
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This TypeScript-based system automates the entire content creation workflow from research to video generation. It crawls real-time news sources (TechCrunch, Twitter/X, LinkedIn), generates content in multiple formats and languages using Claude/OpenAI, and renders videos using Remotion.

## What It Does

The Ultimate AI Content Pipeline is an end-to-end content automation system that:

- **Auto-scans research sources** — Crawls news from TechCrunch, a16z, X, LinkedIn within the last 24h
- **Generates diverse content formats** — Toplists, POV articles, case studies, how-tos in Vietnamese & English
- **Renders videos automatically** — Uses Remotion to create infographics and short-form videos
- **Optimizes for multiple platforms** — Outputs content for Reels, TikTok, Shorts with proper ratios

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

### Environment Configuration

Create a `.env` file with the following variables:

```bash
# AI API Keys
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research API
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_url

# Remotion Settings
REMOTION_AWS_ACCESS_KEY_ID=your_aws_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── crawler/     # News crawling logic
│   │   ├── video/       # Remotion video generation
│   │   └── utils/       # Helper functions
│   └── types/           # TypeScript type definitions
├── public/              # Static assets
└── remotion/            # Remotion video templates
```

## Key Usage Patterns

### 1. Research & Content Crawling

```typescript
import { crawlNewsSource } from '@/lib/crawler/news-crawler';
import { analyzeContent } from '@/lib/ai/analyzer';

// Crawl news from multiple sources
async function gatherResearch(keyword: string) {
  const sources = ['techcrunch', 'twitter', 'linkedin'];
  
  const articles = await Promise.all(
    sources.map(source => 
      crawlNewsSource({
        source,
        keyword,
        timeRange: '24h',
        limit: 10
      })
    )
  );
  
  // Flatten and analyze
  const allArticles = articles.flat();
  const insights = await analyzeContent(allArticles);
  
  return {
    articles: allArticles,
    insights,
    statistics: {
      totalSources: sources.length,
      totalArticles: allArticles.length
    }
  };
}
```

### 2. AI Content Generation with Claude/OpenAI

```typescript
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

// Initialize AI clients
const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY
});

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY
});

// Generate content using Claude
async function generateContentWithClaude(
  research: any,
  format: 'toplist' | 'pov' | 'case-study' | 'how-to',
  language: 'vi' | 'en'
) {
  const systemPrompt = `You are a content creator specialized in ${format} articles.
Generate content in ${language === 'vi' ? 'Vietnamese' : 'English'}.`;

  const userPrompt = `Based on this research:
${JSON.stringify(research.insights, null, 2)}

Create a ${format} article that:
- Uses data-backed insights
- Matches the target audience tone
- Includes specific examples from the research
- Is optimized for social media engagement`;

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    system: systemPrompt,
    messages: [{
      role: 'user',
      content: userPrompt
    }]
  });

  return message.content[0].type === 'text' 
    ? message.content[0].text 
    : '';
}

// Generate content using OpenAI
async function generateContentWithOpenAI(
  research: any,
  format: string,
  tone: 'expert' | 'friendly' | 'humorous'
) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are a ${tone} content writer creating ${format} content.`
      },
      {
        role: 'user',
        content: `Create content based on: ${JSON.stringify(research)}`
      }
    ],
    temperature: 0.7,
    max_tokens: 3000
  });

  return completion.choices[0].message.content;
}
```

### 3. Multilingual Content Generation

```typescript
interface ContentRequest {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  languages: ('vi' | 'en')[];
  tone: 'expert' | 'friendly' | 'humorous';
}

async function generateMultilingualContent(request: ContentRequest) {
  // Gather research
  const research = await gatherResearch(request.keyword);
  
  // Generate content for each language
  const contentPromises = request.languages.map(lang => 
    generateContentWithClaude(research, request.format, lang)
  );
  
  const contents = await Promise.all(contentPromises);
  
  return request.languages.reduce((acc, lang, idx) => {
    acc[lang] = contents[idx];
    return acc;
  }, {} as Record<string, string>);
}

// Usage
const bilingualContent = await generateMultilingualContent({
  keyword: 'AI trends 2024',
  format: 'toplist',
  languages: ['vi', 'en'],
  tone: 'expert'
});
```

### 4. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import { getCompositions } from '@remotion/renderer';

interface VideoConfig {
  content: string;
  title: string;
  platform: 'reels' | 'tiktok' | 'shorts';
}

// Platform-specific dimensions
const platformDimensions = {
  reels: { width: 1080, height: 1920 },
  tiktok: { width: 1080, height: 1920 },
  shorts: { width: 1080, height: 1920 }
};

async function generateVideo(config: VideoConfig) {
  const { content, title, platform } = config;
  const { width, height } = platformDimensions[platform];
  
  // Bundle the Remotion project
  const bundleLocation = await bundle({
    entryPoint: './remotion/index.ts',
    webpackOverride: (config) => config
  });
  
  // Get compositions
  const compositions = await getCompositions(bundleLocation);
  const composition = selectComposition({
    compositions,
    id: 'ContentVideo'
  });
  
  // Prepare input props
  const inputProps = {
    title,
    content: parseContentForVideo(content),
    backgroundColor: '#1a1a1a',
    textColor: '#ffffff'
  };
  
  // Render video
  const outputLocation = `./output/${platform}-${Date.now()}.mp4`;
  
  await renderMedia({
    composition: {
      ...composition,
      width,
      height,
      durationInFrames: 300, // 10 seconds at 30fps
      fps: 30
    },
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps
  });
  
  return outputLocation;
}

// Parse content into video segments
function parseContentForVideo(content: string) {
  const lines = content.split('\n').filter(line => line.trim());
  return lines.slice(0, 5).map((line, idx) => ({
    id: idx,
    text: line.trim(),
    duration: 60 // frames
  }));
}
```

### 5. Complete Workflow Pipeline

```typescript
interface PipelineConfig {
  keyword: string;
  formats: Array<'toplist' | 'pov' | 'case-study' | 'how-to'>;
  languages: Array<'vi' | 'en'>;
  generateVideo: boolean;
  platforms?: Array<'reels' | 'tiktok' | 'shorts'>;
}

async function runContentPipeline(config: PipelineConfig) {
  console.log('🚀 Starting content pipeline...');
  
  // Step 1: Research
  console.log('📡 Gathering research...');
  const research = await gatherResearch(config.keyword);
  
  // Step 2: Generate content for each format and language
  console.log('🧠 Generating content...');
  const contentResults = [];
  
  for (const format of config.formats) {
    for (const lang of config.languages) {
      const content = await generateContentWithClaude(
        research,
        format,
        lang
      );
      
      contentResults.push({
        format,
        language: lang,
        content,
        wordCount: content.split(/\s+/).length
      });
    }
  }
  
  // Step 3: Generate videos if requested
  const videoResults = [];
  if (config.generateVideo && config.platforms) {
    console.log('🎬 Generating videos...');
    
    for (const contentResult of contentResults) {
      if (contentResult.language === 'vi') { // Generate videos for Vietnamese only
        for (const platform of config.platforms) {
          const videoPath = await generateVideo({
            content: contentResult.content,
            title: `${config.keyword} - ${contentResult.format}`,
            platform
          });
          
          videoResults.push({
            format: contentResult.format,
            platform,
            path: videoPath
          });
        }
      }
    }
  }
  
  console.log('✅ Pipeline complete!');
  
  return {
    research: {
      totalArticles: research.articles.length,
      insights: research.insights
    },
    content: contentResults,
    videos: videoResults,
    summary: {
      totalContent: contentResults.length,
      totalVideos: videoResults.length,
      totalWordCount: contentResults.reduce((sum, r) => sum + r.wordCount, 0)
    }
  };
}

// Usage example
const result = await runContentPipeline({
  keyword: 'AI Marketing Tools 2024',
  formats: ['toplist', 'how-to'],
  languages: ['vi', 'en'],
  generateVideo: true,
  platforms: ['reels', 'tiktok']
});
```

## API Integration Examples

### RapidAPI for News Crawling

```typescript
import axios from 'axios';

async function crawlWithRapidAPI(keyword: string) {
  const options = {
    method: 'GET',
    url: 'https://news-api14.p.rapidapi.com/search',
    params: {
      q: keyword,
      language: 'en',
      sortBy: 'publishedAt',
      from: new Date(Date.now() - 24 * 60 * 60 * 1000).toISOString()
    },
    headers: {
      'X-RapidAPI-Key': process.env.RAPIDAPI_KEY,
      'X-RapidAPI-Host': 'news-api14.p.rapidapi.com'
    }
  };
  
  try {
    const response = await axios.request(options);
    return response.data.articles || [];
  } catch (error) {
    console.error('RapidAPI error:', error);
    return [];
  }
}
```

## Running the Application

```bash
# Development mode
npm run dev

# Build for production
npm run build

# Start production server
npm run start

# Run video rendering separately
npm run render
```

## Common Patterns

### Scheduled Content Generation

```typescript
import cron from 'node-cron';

// Run pipeline every day at 6 AM
cron.schedule('0 6 * * *', async () => {
  const result = await runContentPipeline({
    keyword: 'daily tech news',
    formats: ['toplist'],
    languages: ['vi', 'en'],
    generateVideo: true,
    platforms: ['reels']
  });
  
  console.log('Scheduled pipeline completed:', result.summary);
});
```

### Content Storage and Management

```typescript
import fs from 'fs/promises';
import path from 'path';

async function saveContent(content: any, metadata: any) {
  const timestamp = new Date().toISOString().split('T')[0];
  const dirPath = path.join('./content', timestamp);
  
  await fs.mkdir(dirPath, { recursive: true });
  
  // Save content
  await fs.writeFile(
    path.join(dirPath, 'content.json'),
    JSON.stringify({ content, metadata }, null, 2)
  );
  
  // Save markdown version
  const mdContent = `# ${metadata.title}\n\n${content}`;
  await fs.writeFile(
    path.join(dirPath, 'article.md'),
    mdContent
  );
}
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement rate limiting for API calls
import pLimit from 'p-limit';

const limit = pLimit(3); // Max 3 concurrent requests

async function generateContentBatch(requests: ContentRequest[]) {
  return Promise.all(
    requests.map(req => 
      limit(() => generateMultilingualContent(req))
    )
  );
}
```

### Error Handling

```typescript
async function safeGenerateContent(config: any) {
  try {
    return await generateContentWithClaude(config.research, config.format, config.language);
  } catch (error: any) {
    if (error.status === 429) {
      console.log('Rate limited, waiting 60s...');
      await new Promise(resolve => setTimeout(resolve, 60000));
      return safeGenerateContent(config);
    }
    
    if (error.status === 500) {
      console.log('API error, trying OpenAI fallback...');
      return generateContentWithOpenAI(config.research, config.format, 'expert');
    }
    
    throw error;
  }
}
```

### Video Rendering Issues

```typescript
// Handle Remotion rendering errors
async function safeRenderVideo(config: VideoConfig) {
  try {
    return await generateVideo(config);
  } catch (error: any) {
    console.error('Video rendering failed:', error.message);
    
    // Fallback to lower quality
    if (error.message.includes('memory')) {
      console.log('Reducing video quality...');
      // Implement quality reduction logic
    }
    
    throw error;
  }
}
```

### Missing Dependencies

If you encounter errors:

```bash
# Ensure all Remotion dependencies are installed
npm install @remotion/bundler @remotion/renderer @remotion/cli

# Install AI SDKs
npm install @anthropic-ai/sdk openai

# Install utility packages
npm install axios p-limit node-cron
```

## Performance Optimization

```typescript
// Cache research results
import NodeCache from 'node-cache';

const researchCache = new NodeCache({ stdTTL: 3600 }); // 1 hour

async function getCachedResearch(keyword: string) {
  const cached = researchCache.get(keyword);
  if (cached) return cached;
  
  const research = await gatherResearch(keyword);
  researchCache.set(keyword, research);
  return research;
}
```

This skill enables AI coding agents to effectively use the Ultimate AI Content Pipeline for automated content creation, from research crawling to video generation, with support for multiple languages and platforms.
