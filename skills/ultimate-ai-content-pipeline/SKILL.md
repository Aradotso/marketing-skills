---
name: ultimate-ai-content-pipeline
description: Automated content pipeline from research to video generation using AI (Claude, OpenAI) and Remotion for Vietnamese/English content
triggers:
  - how do I automate content creation with AI research
  - generate video content from text automatically
  - set up AI content pipeline with Claude and OpenAI
  - create multilingual content with auto research
  - build automated marketing content workflow
  - use Remotion for AI-generated video content
  - scrape news and generate social media posts
  - automate content from research to video
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

An end-to-end AI-powered content automation system that researches trending topics, generates multilingual content (Vietnamese/English), and automatically renders videos. Built with Next.js, TypeScript, Claude 3, OpenAI, and Remotion.

## What It Does

This pipeline automates the entire content creation workflow:

1. **Auto-Research**: Crawls and analyzes real-time data from TechCrunch, a16z, Twitter/X, LinkedIn
2. **AI Content Generation**: Creates articles in multiple formats (toplist, POV, case study, how-to) using Claude/OpenAI
3. **Multilingual Support**: Generates both English and Vietnamese content simultaneously
4. **Video Rendering**: Automatically converts content into videos/infographics using Remotion
5. **Social Media Optimization**: Exports videos optimized for Reels, TikTok, Shorts

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

### Required Environment Variables

```bash
# AI Services
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Application
NEXT_PUBLIC_APP_URL=http://localhost:3000

# Optional: Database (if using)
DATABASE_URL=your_database_url
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/             # Core libraries
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── research/    # Content research/crawling
│   │   ├── generator/   # Content generation logic
│   │   └── video/       # Remotion video rendering
│   ├── services/        # API services
│   └── types/           # TypeScript types
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Key APIs and Usage

### 1. Content Research

```typescript
import { researchTopics } from '@/lib/research/scanner';

async function gatherResearch(keyword: string) {
  const research = await researchTopics({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeframe: '24h',
    language: 'en'
  });
  
  return {
    articles: research.articles,
    insights: research.insights,
    trends: research.trends,
    dataPoints: research.dataPoints
  };
}
```

### 2. AI Content Generation with Claude

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

async function generateContent(
  research: any,
  format: 'toplist' | 'pov' | 'case-study' | 'how-to'
) {
  const prompt = `
    Based on this research: ${JSON.stringify(research)}
    
    Create a ${format} article in both English and Vietnamese.
    Include data-backed insights and trending information.
    Tone: Professional but engaging
  `;

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [
      { role: 'user', content: prompt }
    ],
  });

  return message.content[0].text;
}
```

### 3. AI Content Generation with OpenAI

```typescript
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

async function generateWithGPT(
  topic: string,
  tone: 'expert' | 'friendly' | 'humorous'
) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: `You are a ${tone} content writer specializing in marketing.`
      },
      {
        role: 'user',
        content: `Write an article about: ${topic}`
      }
    ],
    temperature: 0.7,
  });

  return completion.choices[0].message.content;
}
```

### 4. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import { VideoTemplate } from '../remotion/VideoTemplate';

async function renderContentVideo(content: {
  title: string;
  points: string[];
  language: 'en' | 'vi';
}) {
  const bundled = await bundle({
    entryPoint: './remotion/index.ts',
    webpackOverride: (config) => config,
  });

  const composition = await selectComposition({
    serveUrl: bundled,
    id: 'ContentVideo',
    inputProps: content,
  });

  await renderMedia({
    composition,
    serveUrl: bundled,
    codec: 'h264',
    outputLocation: `out/${content.title}-${content.language}.mp4`,
    inputProps: content,
  });
}
```

### 5. Full Pipeline Integration

```typescript
import { researchTopics } from '@/lib/research/scanner';
import { generateContent } from '@/lib/ai/claude';
import { renderContentVideo } from '@/lib/video/renderer';

