---
name: marketing-pipeline-ai-content-automation
description: AI-powered content automation pipeline for research, scriptwriting, and video generation using Claude/OpenAI
triggers:
  - "automate content creation with AI pipeline"
  - "generate video content from articles automatically"
  - "research and write articles with Claude"
  - "crawl news sources for content ideas"
  - "create multilingual marketing content"
  - "render video from text content"
  - "build automated content workflow"
  - "scrape trending topics for content"
---

# Marketing Pipeline AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

A complete AI-powered content automation system that handles research, scriptwriting, article generation, and video rendering. The pipeline crawls real-time data from news sources (TechCrunch, Twitter, LinkedIn), generates multilingual content using Claude/OpenAI, and automatically renders videos using Remotion.

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
cp .env.example .env.local
```

## Environment Configuration

Create `.env.local` with the following variables:

```bash
# AI Services
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research/Scraping APIs
RAPIDAPI_KEY=your_rapidapi_key

# Optional: Social Media APIs
TWITTER_API_KEY=your_twitter_key
LINKEDIN_API_KEY=your_linkedin_key

# Database (if applicable)
DATABASE_URL=your_database_url

# Remotion (Video Rendering)
REMOTION_LICENSE_KEY=your_remotion_license
```

## Core Architecture

The pipeline consists of 4 main stages:

1. **Research Stage**: Crawl and scrape trending content
2. **Generation Stage**: Create articles with AI (Claude/OpenAI)
3. **Translation Stage**: Generate multilingual versions
4. **Video Stage**: Render videos using Remotion

## Key Usage Patterns

### 1. Running the Complete Pipeline

```typescript
import { ContentPipeline } from './src/pipeline';

const pipeline = new ContentPipeline({
  aiProvider: 'claude', // or 'openai'
  language: ['en', 'vi'],
  format: 'toplist', // or 'pov', 'case-study', 'how-to'
  tone: 'professional' // or 'friendly', 'humorous'
});

// Execute full pipeline
const result = await pipeline.execute({
  keyword: 'AI marketing automation',
  sources: ['techcrunch', 'twitter', 'linkedin'],
  timeRange: '24h'
});

console.log(result);
// {
//   articles: [...],
//   videos: [...],
//   status: 'completed'
// }
```

### 2. Research Module - Crawl Data

```typescript
import { ResearchService } from './src/services/research';

const researcher = new ResearchService({
  rapidApiKey: process.env.RAPIDAPI_KEY
});

// Crawl trending topics
const insights = await researcher.crawlSources({
  keyword: 'content marketing',
  sources: ['techcrunch', 'a16z', 'twitter'],
  limit: 50,
  dateRange: {
    from: '2024-01-01',
    to: '2024-01-02'
  }
});

// Extract key insights
const processed = researcher.extractInsights(insights);
console.log(processed.trends); // Top trends
console.log(processed.statistics); // Data-backed stats
```

### 3. Content Generation with AI

```typescript
import { ContentGenerator } from './src/services/generator';

const generator = new ContentGenerator({
  provider: 'anthropic', // or 'openai'
  apiKey: process.env.ANTHROPIC_API_KEY,
  model: 'claude-3-opus-20240229'
});

// Generate article
const article = await generator.createArticle({
  topic: 'AI Content Automation in 2024',
  format: 'toplist',
  tone: 'expert',
  language: 'en',
  researchData: insights,
  structure: {
    sections: 5,
    wordsPerSection: 300,
    includeCTA: true
  }
});

console.log(article.title);
console.log(article.content);
console.log(article.metadata);
```

### 4. Multi-Language Translation

```typescript
import { TranslationService } from './src/services/translation';

const translator = new TranslationService({
  apiKey: process.env.OPENAI_API_KEY
});

// Translate to Vietnamese
const translated = await translator.translate({
  content: article.content,
  from: 'en',
  to: 'vi',
  preserveFormatting: true,
  adaptCulturally: true
});

console.log(translated.content);
```

### 5. Video Rendering with Remotion

```typescript
import { VideoRenderer } from './src/services/video';
import { bundle } from '@remotion/bundler';
import { renderMedia } from '@remotion/renderer';

const renderer = new VideoRenderer({
  composition: 'ArticleVideo',
  fps: 30,
  durationInFrames: 900 // 30 seconds
});

// Prepare video data from article
const videoData = {
  title: article.title,
  sections: article.sections.map(s => ({
    heading: s.heading,
    points: s.keyPoints,
    duration: 150 // frames
  })),
  branding: {
    logo: '/assets/logo.png',
    colors: {
      primary: '#FF6B6B',
      secondary: '#4ECDC4'
    }
  }
};

// Render video
const output = await renderer.render({
  data: videoData,
  outputFormat: 'mp4',
  aspectRatio: '9:16', // For TikTok/Reels
  quality: 'high'
});

console.log(`Video saved to: ${output.path}`);
```

## API Routes (Next.js)

### Trigger Pipeline Endpoint

```typescript
// pages/api/pipeline/run.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { ContentPipeline } from '@/services/pipeline';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  const { keyword, sources, format, languages } = req.body;

  try {
    const pipeline = new ContentPipeline({
      aiProvider: 'claude',
      language: languages || ['en', 'vi'],
      format: format || 'toplist'
    });

    const result = await pipeline.execute({
      keyword,
      sources: sources || ['techcrunch', 'twitter'],
      timeRange: '24h'
    });

    return res.status(200).json(result);
  } catch (error) {
    console.error('Pipeline error:', error);
    return res.status(500).json({ 
      error: 'Pipeline failed',
      details: error.message 
    });
  }
}
```

### Usage from Frontend

```typescript
// Example React component
import { useState } from 'react';

