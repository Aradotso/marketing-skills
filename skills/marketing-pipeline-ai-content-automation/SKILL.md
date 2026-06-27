---
name: marketing-pipeline-ai-content-automation
description: Automated AI content pipeline for research, scripting, publishing and video generation using Claude, OpenAI, and Remotion
triggers:
  - how do I set up the AI content automation pipeline
  - automate content creation with AI research and video generation
  - use marketing pipeline for automated posting
  - generate content from research to video automatically
  - set up Claude and OpenAI for content automation
  - create automated marketing content workflow
  - configure remotion video rendering for content
  - build AI-powered content pipeline with TypeScript
---

# Marketing Pipeline AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the **Ultimate AI Content Pipeline** - a complete TypeScript-based system that automates content creation from research through video generation. The pipeline crawls fresh news sources, generates multi-format articles using Claude/OpenAI, and renders videos using Remotion.

## What This Project Does

The marketing-pipeline-share project provides:

- **Auto-Research**: Crawls TechCrunch, a16z, Twitter/X, LinkedIn for latest insights (24h data)
- **AI Content Generation**: Creates articles in multiple formats (Toplist, POV, Case Study, How-to) using Claude 3 or OpenAI
- **Multi-language Support**: Generates content in English and Vietnamese simultaneously
- **Video Rendering**: Automatically creates infographics and short videos from content using Remotion
- **Platform Optimization**: Exports videos in proper ratios for Reels, TikTok, Shorts
- **Next.js Interface**: Web UI for managing the entire pipeline

## Installation

### Prerequisites

```bash
# Node.js 18+ required
node --version

# Install dependencies
npm install
# or
yarn install
# or
pnpm install
```

### Environment Configuration

Create a `.env.local` file in the project root:

```bash
# AI Models
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Database (if using)
DATABASE_URL=your_database_connection_string

# Next.js
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

### Running the Application

```bash
# Development mode
npm run dev

# Production build
npm run build
npm start

# Remotion video rendering
npm run remotion
```

## Project Structure

```
marketing-pipeline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI service integrations
│   │   ├── research/    # Web scraping & research
│   │   ├── video/       # Remotion video generation
│   │   └── utils/       # Utility functions
│   └── types/           # TypeScript definitions
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Core API Usage

### 1. Research Module

Automatically fetch and analyze latest content:

```typescript
import { autoResearch } from '@/lib/research/crawler';
import { analyzeInsights } from '@/lib/research/analyzer';

async function gatherResearch(keyword: string) {
  // Crawl multiple sources
  const sources = await autoResearch({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeRange: '24h'
  });

  // Extract insights
  const insights = await analyzeInsights(sources);
  
  return {
    rawData: sources,
    processedInsights: insights,
    timestamp: new Date()
  };
}

// Usage
const research = await gatherResearch('AI automation');
console.log(research.processedInsights);
```

### 2. AI Content Generation

Generate content using Claude or OpenAI:

```typescript
import { generateContent } from '@/lib/ai/content-generator';

async function createArticle(research: any, format: string) {
  const content = await generateContent({
    provider: 'claude', // or 'openai'
    model: 'claude-3-sonnet-20240229',
    format: format, // 'toplist', 'pov', 'case-study', 'how-to'
    language: 'en', // or 'vi'
    tone: 'professional', // or 'friendly', 'humorous'
    research: research.processedInsights,
    wordCount: 1500
  });

  return content;
}

// Generate bilingual content
async function createBilingualContent(research: any) {
  const [englishVersion, vietnameseVersion] = await Promise.all([
    createArticle(research, 'toplist'),
    generateContent({
      provider: 'claude',
      format: 'toplist',
      language: 'vi',
      research: research.processedInsights
    })
  ]);

  return { englishVersion, vietnameseVersion };
}
```

### 3. Claude Integration

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

