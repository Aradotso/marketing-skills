---
name: marketing-pipeline-share-ai-content-automation
description: AI-powered content pipeline for automated research, script generation, and video creation using Claude, OpenAI, and Remotion
triggers:
  - automate my content creation pipeline with AI
  - generate video content from text automatically
  - research and write articles using AI
  - create marketing content with Claude and OpenAI
  - build automated content workflows
  - set up AI content generation system
  - crawl news and generate articles
  - render videos from text using Remotion
---

# Marketing Pipeline Share - AI Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the Ultimate AI Content Pipeline - a complete automated content creation system that handles research, scriptwriting, article generation, and video rendering. The pipeline integrates Claude 3, OpenAI, web scraping, and Remotion for end-to-end content automation.

## What This Project Does

Marketing Pipeline Share is a TypeScript-based Next.js application that automates the entire content creation workflow:

1. **Auto-Research**: Crawls recent content from sources like TechCrunch, a16z, Twitter/X, LinkedIn
2. **AI Content Generation**: Uses Claude/OpenAI to generate articles in multiple formats (toplist, POV, case study, how-to)
3. **Multi-language Support**: Generates content in both English and Vietnamese
4. **Video Rendering**: Automatically creates videos and infographics using Remotion
5. **Publishing Pipeline**: Prepares content for multi-platform distribution (Reels, TikTok, Shorts)

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
# .env.local
OPENAI_API_KEY=your_openai_key_here
ANTHROPIC_API_KEY=your_claude_api_key_here
RAPIDAPI_KEY=your_rapidapi_key_here

# Optional
DATABASE_URL=your_database_connection_string
NEXT_PUBLIC_API_URL=http://localhost:3000
```

### Start Development Server

```bash
npm run dev
# or
yarn dev
# or
pnpm dev
```

Application will be available at `http://localhost:3000`

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/             # Core libraries
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── scrapers/    # Web scraping modules
│   │   ├── video/       # Remotion video rendering
│   │   └── content/     # Content generation logic
│   ├── utils/           # Utility functions
│   └── types/           # TypeScript type definitions
├── public/              # Static assets
└── remotion/            # Remotion video templates
```

## Core API Usage

### 1. Content Research & Scraping

```typescript
// src/lib/scrapers/newsResearch.ts
import { scrapeLatestNews } from '@/lib/scrapers/newsResearch';

interface NewsSource {
  url: string;
  selector: string;
  timeframe: number; // hours
}

async function researchTopic(keyword: string): Promise<ArticleData[]> {
  const sources: NewsSource[] = [
    { url: 'https://techcrunch.com', selector: '.article', timeframe: 24 },
    { url: 'https://a16z.com/blog', selector: '.post', timeframe: 48 }
  ];

  const articles = await scrapeLatestNews(keyword, sources);
  
  return articles.map(article => ({
    title: article.title,
    content: article.content,
    url: article.url,
    publishedAt: article.publishedAt,
    insights: extractInsights(article.content)
  }));
}

function extractInsights(content: string): string[] {
  // Extract key data points, statistics, quotes
  const insights: string[] = [];
  
  // Look for statistics
  const statsRegex = /\d+%|\$\d+[KMB]?|\d+\s?(million|billion)/gi;
  const stats = content.match(statsRegex);
  
  if (stats) {
    insights.push(...stats);
  }
  
  return insights;
}
```

### 2. AI Content Generation with Claude

```typescript
// src/lib/ai/contentGenerator.ts
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

interface ContentRequest {
  topic: string;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'expert' | 'friendly' | 'humorous';
  researchData: ArticleData[];
}

async function generateContent(request: ContentRequest): Promise<string> {
  const systemPrompt = buildSystemPrompt(request.format, request.tone);
  const userPrompt = buildUserPrompt(request.topic, request.researchData);

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4000,
    temperature: 0.7,
    system: systemPrompt,
    messages: [
      {
        role: 'user',
        content: userPrompt
      }
    ]
  });

  return message.content[0].type === 'text' 
    ? message.content[0].text 
    : '';
}

