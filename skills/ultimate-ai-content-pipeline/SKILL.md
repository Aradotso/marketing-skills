---
name: ultimate-ai-content-pipeline
description: Automated content creation pipeline from research to video generation using Claude, OpenAI, and Remotion
triggers:
  - how do I generate automated content with AI research
  - set up content pipeline with video generation
  - automate blog posts from research to video
  - use remotion for content automation
  - create AI-powered marketing content pipeline
  - generate videos from written content automatically
  - build automated content workflow with Claude
  - scrape news and generate content with AI
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the Ultimate AI Content Pipeline, a comprehensive TypeScript-based system that automates content creation from research to video generation using Claude 3, OpenAI, and Remotion.

## What It Does

Ultimate AI Content Pipeline is an end-to-end content automation system that:

- **Auto-scans and researches** trending news from sources like TechCrunch, a16z, Twitter, and LinkedIn
- **Generates content** in multiple formats (toplist, POV, case study, how-to) using Claude/OpenAI
- **Creates bilingual content** (English and Vietnamese) with customizable tone
- **Renders videos automatically** using Remotion for social media platforms
- **Optimizes for multiple platforms** (Reels, TikTok, Shorts)

## Installation

### Prerequisites

```bash
node >= 18.0.0
npm >= 9.0.0
```

### Setup

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install

# Set up environment variables
cp .env.example .env
```

### Environment Configuration

Create a `.env` file in the root directory:

```env
# AI Provider API Keys
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research & Scraping
RAPIDAPI_KEY=your_rapidapi_key
SCRAPING_API_KEY=your_scraping_key

# Remotion Configuration
REMOTION_LICENSE_KEY=your_remotion_license

# Database (if applicable)
DATABASE_URL=postgresql://user:password@localhost:5432/content_db

# Application Settings
NODE_ENV=development
PORT=3000
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── research/          # Content research and scraping
│   ├── generation/        # AI content generation
│   ├── video/            # Remotion video rendering
│   ├── utils/            # Shared utilities
│   └── types/            # TypeScript definitions
├── public/               # Static assets
├── remotion/            # Remotion video templates
└── pages/               # Next.js pages
```

## Key Components

### 1. Research Module

Auto-scrapes content from multiple sources:

```typescript
import { ResearchService } from './src/research/ResearchService';

// Initialize research service
const research = new ResearchService({
  rapidApiKey: process.env.RAPIDAPI_KEY!,
  sources: ['techcrunch', 'a16z', 'twitter', 'linkedin']
});

// Fetch trending content
async function getTrendingContent(keyword: string) {
  const results = await research.scan({
    keyword,
    timeframe: '24h',
    maxResults: 10
  });
  
  return results.map(item => ({
    title: item.title,
    url: item.url,
    summary: item.summary,
    publishedAt: item.publishedAt,
    source: item.source
  }));
}

// Example usage
const trends = await getTrendingContent('AI marketing');
console.log(`Found ${trends.length} trending articles`);
```

### 2. Content Generation

Generate content using Claude or OpenAI:

```typescript
import { ContentGenerator } from './src/generation/ContentGenerator';
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

// Initialize generators
const claude = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY!
});

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY!
});

const generator = new ContentGenerator({ claude, openai });

// Generate content with specific format
async function generateContent(
  researchData: any[],
  format: 'toplist' | 'pov' | 'case-study' | 'how-to'
) {
  const prompt = generator.buildPrompt({
    format,
    data: researchData,
    tone: 'professional',
    language: 'bilingual' // en + vi
  });

  const content = await generator.generate({
    provider: 'claude', // or 'openai'
    model: 'claude-3-5-sonnet-20241022',
    prompt,
    maxTokens: 4000
  });

  return {
    english: content.english,
    vietnamese: content.vietnamese,
    metadata: content.metadata
  };
}

// Example: Generate POV article
const article = await generateContent(trends, 'pov');
```

### 3. Bilingual Content Generation

```typescript
interface ContentRequest {
  topic: string;
  format: ContentFormat;
  tone: 'expert' | 'friendly' | 'humorous';
  targetAudience: string;
}

async function generateBilingualContent(request: ContentRequest) {
  const systemPrompt = `
    You are a bilingual content creator specializing in ${request.format}.
    Generate content in both English and Vietnamese.
    Tone: ${request.tone}
    Target audience: ${request.targetAudience}
  `;

  const response = await claude.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4000,
    messages: [{
      role: 'user',
      content: `Create a ${request.format} article about: ${request.topic}`
    }],
    system: systemPrompt
  });

  return parseStructuredContent(response.content);
}

