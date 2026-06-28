---
name: ultimate-ai-content-pipeline
description: Automated content creation pipeline with AI research, multi-format content generation, and video rendering using Claude, OpenAI, and Remotion
triggers:
  - how do I set up the AI content pipeline
  - generate automated content with research and video
  - create content using Claude and Remotion
  - automate content creation from research to video
  - use the marketing pipeline for content generation
  - build automated content workflow with AI
  - configure the content automation pipeline
  - generate multi-format content with AI research
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

Ultimate AI Content Pipeline is a comprehensive TypeScript-based content automation system that handles the complete content creation workflow: from automated news research, AI-powered content generation in multiple formats, to automatic video rendering. It integrates Claude 3, OpenAI, and Remotion to create a complete content factory.

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

```env
# AI Provider APIs
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs (RapidAPI)
RAPIDAPI_KEY=your_rapidapi_key

# Optional: Database configuration
DATABASE_URL=your_database_url

# Next.js configuration
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/             # Core utilities and integrations
│   │   ├── ai/          # AI provider integrations
│   │   ├── research/    # Content research crawlers
│   │   └── video/       # Remotion video rendering
│   ├── types/           # TypeScript types
│   └── utils/           # Helper functions
├── public/              # Static assets
└── remotion/            # Video templates
```

## Core Components

### 1. Content Research Module

The research module automatically crawls and aggregates content from multiple sources:

```typescript
import { researchTopic } from '@/lib/research/crawler';

// Automatic content research
const researchResults = await researchTopic({
  keyword: 'AI automation',
  sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
  timeframe: '24h',
  language: 'en'
});

// Returns structured data
interface ResearchResult {
  articles: Array<{
    title: string;
    url: string;
    snippet: string;
    source: string;
    publishedAt: Date;
    insights: string[];
  }>;
  trends: string[];
  statistics: Record<string, any>;
}
```

### 2. AI Content Generation

Generate content in multiple formats using Claude or OpenAI:

```typescript
import { generateContent } from '@/lib/ai/content-generator';

const content = await generateContent({
  topic: 'AI Marketing Automation',
  researchData: researchResults,
  format: 'toplist', // 'toplist' | 'pov' | 'case-study' | 'how-to'
  languages: ['en', 'vi'],
  tone: 'expert', // 'expert' | 'friendly' | 'humorous'
  provider: 'claude', // 'claude' | 'openai'
});

// Output structure
interface GeneratedContent {
  title: string;
  content: string;
  metadata: {
    wordCount: number;
    readingTime: number;
    seoScore: number;
  };
  translations: Record<string, string>;
  suggestions: string[];
}
```

### 3. Multi-Format Content Templates

```typescript
// Toplist format
const toplistContent = await generateContent({
  topic: 'Best AI Tools 2026',
  format: 'toplist',
  options: {
    itemCount: 10,
    includeRatings: true,
    includePricing: true,
  }
});

// POV (Point of View) format
const povContent = await generateContent({
  topic: 'Future of Content Marketing',
  format: 'pov',
  options: {
    perspective: 'industry-expert',
    includePersonalStory: true,
  }
});

// Case Study format
const caseStudyContent = await generateContent({
  topic: 'AI Content Success Story',
  format: 'case-study',
  options: {
    includeMetrics: true,
    includeTimeline: true,
    includeLessons: true,
  }
});
```

### 4. Video Generation with Remotion

```typescript
import { renderVideo } from '@/lib/video/renderer';
import { bundle } from '@remotion/bundler';
import { renderMedia } from '@remotion/renderer';

// Define video composition
const videoConfig = {
  content: generatedContent,
  template: 'infographic', // 'infographic' | 'reels' | 'shorts'
  aspectRatio: '9:16', // '9:16' | '16:9' | '1:1'
  duration: 30, // seconds
  style: {
    primaryColor: '#FF6B6B',
    secondaryColor: '#4ECDC4',
    font: 'Inter',
  }
};

// Render video
const video = await renderVideo(videoConfig);

// Output video file
interface VideoOutput {
  videoPath: string;
  thumbnailPath: string;
  duration: number;
  size: number;
  format: string;
}
```