function buildSystemPrompt(format: string, tone: string): string {
  const formatInstructions = {
    'toplist': 'Create a numbered list article with clear rankings and explanations',
    'pov': 'Write from a unique perspective or opinion, making strong arguments',
    'case-study': 'Present a detailed analysis with data, challenges, and outcomes',
    'how-to': 'Create step-by-step instructions that are actionable and clear'
  };

  const toneGuidelines = {
    'expert': 'Use technical terminology, cite sources, maintain professional authority',
    'friendly': 'Use conversational language, relatable examples, warm tone',
    'humorous': 'Include wit, clever analogies, entertaining while informative'
  };

  return `You are an expert content creator specializing in ${format} articles.
${formatInstructions[format]}

Tone: ${toneGuidelines[tone]}

Always include:
- Compelling headline
- Strong introduction hook
- Data-backed insights
- Actionable takeaways
- Clear structure with subheadings`;
}

function buildUserPrompt(topic: string, research: ArticleData[]): string {
  const researchSummary = research
    .map(r => `- ${r.title}\n  Key points: ${r.insights.join(', ')}`)
    .join('\n\n');

  return `Topic: ${topic}

Recent research and insights (last 24-48 hours):
${researchSummary}

Create a comprehensive article incorporating these latest insights and data points.`;
}
```

### 3. Multi-language Content Generation

```typescript
// src/lib/ai/multiLanguage.ts
interface BilingualContent {
  english: string;
  vietnamese: string;
}

async function generateBilingualContent(
  request: ContentRequest
): Promise<BilingualContent> {
  // Generate English version
  const englishContent = await generateContent({
    ...request,
    language: 'en'
  });

  // Generate Vietnamese version
  const vietnameseContent = await generateContent({
    ...request,
    language: 'vi'
  });

  return {
    english: englishContent,
    vietnamese: vietnameseContent
  };
}

// Alternative: Generate once and translate
async function generateAndTranslate(
  request: ContentRequest
): Promise<BilingualContent> {
  const primaryContent = await generateContent(request);
  
  const translation = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4000,
    messages: [
      {
        role: 'user',
        content: `Translate this article to ${request.language === 'en' ? 'Vietnamese' : 'English'}. 
Maintain the tone, style, and structure. Adapt idioms and cultural references appropriately.

${primaryContent}`
      }
    ]
  });

  const translatedText = translation.content[0].type === 'text'
    ? translation.content[0].text
    : '';

  return request.language === 'en'
    ? { english: primaryContent, vietnamese: translatedText }
    : { english: translatedText, vietnamese: primaryContent };
}
```

### 4. Video Generation with Remotion

```typescript
// src/lib/video/renderVideo.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoConfig {
  content: string;
  format: 'reel' | 'tiktok' | 'shorts';
  duration: number; // seconds
}

async function renderContentVideo(config: VideoConfig): Promise<string> {
  const compositionId = 'ContentVideo';
  
  // Parse content into scenes
  const scenes = parseContentIntoScenes(config.content, config.duration);
  
  // Bundle Remotion project
  const bundleLocation = await bundle({
    entryPoint: path.resolve('./remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  // Get composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: {
      scenes,
      format: config.format
    },
  });

  // Render video
  const outputLocation = path.join(
    process.cwd(),
    'output',
    `video-${Date.now()}.mp4`
  );

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: {
      scenes,
      format: config.format
    },
  });

  return outputLocation;
}

function parseContentIntoScenes(content: string, duration: number) {
  // Split content into key points
  const sections = content.split('\n\n').filter(s => s.trim());
  const secondsPerScene = duration / sections.length;

  return sections.map((text, index) => ({
    id: index,
    text: text.trim(),
    startTime: index * secondsPerScene,
    duration: secondsPerScene,
    animation: getAnimationType(index)
  }));
}

function getAnimationType(index: number): string {
  const animations = ['fadeIn', 'slideUp', 'zoom', 'slideLeft'];
  return animations[index % animations.length];
}
```

### 5. Remotion Video Component

```typescript
// remotion/ContentVideo.tsx
import { AbsoluteFill, interpolate, useCurrentFrame, useVideoConfig } from 'remotion';
import { FC } from 'react';

interface Scene {
  id: number;
  text: string;
  startTime: number;
  duration: number;
  animation: string;
}

interface ContentVideoProps {
  scenes: Scene[];
  format: 'reel' | 'tiktok' | 'shorts';
}

