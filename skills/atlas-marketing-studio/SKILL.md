---
name: atlas-marketing-studio
description: Self-hostable AI video ad studio for e-commerce with UGC product ads, reference-ad remakes, AI drama ads, and ad skits powered by Atlas Cloud
triggers:
  - generate AI video ads for ecommerce products
  - create UGC product video advertisements
  - remake reference ads with new products
  - build AI drama ads for social commerce
  - generate short ad skits with AI
  - set up Atlas Marketing Studio
  - configure video ad generation workflow
  - implement AI video credit system
---

# Atlas Marketing Studio Skill

> Skill by [ara.so](https://ara.so) — Marketing Skills collection.

Atlas Marketing Studio is a self-hostable AI video ad studio for e-commerce that generates UGC product ads, reference-ad remakes, AI drama ads, and short ad skits. Built on TypeScript/Next.js, it orchestrates multiple AI models through Atlas Cloud API for complete ad production workflows with credit metering, authentication, and payment integration.

## Installation

```bash
git clone https://github.com/AtlasCloudAI/atlas-marketing-studio.git
cd atlas-marketing-studio
npm install
```

### Environment Setup

Create `.env` for Vercel deployment:

```env
ATLASCLOUD_API_KEY=your_atlas_key
NEXTAUTH_SECRET=your_nextauth_secret
NEXTAUTH_URL=http://localhost:3000
GOOGLE_CLIENT_ID=your_google_client_id
GOOGLE_CLIENT_SECRET=your_google_client_secret
PAYMENT_PROVIDER=atlas
DATABASE_URL=your_neon_postgres_url
BLOB_READ_WRITE_TOKEN=your_vercel_blob_token
```

Or `.dev.vars` for Cloudflare deployment (uses D1/R2 bindings from `wrangler.jsonc`):

```env
ATLASCLOUD_API_KEY=your_atlas_key
NEXTAUTH_SECRET=your_nextauth_secret
NEXTAUTH_URL=http://localhost:8788
GOOGLE_CLIENT_ID=your_google_client_id
GOOGLE_CLIENT_SECRET=your_google_client_secret
PAYMENT_PROVIDER=atlas
```

### Database Initialization

For Vercel (Neon Postgres):
```bash
npm run db:push:vercel
```

For Cloudflare (D1):
```bash
npm run db:push:cf
```

## Key Commands

```bash
# Development - Vercel runtime
npm run dev:vercel

# Development - Cloudflare/OpenNext preview
npm run cf:preview

# Deploy to Vercel
npm run build
vercel deploy

# Deploy to Cloudflare Workers
npm run cf:deploy

# Database migrations
npm run db:push:vercel   # Push schema to Neon
npm run db:push:cf       # Push schema to D1
npm run db:studio        # Open Prisma Studio

# Set Cloudflare secrets
wrangler secret put ATLASCLOUD_API_KEY
wrangler secret put NEXTAUTH_SECRET
```

## Core Workflows

### 1. UGC Product Ad Generation

Generates lip-synced UGC ads from product and presenter photos.

**API Route:** `/api/generate-ugc-ad`

```typescript
import { generateUGCAd } from '@/lib/marketing-studio/ugc-generator';

// Generate UGC ad
const result = await generateUGCAd({
  productImageUrl: 'https://example.com/product.jpg',
  presenterImageUrl: 'https://example.com/presenter.jpg',
  productName: 'Wireless Earbuds',
  targetAudience: 'fitness enthusiasts',
  tone: 'enthusiastic',
  duration: 15,
  atlasApiKey: process.env.ATLASCLOUD_API_KEY!,
});

// Result contains:
// - scriptText: Generated ad script
// - videoUrl: Final lip-synced video
// - creditsUsed: Total credits consumed
```

**UI Component Example:**

```typescript
// src/app/marketing-studio/page.tsx
'use client';

import { useState } from 'react';
import { uploadImage } from '@/lib/upload';

export default function MarketingStudioPage() {
  const [productImage, setProductImage] = useState<File | null>(null);
  const [presenterImage, setPresenterImage] = useState<File | null>(null);
  const [productName, setProductName] = useState('');
  const [generating, setGenerating] = useState(false);

  const handleGenerate = async () => {
    setGenerating(true);
    
    const productUrl = await uploadImage(productImage!);
    const presenterUrl = await uploadImage(presenterImage!);
    
    const response = await fetch('/api/generate-ugc-ad', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        productImageUrl: productUrl,
        presenterImageUrl: presenterUrl,
        productName,
      }),
    });
    
    const result = await response.json();
    setGenerating(false);
  };

  return (
    <div>
      <input type="file" onChange={(e) => setProductImage(e.target.files?.[0] || null)} />
      <input type="file" onChange={(e) => setPresenterImage(e.target.files?.[0] || null)} />
      <input value={productName} onChange={(e) => setProductName(e.target.value)} />
      <button onClick={handleGenerate} disabled={generating}>
        {generating ? 'Generating...' : 'Generate UGC Ad'}
      </button>
    </div>
  );
}
```

### 2. Reference Ad Remake

Remakes existing video ads with new products and presenters.

```typescript
// src/lib/atlas.ts - Atlas Cloud client
export async function remakeReferenceAd({
  referenceVideoUrl,
  productImageUrl,
  presenterImageUrl,
  apiKey,
}: {
  referenceVideoUrl: string;
  productImageUrl: string;
  presenterImageUrl: string;
  apiKey: string;
}) {
  const response = await fetch('https://api.atlascloud.ai/v1/video/edit', {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${apiKey}`,
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({
      model: 'gemini-omni-flash/video-edit',
      referenceVideo: referenceVideoUrl,
      replacements: {
        product: productImageUrl,
        presenter: presenterImageUrl,
      },
    }),
  });

  return await response.json();
}
```

### 3. AI Drama Ad Generation

Creates multi-shot story ads from a single topic.

```typescript
// src/lib/drama/drama-generator.ts
import { generateDramaScript } from './script-generator';
import { generateShotVideo } from './shot-generator';

