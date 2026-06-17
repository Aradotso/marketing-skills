---
name: marketing-pipeline-ai-content-automation
description: Automated content pipeline using AI (Claude, OpenAI) for research, scriptwriting, and video generation with Remotion
triggers:
  - how do I automate content creation with AI pipeline
  - set up automated marketing content generation
  - create AI-powered content workflow from research to video
  - use Claude and OpenAI for automated content scripts
  - generate videos automatically from content using Remotion
  - build automated content pipeline with AI research
  - configure AI content automation with video rendering
  - implement end-to-end AI marketing content system
---

# Marketing Pipeline AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the Ultimate AI Content Pipeline - an automated content creation system that handles research, scriptwriting, multi-format content generation, and video rendering using Claude 3, OpenAI, and Remotion.

## What This Project Does

The Marketing Pipeline is an end-to-end content automation system that:

- **Auto-researches**: Crawls fresh data from TechCrunch, a16z, Twitter/X, LinkedIn within 24h
- **AI content generation**: Creates content in multiple formats (toplist, POV, case study, how-to) using Claude/OpenAI
- **Multi-language**: Generates content in both English and Vietnamese with customizable tone
- **Video automation**: Renders infographics and short-form videos via Remotion
- **Multi-platform**: Exports optimized video formats for Reels, TikTok, Shorts

## Installation

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
yarn install

# Set up environment variables
cp .env.example .env
```

## Configuration

Create a `.env` file in the root directory:

```env
# AI API Keys
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# RapidAPI for research/crawling
RAPIDAPI_KEY=your_rapidapi_key

# Database (if applicable)
DATABASE_URL=your_database_url

# Remotion configuration
REMOTION_AWS_ACCESS_KEY_ID=your_aws_access_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret

# Next.js configuration
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

## Key Architecture

The pipeline follows this flow:

```typescript
// Core pipeline stages
Research → AI Content Generation → Format Optimization → Video Rendering → Distribution
```

## API Usage Patterns

### 1. Research & Data Crawling

```typescript
import { researchService } from '@/services/research';

// Auto-crawl recent content from multiple sources
async function gatherResearch(keyword: string) {
  const research = await researchService.crawl({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeframe: '24h',
    maxResults: 20
  });
  
  return research.insights;
}

// Example usage
const insights = await gatherResearch('AI marketing automation');
```

### 2. AI Content Generation with Claude

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

// Generate content in specific format
async function generateContent(
  topic: string,
  format: 'toplist' | 'pov' | 'case-study' | 'how-to',
  language: 'en' | 'vi',
  tone: 'expert' | 'friendly' | 'humorous'
) {
  const message = await anthropic.messages.create({
    model: 'claude-3-opus-20240229',
    max_tokens: 4096,
    messages: [{
      role: 'user',
      content: `Create a ${format} article about "${topic}" in ${language} with a ${tone} tone. Include data-backed insights and current trends.`
    }]
  });
  
  return message.content[0].text;
}

// Example usage
const article = await generateContent(
  'Top AI Tools for Content Creators',
  'toplist',
  'en',
  'expert'
);
```

### 3. OpenAI Alternative

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateWithGPT(prompt: string, systemRole: string) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      { role: 'system', content: systemRole },
      { role: 'user', content: prompt }
    ],
    temperature: 0.7,
    max_tokens: 3000
  });
  
  return completion.choices[0].message.content;
}

// Example: Generate script
const script = await generateWithGPT(
  'Write a 60-second video script about AI content automation',
  'You are an expert content strategist specializing in short-form video content.'
);
```

### 4. Complete Pipeline Implementation

```typescript
import { researchService } from '@/services/research';
import { contentGenerator } from '@/services/content';
import { videoRenderer } from '@/services/video';

// Full automation pipeline
async function runContentPipeline(keyword: string) {
  try {
    // Step 1: Research
    console.log('🔍 Gathering research...');
    const research = await researchService.crawl({
      keyword,
      sources: ['techcrunch', 'twitter'],
      timeframe: '24h'
    });
    
    // Step 2: Generate content
    console.log('✍️ Generating content...');
    const content = await contentGenerator.create({
      topic: keyword,
      research: research.insights,
      format: 'how-to',
      language: 'en',
      tone: 'friendly'
    });
    
    // Step 3: Create video script
    console.log('🎬 Creating video script...');
    const videoScript = await contentGenerator.createVideoScript({
      content: content.text,
      duration: 60, // seconds
      platform: 'tiktok'
    });
    
    // Step 4: Render video
    console.log('🎥 Rendering video...');
    const video = await videoRenderer.render({
      script: videoScript,
      format: '9:16', // TikTok/Reels format
      outputPath: './output/video.mp4'
    });
    
    return {
      content,
      videoScript,
      videoUrl: video.url
    };
  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}

// Execute pipeline
const result = await runContentPipeline('AI marketing trends 2024');
```

## Remotion Video Generation

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

// Video composition
async function renderContentVideo(scriptData: any) {
  const bundleLocation = await bundle(
    path.join(process.cwd(), 'src/remotion/index.ts')
  );
  
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: scriptData,
  });
  
  const outputLocation = path.join(
    process.cwd(),
    'public/videos',
    `${Date.now()}.mp4`
  );
  
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: scriptData,
  });
  
  return outputLocation;
}

// Example Remotion component
// src/remotion/ContentVideo.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig } from 'remotion';

