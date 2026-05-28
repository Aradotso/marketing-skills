---
name: ultimate-ai-content-pipeline
description: Automated content creation system with AI research, multi-format writing, and video generation using Claude, OpenAI, and Remotion
triggers:
  - how do I set up the AI content pipeline
  - generate automated content with research and video
  - create content from keyword to video automatically
  - use Claude and OpenAI for content generation
  - render videos from AI-generated content with Remotion
  - automate content research and video creation
  - build an AI-powered content workflow
  - integrate content crawling with video generation
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

## Overview

Ultimate AI Content Pipeline is a comprehensive TypeScript-based content automation system that transforms a single keyword into fully-researched articles and rendered videos. It combines:

- **Auto-research**: Web scraping from TechCrunch, a16z, Twitter/X, LinkedIn
- **AI content generation**: Multi-format content creation with Claude 3 and OpenAI
- **Video rendering**: Automatic infographic and video generation using Remotion
- **Multi-language support**: Simultaneous English and Vietnamese content generation

The system creates a complete content production pipeline from research to publication-ready assets.

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

Create a `.env.local` file in the project root:

```bash
# AI Provider Keys
ANTHROPIC_API_KEY=your_anthropic_key
OPENAI_API_KEY=your_openai_key

# Research API Keys
RAPIDAPI_KEY=your_rapidapi_key

# Optional: Custom endpoints
NEXT_PUBLIC_API_URL=http://localhost:3000

# Remotion Configuration
REMOTION_TIMEOUT=120000
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app router
│   ├── components/       # React components
│   ├── lib/             # Core utilities
│   │   ├── ai/          # AI provider integrations
│   │   ├── research/    # Web scraping modules
│   │   └── video/       # Remotion video generation
│   ├── types/           # TypeScript definitions
│   └── utils/           # Helper functions
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Core Components

### 1. Research Module

The research module crawls and aggregates content from multiple sources:

```typescript
import { researchTopic } from '@/lib/research/aggregator';

interface ResearchResult {
  articles: Article[];
  insights: string[];
  trends: string[];
  sources: string[];
}

// Research a topic automatically
const result = await researchTopic({
  keyword: 'AI marketing automation',
  sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
  timeframe: '24h',
  language: 'en'
});

console.log(result.insights);
// ["AI adoption in marketing increased 45% YoY", ...]
```

### 2. AI Content Generation

Generate content in multiple formats using Claude or OpenAI:

```typescript
import { generateContent } from '@/lib/ai/content-generator';

interface ContentOptions {
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  tone: 'expert' | 'friendly' | 'humorous';
  language: 'en' | 'vi';
  provider: 'claude' | 'openai';
}

// Generate a toplist article
const content = await generateContent({
  keyword: 'AI marketing tools',
  research: result, // from research module
  format: 'toplist',
  tone: 'expert',
  language: 'en',
  provider: 'claude'
});

console.log(content.title);
console.log(content.body);
console.log(content.metadata);
```

### 3. Multi-Language Generation

Create content in both English and Vietnamese simultaneously:

```typescript
import { generateBilingual } from '@/lib/ai/bilingual';

const bilingualContent = await generateBilingual({
  keyword: 'content automation',
  research: result,
  format: 'how-to',
  tone: 'friendly'
});

// Access English version
console.log(bilingualContent.en.title);
console.log(bilingualContent.en.body);

// Access Vietnamese version
console.log(bilingualContent.vi.title);
console.log(bilingualContent.vi.body);
```

### 4. Video Generation with Remotion

Transform content into video automatically:

```typescript
import { renderVideo } from '@/lib/video/renderer';
import { bundle } from '@remotion/bundler';
import { renderMedia } from '@remotion/renderer';

interface VideoConfig {
  template: 'infographic' | 'slideshow' | 'reels';
  aspectRatio: '16:9' | '9:16' | '1:1';
  duration: number;
}

// Generate video from content
const video = await renderVideo({
  content: content,
  config: {
    template: 'infographic',
    aspectRatio: '9:16', // For Reels/TikTok
    duration: 30
  }
});

console.log(video.outputPath);
// => /public/videos/output_12345.mp4
```

## API Routes

### POST /api/research

Research a topic and gather insights:

```typescript
// Request
const response = await fetch('/api/research', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    keyword: 'AI content creation',
    sources: ['techcrunch', 'twitter'],
    timeframe: '24h'
  })
});

const data = await response.json();
// { articles: [...], insights: [...], trends: [...] }
```

### POST /api/generate

Generate content from research:

```typescript
// Request
const response = await fetch('/api/generate', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    keyword: 'marketing automation',
    research: researchData,
    format: 'toplist',
    tone: 'expert',
    language: 'en',
    provider: 'claude'
  })
});

const content = await response.json();
// { title: "...", body: "...", metadata: {...} }
```

### POST /api/render-video

Render video from content:

```typescript
// Request
const response = await fetch('/api/render-video', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    contentId: content.id,
    template: 'reels',
    aspectRatio: '9:16'
  })
});

