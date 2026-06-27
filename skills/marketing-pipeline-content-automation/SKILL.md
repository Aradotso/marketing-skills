---
name: marketing-pipeline-content-automation
description: AI-powered content automation pipeline for research, script generation, social media posting, and video creation using Claude/OpenAI and Remotion
triggers:
  - automate content creation with AI research
  - generate scripts and videos from keywords
  - set up AI content pipeline with Claude
  - crawl news and create content automatically
  - build automated marketing content system
  - create social media posts with AI pipeline
  - generate videos from articles using Remotion
  - automate content research and publishing
---

# Marketing Pipeline Content Automation

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to work with the Ultimate AI Content Pipeline, a comprehensive system that automates content creation from research through video generation. The pipeline crawls news sources, generates articles in multiple formats and languages, and renders videos automatically.

## What This Project Does

The Marketing Pipeline is an end-to-end content automation system that:

- **Auto-crawls** news from TechCrunch, a16z, Twitter/X, LinkedIn within the last 24 hours
- **Generates content** in multiple formats (Toplist, POV, Case Study, How-to) using Claude/OpenAI
- **Creates bilingual content** (English & Vietnamese) with customizable tone
- **Renders videos** automatically using Remotion for social media platforms
- **Posts to social media** pages automatically

Built with Next.js and TypeScript, integrating Claude 3, OpenAI, and Remotion.

## Installation

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
yarn install
```

### Environment Configuration

Create a `.env.local` file in the root directory:

```bash
# AI Provider Keys
ANTHROPIC_API_KEY=your_claude_api_key_here
OPENAI_API_KEY=your_openai_api_key_here

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key_here

# Social Media Integration (optional)
FACEBOOK_PAGE_ACCESS_TOKEN=your_token_here
FACEBOOK_PAGE_ID=your_page_id_here

# Remotion Video Rendering
REMOTION_LICENSE_KEY=your_remotion_key_here

# Application URL
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

### Development Server

```bash
npm run dev
# or
yarn dev
```

Access the application at `http://localhost:3000`

## Core Architecture

### Pipeline Flow

```typescript
// Typical pipeline execution flow
// 1. Research Phase → 2. Content Generation → 3. Video Rendering → 4. Publishing

import { ResearchEngine } from '@/lib/research';
import { ContentGenerator } from '@/lib/generator';
import { VideoRenderer } from '@/lib/video';
import { Publisher } from '@/lib/publisher';

// Complete pipeline execution
async function runContentPipeline(keyword: string) {
  // Step 1: Research
  const research = await ResearchEngine.scan({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeframe: '24h'
  });
  
  // Step 2: Generate Content
  const content = await ContentGenerator.create({
    research,
    format: 'case-study',
    languages: ['en', 'vi'],
    tone: 'professional'
  });
  
  // Step 3: Render Video
  const video = await VideoRenderer.render({
    content,
    platform: 'tiktok', // or 'reels', 'shorts'
    aspectRatio: '9:16'
  });
  
  // Step 4: Publish
  await Publisher.post({
    content,
    video,
    platforms: ['facebook', 'twitter']
  });
  
  return { content, video };
}
```

## Research Engine

### Auto-Crawling News Sources

```typescript
// lib/research/crawler.ts
import { ResearchEngine } from '@/lib/research';

async function crawlLatestNews(topic: string) {
  const engine = new ResearchEngine({
    apiKey: process.env.RAPIDAPI_KEY
  });
  
  // Crawl multiple sources
  const results = await engine.crawl({
    keyword: topic,
    sources: {
      techcrunch: {
        enabled: true,
        maxArticles: 10
      },
      twitter: {
        enabled: true,
        hashtags: [`#${topic}`, '#AI', '#tech'],
        maxTweets: 50
      },
      linkedin: {
        enabled: true,
        influencers: ['satyanadella', 'jeffweiner'],
        maxPosts: 20
      }
    },
    timeframe: '24h'
  });
  
  return results;
}

