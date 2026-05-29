---
name: marketing-pipeline-share-automation
description: AI-powered content automation pipeline for research, script generation, and video creation using Claude, OpenAI, and Remotion
triggers:
  - how do I automate content creation with AI
  - generate marketing content pipeline
  - create automated video content from text
  - research and generate content automatically
  - use marketing-pipeline-share for content automation
  - build AI content workflow with Claude and OpenAI
  - automate social media content generation
  - create content pipeline with Remotion video
---

# Marketing Pipeline Share - AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

Marketing Pipeline Share is a complete AI-powered content automation system that handles everything from research and content generation to automatic video rendering. It crawls real-time data from sources like TechCrunch, a16z, Twitter/X, and LinkedIn to create data-backed content in multiple formats and languages, then automatically generates videos using Remotion.

## What It Does

- **Auto-Research**: Crawls and analyzes fresh content from major tech news sources within the last 24 hours
- **AI Content Generation**: Uses Claude 3 and OpenAI to generate content in multiple formats (top lists, POV, case studies, how-tos)
- **Multi-language Support**: Generates content in both English and Vietnamese with customizable tone
- **Video Rendering**: Automatically creates infographics and short-form videos using Remotion
- **Platform Optimization**: Exports videos optimized for Reels, TikTok, and YouTube Shorts

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

## Required Environment Variables

```bash
# AI Services
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_url

# Optional: Content Sources
TWITTER_BEARER_TOKEN=your_twitter_token
```

## Running the Application

```bash
# Development mode
npm run dev

# Production build
npm run build
npm start

# Run video renderer (Remotion)
npm run render
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/                 # Next.js app directory
│   ├── components/          # React components
│   ├── lib/
│   │   ├── ai/             # AI integration (Claude, OpenAI)
│   │   ├── research/       # Content crawling & research
│   │   ├── video/          # Remotion video generation
│   │   └── utils/          # Shared utilities
│   └── types/              # TypeScript types
├── remotion/               # Video templates
└── public/                 # Static assets
```

## Core API Usage

### 1. Research Module - Auto Content Crawling

```typescript
import { researchContent } from '@/lib/research/crawler';
import { analyzeInsights } from '@/lib/research/analyzer';

// Crawl recent content by keyword
async function gatherResearch(keyword: string) {
  const sources = {
    techcrunch: true,
    a16z: true,
    twitter: true,
    linkedin: true
  };

  const rawData = await researchContent({
    keyword,
    sources,
    timeframe: '24h'
  });

  // Extract insights and data points
  const insights = await analyzeInsights(rawData, {
    extractStats: true,
    findTrends: true,
    identifyExperts: true
  });

  return insights;
}

// Usage
const aiResearch = await gatherResearch('artificial intelligence');
```

### 2. AI Content Generation

```typescript
import { generateContent } from '@/lib/ai/content-generator';
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY
});

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY
});

// Generate content with Claude
async function createArticle(research: any, format: string) {
  const content = await generateContent({
    provider: 'claude',
    client: anthropic,
    format: format, // 'toplist' | 'pov' | 'case-study' | 'how-to'
    research: research,
    language: 'both', // 'en' | 'vi' | 'both'
    tone: 'professional', // 'professional' | 'friendly' | 'humorous'
    model: 'claude-3-5-sonnet-20241022'
  });

  return content;
}

// Generate with OpenAI
async function createWithOpenAI(research: any) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: 'You are an expert content creator specializing in tech marketing.'
      },
      {
        role: 'user',
        content: `Create a top 10 list article based on this research: ${JSON.stringify(research)}`
      }
    ],
    temperature: 0.7
  });

  return completion.choices[0].message.content;
}
```

### 3. Content Format Templates

```typescript
import { formatContent } from '@/lib/ai/formatters';

// Top List Format
const topListArticle = await formatContent({
  type: 'toplist',
  title: '10 AI Tools Revolutionizing Marketing in 2026',
  research: insights,
  structure: {
    intro: true,
    itemCount: 10,
    includeStats: true,
    includeExamples: true,
    conclusion: true
  }
});

// POV (Point of View) Format
const povArticle = await formatContent({
  type: 'pov',
  perspective: 'industry-expert',
  research: insights,
  structure: {
    hook: true,
    personalStory: true,
    analysis: true,
    prediction: true
  }
});

// Case Study Format
const caseStudy = await formatContent({
  type: 'case-study',
  research: insights,
  structure: {
    problem: true,
    solution: true,
    implementation: true,
    results: true,
    takeaways: true
  }
});
```

### 4. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

// Render video from content
async function generateVideo(content: any, platform: string) {
  const compositionId = 'ContentVideo';
  
  // Bundle Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.join(process.cwd(), 'remotion/index.ts'),
    webpackOverride: (config) => config
  });

  // Get composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: {
      title: content.title,
      sections: content.sections,
      style: platform // 'reels' | 'tiktok' | 'shorts'
    }
  });

  // Render video
  const outputLocation = path.join(
    process.cwd(),
    'public/videos',
    `${content.slug}-${platform}.mp4`
  );

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: composition.defaultProps
  });

  return outputLocation;
}

// Platform-specific rendering
async function renderForAllPlatforms(content: any) {
  const platforms = ['reels', 'tiktok', 'shorts'];
  
  const videos = await Promise.all(
    platforms.map(platform => generateVideo(content, platform))
  );

  return videos;
}
```

### 5. Complete Pipeline Workflow

```typescript
import { ContentPipeline } from '@/lib/pipeline';