### 5. Complete Pipeline Execution

```typescript
import { runContentPipeline } from '@/lib/pipeline';

// Full automation workflow
const pipeline = await runContentPipeline({
  keyword: 'AI Marketing Trends',
  
  // Research configuration
  research: {
    sources: ['techcrunch', 'a16z', 'linkedin'],
    timeframe: '24h',
    depth: 'comprehensive',
  },
  
  // Content generation
  content: {
    formats: ['toplist', 'pov'],
    languages: ['en', 'vi'],
    tone: 'expert',
    provider: 'claude',
  },
  
  // Video rendering
  video: {
    enabled: true,
    templates: ['reels', 'shorts'],
    aspectRatios: ['9:16', '1:1'],
  },
  
  // Output options
  output: {
    saveToDatabase: true,
    exportFormats: ['markdown', 'html', 'json'],
    generateSEO: true,
  }
});

// Pipeline returns complete package
interface PipelineOutput {
  research: ResearchResult;
  content: GeneratedContent[];
  videos: VideoOutput[];
  seo: {
    metaTitle: string;
    metaDescription: string;
    keywords: string[];
    ogImage: string;
  };
}
```

## API Integration Patterns

### Claude AI Integration

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

async function generateWithClaude(prompt: string, context: string) {
  const message = await anthropic.messages.create({
    model: 'claude-3-opus-20240229',
    max_tokens: 4096,
    messages: [
      {
        role: 'user',
        content: `${context}\n\n${prompt}`
      }
    ],
    temperature: 0.7,
  });
  
  return message.content[0].text;
}
```

### OpenAI Integration

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateWithOpenAI(prompt: string, context: string) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: 'You are an expert content creator and marketer.'
      },
      {
        role: 'user',
        content: `${context}\n\n${prompt}`
      }
    ],
    temperature: 0.7,
    max_tokens: 4096,
  });
  
  return completion.choices[0].message.content;
}
```

### RapidAPI Research Integration

```typescript
async function fetchNewsFromRapidAPI(keyword: string) {
  const response = await fetch(
    `https://api.rapidapi.com/news/search?q=${encodeURIComponent(keyword)}`,
    {
      headers: {
        'X-RapidAPI-Key': process.env.RAPIDAPI_KEY,
        'X-RapidAPI-Host': 'news-api.rapidapi.com',
      },
    }
  );
  
  return await response.json();
}
```

## Remotion Video Templates

Create a custom video composition in `remotion/compositions/`:

```typescript
import { AbsoluteFill, useCurrentFrame, useVideoConfig } from 'remotion';

export const InfographicVideo: React.FC<{
  title: string;
  points: string[];
  colors: { primary: string; secondary: string };
}> = ({ title, points, colors }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  
  const opacity = Math.min(1, frame / (fps * 0.5));
  
  return (
    <AbsoluteFill
      style={{
        backgroundColor: colors.primary,
        justifyContent: 'center',
        alignItems: 'center',
      }}
    >
      <h1
        style={{
          fontSize: 80,
          color: 'white',
          opacity,
          textAlign: 'center',
        }}
      >
        {title}
      </h1>
      
      <div style={{ marginTop: 50 }}>
        {points.map((point, index) => {
          const pointFrame = frame - (fps * (index + 1));
          const pointOpacity = Math.max(0, Math.min(1, pointFrame / 30));
          
          return (
            <p
              key={index}
              style={{
                fontSize: 40,
                color: colors.secondary,
                opacity: pointOpacity,
                margin: '20px 0',
              }}
            >
              • {point}
            </p>
          );
        })}
      </div>
    </AbsoluteFill>
  );
};
```

## Running the Application

```bash
# Development mode
npm run dev

# Build for production
npm run build

# Start production server
npm run start

# Render videos only
npm run render

