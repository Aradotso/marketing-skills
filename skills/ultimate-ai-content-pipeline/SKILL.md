---
name: ultimate-ai-content-pipeline
description: Automated content creation pipeline from research to video generation using Claude, OpenAI, and Remotion
triggers:
  - how do I set up the AI content pipeline system
  - generate automated content with AI research and video
  - create content from keyword to video automatically
  - use Claude and OpenAI for content automation
  - set up automated content research and video generation
  - build an AI-powered content marketing pipeline
  - automate content creation with research and video rendering
  - integrate Remotion for automated video generation
---

# Ultimate AI Content Pipeline Skill

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## What This Project Does

Ultimate AI Content Pipeline is a comprehensive TypeScript-based content automation system that:

- **Auto-researches** trending topics by crawling sources like TechCrunch, a16z, Twitter/X, and LinkedIn
- **Generates content** in multiple formats (listicles, POV, case studies, how-tos) using Claude 3 and OpenAI
- **Creates bilingual content** (English & Vietnamese) with customizable tones
- **Renders videos automatically** using Remotion to convert written content into infographics and short-form videos
- **Optimizes for multi-platform** (Reels, TikTok, Shorts)

## Installation

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

## Environment Configuration

Create a `.env` file in the project root:

```bash
# AI Services
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Video Rendering
REMOTION_LICENSE_KEY=your_remotion_key

# Database (if applicable)
DATABASE_URL=postgresql://user:password@localhost:5432/content_db

# Next.js Config
NEXT_PUBLIC_API_URL=http://localhost:3000
```

## Key Commands

### Development

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Run video rendering
npm run render
```

### Content Generation

```bash
# Generate content from keyword
npm run generate -- --keyword "AI trends" --format "listicle"

# Research and analyze
npm run research -- --sources "techcrunch,twitter" --days 1

# Render video from content
npm run render:video -- --content-id 12345
```

## Core API Usage

### Content Research Module

```typescript
import { ContentResearcher } from '@/lib/research';
import { AnthropicClient } from '@/lib/ai/anthropic';

const researcher = new ContentResearcher({
  apiKey: process.env.RAPIDAPI_KEY,
  sources: ['techcrunch', 'a16z', 'twitter'],
  timeRange: 24 // hours
});

async function researchTopic(keyword: string) {
  // Crawl and aggregate data
  const rawData = await researcher.crawl(keyword);
  
  // Extract insights using Claude
  const claude = new AnthropicClient(process.env.ANTHROPIC_API_KEY);
  const insights = await claude.analyzeData({
    data: rawData,
    prompt: `Extract key insights and trends about ${keyword}`,
    model: 'claude-3-opus-20240229'
  });
  
  return insights;
}
```

### Content Generation with Multiple Formats

```typescript
import { ContentGenerator } from '@/lib/generator';
import { OpenAIClient } from '@/lib/ai/openai';

const generator = new ContentGenerator({
  openai: new OpenAIClient(process.env.OPENAI_API_KEY),
  anthropic: new AnthropicClient(process.env.ANTHROPIC_API_KEY)
});

async function generateContent(keyword: string, format: string) {
  const content = await generator.create({
    keyword,
    format, // 'listicle' | 'pov' | 'case-study' | 'how-to'
    languages: ['en', 'vi'],
    tone: 'expert', // 'expert' | 'friendly' | 'humorous'
    includeData: true
  });
  
  return {
    english: content.en,
    vietnamese: content.vi,
    metadata: content.meta
  };
}
```

### Bilingual Content Generation

```typescript
import { BilingualWriter } from '@/lib/writer';

const writer = new BilingualWriter({
  primaryAI: 'claude', // or 'openai'
  translationStrategy: 'parallel' // generate both simultaneously
});

async function createBilingualPost(research: any) {
  const post = await writer.generate({
    research,
    formats: {
      en: {
        template: 'tech-article',
        tone: 'professional',
        length: 1500
      },
      vi: {
        template: 'tech-article',
        tone: 'friendly',
        length: 1500
      }
    }
  });
  
  return post;
}
```

### Video Rendering with Remotion

```typescript
import { renderMedia } from '@remotion/renderer';
import { bundle } from '@remotion/bundler';
import path from 'path';

