---
name: ultimate-ai-content-pipeline
description: Vietnamese AI content automation system from research to video generation using Claude, OpenAI, and Remotion
triggers:
  - how do I automate content creation with AI research
  - set up Vietnamese content pipeline with Claude and OpenAI
  - generate automated videos from blog posts using Remotion
  - crawl news sources and create content automatically
  - build AI-powered content workflow in TypeScript
  - create multilingual content with automatic video rendering
  - automate social media content from research to video
  - use ultimate AI content pipeline for marketing automation
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This is a comprehensive Vietnamese-focused AI content automation system that handles the entire content creation workflow: from researching trending topics across multiple sources (TechCrunch, a16z, Twitter/X, LinkedIn), generating blog posts in multiple formats and languages (Vietnamese & English), to automatically rendering videos and infographics using Remotion. Built with Next.js and TypeScript, integrating Claude 3, OpenAI, and various content APIs.

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
# AI Providers
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Content Research APIs
RAPIDAPI_KEY=your_rapidapi_key
TWITTER_BEARER_TOKEN=your_twitter_bearer_token

# Optional: Custom API endpoints
NEXT_PUBLIC_API_URL=http://localhost:3000/api

# Remotion (for video rendering)
REMOTION_LICENSE_KEY=your_remotion_license_key
```

## Project Structure

```
marketing-pipeline-share/
├── app/                    # Next.js app directory
│   ├── api/               # API routes
│   │   ├── research/      # Research & crawling endpoints
│   │   ├── generate/      # Content generation endpoints
│   │   └── render/        # Video rendering endpoints
│   └── dashboard/         # UI components
├── lib/
│   ├── ai/               # AI integration modules
│   │   ├── claude.ts     # Claude API wrapper
│   │   └── openai.ts     # OpenAI API wrapper
│   ├── crawler/          # Web scraping utilities
│   └── video/            # Remotion video components
└── remotion/             # Video templates
```

## Core Features

### 1. Auto-Research & Content Crawling

The system automatically crawls multiple sources for trending content:

```typescript
// lib/crawler/research.ts
import { Anthropic } from '@anthropic-ai/sdk';

export interface ResearchSource {
  url: string;
  type: 'techcrunch' | 'a16z' | 'twitter' | 'linkedin';
  timeframe: '24h' | '7d' | '30d';
}

export async function crawlSources(
  keyword: string,
  sources: ResearchSource[]
): Promise<ResearchData[]> {
  const results = await Promise.all(
    sources.map(async (source) => {
      const response = await fetch(`/api/research/crawl`, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          keyword,
          source: source.type,
          timeframe: source.timeframe,
        }),
      });
      return response.json();
    })
  );

  return results.flat();
}

export interface ResearchData {
  title: string;
  url: string;
  summary: string;
  publishedAt: string;
  source: string;
  insights: string[];
}
```

### 2. AI Content Generation

Generate content in multiple formats using Claude or OpenAI:

```typescript
// lib/ai/claude.ts
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

export type ContentFormat = 'toplist' | 'pov' | 'case-study' | 'how-to';
export type Language = 'vi' | 'en';
export type Tone = 'professional' | 'friendly' | 'humorous';

export interface ContentGenerationParams {
  keyword: string;
  researchData: ResearchData[];
  format: ContentFormat;
  language: Language;
  tone: Tone;
  targetAudience?: string;
}

export async function generateContent(
  params: ContentGenerationParams
): Promise<GeneratedContent> {
  const systemPrompt = buildSystemPrompt(params);
  const userPrompt = buildUserPrompt(params);

  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    system: systemPrompt,
    messages: [
      {
        role: 'user',
        content: userPrompt,
      },
    ],
  });

  const content = message.content[0].text;
  return parseGeneratedContent(content, params.language);
}

