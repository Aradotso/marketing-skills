---
name: marketing-pipeline-share-ai-content
description: Automate content creation from research to video generation using AI (Claude/OpenAI) with auto-crawling, multi-format writing, and Remotion video rendering
triggers:
  - create automated content pipeline with AI
  - generate marketing content from research to video
  - automate content workflow with Claude and OpenAI
  - build AI-powered content generation system
  - crawl news and generate social media content
  - create videos from written content automatically
  - set up marketing automation pipeline
  - generate multilingual content with AI research
---

# Marketing Pipeline Share - AI Content Automation Skill

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

Marketing Pipeline Share is a complete AI-powered content automation system that transforms keywords into finished content assets. It automatically researches trending topics by crawling sources like TechCrunch, a16z, Twitter/X, and LinkedIn, then generates multilingual articles (English/Vietnamese) in multiple formats (Toplist, POV, Case Study, How-to), and finally renders videos using Remotion.

**Key capabilities:**
- Auto-scan research from real-time news sources
- Multi-format content generation with Claude 3 or OpenAI
- Automatic video/infographic rendering with Remotion
- Bilingual output (English/Vietnamese)
- Multiple content tones (expert, friendly, humorous)
- Platform-optimized exports (Reels, TikTok, Shorts)

## Installation

### Prerequisites

```bash
node >= 18.0.0
npm or yarn
```

### Setup

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
yarn install

# Set up environment variables
cp .env.example .env
```

### Environment Configuration

Create a `.env` file with the following variables:

```bash
# AI Provider Keys
ANTHROPIC_API_KEY=your_claude_key_here
OPENAI_API_KEY=your_openai_key_here

# RapidAPI for news crawling
RAPIDAPI_KEY=your_rapidapi_key_here

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Remotion rendering
REMOTION_AWS_ACCESS_KEY_ID=your_aws_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret

# Next.js
NEXT_PUBLIC_API_URL=http://localhost:3000
```

### Running the Application

```bash
# Development mode
npm run dev
# or
yarn dev

# Production build
npm run build
npm start
```

## Core Architecture

The pipeline consists of 4 main stages:

1. **Research Module**: Crawls and aggregates content
2. **Content Generation Module**: AI writing with Claude/OpenAI
3. **Video Rendering Module**: Remotion-based video creation
4. **Publishing Module**: Scheduling and distribution

## Research & Crawling Module

### Basic News Crawling

```typescript
import { NewsCrawler } from './lib/crawlers/news-crawler';

interface CrawlConfig {
  sources: string[];
  timeframe: '24h' | '7d' | '30d';
  keywords: string[];
  maxResults: number;
}

async function researchTopic(keyword: string): Promise<ResearchData> {
  const crawler = new NewsCrawler({
    apiKey: process.env.RAPIDAPI_KEY
  });

  const config: CrawlConfig = {
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeframe: '24h',
    keywords: [keyword],
    maxResults: 50
  };

  const results = await crawler.scan(config);
  
  return {
    articles: results.articles,
    insights: await crawler.extractInsights(results),
    trends: await crawler.analyzeTrends(results),
    dataPoints: results.statistics
  };
}
```

### Advanced Crawling with Filtering

```typescript
import { ContentFilter } from './lib/crawlers/filters';

async function crawlWithFilters(keyword: string) {
  const crawler = new NewsCrawler({
    apiKey: process.env.RAPIDAPI_KEY
  });

  const filter = new ContentFilter({
    minEngagement: 100,
    excludeDomains: ['spam-site.com'],
    language: ['en', 'vi'],
    contentType: ['article', 'video'],
    sentiment: ['positive', 'neutral']
  });

  const rawResults = await crawler.scan({
    sources: ['techcrunch', 'twitter'],
    timeframe: '24h',
    keywords: [keyword],
    maxResults: 100
  });

  const filtered = filter.apply(rawResults);
  
  return filtered;
}
```

## Content Generation Module

### Generate Multi-Format Content

```typescript
import { ContentGenerator } from './lib/ai/content-generator';
import { ContentFormat, Language, Tone } from './types';

interface GenerationConfig {
  format: ContentFormat;
  language: Language;
  tone: Tone;
  targetAudience: string;
  wordCount: number;
}