async function generateWithClaude(prompt: string, context: string) {
  const message = await anthropic.messages.create({
    model: 'claude-3-sonnet-20240229',
    max_tokens: 4096,
    temperature: 0.7,
    system: `You are an expert content creator. Generate high-quality, 
             SEO-optimized content based on the latest research data provided.`,
    messages: [
      {
        role: 'user',
        content: `Research Context:\n${context}\n\nTask:\n${prompt}`
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
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateWithOpenAI(prompt: string, research: any) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: 'You are a professional content marketing expert.'
      },
      {
        role: 'user',
        content: `Based on this research:\n${JSON.stringify(research)}\n\n${prompt}`
      }
    ],
    temperature: 0.8,
    max_tokens: 3000
  });

  return completion.choices[0].message.content;
}
```

### 5. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

async function generateContentVideo(content: any, platform: 'reels' | 'tiktok' | 'shorts') {
  // Platform-specific dimensions
  const dimensions = {
    reels: { width: 1080, height: 1920 },
    tiktok: { width: 1080, height: 1920 },
    shorts: { width: 1080, height: 1920 }
  };

  // Bundle Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.join(process.cwd(), 'remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  // Get composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title: content.title,
      keyPoints: content.keyPoints,
      images: content.images,
      platform
    },
  });

  // Render video
  const outputPath = path.join(process.cwd(), 'public/videos', `${content.id}.mp4`);
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    ...dimensions[platform]
  });

  return outputPath;
}

// Usage
const videoPath = await generateContentVideo(article, 'reels');
```

## Complete Pipeline Workflow

```typescript
import { autoResearch } from '@/lib/research/crawler';
import { generateContent } from '@/lib/ai/content-generator';
import { generateContentVideo } from '@/lib/video/renderer';
import { publishToPage } from '@/lib/publishing/auto-post';

async function runFullPipeline(keyword: string) {
  try {
    // Step 1: Research
    console.log('🔍 Starting research...');
    const research = await autoResearch({
      keyword,
      sources: ['techcrunch', 'a16z', 'twitter'],
      timeRange: '24h'
    });

    // Step 2: Generate Content (Bilingual)
    console.log('✍️ Generating content...');
    const [englishContent, vietnameseContent] = await Promise.all([
      generateContent({
        provider: 'claude',
        format: 'toplist',
        language: 'en',
        research: research.processedInsights
      }),
      generateContent({
        provider: 'claude',
        format: 'toplist',
        language: 'vi',
        research: research.processedInsights
      })
    ]);

    // Step 3: Generate Videos
    console.log('🎬 Rendering videos...');
    const [reelsVideo, tikTokVideo] = await Promise.all([
      generateContentVideo(englishContent, 'reels'),
      generateContentVideo(englishContent, 'tiktok')
    ]);

    // Step 4: Auto-publish
    console.log('📤 Publishing...');
    await publishToPage({
      content: englishContent,
      videos: [reelsVideo, tikTokVideo],
      schedule: new Date(Date.now() + 3600000) // 1 hour from now
    });

    return {
      status: 'success',
      content: { englishContent, vietnameseContent },
      videos: { reelsVideo, tikTokVideo }
    };

  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}

// Execute pipeline
const result = await runFullPipeline('AI content automation');
```

## Content Format Templates

### Toplist Format

```typescript
interface ToplistConfig {
  title: string;
  introduction: string;
  items: Array<{
    rank: number;
    title: string;
    description: string;
    keyPoints: string[];
    dataBackup?: string;
  }>;
  conclusion: string;
}

async function generateToplist(research: any): Promise<ToplistConfig> {
  const prompt = `Create a toplist article with:
  - Engaging title
  - 5-7 ranked items
  - Data-backed insights from research
  - Clear actionable takeaways`;

  const content = await generateWithClaude(prompt, JSON.stringify(research));
  return JSON.parse(content);
}
```

### POV (Point of View) Format

```typescript
async function generatePOV(research: any, angle: string) {
  return await generateContent({
    provider: 'openai',
    format: 'pov',
    customPrompt: `Write from the perspective of ${angle}. 
                   Use first-person narrative and strong opinions 
                   backed by the research data.`,
    research
  });
}
```

## Configuration Examples

### Custom Voice Settings

```typescript
const voiceProfiles = {
  expert: {
    tone: 'authoritative',
    vocabulary: 'technical',
    structure: 'formal'
  },
  friendly: {
    tone: 'conversational',
    vocabulary: 'accessible',
    structure: 'casual'
  },
  humorous: {
    tone: 'witty',
    vocabulary: 'creative',
    structure: 'entertaining'
  }
};

async function generateWithVoice(research: any, voice: keyof typeof voiceProfiles) {
  const profile = voiceProfiles[voice];
  
  return await generateContent({
    provider: 'claude',
    research,
    tone: profile.tone,
    customInstructions: `Use ${profile.vocabulary} vocabulary and ${profile.structure} structure.`
  });
}
```

### Multi-Source Research Config

