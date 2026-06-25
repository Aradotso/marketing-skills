---
name: ultimate-ai-content-pipeline
description: Automated content pipeline that researches, generates scripts, and creates videos using AI (Claude/OpenAI) and Remotion
triggers:
  - how do I automate content creation from research to video
  - generate content pipeline with AI research and video rendering
  - set up automated marketing content system with Claude
  - create videos from AI-generated content automatically
  - build end-to-end content automation with research crawling
  - use Remotion to render videos from AI content
  - automate content workflow from keyword to published video
  - integrate Claude and OpenAI for content generation pipeline
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI agents to use the Ultimate AI Content Pipeline - a comprehensive TypeScript-based system that automates the entire content creation workflow: from research scraping, to AI-powered content generation, to automated video rendering with Remotion.

## What It Does

The Ultimate AI Content Pipeline automates 90% of the content creation workflow:

1. **Auto-Research**: Crawls real-time data from sources like TechCrunch, a16z, Twitter/X, LinkedIn
2. **AI Content Generation**: Creates articles in multiple formats (toplist, POV, case study, how-to) using Claude 3 or OpenAI
3. **Multi-language Support**: Generates content in English and Vietnamese simultaneously
4. **Video Rendering**: Automatically converts written content into videos and infographics using Remotion
5. **Platform Optimization**: Exports video in formats optimized for Reels, TikTok, and YouTube Shorts

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

## Configuration

Create a `.env` file in the project root with the following variables:

```env
# AI Provider Keys
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Next.js Configuration
NEXT_PUBLIC_API_URL=http://localhost:3000

# Remotion Configuration
REMOTION_CONCURRENCY=4
REMOTION_VIDEO_FORMAT=mp4
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI provider integrations
│   │   ├── research/    # Web scraping & data gathering
│   │   ├── content/     # Content generation logic
│   │   └── video/       # Remotion video rendering
│   └── types/           # TypeScript type definitions
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Key APIs and Usage Patterns

### 1. Research & Data Gathering

```typescript
import { researchTopic } from '@/lib/research/crawler';

// Automatically crawl and analyze recent content
async function gatherResearch(keyword: string) {
  const research = await researchTopic({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeRange: '24h',
    limit: 50
  });
  
  return {
    insights: research.insights,
    statistics: research.statistics,
    trendingTopics: research.trends,
    sourceUrls: research.sources
  };
}
```

### 2. AI Content Generation

```typescript
import { generateContent } from '@/lib/content/generator';
import { ContentFormat, Language, ToneOfVoice } from '@/types';

// Generate content using Claude or OpenAI
async function createArticle(topic: string, research: any) {
  const content = await generateContent({
    provider: 'claude', // or 'openai'
    model: 'claude-3-5-sonnet-20241022',
    topic,
    format: ContentFormat.HOW_TO,
    languages: [Language.ENGLISH, Language.VIETNAMESE],
    tone: ToneOfVoice.EXPERT,
    research,
    includeStatistics: true,
    wordCount: 1500
  });
  
  return {
    title: content.title,
    body: content.body,
    metadata: content.metadata,
    translations: content.translations
  };
}
```

### 3. Claude Integration

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

async function generateWithClaude(prompt: string, systemPrompt: string) {
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    system: systemPrompt,
    messages: [
      {
        role: 'user',
        content: prompt
      }
    ],
    temperature: 0.7
  });
  
  return message.content[0].text;
}
```

### 4. OpenAI Integration

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateWithOpenAI(prompt: string, systemPrompt: string) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      { role: 'system', content: systemPrompt },
      { role: 'user', content: prompt }
    ],
    temperature: 0.7,
    max_tokens: 4096
  });
  
  return completion.choices[0].message.content;
}
```

### 5. Video Rendering with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import { webpackOverride } from './remotion/webpack-override';

async function renderContentVideo(content: any) {
  // Bundle Remotion project
  const bundleLocation = await bundle({
    entryPoint: './remotion/index.ts',
    webpackOverride
  });
  
  // Select composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title: content.title,
      body: content.body,
      statistics: content.statistics
    }
  });
  
  // Render video
  const outputLocation = `./output/video-${Date.now()}.mp4`;
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: composition.inputProps
  });
  
  return outputLocation;
}
```

### 6. Complete Pipeline Workflow

```typescript
import { researchTopic } from '@/lib/research/crawler';
import { generateContent } from '@/lib/content/generator';
import { renderContentVideo } from '@/lib/video/renderer';
import { publishToSocial } from '@/lib/social/publisher';

async function runContentPipeline(keyword: string) {
  try {
    // Step 1: Research
    console.log('🔍 Researching topic...');
    const research = await researchTopic({
      keyword,
      sources: ['techcrunch', 'a16z', 'twitter'],
      timeRange: '24h'
    });
    
    // Step 2: Generate Content
    console.log('✍️ Generating content...');
    const content = await generateContent({
      provider: 'claude',
      topic: keyword,
      format: ContentFormat.TOPLIST,
      research,
      languages: [Language.ENGLISH, Language.VIETNAMESE]
    });
    
    // Step 3: Render Video
    console.log('🎬 Rendering video...');
    const videoPath = await renderContentVideo({
      title: content.title,
      body: content.body,
      format: 'reels' // or 'tiktok', 'shorts'
    });
    
    // Step 4: Publish (optional)
    console.log('📤 Publishing...');
    await publishToSocial({
      content: content.body,
      video: videoPath,
      platforms: ['facebook', 'instagram', 'tiktok']
    });
    
    return {
      success: true,
      content,
      videoPath
    };
  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}

// Usage
runContentPipeline('AI marketing automation').then(result => {
  console.log('✅ Pipeline completed!', result);
});
```

