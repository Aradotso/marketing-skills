---
name: marketing-pipeline-share-automation
description: Automated content pipeline from research to video generation using AI (Claude/OpenAI) and Remotion
triggers:
  - "automate content creation with AI research"
  - "generate video content from text automatically"
  - "set up marketing pipeline with Claude and OpenAI"
  - "scrape news and create content pipeline"
  - "auto-generate multilingual content posts"
  - "build automated content workflow with Remotion"
  - "create AI-powered content automation system"
  - "configure marketing content pipeline"
---

# Marketing Pipeline Share Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This project is an end-to-end AI-powered content automation system that:
- Auto-scrapes news from TechCrunch, a16z, Twitter/X, LinkedIn
- Generates content in multiple formats (Toplist, POV, Case Study, How-to)
- Creates bilingual content (English & Vietnamese)
- Automatically renders videos and infographics using Remotion
- Supports multiple AI providers (Claude 3, OpenAI)

## Installation

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
pnpm install
# or
yarn install
```

## Environment Configuration

Create a `.env.local` file in the root directory:

```bash
# AI Providers
ANTHROPIC_API_KEY=your_claude_api_key_here
OPENAI_API_KEY=your_openai_api_key_here

# RapidAPI for news scraping
RAPIDAPI_KEY=your_rapidapi_key_here

# Application settings
NEXT_PUBLIC_APP_URL=http://localhost:3000
NODE_ENV=development

# Optional: Database (if using persistence)
DATABASE_URL=your_database_url_here
```

Refer to `.env.example` if included in the project.

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app router pages
│   ├── components/       # React components
│   ├── lib/             
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── scraper/     # News scraping logic
│   │   └── video/       # Remotion video generation
│   ├── services/        # Business logic services
│   └── types/           # TypeScript type definitions
├── public/              # Static assets
└── remotion/            # Remotion video templates
```

## Core Workflows

### 1. Research & News Scraping

```typescript
import { scrapeNews } from '@/lib/scraper/newsAggregator';

async function fetchLatestNews(keyword: string) {
  const sources = ['techcrunch', 'a16z', 'twitter', 'linkedin'];
  
  const newsData = await scrapeNews({
    keyword,
    sources,
    timeframe: '24h',
    limit: 20
  });
  
  return newsData;
}

// Usage
const aiNews = await fetchLatestNews('artificial intelligence');
console.log(aiNews.articles); // Array of scraped articles
```

### 2. AI Content Generation with Claude/OpenAI

```typescript
import { generateContent } from '@/lib/ai/contentGenerator';

interface ContentRequest {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  tone: 'expert' | 'friendly' | 'humorous';
  language: 'en' | 'vi' | 'both';
  researchData: any[];
}

async function createAIContent(request: ContentRequest) {
  const content = await generateContent({
    provider: 'claude', // or 'openai'
    model: 'claude-3-opus-20240229',
    prompt: {
      keyword: request.keyword,
      format: request.format,
      tone: request.tone,
      context: request.researchData
    },
    languages: request.language === 'both' ? ['en', 'vi'] : [request.language]
  });
  
  return content;
}

// Example usage
const blogPost = await createAIContent({
  keyword: 'AI marketing automation',
  format: 'how-to',
  tone: 'expert',
  language: 'both',
  researchData: aiNews.articles
});

console.log(blogPost.en); // English version
console.log(blogPost.vi); // Vietnamese version
```

### 3. Multi-Format Content Generation

```typescript
import { ContentFormatter } from '@/services/contentFormatter';

const formatter = new ContentFormatter();

// Generate toplist format
const toplist = await formatter.generateToplist({
  title: 'Top 10 AI Tools for Marketers',
  items: researchData,
  aiProvider: 'claude',
  includeStats: true
});

// Generate POV article
const povArticle = await formatter.generatePOV({
  topic: 'The Future of AI in Content Marketing',
  perspective: 'industry-expert',
  dataPoints: researchData,
  wordCount: 1500
});

// Generate case study
const caseStudy = await formatter.generateCaseStudy({
  subject: 'How Company X increased ROI by 300%',
  metrics: extractedMetrics,
  challenges: extractedChallenges,
  solutions: extractedSolutions
});
```