export const ContentVideo: FC<ContentVideoProps> = ({ scenes, format }) => {
  const frame = useCurrentFrame();
  const { fps, width, height } = useVideoConfig();

  const dimensions = {
    reel: { width: 1080, height: 1920 },
    tiktok: { width: 1080, height: 1920 },
    shorts: { width: 1080, height: 1920 }
  }[format];

  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a1a' }}>
      {scenes.map((scene) => {
        const startFrame = scene.startTime * fps;
        const durationInFrames = scene.duration * fps;
        const endFrame = startFrame + durationInFrames;

        if (frame < startFrame || frame > endFrame) return null;

        const progress = (frame - startFrame) / durationInFrames;
        const opacity = interpolate(
          progress,
          [0, 0.1, 0.9, 1],
          [0, 1, 1, 0]
        );

        const translateY = interpolate(
          progress,
          [0, 1],
          [50, -50]
        );

        return (
          <AbsoluteFill
            key={scene.id}
            style={{
              justifyContent: 'center',
              alignItems: 'center',
              padding: 60,
              opacity,
              transform: `translateY(${translateY}px)`
            }}
          >
            <h2
              style={{
                fontSize: 48,
                color: 'white',
                textAlign: 'center',
                fontWeight: 'bold',
                lineHeight: 1.4,
                textShadow: '2px 2px 4px rgba(0,0,0,0.5)'
              }}
            >
              {scene.text}
            </h2>
          </AbsoluteFill>
        );
      })}
    </AbsoluteFill>
  );
};
```

## API Routes

### Content Generation Endpoint

```typescript
// src/app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { generateContent } from '@/lib/ai/contentGenerator';
import { researchTopic } from '@/lib/scrapers/newsResearch';

