---
name: marketing-pipeline-ai-content-automation
description: Vietnamese AI-powered content pipeline for automated research, scriptwriting, and video generation using Claude, OpenAI, and Remotion
triggers:
  - how do I automate content creation with AI research
  - generate video content from text automatically
  - build an AI content pipeline with Claude and OpenAI
  - crawl news sources and create marketing content
  - create multilingual content with AI automation
  - set up automated content workflow with Remotion
  - build AI-powered marketing content system
  - automate social media content from research to video
---

# Marketing Pipeline AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the Ultimate AI Content Pipeline - a comprehensive system that automates content creation from research through video generation. The pipeline crawls news sources, generates multilingual content, and creates videos automatically using Claude/OpenAI and Remotion.

## What This Project Does

The Marketing Pipeline automates the entire content creation workflow:

1. **Auto-Research**: Crawls recent news from TechCrunch, a16z, Twitter/X, LinkedIn
2. **AI Content Generation**: Creates articles in multiple formats (toplist, POV, case study, how-to) using Claude 3 or OpenAI
3. **Multilingual Output**: Generates content in both English and Vietnamese
4. **Video Generation**: Renders videos and infographics from content using Remotion
5. **Multi-Platform Publishing**: Optimizes content for Reels, TikTok, Shorts

## Installation

### Prerequisites

```bash
# Node.js 18+ required
node --version

# Install dependencies
npm install
# or
yarn install
# or
pnpm install
```

### Environment Configuration

Create a `.env.local` file in the root directory:

```bash
# AI Model Configuration
ANTHROPIC_API_KEY=your_claude_api_key_here
OPENAI_API_KEY=your_openai_api_key_here

# News API & Research
RAPIDAPI_KEY=your_rapidapi_key_here
TWITTER_API_KEY=your_twitter_api_key_here

# Remotion Video Generation
REMOTION_LICENSE_KEY=your_remotion_license_key_here

# Next.js Configuration
NEXT_PUBLIC_API_URL=http://localhost:3000
```

### Start Development Server

```bash
npm run dev
# or
yarn dev
# or
pnpm dev
```

The application will be available at `http://localhost:3000`

## Project Structure

```
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── crawler/     # News crawling logic
│   │   ├── content/     # Content generation
│   │   └── video/       # Remotion video generation
│   └── utils/           # Utility functions
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Core API Usage

### 1. Content Research & Crawling

```typescript
import { NewsResearcher } from '@/lib/crawler/news-researcher';

// Initialize researcher
const researcher = new NewsResearcher({
  apiKey: process.env.RAPIDAPI_KEY,
  sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
  timeRange: '24h'
});

// Fetch latest news on a topic
const researchData = await researcher.fetchNews({
  keyword: 'AI automation',
  limit: 10,
  language: 'en'
});

// Extract insights
const insights = await researcher.extractInsights(researchData);

console.log(insights);
// {
//   trending: [...],
//   keyPoints: [...],
//   dataPoints: [...],
//   sources: [...]
// }
```

### 2. AI Content Generation with Claude

```typescript
import { ClaudeContentGenerator } from '@/lib/ai/claude-generator';

const generator = new ClaudeContentGenerator({
  apiKey: process.env.ANTHROPIC_API_KEY,
  model: 'claude-3-opus-20240229'
});

// Generate content from research
const content = await generator.generateContent({
  research: insights,
  format: 'toplist', // 'pov' | 'case-study' | 'how-to'
  tone: 'professional', // 'friendly' | 'humorous'
  language: 'vi', // 'en' | 'vi'
  targetAudience: 'marketers'
});

console.log(content);
// {
//   title: "Top 10 AI Automation Tools...",
//   body: "...",
//   meta: { ... },
//   images: [...]
// }
```

### 3. OpenAI Integration (Alternative)

```typescript
import { OpenAIContentGenerator } from '@/lib/ai/openai-generator';

const openaiGenerator = new OpenAIContentGenerator({
  apiKey: process.env.OPENAI_API_KEY,
  model: 'gpt-4-turbo-preview'
});

const content = await openaiGenerator.generateContent({
  research: insights,
  format: 'case-study',
  language: 'en'
});
```

### 4. Multilingual Content Generation

```typescript
import { MultilingualGenerator } from '@/lib/content/multilingual';

const mlGenerator = new MultilingualGenerator({
  claudeKey: process.env.ANTHROPIC_API_KEY,
  openaiKey: process.env.OPENAI_API_KEY
});

// Generate content in both languages simultaneously
const bilingualContent = await mlGenerator.generateBilingual({
  research: insights,
  languages: ['en', 'vi'],
  format: 'how-to'
});

console.log(bilingualContent);
// {
//   en: { title: "...", body: "..." },
//   vi: { title: "...", body: "..." }
// }
```

### 5. Video Generation with Remotion

```typescript
import { RemotionVideoRenderer } from '@/lib/video/remotion-renderer';
import { bundle } from '@remotion/bundler';
import { renderMedia } from '@remotion/renderer';

const renderer = new RemotionVideoRenderer({
  licenseKey: process.env.REMOTION_LICENSE_KEY
});

// Render video from content
const videoPath = await renderer.renderVideo({
  content: content,
  template: 'infographic', // 'reels' | 'shorts' | 'tiktok'
  aspectRatio: '9:16',
  duration: 30
});

console.log(`Video rendered: ${videoPath}`);
```

### 6. Complete Pipeline Example

```typescript
import { ContentPipeline } from '@/lib/content/pipeline';

