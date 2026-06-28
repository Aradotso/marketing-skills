---
name: ultimate-ai-content-pipeline
description: Automated content pipeline from research to video generation using Claude/OpenAI and Remotion
triggers:
  - how do I automate content creation with AI research
  - set up automated video generation from text content
  - create content pipeline with Claude and OpenAI
  - how to use Remotion for automated video rendering
  - build AI-powered content automation system
  - generate videos from blog posts automatically
  - automate content research and scriptwriting
  - create multilingual content with AI pipeline
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This project provides a complete automated content pipeline that handles research, content generation, and video creation. It crawls fresh data from sources like TechCrunch, a16z, Twitter/X, and LinkedIn, generates content in multiple formats and languages using Claude/OpenAI, and automatically renders videos using Remotion.

## What It Does

- **Auto-Research**: Crawls real-time data from news sources and social media
- **AI Content Generation**: Creates content in multiple formats (listicles, POV, case studies, how-tos)
- **Multilingual**: Generates Vietnamese and English versions simultaneously
- **Video Rendering**: Converts written content to videos/infographics via Remotion
- **Multi-Platform**: Exports video in formats optimized for Reels, TikTok, Shorts

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
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_claude_key

# Content Research APIs
RAPIDAPI_KEY=your_rapidapi_key
TWITTER_BEARER_TOKEN=your_twitter_token

# Database (if applicable)
DATABASE_URL=your_database_connection

# Remotion (video rendering)
REMOTION_AWS_ACCESS_KEY_ID=your_aws_key
REMOTION_AWS_SECRET_ACCESS_KEY=your_aws_secret
```

## Project Structure

```
marketing-pipeline-share/
├── src/
│   ├── components/      # React/Next.js UI components
│   ├── lib/
│   │   ├── ai/         # AI integration (Claude, OpenAI)
│   │   ├── crawlers/   # Content research crawlers
│   │   ├── generators/ # Content format generators
│   │   └── video/      # Remotion video rendering
│   ├── pages/          # Next.js pages
│   └── remotion/       # Remotion video templates
├── public/             # Static assets
└── HUONG_DAN_CAI_DAT.md # Detailed setup guide (Vietnamese)
```

## Running the Application

```bash
# Development mode
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Render videos with Remotion
npm run remotion
```

## Core API Usage

### 1. Content Research Pipeline

```typescript
import { ResearchCrawler } from '@/lib/crawlers/research';
import { ContentAnalyzer } from '@/lib/ai/analyzer';

// Crawl recent content on a topic
async function researchTopic(keyword: string) {
  const crawler = new ResearchCrawler({
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeframe: '24h'
  });
  
  const rawData = await crawler.crawl(keyword);
  
  // Analyze with AI
  const analyzer = new ContentAnalyzer({
    apiKey: process.env.ANTHROPIC_API_KEY,
    model: 'claude-3-opus-20240229'
  });
  
  const insights = await analyzer.extractInsights(rawData);
  
  return {
    rawData,
    insights,
    statistics: analyzer.getStatistics(rawData)
  };
}
```

### 2. Multi-Format Content Generation

```typescript
import { ContentGenerator } from '@/lib/generators/content';
import { OpenAI } from 'openai';

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY
});

async function generateContent(research: any, format: string) {
  const generator = new ContentGenerator(openai);
  
  // Generate in multiple formats
  const content = await generator.create({
    format: format, // 'toplist' | 'pov' | 'case-study' | 'how-to'
    research: research.insights,
    languages: ['en', 'vi'],
    tone: 'expert', // 'expert' | 'friendly' | 'humorous'
    targetAudience: 'marketers'
  });
  
  return content;
}
```

### 3. AI Content Writing with Claude

```typescript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY
});

async function writeWithClaude(topic: string, insights: any) {
  const message = await anthropic.messages.create({
    model: 'claude-3-sonnet-20240229',
    max_tokens: 4000,
    messages: [{
      role: 'user',
      content: `Write a comprehensive article about ${topic}.
      
Research insights:
${JSON.stringify(insights, null, 2)}

Format: Professional blog post
Tone: Expert but accessible
Languages: Generate both English and Vietnamese versions
Include: Statistics, real examples, actionable tips`
    }]
  });
  
  return message.content;
}
```

### 4. Video Generation with Remotion

```typescript
import { bundle } from '@remotion/bundler';
import { renderMedia, selectComposition } from '@remotion/renderer';
import { VideoTemplate } from '@/remotion/VideoTemplate';

