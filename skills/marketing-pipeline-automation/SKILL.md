---
name: marketing-pipeline-automation
description: Automated AI content pipeline for research, scripting, video generation, and multi-platform content creation using Claude/OpenAI and Remotion
triggers:
  - how do I automate content research and generation
  - set up AI marketing pipeline with video rendering
  - create automated content from keyword to video
  - use remotion for marketing content automation
  - build AI-powered content pipeline with Claude
  - automate social media content creation with AI
  - generate videos from text content automatically
  - set up multi-format content generation pipeline
---

# Marketing Pipeline Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection

This skill enables AI coding agents to work with the Ultimate AI Content Pipeline, a comprehensive system that automates content creation from research through video generation. The pipeline integrates Claude/OpenAI for content generation, web scraping for real-time research, and Remotion for video rendering.

## What It Does

The Marketing Pipeline automates:
- **Research**: Scrapes latest news from TechCrunch, a16z, Twitter/X, LinkedIn
- **Content Generation**: Creates articles in multiple formats (toplist, POV, case study, how-to)
- **Multi-language**: Generates English and Vietnamese content simultaneously
- **Video Production**: Renders infographics and short videos using Remotion
- **Platform Optimization**: Exports videos for Reels, TikTok, Shorts

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

## Environment Configuration

Create a `.env.local` file in the root directory:

```bash
# AI APIs
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# Research APIs (RapidAPI)
RAPIDAPI_KEY=your_rapidapi_key

# Content Sources
TECHCRUNCH_API_KEY=your_techcrunch_key
TWITTER_BEARER_TOKEN=your_twitter_token

# Database (if using)
DATABASE_URL=your_database_connection_string

# Remotion Configuration
REMOTION_AWS_ACCESS_KEY_ID=your_aws_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integrations (Claude, OpenAI)
│   │   ├── scraper/     # Web scraping modules
│   │   ├── content/     # Content generation logic
│   │   └── video/       # Remotion video rendering
│   ├── remotion/        # Remotion compositions
│   └── types/           # TypeScript definitions
├── public/              # Static assets
└── .env.local          # Environment variables
```

## Core API Usage

### 1. Research & Content Scraping

```typescript
import { researchTopic } from '@/lib/scraper/research';

// Scrape latest content for a keyword
async function gatherResearch(keyword: string) {
  const research = await researchTopic({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeframe: '24h',
    limit: 20
  });
  
  return {
    articles: research.articles,
    insights: research.insights,
    trends: research.trends,
    statistics: research.statistics
  };
}
```

### 2. AI Content Generation

```typescript
import { generateContent } from '@/lib/ai/content-generator';
import { ContentFormat, Language } from '@/types';

async function createArticle(keyword: string, research: any) {
  const content = await generateContent({
    keyword,
    research,
    format: ContentFormat.TOPLIST, // or POV, CASE_STUDY, HOW_TO
    languages: [Language.EN, Language.VI],
    tone: 'expert', // or 'friendly', 'humorous'
    aiProvider: 'claude', // or 'openai'
    model: 'claude-3-opus-20240229'
  });
  
  return {
    english: content.en,
    vietnamese: content.vi,
    metadata: content.metadata
  };
}
```

### 3. Claude Integration

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY
});

async function generateWithClaude(prompt: string, systemPrompt: string) {
  const message = await anthropic.messages.create({
    model: 'claude-3-opus-20240229',
    max_tokens: 4096,
    temperature: 0.7,
    system: systemPrompt,
    messages: [
      {
        role: 'user',
        content: prompt
      }
    ]
  });
  
  return message.content[0].text;
}
```

### 4. OpenAI Integration

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY
});

async function generateWithOpenAI(prompt: string, systemPrompt: string) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      { role: 'system', content: systemPrompt },
      { role: 'user', content: prompt }
    ],
    temperature: 0.7,
    max_tokens: 4000
  });
  
  return completion.choices[0].message.content;
}
```

