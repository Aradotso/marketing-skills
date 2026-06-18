---
name: marketing-pipeline-share-automation
description: Automated AI content pipeline for research, scriptwriting, and video generation using Claude/OpenAI and Remotion
triggers:
  - how do I automate content creation with AI research
  - set up an automated marketing content pipeline
  - generate videos from text content automatically
  - crawl news and create social media posts
  - use Claude and OpenAI for content automation
  - build a content pipeline with Remotion video rendering
  - automate content from research to video generation
  - create multilingual content with AI automation
---

# Marketing Pipeline Share Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to help developers use **marketing-pipeline-share**, an end-to-end automated content creation system that handles research (web crawling), scriptwriting (AI-powered), and video generation (Remotion) in a single pipeline.

## What This Project Does

Marketing Pipeline Share is a TypeScript-based automation system that:
- **Auto-crawls** news sources (TechCrunch, a16z, Twitter, LinkedIn) for fresh content
- **Generates content** in multiple formats (toplist, POV, case study, how-to) using Claude 3 or OpenAI
- **Renders videos** automatically using Remotion for Reels, TikTok, Shorts
- **Supports multilingual output** (English & Vietnamese) with customizable tone
- **Provides a Next.js interface** for managing the entire content pipeline

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

### Environment Setup

Create a `.env.local` file in the root directory:

```bash
# AI Provider APIs
OPENAI_API_KEY=your_openai_key_here
ANTHROPIC_API_KEY=your_claude_key_here

# Research/Crawling APIs
RAPIDAPI_KEY=your_rapidapi_key_here

# Optional: Database/Storage
DATABASE_URL=your_database_url_here

# Remotion Configuration (if applicable)
REMOTION_AWS_ACCESS_KEY_ID=your_aws_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret
```

### Running the Development Server

```bash
npm run dev
# or
yarn dev
# or
pnpm dev
```

The application will be available at `http://localhost:3000`.

## Project Structure

```
marketing-pineline-share/
├── app/                    # Next.js app directory
├── components/             # React components
├── lib/                    # Core libraries
│   ├── ai/                # AI integration (Claude, OpenAI)
│   ├── crawler/           # Web scraping modules
│   ├── video/             # Remotion video generation
│   └── utils/             # Utility functions
├── public/                # Static assets
└── remotion/              # Remotion video templates
```

## Core APIs and Usage

### 1. Research/Crawling Module

```typescript
import { crawlNews } from '@/lib/crawler/news-crawler';

// Crawl news from multiple sources
const researchData = await crawlNews({
  keyword: 'AI automation',
  sources: ['techcrunch', 'a16z', 'twitter'],
  timeframe: '24h',
  maxResults: 20
});

console.log(researchData);
// {
//   articles: [...],
//   insights: [...],
//   trending_topics: [...]
// }
```

### 2. AI Content Generation

```typescript
import { generateContent } from '@/lib/ai/content-generator';

// Generate content using Claude or OpenAI
const content = await generateContent({
  provider: 'claude', // or 'openai'
  model: 'claude-3-opus-20240229',
  topic: 'AI automation trends',
  format: 'toplist', // 'pov', 'case-study', 'how-to'
  language: 'vi', // or 'en'
  tone: 'expert', // 'friendly', 'humorous'
  researchData: researchData,
  wordCount: 1500
});

console.log(content);
// {
//   title: "Top 10 xu hướng AI automation 2024",
//   body: "...",
//   metadata: {...}
// }
```

### 3. Video Generation with Remotion

```typescript
import { renderVideo } from '@/lib/video/remotion-renderer';

// Render video from content
const videoOutput = await renderVideo({
  content: content.body,
  template: 'reels', // 'tiktok', 'shorts', 'infographic'
  aspectRatio: '9:16',
  duration: 60, // seconds
  voiceover: true,
  subtitles: true
});

console.log(videoOutput.videoUrl);
// https://your-storage/videos/output-123.mp4
```

## Common Patterns

### Full Pipeline Example

```typescript
import { ContentPipeline } from '@/lib/pipeline';

async function runFullPipeline(keyword: string) {
  const pipeline = new ContentPipeline({
    aiProvider: 'claude',
    apiKey: process.env.ANTHROPIC_API_KEY
  });

  // Step 1: Research
  const research = await pipeline.research({
    keyword,
    sources: ['techcrunch', 'twitter'],
    depth: 'deep'
  });

  // Step 2: Generate Content
  const content = await pipeline.generateContent({
    research,
    format: 'toplist',
    languages: ['en', 'vi']
  });

  // Step 3: Create Video
  const video = await pipeline.renderVideo({
    content: content.en,
    template: 'reels'
  });

  // Step 4: Schedule/Publish
  await pipeline.publish({
    content,
    video,
    platforms: ['facebook', 'instagram', 'tiktok']
  });

  return { content, video };
}

// Usage
runFullPipeline('marketing automation tools')
  .then(result => console.log('Pipeline completed:', result))
  .catch(err => console.error('Pipeline error:', err));
```

### Custom Content Format

```typescript
import { ClaudeClient } from '@/lib/ai/claude-client';

const claude = new ClaudeClient({
  apiKey: process.env.ANTHROPIC_API_KEY
});

const customPrompt = `
Based on this research data: ${JSON.stringify(researchData)}

