---
name: ultimate-ai-content-pipeline
description: Automated content creation pipeline from research to video generation using AI (Claude/OpenAI) and Remotion
triggers:
  - how do I automate content creation with AI research and video generation
  - set up an AI content pipeline with Claude and OpenAI
  - create automated marketing content with research and video rendering
  - build a content automation system with AI and Remotion
  - generate videos from AI-written content automatically
  - automate content research and video creation pipeline
  - use ultimate AI content pipeline for marketing automation
  - integrate Claude and OpenAI for automated content generation
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

Ultimate AI Content Pipeline is a comprehensive TypeScript-based content automation system that handles the entire content creation workflow: from automated research/crawling, AI-powered content generation (Claude 3, OpenAI), to automatic video rendering (Remotion). It crawls news from sources like TechCrunch, a16z, Twitter/X, and LinkedIn to create data-backed content in multiple formats and languages.

## Installation

```bash
# Clone the repository
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
pnpm install
# or
yarn install
```

## Environment Configuration

Create a `.env.local` file in the root directory:

```bash
# AI Providers
ANTHROPIC_API_KEY=your_claude_api_key
OPENAI_API_KEY=your_openai_api_key

# Research/Crawling APIs
RAPIDAPI_KEY=your_rapidapi_key

# Remotion (Video Rendering)
REMOTION_LICENSE_KEY=your_remotion_license_key

# Next.js Configuration
NEXT_PUBLIC_API_URL=http://localhost:3000
```

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app directory
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── crawler/     # Content research/crawling
│   │   ├── content/     # Content generation logic
│   │   └── video/       # Remotion video generation
│   └── utils/           # Utility functions
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Core Features & Usage

### 1. Automated Research/Crawling

The system automatically crawls and analyzes content from multiple sources:

```typescript
// src/lib/crawler/research.ts
import { RapidAPIClient } from './rapidapi-client';

interface ResearchOptions {
  keyword: string;
  sources: ('techcrunch' | 'twitter' | 'linkedin' | 'a16z')[];
  timeRange: '24h' | '7d' | '30d';
  language?: 'en' | 'vi';
}

export async function conductResearch(options: ResearchOptions) {
  const client = new RapidAPIClient(process.env.RAPIDAPI_KEY!);
  
  const results = await Promise.all(
    options.sources.map(source => 
      client.searchContent({
        source,
        keyword: options.keyword,
        timeRange: options.timeRange
      })
    )
  );
  
  // Aggregate and analyze results
  const insights = analyzeResearchData(results.flat());
  
  return {
    rawData: results,
    insights,
    metadata: {
      keyword: options.keyword,
      sources: options.sources,
      timestamp: new Date().toISOString()
    }
  };
}

function analyzeResearchData(data: any[]) {
  // Extract key insights, trends, and data points
  return {
    trends: extractTrends(data),
    statistics: extractStatistics(data),
    quotes: extractQuotes(data),
    keyTopics: identifyKeyTopics(data)
  };
}
```

### 2. AI Content Generation

Generate content in multiple formats using Claude or OpenAI:

```typescript
// src/lib/ai/content-generator.ts
import Anthropic from '@anthropic-ai/sdk';
import OpenAI from 'openai';

type ContentFormat = 'toplist' | 'pov' | 'case-study' | 'how-to';
type ToneOfVoice = 'expert' | 'friendly' | 'humorous';

interface ContentGenerationOptions {
  researchData: any;
  format: ContentFormat;
  tone: ToneOfVoice;
  language: 'en' | 'vi';
  provider: 'claude' | 'openai';
}

export async function generateContent(options: ContentGenerationOptions) {
  const prompt = buildContentPrompt(options);
  
  if (options.provider === 'claude') {
    return await generateWithClaude(prompt, options);
  } else {
    return await generateWithOpenAI(prompt, options);
  }
}

async function generateWithClaude(prompt: string, options: ContentGenerationOptions) {
  const anthropic = new Anthropic({
    apiKey: process.env.ANTHROPIC_API_KEY!
  });
  
  const message = await anthropic.messages.create({
    model: 'claude-3-5-sonnet-20241022',
    max_tokens: 4096,
    messages: [{
      role: 'user',
      content: prompt
    }]
  });
  
  return {
    content: message.content[0].text,
    metadata: {
      model: 'claude-3-5-sonnet',
      tokens: message.usage.input_tokens + message.usage.output_tokens,
      format: options.format,
      language: options.language
    }
  };
}

async function generateWithOpenAI(prompt: string, options: ContentGenerationOptions) {
  const openai = new OpenAI({
    apiKey: process.env.OPENAI_API_KEY!
  });
  
  const completion = await openai.chat.completions.create({
    model: 'gpt-4-turbo-preview',
    messages: [{
      role: 'user',
      content: prompt
    }],
    temperature: 0.7,
    max_tokens: 4096
  });
  
  return {
    content: completion.choices[0].message.content,
    metadata: {
      model: completion.model,
      tokens: completion.usage?.total_tokens,
      format: options.format,
      language: options.language
    }
  };
}

function buildContentPrompt(options: ContentGenerationOptions): string {
  const { researchData, format, tone, language } = options;
  
  const formatInstructions = {
    'toplist': 'Create a top 10 list article',
    'pov': 'Write from a unique point of view perspective',
    'case-study': 'Develop a detailed case study',
    'how-to': 'Write a step-by-step how-to guide'
  };
  
  const toneInstructions = {
    'expert': 'authoritative and professional',
    'friendly': 'conversational and approachable',
    'humorous': 'engaging with light humor'
  };
  
  return `
You are a professional content creator. ${formatInstructions[format]} based on the following research data.

Tone: ${toneInstructions[tone]}
Language: ${language === 'en' ? 'English' : 'Vietnamese'}

Research Data:
${JSON.stringify(researchData.insights, null, 2)}

Requirements:
1. Use data and statistics from the research
2. Include relevant quotes and examples
3. Structure with clear headings and subheadings
4. Make it engaging and actionable
5. Include a compelling introduction and conclusion

Generate the content now:
`;
}
```

### 3. Video Generation with Remotion

Automatically render videos from generated content:

```typescript
// src/lib/video/video-generator.ts
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import path from 'path';

interface VideoGenerationOptions {
  content: string;
  template: 'infographic' | 'reels' | 'shorts' | 'tiktok';
  aspectRatio: '16:9' | '9:16' | '1:1';
  outputPath: string;
}

export async function generateVideo(options: VideoGenerationOptions) {
  const { content, template, aspectRatio, outputPath } = options;
  
  // Parse content into video segments
  const segments = parseContentToSegments(content);
  
  // Bundle Remotion composition
  const bundleLocation = await bundle({
    entryPoint: path.join(process.cwd(), 'remotion', 'index.ts'),
    webpackOverride: (config) => config,
  });
  
  // Select composition based on template
  const compositionId = getCompositionId(template);
  
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: compositionId,
    inputProps: {
      segments,
      aspectRatio,
      theme: getTheme(template)
    }
  });
  
  // Render video
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: outputPath,
    inputProps: {
      segments,
      aspectRatio,
      theme: getTheme(template)
    }
  });
  
  return {
    success: true,
    outputPath,
    duration: composition.durationInFrames / composition.fps,
    metadata: {
      template,
      aspectRatio,
      segments: segments.length
    }
  };
}

function parseContentToSegments(content: string) {
  // Parse markdown/structured content into video segments
  const lines = content.split('\n').filter(line => line.trim());
  const segments = [];
  
  for (const line of lines) {
    if (line.startsWith('# ')) {
      segments.push({
        type: 'title',
        text: line.replace('# ', ''),
        duration: 3
      });
    } else if (line.startsWith('## ')) {
      segments.push({
        type: 'subtitle',
        text: line.replace('## ', ''),
        duration: 2
      });
    } else if (line.length > 50) {
      segments.push({
        type: 'content',
        text: line,
        duration: 4
      });
    }
  }
  
  return segments;
}

function getCompositionId(template: string): string {
  const templates = {
    'infographic': 'Infographic',
    'reels': 'Reels',
    'shorts': 'Shorts',
    'tiktok': 'TikTok'
  };
  return templates[template] || 'Default';
}

function getTheme(template: string) {
  return {
    primaryColor: '#3B82F6',
    secondaryColor: '#10B981',
    backgroundColor: '#1F2937',
    font: 'Inter'
  };
}
```

### 4. Remotion Video Component Example