async function renderContentVideo(contentId: string, outputPath: string) {
  // Get content data
  const content = await getContentById(contentId);
  
  // Bundle Remotion composition
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./src/video/index.tsx'),
    webpackOverride: (config) => config
  });
  
  // Render video
  await renderMedia({
    composition: {
      id: 'ContentVideo',
      durationInFrames: 300,
      fps: 30,
      width: 1080,
      height: 1920 // 9:16 for Reels/TikTok
    },
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: {
      title: content.title,
      points: content.keyPoints,
      brand: content.brandConfig
    }
  });
  
  return outputPath;
}
```

### Complete Pipeline Example

```typescript
import { ContentPipeline } from '@/lib/pipeline';

const pipeline = new ContentPipeline({
  research: {
    sources: ['techcrunch', 'twitter', 'linkedin'],
    timeRange: 24
  },
  generation: {
    ai: 'claude',
    fallback: 'openai'
  },
  video: {
    enabled: true,
    platforms: ['reels', 'tiktok', 'shorts']
  }
});

async function runFullPipeline(keyword: string) {
  try {
    // Step 1: Research
    const research = await pipeline.research(keyword);
    console.log(`Found ${research.articles.length} articles`);
    
    // Step 2: Generate content
    const content = await pipeline.generate({
      research,
      format: 'listicle',
      count: 5 // top 5 points
    });
    
    // Step 3: Render videos
    const videos = await pipeline.renderVideos({
      content,
      platforms: ['reels', 'tiktok']
    });
    
    // Step 4: Schedule publishing
    await pipeline.schedule({
      content,
      videos,
      publishAt: new Date(Date.now() + 3600000) // 1 hour from now
    });
    
    return {
      content,
      videos,
      status: 'scheduled'
    };
  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}
```

## Content Format Templates

### Listicle Format

```typescript
import { ListicleGenerator } from '@/lib/formats/listicle';

const listicle = new ListicleGenerator();

const content = await listicle.create({
  title: 'Top 10 AI Trends in 2026',
  items: research.insights,
  structure: {
    intro: true,
    itemFormat: 'number-title-description-example',
    conclusion: true,
    cta: 'Learn more about AI automation'
  }
});
```

### POV (Point of View) Format

```typescript
import { POVGenerator } from '@/lib/formats/pov';

const pov = new POVGenerator();

const content = await pov.create({
  topic: 'AI in Marketing',
  perspective: 'marketer',
  stance: 'optimistic',
  evidence: research.insights,
  structure: {
    hook: true,
    argument: true,
    counterargument: false,
    conclusion: true
  }
});
```

### Case Study Format

```typescript
import { CaseStudyGenerator } from '@/lib/formats/case-study';

const caseStudy = new CaseStudyGenerator();

const content = await caseStudy.create({
  company: 'Tech Startup Inc',
  challenge: research.problems[0],
  solution: research.solutions[0],
  results: research.outcomes,
  structure: {
    background: true,
    challenge: true,
    solution: true,
    implementation: true,
    results: true,
    lessons: true
  }
});
```

## Video Composition Components

### Basic Video Template

```typescript
// src/video/ContentVideo.tsx
import { AbsoluteFill, Sequence, useCurrentFrame } from 'remotion';

interface ContentVideoProps {
  title: string;
  points: string[];
  brand: {
    color: string;
    logo: string;
  };
}

export const ContentVideo: React.FC<ContentVideoProps> = ({
  title,
  points,
  brand
}) => {
  const frame = useCurrentFrame();
  
  return (
    <AbsoluteFill style={{ backgroundColor: brand.color }}>
      {/* Intro sequence */}
      <Sequence from={0} durationInFrames={60}>
        <div className="intro">
          <h1>{title}</h1>
        </div>
      </Sequence>
      
      {/* Point sequences */}
      {points.map((point, idx) => (
        <Sequence
          key={idx}
          from={60 + (idx * 60)}
          durationInFrames={60}
        >
          <div className="point">
            <h2>#{idx + 1}</h2>
            <p>{point}</p>
          </div>
        </Sequence>
      ))}
      
      {/* Outro */}
      <Sequence from={60 + (points.length * 60)} durationInFrames={60}>
        <div className="outro">
          <img src={brand.logo} alt="Brand" />
        </div>
      </Sequence>
    </AbsoluteFill>
  );
};
```

## API Integration Patterns

### Claude Integration

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY
});

async function generateWithClaude(prompt: string, context: any) {
  const message = await anthropic.messages.create({
    model: 'claude-3-opus-20240229',
    max_tokens: 4096,
    messages: [{
      role: 'user',
      content: `${prompt}\n\nContext: ${JSON.stringify(context)}`
    }]
  });
  
  return message.content[0].text;
}
```

### OpenAI Integration

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY
});

