---
name: marketing-pipeline-ai-content-automation
description: Automated AI content pipeline for research, scriptwriting, and video generation using Claude/OpenAI and Remotion
triggers:
  - automate content creation with AI research and video
  - set up marketing pipeline with Claude and OpenAI
  - generate content from keyword to video automatically
  - create AI-powered content workflow with Remotion
  - build automated marketing content system
  - implement end-to-end content generation pipeline
  - configure AI content research and video rendering
  - develop automated social media content creation
---

# Marketing Pipeline AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

The Marketing Pipeline AI Content Automation system is a comprehensive TypeScript-based solution that automates the entire content creation workflow from research to video generation. It combines web scraping, AI content generation (Claude 3, OpenAI), and video rendering (Remotion) to create a fully automated content production pipeline.

**Key capabilities:**
- Auto-scrape trending news from TechCrunch, a16z, Twitter, LinkedIn
- Generate multi-format content (listicles, POV, case studies, how-tos)
- Dual-language output (English & Vietnamese)
- Automatic video and infographic rendering
- Platform-optimized exports (Reels, TikTok, Shorts)

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

Create a `.env.local` file in the project root:

```env
# AI Provider Keys
OPENAI_API_KEY=your_openai_key_here
ANTHROPIC_API_KEY=your_anthropic_key_here

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key_here

# Optional: Social Media APIs
TWITTER_API_KEY=your_twitter_key_here
LINKEDIN_API_KEY=your_linkedin_key_here

# Next.js Configuration
NEXT_PUBLIC_APP_URL=http://localhost:3000

# Remotion Configuration
REMOTION_LICENSE_KEY=your_remotion_license_here
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── components/          # React components
│   ├── lib/
│   │   ├── ai/             # AI integration (Claude, OpenAI)
│   │   ├── scraper/        # Web scraping modules
│   │   ├── video/          # Remotion video generation
│   │   └── utils/          # Utility functions
│   ├── pages/              # Next.js pages
│   └── types/              # TypeScript definitions
├── public/                 # Static assets
└── remotion/              # Remotion video templates
```

## Core Modules

### 1. Research & Scraping Module

```typescript
import { ResearchService } from '@/lib/scraper/research-service';

// Initialize research service
const researchService = new ResearchService({
  rapidApiKey: process.env.RAPIDAPI_KEY,
  sources: ['techcrunch', 'a16z', 'twitter', 'linkedin']
});

// Scan for trending topics
async function gatherResearch(keyword: string) {
  const results = await researchService.scanTrending({
    keyword,
    timeRange: '24h',
    minEngagement: 100,
    language: 'en'
  });

  return {
    articles: results.articles,
    insights: results.insights,
    stats: results.statistics,
    trends: results.trendingTopics
  };
}

// Example usage
const research = await gatherResearch('AI marketing automation');
console.log(`Found ${research.articles.length} relevant articles`);
```

### 2. AI Content Generation

```typescript
import { ContentGenerator } from '@/lib/ai/content-generator';
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

// Initialize AI clients
const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY
});

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY
});

const contentGen = new ContentGenerator({ anthropic, openai });

// Generate content with Claude
async function generateContent(
  topic: string,
  format: 'toplist' | 'pov' | 'case-study' | 'how-to',
  researchData: any
) {
  const content = await contentGen.generate({
    topic,
    format,
    tone: 'professional',
    languages: ['en', 'vi'],
    researchContext: researchData,
    model: 'claude-3-5-sonnet-20241022'
  });

  return {
    english: content.en,
    vietnamese: content.vi,
    metadata: content.metadata,
    seo: content.seoOptimization
  };
}

// Example: Generate toplist article
const article = await generateContent(
  'Top 10 AI Tools for Content Marketing',
  'toplist',
  research
);
```

### 3. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import { VideoComposition } from '@/remotion/compositions/ContentVideo';

