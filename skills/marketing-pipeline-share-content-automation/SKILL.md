---
name: marketing-pipeline-share-content-automation
description: AI-powered content pipeline for automated research, scriptwriting, and video generation using Claude, OpenAI, and Remotion
triggers:
  - automate content creation with AI research and video generation
  - set up marketing pipeline for automated content workflow
  - generate videos from AI-written content automatically
  - create content pipeline with Claude and OpenAI
  - build automated marketing content system
  - research and generate social media content with AI
  - automate content from research to video publication
  - set up AI content automation pipeline
---

# Marketing Pipeline Share - AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with Marketing Pipeline Share, an end-to-end content automation system that handles research, scriptwriting, and video generation. The pipeline uses Claude 3/OpenAI for content generation and Remotion for video rendering.

## What It Does

Marketing Pipeline Share automates the entire content creation workflow:

1. **Auto-Research**: Crawls latest news from TechCrunch, a16z, Twitter/X, LinkedIn (last 24h)
2. **AI Content Generation**: Creates articles in multiple formats (Toplist, POV, Case Study, How-to) in English and Vietnamese
3. **Video Rendering**: Automatically generates infographics and short videos using Remotion
4. **Multi-Platform Export**: Outputs optimized content for Reels, TikTok, Shorts

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

Create a `.env` file with the following variables:

```bash
# AI APIs
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# Research/Crawling APIs
RAPIDAPI_KEY=your_rapidapi_key

# Content Sources (optional customization)
TECHCRUNCH_API_URL=https://api.techcrunch.com
TWITTER_API_KEY=your_twitter_api_key

# Database (if used)
DATABASE_URL=your_database_url

# Next.js
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/                 # Next.js app directory
│   ├── components/          # React components
│   ├── lib/
│   │   ├── ai/             # AI integration (Claude, OpenAI)
│   │   ├── crawler/        # News crawling logic
│   │   ├── content/        # Content generation
│   │   └── video/          # Remotion video rendering
│   └── types/              # TypeScript definitions
├── remotion/               # Remotion video templates
└── public/                 # Static assets
```

## Core API Usage

### 1. Research & Data Crawling

```typescript
import { crawlLatestNews } from '@/lib/crawler/news-crawler';
import { analyzeInsights } from '@/lib/ai/insight-analyzer';

// Crawl recent content from multiple sources
async function gatherResearch(keyword: string) {
  const sources = ['techcrunch', 'a16z', 'twitter', 'linkedin'];
  
  const newsData = await crawlLatestNews({
    keyword,
    sources,
    timeframe: '24h',
    limit: 20
  });
  
  // Extract insights using AI
  const insights = await analyzeInsights(newsData, {
    model: 'claude-3-opus',
    apiKey: process.env.ANTHROPIC_API_KEY!
  });
  
  return insights;
}
```

### 2. AI Content Generation

```typescript
import { generateContent } from '@/lib/content/generator';
import { ContentFormat, Language } from '@/types/content';

async function createArticle(topic: string) {
  // Generate bilingual content
  const content = await generateContent({
    topic,
    format: ContentFormat.TOPLIST, // or POV, CASE_STUDY, HOW_TO
    languages: [Language.ENGLISH, Language.VIETNAMESE],
    tone: 'professional', // or 'friendly', 'humorous'
    researchData: await gatherResearch(topic),
    aiProvider: 'claude', // or 'openai'
    apiKey: process.env.ANTHROPIC_API_KEY!
  });
  
  return content;
}
```

### 3. Video Generation with Remotion

```typescript
import { renderVideo } from '@/lib/video/renderer';
import { VideoTemplate } from '@/types/video';

async function generateVideoFromContent(content: any) {
  const videoConfig = {
    template: VideoTemplate.INFOGRAPHIC,
    content: {
      title: content.title,
      keyPoints: content.keyPoints,
      stats: content.statistics
    },
    aspectRatio: '9:16', // For Reels/TikTok/Shorts
    duration: 30, // seconds
    outputPath: './output/videos'
  };
  
  const videoUrl = await renderVideo(videoConfig);
  return videoUrl;
}
```

### 4. Complete Pipeline Execution

```typescript
import { runContentPipeline } from '@/lib/pipeline';

async function executeFullPipeline(keyword: string) {
  const result = await runContentPipeline({
    keyword,
    steps: {
      research: true,
      contentGeneration: true,
      videoRendering: true,
      autoPublish: false // Set true to auto-post
    },
    config: {
      contentFormats: ['toplist', 'case-study'],
      languages: ['en', 'vi'],
      videoFormats: ['reels', 'tiktok'],
      aiModel: 'claude-3-sonnet'
    }
  });
  
  console.log('Generated Articles:', result.articles);
  console.log('Generated Videos:', result.videos);
  
  return result;
}
```

## Content Format Examples

### Toplist Format

```typescript
import { ToplistGenerator } from '@/lib/content/formats/toplist';

const generator = new ToplistGenerator({
  apiKey: process.env.ANTHROPIC_API_KEY!,
  model: 'claude-3-opus'
});

const toplist = await generator.create({
  title: 'Top 10 AI Marketing Tools 2024',
  itemCount: 10,
  includeStats: true,
  researchData: researchResults
});
```

### POV (Point of View) Format

