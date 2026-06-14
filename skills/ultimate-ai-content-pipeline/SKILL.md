---
name: ultimate-ai-content-pipeline
description: Automated content creation pipeline from research to video generation using Claude, OpenAI, and Remotion
triggers:
  - automate content creation with AI research and video generation
  - set up content pipeline with Claude and Remotion
  - build automated marketing content system
  - create AI-powered content workflow from research to video
  - configure content automation with OpenAI and video rendering
  - generate automated content with research and social media videos
  - build end-to-end content creation pipeline
  - set up multi-format AI content generation system
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

A comprehensive TypeScript-based content automation system that handles the complete content lifecycle: automated research from news sources (TechCrunch, a16z, Twitter/X, LinkedIn), AI-powered content generation in multiple formats (Claude/OpenAI), and automatic video rendering (Remotion). Perfect for marketers and content creators looking to automate 90% of their content workflow.

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

Create a `.env.local` file in the project root:

```bash
# AI Services
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# Research APIs
RAPIDAPI_KEY=your_rapidapi_key

# Optional: Content Sources
TWITTER_BEARER_TOKEN=your_twitter_token
LINKEDIN_API_KEY=your_linkedin_key

# Remotion (Video Rendering)
REMOTION_LICENSE_KEY=your_remotion_key

# Database (if applicable)
DATABASE_URL=your_database_url

# Next.js
NEXT_PUBLIC_APP_URL=http://localhost:3000
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

## Core Components

### 1. Research & Content Scraping

The system automatically crawls recent news from multiple sources:

```typescript
// lib/research/crawler.ts
import axios from 'axios';

interface ResearchConfig {
  sources: string[];
  keywords: string[];
  timeframe: number; // hours
}

export async function performResearch(config: ResearchConfig) {
  const results = await Promise.all([
    fetchTechCrunch(config.keywords, config.timeframe),
    fetchA16z(config.keywords),
    fetchTwitter(config.keywords, config.timeframe),
    fetchLinkedIn(config.keywords, config.timeframe)
  ]);
  
  return aggregateResults(results);
}

async function fetchTechCrunch(keywords: string[], hours: number) {
  const response = await axios.get('https://techcrunch.com/wp-json/wp/v2/posts', {
    params: {
      search: keywords.join(' '),
      per_page: 10,
      after: new Date(Date.now() - hours * 60 * 60 * 1000).toISOString()
    }
  });
  
  return response.data.map((post: any) => ({
    title: post.title.rendered,
    content: post.excerpt.rendered,
    url: post.link,
    source: 'TechCrunch',
    publishedAt: post.date
  }));
}

async function fetchTwitter(keywords: string[], hours: number) {
  const query = keywords.join(' OR ');
  const response = await axios.get('https://api.twitter.com/2/tweets/search/recent', {
    headers: {
      'Authorization': `Bearer ${process.env.TWITTER_BEARER_TOKEN}`
    },
    params: {
      query,
      max_results: 100,
      'tweet.fields': 'created_at,public_metrics,entities'
    }
  });
  
  return response.data.data || [];
}
```

### 2. AI Content Generation

Generate content in multiple formats using Claude or OpenAI:

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

export type ContentFormat = 'toplist' | 'pov' | 'case-study' | 'how-to';
export type ToneOfVoice = 'expert' | 'friendly' | 'humorous';
export type Language = 'en' | 'vi';

interface GenerationConfig {
  researchData: any[];
  format: ContentFormat;
  tone: ToneOfVoice;
  language: Language;
  keywords: string[];
}

export async function generateContent(config: GenerationConfig, provider: 'claude' | 'openai' = 'claude') {
  const prompt = buildPrompt(config);
  
  if (provider === 'claude') {
    return generateWithClaude(prompt);
  } else {
    return generateWithOpenAI(prompt);
  }
}

async function generateWithClaude(prompt: string) {
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [{
      role: 'user',
      content: prompt
    }]
  });
  
  return message.content[0].type === 'text' 
    ? message.content[0].text 
    : '';
}

async function generateWithOpenAI(prompt: string) {
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [{
      role: 'user',
      content: prompt
    }],
    max_tokens: 4096
  });
  
  return completion.choices[0].message.content || '';
}

function buildPrompt(config: GenerationConfig): string {
  const formatInstructions = {
    'toplist': 'Create a ranked list article with compelling reasons for each item',
    'pov': 'Write from a unique perspective with personal insights and opinions',
    'case-study': 'Analyze a real-world example with data, challenges, and outcomes',
    'how-to': 'Create a step-by-step tutorial with actionable instructions'
  };
  
  const toneInstructions = {
    'expert': 'Use professional, authoritative language with industry jargon',
    'friendly': 'Write conversationally with warmth and relatability',
    'humorous': 'Include wit, humor, and entertaining observations'
  };
  
  const languageInstructions = {
    'en': 'Write in English',
    'vi': 'Write in Vietnamese'
  };
  
  return `
You are an expert content creator. Using the following research data, create a ${config.format} article.