// Extract insights from crawled data
async function extractInsights(rawData: CrawlResult[]) {
  const insights = await ResearchEngine.analyze({
    data: rawData,
    extractors: [
      'trends',
      'statistics',
      'quotes',
      'case-studies'
    ]
  });
  
  return insights;
}
```

### Using Research Data

```typescript
// Example: Research API endpoint
// pages/api/research/[keyword].ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { ResearchEngine } from '@/lib/research';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  const { keyword } = req.query;
  
  if (typeof keyword !== 'string') {
    return res.status(400).json({ error: 'Invalid keyword' });
  }
  
  try {
    const research = await ResearchEngine.scan({
      keyword,
      sources: ['techcrunch', 'a16z', 'twitter'],
      timeframe: '24h'
    });
    
    const insights = await ResearchEngine.analyze({
      data: research,
      extractors: ['trends', 'statistics', 'quotes']
    });
    
    res.status(200).json({
      keyword,
      articles: research.articles,
      insights,
      timestamp: new Date().toISOString()
    });
  } catch (error) {
    res.status(500).json({ error: 'Research failed' });
  }
}
```

## Content Generation

### Using Claude/OpenAI for Content Creation

```typescript
// lib/generator/content.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY
});

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY
});

interface ContentOptions {
  research: ResearchData;
  format: 'toplist' | 'pov' | 'case-study' | 'how-to';
  language: 'en' | 'vi';
  tone: 'professional' | 'friendly' | 'humorous';
}

async function generateContentWithClaude(options: ContentOptions) {
  const prompt = buildPrompt(options);
  
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
}

async function generateContentWithOpenAI(options: ContentOptions) {
  const prompt = buildPrompt(options);
  
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [
      {
        role: 'system',
        content: 'You are an expert content creator specializing in marketing content.'
      },
      {
        role: 'user',
        content: prompt
      }
    ],
    temperature: 0.7,
    max_tokens: 4096
  });
  
  return completion.choices[0].message.content || '';
}

function buildPrompt(options: ContentOptions): string {
  const { research, format, language, tone } = options;
  
  return `
Create a ${format} article in ${language} with a ${tone} tone.

Research Data:
${JSON.stringify(research.insights, null, 2)}

Format Guidelines:
${getFormatGuidelines(format)}

Requirements:
- Use data-backed insights from the research
- Include statistics and quotes where relevant
- ${language === 'vi' ? 'Write in Vietnamese' : 'Write in English'}
- Maintain a ${tone} tone throughout
- Optimize for social media sharing
`;
}

function getFormatGuidelines(format: string): string {
  const guidelines = {
    'toplist': 'Create a numbered list article (e.g., "Top 10..."). Each item should have a heading, description, and example.',
    'pov': 'Write from a specific perspective or viewpoint. Start with a strong opinion statement.',
    'case-study': 'Analyze a real example with Introduction, Challenge, Solution, Results structure.',
    'how-to': 'Step-by-step guide format with clear actionable instructions.'
  };
  
  return guidelines[format] || '';
}
```

### Multi-Language Content Generation

```typescript
// lib/generator/bilingual.ts
async function generateBilingualContent(options: Omit<ContentOptions, 'language'>) {
  // Generate both English and Vietnamese versions
  const [englishContent, vietnameseContent] = await Promise.all([
    generateContentWithClaude({ ...options, language: 'en' }),
    generateContentWithClaude({ ...options, language: 'vi' })
  ]);
  
  return {
    en: englishContent,
    vi: vietnameseContent
  };
}

// Usage in API route
// pages/api/generate.ts
export default async function handler(req: NextApiRequest, res: NextApiResponse) {
  const { keyword, format, tone } = req.body;
  
  // Get research data
  const research = await ResearchEngine.scan({ keyword });
  
  // Generate bilingual content
  const content = await generateBilingualContent({
    research,
    format,
    tone
  });
  
  res.status(200).json(content);
}
```

## Video Rendering with Remotion

### Setting Up Remotion Compositions

```typescript
// remotion/compositions.tsx
import { Composition } from 'remotion';
import { ArticleVideo } from './templates/ArticleVideo';
import { ToplistVideo } from './templates/ToplistVideo';

