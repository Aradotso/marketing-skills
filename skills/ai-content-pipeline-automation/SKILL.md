---
name: ai-content-pipeline-automation
description: Automated AI content pipeline for research, scriptwriting, video generation, and multi-platform publishing using Claude, OpenAI, and Remotion
triggers:
  - "set up automated content generation pipeline"
  - "create AI-powered marketing content workflow"
  - "generate videos from blog posts automatically"
  - "automate content research and publishing"
  - "build multi-language content automation system"
  - "integrate Claude and OpenAI for content creation"
  - "render videos with Remotion for social media"
  - "scrape news and generate AI content"
---

# AI Content Pipeline Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the Ultimate AI Content Pipeline, a comprehensive TypeScript-based system that automates content creation from research to video generation. The pipeline crawls news sources, generates multi-format content using Claude/OpenAI, and renders videos using Remotion.

## What This Project Does

The AI Content Pipeline automates the entire content creation workflow:

1. **Auto-Research**: Crawls TechCrunch, a16z, Twitter/X, LinkedIn for fresh data (last 24h)
2. **AI Content Generation**: Creates multi-format content (toplist, POV, case study, how-to) in multiple languages
3. **Video Rendering**: Automatically generates infographics and short videos from written content
4. **Multi-Platform Export**: Outputs optimized content for Reels, TikTok, Shorts

Built with Next.js, TypeScript, and integrates with Claude 3, OpenAI, and Remotion.

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
cp .env.example .env.local
```

## Configuration

Create a `.env.local` file with the following environment variables:

```bash
# AI Model APIs
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# News/Data APIs
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Remotion Configuration
REMOTION_AWS_ACCESS_KEY_ID=your_aws_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret

# Application Settings
NEXT_PUBLIC_APP_URL=http://localhost:3000
NODE_ENV=development
```

## Project Structure

```
marketing-pineline-share/
├── app/                    # Next.js app directory
├── components/             # React components
├── lib/                    # Core utilities and services
│   ├── ai/                # AI integration (Claude, OpenAI)
│   ├── scraper/           # News crawling modules
│   └── video/             # Remotion video rendering
├── public/                # Static assets
└── remotion/              # Remotion video templates
```

## Core API Usage

### 1. Content Research & Crawling

```typescript
import { NewsScraperService } from '@/lib/scraper/news-scraper';

// Initialize the news scraper
const scraper = new NewsScraperService({
  apiKey: process.env.RAPIDAPI_KEY,
  sources: ['techcrunch', 'a16z', 'twitter', 'linkedin']
});

// Fetch trending topics from last 24 hours
async function fetchTrendingTopics(keyword: string) {
  const articles = await scraper.searchNews({
    query: keyword,
    timeRange: '24h',
    language: 'en'
  });

  return articles.map(article => ({
    title: article.title,
    url: article.url,
    summary: article.description,
    publishedAt: article.publishedAt,
    source: article.source.name
  }));
}
```

### 2. AI Content Generation with Claude

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

interface ContentRequest {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  researchData: string;
}

async function generateContent(request: ContentRequest) {
  const prompt = `
You are a professional content writer. Based on the following research data, 
create a ${request.format} article about "${request.keyword}" in ${request.language}.

Tone: ${request.tone}

Research Data:
${request.researchData}

Generate a complete article with:
- Compelling headline
- Introduction
- Main content (structured by format)
- Conclusion with CTA
- Meta description for SEO
`;

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [{
      role: 'user',
      content: prompt
    }]
  });

  return message.content[0].text;
}
```

### 3. AI Content Generation with OpenAI

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateContentWithGPT(request: ContentRequest) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are a ${request.tone} content writer specializing in ${request.format} format.`
      },
      {
        role: 'user',
        content: `Create a ${request.language} article about "${request.keyword}" using this research: ${request.researchData}`
      }
    ],
    temperature: 0.7,
    max_tokens: 3000
  });

  return completion.choices[0].message.content;
}
```

### 4. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import { webpackOverride } from './webpack-override';
import path from 'path';

