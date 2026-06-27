---
name: marketing-pipeline-share-automation
description: AI-powered content automation pipeline for research, scriptwriting, and video generation using Claude, OpenAI, and Remotion
triggers:
  - automate content creation with AI pipeline
  - set up marketing content automation
  - generate videos from content automatically
  - create AI-powered content workflow
  - build automated marketing pipeline
  - research and generate content with AI
  - use Remotion for video automation
  - implement content research automation
---

# Marketing Pipeline Share Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables you to use the **Ultimate AI Content Pipeline** - an all-in-one content automation system that handles research, scriptwriting, multi-language content generation, and automatic video rendering. The pipeline integrates Claude 3, OpenAI, and Remotion to transform keywords into polished content and videos.

## What This Project Does

The Marketing Pipeline Share system automates the entire content creation workflow:

1. **Auto-Scan Research**: Crawls real-time data from TechCrunch, a16z, Twitter/X, LinkedIn within the last 24 hours
2. **AI Content Generation**: Uses Claude/OpenAI to create content in multiple formats (toplist, POV, case study, how-to)
3. **Multi-language Support**: Generates content in both English and Vietnamese simultaneously
4. **Video Rendering**: Automatically creates infographics and short-form videos using Remotion
5. **Platform Optimization**: Exports videos optimized for Reels, TikTok, and Shorts

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

```env
# AI Service Keys
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Database (if using)
DATABASE_URL=your_database_url

# Remotion (for video rendering)
REMOTION_LICENSE_KEY=your_remotion_license

# Optional: Custom endpoints
RESEARCH_API_ENDPOINT=https://api.example.com
```

## Running the Application

```bash
# Development mode
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Render videos (Remotion)
npm run render
```

## Key Architecture & Components

### 1. Content Research Module

The research module crawls and aggregates content from multiple sources:

```typescript
// lib/research/crawler.ts
import { fetchTechCrunchArticles, fetchTwitterTrends } from './sources';

interface ResearchConfig {
  keyword: string;
  sources: string[];
  timeRange: '24h' | '7d' | '30d';
  language?: 'en' | 'vi' | 'both';
}

async function performResearch(config: ResearchConfig) {
  const results = await Promise.all([
    fetchTechCrunchArticles(config.keyword, config.timeRange),
    fetchTwitterTrends(config.keyword),
    // Add more sources
  ]);

  return aggregateAndAnalyze(results);
}

export { performResearch };
```

### 2. AI Content Generation

Generate content using Claude or OpenAI based on research data:

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

interface ContentRequest {
  keyword: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  tone: 'expert' | 'friendly' | 'humorous';
  language: 'en' | 'vi';
  researchData: any;
}

async function generateContentWithClaude(request: ContentRequest) {
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [{
      role: 'user',
      content: `Create a ${request.format} article about "${request.keyword}" in ${request.language}.
      
Tone: ${request.tone}
Research data: ${JSON.stringify(request.researchData)}

Structure the content with:
- Engaging headline
- Introduction with hook
- Main content sections
- Data-backed insights
- Call to action`
    }]
  });

  return message.content[0].text;
}

async function generateContentWithOpenAI(request: ContentRequest) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [{
      role: 'system',
      content: `You are an expert content creator specializing in ${request.format} format with a ${request.tone} tone.`
    }, {
      role: 'user',
      content: `Create an article about "${request.keyword}" in ${request.language} using this research: ${JSON.stringify(request.researchData)}`
    }],
    temperature: 0.7,
  });

  return completion.choices[0].message.content;
}

export { generateContentWithClaude, generateContentWithOpenAI };
```

### 3. Multi-Language Content Pipeline

```typescript
// lib/pipeline/multi-language.ts
interface MultiLanguageContent {
  english: string;
  vietnamese: string;
  metadata: {
    keyword: string;
    format: string;
    generatedAt: Date;
  };
}

async function generateMultiLanguageContent(
  keyword: string,
  format: string,
  researchData: any
): Promise<MultiLanguageContent> {
  const [englishContent, vietnameseContent] = await Promise.all([
    generateContentWithClaude({
      keyword,
      format: format as any,
      tone: 'expert',
      language: 'en',
      researchData
    }),
    generateContentWithClaude({
      keyword,
      format: format as any,
      tone: 'expert',
      language: 'vi',
      researchData
    })
  ]);

  return {
    english: englishContent,
    vietnamese: vietnameseContent,
    metadata: {
      keyword,
      format,
      generatedAt: new Date()
    }
  };
}

