---
name: ultimate-ai-content-pipeline
description: Automated Vietnamese/English content creation pipeline from research to video generation using Claude, OpenAI, and Remotion
triggers:
  - "set up automated content generation pipeline"
  - "create AI-powered content workflow with video generation"
  - "build multi-language content automation system"
  - "generate content from research to video automatically"
  - "integrate Claude and OpenAI for content creation"
  - "automate content research and video rendering"
  - "deploy content pipeline with Remotion"
  - "configure AI content generation with multi-format output"
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

A comprehensive TypeScript-based content automation system that handles the entire content lifecycle: automatic news research, AI-powered content generation in multiple formats and languages (Vietnamese/English), and automated video rendering using Remotion.

## What This Project Does

Ultimate AI Content Pipeline is an end-to-end content factory that:

1. **Auto-Research**: Crawls and analyzes real-time data from TechCrunch, a16z, Twitter/X, LinkedIn
2. **Multi-Format Generation**: Creates content in various formats (top lists, POV articles, case studies, how-tos) using Claude 3 and OpenAI
3. **Bilingual Output**: Generates content simultaneously in English and Vietnamese
4. **Video Automation**: Renders infographics and short-form videos via Remotion for Reels/TikTok/Shorts
5. **Flexible Architecture**: Integrates OpenAI, Anthropic Claude, and RapidAPI

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
# AI Provider Keys
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# Research & Data APIs
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_connection

# Next.js Configuration
NEXT_PUBLIC_APP_URL=http://localhost:3000

# Remotion Configuration (for video rendering)
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
│   │   ├── ai/          # AI provider integrations
│   │   ├── research/    # Content research modules
│   │   ├── generators/  # Content generation logic
│   │   └── video/       # Remotion video rendering
│   └── utils/           # Helper functions
├── public/              # Static assets
└── remotion/            # Remotion video templates
```

## Core Features & Usage

### 1. Content Research Module

Automatically crawl and analyze recent content from multiple sources:

```typescript
import { AutoResearch } from '@/lib/research/auto-research';

async function researchTopic(keyword: string) {
  const researcher = new AutoResearch({
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeframe: '24h',
    language: 'auto' // Detects both EN/VI
  });
  
  const insights = await researcher.scan(keyword);
  
  return {
    trends: insights.trends,
    statistics: insights.data,
    sources: insights.references,
    sentiment: insights.sentiment
  };
}

// Example usage
const aiTrends = await researchTopic('artificial intelligence startup funding');
console.log(aiTrends.trends); // Latest AI funding trends
```

### 2. AI Content Generation

Generate content in multiple formats using Claude or OpenAI:

```typescript
import { ContentGenerator } from '@/lib/generators/content-generator';

interface GenerateContentOptions {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi' | 'both';
  tone: 'expert' | 'friendly' | 'humorous';
  provider: 'claude' | 'openai';
}

async function generateContent(options: GenerateContentOptions) {
  const generator = new ContentGenerator({
    apiKey: options.provider === 'claude' 
      ? process.env.ANTHROPIC_API_KEY 
      : process.env.OPENAI_API_KEY,
    provider: options.provider,
    model: options.provider === 'claude' ? 'claude-3-opus-20240229' : 'gpt-4-turbo'
  });
  
  // First, research the topic
  const research = await researchTopic(options.keyword);
  
  // Generate content with research data
  const content = await generator.create({
    topic: options.keyword,
    format: options.format,
    language: options.language,
    tone: options.tone,
    context: research,
    includeImages: true,
    seoOptimized: true
  });
  
  return content;
}

// Example: Generate bilingual POV article
const article = await generateContent({
  keyword: 'AI in Vietnam market',
  format: 'pov',
  language: 'both',
  tone: 'expert',
  provider: 'claude'
});

