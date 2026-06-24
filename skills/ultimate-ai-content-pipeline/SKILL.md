---
name: ultimate-ai-content-pipeline
description: Automated content creation pipeline from research to video generation using Claude/OpenAI and Remotion
triggers:
  - how do I set up the AI content pipeline
  - generate automated content with research
  - create videos from blog posts automatically
  - use Claude for content generation pipeline
  - automate content research and video creation
  - set up Remotion video rendering for content
  - crawl news and generate content automatically
  - build an automated marketing content system
---

# Ultimate AI Content Pipeline Skill

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

Ultimate AI Content Pipeline is an end-to-end automated content creation system built with TypeScript/Next.js that:

- **Auto-crawls** news sources (TechCrunch, a16z, Twitter, LinkedIn) for fresh data
- **Generates content** in multiple formats (listicles, case studies, how-tos) using Claude 3 or OpenAI
- **Supports bilingual** output (English/Vietnamese) with customizable tone
- **Renders videos** automatically using Remotion from written content
- **Optimizes for platforms** like Reels, TikTok, and YouTube Shorts

This system transforms a single keyword into publication-ready articles and social media videos.

## Installation

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
pnpm install
# or
yarn install
```

## Environment Configuration

Create a `.env.local` file in the root directory:

```env
# AI Provider APIs
ANTHROPIC_API_KEY=your_claude_api_key_here
OPENAI_API_KEY=your_openai_api_key_here

# Research/Crawling APIs
RAPIDAPI_KEY=your_rapidapi_key_here

# Optional: Database
DATABASE_URL=postgresql://user:password@localhost:5432/content_db

# Remotion (for video rendering)
REMOTION_LICENSE_KEY=your_remotion_license_key_here

# Next.js
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app router pages
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI provider integrations
│   │   ├── crawler/     # News crawling logic
│   │   ├── content/     # Content generation pipelines
│   │   └── video/       # Remotion video rendering
│   ├── remotion/        # Remotion video compositions
│   └── types/           # TypeScript type definitions
├── public/              # Static assets
└── package.json
```

## Running the Application

```bash
# Development mode
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Render Remotion video (if standalone)
npm run remotion:render
```

The app runs at `http://localhost:3000` by default.

## Core API Usage

### 1. Content Research Pipeline

Crawl and analyze news sources for a topic:

```typescript
import { researchTopic } from '@/lib/crawler/research';

async function gatherInsights(keyword: string) {
  const research = await researchTopic({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeframe: '24h',
    limit: 10
  });

  return {
    articles: research.articles,
    insights: research.insights,
    statistics: research.statistics,
    trends: research.trends
  };
}

// Usage
const data = await gatherInsights('artificial intelligence marketing');
console.log(data.insights);
```

### 2. AI Content Generation

Generate content using Claude or OpenAI:

```typescript
import { generateContent } from '@/lib/ai/content-generator';

async function createArticle(topic: string, format: string) {
  const content = await generateContent({
    provider: 'claude', // or 'openai'
    topic,
    format, // 'toplist', 'case-study', 'how-to', 'pov'
    language: 'en', // or 'vi' for Vietnamese
    tone: 'professional', // 'friendly', 'humorous'
    researchData: await gatherInsights(topic),
    includeStats: true,
    wordCount: 1500
  });

  return {
    title: content.title,
    body: content.body,
    metadata: content.metadata,
    keywords: content.keywords
  };
}

// Usage
const article = await createArticle(
  'AI automation in content marketing',
  'how-to'
);
```

### 3. Bilingual Content Generation

Generate content in multiple languages simultaneously:

```typescript
import { generateBilingualContent } from '@/lib/content/bilingual';

async function createBilingualPost(topic: string) {
  const content = await generateBilingualContent({
    topic,
    languages: ['en', 'vi'],
    format: 'toplist',
    tone: 'professional'
  });

  return {
    english: content.en,
    vietnamese: content.vi,
    sharedMetadata: content.metadata
  };
}
```

### 4. Video Generation with Remotion

Render video from content:

```typescript
import { renderVideo } from '@/lib/video/remotion-renderer';
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';

async function createContentVideo(article: any) {
  const videoData = {
    title: article.title,
    keyPoints: article.keyPoints,
    statistics: article.statistics,
    duration: 30, // seconds
    format: 'reel' // 'reel', 'tiktok', 'shorts'
  };

  const outputPath = await renderVideo({
    compositionId: 'ContentHighlight',
    inputProps: videoData,
    outputFile: `./output/${article.slug}.mp4`,
    codec: 'h264',
    fps: 30
  });

  return outputPath;
}

// Full pipeline example
async function fullContentPipeline(keyword: string) {
  // 1. Research
  const research = await gatherInsights(keyword);
  
  // 2. Generate content
  const article = await generateContent({
    provider: 'claude',
    topic: keyword,
    format: 'toplist',
    language: 'en',
    researchData: research
  });
  
  // 3. Create video
  const videoPath = await createContentVideo(article);
  
  return {
    article,
    videoPath,
    research
  };
}
```