export { generateMultiLanguageContent };
```

### 4. Video Rendering with Remotion

```typescript
// remotion/compositions.tsx
import { Composition } from 'remotion';
import { ContentVideo } from './ContentVideo';

export const RemotionRoot = () => {
  return (
    <>
      <Composition
        id="ContentVideo"
        component={ContentVideo}
        durationInFrames={900} // 30 seconds at 30fps
        fps={30}
        width={1080}
        height={1920} // Vertical for Reels/TikTok
      />
    </>
  );
};
```

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, interpolate } from 'remotion';

interface ContentVideoProps {
  title: string;
  points: string[];
  backgroundColor?: string;
}

export const ContentVideo: React.FC<ContentVideoProps> = ({
  title,
  points,
  backgroundColor = '#1a1a1a'
}) => {
  const frame = useCurrentFrame();
  
  const opacity = interpolate(frame, [0, 30], [0, 1], {
    extrapolateRight: 'clamp',
  });

  return (
    <AbsoluteFill style={{ backgroundColor }}>
      <div style={{ 
        padding: 60, 
        opacity,
        display: 'flex',
        flexDirection: 'column',
        justifyContent: 'center'
      }}>
        <h1 style={{ 
          fontSize: 72, 
          color: 'white',
          fontWeight: 'bold',
          marginBottom: 40
        }}>
          {title}
        </h1>
        {points.map((point, index) => {
          const pointOpacity = interpolate(
            frame,
            [30 + index * 20, 50 + index * 20],
            [0, 1],
            { extrapolateRight: 'clamp' }
          );
          
          return (
            <p key={index} style={{ 
              fontSize: 36, 
              color: 'white',
              marginBottom: 20,
              opacity: pointOpacity
            }}>
              {point}
            </p>
          );
        })}
      </div>
    </AbsoluteFill>
  );
};
```

### 5. Complete Pipeline Orchestration

```typescript
// lib/pipeline/orchestrator.ts
import { performResearch } from '../research/crawler';
import { generateMultiLanguageContent } from './multi-language';
import { renderVideo } from '../video/renderer';

interface PipelineConfig {
  keyword: string;
  contentFormat: 'toplist' | 'pov' | 'case-study' | 'how-to';
  generateVideo: boolean;
  platforms?: ('reels' | 'tiktok' | 'shorts')[];
}

async function runContentPipeline(config: PipelineConfig) {
  // Step 1: Research
  console.log('🔍 Starting research phase...');
  const researchData = await performResearch({
    keyword: config.keyword,
    sources: ['techcrunch', 'twitter', 'linkedin'],
    timeRange: '24h',
    language: 'both'
  });

  // Step 2: Generate content
  console.log('✍️ Generating content...');
  const content = await generateMultiLanguageContent(
    config.keyword,
    config.contentFormat,
    researchData
  );

  // Step 3: Extract key points for video
  const keyPoints = extractKeyPoints(content.english);

  // Step 4: Render video if requested
  let videos: any[] = [];
  if (config.generateVideo) {
    console.log('🎬 Rendering videos...');
    videos = await Promise.all(
      (config.platforms || ['reels']).map(platform =>
        renderVideo({
          title: config.keyword,
          points: keyPoints,
          platform
        })
      )
    );
  }

  return {
    research: researchData,
    content,
    videos,
    generatedAt: new Date()
  };
}

function extractKeyPoints(content: string): string[] {
  // Simple extraction - you can enhance with AI
  const lines = content.split('\n').filter(line => 
    line.startsWith('- ') || line.match(/^\d+\./)
  );
  return lines.slice(0, 5).map(line => 
    line.replace(/^[-\d.]+\s*/, '').trim()
  );
}

export { runContentPipeline };
```

### 6. API Routes (Next.js)

```typescript
// app/api/pipeline/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline/orchestrator';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    
    const result = await runContentPipeline({
      keyword: body.keyword,
      contentFormat: body.format || 'toplist',
      generateVideo: body.generateVideo ?? true,
      platforms: body.platforms || ['reels', 'tiktok']
    });

    return NextResponse.json({
      success: true,
      data: result
    });
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

## Usage Examples

### Basic Content Generation

```typescript
import { runContentPipeline } from './lib/pipeline/orchestrator';

// Generate a toplist article with video
const result = await runContentPipeline({
  keyword: 'AI Marketing Tools 2024',
  contentFormat: 'toplist',
  generateVideo: true,
  platforms: ['reels', 'tiktok', 'shorts']
});