async function generateContent(
  researchData: ResearchData,
  config: GenerationConfig
) {
  const generator = new ContentGenerator({
    provider: 'claude', // or 'openai'
    apiKey: process.env.ANTHROPIC_API_KEY,
    model: 'claude-3-opus-20240229'
  });

  const content = await generator.create({
    topic: researchData.mainTopic,
    insights: researchData.insights,
    format: config.format, // 'toplist' | 'pov' | 'case-study' | 'how-to'
    language: config.language, // 'en' | 'vi'
    tone: config.tone, // 'expert' | 'friendly' | 'humorous'
    targetAudience: config.targetAudience,
    wordCount: config.wordCount,
    includeData: true,
    citations: researchData.sources
  });

  return content;
}
```

### Bilingual Content Generation

```typescript
async function generateBilingualContent(researchData: ResearchData) {
  const generator = new ContentGenerator({
    provider: 'claude',
    apiKey: process.env.ANTHROPIC_API_KEY
  });

  // Generate both languages in parallel
  const [englishContent, vietnameseContent] = await Promise.all([
    generator.create({
      topic: researchData.mainTopic,
      insights: researchData.insights,
      format: 'toplist',
      language: 'en',
      tone: 'expert',
      wordCount: 1500
    }),
    generator.create({
      topic: researchData.mainTopic,
      insights: researchData.insights,
      format: 'toplist',
      language: 'vi',
      tone: 'friendly',
      wordCount: 1500
    })
  ]);

  return {
    english: englishContent,
    vietnamese: vietnameseContent,
    metadata: {
      generatedAt: new Date(),
      model: generator.model,
      sources: researchData.sources.length
    }
  };
}
```

### Using OpenAI Provider

```typescript
import { OpenAIGenerator } from './lib/ai/openai-generator';

async function generateWithOpenAI(researchData: ResearchData) {
  const generator = new OpenAIGenerator({
    apiKey: process.env.OPENAI_API_KEY,
    model: 'gpt-4-turbo-preview',
    temperature: 0.7,
    maxTokens: 4000
  });

  const content = await generator.create({
    topic: researchData.mainTopic,
    format: 'case-study',
    language: 'en',
    tone: 'expert',
    systemPrompt: `You are an expert content strategist specializing in B2B marketing.`,
    additionalContext: researchData.insights
  });

  return content;
}
```

## Video Rendering Module (Remotion)

### Basic Video Generation

```typescript
import { renderVideo } from './lib/video/renderer';
import { VideoComposition } from './remotion/compositions';

interface VideoConfig {
  composition: string;
  inputProps: any;
  outputPath: string;
  format: 'mp4' | 'webm';
  fps: number;
  resolution: [number, number];
}

async function createContentVideo(content: GeneratedContent) {
  const config: VideoConfig = {
    composition: 'ContentVideo',
    inputProps: {
      title: content.title,
      keyPoints: content.keyPoints,
      images: content.images,
      branding: {
        logo: '/assets/logo.png',
        colors: ['#FF6B6B', '#4ECDC4']
      }
    },
    outputPath: `./output/${content.id}.mp4`,
    format: 'mp4',
    fps: 30,
    resolution: [1080, 1920] // Vertical for Reels/TikTok
  };

  const video = await renderVideo(config);
  
  return video;
}
```

### Platform-Specific Video Formats

```typescript
import { PlatformOptimizer } from './lib/video/platform-optimizer';

async function generatePlatformVideos(content: GeneratedContent) {
  const optimizer = new PlatformOptimizer();

  // Generate for multiple platforms
  const videos = await Promise.all([
    // Instagram Reels (1080x1920, 9:16)
    optimizer.render({
      content,
      platform: 'reels',
      resolution: [1080, 1920],
      duration: 60,
      includeSubtitles: true,
      audienceRetention: 'high'
    }),

    // TikTok (1080x1920, 9:16)
    optimizer.render({
      content,
      platform: 'tiktok',
      resolution: [1080, 1920],
      duration: 60,
      includeSubtitles: true,
      trendingEffects: true
    }),

    // YouTube Shorts (1080x1920, 9:16)
    optimizer.render({
      content,
      platform: 'shorts',
      resolution: [1080, 1920],
      duration: 60,
      includeSubtitles: true,
      endScreen: true
    }),

    // LinkedIn (1920x1080, 16:9)
    optimizer.render({
      content,
      platform: 'linkedin',
      resolution: [1920, 1080],
      duration: 120,
      includeSubtitles: true,
      professionalStyle: true
    })
  ]);

  return videos;
}
```

### Custom Remotion Composition

```typescript
// remotion/compositions/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig } from 'remotion';

