---
name: ultimate-ai-content-pipeline
description: Automated content pipeline system that researches, writes scripts, and generates videos using Claude/OpenAI and Remotion
triggers:
  - set up the AI content automation pipeline
  - create automated content with research and video generation
  - build a content pipeline with Claude and Remotion
  - automate content creation from research to video
  - configure the marketing content automation system
  - generate automated blog posts and videos with AI
  - implement the Vietnamese/English content pipeline
  - use the AI research and video rendering system
---

# Ultimate AI Content Pipeline Skill

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the Ultimate AI Content Pipeline, an all-in-one automated content creation system that handles research, scriptwriting, multilingual content generation, and video rendering using Claude/OpenAI and Remotion.

## What This Project Does

Ultimate AI Content Pipeline is a TypeScript/Next.js application that automates the entire content creation workflow:

1. **Auto-Research**: Crawls fresh data from sources like TechCrunch, a16z, Twitter/X, LinkedIn
2. **AI Content Generation**: Creates content in multiple formats (toplist, POV, case study, how-to) using Claude or OpenAI
3. **Multilingual Output**: Generates both English and Vietnamese versions with customizable tone
4. **Video Generation**: Automatically renders videos and infographics using Remotion for social media platforms

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

Create a `.env.local` file with the following required variables:

```bash
# AI Provider Keys
ANTHROPIC_API_KEY=your_anthropic_key_here
OPENAI_API_KEY=your_openai_key_here

# Research & Data APIs
RAPIDAPI_KEY=your_rapidapi_key_here

# Application Settings
NEXT_PUBLIC_APP_URL=http://localhost:3000
NODE_ENV=development

# Optional: Video Rendering
REMOTION_LICENSE_KEY=your_remotion_license_key_here
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/             # Core libraries
│   │   ├── ai/          # AI provider integrations
│   │   ├── research/    # Research crawlers
│   │   └── video/       # Video generation
│   ├── types/           # TypeScript types
│   └── utils/           # Helper functions
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Key API Usage Patterns

### 1. Research & Data Crawling

```typescript
import { ResearchService } from '@/lib/research/research-service';

// Initialize research service
const researchService = new ResearchService({
  rapidApiKey: process.env.RAPIDAPI_KEY,
  sources: ['techcrunch', 'a16z', 'twitter', 'linkedin']
});

// Crawl fresh content on a topic
async function gatherResearch(keyword: string) {
  const results = await researchService.crawl({
    keyword,
    timeRange: '24h',
    maxResults: 20,
    language: 'en'
  });
  
  return results;
}

// Extract insights from research
async function extractInsights(researchData: any[]) {
  const insights = await researchService.analyzeData({
    data: researchData,
    extractStats: true,
    identifyTrends: true
  });
  
  return insights;
}
```

### 2. AI Content Generation

```typescript
import { ContentGenerator } from '@/lib/ai/content-generator';

// Using Claude (Anthropic)
const claudeGenerator = new ContentGenerator({
  provider: 'anthropic',
  apiKey: process.env.ANTHROPIC_API_KEY,
  model: 'claude-3-opus-20240229'
});

// Using OpenAI
const openaiGenerator = new ContentGenerator({
  provider: 'openai',
  apiKey: process.env.OPENAI_API_KEY,
  model: 'gpt-4-turbo-preview'
});

// Generate content with research
async function generateContent(params: {
  keyword: string;
  research: any[];
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  tone: 'expert' | 'friendly' | 'humorous';
  language: 'en' | 'vi';
}) {
  const content = await claudeGenerator.generate({
    prompt: `Write a ${params.format} article about ${params.keyword}`,
    context: params.research,
    format: params.format,
    tone: params.tone,
    language: params.language,
    includeStats: true,
    wordCount: 1500
  });
  
  return content;
}

// Generate bilingual content
async function generateBilingual(keyword: string, research: any[]) {
  const [englishContent, vietnameseContent] = await Promise.all([
    generateContent({ keyword, research, format: 'toplist', tone: 'expert', language: 'en' }),
    generateContent({ keyword, research, format: 'toplist', tone: 'expert', language: 'vi' })
  ]);
  
  return { english: englishContent, vietnamese: vietnameseContent };
}
```

### 3. Video Generation with Remotion

```typescript
import { renderVideo } from '@/lib/video/renderer';
import { VideoComposition } from '@/remotion/compositions';

