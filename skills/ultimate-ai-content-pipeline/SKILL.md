---
name: ultimate-ai-content-pipeline
description: Automated content creation pipeline with AI research, script generation, auto-posting, and video rendering using Claude, OpenAI, and Remotion
triggers:
  - automate content creation with AI research and video generation
  - set up AI content pipeline for social media
  - generate videos from text content automatically
  - create content from trending news and research
  - build automated marketing content workflow
  - configure AI content generation with Claude and OpenAI
  - render video content with Remotion from AI scripts
  - auto-post AI generated content to social platforms
---

# Ultimate AI Content Pipeline

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

This skill enables AI coding agents to help developers use the Ultimate AI Content Pipeline - a comprehensive automated content creation system that handles everything from research and scriptwriting to video generation and auto-posting. The pipeline integrates Claude 3, OpenAI, web scraping, and Remotion video rendering.

## What It Does

The Ultimate AI Content Pipeline automates the entire content creation workflow:

1. **Auto-Research**: Crawls recent news from TechCrunch, a16z, Twitter/X, LinkedIn
2. **AI Content Generation**: Creates articles in multiple formats (lists, POV, case studies, how-to) using Claude/OpenAI
3. **Multi-language Support**: Generates content in both English and Vietnamese
4. **Video Rendering**: Automatically converts text content to videos using Remotion
5. **Auto-Publishing**: Posts content to social media platforms automatically

## Installation

### Prerequisites

```bash
node >= 18.x
npm or yarn
```

### Clone and Setup

```bash
git clone https://github.com/pennydinh/marketing-pineline-share.git
cd marketing-pineline-share

# Install dependencies
npm install
# or
yarn install
```

### Environment Configuration

Create a `.env.local` file in the root directory:

```env
# AI API Keys
ANTHROPIC_API_KEY=your_claude_api_key_here
OPENAI_API_KEY=your_openai_api_key_here

# Research/Scraping APIs
RAPIDAPI_KEY=your_rapidapi_key_here

# Social Media APIs (for auto-posting)
FACEBOOK_ACCESS_TOKEN=your_facebook_token_here
TWITTER_API_KEY=your_twitter_key_here

# Remotion Configuration
REMOTION_RENDERER_URL=http://localhost:3001

# Application Settings
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

### Run Development Server

```bash
npm run dev
# or
yarn dev
```

Access the application at `http://localhost:3000`

## Project Structure

```
marketing-pineline-share/
├── src/
│   ├── app/              # Next.js app router pages
│   ├── components/       # React components
│   ├── lib/
│   │   ├── ai/          # AI integration (Claude, OpenAI)
│   │   ├── scraper/     # Web scraping utilities
│   │   ├── video/       # Remotion video rendering
│   │   └── publisher/   # Social media auto-posting
│   ├── types/           # TypeScript type definitions
│   └── utils/           # Helper functions
├── remotion/            # Remotion video templates
└── public/              # Static assets
```

## Core Components and APIs

### 1. Content Research Module

```typescript
import { scrapeNewsArticles } from '@/lib/scraper/news-scraper';
import { analyzeContent } from '@/lib/ai/content-analyzer';

// Fetch recent trending news
async function researchTopic(keyword: string) {
  const articles = await scrapeNewsArticles({
    keyword,
    sources: ['techcrunch', 'a16z', 'twitter', 'linkedin'],
    timeframe: '24h'
  });

  // AI-powered analysis of scraped content
  const insights = await analyzeContent(articles, {
    provider: 'claude', // or 'openai'
    model: 'claude-3-opus-20240229'
  });

  return {
    articles,
    insights,
    keyPoints: insights.keyPoints,
    trends: insights.trends
  };
}
```

### 2. AI Content Generation

