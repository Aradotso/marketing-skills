---
name: marketing-pipeline-ai-content-automation
description: Automated AI content pipeline for research, scriptwriting, video generation, and social media publishing using Claude, OpenAI, and Remotion
triggers:
  - automate content creation with AI research and video generation
  - build automated marketing content pipeline with Claude and OpenAI
  - generate videos from articles using Remotion automation
  - create multi-format content from trending news automatically
  - set up AI content workflow from research to publication
  - crawl trending topics and generate social media content
  - automate blog posts and video creation for marketing
  - build end-to-end content automation system with TypeScript
---

# Marketing Pipeline AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the Ultimate AI Content Pipeline, a TypeScript-based system that automates the entire content creation workflow from research and scriptwriting to video generation and social media publishing. It leverages Claude 3, OpenAI, and Remotion to create a fully automated content factory.

## What This Project Does

The Marketing Pipeline is an all-in-one content automation system that:

- **Auto-crawls trending content** from TechCrunch, a16z, Twitter/X, LinkedIn within the last 24 hours
- **Generates multi-format articles** (Top Lists, POV, Case Studies, How-Tos) in English and Vietnamese
- **Renders videos and infographics** automatically using Remotion
- **Publishes to social platforms** with optimized formatting for Reels, TikTok, Shorts
- **Provides API endpoints** for integration into existing marketing workflows

## Installation & Setup

### Prerequisites

```bash
# Node.js 18+ required
node --version

# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
yarn install
```

### Environment Configuration

Create a `.env.local` file in the project root:

```bash
# AI Services
ANTHROPIC_API_KEY=your_claude_api_key_here
OPENAI_API_KEY=your_openai_api_key_here

# Content Research APIs
RAPIDAPI_KEY=your_rapidapi_key_here
TWITTER_BEARER_TOKEN=your_twitter_bearer_token

# Database (if using)
DATABASE_URL=postgresql://user:password@localhost:5432/marketing_pipeline

# Remotion Video Rendering
REMOTION_AWS_ACCESS_KEY_ID=your_aws_access_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret_key
REMOTION_S3_BUCKET=your-video-bucket

# Application
NEXT_PUBLIC_API_URL=http://localhost:3000
NODE_ENV=development
```

### Start Development Server

```bash
npm run dev
# or
yarn dev
```

The application will be available at `http://localhost:3000`

## Core Components & Architecture

### 1. Research Module (Content Crawling)

The research module automatically scans trending content from multiple sources:

```typescript
// lib/research/crawler.ts
import { AnthropicClient } from '@/lib/ai/anthropic';

interface ResearchSource {
  name: string;
  url: string;
  selector: string;
}

export async function crawlTrendingTopics(keyword: string): Promise<ResearchData[]> {
  const sources: ResearchSource[] = [
    { name: 'TechCrunch', url: 'https://techcrunch.com/search/', selector: '.post-block' },
    { name: 'a16z', url: 'https://a16z.com/', selector: '.article' },
  ];

  const results: ResearchData[] = [];

  for (const source of sources) {
    try {
      const response = await fetch(`${source.url}${encodeURIComponent(keyword)}`);
      const html = await response.text();
      
      // Extract articles using cheerio or similar
      const articles = parseHTML(html, source.selector);
      results.push(...articles);
    } catch (error) {
      console.error(`Failed to crawl ${source.name}:`, error);
    }
  }

  return results;
}

export async function analyzeResearch(data: ResearchData[]): Promise<ContentInsights> {
  const anthropic = new AnthropicClient(process.env.ANTHROPIC_API_KEY!);
  
  const prompt = `Analyze these trending articles and extract key insights:
${data.map(d => `- ${d.title}: ${d.summary}`).join('\n')}

Provide:
1. Main trends and patterns
2. Key statistics and data points
3. Unique angles for content creation`;

  const insights = await anthropic.generateCompletion({
    model: 'claude-3-sonnet-20240229',
    max_tokens: 2000,
    messages: [{ role: 'user', content: prompt }]
  });

  return parseInsights(insights.content);
}
```

### 2. Content Generation Module

Generate multi-format content with AI:

```typescript
// lib/content/generator.ts
import { OpenAI } from 'openai';
import { Anthropic } from '@anthropic-ai/sdk';

interface ContentConfig {
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'professional' | 'friendly' | 'humorous';
  targetAudience: string;
}

export class ContentGenerator {
  private openai: OpenAI;
  private anthropic: Anthropic;

  constructor() {
    this.openai = new OpenAI({ apiKey: process.env.OPENAI_API_KEY });
    this.anthropic = new Anthropic({ apiKey: process.env.ANTHROPIC_API_KEY });
  }

  async generateArticle(
    keyword: string,
    insights: ContentInsights,
    config: ContentConfig
  ): Promise<ArticleContent> {
    const systemPrompt = this.buildSystemPrompt(config);
    const userPrompt = this.buildUserPrompt(keyword, insights, config);

    // Use Claude for Vietnamese, OpenAI for English (or customize)
    const aiService = config.language === 'vi' ? this.anthropic : this.openai;

    if (aiService === this.anthropic) {
      const response = await this.anthropic.messages.create({
        model: 'claude-3-sonnet-20240229',
        max_tokens: 4000,
        messages: [
          { role: 'user', content: `${systemPrompt}\n\n${userPrompt}` }
        ]
      });

      return this.parseArticleResponse(response.content[0].text);
    } else {
      const response = await this.openai.chat.completions.create({
        model: 'gpt-4-turbo-preview',
        messages: [
          { role: 'system', content: systemPrompt },
          { role: 'user', content: userPrompt }
        ],
        temperature: 0.7,
        max_tokens: 3000
      });

      return this.parseArticleResponse(response.choices[0].message.content!);
    }
  }

  private buildSystemPrompt(config: ContentConfig): string {
    const formatGuides = {
      'toplist': 'Create a numbered list article with rankings and explanations',
      'pov': 'Write from a specific perspective with strong opinions backed by data',
      'case-study': 'Present a detailed case study with problem, solution, and results',
      'how-to': 'Create a step-by-step tutorial with actionable instructions'
    };

    return `You are an expert content creator specializing in ${config.format} articles.
Tone: ${config.tone}
Target Audience: ${config.targetAudience}
Language: ${config.language}

${formatGuides[config.format]}

Include:
- Compelling headline
- Hook paragraph
- Data-backed insights
- Clear structure with subheadings
- Call-to-action`;
  }

  private buildUserPrompt(
    keyword: string,
    insights: ContentInsights,
    config: ContentConfig
  ): string {
    return `Write a ${config.format} article about: ${keyword}

Key Insights to Include:
${insights.trends.map((t, i) => `${i + 1}. ${t}`).join('\n')}

Statistics:
${insights.statistics.map(s => `- ${s}`).join('\n')}

Create engaging, SEO-optimized content that stands out.`;
  }

  private parseArticleResponse(text: string): ArticleContent {
    // Parse structured content from AI response
    const sections = text.split(/\n#{1,2}\s/);
    return {
      headline: sections[0].trim(),
      introduction: sections[1]?.trim() || '',
      body: sections.slice(2, -1).map(s => s.trim()),
      conclusion: sections[sections.length - 1]?.trim() || '',
      metadata: {
        wordCount: text.split(/\s+/).length,
        readingTime: Math.ceil(text.split(/\s+/).length / 200)
      }
    };
  }
}
```

### 3. Video Generation with Remotion

Automatically render videos from article content:

```typescript
// lib/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  composition: string;
  props: Record<string, any>;
  outputPath: string;
  format: 'reel' | 'tiktok' | 'youtube-short';
}

export class VideoRenderer {
  private compositions = {
    reel: { width: 1080, height: 1920, fps: 30 },
    tiktok: { width: 1080, height: 1920, fps: 30 },
    'youtube-short': { width: 1080, height: 1920, fps: 30 }
  };

  async renderFromArticle(
    article: ArticleContent,
    config: VideoConfig
  ): Promise<string> {
    // Bundle Remotion project
    const bundleLocation = await bundle({
      entryPoint: path.join(process.cwd(), 'remotion/index.ts'),
      webpackOverride: (currentConfiguration) => currentConfiguration
    });

    const composition = this.compositions[config.format];
    const compositionId = config.composition || 'ArticleVideo';

    // Prepare video props from article
    const videoProps = {
      title: article.headline,
      keyPoints: article.body.slice(0, 5),
      branding: {
        logo: process.env.NEXT_PUBLIC_LOGO_URL,
        colors: {
          primary: '#3B82F6',
          secondary: '#10B981'
        }
      },
      ...config.props
    };

    // Get composition details
    const comp = await selectComposition({
      serveUrl: bundleLocation,
      id: compositionId,
      inputProps: videoProps
    });

    // Render video
    const outputPath = config.outputPath || 
      path.join(process.cwd(), 'public', 'videos', `${Date.now()}.mp4`);

    await renderMedia({
      composition: comp,
      serveUrl: bundleLocation,
      codec: 'h264',
      outputLocation: outputPath,
      inputProps: videoProps,
      ...composition
    });

    return outputPath;
  }

  async renderInfographic(data: ContentInsights): Promise<string> {
    // Similar rendering for static infographics
    return this.renderFromArticle(
      { headline: 'Key Insights', body: data.trends } as ArticleContent,
      { composition: 'Infographic', props: data, outputPath: '', format: 'reel' }
    );
  }
}
```

