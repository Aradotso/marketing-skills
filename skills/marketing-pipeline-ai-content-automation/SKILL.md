---
name: marketing-pipeline-ai-content-automation
description: Automate content research, scriptwriting, and video generation using AI-powered pipeline with Claude, OpenAI, and Remotion
triggers:
  - how do I set up the AI content pipeline
  - automate content creation with this marketing pipeline
  - generate video from research automatically
  - create content using Claude and OpenAI
  - set up auto-posting content system
  - research and write articles with AI
  - render videos from content pipeline
  - configure the marketing automation pipeline
---

# Marketing Pipeline AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to help developers use the **Ultimate AI Content Pipeline** — an end-to-end automated content creation system that performs research, generates scripts, creates articles, and renders videos using AI (Claude 3, OpenAI) and Remotion.

## What This Project Does

The Marketing Pipeline automates the entire content creation workflow:

1. **Auto-Research**: Crawls and analyzes recent news from TechCrunch, a16z, Twitter/X, LinkedIn
2. **AI Content Generation**: Creates articles in multiple formats (Top List, POV, Case Study, How-to) using Claude/OpenAI
3. **Multi-language Support**: Generates content in both English and Vietnamese
4. **Video Rendering**: Automatically creates infographics and short videos using Remotion
5. **Auto-Publishing**: Schedules and posts content to various platforms

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

## Configuration

Create a `.env` file with the following environment variables:

```bash
# AI APIs
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Database (if using)
DATABASE_URL=your_database_connection_string

# Content Publishing
FACEBOOK_ACCESS_TOKEN=your_fb_token
LINKEDIN_ACCESS_TOKEN=your_linkedin_token

# Remotion Video Rendering
REMOTION_LICENSE_KEY=your_remotion_license

# Next.js
NEXT_PUBLIC_API_URL=http://localhost:3000
```

## Key Commands

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Run content research pipeline
npm run research

# Generate content from research
npm run generate

# Render videos from content
npm run render

# Auto-publish content
npm run publish
```

## Core API Usage

### Research Module

```typescript
import { ResearchService } from './services/research';

// Initialize research service
const researchService = new ResearchService({
  rapidApiKey: process.env.RAPIDAPI_KEY!,
  sources: ['techcrunch', 'a16z', 'twitter', 'linkedin']
});

// Fetch trending topics
const research = await researchService.fetchTrendingTopics({
  keyword: 'AI marketing automation',
  timeframe: '24h',
  limit: 10
});

// Analyze research data
const insights = await researchService.analyzeInsights(research);

console.log(insights);
```

### Content Generation Module

```typescript
import { ContentGenerator } from './services/content-generator';
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY!
});

const generator = new ContentGenerator({
  aiClient: anthropic,
  model: 'claude-3-5-sonnet-20241022'
});

// Generate article from research
const article = await generator.createArticle({
  research: insights,
  format: 'how-to', // 'toplist' | 'pov' | 'case-study' | 'how-to'
  language: 'en', // 'en' | 'vi'
  tone: 'professional', // 'professional' | 'friendly' | 'humorous'
  targetAudience: 'marketers'
});

console.log(article.title);
console.log(article.content);
```

### Multi-Language Content Generation

```typescript
import { MultiLanguageGenerator } from './services/multi-language';

const mlGenerator = new MultiLanguageGenerator({
  claudeApiKey: process.env.ANTHROPIC_API_KEY!,
  openaiApiKey: process.env.OPENAI_API_KEY!
});

// Generate content in both languages simultaneously
const dualContent = await mlGenerator.generateDual({
  topic: 'AI Content Marketing Trends 2026',
  format: 'toplist',
  languages: ['en', 'vi']
});