export const RemotionRoot: React.FC = () => {
  return (
    <>
      <Composition
        id="article-video"
        component={ArticleVideo}
        durationInFrames={300}
        fps={30}
        width={1080}
        height={1920}
        defaultProps={{
          title: '',
          content: '',
          insights: []
        }}
      />
      <Composition
        id="toplist-video"
        component={ToplistVideo}
        durationInFrames={450}
        fps={30}
        width={1080}
        height={1920}
        defaultProps={{
          items: []
        }}
      />
    </>
  );
}
```

### Video Template Component

```typescript
// remotion/templates/ArticleVideo.tsx
import { AbsoluteFill, useCurrentFrame, interpolate, Sequence } from 'remotion';

interface ArticleVideoProps {
  title: string;
  content: string;
  insights: string[];
}

export const ArticleVideo: React.FC<ArticleVideoProps> = ({
  title,
  content,
  insights
}) => {
  const frame = useCurrentFrame();
  
  const opacity = interpolate(frame, [0, 30], [0, 1], {
    extrapolateRight: 'clamp'
  });
  
  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a1a' }}>
      {/* Title sequence */}
      <Sequence from={0} durationInFrames={90}>
        <AbsoluteFill
          style={{
            justifyContent: 'center',
            alignItems: 'center',
            opacity
          }}
        >
          <h1 style={{ color: 'white', fontSize: 60, padding: 40, textAlign: 'center' }}>
            {title}
          </h1>
        </AbsoluteFill>
      </Sequence>
      
      {/* Content sequences */}
      {insights.map((insight, index) => (
        <Sequence
          key={index}
          from={90 + index * 60}
          durationInFrames={60}
        >
          <AbsoluteFill
            style={{
              justifyContent: 'center',
              alignItems: 'center',
              padding: 40
            }}
          >
            <p style={{ color: 'white', fontSize: 40, textAlign: 'center' }}>
              {insight}
            </p>
          </AbsoluteFill>
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};
```

### Rendering Videos Programmatically

```typescript
// lib/video/renderer.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface RenderOptions {
  compositionId: string;
  inputProps: any;
  outputPath: string;
}

export async function renderVideo(options: RenderOptions) {
  const { compositionId, inputProps, outputPath } = options;
  
  // Bundle the Remotion project
  const bundleLocation = await bundle(
    path.join(process.cwd(), 'remotion/index.ts')
  );
  
  // Get composition details
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps
  });
  
  // Render the video
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps
  });
  
  return outputPath;
}

// Usage in API route
// pages/api/render-video.ts
export default async function handler(req: NextApiRequest, res: NextApiResponse) {
  const { content, format } = req.body;
  
  const compositionId = format === 'toplist' ? 'toplist-video' : 'article-video';
  const outputPath = path.join(process.cwd(), 'public/videos', `${Date.now()}.mp4`);
  
  try {
    const videoPath = await renderVideo({
      compositionId,
      inputProps: {
        title: content.title,
        content: content.body,
        insights: content.insights
      },
      outputPath
    });
    
    res.status(200).json({ 
      success: true, 
      videoUrl: videoPath.replace(process.cwd() + '/public', '')
    });
  } catch (error) {
    res.status(500).json({ error: 'Video rendering failed' });
  }
}
```

## Publishing to Social Media

```typescript
// lib/publisher/facebook.ts
import axios from 'axios';

interface FacebookPostOptions {
  message: string;
  videoUrl?: string;
  imageUrl?: string;
}

export async function postToFacebookPage(options: FacebookPostOptions) {
  const pageAccessToken = process.env.FACEBOOK_PAGE_ACCESS_TOKEN;
  const pageId = process.env.FACEBOOK_PAGE_ID;
  
  const endpoint = `https://graph.facebook.com/v18.0/${pageId}/feed`;
  
  const postData: any = {
    message: options.message,
    access_token: pageAccessToken
  };
  
  if (options.videoUrl) {
    postData.link = options.videoUrl;
  }
  
  if (options.imageUrl) {
    postData.picture = options.imageUrl;
  }
  
  const response = await axios.post(endpoint, postData);
  
  return response.data;
}

// Complete workflow with scheduling
// lib/publisher/scheduler.ts
interface ScheduledPost {
  content: BilingualContent;
  videoUrl: string;
  publishAt: Date;
  platforms: ('facebook' | 'twitter')[];
}

