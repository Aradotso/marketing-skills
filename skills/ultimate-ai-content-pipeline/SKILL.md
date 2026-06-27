---
name: ultimate-ai-content-pipeline
description: Automated content creation pipeline from research to video generation using Claude/OpenAI and Remotion
triggers:
  - how do I automate content creation with AI research
  - generate video content from text automatically
  - set up AI content pipeline with Claude and OpenAI
  - create automated marketing content workflow
  - build content automation with remotion video
  - scrape research data and generate articles
  - automate social media content from research to video
  - use AI to create blog posts and videos together
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This TypeScript-based content automation system creates a complete pipeline: from automatic research/web scraping, to AI-powered article generation in multiple formats and languages, to automatic video rendering with Remotion. Perfect for content creators, marketers, and businesses looking to scale their content production.

## What It Does

- **Auto-Research**: Crawls and analyzes real-time data from sources like TechCrunch, a16z, Twitter/X, LinkedIn
- **AI Content Generation**: Creates articles in multiple formats (Toplist, POV, Case Study, How-to) using Claude 3 or OpenAI
- **Multi-language Support**: Generates content simultaneously in English and Vietnamese
- **Video Rendering**: Automatically converts text content into videos/infographics using Remotion
- **Multi-platform Export**: Outputs optimized for Reels, TikTok, Shorts

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

Create a `.env.local` file in the project root:

```bash
# AI API Keys
OPENAI_API_KEY=your_openai_key_here
ANTHROPIC_API_KEY=your_claude_key_here

# Research/Scraping APIs
RAPIDAPI_KEY=your_rapidapi_key_here

# Database (if applicable)
DATABASE_URL=your_database_url_here

# Next.js Configuration
NEXT_PUBLIC_API_URL=http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── scraper/     # Research/crawling logic
│   │   └── video/       # Remotion video rendering
│   ├── types/           # TypeScript type definitions
│   └── utils/           # Helper functions
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Core API Usage

### 1. Research & Data Scraping

```typescript
import { researchTopic } from '@/lib/scraper/research';

async function gatherResearch(keyword: string) {
  const researchData = await researchTopic({
    keyword,
    sources: ['techcrunch', 'twitter', 'linkedin'],
    timeRange: '24h',
    maxResults: 10
  });
  
  return {
    articles: researchData.articles,
    insights: researchData.insights,
    trends: researchData.trends,
    statistics: researchData.statistics
  };
}
```

### 2. AI Content Generation

```typescript
import { generateContent } from '@/lib/ai/content-generator';

async function createArticle(topic: string, researchData: any) {
  // Using Claude
  const claudeContent = await generateContent({
    provider: 'claude',
    model: 'claude-3-opus-20240229',
    prompt: {
      topic,
      format: 'case-study',
      tone: 'professional',
      language: 'en',
      researchData
    },
    apiKey: process.env.ANTHROPIC_API_KEY
  });
  
  // Using OpenAI
  const openAIContent = await generateContent({
    provider: 'openai',
    model: 'gpt-4-turbo-preview',
    prompt: {
      topic,
      format: 'how-to',
      tone: 'friendly',
      language: 'vi',
      researchData
    },
    apiKey: process.env.OPENAI_API_KEY
  });
  
  return { claudeContent, openAIContent };
}
```

### 3. Multi-format Content Generation

```typescript
import { ContentFormat, generateMultiFormat } from '@/lib/ai/formats';

async function generateAllFormats(topic: string, research: any) {
  const formats: ContentFormat[] = [
    'toplist',
    'pov',
    'case-study',
    'how-to'
  ];
  
  const contents = await Promise.all(
    formats.map(format => 
      generateMultiFormat({
        topic,
        format,
        research,
        languages: ['en', 'vi'],
        tone: 'expert'
      })
    )
  );
  
  return contents;
}
```

### 4. Video Rendering with Remotion

```typescript
import { renderVideo } from '@/lib/video/renderer';
import { bundle } from '@remotion/bundler';
import { renderMedia } from '@remotion/renderer';

async function createVideoFromContent(content: any) {
  const videoConfig = {
    composition: 'ContentVideo',
    props: {
      title: content.title,
      subtitle: content.subtitle,
      keyPoints: content.keyPoints,
      statistics: content.statistics,
      brandColors: {
        primary: '#FF6B6B',
        secondary: '#4ECDC4'
      }
    },
    duration: 60, // seconds
    fps: 30,
    resolution: {
      width: 1080,
      height: 1920 // Vertical for Reels/TikTok
    }
  };
  
  const bundled = await bundle({
    entryPoint: './remotion/index.ts',
    webpackOverride: (config) => config
  });
  
  const { outputLocation } = await renderMedia({
    composition: videoConfig.composition,
    serveUrl: bundled,
    codec: 'h264',
    outputLocation: `out/${content.slug}.mp4`,
    inputProps: videoConfig.props
  });
  
  return outputLocation;
}
```

## Complete Pipeline Example

```typescript
import { ContentPipeline } from '@/lib/pipeline';