export const ContentVideo: React.FC<{
  title: string;
  keyPoints: string[];
  images: string[];
  branding: any;
}> = ({ title, keyPoints, images, branding }) => {
  const frame = useCurrentFrame();
  const { fps, durationInFrames } = useVideoConfig();

  const opacity = Math.min(1, frame / 30);

  return (
    <AbsoluteFill style={{ backgroundColor: branding.colors[0] }}>
      {/* Title Section (0-2 seconds) */}
      {frame < fps * 2 && (
        <div style={{ opacity }}>
          <h1>{title}</h1>
        </div>
      )}

      {/* Key Points Section (2-10 seconds) */}
      {frame >= fps * 2 && frame < fps * 10 && (
        <div>
          {keyPoints.map((point, index) => {
            const pointFrame = frame - fps * (2 + index * 2);
            const pointOpacity = Math.min(1, pointFrame / 20);
            
            return (
              <div key={index} style={{ opacity: pointOpacity }}>
                <p>{point}</p>
              </div>
            );
          })}
        </div>
      )}

      {/* Call to Action (last 2 seconds) */}
      {frame >= durationInFrames - fps * 2 && (
        <div>
          <p>Learn more at our website</p>
        </div>
      )}
    </AbsoluteFill>
  );
};
```

## Complete Pipeline Orchestration

### Full Workflow Automation

```typescript
import { ContentPipeline } from './lib/pipeline';

async function runCompletePipeline(keyword: string) {
  const pipeline = new ContentPipeline({
    research: {
      sources: ['techcrunch', 'a16z', 'twitter'],
      timeframe: '24h'
    },
    generation: {
      provider: 'claude',
      apiKey: process.env.ANTHROPIC_API_KEY,
      languages: ['en', 'vi'],
      formats: ['toplist', 'how-to']
    },
    video: {
      platforms: ['reels', 'tiktok', 'shorts'],
      quality: 'high'
    },
    publishing: {
      schedule: true,
      platforms: ['facebook', 'instagram', 'linkedin']
    }
  });

  // Execute entire pipeline
  const result = await pipeline.execute(keyword);

  return {
    research: result.researchData,
    content: result.generatedContent, // Array of content in different formats
    videos: result.renderedVideos, // Videos for each platform
    publishStatus: result.publishingResults,
    analytics: result.pipelineAnalytics
  };
}
```

### Scheduled Pipeline Execution

```typescript
import { CronJob } from 'cron';
import { PipelineScheduler } from './lib/scheduler';