console.log('English:', dualContent.en.content);
console.log('Vietnamese:', dualContent.vi.content);
```

### Video Rendering with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoData {
  title: string;
  points: string[];
  branding: {
    logo: string;
    color: string;
  };
}

async function renderContentVideo(content: VideoData) {
  // Bundle Remotion project
  const bundled = await bundle({
    entryPoint: path.join(process.cwd(), 'remotion/index.ts'),
    webpackOverride: (config) => config
  });

  // Select composition
  const composition = await selectComposition({
    serveUrl: bundled,
    id: 'ContentVideo',
    inputProps: content
  });

  // Render video
  const outputPath = path.join(process.cwd(), 'output', `video-${Date.now()}.mp4`);
  
  await renderMedia({
    composition,
    serveUrl: bundled,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: content
  });

  return outputPath;
}

// Usage
const videoPath = await renderContentVideo({
  title: article.title,
  points: article.keyPoints,
  branding: {
    logo: '/assets/logo.png',
    color: '#FF6B6B'
  }
});
```

## Common Patterns

### End-to-End Content Pipeline

```typescript
import { ContentPipeline } from './services/pipeline';

const pipeline = new ContentPipeline({
  anthropicKey: process.env.ANTHROPIC_API_KEY!,
  openaiKey: process.env.OPENAI_API_KEY!,
  rapidApiKey: process.env.RAPIDAPI_KEY!
});

// Run complete pipeline
async function runFullPipeline(keyword: string) {
  try {
    // Step 1: Research
    console.log('🔍 Starting research...');
    const research = await pipeline.research({
      keyword,
      sources: ['techcrunch', 'a16z'],
      timeframe: '24h'
    });

    // Step 2: Generate content
    console.log('✍️ Generating content...');
    const content = await pipeline.generate({
      research,
      format: 'how-to',
      languages: ['en', 'vi']
    });

    // Step 3: Render video
    console.log('🎬 Rendering video...');
    const video = await pipeline.renderVideo({
      content: content.en,
      aspectRatio: '9:16' // TikTok/Reels format
    });

    // Step 4: Schedule publishing
    console.log('📅 Scheduling posts...');
    await pipeline.schedulePost({
      content: content.en,
      video,
      platforms: ['facebook', 'linkedin', 'twitter'],
      scheduledTime: new Date(Date.now() + 3600000) // 1 hour from now
    });

    return {
      research,
      content,
      video
    };
  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}

// Execute pipeline
runFullPipeline('AI marketing automation').then(result => {
  console.log('✅ Pipeline completed successfully!');
  console.log('Video path:', result.video);
});
```

### Custom Content Formats

```typescript
import { formatters } from './utils/formatters';

// Top List format
const topListArticle = await generator.createArticle({
  research: insights,
  format: 'toplist',
  customPrompt: `
    Create a top 10 list about ${insights.topic}.
    Each item should have:
    - Catchy title
    - Brief explanation (50-100 words)
    - Real-world example
    - Data point or statistic
  `
});

// POV (Point of View) format
const povArticle = await generator.createArticle({
  research: insights,
  format: 'pov',
  customPrompt: `
    Write from the perspective of a marketing director
    who has successfully implemented this strategy.
    Include personal anecdotes and lessons learned.
  `
});

// Case Study format
const caseStudyArticle = await generator.createArticle({
  research: insights,
  format: 'case-study',
  customPrompt: `
    Structure as a business case study with:
    - Challenge/Problem
    - Solution approach
    - Implementation process
    - Results with metrics
    - Key takeaways
  `
});
```

### Batch Content Generation

```typescript
async function generateContentBatch(keywords: string[]) {
  const results = await Promise.all(
    keywords.map(async (keyword) => {
      const research = await researchService.fetchTrendingTopics({
        keyword,
        timeframe: '24h'
      });

      const content = await generator.createArticle({
        research,
        format: 'how-to',
        language: 'en'
      });

      return {
        keyword,
        content,
        research
      };
    })
  );

  return results;
}

// Generate multiple articles
const batch = await generateContentBatch([
  'AI content marketing',
  'Marketing automation tools',
  'Social media trends 2026'
]);
```

## API Routes (Next.js)

### Research Endpoint

```typescript
// pages/api/research.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { ResearchService } from '@/services/research';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  try {
    const { keyword, sources, timeframe } = req.body;

    const researchService = new ResearchService({
      rapidApiKey: process.env.RAPIDAPI_KEY!,
      sources
    });

    const results = await researchService.fetchTrendingTopics({
      keyword,
      timeframe: timeframe || '24h',
      limit: 10
    });

    res.status(200).json({ success: true, data: results });
  } catch (error) {
    console.error('Research API error:', error);
    res.status(500).json({ error: 'Research failed' });
  }
}
```