### 4. API Routes (Next.js)

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { ContentGenerator } from '@/lib/content/generator';
import { VideoRenderer } from '@/lib/video/renderer';
import { crawlTrendingTopics, analyzeResearch } from '@/lib/research/crawler';

export async function POST(req: NextRequest) {
  try {
    const { keyword, format, language, generateVideo } = await req.json();

    // Step 1: Research
    const rawData = await crawlTrendingTopics(keyword);
    const insights = await analyzeResearch(rawData);

    // Step 2: Generate Content
    const generator = new ContentGenerator();
    const article = await generator.generateArticle(keyword, insights, {
      format,
      language,
      tone: 'professional',
      targetAudience: 'marketing professionals'
    });

    // Step 3: Optionally render video
    let videoUrl = null;
    if (generateVideo) {
      const renderer = new VideoRenderer();
      const videoPath = await renderer.renderFromArticle(article, {
        composition: 'ArticleVideo',
        props: {},
        outputPath: '',
        format: 'reel'
      });
      videoUrl = `/videos/${path.basename(videoPath)}`;
    }

    return NextResponse.json({
      success: true,
      data: {
        article,
        videoUrl,
        insights
      }
    });

  } catch (error) {
    console.error('Generation error:', error);
    return NextResponse.json(
      { success: false, error: 'Failed to generate content' },
      { status: 500 }
    );
  }
}
```

## Common Usage Patterns

### Full Pipeline Execution

```typescript
// scripts/generate-content.ts
import { ContentPipeline } from '@/lib/pipeline';

async function runContentPipeline() {
  const pipeline = new ContentPipeline({
    aiProvider: 'claude', // or 'openai'
    language: 'en',
    outputFormats: ['article', 'video', 'social-posts']
  });

  const result = await pipeline.execute({
    keyword: 'AI marketing automation',
    contentType: 'toplist',
    videoFormat: 'reel',
    publishTo: ['twitter', 'linkedin']
  });

  console.log('Generated:', result);
  // {
  //   article: { url: '/blog/ai-marketing-automation', id: '...' },
  //   video: { url: '/videos/123.mp4', duration: 45 },
  //   socialPosts: [{ platform: 'twitter', postId: '...' }]
  // }
}

runContentPipeline();
```

### Batch Content Generation

```typescript
// scripts/batch-generate.ts
const keywords = [
  'AI content marketing',
  'Social media automation',
  'Video marketing trends'
];

const results = await Promise.all(
  keywords.map(keyword => 
    pipeline.execute({
      keyword,
      contentType: 'how-to',
      videoFormat: 'tiktok'
    })
  )
);

console.log(`Generated ${results.length} content pieces`);
```

### Custom Video Compositions

```typescript
// remotion/compositions/ArticleVideo.tsx
import { AbsoluteFill, Sequence, useCurrentFrame } from 'remotion';

export const ArticleVideo: React.FC<{
  title: string;
  keyPoints: string[];
  branding: any;
}> = ({ title, keyPoints, branding }) => {
  const frame = useCurrentFrame();

  return (
    <AbsoluteFill style={{ backgroundColor: branding.colors.primary }}>
      <Sequence from={0} durationInFrames={90}>
        <TitleSlide title={title} />
      </Sequence>
      
      {keyPoints.map((point, i) => (
        <Sequence
          key={i}
          from={90 + i * 60}
          durationInFrames={60}
        >
          <KeyPointSlide text={point} index={i + 1} />
        </Sequence>
      ))}
      
      <Sequence from={90 + keyPoints.length * 60} durationInFrames={60}>
        <CallToAction logo={branding.logo} />
      </Sequence>
    </AbsoluteFill>
  );
};
```

## CLI Commands

```bash
# Generate single content piece
npm run generate -- --keyword "AI tools" --format toplist --lang en