console.log(article.english.title);
console.log(article.vietnamese.title);
```

### 3. Multi-Language Content Generation

Handle Vietnamese and English content simultaneously:

```typescript
import { BilingualGenerator } from '@/lib/generators/bilingual';

async function createBilingualPost(topic: string) {
  const generator = new BilingualGenerator({
    primaryLanguage: 'vi',
    secondaryLanguage: 'en',
    maintainTone: true
  });
  
  const result = await generator.generate({
    topic,
    sections: ['introduction', 'main-content', 'conclusion', 'cta'],
    keywords: {
      vi: ['công nghệ', 'đổi mới', 'AI'],
      en: ['technology', 'innovation', 'AI']
    }
  });
  
  return {
    vietnamese: result.vi,
    english: result.en,
    metadata: result.metadata
  };
}
```

### 4. Video Generation with Remotion

Convert content to video format:

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import { VideoComposition } from '@/remotion/compositions/ContentVideo';

async function generateContentVideo(content: any) {
  // Prepare video data from content
  const videoData = {
    title: content.title,
    keyPoints: content.keyPoints,
    stats: content.statistics,
    branding: {
      logo: '/logo.png',
      colors: ['#FF6B6B', '#4ECDC4']
    },
    duration: 60 // seconds
  };
  
  // Bundle Remotion project
  const bundleLocation = await bundle({
    entryPoint: './remotion/index.ts',
    webpackOverride: (config) => config,
  });
  
  // Select composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: videoData,
  });
  
  // Render video
  const outputLocation = `./public/videos/${Date.now()}.mp4`;
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: videoData,
  });
  
  return outputLocation;
}

// Generate video from article
const videoPath = await generateContentVideo(article);
```

### 5. Complete Pipeline Example

Full workflow from keyword to published content with video:

```typescript
import { ContentPipeline } from '@/lib/pipeline/content-pipeline';

async function runFullPipeline(keyword: string) {
  const pipeline = new ContentPipeline({
    research: {
      enabled: true,
      sources: ['techcrunch', 'twitter', 'linkedin']
    },
    generation: {
      provider: 'claude',
      formats: ['pov', 'toplist'],
      languages: ['en', 'vi']
    },
    video: {
      enabled: true,
      platforms: ['reels', 'tiktok', 'shorts']
    },
    publishing: {
      autoSchedule: true,
      platforms: ['facebook', 'linkedin']
    }
  });
  
  // Execute pipeline
  const result = await pipeline.execute({
    keyword,
    targetAudience: 'marketers',
    goals: ['engagement', 'awareness']
  });
  
  return {
    articles: result.content,
    videos: result.videos,
    scheduledPosts: result.scheduled,
    analytics: result.metadata
  };
}

// Example: Complete AI marketing campaign
const campaign = await runFullPipeline('AI marketing automation trends 2024');
```

## API Integration Patterns

### Claude API Integration

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

async function generateWithClaude(prompt: string, context?: any) {
  const message = await anthropic.messages.create({
    model: 'claude-3-opus-20240229',
    max_tokens: 4096,
    messages: [
      {
        role: 'user',
        content: `${context ? `Context: ${JSON.stringify(context)}\n\n` : ''}${prompt}`
      }
    ],
    system: 'You are an expert content marketer specializing in data-driven, SEO-optimized content creation.'
  });
  
  return message.content[0].text;
}
```

### OpenAI API Integration

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateWithOpenAI(prompt: string, format: string) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo',
    messages: [
      {
        role: 'system',
        content: `You are a professional content creator. Generate content in ${format} format.`
      },
      {
        role: 'user',
        content: prompt
      }
    ],
    temperature: 0.7,
    max_tokens: 3000
  });
  
  return completion.choices[0].message.content;
}
```

## Development Commands

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Start production server
npm run start

# Run Remotion studio (for video editing)
npm run remotion:studio

# Render specific video composition
npm run remotion:render

# Type checking
npm run type-check