RESEARCH DATA:
${JSON.stringify(config.researchData, null, 2)}

REQUIREMENTS:
- Format: ${formatInstructions[config.format]}
- Tone: ${toneInstructions[config.tone]}
- Language: ${languageInstructions[config.language]}
- Keywords to include: ${config.keywords.join(', ')}
- Include data points and statistics from the research
- Make it engaging and shareable on social media
- Length: 800-1200 words

OUTPUT FORMAT:
Return a JSON object with:
{
  "title": "Compelling headline",
  "subtitle": "Engaging subtitle",
  "content": "Full article content in markdown",
  "meta": {
    "readTime": "estimated read time in minutes",
    "keyTakeaways": ["takeaway 1", "takeaway 2", "takeaway 3"]
  },
  "socialSnippets": {
    "twitter": "140 char version",
    "linkedin": "Brief professional summary"
  }
}
`;
}
```

### 3. Multi-Language Content Generation

Generate bilingual content automatically:

```typescript
// lib/ai/multi-language.ts
export async function generateBilingualContent(
  researchData: any[],
  format: ContentFormat,
  tone: ToneOfVoice
) {
  const [englishContent, vietnameseContent] = await Promise.all([
    generateContent({
      researchData,
      format,
      tone,
      language: 'en',
      keywords: []
    }),
    generateContent({
      researchData,
      format,
      tone,
      language: 'vi',
      keywords: []
    })
  ]);
  
  return {
    en: JSON.parse(englishContent),
    vi: JSON.parse(vietnameseContent)
  };
}
```

### 4. Video Generation with Remotion

Automatically create videos from content:

```typescript
// remotion/compositions/ContentVideo.tsx
import { AbsoluteFill, Sequence, useCurrentFrame, useVideoConfig } from 'remotion';
import { z } from 'zod';

export const contentVideoSchema = z.object({
  title: z.string(),
  keyPoints: z.array(z.string()),
  brandColor: z.string(),
  logo: z.string().optional()
});

export const ContentVideo: React.FC<z.infer<typeof contentVideoSchema>> = ({
  title,
  keyPoints,
  brandColor,
  logo
}) => {
  const frame = useCurrentFrame();
  const { fps, durationInFrames } = useVideoConfig();
  
  const titleDuration = fps * 3; // 3 seconds
  const pointDuration = fps * 4; // 4 seconds per point
  
  return (
    <AbsoluteFill style={{ backgroundColor: '#000' }}>
      <Sequence durationInFrames={titleDuration}>
        <TitleScene title={title} brandColor={brandColor} logo={logo} />
      </Sequence>
      
      {keyPoints.map((point, index) => (
        <Sequence
          key={index}
          from={titleDuration + index * pointDuration}
          durationInFrames={pointDuration}
        >
          <PointScene 
            point={point} 
            index={index + 1}
            brandColor={brandColor}
          />
        </Sequence>
      ))}
      
      <Sequence from={durationInFrames - fps * 2}>
        <OutroScene brandColor={brandColor} logo={logo} />
      </Sequence>
    </AbsoluteFill>
  );
};

// Render video programmatically
// lib/video/render.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

export async function renderContentVideo(contentData: {
  title: string;
  keyPoints: string[];
  brandColor?: string;
}) {
  const bundleLocation = await bundle({
    entryPoint: path.join(process.cwd(), 'remotion/index.ts'),
    webpackOverride: (config) => config,
  });

  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title: contentData.title,
      keyPoints: contentData.keyPoints,
      brandColor: contentData.brandColor || '#3B82F6',
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
    inputProps: composition.defaultProps,
  });

  return outputLocation;
}
```

### 5. Complete Content Pipeline

Orchestrate the entire workflow:

```typescript
// lib/pipeline/orchestrator.ts
export interface PipelineConfig {
  keyword: string;
  researchSources: string[];
  contentFormat: ContentFormat;
  tone: ToneOfVoice;
  generateVideo: boolean;
  languages: Language[];
}

export async function runContentPipeline(config: PipelineConfig) {
  console.log('🔍 Starting research phase...');
  const researchData = await performResearch({
    sources: config.researchSources,
    keywords: [config.keyword],
    timeframe: 24
  });
  
  console.log(`📊 Found ${researchData.length} relevant sources`);
  
  console.log('✍️ Generating content...');
  const contentResults: Record<string, any> = {};
  
  for (const language of config.languages) {
    const content = await generateContent({
      researchData,
      format: config.contentFormat,
      tone: config.tone,
      language,
      keywords: [config.keyword]
    });
    
    contentResults[language] = JSON.parse(content);
  }
  
  let videoPath: string | null = null;
  
  if (config.generateVideo) {
    console.log('🎬 Rendering video...');
    const primaryContent = contentResults[config.languages[0]];
    
    videoPath = await renderContentVideo({
      title: primaryContent.title,
      keyPoints: primaryContent.meta.keyTakeaways,
    });
    
    console.log(`✅ Video rendered: ${videoPath}`);
  }
  
  return {
    research: researchData,
    content: contentResults,
    video: videoPath,
    generatedAt: new Date().toISOString()
  };
}
```