# Run research crawler
npm run research -- --keyword "AI trends"
```

## Common Use Cases

### 1. Daily Content Automation

```typescript
// Schedule daily content generation
import { schedulePipeline } from '@/lib/scheduler';

schedulePipeline({
  cron: '0 9 * * *', // Daily at 9 AM
  topics: ['AI', 'Marketing', 'Technology'],
  autoPublish: false,
  notify: true,
});
```

### 2. Batch Content Creation

```typescript
const topics = [
  'AI Marketing Tools',
  'Content Automation',
  'Video Marketing Trends',
];

const batchResults = await Promise.all(
  topics.map(topic => 
    runContentPipeline({
      keyword: topic,
      research: { sources: ['techcrunch'], timeframe: '7d' },
      content: { formats: ['toplist'], languages: ['en'] },
      video: { enabled: true, templates: ['reels'] },
    })
  )
);
```

### 3. Multi-Language Content

```typescript
const multilingualContent = await generateContent({
  topic: 'Future of AI',
  format: 'pov',
  languages: ['en', 'vi', 'es', 'fr'],
  tone: 'expert',
  options: {
    localizeExamples: true,
    culturalAdaptation: true,
  }
});
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement rate limiting
import { Ratelimit } from '@upstash/ratelimit';
import { Redis } from '@upstash/redis';

const ratelimit = new Ratelimit({
  redis: Redis.fromEnv(),
  limiter: Ratelimit.slidingWindow(10, '1 m'),
});

async function generateWithRateLimit(prompt: string) {
  const { success } = await ratelimit.limit('api-generation');
  
  if (!success) {
    throw new Error('Rate limit exceeded. Please try again later.');
  }
  
  return await generateContent({ topic: prompt });
}
```

### Video Rendering Errors

```typescript
// Add error handling for video rendering
try {
  const video = await renderVideo(config);
} catch (error) {
  if (error.message.includes('ffmpeg')) {
    console.error('FFmpeg not installed. Install with: brew install ffmpeg');
  } else if (error.message.includes('memory')) {
    console.error('Insufficient memory. Try reducing video duration or quality.');
  }
  throw error;
}
```

### Research Data Quality

```typescript
// Validate and clean research data
function validateResearchData(data: ResearchResult) {
  return {
    ...data,
    articles: data.articles.filter(article => 
      article.title.length > 10 &&
      article.snippet.length > 50 &&
      article.url.startsWith('http')
    ),
    trends: data.trends.filter(trend => trend.length > 0),
  };
}
```

### Content Quality Control

```typescript
// Add quality checks before publishing
async function validateContentQuality(content: GeneratedContent) {
  const checks = {
    minWordCount: content.metadata.wordCount >= 500,
    hasSEO: content.metadata.seoScore >= 70,
    hasImages: content.content.includes('<img') || content.content.includes('!['),
    noPlaceholders: !content.content.includes('[INSERT'),
  };
  
  const passed = Object.values(checks).every(check => check);
  
  if (!passed) {
    console.warn('Quality checks failed:', checks);
  }
  
  return { passed, checks };
}
```

## Performance Optimization

```typescript
// Cache research results
import { cache } from '@/lib/cache';

async function getCachedResearch(keyword: string) {
  const cacheKey = `research:${keyword}`;
  const cached = await cache.get(cacheKey);
  
  if (cached) {
    return JSON.parse(cached);
  }
  
  const fresh = await researchTopic({ keyword });
  await cache.set(cacheKey, JSON.stringify(fresh), { ex: 3600 }); // 1 hour
  
  return fresh;
}

// Parallel processing
async function generateMultipleFormats(topic: string) {
  const [toplist, pov, caseStudy] = await Promise.all([
    generateContent({ topic, format: 'toplist' }),
    generateContent({ topic, format: 'pov' }),
    generateContent({ topic, format: 'case-study' }),
  ]);
  
  return { toplist, pov, caseStudy };
}
```

This pipeline automates the entire content creation workflow, from research to publication-ready content and videos, making it ideal for marketers, content creators, and businesses looking to scale their content production efficiently.