export async function generateDramaAd({
  topic,
  language = 'en',
  shotCount = 4,
  atlasApiKey,
}: {
  topic: string;
  language?: string;
  shotCount?: number;
  atlasApiKey: string;
}) {
  // Generate script
  const script = await generateDramaScript({
    topic,
    language,
    shotCount,
    atlasApiKey,
  });

  // Generate each shot
  const shots = await Promise.all(
    script.shots.map((shot) =>
      generateShotVideo({
        shotDescription: shot.description,
        dialogue: shot.dialogue,
        referenceImage: shot.referenceImage,
        atlasApiKey,
      })
    )
  );

  // Concatenate shots
  const finalVideo = await concatenateVideos(shots.map(s => s.videoUrl));

  return {
    script,
    shots,
    finalVideoUrl: finalVideo,
    creditsUsed: calculateCredits(shots),
  };
}
```

### 4. Ad Skit Generation

Generates two-person comedy skits from product ideas.

```typescript
// src/app/api/generate-ad-skit/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { getServerSession } from 'next-auth';
import { generateSkitScript } from '@/lib/drama/skit-generator';

export async function POST(req: NextRequest) {
  const session = await getServerSession();
  if (!session?.user) {
    return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
  }

  const { productIdea, language } = await req.json();

  // Generate skit script
  const script = await generateSkitScript({
    productIdea,
    language,
    atlasApiKey: process.env.ATLASCLOUD_API_KEY!,
  });

  // Generate character shots
  const characterA = await generateCharacterVideo(script.characterA);
  const characterB = await generateCharacterVideo(script.characterB);

  return NextResponse.json({
    script,
    videoUrl: await mergeDialogueVideos(characterA, characterB, script.dialogue),
  });
}
```

## Credit System

### Video Pricing Calculation

```typescript
// src/lib/video-pricing.ts
const CREDIT_USD = 0.01; // 1 credit = $0.01 USD
const ACCOUNT_MARKUP = 1.0;
const MARGIN = 1.2;

const PER_SECOND_RATES = {
  '480p': 0.005,
  '720p': 0.01,
  '1080p': 0.02,
};

export function calculateVideoCredits({
  durationSeconds,
  resolution = '720p',
  model = 'seedance-2.0',
}: {
  durationSeconds: number;
  resolution?: '480p' | '720p' | '1080p';
  model?: string;
}): number {
  const baseRate = PER_SECOND_RATES[resolution];
  const credits = Math.ceil(
    baseRate * durationSeconds * ACCOUNT_MARKUP * MARGIN / CREDIT_USD
  );
  return credits;
}