function buildSystemPrompt(params: ContentGenerationParams): string {
  const toneInstructions = {
    professional: 'Write in a professional, authoritative tone',
    friendly: 'Write in a conversational, approachable tone',
    humorous: 'Write with wit and humor while maintaining credibility',
  };

  const formatInstructions = {
    toplist: 'Create a numbered list format with detailed explanations',
    pov: 'Write from a unique perspective or opinion piece',
    'case-study': 'Structure as a detailed case study with data',
    'how-to': 'Create a step-by-step tutorial or guide',
  };

  return `You are an expert content creator specializing in ${params.language === 'vi' ? 'Vietnamese' : 'English'} marketing content.
${toneInstructions[params.tone]}.
${formatInstructions[params.format]}.
Use the provided research data to create data-backed, insightful content.
${params.targetAudience ? `Target audience: ${params.targetAudience}` : ''}`;
}

function buildUserPrompt(params: ContentGenerationParams): string {
  const researchSummary = params.researchData
    .map(
      (data, idx) =>
        `[${idx + 1}] ${data.title} (${data.source})\n${data.summary}\nKey insights: ${data.insights.join(', ')}`
    )
    .join('\n\n');

  return `Keyword: ${params.keyword}

Research Data:
${researchSummary}

Create a comprehensive ${params.format} article in ${params.language === 'vi' ? 'Vietnamese' : 'English'} based on this research.`;
}

export interface GeneratedContent {
  title: string;
  body: string;
  excerpt: string;
  tags: string[];
  metadata: {
    wordCount: number;
    readingTime: number;
  };
}

function parseGeneratedContent(
  content: string,
  language: Language
): GeneratedContent {
  // Parse the generated content into structured format
  const lines = content.split('\n');
  const title = lines[0].replace(/^#\s+/, '');
  const body = lines.slice(2).join('\n');

  return {
    title,
    body,
    excerpt: body.slice(0, 200) + '...',
    tags: extractTags(content),
    metadata: {
      wordCount: body.split(/\s+/).length,
      readingTime: Math.ceil(body.split(/\s+/).length / 200),
    },
  };
}

function extractTags(content: string): string[] {
  // Simple tag extraction logic
  return [];
}
```

### 3. OpenAI Integration (Alternative)

```typescript
// lib/ai/openai.ts
import OpenAI from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

export async function generateContentWithOpenAI(
  params: ContentGenerationParams
): Promise<GeneratedContent> {
  const systemPrompt = buildSystemPrompt(params);
  const userPrompt = buildUserPrompt(params);

  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      { role: 'system', content: systemPrompt },
      { role: 'user', content: userPrompt },
    ],
    temperature: 0.7,
    max_tokens: 4000,
  });

  const content = completion.choices[0].message.content;
  return parseGeneratedContent(content, params.language);
}
```

### 4. Bilingual Content Generation

```typescript
// lib/ai/multilingual.ts
export async function generateBilingualContent(
  params: Omit<ContentGenerationParams, 'language'>
): Promise<{ vi: GeneratedContent; en: GeneratedContent }> {
  const [vietnamese, english] = await Promise.all([
    generateContent({ ...params, language: 'vi' }),
    generateContent({ ...params, language: 'en' }),
  ]);

  return { vi: vietnamese, en: english };
}
```

### 5. Video Generation with Remotion

```typescript
// lib/video/render.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

export interface VideoConfig {
  content: GeneratedContent;
  style: 'infographic' | 'slideshow' | 'animated';
  format: 'reels' | 'tiktok' | 'shorts' | 'landscape';
}

export async function renderContentVideo(
  config: VideoConfig
): Promise<string> {
  const bundleLocation = await bundle(
    path.resolve('./remotion/index.ts'),
    () => undefined,
    {
      webpackOverride: (config) => config,
    }
  );

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: config.style,
    inputProps: {
      content: config.content,
      format: config.format,
    },
  });

  const outputLocation = path.join(
    process.cwd(),
    'public',
    'videos',
    `${Date.now()}.mp4`
  );

  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation,
    inputProps: {
      content: config.content,
      format: config.format,
    },
  });

  return outputLocation;
}
```

### 6. Remotion Video Component Example

```tsx
// remotion/compositions/Infographic.tsx
import { AbsoluteFill, useCurrentFrame, useVideoConfig } from 'remotion';
import { GeneratedContent } from '../../lib/ai/claude';