const video = await response.json();
// { videoUrl: "...", duration: 30, size: "..." }
```

## Complete Pipeline Example

End-to-end content generation workflow:

```typescript
import { ContentPipeline } from '@/lib/pipeline';

async function createCompleteContent(keyword: string) {
  const pipeline = new ContentPipeline({
    anthropicKey: process.env.ANTHROPIC_API_KEY!,
    openaiKey: process.env.OPENAI_API_KEY!,
    rapidApiKey: process.env.RAPIDAPI_KEY!
  });

  // Step 1: Research
  console.log('📡 Researching topic...');
  const research = await pipeline.research({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter'],
    timeframe: '24h'
  });

  // Step 2: Generate bilingual content
  console.log('🧠 Generating content...');
  const content = await pipeline.generateBilingual({
    keyword,
    research,
    format: 'toplist',
    tone: 'expert'
  });

  // Step 3: Render videos
  console.log('🎬 Rendering videos...');
  const videos = await pipeline.renderVideos({
    content: content.en,
    templates: [
      { template: 'reels', aspectRatio: '9:16' },
      { template: 'infographic', aspectRatio: '16:9' }
    ]
  });

  return {
    research,
    content,
    videos
  };
}

// Execute pipeline
const result = await createCompleteContent('AI marketing automation');
console.log('✅ Pipeline complete!');
console.log(`Generated ${result.videos.length} videos`);
```

## Custom Remotion Components

Create custom video templates:

```typescript
// remotion/compositions/CustomTemplate.tsx
import { AbsoluteFill, Sequence, useCurrentFrame } from 'remotion';

export const CustomTemplate: React.FC<{
  title: string;
  points: string[];
  duration: number;
}> = ({ title, points, duration }) => {
  const frame = useCurrentFrame();
  
  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a1a' }}>
      <Sequence from={0} durationInFrames={30}>
        <div style={{ 
          fontSize: 48, 
          color: 'white',
          textAlign: 'center',
          marginTop: 100
        }}>
          {title}
        </div>
      </Sequence>
      
      {points.map((point, index) => (
        <Sequence 
          key={index}
          from={30 + (index * 90)}
          durationInFrames={90}
        >
          <div style={{ 
            fontSize: 32, 
            color: 'white',
            padding: 40
          }}>
            {point}
          </div>
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};
```

Register the template:

```typescript
// remotion/index.ts
import { registerRoot } from 'remotion';
import { CustomTemplate } from './compositions/CustomTemplate';

registerRoot(() => {
  return (
    <>
      <Composition
        id="CustomTemplate"
        component={CustomTemplate}
        durationInFrames={300}
        fps={30}
        width={1080}
        height={1920}
      />
    </>
  );
});
```

## Content Formats

### Toplist Format

```typescript
const toplist = await generateContent({
  format: 'toplist',
  // Generates: "Top 10 AI Marketing Tools in 2024"
  // Structure: Numbered list with descriptions, pros/cons
});
```

### POV (Point of View) Format

```typescript
const pov = await generateContent({
  format: 'pov',
  // Generates: Opinion piece with strong viewpoint
  // Structure: Thesis → Arguments → Conclusion
});
```

### Case Study Format

```typescript
const caseStudy = await generateContent({
  format: 'case-study',
  // Generates: Real-world example analysis
  // Structure: Problem → Solution → Results
});
```

### How-To Format

```typescript
const howTo = await generateContent({
  format: 'how-to',
  // Generates: Step-by-step tutorial
  // Structure: Intro → Steps → Tips → Conclusion
});
```

## Troubleshooting

### API Rate Limits

```typescript
import { retry } from '@/lib/utils/retry';

// Automatic retry with exponential backoff
const content = await retry(
  () => generateContent(options),
  { maxRetries: 3, delay: 1000 }
);
```

### Video Rendering Timeout

```bash
# Increase timeout in .env.local
REMOTION_TIMEOUT=300000  # 5 minutes
```

### Memory Issues with Large Research

```typescript
// Paginate research results
const research = await researchTopic({
  keyword: 'AI tools',
  maxArticles: 20,  // Limit results
  summarize: true   // Pre-summarize content
});
```

### Claude API Errors

```typescript
// Fallback to OpenAI
try {
  const content = await generateContent({
    provider: 'claude',
    ...options
  });
} catch (error) {
  console.log('Claude failed, trying OpenAI...');
  const content = await generateContent({
    provider: 'openai',
    ...options
  });
}
```

## Development Commands

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Preview Remotion compositions
npm run remotion:preview

# Render specific Remotion video
npm run remotion:render -- CustomTemplate output.mp4
```

## Best Practices

1. **Cache research results** to avoid redundant API calls
2. **Use bilingual generation** when targeting multiple markets
3. **Pre-process content** before video rendering for better results
4. **Monitor API usage** to stay within rate limits
5. **Test video templates** with Remotion preview before batch rendering
6. **Store generated content** in a database for reuse and scheduling

This skill enables AI coding agents to effectively use the Ultimate AI Content Pipeline for automated, AI-powered content creation workflows.