```typescript
// remotion/compositions/Infographic.tsx
import { AbsoluteFill, Sequence, useCurrentFrame, useVideoConfig } from 'remotion';
import React from 'react';

interface InfographicProps {
  segments: Array<{
    type: string;
    text: string;
    duration: number;
  }>;
  theme: {
    primaryColor: string;
    secondaryColor: string;
    backgroundColor: string;
    font: string;
  };
}

export const Infographic: React.FC<InfographicProps> = ({ segments, theme }) => {
  const { fps } = useVideoConfig();
  let currentFrame = 0;
  
  return (
    <AbsoluteFill style={{ backgroundColor: theme.backgroundColor }}>
      {segments.map((segment, index) => {
        const from = currentFrame;
        const durationInFrames = segment.duration * fps;
        currentFrame += durationInFrames;
        
        return (
          <Sequence
            key={index}
            from={from}
            durationInFrames={durationInFrames}
          >
            <SegmentComponent segment={segment} theme={theme} />
          </Sequence>
        );
      })}
    </AbsoluteFill>
  );
};

const SegmentComponent: React.FC<{
  segment: any;
  theme: any;
}> = ({ segment, theme }) => {
  const frame = useCurrentFrame();
  const opacity = Math.min(1, frame / 15);
  
  const styles = {
    title: {
      fontSize: 72,
      fontWeight: 'bold',
      color: theme.primaryColor
    },
    subtitle: {
      fontSize: 48,
      color: theme.secondaryColor
    },
    content: {
      fontSize: 36,
      color: '#FFFFFF',
      lineHeight: 1.5
    }
  };
  
  return (
    <AbsoluteFill
      style={{
        justifyContent: 'center',
        alignItems: 'center',
        padding: 60,
        opacity
      }}
    >
      <div style={styles[segment.type] || styles.content}>
        {segment.text}
      </div>
    </AbsoluteFill>
  );
};
```

### 5. Complete Pipeline Integration

```typescript
// src/lib/pipeline/content-pipeline.ts
import { conductResearch } from '../crawler/research';
import { generateContent } from '../ai/content-generator';
import { generateVideo } from '../video/video-generator';

interface PipelineOptions {
  keyword: string;
  contentFormat: 'toplist' | 'pov' | 'case-study' | 'how-to';
  tone: 'expert' | 'friendly' | 'humorous';
  languages: ('en' | 'vi')[];
  videoTemplate?: 'infographic' | 'reels' | 'shorts' | 'tiktok';
  generateVideo: boolean;
}

export async function runContentPipeline(options: PipelineOptions) {
  console.log('Starting content pipeline for:', options.keyword);
  
  // Step 1: Research
  console.log('Phase 1: Conducting research...');
  const researchData = await conductResearch({
    keyword: options.keyword,
    sources: ['techcrunch', 'twitter', 'linkedin', 'a16z'],
    timeRange: '24h'
  });
  
  // Step 2: Generate content for each language
  console.log('Phase 2: Generating content...');
  const contentResults = await Promise.all(
    options.languages.map(async (language) => {
      const content = await generateContent({
        researchData,
        format: options.contentFormat,
        tone: options.tone,
        language,
        provider: 'claude' // or 'openai'
      });
      
      return { language, ...content };
    })
  );
  
  // Step 3: Generate videos if requested
  let videoResults = null;
  if (options.generateVideo && options.videoTemplate) {
    console.log('Phase 3: Generating videos...');
    videoResults = await Promise.all(
      contentResults.map(async (result, index) => {
        const outputPath = `./output/video_${result.language}_${Date.now()}.mp4`;
        
        return await generateVideo({
          content: result.content,
          template: options.videoTemplate!,
          aspectRatio: options.videoTemplate === 'infographic' ? '16:9' : '9:16',
          outputPath
        });
      })
    );
  }
  
  console.log('Pipeline completed successfully!');
  
  return {
    success: true,
    research: researchData,
    content: contentResults,
    videos: videoResults,
    summary: {
      keyword: options.keyword,
      contentPieces: contentResults.length,
      videosGenerated: videoResults?.length || 0,
      timestamp: new Date().toISOString()
    }
  };
}
```

### 6. Next.js API Route Example

