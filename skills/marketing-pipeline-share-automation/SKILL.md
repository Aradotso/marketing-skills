---
name: marketing-pipeline-share-automation
description: Automated AI content pipeline for research, scriptwriting, posting, and video generation using Claude, OpenAI, and Remotion
triggers:
  - automate content creation with AI pipeline
  - generate marketing content from research to video
  - set up automated content workflow with Claude
  - create AI-powered content generation system
  - build content pipeline with video rendering
  - automate blog posts and social media videos
  - integrate Claude and OpenAI for content automation
  - use Remotion to generate marketing videos
---

# Marketing Pipeline Share Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the Ultimate AI Content Pipeline - an end-to-end automated content generation system that handles research, scriptwriting, content creation, and video generation using Claude 3, OpenAI, and Remotion.

## What This Project Does

Marketing Pipeline Share is a comprehensive TypeScript-based automation system that:

- **Auto-scans research**: Crawls real-time data from TechCrunch, a16z, Twitter/X, LinkedIn within 24 hours
- **Generates multi-format content**: Creates articles in various formats (top lists, POV, case studies, how-to) using Claude/OpenAI
- **Multi-language support**: Automatically produces content in both English and Vietnamese
- **Auto-renders videos**: Converts written content into infographics and short-form videos using Remotion
- **Platform optimization**: Exports videos optimized for Reels, TikTok, and Shorts

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

### Required Environment Variables

```bash
# AI Provider API Keys
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_url

# Next.js Configuration
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

### Running the Application

```bash
# Development mode
npm run dev

# Production build
npm run build
npm run start

# Video rendering (Remotion)
npm run render
```

## Core Architecture

### Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/             # Core libraries
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── crawler/     # Research crawling logic
│   │   ├── content/     # Content generation
│   │   └── video/       # Remotion video rendering
│   ├── types/           # TypeScript types
│   └── utils/           # Utility functions
├── remotion/            # Video templates
└── public/              # Static assets
```

## Key Features & Usage

### 1. Research Crawling

```typescript
import { crawlResearch } from '@/lib/crawler';

// Crawl latest news from multiple sources
async function gatherResearch(keyword: string) {
  const sources = ['techcrunch', 'a16z', 'twitter', 'linkedin'];
  
  const research = await crawlResearch({
    keyword,
    sources,
    timeRange: '24h',
    maxResults: 50
  });
  
  return research;
}

// Example usage
const insights = await gatherResearch('AI marketing automation');
console.log(insights.articles);
console.log(insights.trendingTopics);
```

### 2. Content Generation with Claude

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

async function generateContent(
  topic: string,
  format: 'toplist' | 'pov' | 'case-study' | 'how-to',
  language: 'en' | 'vi'
) {
  const prompts = {
    toplist: `Create a top 10 list about ${topic} with data-backed insights`,
    pov: `Write a thought-provoking point of view article about ${topic}`,
    'case-study': `Develop a detailed case study analyzing ${topic}`,
    'how-to': `Create a comprehensive how-to guide for ${topic}`
  };

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [{
      role: 'user',
      content: `${prompts[format]}. Language: ${language}. Include recent data and trends.`
    }]
  });

  return message.content[0].text;
}

// Generate bilingual content
const contentEN = await generateContent('AI content automation', 'toplist', 'en');
const contentVI = await generateContent('AI content automation', 'toplist', 'vi');
```

### 3. OpenAI Integration

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateWithGPT(prompt: string, tone: string) {
  const toneInstructions = {
    expert: 'Use professional, authoritative language with industry terminology',
    friendly: 'Write in a conversational, approachable tone',
    humorous: 'Include light humor and engaging storytelling'
  };

  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are a marketing content expert. ${toneInstructions[tone]}`
      },
      {
        role: 'user',
        content: prompt
      }
    ],
    temperature: 0.7,
  });

  return completion.choices[0].message.content;
}
```

### 4. Video Generation with Remotion

```typescript
// remotion/VideoComposition.tsx
import { Composition } from 'remotion';
import { MarketingVideo } from './templates/MarketingVideo';

export const RemotionRoot: React.FC = () => {
  return (
    <>
      <Composition
        id="MarketingVideo"
        component={MarketingVideo}
        durationInFrames={300}
        fps={30}
        width={1080}
        height={1920}
        defaultProps={{
          title: 'AI Marketing Trends',
          content: [],
          branding: {}
        }}
      />
    </>
  );
};

// Render video programmatically
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';

async function renderContentVideo(
  contentData: { title: string; points: string[] }
) {
  const bundleLocation = await bundle({
    entryPoint: './remotion/index.ts',
    webpackOverride: (config) => config,
  });

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'MarketingVideo',
    inputProps: contentData,
  });

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: `out/${contentData.title}.mp4`,
  });
}
```

### 5. Complete Pipeline Workflow

```typescript
import { crawlResearch } from '@/lib/crawler';
import { generateContent } from '@/lib/ai/claude';
import { renderContentVideo } from '@/lib/video/remotion';
import { publishToSocial } from '@/lib/social/publisher';