function setupAutomation() {
  const scheduler = new PipelineScheduler();

  // Daily content generation at 6 AM
  scheduler.schedule({
    name: 'daily-content',
    cron: '0 6 * * *',
    timezone: 'Asia/Ho_Chi_Minh',
    pipeline: {
      keywords: ['AI marketing', 'content automation', 'social media trends'],
      research: { timeframe: '24h' },
      generation: { 
        formats: ['toplist', 'pov'],
        languages: ['en', 'vi']
      },
      video: { platforms: ['reels', 'tiktok'] }
    }
  });

  // Weekly deep-dive content on Mondays
  scheduler.schedule({
    name: 'weekly-deep-dive',
    cron: '0 8 * * 1',
    timezone: 'Asia/Ho_Chi_Minh',
    pipeline: {
      keywords: ['marketing strategy', 'growth hacking'],
      research: { timeframe: '7d', maxResults: 100 },
      generation: { 
        formats: ['case-study'],
        languages: ['en', 'vi'],
        wordCount: 3000
      },
      video: { platforms: ['youtube', 'linkedin'] }
    }
  });

  scheduler.start();
}
```

## API Routes (Next.js)

### Research API

```typescript
// pages/api/research.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { NewsCrawler } from '@/lib/crawlers/news-crawler';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  const { keyword, sources, timeframe } = req.body;

  try {
    const crawler = new NewsCrawler({
      apiKey: process.env.RAPIDAPI_KEY
    });

    const results = await crawler.scan({
      sources: sources || ['techcrunch', 'twitter'],
      timeframe: timeframe || '24h',
      keywords: [keyword],
      maxResults: 50
    });

    return res.status(200).json({
      success: true,
      data: results,
      timestamp: new Date().toISOString()
    });
  } catch (error) {
    return res.status(500).json({
      success: false,
      error: error.message
    });
  }
}
```

### Content Generation API

```typescript
// pages/api/generate.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { ContentGenerator } from '@/lib/ai/content-generator';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  const { researchData, format, language, tone } = req.body;

  try {
    const generator = new ContentGenerator({
      provider: 'claude',
      apiKey: process.env.ANTHROPIC_API_KEY
    });

    const content = await generator.create({
      topic: researchData.mainTopic,
      insights: researchData.insights,
      format,
      language,
      tone,
      wordCount: 1500
    });

    return res.status(200).json({
      success: true,
      content,
      metadata: {
        generatedAt: new Date().toISOString(),
        format,
        language,
        wordCount: content.split(' ').length
      }
    });
  } catch (error) {
    return res.status(500).json({
      success: false,
      error: error.message
    });
  }
}
```

### Video Rendering API

```typescript
// pages/api/render-video.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { renderVideo } from '@/lib/video/renderer';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  const { content, platform } = req.body;

  try {
    const video = await renderVideo({
      composition: 'ContentVideo',
      inputProps: {
        title: content.title,
        keyPoints: content.keyPoints,
        images: content.images
      },
      outputPath: `./output/${content.id}-${platform}.mp4`,
      format: 'mp4',
      fps: 30,
      resolution: platform === 'linkedin' ? [1920, 1080] : [1080, 1920]
    });

    return res.status(200).json({
      success: true,
      videoUrl: video.url,
      duration: video.duration,
      size: video.size
    });
  } catch (error) {
    return res.status(500).json({
      success: false,
      error: error.message
    });
  }
}
```

## Common Patterns

### Error Handling and Retry Logic

```typescript
import { retry } from './lib/utils/retry';

async function robustContentGeneration(researchData: ResearchData) {
  const generator = new ContentGenerator({
    provider: 'claude',
    apiKey: process.env.ANTHROPIC_API_KEY
  });

  try {
    // Retry up to 3 times with exponential backoff
    const content = await retry(
      () => generator.create({
        topic: researchData.mainTopic,
        insights: researchData.insights,
        format: 'toplist',
        language: 'en'
      }),
      {
        maxAttempts: 3,
        delay: 1000,
        backoff: 2,
        onRetry: (attempt, error) => {
          console.log(`Retry attempt ${attempt} due to: ${error.message}`);
        }
      }
    );

    return content;
  } catch (error) {
    // Fallback to OpenAI if Claude fails
    console.log('Claude failed, falling back to OpenAI');
    
    const openaiGenerator = new OpenAIGenerator({
      apiKey: process.env.OPENAI_API_KEY
    });

    return await openaiGenerator.create({
      topic: researchData.mainTopic,
      format: 'toplist',
      language: 'en'
    });
  }
}
```

### Content Quality Validation

```typescript
import { ContentValidator } from './lib/validation/content-validator';