export const Infographic: React.FC<{ content: GeneratedContent }> = ({
  content,
}) => {
  const frame = useCurrentFrame();
  const { fps, durationInFrames } = useVideoConfig();

  const opacity = Math.min(1, frame / 30);

  return (
    <AbsoluteFill
      style={{
        backgroundColor: '#1a1a1a',
        justifyContent: 'center',
        alignItems: 'center',
      }}
    >
      <div style={{ opacity, padding: 60 }}>
        <h1
          style={{
            color: 'white',
            fontSize: 72,
            fontWeight: 'bold',
            textAlign: 'center',
            marginBottom: 40,
          }}
        >
          {content.title}
        </h1>
        <p
          style={{
            color: '#cccccc',
            fontSize: 32,
            textAlign: 'center',
            lineHeight: 1.6,
          }}
        >
          {content.excerpt}
        </p>
      </div>
    </AbsoluteFill>
  );
};
```

## API Routes

### Research API

```typescript
// app/api/research/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { crawlSources } from '@/lib/crawler/research';

export async function POST(request: NextRequest) {
  try {
    const { keyword, sources } = await request.json();

    const researchData = await crawlSources(keyword, sources);

    return NextResponse.json({
      success: true,
      data: researchData,
    });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

### Content Generation API

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { generateContent } from '@/lib/ai/claude';

export async function POST(request: NextRequest) {
  try {
    const params = await request.json();

    const content = await generateContent(params);

    return NextResponse.json({
      success: true,
      content,
    });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

### Video Rendering API

```typescript
// app/api/render/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { renderContentVideo } from '@/lib/video/render';

export async function POST(request: NextRequest) {
  try {
    const config = await request.json();

    const videoPath = await renderContentVideo(config);

    return NextResponse.json({
      success: true,
      videoUrl: `/videos/${path.basename(videoPath)}`,
    });
  } catch (error) {
    return NextResponse.json(
      { success: false, error: error.message },
      { status: 500 }
    );
  }
}
```

## Complete Workflow Example

```typescript
// Example: Complete content pipeline
import { crawlSources } from '@/lib/crawler/research';
import { generateBilingualContent } from '@/lib/ai/multilingual';
import { renderContentVideo } from '@/lib/video/render';

async function runContentPipeline(keyword: string) {
  // Step 1: Research
  console.log('🔍 Researching trending content...');
  const researchData = await crawlSources(keyword, [
    { url: 'techcrunch.com', type: 'techcrunch', timeframe: '24h' },
    { url: 'a16z.com', type: 'a16z', timeframe: '7d' },
    { url: 'twitter.com', type: 'twitter', timeframe: '24h' },
  ]);

  // Step 2: Generate bilingual content
  console.log('✍️ Generating content in Vietnamese & English...');
  const content = await generateBilingualContent({
    keyword,
    researchData,
    format: 'toplist',
    tone: 'professional',
    targetAudience: 'Marketing professionals and content creators',
  });

  // Step 3: Render videos
  console.log('🎬 Rendering videos...');
  const [viVideo, enVideo] = await Promise.all([
    renderContentVideo({
      content: content.vi,
      style: 'infographic',
      format: 'reels',
    }),
    renderContentVideo({
      content: content.en,
      style: 'infographic',
      format: 'shorts',
    }),
  ]);

  return {
    vietnamese: {
      content: content.vi,
      video: viVideo,
    },
    english: {
      content: content.en,
      video: enVideo,
    },
  };
}

// Usage
runContentPipeline('AI Marketing Automation').then((result) => {
  console.log('✅ Pipeline complete!', result);
});
```

## Common Patterns

### Custom Content Templates

```typescript
// lib/templates/custom-format.ts
export interface CustomTemplate {
  name: string;
  structure: string[];
  requiredSections: string[];
}

export const templates: Record<string, CustomTemplate> = {
  'viral-toplist': {
    name: 'Viral Top List',
    structure: [
      'Hook',
      'Introduction',
      'List Items (5-10)',
      'Data/Statistics',
      'Call-to-Action',
    ],
    requiredSections: ['Hook', 'List Items'],
  },
  'thought-leadership': {
    name: 'Thought Leadership POV',
    structure: [
      'Controversial Statement',
      'Personal Experience',
      'Industry Analysis',
      'Future Predictions',
      'Actionable Takeaways',
    ],
    requiredSections: ['Controversial Statement', 'Industry Analysis'],
  },
};
```

### Batch Processing

```typescript
// lib/batch/processor.ts
export async function batchGenerateContent(
  keywords: string[],
  config: Partial<ContentGenerationParams>
) {
  const results = [];

  for (const keyword of keywords) {
    const researchData = await crawlSources(keyword, config.sources);

    const content = await generateContent({
      keyword,
      researchData,
      format: config.format || 'toplist',
      language: config.language || 'vi',
      tone: config.tone || 'professional',
    });

    results.push({ keyword, content });

    // Rate limiting
    await new Promise((resolve) => setTimeout(resolve, 2000));
  }

  return results;
}
```

## Development Commands

```bash
# Run development server
npm run dev

# Build for production
npm run build

# Start production server
npm run start

# Render Remotion video (development)
npm run remotion

# Bundle Remotion compositions
npm run remotion:bundle

# Render specific composition
npx remotion render src/index.ts Infographic out/video.mp4
```

## Troubleshooting

### API Rate Limits

```typescript
// lib/utils/rate-limiter.ts
export class RateLimiter {
  private queue: Array<() => Promise<any>> = [];
  private processing = false;

  async add<T>(fn: () => Promise<T>, delayMs = 1000): Promise<T> {
    return new Promise((resolve, reject) => {
      this.queue.push(async () => {
        try {
          const result = await fn();
          await new Promise((r) => setTimeout(r, delayMs));
          resolve(result);
        } catch (error) {
          reject(error);
        }
      });

      if (!this.processing) {
        this.process();
      }
    });
  }

  private async process() {
    this.processing = true;
    while (this.queue.length > 0) {
      const fn = this.queue.shift();
      if (fn) await fn();
    }
    this.processing = false;
  }
}

// Usage
const limiter = new RateLimiter();
const result = await limiter.add(() => generateContent(params), 2000);
```

### Error Handling

```typescript
// lib/utils/error-handler.ts
export class ContentPipelineError extends Error {
  constructor(
    message: string,
    public stage: 'research' | 'generation' | 'rendering',
    public originalError?: Error
  ) {
    super(message);
    this.name = 'ContentPipelineError';
  }
}

export async function withErrorHandling<T>(
  fn: () => Promise<T>,
  stage: 'research' | 'generation' | 'rendering'
): Promise<T> {
  try {
    return await fn();
  } catch (error) {
    throw new ContentPipelineError(
      `Failed at ${stage} stage: ${error.message}`,
      stage,
      error
    );
  }
}
```

### Memory Management for Large Batches

```typescript
// Implement streaming for large datasets
export async function* streamResearchData(
  keywords: string[]
): AsyncGenerator<ResearchData[]> {
  for (const keyword of keywords) {
    const data = await crawlSources(keyword, defaultSources);
    yield data;
  }
}

// Usage
for await (const data of streamResearchData(keywords)) {
  const content = await generateContent({ researchData: data, ...config });
  // Process immediately to avoid memory buildup
}
```

## Best Practices

1. **Always use environment variables** for API keys
2. **Implement rate limiting** when calling external APIs
3. **Cache research data** to avoid redundant crawling
4. **Use streaming** for large batch operations
5. **Validate input data** before passing to AI models
6. **Monitor API costs** by tracking token usage
7. **Test video rendering locally** before production deployment
8. **Implement retry logic** for failed API calls
