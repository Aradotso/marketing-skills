---
name: marketing-pipeline-share-automation
description: AI-powered content automation pipeline for research, scriptwriting, and video generation using Claude, OpenAI, and Remotion
triggers:
  - how do I automate content creation with AI
  - set up an AI content pipeline with research and video generation
  - create automated marketing content with Claude and OpenAI
  - generate videos from written content automatically
  - build a content automation workflow
  - scrape news and generate social media content
  - automate content research and scriptwriting
  - use Remotion to render marketing videos
---

# Marketing Pipeline Share Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

**marketing-pipeline-share** is a comprehensive TypeScript-based content automation system that handles the entire content creation pipeline: from automated research (crawling news sources like TechCrunch, a16z, Twitter, LinkedIn), to AI-powered scriptwriting (using Claude 3 and OpenAI), to automatic video generation (via Remotion). The system produces multi-format content in multiple languages, optimized for social media platforms.

### Core Capabilities

- **Auto-Research**: Crawls live news sources for trending topics and data
- **AI Content Generation**: Creates articles in multiple formats (listicles, POV, case studies, how-tos)
- **Multi-Language Support**: Generates content in English and Vietnamese
- **Video Rendering**: Automatically converts text content into videos/infographics
- **Multi-Platform Optimization**: Outputs content optimized for Reels, TikTok, Shorts

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

### Required Environment Variables

Create a `.env.local` file in the project root:

```env
# AI Providers
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key
SERPER_API_KEY=your_serper_key

# Optional: Database
DATABASE_URL=postgresql://user:password@localhost:5432/contentdb

# Optional: Social Media Auto-Post
FACEBOOK_PAGE_ACCESS_TOKEN=your_fb_token
LINKEDIN_ACCESS_TOKEN=your_linkedin_token
```

### Development Setup

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Start production server
npm start
```

## Project Architecture

```
marketing-pipeline-share/
├── src/
│   ├── lib/
│   │   ├── ai/              # AI providers (Claude, OpenAI)
│   │   ├── research/        # News crawling & research
│   │   ├── content/         # Content generation logic
│   │   └── video/           # Remotion video rendering
│   ├── app/                 # Next.js app routes
│   ├── components/          # React components
│   └── remotion/            # Video templates
├── public/
└── scripts/                 # CLI automation scripts
```

## Core Usage Patterns

### 1. Research & Data Crawling

```typescript
import { researchTopic } from '@/lib/research/crawler';

async function gatherResearch(keyword: string) {
  const research = await researchTopic({
    keyword: keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeRange: '24h',
    language: 'en'
  });

  return {
    articles: research.articles,
    insights: research.insights,
    statistics: research.statistics,
    trends: research.trendingTopics
  };
}

// Example: Research AI trends
const aiResearch = await gatherResearch('artificial intelligence 2026');
console.log(aiResearch.insights);
```

### 2. AI Content Generation with Claude

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

async function generateArticle(research: any, format: string) {
  const prompt = `
Based on this research data:
${JSON.stringify(research, null, 2)}

Create a ${format} article with:
- Engaging headline
- Data-backed insights
- Actionable takeaways
- SEO-optimized structure

Target audience: Marketing professionals
Tone: Professional yet approachable
Length: 1200-1500 words
`;

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4000,
    messages: [{
      role: 'user',
      content: prompt
    }]
  });

  return message.content[0].text;
}

// Generate toplist article
const article = await generateArticle(aiResearch, 'toplist');
```

### 3. Multi-Language Content Generation

```typescript
import { generateMultiLanguageContent } from '@/lib/content/generator';

async function createBilingualContent(topic: string) {
  const content = await generateMultiLanguageContent({
    topic: topic,
    languages: ['en', 'vi'],
    format: 'how-to',
    tone: 'friendly',
    includeExamples: true
  });

  return {
    english: content.en,
    vietnamese: content.vi,
    metadata: {
      seo: content.seoData,
      hashtags: content.suggestedHashtags,
      categories: content.categories
    }
  };
}
```

### 4. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import { webpackOverride } from './src/remotion/webpack-override';

async function renderContentVideo(article: any) {
  // Bundle Remotion project
  const bundleLocation = await bundle({
    entryPoint: './src/remotion/index.ts',
    webpackOverride: webpackOverride,
  });

  // Select composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title: article.title,
      points: article.keyPoints,
      branding: {
        logo: '/logo.png',
        colors: {
          primary: '#FF6B6B',
          secondary: '#4ECDC4'
        }
      }
    },
  });

  // Render video
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: `out/${article.slug}.mp4`,
    inputProps: composition.props,
  });

  return `out/${article.slug}.mp4`;
}
```

### 5. Complete Pipeline Automation

```typescript
import { runContentPipeline } from '@/lib/pipeline';

async function automateContentCreation(keyword: string) {
  const pipeline = await runContentPipeline({
    // Step 1: Research
    research: {
      keyword: keyword,
      depth: 'comprehensive',
      sources: ['all']
    },

    // Step 2: Content Generation
    content: {
      formats: ['toplist', 'case-study'],
      languages: ['en', 'vi'],
      aiProvider: 'claude', // or 'openai'
      tone: 'professional'
    },

    // Step 3: Video Rendering
    video: {
      enabled: true,
      platforms: ['reels', 'tiktok', 'shorts'],
      aspectRatios: ['9:16', '1:1'],
      duration: 60 // seconds
    },

    // Step 4: Auto-publish (optional)
    publish: {
      enabled: false, // set true to auto-post
      platforms: ['facebook', 'linkedin'],
      schedule: new Date('2026-06-10T10:00:00Z')
    }
  });

  return {
    articles: pipeline.generatedContent,
    videos: pipeline.renderedVideos,
    analytics: pipeline.metrics
  };
}