### 4. Video Generation with Remotion

```typescript
import { renderVideo } from '@/lib/video/remotionRenderer';
import { VideoTemplate } from '@/remotion/templates';

interface VideoConfig {
  content: {
    title: string;
    points: string[];
    stats?: Record<string, string>;
  };
  template: 'reels' | 'tiktok' | 'shorts';
  duration: number;
}

async function generateContentVideo(config: VideoConfig) {
  const composition = VideoTemplate[config.template];
  
  const videoFile = await renderVideo({
    composition,
    props: {
      title: config.content.title,
      bulletPoints: config.content.points,
      statistics: config.content.stats,
      duration: config.duration
    },
    outputFormat: 'mp4',
    dimensions: getDimensionsForPlatform(config.template)
  });
  
  return videoFile;
}

// Helper function
function getDimensionsForPlatform(platform: string) {
  const dimensions = {
    'reels': { width: 1080, height: 1920 },
    'tiktok': { width: 1080, height: 1920 },
    'shorts': { width: 1080, height: 1920 }
  };
  return dimensions[platform];
}

// Usage
const video = await generateContentVideo({
  content: {
    title: 'AI Marketing Trends 2024',
    points: blogPost.en.keyPoints,
    stats: blogPost.en.statistics
  },
  template: 'reels',
  duration: 60
});
```

## Full Pipeline Example

```typescript
import { ContentPipeline } from '@/services/pipeline';

async function runFullContentPipeline(keyword: string) {
  const pipeline = new ContentPipeline({
    aiProvider: 'claude',
    videoEnabled: true,
    autoPublish: false
  });
  
  // Step 1: Research
  const research = await pipeline.research({
    keyword,
    sources: ['techcrunch', 'a16z', 'linkedin'],
    depth: 'comprehensive'
  });
  
  // Step 2: Generate content in multiple formats
  const content = await pipeline.generate({
    formats: ['toplist', 'how-to', 'case-study'],
    languages: ['en', 'vi'],
    tone: 'expert',
    researchData: research
  });
  
  // Step 3: Create videos
  const videos = await pipeline.renderVideos({
    content: content.toplist.en,
    platforms: ['reels', 'tiktok', 'shorts']
  });
  
  // Step 4: Export results
  const output = await pipeline.export({
    content,
    videos,
    format: 'markdown' // or 'html', 'json'
  });
  
  return {
    articles: content,
    videos,
    exportPath: output.path
  };
}

// Execute pipeline
const result = await runFullContentPipeline('AI content automation');
console.log(`Generated ${result.articles.length} articles`);
console.log(`Rendered ${result.videos.length} videos`);
```

## API Routes (Next.js)

### Create Content Endpoint

```typescript
// src/app/api/content/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { generateContent } from '@/lib/ai/contentGenerator';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const { keyword, format, language, tone } = body;
    
    // Validate input
    if (!keyword || !format) {
      return NextResponse.json(
        { error: 'Missing required fields' },
        { status: 400 }
      );
    }
    
    // Generate content
    const content = await generateContent({
      provider: process.env.AI_PROVIDER || 'claude',
      prompt: { keyword, format, tone },
      languages: language === 'both' ? ['en', 'vi'] : [language]
    });
    
    return NextResponse.json({ success: true, content });
  } catch (error) {
    console.error('Content generation error:', error);
    return NextResponse.json(
      { error: 'Failed to generate content' },
      { status: 500 }
    );
  }
}
```

### Video Rendering Endpoint

```typescript
// src/app/api/video/render/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { renderVideo } from '@/lib/video/remotionRenderer';

export async function POST(request: NextRequest) {
  try {
    const { content, template } = await request.json();
    
    const videoUrl = await renderVideo({
      composition: template,
      props: content,
      outputFormat: 'mp4'
    });
    
    return NextResponse.json({ 
      success: true, 
      videoUrl 
    });
  } catch (error) {
    return NextResponse.json(
      { error: 'Video rendering failed' },
      { status: 500 }
    );
  }
}
```

## CLI Usage (if available)

```bash
# Generate content from keyword
npm run generate -- --keyword "AI marketing" --format toplist --lang both

# Run full pipeline
npm run pipeline -- --keyword "automation tools" --video

# Render video only
npm run render-video -- --input ./content/article.json --template reels
```