```typescript
import { generateContent } from '@/lib/ai/content-generator';

// Generate content in multiple formats
async function createArticle(topic: string, research: any) {
  const content = await generateContent({
    topic,
    research,
    format: 'toplist', // 'pov' | 'case-study' | 'how-to' | 'toplist'
    language: 'both', // 'en' | 'vi' | 'both'
    tone: 'professional', // 'friendly' | 'expert' | 'humorous'
    provider: 'claude', // or 'openai'
    apiKey: process.env.ANTHROPIC_API_KEY
  });

  return {
    english: content.en,
    vietnamese: content.vi,
    metadata: content.metadata,
    seoOptimized: content.seo
  };
}

// Example: Generate POV article with Claude
async function generatePOVArticle(insights: any) {
  const prompt = `Based on these insights: ${JSON.stringify(insights)}
  
  Create a POV (Point of View) article that:
  1. Presents a unique perspective on the trend
  2. Includes data-backed arguments
  3. Engages the reader with a conversational tone
  4. Optimized for social media sharing`;

  const response = await fetch('https://api.anthropic.com/v1/messages', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'x-api-key': process.env.ANTHROPIC_API_KEY!,
      'anthropic-version': '2023-06-01'
    },
    body: JSON.stringify({
      model: 'claude-3-opus-20240229',
      max_tokens: 4096,
      messages: [{
        role: 'user',
        content: prompt
      }]
    })
  });

  const data = await response.json();
  return data.content[0].text;
}
```

### 3. Video Rendering with Remotion

```typescript
import { renderVideo } from '@/lib/video/renderer';
import { ContentToVideoTemplate } from '@/remotion/templates/ContentTemplate';

// Render video from article content
async function generateVideoFromContent(article: any) {
  const videoConfig = {
    template: 'content-to-video',
    composition: 'MainComposition',
    inputProps: {
      title: article.title,
      keyPoints: article.keyPoints,
      stats: article.statistics,
      branding: {
        logo: '/logo.png',
        colors: {
          primary: '#4F46E5',
          secondary: '#10B981'
        }
      }
    },
    outputFormat: {
      width: 1080,
      height: 1920, // 9:16 for Reels/TikTok/Shorts
      fps: 30,
      codec: 'h264'
    }
  };

  const videoUrl = await renderVideo(videoConfig);
  
  return {
    url: videoUrl,
    duration: videoConfig.inputProps.keyPoints.length * 3, // 3s per point
    platform: 'reels' // 'tiktok' | 'shorts' | 'reels'
  };
}

// Custom Remotion composition example
// File: remotion/templates/ContentTemplate.tsx
import { AbsoluteFill, Sequence, useCurrentFrame } from 'remotion';

export const ContentToVideoTemplate: React.FC<{
  title: string;
  keyPoints: string[];
  stats: any;
}> = ({ title, keyPoints, stats }) => {
  const frame = useCurrentFrame();

  return (
    <AbsoluteFill style={{ backgroundColor: '#1F2937' }}>
      <Sequence from={0} durationInFrames={90}>
        <div style={{ 
          fontSize: 60, 
          color: 'white',
          padding: 40,
          fontWeight: 'bold'
        }}>
          {title}
        </div>
      </Sequence>
      
      {keyPoints.map((point, index) => (
        <Sequence 
          key={index}
          from={90 + (index * 90)} 
          durationInFrames={90}
        >
          <div style={{ padding: 40, color: 'white', fontSize: 40 }}>
            {point}
          </div>
        </Sequence>
      ))}
    </AbsoluteFill>
  );
};
```

### 4. Auto-Publishing to Social Media

```typescript
import { publishToFacebook } from '@/lib/publisher/facebook';
import { publishToTwitter } from '@/lib/publisher/twitter';

// Publish content and video to multiple platforms
async function autoPublish(content: any, video: any) {
  const results = await Promise.allSettled([
    // Facebook/Instagram
    publishToFacebook({
      message: content.english.excerpt,
      videoUrl: video.url,
      accessToken: process.env.FACEBOOK_ACCESS_TOKEN!,
      pageId: process.env.FACEBOOK_PAGE_ID!,
      targetPlatform: 'instagram_reels' // or 'facebook_reel'
    }),
    
    // Twitter/X
    publishToTwitter({
      text: content.english.excerpt,
      mediaUrl: video.url,
      apiKey: process.env.TWITTER_API_KEY!,
      apiSecret: process.env.TWITTER_API_SECRET!
    })
  ]);

  return results.map((result, index) => ({
    platform: ['facebook', 'twitter'][index],
    status: result.status,
    data: result.status === 'fulfilled' ? result.value : null,
    error: result.status === 'rejected' ? result.reason : null
  }));
}
```

