---
name: ultimate-ai-content-pipeline
description: Automated content generation pipeline from research to video using Claude, OpenAI, and Remotion
triggers:
  - how do I generate automated content with AI pipeline
  - set up content automation from research to video
  - create AI-powered content workflow with Remotion
  - automate content creation with Claude and OpenAI
  - build content pipeline with automatic research and video generation
  - use ultimate AI content pipeline for marketing automation
  - generate videos from content automatically with AI
  - set up automated content research and publishing system
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

Ultimate AI Content Pipeline is a comprehensive TypeScript-based content automation system that handles the entire content lifecycle: from researching trending topics, generating articles in multiple formats and languages, to automatically rendering videos and infographics using Remotion. Built with Next.js, it integrates Claude 3, OpenAI, and various data sources to create a complete content factory.

## What It Does

- **Auto-Research**: Crawls and analyzes real-time data from TechCrunch, a16z, Twitter/X, LinkedIn within the last 24 hours
- **Multi-Format Content**: Generates articles in various formats (Toplist, POV, Case Study, How-to) using Claude/OpenAI
- **Bilingual Support**: Creates content simultaneously in English and Vietnamese with customizable tone
- **Video Generation**: Automatically renders videos and infographics from content using Remotion
- **Multi-Platform Export**: Optimized video outputs for Reels, TikTok, Shorts

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
cp .env.example .env.local
```

## Environment Configuration

Create a `.env.local` file with the following variables:

```bash
# AI Provider Keys
ANTHROPIC_API_KEY=your_anthropic_key_here
OPENAI_API_KEY=your_openai_key_here

# Research API Keys
RAPIDAPI_KEY=your_rapidapi_key_here

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Remotion Configuration
REMOTION_AWS_ACCESS_KEY_ID=your_aws_access_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret_key

# Next.js
NEXT_PUBLIC_API_URL=http://localhost:3000
```

## Core Architecture

### Project Structure

```
marketing-pineline-share/
├── app/                    # Next.js app directory
├── components/             # React components
├── lib/
│   ├── ai/                # AI integration (Claude, OpenAI)
│   ├── research/          # Content research & crawling
│   ├── content/           # Content generation logic
│   └── video/             # Remotion video rendering
├── remotion/              # Remotion video templates
├── public/                # Static assets
└── types/                 # TypeScript definitions
```

## Usage Patterns

### 1. Research Module - Auto-Scan Content

```typescript
import { researchTopics } from '@/lib/research/scanner';

// Automatically research trending topics
const insights = await researchTopics({
  keyword: 'AI automation',
  sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
  timeRange: '24h',
  depth: 'comprehensive'
});

console.log(insights);
// {
//   trends: [...],
//   keyInsights: [...],
//   dataPoints: [...],
//   sources: [...]
// }
```

### 2. Content Generation with Claude

```typescript
import Anthropic from '@anthropic-ai/sdk';

const client = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

async function generateContent(topic: string, format: string) {
  const message = await client.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [{
      role: 'user',
      content: `Generate a ${format} article about ${topic} in both English and Vietnamese. 
                Include data-backed insights and maintain a professional yet engaging tone.`
    }]
  });

  return message.content;
}

// Generate toplist content
const content = await generateContent('AI marketing tools 2024', 'toplist');
```

### 3. Multi-Format Content Creation

```typescript
import { ContentGenerator } from '@/lib/content/generator';

const generator = new ContentGenerator({
  aiProvider: 'claude', // or 'openai'
  languages: ['en', 'vi'],
  tone: 'professional' // or 'friendly', 'humorous'
});

// Generate case study
const caseStudy = await generator.create({
  type: 'case-study',
  topic: 'How AI transformed content marketing',
  researchData: insights,
  includeStats: true,
  targetAudience: 'marketers'
});

// Generate POV article
const povArticle = await generator.create({
  type: 'pov',
  topic: 'The future of automated content',
  perspective: 'industry-expert',
  languages: ['en', 'vi']
});

// Generate how-to guide
const howToGuide = await generator.create({
  type: 'how-to',
  topic: 'Setting up automated content pipeline',
  stepByStep: true,
  includeCode: true
});
```

### 4. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import { webpackOverride } from './remotion/webpack-override';

async function renderContentVideo(content: any) {
  // Bundle Remotion project
  const bundleLocation = await bundle({
    entryPoint: './remotion/index.ts',
    webpackOverride,
  });

  // Select composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo', // Composition ID
    inputProps: {
      title: content.title,
      points: content.keyPoints,
      theme: 'modern'
    },
  });

  // Render video
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: `out/${content.id}.mp4`,
    inputProps: content,
  });
}

// Render for different platforms
const platforms = [
  { name: 'reels', width: 1080, height: 1920 },
  { name: 'tiktok', width: 1080, height: 1920 },
  { name: 'youtube-shorts', width: 1080, height: 1920 },
];

for (const platform of platforms) {
  await renderContentVideo({
    ...content,
    dimensions: platform
  });
}
```

### 5. Complete Pipeline Workflow