export function PipelineController() {
  const [status, setStatus] = useState('idle');

  const runPipeline = async () => {
    setStatus('running');
    
    const response = await fetch('/api/pipeline/run', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        keyword: 'AI marketing tools',
        sources: ['techcrunch', 'twitter', 'linkedin'],
        format: 'toplist',
        languages: ['en', 'vi']
      })
    });

    const result = await response.json();
    setStatus('completed');
    return result;
  };

  return (
    <button onClick={runPipeline} disabled={status === 'running'}>
      {status === 'running' ? 'Processing...' : 'Generate Content'}
    </button>
  );
}
```

## Content Format Templates

### Top List Format

```typescript
const toplistTemplate = {
  format: 'toplist',
  structure: {
    intro: {
      hook: true,
      context: true,
      preview: true
    },
    items: {
      count: 10,
      sections: ['title', 'description', 'pros', 'cons', 'verdict']
    },
    conclusion: {
      summary: true,
      cta: true
    }
  }
};
```

### POV (Point of View) Format

```typescript
const povTemplate = {
  format: 'pov',
  structure: {
    personalStory: true,
    mainArgument: true,
    supportingPoints: 3,
    counterArguments: true,
    conclusion: true
  }
};
```

### Case Study Format

```typescript
const caseStudyTemplate = {
  format: 'case-study',
  structure: {
    background: true,
    challenge: true,
    solution: true,
    results: {
      metrics: true,
      testimonial: true
    },
    lessons: true
  }
};
```

## CLI Commands

```bash
# Run development server
npm run dev

# Generate content from CLI
npm run generate -- --keyword "AI tools" --format toplist

# Render video only
npm run render-video -- --input article.json --output video.mp4

# Run full pipeline
npm run pipeline -- --config pipeline.config.json

# Test research module
npm run research -- --sources techcrunch,twitter --keyword "AI"
```

## Configuration File

Create `pipeline.config.json`:

```json
{
  "research": {
    "sources": ["techcrunch", "twitter", "linkedin", "a16z"],
    "timeRange": "24h",
    "limit": 50
  },
  "generation": {
    "provider": "claude",
    "model": "claude-3-opus-20240229",
    "temperature": 0.7,
    "maxTokens": 4000
  },
  "translation": {
    "languages": ["en", "vi"],
    "adaptCulturally": true
  },
  "video": {
    "format": "mp4",
    "aspectRatios": ["16:9", "9:16", "1:1"],
    "quality": "high",
    "fps": 30
  },
  "output": {
    "directory": "./output",
    "saveMetadata": true
  }
}
```

## Common Patterns

### Batch Processing Multiple Keywords

```typescript
const keywords = [
  'AI marketing automation',
  'content creation tools',
  'video editing software'
];

const results = await Promise.all(
  keywords.map(keyword => 
    pipeline.execute({ keyword, sources: ['techcrunch'], timeRange: '24h' })
  )
);

results.forEach((result, index) => {
  console.log(`${keywords[index]}: ${result.articles.length} articles generated`);
});
```

### Custom AI Prompts

```typescript
const generator = new ContentGenerator({
  provider: 'anthropic',
  apiKey: process.env.ANTHROPIC_API_KEY
});

// Override default prompt
generator.setCustomPrompt(`
You are a marketing expert writing for B2B SaaS companies.
Create a ${format} article about ${topic}.
Use data from: ${JSON.stringify(researchData)}
Tone: ${tone}
Include: statistics, expert quotes, actionable tips
Length: ${wordCount} words
`);

const article = await generator.generate();
```

### Scheduled Pipeline Execution

```typescript
import cron from 'node-cron';

// Run pipeline every day at 9 AM
cron.schedule('0 9 * * *', async () => {
  console.log('Running daily content pipeline...');
  
  const result = await pipeline.execute({
    keyword: 'trending marketing news',
    sources: ['techcrunch', 'twitter'],
    timeRange: '24h'
  });
  
  // Auto-publish to CMS or social media
  await publishToWordPress(result.articles[0]);
  await postToSocialMedia(result.videos[0]);
});
```

## Troubleshooting

### API Rate Limits

```typescript
import pRetry from 'p-retry';

const generateWithRetry = async (params) => {
  return pRetry(
    async () => {
      return await generator.createArticle(params);
    },
    {
      retries: 3,
      onFailedAttempt: error => {
        console.log(`Attempt ${error.attemptNumber} failed. Retrying...`);
      }
    }
  );
};
```

### Video Rendering Failures

```typescript
// Check Remotion logs
const output = await renderer.render({
  data: videoData,
  verbose: true,
  logLevel: 'verbose'
});

// Fallback to lower quality if high quality fails
try {
  await renderer.render({ quality: 'high' });
} catch (error) {
  console.warn('High quality failed, trying medium...');
  await renderer.render({ quality: 'medium' });
}
```

### Missing Research Data

```typescript
// Validate research results before generation
if (!insights || insights.length === 0) {
  console.warn('No research data found, using fallback');
  
  // Use cached data or generic template
  insights = await loadCachedInsights(keyword);
}
```

### Language Detection Issues

```typescript
import { franc } from 'franc';

// Auto-detect language if not specified
const detectedLang = franc(content);

if (detectedLang !== expectedLang) {
  console.warn(`Language mismatch: expected ${expectedLang}, got ${detectedLang}`);
  // Re-translate or flag for review
}
```

## Performance Tips

1. **Cache research results** for reuse across multiple articles
2. **Use streaming responses** from Claude/OpenAI for faster perceived performance
3. **Parallelize video rendering** for multiple aspect ratios
4. **Implement queue system** for large batch processing
5. **Pre-bundle Remotion compositions** to speed up rendering