export async function schedulePost(post: ScheduledPost) {
  // Store in database for scheduled publishing
  const scheduled = await db.scheduledPosts.create({
    data: {
      content: post.content,
      videoUrl: post.videoUrl,
      publishAt: post.publishAt,
      platforms: post.platforms,
      status: 'pending'
    }
  });
  
  return scheduled;
}
```

## Common Patterns

### End-to-End Pipeline with Error Handling

```typescript
// lib/pipeline/executor.ts
import { z } from 'zod';

const PipelineInputSchema = z.object({
  keyword: z.string().min(1),
  format: z.enum(['toplist', 'pov', 'case-study', 'how-to']),
  tone: z.enum(['professional', 'friendly', 'humorous']),
  platforms: z.array(z.enum(['facebook', 'twitter', 'tiktok'])),
  scheduleAt: z.date().optional()
});

type PipelineInput = z.infer<typeof PipelineInputSchema>;

export async function executePipeline(input: PipelineInput) {
  // Validate input
  const validated = PipelineInputSchema.parse(input);
  
  try {
    // Step 1: Research
    console.log(`[Pipeline] Starting research for: ${validated.keyword}`);
    const research = await ResearchEngine.scan({
      keyword: validated.keyword,
      sources: ['techcrunch', 'a16z', 'twitter'],
      timeframe: '24h'
    });
    
    if (!research.articles.length) {
      throw new Error('No research data found');
    }
    
    // Step 2: Generate content
    console.log('[Pipeline] Generating bilingual content');
    const content = await generateBilingualContent({
      research,
      format: validated.format,
      tone: validated.tone
    });
    
    // Step 3: Render video
    console.log('[Pipeline] Rendering video');
    const videoPath = await renderVideo({
      compositionId: `${validated.format}-video`,
      inputProps: {
        title: content.en.title,
        content: content.en.body,
        insights: research.insights
      },
      outputPath: path.join(process.cwd(), 'public/videos', `${Date.now()}.mp4`)
    });
    
    // Step 4: Publish or schedule
    if (validated.scheduleAt) {
      console.log('[Pipeline] Scheduling post');
      await schedulePost({
        content,
        videoUrl: videoPath,
        publishAt: validated.scheduleAt,
        platforms: validated.platforms
      });
    } else {
      console.log('[Pipeline] Publishing immediately');
      for (const platform of validated.platforms) {
        if (platform === 'facebook') {
          await postToFacebookPage({
            message: content.en.body,
            videoUrl: videoPath
          });
        }
        // Add other platforms as needed
      }
    }
    
    console.log('[Pipeline] Completed successfully');
    return {
      success: true,
      content,
      videoPath
    };
    
  } catch (error) {
    console.error('[Pipeline] Error:', error);
    throw error;
  }
}
```

### React Hook for Pipeline Execution

```typescript
// hooks/usePipeline.ts
import { useState } from 'react';
import axios from 'axios';

export function usePipeline() {
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);
  const [result, setResult] = useState<any>(null);
  
  const execute = async (input: PipelineInput) => {
    setLoading(true);
    setError(null);
    
    try {
      const response = await axios.post('/api/pipeline/execute', input);
      setResult(response.data);
      return response.data;
    } catch (err: any) {
      setError(err.response?.data?.error || 'Pipeline execution failed');
      throw err;
    } finally {
      setLoading(false);
    }
  };
  
  return { execute, loading, error, result };
}

// Usage in component
// components/PipelineForm.tsx
export function PipelineForm() {
  const { execute, loading, error } = usePipeline();
  
  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    
    await execute({
      keyword: 'AI automation',
      format: 'case-study',
      tone: 'professional',
      platforms: ['facebook', 'twitter']
    });
  };
  
  return (
    <form onSubmit={handleSubmit}>
      {/* Form fields */}
      <button type="submit" disabled={loading}>
        {loading ? 'Processing...' : 'Generate Content'}
      </button>
      {error && <p style={{ color: 'red' }}>{error}</p>}
    </form>
  );
}
```

## Troubleshooting

### API Rate Limits

```typescript
// lib/utils/rate-limiter.ts
class RateLimiter {
  private requests: number[] = [];
  private maxRequests: number;
  private windowMs: number;
  