```typescript
import { ContentPipeline } from '@/lib/pipeline';

const pipeline = new ContentPipeline({
  researchEnabled: true,
  contentFormats: ['article', 'video', 'infographic'],
  autoPublish: false
});

// Run complete pipeline
const result = await pipeline.run({
  keyword: 'AI content automation 2024',
  targetLanguages: ['en', 'vi'],
  outputFormats: ['markdown', 'video', 'social-media'],
  videoSpecs: {
    platforms: ['reels', 'tiktok', 'shorts'],
    duration: 60, // seconds
    style: 'modern-minimal'
  }
});

console.log(result);
// {
//   research: { ... },
//   articles: { en: '...', vi: '...' },
//   videos: [
//     { platform: 'reels', url: '...' },
//     { platform: 'tiktok', url: '...' }
//   ],
//   socialPosts: { ... }
// }
```

### 6. API Routes (Next.js)

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { ContentPipeline } from '@/lib/pipeline';

export async function POST(request: NextRequest) {
  try {
    const { keyword, format, languages } = await request.json();
    
    const pipeline = new ContentPipeline();
    const content = await pipeline.run({
      keyword,
      targetLanguages: languages || ['en'],
      outputFormats: [format]
    });
    
    return NextResponse.json({ success: true, data: content });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

## Remotion Video Templates

### Basic Content Video Template

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig, interpolate } from 'remotion';

export const ContentVideo: React.FC<{
  title: string;
  points: string[];
  theme: string;
}> = ({ title, points, theme }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  
  const opacity = interpolate(frame, [0, 30], [0, 1], {
    extrapolateRight: 'clamp',
  });
  
  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a1a' }}>
      <div style={{ opacity, padding: 80 }}>
        <h1 style={{ 
          color: 'white', 
          fontSize: 72, 
          fontWeight: 'bold',
          marginBottom: 40 
        }}>
          {title}
        </h1>
        <ul style={{ color: 'white', fontSize: 36 }}>
          {points.map((point, i) => (
            <li key={i} style={{ marginBottom: 20 }}>
              {point}
            </li>
          ))}
        </ul>
      </div>
    </AbsoluteFill>
  );
};
```

## Common Workflows

### Workflow 1: Daily Trending Content

```typescript
import { DailyContentBot } from '@/lib/automation/daily-bot';

const bot = new DailyContentBot({
  schedule: '0 9 * * *', // 9 AM daily
  topics: ['AI', 'marketing', 'automation'],
  languages: ['en', 'vi'],
  autoPublish: true
});

// Start automated daily content generation
await bot.start();
```

### Workflow 2: Multi-Platform Campaign

```typescript
async function createCampaign(topic: string) {
  // 1. Research
  const research = await researchTopics({
    keyword: topic,
    sources: ['techcrunch', 'twitter'],
    timeRange: '24h'
  });
  
  // 2. Generate content variants
  const contentVariants = await Promise.all([
    generator.create({ type: 'toplist', topic, researchData: research }),
    generator.create({ type: 'pov', topic, researchData: research }),
    generator.create({ type: 'how-to', topic, researchData: research })
  ]);
  
  // 3. Render videos for each variant
  const videos = await Promise.all(
    contentVariants.map(content => renderContentVideo(content))
  );
  
  // 4. Generate social media posts
  const socialPosts = contentVariants.map(content => ({
    facebook: generateFacebookPost(content),
    twitter: generateTwitterThread(content),
    linkedin: generateLinkedInPost(content)
  }));
  
  return { contentVariants, videos, socialPosts };
}
```

## Configuration Options

### Content Generator Config

```typescript
interface ContentConfig {
  aiProvider: 'claude' | 'openai';
  languages: ('en' | 'vi')[];
  tone: 'professional' | 'friendly' | 'humorous' | 'expert';
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  wordCount?: number;
  includeStats?: boolean;
  includeCitations?: boolean;
}
```

### Video Rendering Config

```typescript
interface VideoConfig {
  platforms: ('reels' | 'tiktok' | 'shorts' | 'youtube')[];
  duration: number; // in seconds
  style: 'modern-minimal' | 'vibrant' | 'corporate' | 'creative';
  fps: 30 | 60;
  codec: 'h264' | 'h265';
  quality: 'high' | 'medium' | 'low';
}
```

## Troubleshooting

### API Rate Limits

```typescript
import { RateLimiter } from '@/lib/utils/rate-limiter';

const limiter = new RateLimiter({
  requests: 50,
  per: 'minute'
});

await limiter.throttle(async () => {
  return await client.messages.create({...});
});
```

### Video Rendering Failures

```typescript
try {
  await renderMedia({...});
} catch (error) {
  if (error.message.includes('Out of memory')) {
    // Reduce video quality or duration
    console.log('Retrying with lower quality...');
    await renderMedia({
      ...config,
      quality: 'medium',
      scale: 0.75
    });
  }
}
```

### Research Data Quality

```typescript
// Filter and validate research results
function validateResearchData(insights: any) {
  return {
    ...insights,
    trends: insights.trends.filter(t => t.confidence > 0.7),
    sources: insights.sources.filter(s => s.authority > 0.5)
  };
}

const validatedInsights = validateResearchData(insights);
```

## Development Commands

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Run production server
npm start

# Render Remotion video locally
npm run remotion:render

# Preview Remotion compositions
npm run remotion:preview

# Type checking
npm run type-check

# Linting
npm run lint
```

## Best Practices

1. **Always validate research data** before content generation to ensure quality
2. **Use environment variables** for all API keys and sensitive configuration
3. **Implement rate limiting** when working with external APIs
4. **Cache research results** to avoid redundant API calls
5. **Test video renders** with short durations before full production
6. **Monitor AI token usage** to control costs
7. **Version control video templates** separately for easier iteration