# Batch generation from CSV
npm run batch -- --input keywords.csv --output ./generated

# Render video from existing article
npm run render-video -- --article-id 123 --format reel

# Research only (no generation)
npm run research -- --keyword "marketing trends" --output research.json

# Preview Remotion compositions
npm run remotion:preview

# Deploy to production
npm run build
npm run start
```

## Configuration Files

### Content Templates

```typescript
// config/templates.ts
export const contentTemplates = {
  toplist: {
    structure: ['intro', 'items', 'conclusion'],
    minItems: 5,
    maxItems: 10,
    includeRankings: true
  },
  'case-study': {
    structure: ['background', 'challenge', 'solution', 'results'],
    includeMetrics: true,
    includeTestimonials: true
  },
  'how-to': {
    structure: ['intro', 'prerequisites', 'steps', 'conclusion'],
    minSteps: 3,
    includeVisuals: true
  }
};
```

### Video Presets

```typescript
// config/video-presets.ts
export const videoPresets = {
  reel: {
    width: 1080,
    height: 1920,
    fps: 30,
    durationInFrames: 900, // 30 seconds
    audioEnabled: true
  },
  tiktok: {
    width: 1080,
    height: 1920,
    fps: 30,
    durationInFrames: 1800, // 60 seconds
    audioEnabled: true,
    watermark: true
  }
};
```

## Troubleshooting

### API Rate Limits

```typescript
// lib/utils/rate-limiter.ts
export class RateLimiter {
  private queue: Array<() => Promise<any>> = [];
  private running = 0;
  
  constructor(private maxConcurrent: number, private delayMs: number) {}

  async add<T>(fn: () => Promise<T>): Promise<T> {
    while (this.running >= this.maxConcurrent) {
      await new Promise(resolve => setTimeout(resolve, this.delayMs));
    }
    
    this.running++;
    try {
      return await fn();
    } finally {
      this.running--;
    }
  }
}

// Usage
const limiter = new RateLimiter(3, 1000);
const result = await limiter.add(() => anthropic.messages.create({...}));
```

### Video Rendering Failures

```typescript
// Check Remotion logs
console.log('Remotion bundle location:', bundleLocation);

// Increase timeout for long videos
await renderMedia({
  ...config,
  timeoutInMilliseconds: 120000, // 2 minutes
  onProgress: ({ progress }) => {
    console.log(`Rendering: ${Math.round(progress * 100)}%`);
  }
});
```

### Memory Issues with Large Batches

```typescript
// Process in chunks
async function processBatch(keywords: string[], chunkSize = 5) {
  for (let i = 0; i < keywords.length; i += chunkSize) {
    const chunk = keywords.slice(i, i + chunkSize);
    await Promise.all(chunk.map(k => generateContent(k)));
    
    // Clear memory between chunks
    if (global.gc) global.gc();
    await new Promise(resolve => setTimeout(resolve, 2000));
  }
}
```

### Database Connection Issues

```typescript
// lib/db/connection.ts
import { PrismaClient } from '@prisma/client';

const globalForPrisma = global as unknown as { prisma: PrismaClient };

export const prisma = globalForPrisma.prisma || new PrismaClient({
  log: process.env.NODE_ENV === 'development' ? ['query', 'error'] : ['error'],
});

if (process.env.NODE_ENV !== 'production') globalForPrisma.prisma = prisma;
```

## Advanced Integration Examples

### Webhook-Based Automation

```typescript
// app/api/webhooks/trigger/route.ts
export async function POST(req: NextRequest) {
  const { trigger, keyword } = await req.json();
  
  // Trigger based on external events (e.g., trending topic detected)
  if (trigger === 'trending_topic') {
    await pipeline.execute({ keyword, contentType: 'pov' });
  }
  
  return NextResponse.json({ queued: true });
}
```

### Social Media Publishing

```typescript
// lib/publishing/social.ts
export async function publishToSocial(
  content: ArticleContent,
  platforms: string[]
) {
  const results = await Promise.all(
    platforms.map(async (platform) => {
      switch (platform) {
        case 'twitter':
          return await publishToTwitter(content);
        case 'linkedin':
          return await publishToLinkedIn(content);
        default:
          throw new Error(`Unsupported platform: ${platform}`);
      }
    })
  );
  
  return results;
}
```

This skill enables comprehensive automation of marketing content creation from research through video generation and publishing.