  constructor(maxRequests: number, windowMs: number) {
    this.maxRequests = maxRequests;
    this.windowMs = windowMs;
  }
  
  async throttle() {
    const now = Date.now();
    this.requests = this.requests.filter(time => now - time < this.windowMs);
    
    if (this.requests.length >= this.maxRequests) {
      const oldestRequest = this.requests[0];
      const waitTime = this.windowMs - (now - oldestRequest);
      await new Promise(resolve => setTimeout(resolve, waitTime));
    }
    
    this.requests.push(now);
  }
}

// Usage
const anthropicLimiter = new RateLimiter(50, 60000); // 50 requests per minute

async function generateContentWithRateLimit(options: ContentOptions) {
  await anthropicLimiter.throttle();
  return generateContentWithClaude(options);
}
```

### Video Rendering Memory Issues

```typescript
// For large videos, use chunked rendering
import { renderFrames } from '@remotion/renderer';

async function renderVideoInChunks(options: RenderOptions) {
  const chunkSize = 150; // frames per chunk
  const totalFrames = 300;
  
  for (let i = 0; i < totalFrames; i += chunkSize) {
    await renderFrames({
      ...options,
      frameRange: [i, Math.min(i + chunkSize, totalFrames)]
    });
  }
}
```

### Debugging Content Quality

```typescript
// lib/generator/validator.ts
export function validateContent(content: string): {
  valid: boolean;
  issues: string[];
} {
  const issues: string[] = [];
  
  // Check minimum length
  if (content.length < 500) {
    issues.push('Content too short (minimum 500 characters)');
  }
  
  // Check for placeholder text
  if (content.includes('[INSERT') || content.includes('TODO')) {
    issues.push('Content contains placeholders');
  }
  
  // Check for proper formatting
  if (!content.includes('\n\n')) {
    issues.push('Content lacks paragraph breaks');
  }
  
  return {
    valid: issues.length === 0,
    issues
  };
}

// Use in generation pipeline
const generatedContent = await generateContentWithClaude(options);
const validation = validateContent(generatedContent);

if (!validation.valid) {
  console.warn('Content quality issues:', validation.issues);
  // Regenerate or apply fixes
}
```

### Environment Variable Checks

```typescript
// lib/config/validate-env.ts
export function validateEnvironment() {
  const required = [
    'ANTHROPIC_API_KEY',
    'OPENAI_API_KEY',
    'RAPIDAPI_KEY'
  ];
  
  const missing = required.filter(key => !process.env[key]);
  
  if (missing.length > 0) {
    throw new Error(
      `Missing required environment variables: ${missing.join(', ')}\n` +
      `Please check your .env.local file.`
    );
  }
}

// Call at startup
// pages/_app.tsx
validateEnvironment();
```

## Performance Optimization

```typescript
// Parallel execution for faster pipeline
async function executeOptimizedPipeline(input: PipelineInput) {
  const research = await ResearchEngine.scan({
    keyword: input.keyword,
    sources: ['techcrunch', 'a16z', 'twitter'],
    timeframe: '24h'
  });
  
  // Generate both language versions in parallel
  const [contentEn, contentVi, insights] = await Promise.all([
    generateContentWithClaude({ ...input, language: 'en', research }),
    generateContentWithClaude({ ...input, language: 'vi', research }),
    ResearchEngine.analyze({ data: research, extractors: ['trends', 'statistics'] })
  ]);
  
  // Render video after content is ready
  const videoPath = await renderVideo({
    compositionId: `${input.format}-video`,
    inputProps: { 
      title: contentEn.title, 
      content: contentEn.body,
      insights 
    },
    outputPath: path.join(process.cwd(), 'public/videos', `${Date.now()}.mp4`)
  });
  
  return {
    content: { en: contentEn, vi: contentVi },
    videoPath,
    insights
  };
}
```

This skill enables comprehensive automation of content marketing workflows, from research through publication, leveraging cutting-edge AI and video rendering technologies.