### Content Generation Endpoint

```typescript
// pages/api/generate.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { ContentGenerator } from '@/services/content-generator';
import Anthropic from '@anthropic-ai/sdk';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  try {
    const { research, format, language, tone } = req.body;

    const anthropic = new Anthropic({
      apiKey: process.env.ANTHROPIC_API_KEY!
    });

    const generator = new ContentGenerator({
      aiClient: anthropic,
      model: 'claude-3-5-sonnet-20241022'
    });

    const article = await generator.createArticle({
      research,
      format,
      language,
      tone
    });

    res.status(200).json({ success: true, article });
  } catch (error) {
    console.error('Generation API error:', error);
    res.status(500).json({ error: 'Content generation failed' });
  }
}
```

## Troubleshooting

### API Rate Limits

```typescript
import { RateLimiter } from './utils/rate-limiter';

const limiter = new RateLimiter({
  anthropic: { requests: 50, per: 'minute' },
  openai: { requests: 60, per: 'minute' },
  rapidApi: { requests: 100, per: 'day' }
});

async function safeFetch(apiName: string, fetchFn: () => Promise<any>) {
  await limiter.waitForSlot(apiName);
  try {
    return await fetchFn();
  } catch (error) {
    if (error.status === 429) {
      console.log('Rate limited, waiting...');
      await new Promise(resolve => setTimeout(resolve, 60000));
      return safeFetch(apiName, fetchFn);
    }
    throw error;
  }
}
```

### Video Rendering Timeouts

```typescript
import { renderMedia } from '@remotion/renderer';

async function renderWithTimeout(composition: any, timeout = 300000) {
  return Promise.race([
    renderMedia({
      composition,
      codec: 'h264',
      outputLocation: './output.mp4'
    }),
    new Promise((_, reject) =>
      setTimeout(() => reject(new Error('Render timeout')), timeout)
    )
  ]);
}
```

### Content Quality Validation

```typescript
interface ValidationResult {
  isValid: boolean;
  issues: string[];
  score: number;
}

function validateContent(content: string): ValidationResult {
  const issues: string[] = [];
  let score = 100;

  // Check minimum length
  if (content.length < 500) {
    issues.push('Content too short (min 500 chars)');
    score -= 20;
  }

  // Check for repetition
  const sentences = content.split('.');
  const uniqueSentences = new Set(sentences);
  if (sentences.length - uniqueSentences.size > 3) {
    issues.push('Too much repetition detected');
    score -= 15;
  }

  // Check for proper structure
  if (!content.includes('\n\n')) {
    issues.push('Missing paragraph breaks');
    score -= 10;
  }

  return {
    isValid: score >= 70,
    issues,
    score
  };
}
```

### Error Recovery in Pipeline

```typescript
async function robustPipeline(keyword: string, maxRetries = 3) {
  let attempt = 0;
  const errors: Error[] = [];

  while (attempt < maxRetries) {
    try {
      return await runFullPipeline(keyword);
    } catch (error) {
      attempt++;
      errors.push(error as Error);
      console.log(`Attempt ${attempt} failed:`, error);

      if (attempt < maxRetries) {
        // Exponential backoff
        await new Promise(resolve =>
          setTimeout(resolve, Math.pow(2, attempt) * 1000)
        );
      }
    }
  }

  throw new Error(
    `Pipeline failed after ${maxRetries} attempts: ${errors.map(e => e.message).join(', ')}`
  );
}
```

## Best Practices

1. **Always use environment variables** for API keys and secrets
2. **Implement rate limiting** when calling external APIs
3. **Validate content quality** before publishing
4. **Cache research results** to avoid redundant API calls
5. **Monitor video render times** and set appropriate timeouts
6. **Use batch processing** for multiple content pieces to optimize API usage
7. **Implement retry logic** with exponential backoff for API failures