async function generateWithOpenAI(prompt: string, context: any) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: 'You are a content marketing expert.'
      },
      {
        role: 'user',
        content: `${prompt}\n\nContext: ${JSON.stringify(context)}`
      }
    ],
    temperature: 0.7,
    max_tokens: 2000
  });
  
  return completion.choices[0].message.content;
}
```

## Common Workflows

### Daily Content Generation

```typescript
import { CronJob } from 'cron';

const dailyContentJob = new CronJob('0 9 * * *', async () => {
  const keywords = await getTrendingKeywords();
  
  for (const keyword of keywords) {
    await runFullPipeline(keyword);
  }
});

dailyContentJob.start();
```

### Batch Processing

```typescript
async function batchGenerateContent(keywords: string[]) {
  const results = await Promise.allSettled(
    keywords.map(keyword => runFullPipeline(keyword))
  );
  
  const successful = results.filter(r => r.status === 'fulfilled');
  const failed = results.filter(r => r.status === 'rejected');
  
  console.log(`Generated ${successful.length} pieces of content`);
  console.log(`Failed: ${failed.length}`);
  
  return { successful, failed };
}
```

## Troubleshooting

### API Rate Limits

```typescript
import { RateLimiter } from '@/lib/utils/rate-limiter';

const limiter = new RateLimiter({
  claude: { requestsPerMinute: 50 },
  openai: { requestsPerMinute: 60 },
  rapidapi: { requestsPerMinute: 100 }
});

async function safeAPICall(service: string, fn: () => Promise<any>) {
  await limiter.wait(service);
  return await fn();
}
```

### Video Rendering Failures

```typescript
async function renderWithRetry(contentId: string, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await renderContentVideo(contentId, `/output/${contentId}.mp4`);
    } catch (error) {
      console.error(`Render attempt ${i + 1} failed:`, error);
      if (i === maxRetries - 1) throw error;
      await new Promise(resolve => setTimeout(resolve, 5000 * (i + 1)));
    }
  }
}
```

### Content Quality Validation

```typescript
import { ContentValidator } from '@/lib/validation';

const validator = new ContentValidator();

async function validateContent(content: any) {
  const checks = await validator.validate(content, {
    minLength: 500,
    maxLength: 3000,
    requireSources: true,
    checkGrammar: true,
    checkPlagiarism: true
  });
  
  if (!checks.valid) {
    throw new Error(`Content validation failed: ${checks.errors.join(', ')}`);
  }
  
  return checks;
}
```

## Performance Optimization

### Caching Research Results

```typescript
import { Redis } from 'ioredis';

const redis = new Redis(process.env.REDIS_URL);

async function cachedResearch(keyword: string) {
  const cacheKey = `research:${keyword}`;
  const cached = await redis.get(cacheKey);
  
  if (cached) {
    return JSON.parse(cached);
  }
  
  const research = await researchTopic(keyword);
  await redis.setex(cacheKey, 3600, JSON.stringify(research)); // 1 hour cache
  
  return research;
}
```

### Parallel Video Rendering

```typescript
async function renderMultiplePlatforms(content: any) {
  const platforms = [
    { name: 'reels', width: 1080, height: 1920 },
    { name: 'tiktok', width: 1080, height: 1920 },
    { name: 'youtube-shorts', width: 1080, height: 1920 }
  ];
  
  const videos = await Promise.all(
    platforms.map(platform =>
      renderContentVideo(content.id, `/output/${content.id}-${platform.name}.mp4`)
    )
  );
  
  return videos;
}
```