### 5. Video Rendering with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import { VideoComposition } from '@/remotion/compositions/ContentVideo';

async function renderContentVideo(content: any) {
  // Bundle the Remotion project
  const bundled = await bundle({
    entryPoint: './src/remotion/index.ts',
    webpackOverride: (config) => config
  });
  
  // Select composition
  const composition = await selectComposition({
    serveUrl: bundled,
    id: 'ContentVideo',
    inputProps: {
      title: content.title,
      points: content.keyPoints,
      style: 'modern',
      platform: 'reels' // or 'tiktok', 'shorts'
    }
  });
  
  // Render video
  await renderMedia({
    composition,
    serveUrl: bundled,
    codec: 'h264',
    outputLocation: `out/${content.id}.mp4`,
    inputProps: composition.props
  });
  
  return `out/${content.id}.mp4`;
}
```

### 6. Remotion Video Composition

```typescript
// src/remotion/compositions/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig } from 'remotion';

interface ContentVideoProps {
  title: string;
  points: string[];
  style: 'modern' | 'minimal' | 'bold';
  platform: 'reels' | 'tiktok' | 'shorts';
}

export const ContentVideo: React.FC<ContentVideoProps> = ({
  title,
  points,
  style,
  platform
}) => {
  const frame = useCurrentFrame();
  const { fps, durationInFrames } = useVideoConfig();
  
  const opacity = Math.min(1, frame / 30);
  
  return (
    <AbsoluteFill
      style={{
        backgroundColor: '#000',
        justifyContent: 'center',
        alignItems: 'center'
      }}
    >
      <div style={{ opacity, padding: 40 }}>
        <h1 style={{ color: '#fff', fontSize: 48, marginBottom: 30 }}>
          {title}
        </h1>
        {points.map((point, idx) => (
          <p
            key={idx}
            style={{
              color: '#fff',
              fontSize: 32,
              marginBottom: 20,
              opacity: frame > (idx + 1) * 30 ? 1 : 0
            }}
          >
            • {point}
          </p>
        ))}
      </div>
    </AbsoluteFill>
  );
};
```

## Complete Pipeline Example

```typescript
import { researchTopic } from '@/lib/scraper/research';
import { generateContent } from '@/lib/ai/content-generator';
import { renderContentVideo } from '@/lib/video/renderer';
import { publishToSocial } from '@/lib/social/publisher';

