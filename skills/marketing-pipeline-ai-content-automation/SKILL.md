---
name: marketing-pipeline-ai-content-automation
description: Automate content creation from research to video generation using AI (Claude, OpenAI) and Remotion for Vietnamese and English marketing content
triggers:
  - how do I automate content creation with AI
  - set up automated marketing pipeline
  - generate videos from articles automatically
  - create content with Claude and OpenAI
  - build AI content automation workflow
  - use Remotion for automated video rendering
  - scrape and generate marketing content
  - automate research to video pipeline
---

# Marketing Pipeline AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill covers **marketing-pipeline-share**, an AI-powered content automation system that handles the entire pipeline: research/scraping → AI content generation → automated video rendering. Built with TypeScript, Next.js, Claude/OpenAI APIs, and Remotion.

## What This Project Does

The marketing-pipeline-share project is a complete content production line that:

- **Auto-scrapes** trending news from TechCrunch, a16z, Twitter, LinkedIn
- **Generates content** in multiple formats (toplist, POV, case study, how-to) using Claude 3 or OpenAI
- **Supports bilingual output** (Vietnamese + English) with customizable tone
- **Renders videos automatically** using Remotion for social media (Reels, TikTok, Shorts)
- **Integrates** with RapidAPI, Anthropic, and OpenAI for flexible AI operations

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

### Environment Configuration

Create a `.env.local` file in the root directory:

```bash
# AI API Keys
ANTHROPIC_API_KEY=your_claude_api_key_here
OPENAI_API_KEY=your_openai_api_key_here

# RapidAPI for scraping
RAPIDAPI_KEY=your_rapidapi_key_here

# Database (if applicable)
DATABASE_URL=your_database_connection_string

# Remotion configuration
REMOTION_AWS_ACCESS_KEY_ID=your_aws_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret

# Next.js
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

### Running the Development Server

```bash
npm run dev
# or
yarn dev
# or
pnpm dev
```

Navigate to `http://localhost:3000` to access the interface.

## Key Components & Architecture

### 1. Research/Scraping Module

Automatically crawls content from configured sources:

```typescript
// lib/scrapers/news-scraper.ts
import axios from 'axios';

interface ScrapedArticle {
  title: string;
  url: string;
  content: string;
  publishedAt: Date;
  source: string;
}

export async function scrapeRecentNews(
  keyword: string,
  sources: string[] = ['techcrunch', 'a16z']
): Promise<ScrapedArticle[]> {
  const articles: ScrapedArticle[] = [];
  
  for (const source of sources) {
    try {
      const response = await axios.get(
        `https://api.rapidapi.com/news/${source}`,
        {
          headers: {
            'X-RapidAPI-Key': process.env.RAPIDAPI_KEY!,
            'X-RapidAPI-Host': 'news-scraper.p.rapidapi.com'
          },
          params: {
            q: keyword,
            freshness: 'Day'
          }
        }
      );
      
      articles.push(...response.data.articles.map((article: any) => ({
        title: article.title,
        url: article.url,
        content: article.description,
        publishedAt: new Date(article.publishedAt),
        source: source
      })));
    } catch (error) {
      console.error(`Failed to scrape ${source}:`, error);
    }
  }
  
  return articles;
}
```

### 2. AI Content Generation

Generate content using Claude or OpenAI:

```typescript
// lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

export type ContentFormat = 'toplist' | 'pov' | 'case-study' | 'how-to';
export type Language = 'vi' | 'en';
export type Tone = 'expert' | 'friendly' | 'humorous';

interface GenerateContentOptions {
  keyword: string;
  research: string[];
  format: ContentFormat;
  language: Language;
  tone: Tone;
  provider?: 'claude' | 'openai';
}

export async function generateContent(
  options: GenerateContentOptions
): Promise<string> {
  const { keyword, research, format, language, tone, provider = 'claude' } = options;
  
  const prompt = buildPrompt(keyword, research, format, language, tone);
  
  if (provider === 'claude') {
    const message = await anthropic.messages.create({
      model: 'claude-3-5-sonnet-20241022',
      max_tokens: 4096,
      messages: [
        {
          role: 'user',
          content: prompt
        }
      ]
    });
    
    return message.content[0].type === 'text' 
      ? message.content[0].text 
      : '';
  } else {
    const completion = await openai.chat.completions.create({
      model: 'gpt-4-turbo-preview',
      messages: [
        {
          role: 'system',
          content: 'You are an expert content writer and marketer.'
        },
        {
          role: 'user',
          content: prompt
        }
      ],
      max_tokens: 4096
    });
    
    return completion.choices[0]?.message?.content || '';
  }
}