### 6. API Route Example (Next.js)

```typescript
// app/api/generate/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline/orchestrator';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    
    const result = await runContentPipeline({
      keyword: body.keyword,
      researchSources: body.sources || ['techcrunch', 'a16z', 'twitter'],
      contentFormat: body.format || 'toplist',
      tone: body.tone || 'expert',
      generateVideo: body.generateVideo ?? true,
      languages: body.languages || ['en', 'vi']
    });
    
    return NextResponse.json(result);
  } catch (error: any) {
    return NextResponse.json(
      { error: error.message },
      { status: 500 }
    );
  }
}
```

### 7. Frontend Usage Example

```typescript
// components/ContentGenerator.tsx
'use client';

import { useState } from 'react';

export default function ContentGenerator() {
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState<any>(null);

  async function handleGenerate(e: React.FormEvent<HTMLFormElement>) {
    e.preventDefault();
    setLoading(true);
    
    const formData = new FormData(e.currentTarget);
    
    const response = await fetch('/api/generate', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        keyword: formData.get('keyword'),
        format: formData.get('format'),
        tone: formData.get('tone'),
        generateVideo: formData.get('generateVideo') === 'on',
        languages: ['en', 'vi']
      })
    });
    
    const data = await response.json();
    setResult(data);
    setLoading(false);
  }

  return (
    <div className="max-w-4xl mx-auto p-6">
      <form onSubmit={handleGenerate} className="space-y-4">
        <input
          name="keyword"
          placeholder="Enter keyword (e.g., AI marketing)"
          className="w-full p-3 border rounded"
          required
        />
        
        <select name="format" className="w-full p-3 border rounded">
          <option value="toplist">Top List</option>
          <option value="pov">POV Article</option>
          <option value="case-study">Case Study</option>
          <option value="how-to">How-to Guide</option>
        </select>
        
        <select name="tone" className="w-full p-3 border rounded">
          <option value="expert">Expert</option>
          <option value="friendly">Friendly</option>
          <option value="humorous">Humorous</option>
        </select>
        
        <label className="flex items-center gap-2">
          <input type="checkbox" name="generateVideo" defaultChecked />
          Generate video
        </label>
        
        <button
          type="submit"
          disabled={loading}
          className="w-full bg-blue-600 text-white p-3 rounded font-semibold"
        >
          {loading ? 'Generating...' : 'Generate Content'}
        </button>
      </form>
      
      {result && (
        <div className="mt-8 space-y-6">
          <h2 className="text-2xl font-bold">Results</h2>
          
          {Object.entries(result.content).map(([lang, content]: [string, any]) => (
            <div key={lang} className="border p-4 rounded">
              <h3 className="font-bold text-lg mb-2">
                {lang === 'en' ? '🇬🇧 English' : '🇻🇳 Vietnamese'}
              </h3>
              <h4 className="text-xl mb-2">{content.title}</h4>
              <div className="prose" dangerouslySetInnerHTML={{ __html: content.content }} />
            </div>
          ))}
          
          {result.video && (
            <div className="border p-4 rounded">
              <h3 className="font-bold text-lg mb-2">🎬 Generated Video</h3>
              <video src={result.video} controls className="w-full" />
            </div>
          )}
        </div>
      )}
    </div>
  );
}
```

## Common Patterns

### Batch Content Generation

```typescript
async function generateBatchContent(keywords: string[]) {
  const results = [];
  
  for (const keyword of keywords) {
    const result = await runContentPipeline({
      keyword,
      researchSources: ['techcrunch', 'a16z'],
      contentFormat: 'toplist',
      tone: 'expert',
      generateVideo: false,
      languages: ['en']
    });
    
    results.push(result);
    
    // Rate limiting
    await new Promise(resolve => setTimeout(resolve, 2000));
  }
  
  return results;
}
```

### Custom Video Templates

```typescript
// remotion/compositions/CustomTemplate.tsx
export const CustomBrandedVideo: React.FC<any> = ({ content, brand }) => {
  return (
    <AbsoluteFill style={{ 
      backgroundColor: brand.primaryColor,
      fontFamily: brand.fontFamily 
    }}>
      {/* Custom branded video layout */}
    </AbsoluteFill>
  );
};
```

## Troubleshooting

### API Rate Limits
- Implement exponential backoff for API calls
- Use caching for research data to avoid repeated requests
- Consider upgrading API plans for higher limits

### Video Rendering Issues
- Ensure sufficient memory (4GB+ recommended)
- Use `npx remotion lambda` for cloud rendering if local fails
- Check Remotion license key is valid

### Content Quality
- Fine-tune prompts for better AI outputs
- Provide more context in research phase
- Use Claude 3.5 Sonnet for best results with complex formats

### Performance Optimization
- Cache research results with Redis or similar
- Use queue systems (Bull, BeeQueue) for batch processing
- Implement background jobs for video rendering