interface VideoConfig {
  title: string;
  content: string[];
  aspectRatio: '9:16' | '16:9' | '1:1'; // TikTok/Reels, YouTube, Instagram
  duration: number;
}

async function generateVideo(config: VideoConfig) {
  const compositionId = 'ContentVideo';
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: webpackOverride,
  });

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: {
      title: config.title,
      content: config.content,
      aspectRatio: config.aspectRatio
    },
  });

  const outputLocation = path.join(
    process.cwd(), 
    'public', 
    'videos', 
    `${Date.now()}.mp4`
  );

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: {
      title: config.title,
      content: config.content,
      aspectRatio: config.aspectRatio
    },
  });

  return outputLocation;
}
```

### 5. Complete Pipeline Example

```typescript
import { ContentPipeline } from '@/lib/pipeline/content-pipeline';

async function runCompleteContentPipeline(keyword: string) {
  const pipeline = new ContentPipeline({
    anthropicKey: process.env.ANTHROPIC_API_KEY,
    openaiKey: process.env.OPENAI_API_KEY,
    rapidApiKey: process.env.RAPIDAPI_KEY
  });

  try {
    // Step 1: Research
    console.log('🔍 Researching topic...');
    const research = await pipeline.research(keyword);

    // Step 2: Generate Content (English & Vietnamese)
    console.log('✍️ Generating content...');
    const contentEn = await pipeline.generateContent({
      keyword,
      format: 'toplist',
      language: 'en',
      tone: 'expert',
      researchData: research.summary
    });

    const contentVi = await pipeline.generateContent({
      keyword,
      format: 'toplist',
      language: 'vi',
      tone: 'friendly',
      researchData: research.summary
    });

    // Step 3: Extract key points for video
    const keyPoints = await pipeline.extractKeyPoints(contentEn);

    // Step 4: Generate videos for multiple platforms
    console.log('🎬 Rendering videos...');
    const videoTikTok = await pipeline.renderVideo({
      title: keyPoints.headline,
      content: keyPoints.bullets,
      aspectRatio: '9:16',
      duration: 30
    });

    const videoYouTube = await pipeline.renderVideo({
      title: keyPoints.headline,
      content: keyPoints.bullets,
      aspectRatio: '16:9',
      duration: 60
    });

    return {
      research,
      content: {
        english: contentEn,
        vietnamese: contentVi
      },
      videos: {
        tiktok: videoTikTok,
        youtube: videoYouTube
      }
    };
  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}

// Usage
runCompleteContentPipeline('AI automation tools 2024')
  .then(result => console.log('✅ Pipeline complete:', result))
  .catch(err => console.error('❌ Pipeline failed:', err));
```

## Common Patterns

### Multi-Language Content Generation

```typescript
async function generateMultiLanguageContent(
  keyword: string, 
  languages: ('en' | 'vi')[]
) {
  const results = await Promise.all(
    languages.map(lang => 
      generateContent({
        keyword,
        format: 'how-to',
        language: lang,
        tone: 'friendly',
        researchData: await fetchResearch(keyword)
      })
    )
  );

  return languages.reduce((acc, lang, idx) => {
    acc[lang] = results[idx];
    return acc;
  }, {} as Record<string, string>);
}
```

### Batch Video Rendering

```typescript
async function renderMultiPlatformVideos(content: string) {
  const platforms = [
    { name: 'tiktok', aspectRatio: '9:16' as const, duration: 30 },
    { name: 'youtube', aspectRatio: '16:9' as const, duration: 60 },
    { name: 'instagram', aspectRatio: '1:1' as const, duration: 45 }
  ];

  const videos = await Promise.all(
    platforms.map(platform => 
      generateVideo({
        title: extractTitle(content),
        content: extractBullets(content),
        aspectRatio: platform.aspectRatio,
        duration: platform.duration
      })
    )
  );

  return platforms.reduce((acc, platform, idx) => {
    acc[platform.name] = videos[idx];
    return acc;
  }, {} as Record<string, string>);
}
```

### Content Scheduling

```typescript
interface ScheduledContent {
  content: string;
  publishAt: Date;
  platforms: string[];
  videoUrls?: Record<string, string>;
}

async function scheduleContent(
  keyword: string, 
  publishDate: Date
): Promise<ScheduledContent> {
  const research = await fetchTrendingTopics(keyword);
  const content = await generateContent({
    keyword,
    format: 'toplist',
    language: 'en',
    tone: 'expert',
    researchData: JSON.stringify(research)
  });

  const videos = await renderMultiPlatformVideos(content);

  return {
    content,
    publishAt: publishDate,
    platforms: ['facebook', 'linkedin', 'twitter'],
    videoUrls: videos
  };
}
```

## Development Commands

```bash
# Run development server
npm run dev

# Build for production
npm run build

# Start production server
npm run start

# Run type checking
npm run type-check

# Lint code
npm run lint

# Render Remotion video (local preview)
npm run remotion:preview

# Render Remotion video (production)
npm run remotion:render
```

## API Routes

The Next.js application exposes the following API endpoints:

### POST /api/research
Research a topic and gather data from news sources.

```typescript
// Request
fetch('/api/research', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ keyword: 'AI tools 2024' })
});