// Run the full pipeline
const result = await automateContentCreation('AI marketing automation');
```

## API Routes

### Research Endpoint

```typescript
// app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { researchTopic } from '@/lib/research/crawler';

export async function POST(req: NextRequest) {
  const { keyword, sources, timeRange } = await req.json();

  const research = await researchTopic({
    keyword,
    sources: sources || ['techcrunch', 'a16z'],
    timeRange: timeRange || '24h'
  });

  return NextResponse.json(research);
}
```

### Content Generation Endpoint

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { generateArticle } from '@/lib/content/generator';

export async function POST(req: NextRequest) {
  const { research, format, language, tone } = await req.json();

  const article = await generateArticle({
    researchData: research,
    format: format || 'toplist',
    language: language || 'en',
    tone: tone || 'professional'
  });

  return NextResponse.json({ article });
}
```

### Video Rendering Endpoint

```typescript
// app/api/render/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { renderContentVideo } from '@/lib/video/renderer';

export async function POST(req: NextRequest) {
  const { article, platform } = await req.json();

  const videoPath = await renderContentVideo({
    content: article,
    platform: platform || 'reels',
    aspectRatio: platform === 'reels' ? '9:16' : '1:1'
  });

  return NextResponse.json({ videoUrl: videoPath });
}
```

## CLI Usage

```bash
# Research a topic
node scripts/research.js --keyword "AI trends" --sources techcrunch,a16z

# Generate content
node scripts/generate.js --topic "AI marketing" --format toplist --lang en,vi

# Render video
node scripts/render-video.js --article ./content/ai-marketing.json --platform reels

# Run full pipeline
node scripts/pipeline.js --keyword "AI automation" --auto-publish false
```

## Configuration

### Custom Content Templates

```typescript
// src/lib/content/templates.ts
export const contentTemplates = {
  toplist: {
    structure: [
      'engaging_intro',
      'numbered_list',
      'actionable_conclusion'
    ],
    minItems: 5,
    includeStats: true
  },
  
  caseStudy: {
    structure: [
      'problem_statement',
      'solution_approach',
      'results_metrics',
      'lessons_learned'
    ],
    includeQuotes: true,
    dataVisuals: true
  },

  howTo: {
    structure: [
      'clear_objective',
      'step_by_step',
      'tips_warnings',
      'summary'
    ],
    includeImages: true,
    difficulty: 'beginner' // or 'intermediate', 'advanced'
  }
};
```

### Video Template Customization

```typescript
// src/remotion/compositions/ContentVideo.tsx
import { AbsoluteFill, Sequence, useCurrentFrame } from 'remotion';

export const ContentVideo: React.FC<{
  title: string;
  points: string[];
  branding: any;
}> = ({ title, points, branding }) => {
  const frame = useCurrentFrame();

  return (
    <AbsoluteFill style={{ backgroundColor: branding.colors.primary }}>
      <Sequence from={0} durationInFrames={90}>
        <h1 style={{ fontSize: 60, color: 'white' }}>{title}</h1>
      </Sequence>
      
      {points.map((point, i) => (
        <Sequence key={i} from={90 + i * 120} durationInFrames={120}>
          <div style={{ padding: 40 }}>
            <h2>{point}</h2>
          </div>
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};
```

## Common Workflows

### Daily Content Automation

```typescript
import cron from 'node-cron';
import { automateContentCreation } from '@/lib/pipeline';

// Run every day at 8 AM
cron.schedule('0 8 * * *', async () => {
  const topics = [
    'AI marketing trends',
    'Social media automation',
    'Content strategy 2026'
  ];

  for (const topic of topics) {
    await automateContentCreation(topic);
  }
});
```

### Batch Video Generation

```typescript
async function batchRenderVideos(articles: any[]) {
  const videos = await Promise.all(
    articles.map(article => 
      renderContentVideo({
        content: article,
        platforms: ['reels', 'tiktok', 'shorts']
      })
    )
  );

  return videos;
}
```

## Troubleshooting

### AI API Rate Limits

```typescript
// Implement retry logic with exponential backoff
async function generateWithRetry(prompt: string, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await anthropic.messages.create({
        model: 'claude-3-5-sonnet-20241022',
        max_tokens: 4000,
        messages: [{ role: 'user', content: prompt }]
      });
    } catch (error) {
      if (error.status === 429 && i < maxRetries - 1) {
        await new Promise(resolve => setTimeout(resolve, Math.pow(2, i) * 1000));
        continue;
      }
      throw error;
    }
  }
}
```

### Video Rendering Memory Issues

```typescript
// Reduce concurrency for video rendering
import pLimit from 'p-limit';

const limit = pLimit(2); // Max 2 concurrent renders

const videos = await Promise.all(
  articles.map(article => 
    limit(() => renderContentVideo(article))
  )
);
```

### Research Data Quality

```typescript
// Validate and filter research results
function validateResearch(data: any) {
  return {
    articles: data.articles.filter(a => 
      a.publishedDate > Date.now() - 24 * 60 * 60 * 1000 &&
      a.content.length > 500
    ),
    insights: data.insights.filter(i => i.confidence > 0.7)
  };
}
```

## Best Practices

1. **Cache research data** to avoid redundant API calls
2. **Use queue systems** (Bull, BeeQueue) for video rendering jobs
3. **Implement content moderation** before auto-publishing
4. **Monitor AI costs** with usage tracking
5. **Version control** generated content for rollback capability
6. **Test video outputs** across target platforms before bulk rendering