const pipeline = new ContentPipeline({
  anthropicKey: process.env.ANTHROPIC_API_KEY,
  openaiKey: process.env.OPENAI_API_KEY,
  rapidApiKey: process.env.RAPIDAPI_KEY,
  remotionKey: process.env.REMOTION_LICENSE_KEY
});

// Run full pipeline
const result = await pipeline.run({
  keyword: 'AI marketing automation',
  format: 'toplist',
  languages: ['en', 'vi'],
  generateVideo: true,
  videoFormats: ['reels', 'shorts'],
  autoPublish: false
});

console.log(result);
// {
//   content: { en: {...}, vi: {...} },
//   videos: { reels: "path/to/reels.mp4", shorts: "path/to/shorts.mp4" },
//   metadata: { ... }
// }
```

## Common Patterns

### Pattern 1: Research-Only Mode

```typescript
// Only fetch and analyze research without generating content
const researchOnly = await pipeline.research({
  keyword: 'blockchain marketing',
  sources: ['techcrunch', 'linkedin'],
  depth: 'deep' // 'quick' | 'deep'
});
```

### Pattern 2: Batch Content Generation

```typescript
// Generate multiple content pieces from one research
const batchContent = await pipeline.generateBatch({
  research: insights,
  formats: ['toplist', 'case-study', 'how-to'],
  languages: ['en', 'vi']
});

// Results in 6 pieces of content (3 formats × 2 languages)
```

### Pattern 3: Custom Voice & Tone

```typescript
const customContent = await generator.generateContent({
  research: insights,
  format: 'pov',
  customInstructions: `
    Write in a conversational tone.
    Include personal anecdotes.
    Use Vietnamese internet slang when appropriate.
    Target Gen Z audience.
  `,
  language: 'vi'
});
```

### Pattern 4: Video with Custom Branding

```typescript
const brandedVideo = await renderer.renderVideo({
  content: content,
  template: 'reels',
  branding: {
    logo: '/public/logo.png',
    colors: {
      primary: '#FF6B6B',
      secondary: '#4ECDC4'
    },
    font: 'Inter',
    watermark: true
  }
});
```

## Configuration

### Content Generation Config

```typescript
// src/config/content.config.ts
export const contentConfig = {
  formats: {
    toplist: {
      minItems: 5,
      maxItems: 15,
      includeImages: true
    },
    caseStudy: {
      sections: ['background', 'challenge', 'solution', 'results'],
      minWords: 800
    },
    howTo: {
      stepFormat: 'numbered',
      includeVisuals: true
    }
  },
  ai: {
    temperature: 0.7,
    maxTokens: 4000,
    retryAttempts: 3
  }
};
```

### Remotion Video Config

```typescript
// remotion/config/video.config.ts
export const videoConfig = {
  fps: 30,
  durationInFrames: 900, // 30 seconds at 30fps
  compositions: {
    reels: {
      width: 1080,
      height: 1920,
      defaultProps: {
        transitionDuration: 15
      }
    },
    shorts: {
      width: 1080,
      height: 1920
    },
    tiktok: {
      width: 1080,
      height: 1920
    }
  }
};
```

## CLI Commands (if available)

```bash
# Generate content from keyword
npm run generate -- --keyword "AI marketing" --format toplist --lang vi

# Research only
npm run research -- --keyword "web3" --sources techcrunch,linkedin

# Render video from existing content
npm run render -- --input content.json --template reels

# Run full pipeline
npm run pipeline -- --keyword "automation" --video true
```

## Troubleshooting

### Issue: API Rate Limits

```typescript
// Implement rate limiting
import { RateLimiter } from '@/lib/utils/rate-limiter';

const limiter = new RateLimiter({
  maxRequests: 10,
  perSeconds: 60
});

await limiter.throttle(async () => {
  return await generator.generateContent(params);
});
```

### Issue: Video Rendering Fails

```typescript
// Add error handling and retries
try {
  const video = await renderer.renderVideo(params);
} catch (error) {
  console.error('Video render failed:', error);
  
  // Retry with lower quality
  const fallbackVideo = await renderer.renderVideo({
    ...params,
    quality: 'medium',
    fps: 24
  });
}
```

### Issue: Content Quality Issues

```typescript
// Validate generated content
import { ContentValidator } from '@/lib/utils/validator';

const validator = new ContentValidator();

const isValid = validator.validate(content, {
  minWords: 500,
  requireImages: true,
  checkGrammar: true,
  language: 'vi'
});

if (!isValid) {
  // Regenerate with adjusted parameters
  content = await generator.generateContent({
    ...params,
    temperature: 0.5 // Lower temperature for more focused output
  });
}
```

### Issue: Crawler Blocked

```typescript
// Use proxy rotation
const researcher = new NewsResearcher({
  apiKey: process.env.RAPIDAPI_KEY,
  useProxy: true,
  proxyRotation: true,
  userAgentRotation: true
});
```

## Best Practices

1. **Always validate research data** before generating content
2. **Cache research results** to avoid redundant API calls
3. **Use appropriate AI temperature** (0.7 for creative, 0.3 for factual)
4. **Test video renders** with short durations first
5. **Implement retry logic** for all external API calls
6. **Monitor API quotas** to avoid unexpected failures

This skill enables complete automation of content creation pipelines with Vietnamese language support and multi-platform video distribution capabilities.