async function runContentPipeline(keyword: string) {
  try {
    // Step 1: Research
    console.log('🔍 Gathering research...');
    const research = await crawlResearch({
      keyword,
      sources: ['techcrunch', 'twitter'],
      timeRange: '24h'
    });

    // Step 2: Generate content
    console.log('✍️ Generating content...');
    const content = await generateContent(
      keyword,
      'toplist',
      'en'
    );

    // Step 3: Create video
    console.log('🎬 Rendering video...');
    const videoData = {
      title: keyword,
      points: extractKeyPoints(content),
      insights: research.trendingTopics.slice(0, 5)
    };
    
    await renderContentVideo(videoData);

    // Step 4: Publish
    console.log('📤 Publishing...');
    await publishToSocial({
      content,
      video: `out/${keyword}.mp4`,
      platforms: ['facebook', 'linkedin', 'twitter']
    });

    return {
      success: true,
      content,
      videoPath: `out/${keyword}.mp4`
    };
  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}

// Helper function
function extractKeyPoints(content: string): string[] {
  // Parse markdown or structured content
  const lines = content.split('\n');
  return lines
    .filter(line => line.match(/^\d+\.|^-|^•/))
    .map(line => line.replace(/^\d+\.\s*|^-\s*|^•\s*/, ''))
    .slice(0, 10);
}
```

## API Integration Patterns

### RapidAPI for Data Enrichment

```typescript
import axios from 'axios';

async function enrichWithRapidAPI(topic: string) {
  const options = {
    method: 'GET',
    url: 'https://api.rapidapi.com/search',
    params: { q: topic, limit: '10' },
    headers: {
      'X-RapidAPI-Key': process.env.RAPIDAPI_KEY,
      'X-RapidAPI-Host': 'your-api-host.rapidapi.com'
    }
  };

  try {
    const response = await axios.request(options);
    return response.data;
  } catch (error) {
    console.error('RapidAPI error:', error);
    return null;
  }
}
```

## Configuration

### Content Generation Config

```typescript
// src/config/content.ts
export const contentConfig = {
  formats: ['toplist', 'pov', 'case-study', 'how-to'],
  languages: ['en', 'vi'],
  tones: ['expert', 'friendly', 'humorous'],
  
  ai: {
    claude: {
      model: 'claude-3-5-sonnet-20241022',
      maxTokens: 4096,
      temperature: 0.7
    },
    openai: {
      model: 'gpt-4-turbo-preview',
      maxTokens: 3000,
      temperature: 0.7
    }
  },

  video: {
    defaultFps: 30,
    platforms: {
      reels: { width: 1080, height: 1920 },
      tiktok: { width: 1080, height: 1920 },
      youtube: { width: 1920, height: 1080 }
    }
  }
};
```

### Crawler Configuration

```typescript
// src/config/crawler.ts
export const crawlerConfig = {
  sources: {
    techcrunch: {
      baseUrl: 'https://techcrunch.com',
      selectors: {
        article: '.post-block',
        title: '.post-block__title',
        content: '.article-content'
      }
    },
    twitter: {
      apiVersion: 'v2',
      maxResults: 100
    }
  },
  
  rateLimits: {
    requestsPerMinute: 30,
    concurrent: 5
  }
};
```

## Common Patterns

### Batch Content Generation

```typescript
async function batchGenerateContent(keywords: string[]) {
  const results = await Promise.allSettled(
    keywords.map(async (keyword) => {
      const content = await generateContent(keyword, 'toplist', 'en');
      const contentVI = await generateContent(keyword, 'toplist', 'vi');
      
      return {
        keyword,
        en: content,
        vi: contentVI
      };
    })
  );

  return results
    .filter(r => r.status === 'fulfilled')
    .map(r => r.value);
}
```

### Error Handling & Retries

```typescript
async function withRetry<T>(
  fn: () => Promise<T>,
  maxRetries = 3,
  delay = 1000
): Promise<T> {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      await new Promise(resolve => setTimeout(resolve, delay * (i + 1)));
    }
  }
  throw new Error('Max retries exceeded');
}

// Usage
const content = await withRetry(() => 
  generateContent('AI trends', 'toplist', 'en')
);
```

### Scheduling Content

```typescript
import cron from 'node-cron';

// Schedule daily content generation at 9 AM
cron.schedule('0 9 * * *', async () => {
  const keywords = ['AI marketing', 'content automation', 'video trends'];
  
  for (const keyword of keywords) {
    await runContentPipeline(keyword);
  }
});
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement queue system
import PQueue from 'p-queue';

const queue = new PQueue({
  concurrency: 2,
  interval: 60000, // 1 minute
  intervalCap: 30  // 30 requests per minute
});

async function queuedGenerate(prompt: string) {
  return queue.add(() => generateContent(prompt, 'toplist', 'en'));
}
```

### Video Rendering Memory Issues

```bash
# Increase Node.js memory limit
NODE_OPTIONS="--max-old-space-size=4096" npm run render
```

### Claude API Errors

```typescript
async function generateContentSafe(topic: string) {
  try {
    return await generateContent(topic, 'toplist', 'en');
  } catch (error) {
    if (error.status === 529) {
      console.log('API overloaded, waiting...');
      await new Promise(resolve => setTimeout(resolve, 5000));
      return generateContent(topic, 'toplist', 'en');
    }
    throw error;
  }
}
```

### Missing Environment Variables

```typescript
function validateEnv() {
  const required = [
    'OPENAI_API_KEY',
    'ANTHROPIC_API_KEY',
    'RAPIDAPI_KEY'
  ];
  
  const missing = required.filter(key => !process.env[key]);
  
  if (missing.length > 0) {
    throw new Error(
      `Missing required environment variables: ${missing.join(', ')}`
    );
  }
}

// Call at startup
validateEnv();
```

## Advanced Usage

### Custom Video Templates

```typescript
// remotion/templates/CustomTemplate.tsx
import { AbsoluteFill, useCurrentFrame, interpolate } from 'remotion';

export const CustomMarketingVideo: React.FC<{
  title: string;
  points: string[];
}> = ({ title, points }) => {
  const frame = useCurrentFrame();
  
  const opacity = interpolate(frame, [0, 30], [0, 1], {
    extrapolateRight: 'clamp',
  });

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

This skill provides comprehensive guidance for AI coding agents to effectively use the Marketing Pipeline Share automation system for end-to-end content creation workflows.
