---
name: ultimate-ai-content-pipeline
description: Automated content pipeline from research to video generation using AI (Claude/OpenAI) and Remotion for Vietnamese and English content
triggers:
  - generate content with AI research and video
  - automate content creation with Claude and OpenAI
  - create video content from text automatically
  - build content pipeline with research crawling
  - generate multilingual marketing content with AI
  - create social media videos from articles
  - automate content research and scriptwriting
  - generate TikTok and Reels videos from AI content
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection

A comprehensive TypeScript-based content automation system that handles the entire content lifecycle: from crawling real-time news sources, generating AI-powered articles in multiple formats and languages, to automatically rendering videos for social media platforms.

## What It Does

This project automates the complete content creation workflow:

1. **Auto Research**: Crawls recent content from TechCrunch, a16z, X (Twitter), LinkedIn
2. **AI Content Generation**: Uses Claude 3 and OpenAI to create articles in multiple formats (toplist, POV, case study, how-to)
3. **Multilingual Support**: Generates content in both English and Vietnamese
4. **Video Generation**: Automatically renders videos and infographics using Remotion
5. **Platform Optimization**: Exports videos optimized for Reels, TikTok, and YouTube Shorts

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

Create a `.env.local` file in the root directory:

```bash
# AI API Keys
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# RapidAPI for web crawling
RAPIDAPI_KEY=your_rapidapi_key

# Next.js Configuration
NEXT_PUBLIC_API_URL=http://localhost:3000

# Remotion Configuration
REMOTION_AWS_ACCESS_KEY_ID=your_aws_access_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret_key
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/             # Core utilities
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── crawler/     # Web scraping modules
│   │   └── video/       # Remotion video generation
│   ├── api/             # API routes
│   └── types/           # TypeScript types
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Core API Usage

### 1. Content Research & Crawling

```typescript
import { crawlRecentNews } from '@/lib/crawler/news-crawler';
import { analyzeContent } from '@/lib/ai/content-analyzer';

// Crawl recent news from multiple sources
async function researchTopic(keyword: string) {
  const sources = [
    'techcrunch.com',
    'a16z.com',
    'twitter.com',
    'linkedin.com'
  ];
  
  const articles = await crawlRecentNews({
    keyword,
    sources,
    timeRange: '24h',
    limit: 20
  });
  
  // Analyze and extract insights using AI
  const insights = await analyzeContent(articles, {
    model: 'claude-3-opus-20240229',
    extractDataPoints: true,
    summarize: true
  });
  
  return insights;
}
```

### 2. AI Content Generation

```typescript
import { generateArticle } from '@/lib/ai/content-generator';

// Generate article with Claude or OpenAI
async function createContent(topic: string, format: string) {
  const article = await generateArticle({
    topic,
    format: format, // 'toplist' | 'pov' | 'case-study' | 'how-to'
    language: 'dual', // Generates both EN and VI
    tone: 'professional', // 'friendly' | 'expert' | 'humorous'
    length: 'medium', // 'short' | 'medium' | 'long'
    includeData: true,
    sources: await researchTopic(topic),
    aiProvider: 'claude' // or 'openai'
  });
  
  return article;
}
```

### 3. Video Generation with Remotion

```typescript
import { renderVideo } from '@/lib/video/remotion-renderer';
import { createVideoScript } from '@/lib/ai/script-generator';

// Generate video from article content
async function generateVideo(article: Article) {
  // Create video script from article
  const script = await createVideoScript({
    content: article.content,
    platform: 'reels', // 'reels' | 'tiktok' | 'shorts'
    duration: 30, // seconds
    style: 'infographic'
  });
  
  // Render video using Remotion
  const video = await renderVideo({
    compositionId: 'ContentVideo',
    inputProps: {
      script: script.scenes,
      title: article.title,
      branding: {
        logo: '/logo.png',
        colors: ['#FF6B6B', '#4ECDC4']
      }
    },
    outputFormat: 'mp4',
    resolution: [1080, 1920] // Vertical video
  });
  
  return video;
}
```

## Complete Workflow Example

```typescript
import { ContentPipeline } from '@/lib/pipeline';

// Full automated content pipeline
async function runContentPipeline(keyword: string) {
  const pipeline = new ContentPipeline({
    aiProvider: 'claude',
    videoEnabled: true,
    autoPublish: false
  });
  
  try {
    // Step 1: Research
    console.log('🔍 Researching topic...');
    const research = await pipeline.research({
      keyword,
      sources: ['techcrunch', 'a16z', 'twitter'],
      depth: 'comprehensive'
    });
    
    // Step 2: Generate Content
    console.log('✍️ Generating content...');
    const content = await pipeline.generateContent({
      research,
      formats: ['toplist', 'how-to'],
      languages: ['en', 'vi'],
      count: 2
    });
    
    // Step 3: Create Videos
    console.log('🎬 Rendering videos...');
    const videos = await pipeline.generateVideos({
      articles: content.articles,
      platforms: ['reels', 'tiktok', 'shorts'],
      templates: ['minimal', 'bold']
    });
    
    // Step 4: Export Results
    return {
      articles: content.articles,
      videos: videos.files,
      metadata: {
        keyword,
        generatedAt: new Date(),
        totalArticles: content.articles.length,
        totalVideos: videos.files.length
      }
    };
    
  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}

// Usage
const result = await runContentPipeline('AI marketing automation');
console.log(`Generated ${result.totalArticles} articles and ${result.totalVideos} videos`);
```

## API Routes

### POST /api/research

```typescript
// Request
fetch('/api/research', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    keyword: 'AI content marketing',
    sources: ['techcrunch', 'a16z'],
    timeRange: '24h'
  })
});