async function runContentPipeline(keyword: string) {
  // Step 1: Research
  console.log('🔍 Researching topic...');
  const research = await researchTopics({
    keyword,
    sources: ['techcrunch', 'twitter'],
    timeframe: '24h',
  });

  // Step 2: Generate Content
  console.log('✍️ Generating content...');
  const content = await generateContent(research, 'toplist');

  // Step 3: Parse and structure
  const structured = parseContentToVideo(content);

  // Step 4: Render Videos (both languages)
  console.log('🎬 Rendering videos...');
  await Promise.all([
    renderContentVideo({ ...structured, language: 'en' }),
    renderContentVideo({ ...structured, language: 'vi' }),
  ]);

  console.log('✅ Pipeline complete!');
  return {
    content,
    videos: [
      `out/${structured.title}-en.mp4`,
      `out/${structured.title}-vi.mp4`,
    ],
  };
}
```

## Common Patterns

### Custom Content Formats

```typescript
interface ContentFormat {
  type: 'toplist' | 'pov' | 'case-study' | 'how-to';
  structure: {
    introduction: boolean;
    mainPoints: number;
    conclusion: boolean;
    cta: boolean;
  };
  tone: 'expert' | 'friendly' | 'humorous';
}

async function generateCustomFormat(
  research: any,
  format: ContentFormat
) {
  const systemPrompt = `
    Create a ${format.type} article with ${format.structure.mainPoints} main points.
    ${format.structure.introduction ? 'Include an engaging introduction.' : ''}
    ${format.structure.conclusion ? 'Add a strong conclusion.' : ''}
    ${format.structure.cta ? 'End with a clear call-to-action.' : ''}
    Tone: ${format.tone}
  `;

  // Use with Claude or OpenAI
  const content = await generateContent(research, format);
  return content;
}
```

### Multilingual Content Generation

```typescript
async function generateMultilingual(
  topic: string,
  languages: Array<'en' | 'vi'>
) {
  const research = await researchTopics({ keyword: topic });

  const contents = await Promise.all(
    languages.map(async (lang) => {
      const prompt = `
        Language: ${lang === 'en' ? 'English' : 'Vietnamese'}
        Topic: ${topic}
        Research: ${JSON.stringify(research)}
        
        Create culturally appropriate content for ${lang} audience.
      `;

      const content = await anthropic.messages.create({
        model: 'claude-3-5-sonnet-20241022',
        max_tokens: 4096,
        messages: [{ role: 'user', content: prompt }],
      });

      return {
        language: lang,
        content: content.content[0].text,
      };
    })
  );

  return contents;
}
```

### Video Template Configuration

```typescript
// remotion/VideoTemplate.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig } from 'remotion';

export const VideoTemplate: React.FC<{
  title: string;
  points: string[];
  language: 'en' | 'vi';
}> = ({ title, points, language }) => {
  const frame = useCurrentFrame();
  const { fps, durationInFrames } = useVideoConfig();

  return (
    <AbsoluteFill
      style={{
        backgroundColor: '#1a1a1a',
        justifyContent: 'center',
        alignItems: 'center',
        fontFamily: language === 'vi' ? 'Montserrat' : 'Inter',
      }}
    >
      <h1 style={{ fontSize: 60, color: 'white' }}>{title}</h1>
      {points.map((point, i) => (
        <p key={i} style={{ fontSize: 24, color: '#ccc' }}>
          {point}
        </p>
      ))}
    </AbsoluteFill>
  );
};
```

### Batch Processing

```typescript
async function batchGenerateContent(keywords: string[]) {
  const results = [];

  for (const keyword of keywords) {
    try {
      const result = await runContentPipeline(keyword);
      results.push({ keyword, success: true, ...result });
      
      // Rate limiting
      await new Promise(resolve => setTimeout(resolve, 2000));
    } catch (error) {
      results.push({ keyword, success: false, error: error.message });
    }
  }

  return results;
}
```

## CLI Commands

```bash
# Development
npm run dev          # Start Next.js dev server
npm run build        # Build for production
npm run start        # Start production server