console.log('English content:', result.content.english);
console.log('Vietnamese content:', result.content.vietnamese);
console.log('Videos:', result.videos);
```

### Custom Research Configuration

```typescript
import { performResearch } from './lib/research/crawler';

const research = await performResearch({
  keyword: 'SaaS Growth Strategies',
  sources: ['techcrunch', 'a16z', 'twitter'],
  timeRange: '7d',
  language: 'both'
});

// Use research data with custom content generation
const customContent = await generateContentWithClaude({
  keyword: 'SaaS Growth Strategies',
  format: 'case-study',
  tone: 'expert',
  language: 'en',
  researchData: research
});
```

### Video Rendering Only

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia } from '@remotion/renderer';

const bundleLocation = await bundle({
  entryPoint: './remotion/index.ts',
});

await renderMedia({
  composition: {
    id: 'ContentVideo',
    width: 1080,
    height: 1920,
    fps: 30,
    durationInFrames: 900,
  },
  serveUrl: bundleLocation,
  codec: 'h264',
  outputLocation: `out/video-${Date.now()}.mp4`,
  inputProps: {
    title: 'Top 5 AI Tools',
    points: [
      'ChatGPT for content',
      'Midjourney for images',
      'Runway for videos',
      'Claude for analysis',
      'Notion AI for notes'
    ]
  }
});
```

## Common Patterns

### Scheduled Content Generation

```typescript
// lib/scheduler/cron.ts
import cron from 'node-cron';
import { runContentPipeline } from '../pipeline/orchestrator';

// Run daily at 9 AM
cron.schedule('0 9 * * *', async () => {
  const topics = ['AI News', 'Marketing Trends', 'Tech Updates'];
  
  for (const topic of topics) {
    await runContentPipeline({
      keyword: topic,
      contentFormat: 'toplist',
      generateVideo: true
    });
  }
});
```

### Batch Processing

```typescript
async function batchGenerateContent(keywords: string[]) {
  const results = [];
  
  for (const keyword of keywords) {
    const result = await runContentPipeline({
      keyword,
      contentFormat: 'pov',
      generateVideo: false
    });
    results.push(result);
    
    // Rate limiting
    await new Promise(resolve => setTimeout(resolve, 5000));
  }
  
  return results;
}
```

## Troubleshooting

### API Rate Limits

If you encounter rate limits:

```typescript
// lib/utils/rate-limiter.ts
class RateLimiter {
  private queue: Promise<any> = Promise.resolve();
  
  async throttle<T>(fn: () => Promise<T>, delay: number = 1000): Promise<T> {
    const result = this.queue.then(() => fn());
    this.queue = result.then(() => 
      new Promise(resolve => setTimeout(resolve, delay))
    );
    return result;
  }
}

const limiter = new RateLimiter();

// Usage
const content = await limiter.throttle(() => 
  generateContentWithClaude(request)
);
```

### Video Rendering Issues

If videos fail to render:

```bash
# Ensure ffmpeg is installed
ffmpeg -version

# Install if missing (macOS)
brew install ffmpeg

# Install if missing (Ubuntu)
sudo apt-get install ffmpeg
```

### Memory Issues with Large Content

```typescript
// Increase Node.js memory limit
// package.json
{
  "scripts": {
    "dev": "NODE_OPTIONS='--max-old-space-size=4096' next dev",
    "build": "NODE_OPTIONS='--max-old-space-size=4096' next build"
  }
}
```

### Missing Environment Variables

```typescript
// lib/config/validate.ts
function validateEnv() {
  const required = [
    'OPENAI_API_KEY',
    'ANTHROPIC_API_KEY',
    'RAPIDAPI_KEY'
  ];
  
  const missing = required.filter(key => !process.env[key]);
  
  if (missing.length > 0) {
    throw new Error(`Missing environment variables: ${missing.join(', ')}`);
  }
}

validateEnv();
```

## Advanced Configuration

### Custom AI Models

```typescript
// lib/config/ai-models.ts
export const AI_CONFIG = {
  claude: {
    model: 'claude-3-5-sonnet-20241022',
    maxTokens: 4096,
    temperature: 0.7
  },
  openai: {
    model: 'gpt-4-turbo-preview',
    maxTokens: 4000,
    temperature: 0.7
  }
};
```

### Custom Video Templates

```typescript
// remotion/templates/infographic.tsx
export const InfographicTemplate: React.FC = ({ data }) => {
  return (
    <AbsoluteFill>
      {/* Custom infographic layout */}
    </AbsoluteFill>
  );
};
```

This skill enables complete automation of content creation from research to video rendering, making it easy to scale content production across multiple platforms and languages.