function parseStructuredContent(content: any) {
  // Parse AI response into structured format
  return {
    english: {
      title: '',
      body: '',
      keyPoints: []
    },
    vietnamese: {
      title: '',
      body: '',
      keyPoints: []
    }
  };
}
```

### 4. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import { ContentVideoTemplate } from './remotion/ContentVideo';

async function generateContentVideo(
  content: any,
  platform: 'reels' | 'tiktok' | 'shorts'
) {
  // Platform-specific dimensions
  const dimensions = {
    reels: { width: 1080, height: 1920 },
    tiktok: { width: 1080, height: 1920 },
    shorts: { width: 1080, height: 1920 }
  };

  const bundled = await bundle({
    entryPoint: './remotion/index.ts',
    webpackOverride: (config) => config
  });

  const composition = await selectComposition({
    serveUrl: bundled,
    id: 'ContentVideo',
    inputProps: {
      title: content.title,
      keyPoints: content.keyPoints,
      duration: 30 // seconds
    }
  });

  const outputPath = `./output/${platform}-${Date.now()}.mp4`;

  await renderMedia({
    composition,
    serveUrl: bundled,
    codec: 'h264',
    outputLocation: outputPath,
    ...dimensions[platform]
  });

  return outputPath;
}

// Generate video for multiple platforms
async function generateMultiPlatformVideos(content: any) {
  const platforms = ['reels', 'tiktok', 'shorts'] as const;
  
  const videos = await Promise.all(
    platforms.map(platform => 
      generateContentVideo(content, platform)
    )
  );

  return videos;
}
```

### 5. Complete Pipeline Workflow

```typescript
import { ContentPipeline } from './src/ContentPipeline';

// Initialize the complete pipeline
const pipeline = new ContentPipeline({
  anthropicKey: process.env.ANTHROPIC_API_KEY!,
  openaiKey: process.env.OPENAI_API_KEY!,
  rapidApiKey: process.env.RAPIDAPI_KEY!,
  remotionLicense: process.env.REMOTION_LICENSE_KEY!
});

// Execute full content creation workflow
async function createContentFromKeyword(keyword: string) {
  console.log(`Starting pipeline for: ${keyword}`);

  // Step 1: Research
  const research = await pipeline.research({
    keyword,
    sources: ['techcrunch', 'twitter', 'linkedin'],
    timeframe: '24h'
  });

  console.log(`Found ${research.length} relevant articles`);

  // Step 2: Generate content
  const content = await pipeline.generate({
    research,
    format: 'toplist',
    tone: 'professional',
    bilingual: true
  });

  console.log('Content generated successfully');

  // Step 3: Create visuals
  const videos = await pipeline.renderVideos({
    content,
    platforms: ['reels', 'tiktok', 'shorts']
  });

  console.log(`Generated ${videos.length} videos`);

  return {
    content,
    videos,
    metadata: {
      keyword,
      generatedAt: new Date(),
      wordCount: content.english.body.split(' ').length
    }
  };
}

// Usage
const result = await createContentFromKeyword('AI automation');
console.log('Pipeline complete:', result.metadata);
```

## API Usage Patterns

### Custom Content Formats

```typescript
// Define custom content format
interface CustomFormat {
  sections: Array<{
    type: 'intro' | 'body' | 'conclusion';
    content: string;
    bullets?: string[];
  }>;
  callToAction?: string;
}

async function generateCustomFormat(
  topic: string,
  template: CustomFormat
) {
  const prompt = `
    Create content following this structure:
    ${JSON.stringify(template, null, 2)}
    
    Topic: ${topic}
  `;

  const response = await claude.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 3000,
    messages: [{ role: 'user', content: prompt }]
  });

  return response.content;
}
```

### Batch Processing

```typescript
async function processBatchKeywords(keywords: string[]) {
  const results = [];

  for (const keyword of keywords) {
    try {
      const result = await pipeline.execute({
        keyword,
        format: 'toplist',
        generateVideo: true
      });
      
      results.push({
        keyword,
        status: 'success',
        result
      });
    } catch (error) {
      results.push({
        keyword,
        status: 'error',
        error: error.message
      });
    }
  }

  return results;
}

// Process multiple keywords
const keywords = ['AI tools', 'marketing automation', 'content strategy'];
const batchResults = await processBatchKeywords(keywords);
```

## Configuration

### Customize Research Sources

```typescript
// src/config/sources.ts
export const researchSources = {
  techcrunch: {
    url: 'https://techcrunch.com/wp-json/wp/v2/posts',
    selector: '.article-content',
    rateLimit: 10 // requests per minute
  },
  twitter: {
    apiEndpoint: 'https://api.twitter.com/2/tweets/search/recent',
    maxResults: 100
  },
  linkedin: {
    apiEndpoint: 'https://api.linkedin.com/v2/shares',
    maxResults: 50
  }
};
```

### Video Template Customization

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, Sequence, useCurrentFrame } from 'remotion';