# Video Rendering
npm run remotion:preview   # Preview Remotion compositions
npm run remotion:render    # Render videos

# Type Checking
npm run type-check   # Run TypeScript compiler

# Linting
npm run lint         # Run ESLint
```

## Configuration

### Content Generation Settings

```typescript
// src/config/content.ts
export const contentConfig = {
  research: {
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    maxArticles: 20,
    timeframe: '24h',
  },
  generation: {
    defaultModel: 'claude-3-5-sonnet-20241022',
    fallbackModel: 'gpt-4-turbo-preview',
    maxTokens: 4096,
    temperature: 0.7,
  },
  video: {
    fps: 30,
    durationInFrames: 900, // 30 seconds at 30fps
    width: 1080,
    height: 1920, // Vertical for Reels/TikTok
  },
  languages: ['en', 'vi'],
};
```

### API Rate Limiting

```typescript
import pLimit from 'p-limit';

const limit = pLimit(3); // Max 3 concurrent API calls

async function generateWithRateLimit(topics: string[]) {
  const promises = topics.map(topic =>
    limit(() => generateContent(topic, 'toplist'))
  );
  
  return await Promise.all(promises);
}
```

## Troubleshooting

### API Key Issues

```typescript
// Validate API keys on startup
function validateApiKeys() {
  const required = [
    'ANTHROPIC_API_KEY',
    'OPENAI_API_KEY',
    'RAPIDAPI_KEY',
  ];

  const missing = required.filter(key => !process.env[key]);

  if (missing.length > 0) {
    throw new Error(`Missing required API keys: ${missing.join(', ')}`);
  }
}
```

### Research Data Quality

```typescript
function validateResearchData(research: any) {
  if (!research.articles || research.articles.length === 0) {
    console.warn('⚠️ No articles found. Trying alternative sources...');
    // Implement fallback logic
  }

  if (!research.insights || research.insights.length < 3) {
    console.warn('⚠️ Limited insights. Consider expanding timeframe.');
  }

  return research;
}
```

### Video Rendering Failures

```typescript
async function safeRenderVideo(content: any) {
  try {
    await renderContentVideo(content);
  } catch (error) {
    console.error('Video rendering failed:', error);
    
    // Retry with lower quality
    await renderContentVideo({
      ...content,
      quality: 'medium',
      codec: 'h264-mkv',
    });
  }
}
```

### Memory Management for Large Batches

```typescript
async function processLargeBatch(keywords: string[]) {
  const batchSize = 5;
  const results = [];

  for (let i = 0; i < keywords.length; i += batchSize) {
    const batch = keywords.slice(i, i + batchSize);
    const batchResults = await batchGenerateContent(batch);
    results.push(...batchResults);

    // Clear memory between batches
    if (global.gc) {
      global.gc();
    }
  }

  return results;
}
```

## Advanced Usage

### Custom AI Prompt Templates

```typescript
const promptTemplates = {
  toplist: (research: any, lang: string) => `
    Create a toplist article in ${lang}.
    Research data: ${JSON.stringify(research)}
    Format: 
    - Catchy title
    - Brief intro
    - 7-10 numbered points with data backing
    - Conclusion with actionable takeaway
  `,
  
  pov: (research: any, lang: string) => `
    Write a POV (point of view) article in ${lang}.
    Take a strong stance on: ${research.topic}
    Use these insights: ${research.insights.join(', ')}
    Include personal opinion and industry data.
  `,
};
```

### Webhook Integration for Auto-Publishing

```typescript
async function publishContent(content: any, platform: string) {
  const webhooks = {
    facebook: process.env.FACEBOOK_WEBHOOK_URL,
    twitter: process.env.TWITTER_WEBHOOK_URL,
  };

  await fetch(webhooks[platform], {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(content),
  });
}
```

This skill enables AI agents to help developers build automated, AI-powered content pipelines with multilingual support and video generation capabilities.