async function generateValidatedContent(researchData: ResearchData) {
  const generator = new ContentGenerator({
    provider: 'claude',
    apiKey: process.env.ANTHROPIC_API_KEY
  });

  const validator = new ContentValidator({
    minWordCount: 1000,
    maxWordCount: 2500,
    requiredSections: ['introduction', 'main-points', 'conclusion'],
    checkGrammar: true,
    checkPlagiarism: true,
    checkFactAccuracy: true
  });

  let content = await generator.create({
    topic: researchData.mainTopic,
    insights: researchData.insights,
    format: 'toplist',
    language: 'en'
  });

  const validation = await validator.validate(content);

  if (!validation.passed) {
    console.log('Content validation failed:', validation.issues);
    
    // Regenerate with specific improvements
    content = await generator.create({
      topic: researchData.mainTopic,
      insights: researchData.insights,
      format: 'toplist',
      language: 'en',
      improvements: validation.suggestions
    });
  }

  return {
    content,
    validation,
    qualityScore: validation.score
  };
}
```

### Batch Processing Multiple Keywords

```typescript
async function batchGenerateContent(keywords: string[]) {
  const results = [];

  // Process in batches of 3 to avoid rate limits
  const batchSize = 3;
  
  for (let i = 0; i < keywords.length; i += batchSize) {
    const batch = keywords.slice(i, i + batchSize);
    
    const batchResults = await Promise.all(
      batch.map(async (keyword) => {
        try {
          // Research
          const research = await researchTopic(keyword);
          
          // Generate content in both languages
          const content = await generateBilingualContent(research);
          
          // Render videos
          const videos = await generatePlatformVideos(content.english);
          
          return {
            keyword,
            success: true,
            content,
            videos
          };
        } catch (error) {
          return {
            keyword,
            success: false,
            error: error.message
          };
        }
      })
    );

    results.push(...batchResults);
    
    // Wait 2 seconds between batches
    if (i + batchSize < keywords.length) {
      await new Promise(resolve => setTimeout(resolve, 2000));
    }
  }

  return results;
}
```

## Configuration Files

### Pipeline Configuration

```typescript
// config/pipeline.config.ts
export const pipelineConfig = {
  research: {
    defaultSources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    defaultTimeframe: '24h',
    maxResults: 50,
    filters: {
      minEngagement: 100,
      languages: ['en', 'vi'],
      contentTypes: ['article', 'video', 'tweet']
    }
  },
  
  generation: {
    defaultProvider: 'claude',
    fallbackProvider: 'openai',
    models: {
      claude: 'claude-3-opus-20240229',
      openai: 'gpt-4-turbo-preview'
    },
    defaults: {
      temperature: 0.7,
      maxTokens: 4000,
      topP: 0.9
    }
  },
  
  video: {
    defaultFps: 30,
    defaultQuality: 'high',
    platforms: {
      reels: { resolution: [1080, 1920], maxDuration: 60 },
      tiktok: { resolution: [1080, 1920], maxDuration: 60 },
      shorts: { resolution: [1080, 1920], maxDuration: 60 },
      linkedin: { resolution: [1920, 1080], maxDuration: 120 }
    },
    rendering: {
      concurrency: 2,
      timeout: 300000 // 5 minutes
    }
  },
  
  storage: {
    outputPath: './output',
    videoPath: './output/videos',
    contentPath: './output/content',
    tempPath: './temp'
  }
};
```

## Troubleshooting

### API Rate Limits

```typescript
import { RateLimiter } from './lib/utils/rate-limiter';

// Implement rate limiting for API calls
const limiter = new RateLimiter({
  claude: { requestsPerMinute: 20, tokensPerMinute: 100000 },
  openai: { requestsPerMinute: 60, tokensPerMinute: 150000 },
  rapidapi: { requestsPerMinute: 100 }
});

