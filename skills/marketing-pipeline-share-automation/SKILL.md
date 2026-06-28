---
name: marketing-pipeline-share-automation
description: AI-powered content automation pipeline that researches, generates scripts, and creates videos from keywords using Claude/OpenAI and Remotion
triggers:
  - how do I automate content creation with AI
  - generate marketing content from research to video
  - set up automated content pipeline
  - create videos from text using Remotion
  - research and generate blog posts automatically
  - build AI content automation system
  - automate social media content generation
  - create content pipeline with Claude and OpenAI
---

# Marketing Pipeline Share Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This project is an end-to-end AI content automation pipeline that transforms keywords into complete content packages including research, written content in multiple formats, and rendered videos. It combines web scraping, AI writing (Claude 3/OpenAI), and video generation (Remotion) into a single automated workflow.

## What It Does

1. **Auto-Research**: Crawls news sources (TechCrunch, a16z, Twitter, LinkedIn) for recent data on your topic
2. **AI Content Generation**: Creates content in multiple formats (listicles, POV, case studies, how-tos) using Claude/OpenAI
3. **Multi-language Support**: Generates both English and Vietnamese versions
4. **Video Rendering**: Automatically converts content to videos/infographics using Remotion
5. **Platform Optimization**: Exports videos optimized for Reels, TikTok, YouTube Shorts

## Installation

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
yarn install
```

### Required Environment Variables

Create a `.env.local` file in the root directory:

```env
# AI Provider APIs
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Optional: Custom configurations
CONTENT_LANGUAGE=en,vi
DEFAULT_TONE=professional
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/              # Core libraries
│   │   ├── research/     # Web scraping & research
│   │   ├── ai/           # AI content generation
│   │   └── video/        # Video rendering
│   ├── remotion/         # Remotion video templates
│   └── types/            # TypeScript types
├── public/               # Static assets
└── .env.local           # Environment variables
```

## Core API Usage

### 1. Research Module

Automatically fetch and analyze recent content:

```typescript
import { researchTopic } from '@/lib/research/scraper';

async function gatherResearch(keyword: string) {
  const research = await researchTopic({
    keyword,
    sources: ['techcrunch', 'twitter', 'linkedin'],
    timeRange: '24h',
    maxResults: 20
  });
  
  return {
    articles: research.articles,
    insights: research.extractedInsights,
    trends: research.trendingTopics
  };
}
```

### 2. AI Content Generation

Generate content using Claude or OpenAI:

```typescript
import { generateContent } from '@/lib/ai/generator';

async function createArticle(topic: string, research: any) {
  const content = await generateContent({
    provider: 'claude', // or 'openai'
    model: 'claude-3-opus-20240229',
    prompt: {
      topic,
      format: 'listicle', // 'pov', 'case-study', 'how-to'
      tone: 'professional',
      language: ['en', 'vi'],
      research: research.insights
    }
  });
  
  return {
    title: content.title,
    body: content.body,
    translations: content.translations,
    metadata: content.metadata
  };
}
```

### 3. Video Generation with Remotion

Convert content to video:

```typescript
import { renderVideo } from '@/lib/video/renderer';
import { VideoComposition } from '@/remotion/compositions/ContentVideo';

async function generateContentVideo(article: any) {
  const video = await renderVideo({
    composition: VideoComposition,
    props: {
      title: article.title,
      sections: article.sections,
      style: 'modern',
      platform: 'instagram-reel' // 'tiktok', 'youtube-shorts'
    },
    outputFormat: 'mp4',
    resolution: {
      width: 1080,
      height: 1920 // Vertical for mobile
    }
  });
  
  return video.outputPath;
}
```

## Complete Pipeline Example

End-to-end automation workflow:

```typescript
import { ContentPipeline } from '@/lib/pipeline';

async function runFullPipeline() {
  const pipeline = new ContentPipeline({
    aiProvider: 'claude',
    languages: ['en', 'vi'],
    videoEnabled: true
  });
  
  // Step 1: Research
  const research = await pipeline.research({
    keyword: 'AI marketing automation',
    depth: 'comprehensive'
  });
  
  // Step 2: Generate multiple content formats
  const contentVariations = await pipeline.generateContent({
    research,
    formats: ['listicle', 'how-to', 'case-study'],
    tone: 'professional',
    targetAudience: 'marketers'
  });
  
  // Step 3: Create videos for each format
  const videos = await pipeline.renderVideos({
    content: contentVariations,
    platforms: ['instagram', 'tiktok', 'youtube']
  });
  
  return {
    articles: contentVariations,
    videos: videos,
    publishReady: true
  };
}
```

## Common Patterns

### Custom Research Sources

Add your own data sources:

```typescript
import { CustomScraper } from '@/lib/research/custom-scraper';

const scraper = new CustomScraper({
  sources: [
    {
      name: 'custom-blog',
      url: 'https://example.com/blog',
      selectors: {
        title: '.post-title',
        content: '.post-content',
        date: '.post-date'
      }
    }
  ]
});

