---
name: ultimate-ai-content-pipeline
description: Automated content pipeline from research to video generation using Claude, OpenAI, and Remotion for social media content creation
triggers:
  - how do I automate content creation with AI
  - set up automated marketing content pipeline
  - generate videos from text content automatically
  - crawl news and create social media posts
  - use Claude and OpenAI for content generation
  - create AI-powered content workflow
  - automate video generation with Remotion
  - build automated research to video pipeline
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

A complete TypeScript-based content automation system that transforms keywords into polished content and videos. The pipeline handles web scraping/research, AI-powered content generation in multiple formats and languages, and automatic video rendering for social media platforms.

## What It Does

The Ultimate AI Content Pipeline automates the entire content creation workflow:

1. **Auto-Research**: Crawls news sources (TechCrunch, a16z, Twitter, LinkedIn) for recent data
2. **AI Content Generation**: Uses Claude 3 and OpenAI to generate content in multiple formats (toplist, POV, case study, how-to)
3. **Multi-language Support**: Outputs in English and Vietnamese simultaneously
4. **Video Rendering**: Converts written content into videos/infographics using Remotion
5. **Platform Optimization**: Exports video in formats optimized for Reels, TikTok, Shorts

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

## Configuration

Create a `.env.local` file in the root directory:

```env
# AI Provider Keys
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# RapidAPI for web scraping
RAPIDAPI_KEY=your_rapidapi_key

# Optional: Database configuration
DATABASE_URL=your_database_url

# Next.js Configuration
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

### Required API Keys

- **Anthropic (Claude)**: Get from https://console.anthropic.com/
- **OpenAI**: Get from https://platform.openai.com/
- **RapidAPI**: Get from https://rapidapi.com/ (for news/social scraping APIs)

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI provider integrations
│   │   ├── scraper/     # Web scraping modules
│   │   └── video/       # Remotion video generation
│   └── types/           # TypeScript types
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Core Usage Patterns

### 1. Running the Content Pipeline

```typescript
import { ContentPipeline } from '@/lib/pipeline';

const pipeline = new ContentPipeline({
  aiProvider: 'claude', // or 'openai'
  language: ['en', 'vi'],
  contentFormat: 'toplist',
  generateVideo: true
});

// Execute full pipeline
const result = await pipeline.execute({
  keyword: 'AI Marketing Trends 2024',
  targetAudience: 'marketers',
  tone: 'professional'
});

console.log(result.content); // Generated content
console.log(result.videoUrl); // Rendered video URL
```

### 2. Research & Scraping Module

```typescript
import { ResearchEngine } from '@/lib/scraper/research-engine';

const researcher = new ResearchEngine({
  sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
  timeRange: '24h'
});

// Gather insights on a topic
const insights = await researcher.gather('artificial intelligence marketing');

console.log(insights.articles);    // Array of scraped articles
console.log(insights.trends);      // Identified trends
console.log(insights.statistics);  // Extracted data points
```

### 3. AI Content Generation with Claude

```typescript
import { ClaudeContentGenerator } from '@/lib/ai/claude-generator';

const generator = new ClaudeContentGenerator({
  apiKey: process.env.ANTHROPIC_API_KEY,
  model: 'claude-3-opus-20240229'
});

// Generate content in specific format
const content = await generator.generate({
  topic: 'Top 10 AI Tools for Content Creators',
  format: 'toplist',
  language: 'en',
  tone: 'friendly',
  researchData: insights // From research module
});

console.log(content.title);
console.log(content.body);
console.log(content.sections);
```

### 4. Multi-language Content Generation

```typescript
import { MultiLanguageGenerator } from '@/lib/ai/multi-language';

const mlGenerator = new MultiLanguageGenerator({
  providers: {
    en: 'openai',
    vi: 'claude'
  }
});

// Generate parallel content
const multiContent = await mlGenerator.generateParallel({
  topic: 'Social Media Trends',
  languages: ['en', 'vi'],
  format: 'how-to'
});

console.log(multiContent.en); // English version
console.log(multiContent.vi); // Vietnamese version
```

### 5. Video Generation with Remotion

```typescript
import { VideoRenderer } from '@/lib/video/renderer';
import { bundle } from '@remotion/bundler';

const renderer = new VideoRenderer({
  compositionId: 'ContentVideo',
  outputFormat: 'mp4'
});

// Render video from content
const video = await renderer.render({
  content: content,
  template: 'infographic',
  platform: 'tiktok', // auto-sets aspect ratio
  duration: 30, // seconds
  props: {
    title: content.title,
    points: content.sections,
    branding: {
      logo: '/logo.png',
      colors: ['#FF6B6B', '#4ECDC4']
    }
  }
});

console.log(video.outputPath); // Path to rendered video
```

### 6. Complete Workflow Example

```typescript
import { AutomatedContentWorkflow } from '@/lib/workflow';

