---
name: marketing-pipeline-automation
description: Automate content creation from research to video generation using AI-powered pipeline with Claude, OpenAI, and Remotion
triggers:
  - how do I automate content creation with AI
  - set up automated marketing pipeline
  - generate videos from content automatically
  - crawl news and create content with AI
  - build AI content generation workflow
  - automate research and video creation
  - create marketing content pipeline
  - use Remotion for automated video generation
---

# Marketing Pipeline Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This project provides a complete AI-powered content automation pipeline that handles research, content generation, and video creation. It crawls real-time data from sources like TechCrunch, a16z, Twitter, and LinkedIn, generates content in multiple formats using Claude/OpenAI, and automatically renders videos using Remotion.

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
# AI APIs
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# RapidAPI for data crawling
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_url

# Remotion settings
REMOTION_LICENSE_KEY=your_remotion_license
```

## Project Structure

```
marketing-pipeline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/             
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── crawler/     # News crawling logic
│   │   └── remotion/    # Video generation
│   ├── types/           # TypeScript types
│   └── utils/           # Utility functions
├── public/              # Static assets
└── remotion/            # Remotion video templates
```

## Core Features & Usage

### 1. Content Research & Crawling

The system automatically crawls recent news from multiple sources:

```typescript
import { crawlNews } from '@/lib/crawler/newsScanner';

// Crawl news for a specific topic
const crawlNewsData = async (topic: string) => {
  const sources = ['techcrunch', 'a16z', 'twitter', 'linkedin'];
  
  const newsData = await crawlNews({
    topic,
    sources,
    timeRange: '24h',
    limit: 20
  });
  
  return newsData;
};

// Example usage
const aiNews = await crawlNewsData('artificial intelligence');
console.log(aiNews); // Array of articles with title, content, source, date
```

### 2. AI Content Generation

Generate content in multiple formats using Claude or OpenAI:

```typescript
import { generateContent } from '@/lib/ai/contentGenerator';

interface ContentConfig {
  topic: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  aiProvider: 'claude' | 'openai';
}

const createContent = async (config: ContentConfig) => {
  const content = await generateContent({
    topic: config.topic,
    format: config.format,
    language: config.language,
    tone: config.tone,
    provider: config.aiProvider,
    researchData: await crawlNewsData(config.topic)
  });
  
  return content;
};

// Example: Generate a toplist article
const article = await createContent({
  topic: 'AI Marketing Tools',
  format: 'toplist',
  language: 'en',
  tone: 'expert',
  aiProvider: 'claude'
});
```

### 3. Video Generation with Remotion

Automatically render videos from generated content:

```typescript
import { renderVideo } from '@/lib/remotion/videoRenderer';
import { Composition } from 'remotion';

// Define video composition
export const VideoComposition: React.FC<{
  title: string;
  content: string[];
  duration: number;
}> = ({ title, content, duration }) => {
  return (
    <Composition
      id="ContentVideo"
      component={ContentVideoTemplate}
      durationInFrames={duration * 30}
      fps={30}
      width={1080}
      height={1920}
      defaultProps={{
        title,
        content,
        style: 'modern'
      }}
    />
  );
};

// Render video from content
const createVideoFromContent = async (articleContent: any) => {
  const videoConfig = {
    title: articleContent.title,
    slides: articleContent.keyPoints.map((point: string, index: number) => ({
      text: point,
      duration: 3, // seconds
      animation: 'fadeIn'
    })),
    format: '9:16', // For Reels/TikTok/Shorts
    outputPath: './output/videos'
  };
  
  const videoPath = await renderVideo(videoConfig);
  return videoPath;
};
```

### 4. Complete Pipeline Workflow

Combine all features into a single automated pipeline:

```typescript
import { runContentPipeline } from '@/lib/pipeline';

interface PipelineConfig {
  keywords: string[];
  outputFormats: ('article' | 'video' | 'infographic')[];
  languages: ('en' | 'vi')[];
  autoPublish?: boolean;
}

const runFullPipeline = async (config: PipelineConfig) => {
  const results = await runContentPipeline({
    keywords: config.keywords,
    steps: [
      'research',      // Crawl news
      'generate',      // Create content
      'render',        // Make videos
      'publish'        // Auto-post (optional)
    ],
    options: {
      outputFormats: config.outputFormats,
      languages: config.languages,
      autoPublish: config.autoPublish || false
    }
  });
  
  return results;
};