```typescript
const researchConfig = {
  sources: {
    techcrunch: {
      url: 'https://techcrunch.com',
      selector: 'article',
      priority: 'high'
    },
    a16z: {
      url: 'https://a16z.com/blog',
      selector: '.post',
      priority: 'high'
    },
    twitter: {
      api: 'v2',
      hashtags: ['#AI', '#Marketing', '#Automation'],
      priority: 'medium'
    }
  },
  filters: {
    minEngagement: 100,
    timeRange: '24h',
    relevanceThreshold: 0.7
  }
};
```

## API Routes (Next.js)

### Research Endpoint

```typescript
// app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { autoResearch } from '@/lib/research/crawler';

export async function POST(request: NextRequest) {
  const { keyword, sources, timeRange } = await request.json();

  try {
    const research = await autoResearch({
      keyword,
      sources: sources || ['techcrunch', 'a16z'],
      timeRange: timeRange || '24h'
    });

    return NextResponse.json({
      success: true,
      data: research
    });
  } catch (error) {
    return NextResponse.json({
      success: false,
      error: error.message
    }, { status: 500 });
  }
}
```

### Content Generation Endpoint

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { generateContent } from '@/lib/ai/content-generator';

export async function POST(request: NextRequest) {
  const { provider, format, language, research } = await request.json();

  try {
    const content = await generateContent({
      provider,
      format,
      language,
      research
    });

    return NextResponse.json({
      success: true,
      content
    });
  } catch (error) {
    return NextResponse.json({
      success: false,
      error: error.message
    }, { status: 500 });
  }
}
```

### Video Render Endpoint

```typescript
// app/api/render-video/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { generateContentVideo } from '@/lib/video/renderer';

export async function POST(request: NextRequest) {
  const { content, platform } = await request.json();

  try {
    const videoPath = await generateContentVideo(content, platform);

    return NextResponse.json({
      success: true,
      videoUrl: `/videos/${path.basename(videoPath)}`
    });
  } catch (error) {
    return NextResponse.json({
      success: false,
      error: error.message
    }, { status: 500 });
  }
}
```

## Common Patterns

### Scheduled Content Pipeline

```typescript
import cron from 'node-cron';

// Run pipeline every day at 9 AM
cron.schedule('0 9 * * *', async () => {
  const keywords = ['AI trends', 'Marketing automation', 'Content strategy'];
  
  for (const keyword of keywords) {
    try {
      await runFullPipeline(keyword);
      console.log(`✅ Completed pipeline for: ${keyword}`);
    } catch (error) {
      console.error(`❌ Failed for ${keyword}:`, error);
    }
  }
});
```

### Batch Processing

```typescript
async function batchGenerateContent(keywords: string[]) {
  const results = [];

  for (const keyword of keywords) {
    const research = await autoResearch({ keyword, sources: ['all'], timeRange: '24h' });
    
    const content = await generateContent({
      provider: 'claude',
      format: 'toplist',
      language: 'en',
      research: research.processedInsights
    });

    results.push({ keyword, content });
    
    // Rate limiting
    await new Promise(resolve => setTimeout(resolve, 2000));
  }

  return results;
}
```

### Error Handling and Retry Logic

```typescript
async function generateWithRetry(config: any, maxRetries = 3) {
  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      return await generateContent(config);
    } catch (error) {
      if (attempt === maxRetries) throw error;
      
      console.log(`Retry ${attempt}/${maxRetries} after error:`, error.message);
      await new Promise(resolve => setTimeout(resolve, 1000 * attempt));
    }
  }
}
```

## Troubleshooting

### Common Issues

**API Rate Limits**
```typescript
// Implement rate limiting
import rateLimit from 'express-rate-limit';

const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100 // limit each IP to 100 requests per windowMs
});

// Use with API routes
export const config = {
  api: {
    bodyParser: true,
  },
};
```

**Memory Issues with Video Rendering**
```bash
# Increase Node.js memory limit
NODE_OPTIONS=--max-old-space-size=4096 npm run remotion
```

**Missing Environment Variables**
```typescript
// Validate env vars at startup
function validateEnv() {
  const required = ['OPENAI_API_KEY', 'ANTHROPIC_API_KEY', 'RAPIDAPI_KEY'];
  
  for (const key of required) {
    if (!process.env[key]) {
      throw new Error(`Missing required environment variable: ${key}`);
    }
  }
}

validateEnv();
```

**Research Crawler Blocked**
```typescript
// Use rotating proxies or headers
const crawlerConfig = {
  headers: {
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36',
    'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9',
  },
  timeout: 30000,
  retries: 3
};
```

This skill provides comprehensive coverage of the marketing pipeline automation project, enabling AI agents to effectively assist developers in implementing and customizing the system.