## Content Format Types

```typescript
export enum ContentFormat {
  TOPLIST = 'toplist',
  POV = 'pov',
  CASE_STUDY = 'case_study',
  HOW_TO = 'how_to',
  NEWS = 'news',
  COMPARISON = 'comparison'
}

export enum ToneOfVoice {
  EXPERT = 'expert',
  FRIENDLY = 'friendly',
  HUMOROUS = 'humorous',
  PROFESSIONAL = 'professional',
  CASUAL = 'casual'
}

export enum Language {
  ENGLISH = 'en',
  VIETNAMESE = 'vi'
}
```

## Remotion Video Template Example

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, interpolate } from 'remotion';

export const ContentVideo: React.FC<{
  title: string;
  body: string;
  statistics?: string[];
}> = ({ title, body, statistics }) => {
  const frame = useCurrentFrame();
  
  const opacity = interpolate(frame, [0, 30], [0, 1], {
    extrapolateRight: 'clamp',
  });
  
  return (
    <AbsoluteFill
      style={{
        backgroundColor: '#1a1a1a',
        justifyContent: 'center',
        alignItems: 'center',
        padding: 60,
      }}
    >
      <div style={{ opacity }}>
        <h1 style={{ color: 'white', fontSize: 72, marginBottom: 40 }}>
          {title}
        </h1>
        <p style={{ color: '#ccc', fontSize: 32, lineHeight: 1.6 }}>
          {body.substring(0, 200)}...
        </p>
        {statistics && (
          <div style={{ marginTop: 40 }}>
            {statistics.map((stat, i) => (
              <div key={i} style={{ color: '#00ff88', fontSize: 28, marginTop: 20 }}>
                📊 {stat}
              </div>
            ))}
          </div>
        )}
      </div>
    </AbsoluteFill>
  );
};
```

## CLI Commands

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Run research crawler
npm run research -- --keyword "AI marketing" --sources "techcrunch,a16z"

# Generate content
npm run generate -- --topic "Content automation" --format "how-to"

# Render video
npm run render -- --composition "ContentVideo" --output "./videos"

# Run full pipeline
npm run pipeline -- --keyword "AI trends 2024"
```

## Common Patterns

### Pattern 1: Custom Research Sources

```typescript
import { ResearchSource } from '@/types';

const customSources: ResearchSource[] = [
  {
    name: 'TechCrunch',
    url: 'https://techcrunch.com',
    selector: '.post-block',
    type: 'rss'
  },
  {
    name: 'YCombinator',
    url: 'https://news.ycombinator.com',
    selector: '.athing',
    type: 'html'
  }
];

const research = await researchTopic({
  keyword: 'startup funding',
  customSources,
  timeRange: '48h'
});
```

### Pattern 2: Multi-Provider Fallback

```typescript
async function generateWithFallback(prompt: string) {
  try {
    return await generateWithClaude(prompt, systemPrompt);
  } catch (claudeError) {
    console.warn('Claude failed, falling back to OpenAI');
    return await generateWithOpenAI(prompt, systemPrompt);
  }
}
```

### Pattern 3: Batch Content Generation

```typescript
async function generateBatch(keywords: string[]) {
  const results = await Promise.allSettled(
    keywords.map(keyword => runContentPipeline(keyword))
  );
  
  return results.map((result, i) => ({
    keyword: keywords[i],
    success: result.status === 'fulfilled',
    data: result.status === 'fulfilled' ? result.value : null,
    error: result.status === 'rejected' ? result.reason : null
  }));
}
```

## Troubleshooting

### Issue: API Rate Limits

```typescript
// Implement rate limiting
import pLimit from 'p-limit';

const limit = pLimit(5); // Max 5 concurrent requests

const results = await Promise.all(
  keywords.map(keyword => 
    limit(() => runContentPipeline(keyword))
  )
);
```

### Issue: Video Rendering Timeout

```typescript
// Increase timeout in Remotion config
await renderMedia({
  composition,
  serveUrl: bundleLocation,
  codec: 'h264',
  outputLocation,
  timeoutInMilliseconds: 300000, // 5 minutes
  concurrency: 2 // Reduce if needed
});
```

### Issue: Large Content Token Limits

```typescript
// Chunk content for large articles
function chunkContent(text: string, maxTokens: number = 3000) {
  const words = text.split(' ');
  const chunks = [];
  
  for (let i = 0; i < words.length; i += maxTokens) {
    chunks.push(words.slice(i, i + maxTokens).join(' '));
  }
  
  return chunks;
}

// Generate in chunks
const chunks = chunkContent(research.fullText);
const contentParts = await Promise.all(
  chunks.map(chunk => generateContent({ ...config, research: chunk }))
);
```

### Issue: Missing Research Data

```typescript
// Add validation and retry logic
async function researchWithRetry(keyword: string, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      const research = await researchTopic({ keyword });
      
      if (research.insights.length === 0) {
        throw new Error('No insights found');
      }
      
      return research;
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      await new Promise(resolve => setTimeout(resolve, 2000 * (i + 1)));
    }
  }
}
```

## Best Practices

1. **Always validate research data** before generating content
2. **Use environment variables** for all API keys and secrets
3. **Implement error handling** at each pipeline stage
4. **Cache research results** to avoid redundant API calls
5. **Monitor AI costs** by tracking token usage
6. **Test video rendering** with small compositions first
7. **Use TypeScript types** for better code safety
8. **Implement retry logic** for external API calls
9. **Store generated content** before publishing
10. **Log pipeline progress** for debugging and monitoring