export const ContentVideo: React.FC<{
  title: string;
  points: string[];
  bgColor: string;
}> = ({ title, points, bgColor }) => {
  const frame = useCurrentFrame();
  const { fps } = useVideoConfig();
  
  const opacity = Math.min(1, frame / 30);
  
  return (
    <AbsoluteFill style={{ backgroundColor: bgColor }}>
      <div style={{ 
        padding: 60,
        opacity,
        transform: `translateY(${Math.max(0, 30 - frame)}px)`
      }}>
        <h1 style={{ fontSize: 72, marginBottom: 40 }}>{title}</h1>
        {points.map((point, i) => (
          <p key={i} style={{ fontSize: 36, marginBottom: 20 }}>
            {point}
          </p>
        ))}
      </div>
    </AbsoluteFill>
  );
};
```

## Common Patterns

### Multi-language Content Generation

```typescript
interface ContentRequest {
  topic: string;
  format: string;
  languages: string[];
}

async function generateMultiLanguage(request: ContentRequest) {
  const results = await Promise.all(
    request.languages.map(async (lang) => {
      const content = await contentGenerator.create({
        topic: request.topic,
        format: request.format,
        language: lang,
        tone: 'expert'
      });
      
      return { language: lang, content };
    })
  );
  
  return results;
}

// Generate English and Vietnamese versions
const bilingual = await generateMultiLanguage({
  topic: 'AI Content Marketing',
  format: 'toplist',
  languages: ['en', 'vi']
});
```

### Batch Processing

```typescript
async function batchGenerateContent(keywords: string[]) {
  const results = [];
  
  for (const keyword of keywords) {
    const result = await runContentPipeline(keyword);
    results.push({
      keyword,
      ...result
    });
    
    // Rate limiting
    await new Promise(resolve => setTimeout(resolve, 2000));
  }
  
  return results;
}

// Process multiple topics
const batch = await batchGenerateContent([
  'AI automation tools',
  'Content marketing trends',
  'Video marketing strategies'
]);
```

### Custom Format Templates

```typescript
const formatTemplates = {
  toplist: {
    structure: 'Introduction → List Items → Conclusion',
    systemPrompt: 'Create a ranked list with data-backed reasoning for each item.',
  },
  pov: {
    structure: 'Hook → Personal Angle → Evidence → CTA',
    systemPrompt: 'Write from a unique perspective with strong opinions.',
  },
  'case-study': {
    structure: 'Problem → Solution → Results → Lessons',
    systemPrompt: 'Analyze a real-world example with specific metrics.',
  },
  'how-to': {
    structure: 'Problem → Step-by-step → Tips → Summary',
    systemPrompt: 'Provide actionable, easy-to-follow instructions.',
  }
};

async function generateWithTemplate(
  topic: string,
  format: keyof typeof formatTemplates
) {
  const template = formatTemplates[format];
  
  return await generateWithGPT(
    `Topic: ${topic}\nStructure: ${template.structure}`,
    template.systemPrompt
  );
}
```

## API Endpoints (Next.js)

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';

export async function POST(request: NextRequest) {
  try {
    const { keyword, format, language } = await request.json();
    
    const result = await runContentPipeline(keyword);
    
    return NextResponse.json({
      success: true,
      data: result
    });
  } catch (error) {
    return NextResponse.json({
      success: false,
      error: error.message
    }, { status: 500 });
  }
}

// app/api/research/route.ts
export async function POST(request: NextRequest) {
  const { keyword, sources } = await request.json();
  
  const research = await researchService.crawl({
    keyword,
    sources,
    timeframe: '24h'
  });
  
  return NextResponse.json({ research });
}
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement rate limiting with retry logic
async function apiCallWithRetry(
  apiCall: () => Promise<any>,
  maxRetries = 3
) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await apiCall();
    } catch (error) {
      if (error.status === 429) {
        const delay = Math.pow(2, i) * 1000; // Exponential backoff
        console.log(`Rate limited. Retrying in ${delay}ms...`);
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
// Render videos in chunks for large content
async function renderLargeVideo(segments: any[]) {
  const renderedSegments = [];
  
  for (const segment of segments) {
    const video = await renderContentVideo(segment);
    renderedSegments.push(video);
    
    // Clear memory between renders
    if (global.gc) global.gc();
  }
  
  // Combine segments using ffmpeg or similar
  return combineVideoSegments(renderedSegments);
}
```

### Missing Research Data

```typescript
// Fallback to cached or alternative sources
async function robustResearch(keyword: string) {
  try {
    return await researchService.crawl({ keyword });
  } catch (error) {
    console.warn('Primary research failed, using fallback');
    
    // Try alternative sources or cached data
    return await fallbackResearchService.get(keyword);
  }
}
```

### Language Detection

```typescript
import { detect } from 'languagedetect';

function ensureCorrectLanguage(text: string, expectedLang: string) {
  const detected = detect(text);
  
  if (detected !== expectedLang) {
    console.warn(`Language mismatch: expected ${expectedLang}, got ${detected}`);
    // Re-generate or translate as needed
  }
  
  return text;
}
```

## Best Practices

1. **Always validate research data** before generating content
2. **Cache API responses** to reduce costs and improve speed
3. **Use streaming** for long-form content generation
4. **Implement proper error handling** for each pipeline stage
5. **Monitor API usage** to stay within rate limits
6. **Version control video templates** in Remotion
7. **Test content quality** before bulk generation

## Development Commands

```bash
# Run development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Render test video
npm run remotion:render

# Run tests
npm test
```