// Render video from content
async function createContentVideo(content: {
  title: string;
  points: string[];
  stats: Array<{ label: string; value: string }>;
}) {
  const videoConfig = {
    composition: 'ContentVideo',
    props: {
      title: content.title,
      points: content.points,
      stats: content.stats,
      duration: 60, // seconds
      fps: 30
    },
    outputFormat: 'mp4',
    aspectRatio: '9:16' // For Reels/TikTok/Shorts
  };
  
  const videoPath = await renderVideo(videoConfig);
  return videoPath;
}

// Create multiple aspect ratios
async function renderMultiFormat(content: any) {
  const formats = [
    { ratio: '9:16', platform: 'reels' },
    { ratio: '16:9', platform: 'youtube' },
    { ratio: '1:1', platform: 'instagram' }
  ];
  
  const videos = await Promise.all(
    formats.map(format => 
      renderVideo({
        composition: 'ContentVideo',
        props: { ...content, aspectRatio: format.ratio },
        outputFormat: 'mp4'
      })
    )
  );
  
  return videos;
}
```

### 4. Complete Content Pipeline

```typescript
import { ContentPipeline } from '@/lib/pipeline';

// Full automation workflow
async function runContentPipeline(keyword: string) {
  const pipeline = new ContentPipeline({
    anthropicKey: process.env.ANTHROPIC_API_KEY,
    rapidApiKey: process.env.RAPIDAPI_KEY
  });
  
  // Step 1: Research
  console.log('🔍 Researching topic...');
  const research = await pipeline.research(keyword);
  
  // Step 2: Generate content (bilingual)
  console.log('✍️ Generating content...');
  const content = await pipeline.generateContent({
    keyword,
    research,
    formats: ['toplist', 'how-to'],
    languages: ['en', 'vi'],
    tone: 'expert'
  });
  
  // Step 3: Create videos
  console.log('🎬 Rendering videos...');
  const videos = await pipeline.generateVideos({
    content: content.english,
    platforms: ['reels', 'tiktok', 'youtube-shorts']
  });
  
  // Step 4: Return complete package
  return {
    research,
    content,
    videos,
    metadata: {
      keyword,
      createdAt: new Date(),
      stats: research.stats
    }
  };
}
```

## Next.js API Routes

### Content Generation Endpoint

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { ContentPipeline } from '@/lib/pipeline';

export async function POST(req: NextRequest) {
  try {
    const { keyword, format, language, tone } = await req.json();
    
    const pipeline = new ContentPipeline({
      anthropicKey: process.env.ANTHROPIC_API_KEY,
      rapidApiKey: process.env.RAPIDAPI_KEY
    });
    
    const result = await pipeline.generateContent({
      keyword,
      formats: [format],
      languages: [language],
      tone
    });
    
    return NextResponse.json({ success: true, data: result });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

### Video Rendering Endpoint

```typescript
// src/app/api/render-video/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { renderVideo } from '@/lib/video/renderer';