Create a compelling case study about ${topic} that:
- Starts with a hook
- Includes data-backed insights
- Has 3 main sections
- Ends with actionable takeaways
- Tone: Professional yet engaging
- Language: Vietnamese
`;

const response = await claude.generate({
  model: 'claude-3-opus-20240229',
  prompt: customPrompt,
  maxTokens: 4000
});
```

### Batch Processing

```typescript
import { batchProcess } from '@/lib/utils/batch-processor';

const keywords = [
  'AI marketing tools',
  'content automation',
  'social media trends'
];

const results = await batchProcess(keywords, async (keyword) => {
  const pipeline = new ContentPipeline();
  return await pipeline.run(keyword);
}, {
  concurrency: 3,
  retries: 2
});
```

## Configuration

### Content Format Templates

Located in `lib/config/content-templates.ts`:

```typescript
export const CONTENT_FORMATS = {
  toplist: {
    structure: ['intro', 'items', 'conclusion'],
    minItems: 5,
    maxItems: 15
  },
  pov: {
    structure: ['hook', 'perspective', 'arguments', 'conclusion'],
    tone: ['expert', 'thought-leader']
  },
  casestudy: {
    structure: ['background', 'challenge', 'solution', 'results'],
    dataRequired: true
  },
  howto: {
    structure: ['intro', 'prerequisites', 'steps', 'tips', 'conclusion'],
    stepFormat: 'numbered'
  }
};
```

### Video Templates

Remotion templates in `remotion/templates/`:

```typescript
// remotion/templates/ReelsTemplate.tsx
import { AbsoluteFill, Sequence, useCurrentFrame } from 'remotion';

export const ReelsTemplate: React.FC<{
  title: string;
  points: string[];
  duration: number;
}> = ({ title, points, duration }) => {
  const frame = useCurrentFrame();
  
  return (
    <AbsoluteFill style={{ backgroundColor: '#000' }}>
      <Sequence from={0} durationInFrames={90}>
        <h1>{title}</h1>
      </Sequence>
      {points.map((point, i) => (
        <Sequence
          key={i}
          from={90 + i * 60}
          durationInFrames={60}
        >
          <div className="point">{point}</div>
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};
```

## API Routes (Next.js)

### Generate Content Endpoint

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { ContentPipeline } from '@/lib/pipeline';

export async function POST(req: NextRequest) {
  try {
    const { keyword, format, language } = await req.json();
    
    const pipeline = new ContentPipeline({
      aiProvider: 'claude',
      apiKey: process.env.ANTHROPIC_API_KEY
    });
    
    const result = await pipeline.run({
      keyword,
      format,
      language
    });
    
    return NextResponse.json(result);
  } catch (error) {
    return NextResponse.json(
      { error: error.message },
      { status: 500 }
    );
  }
}
```

### Research Endpoint

```typescript
// app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { crawlNews } from '@/lib/crawler/news-crawler';

export async function POST(req: NextRequest) {
  const { keyword, sources } = await req.json();
  
  const data = await crawlNews({
    keyword,
    sources,
    apiKey: process.env.RAPIDAPI_KEY
  });
  
  return NextResponse.json(data);
}
```

## Troubleshooting

### API Rate Limiting

```typescript
import { RateLimiter } from '@/lib/utils/rate-limiter';

const limiter = new RateLimiter({
  requestsPerMinute: 50,
  provider: 'claude'
});

await limiter.wait(); // Automatically throttles requests
const response = await claude.generate({...});
```

### Error Handling

```typescript
import { PipelineError } from '@/lib/errors';

try {
  await pipeline.run(keyword);
} catch (error) {
  if (error instanceof PipelineError) {
    console.error('Pipeline stage:', error.stage);
    console.error('Details:', error.details);
    
    // Retry specific stage
    if (error.stage === 'research') {
      await pipeline.retryResearch();
    }
  }
}
```

### Video Rendering Issues

```typescript
// Check Remotion configuration
import { getCompositions } from '@remotion/renderer';

const compositions = await getCompositions(
  './remotion/index.ts',
  {
    inputProps: {},
  }
);

console.log('Available compositions:', compositions);
```

### Memory Management for Large Batches

```typescript
import { chunk } from '@/lib/utils/array-helpers';

const largeKeywordList = [...]; // 1000+ keywords

// Process in chunks to avoid memory issues
for (const keywordChunk of chunk(largeKeywordList, 10)) {
  await batchProcess(keywordChunk, processPipeline);
  
  // Allow garbage collection between chunks
  await new Promise(resolve => setTimeout(resolve, 1000));
}
```

## Testing

```bash
# Run tests
npm test

# Test specific module
npm test -- crawler

# Integration tests
npm run test:integration
```

```typescript
// Example test
import { generateContent } from '@/lib/ai/content-generator';

describe('Content Generator', () => {
  it('should generate toplist content', async () => {
    const result = await generateContent({
      provider: 'openai',
      topic: 'test topic',
      format: 'toplist',
      language: 'en'
    });
    
    expect(result.title).toBeDefined();
    expect(result.body).toContain('1.');
  });
});
```

This skill enables comprehensive use of the marketing-pipeline-share system for automated content creation workflows.