# Linting
npm run lint
```

## Common Patterns

### Pattern 1: Scheduled Content Generation

```typescript
import cron from 'node-cron';

// Schedule daily content generation at 6 AM
cron.schedule('0 6 * * *', async () => {
  const topics = await getTrendingTopics();
  
  for (const topic of topics) {
    const content = await generateContent({
      keyword: topic,
      format: 'toplist',
      language: 'both',
      tone: 'friendly',
      provider: 'claude'
    });
    
    await saveToDatabase(content);
    await schedulePosting(content, '12:00');
  }
});
```

### Pattern 2: Batch Processing

```typescript
async function batchGenerateContent(keywords: string[]) {
  const results = await Promise.allSettled(
    keywords.map(keyword => 
      generateContent({
        keyword,
        format: 'how-to',
        language: 'both',
        tone: 'expert',
        provider: 'openai'
      })
    )
  );
  
  return results
    .filter(r => r.status === 'fulfilled')
    .map(r => r.value);
}
```

### Pattern 3: Content Variation Testing

```typescript
async function generateVariations(baseTopic: string) {
  const tones = ['expert', 'friendly', 'humorous'] as const;
  const formats = ['pov', 'toplist', 'case-study'] as const;
  
  const variations = [];
  
  for (const tone of tones) {
    for (const format of formats) {
      const content = await generateContent({
        keyword: baseTopic,
        format,
        language: 'en',
        tone,
        provider: 'claude'
      });
      
      variations.push({
        tone,
        format,
        content,
        id: `${tone}-${format}`
      });
    }
  }
  
  return variations;
}
```

## Troubleshooting

### API Rate Limits

```typescript
import { RateLimiter } from '@/lib/utils/rate-limiter';

const limiter = new RateLimiter({
  requestsPerMinute: 50,
  provider: 'claude'
});

async function generateWithRateLimit(prompt: string) {
  await limiter.acquire();
  
  try {
    return await generateWithClaude(prompt);
  } finally {
    limiter.release();
  }
}
```

### Error Handling

```typescript
import { RetryableError } from '@/lib/utils/errors';

async function robustContentGeneration(options: GenerateContentOptions) {
  const maxRetries = 3;
  let attempt = 0;
  
  while (attempt < maxRetries) {
    try {
      return await generateContent(options);
    } catch (error) {
      attempt++;
      
      if (error instanceof RetryableError && attempt < maxRetries) {
        await new Promise(resolve => setTimeout(resolve, 1000 * attempt));
        continue;
      }
      
      throw error;
    }
  }
}
```

### Video Rendering Memory Issues

```typescript
// Optimize Remotion rendering for large videos
const renderConfig = {
  concurrency: 1, // Reduce concurrent rendering
  enforceAudioTrack: false,
  muted: false,
  numberOfGifLoops: null,
  everyNthFrame: 1,
  frameRange: [0, 1800], // Limit frame range
  chromiumOptions: {
    headless: true,
    args: ['--no-sandbox', '--disable-setuid-sandbox']
  }
};
```

### Language Detection

```typescript
import { detectLanguage } from '@/lib/utils/language';

function ensureBilingualContent(content: any) {
  if (!content.vietnamese || !content.english) {
    const detected = detectLanguage(content.text);
    
    if (detected === 'vi' && !content.english) {
      // Translate to English
      content.english = await translateContent(content.vietnamese, 'en');
    } else if (detected === 'en' && !content.vietnamese) {
      // Translate to Vietnamese
      content.vietnamese = await translateContent(content.english, 'vi');
    }
  }
  
  return content;
}
```

## Best Practices

1. **Always validate research data** before feeding to AI generators
2. **Cache API responses** to minimize costs and improve performance
3. **Use webhooks** for long-running video rendering tasks
4. **Implement content moderation** before auto-publishing
5. **Track content performance** to optimize generation parameters
6. **Version control prompts** for reproducible content quality
7. **Monitor API usage** to stay within budget limits