async function generateWithRateLimit(researchData: ResearchData) {
  await limiter.waitForSlot('claude');
  
  const generator = new ContentGenerator({
    provider: 'claude',
    apiKey: process.env.ANTHROPIC_API_KEY
  });

  return await generator.create({
    topic: researchData.mainTopic,
    format: 'toplist',
    language: 'en'
  });
}
```

### Video Rendering Errors

```typescript
// Handle common Remotion rendering issues
async function safeVideoRender(content: GeneratedContent) {
  try {
    const video = await renderVideo({
      composition: 'ContentVideo',
      inputProps: { ...content },
      outputPath: `./output/${content.id}.mp4`,
      format: 'mp4',
      fps: 30,
      resolution: [1080, 1920]
    });

    return video;
  } catch (error) {
    if (error.message.includes('ENOMEM')) {
      // Memory error - reduce resolution
      console.log('Memory error, reducing resolution');
      return await renderVideo({
        composition: 'ContentVideo',
        inputProps: { ...content },
        outputPath: `./output/${content.id}.mp4`,
        format: 'mp4',
        fps: 24,
        resolution: [720, 1280]
      });
    }
    
    if (error.message.includes('timeout')) {
      // Timeout - increase timeout value
      console.log('Rendering timeout, retrying with extended timeout');
      return await renderVideo({
        composition: 'ContentVideo',
        inputProps: { ...content },
        outputPath: `./output/${content.id}.mp4`,
        format: 'mp4',
        fps: 30,
        resolution: [1080, 1920],
        timeout: 600000 // 10 minutes
      });
    }

    throw error;
  }
}
```

### Missing Environment Variables

```typescript
// Validate environment variables on startup
function validateEnvironment() {
  const required = [
    'ANTHROPIC_API_KEY',
    'OPENAI_API_KEY',
    'RAPIDAPI_KEY'
  ];

  const missing = required.filter(key => !process.env[key]);

  if (missing.length > 0) {
    throw new Error(
      `Missing required environment variables: ${missing.join(', ')}\n` +
      `Please check your .env file and ensure all required keys are set.`
    );
  }

  console.log('✓ All environment variables validated');
}

// Call at app startup
validateEnvironment();
```

### Content Quality Issues

```typescript
// Debug and improve content quality
async function debugContentGeneration(researchData: ResearchData) {
  const generator = new ContentGenerator({
    provider: 'claude',
    apiKey: process.env.ANTHROPIC_API_KEY,
    debug: true // Enable debug mode
  });

  const content = await generator.create({
    topic: researchData.mainTopic,
    insights: researchData.insights,
    format: 'toplist',
    language: 'en',
    // Add explicit instructions for better quality
    additionalInstructions: [
      'Include specific data points and statistics',
      'Use concrete examples from the research',
      'Maintain a consistent tone throughout',
      'Add actionable takeaways in each section'
    ]
  });

  // Log generation metadata
  console.log('Generation metadata:', {
    tokensUsed: content.metadata.tokensUsed,
    model: content.metadata.model,
    processingTime: content.metadata.processingTime,
    qualityScore: content.metadata.qualityScore
  });

  return content;
}
```

## Performance Optimization

### Caching Research Results

```typescript
import { CacheManager } from './lib/cache/cache-manager';

const cache = new CacheManager({
  ttl: 3600, // 1 hour
  storage: 'redis' // or 'memory'
});

async function cachedResearch(keyword: string) {
  const cacheKey = `research:${keyword}:24h`;
  
  // Check cache first
  const cached = await cache.get(cacheKey);
  if (cached) {
    console.log('Using cached research data');
    return cached;
  }

  // If not cached, fetch new data
  const research = await researchTopic(keyword);
  
  // Store in cache
  await cache.set(cacheKey, research);
  
  return research;
}
```

### Parallel Processing

```typescript
async function optimizedPipeline(keyword: string) {
  // Step 1: Research (sequential)
  const research = await researchTopic(keyword);

  // Step 2: Generate content in parallel for multiple formats/languages
  const contentPromises = [
    generateContent(research, { format: 'toplist', language: 'en', tone: 'expert', targetAudience: 'marketers', wordCount: 1500 }),
    generateContent(research, { format: 'toplist', language: 'vi', tone: 'friendly', targetAudience: 'marketers', wordCount: 1500 }),
    generateContent(research, { format: 'how-to', language: 'en', tone: 'friendly', targetAudience: 'beginners', wordCount: 2000 })
  ];

  const contents = await Promise.all(contentPromises);

  // Step 3: Render videos in parallel for all content pieces
  const videoPromises = contents.flatMap(content =>
    ['reels', 'tiktok', 'shorts'].map(platform =>
      renderVideo({
        composition: 'ContentVideo',
        inputProps: content,
        outputPath: `./output/${content.id}-${platform}.mp4`,
        format: 'mp4',
        fps: 30,
        resolution: [1080, 1920]
      })
    )
  );

  const videos = await Promise.all(videoPromises);

  return { research, contents, videos };
}
```

This skill covers the complete Marketing Pipeline Share system for automated content creation from research to video generation, with practical TypeScript examples ready for AI coding agents to help developers implement and customize.