// Response
{
  "articles": [...],
  "summary": "...",
  "sources": [...]
}
```

### POST /api/generate
Generate content using AI models.

```typescript
// Request
fetch('/api/generate', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    keyword: 'marketing automation',
    format: 'toplist',
    language: 'en',
    tone: 'expert',
    model: 'claude' // or 'openai'
  })
});

// Response
{
  "content": "...",
  "metadata": {...}
}
```

### POST /api/video/render
Render a video from content.

```typescript
// Request
fetch('/api/video/render', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    title: 'Top 10 AI Tools',
    content: ['Tool 1...', 'Tool 2...'],
    aspectRatio: '9:16',
    duration: 30
  })
});

// Response
{
  "videoUrl": "/videos/1234567890.mp4",
  "status": "completed"
}
```

## Troubleshooting

### API Rate Limits

```typescript
import { RateLimiter } from '@/lib/utils/rate-limiter';

const limiter = new RateLimiter({
  maxRequests: 10,
  windowMs: 60000 // 1 minute
});

async function generateWithRateLimit(request: ContentRequest) {
  await limiter.waitForSlot();
  return generateContent(request);
}
```

### Video Rendering Memory Issues

If Remotion rendering fails with memory errors:

```typescript
// Increase Node.js heap size
// In package.json scripts:
{
  "scripts": {
    "remotion:render": "node --max-old-space-size=4096 node_modules/@remotion/cli/dist/index.js render"
  }
}
```

### Content Quality Validation

```typescript
function validateGeneratedContent(content: string): boolean {
  const minLength = 500;
  const hasHeadline = content.includes('#') || content.toLowerCase().includes('title:');
  const hasStructure = content.split('\n\n').length >= 3;

  if (content.length < minLength) {
    throw new Error('Content too short');
  }
  if (!hasHeadline) {
    throw new Error('Missing headline');
  }
  if (!hasStructure) {
    throw new Error('Poor content structure');
  }

  return true;
}
```

### Retry Logic for API Calls

```typescript
async function retryWithBackoff<T>(
  fn: () => Promise<T>,
  maxRetries: number = 3
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      const delay = Math.pow(2, i) * 1000;
      await new Promise(resolve => setTimeout(resolve, delay));
    }
  }
  throw new Error('Max retries exceeded');
}

// Usage
const content = await retryWithBackoff(() => 
  generateContent(request)
);
```

## Best Practices

1. **Use environment variables** for all API keys and sensitive configuration
2. **Implement rate limiting** to avoid API quota issues
3. **Cache research data** to minimize redundant API calls
4. **Validate AI-generated content** before publishing
5. **Monitor video rendering** jobs for failures and resource usage
6. **Use TypeScript types** for all API requests and responses
7. **Implement error handling** with proper logging for debugging