// Render video from article content
async function renderContentVideo(
  articleData: any,
  platform: 'reels' | 'tiktok' | 'shorts'
) {
  // Platform-specific dimensions
  const dimensions = {
    reels: { width: 1080, height: 1920 },
    tiktok: { width: 1080, height: 1920 },
    shorts: { width: 1080, height: 1920 }
  };

  const bundleLocation = await bundle({
    entryPoint: './remotion/index.ts',
    webpackOverride: (config) => config
  });

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title: articleData.english.title,
      keyPoints: articleData.english.keyPoints,
      stats: articleData.metadata.stats,
      branding: {
        logo: '/logo.png',
        colors: ['#3B82F6', '#8B5CF6']
      }
    }
  });

  const outputPath = `./output/${platform}-${Date.now()}.mp4`;

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    ...dimensions[platform]
  });

  return outputPath;
}

// Example usage
const videoPath = await renderContentVideo(article, 'reels');
console.log(`Video rendered: ${videoPath}`);
```

### 4. Complete Pipeline Workflow

```typescript
import { Pipeline } from '@/lib/pipeline';

// End-to-end content creation
async function createContentPipeline(keyword: string) {
  const pipeline = new Pipeline({
    anthropicKey: process.env.ANTHROPIC_API_KEY,
    openaiKey: process.env.OPENAI_API_KEY,
    rapidApiKey: process.env.RAPIDAPI_KEY
  });

  // Step 1: Research
  console.log('🔍 Researching...');
  const research = await pipeline.research(keyword);

  // Step 2: Generate content
  console.log('✍️ Generating content...');
  const content = await pipeline.generateContent({
    research,
    formats: ['toplist', 'how-to'],
    languages: ['en', 'vi']
  });

  // Step 3: Create visuals
  console.log('🎨 Creating visuals...');
  const visuals = await pipeline.generateVisuals(content);

  // Step 4: Render videos
  console.log('🎬 Rendering videos...');
  const videos = await pipeline.renderVideos({
    content,
    platforms: ['reels', 'tiktok', 'shorts']
  });

  // Step 5: Schedule posts (optional)
  console.log('📅 Scheduling posts...');
  await pipeline.scheduleToSocial({
    content,
    videos,
    platforms: ['facebook', 'instagram', 'tiktok'],
    scheduleTime: new Date(Date.now() + 3600000) // 1 hour from now
  });

  return {
    research,
    content,
    visuals,
    videos
  };
}

// Run the pipeline
const result = await createContentPipeline('AI video marketing 2026');
```

## Next.js API Routes

### Research Endpoint

```typescript
// pages/api/research.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { ResearchService } from '@/lib/scraper/research-service';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  const { keyword, timeRange = '24h' } = req.body;

  try {
    const researchService = new ResearchService({
      rapidApiKey: process.env.RAPIDAPI_KEY
    });

    const results = await researchService.scanTrending({
      keyword,
      timeRange,
      minEngagement: 50
    });

    res.status(200).json(results);
  } catch (error) {
    res.status(500).json({ 
      error: 'Research failed', 
      details: error.message 
    });
  }
}
```

### Content Generation Endpoint

```typescript
// pages/api/generate.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { ContentGenerator } from '@/lib/ai/content-generator';
import Anthropic from '@anthropic-ai/sdk';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  const { topic, format, tone, researchData } = req.body;

  try {
    const anthropic = new Anthropic({
      apiKey: process.env.ANTHROPIC_API_KEY
    });

    const generator = new ContentGenerator({ anthropic });

    const content = await generator.generate({
      topic,
      format,
      tone,
      languages: ['en', 'vi'],
      researchContext: researchData
    });

    res.status(200).json(content);
  } catch (error) {
    res.status(500).json({ 
      error: 'Content generation failed', 
      details: error.message 
    });
  }
}
```

## CLI Commands

```bash
# Development server
npm run dev

# Build for production
npm run build

# Start production server
npm run start

# Run research only
npm run research -- --keyword "AI marketing" --timerange 24h

# Generate content from existing research
npm run generate -- --input ./research-data.json --format toplist

# Render videos
npm run render -- --input ./content.json --platform reels

# Run complete pipeline
npm run pipeline -- --keyword "content automation" --all
```

## Common Patterns

### Custom Content Format

```typescript
import { ContentGenerator } from '@/lib/ai/content-generator';