async function runContentPipeline(keyword: string) {
  try {
    // Step 1: Research
    console.log('🔍 Researching topic...');
    const research = await researchTopic({
      keyword,
      sources: ['techcrunch', 'a16z', 'twitter'],
      timeframe: '24h',
      limit: 15
    });
    
    // Step 2: Generate Content
    console.log('✍️ Generating content...');
    const content = await generateContent({
      keyword,
      research,
      format: 'toplist',
      languages: ['en', 'vi'],
      tone: 'expert',
      aiProvider: 'claude'
    });
    
    // Step 3: Render Video
    console.log('🎬 Rendering video...');
    const videoPath = await renderContentVideo({
      title: content.en.title,
      keyPoints: content.en.keyPoints,
      style: 'modern',
      platform: 'reels'
    });
    
    // Step 4: Publish (optional)
    console.log('📤 Publishing...');
    await publishToSocial({
      content: content.en.body,
      video: videoPath,
      platforms: ['facebook', 'instagram', 'tiktok']
    });
    
    return {
      success: true,
      content,
      video: videoPath
    };
    
  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}

// Usage
runContentPipeline('AI marketing automation 2024');
```

## Development Commands

```bash
# Start Next.js development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Run Remotion studio for video preview
npm run remotion:studio

# Render a specific video composition
npm run remotion:render ContentVideo out/video.mp4

# Type checking
npm run type-check

# Linting
npm run lint
```

## Content Format Types

```typescript
// src/types/content.ts
export enum ContentFormat {
  TOPLIST = 'toplist',        // Top 10 lists
  POV = 'pov',                // Point of view/opinion
  CASE_STUDY = 'case_study',  // Case study analysis
  HOW_TO = 'how_to'           // Tutorial/guide
}

export enum Language {
  EN = 'en',
  VI = 'vi'
}

export enum ToneStyle {
  EXPERT = 'expert',
  FRIENDLY = 'friendly',
  HUMOROUS = 'humorous'
}

export interface ContentRequest {
  keyword: string;
  research: ResearchData;
  format: ContentFormat;
  languages: Language[];
  tone: ToneStyle;
  aiProvider: 'claude' | 'openai';
  model?: string;
}
```

## Common Patterns

### Multi-Format Content Generation

```typescript
async function generateMultiFormatContent(keyword: string) {
  const research = await researchTopic({ keyword });
  
  const formats = [
    ContentFormat.TOPLIST,
    ContentFormat.POV,
    ContentFormat.HOW_TO
  ];
  
  const results = await Promise.all(
    formats.map(format =>
      generateContent({
        keyword,
        research,
        format,
        languages: [Language.EN, Language.VI],
        tone: 'expert',
        aiProvider: 'claude'
      })
    )
  );
  
  return results;
}
```

### Batch Video Rendering

```typescript
async function renderBatchVideos(contents: any[], platform: string) {
  const renderPromises = contents.map((content, idx) =>
    renderContentVideo({
      title: content.title,
      keyPoints: content.keyPoints,
      style: 'modern',
      platform
    }).then(path => ({
      index: idx,
      path,
      content
    }))
  );
  
  return await Promise.all(renderPromises);
}
```

### Error Handling with Retry

```typescript
async function generateWithRetry(
  prompt: string,
  maxRetries: number = 3
) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await generateWithClaude(prompt, systemPrompt);
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      await new Promise(resolve => setTimeout(resolve, 1000 * (i + 1)));
    }
  }
}
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement rate limiting
import { RateLimiter } from 'limiter';

const limiter = new RateLimiter({
  tokensPerInterval: 10,
  interval: 'minute'
});

async function callAIWithRateLimit(prompt: string) {
  await limiter.removeTokens(1);
  return await generateWithClaude(prompt, systemPrompt);
}
```

### Video Rendering Memory Issues

```typescript
// Use smaller composition sizes or increase memory
const composition = await selectComposition({
  serveUrl: bundled,
  id: 'ContentVideo',
  inputProps: { /* props */ },
  // Reduce quality for faster rendering
  scale: 0.5
});
```

### Research Timeout Issues

```typescript
// Add timeout to scraping
async function researchWithTimeout(keyword: string, timeout: number = 30000) {
  return Promise.race([
    researchTopic({ keyword }),
    new Promise((_, reject) =>
      setTimeout(() => reject(new Error('Research timeout')), timeout)
    )
  ]);
}
```

### Claude/OpenAI Context Length

```typescript
// Chunk large research data
function chunkResearch(research: any, maxTokens: number = 8000) {
  const chunks = [];
  let currentChunk = [];
  let tokenCount = 0;
  
  for (const item of research.articles) {
    const itemTokens = item.content.length / 4; // rough estimate
    if (tokenCount + itemTokens > maxTokens) {
      chunks.push(currentChunk);
      currentChunk = [item];
      tokenCount = itemTokens;
    } else {
      currentChunk.push(item);
      tokenCount += itemTokens;
    }
  }
  
  if (currentChunk.length > 0) chunks.push(currentChunk);
  return chunks;
}
```

## Best Practices

1. **Always use environment variables** for API keys
2. **Implement retry logic** for AI API calls (rate limits are common)
3. **Cache research results** to avoid redundant scraping
4. **Use webhooks or queues** for long-running video renders
5. **Monitor token usage** to control costs
6. **Test video compositions** in Remotion Studio before batch rendering
7. **Validate content quality** before publishing to social platforms