## Complete Workflow Example

```typescript
import { runContentPipeline } from '@/lib/pipeline';

// Execute the full content pipeline
async function executeFullPipeline(keyword: string) {
  try {
    // Step 1: Research
    console.log('🔍 Starting research...');
    const research = await researchTopic(keyword);
    
    // Step 2: Generate Content
    console.log('✍️ Generating content...');
    const article = await createArticle(keyword, research);
    
    // Step 3: Render Video
    console.log('🎬 Rendering video...');
    const video = await generateVideoFromContent(article);
    
    // Step 4: Publish
    console.log('📤 Publishing to platforms...');
    const publishResults = await autoPublish(article, video);
    
    return {
      success: true,
      research,
      article,
      video,
      publishResults
    };
  } catch (error) {
    console.error('Pipeline error:', error);
    throw error;
  }
}

// Usage
executeFullPipeline('AI automation trends 2024')
  .then(result => {
    console.log('✅ Pipeline completed successfully!');
    console.log('Published to:', result.publishResults);
  })
  .catch(error => {
    console.error('❌ Pipeline failed:', error);
  });
```

## API Routes (Next.js)

### Create Content Endpoint

```typescript
// File: src/app/api/content/create/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { runContentPipeline } from '@/lib/pipeline';

export async function POST(request: NextRequest) {
  try {
    const { keyword, format, language, autoPublish } = await request.json();
    
    // Validate input
    if (!keyword) {
      return NextResponse.json(
        { error: 'Keyword is required' },
        { status: 400 }
      );
    }
    
    // Run pipeline
    const result = await runContentPipeline({
      keyword,
      format: format || 'toplist',
      language: language || 'both',
      autoPublish: autoPublish !== false
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

### Render Video Endpoint

```typescript
// File: src/app/api/video/render/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { renderVideo } from '@/lib/video/renderer';

export async function POST(request: NextRequest) {
  try {
    const { articleId, platform } = await request.json();
    
    const videoConfig = {
      articleId,
      platform: platform || 'reels',
      quality: 'high'
    };
    
    const video = await renderVideo(videoConfig);
    
    return NextResponse.json({ 
      success: true, 
      videoUrl: video.url 
    });
  } catch (error: any) {
    return NextResponse.json(
      { error: error.message },
      { status: 500 }
    );
  }
}
```

## Configuration Patterns

### Custom Content Formats

```typescript
// File: src/config/content-formats.ts
export const contentFormats = {
  toplist: {
    structure: 'numbered_list',
    minItems: 5,
    maxItems: 10,
    includeIntro: true,
    includeConclusion: true,
    optimizeFor: 'engagement'
  },
  pov: {
    structure: 'opinion_piece',
    tone: 'conversational',
    includeDataPoints: true,
    wordCount: { min: 800, max: 1200 }
  },
  caseStudy: {
    structure: 'problem_solution_results',
    includeMetrics: true,
    includeQuotes: true,
    wordCount: { min: 1000, max: 1500 }
  },
  howTo: {
    structure: 'step_by_step',
    includeImages: true,
    difficulty: 'beginner', // 'beginner' | 'intermediate' | 'advanced'
    estimatedTime: true
  }
};
```

### AI Provider Configuration

```typescript
// File: src/config/ai-providers.ts
export const aiProviders = {
  claude: {
    models: {
      fast: 'claude-3-haiku-20240307',
      balanced: 'claude-3-sonnet-20240229',
      powerful: 'claude-3-opus-20240229'
    },
    maxTokens: 4096,
    temperature: 0.7
  },
  openai: {
    models: {
      fast: 'gpt-3.5-turbo',
      balanced: 'gpt-4',
      powerful: 'gpt-4-turbo-preview'
    },
    maxTokens: 4096,
    temperature: 0.7
  }
};