async function createContentFromKeyword(keyword: string) {
  const workflow = new AutomatedContentWorkflow();
  
  try {
    // Step 1: Research
    const research = await workflow.research(keyword);
    
    // Step 2: Generate content in multiple languages
    const content = await workflow.generateContent({
      research,
      formats: ['toplist', 'pov'],
      languages: ['en', 'vi']
    });
    
    // Step 3: Create videos
    const videos = await workflow.renderVideos({
      content: content.en.toplist,
      platforms: ['tiktok', 'reels', 'shorts']
    });
    
    // Step 4: Save and schedule
    await workflow.saveAndSchedule({
      content,
      videos,
      scheduledFor: new Date('2024-06-25T10:00:00Z')
    });
    
    return {
      content,
      videos,
      status: 'success'
    };
  } catch (error) {
    console.error('Workflow failed:', error);
    throw error;
  }
}

// Execute
const result = await createContentFromKeyword('AI-powered marketing automation');
```

## Next.js API Routes

### Generate Content API

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { ContentPipeline } from '@/lib/pipeline';

export async function POST(request: NextRequest) {
  const { keyword, format, language } = await request.json();
  
  const pipeline = new ContentPipeline({
    aiProvider: 'claude',
    language: language || ['en'],
    contentFormat: format || 'toplist',
    generateVideo: false
  });
  
  const result = await pipeline.execute({ keyword });
  
  return NextResponse.json(result);
}
```

Usage from frontend:

```typescript
const response = await fetch('/api/generate', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    keyword: 'Content Marketing Tips',
    format: 'how-to',
    language: ['en', 'vi']
  })
});

const data = await response.json();
```

## CLI Commands

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Start production server
npm run start

# Render Remotion video
npm run video:render -- --composition=ContentVideo --output=output.mp4

# Run content generation script
npm run generate -- --keyword="AI Marketing" --format=toplist

# Run research only
npm run research -- --topic="Social Media Trends" --sources=all
```

## Remotion Video Configuration

```typescript
// remotion/Root.tsx
import { Composition } from 'remotion';
import { ContentVideo } from './compositions/ContentVideo';

export const RemotionRoot: React.FC = () => {
  return (
    <>
      <Composition
        id="ContentVideo"
        component={ContentVideo}
        durationInFrames={900} // 30 seconds at 30fps
        fps={30}
        width={1080}
        height={1920} // TikTok/Reels vertical format
        defaultProps={{
          title: 'Your Title',
          sections: [],
          branding: {}
        }}
      />
    </>
  );
};
```

## Common Patterns

### Batch Content Generation

```typescript
const keywords = [
  'AI Marketing Tools',
  'Content Automation',
  'Social Media Strategy'
];

const results = await Promise.all(
  keywords.map(keyword => 
    pipeline.execute({ keyword })
  )
);
```

### Custom Content Templates

```typescript
import { ContentTemplate } from '@/lib/templates';

const customTemplate = new ContentTemplate({
  name: 'product-launch',
  structure: {
    hook: { maxWords: 50 },
    problem: { maxWords: 150 },
    solution: { maxWords: 200 },
    cta: { maxWords: 50 }
  }
});

const content = await generator.generate({
  topic: 'New AI Tool Launch',
  template: customTemplate
});
```

### Scheduling Content Posts

```typescript
import { ContentScheduler } from '@/lib/scheduler';

const scheduler = new ContentScheduler();

await scheduler.schedule({
  content: generatedContent,
  platforms: ['facebook', 'linkedin', 'twitter'],
  publishAt: new Date('2024-06-25T15:00:00Z'),
  autoPublish: true
});
```

## Troubleshooting

### API Rate Limits

```typescript
import { RateLimiter } from '@/lib/utils/rate-limiter';

const limiter = new RateLimiter({
  maxRequests: 50,
  perMinutes: 1
});

const content = await limiter.execute(() => 
  generator.generate({ topic })
);
```

### Video Rendering Fails

- Check Remotion dependencies: `npm install @remotion/cli @remotion/renderer`
- Ensure ffmpeg is installed: `brew install ffmpeg` (macOS)
- Verify composition ID matches registered compositions

### Scraping Issues

```typescript
// Add retry logic
import { retry } from '@/lib/utils/retry';

const insights = await retry(
  () => researcher.gather(topic),
  { attempts: 3, delay: 2000 }
);
```

### AI Provider Errors

```typescript
// Fallback to alternative provider
const generator = new AIGenerator({
  primary: 'claude',
  fallback: 'openai',
  onFallback: (error) => console.log('Using fallback:', error)
});
```

### Memory Issues with Large Content

```typescript
// Stream content generation
const stream = await generator.generateStream({
  topic,
  onChunk: (chunk) => process.stdout.write(chunk)
});
```

## Performance Optimization

```typescript
// Cache research results
import { cacheManager } from '@/lib/cache';

const cachedResearch = await cacheManager.getOrSet(
  `research:${keyword}`,
  () => researcher.gather(keyword),
  { ttl: 3600 } // 1 hour
);

// Parallel processing
const [research, templates] = await Promise.all([
  researcher.gather(keyword),
  loadTemplates()
]);
```

## Testing

```typescript
import { ContentPipeline } from '@/lib/pipeline';

describe('Content Pipeline', () => {
  it('generates content from keyword', async () => {
    const pipeline = new ContentPipeline({
      aiProvider: 'claude',
      language: ['en']
    });
    
    const result = await pipeline.execute({
      keyword: 'Test Topic'
    });
    
    expect(result.content).toBeDefined();
    expect(result.content.title).toContain('Test Topic');
  });
});
```