// Full automation pipeline
async function runContentPipeline(keyword: string) {
  const pipeline = new ContentPipeline({
    aiProvider: 'claude',
    videoEnabled: true,
    platforms: ['reels', 'tiktok', 'shorts'],
    languages: ['en', 'vi']
  });

  // Step 1: Research
  const research = await pipeline.research(keyword, {
    sources: ['techcrunch', 'a16z', 'twitter'],
    timeframe: '24h'
  });

  // Step 2: Generate content
  const articles = await pipeline.generateContent(research, {
    formats: ['toplist', 'pov'],
    count: 2
  });

  // Step 3: Create videos
  const videos = await pipeline.renderVideos(articles);

  // Step 4: Schedule posting (if configured)
  const scheduled = await pipeline.schedule({
    articles,
    videos,
    platforms: ['facebook', 'instagram', 'tiktok'],
    timing: 'optimal'
  });

  return {
    research,
    articles,
    videos,
    scheduled
  };
}

// Execute pipeline
const result = await runContentPipeline('AI marketing tools');
console.log(`Generated ${result.articles.length} articles and ${result.videos.length} videos`);
```

## Remotion Video Templates

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig } from 'remotion';
import { interpolate } from 'remotion';

export const ContentVideo: React.FC<{
  title: string;
  sections: Array<{ heading: string; content: string }>;
  style: string;
}> = ({ title, sections, style }) => {
  const frame = useCurrentFrame();
  const { fps, durationInFrames } = useVideoConfig();

  const opacity = interpolate(
    frame,
    [0, 30],
    [0, 1],
    { extrapolateRight: 'clamp' }
  );

  return (
    <AbsoluteFill style={{ backgroundColor: '#000' }}>
      <div style={{ opacity, padding: 40, color: '#fff' }}>
        <h1 style={{ fontSize: style === 'shorts' ? 48 : 72 }}>
          {title}
        </h1>
        {sections.map((section, idx) => (
          <div key={idx}>
            <h2>{section.heading}</h2>
            <p>{section.content}</p>
          </div>
        ))}
      </div>
    </AbsoluteFill>
  );
};
```

## Configuration

```typescript
// config/pipeline.config.ts
export const pipelineConfig = {
  research: {
    sources: {
      techcrunch: { enabled: true, priority: 1 },
      a16z: { enabled: true, priority: 1 },
      twitter: { enabled: true, priority: 2 },
      linkedin: { enabled: false, priority: 3 }
    },
    timeframe: '24h',
    maxArticles: 50
  },
  
  ai: {
    defaultProvider: 'claude',
    models: {
      claude: 'claude-3-5-sonnet-20241022',
      openai: 'gpt-4-turbo-preview'
    },
    temperature: 0.7,
    maxTokens: 4000
  },
  
  content: {
    defaultLanguages: ['en', 'vi'],
    defaultTone: 'professional',
    includeStats: true,
    includeCitations: true
  },
  
  video: {
    defaultPlatforms: ['reels', 'tiktok', 'shorts'],
    dimensions: {
      reels: { width: 1080, height: 1920 },
      tiktok: { width: 1080, height: 1920 },
      shorts: { width: 1080, height: 1920 }
    },
    fps: 30,
    defaultDuration: 60 // seconds
  }
};
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement retry logic with exponential backoff
import { retry } from '@/lib/utils/retry';

const content = await retry(
  () => generateContent(research),
  {
    maxAttempts: 3,
    delayMs: 1000,
    backoff: 'exponential'
  }
);
```

### Video Rendering Memory Issues

```typescript
// Use chunked rendering for long videos
import { renderFrames } from '@remotion/renderer';

const frames = await renderFrames({
  composition,
  serveUrl: bundleLocation,
  onProgress: ({ renderedFrames, encodedFrames }) => {
    console.log(`Rendered ${renderedFrames}, Encoded ${encodedFrames}`);
  },
  concurrency: 2 // Reduce if memory issues persist
});
```

### Research Data Quality

```typescript
// Filter and validate research data
import { validateResearch } from '@/lib/research/validator';

const validatedData = await validateResearch(rawData, {
  minQualityScore: 0.7,
  requireStats: true,
  requireSources: true,
  deduplicateContent: true
});
```

## Common Patterns

### Batch Content Generation

```typescript
async function batchGenerate(keywords: string[]) {
  const results = [];
  
  for (const keyword of keywords) {
    const research = await gatherResearch(keyword);
    const content = await createArticle(research, 'toplist');
    results.push(content);
    
    // Rate limiting
    await new Promise(resolve => setTimeout(resolve, 2000));
  }
  
  return results;
}
```

### Custom AI Prompts

```typescript
const customPrompt = `
Create a comprehensive article about ${topic}.
Research data: ${JSON.stringify(research)}

Requirements:
- Include at least 5 data points
- Cite all sources
- Use engaging storytelling
- Target audience: ${audience}
- Tone: ${tone}
- Language: ${language}
`;

const response = await anthropic.messages.create({
  model: 'claude-3-5-sonnet-20241022',
  max_tokens: 4000,
  messages: [{ role: 'user', content: customPrompt }]
});
```

This skill enables AI agents to leverage the complete marketing content automation pipeline, from intelligent research to multi-platform video distribution.