## Remotion Video Compositions

Example Remotion composition for content highlights:

```typescript
// src/remotion/ContentHighlight.tsx
import { AbsoluteFill, useCurrentFrame, interpolate } from 'remotion';

interface ContentHighlightProps {
  title: string;
  keyPoints: string[];
  duration: number;
}

export const ContentHighlight: React.FC<ContentHighlightProps> = ({
  title,
  keyPoints,
  duration
}) => {
  const frame = useCurrentFrame();
  const opacity = interpolate(frame, [0, 30], [0, 1]);

  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a1a' }}>
      <div style={{ opacity, padding: '40px', color: 'white' }}>
        <h1 style={{ fontSize: '48px', marginBottom: '30px' }}>
          {title}
        </h1>
        <ul style={{ fontSize: '24px', lineHeight: '1.6' }}>
          {keyPoints.map((point, i) => (
            <li key={i} style={{ marginBottom: '20px' }}>
              {point}
            </li>
          ))}
        </ul>
      </div>
    </AbsoluteFill>
  );
};
```

Register composition:

```typescript
// src/remotion/index.ts
import { Composition } from 'remotion';
import { ContentHighlight } from './ContentHighlight';

export const RemotionRoot = () => {
  return (
    <>
      <Composition
        id="ContentHighlight"
        component={ContentHighlight}
        durationInFrames={900} // 30 seconds at 30fps
        fps={30}
        width={1080}
        height={1920} // Vertical format for Reels/TikTok
        defaultProps={{
          title: 'Default Title',
          keyPoints: [],
          duration: 30
        }}
      />
    </>
  );
};
```

## API Routes (Next.js)

### Generate Content Endpoint

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { generateContent } from '@/lib/ai/content-generator';
import { researchTopic } from '@/lib/crawler/research';

