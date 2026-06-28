---
name: ai-content-pipeline-automation
description: Automate content creation from research to video generation using AI (Claude/OpenAI) and Remotion for Vietnamese/English marketing content
triggers:
  - how do I automate content research and video generation
  - set up AI content pipeline with Claude and Remotion
  - generate marketing content automatically from keywords
  - create videos from blog posts using AI
  - build automated content workflow with TypeScript
  - scrape news and generate social media content
  - use Remotion to render marketing videos
  - configure AI content automation pipeline
---

# AI Content Pipeline Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with **Ultimate AI Content Pipeline**, a TypeScript-based automation system that transforms keywords into complete content packages: research, scripts, blog posts, and rendered videos. The pipeline leverages Claude 3, OpenAI, web scraping, and Remotion for end-to-end content creation.

## What This Project Does

The AI Content Pipeline automates:
- **Research**: Crawls recent news from TechCrunch, a16z, Twitter, LinkedIn within 24h
- **Content Generation**: Creates blog posts in multiple formats (toplist, POV, case study, how-to) in Vietnamese and English
- **Video Rendering**: Automatically generates infographics and short-form videos using Remotion
- **Multi-platform Export**: Optimized output for Reels, TikTok, Shorts

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

```env
# AI Services
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Database (if using)
DATABASE_URL=your_database_url

# Remotion (for video rendering)
REMOTION_AWS_ACCESS_KEY_ID=your_aws_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret
```

### Run Development Server

```bash
npm run dev
# or
yarn dev
# or
pnpm dev
```

Access the application at `http://localhost:3000`

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── scraper/     # Web scraping modules
│   │   └── video/       # Remotion video generation
│   └── types/           # TypeScript type definitions
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Core API Usage

### 1. Research & Content Scraping

```typescript
import { scrapeNewsArticles } from '@/lib/scraper/news-crawler';

interface NewsArticle {
  title: string;
  url: string;
  source: string;
  publishedAt: Date;
  content: string;
  keywords: string[];
}

async function gatherResearch(keyword: string): Promise<NewsArticle[]> {
  const sources = [
    'techcrunch',
    'a16z',
    'twitter',
    'linkedin'
  ];
  
  const articles = await scrapeNewsArticles({
    keyword,
    sources,
    timeRange: '24h',
    limit: 20
  });
  
  return articles;
}

// Usage
const research = await gatherResearch('AI marketing automation');
console.log(`Found ${research.length} relevant articles`);
```

### 2. AI Content Generation with Claude

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

interface ContentGenerationParams {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'vi' | 'en';
  tone: 'professional' | 'friendly' | 'humorous';
  research: NewsArticle[];
}

async function generateContent(params: ContentGenerationParams): Promise<string> {
  const { keyword, format, language, tone, research } = params;
  
  const researchContext = research
    .map(article => `- ${article.title} (${article.source}): ${article.content.slice(0, 200)}...`)
    .join('\n');
  
  const prompt = `You are an expert content creator for ${language === 'vi' ? 'Vietnamese' : 'English'} marketing content.

Topic: ${keyword}
Format: ${format}
Tone: ${tone}

Recent Research:
${researchContext}

Create a comprehensive ${format} article about ${keyword}. Include:
- Compelling headline
- Introduction with hook
- Main content with data-backed insights from the research
- Actionable takeaways
- Conclusion with CTA

${language === 'vi' ? 'Write in Vietnamese.' : 'Write in English.'}`;

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [
      { role: 'user', content: prompt }
    ],
  });
  
  return message.content[0].type === 'text' ? message.content[0].text : '';
}