function buildPrompt(
  keyword: string,
  research: string[],
  format: ContentFormat,
  language: Language,
  tone: Tone
): string {
  const languageInstruction = language === 'vi' 
    ? 'Write in Vietnamese' 
    : 'Write in English';
    
  const toneInstruction = {
    expert: 'Use professional, authoritative tone with data and insights',
    friendly: 'Use conversational, approachable tone',
    humorous: 'Use engaging, witty tone with personality'
  }[tone];
  
  const formatInstruction = {
    'toplist': 'Create a top 10 list format with clear rankings and explanations',
    'pov': 'Write an opinion piece from a unique perspective',
    'case-study': 'Present a detailed case study with data, challenges, and solutions',
    'how-to': 'Create a step-by-step tutorial or guide'
  }[format];
  
  return `
${languageInstruction}. ${toneInstruction}.

Topic/Keyword: ${keyword}

Recent Research Data:
${research.join('\n\n')}

Format: ${formatInstruction}

Requirements:
- Use the research data to make the content timely and data-backed
- Include specific examples and statistics
- Make it engaging and actionable
- Optimize for SEO and readability
- Length: 1500-2000 words

Generate the content now:
  `.trim();
}
```

### 3. Video Generation with Remotion

Automatically render videos from generated content:

```typescript
// lib/video/render-video.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  content: string;
  title: string;
  platform: 'reels' | 'tiktok' | 'shorts';
}

const platformSpecs = {
  reels: { width: 1080, height: 1920, fps: 30 },
  tiktok: { width: 1080, height: 1920, fps: 30 },
  shorts: { width: 1080, height: 1920, fps: 30 }
};

export async function renderContentVideo(
  config: VideoConfig
): Promise<string> {
  const { content, title, platform } = config;
  const specs = platformSpecs[platform];
  
  // Bundle Remotion project
  const bundleLocation = await bundle(
    path.join(process.cwd(), 'remotion/index.ts')
  );
  
  // Get composition
  const compositionId = 'ContentVideo';
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: {
      content: splitContentForSlides(content),
      title
    }
  });
  
  // Output path
  const outputLocation = path.join(
    process.cwd(),
    'public',
    'videos',
    `${Date.now()}-${platform}.mp4`
  );
  
  // Render video
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: {
      content: splitContentForSlides(content),
      title
    },
    ...specs
  });
  
  return outputLocation;
}

function splitContentForSlides(content: string): string[] {
  // Split content into slides (max 150 chars per slide)
  const sentences = content.match(/[^.!?]+[.!?]+/g) || [];
  const slides: string[] = [];
  let currentSlide = '';
  
  for (const sentence of sentences) {
    if ((currentSlide + sentence).length > 150 && currentSlide) {
      slides.push(currentSlide.trim());
      currentSlide = sentence;
    } else {
      currentSlide += ' ' + sentence;
    }
  }
  
  if (currentSlide) {
    slides.push(currentSlide.trim());
  }
  
  return slides;
}
```

### 4. Remotion Video Component

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, interpolate, useCurrentFrame, useVideoConfig } from 'remotion';

interface ContentVideoProps {
  title: string;
  content: string[];
}

export const ContentVideo: React.FC<ContentVideoProps> = ({ title, content }) => {
  const frame = useCurrentFrame();
  const { fps, durationInFrames } = useVideoConfig();
  
  const framesPerSlide = Math.floor(durationInFrames / (content.length + 1));
  const currentSlideIndex = Math.min(
    Math.floor(frame / framesPerSlide),
    content.length
  );
  
  const slideProgress = interpolate(
    frame % framesPerSlide,
    [0, 30],
    [0, 1],
    { extrapolateRight: 'clamp' }
  );
  
  const currentText = currentSlideIndex === 0 
    ? title 
    : content[currentSlideIndex - 1];
  
  return (
    <AbsoluteFill style={{ backgroundColor: '#000' }}>
      <div
        style={{
          display: 'flex',
          alignItems: 'center',
          justifyContent: 'center',
          fontSize: 60,
          color: 'white',
          padding: 80,
          textAlign: 'center',
          opacity: slideProgress,
          transform: `translateY(${(1 - slideProgress) * 50}px)`
        }}
      >
        {currentText}
      </div>
    </AbsoluteFill>
  );
};
```