// Fixed-cost operations
export const FIXED_CREDITS = {
  imageGeneration: 10,
  scriptGeneration: 5,
  tts: 3,
  lipSync: 15,
};
```

### Deducting Credits

```typescript
// src/lib/credits.ts
import { prisma } from '@/lib/db';

export async function deductCredits(
  userId: string,
  amount: number,
  description: string
) {
  const user = await prisma.user.findUnique({ where: { id: userId } });
  
  if (!user || user.credits < amount) {
    throw new Error('Insufficient credits');
  }

  await prisma.user.update({
    where: { id: userId },
    data: { credits: { decrement: amount } },
  });

  await prisma.creditTransaction.create({
    data: {
      userId,
      amount: -amount,
      description,
      type: 'DEDUCTION',
    },
  });
}
```

## Atlas Cloud Integration

### Main Client Setup

```typescript
// src/lib/atlas.ts
export class AtlasCloudClient {
  private apiKey: string;
  private baseUrl = 'https://api.atlascloud.ai/v1';

  constructor(apiKey: string) {
    this.apiKey = apiKey;
  }

  async generateImage({
    prompt,
    model = 'nano-banana/edit',
    width = 1024,
    height = 1024,
  }: {
    prompt: string;
    model?: string;
    width?: number;
    height?: number;
  }) {
    const response = await fetch(`${this.baseUrl}/image/generate`, {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${this.apiKey}`,
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({ prompt, model, width, height }),
    });

    return await response.json();
  }

  async generateVideo({
    imageUrl,
    prompt,
    duration = 5,
    model = 'seedance-2.0',
  }: {
    imageUrl: string;
    prompt: string;
    duration?: number;
    model?: string;
  }) {
    const response = await fetch(`${this.baseUrl}/video/generate`, {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${this.apiKey}`,
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        model,
        image: imageUrl,
        prompt,
        duration,
      }),
    });

    return await response.json();
  }

  async lipSync({
    videoUrl,
    audioUrl,
  }: {
    videoUrl: string;
    audioUrl: string;
  }) {
    const response = await fetch(`${this.baseUrl}/video/lipsync`, {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${this.apiKey}`,
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        video: videoUrl,
        audio: audioUrl,
      }),
    });

    return await response.json();
  }
}
```

### Using the Client

```typescript
// src/app/api/generate-ugc-ad/route.ts
import { AtlasCloudClient } from '@/lib/atlas';

const atlas = new AtlasCloudClient(process.env.ATLASCLOUD_API_KEY!);

// Generate first frame
const firstFrame = await atlas.generateImage({
  prompt: `Product photo: ${productName} with presenter`,
  model: 'nano-banana/edit',
});

// Generate video from image
const video = await atlas.generateVideo({
  imageUrl: firstFrame.imageUrl,
  prompt: 'Presenter enthusiastically presenting product',
  duration: 15,
  model: 'seedance-2.0',
});

// Generate TTS
const audio = await atlas.generateTTS({
  text: scriptText,
  voice: 'en-US-Standard-A',
});

// Lip sync final video
const finalVideo = await atlas.lipSync({
  videoUrl: video.videoUrl,
  audioUrl: audio.audioUrl,
});
```

## Payment Integration

### Stripe Checkout

```typescript
// src/lib/payments/stripe-checkout.ts
import Stripe from 'stripe';

const stripe = new Stripe(process.env.STRIPE_SECRET_KEY!, {
  apiVersion: '2023-10-16',
});

export async function createCheckoutSession({
  userId,
  creditPackId,
  successUrl,
  cancelUrl,
}: {
  userId: string;
  creditPackId: string;
  successUrl: string;
  cancelUrl: string;
}) {
  const pack = CREDIT_PACKS[creditPackId];
  
  const session = await stripe.checkout.sessions.create({
    mode: 'payment',
    line_items: [
      {
        price_data: {
          currency: 'usd',
          product_data: {
            name: pack.name,
            description: `${pack.credits} video generation credits`,
          },
          unit_amount: pack.priceUSD * 100,
        },
        quantity: 1,
      },
    ],
    success_url: successUrl,
    cancel_url: cancelUrl,
    metadata: {
      userId,
      creditPackId,
      credits: pack.credits,
    },
  });

  return session;
}
```

### Redeem Code System

```typescript
// src/app/api/redeem/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { getServerSession } from 'next-auth';
import { prisma } from '@/lib/db';

export async function POST(req: NextRequest) {
  const session = await getServerSession();
  if (!session?.user?.id) {
    return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
  }

  const { code } = await req.json();

  const redeemCode = await prisma.redeemCode.findUnique({
    where: { code },
  });

  if (!redeemCode || redeemCode.used || redeemCode.expiresAt < new Date()) {
    return NextResponse.json({ error: 'Invalid or expired code' }, { status: 400 });
  }

  // Mark code as used
  await prisma.redeemCode.update({
    where: { id: redeemCode.id },
    data: {
      used: true,
      usedAt: new Date(),
      usedBy: session.user.id,
    },
  });

  // Add credits to user
  await prisma.user.update({
    where: { id: session.user.id },
    data: {
      credits: { increment: redeemCode.credits },
    },
  });

  return NextResponse.json({ credits: redeemCode.credits });
}
```

## Database Schema

```prisma
// prisma/schema.prisma
model User {
  id            String    @id @default(cuid())
  email         String    @unique
  name          String?
  image         String?
  credits       Int       @default(100)
  accounts      Account[]
  sessions      Session[]
  creations     Creation[]
  transactions  CreditTransaction[]
  createdAt     DateTime  @default(now())
  updatedAt     DateTime  @updatedAt
}

model Creation {
  id            String   @id @default(cuid())
  userId        String
  user          User     @relation(fields: [userId], references: [id], onDelete: Cascade)
  type          String   // 'ugc-ad', 'reference-ad', 'drama-ad', 'ad-skit'
  status        String   // 'processing', 'completed', 'failed'
  inputData     String   // JSON blob
  outputData    String?  // JSON blob
  videoUrl      String?
  creditsUsed   Int
  createdAt     DateTime @default(now())
  updatedAt     DateTime @updatedAt
}

model CreditTransaction {
  id          String   @id @default(cuid())
  userId      String
  user        User     @relation(fields: [userId], references: [id], onDelete: Cascade)
  amount      Int
  type        String   // 'PURCHASE', 'DEDUCTION', 'REFUND', 'REDEEM'
  description String
  createdAt   DateTime @default(now())
}

model RedeemCode {
  id        String    @id @default(cuid())
  code      String    @unique
  credits   Int
  used      Boolean   @default(false)
  usedAt    DateTime?
  usedBy    String?
  expiresAt DateTime
  createdAt DateTime  @default(now())
}
```

## Configuration

### Pricing Configuration

```typescript
// src/config/pricing.ts
export const CREDIT_PACKS = {
  starter: {
    id: 'starter',
    name: 'Starter Pack',
    credits: 500,
    priceUSD: 5,
    bonusPercent: 0,
  },
  pro: {
    id: 'pro',
    name: 'Pro Pack',
    credits: 2000,
    priceUSD: 18,
    bonusPercent: 10,
  },
  business: {
    id: 'business',
    name: 'Business Pack',
    credits: 5000,
    priceUSD: 40,
    bonusPercent: 20,
  },
};
```

### Platform Adapters

```typescript
// src/lib/db.ts
import { PrismaClient } from '@prisma/client';

const globalForPrisma = global as unknown as { prisma: PrismaClient };

export const prisma =
  globalForPrisma.prisma ||
  new PrismaClient({
    log: process.env.NODE_ENV === 'development' ? ['query', 'error', 'warn'] : ['error'],
  });

if (process.env.NODE_ENV !== 'production') globalForPrisma.prisma = prisma;

// Cloudflare D1 uses @prisma/adapter-d1
// Vercel/Neon uses standard Prisma Client
```

## Media Storage

### Upload Pattern

```typescript
// src/lib/upload.ts
export async function uploadImage(file: File): Promise<string> {
  if (process.env.VERCEL) {
    // Vercel Blob (public)
    const { put } = await import('@vercel/blob');
    const blob = await put(file.name, file, {
      access: 'public',
      token: process.env.BLOB_READ_WRITE_TOKEN,
    });
    return blob.url;
  } else {
    // Cloudflare R2
    const key = `uploads/${Date.now()}-${file.name}`;
    const arrayBuffer = await file.arrayBuffer();
    
    await env.MEDIA_BUCKET.put(key, arrayBuffer, {
      httpMetadata: {
        contentType: file.type,
      },
    });
    
    return `${env.R2_PUBLIC_URL}/${key}`;
  }
}
```

## Common Patterns

### Workflow with Credit Check

```typescript
export async function generateWithCreditsCheck({
  userId,
  workflowFn,
  estimatedCredits,
}: {
  userId: string;
  workflowFn: () => Promise<{ videoUrl: string; creditsUsed: number }>;
  estimatedCredits: number;
}) {
  // Pre-check credits
  const user = await prisma.user.findUnique({ where: { id: userId } });
  if (!user || user.credits < estimatedCredits) {
    throw new Error('Insufficient credits');
  }

  // Create creation record
  const creation = await prisma.creation.create({
    data: {
      userId,
      type: 'ugc-ad',
      status: 'processing',
      inputData: JSON.stringify({}),
      creditsUsed: 0,
    },
  });

  try {
    // Execute workflow
    const result = await workflowFn();

    // Deduct actual credits
    await deductCredits(userId, result.creditsUsed, 'UGC Ad Generation');

    // Update creation
    await prisma.creation.update({
      where: { id: creation.id },
      data: {
        status: 'completed',
        videoUrl: result.videoUrl,
        creditsUsed: result.creditsUsed,
      },
    });

    return result;
  } catch (error) {
    await prisma.creation.update({
      where: { id: creation.id },
      data: { status: 'failed' },
    });
    throw error;
  }
}
```

### Multi-Language Script Generation

```typescript
// src/lib/marketing-studio/script-generator.ts
export async function generateAdScript({
  productName,
  targetAudience,
  tone,
  language = 'en',
  atlasApiKey,
}: {
  productName: string;
  targetAudience: string;
  tone: string;
  language?: string;
  atlasApiKey: string;
}) {
  const prompt = `Generate a ${tone} UGC ad script for "${productName}" targeting ${targetAudience}. 
Language: ${language}. 
Format: 15-second spoken script, first-person perspective.`;

  const response = await fetch('https://api.atlascloud.ai/v1/llm/generate', {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${atlasApiKey}`,
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({
      model: 'gpt-4o-mini',
      prompt,
      maxTokens: 200,
    }),
  });

  const result = await response.json();
  return result.text;
}
```

## Troubleshooting

### Issue: "Insufficient credits" during generation

**Solution:** Check actual credit balance and increase estimated credits buffer:

```typescript
const estimatedCredits = calculateVideoCredits({
  durationSeconds: 15,
  resolution: '720p',
}) * 1.2; // 20% buffer for processing overhead
```

### Issue: Atlas Cloud API timeout

**Solution:** Implement retry logic with exponential backoff:

```typescript
async function callAtlasWithRetry(fn: () => Promise<any>, maxRetries = 3) {
  for (let i = 0; i < maxRetries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (i === maxRetries - 1) throw error;
      await new Promise(resolve => setTimeout(resolve, 1000 * Math.pow(2, i)));
    }
  }
}
```

### Issue: Video URL not accessible by Atlas Cloud

**Solution:** Ensure media storage is publicly accessible:

```typescript
// Cloudflare R2 - set public bucket or use signed URLs
const signedUrl = await env.MEDIA_BUCKET.createSignedUrl(key, {
  expiresIn: 3600,
});

// Vercel Blob - use 'public' access mode
const blob = await put(file.name, file, {
  access: 'public', // Required for Atlas Cloud to fetch
});
```

### Issue: Prisma schema mismatch after deployment

**Solution:** Push schema changes before deploying:

```bash
# Vercel
npm run db:push:vercel
vercel deploy

# Cloudflare
npm run db:push:cf
npm run cf:deploy
```

### Issue: NextAuth session not persisting on Cloudflare

**Solution:** Verify environment variables and session adapter:

```typescript
// src/app/api/auth/[...nextauth]/route.ts
export const authOptions: NextAuthOptions = {
  adapter: PrismaAdapter(prisma),
  session: {
    strategy: 'database', // Required for Cloudflare D1
    maxAge: 30 * 24 * 60 * 60, // 30 days
  },
  // ... rest of config
};
```