// Usage
const content = await generateContent({
  keyword: 'AI Content Marketing',
  format: 'how-to',
  language: 'en',
  tone: 'professional',
  research: research
});
```

### 3. OpenAI Alternative

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateWithOpenAI(params: ContentGenerationParams): Promise<string> {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are an expert marketing content creator specializing in ${params.language} content.`
      },
      {
        role: 'user',
        content: `Create a ${params.format} article about ${params.keyword} in ${params.tone} tone.`
      }
    ],
    temperature: 0.7,
    max_tokens: 3000,
  });
  
  return completion.choices[0].message.content || '';
}
```

### 4. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import { VideoConfig } from '@/types/video';

interface VideoGenerationParams {
  content: string;
  title: string;
  platform: 'reels' | 'tiktok' | 'youtube-shorts';
  style: 'minimal' | 'dynamic' | 'professional';
}

async function generateVideo(params: VideoGenerationParams): Promise<string> {
  const { content, title, platform, style } = params;
  
  // Platform-specific dimensions
  const dimensions = {
    'reels': { width: 1080, height: 1920 },
    'tiktok': { width: 1080, height: 1920 },
    'youtube-shorts': { width: 1080, height: 1920 }
  };
  
  const { width, height } = dimensions[platform];
  
  // Bundle Remotion composition
  const bundleLocation = await bundle({
    entryPoint: './remotion/index.ts',
    webpackOverride: (config) => config,
  });
  
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title,
      content: extractKeyPoints(content),
      style,
    },
  });
  
  const outputLocation = `./output/${Date.now()}-${platform}.mp4`;
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: {
      title,
      content: extractKeyPoints(content),
      style,
    },
  });
  
  return outputLocation;
}

function extractKeyPoints(content: string): string[] {
  // Extract bullet points or key sentences
  const lines = content.split('\n');
  return lines
    .filter(line => line.trim().startsWith('-') || line.trim().startsWith('•'))
    .map(line => line.replace(/^[-•]\s*/, '').trim())
    .slice(0, 5);
}

// Usage
const videoPath = await generateVideo({
  content: generatedContent,
  title: 'AI Marketing Automation 101',
  platform: 'reels',
  style: 'professional'
});
```

### 5. Complete Pipeline Example

```typescript
import { scrapeNewsArticles } from '@/lib/scraper/news-crawler';
import { generateContent } from '@/lib/ai/content-generator';
import { generateVideo } from '@/lib/video/remotion-renderer';

interface PipelineResult {
  research: NewsArticle[];
  englishContent: string;
  vietnameseContent: string;
  videos: {
    reels: string;
    tiktok: string;
    shorts: string;
  };
}

async function runContentPipeline(keyword: string): Promise<PipelineResult> {
  console.log(`🚀 Starting content pipeline for: ${keyword}`);
  
  // Step 1: Research
  console.log('📡 Gathering research...');
  const research = await scrapeNewsArticles({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter'],
    timeRange: '24h',
    limit: 15
  });
  
  // Step 2: Generate English Content
  console.log('🧠 Generating English content...');
  const englishContent = await generateContent({
    keyword,
    format: 'how-to',
    language: 'en',
    tone: 'professional',
    research
  });
  
  // Step 3: Generate Vietnamese Content
  console.log('🧠 Generating Vietnamese content...');
  const vietnameseContent = await generateContent({
    keyword,
    format: 'how-to',
    language: 'vi',
    tone: 'friendly',
    research
  });
  
  // Step 4: Generate Videos for Multiple Platforms
  console.log('🎬 Rendering videos...');
  const [reelsVideo, tiktokVideo, shortsVideo] = await Promise.all([
    generateVideo({
      content: englishContent,
      title: keyword,
      platform: 'reels',
      style: 'dynamic'
    }),
    generateVideo({
      content: englishContent,
      title: keyword,
      platform: 'tiktok',
      style: 'dynamic'
    }),
    generateVideo({
      content: englishContent,
      title: keyword,
      platform: 'youtube-shorts',
      style: 'professional'
    })
  ]);
  
  console.log('✅ Pipeline complete!');
  
  return {
    research,
    englishContent,
    vietnameseContent,
    videos: {
      reels: reelsVideo,
      tiktok: tiktokVideo,
      shorts: shortsVideo
    }
  };
}

// Execute pipeline
const result = await runContentPipeline('AI Marketing Automation 2026');
console.log(`Generated ${result.videos.reels}`);
```

## Remotion Video Template Example

Create a video composition in `remotion/ContentVideo.tsx`:

```typescript
import { AbsoluteFill, Sequence, useCurrentFrame, useVideoConfig } from 'remotion';
import { interpolate } from 'remotion';

interface ContentVideoProps {
  title: string;
  content: string[];
  style: 'minimal' | 'dynamic' | 'professional';
}

export const ContentVideo: React.FC<ContentVideoProps> = ({ title, content, style }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  
  const titleOpacity = interpolate(frame, [0, 30], [0, 1], {
    extrapolateRight: 'clamp',
  });
  
  return (
    <AbsoluteFill style={{ backgroundColor: '#000' }}>
      {/* Title Sequence */}
      <Sequence from={0} durationInFrames={fps * 3}>
        <AbsoluteFill style={{
          justifyContent: 'center',
          alignItems: 'center',
          opacity: titleOpacity,
        }}>
          <h1 style={{
            fontSize: 80,
            color: '#fff',
            textAlign: 'center',
            fontWeight: 'bold',
            padding: '0 40px',
          }}>
            {title}
          </h1>
        </AbsoluteFill>
      </Sequence>
      
      {/* Content Points */}
      {content.map((point, index) => (
        <Sequence
          key={index}
          from={fps * (3 + index * 2)}
          durationInFrames={fps * 2}
        >
          <ContentPoint text={point} index={index} />
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};

const ContentPoint: React.FC<{ text: string; index: number }> = ({ text, index }) => {
  const frame = useCurrentFrame();
  const opacity = interpolate(frame, [0, 15], [0, 1]);
  const translateY = interpolate(frame, [0, 15], [50, 0]);
  
  return (
    <AbsoluteFill style={{
      justifyContent: 'center',
      alignItems: 'center',
      opacity,
    }}>
      <div style={{
        transform: `translateY(${translateY}px)`,
        padding: '40px',
        maxWidth: '80%',
      }}>
        <p style={{
          fontSize: 50,
          color: '#fff',
          textAlign: 'center',
          lineHeight: 1.5,
        }}>
          {text}
        </p>
      </div>
    </AbsoluteFill>
  );
};
```

## Configuration Patterns

### Custom AI Provider Configuration

```typescript
// lib/ai/config.ts
export const aiConfig = {
  providers: {
    claude: {
      model: 'claude-3-5-sonnet-20241022',
      maxTokens: 4096,
      temperature: 0.7,
    },
    openai: {
      model: 'gpt-4-turbo-preview',
      maxTokens: 3000,
      temperature: 0.7,
    },
  },
  contentFormats: {
    'toplist': 'Create a numbered list of top items',
    'pov': 'Write from a specific point of view',
    'case-study': 'Analyze a real-world example',
    'how-to': 'Step-by-step tutorial format',
  },
  languages: ['vi', 'en'],
  tones: ['professional', 'friendly', 'humorous'],
};
```

### Scraper Configuration

```typescript
// lib/scraper/config.ts
export const scraperConfig = {
  sources: {
    techcrunch: {
      baseUrl: 'https://techcrunch.com',
      rateLimit: 1000, // ms between requests
    },
    a16z: {
      baseUrl: 'https://a16z.com',
      rateLimit: 1500,
    },
  },
  headers: {
    'User-Agent': 'Mozilla/5.0 (compatible; ContentBot/1.0)',
  },
  timeout: 10000,
};
```

## Troubleshooting

### API Rate Limits

```typescript
// lib/utils/rate-limiter.ts
class RateLimiter {
  private queue: Array<() => Promise<any>> = [];
  private processing = false;
  
  async add<T>(fn: () => Promise<T>, delay: number = 1000): Promise<T> {
    return new Promise((resolve, reject) => {
      this.queue.push(async () => {
        try {
          const result = await fn();
          resolve(result);
        } catch (error) {
          reject(error);
        }
      });
      
      if (!this.processing) {
        this.process(delay);
      }
    });
  }
  
  private async process(delay: number) {
    this.processing = true;
    
    while (this.queue.length > 0) {
      const fn = this.queue.shift();
      if (fn) await fn();
      await new Promise(resolve => setTimeout(resolve, delay));
    }
    
    this.processing = false;
  }
}

export const rateLimiter = new RateLimiter();
```

### Video Rendering Errors

```typescript
// Check Remotion installation
try {
  const { getCompositions } = require('@remotion/renderer');
  console.log('✅ Remotion installed correctly');
} catch (error) {
  console.error('❌ Remotion not found. Run: npm install @remotion/renderer @remotion/bundler');
}

// Handle rendering failures
async function safeRenderVideo(params: VideoGenerationParams): Promise<string | null> {
  try {
    return await generateVideo(params);
  } catch (error) {
    console.error('Video rendering failed:', error);
    // Fallback: generate static image instead
    return null;
  }
}
```

### Memory Issues with Large Content

```typescript
// Process content in chunks
function chunkArray<T>(array: T[], size: number): T[][] {
  const chunks: T[][] = [];
  for (let i = 0; i < array.length; i += size) {
    chunks.push(array.slice(i, i + size));
  }
  return chunks;
}

async function processLargeDataset(articles: NewsArticle[]) {
  const chunks = chunkArray(articles, 5);
  const results = [];
  
  for (const chunk of chunks) {
    const chunkResults = await Promise.all(
      chunk.map(article => processArticle(article))
    );
    results.push(...chunkResults);
  }
  
  return results;
}
```

This skill provides AI coding agents with comprehensive knowledge to help developers implement automated content marketing pipelines using this TypeScript-based system.