```typescript
import { POVGenerator } from '@/lib/content/formats/pov';

const generator = new POVGenerator({
  apiKey: process.env.OPENAI_API_KEY!,
  model: 'gpt-4-turbo'
});

const povArticle = await generator.create({
  perspective: 'industry-expert',
  topic: 'The Future of AI in Marketing',
  tone: 'thought-leadership',
  wordCount: 1500
});
```

## CLI Commands

```bash
# Run development server
npm run dev

# Generate content from keyword
npm run generate -- --keyword "AI Marketing" --format toplist

# Render video from content
npm run render-video -- --input ./content/article.json --template infographic

# Execute full pipeline
npm run pipeline -- --keyword "Marketing Automation" --all

# Build for production
npm run build

# Start production server
npm start
```

## API Routes (Next.js)

### Generate Content Endpoint

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { generateContent } from '@/lib/content/generator';

export async function POST(request: NextRequest) {
  const { keyword, format, languages } = await request.json();
  
  try {
    const content = await generateContent({
      topic: keyword,
      format,
      languages,
      aiProvider: 'claude',
      apiKey: process.env.ANTHROPIC_API_KEY!
    });
    
    return NextResponse.json({ success: true, content });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

### Research Endpoint

```typescript
// app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { crawlLatestNews } from '@/lib/crawler/news-crawler';

export async function POST(request: NextRequest) {
  const { keyword, sources } = await request.json();
  
  const newsData = await crawlLatestNews({
    keyword,
    sources: sources || ['techcrunch', 'a16z'],
    timeframe: '24h'
  });
  
  return NextResponse.json({ data: newsData });
}
```

## Common Patterns

### Custom Content Template

```typescript
import { ContentTemplate } from '@/lib/content/templates/base';

class CustomTemplate extends ContentTemplate {
  async generate(params: any) {
    const prompt = this.buildPrompt(params);
    
    const response = await this.callAI({
      prompt,
      model: params.model || 'claude-3-sonnet',
      temperature: 0.7,
      maxTokens: 2000
    });
    
    return this.formatOutput(response);
  }
  
  private buildPrompt(params: any): string {
    return `Create a ${params.format} article about ${params.topic}
    with the following research data: ${JSON.stringify(params.researchData)}
    
    Requirements:
    - Tone: ${params.tone}
    - Language: ${params.language}
    - Include data-backed insights
    - Optimize for ${params.platform}`;
  }
}
```

### Video Template Customization

```typescript
// remotion/templates/CustomInfographic.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig } from 'remotion';

export const CustomInfographic: React.FC<{
  title: string;
  dataPoints: Array<{ label: string; value: number }>;
}> = ({ title, dataPoints }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  
  const opacity = Math.min(1, frame / (fps * 0.5));
  
  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a1a' }}>
      <div style={{ opacity }}>
        <h1 style={{ fontSize: 60, color: 'white' }}>{title}</h1>
        {dataPoints.map((point, i) => (
          <div key={i} style={{ 
            fontSize: 40, 
            color: '#00ff88',
            marginTop: 20
          }}>
            {point.label}: {point.value}
          </div>
        ))}
      </div>
    </AbsoluteFill>
  );
};
```

## Troubleshooting

### API Rate Limits

```typescript
import { RateLimiter } from '@/lib/utils/rate-limiter';

const limiter = new RateLimiter({
  requestsPerMinute: 60,
  provider: 'claude'
});

async function safeAPICall(prompt: string) {
  await limiter.wait();
  return await callClaudeAPI(prompt);
}
```

### Error Handling in Pipeline

```typescript
import { PipelineError } from '@/lib/errors';

try {
  await runContentPipeline(config);
} catch (error) {
  if (error instanceof PipelineError) {
    console.error('Pipeline failed at step:', error.step);
    console.error('Reason:', error.message);
    
    // Retry failed step
    if (error.retryable) {
      await retryStep(error.step, error.context);
    }
  }
}
```

### Video Rendering Timeouts

```typescript
import { renderVideo } from '@/lib/video/renderer';

const videoUrl = await renderVideo(config, {
  timeout: 300000, // 5 minutes
  onProgress: (progress) => {
    console.log(`Rendering: ${progress}%`);
  },
  fallbackQuality: 'medium' // Use lower quality on timeout
});
```

### Missing Research Data

```typescript
import { validateResearchData } from '@/lib/utils/validators';

const researchData = await crawlLatestNews({ keyword });

if (!validateResearchData(researchData)) {
  console.warn('Insufficient research data, using fallback sources');
  
  const fallbackData = await fetchFromArchive(keyword);
  return fallbackData;
}
```

## Advanced Configuration

### Multi-Model Fallback

```typescript
const aiConfig = {
  primary: {
    provider: 'claude',
    model: 'claude-3-opus',
    apiKey: process.env.ANTHROPIC_API_KEY!
  },
  fallback: {
    provider: 'openai',
    model: 'gpt-4-turbo',
    apiKey: process.env.OPENAI_API_KEY!
  }
};

const content = await generateContent({
  topic: 'AI Marketing',
  aiConfig
});
```

### Scheduled Pipeline Execution

```typescript
import { scheduleContentGeneration } from '@/lib/scheduler';

scheduleContentGeneration({
  cron: '0 9 * * *', // Daily at 9 AM
  keywords: ['AI', 'Marketing', 'SaaS'],
  autoPublish: true,
  platforms: ['linkedin', 'twitter']
});
```