export async function POST(request: NextRequest) {
  try {
    const { topic, format, language, provider } = await request.json();

    // Research phase
    const research = await researchTopic({
      keyword: topic,
      sources: ['techcrunch', 'twitter'],
      timeframe: '24h'
    });

    // Generation phase
    const content = await generateContent({
      provider: provider || 'claude',
      topic,
      format: format || 'toplist',
      language: language || 'en',
      tone: 'professional',
      researchData: research
    });

    return NextResponse.json({
      success: true,
      content,
      research: research.insights
    });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

### Video Rendering Endpoint

```typescript
// src/app/api/render-video/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { renderVideo } from '@/lib/video/remotion-renderer';

export async function POST(request: NextRequest) {
  try {
    const { articleId, format } = await request.json();
    
    // Fetch article data
    const article = await getArticleById(articleId);
    
    // Render video
    const videoPath = await renderVideo({
      compositionId: 'ContentHighlight',
      inputProps: {
        title: article.title,
        keyPoints: article.keyPoints,
        duration: 30
      },
      outputFile: `./public/videos/${articleId}.mp4`,
      format: format || 'reel'
    });

    return NextResponse.json({
      success: true,
      videoUrl: `/videos/${articleId}.mp4`
    });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

## Content Format Templates

### Toplist Format

```typescript
export const toplistTemplate = {
  provider: 'claude',
  systemPrompt: `You are an expert content writer specializing in toplist articles.
Create engaging, well-researched lists with:
- Compelling introduction
- 5-10 numbered items with detailed explanations
- Data-backed insights
- Actionable takeaways
- SEO-optimized structure`,
  format: {
    structure: ['intro', 'items', 'conclusion'],
    itemFormat: '{number}. {title}\n{description}\n{stats}'
  }
};
```

### Case Study Format

```typescript
export const caseStudyTemplate = {
  provider: 'claude',
  systemPrompt: `Create detailed case studies with:
- Company/situation background
- Challenge identification
- Solution implementation
- Results with metrics
- Key learnings`,
  format: {
    structure: ['background', 'challenge', 'solution', 'results', 'learnings'],
    includeCharts: true
  }
};
```

### How-To Format

```typescript
export const howToTemplate = {
  provider: 'openai',
  systemPrompt: `Write comprehensive how-to guides with:
- Clear step-by-step instructions
- Prerequisites section
- Visual descriptions
- Tips and warnings
- Expected outcomes`,
  format: {
    structure: ['intro', 'prerequisites', 'steps', 'tips', 'conclusion'],
    stepFormat: 'Step {n}: {title}\n{instructions}'
  }
};
```

## Common Patterns

### Full Automated Pipeline

```typescript
import { 
  researchTopic, 
  generateContent, 
  renderVideo,
  publishToSocial 
} from '@/lib';

async function automatedContentFlow(keyword: string) {
  console.log('Starting automated pipeline for:', keyword);

  // Step 1: Research
  const research = await researchTopic({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter'],
    timeframe: '24h',
    limit: 15
  });

  // Step 2: Generate bilingual content
  const content = await generateBilingualContent({
    topic: keyword,
    languages: ['en', 'vi'],
    format: 'toplist',
    tone: 'professional',
    researchData: research
  });

  // Step 3: Create video versions
  const videos = await Promise.all([
    renderVideo({
      compositionId: 'ContentHighlight',
      inputProps: {
        title: content.en.title,
        keyPoints: content.en.keyPoints,
        duration: 30
      },
      format: 'reel'
    }),
    renderVideo({
      compositionId: 'ContentHighlight',
      inputProps: {
        title: content.vi.title,
        keyPoints: content.vi.keyPoints,
        duration: 30
      },
      format: 'tiktok'
    })
  ]);

  // Step 4: Return complete package
  return {
    articles: content,
    videos,
    research: research.insights,
    readyToPublish: true
  };
}
```

### Scheduling Content Generation

```typescript
import cron from 'node-cron';

// Run content generation daily at 6 AM
cron.schedule('0 6 * * *', async () => {
  const topics = await getTrendingTopics();
  
  for (const topic of topics) {
    try {
      const result = await automatedContentFlow(topic);
      await saveToDatabase(result);
      console.log(`Generated content for: ${topic}`);
    } catch (error) {
      console.error(`Failed for ${topic}:`, error);
    }
  }
});
```

## Troubleshooting

### API Rate Limits

```typescript
import { RateLimiter } from '@/lib/utils/rate-limiter';

const claudeLimiter = new RateLimiter({
  maxRequests: 50,
  perMinutes: 1
});

async function generateWithRateLimit(topic: string) {
  await claudeLimiter.wait();
  return await generateContent({ provider: 'claude', topic });
}
```

### Video Rendering Memory Issues

```typescript
// Use chunked rendering for long videos
import { renderVideoChunked } from '@/lib/video/chunked-renderer';

async function renderLongVideo(content: any) {
  return await renderVideoChunked({
    compositionId: 'LongFormContent',
    inputProps: content,
    chunkSize: 300, // frames per chunk
    outputFile: './output/long-video.mp4'
  });
}
```

### AI Response Parsing

```typescript
function parseAIResponse(response: string) {
  try {
    // Try JSON first
    return JSON.parse(response);
  } catch {
    // Fallback to markdown parsing
    return {
      title: extractTitle(response),
      body: extractBody(response),
      keyPoints: extractKeyPoints(response)
    };
  }
}
```

### Crawler Blocking

```typescript
import { createProxyAgent } from '@/lib/crawler/proxy';

async function crawlWithProxy(url: string) {
  const agent = createProxyAgent(process.env.PROXY_URL);
  
  return await fetch(url, {
    agent,
    headers: {
      'User-Agent': 'Mozilla/5.0 (compatible; ContentBot/1.0)'
    }
  });
}
```

## Configuration Options

### AI Provider Configuration

```typescript
// lib/config/ai-providers.ts
export const aiConfig = {
  claude: {
    model: 'claude-3-opus-20240229',
    maxTokens: 4000,
    temperature: 0.7,
    apiKey: process.env.ANTHROPIC_API_KEY
  },
  openai: {
    model: 'gpt-4-turbo-preview',
    maxTokens: 4000,
    temperature: 0.7,
    apiKey: process.env.OPENAI_API_KEY
  }
};
```

### Video Export Settings

```typescript
// lib/config/video-settings.ts
export const videoFormats = {
  reel: {
    width: 1080,
    height: 1920,
    fps: 30,
    codec: 'h264',
    audioBitrate: '128k'
  },
  tiktok: {
    width: 1080,
    height: 1920,
    fps: 30,
    codec: 'h264',
    audioBitrate: '128k'
  },
  shorts: {
    width: 1080,
    height: 1920,
    fps: 30,
    codec: 'h264',
    audioBitrate: '128k'
  },
  landscape: {
    width: 1920,
    height: 1080,
    fps: 30,
    codec: 'h264',
    audioBitrate: '192k'
  }
};
```

This skill covers the essential functionality for using the Ultimate AI Content Pipeline to automate content creation from research through video generation.