const data = await scraper.scrape('AI trends');
```

### Custom Content Templates

Define your own content structure:

```typescript
interface CustomContentTemplate {
  format: 'custom-format';
  sections: {
    hook: string;
    mainPoints: string[];
    callToAction: string;
  };
}

const customContent = await generateContent({
  provider: 'claude',
  template: 'custom-format',
  structure: {
    hook: { maxWords: 50, tone: 'engaging' },
    mainPoints: { count: 5, withData: true },
    callToAction: { style: 'subtle' }
  }
});
```

### Batch Processing

Process multiple keywords in parallel:

```typescript
async function batchContentGeneration(keywords: string[]) {
  const pipeline = new ContentPipeline();
  
  const results = await Promise.all(
    keywords.map(async (keyword) => {
      try {
        const research = await pipeline.research({ keyword });
        const content = await pipeline.generateContent({ 
          research, 
          formats: ['listicle'] 
        });
        return { keyword, success: true, content };
      } catch (error) {
        return { keyword, success: false, error };
      }
    })
  );
  
  return results;
}
```

## Remotion Video Customization

Create custom video templates:

```typescript
// src/remotion/compositions/CustomVideo.tsx
import { AbsoluteFill, useCurrentFrame, interpolate } from 'remotion';

export const CustomVideoTemplate: React.FC<{
  title: string;
  points: string[];
}> = ({ title, points }) => {
  const frame = useCurrentFrame();
  
  const opacity = interpolate(
    frame,
    [0, 30],
    [0, 1],
    { extrapolateRight: 'clamp' }
  );
  
  return (
    <AbsoluteFill style={{ backgroundColor: '#000' }}>
      <div style={{ opacity, padding: 60 }}>
        <h1 style={{ color: '#fff', fontSize: 72 }}>{title}</h1>
        {points.map((point, i) => (
          <p key={i} style={{ color: '#fff', fontSize: 36 }}>
            {point}
          </p>
        ))}
      </div>
    </AbsoluteFill>
  );
};
```

Register the composition:

```typescript
// src/remotion/index.ts
import { registerRoot } from 'remotion';
import { CustomVideoTemplate } from './compositions/CustomVideo';

registerRoot(() => {
  return (
    <>
      <Composition
        id="custom-video"
        component={CustomVideoTemplate}
        durationInFrames={300}
        fps={30}
        width={1080}
        height={1920}
      />
    </>
  );
});
```

## Configuration

### AI Provider Settings

```typescript
// lib/config/ai.ts
export const aiConfig = {
  claude: {
    model: 'claude-3-opus-20240229',
    maxTokens: 4096,
    temperature: 0.7
  },
  openai: {
    model: 'gpt-4-turbo-preview',
    maxTokens: 4096,
    temperature: 0.7
  }
};
```

### Content Format Presets

```typescript
// lib/config/formats.ts
export const contentFormats = {
  listicle: {
    structure: 'numbered-list',
    minItems: 5,
    maxItems: 10,
    includeIntro: true,
    includeConclusion: true
  },
  pov: {
    structure: 'opinion-based',
    perspective: 'first-person',
    includeCounterarguments: true
  },
  caseStudy: {
    structure: 'problem-solution',
    sections: ['background', 'challenge', 'solution', 'results'],
    includeMetrics: true
  }
};
```

## Troubleshooting

### API Rate Limits

```typescript
import { RateLimiter } from '@/lib/utils/rate-limiter';

const limiter = new RateLimiter({
  requestsPerMinute: 10,
  provider: 'anthropic'
});

await limiter.execute(async () => {
  return await generateContent({ /* ... */ });
});
```

### Video Rendering Timeout

Increase timeout for long videos:

```typescript
const video = await renderVideo({
  composition: VideoComposition,
  props: { /* ... */ },
  timeout: 300000, // 5 minutes
  concurrency: 2 // Reduce if memory issues
});
```

### Research Data Quality

Filter low-quality sources:

```typescript
const research = await researchTopic({
  keyword: 'AI marketing',
  filters: {
    minWordCount: 500,
    excludeDomains: ['low-quality-site.com'],
    requireImages: true,
    publishedAfter: new Date(Date.now() - 7 * 24 * 60 * 60 * 1000)
  }
});
```

### Memory Issues with Batch Processing

Use chunking for large batches:

```typescript
import { chunk } from 'lodash';

async function processBatchInChunks(keywords: string[]) {
  const chunks = chunk(keywords, 5); // Process 5 at a time
  const results = [];
  
  for (const chunk of chunks) {
    const chunkResults = await Promise.all(
      chunk.map(k => pipeline.research({ keyword: k }))
    );
    results.push(...chunkResults);
    await new Promise(resolve => setTimeout(resolve, 2000)); // Pause between chunks
  }
  
  return results;
}
```

## Next.js Development

```bash
# Run development server
npm run dev

# Build for production
npm run build

# Start production server
npm start
```

Access the UI at `http://localhost:3000` to interact with the pipeline through a web interface.