async function generateVideo(content: any) {
  // Bundle Remotion project
  const bundleLocation = await bundle({
    entryPoint: './src/remotion/index.ts',
    webpackOverride: (config) => config
  });
  
  // Select composition
  const composition = await selectComposition({
    serveUrl: bundleLocation,
    id: 'ContentVideo',
    inputProps: {
      title: content.title,
      sections: content.sections,
      language: 'vi',
      platform: 'reels' // 'reels' | 'tiktok' | 'shorts'
    }
  });
  
  // Render video
  await renderMedia({
    composition,
    serveUrl: bundleLocation,
    codec: 'h264',
    outputLocation: `out/${content.slug}.mp4`,
    inputProps: composition.inputProps
  });
  
  return `out/${content.slug}.mp4`;
}
```

## Complete Workflow Example

```typescript
import { ResearchCrawler } from '@/lib/crawlers/research';
import { ContentGenerator } from '@/lib/generators/content';
import { VideoRenderer } from '@/lib/video/renderer';

async function completeContentPipeline(keyword: string) {
  // Step 1: Research
  console.log('🔍 Researching topic...');
  const crawler = new ResearchCrawler();
  const research = await crawler.research(keyword, {
    sources: ['techcrunch', 'twitter'],
    limit: 20
  });
  
  // Step 2: Generate Content
  console.log('✍️ Generating content...');
  const generator = new ContentGenerator({
    aiProvider: 'claude',
    apiKey: process.env.ANTHROPIC_API_KEY
  });
  
  const article = await generator.generate({
    topic: keyword,
    research: research.insights,
    format: 'toplist',
    languages: ['en', 'vi']
  });
  
  // Step 3: Create Video
  console.log('🎬 Rendering video...');
  const videoRenderer = new VideoRenderer();
  const videoPath = await videoRenderer.render({
    content: article,
    template: 'modern-infographic',
    aspectRatio: '9:16', // For Reels/TikTok
    duration: 60 // seconds
  });
  
  return {
    article: article,
    video: videoPath,
    metadata: {
      keyword,
      sources: research.sources.length,
      generatedAt: new Date().toISOString()
    }
  };
}

// Usage
completeContentPipeline('AI automation trends 2024')
  .then(result => console.log('✅ Pipeline complete:', result))
  .catch(err => console.error('❌ Error:', err));
```

## Remotion Video Templates

Create custom video templates in `src/remotion/`:

```typescript
// src/remotion/VideoTemplate.tsx
import { AbsoluteFill, useCurrentFrame, interpolate } from 'remotion';

interface VideoProps {
  title: string;
  sections: Array<{ heading: string; content: string }>;
  language: 'en' | 'vi';
}