// Example: Full automation
const pipelineResults = await runFullPipeline({
  keywords: ['AI marketing', 'content automation', 'video generation'],
  outputFormats: ['article', 'video'],
  languages: ['en', 'vi'],
  autoPublish: false
});
```

## API Routes (Next.js)

### Generate Content Endpoint

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { generateContent } from '@/lib/ai/contentGenerator';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const { topic, format, language, tone } = body;
    
    const content = await generateContent({
      topic,
      format,
      language,
      tone,
      provider: 'claude'
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

### Video Render Endpoint

```typescript
// app/api/render-video/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { renderVideo } from '@/lib/remotion/videoRenderer';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const { content, format } = body;
    
    const videoUrl = await renderVideo({
      title: content.title,
      slides: content.slides,
      format: format || '9:16'
    });
    
    return NextResponse.json({ success: true, videoUrl });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

## Common Patterns

### Multi-Language Content Generation

```typescript
const generateMultiLanguageContent = async (topic: string) => {
  const languages = ['en', 'vi'];
  
  const contentPromises = languages.map(lang =>
    generateContent({
      topic,
      format: 'toplist',
      language: lang,
      tone: 'expert',
      provider: 'claude'
    })
  );
  
  const [englishContent, vietnameseContent] = await Promise.all(contentPromises);
  
  return { en: englishContent, vi: vietnameseContent };
};
```

### Batch Video Rendering

```typescript
const renderMultipleVideos = async (articles: any[]) => {
  const renderPromises = articles.map(article =>
    renderVideo({
      title: article.title,
      slides: article.keyPoints.map(point => ({
        text: point,
        duration: 3
      })),
      format: '9:16'
    })
  );
  
  const videos = await Promise.all(renderPromises);
  return videos;
};
```

### Content Scheduling

```typescript
import { scheduleContent } from '@/lib/scheduler';

const scheduleContentPosts = async (content: any[]) => {
  const schedule = content.map((item, index) => ({
    content: item,
    publishAt: new Date(Date.now() + index * 24 * 60 * 60 * 1000), // Daily
    platforms: ['facebook', 'linkedin', 'twitter']
  }));
  
  await scheduleContent(schedule);
};
```

## Development Commands

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Run production server
npm run start

# Type checking
npm run type-check

# Lint code
npm run lint

# Render Remotion video locally
npm run remotion:render
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement retry logic for API calls
const callAIWithRetry = async (prompt: string, maxRetries = 3) => {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await generateContent({ topic: prompt });
    } catch (error) {
      if (error.status === 429 && i < maxRetries - 1) {
        await new Promise(resolve => setTimeout(resolve, 2000 * (i + 1)));
        continue;
      }
      throw error;
    }
  }
};
```

### Video Rendering Memory Issues

```typescript
// Use streaming for large videos
const renderLargeVideo = async (config: any) => {
  const chunks = splitContentIntoChunks(config.slides, 10);
  
  const videoChunks = await Promise.all(
    chunks.map(chunk => renderVideo({ ...config, slides: chunk }))
  );
  
  return mergeVideoChunks(videoChunks);
};
```

### News Crawling Failures

```typescript
// Fallback to multiple sources
const crawlWithFallback = async (topic: string) => {
  const sources = ['techcrunch', 'a16z', 'twitter'];
  
  for (const source of sources) {
    try {
      const data = await crawlNews({ topic, sources: [source] });
      if (data.length > 0) return data;
    } catch (error) {
      console.warn(`Failed to crawl ${source}, trying next...`);
    }
  }
  
  throw new Error('All news sources failed');
};
```

## Best Practices

1. **Cache Research Data**: Store crawled news to avoid redundant API calls
2. **Queue Video Rendering**: Use a job queue for video generation to prevent memory overload
3. **Validate AI Output**: Always validate and sanitize AI-generated content before publishing
4. **Monitor API Usage**: Track API consumption to stay within rate limits
5. **Use Webhooks**: Implement webhooks for long-running video renders
6. **Content Versioning**: Keep versions of generated content for A/B testing