```typescript
// src/app/api/pipeline/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline/content-pipeline';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    
    const {
      keyword,
      contentFormat = 'toplist',
      tone = 'expert',
      languages = ['en', 'vi'],
      videoTemplate,
      generateVideo = false
    } = body;
    
    if (!keyword) {
      return NextResponse.json(
        { error: 'Keyword is required' },
        { status: 400 }
      );
    }
    
    const result = await runContentPipeline({
      keyword,
      contentFormat,
      tone,
      languages,
      videoTemplate,
      generateVideo
    });
    
    return NextResponse.json(result);
    
  } catch (error) {
    console.error('Pipeline error:', error);
    return NextResponse.json(
      { error: 'Pipeline execution failed', details: error.message },
      { status: 500 }
    );
  }
}
```

### 7. Frontend Usage Example

```typescript
// src/components/PipelineForm.tsx
'use client';

import { useState } from 'react';

export default function PipelineForm() {
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState(null);
  
  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    setLoading(true);
    
    const formData = new FormData(e.target as HTMLFormElement);
    
    const response = await fetch('/api/pipeline', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        keyword: formData.get('keyword'),
        contentFormat: formData.get('format'),
        tone: formData.get('tone'),
        languages: ['en', 'vi'],
        videoTemplate: formData.get('videoTemplate'),
        generateVideo: formData.get('generateVideo') === 'on'
      })
    });
    
    const data = await response.json();
    setResult(data);
    setLoading(false);
  };
  
  return (
    <form onSubmit={handleSubmit} className="space-y-4">
      <input
        name="keyword"
        placeholder="Enter keyword"
        required
        className="w-full p-2 border rounded"
      />
      
      <select name="format" className="w-full p-2 border rounded">
        <option value="toplist">Top List</option>
        <option value="pov">POV</option>
        <option value="case-study">Case Study</option>
        <option value="how-to">How-to</option>
      </select>
      
      <select name="tone" className="w-full p-2 border rounded">
        <option value="expert">Expert</option>
        <option value="friendly">Friendly</option>
        <option value="humorous">Humorous</option>
      </select>
      
      <label className="flex items-center gap-2">
        <input type="checkbox" name="generateVideo" />
        Generate Video
      </label>
      
      <select name="videoTemplate" className="w-full p-2 border rounded">
        <option value="infographic">Infographic</option>
        <option value="reels">Reels</option>
        <option value="shorts">Shorts</option>
        <option value="tiktok">TikTok</option>
      </select>
      
      <button
        type="submit"
        disabled={loading}
        className="w-full bg-blue-600 text-white p-2 rounded"
      >
        {loading ? 'Processing...' : 'Start Pipeline'}
      </button>
      
      {result && (
        <div className="mt-4 p-4 bg-gray-100 rounded">
          <pre>{JSON.stringify(result, null, 2)}</pre>
        </div>
      )}
    </form>
  );
}
```

## Development Commands

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Type checking
npm run type-check

# Lint code
npm run lint

# Run tests
npm test
```

## Troubleshooting

### API Key Issues
- Ensure all API keys are properly set in `.env.local`
- Verify API key permissions and quotas
- Check if keys are correctly referenced without quotes

### Crawling/Research Errors
- Verify RapidAPI subscription includes required endpoints
- Check rate limits on news APIs
- Ensure source URLs are accessible

### AI Generation Failures
- Verify Claude/OpenAI API keys have sufficient credits
- Check model availability (some models require waitlist access)
- Reduce max_tokens if hitting quota limits
- Handle rate limiting with exponential backoff

### Video Rendering Issues
- Ensure Remotion license key is valid
- Check ffmpeg is installed: `ffmpeg -version`
- Verify sufficient disk space for video output
- Review Remotion logs for specific error messages

### Memory Issues
- Increase Node.js memory: `NODE_OPTIONS=--max-old-space-size=4096 npm run dev`
- Process videos sequentially instead of parallel for large batches
- Clear output directory regularly

### TypeScript Errors
```bash
# Clean and reinstall dependencies
rm -rf node_modules package-lock.json
npm install

# Regenerate types
npm run type-check
```

## Best Practices

1. **Rate Limiting**: Implement delays between API calls to avoid hitting rate limits
2. **Error Handling**: Always wrap API calls in try-catch blocks
3. **Caching**: Cache research results to avoid redundant API calls
4. **Monitoring**: Log all pipeline executions for debugging
5. **Testing**: Test with small datasets before running full pipeline
6. **Resource Management**: Clean up temporary files and video outputs regularly