export const ContentVideo: React.FC<{
  title: string;
  keyPoints: string[];
  duration: number;
}> = ({ title, keyPoints, duration }) => {
  const frame = useCurrentFrame();

  return (
    <AbsoluteFill style={{ backgroundColor: '#000' }}>
      <Sequence from={0} durationInFrames={30}>
        <h1 style={{ color: '#fff', fontSize: 48 }}>
          {title}
        </h1>
      </Sequence>
      
      {keyPoints.map((point, index) => (
        <Sequence
          key={index}
          from={30 + index * 60}
          durationInFrames={60}
        >
          <p style={{ color: '#fff', fontSize: 32 }}>
            • {point}
          </p>
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};
```

## Running the Application

### Development Mode

```bash
# Start Next.js dev server
npm run dev

# Access at http://localhost:3000
```

### Production Build

```bash
# Build for production
npm run build

# Start production server
npm start
```

### CLI Usage (if available)

```bash
# Generate content from CLI
npm run generate -- --keyword "AI marketing" --format toplist

# Render video only
npm run render-video -- --input ./content.json --platform reels
```

## Common Workflows

### Daily Content Generation

```typescript
import { schedule } from 'node-cron';

// Schedule daily content generation
schedule('0 9 * * *', async () => {
  const todayKeyword = getTrendingKeyword(); // Your logic
  
  const content = await pipeline.execute({
    keyword: todayKeyword,
    format: 'pov',
    generateVideo: true,
    autoPublish: false // Review before publishing
  });

  // Save to database or file system
  await saveContent(content);
  
  console.log(`Daily content generated for: ${todayKeyword}`);
});
```

### A/B Testing Content Formats

```typescript
async function testContentFormats(keyword: string) {
  const formats = ['toplist', 'pov', 'case-study', 'how-to'] as const;
  
  const variants = await Promise.all(
    formats.map(format => 
      pipeline.generate({
        keyword,
        format,
        bilingual: true
      })
    )
  );

  return formats.map((format, index) => ({
    format,
    content: variants[index],
    metrics: {
      wordCount: variants[index].english.body.split(' ').length,
      estimatedReadTime: Math.ceil(
        variants[index].english.body.split(' ').length / 200
      )
    }
  }));
}
```

## Troubleshooting

### API Rate Limits

```typescript
import { RateLimiter } from './src/utils/RateLimiter';

const limiter = new RateLimiter({
  maxRequests: 10,
  perMinutes: 1
});

async function safeApiCall<T>(
  fn: () => Promise<T>
): Promise<T> {
  await limiter.acquire();
  
  try {
    return await fn();
  } catch (error) {
    if (error.status === 429) {
      console.log('Rate limit hit, waiting...');
      await new Promise(resolve => setTimeout(resolve, 60000));
      return safeApiCall(fn);
    }
    throw error;
  }
}
```

### Video Rendering Issues

```typescript
// Check Remotion configuration
async function validateRemotionSetup() {
  try {
    const { getCompositions } = await import('@remotion/renderer');
    const compositions = await getCompositions('./remotion/index.ts');
    console.log('Available compositions:', compositions);
    return true;
  } catch (error) {
    console.error('Remotion setup error:', error);
    return false;
  }
}

// Ensure sufficient resources
const renderConfig = {
  concurrency: 1, // Reduce if out of memory
  enforceAudioTrack: false,
  pixelFormat: 'yuv420p',
  codec: 'h264',
  videoBitrate: '2M'
};
```

### Content Quality Validation

```typescript
function validateContent(content: any): boolean {
  const checks = {
    hasTitle: content.title && content.title.length > 10,
    hasBody: content.body && content.body.length > 500,
    hasBilingual: content.vietnamese && content.english,
    hasKeyPoints: content.keyPoints && content.keyPoints.length >= 3
  };

  const passed = Object.values(checks).every(Boolean);
  
  if (!passed) {
    console.error('Content validation failed:', checks);
  }

  return passed;
}
```

### Error Handling

```typescript
class PipelineError extends Error {
  constructor(
    message: string,
    public stage: 'research' | 'generation' | 'video',
    public details?: any
  ) {
    super(message);
    this.name = 'PipelineError';
  }
}

async function executeWithErrorHandling(keyword: string) {
  try {
    return await pipeline.execute({ keyword });
  } catch (error) {
    if (error instanceof PipelineError) {
      console.error(`Failed at ${error.stage}:`, error.message);
      console.error('Details:', error.details);
      
      // Implement retry or fallback logic
      if (error.stage === 'research') {
        console.log('Retrying with cached data...');
        return await pipeline.executeWithCache({ keyword });
      }
    }
    throw error;
  }
}
```

## Best Practices

1. **Always use environment variables** for API keys and secrets
2. **Implement rate limiting** for external API calls
3. **Validate content quality** before video generation
4. **Cache research results** to avoid redundant API calls
5. **Monitor AI token usage** to control costs
6. **Test video templates** before batch rendering
7. **Use TypeScript strict mode** for type safety
8. **Implement logging** for pipeline monitoring

## Performance Optimization

```typescript
// Cache research results
import NodeCache from 'node-cache';

const cache = new NodeCache({ stdTTL: 3600 }); // 1 hour

async function getCachedResearch(keyword: string) {
  const cacheKey = `research:${keyword}`;
  const cached = cache.get(cacheKey);
  
  if (cached) {
    console.log('Using cached research');
    return cached;
  }

  const fresh = await research.scan({ keyword });
  cache.set(cacheKey, fresh);
  return fresh;
}
```

This skill provides comprehensive guidance for AI coding agents to effectively use the Ultimate AI Content Pipeline for automated content creation and marketing workflows.