// Define custom format
const customFormat = {
  name: 'comparison-article',
  structure: {
    intro: 'Hook and context',
    comparison_table: 'Feature comparison',
    analysis: 'Deep dive analysis',
    verdict: 'Final recommendation',
    cta: 'Call to action'
  },
  tone: 'analytical',
  length: 2000
};

// Generate with custom format
const generator = new ContentGenerator({ 
  anthropic: new Anthropic({ apiKey: process.env.ANTHROPIC_API_KEY })
});

const content = await generator.generateCustom({
  format: customFormat,
  topic: 'Claude vs OpenAI for Content Creation',
  researchData: research
});
```

### Multi-Language Support

```typescript
// Configure language-specific settings
const multiLangConfig = {
  languages: [
    {
      code: 'en',
      tone: 'professional',
      seo: { keywords: ['AI', 'automation', 'marketing'] }
    },
    {
      code: 'vi',
      tone: 'friendly',
      seo: { keywords: ['AI', 'tự động hóa', 'marketing'] }
    }
  ]
};

const content = await contentGen.generate({
  topic: 'Marketing Automation',
  format: 'how-to',
  multiLangConfig
});
```

### Video Template Customization

```typescript
// remotion/compositions/CustomVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig } from 'remotion';

export const CustomVideo: React.FC<{
  title: string;
  keyPoints: string[];
  branding: any;
}> = ({ title, keyPoints, branding }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();

  return (
    <AbsoluteFill style={{ backgroundColor: branding.colors[0] }}>
      <div style={{ 
        fontSize: 60, 
        textAlign: 'center',
        opacity: Math.min(1, frame / fps)
      }}>
        {title}
      </div>
      {/* Add more custom elements */}
    </AbsoluteFill>
  );
};
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement rate limiting
import { RateLimiter } from '@/lib/utils/rate-limiter';

const limiter = new RateLimiter({
  anthropic: { requests: 50, per: 'minute' },
  openai: { requests: 60, per: 'minute' },
  rapidapi: { requests: 100, per: 'hour' }
});

await limiter.waitForSlot('anthropic');
const response = await anthropic.messages.create({...});
```

### Video Rendering Memory Issues

```typescript
// Optimize Remotion rendering
await renderMedia({
  composition,
  serveUrl: bundleLocation,
  codec: 'h264',
  outputLocation: outputPath,
  // Memory optimizations
  concurrency: 1,
  enforceAudioTrack: false,
  muted: false,
  overwrite: true,
  chromiumOptions: {
    gl: 'swiftshader',
    headless: true
  }
});
```

### Research Data Quality

```typescript
// Filter and validate research results
function validateResearch(results: any) {
  return results.articles.filter(article => 
    article.engagement > 50 &&
    article.publishedAt > Date.now() - 86400000 && // Last 24h
    article.content.length > 200 &&
    !article.isSpam
  );
}

const validArticles = validateResearch(research);
```

### Error Handling

```typescript
// Robust error handling
async function safeGenerateContent(params: any) {
  try {
    return await contentGen.generate(params);
  } catch (error) {
    if (error.status === 429) {
      console.log('Rate limit hit, waiting...');
      await new Promise(r => setTimeout(r, 60000));
      return safeGenerateContent(params); // Retry
    }
    
    if (error.status === 500) {
      console.log('Server error, using fallback model');
      return await contentGen.generate({
        ...params,
        model: 'gpt-4o-mini' // Fallback
      });
    }
    
    throw error;
  }
}
```

## Performance Optimization

```typescript
// Parallel processing for multiple content pieces
import pLimit from 'p-limit';

const limit = pLimit(3); // Max 3 concurrent API calls

const topics = [
  'AI Marketing Tools',
  'Content Automation',
  'Video Marketing'
];

const contents = await Promise.all(
  topics.map(topic => 
    limit(() => generateContent(topic, 'toplist', research))
  )
);
```

## Best Practices

1. **Always validate research data** before content generation
2. **Use environment variables** for all API keys and secrets
3. **Implement rate limiting** to avoid API throttling
4. **Cache research results** to reduce API calls
5. **Test video renders** on small compositions first
6. **Monitor token usage** for AI providers
7. **Version control your prompts** and templates
8. **Use TypeScript strict mode** for type safety