## Configuration Options

```typescript
// config/pipeline.config.ts
export const pipelineConfig = {
  ai: {
    defaultProvider: 'claude',
    models: {
      claude: 'claude-3-opus-20240229',
      openai: 'gpt-4-turbo-preview'
    },
    maxTokens: 4000,
    temperature: 0.7
  },
  
  scraper: {
    sources: ['techcrunch', 'a16z', 'linkedin', 'twitter'],
    timeframe: '24h',
    maxArticles: 20,
    filters: {
      minRelevance: 0.7
    }
  },
  
  video: {
    defaultTemplate: 'reels',
    fps: 30,
    quality: 'high',
    watermark: false
  },
  
  output: {
    formats: ['markdown', 'html', 'json'],
    directory: './output',
    includeMetadata: true
  }
};
```

## Common Patterns

### Batch Content Generation

```typescript
async function batchGenerateContent(keywords: string[]) {
  const results = await Promise.all(
    keywords.map(async (keyword) => {
      try {
        return await runFullContentPipeline(keyword);
      } catch (error) {
        console.error(`Failed for keyword: ${keyword}`, error);
        return null;
      }
    })
  );
  
  return results.filter(Boolean);
}

const keywords = ['AI marketing', 'Content automation', 'Video generation'];
const batchResults = await batchGenerateContent(keywords);
```

### Custom Content Templates

```typescript
import { ContentTemplate } from '@/types/content';

const customTemplate: ContentTemplate = {
  name: 'product-review',
  structure: {
    intro: { minWords: 100, tone: 'engaging' },
    pros: { type: 'list', minItems: 5 },
    cons: { type: 'list', minItems: 3 },
    verdict: { minWords: 150, includeRating: true }
  },
  seoOptimization: true
};

const review = await generateContent({
  template: customTemplate,
  keyword: 'best AI tools 2024',
  provider: 'claude'
});
```

## Troubleshooting

### API Key Issues

```typescript
// Verify API keys are loaded
if (!process.env.ANTHROPIC_API_KEY) {
  throw new Error('ANTHROPIC_API_KEY not found in environment variables');
}

// Test API connection
import { testAPIConnection } from '@/lib/ai/test';

await testAPIConnection('claude');
await testAPIConnection('openai');
```

### Rate Limiting

```typescript
import { RateLimiter } from '@/lib/utils/rateLimiter';

const limiter = new RateLimiter({
  maxRequests: 10,
  perMinutes: 1
});

async function generateWithRateLimit(keyword: string) {
  await limiter.checkLimit();
  return await generateContent({ keyword });
}
```

### Video Rendering Errors

```bash
# Check Remotion installation
npx remotion versions

# Test render locally
npx remotion render src/remotion/index.ts VideoComposition output.mp4

# Check FFmpeg installation
ffmpeg -version
```

### Memory Issues with Large Batches

```typescript
// Process in chunks
async function processBatchInChunks(items: string[], chunkSize = 5) {
  const chunks = [];
  for (let i = 0; i < items.length; i += chunkSize) {
    chunks.push(items.slice(i, i + chunkSize));
  }
  
  for (const chunk of chunks) {
    await Promise.all(chunk.map(runFullContentPipeline));
    // Allow garbage collection
    await new Promise(resolve => setTimeout(resolve, 1000));
  }
}
```

## Development

```bash
# Run development server
npm run dev

# Build for production
npm run build

# Type checking
npm run type-check

# Lint code
npm run lint
```

## Integration Examples

### With CMS (e.g., WordPress)

```typescript
import { publishToWordPress } from '@/integrations/wordpress';

const result = await runFullContentPipeline('AI trends');

await publishToWordPress({
  title: result.articles.en.title,
  content: result.articles.en.body,
  featuredImage: result.videos[0].thumbnail,
  status: 'draft' // or 'publish'
});
```

### With Social Media Schedulers

```typescript
import { schedulePost } from '@/integrations/socialScheduler';

await schedulePost({
  platform: 'linkedin',
  content: result.articles.en.summary,
  media: result.videos[0].url,
  scheduledTime: new Date('2024-06-15T10:00:00Z')
});
```