## Complete Workflow Example

```typescript
// app/api/generate-content/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { scrapeRecentNews } from '@/lib/scrapers/news-scraper';
import { generateContent } from '@/lib/ai/content-generator';
import { renderContentVideo } from '@/lib/video/render-video';

export async function POST(request: NextRequest) {
  try {
    const { keyword, format, language, tone, platform } = await request.json();
    
    // Step 1: Research
    console.log('🔍 Scraping research data...');
    const articles = await scrapeRecentNews(keyword);
    const research = articles.map(a => `${a.title}\n${a.content}`);
    
    // Step 2: Generate Content
    console.log('✍️ Generating content with AI...');
    const content = await generateContent({
      keyword,
      research,
      format,
      language,
      tone,
      provider: 'claude'
    });
    
    // Step 3: Render Video
    console.log('🎬 Rendering video...');
    const videoPath = await renderContentVideo({
      content,
      title: keyword,
      platform
    });
    
    return NextResponse.json({
      success: true,
      content,
      videoUrl: videoPath.replace(process.cwd() + '/public', '')
    });
    
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { error: 'Failed to generate content' },
      { status: 500 }
    );
  }
}
```

## Configuration Patterns

### Custom Scraping Sources

```typescript
// config/scraping-sources.ts
export const scrapingSources = {
  techcrunch: {
    url: 'https://techcrunch.com/feed/',
    type: 'rss'
  },
  a16z: {
    url: 'https://a16z.com/feed/',
    type: 'rss'
  },
  twitter: {
    apiEndpoint: 'https://api.twitter.com/2/tweets/search/recent',
    requiresAuth: true
  }
};
```

### Content Templates

```typescript
// config/content-templates.ts
export const contentTemplates = {
  toplist: {
    structure: ['intro', 'item-1-10', 'conclusion'],
    minLength: 1500
  },
  pov: {
    structure: ['hook', 'context', 'opinion', 'supporting-arguments', 'conclusion'],
    minLength: 1200
  },
  'case-study': {
    structure: ['overview', 'challenge', 'solution', 'results', 'learnings'],
    minLength: 2000
  }
};
```

## Troubleshooting

### API Rate Limits

If hitting rate limits:

```typescript
// lib/utils/rate-limiter.ts
export async function withRateLimit<T>(
  fn: () => Promise<T>,
  delayMs: number = 1000
): Promise<T> {
  await new Promise(resolve => setTimeout(resolve, delayMs));
  return fn();
}

// Usage
const content = await withRateLimit(
  () => generateContent(options),
  2000
);
```

### Video Rendering Memory Issues

For large videos, increase Node memory:

```json
// package.json
{
  "scripts": {
    "dev": "NODE_OPTIONS='--max-old-space-size=4096' next dev",
    "build": "NODE_OPTIONS='--max-old-space-size=4096' next build"
  }
}
```

### Scraping Failures

Implement fallback logic:

```typescript
async function scrapeWithFallback(keyword: string) {
  try {
    return await scrapeRecentNews(keyword);
  } catch (error) {
    console.warn('Primary scraper failed, using fallback');
    // Use cached data or alternative source
    return getCachedResearch(keyword);
  }
}
```

## Best Practices

1. **Always use environment variables** for API keys
2. **Cache scraped data** to reduce API calls (use Redis or local storage)
3. **Implement retry logic** for AI API calls
4. **Queue video rendering** for production (use Bull or similar)
5. **Monitor token usage** to control costs
6. **Version your prompts** for consistent output quality

This skill enables AI agents to help developers build complete marketing automation pipelines with research, AI generation, and video rendering capabilities.