export const ContentVideo: React.FC<VideoProps> = ({ title, sections, language }) => {
  const frame = useCurrentFrame();
  
  const opacity = interpolate(frame, [0, 30], [0, 1], {
    extrapolateRight: 'clamp'
  });
  
  return (
    <AbsoluteFill style={{ backgroundColor: '#1a1a1a' }}>
      <div style={{ 
        opacity, 
        padding: 60,
        color: 'white',
        fontSize: 48,
        fontWeight: 'bold'
      }}>
        {title}
      </div>
      
      {sections.map((section, idx) => (
        <Section 
          key={idx}
          heading={section.heading}
          content={section.content}
          startFrame={60 + (idx * 120)}
        />
      ))}
    </AbsoluteFill>
  );
};
```

## API Endpoints (Next.js)

```typescript
// pages/api/generate.ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { ResearchCrawler } from '@/lib/crawlers/research';
import { ContentGenerator } from '@/lib/generators/content';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method not allowed' });
  }
  
  const { keyword, format, languages } = req.body;
  
  try {
    // Research phase
    const crawler = new ResearchCrawler();
    const research = await crawler.research(keyword);
    
    // Generation phase
    const generator = new ContentGenerator({
      aiProvider: 'openai',
      apiKey: process.env.OPENAI_API_KEY
    });
    
    const content = await generator.generate({
      topic: keyword,
      research,
      format,
      languages
    });
    
    res.status(200).json({
      success: true,
      content,
      metadata: {
        sourcesCount: research.sources.length,
        generatedAt: new Date().toISOString()
      }
    });
  } catch (error) {
    res.status(500).json({ 
      error: 'Generation failed',
      message: error.message 
    });
  }
}
```

## Content Format Patterns

### Toplist Format

```typescript
const toplistPrompt = `
Create a toplist article based on this research.

Structure:
1. Engaging introduction
2. List of 5-10 items with:
   - Clear heading
   - Description
   - Real-world example
   - Data/statistics
3. Conclusion with actionable takeaway

Tone: ${tone}
Language: ${language}
`;
```

### Case Study Format

```typescript
const caseStudyPrompt = `
Write a detailed case study.

Include:
- Background & Challenge
- Solution Approach
- Implementation Details
- Results with Metrics
- Key Learnings
- Recommendations

Use real data from: ${research.insights}
`;
```

## Common Patterns

### Chaining Multiple AI Calls

```typescript
async function multiStepGeneration(topic: string) {
  // Step 1: Outline
  const outline = await generateOutline(topic);
  
  // Step 2: Expand each section
  const sections = await Promise.all(
    outline.sections.map(section => 
      expandSection(section, topic)
    )
  );
  
  // Step 3: Generate introduction & conclusion
  const intro = await generateIntro(topic, sections);
  const conclusion = await generateConclusion(sections);
  
  return {
    title: outline.title,
    intro,
    sections,
    conclusion
  };
}
```

### Parallel Language Generation

```typescript
async function generateMultilingual(content: any) {
  const [english, vietnamese] = await Promise.all([
    translate(content, 'en'),
    translate(content, 'vi')
  ]);
  
  return { en: english, vi: vietnamese };
}
```

## Troubleshooting

### API Rate Limits

```typescript
import pLimit from 'p-limit';

// Limit concurrent API calls
const limit = pLimit(3);

const results = await Promise.all(
  items.map(item => 
    limit(() => apiCall(item))
  )
);
```

### Video Rendering Timeout

```typescript
// Increase timeout for long videos
await renderMedia({
  composition,
  serveUrl: bundleLocation,
  timeoutInMilliseconds: 300000, // 5 minutes
  outputLocation: 'out/video.mp4'
});
```

### Memory Issues with Large Crawls

```typescript
// Process in batches
async function crawlInBatches(urls: string[], batchSize = 10) {
  const results = [];
  
  for (let i = 0; i < urls.length; i += batchSize) {
    const batch = urls.slice(i, i + batchSize);
    const batchResults = await Promise.all(
      batch.map(url => crawler.fetch(url))
    );
    results.push(...batchResults);
    
    // Clear memory between batches
    if (global.gc) global.gc();
  }
  
  return results;
}
```

### Claude/OpenAI Error Handling

```typescript
import { RateLimitError, APIError } from '@anthropic-ai/sdk';

async function robustAICall(prompt: string, retries = 3) {
  for (let i = 0; i < retries; i++) {
    try {
      return await anthropic.messages.create({
        model: 'claude-3-sonnet-20240229',
        max_tokens: 4000,
        messages: [{ role: 'user', content: prompt }]
      });
    } catch (error) {
      if (error instanceof RateLimitError && i < retries - 1) {
        await new Promise(resolve => setTimeout(resolve, 2000 * (i + 1)));
        continue;
      }
      throw error;
    }
  }
}
```

## Best Practices

1. **Cache Research Results**: Store crawled data to avoid redundant API calls
2. **Use Streaming**: For long-form content, use streaming responses from Claude/OpenAI
3. **Optimize Video Templates**: Keep Remotion compositions simple for faster rendering
4. **Monitor Costs**: Track API usage across OpenAI, Claude, and RapidAPI
5. **Version Content**: Store generated content with metadata for A/B testing