async function runFullPipeline(keyword: string) {
  const pipeline = new ContentPipeline({
    aiProvider: 'claude',
    videoEnabled: true,
    autoPublish: false
  });
  
  try {
    // Step 1: Research
    const research = await pipeline.research({
      keyword,
      sources: ['techcrunch', 'twitter', 'linkedin'],
      depth: 'deep'
    });
    
    // Step 2: Generate Content
    const content = await pipeline.generateContent({
      research,
      formats: ['toplist', 'how-to'],
      languages: ['en', 'vi'],
      tone: 'professional'
    });
    
    // Step 3: Create Videos
    const videos = await pipeline.renderVideos({
      content,
      platforms: ['reels', 'tiktok', 'shorts'],
      style: 'modern'
    });
    
    // Step 4: Save Results
    const result = await pipeline.save({
      content,
      videos,
      metadata: {
        keyword,
        createdAt: new Date(),
        status: 'ready-to-publish'
      }
    });
    
    return result;
  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}

// Usage
runFullPipeline('AI Marketing Automation 2024')
  .then(result => console.log('Pipeline completed:', result))
  .catch(error => console.error('Pipeline failed:', error));
```

## API Routes (Next.js)

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { ContentPipeline } from '@/lib/pipeline';

export async function POST(request: NextRequest) {
  try {
    const { keyword, formats, languages } = await request.json();
    
    const pipeline = new ContentPipeline({
      aiProvider: 'claude',
      apiKey: process.env.ANTHROPIC_API_KEY
    });
    
    const result = await pipeline.run({
      keyword,
      formats: formats || ['toplist', 'how-to'],
      languages: languages || ['en', 'vi']
    });
    
    return NextResponse.json({
      success: true,
      data: result
    });
  } catch (error) {
    return NextResponse.json({
      success: false,
      error: error.message
    }, { status: 500 });
  }
}
```

## Custom Content Formats

```typescript
// lib/ai/formats/custom-format.ts
export interface CustomFormat {
  name: string;
  structure: string[];
  promptTemplate: string;
}

export const createCustomFormat = (format: CustomFormat) => {
  return async (topic: string, research: any, ai: AIProvider) => {
    const prompt = `
      Create a ${format.name} article about: ${topic}
      
      Structure:
      ${format.structure.map((s, i) => `${i + 1}. ${s}`).join('\n')}
      
      Research Data:
      ${JSON.stringify(research, null, 2)}
      
      ${format.promptTemplate}
    `;
    
    return await ai.generate(prompt);
  };
};

// Usage
const podcastFormat = createCustomFormat({
  name: 'Podcast Script',
  structure: [
    'Hook & Introduction',
    'Main Topic Discussion',
    'Expert Insights',
    'Q&A Section',
    'Call to Action'
  ],
  promptTemplate: 'Write in conversational tone suitable for audio content.'
});
```

## Video Template Customization

```typescript
// remotion/compositions/CustomVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig } from 'remotion';

export const CustomVideo: React.FC<{
  title: string;
  keyPoints: string[];
  brandColors: { primary: string; secondary: string };
}> = ({ title, keyPoints, brandColors }) => {
  const frame = useCurrentFrame();
  const { fps, durationInFrames } = useVideoConfig();
  
  const opacity = Math.min(1, frame / (fps * 0.5));
  
  return (
    <AbsoluteFill style={{ backgroundColor: brandColors.primary }}>
      <div style={{ opacity, padding: 40 }}>
        <h1 style={{ 
          fontSize: 64, 
          color: 'white',
          fontWeight: 'bold'
        }}>
          {title}
        </h1>
        
        <ul style={{ marginTop: 40 }}>
          {keyPoints.map((point, i) => (
            <li 
              key={i}
              style={{
                fontSize: 32,
                color: 'white',
                marginBottom: 20,
                opacity: frame > (fps * (i + 1)) ? 1 : 0,
                transition: 'opacity 0.3s'
              }}
            >
              {point}
            </li>
          ))}
        </ul>
      </div>
    </AbsoluteFill>
  );
};
```

## Common Patterns

### Batch Content Generation

```typescript
async function batchGenerate(keywords: string[]) {
  const pipeline = new ContentPipeline();
  
  const results = await Promise.allSettled(
    keywords.map(keyword => 
      pipeline.run({
        keyword,
        formats: ['toplist'],
        languages: ['en']
      })
    )
  );
  
  const successful = results
    .filter(r => r.status === 'fulfilled')
    .map(r => (r as PromiseFulfilledResult<any>).value);
    
  const failed = results
    .filter(r => r.status === 'rejected')
    .map(r => (r as PromiseRejectedResult).reason);
  
  return { successful, failed };
}
```

### Scheduled Content Generation

```typescript
import { CronJob } from 'cron';

const dailyContentJob = new CronJob('0 9 * * *', async () => {
  const trendingTopics = await fetchTrendingTopics();
  
  for (const topic of trendingTopics.slice(0, 3)) {
    await runFullPipeline(topic);
  }
}, null, true, 'America/New_York');

dailyContentJob.start();
```

## Troubleshooting

### API Rate Limits

```typescript
import pLimit from 'p-limit';

const limit = pLimit(5); // Max 5 concurrent requests

async function generateWithRateLimit(topics: string[]) {
  const promises = topics.map(topic => 
    limit(() => runFullPipeline(topic))
  );
  
  return await Promise.all(promises);
}
```

### Video Rendering Timeouts

```typescript
const videoConfig = {
  timeoutInMilliseconds: 300000, // 5 minutes
  chromiumOptions: {
    headless: true,
    args: ['--no-sandbox', '--disable-setuid-sandbox']
  }
};
```

### Memory Issues with Large Research Data

```typescript
async function optimizeResearchData(data: any) {
  // Limit data size
  return {
    articles: data.articles.slice(0, 10),
    insights: data.insights.slice(0, 5),
    // Remove large unused fields
    trends: data.trends.map(({ title, value }) => ({ title, value }))
  };
}
```

## Development

```bash
# Run development server
npm run dev

# Build for production
npm run build

# Start production server
npm run start

# Render specific video composition
npm run remotion render -- ContentVideo out/video.mp4
```