export async function POST(req: NextRequest) {
  try {
    const { content, platform } = await req.json();
    
    const aspectRatios = {
      'reels': '9:16',
      'youtube': '16:9',
      'tiktok': '9:16',
      'instagram': '1:1'
    };
    
    const videoPath = await renderVideo({
      composition: 'ContentVideo',
      props: {
        ...content,
        aspectRatio: aspectRatios[platform]
      },
      outputFormat: 'mp4'
    });
    
    return NextResponse.json({ 
      success: true, 
      videoUrl: videoPath 
    });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

## TypeScript Type Definitions

```typescript
// src/types/content.ts
export interface ResearchData {
  source: string;
  title: string;
  url: string;
  publishedAt: Date;
  content: string;
  stats?: Record<string, any>;
}

export interface ContentConfig {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  tone: 'expert' | 'friendly' | 'humorous';
  language: 'en' | 'vi';
  wordCount?: number;
  includeStats?: boolean;
}

export interface GeneratedContent {
  title: string;
  content: string;
  language: string;
  format: string;
  metadata: {
    wordCount: number;
    keywords: string[];
    readTime: number;
  };
  sections: Array<{
    heading: string;
    content: string;
  }>;
}

export interface VideoConfig {
  composition: string;
  props: {
    title: string;
    points: string[];
    stats?: Array<{ label: string; value: string }>;
    duration: number;
    fps: number;
    aspectRatio: '9:16' | '16:9' | '1:1';
  };
  outputFormat: 'mp4' | 'webm';
}
```

## Common Workflows

### Daily Content Generation Automation

```typescript
import cron from 'node-cron';
import { ContentPipeline } from '@/lib/pipeline';

// Schedule daily content generation
cron.schedule('0 9 * * *', async () => {
  const topics = [
    'AI marketing trends',
    'Content automation tools',
    'Social media strategies'
  ];
  
  for (const topic of topics) {
    try {
      const result = await runContentPipeline(topic);
      console.log(`✅ Generated content for: ${topic}`);
      
      // Save to database or file system
      await saveContent(result);
    } catch (error) {
      console.error(`❌ Failed for ${topic}:`, error);
    }
  }
});
```

### Batch Video Generation

```typescript
async function batchGenerateVideos(contents: GeneratedContent[]) {
  const platforms = ['reels', 'tiktok', 'youtube-shorts'];
  
  const allVideos = await Promise.all(
    contents.map(async (content) => {
      const platformVideos = await Promise.all(
        platforms.map(platform => 
          renderVideo({
            composition: 'ContentVideo',
            props: {
              title: content.title,
              points: content.sections.map(s => s.heading),
              aspectRatio: platform === 'youtube-shorts' ? '9:16' : '9:16'
            },
            outputFormat: 'mp4'
          })
        )
      );
      
      return {
        contentId: content.metadata.keywords[0],
        videos: platformVideos
      };
    })
  );
  
  return allVideos;
}
```

## Running the Development Server

```bash
# Start Next.js development server
npm run dev

# Access the application
# http://localhost:3000

# Build for production
npm run build

# Start production server
npm start
```

## Remotion Video Commands

```bash
# Preview Remotion compositions
npx remotion preview

# Render a specific composition
npx remotion render ContentVideo output.mp4

# Render with custom props
npx remotion render ContentVideo output.mp4 --props='{"title":"My Title"}'
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement retry logic with exponential backoff
async function generateWithRetry(params: any, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await generateContent(params);
    } catch (error) {
      if (error.message.includes('rate limit')) {
        const delay = Math.pow(2, i) * 1000;
        console.log(`Rate limited, retrying in ${delay}ms...`);
        await new Promise(resolve => setTimeout(resolve, delay));
      } else {
        throw error;
      }
    }
  }
  throw new Error('Max retries exceeded');
}
```

### Video Rendering Memory Issues

```typescript
// Process videos in batches to avoid memory issues
async function renderVideosInBatches(contents: any[], batchSize = 3) {
  const results = [];
  
  for (let i = 0; i < contents.length; i += batchSize) {
    const batch = contents.slice(i, i + batchSize);
    const batchResults = await Promise.all(
      batch.map(content => renderVideo(content))
    );
    results.push(...batchResults);
    
    // Clear memory between batches
    if (global.gc) global.gc();
  }
  
  return results;
}
```

### Research Data Quality

```typescript
// Filter and validate research results
function validateResearch(data: ResearchData[]): ResearchData[] {
  return data.filter(item => {
    // Must be recent (within 48 hours)
    const age = Date.now() - item.publishedAt.getTime();
    if (age > 48 * 60 * 60 * 1000) return false;
    
    // Must have substantial content
    if (item.content.length < 200) return false;
    
    // Must be from trusted source
    const trustedSources = ['techcrunch', 'a16z', 'linkedin'];
    if (!trustedSources.some(s => item.source.includes(s))) return false;
    
    return true;
  });
}
```

### Bilingual Content Consistency

```typescript
// Ensure consistency between language versions
async function generateConsistentBilingual(keyword: string, research: any[]) {
  // Generate English first
  const english = await generateContent({
    keyword,
    research,
    format: 'toplist',
    tone: 'expert',
    language: 'en'
  });
  
  // Use English as reference for Vietnamese
  const vietnamese = await generateContent({
    keyword,
    research,
    format: 'toplist',
    tone: 'expert',
    language: 'vi',
    referenceContent: english // Pass English version as context
  });
  
  return { english, vietnamese };
}
```

This skill provides AI coding agents with comprehensive knowledge to help developers implement, customize, and troubleshoot the Ultimate AI Content Pipeline for automated marketing content creation.