// Response
{
  "articles": [...],
  "insights": {
    "topTrends": [...],
    "dataPoints": [...],
    "keyTopics": [...]
  }
}
```

### POST /api/generate

```typescript
// Request
fetch('/api/generate', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    topic: 'Marketing automation trends 2024',
    format: 'toplist',
    language: 'dual',
    research: researchData
  })
});

// Response
{
  "english": {
    "title": "...",
    "content": "...",
    "meta": {...}
  },
  "vietnamese": {
    "title": "...",
    "content": "...",
    "meta": {...}
  }
}
```

### POST /api/video/render

```typescript
// Request
fetch('/api/video/render', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    articleId: 'article-123',
    platform: 'reels',
    template: 'minimal'
  })
});

// Response
{
  "videoUrl": "https://...",
  "duration": 30,
  "resolution": [1080, 1920],
  "format": "mp4"
}
```

## Running the Development Server

```bash
# Start Next.js dev server
npm run dev

# Start Remotion Studio for video editing
npm run remotion:studio

# Build for production
npm run build

# Start production server
npm run start
```

## Remotion Video Configuration

Create custom video templates in `remotion/` directory:

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig } from 'remotion';

export const ContentVideo: React.FC<{
  title: string;
  script: Scene[];
  branding: BrandConfig;
}> = ({ title, script, branding }) => {
  const frame = useCurrentFrame();
  const { fps, durationInFrames } = useVideoConfig();
  
  return (
    <AbsoluteFill style={{ backgroundColor: branding.colors[0] }}>
      {/* Title Scene */}
      {frame < 60 && (
        <div className="title-scene">
          <h1>{title}</h1>
        </div>
      )}
      
      {/* Content Scenes */}
      {script.map((scene, index) => {
        const startFrame = scene.startTime * fps;
        const endFrame = scene.endTime * fps;
        
        if (frame >= startFrame && frame < endFrame) {
          return (
            <div key={index} className="content-scene">
              <p>{scene.text}</p>
              {scene.image && <img src={scene.image} alt="" />}
            </div>
          );
        }
        return null;
      })}
    </AbsoluteFill>
  );
};
```

## Common Patterns

### Batch Content Generation

```typescript
async function batchGenerateContent(keywords: string[]) {
  const results = await Promise.allSettled(
    keywords.map(keyword => 
      runContentPipeline(keyword)
    )
  );
  
  return results.map((result, index) => ({
    keyword: keywords[index],
    status: result.status,
    data: result.status === 'fulfilled' ? result.value : null,
    error: result.status === 'rejected' ? result.reason : null
  }));
}
```

### Custom AI Prompts

```typescript
import { createCustomPrompt } from '@/lib/ai/prompt-builder';

const customPrompt = createCustomPrompt({
  role: 'expert marketer',
  task: 'create engaging social media post',
  constraints: [
    'max 280 characters',
    'include call-to-action',
    'use 3 relevant hashtags'
  ],
  context: researchData,
  examples: [
    'Example post 1...',
    'Example post 2...'
  ]
});

const result = await generateArticle({
  customPrompt,
  aiProvider: 'openai',
  model: 'gpt-4-turbo'
});
```

### Video Template Switching

```typescript
const templates = {
  minimal: 'MinimalTemplate',
  bold: 'BoldTemplate',
  infographic: 'InfographicTemplate',
  story: 'StoryTemplate'
};

async function renderWithTemplate(
  content: Article, 
  templateName: keyof typeof templates
) {
  return await renderVideo({
    compositionId: templates[templateName],
    inputProps: {
      content: content.content,
      style: templateName
    }
  });
}
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement rate limiting for AI APIs
import { rateLimit } from '@/lib/utils/rate-limiter';

const limiter = rateLimit({
  claude: { requestsPerMinute: 50 },
  openai: { requestsPerMinute: 60 }
});

await limiter.wait('claude');
const result = await generateArticle({ aiProvider: 'claude' });
```

### Crawler Errors

```typescript
// Handle crawler failures gracefully
try {
  const articles = await crawlRecentNews({ keyword });
} catch (error) {
  if (error.code === 'RATE_LIMIT') {
    console.log('Rate limited, using cached data');
    return getCachedArticles(keyword);
  }
  throw error;
}
```

### Video Rendering Timeout

```typescript
// Set appropriate timeout for video rendering
const video = await renderVideo({
  compositionId: 'ContentVideo',
  inputProps: {},
  timeout: 300000, // 5 minutes
  onProgress: (progress) => {
    console.log(`Rendering: ${progress}%`);
  }
});
```

### Memory Issues with Large Content

```typescript
// Stream large content generation
import { streamGeneration } from '@/lib/ai/streaming';

const stream = await streamGeneration({
  topic,
  onChunk: (chunk) => {
    // Process chunk immediately
    processChunk(chunk);
  },
  onComplete: (fullContent) => {
    // Final processing
  }
});
```

## Performance Optimization

```typescript
// Cache research results
import { cache } from '@/lib/cache';

async function cachedResearch(keyword: string) {
  const cacheKey = `research:${keyword}`;
  
  const cached = await cache.get(cacheKey);
  if (cached) return cached;
  
  const research = await researchTopic(keyword);
  await cache.set(cacheKey, research, { ttl: 3600 }); // 1 hour
  
  return research;
}

// Parallel video generation
async function generateMultipleVideos(articles: Article[]) {
  const videos = await Promise.all(
    articles.map(article => generateVideo(article))
  );
  return videos;
}
```