// Usage in code
import { aiProviders } from '@/config/ai-providers';

const model = aiProviders.claude.models.powerful;
```

## Common Patterns

### Batching Content Creation

```typescript
// Create multiple pieces of content from different keywords
async function batchCreateContent(keywords: string[]) {
  const results = await Promise.allSettled(
    keywords.map(keyword => executeFullPipeline(keyword))
  );
  
  return results.map((result, index) => ({
    keyword: keywords[index],
    success: result.status === 'fulfilled',
    data: result.status === 'fulfilled' ? result.value : null,
    error: result.status === 'rejected' ? result.reason : null
  }));
}

// Usage
const keywords = [
  'AI automation 2024',
  'Marketing trends',
  'Content creation tools'
];

batchCreateContent(keywords);
```

### Scheduling Content

```typescript
// Schedule content for future publishing
import { schedulePipeline } from '@/lib/scheduler';

async function scheduleContentCreation(
  keyword: string,
  publishDate: Date
) {
  return await schedulePipeline({
    keyword,
    publishDate,
    autoPublish: true,
    platforms: ['facebook', 'twitter', 'linkedin']
  });
}
```

### Custom Video Templates

```typescript
// Register custom Remotion template
import { registerTemplate } from '@/lib/video/template-registry';

registerTemplate('custom-intro', {
  component: CustomIntroTemplate,
  defaultProps: {
    duration: 5,
    style: 'modern'
  },
  aspectRatios: ['9:16', '16:9', '1:1']
});
```

## Troubleshooting

### API Rate Limits

```typescript
// Implement retry logic with exponential backoff
async function callAIWithRetry(prompt: string, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await generateContent({ prompt });
    } catch (error: any) {
      if (error.status === 429 && i < maxRetries - 1) {
        const delay = Math.pow(2, i) * 1000; // Exponential backoff
        await new Promise(resolve => setTimeout(resolve, delay));
        continue;
      }
      throw error;
    }
  }
}
```

### Video Rendering Memory Issues

```typescript
// For large videos, use chunked rendering
import { renderMediaOnLambda } from '@remotion/lambda';

async function renderLargeVideo(config: any) {
  // Use Remotion Lambda for heavy rendering tasks
  const result = await renderMediaOnLambda({
    region: 'us-east-1',
    functionName: 'remotion-render',
    serveUrl: process.env.REMOTION_SERVE_URL!,
    composition: config.composition,
    inputProps: config.inputProps,
    codec: 'h264',
    privacy: 'public'
  });
  
  return result.url;
}
```

### Scraping Failures

```typescript
// Handle failed scrapes gracefully
import { scrapeWithFallback } from '@/lib/scraper/fallback';

async function reliableScrape(url: string) {
  try {
    return await scrapeNewsArticles({ url });
  } catch (error) {
    console.warn('Primary scraper failed, using fallback');
    return await scrapeWithFallback(url, {
      method: 'api', // Use RapidAPI as fallback
      apiKey: process.env.RAPIDAPI_KEY
    });
  }
}
```

### Environment Variables Not Loading

Ensure `.env.local` is in the root directory and restart the development server:

```bash
# Clear Next.js cache
rm -rf .next

# Restart dev server
npm run dev
```

## CLI Commands

```bash
# Development
npm run dev              # Start development server
npm run build            # Build for production
npm run start            # Start production server

# Remotion
npm run remotion:dev     # Open Remotion studio
npm run remotion:render  # Render video from CLI

# Testing
npm run test             # Run tests
npm run lint             # Lint code
```

## Best Practices

1. **Always validate API keys** before running the pipeline
2. **Cache research data** to avoid redundant scraping
3. **Use queue systems** for batch content creation
4. **Monitor API usage** to stay within rate limits
5. **Test video templates** in Remotion studio before rendering
6. **Version control your content formats** for consistency
7. **Implement proper error logging** for debugging

This skill provides comprehensive coverage of the Ultimate AI Content Pipeline, enabling AI agents to assist developers in building automated content workflows from research through video generation and publishing.