export async function POST(req: NextRequest) {
  try {
    const body = await req.json();
    const { topic, format, language, tone } = body;

    // Step 1: Research
    const researchData = await researchTopic(topic);

    // Step 2: Generate content
    const content = await generateContent({
      topic,
      format,
      language,
      tone,
      researchData
    });

    return NextResponse.json({
      success: true,
      content,
      sources: researchData.length
    });
  } catch (error) {
    console.error('Content generation error:', error);
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
import { renderContentVideo } from '@/lib/video/renderVideo';

export async function POST(req: NextRequest) {
  try {
    const body = await req.json();
    const { content, format, duration } = body;

    const videoPath = await renderContentVideo({
      content,
      format,
      duration: duration || 60
    });

    return NextResponse.json({
      success: true,
      videoPath,
      url: `/output/${path.basename(videoPath)}`
    });
  } catch (error) {
    console.error('Video rendering error:', error);
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

## Common Workflows

### Complete Content Pipeline

```typescript
// src/lib/pipeline/fullPipeline.ts
interface PipelineResult {
  content: BilingualContent;
  videoUrl: string;
  metadata: {
    topic: string;
    sources: number;
    generatedAt: Date;
  };
}

async function runFullPipeline(
  topic: string,
  format: ContentRequest['format']
): Promise<PipelineResult> {
  // 1. Research phase
  console.log('Starting research...');
  const researchData = await researchTopic(topic);
  
  // 2. Content generation
  console.log('Generating content...');
  const bilingualContent = await generateBilingualContent({
    topic,
    format,
    language: 'en',
    tone: 'friendly',
    researchData
  });

  // 3. Video rendering
  console.log('Rendering video...');
  const videoPath = await renderContentVideo({
    content: bilingualContent.english,
    format: 'reel',
    duration: 60
  });

  return {
    content: bilingualContent,
    videoUrl: videoPath,
    metadata: {
      topic,
      sources: researchData.length,
      generatedAt: new Date()
    }
  };
}
```

### Batch Content Generation

```typescript
// src/lib/pipeline/batchGeneration.ts
async function generateMultipleFormats(topic: string) {
  const formats: ContentRequest['format'][] = [
    'toplist',
    'pov',
    'case-study',
    'how-to'
  ];

  const results = await Promise.all(
    formats.map(format =>
      runFullPipeline(topic, format)
    )
  );

  return results.map((result, index) => ({
    format: formats[index],
    ...result
  }));
}
```

## Configuration

### Content Format Templates

```typescript
// src/config/contentTemplates.ts
export const contentTemplates = {
  toplist: {
    structure: [
      'Introduction with hook',
      'Methodology/criteria explanation',
      'Numbered items (5-10)',
      'Conclusion with CTA'
    ],
    minWords: 1200,
    maxWords: 2000
  },
  pov: {
    structure: [
      'Controversial/unique statement',
      'Supporting arguments',
      'Counter-arguments addressed',
      'Strong conclusion'
    ],
    minWords: 800,
    maxWords: 1500
  },
  'case-study': {
    structure: [
      'Background/context',
      'Challenge definition',
      'Solution implementation',
      'Results with data',
      'Key takeaways'
    ],
    minWords: 1500,
    maxWords: 2500
  },
  'how-to': {
    structure: [
      'Problem statement',
      'Prerequisites/requirements',
      'Step-by-step instructions',
      'Tips and warnings',
      'Conclusion'
    ],
    minWords: 1000,
    maxWords: 2000
  }
};
```

### Video Format Specifications

```typescript
// src/config/videoFormats.ts
export const videoFormats = {
  reel: {
    width: 1080,
    height: 1920,
    fps: 30,
    maxDuration: 90,
    codec: 'h264'
  },
  tiktok: {
    width: 1080,
    height: 1920,
    fps: 30,
    maxDuration: 180,
    codec: 'h264'
  },
  shorts: {
    width: 1080,
    height: 1920,
    fps: 30,
    maxDuration: 60,
    codec: 'h264'
  }
};
```

## Troubleshooting

### API Key Issues

```typescript
// src/lib/utils/validateEnv.ts
export function validateEnvironment() {
  const required = [
    'OPENAI_API_KEY',
    'ANTHROPIC_API_KEY',
    'RAPIDAPI_KEY'
  ];

  const missing = required.filter(key => !process.env[key]);

  if (missing.length > 0) {
    throw new Error(
      `Missing required environment variables: ${missing.join(', ')}`
    );
  }
}

// Call at app startup
validateEnvironment();
```

### Rate Limiting

```typescript
// src/lib/utils/rateLimiter.ts
class RateLimiter {
  private queue: Array<() => Promise<any>> = [];
  private processing = false;
  private requestsPerMinute: number;
  private delay: number;

  constructor(requestsPerMinute: number = 10) {
    this.requestsPerMinute = requestsPerMinute;
    this.delay = 60000 / requestsPerMinute;
  }

  async add<T>(fn: () => Promise<T>): Promise<T> {
    return new Promise((resolve, reject) => {
      this.queue.push(async () => {
        try {
          const result = await fn();
          resolve(result);
        } catch (error) {
          reject(error);
        }
      });

      this.process();
    });
  }

  private async process() {
    if (this.processing || this.queue.length === 0) return;

    this.processing = true;

    while (this.queue.length > 0) {
      const fn = this.queue.shift();
      if (fn) await fn();
      await new Promise(resolve => setTimeout(resolve, this.delay));
    }

    this.processing = false;
  }
}

// Usage
const limiter = new RateLimiter(10);

async function rateLimitedGenerate(request: ContentRequest) {
  return limiter.add(() => generateContent(request));
}
```

### Video Rendering Memory Issues

```typescript
// Reduce video quality for memory-constrained environments
const lowMemoryVideoConfig = {
  codec: 'h264',
  crf: 23, // Higher = lower quality but less memory
  pixelFormat: 'yuv420p',
  numberOfGifLoops: null,
  everyNthFrame: 1,
  concurrency: 1 // Render single-threaded
};

await renderMedia({
  composition,
  serveUrl: bundleLocation,
  outputLocation,
  ...lowMemoryVideoConfig
});
```

### Content Quality Validation

```typescript
// src/lib/utils/contentValidator.ts
interface ValidationResult {
  isValid: boolean;
  errors: string[];
  warnings: string[];
}

function validateContent(
  content: string,
  format: string
): ValidationResult {
  const errors: string[] = [];
  const warnings: string[] = [];

  const wordCount = content.split(/\s+/).length;
  const template = contentTemplates[format];

  if (wordCount < template.minWords) {
    errors.push(
      `Content too short: ${wordCount} words (minimum: ${template.minWords})`
    );
  }

  if (wordCount > template.maxWords) {
    warnings.push(
      `Content lengthy: ${wordCount} words (recommended: ${template.maxWords})`
    );
  }

  // Check for headings
  const headingCount = (content.match(/^#{1,3}\s/gm) || []).length;
  if (headingCount < 3) {
    warnings.push('Consider adding more subheadings for better structure');
  }

  return {
    isValid: errors.length === 0,
    errors,
    warnings
  };
}
```

This skill provides comprehensive coverage for automating content creation workflows using the Marketing Pipeline Share project, from research to video generation.
